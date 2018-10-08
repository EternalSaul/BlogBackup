---
title: 'Learning Scala(10):小谈类型'
date: 2017-07-18 14:05:34
toc: true
tags:
 - scala
 - 学习笔记
---

  在上一篇文章里你已经知道了：

   元组和函数字面量是怎么在Scala中的实现方式， 你也知道了->为什么能构造元组，你还知道了怎么用隐含转化在不改变源码的情况下给Scala的类增加方法。

  现在我们来谈谈类型。类型并不只是类，tarit也是类型，类型只是一种规范。

## 类型别名

  类型别名(type alias)会为一个特定的现有类型创建一个新的命名类型。如果你以前写过C语言程序，很多同学会不太喜欢写struct XXX来声明一个结构体变量，而会使用typedef把struct XXX声明为XXX，类似这个功能Scala中也可以声明别名：

**type &lt;identifier &gt;  [type parameter] =type &lt;identifier &gt;  [type parameter]**

<!--more-->

```scala
/*
有时候我们可能经常需要某个固定的元组类型，于是我们可以把它别名化，像类型一样使用它
*/
scala> type three=Tuple3[Int,Int,Double]
defined type alias three

scala> new three(1,2,3.5)
res27: (Int, Int, Double) = (1,2,3.5)
```

## 抽象类型

  抽象类型做法类似于类型别名，但是他们不能用来创建实例，常用于类型参数。

```scala
/*
声明抽象类型T,和一个返回T的函数
*/
scala> trait B{type T;def say:T}
defined trait B

scala> class A
defined class A

/*
这里抽象类型T被定义为了A的类型别名
*/
scala> trait C extends B{
     | type T=A
     | def say=new A
     | }
defined trait C

/*
这段代码等同于
*/
scala> trait B[T]{def say:T}
defined trait B

scala> trait C extends B[A]{def say=new A}
defined trait C

```

## 类型定界

我们可以指定类型参数的限制，其中

### 上界和下界

**上界：限制一个类型只能是该类型或它的某个子类型**

**下界：限制一个类型只能是该类型或它扩展的某个类型**

上界符号用**<:**标识，而下界的标识符为**>:**

```scala
/*
首先定义类A B C D
*/
scala> class A
defined class A

scala> class B extends A
defined class B

scala> class C
defined class C

/*
D的类型参数里标明了传入类型T必须是A的上界，也即使A或者A的子类
*/
scala> class D[T<:A]
defined class D

/*
可以看到由于C不是A的子类，导致编译错误
*/
scala> new D[C]
<console>:13: error: type arguments [C] do not conform to class D's type parameter bounds [T <: A]
       val res0 =
           ^
<console>:14: error: type arguments [C] do not conform to class D's type parameter bounds [T <: A]
       new D[C]
           ^
/*
对应A，B来说正常
*/
scala> new D[A]
res1: D[A] = D@281b2dfd

scala> new D[B]
res2: D[B] = D@12abcd1e

/*
定义E，传入的类型参数T必须是B的下界，即B和B所扩展的类
*/
scala> class E[T>:B]
defined class E

scala> new E[B]
res3: E[B] = E@785ed99c

scala> new E[A]
res4: E[A] = E@559991f5

scala> new E[C]
<console>:13: error: type arguments [C] do not conform to class E's type parameter bounds [T >: B]
       val res5 =
           ^
<console>:14: error: type arguments [C] do not conform to class E's type parameter bounds [T >: B]
       new E[C]
           ^
/*
对于B的子类F，同样报错，因为它不符合下界定义
*/
scala> class F extends B
defined class F

scala> new E[F]
<console>:13: error: type arguments [F] do not conform to class E's type parameter bounds [T >: B]
       val res6 =
           ^
<console>:14: error: type arguments [F] do not conform to class E's type parameter bounds [T >: B]
       new E[F]
           ^
```

### 视界

**视界(view-bound)符号为<%,限定必须接受能转化为该类型的类型，即子类和本身，还有隐含转化 **

```scala
scala> class A
defined class A

/*
定义隐含F，类A可以隐式转化于它
*/
scala> implicit class F(x:A)
defined class F

/*
给函数viewBound添加上界限定
*/
scala> def viewBound[T<:F] {}
viewBound: [T <: F]=> Unit

/*
直接调用成功
*/
scala> viewBound

/*
上界不能传入A,作为类型参数，因为A并不是F或者F的子类
*/
scala> viewBound[A]
<console>:15: error: type arguments [A] do not conform to method viewBound's type parameter bounds [T <: F]
       viewBound[A]
                ^
/*
给函数viewBound添加视界限定
*/
scala> def viewBound[T<%F]{}
viewBound: [T](implicit evidence$1: T => F)Unit

/*
视界可以接收隐含类型转化，这里A可以转化为F，所以调用成功
*/
scala> viewBound[A]
```

### 用定界符声明抽象类型

毫无疑问，抽象类型也可以用定界符来声明，与类型类型参数中使用可以形成差不多的功能

```scala
scala> class A
defined class A

scala> class B extends A
defined class B

/*
用下界定界符来声明抽象数据类型T
*/
scala> abstract class C{type T>:B}
defined class C

/*
满足所声明的下界定义
*/
scala> class D extends C{type T=A}
defined class D
```

## 类型变化

  默认情况下，类型参数给定了类后创建的实例只能与当时给定的类以及参数化类型兼容，这个实例不能存储在类型参数为基类的值中。比如：

```scala
scala> class A
defined class A

scala> class B extends A
defined class B

/*
定义C，它接受一个类型参数
*/
scala> class C[T](val a:T){def say:T=a}
defined class C


scala> val k:C[A]=new C[A](new A)
k: C[A] = C@1c58d7be

/*
可以看到当类型参数给定的实例传给一个类型参数给定的值，即使A是B的基类，编译器却不能自动完成变换操作
*/
scala> val k:C[A]=new C[B](new B)
<console>:15: error: type mismatch;
 found   : C[B]
 required: C[A]
Note: B <: A, but class C is invariant in type T.
You may wish to define T as +T instead. (SLS 4.5)
       val k:C[A]=new C[B](new B)
                  ^
/*
可以看到如果实例创建的时候不给定类型参数，则能进行赋值，因为这时候编译器实际上构建的是C[A]的实例，它通过值的类型来识别这一点,如果没有给定值k的类型，那么编译器还是会自动创建C[B]的实例
*/
scala> val k:C[A]=new C(new B)
k: C[A] = C@23d0944b

/*
可以看到，say函数的返回值类型是A
*/
scala> k.say
res24: A = B@37caecda

scala> val k:C[B]=new C[B](new B)
k: C[B] = C@18e8eb59

scala> k.say
res25: B = B@45ad3cd8
```

###  协变

 所以我们可以得出结论，类型参数默认是不会自动转化为兼容类型的，为了让它们能够自动兼容，那么我们需要使用到**协变类型参数（Covariant type parameter）**,通过在参数类型前加一个**加号（+）**，可以把一个类型参数声明为限变类型。

  因为父类型无法转换为子类型，所以如果类型参数将作为方法的输入参数类型，则该类型参数不能声明为协变类型参数：

```scala
/*
把类型参数T声明为协变类型
*/
scala> class C[+T](val a:T){def say:T=a}
defined class C

scala> val k:C[A]=new C[B](new B)
k: C[A] = C@7846913f

/*
可以看到编译器现在机智的完成了兼容转换
*/
scala> k.say
res29: A = B@514dc5f4

/*
可以看到作为参数类型的T不能作为协变类型
*/
scala> class D[+T]{def say(a:T)=a}
<console>:12: error: covariant type T occurs in contravariant position in type T of value a
       class D[+T]{def say(a:T)=a}
                           ^
```

### 逆变

  逆变（contravariant）是值**一个类型参数可以调整为一个子类型**。逆变类型参数的声明符号是**减号（-）**它可以用于方法的输入参数，但是不允许用作为方法的返回值。

```scala

scala> class A
defined class A

scala> case class B() extends A
defined class B

/*
先声明C,C不是没有逆变类型参数
*/
scala> case class C[T](){def say(a:T)=a.toString}
defined class C

/*
母类无法转换为子类
*/
scala> val s:C[B]=C[A]
<console>:16: error: type mismatch;
 found   : C[A]
 required: C[B]
Note: A >: B, but class C is invariant in type T.
You may wish to define T as -T instead. (SLS 4.5)
       val s:C[B]=C[A]
                   ^
/*
从新声明，有逆变类型参数的C
*/
scala> case class C[-T](){def say(a:T)=a.toString}
defined class C

/*
C[A]可以存储于类型为C[B的值]，可以进行逆变操作
*/
scala> val s:C[B]=C[A]
s: C[B] = C()

/*
逆变变量不可以实现协变操作
*/
scala> val s:C[A]=C[B]
<console>:16: error: type mismatch;
 found   : C[B]
 required: C[A]
       val s:C[A]=C[B]
                   ^
```

通常情况下，我们最好保持类型参数不可变，除非是特别的情况。