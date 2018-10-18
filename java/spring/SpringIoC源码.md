[https://blog.csdn.net/linuu/article/details/50829531]
### 加载过程
- 从xml读取Resource
- 解析成BeanDefinition
- BeanDefinition注册到BeanFactory

### DefaultListableBeanFactory
- Bean存储的地方：持有bednDefinitionMap(ConcurrentHashMap)
- 子类AbstractRefreshableApplicationContext持有DefaultListableBeanFactory的引用
- 继承BeanDefinitionRegistry，注册、取消注册等

### AbstractApplicationContext.refresh()
- 所有的实现类的实例化都会进入这个方法，**最核心的方法**
- 作用是加载Spring容器配置，包括xml property文件和数据库模式等

### 修改Bean的创建
- BeanPostProcessor
  - 允许用户自定义修改创建的Bean实例
  - postProcessBeforeInitialization(Object) 初始化之前处理
  - postProcessAfterInitialization(Object)  初始化之后处理
  - 符合开闭原则
  - 应用举例：开发环境修改python解释器配置
  - 修改的是打印好的文档
- BeanFactoryPostProcessor
  - 允许自定义去修改应用上下文中的beanDefinnition，来适配底层上下文bean的属性值
  - postProcessBeanFactory(ConfigurableListableBeanFactory)
  - 修改的是要打印的work文档
  
### Bean的初始化顺序
- 构造
- BeanNameAware
- BeanFactoryAware
- 执行InitializingBean接口中的afterPropertiesSet的方法
- init-method
- BeanFactoryPostProcessor
  
### BeanFactory和FactoryBean
- FactoryBean是一个接口，getObject()，注册到容器中后通过`&`前缀来区分普通Bean
- beanfactory.getbean()返回的是factoryBean.getObject()中返回的对象

### BeanFactoryAware
- 实现这个接口的bean其实是希望知道自己属于哪一个beanfactory
### BeanNameAware
- 实现该接口的bean会意识到自己在beanfactory的的名字，业务开发不怎么需要

### getBean
- BeanFactory 和 ApplicationContext 加载Bean的区别
  - BeanFactory：第一次使用Bean
  - ApplicationContext
    - singleton lazy-init=false：启动时就加载
    - singleton lazy-init=true： 第一次使用时初始化
    - prototype：第一次使用时初始化
- 都会调用AbstractBeanFactory.doGetBean()
- 转换BeanName，去除前面的&符号(FactoryBean)
- 从缓存获取单例Bean，有缓存的bean则直接返回
- 没有缓存则实例化

### 循环依赖
- AbstractAutowireCapableBeanFactory.doCreateBean()
- 先根据A的无参构造创建Bean，并暴露一个exposedObject用于提前返回暴露的Bean
- 根据B的无参构造Bean，提前暴露Bean
- 给B的bean注入A提前暴露的Bean
- 然后给A的Bean注入B提前暴露的Bean
