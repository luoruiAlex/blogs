- paxos算法主要解决的问题就是如何保证分布式系统中**各个节点都能执行一个相同的操作序列**

### 概念
- 角色：**proposers**，**acceptors**，和learners，此外，议题最开始来源于Client
  - 允许一个节点身兼数职，只是使用不同端口
  - Acceptor们启动不同端口的TCP监听，Proposer来主动连接即可
  - Zookeeper自身包含了Acceptor, Proposer, Learner
    - Zookeeper领导选举就是paxos过程
    - Client对Zookeeper写Znode时，也是要进行Paxos过程的
    - Znode包含了写入Client的IP与端口，其他的Client也可以读取到这个Znode来进行Learner
- 基本约束
  - value(决议)只有被proposer提出后才能被批准(未经批准的决议称为提案proposal)
  - 一次算法执行实例中，只批准(chosen)一个value(多数acceptor接受了同一个value)
  - learners只能获得被批准（chosen）的value 
- 引申约束
  - 一个acceptor必须accept第一次收到的proposal
    - 当且仅当acceptor没有回应过编号大于n的prepare请求时，acceptor接受（accept）编号为n的提案
  - 一旦一个具有value v的提案被批准（chosen），那么之后批准（chosen）的提案必须具有value v(给提单分配变化)
    - 一旦一个具有value v的提案被批准（chosen），那么之后任何acceptor再次接受（accept）的提案必须具有value v
    - 一旦一个具有value v的提案被批准（chosen），那么以后任何proposer提出的提案必须具有value v
    - P2c：如果一个编号为n的提案具有value v，那么存在一个多数派，要么他们中所有人都没有接受（accept）编号小于n 的任何提案，要么他们已经接受（accept）的所有编号小于n的提案中编号最大的那个提案具有value v

### 算法
#### 行为
- Proposer提出议题
- Acceptor初步接受 或者 Acceptor初步不接受
- 如果上一步Acceptor初步接受则Proposer再次向Acceptor确认是否最终接受
- Acceptor 最终接受 或者Acceptor 最终不接受
- Learner最终学习的目标是Acceptor们(所有的Acceptor)最终接受了什么议题(多数Acceptor接受的提议)
#### 决议的提出与批准
- prepare阶段
  - proposer选择一个提案编号n并将prepare请求发送给acceptors中的一个多数派
  - acceptor收到prepare消息后，如果提案的编号大于它已经回复的所有prepare消息，则acceptor将自己上次接受的提案回复给proposer，并承诺不再回复小于n的提案
- 批准阶段
  - 当一个proposer收到了多数acceptors对prepare的回复后，就进入批准阶段。它要向回复prepare请求的acceptors发送accept请求，包括编号n和根据P2c决定的value（如果根据P2c没有已经接受的value，那么它可以自由决定value）。
  - 在不违背自己向其他proposer的承诺的前提下，acceptor收到accept请求后即接受这个请求
#### 决议的发布
- acceptors需要将accept消息发送给learners的一个子集，然后由这些learners去通知所有learners
- 由于消息传递的不确定性，可能会没有任何learner获得了决议批准的消息。当learners需要了解决议通过情况时，可以让一个proposer重新进行一次提案。注意一个learner可能兼任proposer
#### Progress的保证
- 当一个proposer发现存在编号更大的提案时将终止提案
- 如果两个proposer在这种情况下都转而提出一个编号更大的提案，就可能陷入活锁，违背了Progress的要求。这种情况下的解决方案是选举出一个leader，仅允许leader提出提案。
