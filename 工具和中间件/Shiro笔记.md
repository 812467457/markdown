# Shiro

## 一、Shiro是什么

### 1、介绍

* Shiro是Java的一个安全（权限）框架
* Shiro不但适用JavaSE环境，也可以在JavaEE环境使用
* Shiro可以完成：认证、授权、加密、会话管理、与web集成、缓存等。

基本功能如下图所示：

![image-20200822162518174](http://picture.youyouluming.cn/image-20200822162518174.png)

* **Authentication**：身份认证 / 登录，验证用户是不是拥有相应的身份；

* **Authorization**：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

* **Session** **Management**：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通 JavaSE 环境的，也可以是如 Web 环境的；

* **Cryptography**：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

* **Web Support**：Web 支持，可以非常容易的集成到 Web 环境；

* **Caching**：缓存，比如用户登录后，其用户信息、拥有的角色 / 权限不必每次去查，这样可以提高效率；

* **Concurrency**：shiro 支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；

* **Testing**：提供测试支持；

* **Run As**：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

* **Remember Me**：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。

### 2、架构

架构图：

![image-20200822162838505](http://picture.youyouluming.cn/image-20200822162838505.png)

* **Subject**：主体，代表了当前 “用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是 Subject，如网络爬虫，机器人等；即一个抽象概念；**所有 Subject 都绑定到 SecurityManager**，与 Subject 的所有交互都会委托给 SecurityManager；可以把 Subject 认为是一个门面；SecurityManager 才是实际的执行者；
* **SecurityManager**：安全管理器；即所有与安全有关的操作都会与 SecurityManager 交互；且它管理着所有 Subject；可以看出它是 Shiro 的核心，它负责与后边介绍的其他组件进行交互，如果学习过 SpringMVC，你可以把它看成 DispatcherServlet 前端控制器；
* **Realm**：域，Shiro 从从 Realm 获取安全数据（如用户、角色、权限），就是说 **SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法**；也需要从 Realm 得到用户相应的角色 / 权限进行验证用户是否能进行操作；可以把 Realm 看成 DataSource，即安全数据源。也就是说对于我们而言，最简单的一个 Shiro 应用：



## 二、SpringBoot整合Shiro

> 导入依赖

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.6.0</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.5.3</version>
</dependency>
```

> 自定义Realm类

```Java
public class UserRealm extends AuthorizingRealm {
    /**
     * 执行授权逻辑
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        System.out.println("执行授权逻辑");
        return null;
    }

    /**
     * 执行认证逻辑
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        System.out.println("执行认证逻辑");
        return null;
    }
}
```

> shiro基本配置类

```java
@Configuration
public class ShiroConfig {

    /**
     * 创建ShiroFilterFactoryBean
     *
     * @return
     */
    @Bean(name = "getShiroFilterFactoryBean")
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultSecurityManager defaultSecurityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(defaultSecurityManager);
        return shiroFilterFactoryBean;
    }

    /**
     * 创建一个DefaultWebSecurityManager
     *
     * @return
     */
   	@Bean(name = "securityManager")
    public DefaultWebSecurityManager securityManager(@Qualifier("userRealm") UserRealm userRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联一个Realm
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    /**
     * 创建Realm
     *
     * @return
     */
    @Bean(name = "userRealm")
    public UserRealm getRealm() {
        return new UserRealm();
    }
}
```



## Shiro入门使用

### 1、使用Shiro过滤器实现认证资源拦截

> 在shiro的配置类getShiroFilterFactoryBean方法中添加拦截器

```Java
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(securityManager);

        /**
         * 添加shiro过滤器，实现权限相关的拦截器
         * 过滤器：
         *      anon：无需认证可以访问
         *      authc：必须认证才可以访问
         *      user：可以使用记住我功能
         *      perms：该资源必须获取资源权限才可以访问
         *      role：该资源必须获取角色资源才可以访问
         */
        //把需要的过滤器追加到map，使用LinkedHashMap保证顺序
        Map<String, String> map = new LinkedHashMap<>();
       	//添加权限，key为拦截路径，value为过滤器       可以使用通配符 /*
        map.put("/add", "authc");
        map.put("/update", "authc");
        //设置无需拦截的资源
        map.put("/test", "anon");
        //设置登录跳转页面
        shiroFilterFactoryBean.setLoginUrl("/toLogin");
        //把map设置到shiroFilterFactoryBean
        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
        return shiroFilterFactoryBean;
    }
```

结果：此时访问被拦截资源会自动跳转到登录的地址

![image-20200822213546167](http://picture.youyouluming.cn/image-20200822213546167.png)



### 2、用户登录（认证）

> controller中的登录方法

```Java
@RequestMapping("/login")
public String login(String username, String password, Model model) {
    /**
     * 使用shiro的认证
     */
    //获取subject
    Subject subject = SecurityUtils.getSubject();
    //封装用户数据
    UsernamePasswordToken token = new UsernamePasswordToken(username, password);
    //执行登录方法，没有异常就是登录成功
    try {
        subject.login(token);
        //登录成功，跳转到登录成功后的页面
        return "redirect:test";
    } catch (UnknownAccountException e) {
        //e.printStackTrace();
        //UnknownAccountException 用户名不存在
        model.addAttribute("msg", "用户名不存在");
        return "login";
    } catch (IncorrectCredentialsException e) {
        //IncorrectCredentialsException 密码错误
        model.addAttribute("msg", "密码错误");
        return "login";
    }
}
```

> Realm类做认证逻辑，现在用的时模拟数据，使用数据库数据时注入service和之前一样的调用。

```java
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
    System.out.println("执行认证逻辑");
    //模拟用户名和密码
    String username = "jack";
    String password = "123";
    //用户名密码存在authenticationToken中
    UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
    //判断用户名
    if (!username.equals(token.getUsername())) {
        //此处返回null，controller中的登录方法就会抛出UnknownAccountException
        return null;
    }
    //判断密码  返回AuthenticationInfo的子类SimpleAuthenticationInfo
    return new SimpleAuthenticationInfo("",password,"");
}
```



### 3、用户授权

> 在shiro的配置类中ShiroFilterFactoryBean方法对未授权资源进行拦截

```Java
//授权过滤器，访问未授权资源会跳转到一个未授权页面
map.put("/test","perms[user:add]");
//未授权提示
shiroFilterFactoryBean.setUnauthorizedUrl("/unAuth");
```

> 在Realm类中执行授权

```java
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
    System.out.println("执行授权逻辑");
    //给资源授权，创建AuthorizationInfo接口的实现类SimpleAuthorizationInfo
    SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
    //添加授权字符串，要和之前的在配置中过滤器上的授权字符串一致
    info.addStringPermission("user:add");
    return info;
}
```

