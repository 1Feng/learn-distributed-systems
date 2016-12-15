#Summary
###Why
Lammport Clock(Logical Clock) 只能通过因果关系推断其Logical Clock的关系，即：

- 如果a → b, 则C(a) < C(b), 反过来并不一定成立，例如：
  - 同一个proces上的两个事件由a → b 得到C(a) < C(b),
  - 但是a,b可能因为和另一个process上的事件c没有因果关系处于并发状态
  - 但是按照Lammport的描述的Logic Clock的实现，C(c)很有可能满足 C(a) < C(c) < C(b)
  - 然而实际情况是c和a,c和b均无因果关系
  
Vector Clock的出现就是为了解决上述问题。

### What
假设有n个processes，V为n个processes上的事件集合，a,b∈V；

对于vector clock 如果VC(a) < VC(b),仅且仅当：
- ∀i: 0 <= i <= n - 1: VCi(a) <= VCi(b)
- ∃j: 0 <= j <= n - 1: VCj(a) < VCj(b)

通俗的讲就是向量维度匹配并且VC(a)的所有维度都不大于VC(b)并且至少有一个维度小于VC(b),这时候VC(a) < VC(b)

同时：`VC(a) < VC(b)   <==> a  → b`

###How

processes编号0--n-1, VC利用数组实现，初始为[0,0,0...0]

1. 对于process i，本地的VC为VCi,对于任意事件发生后 ++VCi[i]
2. 当向其他process发送数据时，带上本地的VC
3. 当process j接收到VCi时
  - ++VCj[j]
  - ∀k : 0 <= k <= n - 1:  VCj[k] = max(VCi[k], VCj[k])
  
###Weakness

1. partial order not total order
  - 无法满足VC(a) < VC(b)时还是无法解决order问题。dynamo论文中的提到的处理方式是将该问题抛给client根据业务处理（PS：dynamo据说已经不用vector clock了）
2. vector size 随着processes数量线性增长
  - Riak开发者提供了一种[解决方案](http://basho.com/posts/technical/why-vector-clocks-are-hard/),在vector clock中带上各自processes的本地time stamp，当vector size到达指定的阈值后，删除最旧的process的数据；这样造成的问题就是丢失了和最旧的process的因果关系，按照作者的说法，好在这并不会造成数据丢失，just a tradoff！
  
### References
1. [Vector Clock In Wikipedia](https://en.wikipedia.org/wiki/Vector_clock)
2. [《Distributed Systems An Algorithmic Approach Second Edition》 6.3 Vector Clock](https://www.amazon.com/Distributed-Systems-Algorithmic-Approach-Information/dp/1466552972)
3. [Why Vector Clocks Are Hard](http://basho.com/posts/technical/why-vector-clocks-are-hard/)
