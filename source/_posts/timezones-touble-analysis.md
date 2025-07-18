---
title: 记一次时区问题引发的故障
date: 2021-02-01 00:45:45
categories:
- 编程匠艺
tags:
- 故障排查
- 时间
- 时区
---

周末大清早就被报警短信吵醒，故障表现为离线存储集群流量突增，集中在热点分片，而且流量越来越大，系统吞吐、稳定性均在降低。最终发现是因为实例间时区不一致触发一个隐藏的BUG导致。这个问题在分布式系统中很具有代表性，在此做个记录以备忘。

## 症状

- 离线存储集群流量突增，集中在热点分片；
- 消息系统中消息在持续积压；
- 消息系统收到大量重复ACK；
- 业务worker消费到大量重复的消息；

<!--more-->

## 分析

从表象来看，出现了大量重复消息，周末是禁止上线了，上下游也均没有上线通告，排除上线引起。同时也咨询了运营、市场的同学，并没有做相关的运营活动，从gateway的流量来看，也没有大的波动。初步判断是在某种条件下触发了系统的BUG或是存在雪崩的可能。

分析线上日志和模块代码，排除是业务层面发送重复消息的可能性，把范围缩小到了消息系统本身。消息系统在消息超时会将消息重放入队列中，但从逻辑来看，最多也就重放6次（可配置）。但线上个别消息重放次数达到几十万甚至上百万，仔细分析了日志和代码，发现问题出在以下代码：

```golang
tm := time.Unix(timestamp, 0)
```

以上代码去掉了业务逻辑，但这段代码的确是造成这个问题的根本原因。这行代码的目的是使用指定的时间戳生成一个时间格式串的Redis key，消息系统的多个实例会通过本地时间来共享该Key。这个函数的说明如下：

```golang
func Unix(sec int64, nsec int64) Time

Unix returns the local Time corresponding to the given Unix time, sec seconds and nsec nanoseconds since January 1, 1970 UTC.
```

注意，**函数返回的是本地时间，就是说其返回值受本地时区影响。**比如两个实例的时区分别是东八区和西八区，其格式化后的时间字符串是不一样的。这会有什么影响呢？这要看具体业务场景，在我们的系统中，对于两个时区不一致的实例，在一个实例上生成的Redis Key在另一个实例上找不到，触发一个死循环，导致消息被无限重放。

## 如何解决

开发人员估计都没有考虑到这个问题，但这的确是在分布式系统中非常容易犯的一个错误。首先分布式系统中时钟同步就是一个很难得问题。对于上面的case，系统能工作的前提是各实例的本地时间不会出现跳变，时间精度至少要精确到秒，但这者几乎很难保证。

本地时间可以理解为实例本身的状态，由于我们使用的是 Unix 时间戳，其实系统本身并不需要依赖机器的本地时区。在代码层面可以通过设置一个固定的时区，用指定的时间戳生成一致的 Unix 时间即可。

## 总结

- 每个程序员都应该了解时间、时区等基本概念；
- 在分布式系统中，如果要使用 Unix 时间戳来访问分布式系统中的共享资源，不要依赖于本地时间；
