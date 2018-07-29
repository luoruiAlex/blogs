## 配置
- 依赖spring-security-web spring-security-config
- springSecurityFilterChain: DelegatingFilterProxy
- applicationContext-security.xml
  - xmlns:security
  - `<sec:http pattern="/login.jsp" security="none"/>`
  - ```<!-- auto-config='true'将自动配置几种常用的权限控制机制，包括form, anonymous, rememberMe -->
        <sec:http auto-config="true" >
            
            <!--自定义登录页面,该标签如果不设置,Spring Security会自动给你添加一个简单的登录验证界面  -->
            <!-- authentication-failure-url为验证失败跳转的页面 -->
            <!-- default-target-url 验证成功跳转的页面 -->
          <sec:form-login login-page="/login.jsp" authentication-failure-url="/login.jsp" default-target-url="/index.jsp"/>      
          <!-- 该页面需要的权限为ROLE_user,ROLE_是固定的模式 -->
          <sec:intercept-url pattern="/index.jsp" access="ROLE_user"/>
          <!-- 无访问权限后跳转的页面 -->
          <sec:access-denied-handler error-page="/error.jsp"/>
        </sec:http>
  ```
  - ```<sec:authentication-manager>
          <sec:authentication-provider>
          <!-- 配置用户以及相关的权限 -->
          <sec:user-service>
          <sec:user name="TOM" authorities="ROLE_user" password="TOM"/>
          <sec:user name="jack" authorities="ROLE_Manger" password="jack"/>
          </sec:user-service>
          </sec:authentication-provider>
       </sec:authentication-manager>
     ```
  
