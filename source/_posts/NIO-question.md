---
title: NIO_question
date: 2018-09-17 21:21:14
tags:
- java
- IO
categories: 
- 面试
- java
- IO
---
NIO 几个问题 

- Nio和IO有什么区别
- Nio和aio的区别  
- Nio的原理 - Channel和buffer
- directBuffer和buffer的区别

<!-- more -->
next
- netty 源码
---------------------------
# - NIO 基本原理

Buffer 和数组差不多，它有 position、limit、capacity 几个重要属性。put() 一下数据、flip() 切换到读模式、然后用 get() 获取数据、clear() 一下清空数据、重新回到 put() 写入数据。

读操作的时候将 Channel 中的数据填充到 Buffer 中，而写操作时将 Buffer 中的数据写入到 Channel 中。

Selector 建立在非阻塞的基础之上，大家经常听到的 多路复用 在 Java 世界中指的就是它，用于实现一个线程管理多个 Channel。


```
Selector selector = Selector.open();
 
channel.configureBlocking(false);
 
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
 
while(true) {
  // 判断是否有事件准备好
  int readyChannels = selector.select();
  if(readyChannels == 0) continue;
 
  // 遍历
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

# NIO AIO， # - IO NIO BIO 区别
BIO 就是IO， AIO 就是java 1.7的NIO。2
非阻塞 IO 和异步 IO，也就是大家耳熟能详的 NIO 和 AIO。很多初学者可能分不清楚异步和非阻塞的区别，只是在各种场合能听到异步非阻塞这个词。

IO 就是一个线程对一个连接， 阻塞在那里。 

NIO 就是非阻塞。 
    - select：上世纪 80 年代就实现了，它支持注册 FD_SETSIZE(1024) 个 socket，在那个年代肯定是够用的，不过现在嘛，肯定是不行了。
    - poll：1997 年，出现了 poll 作为 select 的替代者，最大的区别就是，poll 不再限制 socket 数量。 只是告诉你y有几个准备好了， 没有告诉你哪个。 意味着你需要每个去轮询。 
    - epoll。 epoll：2002 年随 Linux 内核 2.5.44 发布，epoll 能直接返回具体的准备好的通道，时间复杂度 O(1)。

More New IO，或称 NIO.2，随 JDK 1.7 发布，包括了引入异步 IO 接口和 Paths 等文件访问接口。

- 异步 是说被调用是否通知我， 按照之前的理解。 
- Java 异步 IO 提供了两种使用方式，分别是返回 Future 实例和使用回调函数。2、提供 CompletionHandler 回调函数
- 区别： 我的理解返回future 对象， 和里面有个callback。 




# - directBuffer和buffer

在Java的NIO中，我们一般采用ByteBuffer缓冲区来传输数据
这个问题， directbuffer 是jvm 以外的内存申请的。 direct buffer则是通过JNI在Java的虚拟机外的内存中分配了一块缓冲区。 关于回收， 不是JVM来控制的。 (但Direct Buffer的JAVA对象是归GC管理的，只要GC回收了它的JAVA对象，操作系统才会释放Direct Buffer所申请的空间)


buffer 是 heap buffer， heap buffer这种缓冲区是分配在堆上面的，直接由Java虚拟机负责垃圾回收，可以直接想象成一个字节数组的包装类。

劣势：创建和释放Direct Buffer的代价比Heap Buffer得要高；

优势： 当我们把一个Direct Buffer写入Channel的时候，就好比是“内核缓冲区”的内容直接写入了Channel，这样显然快了，减少了数据拷贝（因为我们平时的read/write都是需要在I/O设备与应用程序空间之间的“内核缓冲区”中转一下的）。而当我们把一个Heap Buffer写入Channel的时候，实际上底层实现会先构建一个临时的Direct Buffer，然后把Heap Buffer的内容复制到这个临时的Direct Buffer上，再把这个Direct Buffer写出去。当然，如果我们多次调用write方法，把一个Heap Buffer写入Channel，底层实现可以重复使用临时的Direct Buffer，这样不至于因为频繁地创建和销毁Direct Buffer影响性能。

结论：Direct Buffer创建和销毁的代价很高，所以要用在尽可能重用的地方。 比如周期长传输文件大采用direct buffer，不然一般情况下就直接用heap buffer 就好。


# - TODO

# refer
- 1 https://blog.csdn.net/u010853261/article/details/53464322
- 2 http://www.importnew.com/28007.html  NIO原理
- 3 https://javadoop.com/post/nio-and-aio NIO.2





