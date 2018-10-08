---
title: 'Practice of Basic Java Skills:非阻塞I/O (3):缓冲区'
date: 2017-07-22 08:12:58
toc: true
tags:
 - Java
 - 非阻塞I/O(NIO)
---

  前面的两个例子的文章中我们都看到了缓冲区的身影，正如我在第二篇文章的开头第二段所说，缓冲区是实现NIO的重要手段，缓冲区也是JAVA-NIO API的基础部分，实际上随便想想也可发现，NIO中已经不再用会阻塞流（Blocking I/O）来写byte或基本类型进行输入输出，而取代它们用来输入输出的是利用通道读写缓冲区，在缓冲区里写字节或基本类型。

  与流基于字节不同：

   **1.通道是基于块的，它一次读写一个缓冲区的数据。**

   **2.通道和缓冲区可以支持同一对象的读写，而流则分为输入输出流。**

  你可以把缓冲区认为是一种特定元素的列表，有时候他以数组来实现，有时不是，除了boolean以外，每个Java的基本类型都对应了一个特定的Buffer子类，比如DoubleBuffer,IntBuffer，但是我们用的最多的还是ByteBuffer，光是看名字的话原因你应该很清楚了。

<!--more-->

## 抽象类Buffer

### 四个重要字段

  上一篇文章我们在很多地方都用了缓冲区的，flip这个方法，在这个方法之后我们将缓冲区写入通道，我们当时介绍的是它将ByteBuffer对象的position置为了0，相当于“绕回”。实际上该方法之前还做了一个操作就是把limit设置为了当前的position，实际上limit和position都是缓存区记录信息的关键部分，实际上这样的关键信息字段有四个，他们都来自于抽象类Buffer，上面提到的所有类型缓冲区的基类，这四个字段分别是：

**1.position**

​    代表缓冲区中将读取或写入的下一个位置，该位置从0开始计数，最大值即缓存区的大小

**2.capacity**

​    代表缓冲区的容量，即缓冲区可以保存元素的最大数目，容量值在创建缓冲区时设置，此后无法修改

**3.limit**

​    缓存区中可访问数据的末尾位置。这个字段限制了读/写可到达的最大的位置，即使缓存区的容量更大，但是却不能读写操作limit这个位置的数据

**4.mark**

​     缓冲区中客户端指定的索引，可以通过mark()来标记为当前位置，而用reset()可将当前位设置为所标记位置。

### Buffer中的常用方法

   读取缓存区是不会改变缓存区的数据的。程序只能调整position和limit的值来控制读取数据的开头和结尾。

   通过postion(),limit(),capacity()，三个方法我们可以读取当前对应的字段的值，而通过position(int),limit(int)两个方法则可以设置当前position和limit的值，并且得到buffer本身的返回值。

​    clear()方法将position设置为0，并将limit设置为容量，从而清空缓冲区。

​    rewind()将位置设置为0，但不改变限度。即可以重新读取缓冲区。

​    flip()方法我们已经接触过，它将limit设置为当前的position再将position设置为0。这样我们就可以读出或排空刚刚写入的数据。

​    remaining()方法返回缓冲区中当前位置与限度之间的元素数。
​    hasRemaining()则如果 remaining()大于0，则返回true

## 创建缓冲区    

  每种类型的缓冲区都有各自的几个工厂方法，比如我们在前两篇文章用到的allocate方法，就是其中之一。

### 分配

**1.一般分配**

  allocate(int)返回一个有指定固定容量大小的空缓冲区，position初始化为0，以该方法创建的缓冲区基于Java的数组实现，可以通过array()和arrayOffet()来访问对应的数组和此缓冲区的第一个元素在缓冲区数组中的偏移量。

**2直接分配（只限于ByteBuffer）**

  allocateDirect()方法不会为缓冲区创建后备数组。虚拟机会对以太网卡，核心内存或其他位置上的缓冲区使用直接内存访问，以此来实现直接分配的ByteBuffer。这种做法可以提升I/O的性能，但是创建和回收直接缓冲区的代价比较高，很耗时。

  **为什么直接内存更快？**

   这里的直接内存，只直接在OS管辖内的内存，利用这部分内存进行直接的操作，而省去了进行复制到JVM堆内存的操作。

  **为什么分配和回收更耗时？**

​    由OS管理的内存，并不是直接从JVM堆内存中划一片来使用，于是JVM必须向OS申请，相当于多做了这一步操作，回收时同理，不是有CG来回收，而是向OS提出申请。

### 包装

  如果你已经有了一个数据数组，你就不需要再分配一个新缓冲区，然后把该数组复制到这个缓冲区，这样太傻了，我们此时就应该用包装操作。

   wrap()接受一个缓冲区类型对应的数组，将这个数组作为其后备数组。修改数组的话同样会修改缓冲区，造成不确定性，所以对数组操作结束前不要包装数组。包装后position为0，而capacity和limit都为array.length。

```java
@Test
public void test4() throws IOException {
  //普通的分配
  ByteBuffer buffer=ByteBuffer.allocate(10);

  //直接分配
  buffer=ByteBuffer.allocateDirect(10);

  //包装数组
  byte[] bs="hello".getBytes();
  buffer.wrap(bs);
}
```



## 缓冲区的填充和排空

### 一般的填充和排空

 每向缓冲区**写入或读取**一个元素时,position相应的加1，请注意黑体，读写都会加1，position的定义是：**代表缓冲区中将读取或写入的下一个位置**，如果试图填充的数据超过容量，将会抛出BufferOverflowException异常。

  每次写完缓冲区后一定要记得使用flip方法，将指示读取的position的位置设置为0，使用get方法可以从缓存区内读取数据，而每次读取一个数据，position也会相应增加1，此时我们可以用hasRemaining来指示我们是否读到了limit。

### 绝对位置上的读取和修改

  另外我们还可以进行绝对位置上的修改和读取，利用绝对位置上的修改和读取，不会改变position的数值，但是如果试图访问一个超过limit的位置，则会抛出IndexOutOfBoundsException异常。

### 批量地填充和排空

get和put有重载方法，它们读入一个数组，还可以读入偏移和长度，进行批量的填充和排空操作，如果缓冲区没有足够的空间容纳put进来的数组，就会抛出BufferOverflowException异常，而如果缓冲区没有足够的数据来填充get提供的数组，那么就抛出BufferUnderflowException异常。

| 方法(以ByteBuffer为例)                     |
| ------------------------------------- |
| get()                                 |
| get(int index)                        |
| get(byte dst[],int offset,int length) |
| get(byte dst[])                       |
| put(byte b)                           |
| put(int index,byte b)                 |
| put(byte[] src,int offset,int length) |
| put(byte[] src)                       |

### 数据转换

  为什么在之前说ByteBuffer是用的最广的缓冲区呢？如果你对java的基本类型很熟悉的话，就知道int啥的基本类型都可以写为字节，你之前也许用过DataOutputStream/DataInputStream，现在你利用ByteBuffer同样可以完成对应任务，比如用getLong方法就可以将8个字节转换为一个long，而putLong则把一个long转换为8个字节存储于缓冲区，对应的还有绝对的填充和排空。

```java
@Test
public void test5() throws IOException {
  //普通的分配
  ByteBuffer buffer=ByteBuffer.allocate(100);
  buffer.putLong(4l);
  buffer.putDouble(4.5);
  System.out.println(buffer.getLong(0));
  System.out.println(buffer.getDouble(8));
}

/*
输出：

  4
  4.5
  
*/
```

### 视图缓冲区

通道只能对ByteBuffer进行读写，但是如果你知道一个Buffer是只包含固定的基本类型，那么完全可以把它转换为一个视图缓冲区(view buffer),对应了IntBuffer,DoubleBuffer等等，修改视图缓存区同时也会修改底层的Bytebuffer缓冲区，但是视图缓冲区有自己的position,limit,capaciy和mark四个基本字段。

```java
	@Test
	public void viewBuffer() throws IOException {
		//普通的分配
		ByteBuffer buffer=ByteBuffer.allocate(100);
         //转换为对应视图缓冲区
		CharBuffer CharBuffer = buffer.asCharBuffer();
		IntBuffer intBuffer = buffer.asIntBuffer();
		DoubleBuffer doubleBuffer = buffer.asDoubleBuffer();
		FloatBuffer floatBuffer = buffer.asFloatBuffer();
	}
```

## 缓冲区的压缩，复制和分片

### 压缩

  抽象方法compact方法，声明在每个特定的抽象类中，比如ByteBuffer,LongBuffer,因为它们返回值不同（可能是为了实现链式编程），当然Java中它们最后都提供了实现，比如在ByteBuffer的子类的HeapByteBuffer和DirectByteBuffer，以及他们各自的Read-only类型中有一个抛出异常的重写。

  compact方法会把当前的position和limit之间的所有元素，移动到缓冲区的开头，并且将position设置为0，将limit设置为capacity的置，并且将mark的值抛弃，然后开始移动元素。当元素复制完之后就把position正好为为总共拷贝的比特数。

```java
@Test
public void test6() {
  //普通的分配
  ByteBuffer buffer=ByteBuffer.allocate(100);
  buffer.putDouble(10, 4.55);
  buffer.position(10);
  buffer.limit(18);
  System.out.println("limit:"+buffer.limit());
  System.out.println("position:"+buffer.position());
  //压缩缓冲区
  buffer.compact();
  System.out.println("limit:"+buffer.limit());
  System.out.println("position:"+buffer.position());
}
/*
输出：

limit:18
position:10
limit:100
position:8

*/
```

  压缩缓冲区操作可以用来一段一段读取缓冲区中的数据，不断的把未读的数据压到缓冲区的开头，然后可以重已存在数据末尾写入，然后做一下flip操作输出又可以重头读。

### 复制

复制和压缩一样，声明在每个特定的缓冲区抽象类中，返回的也是对应特定的缓冲区实例，6种特定的缓冲区的复制方法都是duplicate()。

  dupilicate方法并不是完完全全复制一个一模一样的缓冲区，而是创建一个新的缓冲区共享那个被复制的缓冲区的内容，也就是说如果你改变其中一个缓冲区的内容另一个也被改变，但是如同视图缓冲区那样，新的缓冲区拥有自己的position,limit,mark，如果旧缓冲区是直接内存缓冲区，那么新的也一样，如果旧缓冲区是只读那么新的缓冲区也是只读的。

```java
@Test
public void test7() {
  //普通的分配
  ByteBuffer buffer=ByteBuffer.allocate(100);
  ByteBuffer duplicate = buffer.duplicate();
  duplicate.putInt(12);
  buffer.position();
  System.out.println("buffer:"+buffer.getInt());
}
/*
输出:

buffer:12

*/
```

### 分片

分片是复制的一个变形，声明在每个特定的缓冲区抽象类中，返回的也是对应特定的缓冲区实例，6种特定的缓冲区的复制方法都是slice()。

slice方法也创建一个新的缓冲区对象，但不同的是，它与原对象只共享一部分内容，新的buffer的内容从旧缓冲区的当前的position的位置开始，同样的这种共享关系修改之一会影响另一缓冲区，而position,limit和mark是独立的，新缓冲区的position置为0，capacity和limit将是旧对象的remaining值（由remaining方法返回，即缓冲区中当前位置与限度之间的元素数）。

与复制一样新的缓冲区将继承就缓冲区的read-only或者direct特性。

```java
@Test
public void test8() {
  //普通的分配
  ByteBuffer buffer=ByteBuffer.allocate(100);
  buffer.putDouble(10, 4.55);
  buffer.position(10);
  buffer.limit(18);

  System.out.println("remaining of old:"+buffer.remaining());

  ByteBuffer slice = buffer.slice();

  System.out.println("position of new:"+slice.position());
  System.out.println("limit of new:"+slice.limit());
  System.out.println("capacity of new:"+slice.limit());

}
/*
输出：

remaining of old:8
position of new:0
limit of new:8
capacity of new:8

*/
```

## 缓存区的比较和求哈希码

缓冲区实现了Comparable方法。

以下条件下，两个缓冲区相等：

**1.相同类型**

**2.remaining相同,即limit-position**

**3.相同相对position上的剩余元素彼此相等**

 缓冲区县相等不会考虑缓capacity,limit和mark，也不考虑缓冲区中position指示位置之前的元素，比如

```java
@Test
public void test9() {
  //普通的分配
  ByteBuffer buffer=ByteBuffer.allocate(100);
  ByteBuffer buffer2 = ByteBuffer.allocate(50);
  buffer.putDouble(8,8);
  buffer.position(8);
  buffer.limit(16);
  buffer2.putDouble(8.0);
  buffer2.flip();
  System.out.println(buffer.compareTo(buffer2));
  System.out.println(buffer2.equals(buffer));
  for(int i=0;i<8;i++)
    System.out.println(buffer.get(i+8)+"="+buffer2.get(i));

  //观察ByteBuffer的toString方法
  System.out.println(buffer);
  System.out.println(buffer2);
}

/*
输出：

0
true
64=64
32=32
0=0
0=0
0=0
0=0
0=0
0=0
java.nio.HeapByteBuffer[pos=8 lim=16 cap=100]
java.nio.HeapByteBuffer[pos=0 lim=8 cap=50]


*/
```

  hashCode方法是按照相等性来实现的，相等的缓冲区的散列吗相同。但每次改变缓冲区的数据，缓冲区的散列表都会相应改变。

  compareTo首先会对缓冲区中的剩余元素逐个比较，如果所有元素相等就相等，即返回0，如果有一对元素不相等，则返回这对元素的比较结果，**若比较时**一个缓冲区的remaining比另一个元素的remianing小，则认为小的remaining对应的元素小，并返回差值（调用方法的buffer的remaining减去作为参数的buffer的remianing），需要注意的是它并不会一开始就比较remaining

```java
//来自java 8中的ByteBuffer的compare源码    
public int compareTo(ByteBuffer that) {
  int n = this.position() + Math.min(this.remaining(), that.remaining());
  for (int i = this.position(), j = that.position(); i < n; i++, j++) {
    int cmp = compare(this.get(i), that.get(j));
    if (cmp != 0)
      return cmp;
  }
  return this.remaining() - that.remaining();
}

private static int compare(byte x, byte y) {
  return Byte.compare(x, y);
}
```

