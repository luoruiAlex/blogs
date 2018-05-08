## 常用注解
- @Controller
- @RequestMapping(value="/xx", produces = "application/json;charset=UTF-8")
- @ResponseBody 将返回值放在response体内，而不是返回一个页面
- @RestController = @Controller + @ResponseBody
- @RequestBody 运行request参数在request体中，而不是直接在链接地址后面
- @PathVariable 放在方法参数前接收路径参数

## 配置文件
- DispatcherServlet，会加载配置文件
  - 必须`<load-on-startup>1</load-on-startup>`
  - contextClass 默认**XmlWebApplicationContext**
  - contextConfigLocation 传递给contextClass的初始化字符串支持逗号分割
  - namespace 默认WEB-INF/`[servlet-name]-serverlet`
- `<context-param>contextConfigLocation`
  - 使用ContextLoaderListener配置时，告诉它spring配置文件applicationContext.xml的位置
  - 一般用于加载除了Web层的Bean，比如DAO、Service，以便与其他Web框架集成
  - contextClass默认为WebApplicationContext，创建完成后会将该context放在ServletContext
- `<listener>ContextLoaderListener`在启动Web容器时，自动装配Spring applicationContext.xml的配置信息
- 配置HandlerMapping HandlerAdapter ViewResolver
- 其他
  - POST中文乱码 CharacterEncodingFilter
  
## 获取上下文
- `WebApplicationContext wctx = WebApplicationContextUtils.getWebApplicationContext(servletContext)`

## Java配置
- 配置类：@EnableWebMvc XxConfig extends WebMvcConfigurerAdapter
- ViewResolver
```
InternalResourceViewResolver r = new InternalResourceViewResolver();
r.setPrefix("/WEB-INF/classes/views/");
r.setSuffix(".jsp");
r.setViewClass(JstlView.class);
return r;
```
- 静态资源映射
  - src/main/resources/assets/xx
  - 覆盖`addResourceHandlers`
  ```
  registry.addResourceHandler("/assets/**").addResourceLocations("classpath:/assets/");
  ```
- 拦截器配置
- 1.XxInterceptor extends HandlerInterceptorAdapter(或实现HandlerInterceptor接口)，重写preHandle()和postHandle()、afterCompletion()方法
- 2.重写WebMvcConfigurerAdapter.addInterceptors
  ```
  registry.addInterceptor(xxInterceptor());
  ```
 
- @ControllerAdvice
  - 把@ControllerAdvice注解内部使用@ExceptionHandler、@InitBinder、@ModelAttribute注解的方法应用到所有的@RequestMapping注解的方法
  - @ExceptionHandler最有用，全局处理Controller中的异常
  - @ModelAttribute注解设置键值对到全局`model.addAttribute(key, value)`，全局中的@RequestMapping都能获得(@ModelAttribute(key))
  - @InitBinder设置WebDataBinder，WebDataBinder用于自动绑定前台请求参数到Model中

- 页面转发，重写addViewControllers()方法
```
registry.addViewController("/index").setViewName("/index");
```

- 路径匹配参数配置
  - 默认忽略"."后面的值
  - 重写configurePathMatch()可不忽略
  ```
  configurer.setUseSuffixPatternMatch(false)
  ```

## 文件上传
- `commons-fileupload.commons-fileupload.${version}`
- `<form action="upload" enctype="multipart/form-data" method="post"><input type="file" name="file"/>`
- 添加转向到upload的ViewController
- 添加MultipartResolver(CommonsMultipartResolver)
- MultipartFile接受上传的文件

## HttpMessageConverter
- extends AbstractHttpMessageConverter
  - readInternal()处理请求的数据
  - writeInternal()处理输出的数据到response
- 重写extendMessageConverters()

## 测试
- 1.RunWith(SpringJUinit4ClassRunner.class)
- 2.ContextConfiguration(classes={xxConfig.class})
- 3.@WebAppConfiguration("src/main/resources")
- 4.@Autowired WebApplicationContext wac;
- 5.private MockMvc mockMvc;@Before this.mockMvc = MockMvcBuilders.webAppContext(this.wac).build()
- 6.可以@Autowired来模拟MockHttpSession和MockHttpServletRequest
- 7.预期
```
mockMvc.perform(get("/normal"))
      .andExpect(status().isOk())预期返回状态
      .andExpect(view().name("page")预期view的名称
      .andExpect(content().contentType("text/plain;charset=UTF-8"))预期返回值的媒体类型
      .andExpect(content().string("xx"))预期返回值的内容
      .andExpect(forwardedUrl("/WEB-INF/classes/view/page.jsp"))预期页面转向的真正路径
      .andExpect(model().attribute("key", expectValue))预期model里的值
```
## 原理
### 请求处理
- 检测是否为multipart，如果是则通过MultipartResolver解析
- 1.request => 前端控制器DispatcherServlet(web.xml中配置)
- 2.DispatcherServlet查询一个或多个处理器映射HandlerMapping
- 3.HandlerMapping将请求映射为HandlerExecutionChain(1个Handler处理器(controller) + 多个HandlerInterceptor)
- 4.将Handler处理器包装成HandlerAdapter
- 304 Not Modified缓存支持
- 拦截器的preHandle()
- 5.HandlerAdapter进行逻辑处理后返回ModelView给DispatcherServlet
- 拦截器的postHandle()
- 6.DispatcherServlet使用视图解析器(view resolver)来将逻辑视图匹配为一个特定的视图实现
- 7.视图使用模型数据渲染输出
- 拦截器的afterCompletion()

### DispatcherServlet初始化
- DispatcherServlet extends FrameworkServlet
- FrameworkServlet extends HttpServletBean implements ApplicationContextAware
- HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware
- 1.HttpServletBean实现HttpServlet.init()方法，将Servlet的初始化参数设置到该组件上，留下initServletBean()扩展点
- 2.FrameworkServlet实现initServletBean()方法
  - 初始化WebApplicationContext
  - 如果ContextLoaderListener加载了上下文，则该上下文作为根上下文
  - 初始化上下文时留下onRefresh()扩展点
  - 留下initFrameworkServlet()给用户扩展
- 3.DispatcherServlet实现onRefresh()，初始化默认的策略，比如HandlerMapping HandlerAdapter等
