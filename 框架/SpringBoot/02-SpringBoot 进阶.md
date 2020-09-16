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



### 5、@Cacheable属性

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

@CachePut：修改了数据库某个数据，同时更新缓存，常用在更新操作。

