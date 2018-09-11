## 使用
- 如果get()返回null，则使用set()来加进去

## 原理
- 1.每个线程都有2个ThreadLocal.ThreadLocalMap类型的对象(threadLocals和inheritableThreadLocals)
- 2.get()时先获取当前线程(Thread.currentThread)，然后获取currentThread的ThreadLocalMap
- 3.ThreadLocalMap的Entry的key为当前ThreadLocal，value为ThreadLocal存储的对象。
### ThreadLocalMap
- 没有实现Map接口
- 初始化为一个size=16的Entry数组
- Entry extends WeakReference，key永远都是ThreadLocal对象
  - 没有next字段，即不存在链表
  - hash冲突的解决：每个ThreadLocal都有一个threadLocalHashCode，每初始化一个ThreadLocal对象，hash值就增加一个固定的大小0x61c88647
  - 插入时定位：
    - hash值定位，为空则直接插入Entry
    - 已有Entry而且key为即将插入的key，更新value
    - 已有Entry而且key不为即将插入的key，找下一个空位
    

## 注意事项
- 线程退出时，Thread类会进行清理，包括ThreadLocalMap。
- 线程一直运行或者使用线程池时线程不退出，有可能造成内存泄露。ThreadLocal设置为null只能回收ThreadLocal
  - key==null时，GC会回收threadlocal实例，但是value与CurrentThread存在一个强引用关系
  - 解决办法为显式调用ThreadLocal.remove()
- threadLocal = null可以加速Thread对应所有线程局部变量的回收。Entry extends WeakReference，设置为null后就没有强引用了
