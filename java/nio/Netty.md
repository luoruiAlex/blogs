
### 优点
- 可靠性能力补齐
- 定制能力强
- 解决了已发现的所有NIO的BUG，比如`epoll bug`

### 模型
```
/**
     * 
     * 
     * 
     * 
     *                   ________________________                                 __________________________
     *                  |                        |                               |                          |    
     *                  |   <-----Inbound-----   |                               |   ---inbound------- >    |   ________
     *                  |   _____        ______  |                               |    _______      ____     |  |        |
     *      _______     |  |     |       |    |  |                               |    |     |     |    |    |  |        |  
     *     |       |    |  |  ②  |       |  ③ |  |      ___________________      |    |  ⑤  |     |  ⑥ |    |  |        |
     *     |       |    |  |_____|       |____|  |     |                   |     |    |_____|     |____|    |  |        |     
     *     |client |----|-------______-----------|-----|      network      |-----|--------------------------|--| server |
     *     |       |    |       |     |          |     |___________________|     |          ______          |  |        |
     *     |       |    |       |  ①  |          |                               |          |     |         |  |        |         
     *     |       |    |       |_____|          |                               |          |  ④  |         |  |________|
     *     |       |    |                        |                               |          |_____|         |
     *     |_______|    |   -----Outbound--->    |                               |    <-----outbound----    | 
     *                  |___ChannelPipeline______|                               |______ChannelPipeline_____| 
     *                                                                               
     *  ①：StringEncoder继承于MessageToMessageEncoder，而MessageToMessageEncoder又继承于ChannelOutboundHandlerAdapter
     *  ②：HelloWorldClientHandler.java
     *  ③：StringDecoder继承于MessageToMessageDecoder，而MessageToMessageDecoder又继承于ChannelInboundHandlerAdapter
     *  ④：StringEncoder 编码器
     *  ⑤：StringDecoder 解码器
     *  ⑥：HelloWorldServerHandler.java
     * 
```

### 实用ChannelHandler
- ByteToMessageCodec 系统编码界面框架
- LengthFieldBaseFrameDecoder 基于长度的半包解码器
- LoggingHandler 码流日志打印
- SslHandler SSL安全认证
- IdleStateHandler 链路空闲检查
- ChannelTrafficShapingHandler 流量整形
- Base64Decoder和Base64Encoder Base64编码解码
### Tcp Server的创建
- 1.创建NioEventLoopGroup的两个实例(也可以只要一个)bossGroup、workerGroup作为Reactor线程池
- 2.反射创建NioServerSocketChannel实例
- 3.设置TCP的backlog参数，指定内核为此套接口排队的最大连接个数。
- 4.对于给定的监听套接口，内核要维护连个队列： 未链接队列、已连接队列，backlog被规定为两个队列总和的最大值，Netty默认为100，Lighttpd为128 × 8，其他一般为5
- 5.指定Handler
    * AbstractBootstrap 客户端新接入的连接Socket对应的ChannelPipeline的Handler
    * ServerBootstrap NioServerSocketChannel对应的ChannelPipeline的Handler

### Decoder使用
#### EchoServer.java
```
public void bind(int port) throws Exception {
	EventLoopGroup bossGroup = new NioEventLoopGroup();
	EventLoopGroup workerGroup = new NioEventLoopGroup();
	try
	{
		ServerBootstrap b = new ServerBootstrap();
		b.group(bossGroup, workerGroup)
			.channel(NioServerSocketChannel.class)
			.option(ChannelOption.SO_BACKLOG, 1024)
			.handler(new LoggingHandler(LogLevel.INFO))
			.childHandler(new ChannelInitializer<SocketChannel>()
		{
			@Override
			protected void initChannel(SocketChannel ch) throws Exception
			{
				ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
				ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, delimiter));
				ch.pipeline().addLast(new StringDecoder());
				ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
					int counter = 0;
					@Override
					public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
					{
						String body = (String) msg;
						System.out.println("This is " + ++counter + " times receive client: [" + body + "]");
						body += "$_";
						ByteBuf echo = Unpooled.copiedBuffer(body.getBytes());
						ctx.writeAndFlush(echo);
					}
				});
			}
		});
	  ChannelFuture f = b.bind(port).sync();
		f.channel().closeFuture().sync();
}
finally
{
	bossGroup.shutdownGracefully();
	workerGroup.shutdownGracefully();
	}
}
```
#### EchoClient.java
```
public void connect(String host, int port) throws Exception {
	EventLoopGroup group = new NioEventLoopGroup();
	try
	{
		Bootstrap b = new Bootstrap();
		b.group(group).channel(NioSocketChannel.class)
			.option(ChannelOption.TCP_NODELAY, true)
			.handler(new ChannelInitializer<SocketChannel>()
			{
				@Override
				protected void initChannel(SocketChannel ch) throws Exception
				{
					ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
					ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, delimiter));
					ch.pipeline().addLast(new StringDecoder());
					ch.pipeline().addLast(new ChannelInboundHandlerAdapter(){
						private int counter;
						private static final String ECHO_REQ = "Hi. welcome to Netty.$_";
						@Override
						public void channelActive(ChannelHandlerContext ctx) throws Exception
						{
								for (int i = 0; i < 10; i++) {
										ByteBuf msg = Unpooled.copiedBuffer(ECHO_REQ.getBytes());
										ctx.writeAndFlush(msg);
								}
						}
						@Override
						public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
						{
								System.out.println("This is " + ++counter + " times receive server: [" + msg + "]");
						}
						@Override
						public void channelReadComplete(ChannelHandlerContext ctx) throws Exception
						{
								ctx.flush();
						}
						@Override
						public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception
						{
								cause.printStackTrace();
								ctx.close();
						}
					});
				}
			});
		ChannelFuture f = b.connect(host, port).sync();
		f.channel().closeFuture().sync();
	}
	finally
	{
		group.shutdownGracefully();
	}
}
```





### protobuf
#### SubscribeReq.proto
```
syntax = "proto2";
package netty;
option java_package = "com.huawei.techtest.netty.protobuf";
option java_outer_classname = "SubscribeReqProto";

message SubscribeReq{
	required int32 subReqId = 1;
	required string userName = 2;
	required string productName = 3;
	required string address = 4;
}
```
#### SubscribeResp.proto
```
syntax = "proto2";
package netty;
option java_package = "com.huawei.techtest.netty.protobuf";
option java_outer_classname = "SubscribeReqProto";

message SubscribeReq{
	required int32 subReqId = 1;
	required string userName = 2;
	required string productName = 3;
	required string address = 4;
}
```
#### SubReqServer.java
```
.childHandler(new ChannelInitializer<SocketChannel>()
{

    @Override
    protected void initChannel(SocketChannel ch) throws Exception
    {
        ChannelPipeline p = ch.pipeline();
        p.addLast(new ProtobufVarint32FrameDecoder());
        p.addLast(new ProtobufDecoder(SubscribeReqProto.SubscribeReq.getDefaultInstance()));
        p.addLast(new ProtobufVarint32LengthFieldPrepender());
        p.addLast(new ProtobufEncoder());
        p.addLast(new ChannelInboundHandlerAdapter(){
            @Override
            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
            {
                SubscribeReqProto.SubscribeReq req = (SubscribeReq) msg;
                System.out.println("Service accept client subscribe req : [" + req.toString() + "]");
                ctx.writeAndFlush(resp(req.getSubReqId()));
            }

            private SubscribeRespProto.SubscribeResp resp(int subReqId) {
							SubscribeRespProto.SubscribeResp.Builder b = SubscribeRespProto.SubscribeResp.newBuilder();
							b.setSubReqId(subReqId);
							b.setRespCode(0);
							b.setDesc("Netty book order succeed!");
							return b.build();
            }
        });
```
#### SubReqClient.java
```
.handler(new ChannelInitializer<SocketChannel>()
{
  @Override
  protected void initChannel(SocketChannel ch) throws Exception
  {
    ChannelPipeline p = ch.pipeline();
    p.addLast(new ProtobufVarint32FrameDecoder());
    p.addLast(new ProtobufDecoder(SubscribeRespProto.SubscribeResp.getDefaultInstance()));
    p.addLast(new ProtobufVarint32LengthFieldPrepender());
    p.addLast(new ProtobufEncoder());
    p.addLast(new ChannelInboundHandlerAdapter(){
        @Override
        public void channelActive(ChannelHandlerContext ctx) throws Exception
        {
					for (int i = 0; i < 10; i++) {
							ctx.write(subReq(i));
					}
					ctx.flush();
        }

        @Override
        public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception
        {
					SubscribeRespProto.SubscribeResp resp = (SubscribeResp) msg;
					System.out.println("Client received response : [" + resp.toString() + "]");
        }

        private SubscribeReqProto.SubscribeReq subReq(int i) {
					SubscribeReqProto.SubscribeReq.Builder b = SubscribeReqProto.SubscribeReq.newBuilder();
					b.setSubReqId(i);
					b.setUserName("Bob");
					b.setProductName("Netty book for protobuf");
					b.setAddress("tianjing");
					return b.build();
        }
```

### ByteBuf
