### JDK1.7
#### 数据结构
- Segment数组，每个元素都是一个HashEntry数组
- Segment继承了ReentrantLock，自带锁功能
- 初始化时，计算Segment数组大小ssize(2的幂次方，默认16)和每个Segment中HashEntry数组的大小cap(2的幂次方，最小2)，初始化第一个Segment
#### put
- 根据hash找到Segment，如果未初始化则CAS赋值
- 加锁插入数据
  - tryLock()成功直接插入HashEntry，插入后unlock()释放锁
  - 失败则scanAndLockForPut()重复64次tryLock()，继续失败则lock()挂起
#### size
- 统计每个Segment对象中的元素个数，然后进行累加
- 为了提高准确性
  - 先采用不加锁的方式，连续计算元素的个数，最多计算3次
  - 如果前后两次计算结果都不同，则给每个Segment进行加锁，再计算一次元素的个数；
  
### JDK8
#### 数据结构
- Node数组(table，大小为2的幂次方，默认为16)