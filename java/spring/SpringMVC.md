## 常用注解
- @Controller
- @RequestMapping(value="/xx", produces = "application/json;charset=UTF-8")
- @ResponseBody 将返回值放在response体内，而不是返回一个页面
- @RestController = @Controller + @ResponseBody
- @RequestBody 运行request参数在request体中，而不是直接在链接地址后面
- @PathVariable 放在方法参数前接收路径参数

## 基本配置
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
  - 1.XxInterceptor extends HandlerInterceptorAdapter(或实现HandlerInterceptor接口)，重写preHandle()和postHandle()方法
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
