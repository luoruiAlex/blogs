## ReentrantLock比对象锁多了些高级功能
- 中断响应 lockInterruptibly()
- 锁申请等待限时
  - tryLock(long, TimeUnit) 超时返回false，成功返回true
  - tryLock()不等待，不管是否成功立即返回，成功返回true，否则返回false
- 公平锁 需要维护一个有序队列，性能相对低下

- LockSupport
- park() unpark()可以简单地实现线程暂停和恢复
- 为每个线程准备一个许可，如果许可可用，park()立即消费许可并返回，unpark()使许可变为可用
- park()暂停后线程为WAITING状态
- park()能支持中断影响，但不会抛出InterruptedException，只会默默返回，可以在park()后加使用Thread.isInterrupted()获取中断标记并响应中断

## Condition
- lock.newCondition()
- await() signal()与ReentrantLock锁的获取和释放配套使用

## Semaphore
- new Semphore(num)
- acquire()  release()

## CountDownLatch
- 主线程在CountDownLatch上等待所有任务完成后才能继续
- new CountDownLatch(num)
- awati() countDown()
- 线程数一般与CountDownLatch一致，Executor.newFixedThreadPool(num)

## CyclicBarrier
- 可反复使用，await()调用一次就计数一次
- new CyclicBarrier（int parties, Runnable barrierAction)
- await()

## Phaser
- 可在运行时动态增减参与量
- arriveAndAwaitAdvance()
- arriveAndDeregister()：告知phaser对应的线程已经完成了当前阶段，并它不会参与到下一阶段的操作中。
- bulkRegister(int)：将指定数目的参与者注册到phaser中，所有这些新的参与者都将被当成没有执行完本阶段的线程。
- arrive()：这个方法通知phase对象一个参与者已经完成了当前阶段，但是它不应该等待其他参与者都完成当前阶段，必须小心使用这个方法，因为它不会与其他线程同步

## ReadWriteLock
- 读和读之间不阻塞，读写之间相互阻塞，写写之间相互阻塞
- new ReentrantReadWriteLock()
- Lock readLock = lock.readLock()
- Lock writeLock = lock.writeLock()

## 线程池
### Executors
- ExecutorService newSingleThreadExecutor()
- ExecutorService newFixedThreadPool(int nThreads)
  - corePoolSize=maximumPoolSize
  - LinkedBlockingQueue
  - 线程数量无变化
- ExecutorService newCachedThreadPool()
  - corePoolSize = 0 maximumPoolSize=无穷大
  - SynchronousQueue
  - 无限制增加线程
- ScheduledExecutorService newSingleThreadScheduledExecutor()
- ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
### ExecutorService
- submit(Callable | Runnable)
### ScheduledExecutorService
- ScheduledFuture schedule(Callable | Runnable, long, TimeUnit)
- ScheduledFuture scheduleAtFixedRate(Runnable, initialDelay, period, TimeUnit) 上一次开始时就开始记间隔时间
- ScheduledFuture scheduleWithFixedDelay(Runnable, initialDelay, delay, TimeUnit) 上一次完成才开始计间隔时间，如果遇到异常，后续子任务都会停止调度
### ThreadPoolExecutor
- corePoolSize 线程池最少的线程数
- maximumPoolSize 最大线程数
- keepAliveTime 线程池中线程超过corePoolSize，多余的空闲线程的存活时间
- unit keepAliveTime的单位
- workQueue: BlockingQueue类型
  - SynchronousQueue 无容量
  - ArrayBlockingQueue(int capacity) 固定容量
  - LinkedBlockingQueue 无界
  - PriorityBlockingQueue 无界
- threadFactory
  - 创建线程池中的线程
- handler RejectExecutionHandler
  - AbortPolicy 抛出异常 
  - CallerRunsPolicy 直接在调用者线程中运行被丢弃的任务，会导致性能急剧下降
  - DiscardOledestPolicy 丢弃最老的任务
  - DiscardPolicy 丢弃无法处理的任务
- 自定义ThreadPoolExecutor
  - beforeExecute()
  - afterExecute()
  - terminated() 线程池退出前做清理工作
- shutdown() 不会马上退出，仅仅是不再接受任务而已
- 合适线程池大小估算
  - `Nthreads = Ncpu * Ncpu * (1 + 等待时间/计算时间)`
  - Runtime.getRuntime().availableProcessors()可获取CPU数量
  
## Fork/Join
- 核心思想： 分而治之 + work-stealing
- ForkJoinTask submit(ForkJoinTask)
- ForkJoinTask子类
  - RecursiveAction 无返回值
  - RecursiveTask 有返回值
  
## 并发容器
- ConcurrentHashMap 线程安全的HashMap
- CopyOnWriteArrayList 适用于读多写少的场景
- ConcurrentLinkedQueue 线程安全的LinkedList
- BlockingQueue put() take()为阻塞操作
- ConcurrentSkipListMap 跳表，时间复杂度O(log n)

## Java8增强
- CompletableFuture implements Future, CompletionStage
- ForkJoinPool性能优化，新增getCommonPoolParallelism() 、commonPool()
- atomic：DoubleAccumulator、DoubleAdder、LongAccumulator、 LongAdder
  - 提升性能：将AtomicInteger的内部核心数据value分离成一个数组,每个线程访问时,通过哈希等算法映射到其中一个数字进行计数,而最终的计数结果,则为这个数组的求和累加,其中,热点数据value被分离成多个单元cell,每个cell独自维护内部的值,当前对象的实际值由所有的cell累计合成
- StampedLock：在大多数场景都可以替代ReentrantReadWriteLock。它为读写操作提供了三种模式：Writing、ReadingOptimistic、 Reading
