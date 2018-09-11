## 原理
### CLH队列
- FIFO双向队列
- 仅适用自旋锁，但更容易处理cancel和timeout，所以AQS采用CLH Lock而不是MSC Lock
- 为了处理timeout和cancel，每个node维护一个前驱指针。如果一个node的前驱被cancel，这个node可以前向移动使用前驱的状态字段
### Node
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
  
### 同步状态state
- int类型的成员变量**state**来表示同步状态，state\>0表示已获取锁，state=0表示已释放锁
- getState()、setState(int newState)、compareAndSetState(int expect,int update)
- 自定义子类使用AQS提供的模板方法就可以实现自己的同步语义。Semaphore 用它来表现剩余的许可数，ReentrantLock 用它来表现拥有它的线程已经请求了多少次锁；FutureTask 用它来表现任务的状态(尚未开始、运行、完成和取消)，ReentrantReadWriteLock将32位的state分成高低位，16位写锁计数，16位读锁计数，CountDownLatch代表计数，所有线程都获得锁时，状态为0，开始唤醒。
#### 独占式获取
- acquireQueue()根据公平性原则来自旋，直到获取锁为止。返回线程在等待过程中是否有中断
   - 获取失败，则park等待
- 响应中断
  - 如果提前有中断，直接抛出InterruptedException
  - doActuqireInterruptibly()自旋过程中有中断，不再返回是否有中断，而是抛出InterruptedException
- 超时获取(进一步增强了响应中断)
  - 自旋前判断时间是否小于0，直接返回false
  - 每次自旋都重新计算需要休眠的时间，小于0则返回false
  - 没有超时(休眠时间大于0)
    - 休眠时间小于spinForTimeoutThreshold，则等待，通过LockSupport.parkNanos()
    - spinForTimeoutThreshold 已经非常小了，非常短的时间等待无法做到十分精确，无条件自旋
#### 独占式释放
- tryRelease()成功后，unparkSuccessor(head)
#### 共享式获取
- 自旋中，前驱节点为头结点才会尝试获取
- 获取成功后setHeadAndPropagate：将当前节点设置为头结点，并无条件传播，以确保后续节点能被正常唤醒
 
### 阻塞
- 自旋获取同步状态时有可能阻塞
- shouldParkAfterFailedAcquire
  - 本节点waitStatus=SIGNAL，只有这里返回true
  - 本节点waitStatus\>0(已超时或中断)，返回false之前，本节点从CLH中删除，循环往前寻找到第一个waitStaus\<0的节点，状态设置为SIGNAL
  - 外面的自旋中调用park()阻塞SIGNAL节点
### 唤醒
- 释放同步状态后，则需要唤醒该线程的后继节点
- unparkSuccessor
  - 可能会存在当前线程的后继节点为null，超时、被中断的情况(waitStatus\>0)，从tail开始回溯，找到第一个可用节点(waitStatus\<=0)，唤醒该节点对应的线程

## 应用
### ReentrantLock
- 加锁释放锁委托给Sync的两个子类FairSync和NonfairSync完成
- 获取到同步状态则将exclusiveOwnerThread设置为当前线程
#### 非公平锁
- nonfairTryAcquire
  - state=0表示锁空闲，直接CAS获取
  - 持有锁的线程为当前线程，state增长1，表示重入
  - 非当前线程而且CAS失败，返回false
- tryRelease
  - state减去1
  - 如果state=0，表示完全释放锁，将exclusiveOwnerThread设置为null
  - 返回state是否为0
#### 公平锁
- 区别在于是否按照FIFO的顺序来
- state=0是非公平锁直接CAS获取
- 公平锁先hasQueuedPredecessors判断当前线程是否为头结点，是头结点则尝试获取锁
