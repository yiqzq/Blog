# Netty的Http服务器的服务端实践

[toc]

## Server.java 服务端的启动

```java
package mynetty1;

import com.sun.org.apache.xpath.internal.SourceTree;
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;

import javax.sound.sampled.Port;
import java.net.InetAddress;
import java.net.InetSocketAddress;

/**
 * @author yiqzq
 * @date 2020/4/22 13:26
 */
public class Server {

    static int port = 8888;

    public static void main(String[] args) {
        EventLoopGroup bossgroup = new NioEventLoopGroup();
        EventLoopGroup workgroup = new NioEventLoopGroup();
        ServerBootstrap b = new ServerBootstrap();
        b.group(bossgroup, workgroup).handler(new LoggingHandler(LogLevel.INFO))
                .channel(NioServerSocketChannel.class)
                .childHandler(new HttpServerInitializer());
        ChannelFuture f = null;
        try {
            f = b.bind(port).sync();
            System.out.println("服务端已经启动,并绑定端口!");
            f.channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            bossgroup.shutdownGracefully();
            workgroup.shutdownGracefully();
            System.out.println("服务器已经关闭资源");
        }
    }

}

```

## HttpServerInitializer.java 用于初始化channelhandler

```java
package mynetty1;

import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;

/**
 * @author yiqzq
 * @date 2020/4/22 16:29
 */
public class HttpServerInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        //或者使用HttpRequestDecoder & HttpResponseEncoder
        pipeline.addLast(new HttpServerCodec());
        pipeline.addLast(new HttpObjectAggregator(1024*1024));
        pipeline.addLast(new MyHttpHandler());
    }
}
```

## MyHttpHandler.java 简单处理了Get方法和Post方法的参数获取

```java
package mynetty1;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.handler.codec.http.*;
import io.netty.handler.codec.http.multipart.*;
import io.netty.util.CharsetUtil;
import io.netty.util.concurrent.EventExecutorGroup;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.InetAddress;
import java.net.SocketAddress;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author yiqzq
 * @date 2020/4/22 16:43
 */
public class MyHttpHandler extends SimpleChannelInboundHandler<FullHttpRequest> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, FullHttpRequest req) throws Exception {
        SimpleDateFormat format = new SimpleDateFormat("yyyy年MM月dd日 HH时mm分ss秒");
        Date date = new Date();
        String uri = req.uri();
        HttpMethod method = req.method();
        System.out.println("服务器时间: " + format.format(date) + " 请求的uri是: " + uri);

        if (method.equals(HttpMethod.GET)) {
            System.out.println("使用的是get方法");
            Map<String, List<String>> params = getGetParameters(req);
//            System.out.println(params);
            StringBuilder msgBuilder = new StringBuilder("请问发送的是");
            for (Map.Entry<String, List<String>> attr : params.entrySet()) {
                for (String str : attr.getValue()) {
                    msgBuilder.append(attr.getKey()).append(":").append(str).append(",");
                }
            }
            msgBuilder.append("吗?");
            String msg = msgBuilder.toString();
            HttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1
                    , HttpResponseStatus.OK
                    , Unpooled.copiedBuffer(msg, CharsetUtil.UTF_8));
            ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);

        } else if (method.equals(HttpMethod.POST)) {
            System.out.println("使用的是post方法");
            System.out.println(req);
//            System.out.println("正文:" + getPostMethodContent(req));
//            String name = getPostParameter(req, "name");
//            System.out.println("name:"+name);
            Map<String, String> parameters = getPostParameters(req);
            StringBuilder msgBuilder = new StringBuilder("请问发送的是");
            for (Map.Entry<String, String> it : parameters.entrySet()) {
                msgBuilder.append(it.getKey()).append(":").append(it.getValue());
            }
            msgBuilder.append("吗?");
            String msg = msgBuilder.toString();


            HttpResponse response = new DefaultFullHttpResponse(
                    HttpVersion.HTTP_1_1
                    , HttpResponseStatus.OK
                    , Unpooled.copiedBuffer(msg, CharsetUtil.UTF_8));

            ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);
        } else {

            String msg = "<html><head><title>test</title></head><body>" +
                    "不支持的传输方式,请使用get/post方式<br>你请求的uri为：" + uri +
                    "</body></html>";
            HttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1
                    , HttpResponseStatus.OK
                    , Unpooled.copiedBuffer(msg, CharsetUtil.UTF_8));
            response.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/html; charset=UTF-8");
            ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);
        }

    }

    private Map<String, List<String>> getGetParameters(FullHttpRequest fullHttpRequest) {
        QueryStringDecoder decoder = new QueryStringDecoder(fullHttpRequest.uri());
        Map<String, List<String>> uriAttributes = decoder.parameters();
        //此处仅打印请求参数（你可以根据业务需求自定义处理）
        for (Map.Entry<String, List<String>> attr : uriAttributes.entrySet()) {
            for (String attrVal : attr.getValue()) {
//                System.out.println(attr.getKey() + "=" + attrVal);
            }
        }
        return uriAttributes;
    }

    private String getPostContent(FullHttpRequest fullHttpRequest) {
        ByteBuf bf = fullHttpRequest.content();
        byte[] byteArray = new byte[bf.capacity()];
        bf.readBytes(byteArray);
        return new String(byteArray);
    }


    public static String getPostParameter(FullHttpRequest req, String name) throws IOException {
        HttpPostRequestDecoder decoder = new HttpPostRequestDecoder(req);
        decoder.offer(req);
        InterfaceHttpData parm = decoder.getBodyHttpData(name);
        Attribute data = (Attribute) parm;
        return data.getValue();
    }

    public static Map<String, String> getPostParameters(FullHttpRequest req) throws IOException {

        Map<String, String> parmMap = new HashMap<>();
        HttpPostRequestDecoder decoder = new HttpPostRequestDecoder(req);
        decoder.offer(req);

        List<InterfaceHttpData> parmList = decoder.getBodyHttpDatas();

        for (InterfaceHttpData parm : parmList) {
            Attribute data = (Attribute) parm;
            parmMap.put(data.getName(), data.getValue());
        }
//        System.out.println(parmMap);
        return parmMap;
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("连接的客户端地址: " + ctx.channel().remoteAddress());
        System.out.println("当前主机是: " + InetAddress.getLocalHost().getHostName());
    }
}
```