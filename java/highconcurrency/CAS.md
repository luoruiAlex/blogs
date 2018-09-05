### 3个参数
- 内存值V
- 旧的预期值A
- 要更新的值B
- V等于A时才会将V改成B，返回true，否则什么都不干直接返回false

### Unsafe的方法
- 内存管理：包括内存分配、内存释放等
  - allocateMemory copyMemory freeMemory getAddress getInt等
- 非常规的对象实例化allocateInstance
  - 直接生成对象实例而无需调用构造方法和其它初始化方法
- 操作类、对象、变量：可获取对象的指针
  - 比如objectFieldOffset()用于在静态初始化块中获取对象的某个字段的offset
- 数组操作
  - arrayBaseOffset arrayIndexScale配合定位数组的每个元素，还可分配超大数组(Integer.MAX_VALUE)
- 多线程同步：锁、CAS操作
  - monitorEnter tryMonitorEnter monitorExit已deprecated
  - compareAndSwapInt()的4个参数：需要改变的对象、字段偏移量offset、预期值、修改值
- 挂起与恢复
  - park unpark
- 内存屏障：避免重排
  - loadFence storeFence fullFence

### 多处理器的原子操作
- 总线加锁：是使用处理器提供的一个LOCK#信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住,那么该处理器可以独占使用共享内存
- 缓存式加锁：缓存在内存区域的数据如果在加锁期间，当它执行锁操作写回内存时，处理器不在输出LOCK#信号，而是修改内部的内存地址，利用缓存一致性协议来保证原子性

### ABA问题
- 使用版本号解决
- Java提供了**AtomicStampedReference**来解决
- 4个参数：预期引用、更新后的引用、预期标志、更新后的标志
