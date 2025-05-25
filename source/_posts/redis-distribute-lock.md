---
title: 基于Redis的分布式锁
date: 2021-01-24 09:10:54
categories:
- 分布式系统
tags: 
- 分布式系统
- 分布式锁
- Redis
---

在计算机中，锁是用于解决资源竞争问题。在单机（单进程）环境下, 锁可以使用操作系统提供的同步原语实现。 但在分布式环境下，操作系统提供的同步原语就失效了, 需要一个分布式锁。其原理是需要一个[分布式锁管理器](https://en.wikipedia.org/wiki/Distributed_lock_manager)[1]，提供进程级别访问共享资源的互斥性。本文主要讨论基于Redis实现的分布式锁。

## 分布式锁的使用场景
[Martin Kleppmann](https://martin.kleppmann.com/) 在他的博客 [How to do distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)[2] 中对分布式锁的使用场景做了非常好的概括。从一个更高层次来讲，在一个分布式应用中，使用分布式锁主要有两个原因：效率和正确性。

- **效率**：获取分布式锁可以避免相同的任务被处理两次。如果锁失败，带来的结果就是对某一个子任务多做了一次，其影响完全取决于具体业务场景。比如为了提高消息PUSH效率，需要按用户进行分片，然后每个任务获取一个锁后处理对应的分片，如果锁失败，会造成重复给用户PUSH。
- **正确性**：获取分布式锁是为了防止进程对分布式系统中的资源并发访问而导致系统的状态出现混乱。如果锁失败，两个节点并发的访问同一个共享资源，可能造成数据的不一致性，如文件损坏或数据丢失等。

<!--more-->

## 基于单实例的实现

[Redis](https://redis.io/) 的作者 [antire](http://antirez.com/) 在 [Distributed locks with Redis](https://redis.io/topics/distlock)[3] 中给出了redis单机环境下实现分布式锁的正确做法，其中获取锁：

```shell
SET resource_name my_random_value NX PX 30000
```

其中NX表示，只有当key不存在时才设置成功，PX后跟过期时间30000毫秒，用于设置锁的最长有效时间，才防止客户端崩溃或网络等异常情况下也能够使锁得到释放。之所以对key设置一个随机值，且该随机值需要保证对所有客服端唯一，是为了防止客服端在释放锁时把其他客服端加的锁释放掉。所以在释放锁时需要校验key对应的value值，为了达到原子性，需要使用Lua脚本：

```shell
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

基于单实例的实现存在单点问题。虽然可以使用主从集群规避单点问题，但基于故障转移的实现并不能保证互斥性。以下是一个存在资源竞争的模型：

*1. 客户端A从master实例获得锁。*
*2. master实例在将key同步给slave实例前挂掉。*
*3. slave实例被提升为master。*
*4. 客户端B获取同客户端A的相同资源，客户端B也获得了锁。*

由于主从切换时，key对应的锁丢失，导致客户端A和客户端B均获得了锁，违反了锁的安全性。针对这个问题，antirez提出了Redlock算法，见下文。

## Redlock算法及其讨论
Redlock是客户端侧实现的分布式锁算法，需要N个redis master（N=5是个合理值），客户端在获取锁是，需要依次在N个实例上获取锁，只有当客户端在 N/2+1 个实例上获得锁后，才表示这个客户端获得了锁。具体算法如下： 

*1. 客户端获取当前毫秒时间戳。*
*2.   按照单实例获取锁的方式逐一在N个实例上获取锁。在从每个实例中获取锁期间，超时时间相对于锁的有效期来说，需要设置的比较短，通常在5-50ms。主要是为了防止某个实例挂掉后，可以避免客户端在访问该实例时造成阻塞，以便可以尽快访问下一个实例。*
*3. 客户端在获得锁之后通过减去第1步的时间即可计算出获取锁用的时间。只有当客户端在获取了超过N/2+1个实例所使用的时间小于锁的有效时间时才表示该客户端成功获得了锁。*
*4. 如果客户端获得了锁，锁的有效期为初始有效期减去获取锁所使用的时间。*
*5. 如果客户端没有获得到锁（即客户端获得锁的实例数不足 N/2+1 或获取锁的时间超过锁的有效期）,
则客户端需要释放掉所有实例上的锁。*

Redlock 算法基于以下假设：
*1. 进程间没有同步时间。*
*2. 每个进程间的本地时间流动速率几乎相同，且其误差相较于锁的有效期非常小。*

Martin Kleppmann 在[How to do distribute locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)[2] 中对Redlock做了非常精彩的分析，认为Redlock算法并不是安全的，其主要原因在于以下两点：
*1. 带auto-release的分布式锁需要提供一种机制：在客户端获取锁过期后访问共享资源时任然需要避免违反互斥性。但 Redis并未提供这样的机制。*
*2. 即便在忽略1的情况下，Redlock算法的系统模型在现实系统中并不成立。*

针对以上两点，Redis的作者antirez在他的博客 [Is Redlock safe?](http://antirez.com/news/101)[4] 中做了一一回应。

对于第1点，个人比较赞同antirez的观点，主要原因是对于分布式锁过期后，如果靠DLM去生成一个fencing token，然后让储存线性地接收fencing token较大值。让存储线性化并不一定适用于所有场景，且如何保证生成的 token 是唯一的呢，如果要保证唯一性，就需要read-modify-write，这又需要保证token的一致性。所以说在现实系统中，fencing token 机制也是有很多问题的。

对于第2点，的确可能出现Martin Kleppmann 分析的场景，但在人为管理得当的前提下，机率非常低，在工程上任然有可实践性。如果你需要一个强一致的分布式锁，可以使用像基于 [zookeeper](https://www.oreilly.com/library/view/zookeeper/9781449361297/) 实现的分布式锁。 

## 总结

- 在使用分布式锁时，应该首先要清楚使用分布式锁时是为了效率还是正确性。
- 如果时为了效率，基于redis 单实例的分布式锁能够提供非常好的性能和可操作性。
- 如果时为了正确性，可以使用有强一致保证的分布式协调系统或数据库实现分布式锁。 
- 分布式锁过期后，应该由具体的业务场景去保证一致性，而且总能做到这一点。

## 参考资料

[1] [Distributed lock manager](https://en.wikipedia.org/wiki/Distributed_lock_manager)
[2] [How to do distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
[3] [Distributed locks with Redis](https://redis.io/topics/distlock)
[4] [Is Redlock safe?](http://antirez.com/news/101)
