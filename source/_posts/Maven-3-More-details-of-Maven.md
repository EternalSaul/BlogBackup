---
title: 'Maven (3):依赖包的传递'
date: 2017-08-07 11:33:08
toc: true
tags:
 - Maven
 - 学习笔记
---

  现在你已经学会了在eclipse中构建一个maven项目，但实际上你构建的maven项目并不完备，比如在打包时，你可能已经完成了测试，所以你并不需要将dbunit或者junit等包加入到你最后的打包中，实际上这是可以实现的，如果你以前在maven repository中下载过依赖包，你会发现这个对应依赖包的GAV种还有一些其他的元素标签,如：

```xml
<!-- https://mvnrepository.com/artifact/junit/junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

 这个scope标签对应的就是maven的依赖范围，里面的值有很多种，其中最常见的四种就是：

**1.test**

该范围指依赖只在测试时有效

**2.compile**

该范围是默认范围，你什么都不加就默认为compile，该范围在所有要编译的场景都有效，比如打包，测试等等

**3.provide**

该范围只在编译和测试时有效，打包时不会加入****

**4.runtime**

只在运行时依赖，编译时不依赖

<!--more-->

## 依赖包的传递

### 简单的传递

我们已经了解了maven自动导入依赖包的作为范围，实际上包的依赖是传递的，比如：

  **A->B C->A 那么 C->B**

比如：

![QQ图片20170807174126](\img\QQ图片20170807174126.png)

对于上面三个项目

**app-user -> app-core**

**appfinal -> app-user**

  于是对于这种依赖会影响到包的传递上，可以看到下图，app-final也会引入app-core包：

![QQ图片20170807175113](\img\QQ图片20170807175113.png)

### Maven的就近传递

  我们知道了，maven配置的依赖是会传递的，这很有意思，也值得我们去深究一下，如果我们的依赖不是**简单的单层依赖**会发生什么？

  假设：

​     **A->B ，B->C， C->D1.2**

​    **A->E，E->D1.1**

​    **这里的D对应了不同的版本，1.2高于1.1**

![QQ图片20170808111644](\img\QQ图片20170808111644.png)



这样会发生什么呢？**Maven会自动选择继承高版本的D1.2么？**

**答案是不会！Maven根本分不清你的版本高低问题**

实际上Maven会选择就近继承依赖，也就是说D1.1在树形结构的第二层，而D1.2在第三层，于是**Maven选择了D1.1**

那么事实是不是这样呢？

我们知道对于刚刚说到的三个项目：

![QQ图片20170807174126](\img\QQ图片20170807174126.png)

**app-user -> app-core**

**appfinal -> app-user**

  现在我配置pom.xml让app-core依赖于junit4.1而让app,final依赖于junit3.8,于是出现了下面的结果：

![QQ图片20170808112352](\img\QQ图片20170808112352.png)

我们可以清楚的看到，Maven选择了就近依赖

## 依赖冲突与解决

###   依赖冲突

  从上面一节，我们可以看到依赖具有传递性，那么我们来考虑一下一个有趣的事情，如果发生了这样的情况：

​     **A->B ，B->D1.2**

​    **A->C，C->D1.1**

![QQ图片20170808112722](\img\QQ图片20170808112722.png)

  此时，我们可以看到D1.1和D1.2离根A是一样近的，那么这时候就发生了**包冲突**，冲突之后依赖是怎么传递的呢？

同样回到app的那个例子，现在我们让

**app-final -> app-core->Apache Commons Codec 1.10**

**app-final -> app-user->Apache Commons Codec 1.9**

![QQ图片20170808113508](\img\QQ图片20170808113508.png)

实际上这时候依赖的传递，取决于谁先传递！那么到底是谁先传递呢？我们知道pom.xml是一个xml文件，而xml也是树形结构的，那么在扫描pom.xml文件的时候，扫描的次序就有先后顺序，如果app-final的pom.xml中这样标识依赖：

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <!--请注意core在前，user在后-->
    <dependency>
      <groupId>xin.saul.app</groupId>
	  <artifactId>app-core</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>xin.saul.app</groupId>
	  <artifactId>app-user</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
    </dependency>
  </dependencies>
```

**此时core在前，user在后，所传递的就是core对应的版本，也就是Apache Commons Codec 1.10！！**

![QQ图片20170808114129](\img\QQ图片20170808114129.png)



如果pom.xml依赖关系是这样标识的：

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <!-- 注意现在是user在前，core在后 -->
    <dependency>
      <groupId>xin.saul.app</groupId>
	  <artifactId>app-user</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
      <groupId>xin.saul.app</groupId>
	  <artifactId>app-core</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
    </dependency>
  </dependencies>
```

**这时候，所传递的就是user对应的版本，也就是Apache Commons Codec 1.9！！**

现在结合前面的传递就近原则，我们可以推测出传递关系是按照**树形层次遍历**而来的！！先遍历到谁，就是谁传递，**同样的组和模块的包（groupId和artifactId相同），即使版本不同将会被忽略**

### 解决包冲突

  有时候我们并不知道包之间有着什么样的依赖！但是有时候我们不得不使用某个版本的包，比如说Spring 4.3就比4.1多了不少注解，而Spring 4.3和jackson的老版本有一些冲突，所以我们如何保证一定能使用到不冲突的jackson版本2.8呢？

答案是：排除依赖包

```xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <!-- 注意现在core在前，user在后，但是我们排除了core中codec包的依赖传递 -->
    <dependency>
      <groupId>xin.saul.app</groupId>
	  <artifactId>app-core</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
	  	<exclusions>
		  	<exclusion>
	  		 <groupId>commons-codec</groupId>
	   		 <artifactId>commons-codec</artifactId>
		  	</exclusion>
		</exclusions>
    </dependency>
    <dependency>
      <groupId>xin.saul.app</groupId>
	  <artifactId>app-user</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
    </dependency>
  </dependencies>
```

  把一个包声明为exclusion相当于取消对这个包的依赖，也就是说codec1.10不会进行依赖传递，于是app-final就能依赖于codec1.9了，但是需要注意的是，这种排除依赖，是根本上取消，假设如果我们的pom.xml中这样声明：

```xml

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    <!-- 注意现在只依赖于core -->
    <dependency>
      <groupId>xin.saul.app</groupId>
	  <artifactId>app-core</artifactId>
	  <version>0.0.1-SNAPSHOT</version>
	  	<exclusions>
		  	<exclusion>
	  		 <groupId>commons-codec</groupId>
	   		 <artifactId>commons-codec</artifactId>
		  	</exclusion>
		</exclusions>
    </dependency>
  </dependencies>
```

现在：

![QQ图片20170808120918](\img\QQ图片20170808120918.png)

**但是由于我们排除了对codec的依赖，那么，codec包不会传递！！**

![QQ图片20170808121053](\img\QQ图片20170808121053.png)

  可以看到，app-final中根本没有引入codec1.10，所以使用排除时务必小心，否则会导致工程无法编译成功。







