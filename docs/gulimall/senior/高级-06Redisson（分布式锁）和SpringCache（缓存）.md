## 一、Redisson

- [官网文档地址](https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95)

### 1、入门配置

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110200356.png)

引入依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.12.0</version>
</dependency>
```

配置文件

```java
import org.redisson.Redisson;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.io.IOException;

@Configuration
public class MyRedissonConfig {

    /**
     * 所有对Redisson的是用都是通过RedissonClient对象
     **/
    @Bean(destroyMethod="shutdown")
    public RedissonClient redisson() throws IOException {
        Config config = new Config();
        // redisson集群模式
//        config.useClusterServers()
//                .addNodeAddress("127.0.0.1:7004", "127.0.0.1:7001");
        // redisson单节点模式
        config.useSingleServer().setAddress("redis://192.168.56.10:6379");
        // 根据config创建出RedissonClient实例
        return Redisson.create(config);
    }

}
```

### 2、可重入锁-看门狗

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110213728.png)

#### 2.1 默认加锁时间

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110205020.png)

#### 2.2 手动加锁

> lock.lock(10, TimeUnit.SECONDS); // 加锁以后10秒钟自动解锁
>
> **手动加锁时，不会自动续期，自动解锁时间一定要大于业务的执行时间！！！**

- 1、如果传递了锁的超时时间，源码中发送给redis一个执行脚本，进行占锁，默认超时就是指定的时间
- 2、如果未指定锁的超时时间，就是用30 * 1000【lockWatchdogTimeout看门狗的默认时间】；只要占锁成功，就会启动一个定时任务【重新给锁设置多起事件，新的过期时间就是看门狗的默认时间】，每隔10s都会自动再次续期，续成30s满时间。internalLockLeaseTime【看门狗时间】 / 3，10s

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110213448.png)

#### 2.3 最佳实战

> 尽量使用
> ```java
> lock.lock(30, TimeUnit.SECONDS); 
> ```
> 省掉了整个续期操作，手动解锁。
>
> 超时时间设置30秒。如果业务操作30s还没返回---说明业务已经完蛋了

### 3、读写锁

>保证一定能读到最新数据，修改期间，写锁是一个排他锁（互斥锁、独享锁）。读锁是一个共享锁
>写锁没释放读就必须等待
>写 + 读：等待写锁释放
>写 + 写：阻塞方式
>读 + 写：写锁需要等待
>读 + 读：相当于无锁，并发读，只会在redis中记录好所有当前读锁。它们都会同时加锁成功。
>
>！！！只要有写的地方，都必须等待！！！

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201111105312.png)

### 4、信号量

>应用：停车位、限流

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201111110417.png)

### 5、闭锁

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201111111051.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201111111132.png)

## 二、Spring Cache

### 1、简介

- Spring从3.1开始定义了 org.springframework.cache.Cache 和 org.springframework.cache.CacheManager 接囗来统一不同的缓存技术；并支持使用JCache（JSR107）注解简化我们开发；
- Cache接囗为缓存的组件规范定义，包含缓存的各种操作集合；
  Cache接囗下 Spring 提供了各种 xxxCach的实现；如 RedisCache，EhCacheCache，ConcurrentMapCache等；
- 每次调用需要缓存功能的方法时，Spring会检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。
- 使用 Spring 缓存抽象时我们需要关注以下两点；
   1、确定方法需要被缓存以及他们的缓存策略
   2、从缓存中读取之前缓存存储的数据

### 2、基础概念

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201111114137.png)

### 3、SpringBoot整合Cache

#### 3.1 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<!-- 使用redis作为缓存的话也需要引入redis依赖 -->
```

#### 3.2 配置

（1）自动配置

> CacheAutoConfiguration会导入 RedisCacheConfiguration

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201111115844.png)

> RedisCacheConfiguration 自动配置好了缓存管理器 RedisCacheManager

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201111115820.png)

（2）手动配置

```yml
spring:
  cache:
    type: redis
#    cache-names: name1,name2,name3
```

#### 3.3 开启缓存功能

```java
@EnableCaching
@SpringBootApplication
public class Application {
	...
```

### 4、缓存注解

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201111145122.png)

#### 4.1 @Cacheable

> 读模式下设置缓存

```java
// 简单使用
@Cacheable({"name1","name2"})
public Object getData (){
    // 获取业务数据
    return data;
}
```

> 1、每一个需要缓存的数据我们都需要指定放到哪个名字的缓存。【缓存分区（按业务类型分）】
> 2、@Cacheable代表当前方法的结果需要缓存
> 	如果缓存中有，则不需要调用方法
> 	如果缓存中没有，会调用方法查询结果并放入缓存
> 3、默认行为
> 	1）、如果缓存中有，方法不调用
> 	2）、key默认自动生成   【缓存名字】：：SimpleKey[]（自主生成的key值）
> 	3）、缓存的value值，默认使用jdk序列化机制，将序列化的数据存到redis
> 	4）、默认ttl时间 -1（即永不过期）

#### 4.2 自定义缓存配置

> 1、指定缓存使用的key
>
> key属性支持SpEL表达式，普通字符串记得加单引号

```java
@Cacheable(value = {"name1"}, key = "'level1Categorys'")
```
![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201111152010.png)

> 2、指定缓存数据存活时间

```yml
spring:
  cache:
    redis:
      time-to-live: 1000  ## 毫秒为单位
      # 开启缓存空值，防止缓存穿透
      cache-null-values: true
      # 开启缓存前缀，不设置的情况下默认使用分区名为前缀
      use-key-prefix: true
```

> 3、数据保存为json格式

```java
import org.springframework.boot.autoconfigure.cache.CacheProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

// 2、@ConfigurationProperties(prefix = "spring.cache") -> CacheProperties
// 原本配置文件绑定的配置类是上面这样的。没有放在容器中
// 开启属性绑定功能，绑定CacheProperties属性类
// 这样就能把绑定CacheProperties属性类放进spring容器中
@EnableConfigurationProperties(CacheProperties.class)
@Configuration
@EnableCaching
public class MyCacheConfig {

    @Bean // 3、可以直接设置参数为spring容器中的bean，从而获取CacheProperties.Redis
    RedisCacheConfiguration redisCacheConfiguration(CacheProperties cacheProperties) {
        // 1、使用自定义配置的话会无法获取配置文件中的属性
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
        config.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()));
        config.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
        // 4、使配置文件中的属性生效
        CacheProperties.Redis redisProperties = cacheProperties.getRedis();
        if (redisProperties.getTimeToLive() != null) {
            config = config.entryTtl(redisProperties.getTimeToLive());
        }
        if (redisProperties.getKeyPrefix() != null) {
            config = config.prefixKeysWith(redisProperties.getKeyPrefix());
        }
        if (!redisProperties.isCacheNullValues()) {
            config = config.disableCachingNullValues();
        }
        if (!redisProperties.isUseKeyPrefix()) {
            config = config.disableKeyPrefix();
        }
        return config;
    }
}
```

#### 4.3 @CacheEvict

> 写模式下的-->缓存失效模式

```java
// 更新操作后删除缓存
@CacheEvict(value = {"name1"}, key = "'level1Categorys'")
@Transaction
public void update(){
    // do something...
}

// 删除某个分区下的所有数据
// 约定：存储同一类型的数据，都可以指定成同一个分区，以后好删除
@CacheEvict(value = {"name1"}, allEntries = true)
@Transaction
public void update(){
    // do something...
}
```

#### 4.4 @Caching

> 同时操作多个缓存

```java
@Caching(evict = {
        @CacheEvict(value = {"name1"}, key = "'Categorys-01'"),
        @CacheEvict(value = {"name1"}, key = "'Categorys-02'")
})
@Transaction
public void update(){
    // do something...
}
```

#### 4.5 @CachePut

> 方法完成后将数据写入缓存
>
> 适用于-缓存双写模式

### 5、Spring Cache原理与不足

#### 5.1 原理

> CacheManager(RedisCacheManager)->Cache(RedisCache)->Cache负责缓存的读写

#### 5.2 读模式

- 缓存穿透：大量查询一个null的数据。解决：缓存空数据 cache-null-values: true
- 缓存击穿：高并发同时查询过期数据。解决：cache默认读不加锁，可以设置读锁 @Cacheable(value = "",  sync = true)
- 缓存雪崩：大量key同时过期。解决：加随机时间；其实只要加上过期时间就可以满足要求 time-to-live: 1000

#### 5.3 写模式

- 读写加锁。（读多写少）
- 引入Canal，感知到Mysql的更新去更新缓存
- 读多写多时，直接去数据库查询就行

#### 5.4 总结

> 常规数据（读多写少，即时性、一致性要求不高的数据）：完全可以使用Spring-Cache；写模式（只要缓存的数据有过期时间就足够了）
>
> 特殊数据：特殊设计