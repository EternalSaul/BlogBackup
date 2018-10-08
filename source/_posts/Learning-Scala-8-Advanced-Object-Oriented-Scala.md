---
title: 'Learning Scala(8):Advanced Object-Oriented Scala '
date: 2017-07-17 16:24:45
toc: true
tags:
 - scala
 - 学习笔记
---

  上一篇文章讲了Scala面向对象核心部分，看起来好像Scala的面向对象语法和运用和java似乎并没有什么区别，的确如果只看那一部分，你甚至都可能会认为Scala毫无新鲜感，但实际上，Scala一定是可以给你新鲜感的语言，并且这种感觉是尖锐的，面向对象是很老的概念，Scala既然支持面向对象，又是一门有点野心的语言，那它怎么也得在这个主题是弄点定一点特别东西出来，这一篇文章就是介绍一个Scala面向对象的特别的一些组件，它们分别是对象，Case以及Trait。

## 对象

###   Scala中的对象

  在Scala中对象是一个只能有不超过1个实例的**类类型**，写过一些代码的同学都会立刻想到单例(singleton)模式，实际上没错，用面向对象设计的角度来说它就是一个单例，**它在首次访问时初始化**，就如同你首次调用了单例模式类的getInstance方法一般。

  如果你仔细看了之前的那些代码，会发现Scala中可没有static修饰符，好像也没提到过静态和全局的概念，这里Scala的设计者用**对象**提供了类似的功能，此对象非彼对象，它不再表示的是一个类的实例。

<!--more-->

  Scala的对象概念分离了全局和基于实例的字段和方法，为什么这样说？答案很显然：

**对象是一种类类型，它可以扩展另一个类，被它扩展的类可以在全局实例中使用它的字段和方法**

```scala
scala> :paste -raw
// Entering paste mode (ctrl-D to finish)

class Car {
  def introduce=println("I am a Car")
}

class Bmw extends Car{
  override def introduce=println("I am a BMW")
}

class Benz extends Car{
  override def introduce=println("I am a BENZ")
}

class Audi extends Car{
  override def introduce=println("I am a AUDI")
}

object Car{
  def whatsIt="a Car"
}

/*
可以以对象去扩展一个类
*/
scala> object Audi extends Car
defined object Audi

// Exiting paste mode, now interpreting.

/*
首次访问Car被实例化了
*/
scala> Car
res2: Car.type = Car$@487cd177

/*
可以直接访问Car对象的
*/
scala> res2.whatsIt
res3: String = A Car

scala> new Car
res7: Car = Car@c6c84fa

/*
累的实例不能访问对象的方法
*/
scala> res7.whatsIt
<console>:13: error: value whatsIt is not a member of Car
       res7.whatsIt
            ^
/*
扩展了Car的对象Audi可以去访问Car的方法和字段
*/
scala> Audi.introduce
I am a Car

scala> Audi.cool
res0: String = I am a cool car
```



### 对象的apply方法

  **如果一个对象与一个类的名字相同，那么这个对象就是这个类的伴生对象。**

  前面我们提到过apply方法，它是一种快捷方式，可以直接以括号形式调用而不需要方法名，回想一下我们怎么创建一个列表，List(1,2,3,4)，在这条语句中我们同样没有使用方法名，也许你思考过为什么不需要new语句就可以构造一个实例，实际上它是List对象的apply方法， 举一反三，你回想Option(null),Some(1)等等，都运用了各自**伴生对象**的apply方法并返回一个实例，这是面向对象中的工厂模式。

   一个类和它的伴生对象可以互相访问私有和保护字段和方法。

```scala
scala> :paste -raw
// Entering paste mode (ctrl-D to finish)

class CDROM (private val price:Int){
  def play=for(i<-CDROM.trackList){println(i)}
  def infomation=println(CDROM.cdName)
}

object CDROM{
  /*
  运用apply方法，构成一个工厂方法
  */
  def apply(price:Int)=new CDROM(price)
  private val cdName="Fifty Shades Of Grey"
  private val trackList=List("One Last Night","Love Me Like You do","I'm On Fire")
}

// Exiting paste mode, now interpreting.

/*
工厂模式得到一个实例
*/
scala> CDROM(12)
res0: CDROM = CDROM@69ba3f4e

/*
调用play方法，访问了CDROM对象中的曲目列表
*/
scala> res0.play
One Last Night
Love Me Like You do
I'm On Fire
```

## Case类

  前面讨论了Scala的对象，并且我们可以用它来实现单例和工厂模式，现在我们来看看另一个组件：Case

   **Case是不可实例化的类，它包含了多个自动生成的方法和一个自动生成的伴生对象，这个伴生对象也包含了自己自动生成的方法**

  要创建一个case类，需要在class前加上case关键字。case类会将参数自动转化为值，所以没有必要在参数面前加上val来声明这是一个值，如果希望一个变量作为字段，则需要加上var关键字。**另外只有是case类参数的字段，自动生成的方法才能利用**

   下表是case类自动生成的方法，和伴生对象中的方法

| 方法名      | 位置   | 描述                                      |
| -------- | ---- | --------------------------------------- |
| apply    | 对象   | 工厂方法，用于实例化case                          |
| copy     | 类    | 返回实例的一个副本，并完成所需要的修改，参数是类的字段，默认值设置为当前字段值 |
| equals   | 类    | 如果两个事例中的所有字段匹配，则返回true，也可以用==来调用        |
| hashCode | 类    | 返回实例字段的一个散列码，对基于散列的集合很有用                |
| toString | 类    | 将类名和字段程现为一个String                       |
| unapply  | 对象   | 将实例抽取到一个字段元组，从而可以使用case类实例完成模式匹配        |

​    case类的好处是，如果你觉得麻烦，他可以自动增加以上这些方法，减少你的样板代码量，如果你觉得应该自己动手丰衣足食，可以自己写这些方法和伴生对象。

```scala
scala> case class Intel(series:String,num:Int)
defined class Intel

scala> Intel("i7",4850)
res8: Intel = Intel(i7,4850)

scala> Intel("i7",7700)
res9: Intel = Intel(i7,7700)

scala> Intel("i5",6200)
res10: Intel = Intel(i5,6200)

scala> res10.copy(series="i7")
res11: Intel = Intel(i7,6200)

scala> res10.copy()
res13: Intel = Intel(i5,6200)

scala> res13==res10
res14: Boolean = true

/*
这里运用了case类自动生成的对象中的unapply方法
*/
scala> res8 match{
     |    case Intel("i5",x)=>println(s"i5 ${x}hq")
     |    case Intel("i7",x)=>println(s"i7 ${x}hq")
     | }

i7 4850hq

```

## Trait

  trait类似于java中的interface，但是又有些不同，trait可以继承一个类而interface不允许，可以继承一个类就意味着trait可以提供方法的实现，但它也不同于抽象类，因为抽象类不能继承多个抽象类，而Trait类可以扩展多个Trait类,另外trait是不能实例化的，它不能有类参数。

  定义一个trait需要使用关键字trait而不是class。

```scala
scala> trait FileUtils{
     |      def load(directory:String)=println(s"I have loaded $directory")
     | }
defined trait FileUtils

scala> object VideoPlayer extends FileUtils
defined object VideoPlayer

scala> VideoPlayer.load("C:\\Japanese Movie\\Namitano Yoshii\\MXGS-896.avi")
I have loaded C:\Japanese Movie\Namitano Yoshii\MXGS-896.avi
```

### Trait的多重继承详解

  作为一个JVM语言Scala的trait怎么实现的多重继承呢？JVM只能扩展一个父类，Scala运行在JVM上也逃不出这个限制，但是Scala的编译器会创建各个trait的副本，组成一个继承链，类A扩展了类B,C,D，实际上是A扩展D,D扩展C，C又扩展B它不是一次性又A扩展三个类而来的，而是线性垂直的扩展。

  这里需要注意的是，如果一个类导入了两个trait，它们有相同的字段和方法而没有override关键字，就会直接编译错误，如果你记得的话，对应已经实现的方法，如果要子类中重写，那么必须加override，而scala的trait的多重继承实际上是会转化为垂直扩展的。

  这里你可能想知道trait的多重继承的顺序是什么？比如：

   class A extends B with C with D

   这个继承顺序是重右往左的,编译器重新实现为：

   class A extends D extends C extends B  

### 自类型

自类型是一个trait注解，用来限制类增加trait,它要求如果一个类要扩展这个trait必须有一个特定的类型或自类型。用自类型可以确保trait总是扩展一类类型。

语法:

   **trait....{&lt;identifer&gt;:&lt;type&gt;=> ....}**

自类型的标准标识符是self，不过也可以其他的，除了保留关键字，只要你和你的工作伙伴看得懂就行了。

```scala
scala> class A{ def hi="halo"}
defined class A

/*
创建Trait类B，指定它的子类型为A，并重写toString方法
*/
scala> trait B{self:A=>
     |    override def toString="B:"+hi
     | }
defined trait B

/*
C不是A或者A的子类型
*/
scala> class C extends B
<console>:12: error: illegal inheritance;
 self-type C does not conform to B's selftype B with A
       class C extends B
                       ^

/*
C继承A通过编译
*/
scala> class C extends A with B
defined class C

scala> new C
res22: C = B:halo
```



  自类型更大的好处在于，它声明了自己必是某个类的子类，于是它可以在自己的方法里调用那个类的方法或者某些参数，这样也解决了自类型不能有类参数的问题

```scala

scala> class Car {
     |   def introduce=println("I am a Car")
     | }
defined class Car

scala>

/*
可以在定义trait的时候直接以self调用introduce,因为自类型限定了继承关系，编译器知道AT必定是Car的子类
*/
scala> trait AT {self:Car=>
     |   def talk={self.introduce;println("I am auto")}
     | }
defined trait AT

scala>

scala> class Bmw extends Car with AT{
     |   override def introduce=println("I am a BMW")
     | }
defined class Bmw

scala> new Bmw
res8: Bmw = Bmw@1de73d37

scala> res8.talk
I am a BMW
I am auto
```

### 在类实例化时增加Trait

  还有一种使用trait的方法是在类实例化时为类增加trait，实例化的对象就可以利用trait的功能，这种实例化会用增加的trait去扩展这个类，而不是用类来扩展trait。现在我们来看看，如何使用with关键词在类实例化时增加一个或多个trait。

```scala
scala> class F
defined class F

scala> trait G{def touch=println("I can touch the G point")}
defined trait G

/*
实例化是添加上G
此时一个产生一个匿名类，名为$anon$1
*/
scala> new F with G
res10: F with G = $anon$1@7d4d65f5

scala> res10.touch
I can touch the G point

/*
添加一个自类型
*/
scala> trait K{self:A=> def touch=println("I can touch the K point")}
defined trait K

/*
不满足自类型的实例化扩展发生错误
*/
scala> new F with K
<console>:14: error: illegal inheritance;
 self-type F with K does not conform to K's selftype K with A
       new F with K
                  ^
```

