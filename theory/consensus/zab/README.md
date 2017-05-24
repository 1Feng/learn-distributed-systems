# Summary
> zab是Yahoo提出的leader-base的一致性协议，由于raft晚于该协议猜测raft中有借鉴该协议的一些思想
> 此文仅总结理解的一些重点，并不完整总结该算法

## FLP？

zab 中使用了timeout来进行故障检测，并没有突破FLP

## Zxid

- 高32位：代表epoch，与raft-term或multi-paxos的proposal number语意相同，与raft-term的不同点是自增的时机是在成为leader后
- 低32位：自增id，等同与multi-paxos的instance-id/instance-index 或 raft-log-index

## BroadCast
> Zab broadcast依赖与FIFO（TCP）+ zxid 来保证消息的顺序（causal order + total order）；paxos并不依赖于此而是靠proposal number来保证这一点；而raft则是通过log-index来保证的

Zab的broadcast本质就是放弃了abort动作的2PC协议,即：
- 2PC中P1阶段可以由Participant选择YES or Abort，而Zab-BroadCast的P1阶段follower只能回复YES（即ACK），或者选择放弃该leader


## Recovery

recovery 需要在正确性上保证以下两点：
1. 不要忘记已经交付的消息
2. 忽视应该跳过的消息（即leader 已经 broadcast，但是未获得多数派确认，后续leader又有新的提交，则该消息应该被忽视/放弃）

方法：
- 选举leader时需保证leader拥有多数派认同的最大的zxid；与raft的log-up-to-date语意一致
- 通过epoch来避免宕机恢复的leader提交应忽略的消息；与raft的term作用一致




## Reference
[1]. [A simple totally ordered broadcast protocol](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consensus/zab/A_simple_totally_ordered_broadcast_protocol.pdf)

[2]. [ZooKeeper Internals](http://zookeeper.apache.org/doc/r3.5.0-alpha/zookeeperInternals.html#sc_atomicBroadcast)
