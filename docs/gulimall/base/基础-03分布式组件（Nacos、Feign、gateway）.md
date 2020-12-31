## 一、 注册中心Spring Cloud Alibaba Nacos

要注意nacos集群所在的server，一定要关闭防火墙，否则容易出现各种问题。

搭建nacos集群，然后分别启动各个微服务，将它们注册到Nacos中。

### 1.1 引入maven依赖

### 1.2 配置文件添加服务器地址

```yaml
  application:
    name: gulimall-coupon
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.137.14
```

### 1.3 SpringApplication启动类中增加注解

```java
@SpringBootApplication
@EnableDiscoveryClient
public class GulimallMemberApplication {
    public static void main(String[] args) {
        SpringApplication.run(GulimallMemberApplication.class, args);
    }
}
```



查看注册情况：

http://127.0.0.1:8848/nacos

![1587694451601](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc2OTQ0NTE2MDEucG5n?x-oss-process=image/format,png)

## 二、 RPC服务Spring Cloud Openfeign

### 2.1 引入maven依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 2.2 编写远程服务接口

修改“com.bigdata.gulimall.coupon.controller.CouponController”，添加以下controller方法：

```java
@RequestMapping("/member/list")
public R memberCoupons(){
    CouponEntity couponEntity = new CouponEntity();
    couponEntity.setCouponName("discount 20%");
    return R.ok().put("coupons",Arrays.asList(couponEntity));
}
```

新建“com.bigdata.gulimall.member.feign.CouponFeignService”接口

 声明接口的每一个方法都是调用哪个远程服务的那个请求

```java
@FeignClient("gulimall_coupon")
public interface CouponFeignService {
    @RequestMapping("/coupon/coupon/member/list")
    public R memberCoupons();
}
```

修改“com.bigdata.gulimall.member.GulimallMemberApplication”类，添加上"@EnableFeignClients"：

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients(basePackages = "com.bigdata.gulimall.member.feign")
public class GulimallMemberApplication {
    public static void main(String[] args) {
        SpringApplication.run(GulimallMemberApplication.class, args);
    }
}
```

### 2.3 开启远程调用功能

com.bigdata.gulimall.member.controller.MemberController

```java
    @RequestMapping("/coupons")
    public R test(){
        MemberEntity memberEntity=new MemberEntity();
        memberEntity.setNickname("zhangsan");
        R memberCoupons = couponFeignService.memberCoupons();

        return memberCoupons.put("member",memberEntity).put("coupons",memberCoupons.get("coupons"));
    }
```

### 2.4 测试

访问 http://localhost:8000/member/member/coupons

![1587701348764](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MDEzNDg3NjQucG5n?x-oss-process=image/format,png)

停止“gulimall-coupon”服务，能够看到注册中心显示该服务的健康值为0：

![1587701521184](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MDE1MjExODQucG5n?x-oss-process=image/format,png)

再次访问：http://localhost:8000/member/member/coupons

![1587701587456](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MDE1ODc0NTYucG5n?x-oss-process=image/format,png)

启动“gulimall-coupon”服务，再次访问，又恢复了正常。

## 三、 配置中心Spring Cloud Alibaba Nacos

### 3.1 修改“gulimall-coupon”模块

添加pom依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

创建bootstrap.properties文件，该配置文件会优先于“application.yml”加载。

```properties
spring.application.name=gulimall-coupon
spring.cloud.nacos.config.server-addr=192.168.137.14:8848
```

### 3.2 传统方式

为了详细说明config的使用方法，先来看原始的方式

创建“application.properties”配置文件，添加如下配置内容：

```properties
coupon.user.name="zhangsan"
coupon.user.age=30
```

修改“com.bigdata.gulimall.coupon.controller.CouponController”文件，添加如下内容：

```java
@Value("${coupon.user.name}")
private String name;
@Value("${coupon.user.age}")
private Integer age;

@RequestMapping("/test")
public R getConfigInfo(){
    return R.ok().put("name",name).put("age",age);
}
```

启动“gulimall-coupon”服务：

访问：http://localhost:7000/coupon/coupon/test

![1587716583668](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MTY1ODM2NjgucG5n?x-oss-process=image/format,png)

这样做存在的一个问题，如果频繁的修改application.properties，在需要频繁重新打包部署。下面我们将采用Nacos的配置中心来解决这个问题。

### 3.3 nacos config

1、在Nacos注册中心中，点击“配置列表”，添加配置规则：

![1587716911435](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MTY5MTE0MzUucG5n?x-oss-process=image/format,png)

DataID：gulimall-coupon

配置格式：properties

文件的命名规则为：s p r i n g . a p p l i c a t i o n . n a m e − {spring.application.name}-*spring.app**l**i**c**a**t**i**o**n*.*n**a**m**e*−{spring.profiles.active}.${spring.cloud.nacos.config.file-extension}

${spring.application.name}：为微服务名

${spring.profiles.active}：指明是哪种环境下的配置，如dev、test或info

${spring.cloud.nacos.config.file-extension}：配置文件的扩展名，可以为properties、yml等

2、查看配置：

![1587717125580](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MTcxMjU1ODAucG5n?x-oss-process=image/format,png)

3、修改“com.bigdata.gulimall.coupon.controller.CouponController”类，添加“@RefreshScope”注解

```java
@RestController
@RequestMapping("coupon/coupon")
@RefreshScope
public class CouponController {
```

这样都会动态的从配置中心读取配置.

4、访问：http://localhost:7000/coupon/coupon/test

能够看到读取到了nacos 中的最新的配置信息，并且在指明了相同的配置信息时

> 配置中心中设置的值优先于本地配置。

### 3.4 Nacos三种配置加载方案

Nacos支持“Namespace+group+data ID”的配置解决方案。

详情见：https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-docs/src/main/asciidoc-zh/nacos-config.adoc

##### Namespace方案

通过命名空间实现环境区分

下面是配置实例：

1、创建命名空间：

“命名空间”—>“创建命名空间”：

![1587718802109](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MTg4MDIxMDkucG5n?x-oss-process=image/format,png)

创建三个命名空间，分别为dev，test和prop

2、回到配置列表中，能够看到所创建的三个命名空间

![1587718889316](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MTg4ODkzMTYucG5n?x-oss-process=image/format,png)

下面我们需要在dev命名空间下，创建“gulimall-coupon.properties”配置规则：

![1587719108947](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MTkxMDg5NDcucG5n?x-oss-process=image/format,png)

3、访问：http://localhost:7000/coupon/coupon/test

![1587721184218](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjExODQyMTgucG5n?x-oss-process=image/format,png)

并没有使用我们在dev命名空间下所配置的规则，而是使用的是public命名空间下所配置的规则，这是怎么回事呢？

查看“gulimall-coupon”服务的启动日志：

```verilog
2020-04-24 16:37:24.158  WARN 32792 --- [           main] c.a.c.n.c.NacosPropertySourceBuilder     : Ignore the empty nacos configuration and get it based on dataId[gulimall-coupon] & group[DEFAULT_GROUP]
2020-04-24 16:37:24.163  INFO 32792 --- [           main] c.a.nacos.client.config.utils.JVMUtil    : isMultiInstance:false
2020-04-24 16:37:24.169  INFO 32792 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: [BootstrapPropertySource {name='bootstrapProperties-gulimall-coupon.properties,DEFAULT_GROUP'}, BootstrapPropertySource {name='bootstrapProperties-gulimall-coupon,DEFAULT_GROUP'}]

1234
```

**"gulimall-coupon.properties"**，默认就是public命名空间中的内容中所配置的规则。

4、指定命名空间

如果想要使得我们自定义的命名空间生效，需要在“bootstrap.properties”文件中，指定使用哪个命名空间：

```properties
spring.cloud.nacos.config.namespace=a2c83f0b-e0a8-40fb-9b26-1e9d61be7d6d
1
```

这个命名空间ID来源于我们在第一步所创建的命名空间

![1587718802109](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MTg4MDIxMDkucG5n?x-oss-process=image/format,png)

5、重启“gulimall-coupon”，再次访问：http://localhost:7000/coupon/coupon/test

![1587720311349](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjAzMTEzNDkucG5n?x-oss-process=image/format,png)

但是这种命名空间的粒度还是不够细化，对此我们可以为项目的每个微服务module创建一个命名空间。

6、为所有微服务创建命名空间

![1587720714101](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjA3MTQxMDEucG5n?x-oss-process=image/format,png)

7、回到配置列表选项卡，克隆pulic的配置规则到coupon命名空间下

![1587720883244](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjA4ODMyNDQucG5n?x-oss-process=image/format,png)

切换到coupon命名空间下，查看所克隆的规则：

![1587720963699](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjA5NjM2OTkucG5n?x-oss-process=image/format,png)

8、修改“gulimall-coupon”下的bootstrap.properties文件，添加如下配置信息

```properties
spring.cloud.nacos.config.namespace=7905c915-64ad-4066-8ea9-ef63918e5f79
```

这里指明的是，读取时使用coupon命名空间下的配置。

9、重启“gulimall-coupon”，访问：http://localhost:7000/coupon/coupon/test

![1587721184218](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjExODQyMTgucG5n?x-oss-process=image/format,png)

##### DataID方案

通过指定spring.profile.active和配置文件的DataID，来使不同环境下读取不同的配置，读取配置时，使用的是默认命名空间public，默认分组（default_group）下的DataID。

默认情况，Namespace=public，Group=DEFAULT GROUP，默认Cluster是DEFAULT

##### Group方案

通过Group实现环境区分

实例：通过使用不同的组，来读取不同的配置，还是以上面的gulimall-coupon微服务为例

1、新建“gulimall-coupon.properties”，将它置于“tmp”组下

![1587721616021](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjE2MTYwMjEucG5n?x-oss-process=image/format,png)

2、修改“bootstrap.properties”配置，添加如下的配置

```properties
spring.cloud.nacos.config.group=tmp
```

3、重启“gulimall-coupon”，访问：http://localhost:7000/coupon/coupon/test

![1587721844449](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjE4NDQ0NDkucG5n?x-oss-process=image/format,png)

### 3.5 同时加载多个配置集

当微服务数量很庞大时，将所有配置都书写到一个配置文件中，显然不是太合适。对此我们可以将配置按照功能的不同，拆分为不同的配置文件。

如下面的配置文件：

```yaml
server:
  port: 7000
spring:
  datasource:
    #MySQL配置
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.137.14:3306/gulimall_sms?useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: root
    password: root
  application:
    name: gulimall-coupon
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.137.14:8848

mybatis-plus:
  global-config:
    db-config:
      id-type: auto
  mapper-locations: classpath:/mapper/**/*.xml
```

我们可以将，数据源有关的配置写到一个配置文件中：

```yaml
spring:
  datasource:
    #MySQL配置
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.137.14:3306/gulimall_sms?useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: root
    password: root
```

和框架有关的写到另外一个配置文件中：

```yaml
mybatis-plus:
  global-config:
    db-config:
      id-type: auto
  mapper-locations: classpath:/mapper/**/*.xml
```

也可以将上面的这些配置交给nacos来进行管理。

实例：将“gulimall-coupon”的“application.yml”文件拆分为多个配置，并放置到nacos配置中心

1、创建“datasource.yml”，用于存储和数据源有关的配置

```yml
spring:
  datasource:
    #MySQL配置
    driverClassName: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.137.14:3306/gulimall_sms?useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: root
    password: root
1234567
```

在coupon命名空间中，创建“datasource.yml”配置

![1587722798375](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjI3OTgzNzUucG5n?x-oss-process=image/format,png)

2、将和mybatis相关的配置，放置到“mybatis.yml”中

```yaml
mybatis-plus:
  global-config:
    db-config:
      id-type: auto
  mapper-locations: classpath:/mapper/**/*.xml
12345
```

![1587722710432](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjI3MTA0MzIucG5n?x-oss-process=image/format,png)

3、创建“other.yml”配置，保存其他的配置信息

```yaml
server:
  port: 7000

spring:
  application:
    name: gulimall-coupon
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.137.14:8848
12345678910
```

![1587722998265](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjI5OTgyNjUucG5n?x-oss-process=image/format,png)

现在“mybatis.yml”、“datasource.yml”和“other.yml”共同构成了微服务的配置。

4、修改“gulimall-coupon”的“bootstrap.properties”文件，加载“mybatis.yml”、“datasource.yml”和“other.yml”配置

```properties
spring.cloud.nacos.config.extension-configs[0].data-id=mybatis.yml
spring.cloud.nacos.config.extension-configs[0].group=dev
spring.cloud.nacos.config.extension-configs[0].refresh=true

spring.cloud.nacos.config.extension-configs[1].data-id=datasource.yml
spring.cloud.nacos.config.extension-configs[1].group=dev
spring.cloud.nacos.config.extension-configs[1].refresh=true


spring.cloud.nacos.config.extension-configs[2].data-id=other.yml
spring.cloud.nacos.config.extension-configs[2].group=dev
spring.cloud.nacos.config.extension-configs[2].refresh=true
123456789101112
```

"spring.cloud.nacos.config.ext-config"已经被废弃，建议使用“spring.cloud.nacos.config.extension-configs”

5、注释“application.yml”文件中的所有配置

6、重启“gulimall-coupon”服务，然后访问：http://localhost:7000/coupon/coupon/test

![1587724212905](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjQyMTI5MDUucG5n?x-oss-process=image/format,png)

7、访问：http://localhost:7000/coupon/coupon/list，查看是否能够正常的访问数据库

![1587724350548](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MjQzNTA1NDgucG5n?x-oss-process=image/format,png)

小结：

1)、微服务任何配置信息，任何配置文件都可以放在配置中心；

2)、只需要在bootstrap.properties中，说明加载配置中心的哪些配置文件即可；

3)、@Value, @ConfigurationProperties。都可以用来获取配置中心中所配置的信息；

4)、配置中心有的优先使用配置中心中的，没有则使用本地的配置。

## 四、 网关Spring Cloud gateway

### 4.1、注册“gulimall-gateway”到Nacos

1、创建“gulimall-gateway”

SpringCloud gateway

2、添加“gulimall-common”依赖和“spring-cloud-starter-gateway”依赖

```xml
<dependency>
    <groupId>com.bigdata.gulimall</groupId>
    <artifactId>gulimall-common</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

3、 “com.bigdata.gulimall.gulimallgateway.GulimallGatewayApplication”类上加上“@EnableDiscoveryClient”注解

4、在Nacos中创建“gateway”命名空间，同时在该命名空间中创建“gulimall-gateway.yml”

![1587729576178](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3Mjk1NzYxNzgucG5n?x-oss-process=image/format,png)

5、创建“bootstrap.properties”文件，添加如下配置，指明配置中心地址和所属命名空间

```properties
spring.application.name=gulimall-gateway
spring.cloud.nacos.config.server-addr=192.168.137.14:8848
spring.cloud.nacos.config.namespace=1c82552e-1af0-4ced-9a48-26f19c2d315f
```

6、创建“application.properties”文件，指定服务名和注册中心地址

```properties
spring.application.name=gulimall-gateway
spring.cloud.nacos.discovery.server-addr=192.168.137.14:8848
server.port=88
```

7、启动“gulimall-gateway”

启动报错：

```verilog
Description:
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.
Reason: Failed to determine a suitable driver class
```

解决方法：在“com.bigdata.gulimall.gulimallgateway.GulimallGatewayApplication”中排除和数据源相关的配置

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
```

重新启动

访问：http://192.168.137.14:8848/nacos/#，查看到该服务已经注册到了Nacos中

![1587730035866](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9mZXJtaGFuLm9zcy1jbi1xaW5nZGFvLmFsaXl1bmNzLmNvbS9ndWxpLzE1ODc3MzAwMzU4NjYucG5n?x-oss-process=image/format,png)

### 4.2、案例

现在想要实现针对于“http://localhost:88/hello?url=baidu”，转发到“https://www.baidu.com”，针对于“http://localhost:88/hello?url=qq”的请求，转发到“https://www.qq.com/”

1、创建“application.yml”

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: baidu_route
          uri: https://www.baidu.com
          predicates:
            - Query=url, baidu
        - id: qq_route
          uri: https://www.qq.com/
          predicates:
            - Query=url, qq
```

2、启动“gulimall-gateway”

3、测试

访问：http://localhost:88/hello?url=baidu

访问：http://localhost:88/hello?url=qq