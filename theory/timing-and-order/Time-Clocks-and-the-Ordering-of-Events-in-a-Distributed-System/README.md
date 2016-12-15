#Summary
###Happend before
用→来表示`hanppend before`，对于任意event a, b 有：

1. 如果a和b属于同一个process，并且a comes before b, 则 a → b
2. 如果a是某个process发送信息的event，b是另一个process接收该信息的event，那么 a → b
3. 如果 a → b并且 b → c，那么 a → c

以上本质是基于一个因果关系(causality)来定义的`hanppend before`

`concurrence`意味着a → b不成立并且b→a也不成立，即a,b之间缺少因果关系

b →  c 并且 a  →  c, 但是a,b并不能推导出因果关系，因此`happend before`是partial order.
同时由于a → a不成立，所以`happend before`是反自反(irreflexive)的partial order


### logical clocks

定义Ci(b)为event b在process i 上发生时的clock。

对于任意的events a,b：
> 如果a → b,则C(a)< C(b)

显而易见：

1. 如果a,b同属于process Pi, 并且 a comes before b, 则C(a) < C(b)
2. 如果a是Pi上发送信息的event，b是Pj上接收该信息的event，那么Ci(a) < Cj(b)


具体实现：

1. 对于任意Pi在两个successive event之间会增加Ci, Ci += 1
2. 以下
  - a. 如果a是Pi上发送信息的event，信息m包含一个时间戳Tm = Ci(a)
  - b. 当Pj收到信息m，设置Cj = max(Cj, Tm) + 1

Logical Clock 的缺点：a, b可能同时发生，C(a) < C(b)并不能推断出a → b

###total ordering
> In mathematics, a linear order, total order, simple order, or (non-strict) ordering is a binary relation on some set X, which is antisymmetric, transitive, and total. A set paired with a total order is called a totally ordered set, a linearly ordered set, a simply ordered set, or a chain. ---- from wikipedia

定义关系=>如下：
>如果a属于Pi，b属于Pj，a => b当且仅当要么Ci(a) < Ci(b)要么Ci(a) = Ci(b) 并且Pi < Pj

Pi < Pj可以是process name 字典序或者数字标示的顺序。

paper中举例使用total ordering解决分布式情况下mutual exclusion的问题（案例中假设所有process都不会fail，也没有network partiton）

值得特别强调的一点，这里的total ordering和`hanppend before`没有关系，但是total ordering的意义在于可以用在例如mutual exclusion场景，用顺序来保证fairness（一般的mutual exclusion的关系是FIFO来保证fairness的）


### Anomalous Behavior

例如：

1. event a : P 发送消息到R
2. event b :  P发送消息到Q，Q将消息转发给R

对于P而言 a→b,但是由于网络延迟R就不一定这么认为了。

解决方法有两种：

1. 发送的消息中带上logical clock
2. 利用Physical Clock


###Physical Clocks
大概介绍了什么样（主要指同步）的physical clock可以用来解决上述的问题。
