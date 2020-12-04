# Netty初步学习笔记(一)

[toc]

## 传统BIO的缺点

第一，在任何时候都可能有大量的线程处于休眠状态，只是等待输 入或者输出数据就绪，这可能算是一种资源浪费。

第 二，需要为每个线程的调用栈都分配内存，其默认值 大小区间为 64 KB到1 MB，具体取决于操作系统。

第 三，即使 Java 虚拟机（JVM）在物理上可以支持非常 大数量的线程，但是远在到达该极限之前，上下文切换所带来的开销就会带来麻烦，例如，在达 到 10 000 个连接的时候。

## NIO的出现

NIO有效解决了BIO线程较多的问题，使用一个Selector多路选择器用来检查线程的读写操作是否完成。采用这个设计，主要有以下的好处，

- 使用较少的线程便可以处理许多连接，因此也减少了内存管理和上下文切换所带来开销； 
- 当没有 I/O 操作需要处理的时候，线程也可以被用于其他任务。

但是NIO的缺点也很明显，那就是程序编写比较复杂，容易出错。

## Netty简介

Netty 是一款异步的事件驱动的网络应用程序框架，支持快速地开发可维护的高性能的面向协议的服务器 和客户端。

**为什么说Netty是基于NIO（同步非阻塞）的，但是介绍说的是基于异步事件驱动**

参考自[Netty不是以NIO为底层的嘛？为什么又说自己是异步事件驱动的？](https://segmentfault.com/q/1010000016221286)

`Netty`说自己是异步事件驱动的框架，并没有说网络模型用的是异步模型，异步事件驱动框架体现在所有的`I/O`操作是异步的，所有的`IO`调用会立即返回，并不保证调用成功与否，但是调用会返回`ChannelFuture`，`netty`会通过`ChannelFuture`通知你调用是成功了还是失败了亦或是取消了。

## ByteBuf的三个方法的比较slice()、duplicate()、copy()

这三个方法通常情况会放到一起比较，这三者的返回值都是一个新的 ByteBuf 对象

1. slice() 方法从原始 ByteBuf 中截取一段，这段数据是从 readerIndex 到 writeIndex，同时，返回的新的 ByteBuf 的最大容量 maxCapacity 为原始 ByteBuf 的 readableBytes()
2. duplicate() 方法把整个 ByteBuf 都截取出来，包括所有的数据，指针信息
3. slice() 方法与 duplicate() 方法的相同点是：底层内存以及引用计数与原始的 ByteBuf 共享，也就是说经过 slice() 或者 duplicate() 返回的 ByteBuf 调用 write 系列方法都会影响到 原始的 ByteBuf，但是它们都维持着与原始 ByteBuf 相同的内存引用计数和不同的读写指针
4. slice() 方法与 duplicate() 不同点就是：slice() 只截取从 readerIndex 到 writerIndex 之间的数据，它返回的 ByteBuf 的最大容量被限制到 原始 ByteBuf 的 readableBytes(), 而 duplicate() 是把整个 ByteBuf 都与原始的 ByteBuf 共享
5. slice() 方法与 duplicate() 方法不会拷贝数据，它们只是通过改变读写指针来改变读写的行为，而最后一个方法 copy() 会直接从原始的 ByteBuf 中拷贝所有的信息，包括读写指针以及底层对应的数据，因此，往 copy() 返回的 ByteBuf 中写数据不会影响到原始的 ByteBuf
6. slice() 和 duplicate() 不会改变 ByteBuf 的引用计数，所以原始的 ByteBuf 调用 release() 之后发现引用计数为零，就开始释放内存，调用这两个方法返回的 ByteBuf 也会被释放，这个时候如果再对它们进行读写，就会报错。因此，我们可以通过调用一次 retain() 方法 来增加引用，表示它们对应的底层的内存多了一次引用，引用计数为2，在释放内存的时候，需要调用两次 release() 方法，将引用计数降到零，才会释放内存
7. 这三个方法均维护着自己的读写指针，与原始的 ByteBuf 的读写指针无关，相互之间不受影响

## ChannelHandler的生命周期

ChannelHandler 回调方法的执行顺序为

```
handlerAdded() -> channelRegistered() -> channelActive() -> channelRead() -> channelReadComplete()->channelInactive() -> channelUnregistered() -> handlerRemoved()
```

## Channel、EventLoop 和 EventLoopGroup三者的关系

<img src="C:\Users\39268\AppData\Roaming\Typora\typora-user-images\image-20200503110053352.png" alt="image-20200503110053352" style="zoom:80%;" />

- 一个 EventLoopGroup 包含一个或者多个 EventLoop； 

-  一个 EventLoop 在它的生命周期内只和一个 Thread 绑定； 

- 所有由 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理； 

- 一个 Channel 在它的生命周期内只注册于一个 EventLoop； 

-  一个 EventLoop 可能会被分配给一个或多个 Channel。

## ChannelHandler 安装到 ChannelPipeline 中的过程

- 一个ChannelInitializer的实现被注册到了ServerBootstrap中 

-  当 ChannelInitializer.initChannel()方法被调用时，ChannelInitializer 将在 ChannelPipeline 中安装一组自定义的 ChannelHandler；

-  ChannelInitializer 将它自己从 ChannelPipeline 中移除。

  ## pipeline.writeAndFlush和ctx.writeAndFlush的区别

在Netty中，有两种发送消息的方式。你可以直接写到Channel中，也可以 写到和ChannelHandler相关联的ChannelHandlerContext对象中。前一种方式将会导致消息从ChannelPipeline 的尾端开始流动，而后者将导致消息从 ChannelPipeline 中的下一个 ChannelHandler 开始流动。

## Netty的编解码器

编解码器的作用主要是处理数据的转换。

入站消息会被解码；也就是说，从字节转换为另一种格式，通常是一个 Java 对象。如果是出站消息，则会发生相反方向的转换：它将从它的当前格式被编码为字节。

一般来说，Netty预置的编解码器都是基于以下的两个类。

对于ByteToMessageDecoder来说，只需要重写decode方法即可，MessageToByteEncoder只需要重写encode方法即可。

```java
public abstract class ByteToMessageDecoder extends ChannelInboundHandlerAdapter
```

```java
public abstract class MessageToByteEncoder<I> extends ChannelOutboundHandlerAdapter
```

## 处理业务逻辑的SimpleChannelInboundHandler

```java
public abstract class SimpleChannelInboundHandler<I> extends ChannelInboundHandlerAdapter 
```

在将字节数据转化成java对象后，就需要进行业务逻辑处理，一般来说，继承SimpleChannelInboundHandler\<I>就可以了。传入的泛型参数I就是你要处理的对象类型，然后重写`channelRead0`，在这里实现逻辑就可以了。

```java
protected abstract void channelRead0(ChannelHandlerContext ctx, I msg) throws Exception;
```

当然，由于SimpleChannelInboundHandler继承自ChannelInboundHandlerAdapter，那么也可以重写这里这里面的方法完成其他的功能,这些都是channelHandler生命周期中的一些参数。

```java
public class ChannelInboundHandlerAdapter extends ChannelHandlerAdapter implements ChannelInboundHandler {
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception 
    public void channelActive(ChannelHandlerContext ctx) throws Exception 
    public void channelInactive(ChannelHandlerContext ctx) throws Exception 
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception 
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception 
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception
    public void channelWritabilityChanged(ChannelHandlerContext ctx) throws Exception 
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)  throws Exception 
}
```

## 服务端为什么需要两个EventLoopGroup

第一组将只包含一个 ServerChannel，代表服务器自身的已绑定到某个本地端口的正在监听的套接字。而第二组将包含所有已创建的用来处理传 入客户端连接（对于每个服务器已经接受的连接都有一个）的 Channel。图 3-4 说明了这个模 型，并且展示了为何需要两个不同的 EventLoopGroup。

与 ServerChannel 相关联的 EventLoopGroup 将分配一个负责为传入连接请求创建Channel 的 EventLoop。一旦连接被接受，第二个 EventLoopGroup 就会给它的 Channel 分配一个 EventLoop。

![image-20200503141130573](https://i.loli.net/2020/05/03/56I2mRpYj1GUoMW.png)

**也就是说，第一个EventLoopGroup中的每一个EventLoop处理一个端口，而第二个EventLoopGroup为每一个Channel分配一个EventLoop用于业务处理。**

## Netty的Channel的一些特性

Netty 的 Channel 实现是线程安全的，因此你可以存储一个到 Channel 的引用，并且每当你需要向远程节点写数据时，都可以使用它，即使当时许多线程都在使用它。

## 零拷贝

零拷贝（zero-copy）是一种目前只有在使用 NIO和 Epoll 传输时才可使用的特性。它使你可以快速 高效地将数据从文件系统移动到网络接口，而不需要将其从内核空间复制到用户空间，其在像 FTP 或者 HTTP 这样的协议中可以显著地提升性能。但是，并不是所有的操作系统都支持这一特性。特别地，它对 于实现了数据加密或者压缩的文件系统是不可用的——只能传输文件的原始内容。反过来说，传输已被 加密的文件则不是问题。

## ByteBuf 分配

-  ByteBufAllocator 接口

  - PooledByteBufAllocator
    - 池化了ByteBuf的实例以提高性能并最大限度地减少内存碎片。此实现使用了一种称为jemalloc的已被大量现代操作系统所采用的高效方法来分配内存。在每一个实现了Channel接口的类中，都包含了一个alloc方法，来返回与该Channel绑定的一个ByteBufAllocator。

  - UnpooledByteBufAllocator

- Unpooled 缓冲区
  - 提供了大量静态方法来创建缓冲区的工具类。该工具实际上是通过调用UnpooledByteBufAllocator来实现最终缓冲区的创建工作的。

- ByteBufUtil
  - ByteBufUtil工具的类的关注点是对于已有的缓冲区的操作，如打印（hexDump）、编码、解码、拷贝等。

## 添加 ChannelFutureListener

1. 添加 ChannelFutureListener 到 ChannelFuture

```java
     ChannelFuture future = channel.write(someMessage);
        future.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture f) {
                if (!f.isSuccess()) {
                    f.cause().printStackTrace();
                    f.channel().close();
                }
            }
        });
```

2. 添加 ChannelFutureListener 到 ChannelPromise

```java
public class OutboundExceptionHandler extends ChannelOutboundHandlerAdapter {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
        promise.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture f) {
                if (!f.isSuccess()) {
                    f.cause().printStackTrace();
                    f.channel().close();
                }
            }
        });
    }
}
```

## Netty的任务调度

```java
//延时
Channel ch = ...
ScheduledFuture<?> future = ch.eventLoop().schedule(new Runnable() { 
    @Override 
    public void run() {
    System.out.println("60 seconds later"); }
  
}, 60, TimeUnit.SECONDS);
```

```java
//周期
Channel ch = ...
ScheduledFuture<?> future = ch.eventLoop().scheduleAtFixedRate(
        new Runnable() {
            @Override
            public void run() {
                System.out.println("Run every 60 seconds");
            }
        }, 60, 60, TimeUnit.Seconds);
```

## EventLoop的执行逻辑

![image-20200503230322865](https://i.loli.net/2020/05/03/D4ISjew8kAtJMvC.png)