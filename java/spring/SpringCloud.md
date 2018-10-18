- 作用：分布式集成框架

### 服务注册中心
- Eureka Server
  - 失效剔除：默认每隔60s将清单中的超时(默认90s)未续约的服务剔除
  - 自我保护：运行期间统计心跳失败的比例在15分支内是否低于85%。SERVER回家当前的**实例注册信息**保护起来，让这些实例不会过期，尽可能**保护这些注册实例**
```
register-with-eureka: false 不注册自己
fetch-registry: false       自己就是注册中心
```
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
   
