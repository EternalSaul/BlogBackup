---
title: 'Learning Scala (1):一等公民——first-class function'
date: 2017-07-05 10:50:10
toc: true
tags:
 - scala
 - 学习笔记
categories:
---

  最近一周都在学习Scala这门语言,昨天刚刚看完O‘Reilly动物系列的[《Scala学习手册》](https://book.douban.com/subject/26786632/)，这是一本Scala语法入门书，总共就200来页，对于想学Scala的且有一定java基础的人来说可以入一本看看，Scala和C++一样是一门多范式语言，如果你能有一个安静的看书环境2-3就可以搞定，这本书里并没-有主要介绍Scala的函数式编程特点，只是用来语法入门，如果你想了解Scala函数式编程，在读完这本书之后可以看[《Scala函数式编程》](https://book.douban.com/subject/26772149/)这本书。



## first-class function介绍

  什么是First-class function？

  在[《Scala学习手册》](https://book.douban.com/subject/26786632/)中译者把它叫做首类函数，我觉得这个名字翻译的很奇怪，所以在这里我不用这个名字。    

  first-class为名词作定语，它是用作来修饰类型的（在Scala中函数都有类型），可以把它看做是第一类，一等公民的意思，这样说你可能不太理解，但是如果我告诉你，还有second-class,和third-class你大概就知道了，一个函数可以加上First-class来修饰，是因为它满足了first-class的定义，那么这一、二、三类的区别是什么呢？

| 类型           | 描述                                       |
| ------------ | ---------------------------------------- |
| First-class  | 表示一种类型的值不仅能得到声明和调用，还可以作为一个数据类型作用在这种语言的任务地方 |
| Second-class | 表示一种类型的值可以作为函数的参数，但不能从函数返回，也不能赋给变量       |
| Third-class  | 表示一种类型的值作为函数参数也不行，更不能从函数返回，也不能赋给变量       |



  现在你知道什么是First-class了，如果你以前用过Javascript，你会更加理解一个函数是首类函数的便利性，在函数式编程语言中函数是First-class的，Scala是多范式语言，函数式编程是它的重要特性，所以Scala中的函数是First-class function，理解这一点，对应学习Scala是重要的。

  First-class function与其他数据类型一样可以采用字面量的形式创建，而不必指定标识符，他可以存储在一个容器中，如值，变量或者数据结构，还可以作为另一个函数的参数或者返回值。

<!-- more -->

## 高阶函数

什么是高阶函数（higher-order function）？

---

**如果一个函数接受其他函数作为参数，或者使用函数作为其返回值，那么就它就是一个高阶函数**

---

  你可能用过jquery包装集中的map和each函数，这是两个典型的高阶函数，通过传入一个函数，作为处理逻辑，来对包装集每一项进行处理。当然还有很多其他的著名的高阶函数，比如归约函数reduce等等。



### 授人以渔——声明式编程

---

  高阶函数的一大好处在于：调用者可以指定要做什么（传入具体的函数实现），让高阶函数处理具体的逻辑流，这种方法即**声明式编程（declarative programming）**。通常这与使用函数式编程有关，要求使用高阶函数或其他机制声明要做的工作，而不手动实现。就像是教机器如何去做事情一样。

比如对于一个1..10的累加循环

```scala
scala> val l=List(1,2,3,4,5,6,7,8,9,10);

l: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> val k=l.reduce((a:Int,b:Int)=>a+b);

k: Int = 55

```

---



### 授人以鱼——命令式编程

---

  与其相对的是**命令式编程（imperative programming）**。采用这种方法时总是要明确指定逻辑的操作流。就像是教机器做什么事情一样。

比如对于一个1..10的累加循环

```scala
scala> var k=0
k: Int = 0

scala> for(i<- 1 to 10){k+=i}

scala> k
res1: Int = 55
```

---



##    函数类型和值



###    函数类型

---

  Scala中函数的类型是其输入类型和返回值得一个简单的组合，由一个箭头从输入类型指向输出类型。语法如下

**（[&lt;type&gt;,...]）=>&lt;type&gt;**

比如：

`def plus(x:Int,y:Int):Int=x+y`

类型为(Int,Int)=>Int

`def hello(name:String,y:Int)=name+y`

类型为(String,Int)=>String

### 函数值

现在我们可以尝试把plus函数赋给一个函数值，再调用这个函数值，它会得到与调用plus相同的结果

```scala
scala> val plusCopy:(Int,Int)=>Int=plus
plusCopy: (Int, Int) => Int = $$Lambda$1068/649913869@76eadc5a

scala> plusCopy(1,2)
res0: Int = 3
```

如果我们再把函数值传给另一个值（不指定类型），Scala会使用类型推导使得赋值成功

```scala
scala> val plusMy=plusCopy
plusMy: (Int, Int) => Int = $$Lambda$1068/649913869@76eadc5a

scala> plusMy(1,2)
res1: Int = 3
```

如果直接把plus函数传给一个不指定类型的新值，结果会如何呢？

```scala
scala> val plusUnknowType=plus
<console>:12: error: missing argument list for method plus
Unapplied methods are only converted to functions when a function type is expected.
You can make this conversion explicit by writing `plus _` or `plus(_,_)` instead of `plus`.
       val plusUnknowType=plus
                          ^
```

可以看到，REPL提示没办法进行推导，提示错误。

---



## 函数字面量

函数字面量(function literal)，这个概念有很多名字，比如函数直接量，函数实值等等。**它是一个能正常工作的没有名字的函数**

比如下面的代码：

```scala
scala> val half=(x:Int)=>x/2.0
half: Int => Double = $$Lambda$1088/582025508@6f36e806

scala> half(2)
res4: Double = 1.0
```

(x:Int)=>x/2.0即是一个函数字面量，他定义了一个输入参数x和一个函数体x*2。

函数字面量在任何接受函数类型的地方都可以使用，关于函数字面量的概念还有你可能熟悉的箭头语法，你可能知道如下这些名字：

1. 匿名函数(Anonymous functions) : Scala中函数字面量的正式名字（"Scala语言规范" (Odersky,2011)）
2. Lambda表达式(Lambda expressions) : C#和Java 8中的说法，从数学中的lambda演算(lambda calculus)得来(如：x->x*2)
3. Lambdas : (Lambda expressions) 的缩写
4. function0,function1,function2...  :  Scala编译器对函数字面量的叫法

 函数字面量的使用：

  ```scala

/*
*将一个函数字面量赋值给sayHello,输入值为()表示没有参数
*/
scala> val sayHello=()=>"hello"+new java.util.Date
sayHello: () => String = $$Lambda$1108/33170663@5192b301

/*
*执行函数值sayHello
*/
scala> sayHello()
res9: String = helloWed Jul 05 13:05:29 CST 2017

/*
*将一个函数字面量传给sayHelloToSimth,该字面量是一个匿名高阶函数，接受一个()=>String类型的函数
*/
scala> val sayHelloToSimth=(f:()=>String)=>"Simth "+f()
sayHelloToSimth: (() => String) => String = $$Lambda$1109/383823787@7b9b6a56

/*
*传入函数值sayHello执行sayHelloToSimth
*/
scala> sayHelloToSimth(sayHello)
res10: String = Simth helloWed Jul 05 13:05:42 CST 2017

/*
*传入函数字面量 ()=>"GoodBye 执行sayHelloToSimth
*/
scala> sayHelloToSimth(()=>"GoodBye")
res11: String = Simth GoodBye
  ```
