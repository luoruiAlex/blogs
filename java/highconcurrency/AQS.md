### 定义
- **多线程并发的执行，之间通过某种 共享 状态来同步，只有当状态满足 xxxx 条件，才能触发线程执行 xxxx**，这个共同的语义可以称之为同步器
- AQS(AbstractQueuedSynchronizer)是JUC并发包中的核心基础组件
- 解决了子类实现同步时涉及的大量细节问题，比如获取同步状态、FIFO同步队列

### 节点
- 新加入的节点的前置节点必须状态为SIGNAL，因为只有前置及诶点为SIGNAL时当前节点才可能被唤醒
- CANCELLED表示节点被取消，该节点不再被唤醒
- setHeadAndPropagate：将当前节点设置为头结点，并无条件传播，以确保后续节点能被正常唤醒
- shouldParkAfterFailedAcquire：在获取资源失败后将节点所持有的线程阻塞，阻塞过程中需要确保节点的前置节点为SIGNAL，以确保节点能被正常唤醒
- unparkSuccessor(h)：唤醒该节点的后继线程，有可能该线程的next指向的节点被中断，此时会从尾部开始往前查找，找到一个可以唤醒的节点

### 同步状态
- 该整数可以表现任何状态。比如， Semaphore 用它来表现剩余的许可数，ReentrantLock 用它来表现拥有它的线程已经请求了多少次锁；FutureTask 用它来表现任务的状态(尚未开始、运行、完成和取消)
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
- 同步状态获取(acquireShared)：tryAcquireShared(int arg)方法尝试获取同步状态，返回值为int，当其 >= 0 时，表示能够获取到同步状态，这个时候就可以从自旋过程中退出
- 同步状态释放：因为可能会存在多个线程同时进行释放同步状态资源，所以需要确保同步状态安全地成功释放，一般都是通过CAS和循环来完成的


### 同步队列
- 内置了FIFO同步队列**CLH**来完成资源获取线程的排队工作
- CLH为一个FIFO双向队列，其中一个节点表示一个线程
- 节点保存着线程的引用（thread）、状态（waitStatus）、前驱节点（prev）、后继节点（next）
- **入列**：addWaiter(Node)先尝试快速设置尾节点，如果失败，则用enq(Node)死循环保证设置尾节点（只有设置成功当前线程才会返回）。设置的时候都是通过compareAndSetTail来保证安全添加线程
- **出列**：首节点(FIFO)释放同步状态，唤醒它的后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点。只有一个线程能获取到同步状态，所以不需要CAS来保证。

### 阻塞和唤醒线程
- 获取同步状态失败后加入CLH同步队列，通过自旋不断获取同步状态，在自旋的过程中则需要判断当前线程是否需要阻塞，其主要方法在acquireQueued()

### [好博客](https://www.jianshu.com/u/42116042245c)
### 公平锁与非公平锁
- 非公平锁只要CAS设置同步状态成功，当前线程也就获取锁成功
- 公平锁多了一个hasQueuedPredecessors()
- 在公平锁的获取之前加入该判断是为了确定当前线程是否有前驱节点，如果有，表示有线程比当前线程更早的请求获取锁，需要等待前驱线程获取并释放时候才能获取锁
- 公平锁的总耗时和总切换次数远远超过非公平锁，公平锁以大量的线程切换来换取了锁获取的FIFO原则，而非公平锁虽然可能会导致线程饥饿，但是它的线程切换很少，在一定程度上保证了更大的吞吐量
