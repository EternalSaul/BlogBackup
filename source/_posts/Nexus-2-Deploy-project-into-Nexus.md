---
title: 'Nexus (2):将项目部署到Nexus'
date: 2017-08-12 09:34:11
tags:
  - Nexus
  - Maven
  - 学习笔记
---

  到目前为止我们已经可以通过Nexus的仓库来下载我们所需要的依赖包，现在我们面对另一个问题，那就是如何把我们的项目发布到Nexus仓库上呢？事实上在一个多人写作的多模块项目中，这无疑是非常重要的，我们有时候需要更新其他人的项目来完成测试等等工作，并在最终集成为最终项目。

  实际上以我们上一篇文章讲到的配置项，还不足以让我们发布项目，要发布项目我们还需要更多的配置，比如我们需要配置一个有权发布的用户账号和密码，还需要配置我们发布到哪一个仓库中。

<!--more-->

  首先我们可以在母项目的pom.xml中加入这个：

```xml
<!--配置我们需要发布的release和snapshits仓库--> 
<distributionManagement>
  	<repository>
  		<id>app-releases</id>
  		<name>app releases repo</name>
  		<url>http://localhost:9081/repository/maven-releases/</url>
  	</repository>
  	<snapshotRepository>
  		<id>app-snapshots</id>
  		<name>app snapshots repo</name>
  		<url>http://localhost:9081/repository/maven-snapshots/</url>
  	</snapshotRepository>
  </distributionManagement>
```

另外我们需要在setting.xml中配置我们的用户,但是因为我们用的是Nexus3.5,所以我们需要创建一个新的角色，并把增删改查权限给这个角色，在为该角色创建一个账号（当然你可以直接用admin，但是我不推荐这么做）

这里我创建了一个deployer角色，授予它的权限是nx-repository-view-maven2-\*-\*和nx-repository-admin-*-\*-\*

![QQ图片20170812100953](\img\QQ图片20170812100953.png)

然后我以该角色创建了一个用户deploy

![QQ图片20170812101008](\img\QQ图片20170812101008.png)

现在我们可以配置setting.xml了,加入：

```xml
  <servers>
    <server>
      <!--这里的id要和pom.xml中一致-->
      <id>app-releases</id>
      <username>deploy</username>
      <password>deploy123</password>
    </server>
    <server>
      <id>app-snapshots</id>
      <username>deploy</username>
      <password>deploy123</password>
    </server>
  </servers>
```

然后我们可以执行命令

`mvn deploy`

执行之后我们去查看maven-snapshots仓库，可以看到我们的项目已经发布到了snapshots里面，这里我的母项目里的模块只有app-user，没有其他的。

![QQ图片20170812112536](\img\QQ图片20170812112536.png)









