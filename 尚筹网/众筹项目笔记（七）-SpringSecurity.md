# SpringSecurity

## 一、SpringSecurity简介

SpringSecurity可以做的事情：

* 用户认证：指用户在登录时校验用户名和密码的过程。
* 用户授权：指用户是否有访问某个资源的权限。



## 二、环境搭建

pom.xml

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>4.3.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-orm</artifactId>
    <version>4.3.20.RELEASE</version>
</dependency>
```

spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="cn.yylm.security"/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <mvc:annotation-driven/>
    <mvc:default-servlet-handler/>
</beans>
```

web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         version="2.5">
  <display-name>spring-security-01</display-name>
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>

  <!-- The front controller of this Spring Web application, responsible for
      handling all application requests -->
  <servlet>
    <servlet-name>springDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <!-- Map all requests to the DispatcherServlet for handling -->
  <servlet-mapping>
    <servlet-name>springDispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

导入资源

![image-20200628114729452](http://picture.youyouluming.cn/image-20200628114729452.png)

![image-20200628114744192](http://picture.youyouluming.cn/image-20200628211155241.png)

## 三、加入SpringSecurity环境

### 1、加入SpringSecurity依赖

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>4.2.10.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>4.2.10.RELEASE</version>
</dependency>
<!-- 标签库 -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
    <version>4.2.10.RELEASE</version>
</dependency>
```

### 2、加入SpringSecurity控制权限的Fliter

SpringSecurity使用的是过滤器而不是拦截器InternalResourceViewResolver，所以SpringSecurity不仅可以处理handler中的请求，还可以处理所有的web请求，包括静态资源。

web.xml

```xml
<filter>
  <filter-name>springSecurityFilterChain</filter-name><!--名称固定,不能变 -->
  <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
  <filter-name>springSecurityFilterChain</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

注意：SpringSecurity会根据DelegatingFilterProxy的filter-name到IOC容器查找所需要的bean。所以过滤器名字必须是springSecurityFilterChain

### 3、加入配置类

WebAppSecurityConfig.java

```java
@Configuration      //当前类为配置类
@EnableWebSecurity  //启用web下的权限控制功能
public class WebAppSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder builder) throws Exception {
     	//SpringSecurity环境下与用户登录相关
    }
 	@Override
    protected void configure(HttpSecurity security) throws Exception {
        //SpringSecurity环境下请求授权相关
    }
}
```

配置完成后，所有的资源都被拦截，必须登录后才可以访问。

![image-20200628132245664](http://picture.youyouluming.cn/image-20200628114744192.png)

## 四、SpringSecurity实验

### 1、放行静态资源和首页

WebAppSecurityConfig.java

```Java
@Configuration      //当前类为配置类
@EnableWebSecurity  //启用web下的权限控制功能
public class WebAppSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity security) throws Exception {

        security
                .authorizeRequests()                        //给请求授权
                .antMatchers("/index.jsp")  //跳转页面
                .permitAll()                               //不需要权限 无条件访问
                .antMatchers("/layui/**")   //对layui资源授权
                .permitAll()
                .and()
                .authorizeRequests()                      //给请求授权
                .anyRequest()                             //任意请求
                .authenticated();                         //登录用户才可以访问
    }
}
```

### 2、指定登录页面

WebAppSecurityConfig.java

```java
protected void configure(HttpSecurity security) throws Exception {
    security
            .authorizeRequests()                        //给请求授权
            .antMatchers("/index.jsp")  //对index授权
            .permitAll()                               //不需要权限 无条件访问
            .antMatchers("/layui/**")   //对layui资源授权
            .permitAll()
            .and()
            .authorizeRequests()                      //给请求授权
            .anyRequest()                             //任意请求
            .authenticated()                          //登录用户才可以访问
            .and()
            .formLogin()                              //使用表单形式登录
            .loginPage("/index.jsp")                  //指定登录页面地址，未指定的情况会跳转默认的登录页面
            .loginProcessingUrl("/do/login.html")     //指定提交登录表单地址，指定这个值会覆盖loginPage设置的值
            ;
}
```

### 3、设置登录账号密码

WebAppSecurityConfig.java

```Java
	@Override
    protected void configure(AuthenticationManagerBuilder builder) throws Exception {
        builder.inMemoryAuthentication()
                .withUser("jack").password("123456")    //设置账号密码
                .roles("ADMIN", "学徒")                           //指定当前用户角色
                .and()
                .withUser("rose").password("123456")
                .authorities("update")               //设置当前用户的权限
        ;
    }

@Override
protected void configure(HttpSecurity security) throws Exception {
    security
            .authorizeRequests()                        //给请求授权
            .antMatchers("/index.jsp")  //对index授权
            .permitAll()                               //不需要权限 无条件访问
            .antMatchers("/layui/**")   //对layui资源授权
            .permitAll()
            .and()
            .authorizeRequests()                      //给请求授权
            .anyRequest()                             //任意请求
            .authenticated()                          //登录用户才可以访问
            .and()
            .formLogin()                              //使用表达形式登录
            .loginPage("/index.jsp")                  //指定登录页面，未指定的情况会跳转默认的登录页面
            .loginProcessingUrl("/do/login.html")     //指定提交登录表单地址，指定这个值会覆盖loginPage设置的值
            .usernameParameter("loginAcct")           //登录账号请求参数名   如果不指定就会使用默认的username和password
            .passwordParameter("userPswd")            //登录密码请求参数名
            .defaultSuccessUrl("/main.html")          //登录成功后跳转的地址
    ;
}
```

index.jsp加一个表单和配置

```jsp
<!-- 显示错误信息 -->
<p>${SPRING_SECURITY_LAST_EXCEPTION.message}</p>
	<!-- 登录表单 -->
   <form action="${pageContext.request.contextPath}/do/login.html" method="post">
       <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
       .....
       .....
</form>
```

csrf：防止跨站请求伪造，如果没有禁用该功能，springSecurity要求必须携带该值，否则不处理请求。

### 4、用户注销

#### 4.1、禁用csrf情况

WebAppSecurityConfig.java

```Java
protected void configure(HttpSecurity security) throws Exception {
        security
                .authorizeRequests()                        //给请求授权
                .antMatchers("/index.jsp")  //对index授权
                .permitAll()                               //不需要权限 无条件访问
                .antMatchers("/layui/**")   //对layui资源授权
                .permitAll()
                .and()
                .authorizeRequests()                      //给请求授权
                .anyRequest()                             //任意请求
                .authenticated()                          //登录用户才可以访问
                .and()
                .formLogin()                              //使用表达形式登录
                .loginPage("/index.jsp")                  //指定登录页面，未指定的情况会跳转默认的登录页面
                .loginProcessingUrl("/do/login.html")     //指定提交登录表单地址，指定这个值会覆盖loginPage设置的值
                .usernameParameter("loginAcct")           //登录账号请求参数名   如果不指定就会使用默认的username和password
                .passwordParameter("userPswd")            //登录密码请求参数名
                .defaultSuccessUrl("/main.html")          //登录成功后跳转的地址
                .and()
                .csrf()
                .disable()                                //禁用csrf功能
                .logout()                                 //开启退出功能
                .logoutUrl("/do/logout.html")             //登出功能请求地址
                .logoutSuccessUrl("/index.jsp")           //退出成功后的页面
        ;
    }
```

退出的超链接

```
a href="${pageContext.request.contextPath}/do/logout.html">退出</a>
```

#### 4.2、使用csrf情况

如果CSRF没有被禁用，那么退出请求就必须携带CSRF的值，使用post方式发送

WebAppSecurityConfig.java

```Java
protected void configure(HttpSecurity security) throws Exception {
        security
                .authorizeRequests()                        //给请求授权
                .antMatchers("/index.jsp")  //对index授权
                .permitAll()                               //不需要权限 无条件访问
                .antMatchers("/layui/**")   //对layui资源授权
                .permitAll()
                .and()
                .authorizeRequests()                      //给请求授权
                .anyRequest()                             //任意请求
                .authenticated()                          //登录用户才可以访问
                .and()
                .formLogin()                              //使用表达形式登录
                .loginPage("/index.jsp")                  //指定登录页面，未指定的情况会跳转默认的登录页面
                .loginProcessingUrl("/do/login.html")     //指定提交登录表单地址，指定这个值会覆盖loginPage设置的值
                .usernameParameter("loginAcct")           //登录账号请求参数名   如果不指定就会使用默认的username和password
                .passwordParameter("userPswd")            //登录密码请求参数名
                .defaultSuccessUrl("/main.html")          //登录成功后跳转的地址
                .and()
//                .csrf()
//                .disable()                                //禁用csrf功能
                .logout()                                 //开启退出功能
                .logoutUrl("/do/logout.html")             //登出功能请求地址
                .logoutSuccessUrl("/index.jsp")           //退出成功后的页面
        ;
    }
```

退出的jsp

```jsp
<form id="logoutForm" action="${pageContext.request.contextPath }/do/logout.html" method="post">
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
</form>
<a id="logoutAnchor" href="">退出</a>
<script type="text/javascript">
    window.onload = function() {
       //给退出的超链接绑定一个单击事件
        document.getElementById("logoutAnchor").onclick = function() {
           //获取退出登录的表单再提交
            document.getElementById("logoutForm").submit();
            //取消超链接的默认行为
            return false;
        };
    };
</script>
```

### 5、基于角色或权限进行访问控制

WebAppSecurityConfig.java

```Java
@Override
protected void configure(AuthenticationManagerBuilder builder) throws Exception {
    builder.inMemoryAuthentication()
            .withUser("jack").password("123456")    //设置账号密码
            .roles("ADMIN", "学徒")                 //指定当前用户角色，SpringSecurity底层会在角色的名称前面加上ROLE_前缀，如果是从数据库查询到的角色数据，需要自己加上这个前缀
            .and()
            .withUser("rose").password("123456")
            .authorities("update", "level2权限")    //设置当前用户的权限
    ;
}

@Override
protected void configure(HttpSecurity security) throws Exception {
    security
            .authorizeRequests()                      //给请求授权
            .antMatchers("/index.jsp") 			      //对index授权
            .permitAll()                              //不需要权限 无条件访问
            .antMatchers("/layui/**")     	  		 //对layui资源授权
            .permitAll()
            .antMatchers("/level1/**") 				//对level1的路径设置访问要求
            .hasRole("学徒")                           //要求用户具备学徒的角色
            .antMatchers("/level2/**") 				//对level2的路径设置访问要求
            .hasAnyAuthority("level2权限") 		//要求用户具备特定权限
            .and()
            .authorizeRequests()                      //给请求授权
            .anyRequest()                             //任意请求
            .authenticated()                          //登录用户才可以访问
            .and()
            .formLogin()                              //使用表达形式登录
            .loginPage("/index.jsp")                  //指定登录页面，未指定的情况会跳转默认的登录页面
            .loginProcessingUrl("/do/login.html")     //指定提交登录表单地址，指定这个值会覆盖loginPage设置的值
            .usernameParameter("loginAcct")           //登录账号请求参数名   如果不指定就会使用默认的username和password
            .passwordParameter("userPswd")            //登录密码请求参数名
            .defaultSuccessUrl("/main.html")          //登录成功后跳转的地址
            .and()
            //.csrf()
            //.disable()                              //禁用csrf功能
            .logout()                                 //开启退出功能
            .logoutUrl("/do/logout.html")             //登出功能请求地址
            .logoutSuccessUrl("/index.jsp")           //退出成功后的页面
    ;
}
```

### 6、自定义403（无权限访问）页面

WebAppSecurityConfig.java

```Java
protected void configure(HttpSecurity security) throws Exception {
    security
            .authorizeRequests()                      //给请求授权
            .antMatchers("/index.jsp") //对index授权
            .permitAll()                              //不需要权限 无条件访问
            .antMatchers("/layui/**")  //对layui资源授权
            .permitAll()
            .antMatchers("/level1/**") //对level1的路径设置访问要求
            .hasRole("学徒")                           //要求用户具备学徒的角色
            .antMatchers("/level2/**") //对level2的路径设置访问要求
            .hasAnyAuthority("level2权限") //要求用户具备特定权限
            .and()
            .authorizeRequests()                      //给请求授权
            .anyRequest()                             //任意请求
            .authenticated()                          //登录用户才可以访问
            .and()
            .formLogin()                              //使用表达形式登录
            .loginPage("/index.jsp")                  //指定登录页面，未指定的情况会跳转默认的登录页面
            .loginProcessingUrl("/do/login.html")     //指定提交登录表单地址，指定这个值会覆盖loginPage设置的值
            .usernameParameter("loginAcct")           //登录账号请求参数名   如果不指定就会使用默认的username和password
            .passwordParameter("userPswd")            //登录密码请求参数名
            .defaultSuccessUrl("/main.html")          //登录成功后跳转的地址
            .and()
            //.csrf()
            //.disable()                              //禁用csrf功能
            .logout()                                 //开启退出功能
            .logoutUrl("/do/logout.html")             //登出功能请求地址
            .logoutSuccessUrl("/index.jsp")           //退出成功后的页面
            .and()
            .exceptionHandling()                      //指定异常处理器
            //.accessDeniedPage("/to/no/auth/page.html")//访问被拒绝跳转到已写好的页面
            .accessDeniedHandler(new AccessDeniedHandler() {
                //或者使用accessDeniedHandler可以自定义页面显示的消息
                @Override
                public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AccessDeniedException e) throws IOException, ServletException {
                    httpServletRequest.setAttribute("message","无权访问！");
                    httpServletRequest.getRequestDispatcher("/WEB-INF/views/no_auth.jsp").forward(httpServletRequest,httpServletResponse);
                }
            })
    ;
}
```

### 7、记住我

#### 7.1、基于内存版

![image-20200628211155241](http://picture.youyouluming.cn/image-20200628132245664.png)

在configure方法开启记住我功能，此时前端页面上记住我的checkBox的name设置为rememberMed的默认值remember-me。

![image-20200628211307642](http://picture.youyouluming.cn/image-20200628211307642.png)

是以cookie的形式把用户的信息存储在浏览器中，默认过期时间是14天

#### 7.2、基于数据库版

在服务器重启后内存版的记住我就不能用了，使用数据库改进。

创建数据库

配置数据库环境spring-mvc.xml

```xml
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="username" value="root"/>
    <property name="password" value="admin"/>
    <property name="url" value="jdbc:mysql://localhost:3306/security?useSSL=false"/>
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

创建保存persistent_logins表

```sql
CREATE TABLE `persistent_logins` (
  `username` varchar(64) NOT NULL,
  `series` varchar(64) NOT NULL,
  `token` varchar(64) NOT NULL,
  `last_used` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`series`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

在configure方法内部获取tokenRepository对象，并装配数据源

![image-20200628222531274](http://picture.youyouluming.cn/image-20200628222531274.png)

开启令牌仓库

![image-20200628222550918](http://picture.youyouluming.cn/image-20200628222550918.png)

```Java
protected void configure(HttpSecurity security) throws Exception {
    JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();
    tokenRepository.setDataSource(dataSource);
    security
            .authorizeRequests()                      //给请求授权
            .antMatchers("/index.jsp") //对index授权
            .permitAll()                              //不需要权限 无条件访问
            .antMatchers("/layui/**")  //对layui资源授权
            .permitAll()
            .antMatchers("/level1/**") //对level1的路径设置访问要求
            .hasRole("学徒")                           //要求用户具备学徒的角色
            .antMatchers("/level2/**") //对level2的路径设置访问要求
            .hasAnyAuthority("level2权限") //要求用户具备特定权限
            .and()
            .authorizeRequests()                      //给请求授权
            .anyRequest()                             //任意请求
            .authenticated()                          //登录用户才可以访问
            .and()
            .formLogin()                              //使用表达形式登录
            .loginPage("/index.jsp")                  //指定登录页面，未指定的情况会跳转默认的登录页面
            .loginProcessingUrl("/do/login.html")     //指定提交登录表单地址，指定这个值会覆盖loginPage设置的值
            .usernameParameter("loginAcct")           //登录账号请求参数名   如果不指定就会使用默认的username和password
            .passwordParameter("userPswd")            //登录密码请求参数名
            .defaultSuccessUrl("/main.html")          //登录成功后跳转的地址
            .and()
            //.csrf()
            //.disable()                              //禁用csrf功能
            .logout()                                 //开启退出功能
            .logoutUrl("/do/logout.html")             //登出功能请求地址
            .logoutSuccessUrl("/index.jsp")           //退出成功后的页面
            .and()
            .exceptionHandling()                      //指定异常处理器
            //.accessDeniedPage("/to/no/auth/page.html")//访问被拒绝跳转到已写好的页面
            .accessDeniedHandler(new AccessDeniedHandler() {
                //或者accessDeniedHandler可以自定义页面显示的消息
                @Override
                public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AccessDeniedException e) throws IOException, ServletException {
                    httpServletRequest.setAttribute("message", "无权访问！");
                    httpServletRequest.getRequestDispatcher("/WEB-INF/views/no_auth.jsp").forward(httpServletRequest, httpServletResponse);
                }
            })
            .and()
            .rememberMe()                             //开启记住我功能
            .tokenRepository(tokenRepository)         //启动令牌仓库功能
    ;
}
```

### 8、查询数据库登录

创建用户表

```sql
CREATE TABLE `t_admin` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `loginacct` varchar(255) NOT NULL,
  `userpswd` char(64) NOT NULL,
  `username` varchar(255) NOT NULL,
  `email` varchar(255) NOT NULL,
  `createtime` char(19) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=17 DEFAULT CHARSET=utf8;
```



先要创建一个实现了UserDetailService接口的类，实现loadUserByUsername方法。

```Java
@Component
public class MyUserDetailService implements UserDetailsService {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    /**
     * 根据表单提交的用户名查询user对象，并装配角色和权限等信息
     *
     * @param username 表单提交的用户名
     * @return
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //从数据库查询admin
        String sql = "select id,loginacct,userpswd,username,email from t_admin where loginacct=?";
        List<Admin> list = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(Admin.class), username);
        Admin admin = list.get(0);
        //给admin装配角色权限信息
        List<GrantedAuthority> authorities = new ArrayList<>();
        //注意：如果是角色一定加ROLE_ 用来区分角色和权限
        authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
        authorities.add(new SimpleGrantedAuthority("UPDATE"));
        //把admin和authorities封装到一起
        String userpswd = admin.getUserpswd();
        return new User(username, userpswd, authorities);
    }
}
```

 在配置类的configure方法装配使用

```
@Autowired
private MyUserDetailService userDetailService;
@Override
protected void configure(AuthenticationManagerBuilder builder) throws Exception {
   //装配MyUserDetailService
   builder.userDetailsService(userDetailService);
}
```

### 9、密码加密

SpringSecurity提供了一个密码加密接口PasswordEncoder，该接口有两个方法，一个encode（加密）一个matches（校验）

#### 9.1、实现PasswordEncoder接口

MyPasswordEncoder.java

```java
@Component
public class MyPasswordEncoder implements PasswordEncoder {
    /**
     * 加密
     * @param rawPassword   需要被加密的密码
     * @return
     */
    @Override
    public String encode(CharSequence rawPassword) {
        try {
            //创建MessageDigest对象
            String algorithm = "MD5";
            MessageDigest messageDigest = MessageDigest.getInstance(algorithm);
            //获取rawPassword的字节数组
            byte[] input = ((String) rawPassword).getBytes();
            //加密
            byte[] output = messageDigest.digest(input);
            //转换为16进制数对应的字符，便用保存
            String encoded = new BigInteger(1, output).toString(16);
            return encoded;
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 校验
     * @param rawPassword   明文密码
     * @param encodedPassword   加密后的密码
     * @return
     */
    @Override
    public boolean matches(CharSequence rawPassword, String encodedPassword) {
        //对名文密码先加密后再做校验
        String formPassword = encode(rawPassword);
        //数据库的密码
        String databasePassword = encodedPassword;
        //校验
        return Objects.equals(formPassword,databasePassword);
    }
}
```

#### 9.2、在SpringSecurity装配使用MyPasswordEncoder

```Java
@Autowired
private MyPasswordEncoder encoder;

@Override
protected void configure(AuthenticationManagerBuilder builder) throws Exception {
    //装配MyUserDetailService
    builder.userDetailsService(userDetailService).passwordEncoder(encoder);
}
```

### 10、带盐值的密码加密

之前密码加密，明文密码生成的密文是固定的，加盐之后生成的密文是随机的。

```Java
//把BCryptPasswordEncoder注入IOC容器
@Bean
public BCryptPasswordEncoder getPasswordEncoder(){
    return new BCryptPasswordEncoder();
}
@Override
protected void configure(AuthenticationManagerBuilder builder) throws Exception {
    //使用带盐值的加密
   builder.userDetailsService(userDetailService).passwordEncoder(getPasswordEncoder());
}
```



## 五、在项目中加入SpringSecurity

### 1、环境

* 加入依赖
* 在web.xml配置DelegatingFilterProxy
* 创建配置注解类WebAppSecurityConfig
* 使用springMVC的IOC扫描WebAppSecurityConfig，方便springSecurity进行权限控制

注意：此时启动服务器是会找不到springSecurityFilterChain



### 2、解决找不到springSecurityFilterChain问题

因为WebAppSecurityConfig是放在springMVC的IOC容器中，在服务器启动时会先创建spring的IOC容器，此时会查找springSecurityFilterChain，然后出异常服务器启动失败。

#### 2.1、解决方案一：把两个IOC容器合并

不使用contextConfigLocation，让dispatcherServlet加载所有的spring配置文件

web.xml

![image-20200629205836538](http://picture.youyouluming.cn/image-20200629205836538.png)

![image-20200629205930890](http://picture.youyouluming.cn/image-20200629210807673.png)

#### 2.2、方案二：改源码

创建DelegatingFilterProxy同包同名的类，把源码复制进来。

修改DelegatingFilterProxy源码

![image-20200629210807673](http://picture.youyouluming.cn/image-20200630003206945.png)

```Java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {
    Filter delegateToUse = this.delegate;
    if (delegateToUse == null) {
        synchronized(this.delegateMonitor) {
            delegateToUse = this.delegate;
            if (delegateToUse == null) {
                //旧的查找IOC容器代码
                //WebApplicationContext wac = this.findWebApplicationContext();

                //重新实现
                //获取servletContext对象
                ServletContext sc = this.getServletContext();
                //拼接springMVC将IOC存入ServletContext域时的名字
                String servletName = "dispatcherServlet";
                String attrName = FrameworkServlet.SERVLET_CONTEXT_PREFIX + servletName;
                //根据attrName从ServletContext域中获取IOC容器对象
                WebApplicationContext wac = (WebApplicationContext) sc.getAttribute(attrName);

                if (wac == null) {
                    throw new IllegalStateException("No WebApplicationContext found: no ContextLoaderListener or DispatcherServlet registered?");
                }

                delegateToUse = this.initDelegate(wac);
            }

            this.delegate = delegateToUse;
        }
    }

    this.invokeDelegate(delegateToUse, request, response, filterChain);
}
```

### 3、放行首页和静态资源

WebAppSecurityConfig.java

```Java
@Configuration      //表示当前类是配置类
@EnableWebSecurity  //在web环境下启用SpringSecurity
public class WebAppSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity security) throws Exception {
        security
                .authorizeRequests()                                         //对请求授权
                .antMatchers("/admin/to/login/page.html")    //放行登录页
                .permitAll()                                               //无条件访问
                .antMatchers("/bootstrap/**")               //放行静态资源
                .permitAll()
                .antMatchers("/css/**")
                .permitAll()
                .antMatchers("/fonts/**")
                .permitAll()
                .antMatchers("/img/**")
                .permitAll()
                .antMatchers("/jquery/**")
                .permitAll()
                .antMatchers("/layer/**")
                .permitAll()
                .antMatchers("/script/**")
                .permitAll()
                .antMatchers("/scw/**")
                .permitAll()
                .antMatchers("/ztree/**")
                .permitAll()
                .anyRequest()
                .authenticated()
                ;
    }
}
```



### 4、登录（内存版）

WebAppSecurityConfig.java

```Java
@Configuration      //表示当前类是配置类
@EnableWebSecurity  //在web环境下启用SpringSecurity
public class WebAppSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder builder) throws Exception {
        //使用内存版验证代码
        builder.inMemoryAuthentication().withUser("jack").password("123456").roles("ADMIN");
    }

    @Override
    protected void configure(HttpSecurity security) throws Exception {
        security
                .authorizeRequests()                                         //对请求授权
                .antMatchers("/admin/to/login/page.html")    //放行登录页
                .permitAll()                                               //无条件访问
                .antMatchers("/bootstrap/**")               //放行静态资源
                .permitAll()
                .antMatchers("/css/**")
                .permitAll()
                .antMatchers("/fonts/**")
                .permitAll()
                .antMatchers("/img/**")
                .permitAll()
                .antMatchers("/jquery/**")
                .permitAll()
                .antMatchers("/layer/**")
                .permitAll()
                .antMatchers("/script/**")
                .permitAll()
                .antMatchers("/scw/**")
                .permitAll()
                .antMatchers("/ztree/**")
                .permitAll()
                .anyRequest()                                   //其他任意请求
                .authenticated()                                //认证后访问
                .and()
                .csrf()
                .disable()                                      //禁用csrf
                .formLogin()                                    //开启表单登录配置
                .loginPage("/admin/to/login/page.html")         //登录页面
                .loginProcessingUrl("/security/do/login.html")  //处理登录请求地址
                .usernameParameter("loginAcct")                 //账号的请求参数名称
                .passwordParameter("userPswd")                  //密码的请求参数名称
                .defaultSuccessUrl("/admin/to/main/page.html")  //登录成功后的页面
        ;
    }
}
```

admin-login.jsp

```jsp
<form action="security/do/login.html" method="post" class="form-signin" role="form">
    <h2 class="form-signin-heading"><i class="glyphicon glyphicon-log-in"></i> 管理员登录</h2>
    <p>${requestScope.exception.message}</p>
    <p>${SPRING_SECURITY_LAST_EXCEPTION.message}</p>
    <div class="form-group has-success has-feedback">
        <input type="text" name="loginAcct" value="jack" class="form-control" id="inputUsername1" placeholder="请输入登录账号" autofocus>
        <span class="glyphicon glyphicon-user form-control-feedback"></span>
    </div>
    <div class="form-group has-success has-feedback">
        <input type="text" name="userPswd" value="123123" class="form-control" id="inputPassword1" placeholder="请输入登录密码" style="margin-top:10px;">
        <span class="glyphicon glyphicon-lock form-control-feedback"></span>
    </div>
    <button class="btn btn-lg btn-success btn-block" type="submit"> 登录</button>
</form>
```

注意：把登录的拦截器删除掉

![image-20200630003206945](http://picture.youyouluming.cn/image-20200630104838568.png)

### 5、退出登录

WebAppSecurityConfig.java

![image-20200630104838568](http://picture.youyouluming.cn/image-20200630104914504.png)

include-nav.jsp

![image-20200630104914504](http://picture.youyouluming.cn/image-20200630115208144.png)

### 6、登录（数据库登录）

####  6.1、根据admin查询已分配角色

RoleService.java

![image-20200630115208144](http://picture.youyouluming.cn/image-20200629205930890.png)

RoleServiceImpl.java

```java 
@Override
public List<Role> getUnAssignedRole(Integer adminId) {
    return roleMapper.selectUnAssignedRole(adminId);
}
```

#### 6.2、根据adminId查询已分配权限

AuthService.java

![image-20200630115629309](http://picture.youyouluming.cn/image-20200630115629309.png)

AuthServiceImpl.java

```java
@Override
public List<String> getAssignedAuthNameByAdminId(Integer adminId) {
    return authMapper.selectAuthNameByAdminId(adminId);
}
```

AuthMapper.java

```java
List<String> selectAuthNameByAdminId(Integer adminId);
```

AuthMapper.xml

```xml
<select id="selectAuthNameByAdminId" resultType="java.lang.String">
    SELECT DISTINCT
        t_auth.`name`
    FROM
        t_auth
    LEFT JOIN inner_role_auth ON t_auth.id = inner_role_auth.auth_id
    LEFT JOIN inner_admin_role ON inner_admin_role.role_id = inner_role_auth.role_id
    WHERE
        inner_admin_role.admin_id = #{adminId}
    AND t_auth.`name` != ""
    AND t_auth.`name` IS NOT NULL;
</select>
```

#### 6.3、创建SecurityAdmin类

SecurityAdmin.java

```Java
/**
 * 使用原始的Admin对象对User类扩展
 */
public class SecurityAdmin extends User {
    private Admin originalAdmin;    //原始Admin对象

    public SecurityAdmin(Admin originalAdmin, List<GrantedAuthority> authorities) {
        super(originalAdmin.getLoginAcct(), originalAdmin.getUserPswd(), authorities);
        this.originalAdmin = originalAdmin;
    }

    public Admin getOriginalAdmin() {
        return originalAdmin;
    }
}
```

#### 6.4、创建userdetailsService实现类

CrowdUserDetailsService.java

```Java
@Component
public class CrowdUserDetailsService implements UserDetailsService {
    @Autowired
    private AdminService adminService;
    @Autowired
    private RoleService roleService;
    @Autowired
    private AuthService authService;

    @Override
    public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {
        //根据账号名称查询Admin对象
        Admin admin = adminService.getAdminByLoginAcct(userName);
        //获取adminId
        Integer adminId = admin.getId();
        //根据adminId查询角色信息
        List<Role> assignedRoleList = roleService.getAssignedRole(adminId);
        //根据adminId查询权限信息
        List<String> authNameList = authService.getAssignedAuthNameByAdminId(adminId);
        //创建集合对象存储GrantedAuthority
        List<GrantedAuthority> authorities = new ArrayList<>();
        //遍历assignedRoleList存入角色信息
        for (Role role : assignedRoleList) {
            //角色名前需要加前缀
            String roleName = "ROLE_" + role.getName();
            SimpleGrantedAuthority simpleGrantedAuthority = new SimpleGrantedAuthority(roleName);
            authorities.add(simpleGrantedAuthority);
        }

        //遍历authNameList存入权限信息
        for (String authName : authNameList) {
            SimpleGrantedAuthority simpleGrantedAuthority = new SimpleGrantedAuthority(authName);
            authorities.add(simpleGrantedAuthority);
        }

        //封装SecurityAdmin对象
        SecurityAdmin securityAdmin = new SecurityAdmin(admin, authorities);
        return securityAdmin;
    }
}
```

在配置类开启UserDetailsService登录验证

![image-20200630144105611](http://picture.youyouluming.cn/image-20200630144105611.png)

### 7、密码加密

#### 7.1、把BCryptPasswordEncoder对象放到IOC容器

![image-20200630171136545](http://picture.youyouluming.cn/image-20200630171136545.png)

#### 7.2、使用BCryptPasswordEncoder

![image-20200630171316576](http://picture.youyouluming.cn/image-20200630171316576.png)

#### 7.3、保存用户密码加密

![image-20200630171339079](http://picture.youyouluming.cn/image-20200630171339079.png)



### 8、回显用户昵称

从security中获取用户昵称

先在需要显示的页面导入security的标签库

```jsp
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>
```

在使用

> <security:authentication property="principal.originalAdmin.userName"/>

显示昵称



### 9、对资源进行权限控制

#### 9.1、访问用户分页控制

设置只有“经理”角色可以访问用户分页

![image-20200630204948839](http://picture.youyouluming.cn/image-20200630204948839.png)

#### 9.2、访问角色分页控制

设置只有“部长”角色可以访问角色分页

使用注解方式

在配置类WebAppSecurityConfig上加

![image-20200630210850982](http://picture.youyouluming.cn/image-20200630210850982.png)

在需要进行权限控制的方法上加

![image-20200630210922770](http://picture.youyouluming.cn/image-20200630210922770.png)

基于注解的异常映射

CrowdExceptionResolver.java

```java
@ExceptionHandler(value = Exception.class)
public ModelAndView resolveLoginException(Exception e, HttpServletRequest request, HttpServletResponse response) throws IOException {
    String viewName = "admin-login";
    return commonResolve(e, request, response, viewName);
}
```

#### 9.3、403的页面跳转处理

WebAppSecurityConfig

![image-20200630214932836](http://picture.youyouluming.cn/image-20200630214932836.png)

#### 9.4、对admin的save设置访问权限



![image-20200630222226947](http://picture.youyouluming.cn/image-20200630222226947.png)

#### 9.5、访问用户分页控制升级

只有具备经理角色或user:get权限才可以访问

![image-20200630222416377](http://picture.youyouluming.cn/image-20200630222416377.png)

#### 

