---
img: https://pica.zhimg.com/v2-6e9fdfa6a51b5fd85c96c25a1ba0f289_1440w.jpg
tags: 计算机
title: TCP&UDP
---

####  TCP协议三次握手

**第一次握手**：客户端先向服务端发送一个请求连接的报文段，这个报文段SYN位设置为1,序列号Seq(Sequence Number)设置为某一值，假设为X，发送出去之后客户端进入SYN_SEND状态，等待服务器的确认
**第二次握手**：服务器收到SYN报文段。服务器收到客户端的SYN报文段，需要对这个SYN报文段进行确认，设置Acknowledgment Number为x+1(Sequence Number+1)；同时，自己自己还要发送SYN请求信息，将SYN位置为1，Sequence Number为y。服务器端将上述所有信息放到一个报文段（即SYN+ACK报文段）中，一并发送给客户端，此时服务器进入SYN_RECV状态
**第三次握手**：客户端收到服务器的SYN+ACK报文段。然后将Acknowledgment Number设置为y+1，向服务器发送ACK报文段，这个报文段发送完毕以后，客户端和服务器端都进入ESTABLISHED状态，完成TCP三次握手。

#### 为什么要三次握手而不是两次?

为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。如下：

> “已失效的连接请求报文段”的产生在这样一种情况下：client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间的滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。
>
> 假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。但server却以为新的运输连接已经建立，并一直等待client发来数据。这样，server的很多资源就白白浪费掉了。
>
> 采用“三次握手”的办法可以防止上述现象发生。例如刚才那种情况，client不会向server的确认发出确认。server由于收不到确认，就知道client并没有要求建立连接。”，防止了服务器端的一直等待而浪费资源。
> 完成了三次握手，客户端和服务器端就可以开始传送数据

#### 第三次握手失败了怎么办?

在tcp三次握手中 第二次握手完成后connect 就成功返回了 如果第三次握手的ack包丢了 此时 客户端已认为连接是成功的,如果没有应用层的心跳包,客户端会一直维护这个连接 请问如何避免这种情况？
第二次握手服务器收到SYN包,然后发出SYN+ACK数据包确认收到并且请求建立连接，服务器进入SYN_RECV状态。而这个时候第三次握手时客户端发送ACK给服务器失败了，服务器没办法进入ESTABLISH状态，这个时候肯定不能传输数据的，不论客户端主动发送数据与否，服务器都会有定时器发送第二步SYN+ACK数据包，如果客户端再次发送ACK成功，建立连接。
如果一直不成功，服务器肯定会有超时（大概64s）设置，超时之后会给客户端发「RTS报文」 (连接重置)，进入CLOSED状态，防止SYN洪泛攻击，这个时候客户端应该也会关闭连接

> SYN洪泛攻击:
> SYN攻击利用的是TCP的三次握手机制，攻击端利用伪造的IP地址向被攻击端发出请求，而被攻击端发出的响应 报文将永远发送不到目的地，那么「被攻击端在等待关闭这个连接的过程中消耗了资源」，如果有成千上万的这种连接，主机资源将被耗尽，从而达到攻击的目的。

#### TCP协议四次分手

**第一次分手**：主机1，设置序列号Seq(Sequence Number)和确认包ACK(Acknowledgment Number)，假设seq为x+2,ACK=y+1,再将FIN标志位设置为1,向主机2发送FIN报文段；之后主机1进入FIN_WAIT_1状态；这表示主机1没有数据要发送给主机2了；
**第二次分手**：主机2收到了主机1发送的FIN报文段，向主机1回一个ACK报文段(其值为接收到的FIN报文的seq值+1)；主机1进入FIN_WAIT_2状态,等待主机二的断开请求包FIN；
**第三次分手**：主机2向主机1发送FIN报文段，意思是我可以断开连接了,请求关闭连接，同时主机2进入CLOSE_WAIT状态；
**第四次分手** ：主机1收到主机2发送的FIN报文段，向主机2发送ACK报文段,值为刚刚接收到的FIN包Seq值+1，然后主机1进入TIME_WAIT状态；主机2收到主机1的ACK报文段以后，就关闭连接；此时，主机1等待2MSL后依然没有收到回复，则证明Server端已正常关闭，那好，主机1也可以关闭连接了。

#### 为什么要四次挥手

TCP是全双工模式，这就意味着，当主机1发出FIN报文段时，只是表示主机1已经没有数据要发送了，主机1告诉主机2，它的数据已经全部发送完毕了；但是，这个时候主机1还是可以接受来自主机2的数据；当主机2返回ACK报文段时，表示它已经知道主机1没有数据发送了，但是主机2还是可以发送数据到主机1的；当主机2也发送了FIN报文段时，这个时候就表示主机2也没有数据要发送了，就会告诉主机1，我也没有数据要发送了，之后彼此就会愉快的中断这次TCP连接。如果要正确的理解四次分手的原理，就需要了解四次分手过程中的状态变化。

#### 四次挥手状态解释

FIN_WAIT_1: 这个状态要好好解释一下，其实FIN_WAIT_1和FIN_WAIT_2状态的真正含义都是表示等待对方的FIN报文。而这两种状态的区别是：FIN_WAIT_1状态实际上是当SOCKET在ESTABLISHED状态时，它想主动关闭连接，向对方发送了FIN报文，此时该SOCKET即进入到FIN_WAIT_1状态。「也就是,发出FIN包之后进入FIN_WAIT_1状态」

而当对方回应ACK报文后，则进入到FIN_WAIT_2状态，当然在实际的正常情况下，无论对方何种情况下，都应该马上回应ACK报文，所以FIN_WAIT_1状态一般是比较难见到的，而FIN_WAIT_2状态还有时常常可以用netstat看到。「也就是,发出ACK报文之后进入FIN_WAIT_2状态」最新面试题整理好了，大家可以在Java面试库小程序在线刷题。

主动方FIN_WAIT_2：上面已经详细解释了这种状态，实际上FIN_WAIT_2状态下的SOCKET，表示半连接，也即有一方要求close连接，但另外还告诉对方，我暂时还有点数据需要传送给你(ACK信息)，稍后再关闭连接。

主动方CLOSE_WAIT：这种状态的含义其实是表示在等待关闭。怎么理解呢？当对方close一个SOCKET后发送FIN报文给自己，你系统毫无疑问地会回应一个ACK报文给对方，此时则进入到CLOSE_WAIT状态。接下来呢，实际上你真正需要考虑的事情是察看你是否还有数据发送给对方，如果没有的话，那么你也就可以 close这个SOCKET，发送FIN报文给对方，也即关闭连接。所以你在CLOSE_WAIT状态下，需要完成的事情是等待你去关闭连接。

被动方LAST_ACK: 这个状态还是比较容易好理解的，它是被动关闭一方在发送FIN报文后，最后等待对方的ACK报文。当收到ACK报文后，也即可以进入到CLOSED可用状态了。「也就是接收到了对方的FIN包,自己发出了ACK以及FIN包之后的状态」

被动方TIME_WAIT: 表示收到了对方的FIN报文，并发送出了ACK报文，就等2MSL后即可回到CLOSED可用状态了。如果FIN_WAIT1状态下，收到了对方同时带FIN标志和ACK标志的报文时，可以直接进入到TIME_WAIT状态，而无须经过FIN_WAIT_2状态。想成为架构师，这份架构师图谱建议看看，少走弯路。

主动方CLOSED: 表示连接中断。


#### TCP、UDP区别

TCP是基于连接的，而UDP是基于非连接的。

tcp传输数据稳定可靠，适用于对网络通讯质量要求较高的场景，需要准确无误的传输给对方，比如，传输文件，发送邮件，浏览网页等等

udp的优点是速度快，但是可能产生丢包，所以适用于对实时性要求较高但是对少量丢包并没有太大要求的场景。比如：域名查询，语音通话，视频直播等。udp还有一个非常重要的应用场景就是隧道网络，比如：VXLAN


#### 相关文章

- [面试官：TCP为什么要三次握手与四次分手？大部分人答不上来！](https://mp.weixin.qq.com/s/EDsDatrzmY_w09b5s-HrVw)
- [一文搞定UDP和TCP高频面试题！](https://zhuanlan.zhihu.com/p/108822858?utm_source=wechat_timeline&utm_medium=social&utm_oi=1040923520439672832&from=timeline)
- [近两万字TCP硬核知识，教你吊打面试官！](https://mp.weixin.qq.com/s/0VtugnDC5V30EZrctpGwKA)
- [万字长文带你解析23个问题TCP疑难杂症！](https://mp.weixin.qq.com/s/P103z3rEmKmSgqenjUi9lQ)
- [15张图，了解一下TCP/IP必知也必会的10个问题](https://mp.weixin.qq.com/s/qf8L52VtGTzWcF0NB5Filg)
- [网络篇：朋友面试之TCP/IP，回去等通知吧](https://mp.weixin.qq.com/s/ACQz-OZN-oitoMzdvI3RfA)
- [三次握手和四次挥手到底是个什么鬼东西？](https://mp.weixin.qq.com/s/rT1O1wYoNEcznSGQ3s9FQA)
- [带你了解TCP/IP，UDP，Socket之间关系](https://zhuanlan.zhihu.com/p/345263331)
- [TCP、HTTP、Socket，傻傻分不清？](https://mp.weixin.qq.com/s/IDcnAAW-qHDqqH5jq8GcRg)
- [理解TCP和UDP协议到底有啥区别？](https://mp.weixin.qq.com/s/rbl4WUA8lksfh4s9U9QW3g)
- [HTTP协议一知半解？这篇帮你全面了解](https://mp.weixin.qq.com/s/Rh5BMzZxIYKsHzt-hliKMw)
- [RPC核心，万变不离其宗](https://mp.weixin.qq.com/s/eyXZGlRGaqQ6EPNvGFAgBg)
- [大白话告诉你TCP为什么需要三次握手四次挥手](https://mp.weixin.qq.com/s/tlWDBDu7UBeXaz1LUxA3Jg)
- [能将三次握手理解到这个深度，面试官拍案叫绝！](https://mp.weixin.qq.com/s/upBMDQsc9NfpdAFGZ34O4A)
- [字节一面：“为什么网络要分层？每一层的职责、包含哪些协议？”](https://mp.weixin.qq.com/s/Q_A9NGd-2_HKDJKRoMqz1w)
- [TCP就没什么缺陷吗？](https://mp.weixin.qq.com/s/EFSYnqOZRAXfHReHTqS5vw)
- [一台服务器最大并发tcp连接数多少？65535？](https://mp.weixin.qq.com/s/b_0Q1cAk_z9mYOK32sOz7w)