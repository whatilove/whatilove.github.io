---
layout: post
title: NIO 编程（四）Channel - ServerSocketChannel、SocketChannel
categories: 笔记本
tags: [java]
---

Channel 接口只有两个函数，如下所示：

```

public interface Channel extends Closeable {

    /**
     * Tells whether or not this channel is open.
     *
     * @return <tt>true</tt> if, and only if, this channel is open
     */
    public boolean isOpen();

    /**
     * Closes this channel.
     *
     * <p> After a channel is closed, any further attempt to invoke I/O
     * operations upon it will cause a {@link ClosedChannelException} to be
     * thrown.
     *
     * <p> If this channel is already closed then invoking this method has no
     * effect.
     *
     * <p> This method may be invoked at any time.  If some other thread has
     * already invoked it, however, then another invocation will block until
     * the first invocation is complete, after which it will return without
     * effect. </p>
     *
     * @throws  IOException  If an I/O error occurs
     */
    public void close() throws IOException;

}

```

### FileChannel

Java NIO中的 FileChannel 主要是用来读、写和映射一个系统文件的 Channel，它是一个抽象类，具体由 FileChannelImpl 实现。

`FileChannel无法设置为非阻塞模式，它总是运行在阻塞模式下。`

源码如下：

```
	public abstract class FileChannel {

		// 构造函数
		protected FileChannel() {
		}

		public abstract int read(ByteBuffer dst) throws IOException;

		public abstract long read(ByteBuffer[] dsts, int offset, int length) throws IOException;

		public final long read(ByteBuffer[] dsts) throws IOException {
		    return read(dsts, 0, dsts.length);
		}

		public abstract int write(ByteBuffer src) throws IOException;

		public abstract long write(ByteBuffer[] srcs, int offset, int length) throws IOException;

		public final long write(ByteBuffer[] srcs) throws IOException {
		    return write(srcs, 0, srcs.length);
		}
	}

```

#### 打开 FileChannel

在使用FileChannel之前，必须先打开它。但是，FileChannel 抽象类不能通过实例化得到，需要通过使用一个InputStream、OutputStream或RandomAccessFile来获取一个FileChannel实例。下面是通过RandomAccessFile打开FileChannel的示例：

```
RandomAccessFile randomAccessFile = new RandomAccessFile("/Users/flyxk/yarn.lock", "rw");
FileChannel fileChannel = randomAccessFile.getChannel();

```

getChannel 源码如下：

```
	public FileChannel getChannel() {
		// 加锁
	    synchronized (this) {
	        if (channel == null) {
	        	// 具体实现类为 FileChannelImpl 调用静态方法 open
	            channel = FileChannelImpl.open(fd, true, false, this);
	        }
	        return channel;
	    }
	}

```

#### 从 FileChannel 读取数据

```
	ByteBuffer bf = ByteBuffer.allocate(1024);

	byte[] rets = null;
	while (fileChannel.read(bf) != -1) {
	    bf.flip();// 切换到读模式
	    int remaining = bf.remaining();// 剩余多少没读
	    rets = new byte[remaining];
	    for (int i = 0; i < remaining; i++) {
	        rets[i] = bf.get();
	    }
	    bf.clear();
	    System.out.println(new String(rets));
	}

```

首先，分配一个Buffer。从FileChannel中读取的数据将被读到Buffer中。

然后，调用FileChannel.read()方法。该方法将数据从FileChannel读取到Buffer中。read()方法返回的int值表示了有多少字节被读到了Buffer中。如果返回-1，表示到了文件末尾。

read 源码如下：

```
	public int read(ByteBuffer dst) throws IOException {
	    ensureOpen();
	    if (!readable)
	        throw new NonReadableChannelException();
	    // 写入缓冲区加锁,保证多线程下安全性
	    synchronized (positionLock) {
	        int n = 0;
	        int ti = -1;
	        try {
	            begin();
	            ti = threads.add();
	            if (!isOpen())
	                return 0;
	            do {
	                // 2 通过IOUtil.read实现
	                n = IOUtil.read(fd, dst, -1, nd);
	            } while ((n == IOStatus.INTERRUPTED) && isOpen());
	            return IOStatus.normalize(n);
	        } finally {
	            threads.remove(ti);
	            end(n > 0);
	            assert IOStatus.check(n);
	        }
	    }
	}

```

#### 向 FileChannel 写数据

```
	ByteBuffer bf = ByteBuffer.allocate(1024);
	bf.put("写的内容".getBytes());
	bf.flip();// 切换到读模式
	while (bf.hasRemaining()) {
	    fileChannel.write(bf);
	}

```

注意FileChannel.write()是在while循环中调用的。因为无法保证write()方法一次能向FileChannel写入多少字节，因此需要重复调用write()方法，直到Buffer中已经没有尚未写入通道的字节。

write 源码如下：

```
	public int write(ByteBuffer src) throws IOException {
	    ensureOpen();
	    if (!writable)
	        throw new NonWritableChannelException();
	    // 开始读取之前加锁
	    synchronized (positionLock) {
	        int n = 0;
	        int ti = -1;
	        try {
	            begin();
	            ti = threads.add();
	            if (!isOpen())
	                return 0;
	            do {
	                // 2 通过IOUtil.write实现
	                n = IOUtil.write(fd, src, -1, nd);
	            } while ((n == IOStatus.INTERRUPTED) && isOpen());
	            return IOStatus.normalize(n);
	        } finally {
	            threads.remove(ti);
	            end(n > 0);
	            assert IOStatus.check(n);
	        }
	    }
	}

```
