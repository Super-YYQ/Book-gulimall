## 一、分布式session问题

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223095238.png)

> 两个方面的问题。
>
> 1、集群下多个服务节点之间session不同步问题（多个服务器之间session存储不一样）
>
> 2、分布式下多个服务之间session不共享问题（浏览器无法拿到另一个域名下的session）

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223095359.png)

> 解决方案① 集群下服务节点间session不同步问题
>
> 统一存储，SpringSession

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223100804.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223100924.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223101216.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223101427.png)

> 解决方案② 分布式下多个服务间session不共享问题
>
> 放大作用域（放在统一域名下）
>
> - 没有统一域名的就不行了，比如网易游戏和网易邮箱无法放在同一个域名下

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223102244.png)

## 二、SpringSession

[>>>官方文档<<<](https://docs.spring.io/spring-session/docs/current/reference/html5/index.html#samples)

> maven依赖-->指定session存储（redis）-->配置redis链接-->注解开启SpringSession功能

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223142651.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223154215.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223154402.png)



## 三、单点登录

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223164734.png)

### 3.1 简单Demo

[>>> 于雪里SSO <<<](https://gitee.com/xuxueli0323/xxl-sso?_from=gitee_search)

> 更改host文件，模拟多域名环境

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223170624.png)

> 打包启动服务

```shell
// 在pom文件所在最外层（三个小服务外面）路径下打包
// 清理-打包  跳过测试
$ mvn clean package -Dmaven.skip.test=true

// 在小服务target目录下jar文件处启动，指定端口
$ java -jar xxl-sso-server-1.1.1-SNAPSHOT.jar --server.port=8081

// 打包安装到仓库
$ mvn install
```

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201223171849.png)

### 3.2 单点登录流程

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/Untitled%20Diagram%20(1).png)



## 四、附录

```java
// 获取session和cookie的方法
public void test (HttpSession httpSession, @CookieValue(value = "sso_token",required = false) String sso_token){}

// 放在session中的只是一串令牌，在后端页面（thymeleaf、jsp）中可以直接调用session.value
// 但是在前后端分离项目中不能直接获取，只能通过接口向后端查询
```

