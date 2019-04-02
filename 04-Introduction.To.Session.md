# 浅析 Session
Session 的概念其实蛮大的，上下文不同意义也不太一样。比如 OSI Model 里传输层/会话层/应用层也有会话的概念。不过平时我们一般都指应用层上的，也就是 HTTP Session，这类 Session 大体分为服务端会话和客户端会话。

我们都知道 HTTP 是无状态的，也就是客户端和服务端在完成一次 HTTP 请求/响应后就结束了，下一次请求与之前的没有联系。这就导致我们单从请求上没法直接获取到是谁在请求，所以我们必须在每次请求时**携带**一些数据来标识自己的身份，好让服务端能够识别出我们。

## Client-side sessions
一个完整的 web 应用几乎都需要用户系统，也就是说每个用户在登录/授权后发起的每一个请求都能在服务端找出对应用户，且互不混淆。比如常见的购物车模型，A 用户加入某件商品到购物车，不能影响到其他用户的购物车。

由于 HTTP 的无状态特性，要解决这个问题势必需要引入一种会话跟踪机制，以便能把每个请求都能对应上各自的用户身份。最早使用的是 Cookie 方案，因为 Cookie 在浏览器市场的支持率很高，事实上已经是一种标准，所以很自然的就会想到把用户身份通过请求放入 Cookie 中，之后的每个请求都带上该 Cookie，这样就能识别出用户了。

这种通过使用 Cookie 和加密技术来维护管理状态，且不需要在服务端存储数据的方式叫做客户端会话（Client-side sessions）。这种机制在一些环境下能工作的很好，因为足够简单和高效，然而单纯的把数据存在客户端很容易被拥有进入客户端机器的人或软件给篡改，所以要使用这套方案对会话的加密性和完整性有很高的要求：

1. 加密性：除了服务器之外，什么都不应该能够解释会话数据。 
2. 完整性：除了服务器之外，没有任何东西可以操纵会话数据（意外或恶意）。
3. 真实性：除了服务器之外，任何东西都不应该能够启动有效会话。

还有一个在当时不是问题但是现在是个问题的问题就是**跨平台性差**，因为这种模式全靠浏览器的 Cookie，一旦平台或者设备禁用或不支持 Cookie，就只能“认怂”了。

要做到以上三点，想想就头疼，而且你会发现如果将会话放到服务端去管理，上面的几点几乎是迎刃而解。OK，那我们接下来讲讲服务端会话（Server-side sessions）技术。

## Server-side sessions
服务端 Session 是目前非常成熟的会话管理方案，它的基本原理是服务端为每一个 Session 维护一份会话数据，服务端会为每一个客户端发送一个全局唯一的标识，后者通过传递这个唯一标识向服务端表明身份，从而访问响应的会话信息数据。

创建 Session 大致分为三个步骤：
1. 生成全局唯一标识符（SessionID，常见的如 PHP 的  PHPSESSID，Java 的 JSESSIONID 等）。
2. 创建会话数据空间。由于会话操作非常频繁，所以一般会在内存中创建相应的数据结构，但这种情况下，系统一旦出现故障，所有的会话数据就会丢失。稳妥的做法是将其持久化，虽然会增加 I/O 开销，但有利于 Session 的共享和稳定。
3. 将 SessionID 发送给客户端。

传递 SessionID 一般有两种常用方式：Cookie 和 URL重写。
1. Cookie：服务端只要把 SessionID 塞入 Cookie 中发送到客户端，客户端在每次请求时都会带上这个标识符。
2. URL 重写：服务端在响应体中返回 SessionID，而后客户端在请求时将其拼接在 URL 上从而达到传递效果。一般如果客户端禁用了 Cookie 才会使用这种方式。

因为现代浏览器普遍都实现了 LocalStorage/SessionStorage，也就是说客户端除了 Cookie 外也有存储数据的能力，所以你也可以把这个 SessionID 存在本地，然后每次请求时放入自定义 Header 头中。

## 拓展
> 从 HTTP/1.1 开始引入了长连接的机制（keep-alive-mechanism），可以在多个请求/响应间保持连接。HTTP/2.0 更进一步，可以在单次连接中并行多个请求/响应。

一开始我觉得挺纳闷，这长连接机制不是和 HTTP 的无状态是矛盾的吗。一方面 HTTP 在完成一次请求后就结束了，多个请求间是没有关系的，也就是所谓的无状态（stateless）；但另一方面引入的这个长连接机制不就能够让状态保持下去，也就是有状态了。

首先我们要明确，**长连接指的是一次 TCP 连接中可以有多个 HTTP 请求/响应**，详见 [HTTP persistent connection - Wikipedia](https://en.wikipedia.org/wiki/HTTP_persistent_connection)。所以对于单个 HTTP 请求/响应依然是无状态的。

## 参考链接
- [PHP: Sessions - Manual](https://www.php.net/manual/zh/book.session.php)
- [HTTP cookie - Wikipedia](https://en.wikipedia.org/wiki/HTTP_cookie)
- [Hypertext Transfer Protocol - Wikipedia](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#HTTP_session)
- [Session (computer science) - Wikipedia](https://en.wikipedia.org/wiki/Session_(computer_science))
