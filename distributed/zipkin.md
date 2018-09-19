### 作用
- 分布式实时数据追踪系统（Distributed Tracking System）
- 聚集**来自各个异构系统**的实时监控数据，用来追踪**微服务架构下的系统延时问题**
- 应用系统需要进行装备（instrument）以向 Zipkin 报告数据。Zipkin 的用户界面可以呈现一幅关联图表，以显示有多少被追踪的请求通过了每一层应用。

### 结构
- Trace：Zipkin使用Trace结构表示对一次请求的跟踪
- Span：每个服务的处理跟踪是一个Span，可以理解为一个基本的工作单元
  - traceId：标记一次请求的跟踪，相关的Spans都有相同的traceId
  - id：span id
  - name：span的名称，一般是接口方法的名称
  - parentId：当前Span的父Span id，空表示根Span
  - timestamp：Span创建时的时间戳，使用的单位是微秒（而不是毫秒），所有时间戳都有错误，包括主机之间的时钟偏差以及时间服务重新设置时钟的可能性，所以应尽量记录duration
  - duration：持续时间使用的单位是微秒
  - annotations：注释用于及时记录事件；有一组核心注释用于定义RPC请求的开始和结束
  ```
    cs:Client Send，客户端发起请求；
    sr:Server Receive，服务器接受请求，开始处理；
    ss:Server Send，服务器完成处理，给客户端应答；
    cr:Client Receive，客户端接受应答从服务器；
  ```
  - binaryAnnotations：二进制注释，旨在提供有关RPC的额外信息
- Transport：收集的Spans必须从被追踪的服务运输到Zipkin collector，有三个主要的传输方式：HTTP, Kafka和Scribe
- Components
  - collector：负责将各系统报告过来的追踪数据进行**接收**
  - storage：默认Cassandra，可支持MySQL和ElasticSearch
  - api：json格式
  - ui：端口号默认为9411
