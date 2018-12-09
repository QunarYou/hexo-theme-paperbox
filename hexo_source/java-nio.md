---
title: Java NIO 特性学习
categories:
- articles
tags:
- java
- io
date: 2018/12/08 18:42:07
comments: true
---

#Java NIO 特性学习

[toc]


Java NIO 包含几个核心的组件：
- Channels
- Buffer
- Selectors

##Channels
![Alt text](./QQ图片20150716171015.png)
> 可以理解为资源的一个流，通过这个流资源可以从Channel读取Data到一个Buffer中或者从一个Buffer中写入Data到Channel；


###Channel Implementations
集中Jdk7常用的Channel上线
- FileChannel : 操作文件读取或者写入数据
- DatagramChannel : 从一个网络UDP连接中读取或写入数据
- SocketChannel : 从一个TCP网络连接中读取或写入数据
- ServerSocketChannel: 可以使用这个Channel来监听TCP连接，如同Web Server 当连接来时可以创建出一个SocketChannel

###Base Channel Example
Read Data from Channel
```
	c
```

##Buffer
> Java NIO Buffers 和Channels配合使用，**Read from channels into buffer ,writtern from buffers into channels**

###Buffer Usage
1. 向buffer中写入数据
2. 调用buffer.flip()方法
3. 使用buffer.get()方法获取出数据
4. 调用buffer.clear() 或者 buffer.compact()清理已经读取的数据

```
	RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt" , "rw");
	FileChannel inChannel = aFile.getChannel();
	//Create Buffer with capacity of 48 byte
	ByteBuffer buf = ByteBuffer.allocate(48);

	int bytesRead = inChannel.read(buf);

	while(bytesRead!=-1){

	  buf.flip();  //make buffer ready for read

		  while(buf.hasRemaining()){
		      System.out.print((char) buf.get()); // read 1 byte at a time
		  }

	  buf.clear(); //make buffer ready for writing
	  bytesRead = inChannel.read(buf);
	}
```

###Buffer Capacity , Position and Limit
![Alt text](./QQ图片20150716173614.png)

####Capacity
Buffer在创建的时候会分配规定大小的内存块，并且在buffer写入数据时只能写入这个固定大小的数据。如果Buffer已经放满数据，就需要先read 或者 clear 只能才能继续写入。

####Position
Position 在读模式和模式下面含义不一样。Bufer init之后Position值为0，当数据写入one byte ，long etc Buffer Position相应的移动到下一个位置。并且`Position<=Capacity-1`

当调用`filp()`从Buffer读取数据时会`set position = 0 `,position会随着读取过程向下移动；

####Limit
在Write模式下，Limit等于Capacity的值，表示能写多少数据到Buffer中。

调用`filp()`进入Reade模式会`set limit = position ` ,表示最多能读的数据量。

###常用的Buffer 实现
- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

###SomeMethods of Buffer
#### Allocating a Buffer
通过使用`allocate()` 来分配Buffer大小
>ByteBuffer buf = ByteBuffer.allocate(48);
>CharBuffer buf = CharBuffer.allocate(1024);

####Write Data to a Buffer
向Buffer中写入数据有两种方法：
- 从Channel中读取数据写入Buffer
- 调用Buffer `put()` 方法写入数据
>int bytesRead = inChannel.read(buf); //read into buffer.
>buf.put(127);

####flip()
转换写状态到读状态，发生动作：`set limit = position ; set position=0 ;`

####Reading Data from a Buffer
两种方法：
- 从Buffer中读取数据到Channel
- 读取Buffer中数据给自己,可以使用buffer自带方法 `get()` etc
>//read from buffer into channel.
>int bytesWritten = inChannel.write(buf);
>byte aByte = buf.get();

####rewind()
`set position=0`

####mark() 和 reset()
mark用于打点记录position为止，之后使用reset方法可以将position重置到mark记录的位置。

##Scatter / Gather /Transfers
>Channel 可以执行读取操作将数据读取到多个Buffer中。
>Channel 可以执行写操作将多个Buffer的数据写入Channle中。
>Channel 可以在Channle之间做转换包括`transferFrom() & transferTo `

###Scatter Reads :
![Alt text](./QQ图片20150716181513.png)

>例如我将html的header部分读取到第一个buffer中，将body部分读取到第二个Buffer中
```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(buffers);
```

###Gather Writes
![Alt text](./QQ图片20150716181528.png)
```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(buffers);
```
###Transfers
####TransferFrom()
```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

toChannel.transferFrom(fromChannel, position, count);
```
####TransferTo()
```
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```

##Selector
>Selector组件可以运行判断多个Channel，动态决定使用哪个Channel来执行Read 或者 Write操作。通过这个组件可以上线一个Thread 管理多个Channels 或者 多个网络连接Channel。
![Alt text](./QQ图片20150716182602.png)

###Create a Selector
通过使用`Selector.open()`方法来创建一个Selector
> Selector selector = Selector.open();

###注册Channels到Selector上
为了能让Selector管理Channels需要调用`Selector.register()`注册到Selector

>channel.configureBlocking(false); //The Channel must be in non-blocking mode to be used with a Selector
>SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
注册Channel的时候需要制定Channel需要关心的事件，事件包括：
- Connect  -> SelectionKey.OP_CONNECT
- Accept -> SelectionKey.OP_ACCEPT
- Read -> SelectionKey.OP_READ
- Write ->SelectionKey.OP_WRITE
如果注册多个事件可以使用 `int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;` 通过`int interestSet = selectionKey.interestOps();` 可以得到所有Selector's的事件。

Simple Demo：
```
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);


while(true) {

  int readyChannels = selector.select();

  if(readyChannels == 0) continue;


  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
  }
}
```







---






