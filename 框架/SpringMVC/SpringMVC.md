# SpringMVC入门
## 一、springMVC概述
SpringMVC 是一种基于Java的实现MVC设计模型的请求驱动类型的轻量级Web 框架，属于 Spring       
FrameWork 的后续产品，已经融合在Spring Web Flow里面。Spring 框架提供了构建 Web 应用程序的全功
能 MVC 模块。使用 Spring 可插入的 MVC 架构，从而在使用Spring进行WEB开发时，可以选择使用 Spring 
的 Spring MVC 框架或集成其他MVC开发框架，如Struts1(现在一般不用)，Struts2等。 
SpringMVC已经成为目前最主流的 MVC 框架之一，并且随着Spring3.0的发布，全面超越 Struts2，成
为最优秀的 MVC 框架。 
它通过一套注解，让一个简单的Java类成为处理请求的控制器，而无须实现任何接口。同时它还支持
RESTful编程风格的请求。

## 二、springMVC的入门使用
jsp页面发送请求，springMVC处理请求，并转发到成功页面
### 1. 环境搭建
##### 导入依赖
```xml
<properties>
    <!-- 版本锁定 -->
    <spring.version>5.0.2.RELEASE</spring.version>
</properties>
<dependencies>
    <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>${spring.version}</version>
    </dependency>
    <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-web</artifactId>
          <version>${spring.version}</version>
    </dependency>
    <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-webmvc</artifactId>
          <version>${spring.version}</version>
    </dependency>
    <dependency>
          <groupId>javax.servlet</groupId>
          <artifactId>servlet-api</artifactId>
          <version>2.5</version>
          <scope>provided</scope>
    </dependency>
    <dependency>
          <groupId>javax.servlet.jsp</groupId>
          <artifactId>jsp-api</artifactId>
          <version>2.0</version>
          <scope>provided</scope>
    </dependency>
</dependencies>
```

##### 在web.xml中添加前端控制器配置
```xml
<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <!-- 前端控制器配置  -->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 全局初始化参数，用于加载springmvc.xml配置文件 -->
        <init-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!-- 设置启动服务器加载 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

##### 在resources中添加一个spring的xml配置文件
##### 配置tomcat
### 2. 实现需求：跳转页面并执行方法
##### jsp页面发送请求
```
<a href="/param/testParam">连接</a>
```
##### spring.xml文件的配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:mvc="http://www.springframework.org/schema/mvc"
   xmlns:context="http://www.springframework.org/schema/context"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
<!-- 开启注解扫描,目标类会被创建对象 -->  
<context:component-scan base-package="cn.luming"/>
<!-- 配置视图解析器,指定跳转的页面 -->
<bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!-- 配置文件所在目录 -->
    <property name="prefix" value="/WEB-INF/pages/"/>
    <!-- 配置文件的后缀名 -->
    <property name="suffix" value=".jsp"/>
</bean>
<!-- 开启springMVC注解支持，默认就是开启，此配置不是必要 -->
<mvc:annotation-driven/>
</beans>
```
##### Controller控制器，通过此类完成跳转并执行方法
```Java
@RequestMapping(path = "/param")
@Controller
public class Controller {
    @RequestMapping(path = "/testParam")//请求映射
    public String printHello(){
        System.out.println("hello");
        //在springmvc中会通过视图解析器根据方法返回的名字自动跳转到该jsp页面
        return "success";
    }
}
```

### 3. 入门案例执行流程
1. 当启动Tomcat服务器的时候，因为配置了load-on-startup标签，所以会创建DispatcherServlet对象，
就会加载springmvc.xml配置文件
2. 开启了注解扫描，那么HelloController对象就会被创建
3. 从index.jsp发送请求，请求会先到达DispatcherServlet核心控制器，根据配置@RequestMapping注解
找到执行的具体方法
4. 根据执行方法的返回值，再根据配置的视图解析器，去指定的目录下查找指定名称的JSP文件
5. Tomcat服务器渲染页面，做出响应

### 4. 入门案例组件分析
1. 前端控制器（DispatcherServlet）
2. 处理器映射器（HandlerMapping）
3. 处理器（Handler）
4. 处理器适配器（HandlAdapter）
5. 视图解析器（View Resolver）
6. 视图（View）

## 三、RequestMapping注解的作用
1. 作用客户端发送一个请求，RequestMapping注解来建立该方法和请求的映射关联，加到类上可以表示具体某个类中的方法。
2. 属性
value/path：指映射路径
method：当前方法可以接收哪种的请求方式
params：用于指定限制请求参数的条件。要求请求参数的key和value必须和配置的一样
headers：发送请求中必须包含的请求头

## 四、请求参数绑定
### 1. 绑定机制
1. 表单提交的数据都是k=v格式的  username=haha&password=123
2. SpringMVC的参数绑定过程是把表单提交的请求参数，作为控制器中方法的参数进行绑定的
3. 要求：提交表单的name和参数的名称是相同的

### 2. 支持的数据类型
1. 基本数据类型和字符串类型
2. 实体类型（JavaBean）
3. 集合数据类型（List、map集合等）

### 3. 基本数据类型和字符串类型
1. 提交表单的name和参数的名称是相同的
2. 区分大小写
```jsp
<a href="/param/testParam?username=abc">获取参数</a>
```
```java
@RequestMapping("/testParam")
public String testParam(String username){
    System.out.println(username);
    return "success";
}
```

### 4. 实体类型（JavaBean）
1. 提交表单的name和JavaBean中的属性名称需要一致
2. 如果一个JavaBean类中包含其他的引用类型，那么表单的name属性需要编写成：对象.属性 例如：
address.name
```html
<form action="/account/saveAccount" method="post"><br>
<%-- 参数绑定bean类型 --%>
姓名:<input type="text" name="username"><br>
密码:<input type="text" name="password"><br>
金额:<input type="text" name="money"><br>
<%-- 参数绑定引用类型 --%>
用户姓名:<input type="text" name="user.uname"><br>
用户年龄:<input type="text" name="user.age"><br>
<input type="submit" value="提交"><br>
</form>
```
```java
@RequestMapping("/saveAccount")
public String saveAccount(Account account){
    System.out.println(account);
    return "success";
}
```

### 5. 给集合属性数据封装
1. list集合的JSP页面编写方式：list[0].属性
2. map集合的JSP页面编写方式：map['键名'].属性
```html
<form action="/account/saveAccount" method="post"><br>
    <%-- 参数绑定集合类型 --%>
    用户姓名:<input type="text" name="users[0].uname"><br>
    用户年龄:<input type="text" name="users[0].age"><br>
    用户姓名:<input type="text" name="map['one'].uname"><br>
    用户年龄:<input type="text" name="map['one'].age"><br>
    <input type="submit" value="提交"><br>
</form>
```

### 6. 请求参数中文乱码的解决
在web.xml中配置Spring提供的过滤器类
```xml
<!-- 配置过滤器，解决中文乱码的问题 -->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!-- 指定字符集 -->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 7. 自定义类型转换器
1. 表单提交的任何数据类型全部都是字符串类型，但是后台定义Integer类型，数据也可以封装上，说明
Spring框架内部会默认进行数据类型转换。
2. 如果想自定义数据类型转换，可以实现Converter的接口
 自定义类型转换器    
```java
public Date convert(String source) {
    if (source == null) {
        throw new RuntimeException("字符串为空");
    }
    DateFormat df = new SimpleDateFormat("yyy-MM-dd");
    try {
        return df.parse(source);
    } catch (ParseException e) {
        throw new RuntimeException("格式错误");
    }
}
```
注册自定义类型转换器，在springmvc.xml配置文件中编写配置

```xml
<!-- 注册自定义类型转换器 -->
<bean id="conversionService" 
class="org.springframework.context.support.ConversionServiceFactoryBean">
<property name="converters">
    <set>
        <bean class="cn.luming.utils.StringToDateConverter"/>
    </set>
</property>
</bean>
<!-- 开启Spring对MVC注解的支持 -->
<mvc:annotation-driven conversion-service="conversionService"/>
```

或者在Javabean中需要自定义转换格式的属性上加注解
```java
@DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birthday;
```

### 8. 在控制器中使用原生的ServletAPI对象
只需要在控制器的方法参数定义HttpServletRequest和HttpServletResponse对象

## 五、常用注解
1. RequestParam注解
作用：
把请求中指定的名称参数给控制器中的形参赋值
属性：
value：请求参数中的名称
required：请求参数中是否必须提供此参数，默认true表示必须提供
defaultValue：默认值，如果没传参数就使用默认值
JavaBena中是username，形参使用name，使用RequestParam注解重新指定到Javabean中的名字
```Java
@RequestMapping("/RequestParam")
public String testAnno(@RequestParam(name = "username") String name){
System.out.println(name);
return "success";
}
```
2. RequestBody注解
作用：
用于获取请求体内容，直接使用得到的是键值对结构的数据。get方法不适用
属性
required：是否必须有请求体，默认值是true    
```Java
@RequestMapping("/RequestBody")
public String testAnno02(@RequestBody String body){
    System.out.println(body);
 return "success";
}
```
3. PathVariable注解    
作用：
拥有绑定url中的占位符的。例如：url中有/delete/{id}，{id}就是占位符
属性：
value：指定url中的占位符名称  

* Restful风格的URL
    1. 请求路径一样，可以根据不同的请求方式去执行后台的不同方法
    2. restful风格的URL优点
    3. 结构清晰
    4. 符合标准
    5. 易于理解
    6. 扩展方便    

js代码
```html
<a href="/anno/PathVariable/10">PathVariable</a>
```
控制器代码
```java
@RequestMapping("/PathVariable/{id}")
public String testAnno03(@PathVariable(name = "id") String id){
    System.out.println(id);
 return "success";
}
```
4. RequestHeader注解
作用：
    获取指定请求头的值
属性
    value：请求头的名称

5.  CookieValue注解
作用：
    用于获取指定cookie的名称的值
属性
    value：cookie的名称

6. ModelAttribute注解
作用： 
出现在方法上：表示当前方法会在控制器方法执行前线执行。
出现在参数上：获取指定的数据给参数赋值  
应用场景：
当提交表单数据不是完整的实体数据时，保证没有提交的字段使用数据库原来的数据。
属性：
value：获取数据的key
```Java
//写在方法上,需要返回值
@RequestMapping("/ModelAttribute")
public String testAnno04(User user) {
    System.out.println(user);
    return "success";
}
@ModelAttribute
public User showUser() {
    User user = new User();
    user.setDate(new Date());
    return user;
}
```
```Java
//写在参数上，无返回值
@RequestMapping("/ModelAttribute")
    public String testAnno04(@ModelAttribute("user") User user) {
        System.out.println(user);
        return "success";
    }
    @ModelAttribute
    public void showUser2(Map<String, User> map) {
        User user = new User();
        user.setDate(new Date());
        map.put("user", user);
    }
```

7. SessionAttributes注解
作用：用于多次执行控制器方法间的参数共享，注解在类上
属性：
    value：用于指定存入名称
    type：用于指定存入数据类型
```java
@RequestMapping("/anno")
@Controller
@SessionAttributes(value = {"msg"})
public class AnnoController {
//在Session中存储值
    @RequestMapping("/SessionAttributes")
    public String testAnno05(Model model) {
        model.addAttribute("msg","abc");
        return "session";
    }
    //获取session中的值
    @RequestMapping("/getSessionAttributes")
    public String testAnno06(ModelMap modelMap) {
        String msg = (String) modelMap.get("msg");
        System.out.println(msg);
        return "success";
    }
    //删除session中的值
    @RequestMapping("/delSessionAttributes")
    public String testAnno07(SessionStatus status) {
        status.setComplete();
        return "success";
    }
}
```

## 六、springMVC的数据响应
#### 有返回值的
```Java
@RequestMapping("/testString")
    public String testString(Model model) {
        User user = new User();
        user.setUsername("aaa");
        user.setPassword("123");
        user.setAge(11);
        model.addAttribute(user);
        System.out.println("testString");
        return "success";
    }
    
```
#### 无返回值的
```Java
@RequestMapping("/testVoid")
    public void testVoid(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("testVoid");
        //请求转发
        //request.getRequestDispatcher("/WEB-INF/pages/success.jsp").forward(request,response);
        //重定向
        //response.sendRedirect(request.getContextPath()+"/response.jsp");
        //直接响应
        response.getWriter().println("abc");
        return;
    }
```

#### 返回值是ModelAndView
```Java
@RequestMapping("testModelAndView")
    public ModelAndView testModelAndView(){
        ModelAndView mv = new ModelAndView();
        User user = new User();
        user.setUsername("aaa");
        user.setPassword("123");
        user.setAge(11);
        //把user存储到ModelAndView
        mv.addObject("user",user);
        //指定跳转页面
        mv.setViewName("success");
        return mv;
    }
```

#### ResponseBody响应json数据
##### 1. 过滤静态资源
1. DispatcherServlet会拦截到所有的资源，导致一个问题就是静态资源（img、css、js）也会被拦截到，从而不能被使用。解决问题就是需要配置静态资源不进行拦截，在springmvc.xml配置文件添加如下配置
    1. mvc:resources标签配置不过滤
    2. location元素表示webapp目录下的包下的所有文件
    3. mapping元素表示以/static开头的所有请求路径，如/static/a 或者/static/a/b

```xml
<!-- 设置静态资源不过滤 -->
    <mvc:resources location="/css/" mapping="/css/**"/>  <!-- 样式 -->
    <mvc:resources location="/images/" mapping="/images/**"/>  <!-- 图片 -->
    <mvc:resources location="/js/" mapping="/js/**"/>  <!-- javascript -->
```
##### 2. 使用@RequestBody获取请求体数据
```html
<script src="js/jquery.min.js"></script>
    <script>
        $(function () {
            $("#btn").click(function () {
                // alert("abc");
                //发送Ajax请求
                $.ajax({
                    url: "user/testAjax",
                    contentType: "application/json;charset=utf-8",
                    data: '{"username":"abc","password":"123","age":11}',
                    dataType: "json",
                    type:"post",
                    //成功后的回调函数
                    success:function () {
                    }
                });
            });
        });
    </script>

```
```java
@RequestMapping("/testAjax")
    public void testAjax(@RequestBody String body){
        System.out.println(body);
    }
```
##### 3. 使用@RequestBody注解把json的字符串转换成JavaBean的对象
1. 需要的依赖
```xml
<dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.0</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.9.0</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.9.0</version>
    </dependency>
```
2. 要求方法需要返回JavaBean的对象
```java
@RequestMapping("/testAjax")
    //使用ResponseBody注解把对象转换为json对象响应
    public @ResponseBody User testAjax(@RequestBody User user){
        System.out.println("testAjax");
        System.out.println(user);
        user.setUsername("aaa");
        user.setPassword("111");
        return user;
    }
```
```html
<script>
        $(function () {
            $("#btn").click(function () {
                // alert("abc");
                //发送Ajax请求
                $.ajax({
                    url: "user/testAjax",
                    contentType: "application/json;charset=utf-8",
                    data: '{"username":"abc","password":"123","age":11}',
                    dataType: "json",
                    type:"post",
                    success:function (data) {
                    //获取到从Controller发送的json对象
                        alert(data);
                        alert(data.username);
                        alert(data.password);
                        alert(data.age);
                    }
                });
            });
        });
    </script>
```

## 七、springMVC的文件上传
###  1. SpringMVC传统方式文件上传
1. 导入坐标

```xml
<dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.1</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.4</version>
        </dependency>
    </dependencies>
```

2. SpringMVC框架提供了MultipartFile对象，该对象表示上传的文件，要求变量名称必须和表单file标签的
name属性名称相同。
3. 代码
```Java
@RequestMapping("/fileUpload2")
    public String fileUpload2(HttpServletRequest request, MultipartFile upload) throws Exception {
        System.out.println("fileUpload");
        //上传位置
        String path = request.getSession().getServletContext().getRealPath("/upload/");
        //判断该路径是否存在
        File file = new File(path);
        if (!file.exists()) {
            //创建该文件夹
            file.mkdirs();
        }
        //获取文件名称
        String filename = upload.getOriginalFilename();
        //把文件名称变成唯一值
        String uuid = UUID.randomUUID().toString().replace("-", "");
        filename = uuid + "-" + filename;
        //完成文件上传
        upload.transferTo(new File(path, filename));
        return "success";
    }
```


 jsp页面
> <form action="/user/fileUpload2" method="post" enctype="multipart/form-data">
>    选择文件:<input type="file" name="upload"/><br>
> <input type="submit" value="上传"><br>    


4. 配置文件解析器对象
```xml
<!-- 配置文件解析器对象，要求id名称必须是multipartResolver -->
    <bean id="multipartResolver" 
class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize" value="10485760"/>
    </bean>
```



### 2. SpringMVC跨服务器方式文件上传
1. 搭建资源服务器
2. 实现SpringMVC跨服务器方式文件上传

* 导入开发需要的jar包    
```xml
      <dependency>
          <groupId>com.sun.jersey</groupId>
          <artifactId>jersey-core</artifactId>
          <version>1.18.1</version>
      </dependency>
      <dependency>
          <groupId>com.sun.jersey</groupId>
          <artifactId>jersey-client</artifactId>
          <version>1.18.1</version>
      </dependency>
```

* 编写文件上传的JSP页面    
```html
<h3>跨服务器的文件上传</h3>    
    <form action="user/fileupload3" method="post" enctype="multipart/form-data">
        选择文件：<input type="file" name="upload"/><br/>
        <input type="submit" value="上传文件"/>
    </form>
```
* 编写控制器    
```java
@RequestMapping("/fileUpload3")
    public String fileUpload3(MultipartFile upload) throws Exception {
        System.out.println("fileUpload3");
        //定义上传文件服务器路径
        String path = "http://localhost:9090/uplods/";
        //获取文件名称
        String filename = upload.getOriginalFilename();
        //把文件名称变成唯一值
        String uuid = UUID.randomUUID().toString().replace("-", "").toUpperCase();
        filename = uuid + "-" + filename;
        //创建客户端对象
        Client client = Client.create();
        //和服务器连接
        WebResource webResource = client.resource(path + filename);
        //跨服务器文件上传
        webResource.put(upload.getBytes());
        return "success";
    }
```

## 八、springMVC的异常处理
1. 异常处理思路
Controller调用service，service调用dao，异常都是向上抛出的，最终有DispatcherServlet找异常处理器进行异常的处理。

2. SpringMVC的异常处理
* 自定义异常类
```java
public class SysException extends Exception {
    //提示信息
    private String msg;
    public SysException(String msg) {
        this.msg = msg;
    }
    public String getMsg() {
        return msg;
    }
    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```
* 自定义异常处理器，实现HandlerExceptionResolver接口，当程序出现异常就会调用该方法。
```java 
public class SysExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest,HttpServletResponse httpServletResponse, Object o, Exception ex) {
        SysException e = null;
        if (ex instanceof SysException){
            e = (SysException)ex;
        } else {
            e = new SysException("错误01");
        }
        //创建ModelAndView对象
        ModelAndView mv = new ModelAndView();
        mv.addObject("msg",e.getMsg());
        //指定跳转位置
        mv.setViewName("error");
        return mv;
    }
}
```

*  配置异常处理器
   
```xml
<bean id="sysExceptionResolver" class="cn.luming.exception.SysExceptionResolver"></bean>
```


### SpringMVC框架中的拦截器
1. 拦截器的概述
    1. SpringMVC框架中的拦截器用于对处理器进行预处理和后处理的技术。
    2. 可以定义拦截器链，连接器链就是将拦截器按着一定的顺序结成一条链，在访问被拦截的方法时，拦截器链中的拦截器会按着定义的顺序执行。
    3. 拦截器和过滤器的功能比较类似，有区别
        1. 过滤器是Servlet规范的一部分，任何框架都可以使用过滤器技术。
        2. 拦截器是SpringMVC框架独有的。
        3. 过滤器配置了/*，可以拦截任何资源。
        4. 拦截器只会对控制器（Controller）中的方法进行拦截。
    4. 拦截器也是AOP思想的一种实现方式
    5. 想要自定义拦截器，需要实现HandlerInterceptor接口。

2. 自定义拦截器步骤

    1. 创建类，实现HandlerInterceptor接口，重写需要的方法

        ```java
        public class MyInterceptor01 implements HandlerInterceptor {
            @Override
            /**
                拦截Controller方法
                return true 放行
            */
            public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            /**
                拦截不放行，自己指定跳转页面
                request.getRequestDispatcher("/index.jsp").forward(request,response);
                return false;
                */
                System.out.println("MyInterceptor01");
                return true;
            }
        }
        ```

    2. 在springmvc.xml中配置拦截器类
       
        ```xml
        <!-- 配置拦截器 -->
            <mvc:interceptors>
                <mvc:interceptor>
                    <!-- 要拦截的方法 -->
                    <mvc:mapping path="/user/*"/>
                    <!-- 不要拦截的方法
                    <mvc:exclude-mapping path=""/>
                    -->
                    <!-- 配置拦截器对象 -->
                    <bean class="cn.luming.interceptor.MyInterceptor01"></bean>
                </mvc:interceptor>
            </mvc:interceptors>
        ```
        
    3. HandlerInterceptor接口其他两个方法
        1. postHandle：在Controller方法执行完后再拦截
        2. afterCompletion：Controller执行完后跳转页面才会执行，最后执行

    4. 配置多个拦截器按配置的顺序执行



​        