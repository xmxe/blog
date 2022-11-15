---
title: Zookeeper
img: https://picx1.zhimg.com/v2-0fd0ab66a2f190f2f44cf7884a57ad70_1440w.jpg
categories: Java
tags: 安装
---

> [zookeeper demo](https://github.com/xmxe/demo/tree/master/study-demo/src/main/java/com/xmxe/study_demo/zookeeper)

### 基本原理

#### Zookeeper基本原理

1. ZooKeeper分为**服务器端（Server）**和**客户端（Client）**，客户端可以连接到整个ZooKeeper服务的任意服务器上（除非leaderServes参数被显式设置，leader不允许接受客户端连接）。
2. 客户端使用并维护一个TCP连接，通过这个连接发送请求、接受响应、获取观察的事件以及发送心跳。如果这个TCP连接中断，客户端将自动尝试连接到另外的ZooKeeper服务器。客户端第一次连接到ZooKeeper服务时，接受这个连接的ZooKeeper服务器会为这个客户端建立一个会话。当这个客户端连接到另外的服务器时，这个会话会被新的服务器重新建立。
3. 上图中每一个Server代表一个安装Zookeeper服务的机器，即是整个提供Zookeeper服务的集群（或者是由伪集群组成）；
4. 组成ZooKeeper服务的服务器必须彼此了解。它们维护一个内存中的状态图像，以及持久存储中的事务日志和快照，只要大多数服务器可用，ZooKeeper服务就可用；
5. ZooKeeper启动时，将从实例中选举一个leader，Leader负责处理数据更新等操作，一个更新操作成功的标志是当且仅当大多数Server在内存中成功修改数据。每个Server在内存中存储了一份数据。
6. Zookeeper是可以集群复制的，集群间通过Zab协议（Zookeeper Atomic Broadcast）来保持数据的一致性；
7. Zab协议包含两个阶段：**leader election阶段**和**Atomic Brodcast阶段**。
- 集群中将选举出一个leader，其他的机器则称为follower，所有的写操作都被传送给leader，并通过brodcast将所有的更新告诉给follower。
- 当leader崩溃或者leader失去大多数的follower时，需要重新选举出一个新的leader，让所有的服务器都恢复到一个正确的状态。
-  当leader被选举出来，且大多数服务器完成了和leader的状态同步后，leadder election的过程就结束了，就将会进入到Atomic brodcast的过程。
- Atomic Brodcast同步leader和follower之间的信息，保证leader和follower具有形同的系统状态。

#### Zookeeper角色

启动Zookeeper服务器集群环境后，多个Zookeeper服务器在工作前会选举出一个Leader。选举出 leader前，所有server不区分角色，都需要平等参与投票（obServer除外，不参与投票）；选主过程完成后，存在以下几种角色

- 领导者（leader）: 领导者负责进行投票的发起和决议，更新系统状态。
- 学习者（Learner）或跟随者（Follower）:Follower用于接收客户请求并向客户端返回结果，在选主过程中参与投票。Follower可以接收client请求，如果是写请求将转发给leader来更新系统状态。
- 观察者（ObServer）:ObServer可以接收客户端连接，将写请求转发给leader节点。但Observer不参加投票过程，只同步leader的状态，ObServer的目的是为了扩展系统，提高德取谏度。

##### 为什么需要server?

- ZooKeeper需保证高可用和强一致性;
- 为了支持更多的客户端，需要增加更多的Server;
- Follower增多会导致投票阶段延迟增大，影响性能

##### 在Zookeeper中ObServer起到什么作用？
- ObServer不参与投票过程，只同步leader的状态
- Observers接受客户端的连接，并将写请求转发给leader节点
- 加入更多ObServer节点，提高伸缩性，同时还不影响吞吐率

##### 为什么在Zookeeper中Server数目一般为奇数？
我们知道在Zookeeper中Leader选举算法采用了Zab协议。Zab核心思想是当多数Server写成功，则任务数据写成功。
①如果有3个Server，则最多允许1个Server挂掉。
②如果有4个Server，则同样最多允许1个Server挂掉。
既然3个或者4个Server，同样最多允许1个Server挂掉，那么它们的可靠性是一样的，所以选择奇数个ZooKeeper Server即可，这里选择3个Server。

#### ZooKeeper的写数据流程

1. Client向ZooKeeper的Server1上写数据，发送一个写请求。
2. 如果Server1不是Leader，那么Server1会把接受到的请求进一步转发给Leader，因为每个ZooKeeper的Server里面有一个是Leader。这个Leader会将写请求广播给各个Server，比如Server1和Server2，各个Server写成功后就会通知Leader。
3. 当Leader收到大多数Server数据写成功了，那么就说明数据写成功了。如果这里三个节点的话，只要有两个节点数据写成功了，那么就认为数据写成功了。写成功之后，Leader会告诉Server1数据写成功了。
4. Server1会进一步通知Client数据写成功了，这时就认为整个写操作成功。

#### ZooKeeper应用场景总结

##### 统一命名服务

1. 在分布式环境下，经常需要对应用/服务进行统一命名，便于识别不同服务。
- 类似于域名与ip之间对应关系，ip不容易记住，而域名容易记住。
- 通过名称来获取资源或服务的地址，提供者等信息。
2. 按照层次结构组织服务/应用名称。
- 可将服务名称以及地址信息写到ZooKeeper上，客户端通过ZooKeeper获取可用服务列表类

##### 配置管理
1. 分布式环境下，配置文件管理和同步是一个常见问题。
- 一个集群中，所有节点的配置信息是一致的，比如Hadoop集群。
- 对配置文件修改后，希望能够快速同步到各个节点上。
2. 配置管理可交由ZooKeeper实现。
- 可将配置信息写入ZooKeeper上的一个Znode。
- 各个节点监听这个Znode。
- 一旦Znode中的数据被修改，ZooKeeper将通知各个节点。

##### 集群管理
1. 分布式环境中，实时掌握每个节点的状态是必要的。
- 可根据节点实时状态做出一些调整。

2. 可交由ZooKeeper实现。
- 可将节点信息写入ZooKeeper上的一个Znode。
- 监听这个Znode可获取它的实时状态变化。

3. 典型应用
- Hbase中Master状态监控与选举。

##### 分布式通知与协调

1. 分布式环境中，经常存在一个服务需要知道它所管理的子服务的状态。
- NameNode需知道各个Datanode的状态。
- JobTracker需知道各个TaskTracker的状态。

2. 心跳检测机制可通过ZooKeeper来实现。
3. 信息推送可由ZooKeeper来实现，ZooKeeper相当于一个发布/订阅系统。

##### 分布式锁

处于不同节点上不同的服务，它们可能需要顺序的访问一些资源，这里需要一把分布式的锁。
分布式锁具有以下特性：
1. ZooKeeper是强一致的。比如各个节点上运行一个ZooKeeper客户端，它们同时创建相同的Znode，但是只有一个客户端创建成功。
2. 实现锁的独占性。创建Znode成功的那个客户端才能得到锁，其它客户端只能等待。当前客户端用完这个锁后，会删除这个Znode，其它客户端再尝试创建Znode，获取分布式锁。
3. 控制锁的时序。各个客户端在某个Znode下创建临时Znode，这个类型必须为CreateMode.EPHEMERAL_SEQUENTIAL，这样该Znode可掌握全局访问时序。

##### 分布式队列

分布式队列分为两种：
1. 当一个队列的成员都聚齐时，这个队列才可用，否则一直等待所有成员到达，这种是同步队列。
- 一个job由多个task组成，只有所有任务完成后，job才运行完成。
- 可为job创建一个/job目录，然后在该目录下，为每个完成的task创建一个临时的Znode，一旦临时节点数目达到task总数，则表明job运行完成。

2. 队列按照FIFO方式进行入队和出队操作，例如实现生产者和消费者模型。

### 安装部署

zookeeper的安装模式有三种：

- 单机模式（stand-alone）：单机单server
- 集群模式：多机多server，形成集群
- 伪集群模式：单机多个server，形成伪集群

```
#服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。单位为毫秒
tickTime=2000
#所有跟随者与领导者进行连接并同步的时间，如果在设定的时间内，半数以上的跟随者未能完成同步，领导者便会宣布放弃领导地位，进行另一次的领导选举,#单位为tick值的倍数
initLimit=10
#对于主节点与从节点进行同步操作时的超时时间，单位为tick值的倍数。
syncLimit=5
clientPort=2181
dataDir=/usr/local/zookeeper/data
dataLogDir=/usr/local/zookeeper/dataLog
#server.A=B：C：D：其中A是一个数字，表示这个是第几号服务器；B是这个服务器的ip地址；C 表示的是这个服务器与集群中的Leader服务器交换信息的端口；D表示的是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于B都是一样，所以不同的Zookeeper实例通信端口号不能一样，所以要给它们分配不同的端口号。
server.1=192.168.236.128:2888:3888
server.2=192.168.236.129:2888:3888
server.3=192.168.236.130:2888:3888
```

#### 启动命令

```
./zkServer.sh start
./zkCli.sh -server 127.0.0.1:2181
```

| 参数                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| dataDir                | 用于存放内存数据库快照的文件夹，同时用于集群的myid文件也存在这个文件夹里 |
| dataLogDir             | 用于单独设置transaction log的目录，transaction log分离可以避免和普通log还有快照的竞争。 |
| tickTime               | 心跳时间，为了确保client—server连接存在的，以毫秒为单位，最小超时时间为两个心跳时间 |
| clientPort             | 客户端监听端口                                               |
| globalOutstandingLimit | client请求队列的最大长度，防止内存溢出，默认值为1000。       |
| preAllocSize           | 预分配的Transaction log空间block为proAllocSize KB，默认block为64M，一般不需要更改，除非snapshot过于频繁 |
| snapCount              | 在snapCount个snapshot后写一次transaction log，默认值是100000 |
| traceFile              | 用于记录请求的log，打开会影响性能，用于debug，最好不要定义。 |
| maxClientCnxns         | 最大并发客户端数，用于防止DOS的，默认值是10，设置为0是不加限 |
| clientPortBindAddress  | 可以设置指定的client ip以及端口，不设置的话等于ANY:clientPort |
| minSessionTimeout      | 最小的客户端session超时时间，默认值为2个tickTime，单位是毫秒 |
| maxSessionTimeout      | 最大的客户端session超时时间，默认值为20个tickTime，单位是毫秒 |
| electionAlg            | 用于选举的实现的参数，0为以原始的基于UDP的方式协作，1为不进行用户验证的基于UDP的快速选举，2为进行用户验证的基于UDP的快速选举，3为基于TCP的快速选举，默认值为3。 |
| initLimit              | 多少个tickTime内，允许其他server连接并初始化数据，如果zooKeeper管理的数据较大，则应相应增大这个值 |
| syncLimit              | 多少个tickTime内，允许其他server连接并初始化数据，如果zooKeeper管理的数据较大，则应相应增大这个值 |
| leaderServes           | leader是否接受客户端连接。默认值为yes。leader负责协调更新。当更新吞吐量远高于读取吞吐量时，可以设置为不接受客户端连接，以便leader可以专注于同步协调工作。 |
| server.x=ip:xxxx:xxxx  | 配置集群里面的主机信息，其中server.x的x要写在myid文件中，决定当前机器的id，server.x=第一个port用于连接leader，第二个用于leader选举。如果 electionAlg为0，则不需要第二个port。hostname也可以填ip。|
| group.x=nnnnn[:nnnnn]  | 分组信息，表明哪个组有哪些节点，例如group．1＝1：2：3 group．2＝4：5：6group.3=7:8:9。 |
| weight.x=nnnnn         | 权重信息，表明哪个结点的权重是多少，例如weight．1＝1weight．2＝1weight.3=1 |

### 相关文章

- [ZooKeeper基本原理及安装部署](https://zhuanlan.zhihu.com/p/30024403?utm_source=wechat_timeline&utm_medium=social&utm_oi=1040923520439672832&from=timeline)
- [不耍流氓，有答案的Zookeeper面试题](https://mp.weixin.qq.com/s/QUIEoZLhkF3ozBj_GS9UMA)
- [ZooKeeper不仅仅是注册中心，你还知道有哪些](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491198&idx=1&sn=7bfd06e81d7fd361b1db2631ed81e9f0&source=41#wechat_redirect)
- [什么是Zookeeper?](https://zhuanlan.zhihu.com/p/62526102?utm_source=wechat_timeline&utm_medium=social&utm_oi=1040923520439672832&from=timeline)
- [不懂ZooKeeper？没关系，这一篇给你讲的明明白白](https://mp.weixin.qq.com/s/X-10DjJQE3sIBVmoOVV77g)
- [ZooKeeper的十二连问，你顶得了嘛](https://mp.weixin.qq.com/s/Nx8QrO8bwVRatP6jzb6HZg)
- [ZooKeeper到底解决了什么问题？](https://mp.weixin.qq.com/s/kaXKnzbaq0NNMPKrfowQHg)
- [大白话说清楚Zookeeper的选举机制！](https://mp.weixin.qq.com/s/Co2XO6EiFu-oEsFookpxuw)
- [Zookeeper的选举流程是怎样的？](https://mp.weixin.qq.com/s/RdTDJSMYBqkZGnGsmwoDvw)
- [Zookeeper怎么保证分布式事务的最终一致性？](https://mp.weixin.qq.com/s/n0UKU7BxPT1r4OCPPKHGaA)
- [Zookeeper夺命连环9问](https://mp.weixin.qq.com/s/KyDmcyi6bALQg-W6F-VuUg)
- [不懂Zookeeper？没关系呀，你看这篇就够了！](https://mp.weixin.qq.com/s/u9Gn0XSIB35k3BxlW_E23g)
- [Zookeeper的5个核心知识点！](https://mp.weixin.qq.com/s/ByfASCD2V-JcBtVCXTXPew)
- [Zookeeper典型应用场景介绍](https://mp.weixin.qq.com/s/we1KRq_WtRveHES2Xw6C9A)
- [zkcli.sh命令](https://www.cnblogs.com/chengxuyuanzhilu/p/6698059.html)