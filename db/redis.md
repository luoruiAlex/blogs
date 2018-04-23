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
