---
title: Go并发编程实践
date: 2022-05-06 20:24:21
categories:
- 编程语言
tags:
- Go
- concurrency
---

# 写在前面
之前写过一篇[Go并发编程模式](https://leeyzero.github.io/2021/07/02/go-concurrency-pattern/)，相对比较片面。最近系统看了下[Concurrency in Go](https://edu.anarcho-copy.org/Programming%20Languages/Go/Concurrency%20in%20Go.pdf), 结合自身的一些实践，形成本文，算是一个读书笔记吧。

## 为什么需要并发？
摩尔定律逐渐失效，数据规模的不断增长需要充分挖掘多核计算机性能，而并发编程是利器：
>
>Concurrency is the next major revolution in how we write software.
>
>——The Free Lunch Is Over


更多请参考：[Herb Sutter](https://en.wikipedia.org/wiki/Herb_Sutter) 2005年在Dr. Dobb's 发表的文章：[The Free Lunch Is Over](http://www.gotw.ca/publications/concurrency-ddj.htm)。

<!--more-->

## 为什么并发编程是困难的？
- 竞争条件（Race Conditions）
- 原子性（Atomicity）
- 内存访问同步（Memory Access Synchronization）
- 死锁、活锁和饿死（Deadlocks、Livelocks and Starvation）
- 并发安全性（Determining Concurrency Safety）

# 基础知识
## 并发与并行
并发（concurrency）和并行（parallelism）有相关性但却是截然不同的概念。[Rob Pike](https://en.wikipedia.org/wiki/Rob_Pike)在[Concurrency is not parallelism](https://go.dev/blog/waza-talk)主题演讲中总结的非常好：

>In programming, concurrency is the **composition** of independently executing processes, while parallelism is the simultaneous execution of (possibly related) computations.
>
>Concurrency is about **dealing with** lots of things at once. Parallelism is about **doing** lots of things at once.
>
>——Concurrency is not parallelism

并发是指同时**处理（dealing with）**多件事，并行是指同时**去做（doing）**多件事；并发强调在**逻辑上（logically）**同时发生；并行强调在**物理上（physically）**同时发生。

单核CPU是可以并发的，但不能并行；多核CPU即可以并发，也可以并行。
<p float="left">
    <img src="/images/go-concurrency-in-action/1.png" width="400" />
    <img src="/images/go-concurrency-in-action/2.png" width="400" />
</p>


## 通信顺序进程CSP
**C**ommunicating **S**equential **P**rocess，通信顺序进程是一种并发编程模型，可以说是复兴的古董，最早是[C. A. R. Hoare](http://taggedwiki.zubiaga.org/new_content/9e36a51d3ecb524e685ce5eb5ab8c2e0)在1978年在论文 [Communicating Sequential Processes](http://www.usingcsp.com/cspbook.pdf)提出。

CSP模型也是由独立的、并发执行体组成，实体之间是通过发送消息进行通信。CSP模型不关注发送消息的实体，而是关注发送消息时使用的通道（channel）。这便是我们在Go中

进行并发编程时首先要转变的观念：

> Do not communicate by sharing memory; instead, share memory by communicating.

更多请参考：[Share Memory By Communicating](https://go.dev/blog/codelab-share)。

## Go语言对并发的支持
在Go中，并发和通道变成了一等公民，使用go启动一个协程，使用channel在协程之间通信，使用select进行通信多路复用。



### goroutine

>A goroutine is a function executing concurrently with other goroutines in the same address space.
>
>It is lightweight, costing little more than the allocation of stack space. And the stacks start small, so they are cheap, and grow by allocating (and freeing) heap storage as required.
>
>—— Effective Go



在Go中，启动一个goroutine十分简单：

```
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```


### channel

>A channel provides a mechanism for concurrently executing functions to communicate by sending and receiving values of a specified element type. The value of an uninitialized channel is nil. 


注意区分：

- unbuffered channel：c := make(chan T)
- buffered channel：c := make(chan T, size)
- receive only channel：←chan T
- send only channel：chan← T

更多关于 channel 的使用参考：[Language Specification Select statements](https://go.dev/ref/spec#Channel_types).


### select

在类C的语言中，控制结构通常包括：if，for，while，switch，break，continue，在Go中，使用了一种全新的控制结构 select 用于通信多路复用。
```
select {
case v := <-readChan:
case writeChan <- v:
default:
	fmt.Println("The default clauses in the select statements execute when no other case is ready, meaning that the selects never block.")
}
```

空的select为永远阻塞：
```
select {} // blocked forever
```

更多关于 select 使用参考：[Language Specification Select statements](https://go.dev/ref/spec#Select_statements).

## Go标准库对并发的支持
- sync
- atomic
- context
- errgroup

# Go并发编程模式
## 两个原则
[Rethinking classical concurrency pattern](https://www.youtube.com/watch?v=5zXAHh5tJqQ) 从传统并发编程模式角度出发，总结了两个条Go并发编程的原则以及如何在Go中应用这些模式：

- Start goroutines when you have concurrent work
- Share by communicating

第一条原则是指：如果需要对所做的工作进行并发，那首先需要将工作拆分成可发并的单元。

第二条原则是指：并发之间的通信通过消息传递，而不是共享内存。

## 限定作用域
并发编程时，并发安全是一个非常重要的主题，如何保证操作的安全性，通常有以下四种操作：

- 使用同步原语对共享内存进行同步，如使用sync.Mutex
- 通过通信进行同步，如使用channel
- 不可变数据
- **在限定作用域下被保护数据**


限定作用域并不是一个新鲜的概念，在面向对象（OO）中的封装，函数式编程中的闭包（Closure）都可以理解为限定作用域，

**其本质是通过语法层面对数据的作用域加以限制，使其更容易被理解，并减少或消除被误用的条件。**



未限定作用域：https://go.dev/play/p/4TYhZlPMbds

限定作用域：https://go.dev/play/p/TxgEx047rPl


以下是在给定channel状态后，对其进行read、write和close操作的结果：
<img src="/images/go-concurrency-in-action/3.png" width="500"/>


注意上表中几个**panic**：

- 向已经关闭的channel中写数据会panic
- 关闭状态为nil的channel会panic
- 关闭已经关闭的channel会panic

在限定作用域的例子中，我们将创建、写入、和关闭都限定在chanOwner的作用域中，并返回一个receive only的channel，在consumer中

对receive only进行只读操作，并等待channel关闭。通过限定作用域，可以更明确的表明channel的作用范围，规避一些不正确的用法。

## For-select loop
在Go中，for-loop是一种惯用法，模式如下：
```
for { // Either loop infinitely or range over something 
	select {
        // Do some work with channels
	} 
}
```

通常有以下场景运用：

将迭代器中的值发送到一个channel中：
```
for _, s := range []string{"a", "b", "c"} { 
	select {
	case <-done: 
		return
	case stringStream <- s:
	} 
}
```

从一个channel中读取数据处理后写入到另一个channel中：
```
for s := range in { 
	select {
	case <-done: 
		return
	case out <- process(s):
	} 
}
```

无限循环并等待退出：
```
for { 
	select {
	case <-done: 
		return
	default: 
	}        
	// Do non-preemptable work
}
```

另一种写法是将非强占式逻辑写到default中
```
for { 
	select {
	case <-done: 
		return
	default:
		// Do non-preemptable work
	} 
}
```

注意，以上每种for-loop的变种，select都加了一个←done操作，是为了防止出现死循环导致goroutine泄露的情况。后面会讨论用更通用的context包解决这个问题。

接下来将讨论一些关于如何避免goroutine泄露的模式。

## 防止goroutine泄露
对于一个goroutine，通过有以下三种路径结束其生命周期：

- goroutine完成任务自动退出
- goroutine在执行过程中遇到不可恢复的错误而不能继续后续任务
- 调用端取消goroutine


防止goroutine泄露的一条经验法则是：

>If a goroutine is responsible for creating a goroutine, it is also responsible for ensuring it can stop the goroutine.
>
>—— Concurrency in Go

一个可能发生goroutine泄露的例子：https://go.dev/play/p/bAx4c6CXm7V

防止goroutine泄露：https://go.dev/play/p/ZfELCscQCzl



在这个例子中，main在调用doWork启动一个goroutine后，同时指定了一个done channel，用于超时控制，如果goroutine超过1s未返回，关闭

done channel让goroutine退出。**也就是说创建goroutine的协程一定要保证被创建的协程能够退出**。后面讲到context时，会采用更通用的方式实现。

## Pipeline
[Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines) 中关于pipeline在Go中的定义：

>a pipeline is a series of stages connected by channels, where each stage is a group of goroutines running the same function.
>
>—— Go Concurrency Patterns: Pipelines and cancellation

对于每个阶段（stage），goroutines主要包括以下三个部分：

- 通过inbound channels从上游接受数据
- 处理数据
- 将处理后的数据通过outbound channels发送给下游

除了第一个阶段和最一个阶段外，每个阶段都可以有一个或多个inbound channel和一或多个outbound channel。第一个阶段通常叫**源（source）或生产者（producer）**，

最后一个阶段通常叫**槽（sink）或消费者（consumer）**。


一个包含3个阶段的pipeline的例子：https://go.dev/play/p/D_iBejB3hgW


在这个例子中，原始数据[1, 2, 3]经过每个阶段（gen、add、sq）进行处理，阶段与阶段之间通过channel进行连接，最后可以读取pipeline channel输出结果。

数据通过channel很优雅在每个阶段之间流动，但如果某个阶段处理特别慢的情况下，下游阶段就需要等待上游处理完才能拿到数据。那如何提供各个阶段处理数据的效率呢？

根据pipleline的定义，如果数据之间没有依赖关系的话，每个阶段可以包含一系列goroutine并发（也可能是并行）地处理数据。这便引出Go的另一种并发编程模式——Fan in Fan out。

注意使用Fan in Fan out模式的前提是：

- 阶段处理数据效率影响整体性能
- 数据之间没有依赖

## Fan-out, fan-in 
根据 [Concurrency in Go](https://edu.anarcho-copy.org/Programming%20Languages/Go/Concurrency%20in%20Go.pdf) 中关于Fan-out, fan-in的定义：

>Fan-out is a term to describe the process of starting multiple goroutines to handle input（same input) from the pipeline.
>
>Fan-in is a term to describe the process of combining multiple results into one channel.
>
>—— Concurrency in Go


Fan-out的例子：https://go.dev/play/p/is4YFOJcuIu

Fan-in的例子：https://go.dev/play/p/8rxF_KPZPO6


在上述例子中，如果没有，如果某个阶段如现错误，或者想在中途停止pipeline，该如何实现呢？

标准库context为我们提供了优雅退出协程的接口。

## Context
Go标准库提供了context包用于处理超时、取消、API边界的上下文数据传递。

>package context defines the Context type, which carries deadlines, cancellation signals, and other request-scoped values across API boundaries and between processes.

接口定义也比较简洁：
```
type Context interface {
	// Deadline returns the time when this Context will be canceled, if any.
	Deadline() (deadline time.Time, ok bool)


	// Done returns a channel that is closed when this Context is canceled or times out.
	Done() <-chan struct{}
	
	// Err indicates why this context was canceled, after the Done channel is closed.
	Err() error
	
	// Value returns the value associated with key or nil if none.
	Value(key any) any
}
```
标准库context有对接口的完整说明以及example，在此不展开。另外还有一篇关于context非常好的文章：[Go Concurrency Patterns: Context](https://blog.golang.org/context)

关于context的一个惯用法是：

>Contexts should not be stored inside a struct type, but instead passed to each function that needs it.

至于为什么要这么做，在文章 [Contexts and structs](https://blog.golang.org/context-and-structs) 中有非常详细的理由解释。最主要的原因是：

- 接口维度进行deadline、cancellation以及metadata传递控制；
- 更容易理解，作用域更清晰；
- 在跨库或跨API调用上可以在调用堆栈中暴露context信息，更容易调试；

再回头看pipleline的例子，可以通过在每个阶段传递一个context参数，以进行deadline、cancellation以及request-scoped values控制。

使用context优雅退出的pipeline例子：https://go.dev/play/p/q7u1w60LPNv

# 综合应用
- 串行实现：md5sum-sequential.go
- 并发实现：md5sum-concurrency.go
- 并发优雅退出实现：md5sum-context.go
- 使用errgroup包实现：md5sum-errgroup.go

# 实践
在工程实践中，经常有数据聚合需求，如果各个数据之间没有顺序依赖，可以将获取各个数据子集拆分到不同的并发单元中以提高接口响应效率。以下是伪代码

```
// DataAggregatedAPI ...
func (t *ApiService) DataAggregatedAPI(ctx context.Context, sourceIDs []string) ([]*SourceNode, error) {
	...
	
	// 生成CancelContext
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()
	
	// 生成输入管道
	in := make(chan []string)
	go func() {
		defer close(in)
		for _, id := range sourceIDs {
			select {
			case in <- id:
			case <-ctx.Done():
				return
			}
		}
	}()

	// 2. 并发数据查询
	out := make(chan SourceResp)
	var wg sync.WaitGroup
	for i := 0; i < concurrent; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			for id := range in {
				items, err := t.getSourceBySourceID(id)
				select {
				case out <- SourceResp{items, err}:
				case <-ctx.Done():
					return
				}
			}
		}()
	}
	go func() {
		wg.Wait()
		close(out)
	}()
	
	// 3. 接收结果
	for resp := range out {
		// 任何一个失败，返回失败
		if resp.err != nil {
			return nil, resp.err
		}
		for _, v := range resp.items {
			// 处理结果
		}
	}

	...
}
```

# 总结
我们首先介绍了为什么需要并发以及并发编程的困难性。在此前提下基础上介绍了Go对并发编程的支持以及基本构造块。

然后介绍了Go语言中一些常用的并发编程模式，再通过一个例子md5sum对这些模式进行应用以加深理解。

最后介绍如何将这些模式应用到实际工作一个场景中。

# 参考资料
- [Concurrency is not parallelism](https://go.dev/blog/waza-talk)
- [Concurrency in Go](https://edu.anarcho-copy.org/Programming%20Languages/Go/Concurrency%20in%20Go.pdf)
- [Go Concurrency Patterns](https://talks.golang.org/2012/concurrency.slide#1)
- [Advanced Go Concurrency Patterns](https://talks.golang.org/2013/advconc.slide#1)
- [The Free Lunch Is Over](http://www.gotw.ca/publications/concurrency-ddj.htm)
- [Share Memory By Communicating](https://go.dev/blog/codelab-share)
- [Rethinking classical concurrency pattern](https://www.youtube.com/watch?v=5zXAHh5tJqQ)
- [Effective Go](https://go.dev/doc/effective_go)
- [go-concurrency-patterns all in one](https://github.com/lotusirous/go-concurrency-patterns)
- [Go Concurrency Patterns: Pipelines and cancellation](https://blog.golang.org/pipelines)
- [Go Concurrency Patterns: Timing out, moving on](https://blog.golang.org/concurrency-timeouts)
- [Go Concurrency Patterns: Context](https://blog.golang.org/context)
- [Contexts and structs](https://blog.golang.org/context-and-structs)




