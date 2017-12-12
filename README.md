# NIODemo
一、NIO -- new IO  -- NonBlocking IO
1.概述
	JDK1.4 中提出的一套新的IO机制，区别与传统的BIO（Blocking IO）

	BIO在高并发场景下遇到了一些无法解决的问题 ---- ACCEPT CONNECT READ WRITE
	针对于每一个客户端都需要在服务器端创建对应线程来处理，线程开辟运行是非常耗费资源的，并且服务器所能支持的最大并发线程数量是有限的，所以当高并发到来时，服务器会存在性能瓶颈。
	所以我们想到用少量的线程同时处理多个客户端的连接。而由于传统的BIO阻塞式操作的特点，这种工作方式是无法实现的。

	BIO:面向流操作字节字符,具有方向性
		InputStream OutputStream Reader Writer
	NIO:面向通道操作缓冲区，可以双向传输数据
		Channel	Buffer Selector
	
2.Buffer
	缓冲区，所谓的缓冲区其实就是在内存中开辟的一段连续空间，用来临时存放数据。

	int capacity();
	int position();
	position(int newPosition);
	int limit()
	limit(int newLimit)
		
	//1.创建Buffer
	static ByteBuffer allocate(int capacity)  
	static ByteBuffer wrap(byte[] array) 
	static ByteBuffer wrap(byte[] array, int offset, int length)  

	//2.写数据
	put(byte b) 
	put(byte[] src) 
	putXxx()
	.....

	//3.反转缓冲区
	buf.flip();//此方法等同于buf.limit(buf.position());buf.position(0);

	//4.读取数据
	byte get() 
	get(byte[] dst) 
	......

	//5.判断边界
	int remaning();
	boolean hasRemaning();

	//6.重绕缓冲区
	rewind();

	//7.清空缓冲区
	clear();//将limit设置为capacity大小，postion设置为0，并不真的删除缓冲区中的旧有数据，基于buffer的原理，并不会有问题。

	//8.设置标记，重置标记
	mark();//设置标记
	reset();//重置到标记


3.Channel
	NIO是基于Channel工作操作的是Buffer
	Channel叫做通道，与Stream不同，可以双向的进行数据通信

	ServerSocketChannel
		ServerSocketChannel ssc = ServerSocketChannel.open();
		configureBlocking(false);//设置为非阻塞模式
	SocketChannel 
		SocketChannel sc = SocketChannel.open();
		configureBlocking(false);//设置为非阻塞模式

		**在非阻塞模式下，进行ACCEPT CONNECT READ WRITE 操作时都不会产生阻塞，它们只是尝试完成操作，并不保证操作一定执行完。所以通常我们需要自己控制循环来保证操作执行完成。

		SelectionKey register(Selector sel, int ops)//将当前通道注册到选择器中。sel：要被注册到的选择器。ops注册时指定的要关注的操作，操作可以有四种类型 - SelectionKey.OP_ACCEPT SelectionKey.OP_CONNECT SelectionKey.OP_READ SelectionKey.OP_WRITE

	DatagramChannel

	FileChannel	


4.Selector
	//获取选择器
	Selector.open();

	//将通道注册到选择器中让选择器管理这个通道
	channle.regeister(selc,ops)

	//检查已经注册在选择器上的通道关心的操作是否有已经就绪可以处理的
	int select() //检查注册在选择器上的通道们关心的事件是否已经有就绪可以处理的，如果有此方法返回，返回值为可以处理的数量，如果没有可处理的则此方法阻塞。

	//获取已经就绪的键的集合
	Set<SelectionKey> selectedKeys()
5.粘包问题
	由于TCP传输是一种可靠的连续的数据传输，如果两次传输的数据时间间隔比较短，数据的接收方可能很难判断出两次数据的边界在哪里，感觉就好像两个数据黏着在了一次，无法区分。
	解决方案1:传输固定大小的数据，缺点是，很不灵活，有可能浪费传输空间
	解决方案2:约定一个特殊的字符作为判断的边界，缺点是如果数据中本身就包含这个特殊字符可能还需要进行转义的操作
	解决方案3:使用协议，通过协议传输数据量大小的方式来解决

	HTTP Content-Length

6.开源的NIO结构的服务器框架
	MINA NETTY

7.总结
	阻塞/非阻塞：是从线程的角度来考虑的，考虑线程是不是被挂起的状态
	同步/异步：是从逻辑执行的角度来考虑的，考虑程序在处理一段逻辑时可否并行处理另一端逻辑

	BIO -- jdk1.0 -- BlockingIO -- 同步阻塞式IO -- 面向流操作字节字符 -- 单向传输
	NIO -- jdk1.4 -- NonBlockingIO -- 同步非阻塞式IO -- 面向通道操作缓冲区 -- 双向传输
	AIO -- jdk1.7 -- AnsyncronizeIO	-- 异步非阻塞式IO -- 大量的使用回调函数实现了异步IO操作
