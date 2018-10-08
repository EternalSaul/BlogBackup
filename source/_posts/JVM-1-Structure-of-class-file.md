---
title: 'JVM (1):类文件结构（上）'
date: 2017-11-25 14:02:15
toc: ture
tags:
 - JVM
 - 学习笔记
---

  **平台无关性**是Java语言最重要的特性之一，这个特性由**字节码**这种中间码作为虚拟机执行的文件载体被大家所认识，**一次编译到处运行**的这种**平台无关性**在一个学习者在刚刚学习编程而且选择的是Java语言时就会了解，但是随着学习者技术视野不断扩张，就会意识到jvm不是提供了java运行的平台，而是提供了字节码运行的平台，像Scala，Groovy，Ruby，Python等等语言通过他们自己的编译器，也可以编译成字节码从而在jvm平台上运行，即**语言无关性**。

  一个非凡的公司去设计一款非凡的程序平台，Java在诞生之初，JVM和Java语言的规范就是分开来的，JVM只关心Class文件里的字节码，并不关心它是怎么来的，你甚至可以用16进制编辑器自己写一份字节码，前提你对它的结构非常的清楚，你也可自己设计一门语言，然后让这门语言编译成字节码，运行在jvm平台上，以作为你的毕业设计或者开源项目。

<!--more-->

## 字节码

  上面一段中我提到过"一个非凡的公司去设计一款非凡的程序平台，Java在诞生之初，JVM和Java语言的规范就是分开来的",你可能会怀疑句话，因为一个程序语言编译成一个中间载体，完成这个过程时就顺便实现了所谓的**语言无关性**。实际上高超的设计团队，并不会为了迎合某种语言的需求去设计一种中间载体，而是在这个中间载体的语义范围之内去设计一门能提高生产力的分工度的语言。

  一言蔽之：“**字节码不是但但为了java而设计的，字节码有着比java更强大的语义**”。比如对于函数重载，但但是返回值不同在java语言中是不能判定为不同函数的，但是在class文件中则不然，class文件中只要一个类中方法的描述字段不相同，那么他们就是两个方法。虽然在具体语言上实现这一语义，这会带来新的运行时的麻烦。

## Class文件

  **Class文件对应着唯一一个类或接口的定义信息**，注意这里有几个重点，首先是**唯一一个**，第二是**类或接口**，第三点是**定义信息**

  Class文件，并不一定是指磁盘上的具体文件，它也许是动态生成的比如由CGlib生成，也可能是一个JSP文件生成，也有可能是一段网络流直接读取到内存里，但是不管它怎么样来，这段字节流都符合jvm规范里定义的类的结构，它的基础单位就是字节，各个项目数据严格按照规范排列在这段字节数据中，没有空隙存在，也没有分隔符存在，如果一个数据项不能以8位字节来表示，那么就用若干个字节，以高位在前方式来表示。

  从高级语言编译成字节码的那一刻开始，**在Class文件中每一个数据项都有它存在的意义**，当并非都是运行时必须的，因为**类的属性表（后面会介绍）**要求更为宽泛一些，存在一些如LineNumberTable,LocalVariableTable这样的属性是用来帮助我们编程和调试期间能够确定错误行行号，以及避免局部变量显示的名称是arg0,arg1这样没有意义的符号。**这些运行期不必要的数据项都可通过编译时使用特别的参数来拒绝生成**，但是我建议不要这么做，因为会导致别人很难复用和审查你的代码。

  Class文件格式并不是键值型的（属性表内属性除外），对数据的区分完全是看顺序和偏移。

### Class文件的组成

  要搞清楚什么组成了Class文件，首先要清楚的明白宽泛的讲，Class文件里只有两种数据类型：

1.  基本数据类型：**无符号数**，包括了u1,u2,u4,u8几种分别代表了**不同字节数的无符号数**，这些无符号数用来描述**数字**和**索引引用**,并且可以按照**缩略UTF-8**来表示字符串。这里缩略和普通的UTF-8编码的具体区别大家可以看卡wiki上这段话：

        ```
        The Unicode strings, despite the moniker "UTF-8 string", are not actually encoded according to the Unicode standard, although it is similar. There are two differences (see UTF-8 for a complete discussion). The first is that the codepoint U+0000 is encoded as the two-byte sequence C0 80 (in hex) instead of the standard single-byte encoding 00. The second difference is that supplementary characters (those outside the BMP at U+10000 and above) are encoded using a surrogate-pair construction similar to UTF-16 rather than being directly encoded using UTF-8. In this case each of the two surrogates is encoded separately in UTF-8. For example, U+1D11E is encoded as the 6-byte sequence ED A0 B4 ED B4 9E, rather than the correct 4-byte UTF-8 encoding of F0 9D 84 9E.
        ```

2. 复合数据类型：**表**，“表”可由**其他表**和**无符号数**组成，你可以把Class文件整个看成一张大表，里面有无符号数和许多小表，小表里又有小表，这些表设计者都以"_info”为结尾来命名，但是这些名称并不会体现在class文件里面，这里再次声明一遍，class文件格式没有键（属性表内属性除外）这种东西，完全是靠偏移来确定这个字段是什么，以及某个字段在哪里。

整个Class文件结构如下表顺序所示：

| 类型             | 名称                  | 数量                    |
| -------------- | ------------------- | --------------------- |
| u4             | magic               | 1                     |
| u2             | minor_version       | 1                     |
| u2             | major_version       | 1                     |
| u2             | constant_pool_count | 1                     |
| cp_info        | constant_pool       | constant_pool_count-1 |
| u2             | assess_flags        | 1                     |
| u2             | this_class          | 1                     |
| u2             | super_class         | 1                     |
| u2             | interfaces_count    | 1                     |
| u2             | interfaces          | interface_count       |
| u2             | fields_count        | 1                     |
| field_info     | fields              | fields_count          |
| u2             | methods_count       | 1                     |
| method_info    | methods             | methods_count         |
| u2             | attributes_count    | 1                     |
| attribute_info | attributes          | attributes_count      |

下面我们来拆解这张表，但是我们把比较特别的attribute部分，也就是最后两行另外以一篇新的文章来讲。

#### magic,minor/major_version

class文件打头的6个字节就是**魔术**，**次版本号**以及**主版本号**。

**魔术固定为：0xCAFEBABY**，这个字段唯一的目的就是类加载时验证阶段确定这个类是不是一个class类

**次版本号**和**主板本号**确定这个class文件所支持最低的JDK版本,高版本的JDK能向下兼容，如果JVM检测到版本标识超过了自己支持的版本，就拒绝执行。

比如，你把一个JDK7编译的class文件放到基于JDK6的项目中运行，就可能得到unsupporter major.minor 51.0异常。

jdk7的版本号即51，jdk的版本号是从45开始（1.0和1.1都是45，小版本号不同），1.2开始每一个大版本号加1，jdk1.2是46,1.6是50，1.8是52。

#### 常量池

接下来常量池，由于每个类中的常量不是固定的，所以这部分数据由2个部分组成

1. constant_pool_count，它是一个数值量，用来指定常量词中包含了多少个常量，但**这个数值量不是从0开始计数的，这点需要特别注意，它从1开始计数，1表示没有任何常量，2表示有1个常量**，然后依次后推，至于为什么不从0开始，因为后面的一些表中的引用字段，可能会存在不指向任何常量的情况，这时候就用0做为引用对于常量池的偏移。

2. 即  constant_pool_count-1个cp_info，cp_info是常量项的表结构，它有14种类型（截止jdk1.7），每种类型的常量项的表结构都不同，但是这种表结构的开头都是一个u1类型的tag字段，来指定不同的数值标识这个表结构是哪种类型。

   这14种类型你可以查看wiki：https://en.wikipedia.org/wiki/Java_class_file,里面的The constant pool项目就很清楚的说明了14种类型以及他们对应的tag的值。

   比如对于其中的一种CONSTANT_Class_info表结构：

| 类型   | 名称         | 数量   |
| ---- | ---------- | ---- |
| u1   | tag        | 1    |
| u2   | name_index | 1    |


   这里的tag的值肯定是0x07,标识它的类型为CONSTANT_Class_info，接下来name_index标识了一个索引值，该值指向的是一个常量词中的CONSTANT_Utf8_info结构，这个结构中有该类的全限定名字符串。现在我们来看看这种结构：

| 类型   | 名称     | 数量     |
| ---- | ------ | ------ |
| u1   | tag    | 1      |
| u2   | length | 1      |
| u1   | bytes  | length |

   对应于CONSTANT_Utf8_info的tag值为0x01,然后有一个16位数指定这个utf-8字符串所占的字节数，所以你可以推测一下你在java里直接以赋值形式构造一个所需空间大于64k的数是不可能的，接下来就是一个**缩略utf-8编码**所代表的字符串的字节流了，它的长度是上面给定的length。

   #### 访问标志位

接下来一个u2类型的access_flags字段标识这个类的访问标志位，用来表示该类是不是被abstract等关键字修饰，是不是一个注解或者接口，是否是用户代码产生，是否是一个枚举类型等等，我们知道访问标志只有有和没有的区别，于是一个u2的每一位就代表了一种访问标志有还是没有，一个u2字段可以代表16种标志位，但是实际上只用了其中的8个。

你可以从[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4)，里的4.1小结查看这些标志位的具体信息，这个文档是Java se8的。

   #### this_class,super_class,interface索引集合
| 类型   | 名称               | 数量              |
| ---- | ---------------- | --------------- |
| u2   | this_class       | 1               |
| u2   | super_class      | 1               |
| u2   | interfaces_count | 1               |
| u2   | interfaces       | interface_count |

   ​	它们通通指向的一个常量词中类型为CONSTANT_Class_info的常量，我们上面介绍过这个常量的具体结构，它有一个name_index指向一个CONSTANT_Utf8_info常量，这个常量中包含了一段字符串，具体在这里这段字符串是完整的类名。诸如：java/lang/Object。

   ​	在java中除了Object外，所有的类都有父类，字节码中表现也是一样的。

   #### 字段和方法表集合

| 类型          | 名称            | 数量            |
| ----------- | ------------- | ------------- |
| u2          | field_count   | 1             |
| field_info  | fields        | fields_count  |
| u2          | methods_count | 1             |
| method_info | methods       | methods_count |

   字段表和方法表的结构每个字段的功能上几乎一样，**但是实际上在access_flags的取值上是不同的，另外在属性表中它们也可能有不同的属性**。表结构如下：

   ```
   /*
   这里只列出了方法表其实字段表和它一模一样，只是位置一前一后，从而vm可以判断出谁是方法表，谁是字段表
   */
   method_info {
       u2             access_flags;
       u2             name_index;
       u2             descriptor_index;
       u2             attributes_count;
       attribute_info attributes[attributes_count];
   }
   ```

   1.访问位：不再赘述，和类的访问位功能和形式上差不多，具体有哪些访问位可以参考[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4)

   中4.5和4.6小结查看。

   2.name_index指向一个常量CONSTANT_Utf8_info，标识了方法或者字段名的**简单名称**，这里的简单名称指的是没有类型和参数修饰的名称，比如"inc()"->"inc"

   3.descriptor_index描述字段，这个字段有点复杂

**字段的描述：**

   	首先对于除了long基本类型和void，都用首字符的大写来表示，比如void->V,int->I,double->D,long用J表示。

   ​	为什么long用J表示的原因是因为所有引用类型都用L作为开头表示，引用类型表示的方式为L+类型权限定名，比如对于java.lang.Object->Ljava/lang/Object

   ​	对于数组一个维度就在类型符号面前加一个[,比如对于int[]->[I,对于String[][\]->[[Ljava/lang/String。

**方法的描述**

   ​	有了上面的基础，我们可以来介绍一下方法的descriptor_index字段，对于方法

   ```
   Object m(int i, double d, Thread t) {...}
   ```

   编译器把它转换为一个描述为：

   ```
   (IDLjava/lang/Thread;)Ljava/lang/Object;
   ```

   可以看到即使是在java中**不能重载的同名同方法表不同返回值的方法**实际上它们的描述符是不同的，实际上在class类中，这种**不能重载的同名同方法表不同返回值的方法**是可以存在的，但是运行期如何判断无接受值的方法调用到底调用的是什么方法呢?这是个巨大的问题。

   方法表和属性表中都不会列出从父类继承来的方法和字段，除非是自己在代码里重写了，但是会有你用javap去解析一个class文件时可能会发现没有在你代码里出现过的方法出现在了方法和字段表里，比如\<init>和\<clinit>方法，它们是实例的构造器和类的构造器，再比如对于内部类，我们可以访问外部类的字段和方法，编译器会直接给我们在内部类属性表插一个指向外部类实例的字段。

   **就如我上面所说，对于类，方法和字段三种表中都存在的属性表，我在下篇文章介绍，这个表比较复杂，也能解开你包括，java里方法代码去哪儿了，被设为常量的类变量的赋值的值去哪儿了的疑问。**