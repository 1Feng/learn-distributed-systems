#Why
1983年由Dale Skeen 和 Michael Stonebraker[提出](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consensus/atomic-commit-protocols/3PC/A-Formal-Model-of-Crash-Recovery-in-a-Distributed-System.pdf)了3PC协议来解决2PC阻塞的问题
#What
3PC（two-phase-commit）其实就是将2PC的Phase 2拆分成了两个阶段：

**时序图**

![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consensus/atomic-commit-protocols/3PC/images/3pc.png)

- Phase 1: 
  - Transaction coordinator(TC)首先写日志(write-ahead-log)记录事务执行状态，然后向所有Participants广播PREPARE消息，询问participant是否准备好commit（回复YES）或者选择abort（回复NO）
  - Participant收到PREPARE消息后，开始执行事务（考虑ACID-isolation，此时已经持有各种锁），如果执行中有任何问题则回复abort，如果事务执行完成则回复YES
  - TC收到所有的回复，进入Phase 2
- Phase 2:
  - 如果TC收到的响应均为YES，则向participants广播PRE-COMMIT消息，否则广播ABORT消息（广播之前需更新日志，记录事务执行状态）
  - 如果participant收到PRE-COMMIT消息，回复ACK
  - 如果participant收到ABORT消息，终止事务
- Phase 3:
  - 如果TC在超时时间内收到所有的ack，则向participants广播COMMIT消息，否则广播ABORT消息（广播之前需更新日志，记录事务执行状态）
  - Participant收到COMMIT/ABORT消息后，将事务正式commit/abort（考虑ACID-isolation，commit/abort完成后会释放所有锁）并回复ack

#How


**状态迁移图**
![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consensus/atomic-commit-protocols/3PC/images/3pc-state-machine.gif)

来看异常处理的情况：
- Phase 1:
  - Transaction coordinator（TC）发送PREPARE之后，如果超时时间内未收到响应，则放弃该事务，进入Phase 2 向所有participants广播ABORT
    - 此时收到ABORT的participants会正常终止事务
  - 当Participant收到PREPARE后，如果回复YES的时候超时（无法确定TC是否收到消息），retry几次后进入Phase 2
  - 当Participant收到PREPARE后，如果回复NO的时候超时（无论TC是否收到，TC都会进入Phase 2然后广播ABORT消息），重试几次之后可以主动终止事务
- Phase 2: 
  - TC发送了PRE-COMMIT/ABORT消息之后，如果长时间没有收到ack或者宕机重启之后都会进入Phase 3，发送ABORT消息
  - Participants如果长时间没有收到PRE-COMMIT消息，则可以主动终止事务
  - Participants如果收到PRE-COMMIT后，回复ack之前发生宕机，则可以主动终止事务
- Phase 3: 
  - TC发送了COMMIT/ABORT消息之后，如果长时间没有收到ack或者宕机重启之后都会根据write-ahead-log的内容重新发送消息，重试几次后结束（如果是发送COMMIT，则意味着TC认为事务已经完成；ABORT消息同理）
  - Participants如果长时间没有收到COMMIT/ABORT消息，执行commit
  
#Weakness

> 3PC是一个理想状态的协议，假设fail-stop模型，并且可以通过timeout来准确判断网络故障还是宕机的情景(synchronous systems)下的协议（上文我们是按照真实环境来分析解析的），并且仅支持single-failure

- 所以典型的一个3PC的冲突情景如下：
  - Phase 2 TC 广播PRE-COMMIT消息，如果P1在收到消息前宕机，因而TC在Phase 3广播ABORT消息
  - 在Phase 2，P2回复ack之后进入Phase 3，并且与TC直接发生网络分区(network-partition)导致P2无法收到ABORT消息，故而自行决定commit
- 网络通信需要3 RTT，开销较大

其他:
- 标准的3PC是理想状态下，是fail-stop（the server only exhibits crash failures，且不恢复）模型
- 标准的3PC描述Phase 3时，如果TC收到多数(majority)的ack（其他的认为宕机了），即可广播COMMIT（没有收到ack则意味着participant宕机且不恢复）
- 根据以上两点，所以标准的3PC在synchronous systems（有限的timeout）下是可行的方案（上文的典型冲突情景不再发生）


PS： 
- 根据[F·L·P定理](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consensus/F-L-P/README.md)在asynchronous system 模型下实现分布式共识是不可能的，但是实践之中我们能尽可能的去达成共识

#Reference
[1]. [D. Skeen and M. Stonebraker, “A Formal Model of Crash Recovery in a Distributed Systems,” IEEE Transactions on Software Engineering, SE-9, 3, (May 1983), pp. 219–228.](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consensus/atomic-commit-protocols/3PC/A-Formal-Model-of-Crash-Recovery-in-a-Distributed-System.pdf)

[2]. [Sukumar Ghosh. 《Distributed Systems An Algorithmic Approach Second Edition》 14.5 Atomic Commit Protocols](https://www.amazon.com/Distributed-Systems-Algorithmic-Approach-Information/dp/1466552972)

[3]. [Three-Phase Commit Protocol](http://courses.cs.vt.edu/~cs5204/fall00/distributedDBMS/sreenu/3pc.html)

[4]. [Distributed Systems W4995-1 Fall 2014 lecture17 ](https://roxanageambasu.github.io/ds-class//assets/lectures/lecture17.pdf)
