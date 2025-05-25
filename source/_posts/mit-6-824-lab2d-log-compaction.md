---
title: 'MIT 6.824 Lab2C: 日志压缩'
date: 2022-12-14 09:10:09
categories:
- 分布式系统
tags:
- 分布式系统
- Raft
- MIT6.824
---

上一篇 {% post_link mit-6-824-lab2c-persistence Lab2C: 持久化 %} 中介绍了Raft持久化的实现，本篇介绍Raft日志压缩原理和实现。实验二共包含四个子实验：

- {% post_link mit-6-824-lab2a-leader-election Lab2A: 选主 %}
- {% post_link mit-6-824-lab2b-log-replication Lab2B: 日志复制 %}
- {% post_link mit-6-824-lab2c-persistence Lab2C: 持久化 %}
- {% post_link mit-6-824-lab2d-log-compaction Lab2D: 日志压缩 %}

本文是第四个子实验，需要实现Raft日志压缩，在开始实验前，先阅读以下材料：

- 阅读论文：[In Search of an Understandable Consensus Algorithm](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)，以下简称论文。
- 阅读实验指导：[6.824 Lab 2: Raft Part2D](https://pdos.csail.mit.edu/6.824/labs/lab-raft.html)

## 基本原理

Raft的日志会随着接收客户端的请求数而不断增长，对实际生产环境中的系统而言，内存是有限的，较大的日志，在服务崩溃重启时需要较长时间加载恢复，这会造成服务的可用性问题。日志压缩主要解决这些问题。

Raft采用快照（Snapshot)来实现日志压缩。主要原理时，将当前整个系统的状态作为一个快照（可以理解为一个复本，相当于保证了当前的系统状态）持久化到稳定存储中，然后就可以丢弃这个快照点之前的所有日志了。论文中给出了一个快照示例如下：

<img src="/images/mit-6-824-lab2d/figure12.png" width=500 />

上图中，在快照上，日志长度为7。在index=5处生成了一个快照（snaphot），快照中除记录了状态机的状态：`x <- 0, y <- 9`外，还记录了本次快照包含的最后一个日志记录的index和term（下面会解释为什么需要存储这两个信息）。快照成功后，我们就可以丢弃index=5之前（包括index=5）的所有日志了，日志压缩后长度为2，内存得以释放。

<!--more-->

那快照中为什么需要保存last included index和last included term呢？

因为快照操作并不是只能在Leader上进行。虽然Raft是强领导人，但作者认为，日志复制在各节点上已经达成一致，快照只是对日志的重新组织，并不违背Raft的强领导人特性。另一方面，因为各个节点本地已经拥有快照的所有信息，所有要求Leader来控制各节点的快照，将会导致节点之前大量数据传输，会占用带宽，同时也会增加领导人的实现复杂度。所以快照是可以在Leader和Follower中进行。

但有个问题是，Leader和Follower虽然在大多数情况下日志是同步的，但当新节点加入或节点日志滞后的情况下，如果我们对Leader做了快照，移除了快照部分的日志记录。如果快照部分的日志还没有同步给Follower，Follower怎么复制日志呢？这就是为什么需要保存快照最后一个日志记录的index和term的原因。有了这两个信息还需要配合一个InstallSnapshot RPC操作，才能让Follower也在相同的位置执行快照。论文中Figure 13介绍了InstallSnapshot RPC的实现细节，在此就不展示了。

需要注意的是，**只能对已经提交的日志才能做快照**。快照除了涉及Raft本身提供的能力外，还涉及到状态机的概念，Raft中提供的对外API Install，入参中的snapshot参数就是由状态机生成的。可以结合测试用例一起看，测试用例是Raft的调用者，相当于Raft的用户，在实际系统中，它就是上层server。Lab2D引导中给了一个Raft和上层应用（比如K/V store service）的一个完整交互：

<img src="/images/mit-6-824-lab2d/raft_diagram.png" width=800 />

上层服务决定什么时候需要进行快照（通常是日志大小到达一定阈值）。Raft提供对外API Snapshot，Raft做快照时，会保存状态机的全部状态和快照中的最后一个日志记录的index和term，并进行持久化。对于快照后的Leader，在发送AppendEntries RPC之前，会先判断要同步给Follower的日志是否已经在快照中了，如果已经在快照中了，则发送InstallSnapshot RPC请求。

Follower在接收到InstallSnapshot RPC请求后，如果满足快照条件，会将快照应用到状态机，状态机应用快照成功后，会调用CondInstallSnapshot，此时Follower才真正开始在本地做快照。

## 实现

Lab2D的实现需要增加一些Raft状态，这些状态需要在InstallSnapshot RPC中发送给Follower。实现主要包括三个部分：

- Snapshot
- CondInstallSnashpot
- InstallSnapshot RPC

在实现之前，先看下我们需要增加哪些Raft状态。

### Raft状态

本实验中，我们需要新增以下Raft状态：

- lastIncludedIndex：最近一次快照的最后一个日记记录的index，需要持久化，初始化为0。
- lastIncludedTerm：lastIncludedIndex对应的任期，需要持久化，初始化为0。
- snapshot[]：最近一次快照时状态机的状态。

相应的我们需要修改persit和readPersist，同时新增persistStateAndSnapshot用于快照时持久化Raft状态和Snapshot。

```golang
func (r *Raft) persist() {
    state := r.dumpState()
    r.persister.SaveRaftState(state)
}

func (r *Raft) persistStateAndSnapshot() {
    state := r.dumpState()
    r.persister.SaveStateAndSnapshot(state, r.snapshot)
}

func (r *Raft) readPersist(data []byte) {
    if data == nil || len(data) < 1 { // bootstrap without any state?
        return
    }

    var currentTerm int
    var voteFor int
    var lastIncludedIndex int
    var lastIncludedTerm int
    var log []*LogEntry
    buf := bytes.NewBuffer(data)
    d := labgob.NewDecoder(buf)
    if d.Decode(&currentTerm) != nil ||
        d.Decode(&voteFor) != nil ||
        d.Decode(&lastIncludedIndex) != nil ||
        d.Decode(&lastIncludedTerm) != nil ||
        d.Decode(&log) != nil {
        Warning("raft.readPersist: server[%v] at term[%v] read persist state failed", r.me, r.currentTerm)
    } else {
        r.currentTerm = currentTerm
        r.votedFor = voteFor
        r.lastIncludedIndex = lastIncludedIndex
        r.lastIncludedTerm = lastIncludedTerm
        r.log = log
        r.commitIndex = r.lastIncludedIndex
        r.lastApplied = r.lastIncludedIndex
        r.snapshot = r.persister.ReadSnapshot()
    }
}

func (r *Raft) dumpState() []byte {
    w := new(bytes.Buffer)
    e := labgob.NewEncoder(w)
    e.Encode(r.CurrentTerm())
    e.Encode(r.votedFor)
    e.Encode(r.lastIncludedIndex)
    e.Encode(r.lastIncludedTerm)
    e.Encode(r.log)
    return w.Bytes()
}
```

### Snapshot

Leader和Follower无可以进行Snapshot操作，所以Leader循环和Follower循环进行对Snapshot事件进行处理。只能对已经提交的index进行Snapshot，如果满足快照条件，需要对日志进行压缩，然后更新Raft状态和状态机状态，并持久化。

有一点需要注意，日志压缩时，为了让垃圾回收收到已经压缩的日志，我们不能再使用同一个slice。快照逻辑实现如下：

```golang
func (r *Raft) processSnapshot(req *SnapshotArgs) *SnapshotReply {
    if req.Index <= r.lastIncludedIndex || req.Index > r.commitIndex {
        return newSnapshotReply(false)
    }

    entry := r.getLogEntry(req.Index)
    if entry == nil {
        return newSnapshotReply(false)
    }

    r.compactLog(req.Index)

    r.lastIncludedIndex = entry.Index
    r.lastIncludedTerm = entry.Term
    r.snapshot = req.Snapshot

    // save raft state and snapshot to persist
    r.persistStateAndSnapshot()

    return newSnapshotReply(true)
}

func (r *Raft) compactLog(index int) {
    // Usually the snapshot will contain new information not already in the recipient’s log.
    // In this case, the follower discards its entire log;
    entry := r.getLogEntry(index)
    if entry == nil {
        r.log = []*LogEntry{}
        return
    }

    afterEntries := r.log[r.getInternalLogIndex(index)+1:]
    newLog := make([]*LogEntry, len(afterEntries))
    copy(newLog, afterEntries)
    r.log = newLog
}
```

### CondInstallSnapshot

CondInstallSnapshot的调用时机是Follower接收到InstallSnapshot RPC请求后，将快照应用到状态机时，由状态机调用的。需要注意的是，为了保证Follower快照时的原子性，也就是说提交快照时不能再提交新的请求。我们需要在CondInstallSnapshot中先校验是否在快照提交给状态机期间有新的请求被提交。实现如下：

```golang
func (r *Raft) processCondInstallSnapshotRequest(req *CondInstallSnapshotArgs) *CondInstallSnapshotReply {
    // 说明从快照提交到状态机到状态机调用CondInstallSnapshot期间，有新的日志提交到状态机中
    if req.LastIncludedIndex <= r.commitIndex {
        return newCondInstallSnapshotReply(false)
    }

    r.compactLog(req.LastIncludedIndex)

    r.lastIncludedIndex = req.LastIncludedIndex
    r.lastIncludedTerm = req.LastIncludedTerm
    r.commitIndex = req.LastIncludedIndex
    r.lastApplied = req.LastIncludedIndex
    r.snapshot = req.Snapshot

    r.persistStateAndSnapshot()
    return newCondInstallSnapshotReply(true)
}
```

### InstallSnapshot RPC

发送端Leader在AppendEntries需要先判断待发送的日志是否已经做过快照，如果是，说明Follower中还没有这些日志记录，而这些日志记录已经做了快照，需要发送InstallSnapshot RPC。

处理InstallSnapshot RPC的返回跟AppendEntries RPC类似，但更简单，除了同步term之外，需要更新nextIndex和matchIndex。

```golang
func (r *Raft) broadcastAppendEntries() {
    for peer := range r.peers {
        if peer == r.me {
            continue
        }
        if r.killed() {
            Debug("raft.broadcastAppendEntries: server[%v] state[%v] at term[%v] killed", r.me, r.State(), r.CurrentTerm())
            break
        }

        prevLogIndex := r.getPrevLogIndex(peer)
        if prevLogIndex >= r.lastIncludedIndex {
            entries, prevLogTerm := r.getLogEntriesAfter(prevLogIndex, MaxLogEntriesPerRequest)
            req := newAppendEntriesArgs(r.CurrentTerm(), r.leader, prevLogIndex, prevLogTerm, r.commitIndex, entries)
            r.asyncSendAppendEntries(r.me, peer, req)
        } else {
            req := newInstallSnapshotArgs(r.CurrentTerm(), r.leader, r.lastIncludedIndex, r.lastIncludedTerm, r.snapshot)
            r.asyncInstallSnapshot(r.me, peer, req)
        }
    }
}

func (r *Raft) asyncInstallSnapshot(server int, peer int, req *InstallSnapshotArgs) {
    go func(server int, peer int, req *InstallSnapshotArgs) {
        var reply InstallSnapshotReply
        if ok := r.sendInstallSnapshot(peer, req, &reply); !ok {
            return
        }
        target := &InstallSnapshotReplyEvent{
            Peer:  peer,
            Req:   req,
            Reply: &reply,
        }
        r.sendAsync(target)
    }(r.me, peer, req)
}

func (r *Raft) processInstallSnapshotReply(peer int, req *InstallSnapshotArgs, reply *InstallSnapshotReply) error {
    if reply.Term < r.CurrentTerm() {
        return nil
    }

    if reply.Term > r.CurrentTerm() {
        r.updateCurrentTerm(reply.Term, LEADER_NONE)
        return nil
    }

    r.nextIndex[peer] = req.LastIncludedIndex + 1
    r.matchIndex[peer] = req.LastIncludedIndex
    return nil
}
```

接收端（Follower）实现除了同步term之外（通过规则），需要做一个幂等操作，即：如果接收到lastIncludedIndex已经是旧的快照，不需要再应用到状态机。需要注意的是，Follower在处理InstallSnapshot RPC时，只是将快照提交到状态机，并没有进行日志压缩，因为我们不能保证应该状态机一定成功，所以需要状态机应用成功后再次调用CondInstallSnapshot，然后Raft在进行实际的快照操作。

另外，我对待Apply的消息做了一个队列，由一个单独的线程进行处理，这样就不会因为状态机而阻塞事件循环了。实现如下：

```golang
func (r *Raft) processInstallSnapshotRequest(req *InstallSnapshotArgs) (*InstallSnapshotReply, bool) {
    if req.Term > r.CurrentTerm() {
        r.updateCurrentTerm(req.Term, req.LeaderId)
    }

    reply := newInstallSnapshotReply(r.CurrentTerm())
    if req.Term < r.CurrentTerm() || req.LastIncludedIndex <= r.lastIncludedIndex {
        return reply, true
    }

    // apply snapshot to state machine
    msg := ApplyMsg{
        SnapshotValid: true,
        Snapshot:      req.Snapshot,
        SnapshotTerm:  req.LastIncludedTerm,
        SnapshotIndex: req.LastIncludedIndex,
    }
    r.applyQueue <- msg
    return reply, true
}

func (r *Raft) startLoop() {
    ...
    r.wg.Add(1)
    go func() {
        defer r.wg.Done()
        r.applyLoop()
    }()
}

func (r *Raft) applyLoop() {
    for msg := range r.applyQueue {
        r.applyCh <- msg
        r.sendAsync(msg) 
    }
}

func (r *Raft) processApplyMsg(applyMsg *ApplyMsg) {
    if !applyMsg.CommandValid {
        return
    }
    r.lastApplied = applyMsg.CommandIndex
}
```

## Q&A

### Follower接收到InstallSnapshot时为什么不进行日志压缩？

因为Raft和状态机是属于两个不同层次的服务，为了不让状态机应用操作阻塞事件循环，应用操作是一个异步操作，在一个单独的线程中执行。我们有两种实现方式：

一种是在Raft中进行同步处理，即先在应用线程中等待状态进执行完成后在processApplyMsg中进行日志压缩操作。但如果在提交快照和状态机应用快照完成期间，Raft又提交了新的请求，此时我们不能不能再把commitIndex回退到lastIncludedIndex，但在processApplyMsg中我们也法通过状态机，所以这样实现是存在问题的。

另一种方式是让状态机应用快照完成后，再次通过ConInstallSnapshot告诉Raft，如果Raft在这个期间没有新的日志提交，则Raft才真正开始做快照，成功后告诉状态机，状态机再真正应用快照操作。如果Raft失败，则Raft和状态机均没有应用快照成功。这样就保证了Raft和状态机在应用快照时的一致性。

### 日志压缩中如果index未找到，为什么要移除全部日志？

论文日志压缩有这样一句话：

> Usually the snapshot will contain new information not already in the recipient’s log. In this case, the follower discards its entire log; it is all superseded by the snapshot and may possibly have uncommitted entries that conflict with the snapshot.

大概意思是说，接收者的日志中可能并没有包含快照中的一些日志记录，在这种情况下，Follower需要丢弃全部日志，即便是这些日志跟快照有冲突，仍然以快照为准。这也是可以理解的，因为最终我们是要复制Leader的日志，现在收到Leader的快照必定是已经提交过的，根据Raft的安全性保证，在Follower中丢弃日志是安全。

需要注意的是，只能是Follower在做日志压缩时才能这么做，而Leader不行（想想Raft安全行保证的第2条）。所以上面processSnapshot的实现中，在日志压缩前先校验了index的存在性。

### 事件循环线程和应用线程同步如何同步？

事件循环线程在应用命令/快照时，是先将命令/快照消息发送到应用消息channel中，由应用线程消费该channel再应用到状态机。退出时，需要先让事件循环退出，再关闭应用线程。应用线程只是循环读取applyQueue中的消息，我们只需要在事件循环结束后，关闭applyQueue channel即可。如下：

```golang
func (r *Raft) startLoop() {
    ...
    go func() {
        defer r.wg.Done()
        r.loop()

        // only loop exit, we can close applyQueue channel
        close(r.applyQueue)
    }()
    ...
```

## 测试结果

最后附上Lab2C的测试结果

```shell
$ go test -v -race --run 2D
=== RUN   TestSnapshotBasic2D
Test (2D): snapshots basic ...
  ... Passed --   3.9  3  148   56846  251
--- PASS: TestSnapshotBasic2D (3.90s)
=== RUN   TestSnapshotInstall2D
Test (2D): install snapshots (disconnect) ...
  ... Passed --  37.8  3 1640  479895  377
--- PASS: TestSnapshotInstall2D (37.84s)
=== RUN   TestSnapshotInstallUnreliable2D
Test (2D): install snapshots (disconnect+unreliable) ...
  ... Passed --  46.8  3 2020  546698  345
--- PASS: TestSnapshotInstallUnreliable2D (46.77s)
=== RUN   TestSnapshotInstallCrash2D
Test (2D): install snapshots (crash) ...
  ... Passed --  28.1  3 1014  308631  366
--- PASS: TestSnapshotInstallCrash2D (28.08s)
=== RUN   TestSnapshotInstallUnCrash2D
Test (2D): install snapshots (unreliable+crash) ...
  ... Passed --  31.5  3 1128  325411  388
--- PASS: TestSnapshotInstallUnCrash2D (31.46s)
PASS
ok  	6.824/raft	148.084s
```

## 参考资料

[1] [In Search of an Understandable Consensus Algorithm](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)
[2] [6.824 Lab 2: Raft Part2D](https://pdos.csail.mit.edu/6.824/labs/lab-raft.html)