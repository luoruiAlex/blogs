### 结构体系
- Server = m \* Service
  - Server的端口8005，shutdown="SHUTDOWN"表示在8005端口监听SHUTDOWN命令，收到了就关闭Tomcat
- Service = m \* Connector \+ Container
  - Connector用于处理连接相关的事情，并提供Socket与Request和Response相关的转化。可提供不同协议以及同协议不同端口的连接
  - Container用于封装和管理Servlet，以及处理Request请求

### Connector
- 通过ProtocolHandler来处理请求
- ProtocolHandler包含3个部件
  - Endpoint：处理底层Socket的网络连接
    - 实现**TCP/IP**协议
    - Acceptor用于监听请求
    - Handler用于处理接收到的Socket，在内部调用Processor进行处理
    - AsyncTimout用于检查异步Request的超时
  - Processor：将Endpoint接收到的Socket封装成Request
    - 实现**HTTP等应用层协议**
  - Adapter：将Request交给Container进行具体的处理
    - 将请求适配到Servlet容器

### Container
- Engine：一个Service最多一个Engine，用于管理多个站点
- Host：代表一个站点（也叫虚拟主机）
- Context：代表一个应用程序，对应一个程序或者说是web.xml
- Wrapper：一个Wrapper封装着一个Servlet
```
tomcat_root/
  webapps/      对应一个Host
    docs/       代表一个context
    examples/   代表一个context
    host-manager/
    manager/
    ROOT/
```
- webapps代表一个host站点，webapps下每个文件夹都是一个Context，ROOT目录中存放着主应用，其他目录存放着子应用
- 访问应用Context的时候，如果是ROOT下的则直接使用域名就可以访问，其他的必须加文件夹名
- Container使用Pipeline-Value管道来处理请求(Value表示阀门)，与普通的职责链模式有些差别
  - 每个Pipeline都有特定的Value(BaseValue，不可删除)，而且在管道的最后一个执行
  - 上层容器的管道的BaseValue中会调用下层容器的管道
  - Container的四个子容器分别为：EnginePipeline(StandardEngineValue)、HostPipeline(StandardHostValue)、ContextPipeline(StandardContextValue)、WrapperPipeline(StandardWrapperValue)
