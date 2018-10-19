## List
###　ArrayList
- 底层数组，默认size为10
- 每次增加都判断是否需要扩容，删除时某个索引元素时，需要整体移动数组，所以效率低
- 超出限制后扩容50%，使用Arrays.copyOf，而Arrays.copyOf是调用native System.arrayscopy
### LinkedList
- 底层双向链表

## Map
- 数组都是第一次插入元素的时候创建
### LinkedHashMap
- 继承了HashMap，内部形成双向链表
- 两种排序方式
  - 默认按照插入顺序排序
  - LinkedHashMap(int initialCapacity,float loadFactor,boolean accessOrder) accessOrder设置为true则为按照访问顺序排序
- 按照访问顺序排序则将当前的Entry移动到链表头部
### HashMap
- 底层为数组，每个元素为一个单向链表，默认size为16
- capacity：当前数组容量，始终保持2^n，每次扩容后数组大小为当前2倍
- loadFactor：负载因子，默认为0.75
- threshold：扩容的阈值，capacity \* loadFactor
- indexFor：key的hash值与(数组长度 \- 1)做与运算
- **第一次put时，初始化数组size**，使其为2^n，扩容时，也要使其为2^n
- 为什么长度是2^n，低位全为1，而扩容后只有一位差异，也就是多出了最左位的1，这样在通过 h&(length-1)的时候，只要h对应的最左边的那一个差异位为0，就能保证得到的新的数组索引和老数组索引一致(大大减少了之前已经散列良好的老数组的数据位置重新调换(resize)。简单来说就是**便于通过按位与的散列算法来定位Segment的index**
- 扩容后，长度为之前的2倍
- key的hashcode和equals方法必须同时改写
- jdk8：利用**红黑树**优化链表，链表查找复杂度为O(n)，为红黑树的查找复杂度为O(log n)
  - TREEIFY_THRESHOLD：当bucket大于8的时候就转化为红黑树
  - UNTREEIFY_THRESHOLD：小于6时还原为链表
  - MIN_TREEIFY_CAPACITY：当哈希表中的容量大于这个值时，表中的桶才能进行树形化
  - resize的时候**不需要重新计算hash**
### ConcurrentHashMap
- 分段锁：底层为Segment数组，Segment继承ReentrantLock来加锁，默认有16个Segment
- Segment内部还是数组加链表，HashEntry为元素
- 不允许存key、value为null：支持并发的HashMap导致两次调用间的改变使我们无法知道是不存在还是为值为空
- Segment数组不能扩容
- loadFactor是给每个Segment内部使用的
- get()整个过程都不需要加锁，所以效率非常高
- 问题：查询遍历表效率太低
- jdk8：摒弃了分段锁的方案，直接使用一个大数组
  - put
    - 定位出位置
    - 如果需要扩容则复制到新的数组
    - 为空直接CAS设置其值，失败则自旋保证成
    - 非null的数组元素，synchronized加锁该元素然后操作
    - 如果数量大于 TREEIFY_THRESHOLD 则要转换为红黑树
  - get：value和next都是volatile，具有可见性，无需加锁就能保证可见性
  
### CopyOnWriteArrayList
- 读不加锁
- 写的时候加ReentrantLock，然后拷贝数组，Arrays.copyOf()，本质上调用System.arraycopy()
