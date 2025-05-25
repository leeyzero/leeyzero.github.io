---
title: 记一次服务hang死故障排查
date: 2021-04-24 12:13:02
categories:
- 编程匠艺
tags:
- 故障排查
- 服务hang死
---

之前排查了一个服务hang死的线上故障，觉得这个问题比较有代表性，本文记录故障排查的经过，并做一些总结和思考。

## 现象

周末线上一个比较核心的服务（以下称为X服务）出现大量5xx报警。从客户端看到的现象是客户端请求X服务时，出现大量的链接被重置（connection reset by peer）的错误。

## 分析

从经验来看，90%以上的问题都是上线引入的。故障出现在周末，按规范来说，非特殊情况下，周末是不允许上线，跟相关角色确认后，本次故障不是上线引入。

观察线上流量，在出现故障时间点，流量的确有大幅上涨，从宏观分析来看，有可能流量突然上涨是一个诱因。


任意找了一台有问题的机器，发现了一个比较有意思的现象：**服务X出现了大量CLOSE_WAIT状态的连接。从TCP的状态转换可知，CLOSE_WAIT是被动关闭一方在收到FIN并返回ACK后进入的状态**。TCP状态机如下：

![img](/images/trouble-shooting-for-service-hang/tcpfsm.png)

也就是说X服务在收到客户端请求后，客户端关闭连接，还服务端进程并没有关闭连接，分析了web server的框架代码，正常情况下，一个请求处理完后会关闭这个连接，**出现这种情况可能是业务逻辑出现阻塞，导致服务端请求的连接一直不能关闭**。


查看了一下X服务打开的socket句柄数:

```shell
$ lsof | grep `ps -ef | grep x-service | grep -v 'grep' | awk '{print $2}'` | grep sock | wc -l
10231
```

发现X服务打开的连接数非常多，达到10231个，看下实例的句柄数上限：

```shell
$ unlimit -n
10240
```

句柄数像是已经被耗尽了，用strace看了下X服务的系统调用：

```shell
$ strace -p `ps -ef | grep x-service | grep -v 'grep' | awk '{print $2}'`
```

从系统调用日志中发现了一些关键信息：

![](/images/trouble-shooting-for-service-hang/strace.png)

发现大量accept调用报错，从日志信息可以看出，的确是系统的文件句柄数（FD）耗尽了，导致连接被reset掉，这也能说明为什么客户端侧看到了大量`connection reset by peer`错误了。

X服务打开了大量socket文件句柄，且绝大多数连接状态都是CLOSE_WAIT，如果业务逻辑有阻塞的确可能会出现把句柄数数打满。

分析了一下业务逻辑，业务逻辑本身不复杂，只有一次IO网络请求，是从数据库查询数据，从日志中发现了一些端倪：

<!--more-->

```
....
cost:941.583774ms dbstats:{60 60 60 0 25 1.13992ms 6 0}
```

其中dbstats是mysql 数据库连接当前的一些统计信息，其结构DBStats定义在database/sql/sql.go中，如下：

```
// DBStats contains database statistics.
type DBStats struct {
    MaxOpenConnections int // Maximum number of open connections to the database.

    // Pool Status
    OpenConnections int // The number of established connections both in use and idle.
    InUse           int // The number of connections currently in use.
    Idle            int // The number of idle connections.

    // Counters
    WaitCount         int64         // The total number of connections waited for.
    WaitDuration      time.Duration // The total time blocked waiting for a new connection.
    MaxIdleClosed     int64         // The total number of connections closed due to SetMaxIdleConns.
    MaxIdleTimeClosed int64         // The total number of connections closed due to SetConnMaxIdleTime.
    MaxLifetimeClosed int64         // The total number of connections closed due to SetConnMaxLifetime.
}
```

这种需要强调一下，golang实现的mysql连接驱动实现了连接池，可以设置连接池的大小等信息，Idle连接大小等。从这个日志信息中可以看出，业务逻辑在创建设置了连接池大小限制（MaxOpenConnections=60），处理请求时Idle的连接数为0，InUse的连接数已经达到连接池上限。这个请求耗时近1s，还有25个连接在等待处理。

初步分析了一下，即便是流量过大，导致mysql连接池中的连接被用完，连接使用完后，也应该得到正常处理。且目前的流量已经下来了，但问题依然存在。仔细分析了一下golang对mysql连接池的实现，找到了连接池用完时的处理逻辑，代码在database/sql/sql.go，DB.conn函数中，如下：

```golang
...
 // Out of free connections or we were asked not to use one. If we're not
 // allowed to open any more connections, make a request and wait.
 if db.maxOpen > 0 && db.numOpen >= db.maxOpen {
     // Make the connRequest channel. It's buffered so that the
     // connectionOpener doesn't block while waiting for the req to be read.
     req := make(chan connRequest, 1)
     reqKey := db.nextRequestKeyLocked()
     db.connRequests[reqKey] = req
     db.waitCount++
     db.mu.Unlock()
...
```

当连接池中已打开的连接数超过设置的连接池上限时，会有一个阻塞操作，等待其它连接用完后放入连接池。这下问题就比较清晰了：

**mysql打开一个连接时也是要占用fd资源的，当TCP服务器打开了大量的FD而没有释放，mysql连接池也在不断处理新的请求，当FD用完的时候，TCP服务器和mysql连接池都在等待对方释放FD资源来处理新的请求，导致双方均在等待对方释放FD，导致死锁。**

问题修复就比较简单了，mysql连接池不设置上限，连接数瓶颈交给数据库代理层去解决。


## 复现

为了在线下复现这种场景，我写一个简单服务端和客户端程序。mysql连接池的最大连接数设置的比较小（10个），在处理请求时打开一个mysql连接，同时sleep比较长的时间（600秒），这样就模拟了一个请求处理耗时比较长，导致TCP连接和数据库连接都得不到释放的情况。简单客户端程序开多线程高并发请该服务。

一开始出现大量io time，当服务器上的FD打满后，开始出现connection reset by peer。由于客户端本地句柄数也有限制，多次运行客户端程序后开始稳定出现connection reset by peer的情况，服务端开始稳定出现open too many files; retrying in 1s错误。

**注：以下代码只为复现问题，忽略一些错误处理，切勿用到生产环境**

### 模拟服务端

```golang
package main

import (
    "database/sql"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "time"
    _ "github.com/go-sql-driver/mysql"
)

var db *sql.DB

func dbInit() {
  	db, _ = sql.Open("mysql", "{user}:{pass}@tcp({host}:{port})/{database}?charset=utf8")
    db.SetMaxOpenConns(10)
    db.SetMaxIdleConns(2)
    db.Ping()
}

func dbStatsHandler(w http.ResponseWriter, r *http.Request) {
    s := db.Stats()
    fmt.Fprintf(w, "%+v", s)
}

func dbQueryHandler(w http.ResponseWriter, r *http.Request) {
    start := time.Now()
  	rows, err := db.Query("select {fields} from {table}")
    if err != nil {
      	log.Printf("db.Query fail.err:%v", err)
        return
    }
    defer rows.Close()
    time.Sleep(600 * time.Second) // 模拟请求长时间处理不完
    fmt.Fprintf(w, "cost:%s", time.Since(start).String())
}

func main() {
    dbInit()
    http.HandleFunc("/test/db/query", dbQueryHandler)
    http.HandleFunc("/test/db/stats", dbStatsHandler)
    log.Fatal(http.ListenAndServe(":8000", nil))
}

```


### 模拟客户端

```golang
package main

import (
    "net"
    "net/http"
    "time"
    "log"
    "sync"
    "fmt"
)

const (
    ConnectTimeoutMs = 500
    ReadWriteTimeoutMs = 1000
)

func dbQuery() {
    tr := &http.Transport{
        Dial: func(netw, addr string) (net.Conn, error) {
            conn, err := net.DialTimeout(netw, addr, ConnectTimeoutMs*time.Millisecond)
            if err != nil {
                return nil, err
            }
            conn.SetDeadline(time.Now().Add(ReadWriteTimeoutMs * time.Millisecond))
            return conn, nil
        },
    }
    cli := &http.Client{Transport: tr}
    res, err := cli.Get("http://localhost:8000/test/db/query")
    if err != nil {
        log.Printf("http GET fail.err:%v", err)
        return
    }
    defer res.Body.Close()
    log.Println("http GET succ.")
}

func main() {
    var wg sync.WaitGroup
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func() { // 开协程高并发请求
            defer wg.Done()
            for j := 0; j < 10000; j++ {
                dbQuery()
                time.Sleep(10 * time.Millisecond)
            }
        }()
    }
    wg.Wait()
}
```

## 总结

database/sql实现了mysql连接池，如果设置了最大连接数。需要注意当前正在使用的连接数大于最大连接数时，新的连接请求只有等待连接池中的连接被释放后才能得到处理。database/sql提供了连接池当前的使用情况的一些计数信息，这些信息非常有参考价值，建议依赖mysql高并发的业务打印到日志，以便用于排查问题和调优。

生产环境中的网络问题十分复杂，但如果你知道网络的一些基本原理，至少在主线上不会走偏。FD也是一种系统资源，同样会造成资源竞争问题。

线上故障排查充分体现了基础知识的重要性。每年都有很多新技术出来，但计算机基础技术却很少有重大变革。很多新技术都是在现有技术基础上搭建起来的，身处在技术的海洋中，要把握住主线，不要舍本逐末，捡了芝麻丢了西瓜。
