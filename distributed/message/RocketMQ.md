### 特点
- 原生支持分布式
- 可保证严格的消息顺序
- 亿级消息的堆积能力，同时保持写入低延迟
- 丰富的消息拉取模式Push or Pull
- 自带NameServer进行分布式协调
- 消息失败重试机制、高效的订阅者水平扩展能力、强大的API、事务机制等等

### Group
- RocketMQ只有一种模式，即发布订阅模式
- 生产者、消费者都可以有多个group
- group机制让RocketMQ天然支持消息负载均衡
 
### 部署模式
- 单Master：宕机不可用
- 多Master：宕机的Master上未被消费的消息在Master没有恢复之前不可以订阅
- 多Master多Slave（异步复制）：主备之间短暂延迟，MS级别。Master宕机，消费者可以从Slave上进行消费，不受影响，但是Master的宕机，会导致丢失掉极少量的消息
- 多Master多Slave模式(同步双写)：同步的性能比异步稍低，不丢失消息
