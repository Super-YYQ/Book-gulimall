![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109161920.png)

## 一、模板引擎thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

配置

```yml
spring:
  thymeleaf:
    ## 是否开启缓存，关闭后有修改可以实时看到
    cache: false
```

index页面配置

```javascript
<!DOCTYPE html> // H5标头
<html lang="en" xmlns:th="http://www.thymeleaf.org"> // 名称空间
```

>  // springBoot默认视图解析器路径   classpath:/templates/

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109162910.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109163004.png)

> 语法详情见官网  https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html

## 二、devtools工具

可以在不重启服务器的情况下实时更新页面

> 需配合thymeleaf且关闭缓存

引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

> 修改完页面重新编译下页面就行 快捷键：ctrl + shift + f9

## 三、Nginx域名访问环境

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109165407.png)

> 代理的正向反向是相对于我们上网的电脑来说的
>
> 帮助我们上网的是正向代理，隐藏我们电脑的信息

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109165505.png)

> 访问网址-->本地解析域名-->查询不到就去DNS解析域名-->然后访问真实的网址
>
> 在本地hosts中设置某个域名对应ip为虚拟机地址

```
# gulimall windows中hosts配置
192.168.56.10 gulimall.com
```

```shell
docker update nginx --restart=always // 设置docker启动自启nginx
docker start nginx

// nginx配置文件路径
/nginx/conf/nginx.conf
```
![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109171003.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109171122.png)

```shell
# include /etc/nginx/conf.d/*/conf;表示包含所有该路径下的配置
/nginx/conf.d/default.conf
# 其中包含了一个server块配置
```

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109171330.png)

可以复制一份文件修改为自己项目的配置

```shell
# 复制一份default.conf并改名为gulimall.conf
$ cp default.conf gulimall.conf
```

> 将光标移动到需要删除的行
>
> 按一下ESC键，确保退出编辑模式
>
> 按两次键盘上面的 d键，就可以删除了。
>
> :set number // 显示行号

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109171907.png)

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109170553.png)

## 四、Nginx负载均衡交给网关

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109172152.png)

修改http块

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109172416.png)

修改代理转发的地址（上面自己起的名字）

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109172446.png)

## 五、网关host路由断言

域名访问

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109174047.png)

```yml
## 示例
spring:
  cloud:
    gateway:
      routes:
      - id: fulimall_host_route
        uri: lb:gulimall-product
        predicates:
        - Host=**.gulimall.com,**.anotherhost.org
```

> Nginx转发请求会丢掉很多东西
>
> 比如：Host地址！！！！！！！！

解决方法

修改nginx配置文件，将丢失的信息加上

```shell
proxy_set_header Host $host;
```

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109173441.png)