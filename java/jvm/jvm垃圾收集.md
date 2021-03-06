## 并行、并发
- 并行 Parallel，多线程
- 并发 Concurrent，垃圾收集线程与用户线程同一时间段内交替执行

## GC Roots
- 虚拟机栈中引用的对象
- 方法区中静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中JNI引用的对象

## 收集算法
- 复制算法
  - 新生代垃圾收集器全部使用复制算法，包括G1中的年轻代region
- 标记整理
  - Serial Old、Parallel Old
  - G1用到了标记整理算法
- 标记清除
  - CMS使用标记清除算法

## 垃圾收集器
- Serial
  - 单线程、复制算法
  - \-XX：+UseSerialGC
- ParNew
  - Serial的多线程版本
  - \-XX:+UseParNewGC
  - \-XX:ParallelGCThreads控制线程数量
- Parallel Scavenge
  - 复制算法，关注吞吐量
  - \-XX:MaxGCPauseMillis 收集器将尽力保证内存回收花费的时间不超过设定值。
  - \-XX:GCTimeRatio 默认值为99，就是允许最大1%（即1 /（1+99））的垃圾收集时间
- Serial Old
  - 标记整理算法
  - 可与所有新生代收集器配合使用
  - 如果CMS收集器出现Concurrent Mode Failure，则Serial Old收集器将作为后备收集器。
- Parallel Old
  - 标记整理算法
  - \-XX:+UseParallelOldGC
- CMS
  - 标记清除算法，关注停顿时间，建议内存3G以上
  - \-XX:+UseConcMarkSweepGC：启用CMS \+ ParNew Serial \+ Old
  - \-XX:CMSInitiatingOccupancyFraction决定老年代内存占用多少开始垃圾收集，否则根据统计数据决定
  - \-XX:+ UseCMSCompactAtFullCollection 在Full GC后，进行一次碎片整理
  - \-XX:CMSFullGCBeforeCompaction设置多少次Full GC之后整理碎片
  - `初始标记(STW)->并发标记->重新标记(STW)->并发清除`
  - 触发Full GC
    - concurrent-mode-failure：并发清除时对象进入老年代空间不够(没有大空间或者老年代已用完)
    - promotion-failed：minor gc时，survivor放不下的对象需要进入老年代，此时老年代空间不足
  - 三大缺点
    - 对CPU资源非常敏感，并发阶段会降低吞吐量，默认启动的回收线程数是：(CPU数量+3)/4。（建议CPU个数最少4个）
    - 无法处理浮动垃圾，启动Serial Old来收集
    - 空间碎片化，可能提前触发Full GC
- G1
  - 分region(一个region有可能属于Eden，Survivor或者Tenured内存区域)，还有Humongous内存区块存储超过region 50%的大对象
  - 并行收集。年轻代使用复制算法。建议内存6G以上。
  - `Initial Mark(触发minor gc时顺便进行) -> Concurrent Mark(同时清除对象存活率小或者没有的Tenured region) -> Remark(算法为SATB) -> Clean up/Copy(挑选对象存活率低的region回收，也是和minor gc一起进行)`
