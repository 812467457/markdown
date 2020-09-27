# Java常见面试题

## JavaSE

### 基础部分

#### String、StringBuffer、StringBuilder的区别

String是使用final修饰的，所以在定义后不可以修改，每次修改都是重新创建一个String对象。

StringBuffer是一个可变的字符串，并且线程安全，但是效率过低。

StringBuilder和StringBuffer相同，但是不是线程安全的，相对效率较高。

如果字符串的大小确定，尽量在创建字符串的同时指定大小。



#### final关键字怎么用的

final修饰一个值时，这个值不可以变。

final修饰一个引用对象时，这个引用地址不可以变。

final修饰一个方法时，这个方法不可以被继承修改。

final修饰一个类时，这个类不能被继承。并且final类中的成员变量默认也是final修饰的



### 集合

#### HashMap和Hashtable的区别

1. HashMap继承于AbstractMap，Hashtable继承于Dictionary，两者都实现了Map接口
2. HashMap运行Key和Value为null，Hashtable不可以
3. HashMap是线程不安全的，Hashtable是线程安全的



#### 谈谈HashMap的put方法

HashMap的初始值是16，加载因子是0.75，并且要求该容量必须是2的整数次幂。

在1.7中HashMap的数据结构是数组+链表，扩容采用的插头法，可能造成闭环，线程不安全。

在1.8中HashMap的数据结构是数组+链表+红黑树，在链表的长度大于8时，回自动转为红黑树（如果当前容量小于64，会先扩容）。当红黑树元素少于6个时，会再次从红黑树转为链表。在多线程put操作下，会出现数据被覆盖的情况，同样的线程不安全。



#### 谈谈HashSet

HashSet底层使用依然是HashMap来实现存储，值是Map的Key，value是一个固定值。

为什么采用Hash算法？

JDK1.7版本

在不使用Hash算法时，存放数据的时候需要判断唯一性，就需要进行遍历然后一个一个比较，性能太低。使用hash算法，通过计算存储对象的hashCode，再和数组长度-1做位运算，得到要存储位置的下标。但是依旧会出现一个问题，元素过多之后，可能会出现hash碰撞的问题（不同的对象计算出的hash值相同），这个时候，HashMap就会使用equals()方法进行判断，如果值相同就不存放，如果不相同，就会转换成一个链表，新进来的元素next指向上一个元素。

JDK1.8版本优化

随着元素的增加，链表会越来越长，链表的查找效率过低，所以优化为一个红黑树。



#### ArrayList和LinkList的区别

ArrayList底层数据结构时数组，LinkList底层数据结构是双向链表。

因为ArrayList是数组，所以在获取元素时直接返回数组下标对应的元素即可，但是数组的空间都是连续的，如果修改数据就会牵扯到位置的挪动，因此ArrayList的修改效率低。

LinkList的底层数据结构是双向链表，链表没有下标，所以查询的时候会从第一个元素开始查询，效率过低，但是链表在修改元素时只需要移动链表对应的指针就可以，效率相对高一些。



#### ArrayList的扩容

因为底层是数组，所以容量是固定的，默认容量是10，当传入数据时会判断当前容量是否需要扩容，创建一个新数组，容量时旧数组的1.5倍（右移 >1），将原数组的数据迁移到新数组。



### 异常

#### 请描述Java的异常体系

* Throwable
  * Error：JVM错误，比如内存溢出
  * Exception：异常
    * 运行时异常：不会提醒进行异常捕获，通常是逻辑异常，代码不严谨，比如数组越界、空指针异常。
      * 算术异常
      * 空指针
      * 类型转换
      * 数组越界
    * 非运行时异常：会提醒进行捕获或抛出，异常的产生通常是第三方，系统会提醒我们对此处进行处理
      * IOException
      * SQLException
      * FileNotFountException
      * NoSuchFileException
      * NoSuchMethodException



#### Throw和Throws的区别

* Throw：作用于方法上，用于主动抛出异常
* Throws：作用方法声明上，声明该方法可能会抛出的异常



### IO流

#### IO流的分类

按输入方向：输入流、输出流

按读取单位：字节流、字符流

IO流的四大基类：

* 字节流：
  * 输入：InputStream
    * FileInputStream：读取文件
  * 输出：OutStream
    * FileOutStream：写入文件
* 字符流：
  * 输入：Read
  * 输出：Write

#### IO流的选择

字节流主要用于二进制文件

字符流主要用于文本文件

通常使用高效缓冲流：

* 写数据：BufferedOutputStream
* 读数据：BufferedInputStream



### 反射

#### 什么是反射

反射就是动态的获取任意一个类的对象所有属性和方法的技术，可以执行方法或对属性赋值。

用处：在框架中经常使用反射，比如自动注入，只需要定义需要使用的类并且加上@Autowire注解，就可以访问到当前类的所有方法和属性，并且赋值。



#### 类加载的双亲委派机制

* JVM加载Java类的流程如下：
  * Java源文件 ----> 编译为class文件 ----> 类加载器(ClassLoader)加载class文件 ----> 转换为实例

ClassLoader是如何加载文件的？

根据不同的来源，Java使用不同的加载器来加载

* Java核心类，由BootstrapClassLoader（根加载器）加载，BootstrapClassLoader不继承于Class Loader，是JVM内部实现的，所以通过Java访问不到。
* Java扩展类，由ExtClassLoader（扩展类加载器）加载
* 项目中编写的类，由AppClassLoader（系统加载器）加载

双亲委派：就是加载一个类时，会先获取到一个AppClassLoader的实例，然后向上层层请求，先由BootstrapClassLoader加载，如果BootstrapClassLoader没有再去ExtClassLoader查找，如果还是没有就会来到AppClassLoader查找，如果还没有就会报错。



### 多线程

#### 创建多线程的方式有几种？

1. 继承Thread类：重写run()方法，调用start()启动线程。
2. 实现Runnable接口：重写run()方法，把实现了Runnable接口的实现类当作参数传入Thread类中，调用Thread.start()方法。
3. 实现Callable接口：重写call()方法，该方法可以有返回值，其余功能同Runnable。
4. 使用线程池的方式：使用Executors创建线程池，executorService.execute(实现了Runnable接口的实现类);，executorService.submit(实现了Callable接口的实现类);

#### 多线程的生命周期

* 新建：当一个Thread类或其子类被声明创建时，线程处于新建状态。
* 就绪：处于新建状态的线程被start后，进入线程队列等待CPU时间片，此时具备运行条件，但没分配到cpu资源
* 运行：当就绪线程被调度并获得cpu资源时，便进入运行状态
* 阻塞：在某种情况下，被人为的挂起或者执行输入输出操作时，让出cpu执行器并临时终止运行。
* 死亡：线程完成它全部的工作或线程被提前强制性的终止或出现异常导致结束。



#### sleep和wait的区别

* 类型不同
  * sleep是定义在Thread类上面
  * wait是定义在Object类上面
* 对于资源锁处理不同
  * sleep不会释放锁
  * wait会释放锁
* 使用范围不同
  * sleep可以使用在任何代码块
  * wait必须使用在同步代码块
* 生命周期不同
  * 当线程调用sleep方法时，会进入到timeed waiting状态，等待计时结束自动放开
  * 当线程调用wait方法时，会进入到waiting状态，需要调用notify或notifyAll来唤醒线程



#### synchronized和lock的区别

- 作用位置不同：synchronized给代码块、方法加锁，lock只能给代码块加锁。
- 获取和释放锁机制不同：synchronized无需手动获取锁和释放锁，发生异常会自动解锁，不会出现死锁。lock需要手动加锁和解锁，如果忘记解锁将出现死锁现象。



#### 什么是死锁

死锁就是两个线程互相占用对方所需要的资源，相互等待对方把自己所需要的资源释放出来。

怎么防止死锁？

- 采用trylock(timeout)方法，设置超时时间，超时后会自动退出，防止死锁。

- 减少同步代码的嵌套操作
- 减低锁的粒度，尽量不要多个功能共用一把锁

#### 谈谈你对ThreadLocal的理解

ThreadLocal是一个线程变量的工具类，其内部维护着一个map。

> 通过ThreadLocal的get和set方法查看

```Java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            //把当前ThreadLocal当作建，传进来的具体数据当作值存入map
            map.set(this, value);
        else
            createMap(t, value);
    }
```

可以看出每一个线程都有一个map对象，map对象保存本地线程对象的副本变量，所以对于不同的线程获取副本的值时，别的线程不能获取当前副本的值，形成了副本隔离，互不干扰。

使用场景有：在获取JDBC连接对象时，如果每个DAO都重新获取一次连接对象，那么在service中的事务控制就不会生效了，因为多个JDBC连接之间没有任何联系，这时使用ThreadLocal来改进，创建一个连接的工具类，使用ThreadLocal保存连接对象，这样其他线程使用连接对象时就保证了使用的是同一个对象，并且不会互相干扰。





## JavaWeb

### Servlet生命周期

1. Web容器加载Servlet类并实例化（默认延迟加载一次），可以指定在容器启动时加载`<load-on-startup>1</load-on-startup>`
2. 运行init()方法初始化（执行一次）
3. 用户请求servlet，服务器接收到请求后执行service
4. service运行与请求方式对应的方法（doGet或doPost）
5. 销毁实例时调用destroy方法（执行一次）

### 转发（forward）和重定向（redirect）的 区别

1. 转发是容器控制跳转，浏览器只用把内容读取出来，所以地址栏不变。对于客户端来说始终都是一次请求，所以保存在request的数据可以传递。
2. 重定向是服务器收到请求后，返回一个状态码（302）给浏览器，浏览器解析新的的地址进行跳转，地址栏会变。对于客户端是两次请求，所以request的数据就不可以传递了，如需数据传递就要使用session对象
3. 转发效率更高，推荐使用转发，但是转发不能访问到其他服务器上，重定向可以。



### JSP四大域对象

* ServletContext：Context域，只能在同一个web应用中使用（全局）
* HttpSession：Session域，只能在同一个会话中使用
* HttpServletRequest：Request域，只能在一个请求中使用，转发可以，重定向无效。
* PageContext：page域，只能在当前JSP页面中使用。



### JSP内置对象

| NO   | 内置对象    | 说明                                       |
| ---- | ----------- | ------------------------------------------ |
| 1    | pageContext | 页面上下文对象，可以取得任何范围内的参数。 |
| 2    | request     | 向客户端请求数据                           |
| 3    | response    | 服务器对客户端的响应，传送数据到客户端。   |
| 4    | session     | 会话，保存每个用户的信息，跟踪用户的操作   |
| 5    | application | 应用程序对象，范围是整个应用。             |
| 6    | config      | Servlet的配置，表示容器的配置信息          |
| 7    | out         | 对客户端输出数据                           |
| 8    | page        | JSP实现类的实例                            |
| 9    | exception   | 反映运行异常                               |



### Get和Post的区别

1. Get传递参数是在浏览器的地址栏传递，？连接&分割，以key-value形式。Post一般都是使用表单传递，传递到action指向的URL。
2. Get是不安全的，因为在传送过程中所有的参数都是暴露在浏览器地址栏的。Post相对安全一些，参数放在body里。
3. Get传输的数据又大小限制，Post没有大小限制，上传文件操作只能使用Post。





## Spring

### 谈谈什么是Spring IOC

IOC（控制反转）是Spring最核心的部分，IOC的前提需要先了解DI（依赖注入）。

在没有使用IOC的代码中，存在一个问题，就是多个类之间的互相依赖User user  = new User();，如果某个底层的类做出改动，则依赖它的所有类都有进行改动，这种行为显然不合理，所以引入了DI的理念，就是把下层类作为参数传给上层类，实现上层类对下层类的控制。使用DI后只需要定义好接口提供给外界调用即可，DI注入包括set注入、接口注入、注解注入、构造器注入。

IOC执行过程：在spring容器启动后，读取bean的配置信息（注解、xml配置、配置类），根据Bean注册表的信息实例化Bean，再将Bean的实例化装入到容器中，提供给应用使用。



### Spring Bean生命周期

1. 实例化Bean对象
2. 设置Bean的属性
3. 如果调用了各种Aware接口声明的依赖，就会在Bean初始化的阶段调用对应的方法，如：BeanNameAware，获取Bean的名字。
4. 如果实现了BeanPostProcessor接口（该接口需要另一个类中实现，并注入到bena容器），则会调用BeanPostProcessor的前置初始化方法postProcessBeforeInitialization
5. 如果实现了InitializingBean接口，则会调用init-method方法
6. 调用bean自身的init方法
7. 调用BeanPostProcessor的postProcessAfterInitialization后置方法。
8. 销毁，DisposableBean接口的销毁方法，和自身的销毁方法



### Spring的作用域

1. singleton(单例)：spring默认的作用域，每个IOC容器创建唯一的一个bean实例
2. prototype(多例)：针对每个getBean请求，容器都会单独创建一个bena实例
3. request：针对每个HTTP请求容器都会单独创建一个bean实例
4. session：同一个session范围内使用同一个bean实例
5. GlobalSession：用于protlet容器，全局的HTTP Session 共享一个bean实例



### SpringAOP

在不修改原来代码的基础上对其进行增强，用到了单例模式的设计思想。可以把公共代码进行集中处理，减少重复代码。

SpringAOP多用于在日志系统、安全、事务等功能中。

AOP底层基于动态代理技术，JDK动态代理或cglib动态代理操作字节码的技术，运行时动态生成被调用类型的子类，并且实例化对象再返回给响应的代理对象。

AOP包括四种增强机制：

* 前置增强：在进入方法之前执行。
* 后置增强：在方法return后执行的操作
* 异常增强：在抛出异常后执行的操作
* 最终增强：最后一定会执行的操作
* 环绕增强：集成了以上四种增强。

核心概念：

* Aspect：切面，定义增强方法的代码，将一个普通的类定义为一个增强类
* Pointcut：切入点，定义连接查询条件，哪些类哪些方法需要增强
* Advice：指定什么时候做



### Spring中的设计模式

* 在BeanFatory和Application中使用了工厂模式。
* 在创建Bean的过程中使用了单例模式和原型模式
* AOP技术使用了代理模式、装饰着模式、适配器模式。
* 事件监听使用的是观察者模式。



### Spring的事务

隔离级别：

1. DEFAULT：使用数据库默认的隔离级别。
2. READ_UNCOMMITTED：最低 级别，允许看到其他事务未提交的数据，会产生幻读、脏读、不可重复读。
3. READ_COMMITTED：只能读取已提交的数据，可以防止脏读，会出现幻读和不可重复读。
4. REPEATABLE_READ：防止脏读、不可重复读，会产生幻读。
5. SERIALIZABLE：最高级别，事务处理为串行，阻塞的，能避免所有情况。

脏读、幻读、不可重复读的区别：

* 脏读：在一个事务进行更新数据还未提交的时候，另一个事务来读取数据，在第二个事务把数据读取出来后第一个事务回滚了，这时第二个事务读取的数据就是错误的，所谓的脏读。
* 幻读：好比注册操作，第一个用户注册判断用户名可以用，但是还没提交，这时又来一个用户使用同样的用户名判断同样可以使用，这时只有一个用户可以成功提交，而剩下一个事务明明判断过该用户名可以使用但是提交失败，这就是幻读。
* 不可重复读：一个事务多次读取数据，在读取数据的同时又来一个事务修改了数据，就会造成第一个读取数据的事务读取的数据不一致的问题，就是不可重复读。

传播机制（Propagation）：

* REQUIRED：支持当前事务，如果当前没有事务就新建一个事务。常用。
* SUPPORTS：支持当前事务，如果没有事务，就以无事务的方式运行。
* MANDATORY：支持当前事务，如果没有事务，就抛出异常。
* NESTED：支持当前事务，如果当前事务存在，则执行一个嵌套事务，如果当前没有事务，就新建一个事务。外层事务回滚连带内存事务。
* REQUIRED_NEW：新建事务，如果当前存在事务，就挂起当前事务。
* NOT_SUPPORTED：以非事务方式执行，如果当前存在事务就挂起当前事务。
* NEVER：以非事务方式执行，如果当前存在事务，就抛出异常。




## Mybatis

### Mybatis中#和$的区别

#{}：解析为一个JDBC预编译语句的参数标记符。相当于一个参数占位符？，实际的SQL语句如 `select * from user where name = #{name}`解析为`select * from user where name = ?`，参数是动态传进来的。

${}：是直接把值替换进来，SQL语句如`select * from user where name = ${name}`解析为`select * from user where name = jack`。

总结：预编译的效率更高，SQL可以重复利用，并且能够防止SQL注入，能用预编译就用预编译`#{}`。但是有些特殊情况需要使用`${}`，比如表名、字段名需要动态替换时，就需要用到`${}`，因为预编译会在传进来的参数上加上单引号，表名和字段名不可以加单引号，而$传进来的是什么就是什么。

SQL注入：不使用预编译的情况下,会出现这种情况`select * from user where name = jack and 1 = 1`，这个SQL永远都是true，也就是说可以把所有数据都查询出来，具有安全隐患,如果使用了预编译就会这样`select * from user where name = 'jack and 1 = 1'`，此时会把整个参数当作一个字符串来查询,可以避免SQL注入。



### Mybatis的缓存

缓存就是把查询出来的结果存储起来，下一次查询就不去查询数据库，而是直接查询缓存。为了提高查询效率，降低数据库压力。

一级缓存：在同一个SqlSessin对象，在参数和SQL完全一样的情况下，只执行一次SQL语句，默认开启，不能关闭。

二级缓存：二级缓存在SqlSessionFacory生命周期中，需要手动开启。所有的select语句都会被缓存,所有的更新语句都会刷新缓存.

使用场景：在查询操作高，更新频率低的时候，每一次更新操作都会刷新缓存，所以在使用二级缓存机制时尽量减少更新操作。



### Mybatis分页插件的原理

**有哪些分页方式**？
一般分为物理分页和逻辑分页。

- 逻辑分页：指使用Mybatis自带的RowBounds进行分页，它会一次查询出多条数据，然后检索分页中的数据。一次查询所有。
- 物理分页：指从数据库查询指定条数的数据，平常使用pageHelper实现的就是物理分页。需要多少查询多少

逻辑分页在性能上不如物理分页，每次都查询所有的数据会导致系统性能的下降。

底层原理是使用了一个拦截器，通过start、rows、sql来设置查询的参数。



## SpringMVC

### 谈谈你对SpringMVC的理解

SpringMVC是一个基于Java实现的轻量级的Web框架。

核心组件：

* DispatcherServlet：前端控制器，所有流程的控制中心，控制其他组件的执行。
* Handler(Controller)：后端控制器，负责处理请求的控制逻辑。
* HandlerMapping：映射器对象，用于管理URL与对应的Controller对应的映射。
* HandlerAdapter：适配器，主要用来处理方法参数、注解、视图解析器等。
* ViewResolver：视图解析器，解析对应视图关系。

执行流程：

* 请求匹配到前端控制器（DispatcherServlet）的请求路径映射
* 前端控制器接收到请求后把请求交给处理器映射器（HandlerMapping）
* HandlerMapping根据用户的请求URL查找匹配该URL的Handler，并返回一个执行链（HandlerExecuttionChain）
* DispatcherServlet再请求处理器适配器（HandlerAdapter），调用相应的Handler进行处理并返回ModelAndView给DispatcherServlet。
* DispatcherServlet对View进行渲染，将页面响应给用户



## SpringBoot



## SpringCloud

### 谈谈你对SpringCloud的认识

SpringCloud是一个大的集合，把当前主流框架集中起来，以SpringBoot的方式开发。屏蔽了复杂的配置，提供了一套简单易上手的分布式开发工具箱。

SpringCloud解决了分布式开发中的问题，提供了许多组件来解决问题，比如服务注册与发现，负载均衡，熔断等非业务的共性问题。



### 说说SpringCloud有哪些组件，解决了什么问题

* 注册中心：Eureka，把每个微服务注册到注册中心上，对其他微服务暴露当前微服务
* 客户端负载均衡：Ribbon，指定以轮询或其他方式进行负载均衡
* 声明式远程方法调用：Fegin，只需要创建一个接口并用注解方式配置它，即可完成服务提供方的接口绑定
* 服务降级、熔断：Hystrix，如果某个微服务出现了故障，通过熔断监控应用是否出现故障，如果出现故障则触发降级机制，返回一个提前准备好的一个备选方案，防止长时间等待出现异常。
* 网关：Zuul，主要用来对请求的路由和过滤两个功能，路由是把一个外部的请求转发到具体的微服务上，实现外部统一访问入口的基础，过滤主要是把请求处理过程进行干预，实现请求校验、聚合等功能。
* 数据监控：Hystrix Dash Board，可以查看应用中详细的数据，比如某个微服务的访问量。





## 数据库

### 谈谈数据库设计的三大范式

* 第一范式：列不可分0（原子性）
* 第二范式：要有主键
* 第三范式：不可传递依赖



### 如何防止SQL注入

SQL注入：通过字符串的拼接构成了一种特殊的查询语句，如`select * from user where name = 'or 1=1.....'`，这样不管后面写什么都是正确的。

解决：使用预处理（PreparedStatement）对象，而非Statement对象，并且预处理对象还可以提高SQL的执行效率。

如果使用Mybatis还可以使用#{}传值。



### 谈谈事务的特点

原子性是基础，隔离性是手段，一致性是约束条件，而持久性是目的。简称ACID。

- 原子性：事务中包含的操作，要么一起成功，要么一起失败。
- 一致性：指数据库的数据在事务的操作前后的必须满足的条件约束。比如两个账户转账前和转账后，两个账户的总额不变。
- 隔离性：指一个事务的执行不能被其他事务干扰，设置不同的隔离级别，互相干扰的程度会不同。
- 持久性：指事务一旦提交，结果便是永久性，即使发生宕机，也可以通过日志恢复数据。



### 谈谈索引优化

> 索引概述

不使用索引之前都是把全表扫描到内存，进行轮询查找，在数据量较小时没有什么影响，但是当数据量过大，就会影响性能。

使用索引，就像字典一样，根据数据的信息（比如id主键，唯一键）去精确查找。

> 二叉树

使用二叉树优化索引，因为二叉树的时间复杂度是Ologn，并且在修改数据后有可能变为线性表，使用了二分搜索法，在数据量过大时，发生的IO操作太多，性能不理想。

> B树

B树和二叉树的复杂度相同，都是Ologn，也有可能变为线性表，效率不高

> B+树

B+树相比B树可以存储更多的数据，除了叶子节点，其他节点都不存储数据，只存储孩子节点的引用，这样就可以存储更多节点引用。B+树的叶子节点只存储数据，并且按照大小排列，支持范围统计，叶子节点可以横向跨子树统计。

总结：B+树更适合做优化索引，B+树可以根据查找的索引，对其加载一整个叶子节点的数据，相对来说对磁盘的读写更低，并且查询更稳定。

> Hash索引

理论上Hash索引比B+树效率更高，因为根据查询的关键词获取其对应的Hash值，然后再去查找数据。但是因为Hash表的限制，只能查询指定的数据，不能范围查找。不能使用部分索引键查询。并且Hash表可能变为线性的。



### 什么是悲观锁和乐观锁

- 悲观锁：是利用数据库本身的锁机制来实现，会锁记录。在操作的时候就把数据锁起来，操作结束再解锁。实现方式是`select * from user where id = 1 for update`。好处是安全，缺点是性能差。
- 乐观锁：是一种不锁记录的实现方式，采用CAS（比较并交互）模式，采用version字段作为判断依据。每次对数据更新都会让version+1，这样在每次提交操作时，如果version值已被更改就提交失败。如果使用业务字段作为判断依据，可能出现ABA问题。



## MQ

### 谈谈开发中应用消息中间件的场景

适合场景

* 不需要实时性的

优点

* 解耦
* 异步
* 限流削峰

在不使用 MQ 的应用中，每个操作步骤都会耦合在一起，操作一、二、...是一个同步的状态，如果某个操作出现问题，可能导致整体的问题，并且需要等待上一个操作完成才能进行下一步操作，用户体验不好。

使用了MQ后，用户的请求都会先到达消息中间件，并且直接给用户一个反馈，消息再由消息中间件分发给具体的业务执行，这个过程是异步的，响应时间大大减少，并且用户的体验也有提升。

场景：

* 基础服务：订单、注册、登录  ------ >  发送邮件/发送短信
* 限流削峰：在秒杀的场景，可以使用消息中间件来给定一个最大请求值，当请求达到最大的值，就不会接收请求了。



### 如何保障消息的可达或不丢失

异步 confirm 模式，客户端设置 confirm 监听器，获取 MQ 服务器的异步响应，如果消息传递成功，服务器传递回 ACK=true，否则 ACK=false，publisher-confirm=true。该机制只保证消息到MQ服务器。

消息队列持久化，在消息成功传递到MQ服务器后，对消息队列持久化，否则 MQ 服务器宕机后，数据会丢失。

在消费者方进行手动确认，确保消息真正的被消费者处理了，如果没有正确处理，会重新发送。



### 如何保证消息的幂等性

幂等性指一次请求或多次请求某个资源，对于资源本身应该有相同的结果。

在MQ中指，消费多条相同的消息，得到与消费该消息一次相同的结果。

使用乐观锁机制保障：

在向数据库交互时，携带应该版本号，只有到版本号和数据库中的版本号相等才会操作成功，每一次操作成功，数据库的版本号+1.