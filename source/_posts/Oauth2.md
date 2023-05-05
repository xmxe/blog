
---
title: Oauth2
categories: 技术栈
img: https://picd.zhimg.com/v2-06d026cb3aaade2aa5380ee31d099a85_1440w.jpg

---

###  demo

> fork from [oauth2-samples](https://github.com/lenve/oauth2-samples)

| Demo                     | 文章                                                                                  |
| :----------------------- |:------------------------------------------------------------------------------------|
| authorization_code       | [这个案例写出来，还怕跟面试官扯不明白OAuth2登录流程？](https://mp.weixin.qq.com/s/GXMQI59U6uzmS-C0WQ5iUw)  |
| client_credentials       | [死磕OAuth2，教练我要学全套的！](https://mp.weixin.qq.com/s/33Oxu6YMjwco3WRE07_IiQ)             |
| implicit                 | [死磕OAuth2，教练我要学全套的！](https://mp.weixin.qq.com/s/33Oxu6YMjwco3WRE07_IiQ)             |
| password                 | [死磕OAuth2，教练我要学全套的！](https://mp.weixin.qq.com/s/33Oxu6YMjwco3WRE07_IiQ)             |
| authorization_code_redis | [OAuth2令牌存入Redis](https://mp.weixin.qq.com/s/cGopy8hDPtkn8Q7HUYabbA)                |
| jwt                      | [让OAuth2和JWT在一起愉快玩耍](https://mp.weixin.qq.com/s/xEIWTduDqQuGL7lfiP735w)             |
| oauth2-sso               | [Spring Boot+OAuth2，一个注解搞定单点登录！](https://mp.weixin.qq.com/s/EyAMTbKPqNNnEtZACIsMVw) |
| github_login             | [分分钟让自己的网站接入GitHub第三方登录功能](https://mp.weixin.qq.com/s/tq4Q306J3hJFEtGL1EpOBA)       |

---

### OAuth2.0的四种模式

#### 授权码模式
这种方式是最常用的流程，安全性也最高，它适用于那些有后端的Web应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。令牌获取的流程如下：

![授权码模式](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rCpPJB83SvgzosiboTJxftAhibgOZffmU9RnmNUusomvBtoUKaxEXIU1df2icbUZOwSUeG4G0DxWgjtQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
上图中涉及到两个角色，分别是客户端、认证中心，客户端负责拿令牌，认证中心负责发放令牌。但是不是所有客户端都有权限请求令牌的，需要事先在认证中心申请，比如微信并不是所有网站都能直接接入，而是要去微信后台开通这个权限。至少要提前向认证中心申请的几个参数如下：
client_id：客户端唯一id，认证中心颁发的唯一标识
client_secret：客户端的秘钥，相当于密码
scope：客户端的权限
redirect_uri：授权码模式使用的跳转uri，需要事先告知认证中心。

1. 请求授权码

客户端需要向认证中心拿到授权码，比如第三方登录使用微信，扫一扫登录那一步就是向微信的认证中心获取授权码。
请求的url如下：
```
/oauth/authorize?client_id=&response_type=code&scope=&redirect_uri=
```
上述这个url中携带的几个参数如下：
client_id：客户端的id，这个由认证中心分配，并不是所有的客户端都能随意接入认证中心
response_type：固定值为code，表示要求返回授权码。
scope：表示要求的授权范围，客户端的权限
redirect_uri：跳转的uri，认证中心同意或者拒绝授权跳转的地址，如果同意会在uri后面携带一个code=xxx，这就是授权码

2. 返回授权码
第1步请求之后，认证中心会要求登录、是否同意授权，用户同意授权之后直接跳转到redirect_uri（这个需要事先在认证中心申请配置），授权码会携带在这个地址后面，如下：
```
http://xxxx?code=NMoj5y
```
上述链接中的NMoj5y就是授权码了。

3. 请求令牌
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

4. 返回令牌
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

1. 请求令牌
客户端直接请求令牌，请求的url如下：
```
/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=
```
这个url正是授权码模式中获取授权码的url，各个参数解析如下：
client_id：客户端的唯一Id
response_type：简化模式的固定值为token
scope：客户端的权限
redirect_uri：跳转的uri，这里后面携带的直接是令牌，不是授权码了。

2. 返回令牌
认证中心认证通过后，会跳转到redirect_uri，并且后面携带着令牌，链接如下：
```
https://xxxx#token=NPmdj5
```
token=NPmdj5这一段后面携带的就是认证中心携带的，令牌为NPmdj5。

#### 密码模式

密码模式也很简单，直接通过用户名、密码获取令牌，流程如下：
![密码模式](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rCpPJB83SvgzosiboTJxftAhxGsEsTxPIovmxbYqEqregcCE7o0h7fvcjkGSrdtXqUxFfs4EwqbegQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

1. 请求令牌
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
grant_type：授权类型，密码模式固定值为password
username：用户名
password：密码
client_id：客户端id
client_secret：客户端的秘钥

2. 返回令牌
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

1. 请求令牌
请求的url为如下：
```
/oauth/token?
grant_type=client_credentials&
client_id=&
client_secret=
```
参数解析如下：
grant_type：授权类型，客户端模式固定值为client_credentials
client_id：客户端id
client_secret：客户端秘钥

2. 返回令牌
认证成功后直接返回令牌，格式为JSON数据，如下：
```json
{
    "access_token": "ACCESS_TOKEN",
    "token_type": "bearer",
    "expires_in": 7200,
    "scope": "all"
}
```
---

### 相关文章

- [妹子始终没搞懂OAuth 2.0，今天整合Spring Cloud Security一次说明白！](https://mp.weixin.qq.com/s/i8hvrKPSCwlzpmt_p52ZbA)
- [快速接入GitHub、QQ第三方登录真有那么难吗？](https://mp.weixin.qq.com/s/l1vll9aSL1IzjsI-DhbtUw)
- [Spring Security OAuth2整合企业微信扫码登录](https://mp.weixin.qq.com/s/S7NNeiPJAEtNQtypxrWcmw)
- [实战：画了几张图，终于把OAuth2搞清楚了](https://mp.weixin.qq.com/s/r0H5zsm2H0AZui5nfeElRw)