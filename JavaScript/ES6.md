## let const
- 块级作用域内有效
- 不存在变量提升
- 不允许重复定义
- 允许在块级作用域内声明函数，函数声明语句的行为类似于let，在块级作用域之外不可引用
- 暂时性死区

## Iterator
- 部署了Iterator接口的对象可以进行`for ... of`迭代
- Iterator的next()方法，返回值: {value: 当前成员值, done: 遍历是否结束}
- 默认Iterator接口
	- 默认Iterator接口部署在`[Symbol.iterator]`属性
	- `[Symbol.iterator]`属性对象有next()方法
	- `for ... of`就是调用`[Symbol.iterator]`属性对象的next()方法
- Iterator对象的return() throw()方法
- return必须返回一个对象，主要用于`for ... of`提前退出的情况
- throw主要配合Generator函数使用
- 调用Iterator接口的场合
	- 解构赋值
	- 扩展运算符`...`
	- yield*
	- Array.from()
	- Map() Set() WeakMap() WeakSet()
	- Promise.all()
	- Promise.race()
- 部署了Iterator接口的数据结构
	- String
	- Array
	- Map
	- Set
	- TypedArra
	- 函数的arguments
	- NodeList对象

## 解构Destructuring
- let [var list
