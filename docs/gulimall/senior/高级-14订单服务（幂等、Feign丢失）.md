## 一、订单中心

### 1.1 订单构成

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201230101540.png)

### 1.2 订单状态

代付款 --> 已付款/代发货 --> 待收货/已发货 --> 已完成 --> 已取消 --> 售后中

### 1.3 订单流程

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201230101942.png)



## 二、幂等性处理

> 接口幂等性：保证用户对统一操作发起的一次请求或多次请求的结果时一致的。

**应用场景**

- 用户多次点击按钮
- 用户页面回退后再次提交
- 微服务相互调用，由于网络问题导致请求失败，触发`feign`重试机制
- 其他业务情况

**``幂等性解决方案``**

### 2.1 Token机制

- `Redis Lua` 脚本

**使用**

1、服务端提供了发送 token 的接口。我们在分析业务的时候，哪些业务是存在幂等问题的
就必须在执行业务前，先去获取 token，服务器会把 token保存到 reds 中。
2、然后调用业务接口请求时，把 token 携带过去，一般放在请求头部。
3、服务器判断 token 是否存在 redis 中，存在表示第一次请求，然后删除 token，继续执行业务。
4、如果判断 token 不存在 redis 中，就表示是重复操作，直接返回重复标记给 client，这样就保证了业务代码不被重复执行。

> 订单提交验证令牌时需要使用 lua 脚本，对比令牌和删除令牌需哟啊保持原子性。

**危险性**

先删除 token还是后删除 token ？？
（1）先删除--可能导致，业务确实没有执行，重试还带上之前 token，由于防重设计导致，
请求还是不能执行。
（2）后删除--可能导致，业务处理成功，但是服务闪断，出现超时，没有删除 token，别
人继续重试，导致业务被执行两遍。
（3）我们最好设计为先删除 token，如果业务调用失败，就重新获取 token再次请求。

**注意**

Token获取、比较和刪除必须是原子性
（1） redis.get(token)、 token.equals、 redis.del(token) 如果这几个操作不是原子，可能导
致，高并发下，都get到同样的数据,，判断都成功,继续业务并发执行。
（2）可以在reds使用lua脚本完成这个操作。



### 2.2 各种锁机制

#### 2.2.1 数据库悲观锁、乐观锁

> 悲观锁

``select * from xxx where id = 1 for update``

悲观锁使用时一般伴随事务一起使用，数据锁定时间可能会很长，需要根据实际情况选用。
另外要注意的是，ld字段一定是主键或者唯一索引，不然可能造成锁表的结果，处理起来会非常麻烦。

> 乐观锁

这种方法适合在更新的场景中
``update t_goods set count = count-1, version = version + 1 where good_id = 2 and version = 1``
根据 version版本，也就是在操作库存前先获取当前商品的 version 版本号，然后操作的时候
带上此 version 号。我们梳理下，我们第一次操作库存时，得到 version为1，调用库存服务
version 变成了2；但返回给订单服务出现了问题，订单服务又一次发起调用库存服务，当订
单服务传如的 version 还是1，再执行上面的 sql 语句时，就不会执行；因为 version 已经变
为2了，where条件就不成立。这样就保证了不管调用几次，只会真正的处理一次。

**乐观锁主要使用于处理读多写少的问题**

#### 2.2.2 业务层分布式锁

**场景**

如果多个机器可能在同一时间同时处理相同的数据，比如多台机器定时任务都拿到了相同数据处理，我们就可以加分布式锁，锁定此数据，处理完成后释放锁。获取到锁的必须先判断这个数据是否被处理过。

### 2.3 各种唯一性约束

#### 2.3.1 数据库唯一性约束

插入数据，应该按照唯一索引进行插入，比如订单号，相同的订单就不可能有两条记录插入。
我们在数据库层面防止重复。
这个机制是利用了``数据库的主键唯一约束``的特性，解决了在 insert 场景时幂等问题。但主键的要求不是自增的主键，这样就需要业务生成全局唯一的主键。
如果是``分库分表``场景下，路由规则要保证相同请求下，落地在同一个数据库和同一表中，要不然数据库主键约束就不起效果了，因为是不同的数据库和表主键不相关。

#### 2.3.2 `redis set ` 防重

很多数据需要处理，只能被处理一次，比如我们可以计算数据的MD5将其放入 redis 的 set，每次处理数据，先看这个MD5是否已经存在，存在就不处理。

### 2.4 防重表

使用订单号 orderNo 做为去重表的唯一索引，把唯一索引插入去重表，再进行业务操作，且他们在同一个事务中。这个保证了重复请求时，因为去重表有唯一约束，导致请求失败，避免了冪等问题。这里要注意的是，``去重表和业务表应该在同一库``中，这样就保证了在同一个事务，即使业务操作失败了，也会把去重表的数据回滚。这个很好的保证了数据一致性。
``之前说的 redis 防重也算``

### 2.5 全局请求唯一id

调用接口时，生成一个唯一id，redis 将数据保存到集合中（去重），存在即处理过


## 三、附录

### 3.1 Feign丢失请求头问题

#### 3.1.1 单线程远程调用

> **Feign 远程调用时默认构造的 RequestTemplate，其中没有请求头数据**

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201231134848.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201231140454.png)



> **解决：添加对应拦截器，将请求头信息放进去**

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201231140300.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201231140013.png)

```java
import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
 
import javax.servlet.http.HttpServletRequest;
 
@Configuration
public class GuliFeignConfig {
 
    @Bean("requestInterceptor")
    public RequestInterceptor requestInterceptor() {
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate template) {
                // 使用RequestContextHolder（原理：使用ThreadLocal）拿到刚进来的请求数据
                ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                if (requestAttributes != null) {
                    // 老请求
                    HttpServletRequest request = requestAttributes.getRequest();
                    if (request != null) {
                        // 同步请求头的数据，cookie
                        String cookie = request.getHeader("Cookie");
                        // 给新请求同步了老请求的cookie
                        template.header("Cookie", cookie);
                    }
                }
            }
        };
    }
}
```

#### 3.1.2 多线程异步调用

> **多线程情况下，子线程无法获取主线程 ThreadLocal 中放的 request 数据**

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201231141653.png)

```java
// ############ 由于开启了新的线程,不能获取当前线程的session,需要设置到子线程中
RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();

// 使用异步编排提高效率
CompletableFuture<Void> getAddress = CompletableFuture.runAsync(() -> {
    // ############ 设置子线程的request
    RequestContextHolder.setRequestAttributes(requestAttributes);
    // 收货地址
    List<MemberAddressVo> address = memberFeignService.getAddress(memberResponseVo.getId());
    orderConfirmVo.setAddressVos(address);
}, executor);
CompletableFuture<Void> currentUserCartItem = CompletableFuture.runAsync(() -> {
    // ############ 设置子线程的request
    RequestContextHolder.setRequestAttributes(requestAttributes);
    // 获取当前用户的购物车商品项
    List<OrderItemVo> orderItemVos = cartFeignService.currentUserCartItem();
    orderConfirmVo.setItems(orderItemVos);
}, executor);

// 阻塞等待所有异步任务完成 -->> get()
CompletableFuture.allOf(getAddress,currentUserCartItem).get();
```

