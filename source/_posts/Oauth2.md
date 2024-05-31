
---
title: Oauth2
categories: 技术栈
img: https://picd.zhimg.com/v2-06d026cb3aaade2aa5380ee31d099a85_1440w.jpg

---

> demo
> fork from [oauth2-samples](https://github.com/lenve/oauth2-samples)
>> | Demo                     | 文章                                                                                  |
>> | :----------------------- |:------------------------------------------------------------------------------------|
>> | authorization_code       | [这个案例写出来，还怕跟面试官扯不明白OAuth2登录流程？](https://mp.weixin.qq.com/s/GXMQI59U6uzmS-C0WQ5iUw)  |
>> | client_credentials       | [死磕OAuth2，教练我要学全套的！](https://mp.weixin.qq.com/s/33Oxu6YMjwco3WRE07_IiQ)             |
>> | implicit                 | [死磕OAuth2，教练我要学全套的！](https://mp.weixin.qq.com/s/33Oxu6YMjwco3WRE07_IiQ)             |
>> | password                 | [死磕OAuth2，教练我要学全套的！](https://mp.weixin.qq.com/s/33Oxu6YMjwco3WRE07_IiQ)             |
>> | authorization_code_redis | [OAuth2令牌存入Redis](https://mp.weixin.qq.com/s/cGopy8hDPtkn8Q7HUYabbA)                |
>> | jwt                      | [让OAuth2和JWT在一起愉快玩耍](https://mp.weixin.qq.com/s/xEIWTduDqQuGL7lfiP735w)             |
>> | oauth2-sso               | [Spring Boot+OAuth2，一个注解搞定单点登录！](https://mp.weixin.qq.com/s/EyAMTbKPqNNnEtZACIsMVw) |
>> | github_login             | [分分钟让自己的网站接入GitHub第三方登录功能](https://mp.weixin.qq.com/s/tq4Q306J3hJFEtGL1EpOBA)       |
>>

---

### OAuth2.0的四种模式

#### 授权码模式
这种方式是最常用的流程，安全性也最高，它适用于那些有后端的Web应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。令牌获取的流程如下：

![授权码模式](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rCpPJB83SvgzosiboTJxftAhibgOZffmU9RnmNUusomvBtoUKaxEXIU1df2icbUZOwSUeG4G0DxWgjtQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
上图中涉及到两个角色，分别是客户端、认证中心，客户端负责拿令牌，认证中心负责发放令牌。但是不是所有客户端都有权限请求令牌的，需要事先在认证中心申请，比如微信并不是所有网站都能直接接入，而是要去微信后台开通这个权限。至少要提前向认证中心申请的几个参数如下：

- **client_id**：客户端唯一id，认证中心颁发的唯一标识
- **client_secret**：客户端的秘钥，相当于密码
- **scope**：客户端的权限
- **redirect_uri**：授权码模式使用的跳转uri，需要事先告知认证中心。

**请求授权码**

客户端需要向认证中心拿到授权码，比如第三方登录使用微信，扫一扫登录那一步就是向微信的认证中心获取授权码。请求的url如下：
```
/oauth/authorize?client_id=&response_type=code&scope=&redirect_uri=
```
上述这个url中携带的几个参数如下：

- **client_id**：客户端的id，这个由认证中心分配，并不是所有的客户端都能随意接入认证中心
- **response_type**：固定值为code，表示要求返回授权码。
- **scope**：表示要求的授权范围，客户端的权限
- **redirect_uri**：跳转的uri，认证中心同意或者拒绝授权跳转的地址，如果同意会在uri后面携带一个code=xxx，这就是授权码

**返回授权码**

请求授权码之后，认证中心会要求登录、是否同意授权，用户同意授权之后直接跳转到redirect_uri（这个需要事先在认证中心申请配置），授权码会携带在这个地址后面，如下：

```
http://xxxx?code=NMoj5y
```
上述链接中的NMoj5y就是授权码了。

**请求令牌**

客户端拿到授权码之后，直接携带授权码发送请求给认证中心获取令牌，请求的url如下：
```
/oauth/token?
 client_id=&
 client_secret=&
 grant_type=authorization_code&
 code=NMoj5y&
 redirect_uri=
```
相同的参数同上，不同参数解析如下：

- grant_type：授权类型，授权码固定的值为authorization_code
- code：这个就是上一步获取的授权码

**返回令牌**

认证中心收到令牌请求之后，通过之后，会返回一段JSON数据，其中包含了令牌access_token，如下：

```json
{    
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101
}
```
access_token则是颁发的令牌，refresh_token是刷新令牌，一旦令牌失效则携带这个令牌进行刷新。

#### 简化模式

这种模式不常用，主要针对那些无后台的系统，直接通过web跳转授权，流程如下图：

![简化模式](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rCpPJB83SvgzosiboTJxftAhxGsEsTxPIovmxbYqEqregcCE7o0h7fvcjkGSrdtXqUxFfs4EwqbegQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

**请求令牌**

客户端直接请求令牌，请求的url如下：
```
/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=
```
这个url正是授权码模式中获取授权码的url，各个参数解析如下：

- **client_id**：客户端的唯一Id
- **response_type**：简化模式的固定值为token
- **scope**：客户端的权限
- **redirect_uri**：跳转的uri，这里后面携带的直接是令牌，不是授权码了。

**返回令牌**

认证中心认证通过后，会跳转到redirect_uri，并且后面携带着令牌，链接如下：
```
https://xxxx#token=NPmdj5
```
token=NPmdj5这一段后面携带的就是认证中心携带的，令牌为NPmdj5。

#### 密码模式

密码模式也很简单，直接通过用户名、密码获取令牌，流程如下：
![密码模式](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rCpPJB83SvgzosiboTJxftAhxGsEsTxPIovmxbYqEqregcCE7o0h7fvcjkGSrdtXqUxFfs4EwqbegQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**请求令牌**

认证中心要求客户端输入用户名、密码，认证成功则颁发令牌，请求的url如下：
```
/oauth/token?
  grant_type=password&
  username=&
  password=&
  client_id=&
  client_secret=
```
参数解析如下：

- **grant_type**：授权类型，密码模式固定值为password
- **username**：用户名
- **password**：密码
- **client_id**：客户端id
- **client_secret**：客户端的秘钥

**返回令牌**

上述认证通过，直接返回JSON数据，不需要跳转，如下：
```json
{
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101
}
```
access_token则是颁发的令牌，refresh_token是刷新令牌，一旦令牌失效则携带这个令牌进行刷新。

#### 客户端模式

适用于没有前端的命令行应用，即在命令行下请求令牌。
这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。流程如下：

![客户端模式](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rCpPJB83SvgzosiboTJxftAhxGsEsTxPIovmxbYqEqregcCE7o0h7fvcjkGSrdtXqUxFfs4EwqbegQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**请求令牌**

请求的url为如下：
```
/oauth/token?
grant_type=client_credentials&
client_id=&
client_secret=
```
参数解析如下：

- **grant_type**：授权类型，客户端模式固定值为client_credentials
- **client_id**：客户端id
- **client_secret**：客户端秘钥

**返回令牌**

认证成功后直接返回令牌，格式为JSON数据，如下：
```json
{
    "access_token": "ACCESS_TOKEN",
    "token_type": "bearer",
    "expires_in": 7200,
    "scope": "all"
}
```

### access_token VS refresh_token

#### 介绍

Token作为用户获取受保护资源的凭证，必须设置一个过期时间，否则一次登录便可永久使用，认证功能就失去了意义。但是矛盾在于：过期时间设置得太长，用户数据的安全性将大打折扣，过期时间设置得太短，用户就必须每隔一段时间重新登录，以获取新的凭证，这会极大挫伤用户的积极性。针对这一问题，我们可以利用Access / Refresh Token这一概念来平衡Token安全性和用户体验。

众所周知,在OAuth 2.0授权协议中,有两个令牌token,分别是access_token和refresh_token。我们先看下面两者的介绍：access_token-访问令牌,它是一个用来访问受保护资源的凭证。refresh_token-刷新令牌,它是一个用来获取access token的凭证。这两个令牌的主要区别如下:

- access_token时效短,refresh_token时效长,比如access_token有效期1个小时,refresh_token有效期1天。
- access_token是授权服务器一定颁发的,而refresh_token却是可选的。
- access_token过期后,可以使用refresh_token重新获取,而refresh_token过期后就只能重新授权了,也没有refresh_refresh_token。
- access_token和资源服务器和授权服务器交互,而refresh_token只和授权服务器交互。
- access_token颁发后可以直接使用,而使用refresh_token需要客户端秘钥client_secret。

简单来说：Access Token即“访问令牌”，是客户端向资源服务器换取资源的凭证；Refresh Token即“刷新令牌”，是客户端向认证服务器换取Access Token的凭证。

接下来,我们继续看两个令牌在下面场景的应用,假设有一个用户需要在后台管理界面上操作6个小时。

1. 颁发一个有效性很长的access_token,比如6个小时,或者可以更长,这样用户只需要刚开始登录一次,access_token可以一直使用,直到access_token过期,然后重复,这种是不安全的,access_token的时效太长,也就失去了本身的意义。

2. 颁发一个1小时有效期的access_token,过期后重新登录授权,这样用户需要登录6次,安全倒是有了,但是用户体验极差。

3. 颁发1小时有效期的access_token和6小时有效期的refresh_token,当access_token过期后（或者快要过期的时候）,使用refresh_token获取一个新的access_token,直到refresh_token过期,用户重新登录,这样整个过程中，用户只需要登录一次,用户体验好。**access_token泄露了怎么办?没关系,它很快就会过期。refresh_token泄露了怎么办?没关系,使用refresh_token是需要客户端秘钥client_secret的**。

4. 用户登录后,在后台管理页面上操作1个小时后,离开了一段时间,然后5个小时后,回到管理页面继续操作,此时refresh_token有效期6个小时,一直没有过期,也就可以换取新的access_token,用户在这个过程中,可以不用重复登录。但是在一些安全要求较高的系统中,第二次操作是需要重新登录的,即使refresh_token没有过期,因为中间有几个小时,用户是没有操作的,系统猜测用户已离开,并关闭会话。

所以得出的结论是,refresh_token是一个很巧妙地设计,提升了用户体验的同时,又保证了安全性。另外,在OAuth 2.0安全最佳实践中,推荐refresh_token是一次性的,什么意思呢?使用refresh_token获取access_token时,同时会返回一个新的refresh_token,之前的refresh_token就会失效,但是两个refresh_token的绝对过期时间是一样的,所以不会存在refresh_token快过期就获取一个新的,然后重复,永不过期的情况。

#### Access / Refresh Token如何使用？

1. 用户提供身份信息（一般是用户名密码），利用客户端向认证服务器换取Refresh Token和Access Token；

2. 客户端携带Access Token访问资源服务器，资源服务器识别Access Token并返回资源；

3. 当Access Token过期或失效，客户端再一次访问资源服务器，资源服务器返回“无效token”报错；

4. 客户端通过Refresh Token向认证服务器换取Access Token，认证服务器返回新的Access Token。

用一个现实生活中的比喻来解释Access/Refresh Token的使用过程：假设我在网上预定了一家酒店。如果要入住这家酒店，我必须出示身份证和订单。酒店前台会登记相关证件和订单信息，确认无误后会给我一张票据和一张房卡（票据记录我需要入住多少天，而房卡则让我有当天的入住权）。以上场景中，“身份证和订单”是我的用户名密码，“票据/房卡”是Refresh/Access Token，“前台”是认证服务器，“房间”是资源服务器。在整个入住过程中，“身份证和订单”只在前台使用一次；实际能进入房间的是“房卡”，但是房卡只有一天的有效期；如果房卡过期，我需要凭“票据”去前台刷新“房卡”，获取第二天的入住权。将Token拆分成两个，就是为了解决安全性和用户体验方面的矛盾—Access Token使用频繁，且与用户数据直接关联，安全性方面比较敏感，因此有效期设置得较短，即使Access Token泄漏也将很快失效。利用过期时间较短这个特性，也可以及时更新用户的访问权限（比如管理员缩小了的某员工访问公司数据的权限，当Token过期后换取的新Access Token将立马缩小其访问数据的权限）。而Refresh Token仅用于获取新的Access Token，使用频率较低，不与用户数据直接关联，过期时间允许设置得长一些。这样就解决了用户反复登录的问题。


### 相关文章

| [妹子始终没搞懂OAuth 2.0，今天整合Spring Cloud Security一次说明白！](https://mp.weixin.qq.com/s/i8hvrKPSCwlzpmt_p52ZbA) | [快速接入GitHub、QQ第三方登录真有那么难吗？](https://mp.weixin.qq.com/s/l1vll9aSL1IzjsI-DhbtUw) | [Spring Security OAuth2整合企业微信扫码登录](https://mp.weixin.qq.com/s/S7NNeiPJAEtNQtypxrWcmw) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [实战：画了几张图，终于把OAuth2搞清楚了](https://mp.weixin.qq.com/s/r0H5zsm2H0AZui5nfeElRw) |                                                              |                                                              |