---
title: Go数据结构之channel
date: 2023-02-18 16:53:42
categories:
- 编程语言
tags:
- Go
- channel
- 并发编程
---

channel是一种数据结构，在Go语言中用于协程间通信，是Go语言区别于其它语言的重要特性。Go语言原生支持channel，配合Go语言原生对并发的支持，让并发编程变得简单。正如[Share Memory By Communicating](https://go.dev/blog/codelab-share)对Go并发编程的建议：

> Do not communicate by sharing memory; instead, share memory by communicating.

和传统并发编程使用同步原语共享内存不同，Go并发编程强调通过channel在协程间通信来共享内存，这实际上是对**CSP**[(**C**ommunicating **S**equential **P**rocesses)](http://www.usingcsp.com/)并发模型的一种实现。

从应用层面来讲，channel和协程（goroutine）是配合使用的，本文主要从实现层面介绍channel的内部原理，但不会涉及太多协程管理调度相关的知识。本文先介绍channel的基本用法，然后介绍channel内部数据结构以及实现原理，最后介绍使用channel时应该注意的一些问题。

<!--more-->

## 基本用法

channel本质上是一个先进先出（FIFO）的队列。根据channel是否可以缓存数据，分成两种类型：

- 带缓冲区的channel（buffered channel）
- 不带缓冲区的channel（unbuffered channel）

从数据流向来看，可以分为三种类型：

- 双端channel：支持对channel的读写
- 只读channel：只能从channel中读数据
- 只写channel：只能向channel中写数据

对channel的操作主要包括以下几种：

- 创建channel
- 向channel中写入数据
- 从channel中读取数据
- 关闭channel

channel自身的操作是线程安全（concurrency-safe），所以我们在多个协程（goroutine）中操作channel是安全的，下面简单介绍这几种操作。

### 创建channel

创建channel只能使用make内建函数，第二个可选参数cap表示指定缓冲区大小，不指定表示无缓冲区的channel。

创建无缓冲channel，使用`make(chan T)`，如创建一个类型为int的无缓冲channel：

> ch := make(chan int)

创建有缓存channel，使用`make(chan T, cap)`，如创建一个类型为int，缓冲区大小为10的channel：

> ch := make(chan int, 10)

通常不会创建一个单向流动的channel（没有实际意义），只读或只写channel用于限制channel中的数据流向，通常是为了安全考虑，类似于C++声明的const常量。只读只写channel声明如下：

```golang
// 只读channel
var ch <-chan int

// 只写channel
var ch chan<- int
```

需要注意的是，用var定义的channel初始值是零值，即nil，向值为nil的channel中读或写都会阻塞当前goroutine。

### 向channel中写入数据

向channel中写入数据，语法为`ch <- T`，如向类型为int的channel中发送数据：

```golang
ch := make(chan int, 10)
ch <- 1
```

当channel无缓冲区，或缓冲区已满，且没有接受的goroutine时，发送的goroutine会被阻塞。

### 从channel中读取数据

从channel中读取数据有两种方式如下：

```golang
// 方式一
v := <-ch

// 方式二
comma, ok := <-ch

// 方式三
for v := range ch {
    ...
}
```

对于方式一，当channel无缓冲区，或缓冲区无数据，且没有goroutine向channel写数据时，接收的goroutine会被阻塞。
对于方式二，称为`comma, ok`表示式，这个地方第二个变量ok常常错误地认为是channel关闭的状态，ok确实跟channel是否关闭有关系，但并不准确，更确切地说ok用于检测ch中是否有可读的数据。一个关闭的管道有两种情况：

- channel缓冲区已没有数据；
- channel缓冲区还有数据；

对于第一种情况，第二个变量ok为false；但对于第二种情况，第二个变量ok为true。

对于方式三，会循环中channel中读取数据，当没有可读的数据时会被阻塞，当channel被关闭时，读取完channel中的所有数据后，会退出循环。

### 关闭channel

关闭channel使用函数close。close一个channel会唤醒所有等待在该channel上的goroutine。

需要注意的是，向已经关闭的channel中写数据会panic，所以在不确定是否还有goroutine需要向channel发送数据时，请勿贸然关闭channel，[How to Gracefully Close Channels](https://go101.org/article/channel-closing.html)给了一些优雅关闭channel的原则。另外关闭已经关闭了的channel也会panic。

### 其它操作

可以通过len和cap函数获取channel当前的缓存的数据长度和缓冲区大小。对于值为nil或无缓冲区的channel，这两个操作均为0。对于有缓冲区的channel，len(ch)为缓冲区中缓存的数据个数，cap为缓冲区大小。如：

```golang
ch := make(chan int, 10)
ch <- 1
ch <- 2
fmt.Println(len(ch)) // 2
fmt.Println(cap(ch)) // 10
```

## 数据结构

上述从应用层面介绍channel的使用，这一节将从实现层面看这些操作是如何实现的，源码版本参考版本为`go 1.19`。

我们说channel本质上是一个队列，在channel内部，其数据结构是一个环形队列，为通过`hchan`表示（runtime/hchan.go），如下：

```golang
type hchan struct {
    qcount   uint           // 队列中的元素个数
    dataqsiz uint           // 环形队列的长度，即最大能缓存的元素个数
    buf      unsafe.Pointer // 指向环形队列的指针
    elemsize uint16         // 元素的大小
    closed   uint32         // 标识队列关闭状态
    elemtype *_type         // 元素类型
    sendx    uint           // 队列下标，指向元素写入时存放到环形队列中的位置
    recvx    uint           // 队列下标，指向下一个被读取元素在队列中的位置
    recvq    waitq          // 指向等待读消息的协程队列（链表）
    sendq    waitq          // 指向等待写消息的协程队列（链表）

    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    lock mutex // 互斥锁，用于channel状态的并发读写同步
}

type waitq struct {
    first *sudog
    last  *sudog
}
```

以下将分别以下几个方便介绍channel的具体实现，以下介绍不会深入代码细节，只介绍主要流程：

- 创建channel
- 向channel中写入数据
- 从channel中读取数据
- 关闭channel

### 创建channel

上面提到创建channel是通过`make`函数实现，当用`make`创建一个channel时，编译器转换为运行时的`runtime.makechan`（见`runtime/chan.go`）。创建的channel的过程实际上是初始化`hchan`结构，伪代码如下：

```golang
func makechan(t *chantype, size int) *hchan {
    var c *hchan
    c = new(hchan)
    c.buf = malloc(元素类型大小 * size)
    c.elemsize = 元素类型大小
    c.elemtype = 元素类型
    c.dataqsiz = size

    return c
}
```

### 向channel中写入数据

上面提到向channel中写入数据使用`ch <- T`，编译器会将其解析为运行时的`runtime.chansend`函数（见`runtime/chan.go`），主要流程如下：

- 如果recvq不为空，即有正在等待接收的goroutine，此时通过`runtime.send`会把数据直接传递给recvq队列中的第一个协程。
- 如果缓冲区有空余位置时，则将数据写入缓冲区，结束发送过程。
- 如果缓存区中没有空余位置，则将当前协程入加sendq队列，进入眨眼并等待被读协程唤醒。

### 从channel中读取数据

上面提到从channel中读取数据，编译器会将其解析为运行时的`runtime.chanrecv`函数（见`runtime/chan.go`），主要流程如下：

- 如果sendq不为空，即有正在等待发送的goroutine，此时通过`runtime.recv`直接从阻塞的sendq队列的第一个协程中获取数据。
- 如果缓冲区中有数据，则从缓存区中读取数据，结束读取过程。
- 如果缓冲区中没有数据，则将当前协程加入recvq队列，进入睡眠并等待被写协程唤醒。

### 关闭channel

上面提到关闭channel通过`close`函数实现，编译器会将其解析为运行时的`runtime.closechan`函数（见`runtime/chan.go`），主要流程如下：

- 如果channel为nil或已经关闭，抛出异常。
- 把recvq（等待接收的协程）全部唤醒，这些协程获取的数据都为对应类型的零值。
- 把sendq（等待发送的协程）全部唤醒，但这些协程会触发panic。

## 一些需要注意的问题

- 向值为nil的channel中读写数据，都会阻塞。
- 向已关闭的channel中写入数据会panic。
- 向已关闭的channel中读取数据，如果channel缓冲区中还有数据，可以正常读出，如果缓冲区中无数据，将获得对应类型的零值。
- 重复关闭channel会panic。
- 关闭一个零值channel会panic。

## 参考资料

[1] [Go 语言设计与实现 #Channel](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/)
[2] [Channels in Go](https://go101.org/article/channel.html)
[3] [golang-notes #Channel](https://github.com/cch123/golang-notes/blob/master/channel.md)
[4] [Go专家编程](https://item.jd.com/12920392.html)
[5] [Go channels on steroids](https://docs.google.com/document/d/1yIAYmbvL3JxOKOjuCyon7JhW4cSv1wy5hC0ApeGMV9s/pub)