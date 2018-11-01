### 功能
- RPC调用
- SOA服务治理

### 通信协议
- Dubbo协议(默认)
  - 采用异步单一长连接和 NIO 异步通讯
  - 适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况
- Hessian
  - 多连接、短连接、HTTP协议传输、同步、Hessian二进制序列化
  - 适用于传入传出参数数据包比较大、提供者比消费者个数多、提供者压力较大
  - 参数及返回值需实现 Serializable 接口
  - 参数及返回值不能自定义实现 List, Map, Number, Date, Calendar 等接口，只能用 JDK 自带的实现，因为 hessian 会做特殊处理，自定义实现类中的属性值都会丢失
- HTTP
  - 多连接、短连接、同步传输、表单序列化
  - 适用于传入传出参数数据包大小混合，提供者比消费者个数多
- RMI
- WebService
- Thrift
  - 对原生thrift的扩展
  - thrift不支持null值
- Memcached
- Redis
- Rest

### 注册中心
- Multicast
  - 广播地址，相互发现
  - 只适合小规模应用或开发阶段使用，组播地址段: 224.0.0.0 - 239.255.255.255
  - `<dubbo:registry address="mulitcast://224.5.6.7:1234?unicast=false"/>`
  - 为了减少广播量，Dubbo 缺省使用单播发送提供者地址信息给消费者，如果一个机器上同时启了多个消费者进程，消费者需声明 unicast=false，否则只会有一个消费者能收到消息
  - 另外一种配置方式
  ```
  <dubbo:registry protocol="multicast" address="224.5.6.7:1234">
    <dubbo:parameter key="unicast" value="false" />
  </dubbo:registry>
  ```
- Zookeeper
  - 流程
    - 服务提供者启动时: 向`/dubbo/com.foo.BarService/providers`目录下写入自己的 URL 地址
    - 服务消费者启动时: 订阅`/dubbo/com.foo.BarService/providers`目录下的提供者 URL 地址。并向`/dubbo/com.foo.BarService/consumers`目录写入自己的URL地址
    - 监控中心启动时: 订阅`/dubbo/com.foo.BarService`目录下的所有提供者和消费者 URL 地址
  - 功能
    - 注册中心自动删除异常提供者信息
    - 注册中心重启时，自动恢复注册数据和订阅请求
    - 会话过期时能自动恢复注册数据以及订阅请求
  - 配置
    - check="false"：记录失败注册和订阅请求，后台定时重试
    - username="admin password="admin"设置zookeeper的登录信息
    - group="dubbo"设置 zookeeper 的根节点，不设置将使用无根树
    - `<dubbo:reference group="*" version="*" />`订阅服务的所有分组和所有版本的提供者
    - client="curator"：默认客户端为zkclient，从2.3.0开始可选curator
    - 集群 address="10.20.153.10:2181,10.20.153.11:2181,10.20.153.12:2181"
    - 同一 Zookeeper，分成多组注册中心
    ```
    <dubbo:registry id="chinaRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="china" />
    <dubbo:registry id="intlRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="intl" />
    ```
- Simple注册中心
  - Simple 注册中心本身就是一个普通的 Dubbo 服务，可以减少第三方依赖，使整体通讯方式一致。
  - `<dubbo:protocol port="9090" />`奖Simple注册中心暴露成dubbo服务
  - `address="127.0.0.1:9090`
  - 不支持集群，不适用于生产环境

### 配置
- 分类
  - 服务发现
  - 服务治理
  - 性能调优
- Common
  - dubbo:application：应用信息配置，对应ApplicationConfig
    - name：当前应用名称，用于计算应用之间的依赖关系
  - dubbo:registry：注册中心配置，对应RegistryConfig类
    - 可通过多个registry声明多个注册中心，并在service/reference的registry属性中指定使用的注册中心
    - address：注册中心地址url
  - dubbo:method：方法级配置，对应MethodConfig，该标签同时为dubbo:service/dubbo:reference的子标签
  - dubbo:argument：方法参数配置，对应ArgumentConfig，为dubbo:method的子标签
  - dubbo:parameter：选项参数配置。对应的配置类：java.util.Map。
    - 为protocol service provider reference consumer的子标签
  - dubbo:monitor：监控中心配置，对应MonitorConfig
    - protocol：监控中心协议，默认为dubbo，如果为protocol="registry"，表示provider/consumer从注册中心发现监控中心地址，否则直连监控中心。
    - address：直连监控中心服务器地址，比如address="10.20.130.230:12080"
  - dubbo:module：模块信息配置，对应ModuleConfig
    - name：当前模块名称，用于注册中心计算模块间依赖关系
- Provider
  - dubbo:service：暴露服务，对应ServiceConfig类
    - interface：服务接口名
    - ref：服务对象实现引用，即实现类的Bean
  - dubbo:protocol：provider的协议
    - name：默认为dubbo
    - port：dubbo协议缺省端口为20880，rmi协议缺省端口为1099，http和hessian协议缺省端口为80；如果没有配置port，则自动采用默认端口，如果配置为-1，则会分配一个没有被占用的端口。Dubbo 2.4.0+，分配的端口在协议缺省端口的基础上增长，确保端口段可控。
  - dubbo:provider：服务提供者缺省值配置，对应ProviderConfig
- Consumer
  - dubbo:reference：引用服务，对应ReferenceConfig类
    - id：唯一标识
    - interface：服务接口名
  - dubbo:consumer：服务消费者缺省值配置，对应ConsumerConfig
  
### 特性使用
- 服务分组与服务版本号
  - 接口类并不能唯一确定一个服务，`接口 + 服务分组 + 版本号`才能唯一确定一个服务
  - 接口有多种实现时，使用 service/reference 的group属性区分
  - 同一个group的接口实现不兼容时，可使用版本号兼容。使用service/reference的version属性区分
    - 在低压力时间段，先升级一半提供者为新版本
    - 再将所有消费者升级为新版本
    - 然后将剩下的一半提供者升级为新版本
- 服务直连
  - 为了方便开发及测试，绕过注册中心，直接指定服务提供者
  - 方法
    - 通过-D：`-Dcom.test.UserServiceBo=dubbo://30.8.59.182:20880`表示调用com.test.UserServiceBo接口时候访问30.8.59.182:20880提供的服务
    - dubbo:reference的url属性指定，多个地址用分号隔开
    - `-Ddubbo.resolve.file`指定properties文件，优先级比reference高
- 泛化调用
  - 用于consumer没有API接口类和模型类的情况，参数和返回值中的POJO都用Map表示
  - 泛化调用通常用于框架集成，比如：实现一个通用的服务测试框架，可通过 GenericService 调用所有服务实现
  - 代码
  ```
  referenceConfig.setGeneric(true);
  GenericService service = referenceConfig.get();
  service.$invoke('funcName', new String[]{"java.lang.String"}, new Object[]{"arg"});
  ```
- 异步调用
  - 基于 NIO 的非阻塞实现并行调用，客户端不需要启动多线程即可完成并行调用多个远程服务，相对多线程开销较小
  - 代码
  ```
  referenceConfig.setAsync(true);
  ...异步调用方法都直接返回null
  Future<String> future = RpcContext.getContext().getFuture();
  future.get()可得到正确的返回值
  ```
- 结果缓存
  - consumer端缓存：reference配置`cache="lru"`
  - 三种缓存方案
    - LruCacheFactory
    - ThreadLocalCacheFactory
    - JCacheFactory
- 参数校验
  - javax中的校验注解
  - dubbo:service中配置`validation="true"`
- 启动检查
  - 默认在启动时检查依赖的服务是否可用
  - dubbo:reference设置`check="false"`关闭某个服务的启动检查
  - dubbo:consumer设置`check="false"`关闭所有服务的启动检查
  - dubbo:registry设置`check="false"`关闭注册中心的启动检查
- 集群容错
  - failover(默认)：失败自动切换，**当出现失败，重试其他服务器**，通常用于读或者具有幂等的写操作，但重试会带来更长的延迟，可以通过`retries=n`来设置重试次数(不包含第一次)
  - failfast：快速失败，**只发起一次调用，失败就报错**，通常用于新增记录的操作，比如数据同步
  - failsafe：失败安全，**出现异常直接忽略**，也就是对数据的完整性要求不高，通常用于写入审计日志等操作 
  - failback：失败自动恢复，**失败后后台记录失败的请求，定时重发**，通常用于实时性要求不高的通知类操作
  - forking：**并行调用，同时调用多个服务器，只要有一个返回成功即可**，通常用于实时性要求高的查询操作，但是对资源的浪费更大，具体并行几个可以根据`forks=n`设置 
  - broadcast：广播调用所有提供者，**逐个调用，有一台报错就报错**，通常用于更新所有提供者缓存或者日志等本地资源信息，用的机会较少，并且更新失败的话对系统影响很小的资源
- 负载均衡
  - Random(默认)：随机访问策略，按照**权重**设置**随机概率**，但是一般情况下权重都是设置成一样的，一般是访问量越大随机越均匀
    - 权重轮询调度算法考虑到了不同服务器的处理能力，所以这种均衡算法能确保高性能的服务器得到更多的使用率，避免低性能的服务器负载过重
    - 根据权重总数生成一个基准数，大于该数的权重则表示可调用，这样就实现了权重大的服务调用的概览更大
  - RoundRobin：轮询访问策略，按照**公约后的权重**设置**轮询比率**，但是一般权重一样，就是交替访问，但是存在慢的提供者累积请求问题，解决办法为结合权重把性能低的提供者权重值设置低一点
  - LeastActive：最少活跃数访问策略，相同的活跃数是随机访问的，活跃数就是调用前后的计数差，这样的话处理慢的应用就会收到更少的请求，因为越慢调用前后的计数差就越大
    - 即调用并发数最小的provider
  - ConsistentHash：一致性hash，也就是同一个参数的请求总是到同一个应用，当某一个应用挂掉时，原来发往该应用的请求就会基于虚拟节点，平均分配给其他的提供者，不会引起剧烈的变动
    - 缺省只对第一个参数hash、160个虚拟节点，可通过parameter来配置(hash.arguments hash.nodes)
  - 自定义负载均衡策略
    - 实现AbstractLoadBalance抽象类，然后将该类放入dubbo可发现的扩展点即可

### dubbo-monitor
- 统计服务的调用次数和调用时间，这些数据有助于系统运维和调优

### 源码
#### 分层
- Service层(接口及实现类)
- Config层(ReferenceConfig ServiceConfig)：以下层均为SPI
- Proxy层：扩展接口为 ProxyFactory，dubbo实现的SPI主要JavassistProxyFactory（默认使用）和JdkProxyFactory，用来对服务提供方和服务消费方的服务进行代理
- Registry层：扩展接口为 Registry，实现为ZookeeperRegistry，RedisRegistry，MulticastRegistry，DubboRegistry
- Cluster层：路由、负载均衡、桥接注册中心，扩展接口为 Cluster , Directory , Router ,LoadBalance
- Monitor层：扩展接口为 MonitorFactory , Monitor , MonitorService
- Protocol层：封装RPC调用，扩展接口为 Protocol , Invoker , Exporter
- Exchange层：封装请求响应模式，同步转异步，扩展接口为 Exchanger , ExchangeChannel ,ExchangeClient , ExchangeServer
- Transport层：抽象 mina 和 netty 为统一接口扩展接口为 Channel , Transporter , Client , Server , Codec
- Serialize层：扩展接口为 Serialization, ObjectInput, ObjectOutput, ThreadPool
#### Java SPI(service provider interface)
- 设计目标
  - 为接口寻找服务，接口实现类可插拔，类似于IoC
- 约定
  - 接口类com.test.Myservice
  - jar包的`/META-INF/services`目录下，创建该接口的同名文件(全路径名)com.test.Myservice
  - 文件内容为实现类的全路径名，比如`com.test.MyServiceImpl1`，多个实现类则有多行
  - `ServiceLoader<MyService> serviceLoader = ServiceLoader.load(MyService.class)`
  - 遍历`for (MyService service: serviceLoader){...}`
#### Dubbo中的SPI
- Java SPI的缺点：一次性实例化所有实现
- Dubbo SPI的改进
  - 按需加载
  - 增加了对扩展点IOC和AOP的支持，一个扩展点可以直接setter注入其他扩展点
  - 扩展点加载失败有提示
- 约定
  - spi文件存储在`META-INF/dubbo/internal`、`META-INF/services/`、`META-INF/dubbo/`目录下,并且名称为接口的全路径名
  - 文件内容为`扩展名=com.test.MyServiceImpl1`，通过扩展名来加载指定的实现类，避免资源浪费
- 实现
  - 1.获取ExtensionLoader
    - 入口，扩展点载入器
    - 用于载入Dubbo中的各种可配置组件，比如ProxyFactory、LoadBalance、Protocol、Filter、Container、Cluster、Registry：Dubbo中所有组件都是通过SPI方式来管理的
    - `public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type)` 内部为一个ConcurrentHashMap保持不同的接口类型对应的ExtensionLoader
  - 2.获取适配器实例：`ProxyFactory proxy = ExtensionLoader.getExtensionLoader(ProxyFactory.class).getAdaptiveExtension();`
    - 获取该spi对应的所有实现类的class对象，然后创建适配器实例，最后注入该适配器依赖的其他spi
  - 3.根据名称获取具体的spi实现类，创建类实例，使用wrap包装后返回
    - SPI返回的不是单纯的扩展点的实现类，而是被Wrapper后的增强的类
#### 轮询和容错
- RegistryProtocol.doRefer()根据url获取设置的集群获取扩展，默认为FailoverClusterInvoker
- 使用spi获取LoadBalance配置的负载均衡扩展实现
- FailoverClusterInvoker.doInvoke()选择一个invoker
#### Provider暴露服务
- ServiceConfig类拿到接口实现类ref，然后通过ProxyFactory.getInvoker()使ref生成一个AbstractProxyInvoker实例
  - JavassistProxyFactory.getInvoker()
    - 先创建代理类的wrapper类(**消除反射调用以提升性能**)
    - 创建AbstractProxyInvoker实现类，覆盖doInvoke()方法(收到消费请求后调用该方法)，该方法内部委托wrapper类来具体执行
- Invoker通过Protocol.export()转换为Exporter
  - RegistryProtocol.doLocalExport()打开Netty Server侦听服务，接收请求
  - RegistryProtocol.register注册到服务治理中心
  - MulticastRegistry中通过ConcurrentHashMap来保存，值为ConcurrentHashSet
#### Consumer消费服务
- ReferenceConfig.init()调用Protocol.refer()生成Invoker实例
  - 通过增强的wrapper类一步步调用到具体的spi实现类RegistryProtocol
  - 在注册中心订阅服务提供方地址列表，并创建DubboInvoker的装饰类，用于负载均衡和容错处理
- Invoker在proxyFactory.getProxy(invoker, clazz)方法中转为客户端接口
  - JavassistProxyFactory.getProxy()
    - 代理接口，i别难过使用InvokerInvocationHandler对具体调用进行拦截
#### 线程模型
- 使用Netty通讯，用protocol的`dispatcher`和`threadpool`配置
- Provider使用两级线程池(统称为IO线程)：EventLoopGroup(boss)主要用来接受客户端的链接请求，并把接受的请求分发给EventLoopGroup（worker）来处理
- Provider的逻辑如果比较耗时，比如查询数据库，必须派发到新线程池，否则IO线程将阻塞，不能接收请求
- 所有的线程模型(dispatcher)
  - all 所有消息都派发到线程池，包括请求，响应，连接事件，断开事件，心跳等
  - direct 所有消息都不派发到线程池，全部在 IO 线程上直接执行
  - execution 只有**请求消息**派发到线程池，不含响应，响应和其它连接断开事件，心跳等消息，直接在 IO 线程上执行
  - connection 在 IO 线程上，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池
- ThreadPool的spi实现(threadpool)
  - fixed 固定大小线程池，启动时建立线程，不关闭，一直持有。(缺省)
  - cached 缓存线程池，空闲一分钟自动删除，需要时重建。
  - limited 可伸缩线程池，但池中的线程数只会增长不会收缩。只增长不收缩的目的是为了避免收缩时突然来了大流量引起的性能问题。
