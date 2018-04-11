### 实用ChannelHandler
  - ByteToMessageCodec 系统编码界面框架
  - LengthFieldBaseFrameDecoder 基于长度的半包解码器
  - LoggingHandler 码流日志打印
  - SslHandler SSL安全认证
  - IdleStateHandler 链路空闲检查
  - ChannelTrafficShapingHandler 流量整形
  - Base64Decoder和Base64Encoder Base64编码解码
  
### Tcp Server的创建
  1. 创建NioEventLoopGroup的两个实例(也可以只要一个)bossGroup、workerGroup作为Reactor线程池
  2. 反射创建NioServerSocketChannel实例
  3. 设置TCP的backlog参数，指定内核为此套接口排队的最大连接个数。
  3.1 对于给定的监听套接口，内核要维护连个队列： 未链接队列、已连接队列
