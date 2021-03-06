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
- 作用是刷新容器：加载Spring容器配置，包括xml property文件和数据库模式等。
- `prepareRefersh(); ` refresh之前给configLocations数组赋值
- `ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();`
  - `refreshBeanFactory()`
    - 如果已有容器，则销毁里面的bean再关闭容器(保证只有一个容器)
    - 创建一个BeanFactory
    - 设置factory的序列化ID
    - 定制化beanFactory
    - 加载BeanDefinition：创建BeanDefinitionReader(容器作为BeanDefinitionRegistry传入构造)，然后初始化BeanDefinitionReader，加载BeanDefinition(解析xml等配置文件，然后注册到容器中)
  - `ConfigurableListableBeanFactory beanFactory = getBeanFactory();`返回上一步创建的容器
- `prepareBeanFactory();`准备beanfactory来使用这个上下文.做一些准备工作，例如classloader，beanfactoryPostProcessor等
- 执行prostProcessor、注册BeanPostProcessor、初始化消息源、注册监听器等ApplicationContext的工作，首先就是invokeBeanFactoryPostProcessors(该接口直接修改beanDefinition，所以优先初始化)
- 实例化所有非懒加载的bean
- `finishRefresh()`发布所有的应用

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
- BeanFactoryPostProcessor
- 构造
- BeanNameAware
- BeanFactoryAware
- 执行InitializingBean接口中的afterPropertiesSet的方法
- init-method
- 以上都是在getBean()方法中执行

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
- 没有缓存则尝试从父Factory中获取
- 还没有则实例化
  - 首先标记目前的bean为创建中
  - 根据beanName从beanDefinitionMap获取BeanDefinition，然后获取依赖的Bean
  - 如果依赖的bean还没有创建，则先创建依赖的bean，递归调用
  - 创建Bean，执行aware方法(BeanNameAware, BeanFactoryAware)，初始化(执行InitializingBean接口中的afterPropertiesSet的方法, init-method)
  
### BeanFactory和FactoryBean
- FactoryBean是一个接口，getObject()，注册到容器中后通过`&`前缀来区分普通Bean
- beanfactory.getbean()返回的是factoryBean.getObject()中返回的对象

### BeanFactoryAware
- 实现这个接口的bean其实是希望知道自己属于哪一个beanfactory
### BeanNameAware
- 实现该接口的bean会意识到自己在beanfactory的的名字，业务开发不怎么需要


### 循环依赖
- AbstractAutowireCapableBeanFactory.doCreateBean()
- 先根据A的无参构造创建Bean，并暴露一个exposedObject用于提前返回暴露的Bean
- 根据B的无参构造Bean，提前暴露Bean
- 给B的bean注入A提前暴露的Bean
- 然后给A的Bean注入B提前暴露的Bean
