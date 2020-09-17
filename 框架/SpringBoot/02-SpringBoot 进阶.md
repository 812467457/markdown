# SpringBoot进阶

## 一、SpringBoot与缓存

### 1、介绍

在开发中如果频繁的从数据库查询数据，会对数据库的压力增大，而且一些临时数据也从数据库中获取，对整个系统的性能都有影响，所以引入缓存的概念。

### 2、Spring的缓存抽象

概念：

* Cache：缓存接口，定义缓存操作，如：RedisCache、EhCacheCache、CuncurrentMapCache。
* CacheManager：缓存管理器，管理各种缓存（Cache）组件
* serialize：缓存数据时value序列化
* KeyGenerator：缓存数据时key生成的策略

注解：

* @Cacheable：针对方法，表示根据方法请求参数对其结果进行缓存。
* @CacheEvict：清空缓存
* @CachePut：保证方法被调用，同时希望结果被缓存



### 3、简单使用缓存

* 开启基于注解的缓存：在主启动类上加@EnableCaching

* 在需要使用缓存的service方法上加@Cacheable，属性如下

  * cacheNames/value：指定缓存的名字，缓存管理器（CacheManager）中组件的名称。

  * key：缓存数据使用的key，默认时为方法参数的值。可以使用SpEL表达式。

  * keyGenerator：和key二选一，key的生成器，指定key的生成方式

  * cacheManager：指定缓存管理器。或者使用cacheResolver缓存解析器

  * condition：指定符合条件下才缓存

  * unless：当unless指定的条件为true，这个方法的返回值就不会缓存，可以根据结果进行判断

  * sync：是否使用异步模式 



> 简单使用缓存


```Java
@Cacheable(cacheNames = "emp")
public Employee getEmployeeById(Integer id){
    System.out.println("执行查询");
    return mapper.getEmployee(id);
}
```

效果：对相同参数的查询，只有第一次查询会从数据库获取数据，随后的查询之间从缓存获取数据。



### 4、缓存工作原理

1. 在启动应用后，有一个自动配置类CacheAutoConfiguration，加载缓存的配置类
2. 默认生效的缓存配置类是SimpleCacheConfiguration
3. SimpleCacheConfiguration给容器注册了一个缓存管理器ConcurrentMapCacheManager
4. 获取或创建ConcurrentMapCache类型的缓存组件，把数据保存在ConcurrentMap

@Cacheable执行流程：方法执行前会先检查缓存中有没有数据，默认按照参数的值作为key去查询缓存，如果没有缓存就运行方法并把结果放入缓存，如果有就返回缓存的数据。



### 5、@Cacheable的使用

* cacheNames/value：将方法的返回结果放入哪个缓存中，数组的方式，可以指定多个缓存

* key：缓存是存在一个ConcurrentMap中，key就获取缓存的那个map的key，默认为参数值。可以使用SpELl表达式。

  ```Java
  @Cacheable(cacheNames = "emp",key = "#root.methodName + '[' + #id + ']'")
  public Employee getEmployeeById(Integer id){
      System.out.println("执行查询");
      return mapper.getEmployee(id);
  }
  ```

* keyGenerator：自定义key生成器

  > 自定key生成器
  
  ```Java
  @Configuration
  public class MyCacheConfig {
      @Bean("myKeyGenerator")
      public KeyGenerator keyGenerator() {
          return new KeyGenerator() {
              @Override
              public Object generate(Object target, Method method, Object... params) {
                  return method.getName() + "[" + Arrays.asList(params).toString() + "]";
              }
          };
      }
  }
  ```
  
  > 使用
  
  ```Java
  @Cacheable(cacheNames = "emp",keyGenerator = "myKeyGenerator")
  public Employee getEmployeeById(Integer id){
      System.out.println("执行查询");
      return mapper.getEmployee(id);
  }
  ```
  
* condition：指定条件下缓存，比如id大于1时才缓存，也可以写为`"#a0>1"`，这时查询1号id就不缓存了

  ```Java
  @Cacheable(cacheNames = "emp", keyGenerator = "myKeyGenerator", condition = "#id>1")
  public Employee getEmployeeById(Integer id) {
      System.out.println("执行查询");
      return mapper.getEmployee(id);
  }
  ```

* unless：指定的条件为true时，不缓存，`unless = "#a0 == 2"`表示当第一个参数值为2时，不缓存。

  ```Java
  @Cacheable(cacheNames = "emp", keyGenerator = "myKeyGenerator", condition = "#id>1",unless = "#a0 == 2")
  public Employee getEmployeeById(Integer id) {
      System.out.println("执行查询");
      return mapper.getEmployee(id);
  }
  ```



### 6、@CachePut的使用

@CachePut：修改了数据库某个数据，同时更新缓存，常用在更新操作。同步更新缓存。

> 简单使用

```Java
@CachePut(value = "emp")
public Employee updateEmp(Employee employee) {
    mapper.updateEmployee(employee);
    System.out.println("执行更新" + employee);
    return employee;
}
```

问题引出：先进行一次查询，把id为1的数据存入缓存，然后再对id为1的emp进行更新操作，再次查询，发现缓存中的数据没有变化。

问题解析：CachePut的确对当前的数据重新缓存了，但是使用的Key不一样，缓存中都是键值对的，而key的默认值就是参数，CachePut和Cacheable的key不相同，所以查询的缓存也不相同。

> 指定Cacheable的key和CachePut一样，key = "#result.id"也可以写成key = "#Employee.id"

```Java
@Cacheable(cacheNames = "emp")
public Employee getEmployeeById(Integer id) {
    System.out.println("执行查询:" + id);
    return mapper.getEmployee(id);
}

@CachePut(value = "emp",key = "#result.id")
public Employee updateEmp(Employee employee) {
    mapper.updateEmployee(employee);
    System.out.println("执行更新" + employee);
    return employee;
}
```

CachePut的方法执行完后会向缓存中重新存入数据，下次再次使用Cacheable标注的查询操作，就不用查询数据库了。

**注意：**存和取的key必须一样。



### 7、@CacheEvict的使用

调用该注解标注的方法，会删除缓存中对应key的数据，下一次再查询就会查询数据库

```Java
@CacheEvict(value = "emp",key = "#id")
public void deleteEmp(Integer id){
    System.out.println("删除操作");
}
```

也可以指定所有的缓存，加上`allEntries = true`即可。

还可以设置是否在方法之前执行`beforeInvocation = true`，缓存的清除是否在方法的之前清除，默认false，如果在方法中出错回滚了，没有成功删除，那么缓存也不会被删除。



### 8、@Caching的使用

Caching内部组合了@Cacheable、@CachePut和@CacheEvict注解，当缓存的规则比较复杂就可以使用@Caching。

```Java
@Caching(
        cacheable = {
                @Cacheable(value = "emp", key = "#name")
        },
        put = {
                @CachePut(value = "emp", key = "#result.id"),
                @CachePut(value = "emp", key = "#result.email")
        }
)
public Employee getEmpByName(String name) {
    return mapper.getEmpByName(name);
}
```

这个方法中，使用name查询后就会存入缓存，再次使用id查询或Email查询就之间查询缓存，但是再次使用name查询还是会查询数据库，因为put方法一定会执行，在执行后就会刷新缓存。



### 9、类上的缓存配置@CacheConfig

可以在类上指定缓存的配置，抽取缓存的公共配置。

```Java
@Service
@CacheConfig(cacheNames = "emp")
public class EmployeeService {
    .....
}
```

比如在类上指定缓存的名称，类中的方法就不用一个一个的指定了。



### 10、Redis缓存

直接引入redis的依赖，启动应用就会默认加载RedisCache缓存组件

注意：Redis存储的对象需要支持序列化，所以在bean对象上实现序列化接口。

RedisManager默认使用的时Json格式进行存储，可以自定义RedisCacheManager的配置



## 二、SpringBoot与消息

### 1、RabbitMQ

- 简介：RabbitMQ是AMQP（高级消息队列）的开源实现。

- 核心概念：
  - Message：消息，由消息头和消息体组成，消息是不具名的，消息体是不透明的，消息头是由一系列的可选属性组成。

  - Publicsher：消息生产者，也是一个交换机发送消息的客户端应用程序。

  - Exchange：交换器，用来接收生产者发送的消息并将这些消息路由给服务器的队列。

  - Queue：消息队列，用来保存消息直到发送给消费者。是消息的容器，也是消息的终点。

  - Binding：绑定，用于消息和交换器之间的关联。一个交换器联系多个队列，当交换器收到消息时，根据路由派发的不同的队列。

  - Connection：网络连接，比如一个TCP连接。

  - Channel：信道，多路复用连接中的一条独立的双向数据流通道。

  - Consumer：消费者，用来获取消息的客户端。

  - Virtual Host：虚拟机，表示一批交换机、消息队列相关对象。

  - Broker：表示消息队列服务器的实体。

  - 执行流程

    ![image-20200916220342353](http://picture.youyouluming.cn/image-20200916220342353.png)

### 2、RabbitMQ运行机制

