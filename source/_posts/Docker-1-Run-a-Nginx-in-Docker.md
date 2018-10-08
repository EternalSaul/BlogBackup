---
title: 'Docker (1):Run a Nginx in Docker'
date: 2017-09-13 12:51:35
toc: true
tags:
 - Docker
 - 学习笔记
---

  Docker是在下非常常见的一个应用容器引擎，和MongoDb一样，它也诞生Paas市场混战时的,一家叫dotCloud的Paas提供商。

   它的流行主要是能解决一些业界的痛点：

​    1.解决了运行环境不一致带来的问题

​    2.它的隔离性解决了资源分配过度的问题，它可以限定一个用户最多用多少资源

​    3.因为上两点，我们可以实现新的服务器更快的部署

centos上，直接使用以下命令，就可以自动安装docker

`yum install docker`

这个安装方式的缺点是，版本号可能不是最新的，你也可以采取curl方式来安装它的最新版本：

`curl -s https://get.docker.com|sh`

<!--more-->

# Docker如何工作？

  安装完成之后我们就直接运行它吧，非常简单：

`service docker start`

然后我们可以尝试一下运行我们的helloworld程序

```shell
[root@localhost ~]# docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/

```

hello-world程序设计的很好，它不止输出了一个欢迎字符，而且它告诉了你docker如何工作：

1.docker客户端链接到了docker daemon

2.docker deamon尝试去Docker hub仓库中查找一个叫做hello-world的镜像，拉取到本地

3.从找到的hello-world镜像中创建一个容器,并且执行它，输出了你看到的这段话

4.这段话被显示在了中断里

简单的四句话，体现了Docker的工作流程。

## 仓库，镜像，容器

这里提到了3个概念，即仓库,镜像，容器

### 容器

本质就是一个进程，是可读写的，可以看成一个虚拟机
层级查找，容器在最上层，如果要修改一个镜像文件，则该文件会移动到容器层，然后在容器层实现修改，然后实现层级查找

### 镜像（image）

它是一系列的文件，每一层都是是只读的，这种分层以联合文件系统来实现，每一层加载完成后，这些文件都看做是同一个目录
比如第一层 test1 有 a b两个文件
第二层test2 有c d两个文件
联合文件系统 把他们结合看做test  包括了 a b c d四个文件



### 仓库

仓库保存了一列的镜像，我们可以用Nexus创建本地私服仓库，就像Maven那样，也可以用几个著名的大仓库，比如下两个：

[hub.docker.com]()    docker官方仓库
https://c.163.com/hub#/m/home/   网易蜂巢镜像中心（要登录网易账号）

你可能要问为什么，hello world叫hello-world而不是别的什么，原因就是仓库里它就叫做hello-world，其他的很多镜像我们也可以直接去仓库里找。

## 新的镜像

### 比如先获取Nginx？

如果只能运行hello-world的话那么就太没意思了，但是我前面说过容器几乎可以看成一个虚拟机，所以不难推测出我们的linux上可以运行的服务器程序都可以在docker容器里运行。

比如我们可以通过这种方式来get一个新的镜像：

拉取镜像
docker pull [OPTIONS] NAME[:TAG]

查看本地镜像
docker images \[OPTIONS][REPOSITORY[:TAG]]

我们可以在仓库中搜索镜像的链接，然后把它拉取到本地比如对于Nginx:

![QQ图片20170913135622](\img\QQ图片20170913135622.png)



我们可以查看到它所仓库中对于该镜像的描述，使用下面的命令将其拉取到本地：

`docker pull hub.c.163.com/library/nginx:latest`

### 查看本地镜像列表

拉取之后我们用上面提到的docker images来查看我们的本地镜像：

```shell
[root@localhost ~]# docker images
REPOSITORY                    TAG                 IMAGE ID            CREATED                                                 SIZE
docker.io/hello-world         latest              05a3bd381fc2        Less than                                a second ago   1.84 kB
hub.c.163.com/library/nginx   latest              46102226f2fd        4 months a                               go             109.4 MB

```

### Nginx从镜像到容器

可以看到我们的本地有两个镜像,那么我们要如何来运行Nginx呢？一般来说Nginx是运行在后台的负载均衡服务器，而且可能需要配置很多配置文件，如何来配置Nginx呢？

首先我们需要介绍两个命令：

docker -d -p 主机端口:容器端口 镜像名/镜像id（可缩写）  //手动分配映射端口
docker -d -P 镜像名/镜像id （可缩写）    //这个命令自动给端口号映射到所有的需要映射的端口

```
 //首先我们需要在后台来运行Nginx，把它当做是一个容器，因为我们的镜像比较小
[root@localhost ~]# docker run -d -p 8080:80 46
e333f51f79e74c75f0b89b475588de6c7bdcce3e664d591c8ade4b5baeebd131
```

### 访问Niginx容器

现在我们运行了Nginx，但是你可能要测试一下，我们有两种测试方法，一种就是用浏览器了，但是Linux一般是没有图形界面的，我们可以用curl来获取Nginx的欢迎界面的html：

```shell
[root@localhost ~]# curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

###   宿主windows上访问Nginx

  我们可以看到Nginx的欢迎界面已经可以访问，有些人可能会很奇怪，他在windows上用浏览器访问linxus上的ip地址加端口号的方式为什么访问不到该界面，其实docker容器自己有一个ip地址，我刚刚说他，你几乎可以把它认为是一个在操作一个类似于虚拟机的东西（其实它不是虚拟机，而是基于联合文件系统的上层容器），那么我们在哪里可以访问到这个ip地址呢？

  实际上我们直接可以看路由表或者用ifconfig就知道了,这里我用路由表来显示一下：

```shell
[root@localhost ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    100    0        0 ens33
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.194.0   0.0.0.0         255.255.255.0   U     100    0        0 ens33
```

  从上面这个路由表你可以推测，如果你对192.168.194.1:8080进行curl的话当然是抓取不到HTML欢迎界面的，但是可以通过172.17.0.1:8080来抓取，很遗憾的是你的windows宿主机器中根本不知道172.17.0.1和192.168.194.1有什么关系。

  这种情况下我们只能手动配置路由表了，打开cmd(需要以管理员身份):

```
//我们首先当然要看看192.168.194.1的掩码，因为我们呢要把它设为网关，所以先看看ipv4的路由表

C:\WINDOWS\system32>route print -4
===========================================================================
接口列表
  7...d0 bf 9c 94 fb d4 ......Realtek PCIe GBE Family Controller
 14...f4 06 69 7d ea 1b ......Microsoft Wi-Fi Direct Virtual Adapter
  6...f6 06 69 7d ea 1a ......Microsoft Hosted Network Virtual Adapter
 21...00 50 56 c0 00 01 ......VMware Virtual Ethernet Adapter for VMnet1
 17...00 50 56 c0 00 08 ......VMware Virtual Ethernet Adapter for VMnet8
 12...f4 06 69 7d ea 1a ......Intel(R) Dual Band Wireless-AC 3160
  1...........................Software Loopback Interface 1
===========================================================================

IPv4 路由表
===========================================================================
活动路由:
网络目标        网络掩码          网关       接口   跃点数
          0.0.0.0          0.0.0.0       10.86.80.1     10.86.80.164     55
       10.86.80.0    255.255.240.0            在链路上      10.86.80.164    311
     10.86.80.164  255.255.255.255            在链路上      10.86.80.164    311
     10.86.95.255  255.255.255.255            在链路上      10.86.80.164    311
        127.0.0.0        255.0.0.0            在链路上         127.0.0.1    331
        127.0.0.1  255.255.255.255            在链路上         127.0.0.1    331
  127.255.255.255  255.255.255.255            在链路上         127.0.0.1    331
     192.168.32.0    255.255.255.0            在链路上      192.168.32.1    291
     192.168.32.1  255.255.255.255            在链路上      192.168.32.1    291
   192.168.32.255  255.255.255.255            在链路上      192.168.32.1    291
    192.168.194.0    255.255.255.0            在链路上     192.168.194.1    291
    192.168.194.1  255.255.255.255            在链路上     192.168.194.1    291
  192.168.194.255  255.255.255.255            在链路上     192.168.194.1    291
        224.0.0.0        240.0.0.0            在链路上         127.0.0.1    331
        224.0.0.0        240.0.0.0            在链路上      192.168.32.1    291
        224.0.0.0        240.0.0.0            在链路上     192.168.194.1    291
        224.0.0.0        240.0.0.0            在链路上      10.86.80.164    311
  255.255.255.255  255.255.255.255            在链路上         127.0.0.1    331
  255.255.255.255  255.255.255.255            在链路上      192.168.32.1    291
  255.255.255.255  255.255.255.255            在链路上     192.168.194.1    291
  255.255.255.255  255.255.255.255            在链路上      10.86.80.164    311
===========================================================================
永久路由:
  无
  
//可以看到我们的192.164.194.1/32 在路由表中，我们把它设置为网关
C:\WINDOWS\system32>route add 172.17.0.1 mask 255.255.255.255 192.168.194.1
 操作完成!
```

然后我们再访问172.17.0.1就可以看到Nginx界面了，说些仅仅因为，如果你将来把web程序配置到linxu虚拟机上，我们要用浏览器访问它，不可能用curl来做操作吧！

![QQ图片20170913141920](\img\QQ图片20170913141920.png)

### 如何配置容器？

如果一个容器只能访问，而不能配置，那它毫无作用，实际上我们可以进入容器来对它做配置：

在一个容器中运行一个命令

                    选项      容器名    命令
     docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
  -d, --detach         Detached mode: run command in the background
  --detach-keys        Override the key sequence for detaching a container
  --help               Print usage
  -i, --interactive    Keep STDIN open even if not attached
  --privileged         Give extended privileges to the command
容器名/容器id  -t, --tty            Allocate a pseudo-TTY
  -u, --user           Username or UID (format: <name|uid>[:<group|gid>])

我们常用这个命令，来以一个伪终端进入一个bash程序：

-i 保证输入有效
-t 分配一个伪终端
一般是 docker exec -it 容器名/容器id（可以缩写就像git log） bash 

```
//要进入容器我们先得知道容器名称或者容器的id,使用docker ps命令我们可以看到这些
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                                NAMES
e333f51f79e7        46                  "nginx -g 'daemon off"   34 minutes ago      Up 34 minutes       0.0.0.                                                                              0:8080->80/tcp   desperate_golick

//现在我们以容器id的最短唯一缩写（同git里的版本号缩写）那样进入我们的容器的bash

[root@localhost ~]# docker exec -it e3 bash

//可以看到我们以root进入了容器，它就像一个linux虚拟机一样，有着它的大部分功能
root@e333f51f79e7:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

//我们可以进入/etc/nginx/目录
root@e333f51f79e7:/# cd /etc/nginx/
root@e333f51f79e7:/etc/nginx# ls
conf.d  fastcgi_params  koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params  uwsgi_params  win-utf
```

我们进入了Docker的容器，但是你运行命令马上就会有惊人的发现，这怎么用啊！连vi都没！实际上一开始这个容器是裸的，你要自己装一些常用的软件，比如vim之类的，然后如果你是centos/rhel用户，你尝试yum update的时候，可能会提示你yum都没有，不要悲哀，实际上Docker是更倾向于支持Debian/Ubuntu这边的，所以你得用apt-get 去安装软件：

```shell
apt-get update

apt-get -y install vim

...
```





