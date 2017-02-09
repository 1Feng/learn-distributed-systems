#What
##Atomicity
**描述**：

一个事务包含一系列的操作，这一系列的操作都成功，则意味着事务执行成功；一旦执行过程中发生故障(fault)，数据库需要放弃整个事务，并且撤销已经完成的部分操作

**优势**：

方便异常处理，如果事务终止，应用层面可以确保什么修改都没有发生，可以安全的重试

**典型案例**：

A向B账户转账100元：
 1. 从A的账户减少100元
 2. 从B的账户增加100元
 
如果1执行完成2还未执行，此时数据库故障(`system fails`)，则为了保证Atomicity，数据库的事务系统需要回滚1操作

**其他**：
> 这里需要与concurrency-atomic做一下区分, concurrency-atomic指的是当某个线程执行某个操作时，其他线程不可能看到中间状态(half-finished)

##Consistency

**描述**：

这里的consistency是指，当事务结束时，系统（数据库）处于一个合法的状态(valid state),也就是说系统总是从一个合法的状态迁移至另一个合法的状态

**其他**：

1. ACID-consistency是一个比较模糊的概念，状态迁移是系统的用户来保证的，系统只能保证其中一部分，不能完全覆盖，所以consistency依赖用户而不是系统
2. MSDN给出的例子[[2]](https://msdn.microsoft.com/en-us/library/aa480356.aspx)和Atomicity类似，但是差别在于A中事务终止回滚时因为system fails，而C中事务终止回滚是因为error（比如类型不匹配，数字和字符串做加法？）
2. ACID-consistency 和CAP-consistency直接没有任何关系，仅仅使用了同一个单词而已


##Islation
**描述**：

Isolation是指当多个事务并发(concurrency)执行时，应该彼此之间存在隔离，执行过程中互不影响

##Durability

**描述**：

一旦事务成功提交，即使发生硬件故障或者程序崩溃，任何已经写入的数据都不能丢失


#How
##Atomicity ★★★★
可以利用持久化日志来实现，方便重启回滚
##Consistency ★★
数据库层面做足够的合法性检测，其他由用户层/应用层来保证
##Islation ★★★★★
**先看几点要求**：
- Read commited（weak-islation type） 的两点要求
  - No Dirty Read: 不会读取到其他正在执行的事务中间状体的数据
  - No Dirty Write: 事务不会overwrite到其他事务的uncommitted的数据
- No Read Skew：
  - Read Skew举例：
    - A 在两个账户中各存放了500块钱，现在A要查询两个账户的余额
    - 查询账户1的SQL执行完成，余额500
    - 假设A之前设置了一笔定时的自动转账被触发，从账户2向账户1转100块，事务执行成功，账户1余额600，账户2余额400
    - 查询账户2的SQL执行完成，余额400
    - 在A看来，账户总额少了100块
    - 即使如此这个场景还是可以接受的，因为A可以重新查询，即可获得正常结果
  - 无法接受Read Skew的两个场景：
    - Backup
      - 事务执行的同时，可以完成数据备份 
    - Analytic Queries and Integrity checks
      - 事务执行的同时, 需要完成大量数据的查询或扫描
- Read-Modify-Write / Atomic Write Operation
  - 举例：两个用户同时对一个counter字段做inc操作，后果与多线程并发操作类似会丢失一部分inc操作
- Write Skew
  - 举例（针对multi-object的场景）：
    - 两位医生Alice 和 Bob同时检查当前是否有另外一个人正在值班，如果有则在系统中停止自己的值班状态，然后回家睡觉
    - Alice执行事务如下：
    ```sql
      currently_on_call = (select count(*) from doctors where on_call = true and shift_id = 1234)
      if (currently_on_call >= 2) {
        update doctors set on_call = true where name=‘Alice’ and shift_id = 1234
      }
    ```
    - Bob执行事务如下：
    ```sql
      currently_on_call = (select count(*) from doctors where on_call = true and shift_id = 1234)
      if (currently_on_call >= 2) {
        update doctors set on_call = true where name=‘Bob’ and shift_id = 1234
      }
    ```
    - 有点像是multi-object版本的read-modify-write，但是有本质区别

**解决方案**：
- Read commited
 - Dirty Write: 可以使用row-level lock来避免dirty write
 - Dirty Read: 
  - 同样可以使用row-level lock来避免dirty read,但是缺点在于一个比较耗时的写操作会阻塞住read-only的操作，更严重的是会因此引发连锁反应
  - 更好的解决方法是使用类似于MVCC的snapshot-isolation方案来解决dirty read的问题
- No Read Skew
 - 类似于MVCC的snapshot-isolation方案来解决read skew问题，可同时满足Backup和Analytic Queries and Integrity checks的需求
- Read-Modiry-Write / Atomic Write Operation
 - 使用显示的锁操作(explicit-locking)来实现atomic write operation
 - automatically-detecting-lost-update，一旦检测到lost update，事务需要终止并且retry
 - 实现compare-and-set操作用以支持SQL-where语句
- Write Skew
 - 串行化（serializability）隔离所有事务，这种方式可以解决上述除read skew外所有问题，但是工程实现上往往性能会是一个非常大的问题

> 通常为了实现isolation，都是综合以上各种方案


##Durability ★★★★

磁盘+replica


#Serializability
##What
> serializable-isolation 是最强等级的事务并发隔离，他可以确保即使多个事务是并行(parallel)执行的,最终的结果看起来也像是顺序的（serially），每个时间点只有一个事务在执行

##How
> 根据上述描述，不难看出，其要求是让数据库解决所有的可能的并发竞争问题

- 真的串行化的执行事务：
 - 方法：将所有的事务扔到一个队列里排队，由特定的线程来依次执行
 - 缺点：性能太差
- 存储过程（stored procedures）+ in-memory data：
 - 解释：本质是加快单个事务的执行速度（没有了磁盘IO），以便可以真正串行化事务执行
 - 缺点：存储过程需要用户自己来用SQL/PL完成，调试测试监控都比较棘手，同时一旦用户完成的存储过程性能比较差，会造成恶劣的影响，甚至引发连锁反应
- 数据分区(partitioning)
 - 解释：本质是将单机的性能问题通过scale out来加速
 - 缺点：事务执行涉及的数据不能跨分区
- Two-Phase-Locking(2PL)
 - 描述：
    - 当事务需要读一个object时，必须先以shared mode获取锁；多个事务可以同时以shared mode获取锁，但是一旦有事务以exclusive mode持有了锁，其他事务必须等待
    - 如果事务想要写一个object，必须先以exclusive mode获取锁；区别于shared mode，同一时间只能有一个事务以exclusive mode持有锁
    - 如果事务先读一个object，然后又要写（read-modify-write）,则需要将锁从shared mode升级为exclusive mode
    - 一旦事务获取了锁，除非事务提交或者终止，否则不允许释放锁，这也是二阶段命名的由来；
 - 解释：
    - Expanding phase（扩大阶段--事务执行中）: locks are acquired and no locks are released.
    - Shrinking phase（收缩阶段--事务结束时）: locks are released and no locks are acquired.
 - 缺点：
    - 吞吐量(through-put) 和 响应时间 与仅实现weak-isolation(如read-commit + No Read Skew)相比会比较差
    - deadlock风险增大
- Serializable Snapshot Isolation(SSI)
 - 与之前提到的snapshot-isolation相比，SSI为写操作增加了串行(serialization)冲突检测
    - detecting stale MVCC reads：针对write skew，如果事务提交时检测到之前的前置条件已经不成立了，则终止事务
    - detecting writes that affect prior read：同样考虑write skew，数据库从index-level/table-level保存一些信息，以便当事务提交后可以检测其操作是否造成其他正在执行的事务读取的数据过期（前置条件失效），如果存在则主动通知该事务终止

 
##Serializability VS Linearizability
 - serializability： 事务隔离的属性，指事务执行的结果看起来像顺序的（串行的），以避免write skew
 - linearizability： 指对读写共享数据的新近性（recency guarantee），与事务（把一系列操作看做整体来讨论）无关


#References

[1]. [Martin Kleppmann. 《Designing Data-Intensive Applications》7.Transactions](http://dataintensive.net/)

[2]. [ACID properties](https://msdn.microsoft.com/en-us/library/aa480356.aspx)
