## 核心原理
- 事件驱动 ChannelHandler
- Zero-Copy-Capable Rich Byte Buffer 数据传输时，需要对传输层的保温进行组合或拆分，NIO原生的ByteBuffer需要拷贝内容然后产生新的ByteBuffer，而Netty提供了Composite和Slice来实现零拷贝
- Universal Communication API 对Java的BIO和NIO提供了统一的API

## buffer
- 完全重新实现的，内部有readerIndex和writerIndex，可同时读写，不需要flip()来切换读写模式。
- NIO中的buffer仅仅是传输上的Buffer，而Netty中的buffer是传输buffer和抽象后的逻辑buffer的结合。
- NIO仅仅是一个网络传输框架，而Netty是一个网络应用框架，包括网络以及应用的分层结构
- BigEndianHeapChannelBuffer和LittleEndianHeapChannelBuffer用于不同的网络协议
- DynamicChannelBuffer可根据内容的长度来扩容
- CompositeChannelBuffer由多个ChannelBuffer组合而成，保留了所有子ChannelBuffer的引用，在子ChannelBuffer中读写，从而实现Zero-Copy-Capable
- ByteBufferBackedChannelBuffer封装了NIO的DirectByteBuffer

## Channel
- 一个Channel包含一个ChannelPipeline，所有ChannelHandler都会注册到ChannelPipeline中，并按顺序组织起来
- ChannelEvent是数据或者状态的载体，比如MessageEvent、ChannelStateEvent。当对Channel进行操作时，会产生一个ChannelEvent，并发送到ChannelPipeline。ChannelPipeline会选择一个ChannelHandler进行处理。这个ChannelHandler处理之后，可能会产生新的ChannelEvent，并流转到下一个ChannelHandler。
## ChannelPipeline
- ChannelPipeline包含两条线路：Upstream(接收到的消息、被动的状态改变)和Downstream(发送的消息、主动的状态改变)
## ChannelHandlerContext
- 一个DefaultChannelHandlerContext内部，除了包含一个ChannelHandler，还保存了”next”和”prev”两个指针，从而形成一个双向链表
