# Summary
> 论文中使用了paxos算法进行副本同步，这里仅总结论文中如何保证read-only操作的linearizability

## How

1. 收到read-only请求后，记录下一个slot number
  - slot number = max(已经commit的最大的operation number VS 当前节点成为leader后re-proposed最大的operation number)

2. 向所有replicas发送消息，检查是否有新leader出现（即检查当前节点是否扔是合法leader）
3. 如果加上自身有总数过半的节点仍然认为当前节点是leader则继续，否则丢弃请求
4. 将请求连同1中记录的slot number转发到任意replica（最好是3中回复确认的的replica），称之为replica A
5. replica A等待slot number被执行，之后检测是否有新的paxos configuration被选择，如果有则丢弃请求，否则执行read操作返回结果

## Why

最简单的保证linearizability的read-only的方法是将read-only操作当做写操作一样走一遍paxos流程，但是这样读的性能太低了，并且会导致leader压力巨大

论文中提出的方法省去了走paxos流程的磁盘IO，仅一次广播检测确认leader角色，并将真正的读操作转移到了replica上

那么如何证明呢？
1. read-only linearizability需要保证的是在这个请求到达之前已经成功提交的写入都应该被本次读取看到
2. 我们将read-only request 到来之前已经成功提交的最后一条写入的operation number为 N，则有以下三种情况：
  - 1. N 是前一个leader提交的
  - 2. N 是当前节点成为leader后提交的
  - 3. 当前节点早已经不是leader， N其实是后续leader提交的 
3. 我们只需保证slot number >= N即可保证linearizability
  - 1. N是前一个leader提交的，当前节点成为leader后re-proposed最大的operation number 一定大于等于N
  - 2. N是当前leader提交的，那么一定有slot number >= N
  - 3. 该情况请求会被丢弃，slot number不需要保证大于等于N
 
## Extension
 
TIDB中在使用raft做数据同步的情况下，也使用了一个类似的[方法](https://zhuanlan.zhihu.com/p/25367435)来保证read-only的linearizability：

>当 leader 要处理一个读请求的时候：
>1. 将当前自己的 commit index 记录到一个 local 变量 ReadIndex 里面。
>2. 向其他节点发起一次 heartbeat，如果大多数节点返回了对应的 heartbeat response，那么 leader 就能够确定现在自己仍然是 leader。
>3. Leader 等待自己的状态机执行，直到 apply index 超过了 ReadIndex，这样就能够安全的提供 linearizable read 了。
>4. Leader 执行 read 请求，将结果返回给 client。

其中：
>实现 ReadIndex 的时候有一个 corner case，在 etcd 和 TiKV 最初实现的时候，我们都没有注意到。也就是 leader 刚通过选举成为 leader 的时候，这时候的 commit index 并不能够保证是当前整个系统最新的 commit index，所以 Raft 要求当 leader 选举成功之后，首先提交一个 no-op 的 entry，保证 leader 的 commit index 成为最新的。

与本文中`N是前一个leader提交的，当前节点成为leader后re-proposed最大的operation number 一定大于等于N`是类似的（raft毕竟是paxos的变种）

另外一种方式就是TIDB根据raft论文实现的lease的方式：

>在 Raft 论文里面，提到了一种通过 clock + heartbeat 的 lease read 优化方法。也就是 leader 发送 heartbeat 的时候，会首先记录一个时间点 start，当系统大部分节点都回复了 heartbeat response，那么我们就可以认为 leader 的 lease 有效期可以到 start + election timeout / clock drift bound 这个时间点。

>为什么能够这么认为呢？主要是在于 Raft 的选举机制，因为 follower 会在至少 election timeout 的时间之后，才会重新发生选举，所以下一个 leader 选出来的时间一定可以保证大于 start + election timeout / clock drift bound。

## Referrence
1. [TiKV 源码解析系列 - Lease Read](https://zhuanlan.zhihu.com/p/25367435)
