# Netty的踩坑

[toc]

## http的长连接

初学者比如我，编写代码并不是很熟练，就会出现这个问题。设置了http的响应报文的version是1.1，那么我们知道这个版本的http是支持长连接的，这也就导致在我没有关闭连接的时候，客户端就会持续等待。

解决方法就是`ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);`对write之后调用close监听器。

## 解析post参数

真的好坑，一般我们在学习的时候会一步步打印一个对象里的内容，这也就导致了我这个问题的产生。要注意，有一些只能调用一次，再次调用就无效了，就比如我的这个问题。

在解析post请求的时候，我先写了下面这个方法，作用是获得post请求体的所有内容

```java
private String getPostMethodContent(FullHttpRequest fullHttpRequest) {
    ByteBuf bf = fullHttpRequest.content();
    byte[] byteArray = new byte[bf.capacity()];
    bf.readBytes(byteArray);
    return new String(byteArray);
}
```

之后，我又尝试调用`HttpPostRequestDecoder`来解析post请求的参数，问题就产生，怎么就读不到参数呢？

解决方案：就是注释掉上面的方法，这样就可以获得解析的参数了。

```java
Map<String, String> parmMap = new HashMap<>();
HttpPostRequestDecoder decoder = new HttpPostRequestDecoder(req);
decoder.offer(req);
List<InterfaceHttpData> parmList = decoder.getBodyHttpDatas();

for (InterfaceHttpData parm : parmList) {
    Attribute data = (Attribute) parm;
    parmMap.put(data.getName(), data.getValue());
}
System.out.println(parmMap);
```