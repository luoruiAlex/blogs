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
#### 配置类(@Configuration)中声明
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

### 多线程
- 1.配置类中增加@EnableAysnc开启对异步任务的支持
- 2.配置类中实现AsyncConfigurer接口并重新getAsyncExecutor方法，返回一个ThreadPoolExecutor
- 3.@Async注解表示方法为异步方法，如果在类级别，则类的方法全部为异步。@Async注解后会自动被注入ThreadPoolExecutor，执行异步方法时会异步执行，而不是顺序执行

### 计划任务
- 1.配置类中增加@EnableScheduling
- 2.方法上增加@Scheduled注解
  - fixedRate = long
  - fixedDelay = long
  - cron = String
  
### 条件注解
- 1.定义判断不同的判断条件XxCondition implements Condition，覆盖matches方法
- 2.定义同一个接口的不同实现类
- 3.@Bean下增加不同的Condition注解，返回不同的实现类

### 组合注解
- 自定义注解，可组合多个元注解

### `@Enable*`注解的工作原理
- `@Enable*`都有一个`@Import`注解用于导入配置类
- 方式1：直接导入配置类`@Import(XxConfiguration.class)`
- 方式2：根据条件选择配置类
- 方式3：动态注册Bean(ImportBeanDefinitionRegister接口可在运行时自动添加Bean到已有配置类)

### 测试
- Spring TestContext Framework: `org.springframework.spring-test.${version}`
- 1.@RunWith(SpringJUnit4ClassRunner.class) 在JUit环境下提供Spring TestContext Framework的功能
- 2.@ContextConfiguration(classes={XxConfig.class})加载ApplicationContext和配置类
- 3.@ActiveProfiles("dev")声明活动的Bean
- 4.@Autowired注入Bean
- 5.@Test等测试代码
