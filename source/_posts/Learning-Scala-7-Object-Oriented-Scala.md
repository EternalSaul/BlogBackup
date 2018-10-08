---
title: 'Learning Scala(7):从类开始'
date: 2017-07-16 08:19:29
toc: true
tags:
 - scala
 - 学习笔记
---

### 定义类

你还记得Scala类的层次继承关系么？

![Scala类的继承关系](\img\QQ图片20170716082739.png)

  在上图中，第二层里AnyVal是所有值类型的根，而AnyRef是所有引用类型的根，我们以前说到过Scala是一门多范式语言，现在我们来看看面向对象的Scala，既然是面向对象那么类就是核心构件，然后引出面向对象的三个基本的特性，即多态(Polymorphism)，继承(inheritance)，封装(encapsulation)。作为一门强大的语言，Scala很好的支持这些特性，然而在实现方法上和细节上，你将会看到Scala对比其他语言的不同之处，但是首先我们先快速的定义一个类来开始我们队Scala面向对象特性的探索。

<!--more-->

```scala
/*
有想过如此简单的方式定义一个类么？
*/
scala> class Client
defined class Client

/*
构造一个实例,@406decb2代表内存位置
*/
scala> val c=new Client
c: Client = Client@406decb2

//所以引用对象的子类型都是AnyRef
scala> println(c.isInstanceOf[AnyRef])
true

//所以类型的基类Any
scala> println(c.isInstanceOf[Any])
true
```

  如果你用过java,你可能会发现，Client@406decb2和java里的Object的toString输出很类似，实际上java.lang.Object是JVM中所有类型的根，我们所定义的Client类根本没有重写toString方法，所以REPL调用的是Any的toString方法，实际上等价于java.lang.Object，所以你看到了类似的结果。

   下面我们来看看一个更加完整的Client的定义，它加上了属性并且重写了toString方法

```scala
/*
这里的ip和pn是类参数，用来初始化类的属性
*/
scala> class Client(ip:String,pn:Int){
     |   val ipAddress:String=ip
     |   val portNum:Int=pn
     |   override def toString=s"Client($ipAddress,$portNum)"
     | }
defined class Client

scala> val c=new Client("13.15.14.12",6969)
c: Client = Client(13.15.14.12,6969)

scala> c.ipAddress
res4: String = 13.15.14.12
```

  实际上还可以直接把属性字段声明为类参数，这样你的代码量会变少，只需要在类参数前面加val和var就可以了，比如：

```scala
/*
把属性直接才类参数上声明
*/
scala> class Client(val ip:String,val pn:Int){
     |         override def toString=s"Client($ip,$pn)"
     | }
defined class Client

scala> new Client("13.14.15.16",8869)
res5: Client = Client(13.14.15.16,8869)
```


### 继承

和JAVA一样，Scala也是单继承的，一个类只能直接继承一个类，继承关键字同样是extends，另外在this和super上和java是一样的，不同的是如果你要覆盖父类只能的关键字，你必须用override标识，而java中这是一个可选项，如果你不用override标识会发生什么呢？

```scala
/*
定义一个基类Server
*/
scala> class Server(val ip:String,val pn:Int){
     |        def doPost=s"Server do post"
     |        def doGet=s"Server do get"
     |        override def toString=s"Server($ip,$pn)"
     | }
defined class Server

/*
scala要求重写时则必须加override标识
*/
scala> class ServerPlus(ip:String,pn:Int) extends Server(ip,pn){
     |       def doPost=s"ServerPlus do post"
     | }
<console>:13: error: overriding method doPost in class Server of type => String;
 method doPost needs `override' modifier
                                 def doPost=s"ServerPlus do post"

/*
添加override标识，类定义成功
*/
scala> class ServerPlus(ip:String,pn:Int) extends Server(ip,pn){
     |        override def doPost=s"ServerPlus do post"
     | }
defined class ServerPlus
```

### 简单的泛型类

  我们知道Scala的函数定义可以使用类型参数，那么类的定义是不是也可以加入类型参数呢？作为一门完备而高效的程序语言，毫无疑问，我们可以加入泛型，并且还可以有诸多特别的选择，在这里我们先介绍最简单的泛型，剩下的以后再谈。

```scala
scala> class Session
defined class Session

scala> class BaseDao[T]{
     |   def insert(session:Session,obj:T)="insert into DB"
     |   def find(session:Session,id:Int,obj:T):T={"find from DB";obj}
     |   def update(session:Session,obj:T)="update on DB"
     |   def delete(session:Session,id:Int)="delete from DB"
     | }
                                                   
defined class BaseDao

scala> val serverdao=new BaseDao[Server]
serverdao: BaseDao[Server] = BaseDao@42caaae4

scala> serverdao.find(null,1001,new Server("1.1.1.1",5656))
res11: Server = Server(1.1.1.1,5656)
```

### 抽象类定义

  Scala中抽象类由abstract指定,一个抽象类的子类如果未标识为抽象类，那么它就必须提供抽象类中没有具体起始值和实现的字段和方法的实现，而对应那些已经在抽象类中定义具体值和定义的方法则不要求实现。当然如果有必要你也可以重写。

```scala
scala> class AuthorityService
defined class AuthorityService

/*
定义一个抽象类
*/
scala> abstract class BaseController(val athService:AuthorityService){
     | def redirect:String
     | }
defined class BaseController

/*
定义一个类继承抽象类BaseController,但是不实现它的抽象方法，编译器抛出错误
*/
scala> class ClientController(as:AuthorityService) extends BaseController(as){}
<console>:13: error: class ClientController needs to be abstract, since method redirect in class BaseController of type => String is not defined
       class ClientController(as:AuthorityService) extends BaseController(as){}

/*
对应抽象方法，加不加override其实无所谓
*/
scala> class ClientController(as:AuthorityService) extends BaseController(as){
     | def redirect="I am going to redirect"
     | }
defined class ClientController

scala> class ClientController(as:AuthorityService) extends BaseController(as){
     | override def redirect="I am going to redirect"
     | }
defined class ClientController

scala> new ClientController(new AuthorityService)
res14: ClientController = ClientController@1e6e59d8

/*
可以看到redirect方法其实是以一个值来定义的，调用的时候好像我们就是在调用它的一个属性一样
*/
scala> res14.redirect
res15: String = I am going to redirect
```

### 匿名类

  如果你写过java Swing或者安卓的程序，作为测试的时候你可能并不会把事件拿出来单独作为一个类，而是喜欢用匿名类来添加事件，scala中也一样，我们可以用一个熟悉的监听者模式来看看这一点：

```scala
scala> abstract class ActionListenner{
     |   def onAction
     | }
defined class ActionListenner

scala> class Button{
     |   def setActionListenner(listenner:ActionListenner)=listenner.onAction
     | }
defined class Button

scala> res19.setActionListenner(new ActionListenner{ def onAction=println("I am clicked")})
I am clicked

```

### apply方法

  apply方法是一个快捷方式，它可以直接用小括号触发功能而不需要方法名，比如我们很熟悉的List，访问它的某一个索引值，其实就是调用了它的apply方法：

```scala
scala> val ls=List(1,2,3,4)
ls: List[Int] = List(1, 2, 3, 4)

/*
这里为什么可以直接用小括号访问的本质原因就是：快捷调用List的apply方法
*/
scala> ls(2)
res23: Int = 3

/*
你也可以用方法名来访问
*/
scala> ls.apply(2)
res24: Int = 3
```

### 懒值

  以lazy关键字标识一个值，会让这个值成为一个懒值(lazy value)，懒值并不在类第一次实例化时创建，而是只在第一次实例化它自己时才创建，你可以把它看做是只会执行一次的方法，执行之后就保存了执行该方法的返回值。

```scala
scala> class LazyValueTest{
     | lazy val rv={util.Random.nextDouble}
     | }
defined class LazyValueTest

scala> new LazyValueTest
res49: LazyValueTest = LazyValueTest@12671195

/*
第一次访问懒值，表达式被执行，懒值被置为对应的返回值
*/
scala> res49.rv
res50: Double = 0.6486587816803634

/*
不再去随机浮点，而是用第一次得到的结果
*/
scala> res49.rv
res51: Double = 0.6486587816803634
```

   懒值应该是有额外开销的，每次调用它的时候会先去检查它是不是已经被初始化。懒值这种东西应该用在哪里呢？设想一下一个软件可能对应不同的用户有着不同的频繁程度的操作，有些服务可能一些用户根本不会用到，比如某些付费服务，那么如果你单纯的为了速度这个用户体验，每次都去尽量缓存一些可能根本不会用到的数据，或者对一些服务线程进行缓存，还有某些预加载，其实本质上是浪费了时间也浪费了服务器的计算量，对应这样情况懒值应该是一个比较好的选择。虽然懒值会带来一定开销，比如需要存储这个函数的指令序列等等，但是比起一些总是做无用的磁盘IO等等还是划得来的。

### 私密性控制

  private,protected,public，如果你之前写过java,c++等等代码，就不会对这些关键字有任何模式，然而就好像java和c++的访问权限修饰符有微小的差异一般，Scala在这些修饰符上也存在着和他们的一些差异，比如默认的Scala根本不会增加私密性控制，而不像java那样有一个default权限。

**Scala权限修饰符表**

| 修饰符       | 说明                         |
| --------- | -------------------------- |
| private   | 定义该字段或方法的类才能访问             |
| protected | 只有同一个类或其子类中的代码才能访问这个字段或者方法 |
| public    | 默认修饰符                      |

有时候你可能需要粒度更细的控制，比如你可能希望，一个类对某个包之外所有的代码都是私有的

```scala
private[com.etern.saul] class Hello{
  private[this] hi="halo"
}
```

### final和sealed修饰符

  同java一样，final声明的方法和字段不能被覆盖，final声明的类不能被其他类继承。

  除了Final类，Scala中还提供了Sealed类，seal是密封的意思，一个Sealed类会限制一个类的子类必须位于父类所在的同一个文件中。定义Sealed类，需要在class关键字前加上一个sealed关键字作为前缀，实际上我们已经接触了很多个密封类，只是你可能没有意思到而已，比如我们之前讨论过的一元集合Option，它是抽象类，同时也是密封类，我们讨论过它的两个子类，None和Some。

```scala
/*
尝试继承Option
*/
scala> class s extends Option
<console>:11: error: illegal inheritance from sealed class Option
       class s extends Option
                       ^
```

  可以看到把Option密封化，可以防止别人轻易的增加额外的只类，它可以把子类限制在与父类同一个文件。