## BeanFactory继承体系
- 0.BeanFactory
  - 查找是否包含Bean，取得单个Bean，Bean是Singleton还是Prototype的等操作
- 1.1HierarchicalBeanFactory
  - containsLocalBean(String name)，getParentBeanFactory()
  - ConfigurableBeanFactory
    - 可以添加一些自定义配置
    - setParentBeanFactory(BeanFactory parentBeanFactory) addBeanPostProcessor()
- 1.2ListableBeanFactory
  - 取得所有Bean，和所有BeanName，以及所有BeanDefinition的方法
  - containsBeanDefinition， getBeanDefinitionNames， getBeanDefinitionCount等
  - getBeanNamesForAnnotation，getBeanNamesForType
  - getBeansOfType，getBeansWithAnnotation
- 2.1ConfigurableListableBeanFactory
- AutowireCapableBeanFactory
  - 在BeanFactory基础上实现对已存在实例的管理，可以使用这个接口集成其它框架,捆绑并填充并不由Spring管理生命周期并已存在的实例
  - createBean configureBean destroyBean
  - AbstractAutowireCapableBeanFactory
- 3.DefaultListableBeanFactory
  - 最关键的类
  - ApplicationContext接口的实现类(AbstractRefreshableApplicationContext)，也是通过持有DefaultListableBeanFactory来实现IoC容器的基本功能
- 4.XmlBeanFactory
  - 持有XmlBeanDefinitionReader，构造时读取xml类型的Resource `this.reader.loadBeanDefinitions(resource);`
  
## ApplicationContext继承体系
- ApplicationEventPublisher
  - 定义了发布事件的接口方法publishEvent
- EnvironmentCapable
  - profile 决定让某一组Bean的定义生效
  - property 提供方便的抽象，应用程序可以方便的访问 system property 环境变量自定义属性等
- MessageSource
  - 主要用于实现设置模式中的策略模式
  - 定义了国际化的主要接口
- ResourceLoader
  - 也是策略模式的接口
  - ResourcePatternResolver，增加把一个路径解析成Resources对象的功能
  
## IOC容器初始化
- 构造
  - 1.将Resource路径参数解析成Context需要的路径
  - 2.refresh()
- refresh()启动容器
  - 1.Resource定位(ResourceLoader)
  - 2.BeanDefinition的载入和解析
  - 3.向IoC容器注册BeanDefinition(BeanDefinitionRegistry)
