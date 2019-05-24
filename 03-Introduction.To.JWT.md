# 浅析 JWT

JSON Web Token，简称 JWT，读音是 [dʒɒt]（ jot 的发音），是一种当下比较流行的「跨域认证解决方案」。注意它是一套 RFC 规范，相关的还有 JWE/JWS/JWK/JOSE。它有很多优点，也有局限性，但我们可以配合其他方案做出适合自己业务的一套方案。本篇是对 JWT 做一个简单的介绍和简单实践总结。

> JSON Web Token (JWT) is a compact claims representation format intended for space constrained environments such as HTTP Authorization headers and URI query parameters.

## JWT的组成
JWT 由三部分组成：头部、数据体、签名/加密。

这三部分以 . (英文句号)连接，注意这三部分顺序是固定的，即 **header.payload.signature** 如下示例：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### 1. 头部 The Header
这部分用来描述 JWT 的元数据，比如该 JWT 所使用的签名/加密算法、媒体类型等。

这部分原始数据是一个JSON对象，经过Base64Url编码方式进行编码后得到最终的字符串。其中只有一个属性是必要的：**alg**——加密/签名算法，默认值为**HS256**。

最简单的头部可以表示成这样：

```json
{
	"alg": "none"
}
```

其他**可选**属性：

- **typ**，描述 JWT 的媒体类型，该属性的值只能是 **JWT**，它的作用是与其他 JOSE Header 混合时表明自己身份的一个参数（很少用到）。
- **cty**，描述 JWT 的内容类型。只有当需要一个 Nested JWT 时，才需要该属性，且值必须是 **JWT**。
- **kid**，KeyID，用于提示是哪个密钥参与加密。

> Base64url 编码是 Base64 的一种针对 URL 的特定变种。因为 = 、+、/ 这个三个字符在 URL 中是有特定含义的，所以 Base64url 分别将 = 直接忽略，+ 替换成 -，/ 替换成 _

### 2. 数据体 The Payload
这部分用来描述JWT的内容数据，即存放些什么。

原始数据仍是一个 JSON 对象，经过 Base64url 编码方式进行编码后得到最终的 Payload。这里的数据默认是不加密的，所以不应存放重要数据（当然你可以考虑使用嵌套型 JWT）。官方内置了七个属性，**大小写敏感**，且都是可选属性，如下：

- **iss (Issuer)** 签发人，即签发该 Token 的主体
- **sub (Subject)** 主题，即描述该 Token 的用途，一般就最为用户的唯一标识
- **aud (Audience)** 作用域，即描述这个 Token 是给谁用的，多个的情况下该属性值为一个字符串数组，单个则为一个字符串
- **exp (Expiration Time)** 过期时间，即描述该 Token 在何时失效
- **nbf (Not Before)** 生效时间，即描述该 Token 在何时生效
- **iat (Issued At)** 签发时间，即描述该 Token 在何时被签发的
- **jti (JWT ID)** 唯一标识

除了这几个内置属性，我们也可以自定义其他属性，自由度非常大。

这里对 aud 做一个说明，有如下 Payload：
```json
{
	"iss": "server1",
	"aud": ["http://www.a.com","http://www.b.com"]
}
```

那么如果我拿这个 JWT 去 http://www.c.com 获取有访问权限的资源，就会被拒绝掉，因为 aud 属性明确了这个 Token 是无权访问 www.c.com 的，有同学会说这部分反正不加密，那我本地把 www.c.com 加入进去不就完事了。别急，下面这部分看完先。

### 3. 签名/加密 The signature/encryption data

这部分是相对比较复杂的，因为 JWT 必须符合 JWS/JWE 这两个规范之一，所以针对这部分的数据如何得来就有两种方式，我们先来看一个简单的例子，有如下 JWT：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJkZW1vIiwibmFtZSI6InhmbHkiLCJhZG1pbiI6dHJ1ZX0.5SHkLkM4KAHtOCtLhSNHOgkFZhPO419ukot1C5bgyUM
```

对前两部分用 Base64url 解码后能得出相应原始数据，

Header 部分：
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Payload 部分：
```json
{
  "sub": "demo",
  "name": "xfly",
  "admin": true
}
```

根据 Header 部分的 alg 属性我们可以知道该 JWT 符合 JWS 中的规范，且签名算法是 HS256 也就是 **HMAC SHA-256** 算法，那么我们就可以根据如下公式计算最后的签名部分：
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

其中的密钥是保证签名安全性的关键，所以必须保存好，在本例中密钥是 123456。**因为有这个密钥的存在，所以即便调用方偷偷的修改了前两部分的内容，在验证环节就会出现签名不一致的情况，所以保证了安全性。**

在实现过程中，遇到了这样一个问题：如果使用 **RS256** 这类非对称加密算法，加密出来的是一串二进制数据，所以第三部分还是用 Base64 编码了一层，这样最终的 JWT 就是可读的了。

## Why JWTs
JWTs 相比于在内存中使用随机 Token 的会话管理方式，其最大优势在于认证逻辑的可扩展性。举个例子，对于认证逻辑，完全可以单独部署，或者使用第三方的认证服务。

而相比于使用数据库进行统一存储和管理 Token 的会话管理方式，其最大优势在于消耗小，不需要频繁调用数据库这类 I/O 耗时操作。

## 安全
1. 因为 JWT 的前两个部分仅是做了 Base64 编码处理并非加密，所以在存放数据上不能存放敏感数据。
2. 用来签名/加密的密钥需要妥善保存。
3. 尽可能采用 HTTPS，确保不被窃听。
4. 如果存放在 Cookie 中则强烈建议开启 Http Only，其实官方推荐是放在 LocalStorage 里，然后通过 Header 头进行传递。

> Cookie 的 HTTP Only 这个 Flag 和 HTTPS 并不冲突，你会发现其实还有一个 Secure 的 Flag，这个就是指 HTTPS 了，这两个 Flag 互不影响的，开启 HTTP Only 会导致前端 JavaScript 无法读取该 Cookie，更多的是为了防止 类 XSS 攻击。

## 问题和思考
JWT 的缺点其实也蛮多的，适不适用得具体看业务场景，哪个优势更大用哪个。（一点感悟：在写这篇文章前一直是 JWT 的坚定拥护者，越写越发现其实传统的 Session-Cookie 方案挺好的，很成熟。它们两者都有优缺点，选型上要多思考斟酌才行。）

### 1. 数据臃肿
因为 payload 只是用 Base64 编码，所以一旦存放数据大了，编码之后 JWT 会很长，cookie 很可能放不下，所以还是建议放 LocalStorage，但是每次 HTTP 请求都带上这个**臃肿的 Header 开销也随之变大**。

### 2. 无法废弃和续签
1. 如果有效期设置过长，意味着这个 Token 泄漏后可以被长期利用，危害较大，所以一般我们都会设置一个较短的有效期。由于有效期较短，意味着需要经常进行**重新授权**的操作。
2. 假设在用户操作过程中升级/变更了某些权限，势必需要**刷新**以更新数据。

要解决这个问题，需要在服务端部署额外逻辑，常见的做法是增加刷新机制和黑名单机制，通过 Refresh Token 刷新 JWT，将需要废弃的 Token 加入到黑名单。

### 3. Token 丢失
如果认证逻辑是在自己服务器上做的话，我们的 JWT secret key 一旦丢失或者泄露那只能通过更换 key 这一种办法了，但这样做的话会导致全部用户都需要重新登录，所以 key 的保管很重要。如果我们的认证逻辑放在第三方服务上，那其实我们就完全不用操心这部分了，很贴心吧 :)

我觉得 JWT 的最大优势在于可以把认证逻辑完全从应用服务中剥离出来，交给第三方 JWT 认证服务或者自己部署的认证服务器上。这样就把用户的账号密码等敏感信息放在单独的服务器上，更容易管理和维护（相比而言应用服务器更容易出现漏洞）。这样做的好处很明显：
1. 应用服务器完全不需要关心用户的账号密码，也不需要关心用户的注册登录，只需要校验 JWT 的合法性即可。
2. 应用服务器不需要存储 JWT 的 key，降低泄露密钥的概率。

## 最佳实践：加密算法使用 RS256 而不是 HS256
假设我们的认证逻辑放在认证服务器上（比如说第三方的认证服务），使用 HS256 算法进行加密，整个过程如下：
	1. 用户通过登录接口，携带账号密码请求认证服务器
	2. 认证服务器校验账号密码，校验失败返回登录失败信息，校验成功则进行下一步
	3. 将用户的一些必要信息（主要是用户的唯一标识）封装成 JWT 的 payload 部分
	4. 将 JWT 的 header、payload 分别进行 Base64url 编码，用 `.` 连接后与密钥一起参与 HS256 加密得到 signature
	5. 将三部分用 `.` 连接后组成最终的 JWT 返回

应用服务器在接收到需要认证的接口请求时，先获取请求中携带的 JWT，然后进行校验，这里我们就会看到一些问题。因为是对称式加密算法，所以加密用的密钥和解密用的密钥必须是同一个，否则是应用服务器是没法做校验的。那么我们只能把密钥在应用服务器上也保存一份，从而增加了密钥泄漏的可能性（当然也只是相比于使用非对称式加密算法而言，毕竟在应用服务器里还有很多其他的 secret key，能丢 JWT 的key，其他 key 也能丢。。。）

那我们使用 RS256 非对称式加密算法就不会丢了吗？是的，至少应用服务器不用背这个锅！因为应用服务器根本就不需要存储这份重要的密钥。

简单科普下非对称式加密算法：有两个密钥，一个公开密钥和一个私有密钥，私钥参与加密，公钥用于解密，巧妙之处是解密只能用公钥来解，即便是加密用的密钥也无法对密文进行解密。你可以看到加密和解密需要两个不同的密钥，故称之为非对称加密。

所以我们应用服务器只需要存一份公开密钥用于校验和解密认证服务器签发的 JWT 即可，即便这个公开密钥泄漏了也没事。因为用公开密钥进行加密的密文再用公开密钥去解密是解不出来的，也就是说我们的应用服务器会认为这个 JWT 是无效的！

## 参考链接
- [JSON Web Token Introduction - jwt.io](https://jwt.io/introduction/)
- [RFC 7519 - JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)
- [RFC 7515 - JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515)
- [RFC 7516 - JSON Web Encryption (JWE)](https://tools.ietf.org/html/rfc7516)
- [RFC 7518 - JSON Web Algorithms (JWA)](https://tools.ietf.org/html/rfc7518)
- [RFC 6749 - The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749#section-4.1.3)
- [Refresh Tokens: When to Use Them and How They Interact with JWTs](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)
- [Cross-origin resource sharing - Wikipedia](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)
- [JSON Web Token 入门教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
- [讲真，别再使用JWT了 – ThoughtWorks洞见](http://insights.thoughtworkers.org/do-not-use-jwt-anymore/)
- [Pros and cons in using JWT (JSON Web Tokens) – Rahul Golwalkar – Medium](https://medium.com/@rahulgolwalkar/pros-and-cons-in-using-jwt-json-web-tokens-196ac6d41fb4)
- [Pros and Cons in Using JWT (JSON Web Tokens) | Hacker News](https://news.ycombinator.com/item?id=12332119)
- [https://blog.angular-university.io/angular-jwt/](https://blog.angular-university.io/angular-jwt/) 
- [https://en.wikipedia.org/wiki/Public-key_cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) 
