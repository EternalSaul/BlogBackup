---
title: 'Learning Scala(5):其他集合'
date: 2017-07-15 08:14:55
toc: true
tags:
 - scala
 - 学习笔记
---

##  可变集合简绍

需要明确的是，List,Set,和Map都是不可变集合。你用::操作符也好，映射也好，实质上是构造了一个新的集合而不是在原基础上直接改动。

  不可变集合可以带来很多优势，提高稳定性就是一个非常明显的优点。但是有时候你可能必须使用一个可变的数据结构，作为一种强大的多范式语言，Scala当然也提供了这些,它们属于collection.mutable包，一般来说collection.immutable会自动增加到Scala的当前命名空间，然而collection.mutable不会。

**可变的集合类型和不可变集合类型对比**

| 可变类型                      | 不可变类型                     |
| ------------------------- | ------------------------- |
| collection.mutable.Buffer | collection.immutable.List |
| collection.mutable.Set    | collection.immutable.Set  |
| collection.mutable.Map    | collection.immutable.Map  |

<!--more-->

## 创建可变集合

### 一般的创建

由于collection.mutable不会自动增加到当前的命名空间，所以我们在以构造方法构造可变集合时要加上全名，如下：

```scala
/*
编译器不知道Buffer是什么
因为没有引入collection.mutable
*/
scala> val nums=Buffer(1)
<console>:11: error: not found: value Buffer
       val nums=Buffer(1)

/*
通过完全的名称构造Buffer
*/
scala> val nums=collection.mutable.Buffer(1)
nums: scala.collection.mutable.Buffer[Int] = ArrayBuffer(1)

/*
为Buffer增加元素
*/
scala> for(i <-1 to 10 if(i%2!=0)) nums+=i

scala> nums
res3: scala.collection.mutable.Buffer[Int] = ArrayBuffer(1, 1, 3, 5, 7, 9)
```

### 从不可变集合创建

上次篇文章里讲到过集合之间可以互转，无论可变还是不可变他们都是集合。

```scala
/*
把一个不可变的Map转化为Buffer
*/
scala> Map(("CCTV","新闻联播"),("湖南卫视","快乐大本营"))
res0: scala.collection.immutable.Map[String,String] = Map(CCTV -> 新闻联播, 湖南卫视 -> 快乐大本营)

scala> res0.toBuffer
res1: scala.collection.mutable.Buffer[(String, String)] = ArrayBuffer((CCTV,新闻联播), (湖南卫视,快乐大本营))

/*
把一个Buffer转化为不可变的List
*/
scala> val nums=collection.mutable.Buffer(1,2,3,4,5)
nums: scala.collection.mutable.Buffer[Int] = ArrayBuffer(1, 2, 3, 4, 5)

scala> nums.toList
res2: List[Int] = List(1, 2, 3, 4, 5)

/*
如果是把一个集合转化为Set，则会自动去除重复元素
*/
scala> nums+=1
res3: nums.type = ArrayBuffer(1, 2, 3, 4, 5, 1)

scala> nums+=5
res4: nums.type = ArrayBuffer(1, 2, 3, 4, 5, 1, 5)

scala> nums.toSet
res5: scala.collection.immutable.Set[Int] = Set(5, 1, 2, 3, 4)
```

### 使用集合构建器

Builder是Buffer的一个简化形式，仅限于生成指定的集合类型，而且只支持追加操作。

要为一个特定的集合类型创建构造器，可以调用这个类型的newBuilder方法，并指定集合元素的类型。如果你需要迭代的构造一个可变集合，并且最后需要把它转变为不可变集合，那么集合构造器就是一个非常好的选择。

```scala
/*
创建一个List的构造器
*/
scala> val bd=List.newBuilder[Int]
bd: scala.collection.mutable.Builder[Int,List[Int]] = ListBuffer()

/*
构造器是可变的
*/
scala> bd+=1
res7: bd.type = ListBuffer(1)

scala> bd+=2
res8: bd.type = ListBuffer(1, 2)

scala> bd+=3
res9: bd.type = ListBuffer(1, 2, 3)

//你可以加上一个不可变集合
scala> bd++=List(1,2,4,5)
res10: bd.type = ListBuffer(1, 2, 3, 1, 2, 4, 5)

//也可以加上一个可变集合
scala> bd++=scala.collection.mutable.Buffer(1,2,4,5)
res12: bd.type = ListBuffer(1, 2, 3, 1, 2, 4, 5, 1, 2, 4, 5)

/*
通过result方法得到一个List，因为bd是List的构造器
*/
scala> val ls=bd.result
ls: List[Int] = List(1, 2, 3, 1, 2, 4, 5, 1, 2, 4, 5)
```

## 数组

  Scala中Array其实不在scala.collection包里，它包含了所有的Iterable操作，但它不是Iterable的子类，实质上它是Java数组类型的一个包装器，并且提供了一个被称为隐含类(implicit class)的高级特性。像其他语言的数组那样，它是一个大小不可变而内容可变的集合。

```scala
scala> val c=Array(1,2,3,4,5)
c: Array[Int] = Array(1, 2, 3, 4, 5)

scala> c(5)
java.lang.ArrayIndexOutOfBoundsException: 5
  ... 29 elided

scala> c(1)
res14: Int = 2

scala> val c=Array(1,2,3,4,5)
c: Array[Int] = Array(1, 2, 3, 4, 5)

scala> c(1)
res15: Int = 2

scala> val f=new java.io.File(".").listFiles
f: Array[java.io.File] = Array(.\.android, .\.AndroidStudio2.1, .\.AndroidStudio2.2, .\.bash_history, .\.config, .\.eclipse, .\.emulator_console_auth_token, .\.gitconfig, .\.gradle, .\.h2.server.properties, .\.hilberteffect, .\.node_repl
_history, .\.oracle_jre_usage, .\.p2, .\.scalaide, .\.scala_history, .\.ssh, .\.swt, .\.viminfo, .\.WebStorm11, .\AndroidStudioProjects, .\AppData, .\Application Data, .\authority.sql, .\bsss.sql, .\bsss004.sql, .\bsss~1.sql, .\Contacts,
 .\Cookies, .\debian-8.8.0-i386-CD-1.iso.part, .\Desktop, .\Documents, .\Downloads, .\Favorites, .\Foxit Reader SDK ActiveX.ini, .\Intel, .\IntelGraphicsProfiles, .\joan01.sql, .\Links, .\Local Settings, .\Music, .\My Documents, .\NetHoo
d, .\NTUSER.DAT, .\ntuser.dat.LOG1, .\ntuser.dat.LOG2, .\NTUSER.DAT{ace096cc-49bb-11e7...

scala> f map(_.getName) filter(_.endsWith("sql"))
res19: Array[String] = Array(authority.sql, bsss.sql, bsss004.sql, bsss~1.sql, joan01.sql, sysdba77777.sql, tom01.sql)
```

## 序列

  下图是序列集合的层次体系，可以看到Seq是所有序列的根类型。Seq本身不能实例化，但是可以调用Seq创建List这是一个创建List的快捷方式，然而个人觉得没什么用，只是少了一个字而已。

```scala
scala> val s=Seq(1,2,3,4,5)
s: Seq[Int] = List(1, 2, 3, 4, 5)
```
**序列集合类型层次图**
<img src="\img\QQ图片20170715093053.png" width = "600" height = "400" alt="图片名称" align=center />

**序列类型表**

| 描述                        | 类型名        |
| ------------------------- | :--------- |
| 所有序列的根类型，List的快捷方式        | Seq        |
| 索引序列的根类型，Vector的快捷方式      | IndexedSeq |
| 这一类列表有一个后备Array实例，可以按索引访问 | Vector     |
| 整数范围，动态生成数据               | Range      |
| 线性(链表)序列的根类型              | LinearSeq  |
| 元素的单链表                    | List       |
| 队列                        | Queue      |
| 栈                         | Stack      |
| 懒列表，访问元素时才增加相应元素          | Stream     |
| 字符集合                      | String     |

### String是序列？

其中String也被认为是序列，而且String类型是一个不可变集合，它扩展了Iterable，支持Iterable的操作，同时还可以作为Java字符串的一个包装器，支持java.lang.String操作。

```scala
/*
可以看到String不仅可以调用Iterable的take操作还可以调用String的replace方法
*/
scala> val s="halo,"++"worldly" take 10 replace ('w','W')
s: String = halo,World
```

### 懒集合Stream

  如果你用过hibernate框架，你可能知道一种叫做懒加载的机制。类似的，Stream是由一个或多个起始元素和一个递归函数生成，第一次访问元素时才会把这个元素增加到集合中。

  理论上说流是无界的，只是在访问元素时才会生成这个元素，流也可以以Stream.Empty结束。流是递归数据结构，包括一个表头和一个表尾，其中表头是当前元素，而表尾则指向其余部分，可以利用一个函数以及该函数的递归调用来构造，这个函数返回以个新的流，该函数的递归调用可以构建表尾，也可以使用Stream.cons用表头和表尾构建一个新的流。

```scala
/*
通过点分cons操作来构造递归函数
*/
scala> def inc(i:Int):Stream[Int]=Stream.cons(i,inc(i+1))
inc: (i: Int)Stream[Int]

scala> val s=inc(2)
s: Stream[Int] = Stream(2, ?)

scala> s.take(20).toList
res22: List[Int] = List(2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21)

scala> s
res23: Stream[Int] = Stream(2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, ?)

/*
通过cons操作符#::来构造递归函数
#::也是右操作符
*/
scala> def inc(head:Int):Stream[Int]=head#::inc(head+1)
inc: (head: Int)Stream[Int]

scala> val s=inc(10)
s: Stream[Int] = Stream(10, ?)

scala> s.take(5).toList
res38: List[Int] = List(10, 11, 12, 13, 14)
```

  以#::来构造流可能会让人感到很诧异，我们根本没有显示构造Stream类型，实际上这里编译器可以根据递归函数的类型进行隐式转换。