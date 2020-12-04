# 关于StringDecoder和StringEncoder的作用

[toc]

如果没有StringDecoder和StringEncoder配置在pipeline中,那么每次操作内容都需要下方这么麻烦

```java
public class MyChannelHandler extends SimpleChannelInboundHandler {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        //先解码
        ByteBuf buf = (ByteBuf) msg;
        byte[] request = new byte[buf.readableBytes()];
        buf.readBytes(request);
        String response = new String(request, "UTF-8");
        //编码写入ByteBuf
        ByteBuf rep = Unpooled.copiedBuffer(response.getBytes());
        ctx.writeAndFlush(rep);
    }
}
```

因此，netty为此内置了StringDecoder和StringEncoder，使得不需要自己编写这部分的内容

```java
public class ServerInitializer extends ChannelInitializer<SocketChannel> {
    private static final StringDecoder DECODER = new StringDecoder();
    private static final StringEncoder ENCODER = new StringEncoder();

    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();

        pipeline.addLast(DECODER);
        pipeline.addLast(ENCODER);

    }
}
```

只需要像上述那样配置即可。