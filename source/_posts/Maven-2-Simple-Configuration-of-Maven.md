---
title: 'Maven (2):配置Maven与eclipse'
date: 2017-08-06 11:10:56
toc: true
tags:
 - Maven
 - 学习笔记
---

## 更改本地仓库位置

  .m2文件夹在home目录下，这显然是不合适的，实际上我们可以更改Maven的配置文件setting.xml来更改本地仓库位置,setting.xml的位置在maven安装目录下的conf文件夹中。打开setting文档，然后Ctrl+f搜索到这一段，在后面添加本地仓库路径，我们就把本地的仓库指向了这个新路径，这里是D:\Java\repository。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>D:\Java\repository</localRepository>
```

  当然如果你肯定不想重新下载本身部署在原来仓库的依赖库，也不想把本地的项目从用mvn stall部署到新仓库，你可以把以前仓库的所有文件直接复制过来就可以了。

<!--more-->

### 更改远程网络仓库位置

有时候你会觉得maven的下载速度实在是太慢了，像换到国内镜像，那应该怎么办呢？实际上在Maven安装目录下的lib中，有一个maven-model-builder-（版本号）的jar包，以解压软件打开该jar包，找到maven-model-builder-3.5.0.jar\org\apache\maven\model\pom-4.0.0.xml文件,其中有一段如下：

```xml
  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>

```

把url改掉就相当于改变了，网络中心仓库地址。

## 以mvn建立项目

在上一篇文章中，也许你看到了cmd如何建一个Maven项目，我们必须自己编辑pom.xml文件，新建文件夹等等，实际上它们的套路都是一样的，对于一样的的或许相似度高的东西，在计算机行业可能定是要被自动化代替掉的，毫无疑问，maven也给我们提供了这种命令。

```powershell
//在你要建立项目的目录下执行下面的命令
PS C:\Users\Saulxk\Desktop\maven> mvn archetype:generate

//配置项目
Define value for property 'groupId': com.saul.maven
Define value for property 'artifactId': HelloWorld3
Define value for property 'version' 1.0-SNAPSHOT: : 0.0.1
Define value for property 'package' com.saul.maven: :
Confirm properties configuration:
groupId: com.saul.maven
artifactId: HelloWorld3
version: 0.0.1
package: com.saul.maven

//查看项目
PS C:\Users\Saulxk\Desktop\maven> dir


    目录: C:\Users\Saulxk\Desktop\maven


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         2017/8/5     18:21                HelloWorld
d-----         2017/8/5     18:52                HelloWorld02
d-----         2017/8/6     14:26                HelloWorld3


PS C:\Users\Saulxk\Desktop\maven> cd .\HelloWorld3\
PS C:\Users\Saulxk\Desktop\maven\HelloWorld3> tree
文件夹 PATH 列表
卷序列号为 0000006A 02B2:A470
C:.
└─src
    ├─main
    │  └─java
    │      └─com
    │          └─saul
    │              └─maven
    └─test
        └─java
            └─com
                └─saul
                    └─maven
```

实际上，完全可以这样使用,就不需要一个一个输入Gav信息

`archetype:generate -DgroupId-com.saul.maven=DartifactId-Hello03-Dversion-0.0.1`

## eclipse中使用maven

  现在我们大致了解了maven的配置，但是实际上我们大多数在ide上开发java项目二不是控制台，如果你用的是idea可能你会偏向于使用Gradle,所以现在我们以eclipse为例来介绍如何在eclipse中创建一个maven项目。

  首先我使用的eclipse版本是neno，这里里面自带了maven插件，如果你用的是比较老的版本，可能你需要去下载m2eclipse插件，或者你是时候考虑一下更新你的eclipse版本。

**1.我们显得配置一下自己的maven，虽然eclipse里带了一个版本**

**Window -> Preference -> Maven -> Installations -> Add..**  

![QQ图片20170806175358](\img\QQ图片20170806175358.png)

**2.配置user setting**

**Window -> Preference -> Maven -> User setting**

![QQ图片20170806175549](\img\QQ图片20170806175549.png)

**3.创建maven项目**

**在新建项目的Other选项中可以看到**

![QQ图片20170806175807](\img\QQ图片20170806175807.png)



**第一步直接next，第二步选择构建一个quickstart项目**



![QQ图片20170806180050](\img\QQ图片20170806180050.png)

**4.配置maven项目完成构建**

![QQ图片20170806180309](\img\QQ图片20170806180309.png)



在完成这一步之后，我们就在eclipse中构建了一个maven项目，现在你可能已经可以猜测到，动态web项目也好还是普通的java application也好，都可以用maven项目来代替。

![QQ图片20170806180550](\img\QQ图片20170806180550.png)

可以看到目录和我们手动以mvn archetype:generate构建的项目目录如出一辙，以后我们可以不断地复用和壮大我们的本地仓库，利用maven我们可以自动构建出依赖，而不需要再去手动寻找jar包，或者解决某些麻烦的冲突等等。