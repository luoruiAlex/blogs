# 特点
  - 支持序列化和多种类型的RPC
  - 数据结构变化时，必须重新编辑IDL文件生成代码和编译
  - 可定义字段顺序，支持协议向前兼容
  - 支持8种Java基本类型、Map、Set、List
  - 支持可选和必选定义
# 组成
  - 语言系统及IDL编译器
  - TProtocol RPC的协议层，可选择多种不同序列化方式，比如JSON和Binary（重点关注）
  - TTransport RPC的传输层，可选择不同传输层实现，如socket、NIO、MemoryBuffer
  - TProcessor 协议层和服务实现之间的纽带，负责调用服务实现的接口
  - TServer 聚合TProtocol、TTransport和TProcessor等对象
