---
title: 'Functional Programming(4):Monoid'
date: 2017-09-10 22:17:34
toc: true
tags:
 - FP
 - 学习笔记
 - scala
---
## Monoid的数学定义
  在函数式编程中，我们喜欢用代数来描述一些数据类型，包括特定的法则和被它们限定的操作。比如说n阶可逆矩阵在其乘法运算下，满足结合律，且单位阵为幺元，每个n阶可逆矩阵都有逆阵，即逆元，那么它是一个群。
  实际上一些我们有一些通用的代数模式被用于函数式编程式数据类型的设计，对于函数式编程来说，他们就像在面向对象编程时我们要立刻想到各种设计模式一样重要，运用代数模式我们可以消除大量的重复代码！为什么可以呢？我们先来看看幺半群--Monoid。
  在左孝凌版本的离散数学Monoid不叫幺半群，而叫做独异点：
  >   一个代数系统&lt;S,\*&gt;,S为非空集合，而\*为S上的一个二元运算
  >   如果运算是封闭的
  >   且运算是可结合的，即对于x,y,z ∈S满足(x\*y)\*z=x\*(y\*z)
  >   那么&lt;S,\*&gt;是半群
  >   如果半群众含有幺元（对于任何x∈S，存在e,e\*x=x=x\*e,e即&lt;S,\*&gt;的幺元），那么它就是一个幺半群（独异点）

  <!--more-->

## Monoid示例
  理解这样一个概念的意义是帮助我们去选择纯代数（purely algebraic）结构，我们可以从这个简单的定义中提取一些有意义的东西，即这个代数结构的定义，比如对于字符串，我们思考一下它&lt;String,+&gt;上是不是满足Monoid定义呢？
  首先，&lt;String,+&gt;上的运算是封闭的，这个是肯定的，你不可能把两个字符串相加，加出个别的什么东西来
  其次，&lt;String,+&gt;是可结合的，这个我们可以推论出来对于字符串a,b,c
  **(a+b)+c=a+(b+c)**
  最后，可以找到一个幺元，即空串""，任何字符串s与它做左右的+运算，都等于它本身
  现在我们可以确定，&lt;String,+&gt;,是一个Monoid,于是我们可以定义：
```scala
scala> trait Monoid[S]{
     | def opreate(a:S,b:S):S
     | def identity:S
     | }
defined trait Monoid
```
  现在我们有了Monoid接口定义，它很简单，且体现了除结合律以外的其他两条定义，当然我们用了泛型S来表示集合，这个传入的这个集合必须是非空的。
  那么对于&lt;String,+&gt;，我们有
```scala
scala> class Stringplus extends Monoid[String]{Map[String,Map[String,String]]
     | def opreate(a:String,b:String)=a+b;
     | def identity=""
     | }
defined class Stringplus
```
## 结合律与并行化
   实际上如果你仔细一点想想，就会发现有很多&lt;S,#>,满足Monoid定义，比如整数的加法和乘法，布尔型的与或运算，这些都是最基本的，实际上在Scala中很多集合运算在集合上都是符合Monoid定义的！比如对于List的foldLeft和foldRight操作,对于同样的输入，它们能够得出同样的结果：
```scala
scala> res5.foldLeft(0)(_+_)
res6: Int = 15

scala> res5.foldRight(0)(_+_)
res7: Int = 15

```
  详细理解过我们的Scala后可以知道List的Left操作的效率要高于Right操作，但是这和左右折叠运算并没有关系，有关系的是左右折叠运算都是满足结合律和同一律的！
  那么一个&lt;S,#>是Monoid,和我们的编程有什么关系呢？
  我们可以考虑一下，一个超大型的数据链表L，做某个操作#，如果&lt;L,#&gt;是满足结合律的，我们可以轻易对其进行并行运算，因为总是有(a#b)#c=a#(b#c)，所以先做哪一部分运算似乎无所谓。

## 同态
  有时候我们还可以发现一些函数符合monoid同态（homomorphism）性质：
> 一个f是Monoid同态函数，那么它在Monoid M和N之间遵循一条原则，即对于所有的值x和y满足：
>  M.op(f(x),f(y))==f(N.op(x,y))

  值得注意的是M和N可以是也可以不是同一类型的Monoid！

  比如对于字符串：
  "xxx".length+"yyy".length==("xxx"+"yyy").length，length函数就是&lt;String,+&gt;和&lt;String,+&gt;之间的同态函数，同样的的情况还出现在两个&lt;List,+>之间。

  有时候，两个Monoid&lt;S,\*&gt;，&lt;S,\*&gt;之间是双向同态的，引出了同构(isomorphism)的定义:
  **monoid同质是指在M和N之间存在两个同态函数f和g，如果f和g满足f andThen g和g andThenf是同样的函数。**

  我们在**设计类型的时候**，应该考虑它的函数**是不是可以符合同态性质**，这对我们的编程是有帮助的。
## 用Monoid接口构建类型
###建一个Foldable类型构造器
  就像上面那样我们构造了一个Monoid接口后构造我们的StringPlus，现在我们要来构建新的类型，这些新的类型总会满足一些特性，比如它们都是可折叠的，而且他们的折叠函数与该集合满足我们的Monoid的特性。
  有了这些需求，我们可以构造这样的一个接口：
```scala
scala> trait Foldable[F[_]]{
     | def foldRight[A,B](as:F[A])(z:B)(f:(A,B)=>B):B
     | def foldLeft[A,B](as:F[A])(z:B)(f:(B,A)=>B):B
     | def foldMap[A,B](as:F[A])(f:A=>B)(mb:Monoid[B]):B
     | def concatenate[A](as:F[A])(m:Monoid[A]):A=foldLeft(as)(m.identity)(m.opreate)
     | }
```
  在这里F[\_]的使用或许让你有些奇怪，这里的意思其实就是抽象出一个类型构造器，下划线占位符标识F接受一个类型参数，它是一个构造器而不是类型，这里引入一个新概念，即高阶类型(higher-kinded type),这个名字包含的意义就和高阶函数差不多，只不过这里Foldable接受一个类型参数而不是函数。有了它我们可以实现这一一个类，它包含了List的折叠函数：
```scala
然后我们来实现一个这样的类
scala> class ListFold extends Foldable[List]{
     |        def foldRight[A,B](as:List[A])(z:B)(f:(A,B)=>B):B=as match{
     |           case Nil=>z
     |           case _=>foldRight(as.tail)(f(as.head,z))(f)
     |   }
     |        def foldLeft[A,B](as:List[A])(z:B)(f:(B,A)=>B):B=as match{
     |           case Nil=>z
     |           case _=>foldLeft(as.take(as.length-1))(f(z,as.last))(f)
     |   }
     |        def foldMap[A,B](as:List[A])(f:A=>B)(mb:Monoid[B]):B={
     |           concatenate(as.map(f))(mb)
     |   }
     | }
defined class ListFold
```
### 组合monoid
  某些数据结构如果包含的元素是monoid那么它就构成monoid，比如对于元组Tuple2(A,B),如果AB都是Monoid那么，(A,B)也是Monoid，这个新生产的monoid叫做product。
  我们可以用简单测combinator来实现对Monoid的复杂组合，比如你创建如下函数，它是一个闭包，返回了一个新的Monoid类型，这个类型用来帮助我们组合Map。

```scala
scala> def mapMergeMonoid[K,V](V: Monoid[V]): Monoid[Map[K, V]] =
     | new Monoid[Map[K, V]] {
     | def identity = Map[K,V]()
     | def opreate(a: Map[K, V], b: Map[K, V]) =
     | (a.keySet ++ b.keySet).foldLeft(identity) { (acc,k) =>
     | acc.updated(k, V.opreate(a.getOrElse(k, V.identity),
     | b.getOrElse(k, V.identity)))
     | }
     | }
mapMergeMonoid: [K, V](V: Monoid[V])Monoid[Map[K,V]]

//创建了两个Map
scala> Map("f"->"hello","k"->"hi")
res50: scala.collection.immutable.Map[String,String] = Map(f -> hello, k -> hi)

scala> Map("f"->" chen","m"->"akka")
res51: scala.collection.immutable.Map[String,String] = Map(f -> " chen", m -> akka)

//创建一个对象，能够以Stringplus来组合Map[String,Map[String,String]]
scala> val merge:Monoid[Map[String,Map[String,String]]]=mapMergeMonoid(mapMergeMonoid(new Stringplus))
merge: Monoid[Map[String,Map[String,String]]] = $anon$1@4113ca53

//定义两个Map[String,Map[String,String]]
scala> Map("o1"->res50)
res56: scala.collection.immutable.Map[String,scala.collection.immutable.Map[String,String]] = Map(o1 -> Map(f -> hello, k -> hi))

scala> Map("o1"->res51)Map[String,Map[String,String]]
res57: scala.collection.immutable.Map[String,scala.collection.immutable.Map[String,String]] = Map(o1 -> Map(f -> " chen", m -> akka))

//完成组合
scala> merge.opreate(res56,res57)
res59: Map[String,Map[String,String]] = Map(o1 -> Map(f -> hello chen, k -> hi, m -> akka))
```
