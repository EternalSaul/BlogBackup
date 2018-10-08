---
title: 'Practice of Basic Java Skills:非阻塞I/O (4):通道'
date: 2017-07-23 08:39:30
toc: true
tags:
 - Java
 - 非阻塞I/O(NIO)
---

  上次我们了解了非阻塞I/O中用于数据读写的缓冲区对象，现在我们来看看通道，通道有很多类，层次结构复杂，幸好网络编程中我们不会使用那么多类，和缓冲区一样有一些类是我们经常使用的，它们分别是SoketChannel、ServerSocketChannel以及DatagramChannel,从名字你就可知道最后一个属于UDP协议。通道将缓冲区的数据块移入或移出到各种各样的I/O源。

## SoketChannel

###   建立链接

  我们在前面的客户端和服务器例子中好像不止一次的使用到了这个类，你知道我们可以用它来实现对TCP Socet的读写，使用方法跟一般的Sokect很类似，但是SoketChannel是一个抽象类，你不能直接new一个对象，我们需要调用它的静态工厂方法open来创建一个SoketChannel对象。

<!--more-->

```java
@Test
public void test10() {
  try {

    //该方法会阻塞，直到链接建立
    SocketChannel.open(new InetSocketAddress("8.8.8.8", 80));

    //该方法不立即链接，所以不会阻塞，后续需要用connect来链接
    SocketChannel open = SocketChannel.open();

    /*使用无阻塞方式链接*/
    open.configureBlocking(false);

    /*无阻塞方式链接的话，connect会立即返回，OS在底层负责链接*/
    open.connect(new InetSocketAddress("8.8.8.8", 80));

    /*
    * 非阻塞方式在使用连接前必须调用finishConnect方法
    * 以此来确定是否可以链接
    * 如果已经链接返回true，正在链接则返回false
    * 如果链接失败，则抛出一个异常
    */
    open.finishConnect();


  } catch (IOException e) {
    e.printStackTrace();
  }
}
```

  利用上面的代码，我们知道如何获得一个链接对应服务器服务的通道对象，并知道以confiureBlocking来将其设置为非阻塞模式。

### 读取和写入

  建立一个链接而不通讯是毫无作用的，我们上篇文章里提到过SocketChannel只接受ByteBuffer，于是你可以猜到我们要操控通道读写缓冲区，首先得有个ByteBuffer。

  首先我们来看read方法，这个方法在第一、二片文章我们就接触过了，read方法接受一个或多个ByteBuffer对象，或者还要加上它的偏移值和要读的长度。




| 方法名                                      |
| ---------------------------------------- |
| long read(ByteBuffer[] dsts)             |
| long read(ByteBuffer[] dsts,int offset,int length) |
| int read(ByteBuffer dst)                 |

 在非阻塞状态下read(ByteBuffer dst)中，通道会尽可能用数据填充缓冲区，然后返回填充进去的字节数。如果通道流到了末尾，没有数据了，就返回-1。在非阻塞状态下如果通道流没有字节可以读取，就返回0。

  read(ByteBuffer[] dsts)填充所有缓冲区，直到最后一个缓冲区的remaining为0，而read(ByteBuffer[] dsts,int offset,int length)则是填充从offset开始的length个缓冲区,把一个源的数据填充到多个缓冲区，通常叫做散布（scatter）。

  你可以猜测到，写入通道源也有与读类似的方法

| 方法名                                      |
| ---------------------------------------- |
| long write(ByteBuffer[] dsts)            |
| long write(ByteBuffer[] dsts,int offset,int length) |
| int write(ByteBuffer dst)                |

   你可能忘记了我们之前介绍的flip方法，它会把limit设置为当前的position而position设置为0，写入时我们可能需要这个方法。将多个缓冲区写入一个通道源称之为聚集（gather）。以上三个函数的用法和解释也和read差不多，不过现在我们进行的是缓冲区的排空操作，而非填充。

### 关闭

  我们在通讯结束后，需要将通道关闭，释放其占用的端口号及本地资源，Channel接口中声明了两个方法，即close和isOpen，相信看这两个方法名你大概知道什么意思了，另外由于实现了AutoCloseable，所以我们可以向处理流那样，对通道使用try-with-resources。

## ServerSocketChannel

服务器通道，我们在之前也接触了，该类和ServerSocket一样用作接受入站请求。

```java
@Test
void test11() throws IOException{

  //工厂模式获取一个对象
  ServerSocketChannel server = ServerSocketChannel.open();

  //设置为非阻塞模式
  server.configureBlocking(false);


  /*
  * 绑定一个套接字地址，Java 7以后才实现直接绑定
  * Java 7之前必须先获取ServerSocket,如：
  * server.socket().bind(new InetSocketAddress(8888));
  */
  server.bind(new InetSocketAddress(8888));

  //等待入站请求，在非阻塞模式下会立刻返回null,所以在非阻塞模式下要配合选择器使用
  SocketChannel socket = server.accept();


}
```

## 工具类Channels

  Channels提供了一系列方法提供了传统I/O与通道之间的转换，这种转换是有理由的，有时候你可能想用一般的I/O流和通道并用来改善性能。或者有时候你的客户端或者用了阻塞服务器以非阻塞形式彼此通讯，但要以一般的I/O形式在本地处理数据，比如很多Json解析的API，只提供了接受一个字符串或者I/O流，并不能解析一个缓冲区。

| 方法                                       | 简绍                 |
| ---------------------------------------- | ------------------ |
| ReadableByteChannel newChannel(final InputStream in) | 接受一个输入流返回一个只读的通道   |
| WritableByteChannel newChannel(final OutputStream out) | 接受一个输出流返回一个一个可写的通道 |
| InputStream newInputStream(ReadableByteChannel ch) | 接受一个可写的通道返回一个输入流   |
| OutputStream newOutputStream(final WritableByteChannel ch) | 接受一个可写的通道返回一个输出流   |

 以上只是Channels工具类中几个最基础的方法，它还可以完成对Reader和Writer的转化，还能转化异步通道。

## 异步通道

 异步通道在Java 7后提供，AsychronousSocketChannel和AsychronousServerSocketChannel

对AsychronousSocketChannel进行读写会立刻返回，所做的读写操作会由一个Future和CompletionHanlder处理，你可以用返回的Future或者传入的CompletionHanlder来获取结果。

| 方法                                       |
| ---------------------------------------- |
| Future &lt;Integer&gt; read(ByteBuffer dst) |
| void read(ByteBuffer dst,A attachment,CompletionHandler &lt;Integer,? super A> handler) |
| &lt;A&gt; Future &lt;Integer> write(ByteBuffer src); |
| &lt;A&gt; void write(ByteBuffer src,A attachment,CompletionHandler &lt;Integer,? super A> handler) |
| Future &lt;Void> connect(SocketAddress remote); |

对AsychronousServerSocketChannel进行connect和accept操作也会异步执行并返回Future

| 方法                                       |
| ---------------------------------------- |
| Future &lt;AsynchronousSocketChannel> accept() |
| &lt;A&gt; void accept(A attachment,CompletionHandler &lt;AsynchronousSocketChannel,? super A> handler) |

  如果你想想看，为什么会有异步通道的需求呢？假如一个应用面临了网络拥塞和复杂处理两种问题，等待一个网络请求需要很久，处理一个网络请求的数据又需要很久，比如太空和地面的通信，比如P2P下载程序的多任务下载，那么如果只是进行非阻塞IO的话，还是不行的，因为可能你接收到数据前需要做其他的操作需要花费很久的时间，线程还是会阻塞，这时我们可以用线程池加callable的方法来异步的进行网络操作，当然现在我们有了异步通道就能更简单的完成这一个工作。

```java
@Test
void test12() throws IOException, InterruptedException, ExecutionException{
  AsynchronousSocketChannel open = AsynchronousSocketChannel.open();

  //异步的进行网络连接
  Future<Void> connect = open.connect(new InetSocketAddress("8.8.8.8", 9999));

  //做其他工作...

  ByteBuffer src=ByteBuffer.allocate(100);
  ByteBuffer dst=ByteBuffer.allocate(100);

  //将其他工作得来的内容异步通过网络传递出去
  Future<Integer> write = open.write(src);


  //异步读取服务器数据，通道允许同时读写
  Future<Integer> read = open.read(dst);

  //做其他工作...

  if(read.get().intValue()!=0){
    //处理网络数据返回
  }

  //收尾工作
  open.close();
}
```

