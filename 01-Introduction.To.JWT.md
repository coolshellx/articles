# 浅析 JWT
JSON Web Token，简称 JWT，读音是 [dʒɒt]（ jot 的发音），是一种当下比较流行的「跨域认证解决方案」。注意它是一套 RFC 规范，相关的还有 JWE/JWS/JWK/JOSE。它有很多优点，也有局限性，但我们可以配合其他方案做出适合自己业务的一套方案。比如 JWT 的硬伤——无法在服务端让核发出去的 Token 失效，我们就可以配合 OAuth 规范中的 Refresh Token 来弥补这块（下文会讲到）。

> JSON Web Token (JWT) is a compact claims representation format intended for space constrained environments such as HTTP Authorization headers and URI query parameters.  —— RFC 7519

## Why use it
我们为什么要用它，或者说它好在哪里？一般我们都拿它和传统的基于 Session-Cookie 的管理方式进行大致比较。

传统的基于 Session 的会话管理逻辑大致如下时序图所示：

[![基于 Session 的会话管理时序图](https://ws1.sinaimg.cn/large/e2bc63f6ly1g17u0kad4vj20iu0n075b.jpg "基于 Session 的会话管理时序图")](https://ws1.sinaimg.cn/large/e2bc63f6ly1g17u0kad4vj20iu0n075b.jpg "基于 Session 的会话管理时序图")

看着感觉很顺畅没什么问题，但在实践过程中逐渐暴露出一些问题：

1. 频繁查找 Session 的开销过大。因为 Session 存储在服务端，大部分接口的请求都需要查找 Session 以获取对应用户身份。不管是存储在持久层（数据库）还是内存中，频繁查找带来的压力会随着用户量的上升而急剧增大。
2. 不支持跨域，可扩展性差。举个例子：假设网站A和网站B的用户数据是共享的，当用户在网站A上登录以后，我们希望在访问网站B时也保持登录状态。这个时候就会出现一个情况：生成这个 SessionID 的服务器并不是验证这个 SessionID 的服务器，也就出现了跨域身份认证的问题。除非我们把身份认证的数据也共享，即将 Session 放在持久层单独存储，统一管理，这样就能在多域名下共享了，但是这样做的成本有点高。
3. 安全性较差。Session 放在 Cookie 中容易被 CSRF 攻击，而且在多域名的业务场景下需要额外的做兼容性处理，容易出现安全漏洞。

针对以上这些问题，我们会发现将身份凭证放在服务端管理实在是一件吃力不讨好的事情，就想能不能把这个凭证放在客户端那里。所以这种**无状态**的身份认证方案就出来了，而 JWT 就是其中一个典型代表。

之所以服务端可以不用存储身份凭证，是因为这个凭证是通过特定结构和加密算法组装起来的，服务端在验证的时候只需要通过对应密钥和算法直接验证用户身份，不需要额外查询操作。所以在扩展性上变得非常易于扩展，多项目的情况下只需要保证生成 Token 的机制相同且加解密用的密钥一致即可。

另外在调用接口时将 JWT 放在 URL 或者 Header 头中避免了 Cookie 带来的一些不良因素。

但这个优势同样也是其劣势，签发出去的所有 Token 在其有效期内，服务端是无法将其废止的。因为生成机制不变这些 Token 就都是有效的。要解决这个问题也很简单，配合 Refresh Token 就能很好的解决，后面再讲。

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

（这里的算法包括加密和签名，牵扯到另外好几个规范，如 JWA/JWS/JWE/JWK…但篇幅和精力有限，有兴趣的可以去了解一下，底部有相关链接，都是 RFC，非常原汁原味）

其他**可选**属性包含 **typ** 和 **cty**：

- **typ**，描述 JWT 的媒体类型，该属性的值只能是 **JWT**，它的作用是与其他 JOSE Header 混合时表明自己身份的一个参数（很少用到）。
- **cty**，描述 JWT 的内容类型。只有当需要一个 Nested JWT 时，才需要该属性，且值必须是 **JWT**。

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

这部分是相对比较复杂的，因为 JWT 必须符合 JWS/JWE 这两个规范之一，所以针对这部分的数据如何得来就有两种方式，我们来看一个简单的例子，有如下 JWT：

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

其中的密钥是保证签名安全性的关键，所以必须保存好，在本例中密钥是 123456。因为有这个密钥的存在，所以即便调用方偷偷的修改了前两部分的内容，在验证环节就会出现签名不一致的情况，所以保证了安全性。

一开始我们就说到，由于服务器不再存储凭证，导致的问题就是服务器一旦签发出去，这个 JWT 就只能等它自己失效或者使用黑名单机制禁止掉。但是一旦黑名单体积过大，这块开销也是蛮大的，但是我们使用 JWT 就是不想在服务器存储凭证，怎么办？借鉴一下其他方案，也就是 OAuth 的 Refresh Token 机制。

## Access Token 和 Refresh Token
一般对于 JWT 的使用基本属于 Access Token 的范畴。Access Token 用于获取某个需要访问权限的资源，Refresh Token 用于获取一个新的 Access Token（效果上看相当于刷新）。

我们都明白为什么需要 Access Token，但为什么还需要 Refresh Token？

1. 就像登录会过期，对于一些安全性要求较高的场景，一般我们都会将 Access Token 设置一个较短的有效期，以防泄漏后被长期利用。由于有效期较短，意味着需要经常进行**重新授权**的操作。
2. 假设在用户操作过程中升级/变更了某些权限，势必需要刷新以关联新的权限。

但以上两个原因都会出现一个问题：**重新授权**的操作，对用户来说是**有感知**的被动操作，是极其不友好的体验。举个例子，你在使用一个 APP 的时候老是让你重新登录/授权，非得气炸不可。

为了提高用户体验，减少用户授权认证的次数，同时降低 Access Token 的外泄（Leak）的伤害期，Refresh Token 就显得很重要了。因为担负着刷新的重任，所以有效期相比 Access Token 会长很多，有些会设置一个月（比如微信），有些会设置半年（比如谷歌）。所以一旦 Refresh token 也过期，那么只能让用户重新授权了。

[![](https://ws1.sinaimg.cn/large/e2bc63f6ly1g17u0kaetlj20zk0m8q4o.jpg)](https://ws1.sinaimg.cn/large/e2bc63f6ly1g17u0kaetlj20zk0m8q4o.jpg)

但是仔细一想又会发现这个 Refresh Token 不是一样会出现和 Access Token 的安全泄漏问题嘛，而且这两个 Token 都是给到客户端的，如果 Access Token 被泄漏，极有可能 Refresh Token 也连带着泄漏出去了。而一旦 Refresh Token 被泄漏了，要么被授权服务器拉黑处理掉，要么自身有效期到期，否则危害更大。

所以我们对 Refresh Token 的存储需要有非常高的安全保护措施，加密传输，加密存储。传输上建议使用 TSL，并且只允许在授权服务器和被绑定的客户端之间流通。举个例子：授权服务器 S 核发了一个 Refresh Token RT 给客户端 C，首先这个 RT 是绑定在 C 上的，也就是如果来自其他客户端用这个 RT 是无效的。

## 总结
没有哪一个技术方案能成为普适方案，啥都好用/管用，在用户会话管理场景下也是如此，「没有银弹」。如果业务不涉及跨域，其实 Session-Cookie 的方案就足够了，何况现在 Session 的外部存储方案也比较成熟。

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
- [讲真，别再使用JWT了 – ThoughtWorks洞见](http://insights.thoughtworkers.org/do-not-use-jwt-anymore/)
- [JSON Web Token 入门教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
