### CLH队列
- FIFO双向队列
- Node
  - 代表一个线程，保持线程引用、状态waitStatus、前驱节点pre、后继节点next
  - 状态：
    - 0：新节点，无状态
    - CANCELLED表示节点被取消(超时或者中断)，该节点不再被唤醒，会被踢出队列等待GC回收
    - SIGNAL：节点的继任节点被阻塞了，需要unpark()唤醒它。新加入的节点的前置节点必须状态为SIGNAL，因为只有前置节点为SIGNAL时当前节点才可能被唤醒 
    - CONDITION：节点在条件队列中，基于Condition对象发生了等待，需要Condition对象来激活
    - PROPAGATE：使用在共享模式头结点有可能处于这种状态，表示锁的下一次acquireShared可以无条件传播
  - 入列：addWaiter(Node mode)
    - 如果队列不为空，先通过comppareAndSetTail快速设置尾节点，设置成功则直接返回
    - 否则enq死循环保证安全地设置尾节点(第一个Node入列会创造一个傀儡节点，不代表任何线程、waitStatus=0)
  - 出列：头结点释放同步状态，唤醒后继节点，后继节点获取状态成功时断开头节点的next和本节点的pre(不需要CAS)
- setHeadAndPropagate：将当前节点设置为头结点，并无条件传播，以确保后续节点能被正常唤醒
  
### 获取同步状态
#### 独占式获取
- acquireQueue()根据公平性原则来自旋，直到获取锁为止
