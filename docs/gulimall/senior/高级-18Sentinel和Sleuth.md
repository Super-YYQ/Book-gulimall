## 一、SpringCloud Alibaba-Sentinel

[Sentinel官网](https://sentinelguard.io/zh-cn/docs/quick-start.html)

### 1.1 熔断降级限流

- 熔断

  A服务调用B服务的某个功能，由于网络不稳定问题，或者B服务卡机，导致功能时间超长。如果这样子的次数太多。我们就可以直接将B断路了(A不再请求B接口)，凡是调用B的直接返回降级数据，不必等待B的超长执行。这样B的故障问题，就不会级联影响到A

- 降级

  整个网站处于流量高峰期，服务器压力剧增，根据当前业务情况及流量，对一些服务和页面进行有策略的降级！停止服务，所有的调用直接返回降级数据]。以此缓解服务器资源的的压力，以保证核心业务的正常运行，同时也保持了客户和大部分客户的得到正确的相应

- 限流

  对打入服务的请求流量进行控制，使服务能够承担不超过自己能力的流量压力



> 熔断与降级的区别

相同点:
1、为了保证集群大部分服务的可用性和可靠性，防止崩溃，牺牲小我
2、用户最终都是体验到某个功能不可用
不同点:
1、熔断是被调用方故障，触发的系统主动规则
2、降级是基于全局考虑，（人工）停止一些正常服务，释放资源

### 1.2 Sentinel 与 Hystrix 对比

![image-20210218135409524](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210218135409524.png)

### 1.3 整合SpringCloud

[按照官方整合文档来](https://github.com/alibaba/spring-cloud-alibaba/wiki/Sentinel)

![image-20210218144716690](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210218144716690.png)

### 1.4 图表数据

![image-20210218144052689](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210218144052689.png)

### 1.5 自定义的响应数据

![image-20210218144437185](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210218144437185.png)



## 二、Sleuth + Zipkin 服务链路追踪

### 2.1 为什么用

微服务架构是一个分布式架构，它按业务划分服务单元，一个分布式系统往往有很多个服务单元。由于服务单元数量众多，业务的复杂性，如果出现了错误和异常，很难去定位。主要体现在，一个请求可能需要调用很多个服务，而内部服务的调用复杂性，决定了问题难以定位。所以微服务架构中，必须实现分布式链路追踪，去跟进一个请求到底有哪些服务参与，参与的顺序又是怎样的，从而达到每个请求的步骤清晰可见，出了问题，很快定位。

链路追踪组件有 Google的 Dapper，Twitter 的 Zipkin，以及阿里的 Eagleeye(鹰眼)等，它
们都是非常优秀的链路追踪开源组件

![image-20210219153449137](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210219153449137.png)



### 2.2 Sleuth 调用链

[官网文档](https://docs.spring.io/spring-cloud-sleuth/docs/2.2.7.RELEASE/reference/html/)

![image-20210219154150857](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210219154150857.png)

### 2.3 zipkin 可视化观察

[官网文档](https://zipkin.io/pages/architecture.html)

![image-20210220141102373](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210220141102373.png)



```sh
// 1、docker 安装 zipkin 服务器
docker run -d -p 9411:9411 openzipkin/zipkin
```

```xml
// 2、引入依赖，zipkin 依赖包括sleuth，可以省略 sleuth 的引用
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

```yaml
# 3、添加 zipkin 配置
spring:
	zipkin:
		# zipkin服务器地址
		base-url: http://localhost:9411/
		# 关闭服务发现，否则cloud会把zipkin的当成服务名称
		discoveryClientEnabled: false
        #使用默认 http 方式收集 span 需要配置此项
        sender:
        	type: web
    sleuth:
    	sampler:
    		# 抽样采集率为100%，默认0.1即10%
    		probability: 1
```

> http:localhost:9411/zipkin

![image-20210220142734190](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210220142734190.png)

### 2.4 zipkin 数据持久化

> 一般使用 Elasticsearch
>
> 启动 zipkin 时加上相应参数就行

![image-20210220143432215](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210220143432215.png)