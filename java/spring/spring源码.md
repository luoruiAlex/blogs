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
  - 4.BeanDefinitionRegistry中的BeanDefinition，使用Java的反射机制自动识别出Bean工厂处理器（实现BeanFactoryPostProcessor接口）的Bean，然后调用这些Bean工厂后处理器对BeanDefinitionRegistry中的BeanDefinition进行加工处理
  - 5.从BeanDefinitionRegistry中取出加工后的BeanDefinition，并调用InstantiationStrategy着手进行Bean实例化工作
  - 6.在实例化Bean时，Spring容器使用BeanWrapper对Bean进行封装，BeanWrapper提供了很多以Java反射机制操作Bean的方法，它将结合该Bean的BeanDefinition以及容器中属性编辑器，完成Bean属性的设置工作
  - 7.利用容器中注册的Bean后处理器（实现BeanPostProcessor接口的Bean）对已经完成属性设置工作的Bean进行后续加工，直接装配出一个准备就绪的Bean

## 依赖注入
- getBean() 启动
- AbstractBeanFactory.doGetBean() 依次从缓存->parentBeanFactory->createBean获取Bean
- AbstractAutoWireCapableBeanFactory.createBean()
  - 如果配置了postProcessor，返回proxy
  - 否则调用子方法doCreateBean()
- AbstractAutoWireCapableBeanFactory.doCreateBean()
  - createBeanInstance()
    - 使用构造或autowireConstructor(构造函数的参数使用了Autowire)
    - 使用instantiateBean()方法实例化
  - populateBean() 通过BeanDefinition注入属性
    - applyPropertyValues()
    - resolveValueIfNecessary()，包括resolveReference()，根据value的类型，进行不同的解析
    - setPropertyValue()
  - initializeBean
  
## 循环依赖的解决
- 依次从三层缓存 singletonObjects earlySingletonObjects singletonFactories(因为有些Bean是需要被代理的，总不能把代理前的暴露出去那就毫无意义了) 中获取
- doCreateBean()将类A曝光到singletonFactories中
- 类B从singletonFactories中获取到类A的一个引用，将自己放到 singletonObjects 中
- 类A获取到B对象，填充属性
