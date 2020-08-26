\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.jianshu.com\](https://www.jianshu.com/p/576dbf44b2ae)

[![](https://upload.jianshu.io/users/upload_avatars/1821058/f2a247ca2423?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/d286ee5ad51e)

142016.04.22 16:10:30 字数 2,015 阅读 411,409

> Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于 JSON 的开放标准（[(RFC 7519](https://link.jianshu.com/?t=https://tools.ietf.org/html/rfc7519)). 该 token 被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT 的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该 token 也可直接被用于认证，也可被加密。

起源
--

说起 JWT，我们应该来谈一谈基于 token 的认证和传统的 session 认证的区别。

### 传统的 session 认证

我们知道，http 协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据 http 协议，我们并不能知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为 cookie, 以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了, 这就是传统的基于 session 认证。

但是这种基于 session 的认证使应用本身很难得到扩展，随着不同客户端用户的增加，独立的服务器已无法承载更多的用户，而这时候基于 session 认证应用的问题就会暴露出来.

#### 基于 session 认证所显露的问题

**Session**: 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言 session 都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。

**扩展性**: 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上, 这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。

**CSRF**: 因为是基于 cookie 来进行用户识别的, cookie 如果被截获，用户就会很容易受到跨站请求伪造的攻击。

基于 token 的鉴权机制
--------------

基于 token 的鉴权机制类似于 http 协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于 token 认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。

流程上是这样的：

*   用户使用用户名密码来请求服务器
*   服务器进行验证用户的信息
*   服务器通过验证发送给用户一个 token
*   客户端存储 token，并在每次请求时附送上这个 token 值
*   服务端验证 token 值，并返回数据

这个 token 必须要在每次请求时传递给服务端，它应该保存在请求头里， 另外，服务端要支持`CORS(跨来源资源共享)`策略，一般我们在服务端这么做就可以了`Access-Control-Allow-Origin: *`。

那么我们现在回到 JWT 的主题上。

JWT 长什么样？
---------

JWT 是由三段信息构成的，将这三段信息文本用`.`链接一起就构成了 Jwt 字符串。就像这样:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

JWT 的构成
-------

第一部分我们称它为头部（header), 第二部分我们称其为载荷（payload, 类似于飞机上承载的物品)，第三部分是签证（signature).

### header

jwt 的头部承载两部分信息：

*   声明类型，这里是 jwt
*   声明加密的算法 通常直接使用 HMAC SHA256

完整的头部就像下面这样的 JSON：

```
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```

然后将头部进行 base64 加密（该加密是可以对称解密的), 构成了第一部分.

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

### playload

载荷就是存放有效信息的地方。这个名字像是特指飞机上承载的货品，这些有效信息包含三个部分

*   标准中注册的声明
*   公共的声明
*   私有的声明

**标准中注册的声明** (建议但不强制使用) ：

*   **iss**: jwt 签发者
*   **sub**: jwt 所面向的用户
*   **aud**: 接收 jwt 的一方
*   **exp**: jwt 的过期时间，这个过期时间必须要大于签发时间
*   **nbf**: 定义在什么时间之前，该 jwt 都是不可用的.
*   **iat**: jwt 的签发时间
*   **jti**: jwt 的唯一身份标识，主要用来作为一次性 token, 从而回避重放攻击。

**公共的声明** ：  
公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息. 但不建议添加敏感信息，因为该部分在客户端可解密.

**私有的声明** ：  
私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为 base64 是对称解密的，意味着该部分信息可以归类为明文信息。

定义一个 payload:

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

然后将其进行 base64 加密，得到 Jwt 的第二部分。

```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```

### signature

jwt 的第三部分是一个签证信息，这个签证信息由三部分组成：

*   header (base64 后的)
*   payload (base64 后的)
*   secret

这个部分需要 base64 加密后的 header 和 base64 加密后的 payload 使用`.`连接组成的字符串，然后通过 header 中声明的加密方式进行加盐`secret`组合加密，然后就构成了 jwt 的第三部分。

```
// javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);

var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

将这三部分用`.`连接成一个完整的字符串, 构成了最终的 jwt:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

**注意：secret 是保存在服务器端的，jwt 的签发生成也是在服务器端的，secret 就是用来进行 jwt 的签发和 jwt 的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个 secret, 那就意味着客户端是可以自我签发 jwt 了。**

### 如何应用

一般是在请求头里加入`Authorization`，并加上`Bearer`标注：

```
fetch('api/user/1', {
  headers: {
    'Authorization': 'Bearer ' + token
  }
})
```

服务端会验证 token，如果验证通过就会返回相应的资源。整个流程就是这样的:

![](http://upload-images.jianshu.io/upload_images/1821058-2e28fe6c997a60c9.png)

jwt-diagram

总结
--

### 优点

*   因为 json 的通用性，所以 JWT 是可以进行跨语言支持的，像 JAVA,JavaScript,NodeJS,PHP 等很多语言都可以使用。
*   因为有了 payload 部分，所以 JWT 可以在自身存储一些其他业务逻辑所必要的非敏感信息。
*   便于传输，jwt 的构成非常简单，字节占用很小，所以它是非常便于传输的。
*   它不需要在服务端保存会话信息, 所以它易于应用的扩展

### 安全相关

*   不应该在 jwt 的 payload 部分存放敏感信息，因为该部分是客户端可解密的部分。
*   保护好 secret 私钥，该私钥非常重要。
*   如果可以，请使用 https 协议

[![](https://upload.jianshu.io/users/upload_avatars/1821058/f2a247ca2423?imageMogr2/auto-orient/strip|imageView2/1/w/100/h/100/format/webp)](https://www.jianshu.com/u/d286ee5ad51e)

[Dearmadman](https://www.jianshu.com/u/d286ee5ad51e "Dearmadman")Beautiful work by passionate people.

总资产 17 (约 1.56 元) 共写了 11.1W 字获得 1,289 个赞共 932 个粉丝

### 被以下专题收入，发现更多相似内容

### 推荐阅读[更多精彩内容](https://www.jianshu.com/)

*   1\. 微服务架构介绍 1.1 什么是微服务架构？ 形像一点来说，微服务架构就像搭积木，每个微服务都是一个零件，并使...
    
    [![](https://cdn2.jianshu.io/assets/default_avatar/10-e691107df16746d4a9f3fe9496fd1848.jpg)静修佛缘](https://www.jianshu.com/u/d6e5ad19fcef)阅读 28
    
    [![](https://upload-images.jianshu.io/upload_images/1775814-8884b980e74daa5b?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/15d0a1c366b3)
*   Spring Cloud 为开发人员提供了快速构建分布式系统中一些常见模式的工具（例如配置管理，服务发现，断路器，智...
    
    [![](https://upload-images.jianshu.io/upload_images/7328262-54f7992145380c10.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/46fd0faecac1)
*   注：参考 Spring Security 整合 JSON Web Token(JWT) 提升 REST 安全性，写的...
    
    [![](https://upload-images.jianshu.io/upload_images/3145530-7a18f4fd40be4d85.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240/format/webp)](https://www.jianshu.com/p/beea4bc9056b)