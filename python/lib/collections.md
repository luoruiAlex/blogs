#### collections.namedtuple
- `Point = namedtuple('Point', ['x', 'y'])`
- 生成tuple的子类，不可变

#### collections.deque
- 链表结构
- deque(list)
- appendleft popleft
- deque(maxlen=N) 保留最后N个元素

#### collections.defaultdict
- dict的子类`dd = defaultdict(lambda: 'N/A')`
- dict.get(key\[, defaultvalue])在每次调用的时候指定默认值

#### collections.OrderedDict
- 按照插入的顺序排序
- FIFO

#### collections.Counter
- `counter[key] = counter[key] + 1`

#### heapq
- heapq.nsmallest(N, iterable, key=None) key可以为函数，指定用iterable中每个item的哪个字段比较，比如 lambda s: s\['price']
- heapq.nlargest 相当于 sorted(iterable, key=key)\[:N]
- heapq.heapify(list) 底层将list以堆排序后存储(修改list)，从小到大排序
- heappop() heappush() 可用于实现优先级队列
```
```

#### collections.ChainMap
- 本质上还是一个dict，由一系列dict组成，按顺序在内部的dict依次查找
- 示例：应用程序传参，可按照命令行、环境变量、默认参数的顺序查找值
- ```
  from collections import ChainMap
  import os, argparse

  defaults = {
      'color': 'red',
      'user': 'guest'
  }

  parser = argparse.ArgumentParser()
  parser.add_argument('-u', '--user')
  parser.add_argument('-c', '--color')
  namespace = parser.parse_args()
  command_line_args = {k: v for k, v in vars(namespace).items() if v}
  combined = ChainMap(command_line_args, os.environ, defaults)
  print(combined['color'])
  print(combined['user'])
  ```
