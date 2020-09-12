

# SpringCloud

## 一、介绍

### 1、Spring Cloud核心

SpringCloud是分布式系统的整体解决方案，利用了SpringBoot的开发便利性简化了分布式系统基础设施开发。SpringCloud是基于http协议的

### 2、SpringCloud组件

* 注册中心：Eureka
* 客户端负载均衡：Ribbon
* 声明式远程方法调用：Fegin
* 服务降级、熔断：Hystrix
* 网关：Zuul
* 数据监控：Hystrix Dash Board



## 二、基本环境搭建和测试

### 1、父工程

创建一个pom方式打包的父工程

管理依赖

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Greenwich.SR2</version>
            <type>pom</type>
            <!--import依赖范围表示将spring-cloud-dependencies包中的依赖信息导入-->
            <scope>import</scope>
        </dependency>
        <!-- 导入SpringBoot需要使用的依赖信息-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.1.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 2、common工程(通用)

暂时只有一个实体类

```Java
public class Employee {
    private Integer empId;
    private String empName;
    private Double empSalary;
    get...
    set...
}
```

### 3、provder工程(提供服务)

是一个springBoot工程

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>cn.yylm</groupId>
        <artifactId>spring-cloud-common</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

加一个主启动类

```Java
@SpringBootApplication
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

配置文件

```properties
server:
  port: 8081
```

Handler

```Java
@RestController	//因为这个是提供服务的Handler，以后一定是Java程序调用，所以直接返回JSON数据
public class EmployeeHandler {
    @RequestMapping("/provder/get/emloyee/remote")
    public Employee getEmployee() {
        return new Employee(10, "jack", 999.99);
    }
}
```

### 4、consumer工程(消费者)

是一个springBoot工程

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-cloud-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>cn.yylm</groupId>
        <artifactId>spring-cloud-common</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

加一个主启动类

```Java
@SpringBootApplication
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

创建配置类提供RestTemplate用来获取JSON数据

```Java
@SpringBootConfiguration
public class SpringCloudConfig {
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

handler

```Java
@RestController
public class HumanResourceHandler {
    @Resource
    private RestTemplate restTemplate;
    @RequestMapping("/")
    public Employee getEmployeeRemote() {
        //声明远程服务的主机地址加端口号
        String host = "http://localhost:8081";
        //声明具体调用的功能的url地址
        String url = "/provder/get/emloyee/remote";
        //通过RestTemplate调用远程服务
        return restTemplate.getForObject(host + url, Employee.class);
    }
}
```

配置文件

```properties
server:
  port: 9081
```

## 三、创建Eureka注册中心

### 1、搭建

依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

主启动类

```Java
@EnableEurekaServer     //启用Eureka服务器端功能
@SpringBootApplication
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

配置

```properties
server:
  port: 7011
eureka:
  instance:
    #实例主机地址
    hostname: localhost
  #从客户端角度配置
  client:
    #不需要注册，自己就是注册中心
    register-with-eureka: false
    #从注册中心取回信息
    fetch-registry: false
    #客户端访问eureka的地址
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
```

### 2、注册provider

在provider的pom加入依赖，表示是一个客户端

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

在provider的配置文件配置

```properties
server:
  port: 8081
eureka:
  #配置当前微服务作为eureka的客户端访问eureka服务器端时使用的地址
  client:
    service-url:
      defaultZone: http://localhost:7011/eureka
#指定这个微服务的名称，在注册中心可以找到这个微服务
spring:
  application:
    name: SPRING-CLOUD-PROVIDER
```

### 3、通过ribbon访问eureka

在consumer工程下加入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

修改consumer配置

```properties
server:
  port: 9081
spring:
  application:
    name: SPRING-CLOUD-CONSUMER
eureka:
  client:
    #配置当前微服务作为eureka的客户端
    service-url:
      defaultZone: http://localhost:7011/eureka
```

在RestTemplate方法上加一个注解

```Java
@LoadBalanced   //让RestTemplate有负载均衡的功能，可以通过ribbon访问Provider集群
@Bean
public RestTemplate getRestTemplate(){
    return new RestTemplate();
}
```

在handler中用provider微服务的名字代替原来的端口号

![image-20200704115737329](http://picture.youyouluming.cn/image-20200704115737329.png)

###  4、ribbon测试

测试ribbon的负载均衡是否起作用

复制一个provider微服务，端口号为8082

![image-20200704121618902](http://picture.youyouluming.cn/image-20200704121850882.png)

在provider的handler中获取当前微服务的端口号，以便查看是否有负载均衡

![image-20200704121850882](http://picture.youyouluming.cn/image-20200704124826312.png)

启动所需的微服务

![image-20200704124826312](http://picture.youyouluming.cn/image-20200704121903233.png)

结果

![image-20200704121903233](http://picture.youyouluming.cn/image-20200704173145839.png)

![image-20200704121910397](http://picture.youyouluming.cn/image-20200704121910397.png)

## 四、Feign（远程调用）

### 1、作用

Spring Cloud Feign帮助我们定义和实现依赖服务接口的定义。在Spring Cloud feign的实现下，只需要创建一个接口并用注解方式配置它，即可完成服务提供方的接口绑定，简化了在使用Spring Cloud Ribbon时自行封装服务调用客户端的开发量。

### 2、common

创建接口被其他服务调用

加入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>
```

创建远程调用方法的接口

```Java
//表示当前接口和一个provider对应，value指定要调用provider微服务的名称
@FeignClient(value = "SPRING-CLOUD-PROVIDER")
public interface EmployeeRemoteService {
    //远程调用的接口，RequestMapping映射地址必须一致，方法声明必须一致
    @RequestMapping("/provder/get/emloyee/remote")
    public Employee getEmployee();
}
```

### 3、consumer

controller

```Java
@RestController
public class FeignHumanResourceHandler {
    //装配调用远程服务的接口，后面就可以正常调用该方法了
    @Resource
    private EmployeeRemoteService employeeRemoteService;

    @RequestMapping("/feign/consumer/get/emp")
    public Employee getEmployeeRemote() {
        return employeeRemoteService.getEmployee();
    }

}
```

配置文件

```properties
server:
  port: 7000
spring:
  application:
    name: SPRING-CLOUD-FEIGN-CONSUMER
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7011/eureka
```

### 4、调用流程

浏览器 ---> consumer通过common配置的负载均衡 ---> provider提供服务

### 5、Feign传参问题

common和provider方法参数上都需要加上这个注解才能正常传递参数

![image-20200704173145839](http://picture.youyouluming.cn/image-20200704220028714.png)

## 五、Hystrix(熔断、降级)

### 1、作用

在微服务架构下，存在很多服务之间的调用，如果某个服务出现了故障，那么所有调用这个服务的服务都会无法正常运行，最终导致整个服务崩溃。这样的现象叫做“雪崩”。

为了避免这样的情况发生，springcloud使用了熔断和降级来保证服务运行的稳定性。熔断和降级都属于一种类似开关装置，在某个服务发生故障后，通过熔断监控，向调用者返回一个备用响应方案，而不是长时间的等待或者抛出异常，这样就保证了服务调用方的线程不会被长时间占用，避免了雪崩的出现。熔断主要是提供服务的provider，降级主要是调用服务的consumer。



### 2、使用熔断

provider工程添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-javanica</artifactId>
</dependency>
```

启动类上加一个注解

```Java
@EnableCircuitBreaker   //开启断路器功能
@SpringBootApplication
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

使用一个ResultEntity来处理所有的返回结果

在common创建这个类，所有服务都可能使用到该类

```Java
/**
 * 整个项目统一使用这个类型作为Ajax请求或远程方法调用返回响应的数据格式
 */
public class ResultEntity<T> {
    public static final String SUCCESS = "SUCCESS";
    public static final String FAILED = "FAILED";
    public static final String NO_MESSAGE = "NO_MESSAGE";
    public static final String NO_DATA = "NO_DATA";
	// 操作成功，不需要返回数据
    public static ResultEntity<String> successWithoutData() {
        return new ResultEntity<String>(SUCCESS, NO_MESSAGE, NO_DATA);
    }
    // 操作成功，需要返回数据
    public static <E> ResultEntity<E> successWithData(E data) {
        return new ResultEntity<>(SUCCESS, NO_MESSAGE, data);
    }
    // 操作失败，返回错误消息
    public static <E> ResultEntity<E> failed(String message) {
        return new ResultEntity<>(FAILED, message, null);
    }
    private String result;
    private String message;
    private T data;

    public ResultEntity() {
    }
    public ResultEntity(String result, String message, T data) {
        super();
        this.result = result;
        this.message = message;
        this.data = data;
    }
    @Override
    public String toString() {
        return "ResultEntity[result=" + result + ",message=" + message + ",data=" + data + "]";
    }
    public String getResult() {
        return result;
    }
    public void setResult(String result) {
        this.result = result;
    }
    public String getMessage() {
        return message;
    }
    public void setMessage(String message) {
        this.message = message;
    }
    public T getData() {
        return data;
    }
    public void setData(T data) {
        this.data = data;
    }
}
```

provider工程的handler

```Java
@RestController
public class EmployeeHandler {
    Logger logger = LoggerFactory.getLogger(EmployeeHandler.class);

    @HystrixCommand(fallbackMethod = "getEmpWithCircuitBreakerBackUp")     //指定当前方法熔断后的调用方法
    @RequestMapping("/provider/get/emp/circuit/breaker")
    public ResultEntity<Employee> getEmpWithCircuitBreaker(@RequestParam("signal") String signal) throws InterruptedException {
        //制造异常模拟一个“雪崩”
        if ("bang".equals(signal)) {
            throw new RuntimeException();
        }
        //模仿超时
        if ("sleep".equals(signal)) {
            Thread.sleep(5000);
        }
        return ResultEntity.successWithData(new Employee(10, "jack ", 999.99));
    }
    public ResultEntity<Employee> getEmpWithCircuitBreakerBackUp(@RequestParam("signal") String signal){
        String message = "执行断路方法" + signal;
        return ResultEntity.failed(message);
    }
}
```

### 3、使用降级

服务降级是在客户端（consumer）完成的，当consumer访问一个provider却得不到一个响应时，就执行预先设点好的一个备用解决方案

在common工程加入依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

在接口中加入

```java
@RequestMapping("/provider/get/emp/circuit/breaker")
public ResultEntity<Employee> getEmpWithCircuitBreaker(@RequestParam("signal") String signal);
```

在common工程创建一个实现了FallbackFactory接口的类

```Java
/**
 * 实现Consumer端服务降级功能
 * 实现FallbackFactory接口时要传入@FeignClient注解标记的接口类型
 * 在create()方法中返回@FeignClient注解标记的接口类型的对象，当Provider调用失败后，执行该方法
 */
@Component      //将当前类注入IOC容器
public class MyFallBackFactory implements FallbackFactory<EmployeeRemoteService> {

    @Override
    public EmployeeRemoteService create(Throwable throwable) {
        return new EmployeeRemoteService() {
            @Override
            public Employee getEmployee() {
                return null;
            }

            @Override
            public List<Employee> getEmployeeRemote(String keyword) {
                return null;
            }

            @Override
            public ResultEntity<Employee> getEmpWithCircuitBreaker(String signal) {
                return ResultEntity.failed(throwable.getMessage());
            }
        };
    }
}
```

 在@FeignClient注解上加上指定的fallbackFactory

![image-20200704215857231](http://picture.youyouluming.cn/image-20200704215857231.png)

在consumer的配置中启用降级功能

![image-20200704220028714](http://picture.youyouluming.cn/image-20200705115338950.png)

在consumer的handler调用一下接口的方法

```java
@RequestMapping("/feign/comsumer/test/fallback")
public ResultEntity<Employee> testFallBack(@RequestParam("signal") String signal) {
    return employeeRemoteService.getEmpWithCircuitBreaker(signal);
}
```

### 4、监控系统

在provider加入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

provider配置文件

```properties
management:
  endpoints:
    web:
      exposure:
        include: hystrix.
```

创建一个监控工程spring-cloud-dashboard

加入依赖

```xml
<dependency>    
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

主启动类

```Java
@EnableHystrixDashboard     //启动仪表盘监控功能
@SpringBootApplication
public class DashboardApplication {
    public static void main(String[] args) {
        SpringApplication.run(DashboardApplication.class, args);
    }
}
```

配置文件

```properties
server:
  port: 8000
spring:
  application:
    name: SPRING-CLOUD-DASHBOARD
```

## 六、Zuul（网关）

### 1、作用

Zuul包含了对请求的路由和过滤两个主要功能

其中路由功能负责将外部请求转发到具体微服务实例上，是实现外部访问统一入口的基础。

过滤功能主要是负责对请求的处理过程进行干预，实现请求校验、服务聚合等功能的基础。

### 2、创建Zuul工程

加入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
</dependencies>
```

配置文件

```properties
server:
  port: 9000
spring:
  application:
    name: SPRING-CLOUD-ZUUL
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7011/eureka
```

主启动类

```Java
@EnableZuulProxy        //启用Zuul的网关代理
@SpringBootApplication
public class ZuulApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulApplication.class, args);
    }
}
```

访问测试

![image-20200705115338950](http://picture.youyouluming.cn/image-20200704121618902.png)

使用Zuul网关通过微服务的名称访问consumer服务的实例，微服务名称必须小写

### 3、配置路由规则

使用指定地址代替微服务名称

配置文件

```properties
zuul:
  #忽略微服务名称
  ignored-services: '*'
  #访问所有路径前都要加一个前缀
  prefix: /scw
  routes:
    #自定义路由规则名称
    employee:
      #对哪个微服务进行路由代理
      service-id: spring-cloud-feign-consumer
      #代替微服务名称的路径 **表示匹配多层
      path: /zuul-emp/**
```

### 4、ZuulFilter

对请求进行过滤，创建一个继承了ZuulFilter的类

```java
@Component
public class MyZuulFilter extends ZuulFilter {
    Logger logger = LoggerFactory.getLogger(MyZuulFilter.class);

    @Override
    public String filterType() {
        //返回当前过滤器类型，决定当前过滤器在什么时候执行
        //pre表示在微服务前
        String filterType = "pre";
        return filterType;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * 判断当前请求是否过滤
     *
     * @return
     */
    @Override
    public boolean shouldFilter() {
        //获取RequestContext对象
        RequestContext requestContext = RequestContext.getCurrentContext();
        //获取Request对象
        HttpServletRequest request = requestContext.getRequest();
        //判断当前请求参数是否为signal=hello
        String signal = request.getParameter("signal");
        //如果是就放行
        return "hello".equals(signal);
    }

    //放行后执行run方法
    @Override
    public Object run() throws ZuulException {
        logger.info("run方法执行了");
        //此处返回值忽略
        return null;
    }
}
```