### 特点
- 开启 `-XX:+UseConcMarkSweepGC`
  - 开启后年轻代默认为ParNew GC
  - 年轻代的并行收集线程数默认是(cpu <= 8) ? cpu : 3 + ((cpu * 5) / 8)，可用`-XX:ParallelGCThreads= N` 来调整
- CMS不是Full GC，CMS只会收集老年代和永久代(metaspace)
- 触发：
  - System.gc() 前提是_full_gc_requested=true
  - 默认老年代92%就开始收集
    - `-XX:CMSInitiatingOccupancyFraction=80`可修改
  - 如果之前的 ygc 失败过，或则下次新生代执行 ygc 可能失败，这两种情况下都需要触发 cms gc
  - `-XX:+CMSClassUnloadingEnabled`开启永久代收集且永久代使用了68%
- 触发Full GC
  - 当发生ygc之后，还是没有足够的内存进行分配
  - 上一次 ygc 过程中发生 promotion failure
- compact
  - 用户强制执行了gc，如System.gc()
  - 上一次 ygc 已经失败（发生了promotion failure），或预测下一次 ygc 不会成功
  - 每次触发 full gc，由于`UseCMSCompactAtFullCollection`默认开启

### 流程
- 初始标记(CMS-initial-mark) 
  - 标记GC Roots
  - `-XX:+CMSParallelInitalMarkEnabled`开启初始标记并行化
- 并发标记(CMS-concurrent-mark)
- 并发预清理
- 可终止的预清理
- 重新标记(CMS-remark)
  - 一般占CMS 80%的时间
- 并发清除(CMS-concurrent-sweep) 
- 并发重置(CMS-concurrent-reset)
