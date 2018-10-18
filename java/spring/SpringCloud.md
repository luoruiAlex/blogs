- 作用：分布式集成框架

### 服务注册与发现
- Eureka Server：保证了CAP中的AP
  - 失效剔除：默认每隔60s将清单中的超时(默认90s)未续约的服务剔除
  - 自我保护：运行期间统计心跳失败的比例在15分支内是否低于85%。SERVER回家当前的**实例注册信息**保护起来，让这些实例不会过期，尽可能**保护这些注册实例**
```
register-with-eureka: false 不注册自己
fetch-registry: false       自己就是注册中心，不检索服务
```
- 可在生成环境加入安全校验
- Eureka Client
  - Provider
    - 服务注册：通过REST请求将自己注册到Eureka Server
    - 服务续约：维护心跳
    - 服务下线：Provider进行正常关闭，触发REST请求给Eureka Server
  - Consumer：单纯的Consumer无需对外提供服务，无需注册到Eureka
    - 获取服务：启动时发送REST请求获取服务清单
    - 服务调用：通过**服务名**获取实例信息，优先调用同一个Zone中的Provider
  ```
  eureka:
    client:
      register-with-eureka: false consumer无需注册
      service-url:
        defaultZone: http://eureka1.com:7001/eureka/,http://eureka1.com:7002/eureka/
  ```
 
 ### 服务调用
 - 可用RestTemplate
 - 负载均衡
   - 客户端的负载均衡：Ribbon
   - 服务端的负载均衡：Nginx
 - Ribbon
  - 默认均衡策略为轮询
  - 自定义：继承AbstractLoadBalancerRule，覆盖public Server choose(ILoadBalancer lb, Object key)即可
   
### 服务保护
- 在**高并发**的情况下，由于单个服务的延迟，可能导致**所有的请求都处于延迟状态**，直至雪崩
- 解决方案
  - Fallback(失败快速返回)：熔断机制，避免故障在分布式系统中蔓延
  - 资源/依赖隔离：**为每一个依赖服务创建一个独立的线程池**
- Hystrix：
  - 熔断关键参数：滑动窗口大小(2)，熔断器开关间隔(5s)，错误率(50%)
    - 每当20个请求中，有50%失败时，熔断器就会打开，此时再调用此服务，将会直接返回失败，不再调远程服务。
    - 直到5s钟之后，重新检测该触发条件，判断是否把熔断器关闭，或者继续打开
  - Hystrix仪表盘
    - 实时监控Hystrix的各项指标信息
    - `http://localhost:8001/hystrix.stream`监控单个服务
    - `http://localhost:8001/trubine.stream`监控集群的汇总信息
### Feign
- Ribbon和Hystrix几乎是同时出现，为了简化开发，合并成Spring Cloud Feign
- 此外，Feign还提供了**声明式的服务调用**(不再通过RestTemplate)

### Zuul
- 问题：
  - 路由规则与服务实例的维护间题：**外层的负载均衡(nginx)需要维护所有的服务实例清单**
  - 签名校验、 登录校验冗余问题：为了保证对外服务的安全性， 我们在服务端实现的微服务接口，往往都会有一定的权限校验机制，但我们的服务是独立的，我们不得不在这些应用中都实现这样一套校验逻辑，这就会造成**校验逻辑的冗余**。
- Zuul：
  - SpringCloud Zuul通过与SpringCloud Eureka进行整合，将自身注册为Eureka服务治理下的应用，同时从Eureka中获得了所有其他微服务的实例信息。**外层调用都必须通过API网关，使得将维护服务实例的工作交给了服务治理框架自动完成**。
  - **在API网关服务上进行统一调用来对微服务接口做前置过滤**，以实现对微服务接口的拦截和校验
- Zuul天生就拥有线程隔离和断路器的自我保护功能，以及对服务调用的客户端负载均衡功能。也就是说：Zuul也是支持Hystrix和Ribbon
- zuul做最外层请求的负载均衡 ，而Ribbon和Fegin做的是系统内部各个微服务的service的调用的负载均衡

### SpringCloud Config
- Server：提供配置文件的存储、以接口的形式将配置文件的内容提供出去
  - 配置仓库默认为git，也可配置SVN
- Client：通过接口获取数据、并依据此数据初始化自己的应用
- 其他
  - 配置文件内的信息**加密和解密**
  - 修改了配置文件，希望**不用重启来动态刷新配置**，配合Spring Cloud Bus 使用
