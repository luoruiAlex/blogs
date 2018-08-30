### 定义
- 在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系
- 如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前
- 两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行。如果重排序之后的执行结果与按照happens-before关系来执行的结果一致，那么这种重排序并不非法

### 原则
- 程序次序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作
  - 一段代码在**单线程**中执行的**结果**是有序的
- 监视器锁规则：对一个锁的**解锁**，happens-before于随后对这个锁的**加锁**
  - 一个锁处于被锁定状态，那么必须先执行unlock操作后面才能进行lock操作
- volatile变量规则：对一个volatile域的**写**，happens-before于任意后续对这个volatile域的**读**
  - 标志着volatile保证了线程可见性
- 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C
- 线程启动规则：Thread对象的start()方法happens-before于此线程的每个一个动作
  - 假定线程A在执行过程中，通过执行ThreadB.start()来启动线程B，那么线程A对共享变量的修改在接下来线程B开始执行后确保对线程B可见
- 线程中断规则：对线程interrupt()方法的调用happens-before于被中断线程的代码检测到中断事件的发生
- 线程终结规则：线程中所有的操作都happens-before于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
  - 假定线程A在执行的过程中，通过制定ThreadB.join()等待线程B终止，那么线程B在终止之前对共享变量的修改在线程A等待返回后可见
- 对象终结规则：一个对象的初始化完成**happens-before**于他的**finalize()**方法的开始
