## 安全模型
- Domain(域)
  - 虚拟机会把所有代码加载到不同的系统域和应用域，系统域部分专门负责与关键资源进行交互，而各个应用域部分则通过系统域的部分代理来对各种需要的资源进行访问
  - 虚拟机中不同的受保护域 (Protected Domain)，对应不一样的权限 (Permission)。存在于不同域中的类文件就具有了当前域的全部权限
- permission类
  - 权限类表示对系统资源的访问（也就是说你有什么权限）。java.security.Permission是个抽象类，被子类继承以表示特定的访问
  
## Java应用程序 身份认证与授权机制
### subject
  - 一种 Java 对象，它表示单个实体，如个人
  - 除了包含一组Principal外，Subject还可以包含两组凭证：公用和专用。credential是密码、密钥和令牌等。
### principal
  - 一个Subject可以有许多个相关身份，每个身份都由一个Principal对象表示
  - Principal不是持久性的，所以每次用户登录时都必须将它们添加到Subject


## Spring Security的核心类
- AuthenticationManager
  - ProviderManager为AUthenticationManager的实现了，内部维护List<AuthenticationProvider>，只要有一个通过认证即可
- SecurityContextHolder 身份信息的存放容器(存放时去掉密码，保留是否验证通过)
- AuthenticationProvider　每个AuthenticationProvider代表一种认证方式，比如邮箱加密码认证
    - DaoAuthenticationProvider getCredentials() 用户提交的密码凭证
- Authentication
  - Authentication中的getAuthorities()实际是由UserDetails的getAuthorities()传递而形成的
- UserDetail 代表最详细的用户信息
   - getPassword() 户正确的密码
  - UserDetailService只负责从特定的地方加载用户信息
  
