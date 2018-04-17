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
