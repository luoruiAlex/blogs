## 核心功能
- 独立运行的Spring项目
- 内嵌Servlet容器
- 提供starter简化Maven配置
- 自动配置Spring
- 准生产的应用监控
- 无代码生成和XML配置

## 入口类
- `*Application.class`
- `SpringAppliction.run(*Application.class, args)`

## 注解
- @SpringBootApplication
  - @Configuration
  - @ComponentScan
  - @EnableAutoConfiguration 让SpringBoot根据类路径中的jar包依赖为当前项目进行自动配置
- 关闭某个自动配置 (exclude={xxAutoConfiguration})
  
## 配置
- src/main/resources或类路径的/config下的application.properties application.yaml
- @ImportResource 加载XML配置
- @PropertySource加载properties配置
- application.properites配置后也可用@Value注解
- 类型安全的配置 @Component + @ConfigurationProperties(prefix = "xx")，application.properites中配置xx.keys
- 命令行参数 `java -jar xx.jar --server.port=9090`

## 日志
- 默认为logback
- logging.file = xx.log
- logging.level.packageName = LEVEL

## profiles
- application-dev/prod.properites
- application.properites中`spring.profiles.active=dev`

## 自动配置原理
- 关键在于@EnableAutoConfiguration
- @Import

## SpringMvc
- 页面放到src/main/resources/下，运行时编译到/WEB-INF/classes/vies/下
### 自动配置的ViewResolver
- ContentNegotiatingViewResolver，代理给不同的ViewResolver来处理不同的View，具有最高优先级
- BeanNameViewResolver
- InternalResourceViewResolver
### 自动配置的静态资源(在addResourceHandlers方法中定义了自动配置)
- 类路径下/static、/public、/resources、/META-INF/resources下的静态文件映射为/**
- /META-INF/resources/webjars下的静态文件映射为/webjar/**
### 自动配置的Formatter和Converter
- 只要定义了Converter、GenericConverter和Formatter接口的实现类Bean，这些Bean就会自动注册到SpringMVC中
### 自动配置的HttpMessageConverters
- 默认注册了一些HttpMessageConverters的Bean
- 新增自定义的HttpMessageConverter的Bean可自动注册
### 静态首页
- classpath:/META-INF/resources/index.html
- classpath:/resources/index.html
- classpath:/static/index.html
- classpath:/public/index.html
### 接管Spring Boot的Web配置
- `@Configuration xxConfig extends WebMvcConfigurerAdapter{}`
- 推荐用addXx()方法，不会覆盖自动配置，比如`addViewControllers()`
### 注册Servlet、Filter、Listener
- 方式1：直接注册Bean实例
- 方式2：通过RegistrationBean实例
```
return new ServletRegistrationBean(new XxServlet(), "xx/*");
```
### Servlet容器配置
- 通用
  - server.port=9090
  - server.session-timeout=1800
  - server.context-path=/
- Tomcat
  - server.tomcat.uri-encoding=UTF-8
  - server.tomcat.compression=off
- 替换默认的Tomcat为Jetty
  - `<dependency><exclusions><exclusion><groupId>org.springframework.boot<artifactId>spring-boot-starter-tomcat`
  - `<dependency><groupId>org.springframewokr.boot<artifactId>spring-boot-starter-jetty`
- SSL配置
  - 1.JDK中，`keytool -genkey -alias tomcat -keyalg RSA -keystore C:/.kestore -validity 365`，按提示一步一步生成`.keystore`服务器证书文件
  - 2.证书文件复制到项目根目录，在application.properties中做如下SSL配置
    - server.port=8443
    - server.ssl.key-store=.keystore
    - server.ssl.key-store-passowrd=xxx
    - server.ssl.keyStoreType=JKS
    - server.ssl.keyAlias=tomcat
  - 3.生成客户端证书`keytool -genkey -v -alias clientkey -keyalg RSA -storetype PKCS12 -keystore C:/clientkey.p12`
  - 4.服务器信任客户端证书
  - 5.客户端导入
  - 4.http转向https
  ```
  @Bean
  public EmbeddedServletContainerFactory servletContainer() {
      TomcatEmbeddedServletContainerFactory tomcat = new Tom
  }
  ```
