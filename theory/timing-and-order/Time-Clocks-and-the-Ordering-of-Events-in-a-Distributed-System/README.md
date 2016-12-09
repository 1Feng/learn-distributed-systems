#Summary
###Happend before
用→来表示`hanppend before`，对于任意event a, b 有：

1. 如果a和b属于同一个process，并且a comes before b, 则 a → b
2. 如果a是某个process发送信息的event，b是另一个接收该信息的process的event，那么 a → b
3. 如果 a → b并且 b → c，那么 a → c  

以上本质是基于一个因果关系(causality)来定义的`hanppend before` 

`concurrence`意味着a → b不成立并且b→a也不成立，即a,b之间缺少因果关系,因此`happend before`是partial order.
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

定义关系=>如下：
>如果a属于Pi，b属于Pj，a => b当且仅当要么Ci(a) < Ci(b)要么Ci(a) = Ci(b) 并且Pi < Pj

Pi < Pj可以是process name 字典序或者数字标示的顺序。

paper中举例使用total ordering解决共享资源请求顺序的问题（案例中假设所有process都不会fail，也没有network partiton）

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

数学公式较多，没细看。
