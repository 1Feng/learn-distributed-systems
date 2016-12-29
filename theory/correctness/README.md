#Introduce
> 一般正确性的证明标准有两个，分别是safety properties 和 liveness properites


##Safety Properites
>通常safety properites是指：“bad things never happen”。

####举例
例如互斥操作(不管单机还是分布式)的safety properites可以是:
- 最多只能有一个process or thread进入临界区
- 至少有一个process or thread有资格进入临界区


##Liveness Properites
>通常liveness properites是指："good things eventually happen"。


对应现实世界的一个例子就是“正义终将来临”，至于具体什么时候，不太好说。

liveness properites的描述经常带有"eventually"字样，例如eventually consistency就是liveness properites。

####举例
例如互斥操作(不管单机还是分布式)的liveness properites可以是:
- 每个试图进入临界区的process or thread最终都将进入临界区
- 至少有一个process or thread有资格进入临界区


##Proof
常见的证明方式暂时不做了解 ☻
