### 概念
- Provider/MessageProvider：生产者，分为QueueSender和TopicPublisher
- Consumer/MessageConsumer：消费者，分为QueueReceiver和TopicSubscriber
- PTP：Point To Point，点对点通信消息模型
- Pub/Sub：Publish/Subscribe，发布订阅消息模型
  - 必须先订阅
  - 订阅进程必须一直处于运行状态
- Destination
  - Queue：队列，目标类型之一，和PTP结合
  - Topic：主题，目标类型之一，和Pub/Sub结合
- MessageListener：如果注册了消息监听器，一旦消息到达，将自动调用监听器的onMessage方法

### 消息组成
- 消息头：预定义若干字段用于客户端与JMS提供者之间识别和发送消息
  - JMSDestination
  - JMSReplyTo
  - JMSDeliveryMode：持久模式和非持久模式
  - JMSMessageID
  - JMSTimestamp
  - JMSCorrelationID： 通常用来链接响应消息与请求消息
  - JMSRedelivered：指出消息被过早地发送给了 JMS 程序，程序不知道消息的接收者是谁；由提供者在接收过程中设置
  - JMSType
  - JMSExpiration
  - JMSPriority
- 消息属性
  - 提供给应用程序，可用于消息过滤等功能
- 消息体
  - Text message : javax.jms.TextMessage，表示一个文本对象。
  - Object message : javax.jms.ObjectMessage，表示一个JAVA对象。
  - Bytes message : javax.jms.BytesMessage，表示字节数据。
  - Stream message :javax.jms.StreamMessage，表示java原始值数据流。
  - Map message : javax.jms.MapMessage，表示键值对

### 生产者
- 代码
```
ConnectionFactory factory = new ActiveMQConnectionFactory();
Connection connection = factory.createConnection();
connection.start();
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
Destination destination = session.createQueue("queue1");
MessageProducer messageProducer = session.createProducer(destination);
messageProducer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
TextMessage message = session.createTextMessage();
message.setText("Hello, ActiveMQ");
messageProducer.send(message);
if (connection != null) {
  connection.close();
}
```
- createSession(事务, 确认模式)
  - 事务为true时，确认只能为AUTO_ACKNOWLEDGE
  - 事务为false时，确认有3中模式
    - AUTO_ACKNOWLEDGE：自动确认，客户端发送和接收消息不需要做额外的工作
    - CLIENT_ACKNOWLEDGE：客户端确认。客户端接收到消息后，必须调用javax.jms.Message的acknowledge方法。jms服务器才会删除消息
    - DUPS_OK_ACKNOWLEDGE：允许副本的确认模式。一旦接收方应用程序的方法调用从处理消息处返回，会话对象就会确认消息的接收；而且允许重复确认

### JMS Selectors
- session.createConsumer(destination, condition)
- condition为字符串，比如"a >= 1"，本质上是SQL92语法

### 持久化订阅
- 防止消费者重启后丢失消息
- connection.setClientID("id1")唯一的ID作为标识
- session.createDurableSubscriber(destination, "id1")

### 与Spring整合
- JmsTemplate
