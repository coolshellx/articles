# 浅析 JWT
JSON Web Token，简称 JWT，读音是 [dʒɒt]（ jot 的发音），是一种当下比较流行的「跨域认证解决方案」。注意它是一套 RFC 规范，相关的还有 JWE/JWS/JWK/JOSE。它有很多优点，也有局限性，但我们可以配合其他方案做出适合自己业务的一套方案。本篇是对 JWT 做一个简单的介绍和简单实践总结。

> JSON Web Token (JWT) is a compact claims representation format intended for space constrained environments such as HTTP Authorization headers and URI query parameters.  —— RFC 7519

## What is it
JWT 由三部分组成：the header（头部），the payload（数据体），the signature/encryption data（签名/加密）。

这三部分以 dot  `.` 连接，注意这三部分顺序是固定的，如下示例：
```
# header.payload.signature
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

> Base64url 编码是 Base64 的一种针对 URL 的特定变种。由于 `=` , `+` , `/` 这个三个字符在 URL 中是有特定含义的，所以 Base64url 分别将 `=` 直接忽略，`+` 替换成 `-`，`/` 替换成 `_`。

### The Header

这部分用来描述 JWT 的元数据，即描述该 JWT 使用的签名/加密算法和媒体类型等。

原始数据是一个 JSON 对象，经过 Base64url 编码方式进行编码后得到最终的 Header。必要属性只有一个：**alg**，加密/签名 JWT 时使用的算法，默认算法为 [**HS256**](https://en.wikipedia.org/wiki/HMAC)。比如最简单的 Header 如下，表示该 JWT 是未加密的：
```json
{
	"alg": "none"
}
```

其他**可选**属性：

- **typ**，描述 JWT 的媒体类型，该属性的值只能是 **JWT**，它的作用是与其他 JOSE Header 混合时表明自己身份的一个参数（很少用到）。
- **cty**，描述 JWT 的内容类型。只有当需要一个 Nested JWT 时，才需要该属性，且值必须是 **JWT**。
- **kid**，KeyID，用于提示是哪个密钥参与加密。

### The Payload

这部分用来描述 JWT 的内容数据，即该 JWT 中存放的数据。

原始数据仍是一个 JSON 对象，经过 Base64url 编码方式进行编码后得到最终的 Payload。这里的数据默认是不加密的，所以不应存放重要数据（当然你可以考虑使用嵌套型 JWT）。官方内置了七个属性，**大小写敏感**，且都是可选属性，如下：

- **iss (Issuer)** 签发人，即签发该 Token 的主体
- **sub (Subject)** 主题，即描述该 Token 的用途
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

那么如果我拿这个 JWT 去 `http://www.c.com` 获取有访问权限的资源，就会被拒绝掉，因为 aud 属性明确了这个 Token 是无权访问 `www.c.com` 的，有同学会说这部分反正不加密，那我本地把 `www.c.com` 加入进去不就完事了。别急，下面这部分看完先。

### The signature/encryption data

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

在实现过程中，遇到了这样一个问题：如果使用 RS256 这类非对称加密算法，加密出来的是一串二进制数据，所以第三部分还是用 Base64 编码了一层，这样最终的 JWT 就是可读的了。

## Why use it
1. Stateless 无状态，一方面可以有效减少服务端保存 Session 的负载；另一方面可以方便的进行扩平台的横向扩展，如 SSO 单点授权。
2. 可以有效携带**必要但不敏感**的信息，且是 JSON 这种非常通用的格式。

一般我们都拿它和传统的基于 Session-Cookie 的管理方式进行大致比较。

传统的基于 Session 的会话管理逻辑大致如下时序图所示：

![](https://ws1.sinaimg.cn/large/e2bc63f6ly1g1f457524fj20iu0n075b.jpg)

相比较而言，传统的 Session-Cookie 方式会有几点问题：
1. 频繁查找 Session 的开销过大。因为 Session 存储在服务端，大部分接口的请求都需要查找 Session 以获取对应用户身份。不管是存储在持久层（数据库）还是内存中，频繁查找带来的压力会随着用户量的上升而急剧增大。
2. 不支持跨域，可扩展性差。举个例子：假设网站A和网站B的用户数据是共享的，当用户在网站A上登录以后，我们希望在访问网站B时也保持登录状态。这个时候就会出现一个情况：生成这个 SessionID 的服务器并不是验证这个 SessionID 的服务器，也就出现了跨域身份认证的问题。除非我们把身份认证的数据也共享，即将 Session 放在持久层单独存储，统一管理，这样就能在多域名下共享了，但是这样做的成本有点高。
3. 安全性较差。Session 放在 Cookie 中容易被 CSRF 攻击，而且在多域名的业务场景下需要额外的做兼容性处理，容易出现安全漏洞。

## Security
1. 因为 JWT 的前两个部分仅是做了 Base64 编码处理并非加密，所以在存放数据上不能存放敏感数据。
2. 用来签名/加密的密钥需要妥善保存。
3. 尽可能采用 HTTPS，确保不被窃听。
4. 如果存放在 Cookie 中则强烈建议开启 Http Only，其实官方推荐是放在 LocalStorage 里，然后通过 Header 头进行传递。

## 一些问题和思考
JWT 的缺点其实也蛮多的，适不适用得具体看业务场景，哪个优势更大用哪个。（一点感悟：在写这篇文章前一直是 JWT 的坚定拥护者，越写越发现其实传统的 Session-Cookie 方案挺好的，很成熟。它们两者都有优缺点，选型上要多思考斟酌才行。）

1. 数据臃肿
2. 无法废弃和续签

### 数据臃肿
因为 payload 只是用 Base64 编码，所以一旦存放数据大了，编码之后 JWT 会很长，cookie 很可能放不下，所以还是建议放 LocalStorage，但是每次 HTTP 请求都带上这个臃肿的 Header 开销也随之变大。

### 无法废弃和续签
1. 如果有效期设置过长，意味着这个 Token 泄漏后可以被长期利用，危害较大，所以一般我们都会设置一个较短的有效期。由于有效期较短，意味着需要经常进行**重新授权**的操作。
2. 假设在用户操作过程中升级/变更了某些权限，势必需要刷新以更新数据。

要解决这个问题，需要在服务端部署额外逻辑，常见的做法是增加刷新机制和黑名单机制，通过 Refresh Token 刷新 JWT，将需要废弃的 Token 加入到黑名单。

****
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
