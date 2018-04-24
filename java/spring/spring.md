## 框架设计原则
- 使用POJO进行轻量级和最小侵入式开发
- 通过依赖注入和基于接口编程实现松耦合
- 通过AOP和默认习惯进行声明式编程
- 使用AOP和模板(template)减少模式化代码

## 注解
### 声明Bean
- @Component
- @Service
- @Repository
- @Controller
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
