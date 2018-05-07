## 框架设计原则
- 使用POJO进行轻量级和最小侵入式开发
- 通过依赖注入和基于接口编程实现松耦合
- 通过AOP和默认习惯进行声明式编程
- 使用AOP和模板(template)减少模式化代码

## 注解
### 声明Bean
#### 类路径自动扫描
- @Component
- @Service
- @Repository
- @Controller
#### 配置类(@Configration)中声明
- @Bean
  - 作用在方法上，在方法中new对象
### 注入Bean
- @Autowired
  - 按Type, 默认require为true
  - 可与@Qualifier注解一起用(按name)，@Qualifier可以作用在成员变量、方法入参、构造函数入参
  - @Autowired可以作用在变量、setter方法、构造函数上
- @Inject
  - 按Type
  - 可与@Named一起用，变为按name注入
- @Resource
  - JSR注解
  - 默认按name，没有则用type，可同时指定name和type
  - 可以作用在变量、setter方法
### Java配置
- @Configration 相当于XML中的`<beans>`
- @Bean 相当于XML中的`<bean>`
- @Bean作用的方法的代码中实现Bean注入
- @Value注入资源
  - @Value("普通字符串")
  - @Value("#{systemProperties['os.name']}") 操作系统属性
  - @Value("#{ T(java.lang.Math).random() * 100.0 }") 表达式结果
  - @Value("#{demoService.another}") 其他Bean属性
  - @Value("${book.name}") 配置文件，需要在类上加@PropertySource("xx.properties")指定配置文件

### Bean实例化配置
- @Lazy(true)
- @Primary 多个Bean候选者时用@Primary，否则报错
- 初始化、销毁
  - @Bean的initMethod和destroyMethod属性指定方法
  - @PostConstruct和@PreDestroy注解
- @Scope
  - Singleton
  - Prototype
  - Request
  - Session
  - GlobalSession 只在portal应用中有用

### Profile
- 在@Bean的同时增加@Profile("xx")
- 激活
  - jvm的spring.profiles.active参数
  - Servlet的context parameter(web.xml中或WebApplicationInitializer.onStartup()方法中)

