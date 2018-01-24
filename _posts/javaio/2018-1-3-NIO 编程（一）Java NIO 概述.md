---
layout: post
title: NIO 编程（一）Java NIO 概述
categories: [Java IO]
tags: [java]
---

Java NIO 由以下几个核心部分组成：

1. Channel 通道

2. Buffer 缓冲区

3. Selector 选择器

## Channel 和 Buffer

Channel 有点象流。 数据可以从 Channel 读到 Buffer 中，也可以从 Buffer 写到 Channel 中。

![](/assets/images/post/java/overview-channels-buffers.png)

## Buffer

Buffer 是一个对象，它包含一些要写入或者要读出的数据。在 NIO 类库中加入 Buffer 对象，体现了新库与原 I/O 的一个重要区别。在面向流的 I/O 中，可以将数据直接写入或者将数据直接读取 Stream 对象中。在 NIO 库中所有的数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的，在写入数据时，写入到缓冲区中。任何时候访问 NIO 中的数据，都是通过缓冲区进行操作。

最常用的缓冲区是 ByteBuffer ，一个 ByteBuffer 提供了一组功能用于操作 byte 数组。除了 ByteBuffer 还有其它一些缓冲区，事实上，每一种 Java 基本类型（除了 Boolean类型）都对应一种缓冲区。具体如下：

1. ByteBuffer：字节缓冲区。
2. CharBuffer：字符缓冲区。
3. ShortBuffer：短整形缓冲区。
4. IntBuffer：整形缓冲区。
5. LongBuffer：长整形缓冲区。
6. FloatBuffer：浮点型缓冲区。
7. DoubleBuffer：双精度浮点型缓冲区。

![](/assets/images/post/java/buffer.png)
（Buffer 继承关系图）

Buffer 读写例子：

```
package buffer;

import java.nio.ByteBuffer;


public class BufferTest {

    public static void main(String[] args) {

        int capacity = 1024;
        ByteBuffer b = ByteBuffer.allocate(capacity);

        // 写模式下 limit = capacity
        System.out.println("capacity -> " + b.capacity()); // 1024
        System.out.println("position -> " + b.position()); // 0
        System.out.println("limit -> " + b.limit()); // 1024

        // 增加数据, put 还是 get position 都会向前移动到下一个可读的位置
        b.put((byte) 'a');
        b.put((byte) 'b');
        b.put((byte) 'c');

        System.out.println("capacity -> " + b.capacity()); // 1024
        System.out.println("position -> " + b.position()); // 3
        System.out.println("limit -> " + b.limit()); // 1024

        // 切换到读模式 limit 为 写模式下的 postion, position 被置为0。
        b.flip();

        System.out.println("capacity -> " + b.capacity()); // 1024
        System.out.println("position -> " + b.position()); // 0
        System.out.println("limit -> " + b.limit()); // 3

        // 第一种读取方法
        int remaining = b.remaining();
        for (int i = 0; i < remaining; i++) {
            System.out.println(b.get());
        }

        // 第二种读取方法
//        while (b.hasRemaining()) {
//            System.out.println(b.get());
//        }
    }
}

```

## Channel

Channel 是一个通道，它就像自来水管一样，网络数据通过 Channel 读取和写入。通道与流的不同之处在于通道是双向的，流只是在一个方向上移动（一个流必须是 InputStream 或者 OutputStream 的子类）而通道可以用于读、写或者二者同时进行。

实际上 Channel 可以分为两大类：用于网络读写的 SelectableChannel 和用于文件操作的 FileChannel。

1. FileChannel 从文件中读写数据。
2. DatagramChannel 能通过UDP读写网络中的数据。
3. SocketChannel 能通过TCP读写网络中的数据。
4. ServerSocketChannel可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

![](/assets/images/post/java/channel.png)
（Channel 继承关系图）

Channel 例子：

```
package channel;

import java.io.FileInputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;


public class ChannelTest {

    public static void main(String[] args) throws IOException {

        FileChannel fileChannel = new FileInputStream("/Users/flyxk/untitled/package.json").getChannel();

        ByteBuffer b = ByteBuffer.allocate(1024);

        int readLength;
        while ((readLength = fileChannel.read(b)) != -1) {
            // 切换为读模式
            b.flip();
            byte[] bs = new byte[readLength];
            for (int i = 0; i < readLength; i++) {
                bs[i] = b.get();
            }
            System.out.println(new String(bs));
            b.clear();
        }
    }
}

```

## Selector

选择器提供选择已经就绪的任务的能力。简单来讲 Selector 会不断地轮询多个 Channel，如果某个 Channel 上面发生读或者写事件，这个 Channel 就处于就绪状态，会被 Selector 轮询出来，然后通过 SelectionKey 可以获取就绪 Channel 的集合，进行后续的 I/O 操作。

一个 Selector 可以同时轮询多个 Channel ，它没有最大连接句柄 1024/2048 的限制。这也就意味着只需要一个线程负责 Selector 的轮询，就可以接入成千上万的客户端。

Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用 Selector 就会很方便。例如，在一个聊天服务器中。

这是在一个单线程中使用一个 Selector 处理 3 个 Channel 的图示：

![](/assets/images/post/java/overview-selectors.png)

要使用 Selector，得向 Selector 注册 Channel，然后调用它的 select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。



## 参考

* 《Netty 权威指南》（第2版） 李林锋 著
