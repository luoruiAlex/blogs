- paxos算法主要解决的问题就是如何保证分布式系统中**各个节点都能执行一个相同的操作序列**

### 需要满足的条件
- 只有一个值可能被选中

### 概念
- 角色：proposers，acceptors，和learners（允许身兼数职）
- 基本约束
  - value(决议)只有被proposer提出后才能被批准(未经批准的决议称为提案proposal)
  - 一次算法执行实例中，只批准(chosen)一个value(多数acceptor接受了同一个value)
  - learners只能获得被批准（chosen）的value 
- 引申约束
  - 一个acceptor必须accept第一次收到的proposal
  - 一旦一个具有value v的提案被批准（chosen），那么之后批准（chosen）的提案必须具有value v(给提单分配变化)
    - 一旦一个具有value v的提案被批准（chosen），那么之后任何acceptor再次接受（accept）的提案必须具有value v
    - 一旦一个具有value v的提案被批准（chosen），那么以后任何proposer提出的提案必须具有value v
    - 如果一个编号为n的提案具有value v，那么存在一个多数派，要么他们中所有人都没有接受（accept）编号小于n 的任何提案，要么他们已经接受（accept）的所有编号小于n的提案中编号最大的那个提案具有value v
