#Introduce

> 于2002年提出的CAP理论（三选二的方式来评估分布式系统）确实为分布式系统领域的发展提供了指导价值，但是就今天而言，这套理论已经意义微小了

##Consistent

这里的一致性指的是强一致，又称[linearizable](https://github.com/1Feng/learn-distributed-systems/tree/master/theory/linearizability)或atomic。

论文中的描述如下：

> Under this consistency guarantee, there must exist a total order on all operations such that each operation looks as if it were completed at a single instant.

简单来讲就是如果把分布式系统看做一个黑盒，在外部看起来这个系统就是和单机没有区别。

具体的来说：

> 任意的一条读操作R，如果发生在某条写操作W完成之后，那么R读到的要么是W的内容，要么是W之后的写操作写入的内容

更详细的描述可以参考[linearizable](https://github.com/1Feng/learn-distributed-systems/tree/master/theory/linearizability)

这里的consistent 和 ACID中的consistent是完全不同的概念：
- ACID-consistent特指事务
- CAP-consistent仅仅是请求/响应操作顺序的属性

##Available
论文中的定义：
>For a distributed system to be continuously available, every request received
>by a non-failing node in the system must result in a response

这里的response是指no-error response

即使是Probabilistic availability，在任意的failures发生时也不会影响针对CAP-available的结论，但是这里为了简单起见特指100% availability。


如果专门针对partition-tolerance而言的话，available可以描述为：

>even when severe
>network failures occur, every request must terminate.

terminate 是指任意使用该分布式系统的算法都会终止，注意是算法的终止。

##Partition Tolerance

> 网络割接和交换机故障都会造成network partition

network partition 图示:

![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/out-of-date-cap-theorem/images/network-partition.png)

CAP的问题也是从这里开始体现：

- partition tolerance并非和CA对等的属性，而是一种因果的关系：partition发生时是选A还是选C，即如何去tolerant partition，
- 分布式系统需要考虑的其他[网络问题](https://github.com/1Feng/learn-distributed-systems/tree/master/theory/unreliable-network)也很多，包括延迟，网络不可靠等，并不仅仅是partition，所以使用CA,CP,AP去描述一个分布式系统并不完整
- 很多分布式系统可以根据业务需求降低对consistent的要求，降低对available的要求，所以根本无法用CAP来描述

##Partition in practice

> 尽管network partition不能涵盖分布式系统所有需要面对的网络问题，但是它确实是网络问题中的一个难点和重点

###single-leader-Architecture
![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/out-of-date-cap-theorem/images/single-leader.png)

当某个client和leader处于不同partition时，此时CAP-available丢失，如果按照CAP理论，只能称之为CP

###multi-leader-Architecture

**情景一**

![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/out-of-date-cap-theorem/images/multi-leader-c.png)

某个client和所有的leader都不在一个partition，此时CAP-available丢失，如果按照CAP理论，只能称之为CP

如果你允许（业务上允许）图示中的client2对replica进行read操作，则CAP-consistent也会丢失，只能称之为P（CAP的3选2现在成了3选1）
**情景二**

![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/out-of-date-cap-theorem/images/multi-leader.png)

leaders不在一个partition，此时CAP-consistent丢失，如果按照CAP理论，只能称之为AP

###dynamo-style-Architecture(no-leader)
![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/out-of-date-cap-theorem/images/dynamo-style.png)

R + W > N,但是当network partition发生时，如果某个client被划分到了节点较少的一侧，那么CAP-available丢失，只能称之为CP；

如果你允许（业务上允许）图示中的client2进行read操作，则CAP-consistent也会丢失，只能称之为P（CAP的3选2现在成了3选1）

##References
1. [ Martin Kleppmann. please-stop-calling-databases-cp-or-ap](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)
2. [Martin Kleppmann. 《Designing Data-Intensive Applications》9.Linearizability](http://dataintensive.net/)
