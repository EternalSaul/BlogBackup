---
title: 'Nexus (1):用Nexus构建Maven私服'
date: 2017-08-11 10:09:03
tags:
 - Nexus
 - Maven
 - 学习笔记
---

## 为什么需要配置Maven私服

  在我们前面介绍的Maven（1-4）中我们都没有配置本地的中央仓库，我们的Maven如果在个人电脑上的本地仓库里没有找到相关依赖，就会直接连接互联网去寻找相应的依赖，这如果是在家里做做练习没什么问题，实际上在公司中，数千个程序员同时通过公司的网络端口去下载同一个依赖包，比如一个spring的依赖大概1M，我们就会去重复请求数千份下载，对于一个学计算机的人来说，这肯定是不合适的。**对于远程仓库的频繁访问也可能会导致它的服务器误以为我们公司的IP对它进行网络攻击，从而限制我们访问**，并且远程服务器本身的访问就很慢，这种重复下载可能需要很多小时才能完成，程序员的时薪是不便宜的，他们的耐心也不是无限的。

![QQ图片20170811104212](\img\QQ图片20170811104212.png)

  <!--more-->

为了解决上述这样的情况我们在本地局域网中引入一台私有的服务器，用来做公司内部的中央仓库，如下图：

![QQ图片20170811104008](\img\QQ图片20170811104008.png)

  这样的话，我们就不会一直去请求非常慢的外网，而是直接去请求本地的一台私服仓库获取依赖，如果私服仓库中没有该依赖，它才会去请求远程中央仓库，把该依赖包下载下来，从而导致未来的本地仓库中没有的依赖可以直接到私服仓库中就可以获取到，而不用做重复的慢速互联网的访问。我们在本地仓库和远程仓库中加了一层私服，这层私服就类似于一个中介者，**只有它能够访问远程仓库，而本地仓库只能访问它。**

## 使用Nexus

  首先你需要下载一个Nexus并把它的bin目录配置到环境变量，这种约定俗称的东西我就不多说了，获取免费的oss版的Nexus，可以[点击这里](https://www.sonatype.com/download-oss-sonatype)。

   这里我就下了直接下了最新的3.5版本（从3.0开始它要求Java8 以上环境），目前如果你在windows上使用的话，它提供了exe安装包（无需配置），这里我下载的依然是zip，把解压到某个文件夹后配置环境变量，你需要以管理员打开cmd，运行一下

`nexus.exe /run`

当你看见类似于这种东西，说明你已经安装成功

![QQ图片20170811140634](\img\QQ图片20170811140634.png)

  现在可以进入localhost:8081，就可以看到界面。但是以nexus.exe /run运行nexus会阻塞cmd，这样的运行方式不是我们想要的，于是我们可以通过

`nexus.exe /install`      安装nexus到服务

`nexus.exe /start`          启动nexus(你也可以直在windows的服务中启动，我推荐顺便把nexus服务设置为手动启动)

这种通过服务来启动的方式，不会阻塞进程，我推荐这种方式。

这里我配置了一下%NEXUS_HOME%/etc中的nexus-default.properties,这里可以改变端口，我推荐不要使用8081这个端口，我把它改为了9081这个，其他的就没必要改了

```xml-dtd
## DO NOT EDIT - CUSTOMIZATIONS BELONG IN $data-dir/etc/nexus.properties
##
# Jetty section
application-port=9081
application-host=0.0.0.0
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
nexus-context-path=/

# Nexus section
nexus-edition=nexus-pro-edition
nexus-features=\
 nexus-pro-feature
```

  启动服务后你如果你立即访问9081端口，可能会出现404，我就出现了这种情况，搞得我很纳闷，实际上过几分钟访问就可以了，另外3.5的web配置界面的Js在edge浏览器下有点不兼容,比如弹出如下错误，你可以使用chrome。

![QQ图片20170811145714](\img\QQ图片20170811145714.png)

chorme下正常访问，默认管理员账户admin,密码是admin123：

![QQ图片20170811145935](\img\QQ图片20170811145935.png)

## Nexus+Maven

### Nexus仓库类型

首先对于Nexus中我们可以看见4个Maven2仓库，这正是我们首先要了解的：

![QQ图片20170811151421](\img\QQ图片20170811151421.png)

**1.central**

**proxy类型（代理远程的仓库），它存放的是从网络中央工厂中下载的依赖包**

**2.public**

**group类型（合并多个hosted和proxy仓库），它是一个仓库组，用来存放需要的仓库**

**3.releases/snapshots**

**hosted类型（本地仓库），它们通常是存放公司自己的构件，对应着reeases和snapshots两个不同的版本**

### 配置Maven使用Nexus代理

  我们在[Maven (4)](/2017/08/08/Maven-4-Aggregation-Inheritance/)中讨论过如何用一个母项目对项目模块进行综合的配置，然后通过继承的方式让各个模块的配置统一，现在我们在母项目中来配置Nexus代理从而实现对整个项目的配置。

```xml
  <repositories>
  	<repository>
  		<id>nexus03</id>
  		<name>saul central</name>
  		<url>http://localhost:9081/repository/maven-central/</url>
  	</repository>
  	<repository>
  	  	<id>nexus02</id>
  		<name>saul releases</name>
  		<url>http://localhost:9081/repository/maven-releases/</url>
  	</repository>
  	  	<repository>
  	  	<id>nexus01</id>
  		<name>saul snapshots</name>
  		<url>http://localhost:9081/repository/maven-snapshots/</url>
  	</repository>
  </repositories>
```

  如果我们需要在pom.xml中像上面这样增加三个仓库位置，但是这样显然是很繁琐的，在前面我们介绍过**group类型的仓库是用来合并多个hosted和proxy仓库**，所以其实我们上面的三个仓库可以直接用一个group类型的仓库来代替~。

![QQ图片20170811174853](\img\QQ图片20170811174853.png)

查看maven-public配置,我们可以发现它包含了我们需要的三个成员仓库，于是我们可以只在母项目的pom.xml添加：

```xml
  <repositories>
	<repository>
       <id>nexus</id>
       <name>SaulPublic</name>
      <!--public是组类型仓库，他已经包含了我们需要的三个仓库-->
       <url>http://localhost:9081/repository/maven-public/</url>
     </repository>
  </repositories>
```

现在如果我们加入一个gson的依赖，就可以清楚的看到我们的maven项目，如果在本地仓库没找到依赖包，就会从我们的私服的maven-central仓库中中去寻找该包，而如果我们的仓库中没有该包，那私服仓库就会去远程仓库里找。

![QQ图片20170811180054](\img\QQ图片20170811180054.png)

现在我们可以在已有的项目中设置我们的私服了，但是我们一般不这么做，因为对于每一个项目我们都会应用到我们的私服仓库，而不是对单单几个而已，此时，我们可以配置Maven的setting.xml来完成这一设置。

我们可以在setting.xml中的profiles元素中加入：

```xml
<profiles> 
  <profile>
    <id>nexusRepository</id> 
    <repositories>
      <repository>
        <id>nexus</id>
        <name>SaulPublic</name>
        <url>http://localhost:9081/repository/maven-public/</url>
        <releases>
          <enabled>true</enabled>
        </releases>
        <snapshots>
          <enabled>true</enabled>
        </snapshots>
      </repository>
    </repositories>
  </profile>   
</profiles>

<!--在settings中激活nexusRepository！这里不是profiles-->
<activeProfiles>
  <activeProfile>nexusRepository</activeProfile>
</activeProfiles> 
  
```

于是以后我们每次更新依赖是，都会到nexusRepository所配置的url:http://localhost:9081/repository/maven-public寻找依赖，但是这依然是有缺点的，记得我们在[第二篇Maven文章](http://saul.xin/2017/08/06/Maven-2-Simple-Configuration-of-Maven/#more)中提到过一个叫做maven-model-builder（版本号）的jar包 ，里面包含在我们的Maven所要去寻找的远程仓库的地址，实际上这个地址的存在会影响我们的私服仓库，因为如果我们的私服一旦关闭，事实上我们的Maven就会去这个远程仓库里查找依赖包！

##  配置镜像

于是我们得用新的方法，即镜像，我们在settings.xml中做如下配置：

```xml
<!--设置镜像-->
<mirrors>
     <mirror>
      <id>nexuslMirror</id>
<!--设置*标识所有仓库都使用该镜像-->
      <mirrorOf>*</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://localhost:9081/repository/maven-public/</url>
    </mirror> 
</mirrors>

<profiles> 
     <profile>
       <id>centralRepository</id> 
      <repositories>
        <repository>
          <id>central</id>
          <name>Central Repository</name>
          <!--镜像已经是所有仓库的镜像，url已经没有意义，它自己去会找镜像的url-->
          <url>http://*</url>
          <releases>
              <enabled>true</enabled>
          </releases>
          <snapshots>
              <enabled>true</enabled>
          </snapshots>
        </repository>
      </repositories>
    </profile>  
</profiles>

<!--在settings中激活centralRepository！这里不是profiles-->
<activeProfiles>
  <activeProfile>centralRepository</activeProfile>
</activeProfiles> 
```

 这样配置之后，我们的settings中的centralRepository就会覆盖maven-model-builder（版本号）的jar包 中的配置，每当Maven寻找依赖时，它会发现settings.xml中centralRepository被设置，从而去寻找镜像中指向的地址，如果改地址找不到或者无法访问，Maven就无法下载依赖包，从而报错~

















