# Spring应用

## 一、Spring框架整体介绍

![image-20200727144451607](http://picture.youyouluming.cn/image-20200727144451607.png)

### 1、Core Container

Core和Beans模块是框架的基础部分，提供Ioc（反转控制）和DI（依赖注入）特性，

* Core：包含Spring框架的基本核心工具类，Spring的其他组件都要用到这个包里的类，是其他组件的基本核心。
* Beans：包含访问配置文件、创建管理Bean对象以及IOC和DI操作相关的类
* Context：处理BeanFactory，和ApplicationContext作用相同，构建于Core和Beans模块基础上，为Spring核心提供了大量的扩展，如国际化，事件传播、资源 加载和对Context的透明创建的支持，ApplicationContext接口时Context模块的关键。
  * BeanFactory和ApplicationContext本质的区别
    * BeanFactory：两者加载时机不同，在容器启动时不会实例化bean，只有在第一次使用的时候才会去实例化
    * ApplicationContext：容器一启动就把所有Bean实例化了，可以设置Bean延迟实例化。
* SpEL：提供表达式语言，用于查询和操作对象。

### 2、DataAccsee/Integration

* JDBC：包含Spring对JDBC访问的所有类
* ORM：提供了一个交互层，可以混合使用所有Spring提供的特性进行O/R映射，如简单的声明式事务。
* OXM：提供了一个对ObjecvXML映射实现的抽象层
* JMS：包含用于生产和消费消息的特点。
* Transaction：支持编程和声明式事务，这些事务必须实现特定接口，并对所有POJO都适用

### 3、Spring WEB

提供了基础的面向web的集成特性。例如：多文件上载功能和使用 Servlet listeners 和 web-oriented application context 初始化 IoC 容器。它还包含 HTTP client 和 Spring 远程支持的 web-related 部分。

### 4、Spring AOP

* Aspects：提供了对AspectJ的支持
* Instrumenttation：提供了class Instrumenttation支持和classoloader实现，使得可以在特定的应用服务器上使用。

### 5、Test

模块使用 JUnit 或 TestNG 支持单元测试和整合测试Spring组件。

## 二、Spring IOC 底层注解使用

### 1、IOC核心思想

在IOC容器出现之前，项目的类和类、类和配置文件之间都是一种互相依赖的关系，如果有一处依赖出了问题，可能整个项目都无法运行。而IOC容器改进了这一问题，资源和使用资源的一方都不去管理资源，把这个管理的权限交给一个不使用资源的第三方管理，带来的好处就是：一、资源资源的集中管理、资源的配置成本降低。二、降低资源双方的依赖程度，降低耦合度。

### 2、配置类方式配置并使用bean

#### 2.1、配置类

```Java
@Configuration
public class MainConfig {
    @Bean
    public Person person(){
        return  new Person();
    }
}
```

这样的写法和在xml配置文件配置的bean效果一样，bean的默认名称就是方法名称，也可以使用`name = "person"`的方式指定bean名称

#### 2.2、使用bean

```Java
public class Main {
    public static void main(String[] args) {
        
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        System.out.println(applicationContext.getBean("person"));
    }
}
```

传入配置类即可读取到bean对象

### 3、包扫描

> 简单使用

在配置类上加`@ComponentScan(basePackages = {"cn.yylm"})`注解，就可以扫描指定目录下的所有bean对象加载到IOC容器中。

> 排除指定的包

```Java
@ComponentScan(basePackages = {"cn.yylm"}, excludeFilters = {
        //排除指定的注解不扫描
        @ComponentScan.Filter(type = FilterType.ANNOTATION, value = {Controller.class}),
        //排除指定的类不扫描
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = {Person.class})
})
```

> 包含指定的包

```Java
@ComponentScan(basePackages = {"cn.yylm"}, includeFilters = {
        //排除指定的注解不扫描
        @ComponentScan.Filter(type = FilterType.ANNOTATION, value = {Controller.class}),
        //排除指定的类不扫描
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = {Person.class})
},useDefaultFilters = false)
```

注意：使用包含的用法要把useDefaultFilters设为false

### 4、作用域

#### 4.1、概述

在不指定@Scope的情况下，所有的bean都是单例的bean，并且在容器启动实例的时候都已经创建好了

```java
 	@Bean
    public Person person(){
        return  new Person();
    }
```

在指定`@Scope(value = "prototype")`表示为多实例的，并且不会再IOC容器启动时不会创建对象，而是再第一次使用时创建

```Java
@Bean(value = "person")
@Scope(value = "prototype")
public Person person() {
    return new Person();
}
```

Scope的取值：

* singleton：默认取值，单例的
* prototype：多例的
* request：同一次请求
* session：同一个会话级别

#### 4.2、测试

> 单例测试

准备一个实体类，构造方法打印person

```Java
public class Person {
    public Person() {
        System.out.println("person");
    }
}
```

配置类中配置bean

```Java
@Configuration
@ComponentScan(basePackages = {"cn.yylm"})
public class MainConfig {
    @Bean(value = "person")
    public Person person() {
        return new Person();
    }
}
```

在main方法中测试

```Java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
Person person1 = (Person) applicationContext.getBean("person");
Person person2 = (Person) applicationContext.getBean("person");
System.out.println(person1 == person2);
```

目前是默认的单例模式

输出为

![image-20200728192101308](http://picture.youyouluming.cn/image-20200728192101308.png)

可以看出只有一个实例对象。

> 多例情况如下

在配置中加上`value = "prototype"`

```Java
@Bean(value = "person")
@Scope(value = "prototype")
public Person person() {
    return new Person();
}
```

再次运行main方法

![image-20200728192219832](http://picture.youyouluming.cn/image-20200728192219832.png)

此时便是多例模式了

测试

> 懒加载测试

在配置类上加@Lazy注解

```Java
@Bean(value = "person")
@Lazy
public Person person() {
    return new Person();
}
```

此时main方法不去创建bean的实例

```Java
public static void main(String[] args) {
     AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
}
```

运行后不会有输出，只有在第一次创建实例时才会加载。

### 5、条件判断

如果有一个bean对象需要有另一个bean对象的存在才会创建实例，这时候就用到了条件判断的注解`@Conditional`。

假设现在又两个bena，一个testLog，一个testAspect，testLog需要testAspect那么使用代码这样实现。

> 首先需要一个实现了Condition接口的类，用来定义判断条件

```Java
public class TestConditional implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //判断容器中是否有testAspect的bean对象
        if (context.getBeanFactory().containsBean("testAspect")) {
            return true;
        }
        return false;
    }
}
```

> 注入bean，在@Conditional注解的value值就是刚才实现Condition的类

```Java
@Bean(name = "testAspect")
public TestAspect testAspect(){
    return new TestAspect();
}

@Bean(name = "testLog")
@Conditional(value = TestConditional.class)  //如果容器中有testAspect，TestLog才会被实例化
public TestLog testLog(){
    return new TestLog();
}
```

> 两个bena都创建实例的结果

![image-20200729104326299](http://picture.youyouluming.cn/image-20200729104326299.png)

> 不创建testAspect的实例就会找不到bean

### 6、IOC容器添加组件

#### 6.1、主要方式

1. @Bean

2. @CompentScan + @Controller/@Service/@Respositort/@Compent
3. @Import
4. FactoryBean

#### 6.2、@Import

> 使用@Import向容器中添加组件

```Java
@Configuration
@Import(value = {Person.class})
public class ImportTestConfig {
}
```

> 使用ImportSelector导入组件
>
> TestSelector是自定义的一个实现ImportSelector接口的类，selectImports方法返回要导入组件的全路径类名

```Java
@Configuration
@Import(value = {Person.class, TestSelector.class})
public class ImportTestConfig {
}

public class TestSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"cn.yylm.entity.Car"};
    }
}
```

> 使用ImportBeanDefinitionRegistrar导入组件（可以自定义bean的名称）
>
> @Import注解导入bean和@CompentScan+@Controller...导入bean的区别：@Import主要用于导入第三方组件，@CompentScan+@Controller...导入的一般是内部的组件。

```Java
@Configuration
@Import(value = {Person.class, TestSelector.class, TestImportBeanDefinitionRegistrar.class})
public class ImportTestConfig {
}

public class TestImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //创建一个bean对象
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(User.class);
        //把bean对象注入到容器
        registry.registerBeanDefinition("user", rootBeanDefinition);
    }
}
```

> 通过FactoryBean接口来实现导入组件，可以指定名称获取，可以获取到memberFactoryBean自己

```Java
@Configuration
@Import(value = {Person.class, TestSelector.class, TestImportBeanDefinitionRegistrar.class })
public class ImportTestConfig {
    @Bean
    public MemberFactoryBean memberFactoryBean(){
        return new MemberFactoryBean();
    }
}

public class MemberFactoryBean implements FactoryBean<Member> {
    //返回Bean对象
    @Override
    public Member getObject() throws Exception {
        return new Member();
    }
    //返回bean对象类型
    @Override
    public Class<?> getObjectType() {
        return Member.class;
    }
    //是否单例
    @Override
    public boolean isSingleton() {
        return false;
    }
}

public static void main(String[] args) {
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(ImportTestConfig.class);
    //获取bean对象，通过getObject方法
    Object memberFactoryBean = applicationContext.getBean("memberFactoryBean");
    //使用'&'获取memberFactoryBean对象 
    Object memberFactoryBean1 = applicationContext.getBean("&memberFactoryBean");
    System.out.println(memberFactoryBean);
    System.out.println(memberFactoryBean1);
}
```

## 三、Bean的生命周期

### 1、概述

> bean的生命周期：bean的创建 --> 初始化 --> 销毁方法
>
> 单实例：容器启动bean对象就创建了，容器销毁也回复触发bean对象的销毁方法
>
> 多实例：容器启动bean不会被创建，只有在第一次使用才会被创建，bean的销毁不受ioc容器管理

```Java
@Configuration
@ComponentScan(basePackages = {"cn.yylm"})
public class MainConfig {
    @Bean(value = "person",initMethod = "init",destroyMethod = "destroy")
    public Person person() {
        return new Person();
    }
}

public class Person {
    public Person() {
        System.out.println("person");
    }
    public void init(){
        System.out.println("person init");
    }
    public void destroy(){
        System.out.println("person destroy");
    }
}
```

### 2、Bean的后置处理器

> BeanPostProcessor在bean的初始化前和初始化后触发，主要用来修改bean的属性。

```Java
public class TestBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization");
        return bean;
    }
}

@Configuration
@ComponentScan(basePackages = {"cn.yylm"})
public class MainConfig {
    @Bean(value = "person",initMethod = "init",destroyMethod = "destroy")
    //@Scope(value = "prototype")
    public Person person() {
        return new Person();
    }

    @Bean
    public TestBeanPostProcessor testBeanPostProcessor(){
        return new TestBeanPostProcessor();
    }
}    
```

输出结果

![image-20200729135510060](http://picture.youyouluming.cn/image-20200729135510060.png)

## 四、给组件赋值

> 使用@Value + @PropertySource注解给组件赋值

```Java
public class Person {
    //value普通方式
    @Value("jack")
    private String name;

    //spel方式
    @Value("#{25-2}")
    private Integer age;

    //外部配置文件
    @Value("${person.address}")
    private String address;
}

@Configuration
@ComponentScan(basePackages = {"cn.yylm"})
@PropertySource(value = {"classpath:person.properties"})
public class MainConfig {
    @Bean(value = "person")
    public Person person(){
        return  new Person();
    }
}
```

输出结果：`Person{name='jack', age=23, address='china'}`

## 五、自动装配

> 使用Autowired注解进行自动装配，如果 ioc容器存在多个相同类型的组件，就按照名称来装配

```Java
@Service
public class TestService {
    @Autowired
    private TestDao testDao;
}
```

> 也可以使用@Qualifier注解指定装配的组件的名称，或者在配置类的@Bean上加@Primary注解

```Java
@Service
public class TestService {
    @Autowired
    @Qualifier(value = "testDao")
    private TestDao testDao;
}
```

> 如果使用@Qualifier注解但是没有找到对应的名称的bean就会报异常，把required指定为fale即可

```Java
@Service
public class TestService {
    @Autowired(required = false)
    @Qualifier(value = "testDao")
    private TestDao testDao;
}
```

> @Resource注解，功能和Autowired注解一样，但是不支持@Qualifier和@Primary 

