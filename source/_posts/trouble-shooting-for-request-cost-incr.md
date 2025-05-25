---
title: 记一次接口耗时上涨故障排查
date: 2021-04-18 23:02
categories:
- 编程匠艺
tag:
- 故障排查
- 接口耗时
---

最近排查一个线上接口耗时上涨的问题，其中用到排查思路和相关工具有一定借鉴作用，在此做个记录,同时供网友参考。由于涉及到一些敏感信息，所以本文会对一些名称进行模糊化。

## 现象

监控发现对外接口的整体耗时上涨了近100ms，涨了近50%，甚至有少部分接口出现超时现象，已经影响到部分业务的用户体验。

## 分析

### 宏观分析

对比故障发生的时候点，检查对应时间结点是否内部服务有上线，通常情况下，90%以上的故障都是由上线引起的。但排查后，业务侧并未上过线，排除是业务上线引入的故障。

对比不同机房，服务部署在三个机房，记为机房A，机房B和机房C，发现耗时主要发生在机房A，机房B和机房C并不存在这个问题，怀疑是机房的网络链路问题。

联系负责机房的同学一起排查，未发现网络链路问题。将某个接口所有的流量从机房A切到机房B，发该接口的耗时恢复到之前的水平，看起来的确像是出现机房A网络存在问题。但OP同学再三确认，未发现异常。

<!--more-->

### 业务分析

既然宏观层面看不出问题，那就从业务侧找具体case来分析。初步排查，所有接口耗时均有上涨，去除极端case，找一个平均接口耗时的接口进行分析，就称为接口a吧。先分析下这个接口的top 30耗时情况：

```shell
$ sed -nr 's/.*([0-9]{2}:[0-9]{2}:[0-9]{2}) logid=([0-9]+).*status=2 cost_time=([0-9]+) uri={api a} .*/\1 \2 \3/p' api_server.log | sort -k3 -nr | head -n 30
1  09:12:07 4286155037 6751
2  09:49:49 217512222 5873
3  09:49:49 217544086 5816
4  09:22:27 53384959 5135
5  09:21:48 49524458 5120
...
```

上面命令是从日志中过滤出接口a的请求时间、logid、接口耗时（单位ms），发现耗时甚至还有不少5s以上的，随机找了一个耗时在3s左右的logid进行分析：

```shell
$ grep 182288849 api_server.log{,.wf}
```

上面命令通过logid串联对应的一次请求的所有日志。结合代码和日志分析，发现从接收到请求到开始请求下游服务的过程中就耗时了3s，这中间只有一个网络请求，跟本机一个http服务（以下简称为ServiceA）交互，也就是说是走的环回网络。

**通常情况下，请求环回网络耗时应该在1ms以内（也要看具体业务场景）**，但该请求耗时却超过3s。

目前来看，这3s要么是业务侧发的慢，要么是ServiceA吐数据慢，或者是网络传输慢。之前排查网络环境，并未发现异常，所以这个耗时要么是ServiceA引起的，要么是业务侧引起的。

### 定点分析

由于同一个logid是可以串联出ServiceA的，ServiceA是用nginx起的一个http server，串联出日志后发现ServiceA处理请求很快，$request_time为0，nginx官方对$request_time的定义是：

```shell
$request_time
request processing time in seconds with a milliseconds resolution; time elapsed between the first bytes were read from the client and the log write after the last bytes were sent to the client
```

以上定义是说，$request_time是nginx在读取到客户端的第一个字节到发送数据给客户端之间的时间，看起来ServiceA处理请求很快呀。但耗时也可能发生在ServiceA建立好连接后到读取数据之间。看来只能抓包看了。

抓包工具非tcpdump不可了（注意，tcpdump需要管理员权限）。但如何抓取出出现长耗时的包呢，主要有两个难点：

- 耗时的请求不是稳定复现。
- 如何从pcap包中过滤出耗时较长，且跟serviceA交互的完整请求TCP包？

**思路：tcpdump工具的-A命令可以打印出数据包的ASCII值，由于请求交互时都带了logid，可以通过logid把完整的TCP包过滤出来。**具体流程如下：

1、先用tcpdump抓环回网卡lo在8240端口上的所有数据包并dump到文件中（serviceA监听8240端口）

```shell
$ tcpdump -i lo -n tcp port 8240 -A -w data.pcap
```

2、每分钟统计一次机器上长耗时情况，当发现出现长耗时（大于2s）的接口请求时，停止抓包。

```shell
$ grep 'serviceA reqest_cost' api_server.log | grep '14 21:36' | sed -nr 's/.*logId=\[([0-9]+)\].*serviceA_cost=([0-9]+\.[0-9]+).*/\1 \2/p' | sort  -k2 -nr | head -n 10
```

3、通过长耗时请求的logid串联出对应的serviceA的日志

```shell
$ grep {logid} /path/to/serviceA/logs/access.log
```

通过日志串联，可以拿到客户端的ip和port

```shell
remote_addr=127.0.0.37 remote_port=19001
```

4、通过logid在pcap中找到对应的TCP包

```shell
$ fgrep -a -n '310955179' -B 10
```

同样，通过日志串联在pcap中拿到serviceA的ip和端口

```shell
Host: 127.0.0.37:8240
```

5、通过源主机ip，port和目标主机的ip，port过滤网络包

```shell
$ tcpdump -i lo -n '(src host 127.0.0.37 and src port 19001 and dst host 127.0.0.37 and dst port 8240) or (dst host 127.0.0.37 and dst port 19001 and src host 127.0.0.37 and src port 8240)' -r data.pcap -vv
reading from file data1.pcap, link-type EN10MB (Ethernet)
21:36:05.843275 IP (tos 0x0, ttl 64, id 16029, offset 0, flags [DF], proto TCP (6), length 60)
    127.0.0.37.19001 > 127.0.0.37.8240: Flags [S], cksum 0x7e96 (correct), seq 2331035588, win 32792, options [mss 16396,sackOK,TS val 1513486681 ecr 0,nop,wscale 8], length 0
21:36:05.843312 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto TCP (6), length 60)
    127.0.0.37.8240 > 127.0.0.37.19001: Flags [S.], cksum 0x04ed (correct), seq 476776886, ack 2331035589, win 32768, options [mss 16396,sackOK,TS val 1513486681 ecr 1513486681,nop,wscale 8], length 0
21:36:05.843339 IP (tos 0x0, ttl 64, id 16030, offset 0, flags [DF], proto TCP (6), length 52)
    127.0.0.37.19001 > 127.0.0.37.8240: Flags [.], cksum 0xed91 (correct), seq 1, ack 1, win 129, options [nop,nop,TS val 1513486681 ecr 1513486681], length 0
21:36:05.843422 IP (tos 0x0, ttl 64, id 16031, offset 0, flags [DF], proto TCP (6), length 337)
    127.0.0.37.19001 > 127.0.0.37.8240: Flags [P.], cksum 0xff8d (incorrect -> 0xc320), seq 1:286, ack 1, win 129, options [nop,nop,TS val 1513486681 ecr 1513486681], length 285
21:36:05.843442 IP (tos 0x0, ttl 64, id 61678, offset 0, flags [DF], proto TCP (6), length 52)
    127.0.0.37.8240 > 127.0.0.37.19001: Flags [.], cksum 0xec70 (correct), seq 1, ack 286, win 133, options [nop,nop,TS val 1513486681 ecr 1513486681], length 0
21:36:08.905498 IP (tos 0x0, ttl 64, id 61679, offset 0, flags [DF], proto TCP (6), length 471)
    127.0.0.37.8240 > 127.0.0.37.19001: Flags [P.], cksum 0x0014 (incorrect -> 0xd2df), seq 1:420, ack 286, win 133, options [nop,nop,TS val 1513489743 ecr 1513486681], length 419
21:36:08.905522 IP (tos 0x0, ttl 64, id 16032, offset 0, flags [DF], proto TCP (6), length 52)
    127.0.0.37.19001 > 127.0.0.37.8240: Flags [.], cksum 0xd2e1 (correct), seq 286, ack 420, win 133, options [nop,nop,TS val 1513489743 ecr 1513489743], length 0
21:36:08.905647 IP (tos 0x0, ttl 64, id 16033, offset 0, flags [DF], proto TCP (6), length 52)
    127.0.0.37.19001 > 127.0.0.37.8240: Flags [F.], cksum 0xd2e0 (correct), seq 286, ack 420, win 133, options [nop,nop,TS val 1513489743 ecr 1513489743], length 0
21:36:08.905672 IP (tos 0x0, ttl 64, id 61680, offset 0, flags [DF], proto TCP (6), length 52)
    127.0.0.37.8240 > 127.0.0.37.19001: Flags [.], cksum 0xd2df (correct), seq 420, ack 287, win 133, options [nop,nop,TS val 1513489744 ecr 1513489743], length 0
```

从TCP数据包来看：

- 21:36:05.843275，开始建立连接
- 21:36:05.843339，已完成三次握手
- 21:36:05.843422，客户端发起请求
- **21:36:05.843442，服务端确认**
- **21:36:08.905498，服务端吐数据**
- 21:36:08.905522，客户端确认
- ….
- 21:36:08.905672，连接断开

从TCP数据包来看，服务端在收到请求后，立即确认（内核行为），但吐数据却是在3s后，进一步确认serviceA服务的问题。

### 进程分析

虽然已经确认是serviceA的问题导致，但我还是进一步分析，是什么造成nginx在吐数据的时候出现阻塞。nginx是单线程执行，通过阻塞操作出现一些系统调用上上，如写磁盘IO、网络IO、锁等。strace是分析进程系统调用的利器。

1、ps查看serviceA进程

```shell
$ ps -ef | grep nginx
```

2、strace查看系统调用

```shell
# 这个地方可以使用-e trace=write过滤系统调用，这个地方不过滤是想看看是不是其它系统调用引起的
$ strace -p30578 -tt -i -T 
```

日志比较多，但开启几分钟后出现了的确出现了3s左右的阻塞。具体日志就不贴出来了，从日志中可以看出：有大文件写入操作，文件大小22M。fd=4 close后有两次释放内存操作，耗时也比较小，但从11:10:12的munamp到11:10:14 write日志中间花了2s+。

经负责serviceA的同学分析后，发现主要耗时是在json序列化写磁盘以及lua map的deep copy操作。

最后回答一下为什么只有机房A出现耗时上涨：因为serviceA在部署在了机房A，另外两个机房还未部署。

## 总结

- 问题的产生总有它的原因，当没有思路时，停下来仔细梳理每一个细节，大胆猜想，小心求证。
- 由于现在普遍采用微服务架构，服务间依赖关系比较多。但排查问题的流程却并不完善，出现问题时，应该让整条数据链路上的值班同学参与到故障排查中，而不是问题症状方去全链路排查, 这样率非常低。
- 系统、网络等基础知识非常重要，有助于你尽快发现问题、收敛问题。
- 熟练使用常用工具，提升排查问题效率。
