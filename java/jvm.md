## 并行、并发
- 并行 Parallel，多线程
- 并发 Concurrent，垃圾收集线程与用户线程同一时间段内交替执行

## 收集算法
- 复制算法
  - 新生代垃圾收集器全部使用复制算法
  - G1用到了复制算法
- 标记整理
  - Serial Old、Parallel Old
  - G1用到了标记整理算法
- 标记清除
  - CMS使用标记清除算法

## 垃圾收集器
- Serial
  - 单线程、复制算法
- ParNew
  - Serial的多线程版本
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
- CMS
  - 标记清除算法，关注停顿时间
  - `初始标记(STW)->并发标记->重新标记(STW)->并发清除`
  - 三大缺点
    - 对CPU资源非常敏感，并发阶段会降低吞吐量，默认启动的回收线程数是：(CPU数量+3)/4。（建议CPU个数最少4个）
    - 无法处理浮动垃圾
    - 空间碎片化，可能提前触发Full GC \-XX:UseCMSCompactAtFullCollection开启Full GC后整理碎片，\-XX:CMSFullGCBeforeCompaction设置多少次Full GC之后整理碎片
