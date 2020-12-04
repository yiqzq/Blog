# HTTP协议的Post请求的提交方式

[toc]

今天在学习Netty的Http协议的时候，对Http协议有了更加深刻的理解。Http毕竟只是一种协议，具体的实现还是要依靠开发人员来实现的。

就拿这篇文章所要说的Post请求为例。

我们知道Post请求的为了消息的安全性，是将请求消息放在body中，不能够像get一样在url中传输。但是在body中传输就会涉及到一个以何种方式来提交数据，这主要涉及到了请求头中的`Content-Type`字段。

下面对所涉到的可选属性进行举例。

注意：下面所有的例子都是采用postman来测试的。

## Content-Type: application/x-www-form-urlencoded

窗体数据被编码为名称/值对，这是标准且默认的编码格式。

这种编码方法会把多个传输的数据，中间用`&`来连接。

下图是要发送的数据

<img src="https://i.loli.net/2020/04/22/OQnHPstBLdgqraN.png" alt="image-20200422224311164" style="zoom:80%;" />

下图就是java后台所接受到的格式，可以看到和之前描述的一样，正文采用`&`来连接多个数据，其实这也是get方法的编码格式，仔细看，这难道不就是get传输数据的方法，只不过get为了区分数据和url采用`?`来区分。

<img src="https://i.loli.net/2020/04/22/UM6y17ACkswXhBH.png" alt="image-20200422224421908" style="zoom:80%;" />

## Content-Type: multipart/form-data

使用表单上传文件时，必须指定表单的 enctype属性值为 multipart/form-data，才能够使用这种编码格式。在这种格式中，请求体被分割成多部分，每部分使用 --boundary分割。一般这个格式被用于传输文件。

这里`Content-Type`告诉我们，发包是以`multipart/form-data`格式来传输，另外，还有`boundary`用于分割数据。当文件太长，HTTP无法在一个包之内发送完毕，就需要分割数据，分割成一个一个chunk发送给服务端。

<img src="https://i.loli.net/2020/04/22/jIRNnoMqmh9ByAi.png" alt="image-20200422224724068" style="zoom:80%;" />

<img src="https://i.loli.net/2020/04/22/uA4yYKC9oXwVW5x.png" alt="image-20200422224658790" style="zoom:80%;" />



## Content-Type: application/json

application/x-www-form-urlencoded这种编码方式用来处理简单的请求是没有问题的，但是如果有一些嵌套的请求出现，那么最好是使用json这种格式，也就是设置Content-Type为application/json。

<img src="https://i.loli.net/2020/04/22/aVUvdRXkwJB57SN.png" alt="image-20200422225408377" style="zoom:80%;" />



<img src="https://i.loli.net/2020/04/22/TgjV2Ue1LMyQ6PW.png" alt="image-20200422225457422" style="zoom:80%;" />

## Content-Type: text/xml

不知道为啥我的postman没有这个选项，这里就不展示了。

下面是一张来源于网上的图

<img src="https://i.loli.net/2020/04/22/tdQiC2WeTJf4uIE.png" alt="image-20200422230835116" style="zoom:80%;" />

当然提交方式不止这4种，还有很多，这里只是讲述了常见的4种类型。