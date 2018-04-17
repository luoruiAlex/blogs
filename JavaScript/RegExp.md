#### 2个类共6个方法
### 创建正则对象
- 字面量 性能高
- 构造函数 可在运行时动态决定正则，转义需要两个斜杠 `\\`

### RegExp对象
#### 属性
- reg.lastIndex，用于记录上一次匹配结束的位置，包括exec方法。lastIndex是唯一可写的属性
- reg.i reg.g reg.m(改变$和^的行为) reg.source

#### 方法
##### 1.`exec`
- 每次调用都只返回一个匹配对象，`[ 'dbd', 'b', index: 7, input: 'cdbbdbsdbdbz' ]`
- 拿到全部匹配就需要while循环获取直到返回null为止
##### 2.`test`
- reg.test(str)与reg.exec(str)!=null是等价的
- 当reg有设置了g(global匹配时)，会从`reg.lastIndex`开始继续匹配，可设置reg.lastIndex的值或者取消全局匹配来重新设置从头开始匹配

#### String对象
##### 1.match
- 如果global，返回所有匹配的数组 `["dbbd", "dbd"]`
- 没有global，返回第一个匹配的字符串，以及相应的捕获内容 `["dbbd", "bb", index: 1, input: "cdbbdbsdbdbz"]`
##### 2.replace
- 两个参数都是字符串： 只替换第一个
- 第一个参数是正则，第二个参数是字符串： 正则无global则只替换第一个
- 第一个参数是正则，第二个参数是带$符的字符串
  - `'这是一段原始文本,"3c这要替换4d"!'.replace( /([0-9])([a-z])/g,"$1" )` =>`这是一段原始文本,"3这要替换4"!`
  - $1-99表示正则中的第i个子表示...
- 第一个参数是正则，第二个为函数`function (arg1匹配的整体, arg2第一个子表达式, arg3第二个子表达式, arg4子匹配在stringObject中出现的位置, arg5stringObject本身)`, 根据函数返回值为相应替换的文本
##### 3.search
 - 返回index或者-1
##### 4.split
- split(separator,limit) 无limit则表示不限制子串的个数
