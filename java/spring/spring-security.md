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
        </sec:http>```
  - 
    ```<http auto-config='true'>
        <intercept-url pattern="/admin.jsp" access="hasRole('ROLE_ADMIN')" />
        <intercept-url pattern="/**" access="hasRole('ROLE_USER')" />
    </http>
	<authentication-manager alias="authenticationManager">  
	    <authentication-provider ref="authenticationProvider" />  
	</authentication-manager>  
	<beans:bean id="authenticationProvider"  
	    class="org.springframework.security.authentication.dao.DaoAuthenticationProvider">  
	    <beans:property name="userDetailsService" ref="myUserDetailsService" />  
	    <beans:property name="hideUserNotFoundExceptions" value="false" />   
	</beans:bean>```
  
