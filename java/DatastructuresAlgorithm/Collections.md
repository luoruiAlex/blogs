## List
###　ArrayList
- 底层数组，默认size为10
- 每次增加都判断是否需要扩容，删除时某个索引元素时，需要整体移动数组，所以效率低
- 超出限制后扩容50%，使用Arrays.copyOf，而Arrays.copyOf是调用native System.arrayscopy
### LinkedList
- 底层双向链表

## Map
### HashMap
- 底层为数组，每个元素为一个单向链表，默认size为16
- capacity：当前数组容量，始终保持2^n，每次扩容后数组大小为当前2倍
- loadFactor：负载因子，默认为0.75
- threshold：扩容的阈值，capacity \* loadFactor
- indexFor：key的hash值与(数组长度 \- 1)做与运算
- 第一次put时，初始化数组size，使其为2^n，扩容时，也要使其为2^n
- key的hashcode和equals方法必须同时改写
- jdk8：利用**红黑树**优化链表，链表查找复杂度为O(n，)为红黑树的查找复杂度为O(log n)
  - TREEIFY_THRESHOLD：当bucket大于8的时候就转化为红黑树
  - UNTREEIFY_THRESHOLD：小于6时还原为链表
  - MIN_TREEIFY_CAPACITY：当哈希表中的容量大于这个值时，表中的桶才能进行树形化
  - resize的时候**不需要重新计算hash**
### ConcurrentHashMap
- 分段锁：底层为Segment数组，Segment集成ReentrantLock来加锁，默认有16个Segment
- Segment内部还是数组加链表
- 不允许存key、value为null：支持并发的HashMap导致两次调用间的改变使我们无法知道是不存在还是为值为空
- Segment数组不能扩容
- loadFactor是给每个Segment内部使用的
