#Introduce
>对于CAP而言，partition-tolerant是客观的必须要做的，不然不能称之为分布式系统；而consistent和available则是主观的，
>应当根据业务需求适当调整的。相对于linearizability的强一致，其实还有多种弱一致性模型可以供系统设计时参考, 这里着重描述两种重要的一致性模型

##Data-centric consistent models
###Causal Consistency
> 与linearizability相同，causal consistency同样属于data-centric consistent models。与前者明显的区别在于，linearizability的系统的所有操作都存在total order，而causal consistency只需要partial order即可。

####定义：
> 对于所有的进程看到的所有的写操作，都是因果相关的（causally related）且顺序相同。所有的读操作看到的结果也需要和写的因果顺序一致

如图：
![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consistency/weak-consistency/images/causal-consistency.png)

两次写操作没有因果关系（concurrence），所以后续的两个client的读结果不相同，但这符合causal consistency的定义

####How

实现causally related partital order即可，例如vector clock + causal order multicast protocol

##Client-centric consistent models

###Eventual Consistency

> 最终一致性比较好容易理解，很多primary-backup(asynchronous)架构的RDBMS都是使用的最终一致性模型

####定义：
> 如果没有新的更新/写入，最终所有的clients都会看到最新的数据

最终是多久，不好说...

####典型例子：

DNS系统

####How

 asynchronous log shipping + gossip protocal

##References
1. [《Distributed Systems An Algorithmic Approach Second Edition》16.3 16.4](https://www.amazon.com/Distributed-Systems-Algorithmic-Approach-Information/dp/1466552972)
