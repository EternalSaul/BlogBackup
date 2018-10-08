---
title: 'Practice of Basic Java Skills:非阻塞I/O (5):选择器'
date: 2017-07-24 08:53:41
toc: true
tags:
 - Java
 - 非阻塞I/O(NIO)
---

  循环的查询一个服务器socket通道是否接受到一个请求或者一个Socket通道是否能读写数据，是没有任何意义的，类似于我们不想要程序查询方式，所以有了中断一样，为了不让非阻塞I/O的理想背道而驰，我们引入了选择器（Selecor），让我们能够不让线程做一些毫无意义的查询。

  在前面的服务器和客户端例子中，我们接触了选择器，你可以发现目前为止NIO包中多数实例都是通过工厂方法构建的，Selector也不例外，我们用了Selector类的静态工厂open方法来构造一个实例，然后调用通道的register方法将通道注册进该选择器。

## 选择器

### 选择器的注册

| 方法                                       |
| ---------------------------------------- |
| SelectionKey register(Selector sel, int ops) |
| SelectionKey register(Selector sel, int ops, Object att) |

  通道的register方法中，第一个参数是接受一个选择器实例，第二个参数是操作数，用于标志要选择的通道类型

**SelectionKey.OP_ACCEPT= 1 << 4=16**

**SelectionKey .OP_CONNECT= 1 << 3=8**

**SelectionKey .OP_WRITE=1 << 2=4**

**SelectionKey .OP_READ= 1 << 0=1**

<!--more-->

看见位操作符，你应该可以联想到，是不是可以用|操作符组合操作，答案确实如此，但是如果遇到一个不可能的组合时就会抛出一个异常

```java
@Test
public void test13() throws IOException{
	Selector selector = Selector.open();
	SocketChannel socketChannel = SocketChannel.open();
    //这种组合不可能，会抛出异常，因为普通Socket肯定不是可接受的
	socketChannel.register(selector, SelectionKey.OP_ACCEPT|SelectionKey.OP_READ);
}
```
 第三个参数是SelectionKey的附件(attachment),一般为null。如果传入了一个对象则可以用attachment方法取出来，一般用作辅助处理。

### 获取就绪的通道

  我们之前在服务器例子中用select()方法来阻塞线程，直到有通道可用才会返回值。其实还有其他的方法做类似的事情

| 方法                       |                                  |
| ------------------------ | -------------------------------- |
| int selectNow()          | 非阻塞选择，即使没有通道可用也立刻返回              |
| int select(long timeout) | 阻塞选择，如果在timeout个毫秒内没有通道可用则不作任何操作 |
| int select()             | 阻塞选择                             |

如果你已经确认了有可用的通道，那么我们可以使用

**Set&lt;SelectionKey &gt; selectedKeys()**

  该方法取得一个选择键集合，我们可以获取该集合的迭代器来处理每一个SelectionKey，然后删除键来防止重复选择同一个键。这个我们在非阻塞I/O服务器中也已经做过了。

当选择器不再需要时，我们需要调用close()方法来释放其所占的资源。

### 处理SelectionKey

  当一个选择器注册了多个通道的时候且他们筛选条件又不同的时候，我们可以用以下四个方法判断我们获取到的SelectionKey属于哪一种通道，而分别做出处理，比如在第二篇文章中我们在单线程中实现了hello的非阻塞服务器，我们把服务器和一般通信通道注册在了同一个选择器中，并用SelectionKey的筛选方法进行了筛选，这四个方法分别是

**boolean isAccptable()**

**boolean isConnectable()**

**boolean isReadable()**

**boolean isWritable()**

他们对应着四个不同的操作数，上面我们已经见过了。

下面，我们就可以用

**selectableChannel channel()**

来获取通道对象，如果你注册时添加了附件对象，那么可以用**Object attachment()**方法来取得它。



如果一个通道在处理完之后就不再使用，那么我们应该直接把它对应的SelectionKey对象从选择器中移除，此时调用SelectionKey的**cancel()**方法可以做到这一点，这样我们选择器就不会再花时间去关系它所对应的通道的状态。



现在让我们再来回顾一下第二篇文章给出简单的非阻塞I/O服务器的通道选择部分的代码，你会发现一切变得很自然。

```java
		while(true){
			//等待至少有一个通道可用
			selector.select();
			
			//获取可用的SelectionKey集合
			Set<SelectionKey> reKeys=selector.selectedKeys();
			
			//构建迭代器
			Iterator<SelectionKey> iterator=reKeys.iterator();
			
			//迭代器遍历
			while(iterator.hasNext()){
				
				//获取当前的SelectionKey
				SelectionKey key=iterator.next();
				
				//删除已经遍历过的SelectionKey
				iterator.remove();
				
				SocketChannel client=null;
				ByteBuffer buffer=null;
				
				if(key.isAcceptable()){
					
					//等待接受一个请求
					client=serverSocketChannel.accept();
					
					//请求Sokect通道设置为非阻塞模式
					client.configureBlocking(false);
					
					/*
					 * 对于客户端通道，需要关心是否可以写入数据，这里我们需要获取SelectionKey对象
					 * SelectionKey中有一个指示当前链接状态的对象
					 */
					SelectionKey writableKey=client.register(selector, SelectionKey.OP_WRITE);
					
					//获取一个缓冲区
					buffer=ByteBuffer.allocate(1024);
					
					//写入内容
					buffer.put("hello\r\n".getBytes(), 0, "hello\r\n".getBytes().length);
					
					//绕回缓冲区，使得输出通道从所读取数据的头部而非尾部写入，即使让读写指向位置(position字段)为0
					buffer.flip();
					
					writableKey.attach(buffer);
					
				}else if(key.isWritable()){
					client=(SocketChannel) key.channel();
					buffer=(ByteBuffer) key.attachment();
					
					//先清空缓冲区
					buffer.clear();
					
					//写入内容到换出区
					buffer.put("hello\r\n".getBytes(), 0, "hello\r\n".getBytes().length);
					
					//绕回缓冲区，使得输出通道从所读取数据的头部而非尾部写入，即使让读写指向位置(position字段)为0
					buffer.flip();
				}
				
				//写入输出通道
				client.write(buffer);
			}
```

