## 原理
- BSON格式，轻量级的二进制
- MongoDB传输协议
- 数据库存在 dbname.ns、dbname.0、dbname.1等文件中，每次增倍，最大2G，还会预分配数据文件以提高性能
- 内存映射存储引擎
- 数据库按照命名空间组织
  - 每种类型数据分开存放
  - 命名空间的元数据放在dbname.ns文件中
  - 每个命名空间的数据分为若干组，放到数据文件的某一个区域内，成为数据域
  - 每个命名空间可以有几个不同的数据域，在磁盘上不连续


## 基础概念
- 文档
  - 多个key及其关联的值有序地放置在一起就是文档，类似于有序字典
  - 有序的
  - key为字符串，可使用大部分UTF-8字符，区分大小写，不能重复
    - `\0`代表key的结尾
    - `.`和`$`有特殊用途
    - `_`开头的key保留
- 集合
  - 一组文档，文档类似于数据库的行，集合则类似于表
  - 集合是无模式的，不同key、不同值类型的文档可以放到同一个文档中
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
  - db.collect_name.findOne()
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
- 建议由客户端生成

## 增删改
- 插入先转换成BSON格式，最大16MB，建议批量插入
- db.collect_name.remove()不会删除集合本身和索引
- db.drop_collection(collect_name)删除所有数据会更快，同时会删除集合本身和索引
- 更新
  - 多个更新同时发生，最后的更新为准