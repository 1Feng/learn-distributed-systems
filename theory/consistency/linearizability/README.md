#Introduce
> 所谓的linearizability其目的在于描述系统的数据，对外看起来就像只有一份，所有针对这部分数据的操作都是原子(ACID-atomic)的；在分布式系统领域来讲和CAP-consistent是等价的；在多核并发编程时由于存在CPU-Cache一致性问题，linearizability的概念同样适用。

##What
定义同CAP-consistent

> 任意的一条读操作R，如果发生在某条写操作W完成之后（或执行过程中），那么R读到的要么是W的内容，要么是W之后的写操作写入的内容

这里的定义与CAP-consistent略有出入，为什么放宽限制为`或执行过程中`呢？因为定义之中所有的之前之后是否完成都是所谓`上帝视角`来判定的；对于client而言只有`clients之间额外的交流沟通`（参考后文），而对于`clients之间额外的交流沟通`而言，W完成与否也是无法判定的，考虑即使是执行W的client，也只能拿到W完成的响应时间，并不能真正知道server端W完成的时间（中间有网络延迟，物理时钟有误差等），即使利用因果关系进行`clients之间额外的交流沟通`也无从考证真正完成的时序。因此W是否真的完成并意义不大。

结合图示来看：
![Alt text](https://github.com/1Feng/learn-distributed-systems/blob/master/theory/consistency/linearizability/images/linearizability.png)
一旦有client读取到了写入的值，即使这个写入操作还没有完成，那么后续的读取操作都应该能读到该值或者之后写入的值


##Why
- 对于clients而言，一旦存在`额外的交流沟通的渠道`，linearizability问题就会凸显，例如：
  - A,B两个人去刷飞机票，A刷到了，B没有刷到（显示全部售光），如果A,B之间没有交流，即使B刷票先于A，则交易看起来也没有什么问题
  - 但如果A,B两个人存在交流，例如B没有刷到票，然后跑去隔壁房间问A，恰巧碰到A正在刷，并且刷票成功（B刷票 happened before A刷票），则交易存在问题
- 如果能够提供linearizability的分布式系统，则：
  - 可以利用该系统实现分布式锁操作
  - 利用锁操作又可以用进行leader-election
  - 利用锁操作可以达成uniqueness guarantees
  - 提供全局单调递增的fencing number

##How
如果可以保证分布式系统的各操作时序可比较（total order），则linearizability可达成；所以linearizability的实现问题又转换成了如何解决fault-tolent total order broadcast

##Weakness
同实现fault-tolent total order broadcast

##References
1. [Martin Kleppmann. 《Designing Data-Intensive Applications》9.Linearizability](http://dataintensive.net/)