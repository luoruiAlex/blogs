## 方案对比
- BIO 同步阻塞，一个连接一个线程
- NIO 同步非阻塞，一个请求一个线程。
    - 基于Reactor的事件驱动思想，当一个连接建立后，不需要对应一个线程。连接注册到多路复用器上，所有连接用一个线程搞定。多路复用器轮询时发现连接上有请求时才开启线程处理
- AIO(NIO.2) 异步非阻塞，一个有效请求一个线程。
    - 基于Proactor模型思想，数据准备好了才会通知使用者。
    
## 事件分离器
- 分离IO请求与读写操作
- 1、 Reactor
    - 同步
    - 应用程序在分离器注册'读/写'就绪事件处理器
    - 分离器监听到事件后通知事件处理器进行读写操作
- 2、 Proactor
    - 异步
    - 应用程序在分离器注册'读/写'完成事件处理器
    - 操作系统读写后通知分离器

## NIO
### 1. Channel
- 双向、可异步、总是经过Buffer
- FileChannel
    - `RandomAccessFile.getChannel()` `FileInputStream.getChannel()`
- SocketChannel
    - `SocketChannel.open()`
    - `configureBlocking(false)` `read()` `write()`
- ServerSocketChannel
    - `SelectionKey.channel()`
- DatagramChannel
    - `DatagramChannel.open`
    - `receive()` `send()`

### 2. Buffer
- ByteBuffer MappedByteBuffer HeapByteBuffer DirectByteBuffer
- ShortBuffer IntBuffer LongBuffer CharBuffer FloatBuffer DoubleBuffer
- 写 channel.read(buf) buf.put()
- 读 channel.write(buf) buf.get()
- 属性 capacity position limit mark(mark() reset())
- 写模式->读模式 flip(); rewind(): position->0; clear(): position->0 limit->capacity; compact()

### 3. Selector
- 单线程处理多个channels
- Selector.open() channel.register(selector, selectionkey)
- SocketChannel/ServerSocketChannel有4种类型的事件： connect/accept read/write

## Netty
- 可靠性能力补齐
- 定制能力强
- 解决了已发现的所有NIO的BUG，比如`epoll bug`
