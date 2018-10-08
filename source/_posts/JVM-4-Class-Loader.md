---
title: 'JVM(4):类加载器机制'
date: 2017-12-05 11:45:40
toc: true
tags:
 - JVM
 - 学习笔记
---

  类加载执行的前提是我们能以一个类的全限定名来获取字节流，至于字节流在哪里生成，我们从哪里获取这并不是虚拟机所关心的，作为一个出色的设计团队，Java的设计者最初为了满足Java Applet，设计了大名鼎鼎的类加载器机制，**即让通过全限定名获取字节流的这个动作，放到JVM的外部，用户可以选择自行实现一些特殊的加载方案**，现在你知道的随着flash的流行，Java Applet这种技术已经淘汰了，现在随着html5的新起，flash也基本上接近夕阳了，可见技术的变化是普遍而剧烈的。虽然Applet死了很多年，但是类加载器却被作为一种精髓留在了java体系中，原因是它在OSGi,代码加密，热部署等等应用中有了新生的土壤。

  类加载器还有一个作用就是和类的标识一起来确定这个类的唯一性，若N1是类C的全限定名称标识，而L1是加载类C类加载器，那么如果存在一个类D的全限定名称为N2,而L2是类D的加载器，那么：

 当且仅当，N1=N2且L1=L2时C和D才被认为是同一个类，即C&lt;N1,L1&gt;=D&lt;N2,L2&gt;

 这种相等，关系到类的Class对象的equals方法，isAssignableFrom()方法以及isInstance()方法和instanceof关键字的判断结果。

<!--more-->

```java
public class Hello {

	public static void main(String[] args) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
      	//从写一个自定义类加载器，用于加载同路径下的类，如果一个类名不是同路径下的就用父类加载器加载
		ClassLoader loader=new ClassLoader() {
			public java.lang.Class<?&gt; loadClass(String name) throws ClassNotFoundException {
				try {
					String fn=name.substring(name.lastIndexOf(".")+1)+".class";
					InputStream resourceAsStream = getClass().getResourceAsStream(fn);
                  	  //同路径下没有该类，用父类加载器加载
					if(resourceAsStream==null) return super.loadClass(name);
					byte[] bs=new byte[resourceAsStream.available()];
					resourceAsStream.read(bs);
					return defineClass(name, bs, 0, bs.length);
				} catch (ClassFormatError e) {
					e.printStackTrace();
				} catch (IOException e) {
					e.printStackTrace();
				}
				return null;
			};
		};
		Object newInstance = loader.loadClass("test.Hello").newInstance();
		Hello hello = new Hello();
		System.out.println(newInstance.getClass().getName());
		System.out.println(hello.getClass().getName());
		System.out.println(newInstance.getClass().equals(hello.getClass()));
	}
}

//可以看出虽然全限定名相同但是却是不同的类
输出：
test.Hello
test.Hello
false
```

## 类加载器分类

  在官方文档中，我们中看到了两种类加载器的区分，一种就是[启动类加载器](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.3.1)（Bootstrap Class Loader ）还一种就是[用户定义加载器](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.3.2)（ User-defined Class Loader ）。

原文：

```
There are two kinds of class loaders: the bootstrap class loader supplied by the Java Virtual Machine, and user-defined class loaders. Every user-defined class loader is an instance of a subclass of the abstract class ClassLoader
```

### 启动类加载器

  这个**启动类加载器**是虚拟机本身定义的，它是以C++来实现的，用于加载java的核心库&lt;JAVA_HOME&gt;\lib中或者-Xbootclasspath参数指定的，这些库都要被虚拟机所识别比如rt.jar，随便定义一个库放到以上文件夹里是不会被虚拟机加载的。

### 用户自定义类加载器

  对于**用户定义类加载器**来说，它的工作步骤大概是这样的：

  首先JVM判断用户定义加载器L是不是已经记录为全限定名为N的类或者接口的初始加载器，如果是就直接返回这个类C，没必要从新创建一个类。

  否则，JVM就调用L的loadClass方法，参数为N，这此调用创建一个新的类或者接口C并返回，然后JVM会记录L是C的初始加载器。详细来说这个过程会执行接下来的两个操作中的一个：

  **操作1**：类加载器L可以创建一个字节数组用于表示C在Class文件结构中的字节流，之后L调用Classloader类中的defineClass方法，defineClass方法会让JVM用这个字节数组派生出一个类&lt;N,L&gt;（类的唯一性标识，上面提到过）

  **操作2**：类加载器L可以把C的加载委托给另一个加载器L',这个过程通过用N直接或者间接地调用L‘的方法的完成，一般是loadClass方法，然后调用的结果是C。

无论是操作1还是操作2，如果类加载器不能加载全限定名为N的类，就会抛出ClassNotFoundException异常实例。

用户定义类加载器这个分类，并不是指这个加载器一定是用户定义的，比如有两个系统定义的加载器也属于这个分类：

1.扩展类加载器(extensions class loader)：用于加载Java的扩展库，即&lt;JAVA_HOME&gt;\lib\ext目录中的或者由java.ext.dirs系统变量指定的。

2.应用类加载器(Application class loader)：根据Java应用的类路径CLASSPATH来加载Java类，一般Java应用的类都由它加载，我们可以通过ClassLoader中的getSystemClassLoader方法来获取它。

## 双亲委派模式

![QQ图片20171205143304](\img\QQ图片20171205143304.png)

从图中（这里箭头不是继承关系）可以看出，除了顶层的启动类加载器以外，其余的类加载器都应当有自己委派类加载器，一般用组合关系来实现这种委派关系。

双亲委派模式是指：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求为派给上层的加载器去完成，只有当**上层加载器不能加载的时候才会尝试自己去加载**，这样就保证了java类有一种层次关系，保证了即使你用不同的类加载器去执行类加载，但是实际上我们一般都会得到相同的&lt;N,L&gt;,不会导致类关系的错乱。

双亲委派模式是保证Java程序稳定运行的重要部分。但并不是强制约束模型，实际上在OSGi等环境下这种模式已经被改造，不再基于树桩结构而是基于网状结构。