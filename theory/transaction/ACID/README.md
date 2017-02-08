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

1. ACID-consistency是一个比较模糊的概念，状态迁移是系统的用户来保证的，系统只能保证其中一部分，不能完全覆盖，所以consistency依赖与用户而不是系统
2. 微软给出的例子[[2]](https://msdn.microsoft.com/en-us/library/aa480356.aspx)和Atomicity类似，但是差别在于A中事务终止回滚时因为system fails，而C中事务终止回滚是因为error（比如类型不匹配，数字和字符串做加法？）
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
先看几点要求：
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
- Read-Modify-Write / atomic write operation
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
    
##Durability ★★★★

磁盘+replica

#References

2.[ACID properties](https://msdn.microsoft.com/en-us/library/aa480356.aspx)
