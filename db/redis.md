作用： 数据库、缓存、消息中间件

## 命令行
### string
- SET key value [EX seconds] [PX milliseconds] [NX|XX]
  - NX只有键不存在的时候才可以设置成功
  - XX：只有键在已经存在的时候才可以设置成功
  - SETRANGE key offset value，如果value比原来的value要短，用`\x00`来补充
- GET key
  - GETRANGE key start end 
  - `(nil)`表示没有值
- GETSET key value 设置值并返回旧值或`(nil)`
- MGET key1 [key2 ... ]
- MSET key1 value1 [key2 ... ]
- STRLEN key 获取value的长度，如果不含key则返回0
- INCR 将value的值递增1，如果key不存在则设置为0
  - INCRBY INT
  - INCRBYFLOAT FLOAT
  - DECR
  - DECRBY
- APPEND key value 返回追加后的字符串长度
- DBSIZE
  - `(integer): size值`
### hash
- HSET key field value
- HGET key field
- HSETNX key field value只有key不存在的时候成功，返回1，否则返回0
- HMSET HMSET
- HGETALL key 返回field和value
- HKEYS key
- HVALS key
- HEXISTS key field
- HLEN key
- HINCRBY key field increment
  - HINCRBYFLOAT key field increment
- HDEL key field[field2...]
### list
- 双向链表实现
- LPUSH key value1[...value2] 在左侧插入
  - RPUSH
  - LPUSHX 只有当list存在的时候才插入
  - RPUSHX
- LPOP key 移除并返回左侧第一个元素
  - RPOP
- LLEN key 返回list长度，不存在返回0，不是list返回error
- LRANGE key start stop
- LREM 从存于 key 的列表里移除前 count 次出现的值为 value 的元素
- LINDEX key index 返回对应位置上的元素
- LSET key index value
- LTRIM key start stop 修剪已存在的list
- LINSERT key BEFORE|AFTER pivot value
### set
- 使用值为空的hash table实现，增删复杂度为O(1)
- SADD key member[...member2]
- SMEMBERS key 返回所有元素
- SISMEMBER key member 是否包含
- SREM key member[...member2]
- SPOP key [count] 随机删除并返回
- SRANDMEMBER key [count] 随机返回
- SDIFF key [key2...] 差集
  - SDIFFSTORE destination key [key2...] 不返回结果，而是将结果存放在destination中，如果destination已存在则覆盖
- SINTER key [key2...] 交集
  - SINTERSTORE
- SUNION key [key2...] 并集
  - SUNIONSTORE
- SCARD key 返回元素个数
- SMOVE source destination member
### sorted set
- hash table和skip list实现，比list更耗内存
- ZADD key [NX|XX] [CH] [INCR] score member[... score2 member2]
  - XX 仅更新已存在的成员，不新增
  - NX 仅新增，不更新
  - CH 返回changed的成员个数
  - INCR 对成员的分数进行递增操作
- ZSCORE key member 返回member的score值或者返回(nil)
- ZRANGE key start stop [WITHSCORES] 按照元素的分数从小到大的顺序返回指定索引start到stop之间所有元素（包含两端）
  - ZREVRANGE score从大到小排列
  - ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
  - ZREVRANGEBYSCORE
- ZINCRBY key increment member 如果member不存在则增加一个，score为increment
- ZCARD key 返回元素个数
- ZCOUNT ZCOUNT key min max 默认包括score值等于min或max
- ZREM key member [member ...]
  - ZREMRANGEBYRANK key start stop
  - ZREMRANGEBYSCORE
- ZRANK key member 返回排名
  - ZREVRANK key member
- ZINTERSTORE
  - ZUNIONSTORE
### KEY
- KEYS pattern
- EXISTS key[ke2...] 判断key是否存在，返回1表示存在，0表示不存在
- SELECT NUM 默认有16个数据库，默认连到0号数据库，select可切换数据
- TYPE key，返回string list set zset hash等不同类型，不存在的返回none
- EXPIRE key seconds
  - EXPIREAT key timestamp
  - PEXPIRE key milliseconds
- TTL key 返回剩余时间，单位是秒
  - PTTL key 毫秒
  - PERSIST key
- DEL key
- RENMAE key
- DUMP key
  - RESTORE key[ttl serialized-value]
- MOVE key db
### 连接命令
- PING [echocontent]  测试连接是否可用或者测试连接的延时
- ECHO messge 返回消息
- QUIT 请求服务器关闭连接
- AUTH 为请求带上密码
  - CONFIG SET requirepass xxx后就需要带上密码
  
## SORT排序
- 可对list、set、sorted set的key进行排序，并完成类似join查询的任务
- SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC|DESC] [ALPHA] [STORE destination]
  - 默认ASC
  - ALPHA 按字母ASCII码大小排序
  - LIMIT offset count offset偏移量，count是数目
  - GET参数不影响排序，它的作用是使SORT命令返回的结果不再是元素自身，而是GET参数中指定的值
  - BY只能有一个，而GET可以有很多个
- 性能
  - 时间复杂度 O(n + mlongm)
  - 尽可能减少待排序键中的元素的数量（减小n）
  - 使用LIMIT参数值获取需要的数据（减小m）
  - 如果要排序的数据量较大，尽可能使用STORE参数将结果缓存

## 配置
### 配置文件 redis.conf
- bind
- port
- timeout
- loglevel
  - debug
  - verbose
  - notice 适合生产环境
  - warning
- daemonize
  - yes redis以守护进程模式运行
- logfile "" 日志的记录方式，默认为标准输出
- databases 16    //默认数据库数量为16个，编号从0到15
- pidfile 设置redis的PID文件位置
- save `<seconds>` `<changed>` 多少sec至少有多少个changed才将其同步到磁盘中的数据文件里，比如 save 900 10
- rdbcompression yes  //存储本地数据库是否启用LZF压缩，默认yes
- dbfilename dump.rdb //指定本地数据库文件名，默认为dump.rdb
- dir 设置持久化文件存放位置

## 事务
### 概念与原理
- 一组命令的集合，要么都执行，要么都不执行
- 先将属于一个事务的命令发送给Redis，然后在让Redis一次执行这些命令
### 命令
- MULTI
- EXEC 
  - 执行事务中所有在排队等待的指令并将链接状态恢复到正常
  - 当使用WATCH 时，只有当被监视的键没有被修改，且允许检查设定机制时，EXEC会被执行
  - EXEC前如果客户端断线，事务队列会被情况，断线前已开始EXEC，则全部执行
- WATCH key [key2...]
  - 被监控的键一旦被修改或删除，之后的事务就不会执行
  - 监控一直会持续到EXEC命令
  - UNWATCH 如果执行EXEC 或者DISCARD， 则不需要手动执行UNWATCH
- DISCARD
  - 取消一个事务中所有在排队等待的指令，并且将连接状态恢复到正常
  - 如果已使用WATCH，DISCARD将释放所有被WATCH的key
### 容错
- 有一个命令语法错误就直接返回，全部不执行
- 一个命令运行错误，其他命令继续执行
- 不支持回滚
