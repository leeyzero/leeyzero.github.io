---
title: 'MIT 6.824 Lab2C: 持久化'
date: 2022-12-14 09:09:45
categories:
- 分布式系统
tags:
- 分布式系统
- Raft
- MIT6.824
---

上一篇 {% post_link mit-6-824-lab2b-log-replication Lab2B: 日志复制 %} 中介绍了Raft日志复制的实现，本篇介绍Raft持久化的原理和实现。实验二共包含四个子实验：

- {% post_link mit-6-824-lab2a-leader-election Lab2A: 选主 %}
- {% post_link mit-6-824-lab2b-log-replication Lab2B: 日志复制 %}
- {% post_link mit-6-824-lab2c-persistence Lab2C: 持久化 %}
- {% post_link mit-6-824-lab2d-log-compaction Lab2D: 日志压缩 %}

本文是第三个子实验，需要实现Raft持久化。持久化是指将Raft的部分状态存储到非易失的存储介质中，这样即便节点崩溃，也不至于丢失数据，当服务恢复后，可以从之前持久化的状态处开始工作。在开始实验前，先阅读以下材料：

- 阅读论文：[In Search of an Understandable Consensus Algorithm](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)，以下简称论文。
- 阅读实验指导：[6.824 Lab 2: Raft Part2C](https://pdos.csail.mit.edu/6.824/labs/lab-raft.html)

Raft持久化本来是属于日志复制的一部分，但上一篇中我们已经看到，日志复制的原理和实现已经比较复杂了，所以课程将持久化单独拆分成了独立的一个子实验。

Lab2C比想象的简单，不是它本身就简单，而课程设置的比较简单。在实际生产环境中，持久化一般会将日志写入磁盘，涉及到磁盘I/O操作以及对内存状态的序列化与反序列化。在这个实验中不需要进行这样操作，实验中已经实现了一个Persister对象，将Raft状态的持久化封装成接口。它的实现也比较简单，只是模拟了持久化（实际上还是存储在内存）。

可能这个实验面向的人群主要是学生，主要实验目的是想让学生明白Raft持久化原理和意义，认为持久化到磁盘只是一种实现方式，并不是本实验的重点吧。但对一个软件工程师而言，熟练掌握磁盘I/O操作是一项基本功。

## 基本原理

Raft持久化是指将Raft状态序列化后存储到存储介质，本实验无需关心存储介质的问题，所以我们只需要对Raft状态进行序列化和反序列化。实现中提供了labgob包可能对任意类型的状态进行序列化和反序列化，这里就不展开了，大家看看labgob包的测试用例就会使用。

那我们需要持久化哪些Raft状态呢？

再次回顾论文Figure 8（这个图非常非常重要）中的左上图，需要对以下三个状态进行持久化：

- currentTerm：当前任期，在Lab2A中已经介绍过了。
- votedFor：当前任期中，投票给了谁，在Lab2A中已经介绍过了。
- log[]：日志，每一项是一个LogEntry，在上一篇中已经定义过了。

那我们什么时候持久化Raft状态呢？

按照论文中的说明，节点在追加日志后，需要将日志持久化。追加日志有两种情况：一种是Leader接受客户端命令，将命令作为一条新的日志记录追加到日志中；另一种是Follower复制Leader的日志记录追加日志。所以我们只需要在这两种追加日志的情况下，持久化Raft状态即可。

<!--more-->

## 实现

在Lab2C中，我们需要实现Raft状态的持久，主要包括三个部分：

- persis：持久化，也就是Raft状态序列化，存储到持久对象Persister中。
- readPersis：加载持久化数据，也就是将持久化对象中的数据反序列化为Raft状态。
- 日志持久化：在日志记录发生变更后持久化。

### persist

persist 负责将Raft中的currentTerm、votedFor、log状态序列化后持久化到Persister中。该函数在日志追加成功后调用。

```golang
func (r *Raft) persist() {
    w := new(bytes.Buffer)
    e := labgob.NewEncoder(w)
    e.Encode(r.CurrentTerm())
    e.Encode(r.votedFor)
    e.Encode(r.log)
    state := w.Bytes()
    r.persister.SaveRaftState(state)
}
```

### readPersist

readPersis 负责将Persister对象中持久化的数据反序列化成Raft状态。该函数在Raft初始化时调用。

```golang
func (r *Raft) readPersist(data []byte) {
    if data == nil || len(data) < 1 { // bootstrap without any state?
        return
    }

    var currentTerm int
    var voteFor int
    var log []*LogEntry
    buf := bytes.NewBuffer(data)
    d := labgob.NewDecoder(buf)
    if d.Decode(&currentTerm) != nil ||
        d.Decode(&voteFor) != nil ||
        d.Decode(&log) != nil {
        Warning("raft.readPersist: server[%v] at term[%v] read persist state failed", r.me, r.currentTerm)
    } else {
        r.currentTerm = currentTerm
        r.votedFor = voteFor
        r.log = log
    }
}
```

### 日志持久化

Leader接收命令追加日志成功后持久化。

```golang
func (r *Raft) processCommandRequest(req *CommandArgs) (*CommandReply, error) {
    entry := r.createLogEntry(req.Command)
    if err := r.appendLogEntry(entry); err != nil {
        return nil, err
    }

    r.nextIndex[r.me] = entry.Index + 1
    r.matchIndex[r.me] = entry.Index

    // Added for Lab2C
    r.persist()
    return newCommandReply(entry.Index, entry.Term), nil
}
```

AppendEntries RPC 接收端在成功追加日志后持久化。

```golang
func (r *Raft) processAppendEntriesRequest(req *AppendEntriesArgs) (*AppendEntriesReply, bool) {
    ...
    if err := r.appendLogEntries(req.Entries); err != nil {
        Info("raft.processAppendEntriesRequest: server[%v] appendLogEntries[%v] err[%v]", r.me, len(req.Entries), err)
        return newAppendEntriesReply(r.CurrentTerm(), false, r.currentLogIndex(), 0), true
    }

    // Added for Lab2C
    if len(req.Entries) > 0 {
        r.persist()
    }
    ...
}
```

## 测试结果

最后附上Lab2C的测试结果

```shell
$ go test -v -race --run 2C
=== RUN   TestPersist12C
Test (2C): basic persistence ...
  ... Passed --   3.4  3  118   32539    6
--- PASS: TestPersist12C (3.37s)
=== RUN   TestPersist22C
Test (2C): more persistence ...
  ... Passed --  16.0  5 1572  370754   16
--- PASS: TestPersist22C (15.98s)
=== RUN   TestPersist32C
Test (2C): partitioned leader and one follower crash, leader restarts ...
  ... Passed --   1.6  3   46   12601    4
--- PASS: TestPersist32C (1.59s)
=== RUN   TestFigure82C
Test (2C): Figure 8 ...
  ... Passed --  30.1  5 1236  276815   38
--- PASS: TestFigure82C (30.10s)
=== RUN   TestUnreliableAgree2C
Test (2C): unreliable agreement ...
  ... Passed --   3.0  5  224   84238  246
--- PASS: TestUnreliableAgree2C (2.98s)
=== RUN   TestFigure8Unreliable2C
Test (2C): Figure 8 (unreliable) ...
  ... Passed --  34.0  5 5096 7809480  465
--- PASS: TestFigure8Unreliable2C (33.99s)
=== RUN   TestReliableChurn2C
Test (2C): churn ...
  ... Passed --  16.2  5 1376 1153923  633
--- PASS: TestReliableChurn2C (16.18s)
=== RUN   TestUnreliableChurn2C
Test (2C): unreliable churn ...
  ... Passed --  16.1  5 1228  940915  468
--- PASS: TestUnreliableChurn2C (16.15s)
PASS
ok  	6.824/raft	120.385s
```

## 参考资料

[1] [In Search of an Understandable Consensus Algorithm](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)
[2] [6.824 Lab 2: Raft Part2C](https://pdos.csail.mit.edu/6.824/labs/lab-raft.html)