---
title: Go并发编程模式
date: 2021-07-02 23:38:31
categories:
- 编程语言
tags:
- Go
- 并发
- 模式
---


## 简介

并发是Go语言最重要的语言特性之一，是Go语言区别于其它语言的重要特征。Go语言原生支持并发，可以充分发挥多核CPU的机器性能，同时在语言层面上，Go以十分简洁的语法提供丰富的并发能力，让并发编程并得简单。本文总结Go语言并发编程的常用模式，模式也就是我们说的套路，先学会模仿，再学会融汇贯通，最后才能创造出新的模式，这就是所谓的无招胜有招吧。

## Go对并发的支持

对于桌面客户端应用程序，为了提升客户端的运行效率和简化多线程编程，通常的做法是在客户端启动时，启动三个常驻线程，分别是UI线程，逻辑线程和IO线程。每个线程维护一个事件循环，线程之间通信（数据传输）只能通过消息队列进行传递，消息队列是线程安全。

这么做主要有两个好处，一是常驻线程避免创建和销毁线程的开销，多核情况下，通常没有线程上下文切换，线程运行效率高。二是通过消息队列传递数据，各个线程对数据进行操作时不用所以资源竞争问题，这极大降低了多线程编程的难度，并且设计出的程序也更简单，更容易理解。

接触到Go之后，才知道这种编程模型其实就是C. A. R. Hoare的[Communicating Sequential Processes, CSP](http://www.usingcsp.com/)。Go语言原生支持并发（goroutine）和通道（channel），并且还提供了[select](https://golang.org/ref/spec#Select_statements)对多路通道进行控制，极大简化了并发编程的难度。

接下来，本文将介绍Go并发编程的常用模式。本文并不适合没有Go语言基础的读者，在阅读以下内容前，请先查阅以下内容：

- [The Go Programming Language Specification](https://golang.org/ref/spec)
- [Effective Go](https://golang.org/doc/effective_go)

<!--more-->

## 基本原则

正如文章 [Share Memory By Communicating](https://blog.golang.org/codelab-share) 所概括的：

> *Do not communicate by sharing memory; instead, share memory by communicating.*

这跟传统多线程编程有着本质的区别：传统多线程编程是通过共享状态进行通信，为了防止共享状态存在竞争问题，通常是使用一些同步原语，如互斥量、临界区、信号量等对共享状态进行同步；而Go语言中的并发编程是通过使用channel在goroutine之间传递消息，这种能够保证在某一时刻goroutine能够对数据进行独占访问，不存在资源竞争问题。


## Fan-out, Fan-in

**多个处理函数可以从同一个channel中读取数据，直到channel关闭，这称为Fan-out。同一个处理函数通过对多个输入channel进行多路复用，并将多个输入channel定向到一个单一的channel中，当多个输入channel均关闭后，将单一channel也关闭，这被称为Fan-in。**

<img src="/images/go-concurency-pattern/fan-out.png" alt="image" style="zoom:45%;" /> 

<img src="/images/go-concurency-pattern/fan-in.png" alt="image" style="zoom:45%;" />

Fan-out, Fan-in有点像算法中的divide-conquer-combine。Fan-out模式提供一种分治处理的能力，将任务拆分成一组并发的worker进行处理，可以充分利用机器CPU和I/O。Fan-in则将各worker处理的结果进行合并，收拢到统一入口，可以对整体结果做一些加工处理。 


## Pipeline

**在Go语言中，Pipeline是一系列通过channel连接的处理阶段，每个阶段可以理解为由一组goroutine运行的相同函数。**对于每个阶段，goroutine的处理流程大致为：

- 通过输入channel从上游（上一阶段）接收数据；
- 处理数据并生成新的数据；
- 将新生成的数据通过输出channel发送给下游（下一阶段）；

除了第一个阶段和最后一个阶段外，每个阶段都有一个或多个输入和输出channel。第一个阶段被称为源（source）或生产者（producer），最后一个阶段，被称为槽（sink）或消费者（consumer）。

对于pipeline的惯用法，有两条指导原则：

- 对于每个阶段，当向输出channel发送完数据后应该即时关闭channel。
- 对于每个阶段，持续从输入channel中读取数据，直到channel被关闭或被取消（下面会单独介绍取消操作）。


## Timeout

并发编程通常要处理超时的问题，在实际应用中，通过网络请求获取远端数据就是一个很典型的例子。为了不阻塞当前线程，通过需要在网络线程（也称为I/O线程）去发送网络请求，由于网络通常是不可靠的，通常需要设置一个超时时间，如果在既定的超时时间范围内，请求还是没有完成，就取消网络请求。


在Go语言中通过Goroutine开启一个并发任务，通过channel在Goroutine之间通信，使用select对控制多个channel。对于timeout，可以直接使用标准库函数time.After，开启一个定时器，当到达指定时间后，定时器会往channel中发送一个消息，如下例：

```golang
func Request(query string) <-chan string {
    out := make(chan string)
    go func(query string) {
      	// 模拟请求耗时较长
        time.Sleep(3 * time.Second)
        out <- "world"
    }(query)
    return out
}

func main() {
    out := Request("hello")
    select {
    case result := <-out:
        fmt.Println("fetch result:", result)
    case <-time.After(1 * time.Second):
        fmt.Println("timeout")
    }
}
```



## Cancellation

除了超时控制之外，并发编程中另一个经常需要处理的问题就是取消操作。和超时控制类似，取消操作是指调用方可以在对运行的goroutine进行取消。考虑一个经典的生产-消息者模型：生产者持续生产消息，消息者持续消费消息，直到应用程序终止。

```golang
func produce(done <-chan struct{}) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for {
            select {
            case <-time.Tick(100 * time.Millisecond):
                out <- rand.Int() % 100
            case <-done:
                fmt.Println("producer quit")
                return
            }
        }
    }()
    return out
}

func consume(done <-chan struct{}, ch <-chan int) {
    go func() {
        for {
            select {
            case n := <-ch:
                fmt.Println(n)
            case <-done:
                fmt.Println("consumer quit")
                return
            }
        }
    }()
}

func main() {
    done := make(chan struct{})
    out := produce(done)
    consume(done, out)
    time.Sleep(1 * time.Second)
    close(done)
}
```



在以上例子中，我们使用done来通知各gorountime退出循环，使用select来控制多个channel的接收和发送，当main函数要退出时，关闭done，goroutime product和consumer会读取到这个变化，退出循环，这样就正确的退出的goroutine。其实Go标准库中的context包已经为我们提供了类似的功能，我们只需要简单改一下代码即可：

```golang
func produce(ctx context.Context) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for {
			select {
			case <-time.Tick(100 * time.Millisecond):
				out <- rand.Int() % 100
			case <-ctx.Done():
				fmt.Println("producer quit")
				return
			}
		}
	}()
	return out
}

func consume(ctx context.Context, ch <-chan int) {
	go func() {
		for {
			select {
			case n := <-ch:
				fmt.Println(n)
			case <-ctx.Done():
				fmt.Println("consumer quit")
				return
			}
		}
	}()
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()
	out := produce(ctx)
	consume(ctx, out)
	time.Sleep(1 * time.Second)
}
```



## 例子

[Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines) 中给出了一个计算文件md5的实用程序，使用了pipline，fan-out, fan-in，cancellation等常用并发编程模式，是学习Go并发编程特别好的例子。整个代码才100余行就能实例md5sum的功能，而且效率还很高。在这个小程序中，使用pipeline将整个计算过程分成三个阶段：

- 遍历目录
- 计算文件md5
- 收集计算结果

我对原程序做了一点点修改，引入了context包用于cancellation，当遍历文件时任何一部出现错误时，可以进行优雅退出；同时对bounded concurency部分代码做了优化，使其更简洁和更具可读性，具体代码参考[md5sum](https://github.com/leeyzero/go-tools/blob/main/md5sum.go)。

在这个例子中，还可以加入timeout超时控制，以指定程序的执行时长，如果在指定的超时时间内未完成所有文件的md5计算，则程序提示执行超时，进行退出。增加这个功能其实比较简单，可以直接使用context.WithTimeout实现，感兴趣的读者可以自己动手进行实现。



## 总结

Go语言原生支持并发操作，go、channel都成为了一等公民，同时使用select可以对多路channel发送和接收进行控制，极大降低了并发编程的难度。首先一个基本原则是goroutine之间通过通信共享内存，在此基本上我们介绍和Fan-out, Fan-in，Pipeline，Timeout，Cancellation几个常用的Go并发编程模式，最后给出了一个综合应用的例子以强化以上基本概念。

## 参考资料

[1] [Share Memory By Communicating](https://blog.golang.org/codelab-share)
[2] [Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)
[3] [Go Concurrency Patterns: Timing out, moving on](https://blog.golang.org/concurrency-timeouts)
[4] [Go Concurrency Patterns: Context](https://blog.golang.org/context)
[5] [Advanced Go Concurrency](https://encore.dev/blog/advanced-go-concurrency)
[6] [Mutexes and Semaphores Demystified](https://barrgroup.com/Embedded-Systems/How-To/RTOS-Mutex-Semaphore)
[7] [Rethinking Classical Concurrency Patterns](https://github.com/golang/go/wiki/Go-Community-Slides#rethinking-classical-concurrency-patterns)
[8] [Go Concurrency Patterns](https://talks.golang.org/2012/concurrency.slide#1)
[9] [Advanced Go Concurrency Patterns](https://talks.golang.org/2013/advconc.slide#1)

