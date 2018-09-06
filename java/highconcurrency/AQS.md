### 定义
- **多线程并发的执行，之间通过某种 共享 状态来同步，只有当状态满足 xxxx 条件，才能触发线程执行 xxxx**，这个共同的语义可以称之为同步器
- AQS(AbstractQueuedSynchronizer)是JUC并发包中的核心基础组件
- 解决了子类实现同步时涉及的大量细节问题，比如获取同步状态、FIFO同步队列
- 3个关键点
  - 原子操作同步状态：子类的tryAcquire tryRelease
  - 阻塞或唤醒一个线程：LockSupport
  - 内部维护一个队列：CLH 条件队列

- 控制公平
  - 新到的线程和队列头部的线程一起公平竞争，所以FIFO不一定是公平的
  - 为了保证公平，tryAcquire方法，不在队首返回false即可。
  
### 节点
- 状态：
  - 0：新节点，无状态
  - CANCELLED表示节点被取消(超时或者中断)，该节点不再被唤醒，会被踢出队列等待GC回收
  - SIGNAL：节点的继任节点被阻塞了，需要unpark()唤醒它。新加入的节点的前置节点必须状态为SIGNAL，因为只有前置节点为SIGNAL时当前节点才可能被唤醒 
  - CONDITION：节点在条件队列中，基于Condition对象发生了等待，需要Condition对象来激活
  - PROPAGATE：使用在共享模式头结点有可能处于这种状态，表示锁的下一次acquireShared可以无条件传播
- setHeadAndPropagate：将当前节点设置为头结点，并无条件传播，以确保后续节点能被正常唤醒
  
### 基本方法
#### 获取状态
- acquire
```
if (!tryAcquire(arg) &&
  acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
  selfInterrupt();
```
- addWaiter(Node)在tryAcquire失败后调用，返回了最终要写入的node节点，作为acquireQueued()方法的入参
```
// 创建新节点：包含当前线程和同步为共享同步还是排他同步这两个信息
Node node = new Node(Thread.currentThread(), mode);   
Node pred = tail;
if (pred != null) {
    node.prev = pred;
    if (compareAndSetTail(pred, node)) { //在链表尾部加入，同enq，但只尝试一次入队列
        pred.next = node;
        return node;
    }
}
enq(node);
return node;
```
- enq(Node) //初始化CLH队列，确保node进入CLH队列
```
for (;;) { // 死循环保证成功，CAS保证多线程竞争时一次只有一个成功
            Node t = tail;
            if (t == null) {                        // 初始化CLH队列，head结点使用的是傀儡结点
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) { //在链表尾部加入
                    t.next = node;
                    return t;
                }
            }
        }
```
- acquireQueued(Node) 排他非响应中断模式下获取同步状态，也用于等待condition情况
```
boolean failed = true;
try {
    boolean interrupted = false;
    for (;;) {
        final Node p = node.predecessor();
        // 死循环只有在这个条件下推出：前前驱节点是头结点head，而且获取同步状态tryAcquire(arg)成功
        if (p == head && tryAcquire(arg)) { 
            // 设置当前节点(addWaiter加入CLH的结点)为头结点
            setHead(node);
            p.next = null; // help GC       // 执行完的结点
            failed = false;
            return interrupted;
        }
        // 不满足条件的线程在循环中不断进入等待状态:
        // 1.前驱节点状态设置为SIGNAL
        // 2.park()方法，将当前线程挂起为WAITING状态，需要中断或者unpark()来唤醒，至此线程彻底阻塞
        if (shouldParkAfterFailedAcquire(p, node) &&
            parkAndCheckInterrupt())       
            interrupted = true;
    }
} finally {
    if (failed)
        cancelAcquire(node);
}
```
- shouldParkAfterFailedAcquire(pred, node) // 设置node的前驱节点为SIGNAL
```
int ws = pred.waitStatus;
if (ws == Node.SIGNAL) // 前驱节点为SIGNAL，表示本节点可以被唤醒，可以放心地阻塞当前线程
    return true;
if (ws > 0) {         // 前驱节点为CANCELLED
    do {
        node.prev = pred = pred.prev;
    } while (pred.waitStatus > 0); // 从前驱节点开始找到第一个没有被CANCELLED的结点
    pred.next = node;              // 保证acquireQueued下一轮循环时node的前驱节点为SIGNAL
} else {
    compareAndSetWaitStatus(pred, ws, Node.SIGNAL); // 设置前驱节点为SIGNAL
}
return false;
```
- acquireShared
```
// 共享模式，所以判断获取状态的条件不同
if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
```
- doAcquireShared(arg) 共享非中断获模式下取状态
```
//初始化队列，增加当前线程代表的节点
final Node node = addWaiter(Node.SHARED); 
boolean failed = true;
try {
    boolean interrupted = false;
    for (;;) {
        final Node p = node.predecessor();
        if (p == head) {
            int r = tryAcquireShared(arg);
            if (r >= 0) {
                setHeadAndPropagate(node, r);
                p.next = null; // help GC
                if (interrupted)
                    selfInterrupt();
                failed = false;
                return;
            }
        }
        if (shouldParkAfterFailedAcquire(p, node) &&
            parkAndCheckInterrupt())
            interrupted = true;
    }
} finally {
    if (failed)
        cancelAcquire(node);
}
```
- setHeadAndPropagate(node, propagate)将当前节点设置为头节点，并无条件传播，确保后续节点能被正常唤醒
```
Node h = head; // Record old head for check below
setHead(node);
if (propagate > 0 || h == null || h.waitStatus < 0 ||
    (h = head) == null || h.waitStatus < 0) {
    Node s = node.next;
    if (s == null || s.isShared())
        doReleaseShared();
}
```
#### 释放状态
- release(int arg)
```
if (tryRelease(arg)) { // 状态彻底释放
    Node h = head;
    if (h != null && h.waitStatus != 0) // 头结点的后继节点需要唤醒
        unparkSuccessor(h);
    return true;
}
return false;
```
- 唤醒线程unparkSucessor(Node) unpark()唤醒最前面等待的节点(不一定是头结点后面的节点)
```
private void unparkSuccessor(Node node) { //传入的是head节点，head节点表示已执行完的节点
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0); // 处理头结点，非CANCELLED状态重置为0

    Node s = node.next;                       // 寻找下个节点
    // 如果是CANCELLED状态(>0)，说明节点中断了，从队尾开始寻找最前面等待的节点
    if (s == null || s.waitStatus > 0) {      
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
      // 唤醒下个节点里的线程，与release()中的判断条件对应
        LockSupport.unpark(s.thread);       
}
```


### 队列
#### CLH队列锁
- 仅适用自旋锁，但更容易处理cancel和timeout，所以AQS采用CLH Lock而不是MSC Lock
- 进出队快，无锁，有竞争也能很快插入，检查是否有线程在等待也容易(检查头尾指针是否相同)
- AQS中的CLH的改进：
  - 为了处理timeout和cancel，每个node维护一个前驱指针。如果一个node的前驱被cancel，这个node可以前向移动使用前驱的状态字段。
  - 每个node里使用一个状态字段去控制阻塞，而不是自旋。一个排队的线程调用acquire，只有在通过了子类实现的tryAcquire才能返回，确保只有队头线程才允许调用tryAcquire
  - head结点使用的是傀儡结点
#### 条件队列
- 条件结点Condition内部的nextWaiter指针穿起来
- 条件结点内部也有状态字段，修复了JVM内置同步器的不足，一个锁可以有多个条件。
- 添加队列中的线程获取锁之前，必须先被transfer到同步队列中去

### 同步状态
- 该整数可以表现任何状态。Semaphore 用它来表现剩余的许可数，ReentrantLock 用它来表现拥有它的线程已经请求了多少次锁；FutureTask 用它来表现任务的状态(尚未开始、运行、完成和取消)，ReentrantReadWriteLock将32位的state分成高低位，16位写锁计数，16位读锁计数，CountDownLatch代表计数，所有线程都获得锁时，状态为0，开始唤醒
- int类型的成员变量**state**来表示同步状态，state>0表示已获取锁，state=0表示已释放锁
- getState()、setState(int newState)、compareAndSetState(int expect,int update)
- 自定义子类使用AQS提供的模板方法就可以实现自己的同步语义
#### 独占式
- 同一时刻仅有一个线程持有同步状态
- 同步状态获取(acquire)
  - **AQS的模板方法acquire通过调用子类自定义实现的tryAcquire获取同步状态失败后->将线程构造成Node节点(addWaiter)->将Node节点添加到同步队列对尾(addWaiter)->每个节点以自旋的方法获取同步状态(acquirQueued)。在节点自旋获取同步状态时，只有其前驱节点是头节点的时候才会尝试获取同步状态，如果该节点的前驱不是头节点或者该节点的前驱节点是头节点单获取同步状态失败，则判断当前线程需要阻塞，如果需要阻塞则需要被唤醒过后才返回。**
  - 方法定义
  ```
   public final void acquire(int arg) {
          if (!tryAcquire(arg) &&
              acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
              selfInterrupt();
      }
  ```
  - acquireQueued：当前线程会根据公平性原则来进行阻塞等待（自旋）,直到获取锁为止；并且返回当前线程在等待过程中有没有中断过。当前线程会一直尝试获取同步状态，当然前提是**只有其前驱节点为头结点才能够尝试获取同步状态**，理由：
    - 保持FIFO同步队列原则。
    - 头节点释放同步状态后，将会唤醒其后继节点，后继节点被唤醒后需要检查自己是否为头节点。
  - selfInterrupt：产生一个中断
- 获取相应中断(acqurieInterruptibly)
  - 方法定义
  ```
  public final void acquireInterruptibly(int arg)
          throws InterruptedException {
      if (Thread.interrupted())
          throw new InterruptedException();
      if (!tryAcquire(arg))
          doAcquireInterruptibly(arg);
  }
  ```
  - doAcquireInterruptibly与acquire方法的差别在于在中断方法处不再是使用interrupted标志，而是直接抛出InterruptedException异
- 独占式超时获取(tryAcquireNanos)为acquireInterruptibly的进一步增强，它除了响应中断外，还有超时控制
- 同步状态释放(release)
  - 定义
  ```
  public final boolean release(int arg) {
      if (tryRelease(arg)) { // 释放同步状态
          Node h = head;
          if (h != null && h.waitStatus != 0)
              unparkSuccessor(h); //唤醒后继节点
          return true;
      }
      return false;
  }
  ```
#### 共享式
- **在同一时刻可以有多个线程获取同步状态**
- 同步状态获取(acquireShared)：`if (tryAcquireShared(arg) < 0) doAcquireShared(arg)`
  - 创建新节点(共享模式)，加入到队尾
  - 判断新节点的前驱节点是否为头结点，如果不是，则设置前驱节点的waitStatus为SIGNAL，此时当前线程可安全地挂起
  - 如果是头结点，前驱节点在共享模式下获取锁(子类实现tryAcqurieShared)，如果获取成功，就把当前节点设置为头结点。
  - 设置为头结点后，满足释放锁条件(允许传播或者需要通知继任节点或者继任节点为共享模式的节点)就阻塞等待释放锁
- 同步状态释放：因为可能会存在多个线程同时进行释放同步状态资源，所以需要确保同步状态安全地成功释放，一般都是通过CAS和循环来完成的
  - `if(tryReleaseShared) {doReleaseShared}`
  - 把当前节点设置为SIGNAL或者PROPAGATE
  - 如果当前节点不是头结点、尾节点，先判断当前节点状态是否为SIGNAL，如果是就设置为0，因为共享模式下更多使用PROPAGATE来传播。SIGNAL会被经过两步改为PROPAGATE：compareAndSetWaitStatus(h, Node.SIGNAL, 0) compareAndSetWaitStatus(h, 0, Node.PROPAGATE)
    - 为什么是2步：如果直接从SIGNAL到PROPAGATE,那么到unparkSuccessor方法里面又被设置为0：SIGNAL--PROPAGATE---0----PROPAGATE 对头结点相当于多做了一次compareAndSet操作
- CountDownLatch
  - state的初始值为count值
  - 调用await时tryAcqurieShared返回-1(state=count)，即这些线程都阻塞在doAcqurieShared的for循环中
  - 调用countDown时tryReleaseShared递减state的值，直到0返回false，此时线程退出for循环
  - 退出for循环后doReleaseShared，头结点为PROPAGATE，而且节点都是共享模式，头结点传播后结点都获取锁


### 同步队列
- 内置了FIFO同步队列**CLH**来完成资源获取线程的排队工作
- CLH为一个FIFO双向队列，其中一个节点表示一个线程
- 节点保存着线程的引用（thread）、状态（waitStatus）、前驱节点（prev）、后继节点（next）
- **入列**：addWaiter(Node)先尝试快速设置尾节点，如果失败，则用enq(Node)死循环保证设置尾节点（只有设置成功当前线程才会返回）。设置的时候都是通过compareAndSetTail来保证安全添加线程
- **出列**：首节点(FIFO)释放同步状态，唤醒它的后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点。只有一个线程能获取到同步状态，所以不需要CAS来保证。

### 阻塞和唤醒线程
- 获取同步状态失败后加入CLH同步队列，通过自旋不断获取同步状态，在自旋的过程中则需要判断当前线程是否需要阻塞，其主要方法在acquireQueued()

### 博客
- [博客1](https://blog.csdn.net/aesop_wubo/article/category/1147178)
- [博客2](https://www.jianshu.com/u/42116042245c)
### 公平锁与非公平锁
- 非公平锁只要CAS设置同步状态成功，当前线程也就获取锁成功
- 公平锁多了一个hasQueuedPredecessors()
- 在公平锁的获取之前加入该判断是为了确定当前线程是否有前驱节点，如果有，表示有线程比当前线程更早的请求获取锁，需要等待前驱线程获取并释放时候才能获取锁
- 公平锁的总耗时和总切换次数远远超过非公平锁，公平锁以大量的线程切换来换取了锁获取的FIFO原则，而非公平锁虽然可能会导致线程饥饿，但是它的线程切换很少，在一定程度上保证了更大的吞吐量
