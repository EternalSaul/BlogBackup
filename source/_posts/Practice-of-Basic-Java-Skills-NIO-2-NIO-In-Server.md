---
title: 'Practice of Basic Java Skills:非阻塞I/O (2)——服务器非阻塞I/O'
date: 2017-07-21 10:44:19
tags:
 - Java
 - 非阻塞I/O(NIO)
---

  服务器中可能更需要非阻塞I/O，特别是在你不需要维持那么多线程或者机器维持不了那么多线程的时候，上一篇文章我们看到了通道和缓冲区，它们是用来实现非阻塞I/O的重要手段，现在在服务器中我们可能还需要选择器（Selector）,选择器用来让服务器判断哪些链接是已经准备好手法数据的，在一般的多线程I/O中，你并不需要做这一点，因为它们总是阻塞自己的线程，直到数据开始传递。

   现在用以下代码构造一个极其简单的非阻塞I/O服务器,服务器和上个服务器实现的是一样的功能，只是这里我们不再让服务器线程阻塞一毫秒，一旦发现有客户机入站，就建立连接并不断的输出hello。

<!--more-->

```json
	@Test
	public void NIOServer() throws IOException, InterruptedException {
		//用工厂方法，创建一个服务器Socket通道
		ServerSocketChannel serverSocketChannel=ServerSocketChannel.open();
		
		/*
		 * 绑定本机端口号9999，这个方法在java 7之后才有，如果在以前的版本上写，
		 * 可能需要以socket方法先返回一个服务器socket才能绑定端口号
		 */
		serverSocketChannel.bind(new InetSocketAddress(9999));
		
		/*
		 * 设定其为非阻塞模式
		 * 这会导致accept方法，在没有入站连接的情况下会立刻返回一个null
		 */
		serverSocketChannel.configureBlocking(false);
		
		
		//创建一个Selector选择器
		Selector selector=Selector.open();
		
		/*
		 * 为每个服务器通道注册一个选择器
		 * 对于服务器通过只需要关心服务器Socket是不是准备好接受一个新的链接
		 */
		serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
		
		while(true){//服务器开始工作
			
			//等待至少有一个通道可用，如果一个都没有就阻塞
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
				
				if(key.isAcceptable()){//获取到有入站链接
					
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
					
					//绕回缓冲区，使得输出通道从所读取数据的头部而非尾部写入，即使让读写指向位置 (position字段)为0
					buffer.flip();
					
					writableKey.attach(buffer);
					
				}else if(key.isWritable()){//获取到可写的用户Socket
					client=(SocketChannel) key.channel();
					buffer=(ByteBuffer) key.attachment();
					
					//先清空缓冲区
					buffer.clear();
					
					//写入内容到换出区
					buffer.put("hello\r\n".getBytes(), 0, "hello\r\n".getBytes().length);
					
					//绕回缓冲区，使得输出通道从所读取数据的头部而非尾部写入，即使让读写指向位(position字段)为0
					buffer.flip();
				}
				
				//写入输出通道
				client.write(buffer);
			}
		}
		
	}
```

  在这个简单是实例服务器中，展示了一个非阻塞I/O的服务器在一个线程中如何区分客户端socket请求和可写的客户端socket，这里也可以把服务和监听请求两个功能分成两个线程去做，不过重点不在这里，而是通过这个例子你明白了，通过非阻塞I/O我们可以不需要为每个请求分配一个线程，但又不会阻塞当前线程，在性能上，一个线程不能允许在多个CPU上，但是多个线程是可以允许在多个CPU上的，如果你玩过《特大城市XXL》这个游戏的话，你也许能理解，如果不能充分利用多核心特性，对软件性能是很不利的。