# Why
针对数据库事务ACID-Atomicity，单机可以使用write-ahead-log实现1PC（one-phase-commit）即可，但是如果是分布式环境，考虑机器故障，网络不可靠1PC无法完成ACID-Atomicity

# What
2PC（two-phase-commit）是已故图灵奖得主，事务处理领域大师[Jim Gray](http://jimgray.azurewebsites.net/default.htm)提出的，用以解决分布式数据库事务ACID-Atomicity的一种共识(consensus)算法

![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consensus/atomic-commit-protocols/2PC/images/2pc.png)

- Phase 1: 
  - Transaction coordinator首先写日志(write-ahead-log)记录事务执行状态，然后向所有Participants广播PREPARE消息，询问participant是否准备好commit（回复YES）或者选择abort（回复NO）
  - Participant收到PREPARE消息后，开始执行事务（考虑ACID-isolation，此时已经持有各种锁），如果执行中有任何问题则回复abort，如果事务执行完成则回复YES
  - Transaction coordinator收到所有的回复，进入Phase 2
- Phase 2:
  - 如果Ttransaction coordinator超时时间内收到的响应均为YES，则向participants广播COMMIT消息，否则广播ABORT消息（广播之前需更新日志，记录事务执行状态）
  - participant收到COMMIT/ABORT消息后，将事务正式commit/abort（考虑ACID-isolation，commit/abort完成后会释放所有锁）并回复ack

# How
来看异常处理的情况：
- Phase 1:
  - Transaction coordinator（TC）发送PREPARE之后，如果超时时间内未收到响应，则放弃该事务，进入Phase 2 向所有participants广播ABORT
    - 此时收到ABORT的participants会正常终止事务
  - 当Participant收到PREPARE后，如果回复YES的时候超时（无法确定TC是否收到消息），retry几次后进入Phase 2
  - 当Participant收到PREPARE后，如果回复NO的时候超时（无论TC是否收到，TC都会进入Phase 2然后广播ABORT消息），重试几次之后可以主动终止事务
- Phase 2: 
  - TC发送了COMMIT/ABORT消息之后，如果长时间没有收到ack或者宕机重启之后都会根据write-ahead-log的内容重新发送消息，直到收到ack为止（无限重试）
  - 一旦进入Phase 2，Participants会失去主动终止或提交事务的权利，只能等待TC发送的COMMIT/ABORT消息，亦或者主动发送get status消息
  - 事务是有一个全局唯一的事务ID唯一确认的，这一点可以确保TC重新发送COMMIT/ABORT消息时恢复连接的participant可以识别并回复ack
  
# Weakness
> 2PC is a blocking protocol

由于TC宕机或者与部分participant断开连接（或者Participant宕机），则意味着阻塞（blocking），直到宕机恢复网络恢复为止。

以TC宕机为例，考虑ACID-isolation 这会导致participant长时间持有lock而不释放，影响participant可用性

# Reference
[1]. [Martin Kleppmann. 《Designing Data-Intensive Applications》9.Consistency and Consensus](http://dataintensive.net/)

[2]. [Sukumar Ghosh. 《Distributed Systems An Algorithmic Approach Second Edition》 14.5 Atomic Commit Protocols](https://www.amazon.com/Distributed-Systems-Algorithmic-Approach-Information/dp/1466552972)

[3]. [Notes on Data Base Operating Systems. Jim Gray. IBM Research Laboratory. San Jose, California. 95193. Summer 1977](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consensus/atomic-commit-protocols/2PC/DBOS.pdf)
