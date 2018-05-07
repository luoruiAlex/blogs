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
- `new Promise((resolve, reject) =>{}).then(resolve1).then(resolve2).catch(reject).finally()`
- new出来后，构造函数中传入的函数(函数中调用resovle和reject回调)立即执行，然后then指定回调，然后在所有同步任务执行完后执行回调。
- `then(resolved, rejection)`返回一个新的Promise实例，所以可以连续调用`then()`方法
- `catch()`相当于`then(null, rejection)`。Promise的错误有"冒泡"的性质，会一直传递，直到被捕获为止。如果未catch，错误也不会传递到外层代码。
- `finally()`，无法知道前面的Promise的状态是fulfilled还是rejected
- `all()`用于将多个Promise实例包装成一个Promise实例`Promise.all([p1, p2])`，所有成员Promise实例都变成fulfilled组合Promise才会变成fulfilled，否则只要有一个成员Promise变成rejected，组合Promise就会变成rejected
- `race()` 率先改变的成员Promise实例的返回值，就传递给组合Promise的回调函数
- `Promise.resolve()`，将现有对象转为Promise对象
	- 参数是一个Promise实例对象，直接返回该对象
	- 参数为有`then()`方法的对象，将该对象包装为Promise对象并立即执行该`then()`方法
	- 其他参数，返回一个状态为resolved的Promise对象，`Promise.resolve(arg)`相当于`new Promise(resolve => resolve(arg))`
	- 无参数，直接返回一个状态为resolved的Promise对象
- `Promise.reject()`

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
- 调用生成器返回一个内部指针对象，每次调用指针对象的`next()`方法返回{value, done}
- yield表达式
	- 必须用在`function*`里面
	- 如果嵌入在表达式中使用，比如加括号，比如`'hello' + (yield)`
	- 如果用作函数参数或放在赋值表达式的右边，可以不加括号，如`foo(yield 'a', yield 'b');`
- generator可用于给对象部署Iterator接口 `obj[Symbol.iterator] = generator`
- `next()`方法的参数就是yield表达式的返回值，所以第一次使用`next()`方法时，传递参数是无效的（第一次调用next是启动遍历器对象）
- `for...of`可自动遍历Generator生成的Iterator对象，无需调用`next()`
-	`Iterator.thorw()`可在函数体外抛出错误，然后在Generator函数体内捕获
	- catch一次就只能捕获一次
	- `throw()`接收的参数会被catch接收
	- `throw()`方法与全局的throw命令不同，后者只能被函数体外的catch语句捕获，而且两者无关。`throw()`可以使我们只用一个`try...catch`来处理多个yield表达式的错误
	- `throw()`被捕获之后，会附带执行一次`next()`
	- 内部错误如果没有被内部捕获，就不会再执行，`next()`返回`{value: undefined, done: true}`
	```
		var g = function* () {
				try {
						yield;
				} catch(e) {
						console.log('inter', e);
				}
		};
		var i = g();
		i.next();
		try {
				i.throw('a');
				i.throw('b');
		} catch(e) {
				console.log('outer', e);
		}
	```
- `Iterator.return()`
	- 通过return(value)返回给定的值value，并结束遍历
	- 如果generator函数内部有`try...finally`代码块，那么return调用后，会先执行finally，然后执行return
- `next()``throw()``return()`
	- 1.都是让generator恢复执行
	- 2.使用不同的语句替换`yield`表达式，`next(value)`将`yield`表达式替换成value，`throw(value)`将`yield`表达式替换成`thorw value`，`return(value)`将`yield`表达式替换成`return value`
- yield* 
	- 一个generator内部调用另一个generator，`yield* generator2()`
	- 被调用的generator有return语句，那么就可以向代理它的generator返回数据
	- `yield*`可遍历所有部署了Iterator接口的数据结构，比如数组、字符串等，相当于调用了`for...of`
	- `yield*`取出嵌套数组的所有成员
	```
	function* iterTree(tree) {
    if (Array.isArray(tree)) {
        for (let i = 0; i < tree.length; i++) {
            yield* iterTree(tree[i]);
        }
    } else {
        yield tree;
    }
	}
	```
- generator作为属性，可简写为 `{* attrname(){}}`
- generator函数的this
	- generator函数总是返回一个iterator，这个遍历器是generator函数的实例
	- 默认情况下拿不到this和this中的属性
- generator实现状态机
```
var clock = function* () {
    while(true) {
        console.log('Tick');
        yield;
        console.log('Tock');
        yield;
    }
};
```
- 应用
	- 异步操作的同步化表达
	- 控制流管理
	- 部署Iterator接口
	- 作为数组结构，对任意表达式提供类似数组的接口
### Generator的异步应用
- 定义：一个不连续的任务(中间可以插入其他任务)
- 传统异步编程方法
	- 回调函数:为什么Node.js中回调的第一个单数，必须是err，因为分段执行第一段后，原来的上下文已无法捕捉，此时抛出的错误只能当参数传入第二段
	- 事件监听
	- 发布/订阅
	- Promise对象，解决了多层回调之间的强耦合问题(callback hell)，但是带来了代码冗余，原来的语义都变成then而变得很不清楚

## 解构Destructuring
- 可结构数组、对象或迭代器
- 可迭代嵌套结构
- 如果解构的对象是null或undefined，得到TypeError
- 如果解构其他原始类型，得到undefined
- 可与迭代器协同使用，比如 `for(var [key] of map)` `for(var[,value] of map)`
- 可实现多重返回值
