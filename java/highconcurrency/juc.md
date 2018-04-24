## ReentrantLock比对象锁多了些高级功能
- 中断响应 lockInterruptibly()
- 锁申请等待限时
  - tryLock(long, TimeUnit) 超时返回false，成功返回true
  - tryLock()不等待，不管是否成功立即返回，成功返回true，否则返回false
- 公平锁 需要维护一个有序队列，性能相对低下

## Condition
- await() signal()与ReentrantLock锁的获取和释放配套使用
