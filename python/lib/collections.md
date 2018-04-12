#### collections.namedtuple
- `Point = namedtuple('Point', ['x', 'y'])`
- 生成tuple的子类，不可变

#### collections.deque
- 链表结构
- appendleft popleft
- deque(maxlen=N) 保留最后N个元素

#### collections.defaultdict
- dict的子类`dd = defaultdict(lambda: 'N/A')`
- dict.get(key\[, defaultvalue])在每次调用的时候指定默认值

#### collections.OrderedDict

#### collections.Counter

#### heapq
- heapq.nsmallest(N, iterable, key=None) key可以为函数，指定用iterable中每个item的哪个字段比较，比如 lambda s: s\['price']
- heapq.nlargest 相当于 sorted(iterable, key=key)\[:N]
- heapq.heapify(list) 底层将list以堆排序后存储(修改list)，从小到大排序
- heappop() heappush() 可用于实现优先级队列
```
```
