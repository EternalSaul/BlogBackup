---
title: 'Functional Programming (2):函数式错误处理'
date: 2017-07-28 16:09:55
toc: true
tags:
 - FP
 - scala
---

  抛出异常是有副作用的，事实上我们可以用普通的值来表现成功和异常，以这种方式返回错误，符合引用透明的特点。

  我之前写了一篇博文介绍Scala中的[一元集合](/2017/07/15/Learning-Scala-6-Monadic/)，我在其中有介绍过Option，Try，两种一元集合，它们都是sealed类，且都有两个已经实现子类，来分别代表结果的两种对立情况。并且我也介绍了如何用Try来进行错误的处理。在这篇文章，我将介绍函数式编程为什么要这样做，同时更详细的说明如何这样做。

<!--more-->

## 一般的异常处理

  我想try/catch语法你已经不止一次的接触过了，当然伴随他而生的还有讨厌的null检测，scala中也可以用try/catch语法，结合模式匹配和Option似乎比java里更完美。如：

```scala
scala> def getValue(s:String){
     | val o=Option(s);
     | try{println(o.get)}catch{case e:Exception => println("error!!")}
     | }
getValue: (s: String)Unit

scala> getValue(null)
error!!

scala> getValue("hello")
hello
```

  然而抛出异常的语句throw new Exception("error!!")是有副作用的，它依赖于上下文，不符合透明引用。比如对如下代码：

```scala
/*
引用透明（Referential transparency）是指一个表达式可以被它的对应值（corresponding value）取代而不改变程序的行为.
*/
scala>  def sayHello(name:String):String={
     |      val hi:String=throw new Exception("Mr Trump is so busy")
     |      try{
     |            name+hi;
     |      }catch{
     |          case e:Exception => name;
     |      }
     |  }
sayHello: (name: String)String

scala> sayHello("Tom")
java.lang.Exception: Mr Trump is so busy
  at .sayHello(<console>:12)
  ... 29 elided

/*
对于表达式hi:String=throw new Exception("Mr Trump is so busy"),我们把hi设置为懒值，即我们把该表达式嵌入了一个表达式，但是可以看到出现了不同的结果，即改变程序的行为.所以异常抛出不是引用透明的。
*/
scala>  def sayHello(name:String):String={
     |      lazy val hi:String=throw new Exception("Mr Trump is so busy")
     |      try{
     |            name+hi;
     |      }catch{
     |          case e:Exception => name;
     |      }
     |  }
sayHello: (name: String)String

scala> sayHello("Tom")
res2: String = Tom
```

可以看到，抛出异常的语句并不是引用透明的语句，于是用它在函数中处理异常，会使得该函数不符合纯函数定义，不符合函数式编程特点。

## 用Option来实现对异常的包装

  在[一元集合](/2017/07/15/Learning-Scala-6-Monadic/)中，我介绍了util.Try来处理异常，它实现了对异常的包装，返回一个Success或者Failure安全的表达式中是否发生了异常，但类似的，我们是否可以用Option来自己实现一个异常处理？

  这里我们又要用到我的另一篇文章中提到的[传名参数](/2017/07/12/Learning-Scala-2-By-name-parameter/)，传名参数接受一个返回值类型固定的表达式，如果配合类型参数，那么我们可以将它改变为接受一个任意表达式，比如：

```scala
/*
以传名参数和类型参数结合构造函数Try使其接受任意表达式，并包裹在Some中执行，如果抛出任何异常则返回None
*/
scala> def Try[T](f: =>T)={
     | try Some(f) catch{case e:Exception=>None}
     | }
Try: [T](f: => T)Option[T]

scala> Try(Option(null).get)
res33: Option[Null] = None

scala> Try(Option(1).get)
res34: Option[Int] = Some(1)

scala> Try(throw new Exception("123"))
res35: Option[Nothing] = None
```

## Either

用普通的值来包装失败和异常，除了Option和util.Try外还有一种trait即Either，我们在上个例子中用Option实现了对异常的包装，但是你可以看到的是，返回一个None并没有提供其他的任何错误详细信息，而None则只能代表空集合表示没有可用值，而Option是一个sealed类,此时如果不用util.Try我们应该怎么以Either来包装异常呢？

**sealed trait Either[+E,+A]**

**case class Left\[+E](value:E) extends Either[E,Nothing]**

**case class Right\[+A](value:A) extends Either[Nothing,A]**

一元集合Either同样有两种子类型，其中Left表示失败，Right表示成功，与Option不同的是，Option两个子类都有一个值，现在我们可以重新实现Try，它能够显示出异常的具体原因。

```scala
/*
以Either重新实现的Try函数
*/
scala> def Try[T](f: =>T)={
     | try Right(f) catch{case e:Exception=>Left(e)}
     | }
Try: [T](f: => T)scala.util.Either[Exception,T]

scala> Try(Option(null).get)
res60: scala.util.Either[Exception,Null] = Left(java.util.NoSuchElementException: None.get)
```

