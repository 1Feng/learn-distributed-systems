# Introduce

> 众所周知TCP是可靠的网络传输协议，但是为什么在分布式系统中又认为网络是不可靠的呢？通常有以下两点：
> 1. 发送方无法确定接收方已经收到请求
> 2. 发送方无法无法知晓接收方是否处理完请求
>
> 可以看出，以上指的都是从应用层的角度观察的结果，而引起以上问题的原因可能有：
> - 消息在路由队列中等待转发
> - 接收方队列满，发生丢包
> - 接收方处理完成，回复的消息在排队或发生丢包
> - gc-stop-the-world等

## Synchronous network

像电话网络，有线电视网络等都是所谓synchronous network，他的特点如下：
- 一旦连接建立，即享用专线
- 专线享有固定的带宽
- 路由(routers)没有队列

以上决定了synchronous network的最大网络延迟是固定有上限的，即可以用timeout来判断消息传输是否存在问题


## Asynchronous network

既然有synchronous network为什么还要搞以太网这一套呢？原因是为了充分利用带宽，由于互联网上数据传输的大小都不是固定的，使用专线意味着带宽资源的浪费。

因此，Ehernet && ip 使用了packed-switched协议, 具体如下：
1. 路由引入队列，最大化线路使用率
2. TCP层引入send buffer && recv buffer来动态的适配数据传输速率(滑动窗口)

上述的优化本质是在latency和resource utilization之间做trade-off，也因此导致了无上限的延迟时间，即无法选择一个合适的timeoout来进行传输故障检测
