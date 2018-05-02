## 扩展
### string
- 模板字符串
	- 对象默认调用toString()方法
	- 模板可嵌套
	- `\ $ {`必须用反斜杠转义
- codePointAt() fromCodePoint()弥补了charCodeAt() fromCharCode()的不足
- 部署了Iterator接口，可以用`from...of`循环遍历
- includes()、startsWith()、endsWith()弥补了indexOf()的不足
- repeat()
- padStart()、padEnd()，默认用空格补全
### 数组
- forEach()不能使用break和return，可用`for...of`遍历
### 函数
- 不定参数`(...arg)s`
	- 在所有函数参数中，只有最后一个才可以被标记为不定参数。
	- 如果没有额外的参数，不定参数就是一个空数组，不定参数永远不会是undefined
- 默认参数`(arg=xx)`
	- 默认值表达式在函数调用时从左到右求值，因此默认值表达式可使用该参数之前已填充好的其他参数值，比如`func(arg1=1, arg2=(arg1===1) ?  2 : 3)`
	- 传递undefined值等于不传值，没有默认值的参数隐式默认为undefined
- 箭头函数
	- 箭头函数没有自己的this值，this值继承自外围作用域
	
## Set和Map
- Set中的比较类似于`===`，但是NaN只能加入1个
	
## Symbols
- `let s = Symbol();`，新的symbol值与任何职都不相等
- Symbol.for(string)，访问symbol注册表，相同string返回相同的symbol
- 一些标准也定义了symbol，比如Symbol.iterator

## Promise
- 一个对象，保存着某个未来才会结束的时间的结果
- Promise对象的状态不受外界的影响，只有异步操作的结果能决定当前的状态
	- pending(进行中)
	- fulfilled(已成功)
	- rejected(已失败)
- Promise对象的状态只能从pending变为fulfilled或者rejected，一旦改变，就不会再变，任何时候都能得到这个结果
- 一旦创建就立即执行，无法中途取消
- pending时，无法得知是刚刚开始还是即将完成

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

## 生成器
- 定义: `function*` `yield`
- 生成器是迭代器，所有生成器都有内建的`next()`和`[Symbol.iterator]()`方法的实现

## 解构Destructuring
- 可结构数组、对象或迭代器
- 可迭代嵌套结构
- 如果解构的对象是null或undefined，得到TypeError
- 如果解构其他原始类型，得到undefined
- 可与迭代器协同使用，比如 `for(var [key] of map)` `for(var[,value] of map)`
- 可实现多重返回值
