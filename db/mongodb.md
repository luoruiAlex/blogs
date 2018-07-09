## 原理
- BSON格式，轻量级的二进制
- MongoDB传输协议，驱动在TCP/IP协议的基础上简单封装了MongoDB协议，用来与MongoDB交互
- 每个数据库有几个独立的文件，每个数据有一个.ns文件和若干数据文件，即 dbname.ns、dbname.0、dbname.1等文件中，每次增倍，最大2G，这样可以让小数据库不浪费太多的磁盘空间，大数据使用磁盘上的连续空间。此外，还会预分配数据文件以提高性能
- 内存映射存储引擎
- 数据库按照命名空间组织
  - 每种类型数据分开存放
  - 命名空间的元数据放在dbname.ns文件中
  - 每个命名空间的数据分为若干组，放到数据文件的某一个区域内，成为数据域
  - 每个命名空间可以有几个不同的数据域，在磁盘上不连续


## 基础概念
- 文档
  - 多个key及其关联的值有序地放置在一起就是文档，类似于有序字典
  - 有序的(顺序不同的不是同一个文档)
  - key为字符串，可使用大部分UTF-8字符，区分大小写，不能重复
    - `\0`代表key的结尾
    - `.`和`$`有特殊用途
    - `_`开头的key保留
- 集合
  - 一组文档，文档类似于数据库的行，集合则类似于表
  - 集合是无模式的，不同key、不同值类型的文档可以放到同一个集合中
  - 集合命名
    - 不能是空字符串""
    - 不能含`\0`
    - 不能以`system.`，这是系统集合的前缀
    - `$`保留
  - 子集合
    - 使用`.`按命名空间分开
- 数据库
  - 集合组成数据库
  - 一个MongoDB实例可承载多个数据库
  - 命名
    - 不能是空字符串""
    - 不能含空格、`.`、`$`、`\`、`/`、`\0`
    - 应全部小写
    - 最多64字节
  - 特殊作用数据库
    - admin 这个数据库里面的用户拥有所有数据库的权限
    - local 这个数据库永远不会被复制，可用于存储先于本地单台服务器的任意集合
    - config 保存分片的相关信息
 
 ## 操作
 - 启动服务器 mongod --dbpath xx
### Shell
- 启动：mongo
- 一个功能完备的JavaScript解释器，可运行任意JavaScript程序
- 默认连接到MongoDB服务器的test数据库，并将连接赋值给全局变量db
- 非JavaScript的扩展
  - use db_name 切换数据库
  - db.collect_name.insert(jsObj)，有一个额外的键`_id`
  - db.collect_name.find()
  - var xx = db.collect_name.findOne()
  - db.collect_name.update({key: value}, obj) {key, value}为限定条件，obj的键值对会加入文档
  - db.collect_name.remove({key: value}) 如果不加限定条件则会删除所有
  - help
  - db.help()
  - db.collect_name.help()
  - 输入函数名，不加括号，则会显示函数源码
  
## 数据类型
- null 空值或者不存在的字段
- boolean true/false
- 32位整数 Shell中不可用
- 64位整数 Shell中不可用
- 64位浮点数 Shell中的数字都是这话类型
- 字符串 任意UTF-8字符串
- 符号  Shell中不支持，会将符号转为字符串
- 对象id 12字节的唯一ID
- 日期 标准纪元开始的毫秒数，不存储时期
- 正则表达式 采用JS的语法
- 代码
- 二进制数据 Shell中不可用
- 最大值、最小值
- 未定义 undefined
- 数组
- 内嵌文档 即包含别的文档，相当于JS中的对象作为value

## `_id`
- 默认是ObjectId类型
- 建议在客户端由驱动程序生成
- 12字节，时间戳(4) + 机器标识符(3) + PID(2) + 计数器(3)
- 前面9字节提供了秒级别的唯一性
- 时间戳在前，ObjectId大致会按照插入的顺序排序，前面4个字节还可以获取时间信息
- 时间戳的实际值不重要，因为总是不停增加，所以无需同步服务器时间

## 增删改
- 插入先转换成BSON格式，最大16MB，建议批量插入
- db.collect_name.remove()不会删除集合本身和索引
- db.collect_name.drop()删除所有数据会更快，同时会删除集合本身和索引
- 更新
  - 更新操作是原子的，多个更新同时发生，最后的更新为准
- 修改器
  - $inc `db.blog.update({"url": "aaa"}, {"$inc": {"page": 1}})` 只能用于数字类型
  - $set $unset
  - $push 往数组中添加 $pop 
  - $ne 判断值是否在数组中`db.blog.update({"authors": {"$ne": "Michael"}}, XX)`
  - $addToSet添加可避免重复
  - 查询修改数组 `db.blog.update({"comments.author": "John"}, {"$set": {"comments.$.author": "Jim"}})`
  - 修改速度：如果不改变文档的大小，则非常快，否则性能下降
- upsert 为upadte函数的第3个参数，为true时表示如果没有匹配的则创建新的
  - db.blog.save为一个shell函数，可以在文档不存在时创建
- multi  为update函数的第4个参数，为true表示更新符合条件的所有文档，否则只更新第一个
- 返回已更新的文档
- 连接和请求 每个连接有一个请求队列，队列中的请求都处理完才会执行后续的请求

## 查询
- 第1个参数为查询条件
- 第2个参数指定返回的键
  - 永远返回 \_id
  - `db.blog.find({}, {"username": 1, "email": 1})` 指定返回username和email
  - `db.blog.find({}, {"username": 0)` 指定不返回username
- 值必须是常量，不能引用文档中的其他键的值
- 查询条件
  - $lt $lte $gt $gte $ne db.users.find({"age" {“gte": 18, "$lte": 30}})
  - $in \[] $nin
  - $or `{"sort": [{"ticket_no": 725}, {"winner": true}]}`
  - $not
- 特定于类型的查询
- where
```
db.blog.find({"$where": function() {
 for (var current in this) {
  for (var other in this) {
    if (current != other && this[current] == this[other] {
      return true;
    }
  }
 }
 return false;
}});
```
- 游标
  - `var cursor = db.collection.find()`
  - `while (cursor.hasNext()) { obj = cursor.next();}`
  - `cursor.forEach(function(x){});
  - 几乎所有的游标对象的方法都返回游标本身，在find的时候，不会立即查询数据库，如下几个是等价的
    - `var cursor = db.collection.find().sort({"x": 1}).limit(1).skip(10);`
    - `var cursor = db.collection.find().skip(10).sort({"x": 1}).limit(1);`
    - `var cursor = db.collection.find().limit(1).sort({"x": 1}).skip(10);`
  - hasNext()时，会查询前100个或者前4MB数据(两者中取小)
  - 避免使用skip略过大量结果
    - 分页时不用skip：使用上次的查询结果作为条件，可实现分页
    - 随机选取文档不用skip：插入时加一个额外的随机key，查询的时候也生成随机值作为条件
