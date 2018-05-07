## 常用注解
- @Controller
- @RequestMapping(value="/xx", produces = "application/json;charset=UTF-8")
- @ResponseBody 将返回值放在response体内，而不是返回一个页面
- @RestController = @Controller + @ResponseBody
- @RequestBody 运行request参数在request体中，而不是直接在链接地址后面
- @PathVariable 放在方法参数前接收路径参数

## 基本配置
- 配置类：@EnableWebMvc XxConfig extends WebMvcConfigurerAdapter
- 静态资源映射
  - src/main/resources/assets/xx
  - 覆盖`addResourceHandlers`
  ```
  registry.addResourceHandler("/assets/**").addResourceLocations("classpath:/assets/");
  ```
- 拦截器配置
  - 1.XxInterceptor extends HandlerInterceptorAdapter(或实现HandlerInterceptor接口)，重写preHandle()和postHandle()方法
  - 2.重写WebMvcConfigurerAdapter.addInterceptors
  ```
  registry.addInterceptor(xxInterceptor());
  ```
- @ControllerAdvice
  
