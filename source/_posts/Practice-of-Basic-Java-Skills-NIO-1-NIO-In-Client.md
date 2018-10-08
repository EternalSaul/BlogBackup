---
title: 'Practice of Basic Java Skills:非阻塞I/O (1)——客户端非阻塞I/O'
date: 2017-07-20 15:54:23
toc: true
tags:
 - Java
 - 非阻塞I/O(NIO)
---

  在机器中每一级总线的速度差异是巨大的，cpu肯定不愿意等磁盘，对于比磁盘还要慢的网络它就更不愿意了，如果你用过某些速度很慢的VPS的话，你可以观察到每次敲一个键然后等2-3秒出来的情况。

  对于这种网络比CPU慢几个数量级的情况，传统的解决手法是用缓存和多线程，多线程处理不同的数据连接，然后把处理的数据放到缓存区中，一旦网络可以发送数据了就发送它。在你学习如何用ServerSocket等等去搭建一个应用层服务器的时候，你可能会以一个请求一个线程的方式，我有理由相信你肯定是这么干的。实际上以现在的硬件水平，这并不是多大的事情，但是如果你想开发一个应用而你又没有什么钱，只能用用5刀/月的廉价VPS的话，这可能会是一个问题。

  实际上非阻塞I/O在硬件允许的一般情况下都不会比多线程阻塞I/O快，非阻塞I/O的优势基于阻塞I/O的方式，即CPU不会浪费时间去等待一个I/O完成。

<!--more-->

## 客户端非阻塞I/O

  非阻塞I/O虽然主要用于服务器，但是的确是可以用于客户端的。

  首先让我们来写一个最简单的服务器端，这个服务器端监听本机端口9999，一旦它发现有客户端一个请求过来，就每隔一毫秒给客户端发送一句hello

```java
@Test
public void server() throws IOException, InterruptedException {
  ServerSocket serverSocket=new ServerSocket(9999);
  Socket socket=serverSocket.accept();
  OutputStream outputStream=socket.getOutputStream();
  OutputStreamWriter writer=new OutputStreamWriter(outputStream, "utf-8");
  for(int i=0;i<1000;i++){
    //让线程阻塞1毫秒
    Thread.sleep(1);
    //向底层流写入一个hello
    writer.write("hello\r\n");
    //发送
    writer.flush();
  }
  //关闭
  socket.close();
}
```

首先让我们来看非阻塞模式下的客户端I/O

```java
@Test
public void client1() throws IOException {
  
  SocketAddress address=new InetSocketAddress("127.0.0.1", 9999);
  //用静态工厂方法来创建一个java.nio.channels.SocketChannel对象
  SocketChannel client=SocketChannel.open(address);
  
  /*
  参数为true通道以阻塞模式工作，与一般的I/O流无异
  参数为false通道端以非阻塞模式工作
  */
  client.configureBlocking(false);
  
  //获取一个缓冲区
  ByteBuffer buffer=ByteBuffer.allocate(1024);
  
  //使用工具类Channels将System.out对象封装在一个通道中
  WritableByteChannel output=Channels.newChannel(System.out);
  int s=0;
  /*
  SocketChannel的read方法，如果对象设置了阻塞模式，则在读不到数据的时候就直接返回0，而读取到数据的时候则返回读取到的字节数，或者数据末尾时返回-1标志结束，如果是在阻塞的模式下则与普通的io无异，同样会阻塞线程
  */
  while((s=client.read(buffer))!=-1){
    if(s==0){
      //对于没有数据的情况，不会阻塞而是输出一个nio
      System.out.println("nio");
    }else{
      //绕回缓冲区，让System.out从头开始读数据
      buffer.flip();
      output.write(buffer);
      //清空缓冲，节省时间
      buffer.clear();
    }
  }
}

/*
输出：很多个nio中夹杂着hello

 ...
nio
nio
nio
nio
nio
nio
nio
nio
nio
hello
nio
nio
nio
nio
nio
nio
nio
nio
nio
nio
nio
 ...

*/
```

  可以清楚的看到，客户端不会等待服务器1毫秒一次的输出，而是转而可以做其他的事情，这样的话在这个线程中我们实际上可以处理多个对服务器的请求，因为快的请求并不会被慢的请求阻塞，不会像开车时老司机们总是抱怨年轻司机导致堵车。

  对于传统的IO而言，我们可能需要为每一个请求分配一个线程，有时候比如，如果你做的一个BT下载器，需要加入一个磁力链接的解析功能，你可能会去不同的站点来搜索这个磁力链接对应的种子，比如说vuze等等，这时候你可能会希望不需要多线程，而只希望用一个线程，却要来取得最快返回结果的那个，用Future对象来限制在一定时间内获取结果，这时候用非阻塞I/O就是很好的。