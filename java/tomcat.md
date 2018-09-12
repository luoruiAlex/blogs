### 结构体系
- Server = m \* Service
  - Server的端口8005，shutdown="SHUTDOWN"表示在8005端口监听SHUTDOWN命令，收到了就关闭Tomcat
- Service = m \* Connector \+ Container
  - Connector用于处理连接相关的事情，并提供Socket与Request和Response相关的转化。可提供不同协议以及同协议不同端口的连接
  - Container用于封装和管理Servlet，以及处理Request请求
