---
title: 'Maven (1):Maven可以干什么？'
date: 2017-08-05 18:41:01
toc: true
tags:
 - Maven
 - 学习笔记
---

​    Maven是Apache出的一款非常好用的项目管理工具，如果你对这个名词比较模糊， 那我们先用maven来做一些事情，感受一下它给我们带来的好处，但首先我们还是要安装与配置它，这是必可少的。

## Maven的安装与配置

  首先下载[Maven 3.5.0](http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.0/binaries/apache-maven-3.5.0-bin.zip)的压缩包,解压后配置环境变量。就是把bin目录加入到path中，配置好了以后测试一下。即在控制台中输入mvn -v，查看maven的版本信息，如果显示出如下类似的信息说明已经配置成功了。

```powershell
C:\Users\Saulxk>mvn -v
Apache Maven 3.5.0 (ff8f5e7444045639af65f6095c62210b5713f426; 2017-04-04T03:39:06+08:00)
Maven home: D:\apache-maven-3.5.0\bin\..
Java version: 1.8.0_73, vendor: Oracle Corporation
Java home: C:\Program Files\Java\jdk1.8.0_73\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```

## 第一个maven项目

  构建这个项目主要是让大家知道maven是干什么的，现在我们不打开eclipse或者其他ide，我们直接用cmd构建一个目录，并且在记事本里编辑java文件，这样你大概知道了，如果一开始学java你是用命令行去编译文件，很重要的一步是你要引入各种各样的包，也许你会把它们加到环境变量中，以至javac能够通过，这样的方法当然是不好的，后来改用ide后，你也许是把jar直接放入项目中一个lib文件夹中然后添加依赖，或者直接引入包，现在我们尝试用maven来完成这些操作，我们将以这个小小的例子看到maven功能和威力~~~

<!--more-->

### 构建项目和pom.xml

  首先构建项目目录：

```powershell
C:\Users\Saulxk\Desktop\maven>tree HelloWorld
文件夹 PATH 列表
卷序列号为 00000197 02B2:A470
C:\USERS\SAULXK\DESKTOP\MAVEN\HELLOWORLD
└─src
    ├─main
    │  └─java
    │      └─com
    │          └─saul
    └─test
        
C:\Users\Saulxk\Desktop\maven>notepad HelloWorld\src\main\java\com\saul\Hello.java
```

首先我们需要在项目根目录下编辑一个pom.xml文件，这里项目根目录就是helloworld,这个文件用于定义maven项目的配置，首先我们创建一个最简单的文件，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--命名空间固定写法-->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
          http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!--配置maven坐标(GAV)-->
  <groupId>com.saul.maven</groupId>        <!--项目id-->
  <artifactId>HelloWorld-01</artifactId>        <!--模块id-->
  <version>0.0.1</version>                 <!--版本号id-->


</project>
```

### 加入一个java文件

然后我们编辑一个java文件,其目录就是上面用命令打开的目录

```java
package com.saul;
 
public class Hello
{
	public String sayHello(String name){
		return "hello:"+name;
	}
}
```

编辑完成以后，我们进入Helloworld目录，执行mvn compile命令，编译我们的项目，可以看到maven开始下载一列的插件，并且成功编译了我们的Hello.java文件。

```powershell
/*
执行编译命令
*/
C:\Users\Saulxk\Desktop\maven\HelloWorld>mvn compile


/*
再次查看项目的树形目录，可以看到在项目根路径下多出了一个target文件夹
其中classes中放置着对应的java文件编译过的类
*/
C:\Users\Saulxk\Desktop\maven>tree HelloWorld
文件夹 PATH 列表
卷序列号为 00000203 02B2:A470
C:\USERS\SAULXK\DESKTOP\MAVEN\HELLOWORLD
├─src
│  ├─main
│  │  └─java
│  │      └─com
│  │          └─saul
│  └─test
└─target
    ├─classes
    │  └─com
    │      └─saul
    └─maven-status
        └─maven-compiler-plugin
            └─compile
                └─default-compile
```

### 引入其他包

现在我们再创建一个junit测试类，来探究maven对junit包的引入过程。

此时我们构建一个Test类，把它放入test文件夹中的com.saul.test包中

```powershell
PS C:\Users\Saulxk\Desktop\maven\HelloWorld> tree
文件夹 PATH 列表
卷序列号为 00000041 02B2:A470
C:.
├─src
│  ├─main
│  │  └─java
│  │      └─com
│  │          └─saul
│  └─test
│      └─java
│          └─com
│              └─saul
│                  └─test
└─target
    ├─classes
    │  └─com
    │      └─saul
    └─maven-status
        └─maven-compiler-plugin
            └─compile
                └─default-compile
```

Test.java

```java
package com.saul.test;

import com.saul.*;
import org.junit.*;
import static org.junit.Assert.*;

public class Test
{
  @org.junit.Test
  public void test01(){
	Hello h=new Hello();
	String s=h.sayHello("maven");
	assertEquals(s,"hello:maven");
  }

}
```

可以看到Test.java文件中引入了org.junit但是，我们的项目路径中并没有该包，于是javac编译时会去环境变量中找，显然在我的环境变量中也没有这个包，编译必定失败，这时候如何引入org.junit呢？答案就是maven的配置文件pom.xml。首先我们可以去[maven仓库](http://mvnrepository.com)中搜索一下junit的依赖元素。如图：

![QQ图片20170805141557](\img\QQ图片20170805141557.png)

现在编辑我们的pom.xml文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--命名空间固定写法-->
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
          http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!--配置maven坐标-->
  <groupId>com.saul.maven</groupId>        <!--项目id-->
  <artifactId>HelloWorld-01</artifactId>        <!--模块id-->
  <version>0.0.1</version>                 <!--版本号id-->

  <!--我们添加了一个denpendencies元素，并把junit的依赖元素加入其中-->
  <dependencies>
      <dependency>
          <groupId>junit</groupId>        <!--项目id-->
          <artifactId>junit</artifactId>        <!--模块id-->
          <version>4.10</version>                 <!--版本号id-->
          <scope>test</scope>
      </dependency>
  </dependencies>
</project>
```

### test操作

现在我们用一个新的命令，该命令将完成编译和测试

`PS C:\Users\Saulxk\Desktop\maven\HelloWorld> mvn test`

  我们会发现，编译且测试成功，junit被自动加入到了我们的项目中，而我们没有手动载入jar包，也没有配置class_path，我们只是配置了一下xml文件，现在你知道maven给我带来了什么。我们的项目只要按照一定的目录构造，那么maven会自动根据pom.xml文件，对项目进行编译，测试，发布等操作，你不需要再去配置一些莫名其妙的东西，更便利的是你不再需要某些版本中包有冲突。

  机智的同学可能会发现，既然自动执行了测试，那么测试报告在哪里呢？此时我们仔细查看项目的树状目录可以看到，maven新生成了两个目录：

%项目目录%\target\surefire-reports   保存测试结果，成功和失败都保存

%项目目录%\target\test-classes  保存编译后测试类的字节码文件

```powershell
PS C:\Users\Saulxk\Desktop\maven\HelloWorld> tree
文件夹 PATH 列表
卷序列号为 000000CF 02B2:A470
C:.
├─src
│  ├─main
│  │  └─java
│  │      └─com
│  │          └─saul
│  └─test
│      └─java
│          └─com
│              └─saul
│                  └─test
└─target
    ├─classes
    │  └─com
    │      └─saul
    ├─maven-status
    │  └─maven-compiler-plugin
    │      ├─compile
    │      │  └─default-compile
    │      └─testCompile
    │          └─default-testCompile
    ├─surefire-reports
    └─test-classes
        └─com
            └─saul
                └─test

PS C:\Users\Saulxk\Desktop\maven\HelloWorld> cd .\target\surefire-reports\
PS C:\Users\Saulxk\Desktop\maven\HelloWorld\target\surefire-reports> dir

    目录: C:\Users\Saulxk\Desktop\maven\HelloWorld\target\surefire-reports

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         2017/8/5     14:38            266 com.saul.test.Test.txt
-a----         2017/8/5     14:38           5932 TEST-com.saul.test.Test.xml


```

### package操作

现在我们在来看一个新的操作

`mvn package`

这个命令，会把我们的项目打包成jar文件！

```powershell
PS C:\Users\Saulxk\Desktop\maven\HelloWorld> mvn package
PS C:\Users\Saulxk\Desktop\maven\HelloWorld> cd .\target\
PS C:\Users\Saulxk\Desktop\maven\HelloWorld\target> dir

    目录: C:\Users\Saulxk\Desktop\maven\HelloWorld\target

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         2017/8/5     13:17                classes
d-----         2017/8/5     17:54                maven-archiver
d-----         2017/8/5     13:14                maven-status
d-----         2017/8/5     14:33                surefire-reports
d-----         2017/8/5     14:32                test-classes
-a----         2017/8/5     17:54           2351 HelloWorld-01-0.0.1.jar
```

## 本地项目的关联

现在我们新建第二个项目，并且在这个项目中我们要使用第一个项目中的Hello类。

当然按照步骤我们首先创建pom.xml和固定的目录

```powershell
PS C:\Users\Saulxk\Desktop\maven\HelloWorld02> tree
文件夹 PATH 列表
卷序列号为 0000009A 02B2:A470
C:.
└─src
    ├─main
    │  └─java
    │      
    │          
    └─test
```

然后我们在Java目录下建立com.saul包，并增加一个java类

```java
package com.saul;
import com.saul.Hello;

public class Saul{
	
    public String sayHi(String name){
	Hello hi=new Hello();
        return hi.sayHello(name);	
    }	

}
```

此时如果尝试以mvn compile编译，肯定是失败的，在这里com.saul.Hello是无中生有的，编译器并不知道它在哪，于是我们需要改变一下pom.xml文件，你应该，我们在定义HelloWorld项目时定义了它的坐标(GAV)，对于其他项目来说也就是dependency。

```xml
  <!--配置maven坐标-->
  <groupId>com.saul.maven</groupId>        <!--项目id-->
  <artifactId>HelloWorld-01</artifactId>        <!--模块id-->
  <version>0.0.1</version>                 <!--版本号id-->
```

很熟悉不是么？

我们把HelloWorld2的pom.xml改写为

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
          http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.saul.maven</groupId>
  <artifactId>HelloWorld-02</artifactId>
  <version>0.0.1</version>
  <!--我们添加了一个denpendencies元素，并把junit的依赖元素加入其中-->
  <dependencies>
    <dependency>
      <!--配置maven坐标-->
      <groupId>com.saul.maven</groupId>        <!--项目id-->
      <artifactId>HelloWorld-01</artifactId>        <!--模块id-->
      <version>0.0.1</version>                 <!--版本号id-->
    </dependency>
  </dependencies>
</project>
```

现在我们还是不能正常编译，编译器无法找到HelloWorld项目，那是因为我们并没有对该项目进行mvn install 

```powershell
PS C:\Users\Saulxk\Desktop\maven\HelloWorld> mvn install
```

现在转化到HelloWorld2项目，进行mvn compile发现成功执行。

  那么你会问mvn install 到底做了什么呢？实际默认maven把我们在home目录下，Windows就是对应的user目录下的.m2文件夹设置为了本地仓库，pom.xml文件会引导maven去寻找相应的依赖包，如果在本地.m2中找到就用它，如果在.m2中不存在，那么就会去互联网的仓库中寻找，mvn install实际上就是把Hello项目放在了.m2中。比如我们可以查看一下：

```powershell
PS C:\Users\Saulxk\.m2> cd .\repository\
PS C:\Users\Saulxk\.m2\repository> cd .\com\
PS C:\Users\Saulxk\.m2\repository\com> tree
文件夹 PATH 列表
卷序列号为 00000014 02B2:A470
C:.
├─google
│  ├─code
│  │  └─findbugs
│  │      └─jsr305
│  │          └─2.0.1
│  ├─collections
│  │  └─google-collections
│  │      └─1.0
│  └─google
│      └─1
└─saul
    └─maven
        └─HelloWorld-01
            └─0.0.1
```

