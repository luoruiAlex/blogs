### 作用
- AQS(AbstractQueuedSynchronizer)是JUC并发包中的核心基础组件
- 解决了子类实现同步时涉及的大量细节问题，比如获取同步状态、FIFO同步队列

### 同步状态
- int类型的成员变量**state**来表示同步状态，state>0表示已获取锁，state=0表示已释放锁
- getState()、setState(int newState)、compareAndSetState(int expect,int update)
- 自定义子类使用AQS提供的模板方法就可以实现自己的同步语义
#### 独占式
- 同一时刻仅有一个线程持有同步状态
- 同步状态获取(acquire)
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
#### 

### 同步队列
- 内置了FIFO同步队列**CLH**来完成资源获取线程的排队工作
- CLH为一个FIFO双向队列，其中一个节点表示一个线程
- 节点保存着线程的引用（thread）、状态（waitStatus）、前驱节点（prev）、后继节点（next）
- **入列**：addWaiter(Node)先尝试快速设置尾节点，如果失败，则用enq(Node)死循环保证设置尾节点（只有设置成功当前线程才会返回）。设置的时候都是通过compareAndSetTail来保证安全添加线程
- **出列**：首节点(FIFO)释放同步状态，唤醒它的后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点。只有一个线程能获取到同步状态，所以不需要CAS来保证。
