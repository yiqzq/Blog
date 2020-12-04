# JAVA网络编程代码实例

[toc]

## BIO模型

### 特点

同步非阻塞，对于每一个客户端连接，由Acceptor线程负责监听客户端的连接，对于每一个客户端请求都会启用一个新的线程处理这个请求。

传统的BIO模型的特点是在高并发情况下会创建许多线程来处理请求，但是创建过多的线程会影响服务器的性能。

对于BIO的一个优化操作就是使用线程池来管理线程，那么就可以控制线程的创建与销毁，减少因为线程创建导致服务器崩溃的问题，当然本质还是同步阻塞的IO模型。

### 瓶颈

1. 高并发下线程过多,可以由线程池优化
2. BIO每有一个请求就由一个线程去处理，但是如果客户端没有发送数据，read方法就会一直阻塞，这其实没有意义的，也就是BIO模型也许会存在很多无用的连接，同理在write的时候也会造成阻塞，这个等待其实也是无意义的。



<img src="https://i.loli.net/2020/04/05/vqjntWkxZdD1w86.png" alt="image-20200405214751621" style="zoom:80%;" />



```java
//客户端,缺点是没有办法多行输入,每次只能发送一行数据
import javax.xml.crypto.Data;
import java.io.*;
import java.net.Socket;
import java.util.Date;
import java.util.Scanner;

public class BIOClient {
    public static void main(String[] args) throws IOException {
        int port = 8888;
        String ip = "127.0.0.1";
        Socket socket = new Socket(ip, port);
        BufferedReader in = null;
        PrintWriter out = null;
        System.out.println("客户端启动!");
        try {
            Scanner scanner = new Scanner(System.in);
            while (true) {
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                out = new PrintWriter(socket.getOutputStream(), true);
                String input = scanner.nextLine();
                out.println(input);
                if (input.equals("exit")) break;
                System.out.println("输入完毕,等待回应");
                String data = in.readLine();
                System.out.println("输入完毕,有回应了");
                System.out.println(new Date() + "客户端收到服务端响应内容:" + data);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (in != null) {
                    in.close();
                }
                if (out != null) {
                    out.close();
                }
                socket.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }


    }

}
```

```java
//服务端
import java.io.*;

import java.net.ServerSocket;
import java.net.Socket;
import java.util.Date;
import java.util.Scanner;

public class BIOServer {
    public static void main(String[] args) throws IOException {
        int port = 8888;
        ServerSocket serverSocket = new ServerSocket(port);
        System.out.println("服务器启动!");
        while (true) {
            Socket socket = serverSocket.accept();
            System.out.println("服务器收到连接了");
            new Thread(new SocketHandle(socket)).start();
        }
    }
}

class SocketHandle implements Runnable {
    Socket socket;


    SocketHandle(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        BufferedReader in = null;
        PrintWriter out = null;
        Scanner scanner = new Scanner(System.in);
        try {
            while (true) {
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                out = new PrintWriter(socket.getOutputStream(), true);
                String data = in.readLine();
                if (data.equals("exit")) {
                    break;
                }
                System.out.println(new Date() + "服务端收到客户端的消息:" + data);

                System.out.println("服务端接收完毕,准备发送消息");
                out.println(scanner.nextLine());
                System.out.println("服务端接收完毕,发送消息完毕");
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (in != null) {
                    in.close();
                }
                if (out != null) {
                    out.close();
                }
                socket.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        System.out.println("当前线程结束了");
    }
}
```

## NIO模型

NIO有三大核心组件，Selector，Channel，Buffer。

其中Selector其实是操作系统多路复用的一种封装。

主要有三种不同的多路复用组件。

**select**：上世纪 80 年代就实现了，它支持注册 FD_SETSIZE(1024) 个 socket，在那个年代肯定是够用的，不过现在嘛，肯定是不行了。同时只能监听哪些Channel可以操作，具体是哪个需要O（n）遍历

**poll**：1997 年，出现了 poll 作为 select 的替代者，最大的区别就是，poll 不再限制 socket 数量，时间复杂度也是O（n）的

**epoll**：2002 年随 Linux 内核 2.5.44 发布，epoll 能直接返回具体的准备好的通道，时间复杂度 O(1)。

```java
//服务端
import java.io.*;
import java.net.InetSocketAddress;
import java.nio.*;
import java.nio.channels.*;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.util.Iterator;
import java.util.Set;

public class SelectorServer {

    public static void main(String[] args) throws IOException {
        Selector selector = Selector.open();

        ServerSocketChannel server = ServerSocketChannel.open();
        server.socket().bind(new InetSocketAddress(8888));

        // 将其注册到 Selector 中，监听 OP_ACCEPT 事件
        server.configureBlocking(false);
        server.register(selector, SelectionKey.OP_ACCEPT);

        while (true) {
            int readyChannels = selector.select();
            if (readyChannels == 0) {
                continue;
            }
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            // 遍历
            Iterator<SelectionKey> iterator = readyKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove();

                if (key.isAcceptable()) {
                    // 有已经接受的新的到服务端的连接
                    SocketChannel socketChannel = server.accept();

                    // 有新的连接并不代表这个通道就有数据，
                    // 这里将这个新的 SocketChannel 注册到 Selector，监听 OP_READ 事件，等待数据
                    socketChannel.configureBlocking(false);
                    socketChannel.register(selector, SelectionKey.OP_READ);
                } else if (key.isReadable()) {
                    // 有数据可读
                    // 上面一个 if 分支中注册了监听 OP_READ 事件的 SocketChannel
                    SocketChannel socketChannel = (SocketChannel) key.channel();
                    ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                    int num = socketChannel.read(readBuffer);
                    if (num > 0) {
                        // 处理进来的数据...
                        System.out.println("收到数据：" + new String(readBuffer.array()).trim());
                        ByteBuffer buffer = ByteBuffer.wrap("返回给客户端的数据...".getBytes());
                        socketChannel.write(buffer);
                    } else if (num == -1) {
                        // -1 代表连接已经关闭
                        socketChannel.close();
                    }
                }
            }
        }
    }
}
```

```java
//客户端
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.*;
import java.nio.*;

public class SocketChannelClient {
    public static void main(String[] args) throws IOException {
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.connect(new InetSocketAddress("localhost", 8888));

        // 发送请求
        ByteBuffer buffer = ByteBuffer.wrap("1234567890".getBytes());
        socketChannel.write(buffer);

        // 读取响应
        ByteBuffer readBuffer = ByteBuffer.allocate(1024);
        int num;
        if ((num = socketChannel.read(readBuffer)) > 0) {
            readBuffer.flip();

            byte[] re = new byte[num];
            readBuffer.get(re);

            String result = new String(re, "UTF-8");
            System.out.println("返回值: " + result);
        }
    }
}
```

## AIO模型

AIO引入了新的异步通道的概念，并提供了异步文件通道和异步套接字通道的实现。异步通道提供两种方式获取获取操作结果。

1. 通过java.util.concurrent.Future类来表示异步操作的结果；
2. 在执行异步操作的时候传入一个java.nio.channels。

CompletionHandler接口的实现类作为操作完成的回调。



关于accept方法的两个参数

```java
//attachment：用于传递一些信息，比如获取连接之后重新调用accept方法
public abstract <A> void accept(A attachment,
                    CompletionHandler<AsynchronousSocketChannel,? super A> handler);
```

**CompletionHandler：用户处理器**

异步IO操作结果的回调接口，用于定义在IO操作完成后所作的回调工作。

```java
//服务端
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.SocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;

public class AIOServer {

    public static void main(String[] args) throws IOException {

        // 实例化，并监听端口
        AsynchronousServerSocketChannel server =
                AsynchronousServerSocketChannel.open().bind(new InetSocketAddress(8888));

        // 自己定义一个 Attachment 类，用于传递一些信息
        Attachment att = new Attachment();
        att.setServer(server);

        server.accept(att, new CompletionHandler<AsynchronousSocketChannel, Attachment>() {
            @Override
            public void completed(AsynchronousSocketChannel client, Attachment att) {
                try {
                    SocketAddress clientAddr = client.getRemoteAddress();
                    System.out.println("收到新的连接：" + clientAddr);

                    // 收到新的连接后，server 应该重新调用 accept 方法等待新的连接进来
                    att.getServer().accept(att, this);

                    Attachment newAtt = new Attachment();
                    newAtt.setServer(server);
                    newAtt.setClient(client);
                    newAtt.setReadMode(true);
                    newAtt.setBuffer(ByteBuffer.allocate(2048));

                    // 这里也可以继续使用匿名实现类，不过代码不好看，所以这里专门定义一个类
                    client.read(newAtt.getBuffer(), newAtt, new ChannelHandler());
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }

            @Override
            public void failed(Throwable t, Attachment att) {
                System.out.println("accept failed");
            }
        });
        // 为了防止 main 线程退出
        try {
            Thread.currentThread().join();
        } catch (InterruptedException e) {
        }
    }
}
/******************************************************/

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.CompletionHandler;
import java.nio.charset.Charset;

public class ChannelHandler implements CompletionHandler<Integer, Attachment> {

    @Override
    public void completed(Integer result, Attachment att) {
        if (att.isReadMode()) {
            // 读取来自客户端的数据
            ByteBuffer buffer = att.getBuffer();
            buffer.flip();
            byte bytes[] = new byte[buffer.limit()];
            buffer.get(bytes);
            String msg = new String(buffer.array()).toString().trim();
            System.out.println("收到来自客户端的数据: " + msg);

            // 响应客户端请求，返回数据
            buffer.clear();
            buffer.put("Response from server!".getBytes(Charset.forName("UTF-8")));
            att.setReadMode(false);
            buffer.flip();
            // 写数据到客户端也是异步
            att.getClient().write(buffer, att, this);
        } else {
            // 到这里，说明往客户端写数据也结束了，有以下两种选择:
            // 1. 继续等待客户端发送新的数据过来
//            att.setReadMode(true);
//            att.getBuffer().clear();
//            att.getClient().read(att.getBuffer(), att, this);
            // 2. 既然服务端已经返回数据给客户端，断开这次的连接
            try {
                att.getClient().close();
            } catch (IOException e) {
            }
        }
    }

    @Override
    public void failed(Throwable t, Attachment att) {
        System.out.println("连接断开");
    }
}
```

```java
//客户端
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.charset.Charset;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

public class AIOClient {

    public static void main(String[] args) throws Exception {
        AsynchronousSocketChannel client = AsynchronousSocketChannel.open();
        // 来个 Future 形式的
        Future<?> future = client.connect(new InetSocketAddress("127.0.0.1",8888));
        // 阻塞一下，等待连接成功
        future.get();

        Attachment att = new Attachment();
        att.setClient(client);
        att.setReadMode(false);
        att.setBuffer(ByteBuffer.allocate(2048));
        byte[] data = "I am obot!".getBytes();
        att.getBuffer().put(data);
        att.getBuffer().flip();

        // 异步发送数据到服务端
        client.write(att.getBuffer(), att, new ClientChannelHandler());

        // 这里休息一下再退出，给出足够的时间处理数据
        Thread.sleep(2000);
    }
}
/*****************************************/

import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.CompletionHandler;
import java.nio.charset.Charset;

public class ClientChannelHandler implements CompletionHandler<Integer, Attachment> {

    @Override
    public void completed(Integer result, Attachment att) {
        ByteBuffer buffer = att.getBuffer();
        if (att.isReadMode()) {
            // 读取来自服务端的数据
            buffer.flip();
            byte[] bytes = new byte[buffer.limit()];
            buffer.get(bytes);
            String msg = new String(bytes, Charset.forName("UTF-8"));
            System.out.println("收到来自服务端的响应数据: " + msg);

            // 接下来，有以下两种选择:
            // 1. 向服务端发送新的数据
//            att.setReadMode(false);
//            buffer.clear();
//            String newMsg = "new message from client";
//            byte[] data = newMsg.getBytes(Charset.forName("UTF-8"));
//            buffer.put(data);
//            buffer.flip();
//            att.getClient().write(buffer, att, this);
            // 2. 关闭连接
            try {
                att.getClient().close();
            } catch (IOException e) {
            }
        } else {
            // 写操作完成后，会进到这里
            att.setReadMode(true);
            buffer.clear();
            att.getClient().read(buffer, att, this);
        }
    }

    @Override
    public void failed(Throwable t, Attachment att) {
        System.out.println("服务器无响应");
    }
}
```

```java
//Attachment
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.nio.channels.AsynchronousSocketChannel;

public class Attachment {
    private AsynchronousServerSocketChannel server;
    private AsynchronousSocketChannel client;
    private boolean isReadMode;
    private ByteBuffer buffer;

    public AsynchronousServerSocketChannel getServer() {
        return server;
    }

    public void setServer(AsynchronousServerSocketChannel server) {
        this.server = server;
    }

    public AsynchronousSocketChannel getClient() {
        return client;
    }

    public void setClient(AsynchronousSocketChannel client) {
        this.client = client;
    }

    public boolean isReadMode() {
        return isReadMode;
    }

    public void setReadMode(boolean readMode) {
        isReadMode = readMode;
    }

    public ByteBuffer getBuffer() {
        return buffer;
    }

    public void setBuffer(ByteBuffer buffer) {
        this.buffer = buffer;
    }

}
```

## 参考内容

[Java 非阻塞 IO 和异步 IO](https://www.javadoop.com/post/nio-and-aio)