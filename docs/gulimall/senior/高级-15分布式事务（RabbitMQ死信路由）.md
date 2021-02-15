## 一、本地事务

### 1.1 事务的基本性质

数据库事务的几个特性：原子性（ Atomicity）、一致性（ Consistency）、隔离性或独立性（ Isolation）和持久性（Durabilily），简称就是ACID。

- 原子性：一系列的操作整体不可拆分，要么同时成功，要么同时失败。
- 一致性：数据在事务的前后，业务整体一致。
  - 转账。A：1000   B：1000 ；   转200   事务成功；  A：800    B：1200
- 隔离性：事务之间互相隔离。
- 持久性：一旦事务成功，数据一定会落盘在数据库。


在以往的单体应用中，我们多个业务操作使用同一条连接操作不同的数据表，一旦有异常，我们可以很容易的整体回滚。

![image-20210212171528139](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210212171528139.png)

>一个事务开始，代表以下的所有操作都在同一个连接里面。



### 1.2 事务的隔离级别

- READ UNCOMMITTED（读未提交）
  该隔离级别的事务会读到其它未提交事务的数据，此现象也称之为脏读。

- READ COMMITTED（读提交）
  一个事务可以读取另一个已提交的事务，多次读取会造成不一样的结果，此现象称为不可重复读问题， Oracle 和 SQL Server 的默认隔离级别

- REPEATABLE READ（可重复读）
  该隔离级别是 MySQL 默认的隔离级别，在同一个事务里， select的结果是事务开始时时间点的状态，因此，同样的select操作读到的结果会是一致的，但是，会有幻读现象。MySQL的 InnoDB 引擎可以通过 next-key locks 机制（参考下文"行锁的算法"一节）来避免幻读。

```java
// 调整事务隔离级别（可重复读）
@Transaction(isolation = Isolation.REPEATABLE_READ)
```

- SERIALIZABLE（序列化）
  在该隔离级别下事务都是串行顺序执行的，MySQL数据库的 InnoDB 引擎会给读操作隐式
  加一把读共享锁，从而避免了脏读、不可重读复读和幻读问题

  

### 1.3 事务的传播行为

![image-20210212190829533](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210212190829533.png)

1、 PROPAGATION REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务
就加入该事务，该设置是最常用的设置。
2、 PROPAGATION SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当
前不存在事务，就以非事务执行。
3、 PROPAGATION MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果
当前不存在事务，就抛出异常。
4、 PROPAGATION REQUIRES NEW：创建新事务，无论当前存不存在事务，都创建新事务。
5、 PROPAGATION NOT SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当
前事务挂起。
6、 PROPAGATION NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
7、 PROPAGATION NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务
则执行与 PROPAGATION REQUIRED类似的操作。



### 1.4 SpringBoot 事务

#### 1.4.1 事务的自动配置

TransactionAutoConfiguration

#### 1.4.2 本地事务失效问题

> 同一个对象事务方法互调默认失效，因为事务是使用代理对象来控制的，而同一个对象内绕过了代理对象。

- 解决方式：使用代理对象来调用事务方法
- 1）、引入 aop-starter；spring-boot-starter-aop； 引入了aspectj
- 2）、@EnableAspectJAutoProxy(exposeProxy = true) --对外暴露代理对象；开启aspectj动态代理功能。以后所有的动态代理都是aspectj创建的（即使没有接口也可以创建动态代理）。

![image-20210212191922826](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210212191922826.png)



## 二、分布式事务

> 本地事务，在分布式系统中，只能控制自己的回滚（@Transaction），控制不了其他服务的回滚。
>
> 分布式事务：最大原因。网络问题+分布式机器。

![image-20210212170802055](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210212170802055.png)

### 2.1 为什么有分布式事务

> 分布式系统经常出项异常：机器宕机、网络异常、消息丢失、消息乱序、数据错误、不可靠的TCP、存储数据丢失...
>
> 分布式事务是企业集成中的一个技术难点，也是每一个分布式系统架构中都会涉及的一个东西，热别是在微服务架构中，几乎可以说是无法避免的。

![image-20210212192511354](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210212192511354.png)



### 2.2 CAP 定理与 BASE 理论

#### 2.2.1 CAP 定理

CAP原则又称CAP定理，指的是在一个分布式系统中

- 一致性（Consistency）
  - 在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访
    问同一份最新的数据副本）
- 可用性（Availability）
  - 在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据
    更新具备高可用性）
- 分区容错性（Partition tolerance）
  - 大多数分布式系统都分布在多个子网络。每个子网络就叫做一个区（partition）。
    分区容错的意思是，区间通信可能失败。比如，一台服务器放在中国，另一台服务
    器放在美国，这就是两个区，它们之间可能无法通信。

> CAP原则指的是，这三个要素最多只能同时实现两点，不可能三者兼顾。
>
> 一般来说，分区容错无法避免，因此可以认为CAP的 P 总是成立。所以C和A无法同时做到。

![image-20210212193616921](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210212193616921.png)

> 分布式系统中实现一致性（C、P）的 raft 算法。
>
> [http://thesecretlivesofdata.com/raft/](http://thesecretlivesofdata.com/raft/)

#### 2.2.2 面临问题

对于多数大型互联网应用的场景，主机众多、部署分散，而且现在的集群规模越来越大，所以节点故障、网络故障是常态，而且要保证服务可用性达到99999（N个9），即保证P和A，舍弃C。

#### 2.2.3 BASE 理论

是对CAP理论的延伸，思想是即使无法做到强一致性（CAP的一致性就是强一致性），但可以采用适当的采取弱一致性，即最终一致性。

BASE是指

- 基本可用（Basically Available）
  - 基本可用是指分布式系统在出现故障的时候，允许损失部分可用性（例如响应时间功能上的可用性），允许损失部分可用性。需要注意的是，基本可用绝不等价于系统不可用。
    - 响应时间上的损失：正常情况下搜索引擎需要在05秒之内返回给用户相应的查询结果，但由于出现故障（比如系统部分机房发生断电或断网故障），查询结果的响应时间增加到了1~2秒。
    - 功能上的损失：购物网站在购物高峰（如双十一）时，为了保护系统的稳定性，部分消费者可能会被引导到一个降级页面。
  
- 软状态（Soft State）

  - 软状态是指允许系统存在中间状态，而该中间状态不会影响系统整体可用性。分布式存储中一般一份数据会有多个副本，允许不同副本同步的延时就是软状态的体现。 mysql replication 的异步复制也是一种体现。
  
- 最终一致性（Eventual Consistency）

    - 最终一致性是指系统中的所有数据副本经过一定时间后，最终能够达到一致的状态。弱一致性和强一致性相反，最终一致性是弱一致性的一种特殊情况。

#### 2.2.4 强一致性、弱一致性、最终一致性

从客户端角度，多进程并发访问时，更新过的数据在不同进程如何获取的不同策略，决定了不同的一致性。对于关系型数据库，要求更新过的数据能被后续的访问都能看到，这是强一致性。如果能容忍后续的部分或者全部访问不到，则是弱一致性。如果经过一段时间后要求能访问到更新后的数据，则是最终一致性。

###  2.3 分布式事务几种方案

#### 2.3.1 2PC 模式

数据库支持的2PC【2 phase commit 二阶提交】，又叫做 XA Transactions。
MySQL从5.5版本开始支持， SQL Server 2005开始支持， Oracle7 开始支持。
其中，XA是一个两阶段提交协议，该协议分为以下两个阶段：
第一阶段：事务协调器要求每个涉及到事务的数据库预提交（precommit）此操作，并反映是
否可以提交。
第二阶段：事务协调器要求每个数据库提交数据。
其中，如果有任何一个数据库否决此次提交，那么所有数据库都会被要求回滚它们在此事务中的那部分信息。

![image-20210213122336109](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213122336109.png)

- XA协议比较简单，而且一旦商业数据库实现了XA协议，使用分布式事务的成本也比较低。
- XA性能不理想，特别是在交易下单链路，往往并发量很高，XA无法满足高并发场景。
- XA目前在商业效据库支持的比较理想，在 mysql 数据库中支持的不太理想，mysql 的XA实现，没有记录 prepare 阶段日志，主备切换会导致主库与备库数据不一致。
- 许多 nosql 也没有支持XA，这让XA的应用场景变得非常狭隘。
- 也有3PC（三次提交），引入了超时机制（无论协调者还是参与者，在向对方发送请求后，若长时间未收到回应则做出相应处理）

#### 2.3.2 柔性事务-TCC事务补偿型方案

刚性事务：遵循ACD原则，强一致性。
柔性事务：遵循BAsE理论，最终一致性。
与刚性事务不同，柔性事务允许一定时间内，不同节点的数据不一致，但要求最终一致。

![image-20210213122916270](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213122916270.png)

一阶段 prepare 行为：调用自定义的 prepare 逻辑。
二阶段 commit 行为：调用自定义的 commit 逻辑。
二阶段 ro‖back 行为：调用自定义的 rollback 逻辑
所谓TCC模式，是指支持把自定义的分支事务纳入到全局事务的管理中。

#### 2.3.3 柔性事务-最大努力通知型方案

按规律进行通知，不保证数据一定能通知成功，但会提供可查询操作接囗进行核对。这种方案主要用在与第三方系统通讯时，比如：调用微信或支付宝支付后的支付结果通知。这种方案也是结合MQ进行实现，例如：通过 MQ 发送 http 请求，设置最大通知次数。达到通知次数后即不再通知。

案例：银行通知、商户通知等（各大交易业务平台间的商户通知：多次通知、查询校对、对账文件），支付宝的支付成功异步



#### 2.3.4 柔性事务-可靠消息+最终一致性方案（异步确保型）

实现：业务处理服务在业务事务提交之前，向实时消息服务请求发送消息，实时消息服务只记录消息数据，而不是真正的发送。业务处理服务在业务事务提交之后，向实时消息服务确认发送。只有在得到确认发送指令后，实时消息服务才会真正发送。



## 三、SEATA（SpringCloud Alibaba 组件）

> [STETA 官网](https://seata.io/zh-cn/docs/overview/what-is-seata.html)

![image-20210213125519856](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213125519856.png)

### 3.1 使用 SEATA

> AT 模式，二阶段提交的变种，不适用于高并发

- 引入依赖 spring-cloud-starter-alibaba-seata

- 每个微服务需要一个 udo_log 表

- 安装事务协调器并启动：seata-server （版本对应依赖包引入的 seata-all 版本），修改 conf 文件夹下配置（registry.conf、file.conf）

- 开启全局事务（分布式大事务的入口业务代码上 @GlobalTransaction，每一个小事务中标注 @Transaction ）

- 所有想要用到分布式事务的微服务使用 seata DataSourceProxy 代理自己的数据源

  ![image-20210213131140586](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213131140586.png)

- 每个微服务都需要导入 registry.conf 、file.conf

  ![image-20210213131422130](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213131422130.png)

![image-20210213131516617](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213131516617.png)

## 四、高并发下最终一致性事务方案

![image-20210213132914684](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213132914684.png)

### 4.1 定时任务缺点

![image-20210213133315351](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213133315351.png)

![image-20210213133409972](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213133409972.png)



### 4.2 RabbitMQ消息延时队列

> 使用死信路由的过期时间（TTL），实现延时任务。

![image-20210213133834726](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213133834726.png)

#### 4.2.1 消息的TTL（Time To Live）

- 消息的TTL就是消息的存活时间。
- RabbitMQ 可以对队列和消息分别设置TTL
  - 对队列设置就是队列没有消费者连着的保留时间，也可以对毎一个单独的消息做单独的设置。超过了这个时间，我们认为这个消息就死了，称之为死信。
  - 如果队列设置了，消息也设置了，那么会取小的。所以一个消息如果被路由到不同的队列中，这个消息死亡的时间有可能不一样（不同的队列设置）。这里单讲单个消息的TTL，因为它才是实现延迟任务的关键。可以通过设置消息的 expiration 字段或者 x- message-ttl 属性来设置时间，两者是一样的效果。

#### 4.2.2 Dead Letter Exchanges （DLX）

- 一个消息在满足如下条件下，会进死信路由，记住这里是路由而不是队列，一个路由可以对应很多队列。(什么是死信)

  - 一个消息被 Consumer 拒收了，并且 reject 方法的参数里 requeue 是 false，也就是说不会被再次放在队列里，被其他消费者使用。( basic.reject / basic.nack) requeue=false

  - 上面的消息的TTL到了，消息过期了。

  - 队列的长度限制满了。排在前面的消息会被丢弃或者扔到死信路由上。

- Dead Letter Exchange 其实就是一种普通的 exchange，和创建其他 exchange 没有两样。只是在某一个设置 Dead Letter Exchange 的队列中有消息过期了，会自动触发消息的转发，发送到 Dead Letter Exchange 中去。
- 我们既可以控制消息在一段时间后变成死信，又可以控制变成死信的消息被路由到某一个指定的交换机，结合二者，其实就可以实现一个延时队列。

#### 4.2.3 延时队列实现

![image-20210213135422328](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213135422328.png)



![image-20210213135349551](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213135349551.png)

![image-20210213135928508](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213135928508.png)

#### 4.2.4 RabbitMQ 配置

> 使用 @Bean 将 Bingding、Queue、Exchange 创建出来
>
> ！！！ RabbitMQ 中一旦使用 @Bean 创建了组件（队列、交换机、绑定），属性改变后也不会覆盖原来的组件。只能先删除原有的组件。
>
> ！！！只有在程序第一次连上 RabbitMQ 服务器监听消息的时候发现组件不存在才会创建相关组件。

![image-20210213141134919](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210213141134919.png)

### 4.3 保证消息可靠性

#### 4.3.1 消息丢失

- 消息发送出去，由于网络问题没有抵达服务器
  - 做好容错方法（try-catch），发送消息可能会网络失败，失败后要有重试机制，可记录到数据库，采用定期扫描重发的方式
  - 做好日志记录，每个消息状态是否都被服务器收到都应该记录
  - 做好定期重发，如果消息没有发送成功，定期去数据库扫描末成功的消息进行重发
- 消息抵达 Broker,，Broker 要将消息写入磁盘(持久化)才算成功。此时 Broker 尚未持久化完成，宕机
  - publisher 也必须加入确认回调机制，确认成功的消息，修改数据库消息状态。
- 自动ACK的状态下。消费者收到消息，但没来得及消息然后宕机
  - 一定开启手动ACK，消费成功才移除，失败或者没来得及处理就 noAck 并重新入队

> 防止消息丢失最重要
>
> 1、做好消息确认机制（publisher，consumer【手动ack】）
>
> 2、每一条发送的消息数据库做好记录，定期将失败的消息再发送一次

#### 4.3.2 消息重复

- 消息消费成功，事务已经提交，ack时，机器宕机。导致没有ack成功,
  Broker的消息重新由 unack 变为 ready，并发送给其他消费者
- 消息消费失败，由于重试机制，自动又将消息发送出去
- 成功消费，ack 时宕机，消息由 unack  变为 ready,，Broker又重新发送
  - 消费者的业务消费接口应该设计为幂等性的。比如扣库存有工作单的状态标志
  - 使用防重表（redis/mysq），发送消息每一个都有业务的唯一标识，处理过就不用处理
  - rabbitMQ 的每一个消息都有 redelivered 字段，可以获取是否是被重新投递过来的，而不是第一次投递过来的

#### 4.3.3 消息积压

- 消费者宕机积压
- 消费者消费能力不足积压
- 发送者发送流量太大
  - 上线更多的消费者，进行正常消费
  - 上线专门的队列消费服务，将消息先批量取出来，记录数据库，离线慢慢处理

