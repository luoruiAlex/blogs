- 默认使用字典数据结构
- globals() locals()各返回一个字典，locals()在模块级别执行时与globals()一样
- globals()[key] = value locals[key] = value 可间接定义变量，但应避免使用这种方式，函数执行使用缓存机制，直接修改未必有效。
- 使用 is 来判断两个对象是否为同一个，使用 id() 也可以判断
- LEGB:
  - L-Local(function):函数内的名字空间
  - E-Enclosing function locals:外部嵌套函数的名字空间(例如closure)
  - G-Global(module):函数定义所在模块（文件）的名字空间
  - B-Builtin(Python):Python内置模块的名字空间
