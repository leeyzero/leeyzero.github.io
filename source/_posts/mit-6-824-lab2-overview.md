---
title: 'MIT 6.824 Lab2: 概述'
date: 2022-12-14 09:07:35
categories:
- 分布式系统
tags:
- 分布式系统
- Raft
- MIT6.824
---

[6.824](https://pdos.csail.mit.edu/6.824/schedule.html) 是[MIT](https://www.mit.edu/)推出的一个分布式系统课程，讲师是大名鼎鼎的[Robert Tappan Morris](https://en.wikipedia.org/wiki/Robert_Tappan_Morris)。[Lab2](https://pdos.csail.mit.edu/6.824/labs/lab-raft.html) 是课程中的第二个实验，实验要求需要用Go语言实现Raft。[Raft](https://raft.github.io/)是为可理解而设计的共识算法（consensus algorithm），它在性能和容错性上等价于[Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science))，但结构却完全不一样。Raft通过减少状态空间和将问题分解为几个独立的子问题，使得Raft更容易理解，也更利于工程实现。

在开始实验前，需要先阅读以下材料：

- [6.824 Schedule: Spring 2022](https://pdos.csail.mit.edu/6.824/schedule.html)
- [In Search of an Understandable Consensus Algorithm](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)
- [Lab Guidance](https://pdos.csail.mit.edu/6.824/labs/guidance.html)
- [Students' Guide to Raft](https://thesquareplanet.com/blog/students-guide-to-raft/#an-aside-on-optimizations)

以上材料是必读的，在YouTube上有一个配套的[教学视频](https://www.youtube.com/watch?v=64Zp3tzNbpE&ab_channel=MIT6.824%3ADistributedSystems)，英文比较吃力的同学可以在B站看翻译后的[视频](https://www.bilibili.com/video/BV1r7411R7TL/?spm_id_from=333.788.top_right_bar_window_default_collection.content.click&vd_source=fe426ffb35d856437b6ec091c7ac0e56)。

共识算法是分布式系统最核心的部分，也是非常难的部分，Paxos的主要问题是难以理解，而且作者[Leslie Lamport](http://lamport.azurewebsites.net/pubs/pubs.html)在论文中并没有给出具体的实现细节，正如Chubby的实现者所述：

> There are significant gaps between the description of the Paxos algorithm and the needs of a real-world system. . . . the final system will be based on an unproven protocol.

大概意思是说，Paxos算法的描述和现实世界实际需求存在着显著差距，最终的系统都是基于未经证明的协议。

<!--more-->

Raft正如论文题目[In Search of an Understandable Consensus Algorithm](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)所示，寻找一种可理解的共识算法，其设计初衷是为了可理解性。Raft共识算法将共识协议拆分成三个独立的部分：

- 选主（Leader election）
- 日志复制（Log replication）
- 安全性（Satety）

以上是Raft的必选部分，安全性是对选主以及日志复制安全性的证明。除此之外还有是成员变更（Membership changes）以及日志压缩，作为可选部分。对于现实生产环境而言，日志压缩是一个必选的特性。

共识算法通常应用在复制状态机中，在一个复制状态机架构中，应用服务接受客户端请求，并转化为状态机命令后提交给共识模块，共识模块负责将日志复制给集群中的其它节点。当共识模块认为日志安全复制后，会将日志应用到状态机中，应用成功后返回客户端请求。复制状态机架构如下：

<img src="/images/mit-6-824-lab2-overview/figure-1.png" width=500 />

Raft的工作原理是，Raft首先选举出一个leader，然后由leader负责接收上层应用服务的命令请求。leader接收到命令后，首先将命令记录到日志，然后复制给集群中的其它节点。当超过半数的节点日志复制成功后，leader提交日志，并将日志应用到状态机中。

实验二将Raft共识算法的实现拆分成四个子实验，分别是：

- {% post_link mit-6-824-lab2a-leader-election Lab2A: 选主 %}
- {% post_link mit-6-824-lab2b-log-replication Lab2B: 日志复制 %}
- {% post_link mit-6-824-lab2c-persistence Lab2C: 持久化 %}
- {% post_link mit-6-824-lab2d-log-compaction Lab2D: 日志压缩 %}

接下来就分别介绍这四个子实验中实现的一些细节和问题，算是自己的一个学习笔记。需要特别强调的是，一定要自己动手实现。论文中除了介绍原理外，还详细介绍了实现细节，但有些细节在论文中并没有给出，如果需要通过4个实验中的所有测试用例，需要自己填补这些空白。下面给一些自己的学习建议：

- 多读几遍论文（我至少读了5遍以上），特别是对Figure 2中的每一个Raft状态和RPC交互，都要深入理解，要不然会写出一大堆BUG。
- 理解测试用例，测试用例是系统的一个外部用户，通过测试用例，可以从一个外部使用者的角度了解Raft是怎么工作的。
- 日志一定要打印出当前节点信息和任期（Term）信息，调度日志最好打印出完整的RPC交互日志。实验一个难点在于RPC交互有大量的并发操作，它们的交互顺序是不确定的，这要求我们在实现时要充分考虑各种异常情况。当测试用例失败时，需要快速从日志中分析出具体原因，没有足够的日志信息，排查问题会让你崩溃。
- 有些细节在论文并没有给出，比如日志冲突的优化，先自己想想怎么实现，没有思路时[Students' Guide to Raft](https://thesquareplanet.com/blog/students-guide-to-raft/#an-aside-on-optimizations)是一份不错的参考资料。
