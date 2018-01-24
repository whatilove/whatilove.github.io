---
layout: post
title: NIO 编程（五）ServerSocketChannel、SocketChannel
categories: Java IO
tags: [java]
---

## ServerSocketChannel

Java NIO中的 ServerSocketChannel 是一个可以监听新进来的TCP连接的通道, 就像标准IO中的ServerSocket一样。ServerSocketChannel类在 java.nio.channels包中。

```
	ServerSocketChannel server = ServerSocketChannel.open();
	server.socket().bind(new InetSocketAddress(8888));
	while (true) {
	    SocketChannel socketChannel = server.accept();
	    // do something with socketChannel...
	}
```

### 打开 ServerSocketChannel

通过调用 ServerSocketChannel.open() 方法来打开ServerSocketChannel.如：

```
	ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

```

### 关闭 ServerSocketChannel

通过调用ServerSocketChannel.close() 方法来关闭ServerSocketChannel. 如：

```
	serverSocketChannel.close();

```

### 监听新进来的连接

通过 ServerSocketChannel.accept() 方法监听新进来的连接。当 accept()方法返回的时候,它返回一个包含新进来的连接的 SocketChannel。因此, accept()方法会一直阻塞到有新连接到达。

```
	while (true) {
	    SocketChannel socketChannel = server.accept();
	    // do something with socketChannel...
	}

```

### 非阻塞模式

ServerSocketChannel可以设置成非阻塞模式。在非阻塞模式下，accept() 方法会立刻返回，如果还没有新进来的连接,返回的将是null。 因此，需要检查返回的SocketChannel是否是null.如：

```
	ServerSocketChannel server = ServerSocketChannel.open();
	server.socket().bind(new InetSocketAddress(8888));
	server.configureBlocking(false); // 设置为非阻塞模式
	while (true) {
	    SocketChannel socketChannel = server.accept();
	    if (socketChannel != null) {
	    	//do something with socketChannel...
		}
	}

```

## SocketChannel

Java NIO 中的 SocketChannel 是一个连接到 TCP 网络套接字的通道。可以通过以下2种方式创建 SocketChannel：

1、打开一个 SocketChannel 并连接到互联网上的某台服务器。
2、一个新连接到达 ServerSocketChannel 时，会创建一个 SocketChannel。

### 打开 SocketChannel

```
SocketChannel socketChannel = SocketChannel.open();             
socketChannel.connect(new InetSocketAddress("127.0.0.1", 8888));

```

### 关闭 SocketChannel

当用完SocketChannel之后调用SocketChannel.close()关闭SocketChannel：

```
socketChannel.close();

```

### 从 SocketChannel 读取数据

要从 SocketChannel 中读取数据，调用 read() 方法。以下是例子：

```
ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
int bytesRead = socketChannel.read(byteBuffer);

```

首先，分配一个Buffer。从 SocketChannel 读取到的数据将会放到这个 Buffer 中。

然后，调用 SocketChannel.read()。该方法将数据从 SocketChannel 读到 Buffer 中。read()方法返回的 int 值表示读了多少字节进 Buffer 里。如果返回的是-1，表示已经读到了流的末尾（连接关闭了）。

### 写入 SocketChannel

写数据到 SocketChannel 用的是 SocketChannel.write() 方法，该方法以一个 Buffer 作为参数。示例如下：

```
	ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
	byteBuffer.put("test".getBytes());

	byteBuffer.flip();

	while (byteBuffer.hasRemaining()) {
	    socketChannel.write(byteBuffer);
	}

```

注意 SocketChannel.write() 方法的调用是在一个 while 循环中的。Write() 方法无法保证能写多少字节到 SocketChannel。所以，我们重复调用 write()直到 Buffer 没有要写的字节为止。

### 非阻塞模式

可以设置 SocketChannel 为非阻塞模式（non-blocking mode）.设置之后，就可以在异步模式下调用connect(), read() 和write()了。
如果 SocketChannel 在非阻塞模式下，此时调用 connect()，该方法可能在连接建立之前就返回了。为了确定连接是否建立，可以调用 finishConnect() 的方法。

```
SocketChannel socketChannel = SocketChannel.open();
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("127.0.0.1", 8888));

while (!socketChannel.finishConnect()) {
    //wait, or do something else...
}

```

### 非阻塞模式与选择器

非阻塞模式与选择器搭配会工作的更好，通过将一或多个SocketChannel注册到Selector，可以询问选择器哪个通道已经准备好了读取，写入等。

## 参考

[参考并发编程网 - ifeve.com](http://ifeve.com/java-nio-all/)
