[https://blog.csdn.net/linuu/article/details/50829531]

### DefaultListableBeanFactory
- Bean存储的地方：持有bednDefinitionMap(ConcurrentHashMap)
- 子类AbstractRefreshableApplicationContext持有DefaultListableBeanFactory的引用
- 继承BeanDefinitionRegistry，注册、取消注册等
