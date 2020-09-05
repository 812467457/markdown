# SpringBoot

## 一、SpringBoot简介

### 1、SpringBoot的优势

1. 配置简单：SpringBoot是对spring的进一步封装，基于注解开发，如果需要配置文件，使用的也是yml和properties，舍弃了xml。
2. 独立运行：每个工程都可以单独打成一个jar包，其中内置tomcat或其他servlet容器，可以独立运行，最适合于微服务开发。
3. 启动器：每个特定场景下的需求都封装成了一个starter，导入starter就可以使用这个场景下所需的一切，包括自动化配置和jar包。



## 二、SpringBoot入门案例

### 1、创建SpringBoot项目

直接创建SpringBoot项目，勾选所需要的starter，项目创建完成后自动导入相对应的jar包

![image-20200701192821235](http://picture.youyouluming.cn/image-20200701192821235.png)

pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

### 2、创建启动类

SpringBoot项目会有一个启动类，也就是程序主入口。

HelloSpringBootApplication.java

```Java
@SpringBootApplication	//表示当前类为SpringBoot应用主启动类
public class HelloSpringBootApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloSpringBootApplication.class, args);
    }
}
```

写一个Handler类测试环境搭建是否成功

```Java
@RestController
public class HelloHandler {
    @RequestMapping("/get/spring/boot/message")
    public String getMessage(){
        return "ok!";
    }
}
```

注意：在springBoot环境下，主启动类所在包和所在子包都会被自动扫描到。

![image-20200701225906070](http://picture.youyouluming.cn/image-20200701225906070.png)

或者手动配置，在主启动类上加@ComponetScan注解可以指定扫描的包，但约定规则会失效。

## 三、SpringBoot配置文件

### 1、properties配置文件

application.properties

```properties
#配置端口号
server.port=8081
#配置访问路径
server.servlet.context-path=/test
```

### 2、yml配置文件

#### 2.1、yml基本语法

application.yml

```yml
server:
  #端口号
  port: 8081
  servlet:
    #访问路径
    context-path: /test
```

必须使用空格缩进，缩进的数量每一层级的都要一样，区分大小写

注意：不管是properties还是yml配置文件，都必须叫application，否则spring Boot不能识别。

#### 2.2、yml数据类型

* 字面量：字符串、数值、布尔值。注意如果数据库密码是0开头，数据库密码需要加上引号，否则springboot按照8进制数据解析。
* 对象、map
* 数组

#### 2.3、yml测试

使用yml配置文件给实体类中的属性赋值

实体类

```java
@Component  //先把当前实体类放到IOC容器中，才能读取yml配置文件的数据
@ConfigurationProperties(prefix = "student")    //表示和配置文件对应，读取配置文件中以student开头的数据
public class Student {
    private Integer stuId;
    private String name;
    private Boolean graduated;
    private String[] subject;
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birthday;
    private Map<String,String> teachers;
    private Address address;
    get...
    set...    
}
```

需要加一个依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

yml配置文件

```properties
student:
  stu-id: 5
  name: jack
  graduated: false
  subject:
    - java
    - mysql
  birthday: 2020-02-02
  teachers:
    java: tom
    mysql: marry
  address:
    province: abc
    city: def
    street: fgh
```

也可以使用@Value注解读取yml中的数据

yml

```properties
this.is.a.message: "hello world"
```

![image-20200702142918289](http://picture.youyouluming.cn/image-20200702142918289.png)

也可以为msg属性赋值

#### 2.4、yml设置日志打印级别

```properties
#全局打印：所有的日志信息都以debug级别打印
#logging:
#  level:
#    root: debug

#局部打印：指定某个包下以debug级别打印
logging:
  level:
    cn:
      yylm:
        springboot: debug
```



## 四、SpringBoot相关注解

### 1、spring注解

#### 2.1、@Configuration注解

表示当前类为配置类

```Java
@Configuration
public class SpringAnnotationConfig {
}

```

#### 2.2、@Bean注解

相当于xml中的bean标签，把对象放到IOC容器管理

```java
@Configuration
public class SpringAnnotationConfig {
    @Bean
    public Student getStudent(){
        return new Student();
    }
}
```

#### 2.3、@Import注解

和@Bean注解类似，也是把一个对象放到IOC容器里，但是比@Bean更方便

```Java
@Configuration
@Import(Student.class)
public class SpringAnnotationConfig {

}
```

#### 2.4、@Conditional*注解

一个类在满足条件时才加入IOC容器

#### 2.5、@ComponentScan注解

指定IOC扫描的包，相当于在xml中配置的context:component-scan

```java
@ComponentScan(
        value = "cn.yylm.springboot",
        //取消默认规则
        useDefaultFilters = false,
        //自定义扫描规则
        includeFilters = {
                //指定只扫描Controller注解
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Controller.class)
        },
        //排除条件
        excludeFilters = {
                //排除service注解
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Service.class)
        })
public class SpringAnnotationConfig {

}
```

### 2、SpringBoot注解

#### 2.1、@SpringBootApplication注解

@Configuration注解SpringBoot版，表示当前为配置类

#### 2.2、@EnableAutoConfiguration注解

可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。

#### 2.3、@AutoConfigurationPackage注解

指定自动化配置的包，将主类所在包和它的自爆中所有组件都扫描的IOC容器中。

#### 2.4、@SpringBootApplication注解
表示当前类为SpringBoot应用，包含前面三个注解。



## 五、SpringBoot整合其他组件

### 1、整合Mybatis_xml版本

#### 1.1、引入started

![image-20200702182458780](http://picture.youyouluming.cn/image-20200702182458780.png)

#### 1.2、配置数据库信息

application.yml

```properties
#配置数据源
spring:
  datasource:
    username: root
    password: admin
    url: jdbc:mysql://localhost:3306/project_crowd?serverTimezone=GMT%2B8&useSSL=false&useUnicode=true&characterEncoding=UTF-8
    driver-class-name: com.mysql.cj.jdbc.Driver
#mybatis配置
mybatis:
  config-location: classpath:/mybatis/mybatis-config.xml
  mapper-locations: classpath:/mybatis/mapper/*.xml
```

#### 1.3、配置mapper信息

AdminMapper.java

```java
public interface AdminMapper {
    Admin selectByPrimaryKey(Integer id);
}
```

AdminMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="cn.yylm.springboot.mapper.AdminMapper">
    <resultMap id="BaseResultMap" type="cn.yylm.springboot.entity.Admin">
        <id column="id" property="id" jdbcType="INTEGER"/>
        <result column="login_acct" property="loginAcct" jdbcType="VARCHAR"/>
        <result column="user_pswd" property="userPswd" jdbcType="CHAR"/>
        <result column="user_name" property="userName" jdbcType="VARCHAR"/>
        <result column="email" property="email" jdbcType="VARCHAR"/>
        <result column="create_time" property="createTime" jdbcType="CHAR"/>
    </resultMap>
    <select id="selectByPrimaryKey" resultMap="BaseResultMap">
        select * from t_admin where id=#{id}
    </select>
</mapper>
```

#### 1.4配置Handler

```java
@Controller
public class AdminHandler {
    @Autowired
    private AdminService adminService;
    
 	@ResponseBody
    @RequestMapping("/getAdminById/{id}")
    public Admin getAdminById(@PathVariable("id") Integer id) {
        return adminService.getAdminById(id);
    }
}
```

 #### 1.5、主程序

```java
@MapperScan("cn.yylm.springboot.mapper")    //扫描mapper所在的包
@SpringBootApplication
public class SpringbootMybatisXmlApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisXmlApplication.class, args);
    }
}
```

运行结果

![image-20200702200709465](http://picture.youyouluming.cn/image-20200702200709465.png)

#### 1.6、声明式事务

在主程序上加@EnableTransactionManagement注解

```
@EnableTransactionManagement    //启动声明式事务
@MapperScan("cn.yylm.springboot.mapper")    //扫描mapper所在的包
@SpringBootApplication
public class SpringbootMybatisXmlApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisXmlApplication.class, args);
    }
}
```

然后再需要使用事务的service类里的方法上或类上加@Transactional，并且设置隔离级别、传播行为和回滚等。

```Java
@Transactional
@Override
public Admin getAdminById(Integer id) {
    return adminMapper.selectByPrimaryKey(id);
}
```

### 2、整合Mybatis_注解版

把xml文件都删除掉，mapper接口直接使用注解查询

```Java
public interface AdminMapper {

    //实体类和数据库表字段不一致时，使用results映射
    @Results(id = "ResultMap", value = {
            @Result(property = "id", column = "id", id = true, jdbcType = JdbcType.INTEGER),
            @Result(property = "loginAcct", column = "login_acct", jdbcType = JdbcType.VARCHAR),
            @Result(property = "userPswd", column = "user_pswd", jdbcType = JdbcType.CHAR),
            @Result(property = "userName", column = "user_name", jdbcType = JdbcType.VARCHAR),
            @Result(property = "email", column = "email", jdbcType = JdbcType.VARCHAR),
            @Result(property = "createTime", column = "create_time", jdbcType = JdbcType.VARCHAR)
    })
    @Select(value = "select * from t_admin where id = #{id}")
    Admin selectByPrimaryKey(Integer id);
}
```

### 3、整合druid

#### 3.1、导入依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.5</version>
</dependency>
```

#### 3.2、配置数据源

```properties
#配置数据源
spring:
  datasource:
    username: root
    password: admin
    url: jdbc:mysql://localhost:3306/project_crowd?serverTimezone=GMT%2B8&useSSL=false&useUnicode=true&characterEncoding=UTF-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
#日志
logging:
  level:
    cn.yylm.springboot.mapper: debug
```

#### 3.3、集成durid监控系统

如果使用监控系统，就不能再用配置文件去配置数据库连接池了，使用配置类配置。

DruidDataSourceConfig.java

```Java
@SpringBootConfiguration
public class DruidDataSourceConfig {
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource dataSource() throws SQLException {
        DruidDataSource dataSource = new DruidDataSource();
        //配置监控统计拦截的Filters
        dataSource.setFilters("stat");
        return dataSource;
    }

    @Bean
    public ServletRegistrationBean statViewServlet() {
        //监控系统的登录路径
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String, String> initParams = new HashMap<>();
        //监控系统的登录账号密码
        initParams.put("loginUsername", "admin");
        initParams.put("loginPassword", "123456");
        initParams.put("allow", "");// 默认就是允许所有访问
        initParams.put("deny", "192.168.15.21");//拒绝哪个ip访问
        bean.setInitParameters(initParams);
        return bean;
    }
    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter() {
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        Map<String, String> initParams = new HashMap<>();
        initParams.put("exclusions", "*.js,*.css,/druid/*");//排除过滤
        bean.setInitParameters(initParams);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}
```

### 4、整合web三大组件

#### 4.1、Listener

```java
@WebListener
public class HelloListener implements ServletContextListener {
    @Override
    public void contextDestroyed(ServletContextEvent arg0) {
        System.out.println("应用销毁了....HelloListener");
    }

    @Override
    public void contextInitialized(ServletContextEvent arg0) {
        System.out.println("应用启动了....HelloListener");
    }
}
```

#### 4.2、Filter

```Java
@WebFilter(urlPatterns = "/*")
public class HelloFilter implements Filter {

    @Override
    public void destroy() {

    }

    @Override
    public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2)
            throws IOException, ServletException {
        System.out.println("HelloFilter............放行之前");
        arg2.doFilter(arg0, arg1);
        System.out.println("HelloFilter............放行之后");
    }

    @Override
    public void init(FilterConfig arg0) throws ServletException {

    }

}
```

#### 4.3、Servlet

```Java
@WebServlet(urlPatterns = "/my")
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException, IOException {
        resp.getWriter().write("MyServlet do.......");
    }

}
```

#### 4.4、主程序

主程序需要加一个ServletComponentScan  注解

```Java
@ServletComponentScan       //主要扫描web三大组件
@EnableTransactionManagement    //启动声明式事务
@MapperScan("cn.yylm.springboot.mapper")    //扫描mapper所在的包
@SpringBootApplication
public class SpringbootMybatisXmlApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisXmlApplication.class, args);
    }
}
```

### 5、整合Redis

#### 5.1、依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

#### 5.2、配置文件

```properties
spring:
  redis:
    host: localhost
    port: 6379
```

#### 5.3、使用RedisTemplate存储数据

```Java
@SpringBootTest
class SpringBootRedisApplicationTests {
    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    Logger logger = LoggerFactory.getLogger(SpringBootRedisApplicationTests.class);

    @Test
    void contextLoads() {
        //获取用来操作String类型数据的ValueOperations对象
        ValueOperations<String, String> operations = redisTemplate.opsForValue();
        String key = "name";
        String value = "jack";
        operations.set(key, value);
        String getKey = operations.get(key);
        logger.info(getKey);
    }

}
```

使用RedisTemplate会有一个泛型的问题，如果指定的泛型是Object，存到redis中会进行序列化，导致字符串看不懂。

#### 5.4、使用StringRedisTemplate

```Java
@SpringBootTest
class SpringBootRedisApplicationTests {
    Logger logger = LoggerFactory.getLogger(SpringBootRedisApplicationTests.class);

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test01() {
        ValueOperations<String, String> operations = stringRedisTemplate.opsForValue();
        String key = "name";
        String value = "abc";
        operations.set(key, value);
        String getKey = operations.get(key);
        logger.info(getKey);

    }
}    
```

StringRedisTemplate是SpringBoot提供的一个Template，继承原生的RedisTemplate，对其进行了封装，不必考虑泛型的问题，都是String类型。

### 6、整合Thymeleaf

#### 6.1、Thymeleaf

Thymeleaf是一种视图模板技术，之前使用的JSP也是视图模板技术的一种，帮助我们实现服务器端渲染的。SpringBoot推荐使用Thymeleaf作为视图模板技术。

服务器端渲染：执行动态代码，将其计算得到具体值。

JSP不能直接在浏览器上查看布局、动态效果等。Thymeleaf虽然也包含动态的效果，但是不影响原本html标签的显示。

#### 6.2、依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

#### 6.3、配置

```properties
spring:
  thymeleaf:
    #前缀
    prefix: classpath:/templates/
    #后缀
    suffix: .html
    #开发的时候禁用缓存
    cache: false
```

#### 6.4、测试

handler

```Java
@Controller
public class TemplatesHandler {
    @RequestMapping("/test/thymeleaf")
    public String testThymeleaf() {
        return "success";
    }
}
```

success.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org" >
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h3>替换标签体</h3>
<p th:text="启动服务器才能看见的内容">不启动服务器，直接打开html就可以看见</p>
<h3>替换属性值</h3>
<label>
    <input type="text" value="直接打开html看到的value" th:value="启动服务器后看到的value">
</label>
</body>
</html>
```

在html上加`xmlns:th="http://www.thymeleaf.org" `名称空间，然后使用th:xxx就可以显示动态数据了，把之前的值替换掉。没有启动服务器也可以打开这个html标签，看到静态数据。

### 7、Thymelea语法

 #### 7.1、访问属性域

handler

```java
@Autowired
private ServletContext servletContext;
@RequestMapping("/test/thymeleaf")
public String testThymeleaf(Model model, HttpSession session) {
    //请求域
    model.addAttribute("model","modelValue");
    //会话域
    session.setAttribute("session","sessionValue");
    //应用域
    servletContext.setAttribute("servletContext","servletContextValue");
    return "success";
}
```

html

```html
<h3>访问属性域</h3>
<p th:text="${model}">访问请求域</p>
<p th:text="${session.session}">访问会话域</p>
<p th:text="${application.servletContext}">访问application域</p>
```

#### 7.2、获取contextPath值

```html
<h3>获取contextPath值</h3>
<p th:text="@{abc/def/qwe}">@{}把contextPath的值添加到指定地址前</p>
```

#### 7.3、直接执行表达式

```html
<h3>直接执行表达式</h3>
<p>[[${session.session}]]</p>
```

#### 7.4、执行分支和迭代

```html
<h3>循环测试</h3>
<div>
    <!-- th:each="str: ${list}"遍历 list 取出每个str th:text="${str} 使用str的值替换标签原来的值 -->
    <p th:text="${str}" th:each="str: ${list}">遍历后的值替换这里的值</p>
</div>
```

#### 7.5、引入外部文件

被引用的html

```html
<div th:fragment="myPart" xmlns:th="http://www.w3.org/1999/xhtml">
    <p>引入的外部html</p>
</div>
```

使用引入文件的html

```
<h3>引入外部文件</h3>
<div th:insert="~{include/part::myPart}"></div>
<div th:replace="~{include/part::myPart}"></div>
<div th:include="~{include/part::myPart}"></div>
```

~{用来找到文件，会去拼前后缀  :: 被引用的th:fragment的值}

insert：插入到当前标签内部

replace：替换当前标签

include：表标签内的值替换