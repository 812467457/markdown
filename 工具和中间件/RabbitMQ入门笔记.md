# RabbitMQ 基础

## 一、入门

### 1、概述

* MQ概述
  * MQ 全称（Message Queue）消息队列，是在消息的传输中保存消息的容器，多用于分布式系统之间的通信。

* MQ优势
  * 应用解耦：加入MQ 后，如果某个服务故障，不会影响到其他服务，提高系统的容错性和可维护性。
  * 异步提速：用户请求某个服务后立马得到 MQ 的相应，提升用户体验和系统吞吐量。
  * 削峰填谷：在高并发的环境中，使用 MQ 去处理大量的请求，把请求放在消息队列慢慢处理，防止请求过多使服务器宕机。
* MQ的劣势
  * 系统可用性降低：MQ 的宕机会对业务造成影响。
  * 系统复杂性提高：使用 MQ 异步调用后，需要保证消息没有被重复消费、丢失、传递的顺序。
  * 一致性问题：保证各个模块之间数据的一致性。

* RabbitMQ概述
  * RabbitMQ 是基于 AMQP 协议的一款消息队列产品。
  * RabbitMQ提供了六种工作模式



### 2、安装 RabbitMQ 

#### 2.1、 安装依赖环境

在线安装依赖环境：

```shell
yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz
```

#### 2.2、 安装 Erlang

把对应的安装包上传到Linux

![image-20200924153914285](http://picture.youyouluming.cn/image-20200924153914285.png)



```shell
#安装
rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm
```

#### 2.3、安装 RabbitMQ

```shell
# 安装
rpm -ivh *.rpm --force --nodeps socat-1.7.3.2-1.1.el7.x86_64.rpm
# 安装
rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm
```

#### 2.4、 开启管理界面及配置

```sh
# 开启管理界面
rabbitmq-plugins enable rabbitmq_management
# 修改默认配置信息
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app 
# 比如修改密码、配置等等，例如：loopback_users 中的 <<"guest">>,只保留guest
```




#### 2.5、 启动

```sh
service rabbitmq-server start # 启动服务
service rabbitmq-server stop # 停止服务
service rabbitmq-server restart # 重启服务
```



- 设置配置文件

```shell
cd /usr/share/doc/rabbitmq-server-3.6.5/

cp rabbitmq.config.example /etc/rabbitmq/rabbitmq.config

```

#### 2.6、 用户角色

RabbitMQ在安装好后，可以访问`http://ip地址:15672` ；其自带了guest/guest的用户名和密码；如果需要创建自定义用户；那么也可以登录管理界面后，如下操作：

![1565098043833](http://picture.youyouluming.cn/1565098043833.png) 



![1565098315375](http://picture.youyouluming.cn/1565098315375.png)

**角色说明**：

1、 超级管理员(administrator)

可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。

2、 监控者(monitoring)

可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)

3、 策略制定者(policymaker)

可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。

4、 普通管理者(management)

仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。

5、 其他

无法登陆管理控制台，通常就是普通的生产者和消费者。

#### 2.7、 Virtual Hosts配置

像mysql拥有数据库的概念并且可以指定用户对库和表等操作的权限。RabbitMQ也有类似的权限管理；在RabbitMQ中可以虚拟消息服务器Virtual Host，每个Virtual Hosts相当于一个相对独立的RabbitMQ服务器，每个VirtualHost之间是相互隔离的。exchange、queue、message不能互通。 相当于mysql的db。Virtual Name一般以/开头。



#### 2.8、 创建Virtual Hosts

![1565098496482](http://picture.youyouluming.cn/1565098496482.png)

#### 2.9、 设置Virtual Hosts权限

![1565098585317](http://picture.youyouluming.cn/1565098585317.png)



![1565098719054](http://picture.youyouluming.cn/1565098719054.png)



### 3、简单使用

需求：一个生产者发送消息，一个消费者接受消息。

准备一个生产者工程和消费者工程，然后导入依赖

```xml
<dependencies>
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.6.0</version>
    </dependency>
</dependencies>


<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```



#### 3.1、生产者

```Java
/**
 * 生产者：发送消息
 */
public class HelloProducer {
    public static void main(String[] args) throws IOException, TimeoutException {
        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置参数
        factory.setHost("47.95.226.96");   //设置IP
        factory.setPort(5672);              //设置端口
        factory.setVirtualHost("/yylm");    //设置虚拟机
        factory.setUsername("jack");        //设置用户
        factory.setPassword("123456");      //设置密码
        //创建连接
        Connection connection = factory.newConnection();
        //创建channel
        Channel channel = connection.createChannel();
        //创建队列Queue
        /*
            channel.queueDeclare(....)参数解释
            String queue                        队列名称，如果指定的队列不存在则会自动创建队列
            boolean durable                     是否持久化，当mq重启后消息还在
            boolean exclusive                   是否独占（只能有一个消费者监听队列），当连接关闭是是否删除队列。
            boolean autoDelete                  是否自动删除，当没有消费者时，自动删除
            Map<String, Object> arguments       参数
         */
        channel.queueDeclare("hello_world", true, false, false, null);
        //发送消息
        /*
            channel.basicPublish(...)参数解释
            String exchange                     交换机名称，简单模式下使用默认的。
            String routingKey                   路由名称，如果使用默认的交换机，路由名称必须和队列名称一样
            BasicProperties props               配置信息
            byte[] body                         发送的消息数据
         */
        String body = "hello rabbitMQ!";
        channel.basicPublish("", "hello_world", null, body.getBytes());
        //释放资源
        channel.close();
        connection.close();
    }
}
```

运行后RabbitMQ的管理界面会有一个队列

![image-20200924170040736](http://picture.youyouluming.cn/image-20200924170040736.png)



#### 3.2、消费者

```Java
/**
 * 消费者：接受消息
 */
public class HelloConsumer {
    public static void main(String[] args) throws IOException, TimeoutException {
        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置参数
        factory.setHost("47.95.226.96");   //设置IP
        factory.setPort(5672);              //设置端口
        factory.setVirtualHost("/yylm");    //设置虚拟机
        factory.setUsername("jack");        //设置用户
        factory.setPassword("123456");      //设置密码
        //创建连接
        Connection connection = factory.newConnection();
        //创建channel
        Channel channel = connection.createChannel();
        //创建队列Queue
        channel.queueDeclare("hello_world", true, false, false, null);
        //接收消息
        /*
            channel.basicConsume(...)   参数解释
            String queue                队列名称
            boolean autoAck             是否自动确认
            Consumer callback           回调函数
         */
        Consumer consumer = new DefaultConsumer(channel) {
            /**
             * 回调方法，收到消息后执行该方法
             * @param consumerTag   消息标识
             * @param envelope      获取信息，交换机、路由key...
             * @param properties    配置信息
             * @param body          数据
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumerTag" + consumerTag);
                System.out.println("envelope.getExchange" + envelope.getExchange());
                System.out.println("envelope.getRoutingKey" + envelope.getRoutingKey());
                System.out.println("properties" + properties);
                System.out.println("body" + new String(body));
            }
        };
        channel.basicConsume("hello_world", true, consumer);

        //消费者不必关闭资源，保持监听状态
    }
}
```

![image-20200924172255150](http://picture.youyouluming.cn/image-20200924172255150.png)



## 二、RabbitMQ 的工作模式

上面的案例就使用了 RabbitMQ 的简单工作模式。

不同的工作模式就是消息的路由和策略不一样。是消息分发的一种方式。

### 1、Work queues（工作队列）

![image-20200924174439107](http://picture.youyouluming.cn/image-20200924174439107.png)

一个生产者生产消息放到消息队列，两个或多个消费者争抢消息，只有一个消费者可以获得消息。

应用场景：对应任务过重或任务较多情况使用工作队列可以提高处理任务的速度。

代码和简单模式类似，在生产者生产十条消息，使用两个消费者获取消息，两个消费者会按照一人获取一次消息。



### 2、Pub/Sub（订阅模式）

![image-20200924174514738](http://picture.youyouluming.cn/image-20200924174514738.png)

生产者把消息发给交换机，交换机把消息根据路由规则分发给不同的消息队列，消费者监听队列获取消息。

Exchange（交换机）：一方面接收生产者发送的消息，另一方面是处理消息。主要有以下三种交换机

* Fanout：广播，将消息交给所有绑定到交换机的队列。
* Direct：定向，把消息交给符合指定 routing key 的队列
* Topic：通配符，把消息交给 routing pattern（路由模式）的队列

交换机只负责转发消息，不保存消息，因此，如果没有队列与交换机绑定，或者没有符合的路由规则，那么消息就会消失。

Pub/Sub 的生产者模式

```Java
/**
 * 订阅模式
 * 生产者：发送消息
 */
public class HelloProducer_PubSub {
    public static void main(String[] args) throws IOException, TimeoutException {
        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置参数
        factory.setHost("47.95.226.96");   //设置IP
        factory.setPort(5672);              //设置端口
        factory.setVirtualHost("/yylm");    //设置虚拟机
        factory.setUsername("jack");        //设置用户
        factory.setPassword("123456");      //设置密码
        //创建连接
        Connection connection = factory.newConnection();
        //创建channel
        Channel channel = connection.createChannel();
        //创建交换机
        /*
            channel.exchangeDeclare()       参数解释
            String exchange                 交换机名称
            BuiltinExchangeType type        交换机的类型，枚举，其中定义了四个类型
            boolean durable                 是否持久化
            boolean autoDelete              是否自动删除
            boolean internal                内部使用，通常false
            Map<String, Object> arguments   参数列表
         */
        String exchangeName = "test_fanout";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.FANOUT, true, false, false, null);
        //创建队列
        String queue1Name = "test_fanout_queue1";
        String queue2Name = "test_fanout_queue2";
        channel.queueDeclare(queue1Name, true, false, false, null);
        channel.queueDeclare(queue2Name, true, false, false, null);
        //绑定队列和交换机
        /*
            channel.queueBind()             参数解释
            String queue                    队列名称
            String exchange                 交换机名称
            String routingKey               路由键，绑定规则，交换机为fanout，routingKey设置为空字符串，意味着交换机会把每一个消息分发给队列
            Map<String, Object> arguments   参数
         */
        channel.queueBind(queue1Name, exchangeName, "", null);
        channel.queueBind(queue2Name, exchangeName, "", null);
        //发送消息
        String body = "PubSub_info";
        channel.basicPublish(exchangeName, "", null, body.getBytes());
        //释放资源
        channel.close();
        connection.close();
    }
}
```

消费者和之前一样，把队列换到生产者创建的队列



### 3、Routing（路由模式）

![image-20200924211305141](http://picture.youyouluming.cn/image-20200924211305141.png)

说明：

* 队列与交换机绑定，不能任意绑定，需要指定一个 routingKey（路由规则）
* 消息在向交换机发送消息时必须指定 routingKey
* 交换器不再把消息交给每一个队列了，而是根据 routingKey 进行判断，只有队列的 routingKey 和消息 routingKey 一致才会收到消息。

> 生产者需要改动的地方

```Java
...
//创建交换机
String exchangeName = "test_direct";
//工作模式改为DIRECT
channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT, true, false, false, null);
//创建队列
String queue1Name = "test_direct_queue1";
String queue2Name = "test_direct_queue2";
channel.queueDeclare(queue1Name, true, false, false, null);
channel.queueDeclare(queue2Name, true, false, false, null);
//绑定队列和交换机
//队列一的绑定，此处的routingKey不能为空，需要指定规则
channel.queueBind(queue1Name, exchangeName, "error", null);
//队列二的绑定
channel.queueBind(queue2Name, exchangeName, "info", null);
channel.queueBind(queue2Name, exchangeName, "error", null);
channel.queueBind(queue2Name, exchangeName, "warning", null);
//发送消息
String body = "log-level=info";
//发送消息时指定消息的routingKey
channel.basicPublish(exchangeName, "warning", null, body.getBytes());
...
```



### 4、Topics（通配符模式）

![image-20200924221940580](http://picture.youyouluming.cn/image-20200924221940580.png)

Topics 模式可以实现 Pub/Sub 发布与订阅模式和 Routing 路由模式的功能，只是 Toics 在配置 RoutingKey 时可以通配符，显示更加灵活。



> 部分代码

```Java
...
String exchangeName = "test_topics";
channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC, true, false, false, null);
//创建队列
String queue1Name = "test_topics_queue1";
String queue2Name = "test_topics_queue2";
channel.queueDeclare(queue1Name, true, false, false, null);
channel.queueDeclare(queue2Name, true, false, false, null);
//绑定队列和交换机
//声明routingKey  系统的名称.日志的级别
//所有的error级别的日志和order系统的日志存入数据库
channel.queueBind(queue1Name, exchangeName, "#.error", null);
channel.queueBind(queue1Name, exchangeName, "order.*", null);
channel.queueBind(queue2Name, exchangeName, "*.*", null);
//发送消息
String body = "PubSub_info";
channel.basicPublish(exchangeName, "aa.info", null, body.getBytes());
...
```



## 三、整合RabbitMQ

### 1、Spring整合RabbitMQ

> 导入依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.7.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit</artifactId>
            <version>2.1.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.1.7.RELEASE</version>
        </dependency>
    </dependencies>
	
	<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```



> 把配置信息提取出来

```properties
rabbitmq.host=47.95.226.96
rabbitmq.port=5672
rabbitmq.username=jack
rabbitmq.password=123456
rabbitmq.virtual-host=/yylm
```



#### 1.1、生产者

> Spring整合RabbitMQ的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/rabbit
       http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">
    <!--加载配置文件-->
    <context:property-placeholder location="classpath:rabbitmq.properties"/>

    <!-- 定义rabbitmq connectionFactory -->
    <rabbit:connection-factory id="connectionFactory" host="${rabbitmq.host}"
                               port="${rabbitmq.port}"
                               username="${rabbitmq.username}"
                               password="${rabbitmq.password}"
                               virtual-host="${rabbitmq.virtual-host}"/>
    <!--定义管理交换机、队列-->
    <rabbit:admin connection-factory="connectionFactory"/>

    <!--定义持久化队列，不存在则自动创建；不绑定到交换机则绑定到默认交换机
    默认交换机类型为direct，名字为：""，路由键为队列的名称
    -->
    <rabbit:queue id="spring_queue" name="spring_queue" auto-declare="true"/>

    <!-- ~~~~~~~~~~~~~~~~~~~~~~~~~~~~广播；所有队列都能收到消息~~~~~~~~~~~~~~~~~~~~~~~~~~~~ -->
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_fanout_queue_1" name="spring_fanout_queue_1" auto-declare="true"/>

    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_fanout_queue_2" name="spring_fanout_queue_2" auto-declare="true"/>

    <!--定义广播类型交换机；并绑定上述两个队列-->
    <rabbit:fanout-exchange id="spring_fanout_exchange" name="spring_fanout_exchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding queue="spring_fanout_queue_1"/>
            <rabbit:binding queue="spring_fanout_queue_2"/>
        </rabbit:bindings>
    </rabbit:fanout-exchange>
    
    <!--direct类型的交换机绑定队列，要指定队列和路由键-->
    <rabbit:direct-exchange name="spring_direct_exchange" id="spring_direct_exchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding queue="spring_direct_queue_1" key="error"/>
        </rabbit:bindings>
    </rabbit:direct-exchange>

    <!-- ~~~~~~~~~~~~~~~~~~~~~~~~~~~~通配符；*匹配一个单词，#匹配多个单词 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ -->
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_topic_queue_star" name="spring_topic_queue_star" auto-declare="true"/>
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_topic_queue_well" name="spring_topic_queue_well" auto-declare="true"/>
    <!--定义广播交换机中的持久化队列，不存在则自动创建-->
    <rabbit:queue id="spring_topic_queue_well2" name="spring_topic_queue_well2" auto-declare="true"/>
	<!--topic类型交换机绑定队列，指定通配符-->
    <rabbit:topic-exchange id="spring_topic_exchange" name="spring_topic_exchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding pattern="marry.*" queue="spring_topic_queue_star"/>
            <rabbit:binding pattern="july.#" queue="spring_topic_queue_well"/>
            <rabbit:binding pattern="tom.#" queue="spring_topic_queue_well2"/>
        </rabbit:bindings>
    </rabbit:topic-exchange>

    <!--定义rabbitTemplate对象操作可以在代码中方便发送消息-->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory"/>
</beans>
```



> 测试代码

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-rabbitmq-producer.xml")
public class Producer {
    //注入rabbitTemplate
    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 发送简单模式消息
     */
    @Test
    public void test1() {
        String body = "hello-spring-rabbitMq";
        //convertAndSend(队列名称，发送的消息)
        rabbitTemplate.convertAndSend("spring_queue", body);
    }

    /**
     * 发送Fanout消息
     */
    @Test
    public void test2() {
        String body = "hello-spring-rabbitMq-fanout";
        //convertAndSend(交换机，路由键，消息)
        rabbitTemplate.convertAndSend("spring_fanout_exchange", "", body);
    }

    /**
     * 发送Topic消息
     */
    @Test
    public void test3() {
        String body = "hello-spring-rabbitMq-Topic";
        //convertAndSend(交换机，路由键，消息)
        rabbitTemplate.convertAndSend("spring_topic_exchange", "marry.hello", body);
    }
}
```



#### 1.2、消费者

> 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/rabbit
       http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">
    <!--加载配置文件-->
    <context:property-placeholder location="classpath:rabbitmq.properties"/>

    <!-- 定义rabbitmq connectionFactory -->
    <rabbit:connection-factory id="connectionFactory" host="${rabbitmq.host}"
                               port="${rabbitmq.port}"
                               username="${rabbitmq.username}"
                               password="${rabbitmq.password}"
                               virtual-host="${rabbitmq.virtual-host}"/>

    <!--定义监听器-->
    <bean id="springQueueListener" class="cn.yylm.rabbitmq.listener.SpringQueueListener"/>

    <!--把监听器注册到容器-->
    <rabbit:listener-container connection-factory="connectionFactory" auto-declare="true">
        <rabbit:listener ref="springQueueListener" queue-names="spring_queue"/>
    </rabbit:listener-container>
</beans>
```



> 监听器类

```Java
public class SpringQueueListener implements MessageListener {
    @Override
    public void onMessage(Message message) {

        System.out.println(new String(message.getBody()));
    }
}
```



> 使用一个测试类运行把配置文件加载进来

```Java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-rabbitmq-consumer.xml")
public class ConsumerTest {

    @Test
    public void test(){
        while (true) {

        }
    }
}
```



### 2、SpringBoot整合RabbitMQ

> 导入依赖

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



> 配置文件

```yml
spring:
  rabbitmq:
    host: 47.95.226.96
    port: 5672
    username: jack
    password: 123456
    virtual-host: /yylm
```



#### 2.1、生产者

> 使用一个配置类，配置交换机、队列和绑定关系

```java
@Configuration
public class RabbitConfig {
    public static final String EXCHANGE_NAME = "boot_topic_exchange";
    public static final String QUEUE_NAME = "boot_queue";

    //配置交换机
    @Bean("bootExchange")
    public Exchange bootExchange() {
        //ExchangeBuilder可以构建四种交换机类型
        //topicExchange(EXCHANGE_NAME)交换机名称，durable是否持久化,build构建
        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(true).build();
    }

    //配置队列
    @Bean("bootQueue")
    public Queue bootQueue() {
        //创建队列，指定名称
        return QueueBuilder.durable(QUEUE_NAME).build();
    }

    //配置交换机和队列的绑定
    @Bean()
    public Binding bindQueueExchange(@Qualifier("bootQueue") Queue queue,@Qualifier("bootExchange") Exchange exchange) {
        //绑定的队列和交换机，指定路由键，是否需要参数
        return BindingBuilder.bind(queue).to(exchange).with("boot.#").noargs();
    }
}
```



> 测试

```java 
@SpringBootTest
class SpringbootRabbitmqProducerApplicationTests {
    @Autowired
    private RabbitTemplate template;

    @Test
    void test() {
      template.convertAndSend(RabbitConfig.EXCHANGE_NAME,"boot.hello","hello boot rabbitMQ");
    }
}
```



#### 2.2、消费者

> 监听队列的消息

```java
@Component
public class RabbitMQListener {
    @RabbitListener(queues = "boot_queue")
    public void listenerQueue(Message message){
        System.out.println(new String(message.getBody()));
    }
}
```