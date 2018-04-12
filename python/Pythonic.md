##### 解包
- 接收多余部分 `a, *b, c = sequence/Iterable`
- 丢弃多余部分 `_, a, b, _ = xx` `_*, a = xx`
- 可扩展到任意需要iterable的地方
  - 迭代中使用 `for x, *y in iterable`
  - 字符串操作中使用 `a, *b = str.split('x')` 
