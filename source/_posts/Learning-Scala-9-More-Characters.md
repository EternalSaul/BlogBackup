---
title: 'Learning Scala (9):更多Scala特性'
date: 2017-07-18 08:06:00
toc: true
tags:
 - scala
 - 学习笔记
---

  Scala利用常规类构建高层元组和函数字面量，这一篇文章主要是简单的介绍一下以常规类作为基础的元组和函数。

## TupleX和FunctionX

  创建一个元组和函数字面量的语法非常快捷，而且直观，它不需要new关键字，好像就是与生俱来的语法优势一样，实际上元组和函数字面量的具体实现只是普通的类。

```scala
/*
Scala的语法简洁而直观
*/

scala> val abc=("a","b","c")
abc: (String, String, String) = (a,b,c)

scala> val f=(x:Int)=>x*2
f: Int => Int = $$Lambda$1022/627707285@2f3928ac
```

<!--more-->
### 元组的实现

  元组实际上是一个TupleX[Y] case类的实例，X是一个从1到22的整数，代表了元组里的元素的个数。比如Tuple1[A],Tuple2[A,B],以此类推。

```scala
/*
可以看到Tuple3(1,2,3)和(1,2,3)构建了相同的结果，你现在可以知道元组的表属性语法就是case类的快捷方式
*/
scala> val k=(1,2,3)
k: (Int, Int, Int) = (1,2,3)

scala> val kt=Tuple3(1,2,3)
kt: (Int, Int, Int) = (1,2,3)
```

   实际上你也可以自己来写一个元组的case类，TupleX[Y]分别用相同的数扩展了 trait类:ProductX,这些trait提供了一些有用的操作，比如productElement可以提供一种非安全的方式来访问元组的第n个元素。

### 函数值的实现

  函数值的实现类似，但不是用case类来实现的，而是表现为一个trait类FunctionX[Y]的实例，函数根据函数元数从0到22编号，实际类型参数是X+1个(因为存在返回值)，函数的逻辑即在apply方法里实现，所以你可以知道写一个函数字面量的时候，实际上是把它转化为了一个扩展FunctionX的改写了apply方法体的一个新类。

  **通过这种将函数强制转化为一个类方法的机制，Scala的函数值就与jvm兼容了**

```scala
scala> val f=(k:String)=>s"hello $k"
f: String => String = $$Lambda$1063/361948480@3e2772a9

/*
类型参数中，第一个String是参数类型，第二个是返回值
*/
scala> val ff=new Function1[String,String]{
     |     def apply(k:String)=s"hello $k"
     | }
ff: String => String = <function1>

/*
可以看到实际上它们的都是类
*/
scala> f.apply("tom")
res3: String = hello tom

scala> ff.apply("tom")
res4: String = hello tom

```

Function1 trait包含两个特殊方法，而Function0或其他任何FunctionX中都没有这两个方法，利用这两个方法可以将多个Function1实例结合为一个新的Function1，调用的按顺序执行所有函数。现在是第一个函数的返回值必须要与第二个的输入类型相同。这两个方法分别是andThen和compose。andThen由两个函数值创建一个新函数值，它们组合的顺序是从左往右，compose则相反。

```scala
scala> val p=(i:Int)=>i*2
p: Int => Int = $$Lambda$1148/1941163569@4687366b

scala> val q=(i:Int)=>i+1
q: Int => Int = $$Lambda$1149/881578083@3d73cd78

scala> (p andThen q)(1)
res5: Int = 3

scala> (p compose q)(1)
res6: Int = 4

scala> (p compose q andThen q)(1)
res7: Int = 5
```

## 隐含参数

  我们知道scala的函数可以部分应用，但是如果调用一个函数却不指定所有参数的情况下，这个函数能不能执行呢？你肯定会想到默认参数，这是一种办法，还有一种方法也行，而且它不要求你知道函数缺少参数的正确值。这种方法就是使用隐含参数(implicit parameter)，这种方式是在调用者的命名空间提供默认值，函数可以定义一个隐含参数。

  用implicit关键字可以标识变量或函数定义为隐含的，可以用隐含值或变量填充一个函数调用中的隐含调用。需要注意的是：**implicit关键字是对一个参数列表而言的所有参数都是implicit，而非只是标明一个参数，它只能加载参数列表开始的时候**，所以一般用隐含参数另列一个参数列表

```scala
/*
implict的作用范围，是整个参数列表，它不能只标识一个参数
*/
scala> def hello(name:String,implicit hello:String)=println(hello+name)
<console>:1: error: identifier expected but 'implicit' found.
def hello(name:String,implicit hello:String)=println(hello+name)
                      ^
/*
把implict参数另列出来
*/
scala> def hello(name:String)(implicit hello:String)=println(hello+name)
hello: (name: String)(implicit hello: String)Unit

/*
此时命名空间里找不到implicit值，不能直接调用函数
*/
scala> hello("Tom")
<console>:13: error: could not find implicit value for parameter hello: String
       hello("Tom")
            ^
/*
同样可以传值，如果你想的话
*/
scala> hello("Tom")("hi ")
hi Tom

/*
在命名空间中定义一个implicit值
*/
scala> implicit val hes="halo"
hes: String = halo

/*
隐含参数调用成功
*/
scala> hello("Tom")
haloTom

/*
更底层的命名空间
*/
scala> case class Hey(){
     | implicit val hes="hello "
     | def print=hello("Jack")
     | }
defined class Hey

scala> Hey()
res15: Hey = Hey()

/*
输出了Hey case空间中对应的隐含值
*/
scala> res15.print
hello Jack
```

  隐含参数有可能会让你的小伙伴无法看懂你在写什么，hello("Tom")和hello("Tom")("hello ")可能会让人以为调用的是两个不同的函数，所以什么时候用需要具体把握。

## 隐含类

  类的隐含转化是Scala的另一个隐含特性，隐含类(implicit class)是一种**类类型**，它可以与另一个类自动转换。

  如果Scala编译器发现了一个实例要访问一个未知的方法或者字段，它就会使用隐含转换。它会检查当前命名空间的隐含转换，这个过程由两部完成：

1. 取该实例作为一个参数
2. 实现缺少的字段或方法

有了这种方式，我们就可以为类添加方法而不改变源代码，比如：

```scala
/*
创建一个MathUtils包含了一个隐含类
*/
scala> object MathUtils{
     |       implicit class Calculate(val x:Int){
     |           def cube=x*x*x
     |  }
     |  }
defined object MathUtils

/*
导入命名空间
*/
scala> import MathUtils._
import MathUtils._

/*
整形直接调用了cube方法，因为隐含类
*/
scala> 3.cube
res22: Int = 27
```

定义和使用隐含类存在了如下限制：

1. 隐含类必须在另一个对象、类或者trait中定义。
2. 隐含类必须去一个非隐含类的参数
3. 隐含类类名不能与当前命名空间里的另一个对象、类或者trait冲突。所以case类一定不能成为隐含类因为它有伴生对象

 scala.Predef对象中的成员会自动的增加到命名空间，它提供了很多类型特性，其中也有隐含转化，来支持Scala中的种种表述性语法，比如构造元组的->操作符,如下面提到的这个简化版本

```scala
implicit class ArrowAssoc[A](x:A){
  def->[B](y:B)=Tuple2(x,y)
}
```

