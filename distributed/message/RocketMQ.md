### 特点
- 原生支持分布式
- 可保证严格的消息顺序
- 亿级消息的堆积能力，同时保持写入低延迟
- 丰富的消息拉取模式Push or Pull
- 自带NameServer进行分布式协调
- 消息失败重试机制、高效的订阅者水平扩展能力、强大的API、事务机制等等
- 消息默认都存储、天然负载均衡

### 核心概念
- Message
  - messageId
  - messageKey
  - Topic
  - Tag
  - Key
  - 大小建议不超过512K
- Topic
  - tags
  - subTopics
- Queue 1 : 1 Offset
- Broker
  - ASYNC_MASTER SYNC_MASTER SLAVE
  - 防止丢失消息
    - SYNC_MASTER(同步双写Master) \+ SLAVE
  - 高可用(可能丢失)
    - ASYNC_MASTER（异步复制Master） \+ SLAVE，也可以不用SLAVE
  - FlushDiskType
    - 建议用ASYNC_FLUSH
    - SYNC_FLUSH影响性能
- Producer
  - SendResult之SendStatus
    - FLUSH_DISK_TIMEOUT：Broker设置为SYNC_FLUSH而超过了syncFlushTimeout(默认5s)
    - FLUSH_SLAVE_TIMEOUT：Broker为SYNC_MASTER而且slave Broker同步超时(syncFlushTimeout)
    - SLAVE_NOT_AVAILABLE：Bbroker为SYNC_MASTER但是没有配置slave Broker
    - SEND_OK：并不意味着可靠，要防止丢失消息，必须配置SYNC_MASTER 或者 SYNC_FLUSH
  - FLUSH_DISK_TIMEOUT FLUSH_SLAVE_TIMEOUT同时borker关闭，则会丢失消息，此时可根据需要重发消息。但是SLAVE_NOT_AVAILABLE时重发消息时无效的，只能管理员来处理。
  - 默认同步发送消息
  - 如果要求高性能(比如大数据处理)：可改为异步发送，而且创建多个producer(3~5个已足够，要给每个producer设置InstanceName)
  - producer是线程安全的
  - RocketMQ提供了3中模式的Producer
    - NormalProducer
    - OrderProducer(producer端send有MessageQueueSelector，consumer使用MessageListenerOrderly)
    - TransactionProducer
- Consumer
  - 不同的Consumer Group可以消费同一个topic，它们各自有自己的consuming offsets
  - MessageListener
    - Orderly：不建议抛出异常，而是用ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT 替代。
    - Concurrently：不建议抛出异常，而是用ConsumeOrderlyStatus.RECONSUME_LATER 替代。
    - Blocking：不建议阻塞，会阻塞线程池
  - 线程数：setConsumeThreadMin setConsumeThreadMax
  - ConsumeFromWhere
    - CONSUME_FROM_LAST_OFFSET 忽略历史消息
    - CONSUME_FROM_FIRST_OFFSET 消费broker中的每一条消息
    - CONSUME_FROM_TIMESTAMP 可消费随后指定时间以后的消息
  - 注意重复消息
- NameServer
  - 优先级：Programmatic Way > Java Options > Environment Variable > HTTP Endpoint(`http://jmenv.tbsite.net:8080/rocketmq/nsaddr`)
- 环境配置
  - jvm
    - `-server -Xms8g -Xmx8g -Xmn4g`
    - 禁用偏向锁减少暂停
    - G1：`-XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30`
    - `-XX:MaxGCPauseMillis`不能设置太小，否则Minor GC太频繁
    - 日志：`-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m`。如果GC日志增加了borker的延迟，则考虑使用`-Xloggc:/dev/shm/mq_gc_%p.log`

### 组成
- NameServer：大致相当于 jndi技术，更新和发现 broker服务。一个几乎无状态节点，可集群部署，节点之间无任何信息同步
- Broker：消息中转角色，负责存储和转发消息。Broker分为Master和Slave
  - Master和Slave的对应关系通过指定相同的BrokerName，不同的BrokerId来定义，BrokerId为0表示Master，BrokerId非0表示Slave
  - 所有的Broker和**Name Server**上的节点建立**长连接**,**定时注册Topic信息到所有Name Server**
  - 单个broker和**所有nameserver**保持长连接
  - 每隔30秒（此时间无法更改）向**所有nameserver**发送**心跳**，心跳包含了自身的topic配置信息
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
  - 重试机制：指定时间内没有发送成功，则重试N次
  ```
  producer.setRetryTimesWhenSendFailed(num);
  producer.send(msg, xxms);
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
  - 失败重试
    - timeout：内部不断重试直至发送成功为止
    - exception：消息重试直至进入死信队列(CONSUME_SUCCESS RECONSUME_LATER，实际使用过程中可判断已重复一定次数然后使用其他方式处理)
  - 多线程消费
  ```
  consumer.setConsumerThreadMin(10);
  consumer.setConsumerThreadMax(20);
  ```
  
### 存储特点
- Consumer消费消息过程，使用了零拷贝
- 零拷贝使用mmap+write方式，因为有小块数据传输的需求，效果会比sendfile更好
- sendfile大块文件传输效率高

### Group
- RocketMQ只有一种模式，即发布订阅模式
- 生产者、消费者都可以有多个group
- group机制让RocketMQ**天然支持消息负载均衡**

### 消费方式
- 集群消费(默认，即负载均衡消费)  MessageModel.CLUSTERING
- 广播消费：Pub/Sub模式，消息会发给Consume Group中的每一个消费者进行消费。MessageModel.BROADCASTING
  `consumer.setMessageModel(MessageModel.BROADCASTING);`

### Pull Consumer
- 本质上PUSH PULL都是拉取模式PULL，即消费者从MQ中轮询取得消息
- 在Push模式下，Consumer把轮询过程封装了，并注册了MessageListener监听器，取到消息后，唤醒MessageListener监听器中的consumeMessage()进行消费，所以给我们造成了感觉上好像是“推消息”
- 在Pull模式下，需要特别注意的是，本质上是从一个Topic下的所有Queue进行拉取，而且每个Queue都必须记录**拉取位置**，否则会导致重复消费。还有**拉取的时间间隔，拉取的大小**等等。这些在MQPullConsumerScheduleService中设置
- Push方式都有返回值，Pull的doPullTask的返回值是void，需要注意每条消息消费的异常情况

### 消息重复消费
- consume broker的重启
- 水平扩容
- 不遵守先订阅后生产消息
 
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
  - `producer.send(msg, MessageQueueSelector)` 重写select方法
  - `registerMessageListener(MessageListenerOrderly)`
  - ConsumeOrderlyStatus
    - SUCCESS
    - SUSPEND_CURRENT_QUEUE_A_MOMENT：延缓多长时间执行，在当前队列里
    - COMMIT：Deprecated
    - ROLLBACK：Deprecated
  - 分类
    - 全局有序消息
    - 局部有序消息
  - 原理
    - topic只是消息的逻辑分类，内部由queue组成(一个topic可能有多个queue)
    - topic的queue数目改成1，发送有序则消费必然有序，此时为全局有序
    - 发送时按照特定规律发送到不同的queue，而消费时按照queue进行消费，则可保证局部有序，且性能比全局有序大大提升
- 延时消息(Scheduled Message)
  - 当 producer 将消息发送到 broker 后，会延时一定时间后才投递给 consumer 进行消费
  - 延时等级：1s，5s，10s，30s，1m，2m，3m，4m，5m，6m，7m，8m，9m，10m，20m，30m，1h，2h。level=0，表示不延时。level=1，表示 1 级延时，对应延时 1s。level=2 表示 2 级延时，对应5s，以此类推
  - 比如网购，下单后超时未支付则关闭订单
  - `msg.setDelayTimeLevel(3);`
  
### 批量发送
- 批量发送小型消息以提升性能
- `send(List<Message>)`
- 要求topic waitStatusOK相同，且不支持延迟消息
  
### 解决分布式事务
- 限制：
  - 不支持延迟和批量发送
  - 事务性消息可能被重复消费，未来防止重复消费太多次，默认消费15次(transactionCheckMax可修改)，超过次数则broker丢弃消息并打印错误日志(可通过覆盖AbstractTransactionCheckListener修改处理方法)
  - 事务性消息可能被重复check，在transactionMsgTimeout(broker中配置)时间后被check，用户属性CHECK_IMMUNITY_TIME_IN_SECONDS可覆盖该配置
  - 已提交的Message重新提交可能会失败，如果要保证消息不丢失而且保证事务完整，建议用同步双写
  - Message的Producer ID不能合其他类型的Message共享。
- 事务状态
  - TransactionStatus.CommitTransaction 允许consumer消费这条消息
  - TransactionStatus.RollbackTransaction 消息将被删除，不允许消费
  - TransactionStatus.Unknown 中间状态，MQ需要检查来决定状态
- producer端：使用**TransactionMQProducer**(T1业务表/T2**消息确认表**)
- producer.setTransactionListener(**TransactionListener**)
  - `TransactionStatus executeLocalTransaction()`：当发送半程消息成功用于执行本地事务
  - `TransactionStatus checkLocalTransaction()`：用于检查本地事务状态并响应MQ的检查请求
- producer.setExecutorService(ExecutorService)
- sendMessageInTransaction(msg, arg)
  - 阶段1：发送prepare消息，消息到MQ之后回调**本地事务**，此时事务消息对consumer还**不可见**。
  - 阶段2：执行本地事务(executeLocalTransaction)，一个事务中对T1(业务表)/T2(消息确认表)操作
    - T2(status、updatetime)
  - 阶段3：发送确认消息（向MQ发送消息，prepare消息是否对consumer可见由这里的消息中的状态码决定）
  - 定时任务扫描T2表(指定status和updatetime条件)将确认消息发送失败的消息找出来，更新updatetime，并发送给MQ
- consumer端(T3业务表/T4定时任务表/T5日志记录表)
  - 有一个定时任务扫描T5获取一段时间内的数据(T4表中记录time，每次启动先更新time)发送给producer端，让producer更新T2表的status、updatetime

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
- autoCreateTopicEnable：线上应关闭，否则无法负载均衡。应该手动创建topic。

### RocketMQ Filter
- Tag机制已经足够应付日常绝大数的过滤功能，除非要求性能高
- RocketMQ Filter的性能要更高效些，因为RocketMQ是在broker上将过滤后的数据发往filter，然后消费者直接从filter上取得数据；而ActiveMQ是消费者直接在broker上进行过滤消费

### 其他
- logappender：自定义日志输出
- OpenMessaging：支持OpenMessaging分布式消息分发规范
