# NIO三大核心组件学习笔记

[toc]

## Buffer

NIO的读写操作都是建立在BUffer上面的，Buffer就是字面意思--缓冲区。Buffer本质上是一块内存，然后我们对这块内存进行操作。

Buffer对于每一个具体的基本类型一般都有一个对应的缓冲。但是使用最为广泛的还是ByteBuffer，因为流的传输不就是字节。

另外说一句，ByteBuffer有两个直接可以使用子类，HeapByteBuffer和DirectBufferBuffer。他们之间的区别在于HeapByteBuffer的底层是通过new出来的新对象，所以一定在堆上分配的存储空间，属于jvm所能够控制的范围。而DirectBufferBuffer只是在jvm上保存了一个引用，其主要的内存分配在是在jvm之外的，通过unsafe类的native方法进行分配。

<img src="https://i.loli.net/2020/04/06/ZRDEjhPcqF6dVJU.png" alt="image-20200406131658433" style="zoom:80%;" />



<img src="https://i.loli.net/2020/04/06/X2J9pEVF1Yz46Id.png" alt="image-20200406130204071" style="zoom:80%;" />

```java
/*
Buffer是一个抽象类，里面有几个特别重要的属性
1. capacity 这个就是Buffer的最大容量
2. position 就是标记当前写入的位置或者读取的位置
3. limit的作用比较巧妙。主要分为写入模式和读取模式两种
	在写入的模式的时候，limit=capacity，表示最大写入数据量
	在读取模式的时候，limit表示最大读取的数据量（在读取之前会调用flip方法，切换模式，将limit=position，并且position=0）
4. mark用于临时标记position的位置，以便后续的使用
*/
public abstract class Buffer {
    static final int SPLITERATOR_CHARACTERISTICS =
        Spliterator.SIZED | Spliterator.SUBSIZED | Spliterator.ORDERED;
    // Invariants: mark <= position <= limit <= capacity
    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;
```

<img src="https://i.loli.net/2020/04/06/KSA9lOoN2syxwdH.png" alt="image-20200406130835527" style="zoom:80%;" />

### Buffer的基本使用

```java
//初始化ByteBuffer 
ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
```

### 填充的put方法

```java
// 填充一个 byte 值
public abstract ByteBuffer put(byte b);
// 在指定位置填充一个 int 值
public abstract ByteBuffer put(int index, byte b);
// 将一个数组中的值填充进去
public final ByteBuffer put(byte[] src) {...}
public ByteBuffer put(byte[] src, int offset, int length) {...}
```

### 获取的get方法

```java
// 根据 position 来获取数据
public abstract byte get();
// 获取指定位置的数据
public abstract byte get(int index);
// 将 Buffer 中的数据写入到数组中
public ByteBuffer get(byte[] dst)
```

### 切换读写模式的flip方法

```java
public final Buffer flip() {
    limit = position; // 将 limit 设置为实际写入的数据数量
    position = 0; // 重置 position 为 0
    mark = -1; // mark 之后再说
    return this;
}
```

### 重新读取的rewind方法

```java
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```

### 重置缓冲区的clear方法

```java
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
```

### 重置缓冲区的compact方法

```java
/*
这个方法于clear最大区别在于clear直接清空了三个重要的参数。
而compact会将当前未读取的内容移到缓冲开头，然后写入的内容会接在未读取数据之后
*/
public ByteBuffer compact() {
    System.arraycopy(hb, ix(position()), hb, ix(0), remaining());
    position(remaining());
    limit(capacity());
    discardMark();
    return this;
}
```

## Channel

Channel的字面意思是通道，也就是说我们读写数据是依靠Channel进行的。

下图是Channel的分类。

<img src="https://i.loli.net/2020/04/06/FZDx4rgYfLkQcwB.png" alt="image-20200406141743744" style="zoom:80%;" />

### FileChannel

FileChannel指的是文件通道，这个是阻塞的。

#### FileChannel的读操作

```java
/*
基本操作
1. 从channel中读取数据到buffer
2. flip
3. 读取buffer中的数据
*/
ByteBuffer buffer = ByteBuffer.allocate(100);
// fileChannel中读出来，写到buffer中
int read = fileChannel.read(buffer);
buffer.flip();
while (buffer.hasRemaining()) {
	System.out.print((char) buffer.get());
}
```

#### FileChannel的写操作

```java
/*
基本操作
1. 写入数据到buffer
2. flip
3. channel去读取buffer中的数据
*/
String str = " is best";
buffer.clear();
buffer.put(str.getBytes());
buffer.flip();
// 从buffer中读出来，通过channel写入到文件中
while (buffer.hasRemaining()) {
	fileChannel.write(buffer);
}
```

### SocketChannel

SocketChannel可以理解为TCP的客户端（这看起来只是SocketChannel的一部分功能）

```java
// 打开一个通道
SocketChannel socketChannel = SocketChannel.open();
// 发起连接
socketChannel.connect(new InetSocketAddress("https://www.javadoop.com", 80));
```

```java
//严格来说应该和前面的FileChannel一样的操作
// 读取数据
socketChannel.read(buffer);

// 写入数据到网络连接中
while(buffer.hasRemaining()) {
    socketChannel.write(buffer);   
}
```

### ServerSocketChannel

ServerSocketChannel可以理解为TCP的服务端。

```java
// 实例化
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
// 监听 8080 端口
serverSocketChannel.socket().bind(new InetSocketAddress(8080));

while (true) {
    // 一旦有一个 TCP 连接进来，就对应创建一个 SocketChannel 进行处理
    //socketChannel 在这里表示一个网络通道，不单单是只TCP客户端
    SocketChannel socketChannel = serverSocketChannel.accept();
}
```

### DatagramChannel

DatagramChannel指的是数据报通道，指的是UDP协议，DatagramChannel不区分客户端和服务端。

```java
//监听端口
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9090));
ByteBuffer buf = ByteBuffer.allocate(48);
channel.receive(buf);
```

```java
//发送数据
String newData = "New String to write to file..."
                    + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.put(newData.getBytes());
buf.flip();

int bytesSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));
```

## Selector

Selector指的是多路复用器，在Java的NIO中，主要就是使用Selector来轮询监听Channel的方式，通过一个线程来管理Channel，减少线程切换的带来的性能消耗。

 ![image-20200406143811776](C:\Users\39268\Desktop\image-20200406143811776.png)

```java
//创建多路复用器
Selector selector = Selector.open();
//设置未非阻塞，因为Selector是非阻塞的，所以注册的channel必须支持非阻塞模式，处理FileChannel，其余都支持
channel.configureBlocking(false);
//register方法，将当前channel注册到selector上面，并让selector监听Selectionkey.OP_READ事件
SelectionKey key = channel.register(selector, Selectionkey.OP_READ);
```

**一个SelectionKey键表示了一个特定的通道对象和一个特定的选择器对象之间的注册关系。**

```java
/**
我们可以看到SelectionKey中有获得SelectableChannel的channel（）方法
有获得Selector的selector方法
同时也可以判断当前状态的isxxxx方法
*/
public abstract class SelectionKey {

    protected SelectionKey() { }

    public abstract SelectableChannel channel();

    public abstract Selector selector();
    
     public final boolean isReadable() {
        return (readyOps() & OP_READ) != 0;
    }

    public final boolean isWritable() {
        return (readyOps() & OP_WRITE) != 0;
    }

    public final boolean isConnectable() {
        return (readyOps() & OP_CONNECT) != 0;
    }

    public final boolean isAcceptable() {
        return (readyOps() & OP_ACCEPT) != 0;
    }

```

## 参考内容

[Java NIO：Buffer、Channel 和 Selector](https://www.javadoop.com/post/java-nio)

[Java NIO 之 Selector（选择器）](https://zhuanlan.zhihu.com/p/36930888)