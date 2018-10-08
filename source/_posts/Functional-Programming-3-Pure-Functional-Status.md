---
title: 'Functional Programming (3):纯函数式状态'
date: 2017-07-29 15:32:43
toc: true
tags:
 - scala
 - FP
---

  如何以纯函数来实现有状态的API？你曾经写程序的时候可能会发现，函数的输出并不一定和输入有关，有可能和一个状态值有关，比如某平台在工作日的8:00-20:00才允许登入，你可能会写一个if判断来判断当前时间是不是属于允许登录时段来作为登录方法，但是现在情况有所不同，让我们再次来看函数式代码的定义：                                           

​     **在函数式代码中，函数的输出仅依赖于传入的参数，同样的输入意味着同样的输出，和局部或全局状态无关 **

  既然我们不能允许一个全局状态量来决定我们的输出，肯定不意味着我们没办法写这样的程序，只不过现在我们要抛弃传统的命令式编程，转换到函数式上来。

<!--more-->

## 一个广泛的例子

  现在可以举一个最最广泛的例子，我在《Scala函数式编程》一书中看到了它，在我一开始学Scala时，我也曾经思考过这一过程如何以函数式方式来实现，那就是线性同余算法，它广泛用于取随机数，它需要读取一个种子来作为初始的起点，相同的种子所生成的伪随机数列是相同的，所以很多时候我们都以时间来作为初始种子，已达到伪随机目的，可是现在在函数式编程中，这样做似乎不合时宜了，因为取时间就不是纯函数，比如按往常我们可能会这样取一个随机数：

```scala
scala> math.random
res0: Double = 0.210856104557974

scala> math.random
res1: Double = 0.8369504654147353
```

  很显然，以上取随机数的方式肯定不是纯函数实现的。现在我们考虑如何以纯函数形式构造一个随机数生成器，显然我们需要显示输入种子，之后我们应该返回新的种子和生成的数，但是在这之前让我们先来回顾线性同余算法（LCG），以至于我们可以构造我们想要的。

### 线性同余法

```javascript
/*

m是一个大数，一般取2^w,w为机器字长

a∈[0.01m,0.99m],该区间内随便取

b是正整数，可以就取1

以上取法不考虑线性同余算法的周期最大化

*/

function(m,seed,a,b){
     return (a*seed+b) mod m
}
```



现在我们了解了线性同余算法，用纯函数形式生成一个随机数生成器，应该同时返回生成值和新的状态。

```scala
scala> case class Random(seed:Long){
     | def nextInt={
     | val newSeed=(seed*0x5E4CBAEFFEA1L+1)&0xFFFFFFFFFFFFL
     | val nextRNG=Random(newSeed)
     | val n=newSeed.toInt
     | (n,nextRNG)
     | }
     | }
defined class Random

scala> val (n1,r1)=Random(54548).nextInt
n1: Int = 1234950549
r1: Random = Random(58859466774933)
```

现在我们以纯函数形式生成了一个随机数生成器，该随机数生成器对应固定的输入总是对应着一个固定的元组输出。

  写带状态的纯函数程序，都是基于这样一个简单的例子，用函数接受一个状态参数，结果里一并返回一个新的状态。