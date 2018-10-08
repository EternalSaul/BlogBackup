---
title: 'Learning Scala (2):函数的部分应用与柯里化'
date: 2017-07-06 16:08:28
toc: true
tags:
 - scala
 - 学习笔记
---



## 占位符语法

  占位符语法(Placeholder syntax) 是函数字面量的一种**缩写形式**，将命名参数替换为通配符(_)。以下情况可以使用：

1. 函数显示类型在字面量以外指定
2. 参数最多只使用一次

比如：

```scala
scala> val half:Int=>Double=_/2.0
half: Int => Double = $$Lambda$1121/2012831257@4607cbe2

scala> half(3)
res13: Double = 1.5
```
<!-- more -->

另外可以通过占位符来简化函数字面量题，如下：

```scala
scala> def safeStringOp(s:String,f:String=>String)={
     | if(s!=null) f(s) else s
     | }
safeStringOp: (s: String, f: String => String)String

scala> safeStringOp(null,_.reverse)
res21: String = null

scala> safeStringOp("same",_.reverse)
res22: String = emas
```

本来以一般的函数字面量应该传入s=>s.reverse,而此时_.reverse效果相同。

你可能会问，为什么编译器能够识别reverse函数？如果你仔细阅读了safeStringOp函数的定义你就不难发现,safeStringOp的第二个类型是String=>String，正是因为如此，编译器才知道传入的reverse是做什么操作的。

Scala编译器会0把通配符_看做是一个输入参数。

那么？是否可以有多个通配符_，对应多个参数呢？答案是:yes

```scala
/*
定义一个高阶函数，它接受两个整形，和一个(Int,Int)=>Int函数类型
*/
scala> def unite(x:Int,y:Int,f:(Int,Int)=>Int)={
     | f(x,y)}
unite: (x: Int, y: Int, f: (Int, Int) => Int)Int

/*
以_+_占位符的形式来代替普通的函数字面量,相当于(x,y)=>x+y
*/
scala> unite(1,2,_+_)
res30: Int = 3


/*
三元素函数
*/
scala> def uniteplus(x:Int,y:Int,z:Int,f:(Int,Int,Int)=>Int)={
     | f(x,y,z)}
uniteplus: (x: Int, y: Int, z: Int, f: (Int, Int, Int) => Int)Int

/*
以占位符取代
*/
scala> uniteplus(1,2,3,_*_*_)
res45: Int = 6

/*
在占位符语法定义的函数中使用类型参数
*/
scala> def tripleOp[A,B](a:A,b:A,c:A,f:(A,A,A)=>B)=f(a,b,c)
tripleOp: [A, B](a: A, b: A, c: A, f: (A, A, A) => B)B

scala> tripleOp[Int,Int](1,2,3,_+_+_)
res1: Int = 6

scala> tripleOp[Int,Boolean](1,2,3,_+_>_)
res3: Boolean = false
```

## 部分应用函数

  我们在调用一个函数时，如果没有默认参数值，通常要指定所以的参数，如果你想重用一个函数调用，而且希望保留一些参数不想再次输入应该怎么做呢？

  事实上这个过程叫做部分应用(partially apply)一个函数，我们使用通配符来替代那些还没明确传入值的参数，并指定他们的类型。比如：

```scala
/*
该例中我们构造了一个计算某数倍数的函数
*/
scala> def timesCalculate(a:Int,times:Int)=a*times
timesCalculate: (a: Int, times: Int)Int

/*
我们部分应用该函数，给它的第一个参数赋值为3，该语句将返回一个新的Int=>Int型函数，原函数的第一个参数被固定了
*/
scala> val tc=timesCalculate(3,_:Int)
tc: Int => Int = $$Lambda$1094/1057262726@255d9277

/*
传入第二个参数，构成计算3*5得到了正确的结果
*/
scala> tc(5)
res7: Int = 15
```

## 多参数列表函数

  部分函数应用足够简洁么？我们是否有手段可以把它变得更简洁呢？

  首先我们要接触一个新的概念，即有多个参数列表的函数。

   比如在上一个例子中计算某数倍数的函数可以写成有两个参数列表的形式，它和上面的只有一个参数列表的函数具有不同的类型，它可以认为是多个函数的一个链。

   ```scala
def timesCalculate(a:Int)(times:Int)=a*times

/*
可以看到新的 timesCalculate类型并不是(Int,Int)=>Int
而是Int => (Int => Int)
导致REPL报错
*/
scala> val tc:(Int,Int)=>Int=timesCalculate
<console>:12: error: type mismatch;
 found   : Int => (Int => Int)
 required: (Int, Int) => Int
       val tc:(Int,Int)=>Int=timesCalculate
                             ^
/*
改变类型，赋值成功
*/
scala> val tc:Int=>Int=>Int=timesCalculate
tc: Int => (Int => Int) = $$Lambda$1153/106193777@2b458cd6
   ```



## 函数柯里化

了解了多参数列表函数，我们们现在介绍一种更加简洁的函数部分应用方式，即函数柯里化(currying)               

 [it was possible to transform a function with multiple arguments into a chain of single-argument functions instead. This transformation is the process now known as currying.](https://en.wikipedia.org/wiki/Currying#cite_note-8)

 可以把一个多参数函数转化为以一个单参数函数的函数链代替，这个转化过程即现在所称的柯里化

  柯里化和部分应用函数技术是近似的，但它们却不是一样的，前面所提到的部分应用函数技术并没有把一个多参数函数转化为一个单参数函数的函数链形式。

  比如对

`def timesCalculate(a:Int,times:Int)=a*times` 

类型为:**(Int,Int)=> Int**


柯里化后：

`def timesCalculate(a:Int)(times:Int)=a*times`

类型为**:Int => (Int => Int)**

你可以发现在scala中对一个多参数函数柯里化，即是把它转化为单参数的多参数列表（每个参数列表只有一个参数）的形式，我们在上面讨论过，**多参数列表可以看做是多个函数的一个链，如果每个参数列表只有一个参数，那么可看做是单参数函数的函数链形式**

```scala
/*
调用柯里化后的新的多参数链表函数timesCalculate
最后一个参数可以不指定类型
*/
scala> val tc=timesCalculate(3)_
tc: Int => Int = $$Lambda$1155/2093625852@619c3546

/*
尝试用通配符来取代第一个参数，无法通过编译
因为编译器无法猜测其类型
*/
scala> val tc=timesCalculate(_)(3)
<console>:12: error: missing parameter type for expanded function ((x$1: <error>) => timesCalculate(x$1)(3))
       val tc=timesCalculate(_)(3)
                             ^

/*
为占位符添加类型，编译成功
*/
scala> val tc=timesCalculate(_:Int)(3)
tc: Int => Int = $$Lambda$1160/2146473561@517fbf62
```



