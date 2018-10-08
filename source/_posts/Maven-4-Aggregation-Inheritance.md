---
title: 'Maven (4):聚合与继承'
date: 2017-08-08 13:52:15
toc: true
tags:
 - Maven
 - 学习笔记
---

  前面几篇文章，我们都是把项目拆分成三个模块来管理，每次编译时，我们可能要把每个模块编译一次，假设我们真实做项目有100多个模块，我想你肯定不太愿意去将他们一个个编译，那么Maven作为一个成熟的项目管理和构建工具，肯定提供了某种方式让我们避免做一些从复性的工作，这篇文章主要就是介绍我们如何完成多个模块的聚合。

## 隐式变量

  首先我想介绍，隐式变量，在工程中，我们都习惯于把很多配置项加到某个配置文件然后通过${}的方式来访问，比如数据库数据源的配置，还有一些文件保存位置等等，这样便于我们统一管理配置项，不用到时候一个个去改，Maven的隐式变量也有这个功能。

<!--more-->

  Maven的隐式变量分为3种：

### env变量

  对应操作系统的环境变量，${env.XXXX}来表示，比如\${env.JAVA_HOME}标明了我在环境变量中配置的Jdk路径

### project变量

   这个变量对应着pom.xml的project种的元素，比如对于如下pom.xml:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <!--${project.XXXX}表示的就是一部分的元素-->
  <groupId>xin.saul.app</groupId>
  <artifactId>app-final</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>app-final</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    
    <!--此处${project.groupId}表示的是project元素中的groupId,这是即使xin.saul.app-->
    <dependency>
      <groupId>${project.groupId}</groupId>
	  <artifactId>app-core</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>${project.groupId}</groupId>
	  <artifactId>app-user</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
    </dependency>
  </dependencies>
</project>

```

除了这样表示以外，其实我们可以以层级表示，比如对于project的子元素build(可以自己在eclipse环境下用代码补全测试看看有哪些子元素)

**build中有很多项，其中有一些项在没有配置的情况下有默认值（缺省值）**

比如：

​    **${project.build.directory}  表示构建目录，默认为target**

​    **${project.build.outputDirectory}    表示过程输出目录，默认为target/classes**

​    **${project.build.finalName} 表示输出内容，默认为 \${project.artifacetId}- \${project.version}** 

​    **${project.packaging} 打包类型，默认为jar**

​    **${basedir} 项目根目录**

### settings变量

   以${setting.XXX}表示，指向的是maven的setting文件的信息，我们曾在[第二篇文章](/2017/08/06/Maven-2-Simple-Configuration-of-Maven/)中配置过该文件

## 聚合

​    在学习了隐式变量以后，我们再来看如果把多个模块聚集在一起编译测试，而不是一个个来分别点击编译和测试。

​    现在我们构建一个新的简单maven项目app-aggregation，它专门用来做聚合操作。将app-aggretion的pom.xml文件配置为：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>xin.saul.app</groupId>
  <artifactId>app-aggregation</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  
  <!-- 将包打成pom类型，这里不能打成jar或者war -->
  <packaging>pom</packaging>
  
  <!-- 配置模块 -->
  <modules>
  	<module>../app-core</module>
  	<module>../app-user</module>
  	<module>../app-final</module>
  </modules>
</project>
```

  现在编译app-aggregation，就会同时编译modules元素中的模块，即实现了模块的聚合编译

实际上如果你不在eclipse中编译的话，可以直接把所有项目添加到一个文件夹，然后把这个文件夹中建立pom.xml文件并配置模块，当做maven项目来用mvn compile来编译。

##  继承

   很多时候我们在pom.xml中写的配置都是重复的，而且我们如果要改的话，那必须改所有项目模块的pom.xml，如果有几百个你可能会拒绝这种低效的工作，实际上我们可以通过继承来将一些公用配置写在一个pom.xml中，配合隐式变量我们就可以只改这个pom.xml从而实现对整个项目的更改。

   现在我们创建一个新的项目app-parent，它用来做共有的配置。

现在对于app-final的pom.xml:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>xin.saul.app</groupId>
  <artifactId>app-final</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
  

  <name>app-final</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <!-- 注意现在user在前，core在后 -->
    <dependency>
      <groupId>${project.groupId}</groupId>
	  <artifactId>app-core</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>${project.groupId}</groupId>
	  <artifactId>app-user</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
    </dependency>
  </dependencies>
</project>

```

我们要对它做一些改动，把公共部分转移到app-parent/pom.xml中，并且在app-final/pom.xml以隐式变量实现复用。

母项目pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>xin.saul.app</groupId>
  <artifactId>app-parent</artifactId>
  <!-- 以全局属性来配置版本号 -->
  <version>${global.version}</version>
  
  <!-- 将包打成pom类型，这里不能打成jar或者war -->
  <packaging>pom</packaging>
  
  <!-- 配置模块 -->
  <modules>
  	<module>../app-core</module>
  	<module>../app-user</module>
  	<module>../app-final</module>
  </modules>
  
    <url>http://maven.apache.org</url>
    
  <!-- 声明全局属性 -->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <global.version>0.0.1-SNAPSHOT</global.version>
  </properties>
    
  <!-- 配置依赖管理，这里有全局的依赖包和他们对应的版本号，依赖包会不会属于子项目的依赖，取决于是否在子项目中配置denpendency -->
  <dependencyManagement>
  	<dependencies>
		<dependency>
		  <groupId>junit</groupId>
	      <artifactId>junit</artifactId>
	      <version>4.10</version>
		</dependency>  	
		<dependency>
		    <groupId>commons-codec</groupId>
		    <artifactId>commons-codec</artifactId>
		    <version>1.10</version>
		</dependency>
  	</dependencies>
  </dependencyManagement>
  
</project>
```

下面我们来看子模块的配置

首先看app-core的配置,app-core中接受了全局的两个依赖junit和codec

![QQ图片20170809101651](\img\QQ图片20170809101651.png)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
  	  <groupId>xin.saul.app</groupId>
	  <artifactId>app-parent</artifactId>
	  <relativePath>../app-parent/pom.xml</relativePath>
	  <!-- 版本号已经在母pom.xml中的全局属性中声明，并且在这里用隐式变量引入 -->
	  <version>${global.version}</version>
  </parent>
  
  <!-- 重复的groupId和version都不需要填写 -->
  <artifactId>app-core</artifactId>
  <packaging>jar</packaging>

  <name>app-core</name>


  <dependencies>
    <!-- 接受全局的依赖，这里不用写版本号，因为已经在母pom.xml中写了，如果这里不填写commons-codec的依赖的话该子模块就不会引入commons-codec包 -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
    </dependency>
    <!-- https://mvnrepository.com/artifact/commons-codec/commons-codec -->
	<dependency>
	    <groupId>commons-codec</groupId>
	    <artifactId>commons-codec</artifactId>
	</dependency>
  </dependencies>
</project>

```

再来看app-user,它只在依赖中填写了junit的依赖，但是却声明了与母pom.xml中不同的版本，根据就近原则，它会采用自己的版本，并且app-user声明了母pom.xml中没有的依赖maybatis，对于母pom.xml中的依赖codec我们没有看到它被引入，因为他在子模块的pom.xml中没有被声明！



![QQ图片20170809101956](\img\QQ图片20170809101956.png)

```xml
<!--app-user的pom.xml-->

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  
  <parent>
  	  <groupId>xin.saul.app</groupId>
	  <artifactId>app-parent</artifactId>
	  <relativePath>../app-parent/pom.xml</relativePath>
	   <version>${global.version}</version>
  </parent>
  
  <artifactId>app-user</artifactId>
  <packaging>jar</packaging>

  <name>app-user</name>

  <dependencies>
    <!--声明了junit依赖但用了不同的版本，且没声明codec-->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
    </dependency>
    <!--声明了自己的依赖-->
    <dependency>
	    <groupId>org.mybatis</groupId>  
	    <artifactId>mybatis-spring</artifactId>  
	    <version>1.2.2</version>  
    </dependency>
  </dependencies>
</project>

```

