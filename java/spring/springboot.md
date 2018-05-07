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
