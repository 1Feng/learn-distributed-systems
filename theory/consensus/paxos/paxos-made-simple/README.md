# Summary
> paxos算法的的核心思想是“与其预测未来，不如限制当下”，即通过保证当前的操作，来一步一步达到预期


# Theory
## 要求

**Safety**:
- 只有一个被提议的value被选定(validity)
- 两个不同的进程不能做出不一样的的选择(agreement)

**Liveness**
- 最终会有被提议的value被选定
- 如果一个value被选定，任意进程最终一定会得知这一结果

## 推导过程

首先设定三个角色 proposers，acceptors，learners。

> 要想有value被选定，则acceptor必须要接受proposer的提议，于是我们要求


- **P1. 任意acceptor必须接受(accept)它收到的的第一个提议(proposal)**

> 那么问题来了，多个proposers会提议多个value，无法满足safety.
> 于是我们考虑让acceptor可以接受（accept）多个提议，为了便于区分，我们考虑为提议增加一个total order的序号(proposal number)，即提议由proposal number + value组成
> 但是最终我们是要选定(chosen)一个value的, 于是我们考虑可以接受多个提议，但是我们必须保证这些提议的value都是一样的，于是我们进一步要求：

- **P2. 如果value为v的提议被选定(chosen)，则所有number更大的且被选定的提议的value也必须为v**

> 一个提议如果被选定(chosen)，那么至少被一个acceptor接受(accepted)过, 所以我们可以通过满足如下条件来达成P2

- **P2a. 如果value为v的提议被选定(chosen)，那么所有number更大的且被任意acceptor接受过（accepted）的提议其value也必须是v**

> 考虑一个acceptor c从没有收到提议，此时一个从故障中恢复的proposer发起了一个更高number的提议，且该提议与已经chosen的value不一样。按照P1，c肯定会accept该提议,
> 这样便违反了2a。于是我们强化一下P2a的要求

- **P2b. 如果value为v的提议被选定(chosen)，那么由proposer发起的number更大的提议的value也必须是v**

> P2b通过限定proposer的动作来满足P2，通过归纳法我们可以得知，只要保证如下规则，就可以满足P2b

- **P2c. 那么对于大多数(majority)acceptors,我们称之为集合S；如果一个提议(n，v)被发起，则要么1成立，要么2成立**
  - **S中不存在acceptor接受过(accepted) number 小于n的提议**
  - **v是S接受过的(accepted)所有提议里number小于n的提议中number最大的提议的value**
  
> 只要满足P2c就可以满足P2b，进而满足P2；至此我们便有了更具体的方式来实现P2c,具体如下：
  1. proposer选择一个proposal number n，然后向每个acceptors发起请求，要求acceptors：
    - 保证不再接受(accept)number小于n的提议，并且
    - 如果已经接受过(accepted)number小于n的提议，则这些提议中number小于n的最大的number以及该提议的value返回给proposer
  2. 如果proposer收到大多数(majority)的acceptors的响应，则proposer可以发起一个序号为number的提议，其value是v
    - v是所有acceptor响应的(mi, vi)中最大的m对应的v
    - 如果没有acceptor响应(mi, vi)，则v由proposer自己决定

> 以上我们称之为PREPARE请求。利用PREPARE请求，我们完成了一个学习的过程，从而实现了P2c; proposal的具体实现我们归纳出来了，对应的acceptor的的要求也很容易得出：

- **当且仅当(iff)acceptor 没有响应number大于n的prepare请求时，才可以接受(accept)number为n的提议**

> 由于acceptor收到prepare请求后会保证不再接受(accept) proposal number小于n的提议，则acceptor便没有必要再回复proposal number小于n的prepare请求，我们可以直接忽略，或回复error或null使proposer放弃后续提议.
> 于是我们可以将proposer和acceptor的动作综合起来描述如下：

- **Phase 1**
  - proposer生成一个proposer number n，然后发送prepare请求到所有（其实也可以是majority，但越多越能保证收到过半数的回复）acceptors
  - acceptor收到prepare请求后：
    - 如果之前有收到proposal number > n的prepare请求，则直接忽略该prepare请求，否则
    - 回复该prepare请求，同时如果之前有接受（accept）提议，则回复内容中带上接受的提议value和对应该value的最大的proposal number
- **Phase 2**
  - proposer收到总数过半（majority）的回复后：
    - 如果所有回复中都没有携带提议value，则proposal自己选择一个提议value 
    - 否则从所有回复中选择proposal number最大的的value 
    - 向所有(其实也可以是majority，但越多越能保证收到过半数的accept）acceptor发送上述得到的提议value和proposer number n
  - acceptor收到提议请求后：
    - 如果之前没有回复proposal number > n的prepare请求，则接受（accept）该请求

> 以上可以完成总数过半的acceptor 接受（accept）一个value，但并不代表被chosen，该value被chosen需要：

- **由learner来找出哪个提议（proposal number ＋ value）被总数过半的的acceptors接受了（accepted），方式有如下：**
  - **由接受（accept）提议的acceptor向所有learner发送通知消息，开销 M＊N次通信（假设M个接受该提议的acceptor，N个learner）**
  - **由接受（accept）提议的acceptor向某个learner发送通知消息，由该learner确定chosen结果后再广而告之，开销M＋N次通信**
  - **扩大方法二中某个learner为多个learner，适当增加开销，但可以保证可靠性（learner单点问题）**
  
TO BE CONTINUE!
 TO BE CONTINUE!
