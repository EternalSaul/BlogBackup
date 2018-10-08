---
title: 'Learning Scala(6):一元集合'
date: 2017-07-15 11:06:19
toc: true
tags:
 - scala
 - 学习笔记
---

 **一元(Monadic)集合支持Iterable中的变换操作，但是包含的元素不能多于1个。**

## Option

### Some和None

  Option是一个一元集合类型，标识一个值的存在或者不存在。你可以看到很多编程语言近年来都增加了类似于Option的这样的一种特性，比如Java 8中的Optional和Swift中的Option,因为很多开发人员认为Option可以安全的代替null值，减少空指针异常发生的可能性，还有一些开发人员把它看作是构建操作链的安全方法，确保链中只含有有效的值。

  Option类型本身没有实现，它依赖于两个子类型提供具体的实现：Some和None，其中Some是一个类型参数化的单元素集合，而None是一个空集合。

```scala
//创建一个值为kkk的字符串
scala> val s="kkk"
s: String = kkk

//调用Option，构造出Some类型
scala> val a=Option(s)
a: Option[String] = Some(kkk)

//创建一个空字符串
scala> val n:String=null
n: String = null

/*
调用Option，构造出None类型
以此你可以通过这样的新方式来判断一个值或者变量是不是null
*/
scala> val b=Option(n)
b: Option[String] = None

/*
当然如果不是在REPL下而是在IDE下编写Scala程序，你并不会直接看到toString的输出，你也就无法判定具体的类型到底是None还是Some，这样的话我们需要借助两个操作来判断：isDefined和isEmpty
*/
scala> a.isDefined
res40: Boolean = true

scala> a.isEmpty
res41: Boolean = false

scala> b.isEmpty
res42: Boolean = true

scala> b.isDefined
res43: Boolean = false
```

<!--more-->

### 代替null的Option

用None来取代null,比如下列函数求某个数的平方根,在接受负数时返回None，这比之返回一个null要安全许多，返回一个null则需要写一大段模板代码来避免空指针异常。

```scala
scala> def getRoot(x:Double):Option[Double]={
     | if(x<0) None else Option(Math.sqrt(x))}
getRoot: (x: Double)Option[Double]

scala> getRoot(1)
res44: Option[Double] = Some(1.0)

scala> getRoot(-1)
res45: Option[Double] = None

/*
使用headOption来安全的处理空集合
*/
scala> ls.headOption
res50: Option[Int] = Some(1)

scala> val ls=List(1,2,3,45,7,6)
ls: List[Int] = List(1, 2, 3, 45, 7, 6)

scala> ls.headOption
res51: Option[Int] = Some(1)

scala> val lss=List()
lss: List[Nothing] = List()

scala> lss.headOption
res52: Option[Nothing] = None

/*
find可以找到集合中与给定谓词匹配的第一个项，它也安全的返回一个Option
*/

scala> ls.find(_>40)
res55: Option[Int] = Some(45)

scala> ls.find(_>46)
res56: Option[Int] = None

/*
在一般的命令式编程中，返回一个Option带来的好处好像不够明显，我们以函数式编程写一段你就会发现这种好处非常明显，如下面这种链式编程中，返回一个None的优势是巨大的，如果是一个null估计在执行第二个链式会报出异常，从而导致程序出现某些不太和谐的结果
在第二个链中filter之后，因为找不到ester开头的字符串，所以返回了一个None值，紧接着None调用map还是None，一直到末尾最终返回了None
*/
scala> val ss=List("eternal","eternity","eternally")
ss: List[String] = List(eternal, eternity, eternally)

scala> ss find (_.endsWith("ity")) filter (_.startsWith("eter")) map (_.toUpperCase) map (_.size)
res65: Option[Int] = Some(8)

scala> ss find (_.endsWith("ity")) filter (_.startsWith("ester")) map (_.toUpperCase) map (_.size)
res66: Option[Int] = None
```

### 从Option抽取值

  Option集合还提供了一些操作来存储和变换可能存在也可能不存在的值，当然它也提供了一些操作来抽取可能存在的值。

  Option集合有一个不安全的抽取操作：get，对于Some而言可以成功的取得其中包含的值，但是对于None而言则会抛出一个java.util.NoSuchElementException异常

```scala
scala> None.get
java.util.NoSuchElementException: None.get
  at scala.None$.get(Option.scala:349)
  ... 29 elided

//定义一个随机函数，作为下面使用
scala> def nextOption = if(util.Random.nextInt >0) Some(1) else None
nextOption: Option[Int]
```

所以对于get操作应该尽量得避免使用，它破坏了Option的初衷，Option提供了几个安全的抽取操作,如:

| 操作名       | 示例                                       | 描述                                       |
| --------- | ---------------------------------------- | ---------------------------------------- |
| fold      | nextOption.fold(-1)(x=>x)                | 对于Some，从给定的函数返回值（根据嵌入的值），或者返回起始值。也可以foldLeft、foldRight和reduceXXX方法将一个Option规约为它的内嵌值或者一个计算值 |
| getOrElse | nextOption getOrElse 5 or nextOption getOrElse{println("error!");-1} | 为Some返回值，或者为None返回传名参数的结果                |
| orElse    | nextOption orElse nextOption             | 并不真正的抽取值，而是试图为None填入一个值。如果非空，则返回这个Option，否则从给定的传名参数返回一个Option |
| 匹配表达式     | nextOption match {case Some(x) => x; case None=> -1} | 使用一个匹配表达式处理值（如果存在）。Some(x)表达式将其数据抽取到指定的值x，这可能会作为匹配表达式的返回值，或在将来的变换中重用 |

## Try集合

  Scala支持try/catch块，但是使用Scala编程更加推荐使用一种更安全，更具有表述性的一元方法来处理错误，这就是util.Try类型，它同样没有具体的实现，而是提供了两个已经实现的子类型Success和Failure，从名字你可以看出一个代表了成功，而另一个则包含所抛出的Exception异常。

util.Try集合将错误处理转变为集合管理，比如：

```scala
/*
当nextOption返回None时调用get会抛出java.util.NoSuchElementException
*/
scala> util.Try(nextOption.get)
res106: scala.util.Try[Int] = Success(1)

scala> util.Try(nextOption.get)
res107: scala.util.Try[Int] = Failure(java.util.NoSuchElementException: None.get)
```

对于异常处理，没有同用的方法但是可以参考下面几种，在适合的时候给予选择

```scala
scala> def nextError=util.Try{nextOption.get}
nextError: scala.util.Try[Int]

/*
将当前返回值映射到一个新的内嵌返回值，或者一个异常
*/
scala> nextError flatMap {_=>nextError}
res124: scala.util.Try[Int] = Failure(java.util.NoSuchElementException: None.get)

scala> nextError flatMap {_=>nextError}
res125: scala.util.Try[Int] = Success(1)

/*
返回Success中的内嵌值，如果发生异常则返回给定的默认值
*/
scala> nextError getOrElse 0
res1: Int = 1

scala> nextError getOrElse 0
res2: Int = 0

/*
转化为Option，对于异常返回None
*/
scala> nextError.toOption
res5: Option[Int] = None

scala> nextError.toOption
res6: Option[Int] = Some(1)

/*
运用模式匹配
*/
scala> nextError match {case util.Success(x)=>x
     | case _ => -1 }
res8: Int = -1

scala> nextError match {case util.Success(x)=>x
     | case _ => -1 }
res9: Int = 1
```

## Future集合

  concurrent.Future集合，会发起一个后台任务，它也表示一个可能的值，但是作为一个后台任务的返回值你可能不是立即能得到它，如果你是java程序员，你应该很熟悉Future这个名字，java中它用于获取一个有返回值的线程任务的返回回值。

  scala中future的使用很容易，比如

```scala
/*
在创建future前我们必须先指定明确的上下文，不然就会看到如下的错误提示
*/
scala> val f = concurrent.Future{println("ss")}
<console>:11: error: Cannot find an implicit ExecutionContext. You might pass
an (implicit ec: ExecutionContext) parameter to your method
or import scala.concurrent.ExecutionContext.Implicits.global.
       val f = concurrent.Future{println("ss")}

/*
现在我们事先把上下文设置为global
*/
scala> import concurrent.ExecutionContext.Implicits.global
import concurrent.ExecutionContext.Implicits.global

scala> val f = concurrent.Future{println("ss")}
ss
f: scala.concurrent.Future[Unit] = Future(<not completed>)

/*
可以用Thread.sleep让后台线程睡眠，而不阻塞主线程
*/
scala> val f = concurrent.Future{Thread.sleep(2000);println("hi");}
f: scala.concurrent.Future[Unit] = Future(<not completed>)

//主线程输出
scala> println("waiting")
waiting

//后台线程输出
scala> hi
```

### 异步Future

  Future可以进行异步处理也可以进行同步处理，不过更有趣的是，Future是一个一元集合，我们反复讲了scala中的集合都可以做某类事情，Future也一样，将一个Future串链一个函数或另一个Future，当前的Future执行完后，就可以把结果传递给新的函数或future.

  以上面描述的那种方式处理的Future最终返回一个util.Try，如果操作序列中有一环失败了（抛出了一个异常或者不再有值），那么就抛出一个异常，否则返回一个最终结果。

  对于Thread.sleep实际上效率不怎么高，因为你并不知道这个任务实际上要执行多少时间，我们可以以回调函数来代替它,Scala提供了很多回调函数,比如：

```scala
def nextFtr(i:Int =0) =Future{
def rand(x:Int)=util.Random.nextInt(x)
Thread.sleep(rand(1200))
if(rand(3)>0)(i+1) else throw new Exception
}

/*
将第二个future串链到第一个future,并返回第一个future，如果第一个future不成则调用第二个future
*/
scala> val l=nextFtr(1) fallbackTo nextFtr(2)
l: scala.concurrent.Future[Int] = Future(<not completed>)

scala> l
res7: scala.concurrent.Future[Int] = Future(Success(2))


```

| 操作名                  | 示例                                       | 描述                                       |
| -------------------- | ---------------------------------------- | ---------------------------------------- |
| fallbackTo           | nextFtr(1) fallbackTo nextFtr(2)         | 将第二个future串链到第一个future,并返回第一个future，如果第一个future不成功，则调用第二个future |
| flatMap              | nextFtr(1) flatMap nextFtr               | 将第二个future串链到第一个future,并返回第一个future，如果第一个future成功，则用它的返回值调用第二个future |
| map                  | nextFtr(1) map (_*2)                     | 将给定的函数串链到Future,返回一个新的future。如果这个future成功，其返回值将用来调用这个函数 |
| onComplete           | nextFtr() onComplete{_ getOrElse 0}      | future任务完成后，将调用一个util.Try调用指定函数其中包含一个值（如果成功）或者一个异常（如果失败） |
| onFailure（2.12.0后废止） | future任务完成后，将调用一个util.Try调用指定函数其中包含一个值（如果成功）或者一个异常（如果失败） | future任务抛出一个异常，将使用该异常来调用给定函数             |
| onSuccess（2.12.0后废止） | nextFtr() onSuccess{case x=>s"Got $x"}   | future任务成功完成，将使用返回值调用函数                  |
| Future.sequence      | concurrent.Future sequence List(nextFtr(1),nextFtr(2),nextFtr(3)) | 并发地运行给定序列中的future,返回一个新的future如果序列中的所有future都成功，则返回其返回值的一个列表，否则返回future中出现的第一个异常 |

### 同步Future

有时你不得不阻塞线程以等待某个结果或者时刻，比如生产者和消费者问题，阻塞当前的线程并等待另一个线程完成，需要使用concurrent.Await.result()，它取后台线程的和一个最大的等待时间作为参数。如果一个future不能按给定的最大时间之内完成则抛出一个java.util.concurrent.TimeoutException。

```scala
/*
引入命名空间
*/
scala> import concurrent.duration._
import concurrent.duration._

/*
创建一个持续时间(Duration)参数
*/
scala> val maxTime=Duration(10,SECONDS)
maxTime: scala.concurrent.duration.FiniteDuration = 10 seconds

/*
开始任务
*/
scala> val a=concurrent.Await.result(nextFtr(5),maxTime)
a: Int = 6

/*
一个超时的任务
*/

scala> val maxTime=Duration(1,SECONDS)
maxTime: scala.concurrent.duration.FiniteDuration = 1 second

scala> def fut()=concurrent.Future{Thread.sleep(5000); println("I am late");1}
fut: ()scala.concurrent.Future[Int]

/*
抛出异常
*/
scala> val a=concurrent.Await.result(fut,maxTime)
java.util.concurrent.TimeoutException: Futures timed out after [1 second]
  at scala.concurrent.impl.Promise$DefaultPromise.ready(Promise.scala:255)
  at scala.concurrent.impl.Promise$DefaultPromise.result(Promise.scala:259)
  at scala.concurrent.Await$.$anonfun$result$1(package.scala:215)
  at scala.concurrent.BlockContext$DefaultBlockContext$.blockOn(BlockContext.scala:53)
  at scala.concurrent.Await$.result(package.scala:142)
  ... 29 elided

/*
只是主线程异常，后台线程并没有被中断，依然输出了 I am late
*/
scala> I am late


```

