## 使用
- 如果get()返回null，则使用set()来加进去

## 原理
- 1.每个线程都有2个ThreadLocal.ThreadLocalMap类型的对象(threadLocals和inheritableThreadLocals)
- 2.get()时先获取当前线程(Thread.currentThread)，然后获取currentThread的ThreadLocalMap
- 3.ThreadLocalMap的Entry的key为当前ThreadLocal，value为ThreadLocal存储的对象。()

## 注意事项
- 线程退出时，Thread类会进行清理，包括ThreadLocalMap。
- 使用线程池时线程不退出，有可能造成内存泄露。解决办法为ThreadLocal.remove()
- threadLocal = null可以加速Thread对应所有线程局部变量的回收。Entry extends WeakReference，设置为null后就没有强引用了
