### 功能
- 远程服务调用
- SOA服务治理

### 通信协议
- Dubbo协议(默认)
- Hessian
- HTTP
- RMI
- WebService
- Thrift
- Memcached
- Redis

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
