---
title: 'JVM(2):类文件结构（下）'
date: 2017-11-29 11:55:41
toc: true
tags:
 - JVM
 - 学习笔记
---

  如上篇文章所说，这篇文章我介绍一下类文件结构里比较复杂的**属性表**，属性列表这项数据可能存在于字段表，方法表，和类文件本身结构以及Code属性表中，我们可以 参照面的类C结构，来确认这一点。

```
/*
类文件结构
*/
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    /*属性表*/
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}

/*方法表（和字段表结构相同）*/
field_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    /*属性表*/
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

<!--more-->

  属性表的结构要求比之class文件中的其他数据项要弱的多，其他数据项由于是完全按照偏移来确定的，所以顺序必须要是JVM规范给定的顺序，而这个属性表不一样，属性表相当于一种**键值结构**：

```
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

  这个构造是属性表的基本构造，由于有很多种不同的属性表，所以对于第三个数据项这个info（相当于值），可能内部被划分为了很多个其他的数据项，现在我们暂时先放下这个疑问，上面我说属性表相当于一种键值结构，attribute_name_index相当于键，它指向了一个CONSTANT_Utf8_info结构，代表了这个属性的名字。

  有了这个键值的概念我们就不难理解为什么属性表并不需要按一定顺序排列每一个属性，因为JVM是按照这个attribute_name_index指向的名字来确定这个属性是什么属性，它的info的具体结构是怎么样的，所以你如果自己实现JVM和编译器，你可以自己定义属性名（不能与已有的属性重复），然后为了提供解析操作的实际代码，对于JVM来说，如果碰到它不认识的属性名，它就选择直接忽略了。

  JVM规范给出了一些预定义的属性，要求java虚拟机应当能识别这些属性，在[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7)（Java虚拟机规范 SE8）中，我们可以从4.7节得到所有的23项属性的详细信息。

首先我们来看看5项关乎到class文件能不能被JVM正确解释的属性:

### ConstantValue

ConstantValue的作用是通知虚拟机自动为静态变量赋值（即被static属性标识的字段），只有被static标识的字段虚拟机才查看ConstantValue属性，否则即使有该属性也被忽略，静态变量其实有两种赋值方式：

**clinit赋值**

一种就是非常量型静态变量，如`static int k=1`，这种变量是在类加载的初始化阶段，执行clinit函数来进行赋值的，clinit函数是编译器自动生成的，赋值字节码指令已经被写在了里面。

**ConstantValue赋值**

对于如**基本类型和String**（其他引用类型不支持），如果字段被static和final同时修饰，这种情况在编译期你就可以确定它是哪个值了，所以直接生成ConstantValue属性来进行初始化（编译期）。

以上两种种赋值的条件差异是javac自行决定的，如果你自己实现一个编译器也可以对没有final修饰的static变量进行编译期初始化，JVM规范（SE 8）上是这么说的：

```
/*
这段话里没有提到final字段，那么我们可以猜测javac对只有static修饰的字段，根本不生成ConstantValue属性
*/
If the ACC_STATIC flag in the access_flags item of the field_info structure is set, then the field represented by the field_info structure is assigned the value represented by its ConstantValue attribute as part of the initialization of the class or interface declaring the field (§5.5). This occurs prior to the invocation of the class or interface initialization method of that class or interface (§2.9). 
Otherwise, the Java Virtual Machine must silently ignore the attribute. 
```

```
ConstantValue_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    /*info部分*/
    u2 constantvalue_index;
}
```

  具体你可以看到constantvalue_index是一个指向常量池的u2类型，而常量池中字面量常量只有基本类型和String，所以对于其他引用类型，你根本不可能用ConstantValue这种东西来定值。

### Code

Code属性只**可能**存在于method_info中，抽象方法之类的没有方法体的当然就没有Code属性这种东西了，具体的code属性如下表所示：

```
Code_attribute {
    u2 attribute_name_index;//固定指向值为Code的一个CONSTANT_Utf8_info
    u4 attribute_length;
    /*info部分*/
    u2 max_stack;//操作数栈的最大深度
    u2 max_locals;//局部变量表的存储空间，单位Slot(4个字节)
    /*代码部分*/
    u4 code_length;
    u1 code[code_length];
    /*显示异常处理表部分*/
    u2 exception_table_length;
    exception_info exception_table[exception_table_length];
    /*属性表部分*/
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}

exception_info{
	u2 start_pc;
	u2 end_pc;
	u2 handler_pc;
	u2 catch_type;
}
```

**字节码部分**

code_length和code[]一起构成了方法体被编译之后的字节码指令流,jvm的字节码指令是单字节的基于栈的字节码，单字节意思就是最多256条，目前只用了200多条，详细是怎么样的，我以后会写关于**字节码执行引擎**的文章，详细讨论这个问题。

**显示异常处理表部分**

显示异常处理表部分与try/catch语句有关，我们可以看到exception_info这个结构，指示了字节码从start_pc行到end_pc行（不包含），出现了catch_type类型或者其子类的话，就转跳到handler_pc行进行处理。

**属性表部分**

code中的属性表中的属性一般都是运行时非必须的，但这些熟悉往往会在程序员编程和调试时有着巨大的作用，比如我们编程时的方法的参数的参数名，或者调试时出现异常标识的异常出现的行号等等，主要有下面几种:

1.[LineNumberTable](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.12)  //用于标识java代码中实际行号和字节码偏移量的对应关系

2.[LocalVariableTable](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.13)  //用于描述局部变量表中变量与java源码中定义变量的对应关系

3.[LocalVariableTypeTable](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.14)  //1.5引入泛型后用来代替LocalVariableTable

4.[StackMapTable](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.4)	//用于做类加载阶段的类型验证，下面我们会详细介绍它。

以上属性的具体表现形式我已经表示了官方文档超链接，大家可以自己去看看，或者参考一下周志明的[《深度理解java虚拟机》](https://book.douban.com/subject/24722612/)，里面有一个简单的描述。

### StackMapTable

如上面所说，StackMapTable是一个只会存在于Code中的属性，它是在jdk 1.6后被引入的，用于代替以前开销极大的通过数据流分析的类型推导。如果一个Code的属性表里不包含StackMapTable属性，那么虚拟机默认它包含了一个隐式的stack map attribute，相当于一个StackMapTable但是number_of_entries为0。

```
StackMapTable_attribute {
    u2              attribute_name_index;
    u4              attribute_length;
    u2              number_of_entries;
    stack_map_frame entries[number_of_entries];
}
```

**每一个stack_map_frame都显示或者隐式的指明它所应用的字节码偏移量，以及该偏移量对应的一组局部变量和操作数栈项目的验证类型（Verifiacation Types）。**

我还是在这里详细说一下这个stack_map_frame，以及它内部可能会包含的标识验证类型的另一个结构verification_type_info,如下:

```
/*
这里用union标识的意思是这种结构是C语言中类似union的结构，就是说这里面的项目只会出现一项
*/
union stack_map_frame {
    same_frame;
    same_locals_1_stack_item_frame;
    same_locals_1_stack_item_frame_extended;
    chop_frame;
    same_frame_extended;
    append_frame;
    full_frame;
}

union verification_type_info {
    Top_variable_info;
    Integer_variable_info;
    Float_variable_info;
    Long_variable_info;
    Double_variable_info;
    Null_variable_info;
    UninitializedThis_variable_info;
    Object_variable_info;
    Uninitialized_variable_info;
}
```

#### verification_type_info

verification_type_info用于指示一个或者两个**位置**（如果是64位类型则占2个slot）的类型，**位置指的是一个局部变量或者一个操作数栈中的项目**，验证类型就是用这个类似于union结构的verification_type_info来表示的，由一个单字节的tag来表示这个union里用的是哪一个项目，在tag后跟着零个或者多个字节，用来表示更多的信息。具体的union可能表示的项目有9个，我们这里列举几个，其余的你可以参考[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.4)：

```
/*
如果verification_type_info中的tag是5，那么它指示对应位置的验证类型是null
*/
Null_variable_info {
    u1 tag = ITEM_Null; /* 5 */
}


/*
如果tag是7,那么它表示该位置验证类型是由常量池的index为cpool_index的CONSTANT_Class_info结构指示的
*/
Object_variable_info {
    u1 tag = ITEM_Object; /* 7 */
    u2 cpool_index;
}

/*
如果tag是8，则说明这个位置的验证类型还没有初始化，offset项目指示了包含该StackMapTable属性的Code的code数组中的偏移，该偏移对应的字节码是一个new指令，将会创建存储在该位置的对象
*/
Uninitialized_variable_info {
    u1 tag = ITEM_Uninitialized; /* 8 */
    u2 offset;
}

/*
这里还存在一个Top_variable_info类型可能大家看名字也看不明白，我在这里说一下
对于Long和Double这两种类型来说，他们是64位的，所以是占2个slot，那么他们该怎么确定类型呢？
如果第一个位置是一个局部变量，那么
它一定不是最大index的那个局部变量，它的下一个变量的验证类型是top
如果第一个位置是一个操作数栈项目，那么
它一定不是栈顶元素
它上面的那个元素（更靠近栈顶的）的验证类型是top
*/
```

#### stack_map_frame

  接下来我们来看一下stack_map_frame这个结构，中文就把它叫做**栈映射帧**，它同样是类似于union的，然后它也有一个单字节的frame_type数据项来指示这union到底表示的是哪一种项目,一共有7种可能上面已经列出来了。

  每一个**栈映射帧**都会依赖于前一个frame中的一些语义，第一个**栈映射帧**其实是隐式的，这个隐式的frame会被类型检查器（type checker）根据Code属性的所在的方法描述符给计算出来，在上面的entries[]中的0号元素其实是第二个,所以上面有提到过`如果一个Code的属性表里不包含StackMapTable属性，那么虚拟机默认它包含了一个隐式的stack map attribute，相当于一个StackMapTable但是number_of_entries为0。`

  栈映射帧所用的字节码偏移是通过它里面指定的offset_delta计算出来的，并且offset_delta+1还会作为前个帧的字节码偏移，除非前一个帧是首帧（initial frame），这种情况下字节码偏移就是首帧指定的offset_delta。

  为什么使用**偏移增量**（即offset_delta），而不是存储实际的字节码偏移量，是因为根据定义，我们可以确保栈映射帧处于正确的排序顺序。

我这里同样拿出几种来介绍，其余的可以参考[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.4)。

```
/*
当frame_type在0-63之间，就代表了这个frame中存储的是一个same_frame，它表明这个frame所代表的局部变量与前一个frame相同而且操作数栈为空，frame_type的数值就是offset_delta
*/
same_frame {
    u1 frame_type = SAME; /* 0-63 */
}

/*
只有当fram_type为247的时候才确定类型为same_locals_1_stack_item_frame_extended，它表明这个frame所代表的局部变量与前一个frame相同而且操作数栈为空，但是明确给出了offset_delta，然后后面跟着类型数据项，上面我们已经介绍过了
*/
same_locals_1_stack_item_frame_extended {
    u1 frame_type = SAME_LOCALS_1_STACK_ITEM_EXTENDED; /* 247 */
    u2 offset_delta;
    verification_type_info stack[1];
}

/*
这个类型中，它表明这个frame所代表的局部变量与前一个frame相同而且操作数栈为空，除非最后的k个局部变量缺失，k等于251-frame_type。
*/
chop_frame {
    u1 frame_type = CHOP; /* 248-250 */
    u2 offset_delta;
}
```

### Exceptions

异常属性和上面code属性中提到的显示异常处理表不同，它指示了方法签名中throws后面标识的可能会抛出的异常

```
Exceptions_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 number_of_exceptions;
    u2 exception_index_table[number_of_exceptions];
}
```

这个属性比较简单，exception_index_table[]的每个u2项目都指向常量池中一个标识了异常类型的CONSTANT_Class_info型的常量。

### BootstrapMethods

BootstrapMethods属性在jdk 1.7后增加，它仅存在于类文件属性表里，目的是为了保存invokedynamic指令引用的引导方法限定符。

```
BootstrapMethods_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 num_bootstrap_methods;
    bootstrap_method bootstrap_methods[num_bootstrap_methods];
}

bootstrap_method{   
	u2 bootstrap_method_ref;//指向常量池中的一个CONSTANT_MethodHandle_info
	u2 num_bootstrap_arguments;
	u2 bootstrap_arguments[num_bootstrap_arguments];
}
```

bootstrap_arguments[]中的每一个项目，都是对常量池中的以下几种结构之一的索引

1.CONSTANT_String_info, CONSTANT_Class_info
2.CONSTANT_Integer_info, CONSTANT_Long_info
3.CONSTANT_Float_info, CONSTANT_Double_info
4.CONSTANT_MethodHandle_info
5.CONSTANT_MethodType_info

然后我们来看看在Java平台下类关乎到文件能否正确解析的属性，一共有12个，这里我也说几个：

1.InnerClasses
2.EnclosingMethod
3.Synthetic
4.Signature
5.RuntimeVisibleAnnotations
6.RuntimeInvisibleAnnotations
7.RuntimeVisibleParameterAnnotations
8.RuntimeInvisibleParameterAnnotations
9.RuntimeVisibleTypeAnnotations
10.RuntimeInvisibleTypeAnnotations
11.AnnotationDefault
12.MethodParameters

### InnerClasses

首先看看这个InnerClasses,看名字就知道，这个属性就是记录内部类的，准确来说它记录了内部类和宿主类之间的关系。

```
InnerClasses_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 number_of_classes;
    inner_class_info classes[number_of_classes];
}

inner_class_info{   
	u2 inner_class_info_index;
	u2 outer_class_info_index;
	u2 inner_name_index;
    u2 inner_class_access_flags;
}
```

  对于inner_class_info来说，开始4个字节就是两个个指向常量池中某个CONSTANT_Class_info的常量索引，分别代表了内部类和宿主类的符号引用。

  inner_name_index是一个CONSTANT_Ut8_info常量索引，这个常量即内部类类名，如果是**匿名内部类**，则这项值为0，即没有引用任何常量（上一篇文章中说过这个问题）。

  最后的inner_class_access_flags是访问标志位，这个具体有哪些标志可以参考[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.6)。

### Signature

1.5之后，为了支持擦出法实现的伪泛型，jvm引入Singature。如果一个类型，接口，构造器，方法，或者字段（field）的声明中包含了类型变量或者参数化类型，则javac必须为他们添上signature属性。

```
Signature_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 signature_index;
}
```

signature_index是一个指向常量池CONSTANT_Utf8_info的符号引用，里面包含的字符串表示类，方法或者字段签名，具体取决于Signature属于哪个表结构，签名字符串的表述形式类似于我们上面介绍过的描述符，具体可以看[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.9.1)。

我们运用java的反射api可以获取到泛型的信息，实际上就是来源于Signature属性。

### MethodParameters

最后我们来看看方法参数表,他只存在方法表(method_info)中，且每个方法表最多只能有一个。

```
MethodParameters_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 parameters_count;
    parameter_info parameters[parameters_count];
}

parameter_info{  
	u2 name_index;
	u2 access_flags;
}
```

对于parameter_info来说

name_index是一个指向CONSTANT_Utf8\_info的符号引用，它的值如果为0，那么表示这是一个无名的形参，如果非0则表示了一个**有效的**参数限定名称。

access_flags,有三种值

0x0010 (ACC_FINAL),这个大家都用过，表示这个参数是不可变的

另外两种可能大家没有接触过

0x1000 (ACC_SYNTHETIC)和 0x8000 (ACC_MANDATED)

ACC_SYNTHETIC表示这个参数不是由用户代码生成的，而是由比如编译器这样的东西加进去的，比如this这个参数。

ACC_MANDATED则表示这个形参是隐式声明在代码中的

还有6项属性不是运行时必须的，但是我们在编写代码时很多信息来源于他们：

**SourceFile**

  在类属性表，指定该类对应的源文件名

**SourceDebugExtension**

这个属性用于提供额外的调试信息，有些字节码的来源并不是.java文件，比如可以来自JSP，对于JSP的调试我们是无法通过但但从LineNumberTable来定位到JSP行号和堆栈的对应关系的，于是1.6后引入了该属性，以至于新的调试标准机制（JSR-45中定义）可以用它来作为调试信息。

**下面三个属性，我在介绍Code属性时已经说过了**

**LineNumberTable**
**LocalVariableTable**
**LocalVariableTypeTable**

**Deprecated**

  Deprecated指定某个类，字段或者方法不再推荐使用，即在源码中用@deprecated标识