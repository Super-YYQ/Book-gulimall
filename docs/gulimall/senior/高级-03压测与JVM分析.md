## 一、压力测试

> 压力测试考察当前软硬件环境下系统所能承受的最大负荷并帮助找岀系统瓶颈所在。压测都是为了系统在线上的处理能力和稳定性维持在一个标准范围内,做到心中有数。
> 使用压力测，我们有希望找到很多种用其他测试方法更难发现的错误。有两种错误类型是：内存泄漏，并发与同步。
> 有效的压力测试系统将应用以下这些关键条件重复，并发，量级，随机变化。

### 1、性能指标

- 响应时间( Response Time:RT）
  响应时间指用户从客户端发起一个请求开始，到客户端接收到从服务器端返回的响应结束，整个过程所耗费的时间。
- HPS( Hits Per Second)：每秒点击次数，单位是次/秒。
- TPS( Transaction per Second)：系统每秒处理交易数，单位是笔/秒。
- QPS( Query per Second)：系统每秒处理查询次数，单位是次秒。
  对于互联网业务中，如果某些业务有且仅有一个请求连接，那么TPs=Qps=HPS,
  般情况下用TPs来衡量整个业务流程，用QPs来衡量接囗查询次数，用HPs来表
  示对服务器单击请求。
- 无论TPS、αPS、HPS，此指标是衡量系统处理能力非常重要的指标，越大越好，根据经验，一般情况下：
      金融行业：1000TPS~50000TPS，不包括互联网化的活动
      保险行业：100TPS~100000TPS，不包括互联网化的活动
      制造行业：10TPS~5000TPS
      互联网电子商务：100000TPS~1000000TPS
      互联网中型网站：1000TPS~50000TPS
      互联网小型网站：500TPS~10000TPS
- 最大响应时间( Max Response Time)指用户发出请求或者指令到系统做出反应(响应)的最大时间。

- 最少响应时间( Mininum Response Time)指用户发出请求或者指令到系统做出反应(响应)的最少时间。
- 90%应时间(90% Response Time)是指所有用户的响应时间进行排序，第90%的响应时间。
- 从外部看，性能测试主要关注如下三个指标
      吞吐量：每秒钟系统能够处理的请求数、任务数。
      响应时间：服务处理一个请求或一个任务的耗时。
      错误率：一批请求中结果出错的请求所占比例。

## 二、JMter压力测试工具

### 1、下载运行

#### 1.1 下载

[官网下载地址](https://jmeter.apache.org/download_jmeter.cgi)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109193852.png)

#### 1.2 运行bin目录下bat文件

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109194044.png)

#### 1.3 简体中文设置

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109194205.png)

### 2、测试实例

#### 2.1 测试计划新增线程组

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109194317.png)

#### 2.2 配置线程数

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109194643.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109194803.png)

#### 2.3 设置请求（取样器）

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109194849.png)

#### 2.4 响应结果分析

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109195057.png)

### 3、 其他

#### 3.1 JMter Address Already in use 错误解决

windows本身提供的端囗访问机制的问题。
Windows提供给TCP/IP链接的端囗为1024-5000，并且要四分钟来循环回收他们。就导致我们在短时间内跑大量的请求时将端囗占满了。

> 解决方法：
>
> 1、cmd中，用 regedit命令打开注册表
> 2、在 HKEY LOCAL MACHINE\ SYSTEM\ CurrentControlSet\ Services \Tcpip\ Parameters 下，
> ​    （1）右击 parameters，添加一个新的 DWORD，名字为 MaxUserPort
> ​    （2）然后双击 MaxUserPort，输入数值数据为65534，基数选择十进制（如果是分布式运行的话，控制机器和负载机器都需要这样操作哦）
> 3、修改配置完毕之后记得重启机器才会生效

#### 3.2 优化

> 增加项目内存

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109195547.png)

- 影响性能考虑点包括：
  数据库、应用程序、中间件( (tomact、 Nginx)、网络和操作系统等方面
- 首先考虑自己的应用属于CPU密集型还是IO密集型

## 三、性能监控及JVM分析

### 1、 JVM内存模型

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109201013.png)

- 程序计数器 Program Counter Register
  记录的是正在执行的虚拟机字节码指令的地址，
  此内存区域是唯一一个在ANA虛拟机规范中没有规定任何 OutOfMemoryError的区域

- 虚拟机：VM Stack
  - 描述的是JAVA方法执行的内存模型，每个方法在执行的时候都会创建一个栈帧，于存储局部变量表，操作数栈，动态链接，方法接囗等信息
  - 局部变量表存储了编译期可知的各种基本数据类型、对象引用
  - 线程请求的栈深度不够会报 StackOverflowError异常
  - 栈动态扩展的容量不够会报 OutOfMemoryError异常
  - 虚拟机栈是线程隔离的，即每个线程都有自己独立的虚拟机栈
  
- 本地方法: Native Stack
  - 本地方法栈类似于虚拟机栈，只不过本地方法栈使用的是本地方法

- 堆：HEAP
  - 几乎所有的对象实例都在堆上分配内存

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109201657.png)

###  2、 堆
所有的对象实例以及数组都要在堆上分配。堆是垃圾收集器管理的主要区域，也被称为“GC堆”，也是我们优化最多考虑的地方
堆可以细分为:

- 新生代
  - Eden空间
  - From Survivor空间
  - To Survivor空间
- 老年代
- 永久代/元空间
    - Java8以前永久代，受JVM管理，java8以后元空间，直接使用物理内存。因此默认情况下，元空间的大小仅受本地内存限制。

垃圾回收

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109202041.png)

> 从Java8开始，HotSpot已经完全将永久代( Permanent Generation)移除，取而代之的是一个新的区域一元空间( Metaspace)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109202200.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109202527.png)

### 3、 jconsole与jvisualvm

jdk的两个小工具 jconsole、jvisualvm(升级版的 jconsole)；通过命令行启动，可监控本地和远程应用。远程应用需要配置

```shell
// jconsole命令行启动
C:\Users\***>jconsole
```

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109203154.png)

```shell
// jvisualvm命令行启动
C:\Users\***>jvisualvm
```

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110084106.png)

运行：正在运行的
休眠：sleep
等待：wait
驻留：线程池里面的空闲线程
监视：阻塞的线程，正在等待钡锁

#### 4.1 安装插件查看gc

> 工具-插件报错

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110084449.png)

> 解决方法（指定对的插件中心版本）

- 打开插件中心网址  http://visualvm.github.io/pluginscenters.html

- 找到对应JDK版本

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110084948.png)

- 替换插件中心网址

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110085044.png)

- 安装Visual GC（需要重启生效）

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110085303.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110085701.png)

## 四、优化

### 1、压测测试

> 在JMter中可以在线程组-高级中设置勾选页面渲染模拟页面下载图片等操作

#### 1.1 查看虚拟机镜像状态

> JMter持续发送请求，docker查看资源使用情况

```shell
$ docker stas
```

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110085924.png)

#### 1.2 指标汇总分析

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110091450.png)

### 2、优化考虑

- 中间件越多，性能损失越大，大所损失在网络交互中

- 老年代（新生代）GC很耗内存（加大内存分配）

  > -Xmx 最大 -Xms 最小 -Xmn 新生代内存

  ![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110093537.png)

- 业务

  - Db（Mysql优化-索引）
  - 模板渲染速度（thymeleaf缓存）
  - 静态资源（动静分离）

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110092027.png)

#### 2.1 静态资源配置Nginx

（1）将静态文件全部放到Nginx目录下

（2）将服务端index.html页面中所有加载静态资源的路径全部配置/static/前缀

（3）配置Nginx配置文件conf的location路径配置

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201110094446.png)
