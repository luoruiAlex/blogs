### 特点
- 原生支持分布式
- 可保证严格的消息顺序
- 亿级消息的堆积能力，同时保持写入低延迟
- 丰富的消息拉取模式Push or Pull
- 自带NameServer进行分布式协调
- 消息失败重试机制、高效的订阅者水平扩展能力、强大的API、事务机制等等

### 组成
- NameServer：大致相当于 jndi技术，更新和发现 broker服务。一个几乎无状态节点，可集群部署，节点之间无任何信息同步
- Broker：消息中转角色，负责存储和转发消息。Broker分为Master和Slave
  - Master和Slave的对应关系通过指定相同的BrokerName，不同的BrokerId来定义，BrokerId为0表示Master，BrokerId非0表示Slave
  - 所有的Broker和Name Server上的节点建立长连接,定时注册Topic信息到所有Name Server
  - 单个broker和所有nameserver保持长连接
- Producer
  - Producer与Name Server其中一个节点建立连接
  - 定期从Name Server取Topic信息，并与提供该Topic信息的Master建立长连接
  - 可建立集群
- Consumer
  - Consumer 与Name Server 集群中的其中一个节点（随机选择）建立长连接
  - 定期从Name Server 取Topic 路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向Master、Slave发送心跳
  - Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定
  - 可建立集群
  
### 存储特点
- Consumer消费消息过程，使用了零拷贝
- 零拷贝使用mmap+write方式，因为有小块数据传输的需求，效果会比sendfile更好
- sendfile大块文件传输效率高

### Group
- RocketMQ只有一种模式，即发布订阅模式
- 生产者、消费者都可以有多个group
- group机制让RocketMQ天然支持消息负载均衡
 
### 部署模式
- 单Master：宕机不可用
- 多Master：宕机的Master上未被消费的消息在Master没有恢复之前不可以订阅
- 多Master多Slave（异步复制）：主备之间短暂延迟，MS级别。Master宕机，消费者可以从Slave上进行消费，不受影响，但是Master的宕机，会导致丢失掉极少量的消息
- 多Master多Slave模式(同步双写)：同步的性能比异步稍低，不丢失消息
