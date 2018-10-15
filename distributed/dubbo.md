### 功能
- 远程服务调用
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
- dubbo:monitor：监控中心配置，对应MonitorConfig
  - protocol：监控中心协议，默认为dubbo，如果为protocol="registry"，表示从注册中心发现监控中心地址，否则直连监控中心。
  - address：直连监控中心服务器地址，比如address="10.20.130.230:12080"
- dubbo:module：模块信息配置，对应ModuleConfig
  - name：当前模块名称，用于注册中心计算模块间依赖关系

