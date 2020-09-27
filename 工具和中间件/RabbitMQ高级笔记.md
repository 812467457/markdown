# RabbitMQ 高级

## 一、高级特性

### 1、消息可靠性投递

在使用RabbitMQ时，为了防止消息丢失或投递失败的情况，RabbitMQ提供了两种方式来控制消息投递的可靠性模式。

* Confirm ：确认模式
  * 消息从 producer 到 exchange 则会返回一个 callfirmCallBack
* return ：回退模式
  *  消息从 producer 到 exchange 投递失败则会返回一个 returnCallBack



#### 1.1、Confirm 确认模式

* 在 connection-factory 开启确认模式
* 在 rabbitTemplate 定义 CallfirmCallback 回调函数

> 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xmlns:ra="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/rabbit
       http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">
    <!--加载配置文件-->
    <context:property-placeholder location="classpath:rabbitmq.properties"/>

    <!-- 定义rabbitmq connectionFactory -->
    <!--开启确认模式 publisher-confirms="true"-->
    <rabbit:connection-factory id="connectionFactory" host="${rabbitmq.host}"
                               port="${rabbitmq.port}"
                               username="${rabbitmq.username}"
                               password="${rabbitmq.password}"
                               virtual-host="${rabbitmq.virtual-host}"
                               publisher-confirms="true"/>
    <!--定义管理交换机、队列-->
    <rabbit:admin connection-factory="connectionFactory"/>

    <!--定义rabbitTemplate对象操作可以在代码中方便发送消息-->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory"/>

    <!--消息可靠性投递-->
    <!--定义队列-->
    <rabbit:queue id="test_queue_confirm" name="test_queue_confirm">

    </rabbit:queue>
    <!--定义交换器-->
    <rabbit:direct-exchange name="test_exchange_confirm">
        <rabbit:bindings>
            <rabbit:binding queue="test_queue_confirm" key="confirm"/>
        </rabbit:bindings>
    </rabbit:direct-exchange>
</beans>
```



> 生产者发送消息

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:spring-rabbitmq-producer.xml")
public class Producer {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    /**
     * 确认模式
     */
    @Test
    public void test1() {
        //定义回调
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            /**
             * 回调方法
             * @param correlationData   相关配置信息
             * @param b               	交换机是否成功收到消息
             * @param s             	失败的原因
             */
            @Override
            public void confirm(CorrelationData correlationData, boolean b, String s) {
                System.out.println("confirm run + " + "ACK=" + b);
                if (b) {
                    System.out.println("success");
                } else {
                    System.out.println("failed,cause=" + s);
                }
            }
        });

        //发送消息
        rabbitTemplate.convertAndSend("test_exchange_confirm", "confirm", "message");
    }
}
```



#### 1.2、回退模式

* 在 connection-factory 开启开启回退模式
* 设置ReturnCallback
* 设置交换机处理消息的机制
  * 如果没有路由到队列，则丢弃消息
  * 如果没有路由到队列，把消息返回给发送方

> 测试，只有发送消息失败，才会执行回调方法returnedMessage

```java
@Test
public void test2() throws InterruptedException {
    //设置交换机处理失败的模式
    rabbitTemplate.setMandatory(true);

    //设置returnCallback
    rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback() {
        /**
         *
         * @param message   消息对象
         * @param i         错误码
         * @param s         错误信息
         * @param s1        交换机
         * @param s2        路由键
         */
        @Override
        public void returnedMessage(Message message, int i, String s, String s1, String s2) {
            System.out.println("returnedMessage run");
        }
    });

    //发送消息
    rabbitTemplate.convertAndSend("test_exchange_confirm", "confirm123", "message...return");
}
```



### 2、Consumer ACK

消费端收到消息后确认方式，有三种方式。

* 自动确认：acknowledge="none"，一旦消息被消费者收到，就会自动确认，并且删除队列中的消息。如果没有收到消息或出现异常，那么消息将丢失。
* 手动确认：acknowledge="manual"，如果出现异常，可以调用channel.baiscNack()方法，让其自动发送消息。
* 根据异常情况来确认：acknowledge="auto"

> 实现步骤

* 在配置文件中的 listener-container 下设置 acknowledge="manual" 手动签收
* 监听器类实现 ChannelAwareMessageListener 接口，重写 onMessage 方法
* 如果消息成功处理，调用 channel 的 basicACK 方法签收
* 如果消息处理失败，调用 channel 的 basicNack 方法拒绝签收，重新发送给消费者



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
    <context:component-scan base-package="cn.yylm.rabbitmq.listener"/>
    
    <!--定义监听器容器-->
    <!--acknowledge="manual" 设置手动签收-->
    <rabbit:listener-container connection-factory="connectionFactory" acknowledge="manual">
        <rabbit:listener ref="ackListener" queue-names="test_queue_confirm"/>
    </rabbit:listener-container>
</beans>
```

> 监听器

```java
@Component("ackListener")
public class ACKListener implements ChannelAwareMessageListener {
    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        try {
            System.out.println(new String(message.getBody()));
            //如果出现异常就会一直尝试签收
            //int i = 1 / 0;
            System.out.println("业务逻辑处理");
            //手动签收
            /*
                long deliveryTag    该消息的index
                boolean multiple    是否批量.true:将一次性拒绝所有小于deliveryTag的消息。
             */
            channel.basicAck(deliveryTag,true);
        } catch (Exception e) {
            //出现异常，拒绝签收
            /*
                long deliveryTag
                boolean multiple
                boolean requeue     是否重回队列
             */
            channel.basicNack(deliveryTag,true,true);
        }
    }
}
```





### 3、消费端限流

* 确保 ack 机制为手动确认
* 在 listener-container 中配置 prefetch=N 设置一次拉取消息的最大量，直到手动确认消费完毕后，才会拉取下一条消息。

> 配置文件

![image-20200925171138819](C:\Users\luming\AppData\Roaming\Typora\typora-user-images\image-20200925171138819.png)

> 监听器

```java
@Component("qosListener")
public class QosListener implements ChannelAwareMessageListener {
    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        System.out.println(new String(message.getBody()));
        channel.basicAck(message.getMessageProperties().getDeliveryTag(),true);
    }
}
```



### 4、TTL

* TTL 为存活时间
* 当消息到达存活时间后，还没有被消费，会被自动清除。
* RabbitMQ 可以设置对消息的过期时间，也可以设置对整个队列的过期时间

> 配置

```xml
<!--TTL 队列-->
<rabbit:queue name="test_queue_ttl" id="test_queue_ttl">
    <!--
        指定队列的参数
        key="x-message-ttl" 设置队列的过期时间
        value="10000" 具体时间，毫米
        value-type="java.lang.Integer"  value的类型
    -->
    <rabbit:queue-arguments>
        <entry key="x-message-ttl" value="10000" value-type="java.lang.Integer"/>
    </rabbit:queue-arguments>
</rabbit:queue>

<!--TTL 交换机-->
<rabbit:topic-exchange name="test_exchange_ttl">
    <rabbit:bindings>
        <rabbit:binding pattern="ttl.*" queue="test_queue_ttl"/>
    </rabbit:bindings>
</rabbit:topic-exchange>
```

> 生产者测试发送数据

```java
@Test
public void test3() throws InterruptedException {
    for (int i = 0; i < 10; i++) {
        //队列统一的过期时间
        //rabbitTemplate.convertAndSend("test_exchange_ttl", "ttl.test", i + "message_TTL_Test");
        //设置单个消息的过期时间
        rabbitTemplate.convertAndSend("test_exchange_ttl", "ttl.test", i + "message_TTL_Test", new MessagePostProcessor() {
            /**
             * 消息后处理对象
             * @param message
             * @return
             * @throws AmqpException
             */
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                //设置消息的过期时间
                message.getMessageProperties().setExpiration("5000");
                return message;
            }
        });
    }
}
```



### 5、死信队列

死信队列，也是死性交换机，当消息成为 Dead message 后，可以被重新发送到另一个交换机，这个交换机就是 DLX。

消息成为死信的三种情况：

* 队列消息长度达到限制
* 消费者拒接消费信息，basicNack/basicReject，并且不把消息重新放回源目标队列，requeue=false
* 原队列存在消息过期设置，消息达到超时时间未被消费

队列绑定死信交换机的参数

* x-dead-letter-exchange：设置交换机的名称
* x-dead-letter-routing-key：相当于生产者向交换机发消息，需要一个 routingKey，从队列向死信队列发送消息也需要一个 routingkey

![image-20200925230058550](C:\Users\luming\AppData\Roaming\Typora\typora-user-images\image-20200925230058550.png)



> 声明死信队列的配置

```xml
<!--声明正常的队列和交换机-->
<rabbit:queue name="test_queue_dlx" id="test_queue_dlx">
    <!--为正常队列设置参数-->
    <rabbit:queue-arguments>
        <!--绑定死信交换机-->
        <entry key="x-dead-letter-exchange" value="exchange_dlx"/>
        <!--设置死信队列的 routingkey，此处的value需要遵守死信交换机的routingkey 即可-->
        <entry key="x-dead-letter-routing-key" value="dlx.aa"/>

        <!--成为死信消息的条件-->
        <!--设置队列过期时间-->
        <entry key="x-message-ttl" value="10000" value-type="java.lang.Integer"/>
        <!--设置队列长度限制-->
        <entry key="x-max-length" value="10" value-type="java.lang.Integer"/>
    </rabbit:queue-arguments>
</rabbit:queue>
<rabbit:topic-exchange name="test_exchange_dlx">
    <rabbit:bindings>
        <rabbit:binding pattern="text.dlx.#" queue="test_queue_dlx"/>
    </rabbit:bindings>
</rabbit:topic-exchange>

<!--声明死信队列和交换机,和普通队列没有区别-->
<rabbit:queue name="queue_dlx" id="queue_dlx"/>
<rabbit:topic-exchange name="exchange_dlx" id="exchange_dlx">
    <rabbit:bindings>
        <rabbit:binding pattern="dlx.#" queue="queue_dlx"/>
    </rabbit:bindings>
</rabbit:topic-exchange>
```



### 6、延迟队列

延迟队列，即消息进入队列不会被立马消费，而是只有达到指定时间后才会被消费。

但是 RabbitMQ 没有提供延迟队列，所以使用 TTL + 死信队列实现延迟队列。

![image-20200926094136645](C:\Users\luming\AppData\Roaming\Typora\typora-user-images\image-20200926094136645.png)



模拟一个订单系统和库存系统，当一个订单超过十秒未支付，才会被库存系统读取。

> 配置文件

```xml
<!--定义正常的交换机和队列-->
<rabbit:queue name="order_queue" id="order_queue">
    <!--设置正常队列的参数-->
    <rabbit:queue-arguments>
        <entry key="x-dead-letter-exchange" value="order_exchange_dlx"/>
        <entry key="x-dead-letter-routing-key" value="dlx.order.cancel"/>
        <entry key="x-message-ttl" value="10000" value-type="java.lang.Integer"/>
    </rabbit:queue-arguments>
</rabbit:queue>
<rabbit:topic-exchange name="order_exchange" id="order_exchange">
    <rabbit:bindings>
        <rabbit:binding pattern="order.#" queue="order_queue"/>
    </rabbit:bindings>
</rabbit:topic-exchange>

<!--定义死信的交换机和队列-->
<rabbit:queue name="order_queue_dlx" id="order_queue_dlx"/>
<rabbit:topic-exchange name="order_exchange_dlx" id="order_exchange_dlx">
    <rabbit:bindings>
        <rabbit:binding pattern="dlx.order.#" queue="order_queue_dlx"/>
    </rabbit:bindings>
</rabbit:topic-exchange>
```

注意：消费者需要监听死信队列。当十秒过去，消息会从普通队列进入死信队列，然后消费者获取消息。





## 二、RabbitMQ的应用问题

### 1、消息可靠性的保障

![image-20200926102727635](C:\Users\luming\AppData\Roaming\Typora\typora-user-images\image-20200926102727635.png)



### 2、消息幂等性保障

幂等性指一次请求或多次请求某个资源，对于资源本身应该有相同的结果。

在MQ中指，消费多条相同的消息，得到与消费该消息一次相同的结果。

使用乐观锁机制保障：

在向数据库交互时，携带应该版本号，只有到版本号和数据库中的版本号相等才会操作成功，每一次操作成功，数据库的版本号+1.