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
- 所有的实现类的实例化都会进入这个方法
- 作用是加载Spring容器配置，包括xml property文件和数据库模式等

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
