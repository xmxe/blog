
---
title: JWT-Json Web Token
categories: 技术栈
img: https://pic1.zhimg.com/v2-58b976a41c44871cd549914566d602de_1440w.jpg

---

# JWT基础概念详解

## 什么是JWT?

JWT（JSON Web Token）是目前最流行的跨域认证解决方案，是一种基于Token的认证授权机制。从JWT的全称可以看出，JWT本身也是Token，一种规范化之后的JSON结构的Token。JWT自身包含了身份验证所需要的所有信息，因此，我们的服务器不需要存储Session信息。这显然增加了系统的可用性和伸缩性，大大减轻了服务端的压力。可以看出，**JWT更符合设计RESTful API时的「Stateless（无状态）」原则**。并且，使用JWT认证可以有效避免CSRF攻击，因为JWT一般是存在在local Storage中，使用JWT进行身份验证的过程中是不会涉及到Cookie的。下面是[RFC7519](https://tools.ietf.org/html/rfc7519)对JWT做的较为正式的定义。

> JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties. The claims in a JWT are encoded as a JSON object that is used as the payload of a JSON Web Signature (JWS) structure or as the plaintext of a JSON Web Encryption (JWE) structure, enabling the claims to be digitally signed or integrity protected with a Message Authentication Code (MAC) and/or encrypted. ——[JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)

## JWT由哪些部分组成？

![此图片来源于：https://supertokens.com/blog/oauth-vs-jwt](https://oss.javaguide.cn/javaguide/system-design/jwt/jwt-composition.png)

JWT本质上就是一组字串，通过（`.`）切分成三个为Base64编码的部分：

- **Header**：描述JWT的元数据，定义了生成签名的算法以及Token的类型。
- **Payload**：用来存放实际需要传递的数据
- **Signature（签名）**：服务器通过Payload、Header和一个密钥(Secret)使用Header里面指定的签名算法（默认是 HMAC SHA256）生成。

JWT通常是这样的：`xxxxx.yyyyy.zzzzz`。示例：


```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

你可以在[jwt.io](https://jwt.io/)这个网站上对其JWT进行解码，解码之后得到的就是Header、Payload、Signature这三部分。Header和Payload都是JSON格式的数据，Signature由Payload、Header和Secret(密钥)通过特定的计算公式和加密算法得到。

![img](https://oss.javaguide.cn/javaguide/system-design/jwt/jwt.io.png)

### Header

Header通常由两部分组成：

- typ（Type）：令牌类型，也就是JWT。
- alg（Algorithm）：签名算法，比如HS256。

示例：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

JSON形式的Header被转换成Base64编码，成为JWT的第一部分。

### Payload

Payload也是JSON格式数据，其中包含了Claims(声明，包含JWT的相关信息)。Claims分为三种类型：

- **Registered Claims（注册声明）**：预定义的一些声明，建议使用，但不是强制性的。
- **Public Claims（公有声明）**：JWT签发方可以自定义的声明，但是为了避免冲突，应该在[IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml)中定义它们。
- **PrivateClaims（私有声明）**：JWT签发方因为项目需要而自定义的声明，更符合实际项目场景使用。

下面是一些常见的注册声明：

- iss（issuer）：JWT签发方。
- iat（issuedattime）：JWT签发时间。
- sub（subject）：JWT主题。
- aud（audience）：JWT接收方。
- exp（expirationtime）：JWT的过期时间。
- nbf（notbeforetime）：JWT生效时间，早于该定义的时间的JWT不能被接受处理。
- jti（JWTID）：JWT唯一标识。

示例：


```json
{
  "uid": "ff1212f5-d8d1-4496-bf41-d2dda73de19a",
  "sub": "1234567890",
  "name": "John Doe",
  "exp": 15323232,
  "iat": 1516239022,
  "scope": ["admin","user"]
}
```

Payload部分默认是不加密的，**一定不要将隐私信息存放在Payload当中**！！！JSON形式的Payload被转换成Base64编码，成为JWT的第二部分。

### Signature

Signature部分是对前两部分的签名，作用是防止JWT（主要是payload）被篡改。这个签名的生成需要用到：

- Header+Payload。
- 存放在服务端的密钥(一定不要泄露出去)。
- 签名算法。

签名的计算公式如下：

```text
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把Header、Payload、Signature三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，这个字符串就是JWT。

## 如何基于JWT进行身份验证？

在基于JWT进行身份验证的的应用程序中，服务器通过Payload、Header和Secret(密钥)创建JWT并将JWT发送给客户端。客户端接收到JWT之后，会将其保存在Cookie或者localStorage里面，以后客户端发出的所有请求都会携带这个令牌。

![JWT身份验证示意图](https://oss.javaguide.cn/github/javaguide/system-design/jwt/jwt-authentication%20process.png)

简化后的步骤如下：

1. 用户向服务器发送用户名、密码以及验证码用于登陆系统。
2. 如果用户用户名、密码以及验证码校验正确的话，服务端会返回已经签名的Token，也就是JWT。
3. 用户以后每次向后端发请求都在Header中带上这个JWT。
4. 服务端检查JWT并从中获取用户相关信息。

两点建议：

1. 建议将JWT存放在localStorage中，放在Cookie中会有CSRF风险。
2. 请求服务端并携带JWT的常见做法是将其放在HTTP Header的`Authorization`字段中（`Authorization:BearerToken`）。

> **[spring-security-jwt-guide](https://github.com/Snailclimb/spring-security-jwt-guide)** 就是一个基于JWT来做身份认证的简单案例，感兴趣的可以看看。

## 如何防止JWT被篡改？

有了签名之后，即使JWT被泄露或者截获，黑客也没办法同时篡改Signature、Header、Payload。这是为什么呢？因为服务端拿到JWT之后，会解析出其中包含的Header、Payload以及Signature。服务端会根据Header、Payload、密钥再次生成一个Signature。拿新生成的Signature和JWT中的Signature作对比，如果一样就说明Header和Payload没有被修改。不过，如果服务端的秘钥也被泄露的话，黑客就可以同时篡改Signature、Header、Payload了。黑客直接修改了Header和Payload之后，再重新生成一个Signature就可以了。

**密钥一定保管好，一定不要泄露出去。JWT安全的核心在于签名，签名安全的核心在密钥**。

## 如何加强JWT的安全性？

1. 使用安全系数高的加密算法。
2. 使用成熟的开源库，没必要造轮子。
3. JWT存放在localStorage中而不是Cookie中，避免CSRF风险。
4. 一定不要将隐私信息存放在Payload当中。
5. 密钥一定保管好，一定不要泄露出去。JWT安全的核心在于签名，签名安全的核心在密钥。
6. Payload要加入`exp`（JWT的过期时间），永久有效的JWT不合理。并且，JWT的过期时间不易过长。
7. ......

> [原文链接](https://javaguide.cn/system-design/security/jwt-intro.html)

# JWT身份认证优缺点分析

## JWT的优势

相比于Session认证的方式来说，使用JWT进行身份认证主要有下面4个优势。

### 无状态

JWT自身包含了身份验证所需要的所有信息，因此，我们的服务器不需要存储Session信息。这显然增加了系统的可用性和伸缩性，大大减轻了服务端的压力。不过，也正是由于JWT的无状态，也导致了它最大的缺点：不可控！就比如说，我们想要在JWT有效期内废弃一个JWT或者更改它的权限的话，并不会立即生效，通常需要等到有效期过后才可以。再比如说，当用户Log out的话，JWT也还有效。除非，我们在后端增加额外的处理逻辑比如将失效的JWT存储起来，后端先验证JWT是否有效再进行处理。具体的解决办法，我们会在后面的内容中详细介绍到，这里只是简单提一下。

### 有效避免了CSRF攻击

**CSRF（CrossSiteRequestForgery）一般被翻译为跨站请求伪造**，属于网络攻击领域范围。相比于SQL脚本注入、XSS等安全攻击方式，CSRF的知名度并没有它们高。但是，它的确是我们开发系统时必须要考虑的安全隐患。就连业内技术标杆Google的产品Gmail也曾在2007年的时候爆出过CSRF漏洞，这给Gmail的用户造成了很大的损失。

那么究竟什么是跨站请求伪造呢？简单来说就是用你的身份去做一些不好的事情（发送一些对你不友好的请求比如恶意转账）。举个简单的例子：小壮登录了某网上银行，他来到了网上银行的帖子区，看到一个帖子下面有一个链接写着“科学理财，年盈利率过万”，小壮好奇的点开了这个链接，结果发现自己的账户少了10000元。这是这么回事呢？原来黑客在链接中藏了一个请求，这个请求直接利用小壮的身份给银行发送了一个转账请求，也就是通过你的Cookie向银行发出请求。

```html
<a src="http://www.mybank.com/Transfer?bankId=11&money=10000">科学理财，年盈利率过万</a>
```

CSRF攻击需要依赖Cookie，Session认证中Cookie中的SessionID是由浏览器发送到服务端的，只要发出请求，Cookie就会被携带。借助这个特性，即使黑客无法获取你的SessionID，只要让你误点攻击链接，就可以达到攻击效果。另外，并不是必须点击链接才可以达到攻击效果，很多时候，只要你打开了某个页面，CSRF攻击就会发生。


```html
<img src="http://www.mybank.com/Transfer?bankId=11&money=10000"/>
```

**那为什么JWT不会存在这种问题呢？**

一般情况下我们使用JWT的话，在我们登录成功获得JWT之后，一般会选择存放在localStorage中。前端的每一个请求后续都会附带上这个JWT，整个过程压根不会涉及到Cookie。因此，即使你点击了非法链接发送了请求到服务端，这个非法请求也是不会携带JWT的，所以这个请求将是非法的。总结来说就一句话：**使用JWT进行身份验证不需要依赖Cookie，因此可以避免CSRF攻击**。不过，这样也会存在XSS攻击的风险。为了避免XSS攻击，你可以选择将JWT存储在标记为httpOnly的Cookie中。但是，这样又导致了你必须自己提供CSRF保护，因此，实际项目中我们通常也不会这么做。常见的避免XSS攻击的方式是过滤掉请求中存在XSS攻击风险的可疑字符串。在Spring项目中，我们一般是通过创建XSS过滤器来实现的。


```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class XSSFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
      FilterChain chain) throws IOException, ServletException {
        XSSRequestWrapper wrappedRequest =
          new XSSRequestWrapper((HttpServletRequest) request);
        chain.doFilter(wrappedRequest, response);
    }

    // other methods
}
```

### 适合移动端应用

使用Session进行身份认证的话，需要保存一份信息在服务器端，而且这种方式会依赖到Cookie（需要Cookie保存SessionId），所以不适合移动端。但是，使用JWT进行身份认证就不会存在这种问题，因为只要JWT可以被客户端存储就能够使用，而且JWT还可以跨语言使用。

### 单点登录友好

使用Session进行身份认证的话，实现单点登录，需要我们把用户的Session信息保存在一台电脑上，并且还会遇到常见的Cookie跨域的问题。但是，使用JWT进行认证的话，JWT被保存在客户端，不会存在这些问题。

## JWT身份认证常见问题及解决办法

### 注销登录等场景下JWT还有效

与之类似的具体相关场景有：

- 退出登录;
- 修改密码;
- 服务端修改了某个用户具有的权限或者角色；
- 用户的帐户被封禁/删除；
- 用户被服务端强制注销；
- 用户被踢下线；
- ......

这个问题不存在于Session认证方式中，因为在Session认证方式中，遇到这种情况的话服务端删除对应的Session记录即可。但是，使用JWT认证的方式就不好解决了。我们也说过了，JWT一旦派发出去，如果后端不增加其他逻辑的话，它在失效之前都是有效的。那我们如何解决这个问题呢？查阅了很多资料，我简单总结了下面4种方案：

**1、将JWT存入内存数据库**

将JWT存入DB中，Redis内存数据库在这里是不错的选择。如果需要让某个JWT失效就直接从Redis中删除这个JWT即可。但是，这样会导致每次使用JWT发送请求都要先从DB中查询JWT是否存在的步骤，而且违背了JWT的无状态原则。

**2、黑名单机制**

和上面的方式类似，使用内存数据库比如Redis维护一个黑名单，如果想让某个JWT失效的话就直接将这个JWT加入到黑名单即可。然后，每次使用JWT进行请求的话都会先判断这个JWT是否存在于黑名单中。

前两种方案的核心在于将有效的JWT存储起来或者将指定的JWT拉入黑名单。虽然这两种方案都违背了JWT的无状态原则，但是一般实际项目中我们通常还是会使用这两种方案。

**3、修改密钥(Secret)**:

我们为每个用户都创建一个专属密钥，如果我们想让某个JWT失效，我们直接修改对应用户的密钥即可。但是，这样相比于前两种引入内存数据库带来了危害更大：

- 如果服务是分布式的，则每次发出新的JWT时都必须在多台机器同步密钥。为此，你需要将密钥存储在数据库或其他外部服务中，这样和Session认证就没太大区别了。
- 如果用户同时在两个浏览器打开系统，或者在手机端也打开了系统，如果它从一个地方将账号退出，那么其他地方都要重新进行登录，这是不可取的。

**4、保持令牌的有效期限短并经常轮换**

很简单的一种方式。但是，会导致用户登录状态不会被持久记录，而且需要用户经常登录。另外，对于修改密码后JWT还有效问题的解决还是比较容易的。说一种我觉得比较好的方式：**使用用户的密码的哈希值对JWT进行签名。因此，如果密码更改，则任何先前的令牌将自动无法验证**。

### JWT的续签问题

JWT有效期一般都建议设置的不太长，那么JWT过期后如何认证，如何实现动态刷新JWT，避免用户经常需要重新登录？我们先来看看在Session认证中一般的做法：假如Session的有效期30分钟，如果30分钟内用户有访问，就把Session有效期延长30分钟。JWT认证的话，我们应该如何解决续签问题呢？查阅了很多资料，我简单总结了下面4种方案：

**1、类似于Session认证中的做法**

这种方案满足于大部分场景。假设服务端给的JWT有效期设置为30分钟，服务端每次进行校验时，如果发现JWT的有效期马上快过期了，服务端就重新生成JWT给客户端。客户端每次请求都检查新旧JWT，如果不一致，则更新本地的JWT。这种做法的问题是仅仅在快过期的时候请求才会更新JWT,对客户端不是很友好。

**2、每次请求都返回新JWT**

这种方案的的思路很简单，但是，开销会比较大，尤其是在服务端要存储维护JWT的情况下。

**3、JWT有效期设置到半夜**

这种方案是一种折衷的方案，保证了大部分用户白天可以正常登录，适用于对安全性要求不高的系统。

**4、用户登录返回两个JWT**

第一个是accessJWT，它的过期时间JWT本身的过期时间比如半个小时，另外一个是refreshJWT它的过期时间更长一点比如为1天。客户端登录后，将accessJWT和refreshJWT保存在本地，每次访问将accessJWT传给服务端。服务端校验accessJWT的有效性，如果过期的话，就将refreshJWT传给服务端。如果有效，服务端就生成新的accessJWT给客户端。否则，客户端就重新登录即可。这种方案的不足是：

- 需要客户端来配合；
- 用户注销的时候需要同时保证两个JWT都无效；
- 重新请求获取JWT的过程中会有短暂JWT不可用的情况（可以通过在客户端设置定时器，当accessJWT快过期的时候，提前去通过refreshJWT获取新的accessJWT）;
- 存在安全问题，只要拿到了未过期的refreshJWT就一直可以获取到accessJWT。

## 总结

JWT其中一个很重要的优势是无状态，但实际上，我们想要在实际项目中合理使用JWT的话，也还是需要保存JWT信息。JWT也不是银弹，也有很多缺陷，具体是选择JWT还是Session方案还是要看项目的具体需求。万万不可尬吹JWT，而看不起其他身份认证方案。另外，不用JWT直接使用普通的Token(随机生成，不包含具体的信息)结合Redis来做身份认证也是可以的。

> [原文链接](https://javaguide.cn/system-design/security/advantages&disadvantages-of-jwt.html)


# README

> 此段内容原本是在[自己的jwt项目](https://github.com/xmxe/jwt)中README.md中的内容

## JWT简介

### 什么是JWT

在介绍JWT之前，我们先来回顾一下利用token进行用户身份验证的流程：
1. 客户端使用用户名和密码请求登录
2. 服务端收到请求，验证用户名和密码
3. 验证成功后，服务端会签发一个token，再把这个token返回给客户端
4. 客户端收到token后可以把它存储起来，比如放到cookie中
5. 客户端每次向服务端请求资源时需要携带服务端签发的token，可以在cookie或者header中携带
6. 服务端收到请求，然后去验证客户端请求里面带着的token，如果验证成功，就向客户端返回请求数据

这种基于token的认证方式相比传统的session认证方式更节约服务器资源，并且对移动端和分布式更加友好。其优点如下：
- 支持跨域访问：cookie是无法跨域的，而token由于没有用到cookie(前提是将token放到请求头中)，所以跨域后不会存在信息丢失问题
- 无状态：token机制在服务端不需要存储session信息，因为token自身包含了所有登录用户的信息，所以可以减轻服务端压力
- 更适用CDN：可以通过内容分发网络请求服务端的所有资料
更适用于移动端：当客户端是非浏览器平台时，cookie是不被支持的，此时采用token认证方式会简单很多
- 无需考虑CSRF：由于不再依赖cookie，所以采用token认证方式不会发生CSRF，所以也就无需考虑CSRF的防御

而JWT就是上述流程当中token的一种具体实现方式，其全称是JSON Web Token，[官网地址](https://jwt.io/)。通俗地说，**JWT的本质就是一个字符串**，它是将用户信息保存到一个Json字符串中，然后进行编码后得到一个JWT token，并且这个JWT token带有签名信息，接收后可以校验是否被篡改，所以可以用于在各方之间安全地将信息作为Json对象传输。JWT的认证流程如下：

1. 首先，前端通过Web表单将自己的用户名和密码发送到后端的接口，这个过程一般是一个POST请求。建议的方式是通过SSL加密的传输(HTTPS)，从而避免敏感信息被嗅探
2. 后端核对用户名和密码成功后，将包含用户信息的数据作为JWT的Payload，将其与JWT Header分别进行Base64编码拼接后签名，形成一个JWT Token，形成的JWT Token就是一个如同lll.zzz.xxx的字符串
3. 后端将JWT Token字符串作为登录成功的结果返回给前端。前端可以将返回的结果保存在浏览器中，退出登录时删除保存的JWT Token即可
4. 前端在每次请求时将JWT Token放入HTTP请求头中的Authorization属性中(解决XSS和XSRF问题)
5. 后端检查前端传过来的JWT Token，验证其有效性，比如检查签名是否正确、是否过期、token的接收方是否是自己等等
6. 验证通过后，后端解析出JWT Token中包含的用户信息，进行其他逻辑操作(一般是根据用户信息得到权限等)，返回结果
![](https://img-blog.csdnimg.cn/img_convert/900b3e81f832b2f08c2e8aabb540536a.png)

### 为什么要用JWT

1. 传统Session认证的弊端
我们知道HTTP本身是一种无状态的协议，这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，认证通过后HTTP协议不会记录下认证后的状态，那么下一次请求时，用户还要再一次进行认证，因为根据HTTP协议，我们并不知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在用户首次登录成功后，在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie，以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了，这是传统的基于session认证的过程
然而，传统的session认证有如下的问题：
    - 每个用户的登录信息都会保存到服务器的session中，随着用户的增多，服务器开销会明显增大
    - 由于session是存在与服务器的物理内存中，所以在分布式系统中，这种方式将会失效。虽然可以将session统一保存到Redis中，但是这样做无疑增加了系统的复杂性，对于不需要redis的应用也会白白多引入一个缓存中间件
    - 对于非浏览器的客户端、手机移动端等不适用，因为session依赖于cookie，而移动端经常没有cookie
    - 因为session认证本质基于cookie，所以如果cookie被截获，用户很容易收到跨站请求伪造攻击。并且如果浏览器禁用了cookie，这种方式也会失效
    - 前后端分离系统中更加不适用，后端部署复杂，前端发送的请求往往经过多个中间件到达后端，cookie中关于session的信息会转发多次
    - 由于基于Cookie，而cookie无法跨域，所以session的认证也无法跨域，对单点登录不适用

2. JWT认证的优势
   对比传统的session认证方式，JWT的优势是：
    - 简洁：JWT Token数据量小，传输速度也很快
    - 因为JWT Token是以JSON加密形式保存在客户端的，所以JWT是跨语言的，原则上任何web形式都支持
    - 不需要在服务端保存会话信息，也就是说不依赖于cookie和session，所以没有了传统session认证的弊端，特别适用于分布式微服务
    - 单点登录友好：使用Session进行身份认证的话，由于cookie无法跨域，难以实现单点登录。但是，使用token进行认证的话，token可以被保存在客户端的任意位置的内存中，不一定是cookie，所以不依赖cookie，不会存在这些问题
    - 适合移动端应用：使用Session进行身份认证的话，需要保存一份信息在服务器端，而且这种方式会依赖到Cookie（需要Cookie保存SessionId），所以不适合移动端

    因为这些优势，目前无论单体应用还是分布式应用，都更加推荐用JWT token的方式进行用户认证

### JWT结构

JWT由3部分组成：标头(Header)、有效载荷(Payload)和签名(Signature)。在传输的时候，会将JWT的3部分分别进行Base64编码后用.进行连接形成最终传输的字符串

```
JWT String = Base64(Header).Base64(Payload).HMACSHA256(base64UrlEncode(header)+"."+base64UrlEncode(payload),secret)
```

**Header**
JWT头是一个描述JWT元数据的JSON对象，alg属性表示签名使用的算法，默认为HMAC SHA256（写为HS256）,typ属性表示令牌的类型，JWT令牌统一写为JWT。最后，使用Base64 URL算法将上述JSON对象转换为字符串保存
```json
{
"alg": "HS256",
"typ": "JWT"
}
```

**Payload**
有效载荷部分，是JWT的主体内容部分，也是一个JSON对象，包含需要传递的数据。JWT指定七个默认字段供选择
```
iss：发行人
exp：到期时间
sub：主题
aud：用户
nbf：在此之前不可用
iat：发布时间
jti：JWT ID用于标识该JWT
```
这些预定义的字段并不要求强制使用。除以上默认字段外，我们还可以自定义私有字段，一般会把包含用户信息的数据放到payload中，如下例：
```json
{
"sub": "1234567890",
"name": "Helen",
"admin": true
}
```
请注意，默认情况下JWT是未加密的，因为只是采用base64算法，拿到JWT字符串后可以转换回原本的JSON数据，任何人都可以解读其内容，因此不要构建隐私信息字段，比如用户的密码一定不能保存到JWT中，以防止信息泄露。JWT只是适合在网络中传输一些非敏感的信息

**Signature**
签名哈希部分是对上面两部分数据签名，需要使用base64编码后的header和payload数据，通过指定的算法生成哈希，以确保数据不会被篡改。首先，需要指定一个密钥（secret）。该密码仅仅为保存在服务器中，并且不能向用户公开。然后，使用header中指定的签名算法（默认情况下为HMAC SHA256）根据以下公式生成签名
```
HMACSHA256(base64UrlEncode(header)+"."+base64UrlEncode(payload),secret)
```
在计算出签名哈希后，JWT头，有效载荷和签名哈希的三个部分组合成一个字符串，每个部分用.分隔，就构成整个JWT对象

> 注意JWT每部分的作用，在服务端接收到客户端发送过来的JWT token之后：
 > - header和payload可以直接利用base64解码出原文，从header中获取哈希签名的算法，从payload中获取有效数据
 > - signature由于使用了不可逆的加密算法，无法解码出原文，它的作用是校验token有没有被篡改。服务端获取header中的加密算法之后，利用该算法加上secretKey对header、payload进行加密，比对加密后的数据和客户端发送过来的是否一致。注意secretKey只能保存在服务端，而且对于不同的加密算法其含义有所不同，一般对于MD5类型的摘要加密算法，secretKey实际上代表的是盐值

## 相关文章

- [还分不清Cookie、Session、Token、JWT？](https://mp.weixin.qq.com/s/skZL7RR3SftrB4SNZx59ZA)
- [JWT实现登录认证+Token自动续期方案](https://mp.weixin.qq.com/s/i73E4zbTh_JCuRCqH_NoVQ)