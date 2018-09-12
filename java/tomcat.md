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
  
### 配置文件server.xml
- Server 最外层元素
  - port
  - shutdown
- Listener
  - className：指定实现org.apache.catalina.LifecycleListener接口的类
  - 可嵌在Server Engine Host Context内
- GlobalNamingResources
  - 用于配置JNDI
- Service
  - className：指定实现org.apache.catalina. Service接口的类，默认值为org.apache.catalina.core.StandardService
  - name：Service的名字
- Executor
  - Service提供的线程池，供Service内各组件使用
  - className：指定实现org.apache.catalina. Executor接口的类，默认值为org.apache.catalina.core. StandardThreadExecutor
  - name：线程池的名字
  - daemon：是否为守护线程，默认值为true
  - maxIdleTime(60 000) maxQueueSize(Integer.MAX_VALUE) maxThreads(200) minSpareThreads(25)
  - namePrefix：线程名字的前缀。线程名字通常为namePrefix+ threadNumber
  - prestartminSpareThreads：是否在Executor启动时，就生成minSpareThreads个线程。默认为false
  - threadPriority：Executor内线程的优先级，默认值为5（Thread.NORM_PRIORITY）
  - threadRenewalDelay：重建线程的时间间隔。重建线程池内的线程时，为了避免线程同时重建，每隔threadRenewalDelay（单位：ms）重建一个线程。默认值为1000，设置为负则不重建
- Connector
  - Connector是Tomcat接收请求的入口，每个Connector有自己专属的监听端口。Connector有两种：HTTP Connector和AJP Connector。
  - port：Connector接收请求的端口
  - protocol：Connector使用的协议（HTTP/1.1或AJP/1.3）
  - connectionTimeout：每个请求的最长连接时间（单位：ms）
  - redirectPort：处理http请求时，收到一个SSL传输请求，该SSL传输请求将转移到此端口处理
  - executor：指定线程池，如果没设置executor，可在Connector标签内设置maxThreads（默认200）、minSpareThreads（默认10）
  - acceptCount：Connector请求队列的上限。默认为100。当该Connector的请求队列超过acceptCount时，将拒绝接收请求
- Engine
  - name：Engine的名字
  - defaultHost：指定默认Host。Engine接收来自Connector的请求，然后将请求传递给defaultHost，defaultHost 负责处理请求
  - className：指定实现org.apache.catalina. Engine接口的类，默认值为org.apache.catalina.core. StandardEngine
  -  backgroundProcessorDelay：Engine及其部分子组件（Host、Context）调用backgroundProcessor方法的时间间隔。backgroundProcessorDelay为负，将不调用backgroundProcessor。backgroundProcessorDelay的默认值为10
  - Tomcat启动后，Engine、Host、Context会启动一个后台线程，定期调用backgroundProcessor方法。backgroundProcessor方法主要用于重新加载Web应用程序的类文件和资源、扫描Session过期
  -  jvmRoute：Tomcat集群节点的id。部署Tomcat集群时会用到该属性
- Host
  - name：Host的名字
  - appBase：存放Web项目的目录（绝对路径、相对路径均可）
  - unpackWARs：当appBase下有WAR格式的项目时，是否将其解压（解成目录结构的Web项目）。设成false，则直接从WAR文件运行Web项目
  - autoDeploy：是否开启自动部署。设为true，Tomcat检测到appBase有新添加的Web项目时，会自动将其部署
  - startStopThreads：线程池内的线程数量。Tomcat启动时，Host提供一个线程池，用于部署Web项目，startStopThreads为0，并行线程数=系统CPU核数；startStopThreads为负数，并行线程数=系统CPU核数+startStopThreads，如果（系统CPU核数+startStopThreads）小于1，并行线程数设为1；startStopThreads为正数，并行线程数= startStopThreads，startStopThreads默认值为1
  - startStopThreads为默认值时，Host只提供一个线程，用于部署Host下的所有Web项目。如果Host下的Web项目较多，由于只有一个线程负责部署这些项目，因此这些项目将依次部署，最终导致Tomcat的启动时间较长。此时，修改startStopThreads值，增加Host部署Web项目的并行线程数，可降低Tomcat的启动时间
- Context
  - path：该Web项目的URL入口。path设置为””，输入`http://localhost:8080`即可访问MyApp；path设置为”/test/MyApp”，输入`http://localhost:8080/test/MyApp`才能访问MyApp
  - docBase：Web项目的路径，绝对路径、相对路径均可（相对路径是相对于CATALINA_HOME\webapps）
  - reloadable：设置为true，Tomcat会自动监控Web项目的/WEB-INF/classes/和/WEB-INF/lib变化，当检测到变化时，会重新部署Web项目。reloadable默认值为false。通常项目开发过程中设为true，项目发布的则设为false
  - crossContext：设置为true，该Web项目的Session信息可以共享给同一host下的其他Web项目。默认为false
- Realm
  - Realm可以理解为包含用户、密码、角色的”数据库”。Tomcat定义了多种Realm实现：JDBC Database Realm、DataSource Database Realm、JNDI Directory Realm、UserDatabase Realm等
- Valve
  - Valve可以理解为Tomcat的拦截器，而我们常用filter为项目内的拦截器。Valve可以用于Tomcat的日志、权限等。Valve可嵌在Engine、Host、Context内

### 类加载机制
- 启动时，会创建几种类加载器
  - Bootstrap加载器，加载JVM启动所需类和标准扩展类(jre/lib/ext)
  - System类加载器，加载tomcat启动的类，比如bootstrap.jar，通常在catalina.bat catalina.sh中指定，位于CATALINA_HOME/bin下
  - Common类加载器，加载tomcat依赖的类，位于CATANALIA_HOME/lib下，比如servlet\-api.jar
  - webapp应用类加载器：每个应用部署后，都会创建一个唯一的类加载器，加载/WEB\-INF/lib下的jar和WEB\-INF/classes下的类文件
