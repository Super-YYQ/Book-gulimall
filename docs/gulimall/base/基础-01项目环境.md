## 一、 项目架构

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/654asdf165.png)

## 二、创建虚拟机环境

### 2.1 创建虚拟机

#### （1）下载VirtualBox

开源虚拟机软件

> https://www.virtualbox.org/wiki/Downloads

#### （2）下载Vagrant

不翻墙下不了..

> https://www.vagrantup.com/downloads

Vagrant是一个基于Ruby的工具，用于创建和部署虚拟化开发环境。它使用Oracle的开源VirtualBox虚拟化系统，使用Chef创建自动化虚拟环境。

BBC Vagrant 是基于VirtualBox创建的虚拟机，并通过Vagrant进行打包而得到的VM环境。在虚拟机中部署好开发环境并建立虚拟机和实体机的文件共享，在开发时，可以通过实体机进行文件修改，并经过虚拟机中的环境执行，从而实现不同操作系统的工作环境的轻松部署。

- 下载box文件。

- 警告！！！！vagrant库文件夹不能存在中文！！！

#### （3）创建centos7

- 执行vagrant init 。（初始化当前文件夹为box，并创建一个配置文件Vagrantfile）

  ```shell
  vagrant init centos/7
  
  // Vagrantfile
  Vagrant.configure("2") do |config|
    config.vm.box = "centos/7"
  end
  ```

- 执行vagrant up。（会根据Vagrantfile中配置的名字去仓库中寻找对应的镜像） ---- 也可以自己下载镜像然后添加

  ```shell
  vagrant up
  ```

- vagrant ssh 开启SSH，并登陆到centos7

  ```shell
  vagrant ssh
  ```

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201109145455.png)

vagrant常用命令

| vagrant box list                     | 查看目前已有的box     |
| ------------------------------------ | --------------------- |
| vagrant box add [BOX NAME] [BOX URL] | 新增加一个box         |
| vagrant box remove [BOX NAME]        | 删除指定box           |
| vagrant init [BOX NAME]              | 初始化配置vagrantfile |
| vagrant up                           | 启动虚拟机            |
| vagrant ssh                          | ssh登录虚拟机         |
| vagrant suspend                      | 挂起虚拟机            |
| vagrant reload                       | 重启虚拟机            |
| vagrant halt                         | 关闭虚拟机            |
| vagrant status                       | 查看虚拟机状态        |
| vagrant destroy                      | 删除虚拟机            |

- 所有操作需要在workspace目录下执行

- 浏览页面是127.0.0.1:8000。对应workspace/ecstore/

- mysql网页配置端口是3306，如果需要从母机连接，需要访问33060端口。

- 如果需要连接ssh，为：vagrant ssh 即可。

- windows下，因为virtualbox本身一个bug，所以不能使用4.3.18版本，4.3.12版本可以用。

- win8可能需要改bios（开启intel的虚拟技术intel Virtualization Technology）

- box中的系统是64位的，建议在64位操作系统中使用

#### （4）配置虚拟机ip


```shell
[vagrant@localhost .ssh]$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85616sec preferred_lft 85616sec
    inet6 fe80::5054:ff:fe4d:77d3/64 scope link
       valid_lft forever preferred_lft forever
```
```
~ ipconfig
// Windows IP 配置
以太网适配器 VirtualBox Host-Only Network:
   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::d9e8:f834:737c:defc%21
   IPv4 地址 . . . . . . . . . . . . : 192.168.56.1  // 注意这里
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . :

```

配置网络信息，打开"Vagrantfile"文件：

```
config.vm.network "private_network", ip: "192.168.56.10"
```

修改完成后，重启启动vagrant

```shell
vagrant reload
```

检查宿主机和virtualBox之间的通信是否正常

```
[vagrant@localhost ~]$ ping 10.18.25.194 // 本机ip
PING 10.18.25.194 (10.18.25.194) 56(84) bytes of data.
64 bytes from 10.18.25.194: icmp_seq=1 ttl=127 time=0.385 ms
64 bytes from 10.18.25.194: icmp_seq=2 ttl=127 time=0.402 ms
64 bytes from 10.18.25.194: icmp_seq=3 ttl=127 time=0.346 ms
64 bytes from 10.18.25.194: icmp_seq=4 ttl=127 time=0.391 ms

[vagrant@localhost ~]$ ping baidu.com
PING baidu.com (39.156.69.79) 56(84) bytes of data.
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=1 ttl=45 time=22.5 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=2 ttl=45 time=22.0 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=3 ttl=45 time=22.0 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=4 ttl=45 time=22.0 ms
```

开启远程登陆，修改“/etc/ssh/sshd_config”

```shell
PermitRootLogin yes 
PasswordAuthentication yes
```

然后重启SSHD

```shell
systemctl restart sshd
```

### 2.2 安装docker

Docker安装文档  https://docs.docker.com/engine/install/centos/

1、卸载系统之前的 docker

```shell
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

2、设置redis存储库

```shell
$ sudo yum install -y yum-utils

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
3、安装docker Engine和容器

```shell
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

4、启动docker

```shell
$ sudo systemctl start docker
```

5、开启自启动

```shell
$ sudo systemctl enable docker
```

6、docker仓库阿里云加速

登录阿里云-控制台-容器镜像服务

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201106171659.png)

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://chrkdir0.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



### 2.3 docker中安装mysql

下载mysql

```shell
[root@hadoop-104 module]# docker pull mysql:5.7
5.7: Pulling from library/mysql
123275d6e508: Already exists 
27cddf5c7140: Pull complete 
c17d442e14c9: Pull complete 
2eb72ffed068: Pull complete 
d4aa125eb616: Pull complete 
52560afb169c: Pull complete 
68190f37a1d2: Pull complete 
3fd1dc6e2990: Pull complete 
85a79b83df29: Pull complete 
35e0b437fe88: Pull complete 
992f6a10268c: Pull complete 
Digest: sha256:82b72085b2fcff073a6616b84c7c3bcbb36e2d13af838cec11a9ed1d0b183f5e
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```

查看镜像

```
[root@hadoop-104 module]# docker images
REPOSITORY  TAG    IMAGE ID     CREATED       SIZE
mysql       5.7    f5829c0eee9e 2 hours ago   455MB
[root@hadoop-104 module]# 
```

启动mysql，并映射相关文件

```shell
sudo docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

修改配置

```properties
[root@hadoop-104 conf]# pwd
/mydata/mysql/conf
[root@hadoop-104 conf]# cat my.cnf
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
[root@hadoop-104 conf]# docker restart mysql
mysql
[root@hadoop-104 conf]# 
```

进入容器查看配置：

```shell
[root@hadoop-104 conf]# docker exec -it mysql /bin/bash
root@b3a74e031bd7:/# whereis mysql
mysql: /usr/bin/mysql /usr/lib/mysql /etc/mysql /usr/share/mysql

root@b3a74e031bd7:/# ls /etc/mysql 
my.cnf
root@b3a74e031bd7:/# cat /etc/mysql/my.cnf 
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
root@b3a74e031bd7:/# 
```

设置启动docker时，即运行mysql

```
[root@hadoop-104 ~]# docker update mysql --restart=always
mysql
[root@hadoop-104 ~]# 
```

### 2.4 docker中安装redis

下载redis

```shell
[root@hadoop-104 ~]# docker pull redis
Using default tag: latest
latest: Pulling from library/redis
123275d6e508: Already exists 
f2edbd6a658e: Pull complete 
66960bede47c: Pull complete 
79dc0b596c90: Pull complete 
de36df38e0b6: Pull complete 
602cd484ff92: Pull complete 
Digest: sha256:1d0b903e3770c2c3c79961b73a53e963f4fd4b2674c2c4911472e8a054cb5728
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```

启动redis

```shell
[root@hadoop-104 ~]# mkdir -p /mydata/redis/conf
[root@hadoop-104 ~]# touch /mydata/redis/conf/redis.conf
[root@hadoop-104 ~]# echo "appendonly yes"  >> /mydata/redis/conf/redis.conf
[root@hadoop-104 ~]# docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data \
> -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
> -d redis redis-server /etc/redis/redis.conf
ce7ae709711986e3f90c9278b284fe6f51f1c1102ba05f3692f0e934ceca1565
[root@hadoop-104 ~]#
```

测试redis

```shell
[root@hadoop-104 ~]# docker exec -it redis redis-cli
127.0.0.1:6379> set key1 v1
OK
127.0.0.1:6379> get key1
"v1"
127.0.0.1:6379> 
```

设置redis容器在docker启动的时候启动

```shell
[root@hadoop-104 ~]# docker update redis --restart=always
redis
[root@hadoop-104 ~]# 
```

## 三、开发环境

### 3.1 IDEA VsCode

vscode插件列表

![](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/20201106172209.png)

### 3.2 Git bash

配置密钥

创建仓库Gitee或GitHub

### 3.3 初始化数据库

