## 一、MQ概述

- 1、大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力
- 2、消息服务中两个重要概念
  消息代理（message broker）和目的地（destination）
  当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目
  的地
- 3、消息队列主要有两种形式的目的地
  - 1、队列（queue）：点对点消息通信（point-to- point）
  - 2、主题（topic）：发布（publish）/订阅（subscribe）消息通信
- 4、点对点式
  消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容
  消息读取后被移出队列
  消息只有唯一的发送者和接受者，但并不是说只能有一个接收者
- 5、发布订阅式
  发送者(发布者）发送消息到主题，多个接收者(订阅者）监听(订阅）这个主题，那么
  就会在消息到达时同时收到消息
- 6、JMS（Java Message Service）JAVA消息服务
  基于JVM消息代理的规范。 ActiveMQ、 HornetMQ是JMS实现
- 7、AMQP(Advanced Message Queuing Protocol）
  -- 高级消息队列协议，也是一个消息代理的规范，兼容JMS
  -- RabbitMQ是AMQP的实现

|              | JMS                                                          | AMQP                                                         |
| ------------ | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 定义         | Java api                                                     | Wire-protocol                                                |
| 跨语言       | 否                                                           | 是                                                           |
| 跨平台       | 否                                                           | 是                                                           |
| Model        | 提供两种消息模型：<br />（1）、Peer-2-Peer<br />（2）、Pub/sub | 提供了五种消息模型：<br />（1）、direct exchange<br />（2）、fanout exchange<br />（3）、topic change<br />（4）、headers exchange<br />（5）、system exchange<br />本质来讲，后四种和JMS的pub/sub模型没有太大差别，仅是在路由机制上做了更详细的划分； |
| 支持消息类型 | 多种消息类型：<br />TextMessage<br />MapMessage<br />BytesMessage<br />StreamMessage<br />ObjectMessage<br />Message （只有消息头和属性） | byte[]<br />当实际应用时，有复杂的消息，可以将消息序列化后发送。 |
| 综合评价     | JMS 定义了JAVA API层面的标准；在java体系中，多个client均可以通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差； | AMQP定义了wire-level层的协议标准；天然具有跨平台、跨语言特性。 |

- 8、Spring支持
  - spring-Jms提供了对JMS的支持
  - spring-rabbit提供了对AMQP的支持
  - 需要 Connection Factory的实现来连接消息代理
  - 提供 JmsTemplate、 RabbitTemplate来发送消息
  - @ JmsListener（JMS）、@ RabbitListener（AMQP）注解在方法上监听消息代理发
    布的消息
  - @ EnableJms、@ EnableRabbit开启支持

- 9、Spring booti自动配置

  - JmsAutoConfiguration

  - RabbitAutoConfiguration

- 10、市面的MQ产品
    - Active MC、 RabbitMQ、 RocketMQ、 Kafka



## 二、RabbitMQ概念

[>>>相关链接<<<](https://blog.csdn.net/bestmy/article/details/84304964)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201228163444.png)



## 三、安装RabbitMQ

> Docker

- 4369,25672: Erlang 发现 & 集群端口
- 5672,5671: AMQP端口
- 15672: Web管理后台端口
- 1883,8883: MQTT协议端口
- 61613, 61614: STOMP协议端口

```shell
$ docker pull rabbitmq
$ docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management
```

>  手动下载安装

```shell
$ cd 安装目录
$ # 启用 rabbitmq management插件
$ sudo sbin/rabbitmq-plugins enable rabbitmq_management
$ # 配置环境变量(可选)
$ rabbitmq-server -detached # 后台启动
# 查看状态 浏览器内输入 http://localhost:15672, 默认的用户名密码都是 guest。
$ rabbitmqctl status
$ rabbitmqctl stop # 关闭
```
```shell
# Setting for RabbitMQ
export RABBIT_HOME=/usr/local/Cellar/rabbitmq/3.8.9_1
export PATH=$PATH:$RABBIT_HOME/sbin
```



## 四、RabbitMQ运行机制

 　　AMQP中消息的路由过程和Java开发者熟悉的JMS存在一些差别，AMQP中增加了Exchange和Binding的角色，生产者把消息发布到Exchange上，Binding决定发布到Exchange上的消息应该发送到那个队列上，消息最终到达队列并被消费者接收。

> Exchange（交换器）类型

　　Exchange分发消息时根据类型的不同分发策略也不相同，目前共有4种类型：direct（默认，点对点，提前预知性的绑定）、fanout（群发性的绑定，类似于多播）、topic（正则，归类性的绑定）、headers（and、or的绑定），headers匹配消息的header而不是routing-key（路由键），除此之外，headers交换器和direct交换器完全一致，但性能差好多，目前几乎已经不用了。

　　1.Direct Exchange（默认，点对点）

　　消息中的routing-key（路由键）如果和Binding中的binding-key一致，交换器就将该消息发到对应的队列中。如果一个队列绑定到交换器要求路由键为“dog”，则只转发routing-key标记为dog的消息，它是完全匹配、单播的模式。

　　2.Fanout Exchange（类似于多播）

　　每个发到fanout类型交换器的消息都会分到所有绑定的队列上去，fanout交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上，很像子网广播，每台子网内的主机都获得了一份复制的消息，fanout类型转发消息是最快的。

　　队列中的数据无论多少个队列，其数据一致。

　　使用场景：

　　　　1）、订单流程，如订单提交后，同时向客户发送短信及邮件等。

　　　　2）、C/S软件弹出消息，通过轮询的方式，在C/S中绑定fanout Exchange，这时候服务器有消息的话，可以及时推送。

　　　　3）、类似于淘宝的部分流程，如催付，付款后提醒，发货提醒，签收提醒，如给用户关联的推荐使用短信和邮件分别发送等。

　　　　　　。。。。。

　　3.Topic Exchange（通配符匹配）

　　binding-key支持通配符，有两个通配符，“#”代表0个或多个单词，“*”代表一个单词，若消息的routing-key与之匹配，则将消息发至该队列。

　　4.Headers Exchange（Headers匹配）

　　headers采用muliple attribute来替代routing-key，通过设置headers中的x-match属性为all或any进行匹配，all：所有的header头信息必须匹配。any：只要有一个匹配就可以了。



## 五、RabbitMQ整合

### 5.1 引入 spring-boot-starter-amqp

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

> 全局搜索自动配置类 **AuroConfiguration
>
> 给容器中自动配置了一些bean

### 5.2 application.yml 配置

```java
@ConfigurationProperties(prefix = "spring.rabbitmq")
```

```yaml
spring:
	rabbitmq:
		host: 192.168.56.10
		port: 5672
		virtual-host: /
```

### 5.3 @EnableRabbit



## 六、RabbitNQ使用

### 6.1 AmqpAdmin管理组件

```java
import org.springframework.amqp.core.AmqpAdmin;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
......
    
@Autowired
AmqpAdmin amqpAdmin;

public void test (){
    // 创建点对点交换机
    // name, durable[持久化], autoDelete, arguments
    DirectExchange directExchange = new DirectExchange("hello-java-exchange", true, false, null);
    amqpAdmin.declareExchange(directExchange);

    // 创建队列
    // name, durable, exclusive[排他，只能连一个交换机], autoDelete, arguments
    Queue queue = new Queue("hello-java-queue", true, false, false, null);
    amqpAdmin.declareQueue(queue);

    // 将exchange指定的交换机和destination目的地进行绑定，使用routingKey作为指定的路由键
    // destination[目的地], destinationType[目的地类型], exchange, routingKey, arguments
    Binding binding = new Binding("hello-java-queue",  Binding.DestinationType.QUEUE, "hello-java-exchange", "hello.java", null);
    amqpAdmin.declareBinding(binding);
}
```



### 6.2 RabbitTemplate

> 序列化配置

```java
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyRabbitConfig {

    @Bean
    public MessageConverter messageConverter() {
        // 【如果发送消息是个对象，会使用序列化机制将对象写出去，所以对象必须实现Serializable】
        // 自定义json序列化
        return new Jackson2JsonMessageConverter();
    }
}
```



> 发送消息

```java
import org.springframework.amqp.rabbit.core.RabbitTemplate;
......

@Autowired
RabbitTemplate rabbitTemplate;

public void sendMessage() {
    // 转换且发送消息
    // String exchange, String routingKey, Object message, CorrelationDat[消息唯一ID]
    rabbitTemplate.convertAndSend("hello-java-exchange", "hello.java", "Hello World", new CorrelationData(UUID.randomUUID().toString());
}
```



> 接收消息 

- @RabbitListener 标记在方法上

```java
import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
......
    
// 使用@RabbitListener：必须有@EnableRabbit且必须处于容器中
// 生命监听的所有队列（数组）
@RabbitListener(queues = {"hello-java-queue"})
public void receiveMessage(Message message, MyEntity myEntity, Channel channel) {
    // 1、 接收消息类型 org.springframework.amqp.core.Message
    byte[] body = message.getBody();
    // 将body转化为对象......
    // 消息的属性头信息
    MessageProperties messageProperties = message.getMessageProperties();

    // 2、直接用传递的参数类型直接接收
    // myEntity......spring自动转化

    // 3、使用当前传输数据的通道
    // channel.***
}
```

- @RabbitListener 标记在类上，@RabbitHandler 标记在方法上

  方法重载，根据接收参数的不同进行不同的处理

```java
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
......

@RabbitListener(queues = {"hello-java-queue"})
public class mq {

    @RabbitHandler
    public void receiveMessage(MyEntity myEntity) {}

    @RabbitHandler
    public void receiveMessage2(MyOrder myOrder) {}
}
```



## 七、消息确认机制-可靠抵达

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201229142657.png)

> 官方介绍
>
> Using standard AMQP 0-9-1, the only way to guarantee that a message isn't lost is by using transactions -- make the channel transactional then for each message or set of messages publish, commit. In this case, transactions are unnecessarily heavyweight and decrease throughput by a factor of 250. To remedy this, a confirmation mechanism was introduced. It mimics the consumer acknowledgements mechanism already present in the protocol.
>
> （保证消息不丢失可以使用事务消息，但性能会下降250倍，为此引入确认机制）

RabbitMQ的消息确认有两种。

**一种是消息发送确认**。这种是用来确认生产者将消息发送给交换器，交换器传递给队列的过程中，消息是否成功投递。发送确认分为两步，一是确认是否到达交换器（confirmCallback），二是确认是否到达队列（returnCallback）。

**第二种是消费接收确认**。这种是确认消费者是否成功消费了队列中的消息（ack）。

### 7.1 生产端confirmCallback和returnCallback

```yaml
# spring-boot中配置
spring:
	rabbitmq:
		# 开启发送端确认  confirmCallback
		publisher-confirms: true
		#######################################
		# 开启发送消息抵达队列确认  returnCallback
		publisher-returns: true
		# 只要抵达队列，以异步发送优先回调returnConfirm
		template.mandatory: true
```

```java
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;

@Configuration
public class MyRabbitConfig {

    @Bean
    public MessageConverter messageConverter() {
        // 【如果发送消息是个对象，会使用序列化机制将对象写出去，所以对象必须实现Serializable】
        // 自定义json序列化
        return new Jackson2JsonMessageConverter();
    }

    @Autowired
    RabbitTemplate rabbitTemplate;

    /**
     * RabbitCallback是RabbitTemplate类中的一个内部接口
     * 定制RabbitTemplate
     **/
    @PostConstruct // 对象创建完成后执行此方法
    public void initRabbitTemplate(){
        // 设置确认回调
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            /**
             * 只要消息抵达Broke就ack=true
             * @param correlationData 当前消息的唯一关联数据（消息唯一ID，在发送时指定）
             * @param ack 消息是否成功收到
             * @param cause 失败的原因
             **/
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                // do something .......
            }
        });

        // 设置消息抵达队列的确认回调
        rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback() {
            /**
             * 只要消息没有投递给指定队列，就触发这个回调
             * @param message 投递失败的消息详情
             * @param replyCode 回复的状态码
             * @param replyText 失败的原因
             * @param exchange 当时这个消息发送给哪个交换机
             * @param routingKey 当时消息用的哪个路由键
             **/
            @Override
            public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
                // do something .......
            }
        });
    }
}
```



### 7.3 消费端ack

> 保证每个消息被正确消费，此时才可以broker删除这个消息
>
> 默认是自动确认的，只要消息收到，客户端会自动确认，服务端就会移除这个消息
>
> - 问题：只成功处理一个消息后宕机，队列中消息被完全删除，发生消息丢失
> - 所以需要手动确认【只要我们没有告诉MQ签收，没有ack，消息就不会丢失，就算宕机，消息也会重新编程Ready，下次有新的Consumer连接进来发给它】

（1）确认模式

- AcknowledgeMode.NONE：不确认
- AcknowledgeMode.AUTO：自动确认
- AcknowledgeMode.MANUAL：手动确认

```yaml
# spring-boot中配置
spring:
	rabbitmq:
		# 手动ack确认
		listener:
			simple:
				acknowledge-mode: manual
```

```java
@RabbitListener(queues = {"hello-java-queue"})
public void receiveMessage(Message message, Channel channel) {
    long deliveryTag = message.getMessageProperties().getDeliveryTag();
    try {
        if(deliveryTag%2==0){
            /**
             * 手动签收确认
              @param deliveryTag 交货标签 channel 内按顺序自增
             * @param multiple 是否批量签收
             **/
            channel.basicAck(deliveryTag,false);
        }else {
            // 拒绝签收
            // long deliveryTag, boolean multiple, boolean requeue[重新入队或直接丢弃]
            channel.basicNack(deliveryTag,false,true);
            // basicNack可以批量，basicReject不能
            // long deliveryTag, boolean requeue[重新入队或直接丢弃]
            channel.basicReject(deliveryTag,true);
        }
    } catch (IOException e) {
        // 网络中断....
        e.printStackTrace();
    }
}
```



## 八、附录

### 1、端口占用查询

```shell
// ---cmd窗口---
// 查看所有端口
$ netstat -ano
// 筛选特定端口
$ netstat -ano|findstr 3306
  TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       7992
  TCP    [::]:3306              [::]:0                 LISTENING       7992

// 查询进程
$ tasklist|findstr 7992
mysqld.exe                    7992 Services                   0      1,556 K
```

