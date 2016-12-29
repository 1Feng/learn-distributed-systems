#Introduce

>分布式环境面临的两个主要的问题就是网络不可靠和时钟不可靠，这里主要总结时钟问题

##Physical Clocks
我们日常使用的计算机和服务器的物理时钟都是使用的石英(quartz)时钟，这类时钟天生存在误差，虽然铯原子钟的精度更高但是造价昂贵，并不适合商用计算机。

对于商用计算机的时钟误差，通常使用NTP协议来进行时钟同步，然而由于网络的不可靠以及时钟误差NTP同步也会有些问题。

商用计算机利用时英时钟在计算机上实现了两种clock:
- wall clock
  - 受NTP同步的影响，时钟会jump forward 或 jump backward来完成时钟同步
  - 如linux上的int gettimeofday(struct timeval *tv, struct timezone *tz);,返回1970-01-01 00:00:00 +0000 (UTC)至今的秒数和豪秒数
  
- monotonic clock
  - 不受NTP影响，或者，受NTP同步的影响，时钟只会降低或者升高频率，以尽快完成时钟同步
  - 如linux上的int clock_gettime(clockid_t clk_id, struct timespec *tp),clk_id为CLOCK_MONOTONIC_RAW(本质是jiffies)或者是CLOCK_MONOTONIC分别对象上述不受NTP影响和受NTP影响两种

适用性：
- wall clock
  - 适用于：
    - 单机保证时序
  - 不适用：
    - 单机计算duration或elapsed time，例如统计timeout，expire
    - 分布式环境下的时序问题
- monotonic clock
  - 适用于：
    - 单机计算duration或elapsed time，例如统计timeout，expire
    - 单机保证时序
  - 不适用：
    - 分布式环境下时序问题

那么分布式环境下的时序问题如何解决呢?
- 全序(total order)或者高精度的时间点共识(强调某个时间点)：
  - 使用原子钟加更严格复杂的时钟同步策略来保证误差
- 偏序（partial order）：
  - 利用因果关系来解决时序问题，即logic clock
  
##Logic Clock

利用因果关系来实现 [Logic Clock](https://github.com/1Feng/learn-distributed-systems/tree/master/theory/timing-and-order/Time-Clocks-and-the-Ordering-of-Events-in-a-Distributed-System)

利用Logic Clock来保证时序(偏序) [Vector Clock](https://github.com/1Feng/learn-distributed-systems/tree/master/theory/timing-and-order/vector-clock)
  

##其他 
一个错误使用wall clock的[案例](http://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
  
  
##References
1. [《Designing Data-Intensive Applications》8.Unreliable Clocks](http://dataintensive.net/)
  
  
