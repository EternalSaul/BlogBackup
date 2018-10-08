---
title: 'Learning Scala(4):不可变集合'
date: 2017-07-14 12:31:01
toc: true
tags:
 - scala
 - 学习笔记
---

## 不可变集合

  因为是JVM语言，Scala代码可以访问和使用整个JAVA集合库,但是如果你想体会到Scala中高阶函数带来的种种好处，你应该选择Scala本身提供的集合。

  Scala集合提供了大量的高阶操作，如map,reduce,filter等等，如果你经常使用jquery中的包装集的各种操作，你就会立刻联想到这些高阶操作带来的好处。Scala的集合又分为可变和不可变集合两类，利用不可变性可以保证程序更加稳定，当然在必要时也可以转化为可变类型。在这里，我们首先来看三种不可变集合，它们分别是List,Set,Map。

<!--more-->

### 列表

List类型表示的是一个不可变的单链表。可以通过下面方式来构造一个列表:

```scala
//构造整数列表nums
scala> val nums=List(3,2,1,4,5,6)
nums: List[Int] = List(3, 2, 1, 4, 5, 6)

/*
查看nums的大小
*/
scala> println(s"nums size is ${nums.size}")
nums size is 6

/*
查看nums
*/
scala> println(s"$nums")
List(3, 2, 1, 4, 5, 6)

/*
访问nums的首元素
这里的返回类型是元素类型:Int
*/
scala> nums.head
res27: Int = 3

/*
访问nums的其余元素
这里的返回类型是列表类型:List[Int]
*/
scala> nums.tail
res28: List[Int] = List(2, 1, 4, 5, 6)

//利用索引(从0开始)访问单个元素
scala> nums(1)
res38: Int = 2

scala> nums(0)
res40: Int = 3


/*
利用for循环结构对列表进行遍历
*/
scala> var total=0
total: Int = 0

scala> for(i<-nums){total+=i}

scala> total
res42: Int = 21

```



  在前几次博文中我们提到了高阶函数(higher-order function)，它包含一个函数类型的值作为输入参数或返回值。

  我们之前已经说了高阶函数在scala中的重要地位和普遍性，在scala集合中当然也大量使用了高阶函数来完成迭代，映射，归约等等操作，比如如下几个操作就是他们中的典型分子

```scala
/*
使用归约操作计算nums所有元素的和
*/
scala> nums.reduce(_+_)
res48: Int = 21

/*
使用映射操作得到nums中的元素+1后的列表
*/
scala> nums.map(_+1)
res49: List[Int] = List(4, 3, 2, 5, 6, 7)

/*
遍历整个nums列表，输出每一个元素值
*/
scala> nums.foreach(println(_))
3
2
1
4
5
6
```



### 集

Set是一个不可变得无序集合，只包含不重复的唯一元素。它也支持与List的类似操作。

```scala
//创建Set
scala> val numset=Set(1,2,3,4,6,7)
numset: scala.collection.immutable.Set[Int] = Set(1, 6, 2, 7, 3, 4)

scala> numset.reduce(_+_)
res52: Int = 23

/*
访问Set中是否存在某个元素，如果存在则返回true不存在则返回false
*/
scala> numset(2)
res54: Boolean = true

scala> numset(3)
res55: Boolean = true

scala> numset(10)
res56: Boolean = false
```



### 映射

Map则是一个不可变得键/值库,创建Map时指定键-值对为元组(Tuple)。

```scala
//创建Map,这里如果你以("失控","凯文凯利")的形式构造元组也是没有问题的
scala> val mmap=Map("失控"->"凯文凯利","国富论"->"亚当斯密","通论"->"凯恩斯","数据结构与算法分析"->"Mark Allen Weiss")
mmap: scala.collection.immutable.Map[String,String] = Map(失控 -> 凯文凯利, 国富论 -> 亚当斯密, 通论 -> 凯恩斯, 数据结构与算法分析 -> Mark Allen Weiss)

scala> mmap("国富论")
res64: String = 亚当斯密

scala> mmap("数据结构与算法分析")
res65: String = Mark Allen Weiss

scala> mmap("通论")
res66: String = 凯恩斯

scala> for(i<-mmap)(println(i))
(失控,凯文凯利)
(国富论,亚当斯密)
(通论,凯恩斯)
(数据结构与算法分析,Mark Allen Weiss)

scala> for(i<-mmap)(println(i._2))
凯文凯利
亚当斯密
凯恩斯
Mark Allen Weiss
```





  现在我们了解了最最常用的不可变集合的三种类型，它们都是Iterable的子类型，我们还接触了Iterable的子类型的三个常用的高阶方法foreach,map,reduce，但这些都只是Scala集合中的一些皮毛而已。下面我们来更加详细的了解Scala的不可变集合。

## List进阶

###   List结构

   你也许会发现在前面的List代码中使用tail和head分别得到了一个新的List和一个元素，如果你进一步对新得到的List使用tail,会得到越来越短的List，直到List(),如果你还没有尝试的话，可以看如下代码:

```scala
scala> val nums=List(3,2,1,4,5,6)
nums: List[Int] = List(3, 2, 1, 4, 5, 6)

scala> nums.tail
res28: List[Int] = List(2, 1, 4, 5, 6)

scala> val h=res28.tail
h: List[Int] = List(1, 4, 5, 6)

scala> res28.tail
res32: List[Int] = List(1, 4, 5, 6)

scala> res32.tail
res33: List[Int] = List(4, 5, 6)

scala> res33.tail
res34: List[Int] = List(5, 6)

scala> res34.tail
res35: List[Int] = List(6)

scala> res35.tail
res36: List[Int] = List()

scala> res36.tail
java.lang.UnsupportedOperationException: tail of empty list
  at scala.collection.immutable.Nil$.tail(List.scala:430)
  at scala.collection.immutable.Nil$.tail(List.scala:425)
  ... 29 elided

```

  事实上对于一个列表，我们可以把它分成为表头和表尾两部分，它们分别是head和tail成员方法得到的返回值。作为一个不可变得递归数据结构，列表中的每一项都有自己的表头和越来越短的表尾。这样我们知道List是一个链表结构，所以每次用size去获取List的大小时，它总会遍历一遍List.

  List中提供了一个isEmpty方法判断列表是不是已经为空,可以以这个方法判断是否停止对列表的遍历，比如：

  ```scala
scala> var i=nums
i: List[Int] = List(3, 2, 1, 4, 5, 6)
/*
用while循环来遍历nums
*/
scala> while(!i.isEmpty){println(i.head);i=i.tail}
3
2
1
4
5
6
  ```

### 列表结尾Nil

事实上，所有列表都有一个Nil作为终结点，Nil实际上是List[Nothing]的一个单例，正如你所知Nothing是所有其他Scala类型的一个不可实例化的子类型，List[Nothing]得以与所有其他类型的列表兼容，如果你创建的是一个单元素的新列表实际上是创建一个列表元素，而改元素指向Nil作为其表尾。 

```scala
/*
测试Nil的类型
*/
scala> List()
res1: List[Nothing] = List()

scala> res1==Nil
res2: Boolean = true


/*
测试列表结尾是否为Nil
*/
scala> List(1)
res4: List[Int] = List(1)

scala> res4.tail==Nil
res6: Boolean = true

```

### Cons操作符

  Nil是任何列表的表尾，那么如果你仔细思考一下会考虑是不是可以以空列表Nil为起始，不断增加元素，构造出一个长列表呢？事实上有一种叫做Cons(construct的简写)的操作符可以做到这一点。

  ::操作符是右结合的,意味着它是操作符右边实体调用。实际上，::只是列表的一个方法，它取一个值将它作为新列表的表头，而表尾指向调用::的列表。我们可以验证这一点:

```scala
scala> val l=List()
l: List[Nothing] = List()

/*
右结合方法调用::
*/
scala> "Sun"::l
res0: List[String] = List(Sun)

/*
点记发调用::
*/
scala> l.::("Intel")
res2: List[String] = List(Intel)

/*
连续的右结合构造一个新列表
*/
scala> "Jetbrains"::"Microsoft"::"Google"::"Intel"::"Sun"::l
res3: List[String] = List(Jetbrains, Microsoft, Google, Intel, Sun)
```

### 列表的基本操作

  scala的列表提供了许多常用的基本操作，比如追加元素，分解列表等等，但是你要知道，这一节我们讲的是不可变列表，它所谓的改变只是返回了一个新的列表而已。

| 方法名称      | 示例                                     | 描述                                       |
| --------- | -------------------------------------- | ---------------------------------------- |
| partition | List(1,2,3,4,5) partition (_<3)        | 根据一个true/false函数的结果，将元素分组为两个列表构成的一个元组    |
| reverse   | List(1,2,3,4,5).reverse                | 逆置列表                                     |
| slice     | List(1,2,3,4,5) slice (1,3)            | 返回列表的一部分，从第一个索引直到第二个索引(不包括第二个索引)，这里返回List(2,3),对应索引为1,2 |
| sortBy    | List("apple","to") sortBy (.size)      | 按给定函数返回的值对列表排序                           |
| sorted    | List("apple","to").sorted              | 按自然值对核心Scala类型的列表排序                      |
| splitAt   | List(1,2,3,4,5) splitAt 2              | 根据位于一个给定索引前面还是后面将元素分组为由两个列表构成的元组,这里返回(List(1, 2),List(3, 4, 5)) |
| take      | List(1,2,3,4,5) take 3                 | 抽取前n个元素                                  |
| zip       | List(1,2) zip List("A","B")            | 将两个列表合并为一个元组列表，每个元组包含两个列表中各个索引的相应元素，这里返回List((1,A), (2,B)) |
| ::        | "Intel"::"Sun"::Nil                    | 为列表最加单个元素，是右结合操作符                        |
| :::       | List(1,2) ::: List(2,3)                | 在列表前面追加另一个列表，是右结合操作符                     |
| ++        | List(1,2) ++ Set(2,3,3)                | 为列表追加另一个集合                               |
| ==        | List(1,2) == List(1,2)                 | 如果集合类型和内容都相同就返回true                      |
| distinct  | List(1, 2, 2, 3).distinct              | 返回不包含重复元素的列表版本                           |
| drop      | List('a','b','c','d') drop 2           | 从列表中除去前n个元素                              |
| filter    | List(23,8,14,21) filter (_>18)         | 从列表返回经过一个true/false的函数验证的元素              |
| flatten   | List(List(1, 2),List(3, 4, 5)).flatten | 将一个列表的列表转化为元素列表                          |

  在上表中同时使用了操作符和点记法两种方式，在没有操作参数时必须使用点记法比如list.sorted，其余情况可以根据个人喜好来使用。

  如果你详细考量这些方法，结合列表本质结构是单列表的特性，你很容易发现，对于::，drop,take这样的操作是在列表的前面部分完成的，不存在性能损失，如果是反向操作于列表末尾，那么可能需要一个完整的遍历，比如以上三个操作的反向操作，:+,dropRight和takeRight。

```scala
/*
看起来它们只做了与::，drop,take类似的很简单的操作，实际上它们需要遍历操作来复制整个或者某些元素
在列表比较大的时候这是相当大的开销
*/
scala> List(1,2,3):+4
res30: List[Int] = List(1, 2, 3, 4)

scala> List(1,2,3) dropRight 2
res31: List[Int] = List(1)

scala> List(1,2,3) takeRight 2
res32: List[Int] = List(2, 3) 
```

### 映射列表

集合论中，映射就是一个集合的各个元素和另一个结合中的各个元素之间创建一个关联。如此你大概知道了List的map方法是干什么的了。

| 操作名     | 示例                                       | 描述                              |
| ------- | ---------------------------------------- | ------------------------------- |
| collect | List(0,1,0) collect {case 1=>"ok"}       | 使用一个偏函数转化各个元素，保留可应用的元素          |
| flatMap | List("hello,world") flatMap (_.split(‘,’)) | 使用一个给定函数转化各个元素，将结果列表“扁平化”到这个列表中 |
| map     | List("milk","tea") map (_.toUpperCase)   | 使用给定函数转化各个元素                    |

```scala
cala> List(0,1,0) collect {case 1=>"ok"}
es35: List[String] = List(ok)

cala> List("hello,world") flatMap (_.split(','))
es36: List[String] = List(hello, world)

cala> List("milk","tea") map (_.toUpperCase)
es37: List[String] = List(MILK, TEA)
```

### 归约列表

如何把列表收缩为单个值？这称之为归约(reducing)操作。

**数学归约操作**

| 操作名     | 示例                  | 描述       |
| ------- | ------------------- | -------- |
| max     | List(1,2,3).max     | 取最大      |
| min     | List(1,2,3).min     | 取最小      |
| product | List(1,2,3).product | 将列表中的数相乘 |
| sum     | List(1,2,3).sum     | 求和       |

**布尔归约操作**

| 操作名        | 示例                             | 描述                        |
| ---------- | ------------------------------ | ------------------------- |
| contains   | List(34,28,18) contains 29     | 检查列表中是否包含这个元素             |
| endsWith   | List(0,4,3) endsWith List(4,3) | 检查列表是否以一个给定的列表结尾          |
| exists     | List(0,4,3) exists (_<4)       | 检查一个谓词是否至少对列表中的一个元素返回true |
| forall     | List(0,4,3) forall (_<4)       | 检查一个谓词是否至少对列表中的所有元素返回true |
| startsWith | List(0,4,3) startsWith List(0) | 测试列表是否以给定的一个列表开头          |

  如果你觉得以上的归约方法还不足以满足你的编程需求的话我们可以尝试使用scala列表内提供的通用归约操作。	传入函数字面量来构造出你需要的归约方法

**通用归约操作**

| 操作名         | 示例                             | 描述                              |
| ----------- | ------------------------------ | ------------------------------- |
| fold        | List(4,5,6).fold(0)(\_+_)      | 给定一个起始值和一个归约函数来归约列表             |
| foldLeft    | List(4,5,6).foldLeft(0)(\_+_)  | 给定一个起始值和一个归约函数来从左到右归约列表         |
| foldRight   | List(4,5,6).foldright(0)(\_+_) | 给定一个起始值和一个归约函数来从右到左归约列表         |
| reduce      | List(4,5,6).reduce(\_+_)       | 给定一个归约函数，从列表的第一个元素开始归约列表        |
| reduceLeft  | List(4,5,6).reduceLeft(\_+_)   | 给定一个归约函数，从列表的第一个元素开始从左到右归约列表    |
| reduceRight | List(4,5,6).reduceRight(\_+_)  | 给定一个归约函数，从列表的第一个元素开始从右到左归约列表    |
| scan        | List(4,5,6).scan(0)(\_+_)      | 给定一个起始值和一个归约函数，返回各个累加值得一个列表     |
| scanLeft    | List(4,5,6).scanLeft(0)(\_+_)  | 给定一个起始值和一个归约函数，从左到右返回各个累加值得一个列表 |
| scanRight   | List(4,5,6).scanRight(\_+_)    | 给定一个起始值和一个归约函数，从右到左返回各个累加值得一个列表 |

  这三种不同的归约操作之间的差别其实不是多么的重要，但是同一个归约操作它的无方向和有方向版本的差别就很重要了。

  fold,reduce,scan的无方向版本只接受返回与列表元素类型相同的函数，但是有方向版本则不同，比如：

```scala
/*
以foldLeft实现一个forall布尔操作，如果是fold就做不到，因为它只能接受
*/
scala> List(4,5,6).foldLeft(false)((b:Boolean,x:Int)=>x>5)
res69: Boolean = true

scala> List(4,5,6).foldLeft(false)((b:Boolean,x:Int)=>x>6)
res70: Boolean = false

```

通常指定迭代顺序从左往右能实现开销小一点的操作，因为从左往右的实现需要的遍历更少。

### 转换集合

 列表是最常用的集合，但却不是唯一的集合，有时候我们需要把它转化为scala的其他类型的集合或者字符串,或者把其他的集合类型转化为列表，幸运的是scala集合内置了这些方法，以至于我们可以很容易的实现这一点

| 操作名      | 示例                                | 描述                                       |
| -------- | --------------------------------- | ---------------------------------------- |
| mkString | List(4,5,6).mkString(",")         | 使用给定分隔符将一个集合呈现为一个String,这里变为"4,5,6"      |
| toBuffer | List(4,5,6).toBuffer              | 将一个不可变集合转换为一个可变集合                        |
| toList   | Map(4->“apple”,5->"scala").toList | 将一个集合转化为一个List                           |
| toMap    | Set(4->“apple”,5->"scala").toMap  | 将一个2元元组（长度为2）的集合转化为一个Map                 |
| toSet    | List(4,5,6).toSet                 | 将一个集合转化为一个Set                            |
| toString | List(4,5,6).toString              | 将一个集合呈现为一个String,包括集合的类型,这里变为"List(4, 5, 6)" |

对于转换集合，你可能还有一个地方很想知道，那就是作为一个在JVM上编译和运行的语言，Scala经常需要与JDK和其他的Java库进行交互。那么我们如何进行Java和Scala中的集合转化呢？答案是：将JavaConverters和他的方法增加到当前命名空间！如下代码:

```scala
/*
引入JavaConverters及其方法
*/
scala> import collection.JavaConverters._
import collection.JavaConverters._

/*
asJava方法将scala.collection.immutable.List转化为Java类型
*/
scala> List(1,2,3,4).asJava
res80: java.util.List[Int] = [1, 2, 3, 4]

/*
asScala方法将java类型转化为scala类型
*/
scala> new java.util.ArrayList(5).asScala
res81: scala.collection.mutable.Buffer[Nothing] = Buffer()
```

### 集合的模式匹配

  模式匹配是Scala语言的一个核心特性，而不只是标准集合库中的一个操作，使用这个特性可以简化逻辑，算是一个很棒的语法糖，如果你是使用java的话实现类似的逻辑需要用大量的代码，如下我给出了几个对几个模式匹配，相信你可以一眼就看出其中对比其他语言带来的好处。

```scala
/*
还记得模式匹配的占位符么？它还可以这样使用
*/
scala> val code=('h',204,true) match {
     | case (_,_,false) => 501
     | case ('c',_,true) => 301
     | case ('h',_,true) => 201
     | case _ => {println("error!");404
     | }
     | }
code: Int = 201

scala> val code=('x',204,true) match {
     | case (_,_,false) => 501
     | case ('c',_,true) => 301
     | case ('h',_,true) => 201
     | case _ => {println("error!");404
     | }
     | }
error!
code: Int = 404

/*
列表的值绑定
*/
scala> List('x',"world","kk") match {
     | case List('x',x,_) => s"second $x"
     | case List('f',_,x) => s"first $x"
     | case _=>"error"}
res85: String = second world
```

