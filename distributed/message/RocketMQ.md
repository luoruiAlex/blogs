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
  - 每隔30秒（此时间无法更改）向所有nameserver发送**心跳**，心跳包含了自身的topic配置信息
  - 负载均衡
    - 一个topic分布在多个broker上，一个broker可以配置多个topic，它们是多对多的关系
    - 如果某个topic消息量很大，应该给它多配置几个队列，并且尽量多分布在不同broker上，减轻某个broker的压力
    - topic消息量都比较均匀的情况下，如果某个broker上的队列越多，则该broker压力越大
- Producer
  - Producer与Name Server其中一个节点建立连接
  - 定期(默认30秒)从Name Server取Topic信息，并与提供该Topic信息的Master建立长连接
  - 可建立集群
  - code
  ```
  DefaultMQProducer producer = new DefaultMQProducer("producerGroupName");
  producer.setNamesrvAddr("10.1.54.121:9876;10.1.54.122:9876");
  producer.start();
  ...
  producer.send
  ...
  producer.shutdown();
  ```
- Consumer
  - Consumer 与Name Server 集群中的其中一个节点（随机选择）建立长连接
  - 定期(默认30秒)从Name Server 取Topic 路由信息，并向提供Topic服务的Master、Slave建立长连接，且定时向Master、Slave发送心跳
  - Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定
  - 可建立集群
  - code
  ```
  DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumerGroupName");
  consumer.setNamesrvAddr("10.1.54.121:9876;10.1.54.122:9876");
  //设置消费策略
  //CONSUME_FROM_LAST_OFFSET 默认策略，从该队列最尾开始消费，即跳过历史消息
  //CONSUME_FROM_FIRST_OFFSET 从队列最开始开始消费，即历史消息（还储存在broker的）全部消费一遍
  //CONSUME_FROM_TIMESTAMP 从某个时间点开始消费，和setConsumeTimestamp()配合使用，默认是半个小时以前
  consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
  //设置consumer所订阅的Topic和Tag，*代表全部的Tag
  consumer.subscribe("TopicTestConcurrent", "TagA || TagC || TagD");
  consumer.registerMessageListener...
  consumer.start()
  ```
  
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

### 消息类型
- 无序消息
  - `new Message(topic, tags, body)`
  - `producer.send(msg)`
  - `registerMessageListener(MessageListenerConcurrently)`
- 有序消息
  - `new Message(topic, tags, keys, body)`
  - `producer.send(msg, MessageQueueSelector)`
  - `registerMessageListener(MessageListenerOrderly)`
  - 分类
    - 全局有序消息
    - 局部有序消息
  - 原理
    - topic只是消息的逻辑分类，内部由queue组成(一个topic可能有多个queue)
    - topic的queue数目改成1，发送有序则消费必然有序，此时为全局有序
    - 发送时按照特定规律发送到不同的queue，而消费时按照queue进行消费，则可保证局部有序，且性能比全局有序大大提升
- 延时消息
  - 当 producer 将消息发送到 broker 后，会延时一定时间后才投递给 consumer 进行消费
  - 延时等级：1s，5s，10s，30s，1m，2m，3m，4m，5m，6m，7m，8m，9m，10m，20m，30m，1h，2h。level=0，表示不延时。level=1，表示 1 级延时，对应延时 1s。level=2 表示 2 级延时，对应5s，以此类推
  - 比如网购，下单后超时未支付则关闭订单
  - `msg.setDelayTimeLevel(3);`
  
### 使用细节
- 消息过滤
  - 可用多个topic区分，也可用同一个topic下不同tag区分
  - 一般用tag来区分同一个topic下相互关联的消息，而不同topic之间的消息没有必然联系
- 订阅关系一致性
  - 使用同一个 group name，订阅的 tag 也一样的consumer实例才能组成consumer集群
  - 订阅关系一致性：订阅的 Topic 和 Topic中的tag必须都一致
  - 作用：保证了同一个消费者集群中 consumer 实例的正常运行，避免消息逻辑的混乱和消息的丢失。
  - 在 producer 端要做好消息的分类，便于 consumer 可以使用 tag 进行消息的准确订阅，而在 consumer 端，则要保证订阅关系一致性
- 消息重试
  - 定义：当消费者消费消息失败后，broker 会重新投递该消息，直到消费成功。消费者返回状态值给broker来决定是否重试。
  - 消息重试只针对集群消费模式，广播消费没有消息重试的特性，消费失败之后，只会继续消费下一条消息
  - 默认每条消息最多重试16次，防止消息本身为脏数据。每次重试间隔不断增加。彻底失败之后将消息放到相对应的死信队列中，然后人工补偿处理。
  - 失败情况
    - 返回 ConsumeConcurrentlyStatus.RECONSUME_LATER
    - 返回 null
    - 抛出异常：不会重试
- 消息幂等
  - 对于一条消息的处理结果，不管这条消息被处理多少次，最终的结果都一样
  - 对于一些允许消息重复的场景，大可以不必关心消费幂等
  - 对于那些不允许消息重复的业务场景来说，处理建议就是通过业务上的唯一标识来作为幂等处理的依据
