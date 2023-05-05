---
title: 消息队列-MQ
categories: 技术栈
index_img: /assert/mq.jpg
img: https://img1.baidu.com/it/u=1369410832,3000356447&fm=253&fmt=auto&app=138&f=PNG?w=500&h=539

---


## 什么是消息队列？

我们可以把消息队列看作是一个存放消息的容器，当我们需要使用消息的时候，直接从容器中取出消息供自己使用即可。由于队列Queue是一种先进先出的数据结构，所以消费消息时也是按照顺序来消费的。

![Message queue](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/消息队列/message-queue-small.png)

参与消息传递的双方称为**生产者**和**消费者**，生产者负责发送消息，消费者负责处理消息。

![发布/订阅（Pub/Sub）模型](https://javaguide.cn/assets/message-queue-pub-sub-model.63a717b4.png)

我们知道操作系统中的进程通信的一种很重要的方式就是消息队列。我们这里提到的消息队列稍微有点区别，更多指的是各个服务以及系统内部各个组件/模块之前的通信，属于一种**中间件**。

维基百科是这样介绍中间件的：

> 中间件（英语：Middleware），又译中间件、中介层，是一类提供系统软件和应用软件之间连接、便于软件各部件之间的沟通的软件，应用软件可以借助中间件在不同的技术架构之间共享信息与资源。中间件位于客户机服务器的操作系统之上，管理着计算资源和网络通信。

简单来说：**中间件就是一类为应用软件服务的软件，应用软件是为用户服务的，用户不会接触或者使用到中间件。**

除了消息队列之外，常见的中间件还有RPC框架、分布式组件、HTTP服务器、任务调度框架、配置中心、数据库层的分库分表工具和数据迁移工具等等。

关于中间件比较详细的介绍可以参考阿里巴巴淘系技术的一篇回答：https://www.zhihu.com/question/19730582/answer/1663627873。

随着分布式和微服务系统的发展，消息队列在系统设计中有了更大的发挥空间，使用消息队列可以降低系统耦合性、实现任务异步、有效地进行流量削峰，是分布式和微服务系统中重要的组件之一。

## 消息队列有什么用？

通常来说，使用消息队列能为我们的系统带来下面三点好处：

1. **通过异步处理提高系统性能（减少响应所需时间）**
2. **削峰/限流**
3. **降低系统耦合性。**

如果在面试的时候你被面试官问到这个问题的话，一般情况是你在你的简历上涉及到消息队列这方面的内容，这个时候推荐你结合你自己的项目来回答。

### 通过异步处理提高系统性能（减少响应所需时间）

![通过异步处理提高系统性能](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/Asynchronous-message-queue.png)

将用户的请求数据存储到消息队列之后就立即返回结果。随后，系统再对消息进行消费。

因为用户请求数据写入消息队列之后就立即返回给用户了，但是请求数据在后续的业务校验、写数据库等操作中可能失败。因此，**使用消息队列进行异步处理之后，需要适当修改业务流程进行配合**，比如用户在提交订单之后，订单数据写入消息队列，不能立即返回用户订单提交成功，需要在消息队列的订单消费者进程真正处理完该订单之后，甚至出库后，再通过电子邮件或短信通知用户订单成功，以免交易纠纷。这就类似我们平时手机订火车票和电影票。

### 削峰/限流

**先将短时间高并发产生的事务消息存储在消息队列中，然后后端服务再慢慢根据自己的能力去消费这些消息，这样就避免直接把后端服务打垮掉。**

举例：在电子商务一些秒杀、促销活动中，合理使用消息队列可以有效抵御促销活动刚开始大量订单涌入对系统的冲击。如下图所示：

![削峰](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/削峰-消息队列.png)

### 降低系统耦合性

使用消息队列还可以降低系统耦合性。我们知道如果模块之间不存在直接调用，那么新增模块或者修改模块就对其他模块影响较小，这样系统的可扩展性无疑更好一些。还是直接上图吧：

![解耦](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/消息队列-解耦.png)

生产者（客户端）发送消息到消息队列中去，接受者（服务端）处理消息，需要消费的系统直接去消息队列取消息进行消费即可而不需要和其他系统有耦合，这显然也提高了系统的扩展性。

**消息队列使用发布-订阅模式工作，消息发送者（生产者）发布消息，一个或多个消息接受者（消费者）订阅消息。**从上图可以看到**消息发送者（生产者）和消息接受者（消费者）之间没有直接耦合**，消息发送者将消息发送至分布式消息队列即结束对消息的处理，消息接受者从分布式消息队列获取该消息后进行后续处理，并不需要知道该消息从何而来。**对新增业务，只要对该类消息感兴趣，即可订阅该消息，对原有系统和业务没有任何影响，从而实现网站业务的可扩展性设计**。

消息接受者对消息进行过滤、处理、包装后，构造成一个新的消息类型，将消息继续发送出去，等待其他消息接受者订阅该消息。因此基于事件（消息对象）驱动的业务架构可以是一系列流程。

另外，为了避免消息队列服务器宕机造成消息丢失，会将成功发送到消息队列的消息存储在消息生产者服务器上，等消息真正被消费者服务器处理后才删除消息。在消息队列服务器宕机后，生产者服务器会选择分布式消息队列服务器集群中的其他服务器发布消息。

**备注：**不要认为消息队列只能利用发布-订阅模式工作，只不过在解耦这个特定业务环境下是使用发布-订阅模式的。除了发布-订阅模式，还有点对点订阅模式（一个消息只有一个消费者），我们比较常用的是发布-订阅模式。另外，这两种消息模型是JMS提供的，AMQP协议还提供了另外5种消息模型。

### 实现分布式事务

我们知道分布式事务的解决方案之一就是MQ事务。

RocketMQ、Kafka、Pulsar、QMQ都提供了事务相关的功能。事务允许事件流应用将消费，处理，生产消息整个过程定义为一个原子操作。

![分布式事务详解-MQ事务](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/csdn/07b338324a7d8894b8aef4b659b76d92.png)

## 使用消息队列会带来哪些问题？

- **系统可用性降低：**系统可用性在某种程度上降低，为什么这样说呢？在加入MQ之前，你不用考虑消息丢失或者说MQ挂掉等等的情况，但是，引入MQ之后你就需要去考虑了！
- **系统复杂性提高：**加入MQ之后，你需要保证消息没有被重复消费、处理消息丢失的情况、保证消息传递的顺序性等等问题！
- **一致性问题：**我上面讲了消息队列可以实现异步，消息队列带来的异步确实可以提高系统响应速度。但是，万一消息的真正消费者并没有正确消费消息怎么办？这样就会导致数据不一致的情况了!

## JMS和AMQP

### JMS是什么？

JMS（JAVA Message Service,java消息服务）是Java的消息服务，JMS的客户端之间可以通过JMS服务进行异步的消息传输。**JMS（JAVA Message Service，Java消息服务）API是一个消息服务的标准或者说是规范**，允许应用程序组件基于JavaEE平台创建、发送、接收和读取消息。它使分布式通信耦合度更低，消息服务更加可靠以及异步性。

JMS定义了五种不同的消息正文格式以及调用的消息类型，允许你发送并接收以一些不同形式的数据：

- StreamMessage：Java原始值的数据流
- MapMessage：一套名称-值对
- TextMessage：一个字符串对象
- ObjectMessage：一个序列化的Java对象
- BytesMessage：一个字节的数据流

**ActiveMQ（已被淘汰）就是基于JMS规范实现的。**

### JMS两种消息模型

#### 点到点（P2P）模型

![队列模型](https://javaguide.cn/assets/message-queue-queue-model.3aa809bf.png)

使用**队列（Queue）作为消息通信载体；满足生产者与消费者模式**，一条消息只能被一个消费者使用，未被消费的消息在队列中保留直到被消费或超时。比如：我们生产者发送100条消息的话，两个消费者来消费一般情况下两个消费者会按照消息发送的顺序各自消费一半（也就是你一个我一个的消费。）

#### 发布/订阅（Pub/Sub）模型

![发布/订阅（Pub/Sub）模型](https://javaguide.cn/assets/message-queue-pub-sub-model.63a717b4.png)

发布订阅模型（Pub/Sub）使用**主题（Topic）作为消息通信载体，类似于广播模式**；发布者发布一条消息，该消息通过主题传递给所有的订阅者，**在一条消息广播之后才订阅的用户则是收不到该条消息的**。

### AMQP是什么？

AMQP，即Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准**高级消息队列协议**（二进制应用层协议），是应用层协议的一个开放标准，为面向消息的中间件设计，兼容JMS。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件同产品，不同的开发语言等条件的限制。

**RabbitMQ就是基于AMQP协议实现的。**

### JMS vs AMQP

|   对比方向   | JMS                                     | AMQP                                                         |
| :----------: | :-------------------------------------- | :----------------------------------------------------------- |
|     定义     | Java API                                | 协议                                                         |
|    跨语言    | 否                                      | 是                                                           |
|    跨平台    | 否                                      | 是                                                           |
| 支持消息类型 | 提供两种消息模型：①Peer-2-Peer;②Pub/sub | 提供了五种消息模型：①direct exchange；②fanout exchange；③topic change；④headers exchange；⑤system exchange。本质来讲，后四种和JMS的pub/sub模型没有太大差别，仅是在路由机制上做了更详细的划分； |
| 支持消息类型 | 支持多种消息类型 ，我们在上面提到过     | byte[]（二进制）                                             |

**总结：**

- AMQP为消息定义了线路层（wire-levelprotocol）的协议，而JMS所定义的是API规范。在Java体系中，多个client均可以通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差。而AMQP天然具有跨平台、跨语言特性。
- JMS支持`TextMessage`、`MapMessage`等复杂的消息类型；而AMQP仅支持`byte[]`消息类型（复杂的类型可序列化后发送）。
- 由于Exchange提供的路由算法，AMQP可以提供多样化的路由方式来传递消息到消息队列，而JMS仅支持队列和主题/订阅方式两种。

## RPC和消息队列的区别

RPC和消息队列都是分布式微服务系统中重要的组件之一，下面我们来简单对比一下两者：

- **从用途来看**：RPC主要用来解决两个服务的远程通信问题，不需要了解底层网络的通信机制。通过RPC可以帮助我们调用远程计算机上某个服务的方法，这个过程就像调用本地方法一样简单。消息队列主要用来降低系统耦合性、实现任务异步、有效地进行流量削峰。
- **从通信方式来看**：RPC是双向直接网络通讯，消息队列是单向引入中间载体的网络通讯。
- **从架构上来看**：消息队列需要把消息存储起来，RPC则没有这个要求，因为前面也说了RPC是双向直接网络通讯。
- **从请求处理的时效性来看**：通过RPC发出的调用一般会立即被处理，存放在消息队列中的消息并不一定会立即被处理。

RPC和消息队列本质上是网络通讯的两种不同的实现机制，两者的用途不同，万不可将两者混为一谈。

## 消息队列技术选型

### Kafka

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/high-performance/message-queue/kafka-logo.png)

Kafka是LinkedIn开源的一个分布式流式处理平台，已经成为Apache顶级项目，早期被用来用于处理海量的日志，后面才慢慢发展成了一款功能全面的高性能消息队列。

流式处理平台具有三个关键功能：

1. **消息队列**：发布和订阅消息流，这个功能类似于消息队列，这也是Kafka也被归类为消息队列的原因。
2. **容错的持久方式存储记录消息流**：Kafka会把消息持久化到磁盘，有效避免了消息丢失的风险。
3. **流式处理平台：**在消息发布的时候进行处理，Kafka提供了一个完整的流式处理类库。

Kafka是一个分布式系统，由通过高性能TCP网络协议进行通信的服务器和客户端组成，可以部署在在本地和云环境中的裸机硬件、虚拟机和容器上。

在Kafka2.8之前，Kafka最被大家诟病的就是其重度依赖于Zookeeper做元数据管理和集群的高可用。在Kafka2.8之后，引入了基于Raft协议的KRaft模式，不再依赖Zookeeper，大大简化了Kafka的架构，让你可以以一种轻量级的方式来使用Kafka。

不过，要提示一下：**如果要使用KRaft模式的话，建议选择较高版本的Kafka，因为这个功能还在持续完善优化中。Kafka3.3.1版本是第一个将KRaft（KafkaRaft）共识协议标记为生产就绪的版本。**

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/high-performance/message-queue/kafka3.3.1-kraft-%20production-ready.png)

[Kafka官网](http://kafka.apache.org/)

[Kafka更新记录](https://kafka.apache.org/downloads)


#### Kafka配置

**consumer**
```properties
# 消费者所属消费组的唯一标识
group.id
# 一次拉取请求的最大消息数，默认500条
max.poll.records
# 指定拉取消息线程最长空闲时间，默认300000ms
max.poll.interval.ms
# 检测消费者是否失效的超时时间，默认10000ms
session.timeout.ms
# 消费者心跳时间，默认3000ms
heartbeat.interval.ms
# 连接集群broker地址
bootstrap.servers
# 是否开启自动提交消费位移的功能，默认true
enable.auto.commit
# 自动提交消费位移的时间间隔，默认5000ms
auto.commit.interval.ms
# 消费者的分区配置策略,默认RangeAssignor
partition.assignment.strategy
# 如果分区没有初始偏移量，或者当前偏移量服务器上不存在时，将使用的偏移量设置，earliest从头开始消费，latest从最近的开始消费，none抛出异常，如果存在已经提交的offest时,不管设置为earliest或者latest都会从已经提交的offest处开始消费,如果不存在已经提交的offest时,earliest表示从头开始消费,latest表示从最新的数据消费,也就是新产生的数据.none topic各分区都存在已提交的offset时，从提交的offest处开始消费；只要有一个分区不存在已提交的offset，则抛出异常.kafka-0.10.1.X版本之前: auto.offset.reset的值为smallest,和,largest.(offest保存在zk中).kafka-0.10.1.X版本之后: auto.offset.reset的值更改为:earliest,latest,和none(offest保存在kafka的一个特殊的topic,名为:__consumer_offsets里面)
auto.offset.reset
# 消费者客户端一次请求从Kafka拉取消息的最小数据量，如果Kafka返回的数据量小于该值，会一直等待，直到满足这个配置大小，默认1b
fetch.min.bytes
# 消费者客户端一次请求从Kafka拉取消息的最大数据量，默认50MB
fetch.max.bytes
# 从Kafka拉取消息时，在不满足fetch.min.bytes条件时，等待的最大时间，默认500ms
fetch.max.wait.ms
# 强制刷新元数据时间，毫秒，默认300000，5分钟
metadata.max.age.ms
# 设置从每个分区里返回给消费者的最大数据量，区别于fetch.max.bytes，默认1MB
max.partition.fetch.bytes
# Socket发送缓冲区大小，默认128kb,-1将使用操作系统的设置
send.buffer.bytes
# Socket发送缓冲区大小，默认64kb,-1将使用操作系统的设置
receive.buffer.bytes
# 消费者客户端的id
client.id
# 连接失败后，尝试连接Kafka的时间间隔，默认50ms
reconnect.backoff.ms
# 尝试连接到Kafka，生产者客户端等待的最大时间，默认1000ms
reconnect.backoff.max.ms
# 消息发送失败重试时间间隔，默认100ms
retry.backoff.ms
# 样本计算时间窗口，默认30000ms
metrics.sample.window.ms
# 用于维护metrics的样本数量，默认2
metrics.num.samples
# metrics日志记录级别，默认info
metrics.log.level
# 类的列表，用于衡量指标，默认空list
metric.reporters
# 自动检查CRC32记录的消耗
check.crcs
# key反序列化方式
key.deserializer
# value反序列化方式
value.deserializer
# 设置多久之后关闭空闲连接，默认540000ms
connections.max.idle.ms
# 客户端将等待请求的响应的最大时间,如果在这个时间内没有收到响应，客户端将重发请求，超过重试次数将抛异常，默认30000ms
request.timeout.ms
# 设置消费者api超时时间，默认60000ms
default.api.timeout.ms
# 自定义拦截器
interceptor.classes
# 内部的主题:consumer_offsets和一transaction_state。该参数用来指定Kafka中的内部主题是否可以向消费者公开，默认值为true。如果设置为true，那么只能使用subscribe(Collection)的方式而不能使用subscribe(Pattern)的方式来订阅内部主题，设置为false则没有这个限制。
exclude.internal.topics
# 用来配置消费者的事务隔离级别。如果设置为“read committed”，那么消费者就会忽略事务未提交的消息，即只能消费到LSO(LastStableOffset)的位置，默认情况下为“read_uncommitted”，即可以消费到HW(High Watermark)处的位置
isolation.level
key.deserializer = org.apache.kafka.common.serialization.StringDeserializer
value.deserializer = org.apache.kafka.common.serialization.StringDeserializer

```
**producer**

```properties
# Snappy压缩技术是Google开发的，它可以在提供较好的压缩比的同时，减少对CPU的使用率并保证好的性能，所以建议在同时考虑性能和带宽的情况下使用。Gzip压缩技术通常会使用更多的CPU和时间，但会产生更好的压缩比，所以建议在网络带宽更受限制的情况下使用，默认不压缩，该参数可以设置成snappy、gzip或lz4对发送给broker的消息进行压缩
compression.type=Gzip
# 请求的最大字节数。这也是对最大消息大小的有效限制。注意：server具有自己对消息大小的限制，这些大小和这个设置不同。此项设置将会限制producer每次批量发送请求的数目，以防发出巨量的请求。
max.request.size=1048576
# TCP的接收缓存SO_RCVBUF空间大小，用于读取数据
receive.buffer.bytes=32768
# client等待请求响应的最大时间,如果在这个时间内没有收到响应，客户端将重发请求，超过重试次数发送失败
request.timeout.ms=30000
# TCP的发送缓存SO_SNDBUF空间大小，用于发送数据
send.buffer.bytes=131072
# 指定server等待来自followers的确认的最大时间，根据acks的设置，超时则返回error
timeout.ms=30000
# 在block前一个connection上允许最大未确认的requests数量。当设为1时，即是消息保证有序模式，注意：这里的消息保证有序是指对于单个Partition的消息有顺序，因此若要保证全局消息有序，可以只使用一个Partition，当然也会降低性能
max.in.flight.requests.per.connection=5
# 在第一次将数据发送到某topic时，需先fetch该topic的metadata，得知哪些服务器持有该topic的partition，该值为最长获取metadata时间
metadata.fetch.timeout.ms=60000
# 连接失败时，当我们重新连接时的等待时间
reconnect.backoff.ms=50
# 在重试发送失败的request前的等待时间，防止若目的Broker完全挂掉的情况下Producer一直陷入死循环发送，折中的方法
retry.backoff.ms=100
# metrics系统维护可配置的样本数量，在一个可修正的window size
metrics.sample.window.ms=30000
# 用于维护metrics的样本数
metrics.num.samples=2
# 类的列表，用于衡量指标。实现MetricReporter接口
metric.reporters=[]
# 强制刷新metadata的周期，即使leader没有变化
metadata.max.age.ms=300000
# 与broker会话协议，取值：LAINTEXT,SSL,SASL_PLAINTEXT,SASL_SSL
security.protocol=PLAINTEXT
# 分区类，实现Partitioner接口
partitioner.class=class org.apache.kafka.clients.producer.internals.DefaultPartitioner
# 控制block的时长，当buffer空间不够或者metadata丢失时产生block
max.block.ms=60000
# 关闭达到该时间的空闲连接
connections.max.idle.ms=540000
# 当向server发出请求时，这个字符串会发送给server，目的是能够追踪请求源
client.id=""
# acks=0配置适用于实现非常高的吞吐量,acks=all这是最安全的模式。Server完成producer request前需要确认的数量。acks=0时，producer不会等待确认，直接添加到socket等待发送；acks=1时，等待leader写到local log就行；acks=all或acks=-1时，等待isr中所有副本确认（注意：确认都是broker接收到消息放入内存就直接返回确认，不是需要等待数据写入磁盘后才返回确认，这也是kafka快的原因）
acks = all
# 发生错误时，重传次数。当开启重传时，需要将`max.in.flight.requests.per.connection`设置为1，否则可能导致失序
retries = 0
# 发送到同一个partition的消息会被先存储在batch中，该参数指定一个batch可以使用的内存大小，单位是byte。不一定需要等到batch被填满才能发送Producer可以将发往同一个Partition的数据做成一个Produce Request发送请求，即Batch批处理，以减少请求次数，该值即为每次批处理的大小。另外每个Request请求包含多个Batch，每个Batch对应一个Partition，且一个Request发送的目的Broker均为这些partition的leader副本。若将该值设为0，则不会进行批处理
batch.size = 16384
# 生产者在发送消息前等待linger.ms，从而等待更多的消息加入到batch中。如果batch被填满或者linger.ms达到上限，就把batch中的消息发送出去,Producer默认会把两次发送时间间隔内收集到的所有Requests进行一次聚合然后再发送，以此提高吞吐量，而linger.ms则更进一步，这个参数为每次发送增加一些delay，以此来聚合更多的Message。官网解释翻译：producer会将request传输之间到达的所有records聚合到一个批请求。通常这个值发生在欠负载情况下，record到达速度快于发送。但是在某些场景下，client即使在正常负载下也期望减少请求数量。这个设置就是如此，通过人工添加少量时延，而不是立马发送一个record,producer会等待所给的时延，以让其他records发送出去，这样就会被聚合在一起。这个类似于TCP的Nagle算法。该设置给了batch的时延上限：当我们获得一个partition的batch.size大小的records，就会立即发送出去，而不管该设置；但是如果对于这个partition没有累积到足够的record，会linger指定的时间等待更多的records出现。该设置的默认值为0(无时延)。例如，设置linger.ms=5，会减少request发送的数量，但是在无负载下会增加5ms的发送时延。
linger.ms = 1
# Producer可以用来缓存数据的内存大小。该值实际为RecordAccumulator类中的BufferPool，即Producer所管理的最大内存。如果数据产生速度大于向broker发送的速度，producer会阻塞max.block.ms，超时则抛出异常
buffer.memory = 33554432
key.serializer = org.apache.kafka.common.serialization.StringSerializer
value.serializer = org.apache.kafka.common.serialization.StringSerializer
```

#### Kafka是什么？主要应用场景有哪些？

Kafka是一个分布式流式处理平台。这到底是什么意思呢？

流平台具有三个关键功能：

1. **消息队列**：发布和订阅消息流，这个功能类似于消息队列，这也是Kafka也被归类为消息队列的原因。
2. **容错的持久方式存储记录消息流**：Kafka会把消息持久化到磁盘，有效避免了消息丢失的风险。
3. **流式处理平台：**在消息发布的时候进行处理，Kafka提供了一个完整的流式处理类库。

Kafka主要有两大应用场景：

1. **消息队列**：建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。
2. **数据处理**：构建实时的流数据处理程序来转换或处理数据流。

#### 和其他消息队列相比,Kafka的优势在哪里？

我们现在经常提到Kafka的时候就已经默认它是一个非常优秀的消息队列了，我们也会经常拿它跟RocketMQ、RabbitMQ对比。我觉得Kafka相比其他消息队列主要的优势如下：

1. **极致的性能**：基于Scala和Java语言开发，设计中大量使用了批量处理和异步的思想，最高可以每秒处理千万级别的消息。
2. **生态系统兼容性无可匹敌**：Kafka与周边生态系统的兼容性是最好的没有之一，尤其在大数据和流计算领域。

实际上在早期的时候Kafka并不是一个合格的消息队列，早期的Kafka在消息队列领域就像是一个衣衫褴褛的孩子一样，功能不完备并且有一些小问题比如丢失消息、不保证消息可靠性等等。当然，这也和LinkedIn最早开发Kafka用于处理海量的日志有很大关系，哈哈哈，人家本来最开始就不是为了作为消息队列滴，谁知道后面误打误撞在消息队列领域占据了一席之地。

随着后续的发展，这些短板都被Kafka逐步修复完善。所以，**Kafka作为消息队列不可靠这个说法已经过时！**

#### 队列模型了解吗？Kafka的消息模型知道吗？

> 题外话：早期的JMS和AMQP属于消息服务领域权威组织所做的相关的标准，我在[《消息队列其实很简单》](https://github.com/Snailclimb/JavaGuide#数据通信中间件)这篇文章中介绍过。但是，这些标准的进化跟不上消息队列的演进速度，这些标准实际上已经属于废弃状态。所以，可能存在的情况是：不同的消息队列都有自己的一套消息模型。

##### 队列模型：早期的消息模型

![队列模型](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/队列模型23.png)

**使用队列（Queue）作为消息通信载体，满足生产者与消费者模式，一条消息只能被一个消费者使用，未被消费的消息在队列中保留直到被消费或超时。**比如：我们生产者发送100条消息的话，两个消费者来消费一般情况下两个消费者会按照消息发送的顺序各自消费一半（也就是你一个我一个的消费。）

**队列模型存在的问题：**

假如我们存在这样一种情况：我们需要将生产者产生的消息分发给多个消费者，并且每个消费者都能接收到完整的消息内容。

这种情况，队列模型就不好解决了。很多比较杠精的人就说：我们可以为每个消费者创建一个单独的队列，让生产者发送多份。这是一种非常愚蠢的做法，浪费资源不说，还违背了使用消息队列的目的。

##### 发布-订阅模型:Kafka消息模型

发布-订阅模型主要是为了解决队列模型存在的问题。

![发布订阅模型](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/java-guide-blog/发布订阅模型.png)

发布订阅模型（Pub-Sub）使用**主题（Topic）**作为消息通信载体，类似于**广播模式**；发布者发布一条消息，该消息通过主题传递给所有的订阅者，**在一条消息广播之后才订阅的用户则是收不到该条消息的**。

**在发布-订阅模型中，如果只有一个订阅者，那它和队列模型就基本是一样的了。所以说，发布-订阅模型在功能层面上是可以兼容队列模型的。**

**Kafka采用的就是发布-订阅模型。**

> **RocketMQ的消息模型和Kafka基本是完全一样的。唯一的区别是Kafka中没有队列这个概念，与之对应的是Partition（分区）。**

#### 什么是Producer、Consumer、Broker、Topic、Partition？

Kafka将生产者发布的消息发送到**Topic（主题）**中，需要这些消息的消费者可以订阅这些**Topic（主题）**，如下图所示：

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/high-performance/message-queue20210507200944439.png)

上面这张图也为我们引出了，Kafka比较重要的几个概念：

1. **Producer（生产者）**:产生消息的一方。
2. **Consumer（消费者）**:消费消息的一方。
3. **Broker（代理）**:可以看作是一个独立的Kafka实例。多个KafkaBroker组成一个KafkaCluster。

同时，你一定也注意到每个Broker中又包含了Topic以及Partition这两个重要的概念：

- **Topic（主题）**:Producer将消息发送到特定的主题，Consumer通过订阅特定的Topic(主题)来消费消息。
- **Partition（分区）**:Partition属于Topic的一部分。一个Topic可以有多个Partition，并且同一Topic下的Partition可以分布在不同的Broker上，这也就表明一个Topic可以横跨多个Broker。这正如我上面所画的图一样。

> 划重点：**Kafka中的Partition（分区）实际上可以对应成为消息队列中的队列。这样是不是更好理解一点？**

#### Kafka的多副本机制了解吗？带来了什么好处？

还有一点我觉得比较重要的是Kafka为分区（Partition）引入了多副本（Replica）机制。分区（Partition）中的多个副本之间会有一个叫做leader的家伙，其他副本称为follower。我们发送的消息会被发送到leader副本，然后follower副本才能从leader副本中拉取消息进行同步。

> 生产者和消费者只与leader副本交互。你可以理解为其他副本只是leader副本的拷贝，它们的存在只是为了保证消息存储的安全性。当leader副本发生故障时会从follower中选举出一个leader,但是follower中如果有和leader同步程度达不到要求的参加不了leader的竞选。

**Kafka的多分区（Partition）以及多副本（Replica）机制有什么好处呢？**

1. Kafka通过给特定Topic指定多个Partition,而各个Partition可以分布在不同的Broker上,这样便能提供比较好的并发能力（负载均衡）。
2. Partition可以指定对应的Replica数,这也极大地提高了消息存储的安全性,提高了容灾能力，不过也相应的增加了所需要的存储空间。

#### Zookeeper在Kafka中的作用知道吗？

> **要想搞懂zookeeper在Kafka中的作用一定要自己搭建一个Kafka环境然后自己进zookeeper去看一下有哪些文件夹和Kafka有关，每个节点又保存了什么信息。**一定不要光看不实践，这样学来的也终会忘记！这部分内容参考和借鉴了这篇文章：https://www.jianshu.com/p/a036405f989c。

下图就是我的本地Zookeeper，它成功和我本地的Kafka关联上（以下文件夹结构借助idea插件Zookeepertool实现）。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/zookeeper-kafka.jpg)

ZooKeeper主要为Kafka提供元数据的管理的功能。

从图中我们可以看出，Zookeeper主要为Kafka做了下面这些事情：

1. **Broker注册**：在Zookeeper上会有一个专门**用来进行Broker服务器列表记录**的节点。每个Broker在启动时，都会到Zookeeper上进行注册，即到`/brokers/ids`下创建属于自己的节点。每个Broker就会将自己的IP地址和端口等信息记录到该节点中去
2. **Topic注册**：在Kafka中，同一个**Topic的消息会被分成多个分区**并将其分布在多个Broker上，**这些分区信息及与Broker的对应关系**也都是由Zookeeper在维护。比如我创建了一个名字为my-topic的主题并且它有两个分区，对应到zookeeper中会创建这些文件夹：`/brokers/topics/my-topic/Partitions/0`、`/brokers/topics/my-topic/Partitions/1`
3. **负载均衡**：上面也说过了Kafka通过给特定Topic指定多个Partition,而各个Partition可以分布在不同的Broker上,这样便能提供比较好的并发能力。对于同一个Topic的不同Partition，Kafka会尽力将这些Partition分布到不同的Broker服务器上。当生产者产生消息后也会尽量投递到不同Broker的Partition里面。当Consumer消费的时候，Zookeeper可以根据当前的Partition数量以及Consumer数量来实现动态负载均衡。
4. ......

#### Kafka如何保证消息的消费顺序？

我们在使用消息队列的过程中经常有业务场景需要严格保证消息的消费顺序，比如我们同时发了2个消息，这2个消息对应的操作分别对应的数据库操作是：

1. 更改用户会员等级。
2. 根据会员等级计算订单价格。

假如这两条消息的消费顺序不一样造成的最终结果就会截然不同。

我们知道Kafka中Partition(分区)是真正保存消息的地方，我们发送的消息都被放在了这里。而我们的Partition(分区)又存在于Topic(主题)这个概念中，并且我们可以给特定Topic指定多个Partition。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/KafkaTopicPartionsLayout.png)

每次添加消息到Partition(分区)的时候都会采用尾加法，如上图所示。**Kafka只能为我们保证Partition(分区)中的消息有序。**

> 消息在被追加到Partition(分区)的时候都会分配一个特定的偏移量（offset）。Kafka通过偏移量（offset）来保证消息在分区内的顺序性。

所以，我们就有一种很简单的保证消息消费顺序的方法：**1个Topic只对应一个Partition**。这样当然可以解决问题，但是破坏了Kafka的设计初衷。

Kafka中发送1条消息的时候，可以指定topic,partition,key,data（数据）4个参数。如果你发送消息的时候指定了Partition的话，所有消息都会被发送到指定的Partition。并且，同一个key的消息可以保证只发送到同一个partition，这个我们可以采用表/对象的id来作为key。

总结一下，对于如何保证Kafka中消息消费的顺序，有了下面两种方法：

1. 1个Topic只对应一个Partition。
2. （推荐）发送消息的时候指定key/Partition。

当然不仅仅只有上面两种方法，上面两种方法是我觉得比较好理解的，

#### Kafka如何保证消息不丢失

1. 生产者丢失消息的情况

生产者(Producer)调用`send`方法发送消息之后，消息可能因为网络问题并没有发送过去。

所以，我们不能默认在调用`send`方法发送消息之后消息发送成功了。为了确定消息是发送成功，我们要判断消息发送的结果。但是要注意的是Kafka生产者(Producer)使用`send`方法发送消息实际上是异步的操作，我们可以通过`get()`方法获取调用结果，但是这样也让它变为了同步操作，示例代码如下：

> **详细代码见我的这篇文章：[Kafka系列第三篇！10分钟学会如何在Spring Boot程序中使用Kafka作为消息队列](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486269&idx=2&sn=ec00417ad641dd8c3d145d74cafa09ce&chksm=cea244f6f9d5cde0c8eb233fcc4cf82e11acd06446719a7af55230649863a3ddd95f78d111de&token=1633957262&lang=zh_CN#rd)**

```java
SendResult<String, Object> sendResult = kafkaTemplate.send(topic, o).get();
if (sendResult.getRecordMetadata() != null) {
  logger.info("生产者成功发送消息到" + sendResult.getProducerRecord().topic() + "-> " + sendRe
              sult.getProducerRecord().value().toString());
}
```

但是一般不推荐这么做！可以采用为其添加回调函数的形式，示例代码如下：


```java
        ListenableFuture<SendResult<String, Object>> future = kafkaTemplate.send(topic, o);
        future.addCallback(result -> logger.info("生产者成功发送消息到topic:{} partition:{}的消息", result.getRecordMetadata().topic(), result.getRecordMetadata().partition()),
                ex -> logger.error("生产者发送消失败，原因：{}", ex.getMessage()));
```

如果消息发送失败的话，我们检查失败的原因之后重新发送即可！

**另外这里推荐为Producer的`retries`（重试次数）设置一个比较合理的值，一般是3，但是为了保证消息不丢失的话一般会设置比较大一点。设置完成之后，当出现网络问题之后能够自动重试消息发送，避免消息丢失。另外，建议还要设置重试间隔，因为间隔太小的话重试的效果就不明显了，网络波动一次你3次一下子就重试完了**

2. 消费者丢失消息的情况

我们知道消息在被追加到Partition(分区)的时候都会分配一个特定的偏移量（offset）。偏移量(offset)表示Consumer当前消费到的Partition(分区)的所在的位置。Kafka通过偏移量（offset）可以保证消息在分区内的顺序性。

![kafka offset](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/kafka-offset.jpg)

当消费者拉取到了分区的某个消息之后，消费者会自动提交了offset。自动提交的话会有一个问题，试想一下，当消费者刚拿到这个消息准备进行真正消费的时候，突然挂掉了，消息实际上并没有被消费，但是offset却被自动提交了。

**解决办法也比较粗暴，我们手动关闭自动提交offset，每次在真正消费完消息之后再自己手动提交offset。**但是，细心的朋友一定会发现，这样会带来消息被重新消费的问题。比如你刚刚消费完消息之后，还没提交offset，结果自己挂掉了，那么这个消息理论上就会被消费两次。

3. Kafka弄丢了消息

我们知道Kafka为分区（Partition）引入了多副本（Replica）机制。分区（Partition）中的多个副本之间会有一个叫做leader的家伙，其他副本称为follower。我们发送的消息会被发送到leader副本，然后follower副本才能从leader副本中拉取消息进行同步。生产者和消费者只与leader副本交互。你可以理解为其他副本只是leader副本的拷贝，它们的存在只是为了保证消息存储的安全性。

**试想一种情况：假如leader副本所在的broker突然挂掉，那么就要从follower副本重新选出一个leader，但是leader的数据还有一些没有被follower副本的同步的话，就会造成消息丢失。**

**设置acks=all**

解决办法就是我们设置**acks=all**。acks是Kafka生产者(Producer)很重要的一个参数。

acks的默认值即为1，代表我们的消息被leader副本接收之后就算被成功发送。当我们配置**acks=all**表示只有所有ISR列表的副本全部收到消息时，生产者才会接收到来自服务器的响应.这种模式是最高级别的，也是最安全的，可以确保不止一个Broker接收到了消息.该模式的延迟会很高.

**设置replication.factor>=3**

为了保证leader副本能有follower副本能同步消息，我们一般会为topic设置**replication.factor>=3**。这样就可以保证每个分区(partition)至少有3个副本。虽然造成了数据冗余，但是带来了数据的安全性。

**设置min.insync.replicas>1**

一般情况下我们还需要设置**min.insync.replicas>1**，这样配置代表消息至少要被写入到2个副本才算是被成功发送。**min.insync.replicas**的默认值为1，在实际生产中应尽量避免默认值1。

但是，为了保证整个Kafka服务的高可用性，你需要确保**replication.factor>min.insync.replicas**。为什么呢？设想一下假如两者相等的话，只要是有一个副本挂掉，整个分区就无法正常工作了。这明显违反高可用性！一般推荐设置成**replication.factor=min.insync.replicas+1**。

**设置unclean.leader.election.enable=false**

> **Kafka0.11.0.0版本开始unclean.leader.election.enable参数的默认值由原来的true改为false**

我们最开始也说了我们发送的消息会被发送到leader副本，然后follower副本才能从leader副本中拉取消息进行同步。多个follower副本之间的消息同步情况不一样，当我们配置了**unclean.leader.election.enable=false**的话，当leader副本发生故障时就不会从follower副本中和leader同步程度达不到要求的副本中选择出leader，这样降低了消息丢失的可能性。

#### Kafka如何保证消息不重复消费

**kafka出现消息重复消费的原因：**

- 服务端侧已经消费的数据没有成功提交offset（根本原因）。
- Kafka侧由于服务端处理业务时间长或者网络链接等等原因让Kafka认为服务假死，触发了分区rebalance。

**解决方案：**

- 消费消息服务做幂等校验，比如Redis的set、MySQL的主键等天然的幂等功能。这种方法最有效。

- 将`enable.auto.commit`参数设置为false，关闭自动提交，开发者在代码中手动提交offset。那么这里会有个问题：什么时候提交offset合适？

  - 处理完消息再提交：依旧有消息重复消费的风险，和自动提交一样
  - 拉取到消息即提交：会有消息丢失的风险。允许消息延时的场景，一般会采用这种方式。然后，通过定时任务在业务不繁忙（比如凌晨）的时候做数据兜底

> [原文链接](https://javaguide.cn/high-performance/message-queue/kafka-questions-01.html)

#### 相关文章

- [Kafka安装及快速入门](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493725&idx=1&sn=96eb8c16aa8cfe8336334d93c4034ed2&source=41#wechat_redirect)
- [带你涨姿势的认识一下kafka](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491187&idx=1&sn=f0f092e675677d87ea0675f65bb1eb14&source=41#wechat_redirect)
- [Kafka Shell基本命令](https://www.cnblogs.com/xiaodf/p/6093261.html)
- [漫画：图解Kafka，看本篇就足够啦](https://mp.weixin.qq.com/s/FP3z5JMRB9Oo5pr22kr3XA)
- [你能说出Kafka这些原理吗？](https://mp.weixin.qq.com/s/lXds_G5LLs7CArMLyTAF6g)
- [Kafka基本架构及原理](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493937&idx=1&sn=3b2710a62cbb51b3f7c607bfd97664c1&source=41#wechat_redirect)
- [9个Kafka面试题，你会几个？](https://mp.weixin.qq.com/s/LAXrK6WT1bv7I63BmvpXQg)
- [Kafka中副本机制的设计和原理](https://mp.weixin.qq.com/s/yIPIABpAzaHJvGoJ6pv0kg)
- [从面试角度一文学完Kafka](https://mp.weixin.qq.com/s/Ndwrlst-aVdUpcNz-inM3Q)
- [支持百万级TPS，Kafka是怎么做到的？](https://mp.weixin.qq.com/s/Fvd_RcdMHzcwm3fBlpT8vw)
- [Kafka深度剖析](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494362&idx=1&sn=f16c4f215dd83c81957b0b431b147902&source=41#wechat_redirect)
- [Kafka性能篇：为何Kafka这么"快"](https://mp.weixin.qq.com/s/2wGCEWisbynCIjRsO2gxDg)
- [Kafka为什么吞吐量大、速度快？](https://mp.weixin.qq.com/s/fOhlF6crYGXRNHjV0nLQLQ)
- [图解kafka架构与工作原理](https://mp.weixin.qq.com/s/V3BlPGOuGPnmT36a500o7g)
- [面试官：聊一聊kafka线上会遇到哪些问题？](https://mp.weixin.qq.com/s/cmDqGpTiHKRqcSaQVesJVA)
- [《面试八股文》之Kafka21卷](https://mp.weixin.qq.com/s/l75MIcG4cf8C59z1JcfygA)



### RocketMQ

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/high-performance/message-queue/rocketmq-logo.png)

RocketMQ是阿里开源的一款云原生“消息、事件、流”实时数据处理平台，借鉴了Kafka，已经成为Apache顶级项目。

RocketMQ的核心特性（摘自RocketMQ官网）：

- 云原生：生与云，长与云，无限弹性扩缩，K8s友好
- 高吞吐：万亿级吞吐保证，同时满足微服务与大数据场景。
- 流处理：提供轻量、高扩展、高性能和丰富功能的流计算引擎。
- 金融级：金融级的稳定性，广泛用于交易核心链路。
- 架构极简：零外部依赖，Shared-nothing架构。
- 生态友好：无缝对接微服务、实时计算、数据湖等周边生态。

根据官网介绍：

> ApacheRocketMQ自诞生以来，因其架构简单、业务功能丰富、具备极强可扩展性等特点被众多企业开发者以及云厂商广泛采用。历经十余年的大规模场景打磨，RocketMQ已经成为业内共识的金融级可靠业务消息首选方案，被广泛应用于互联网、大数据、移动互联网、物联网等领域的业务场景。

[RocketMQ官网](https://rocketmq.apache.org/)（文档很详细，推荐阅读）

[RocketMQ更新记录](https://github.com/apache/rocketmq/releases)

#### RocketMQ基础知识总结

##### 队列模型和主题模型

在谈RocketMQ的技术架构之前，我们先来了解一下两个名词概念——**队列模型**和**主题模型**。

首先我问一个问题，消息队列为什么要叫消息队列？

你可能觉得很弱智，这玩意不就是存放消息的队列嘛？不叫消息队列叫什么？

的确，早期的消息中间件是通过**队列**这一模型来实现的，可能是历史原因，我们都习惯把消息中间件成为消息队列。

但是，如今例如RocketMQ、Kafka这些优秀的消息中间件不仅仅是通过一个**队列**来实现消息存储的。

###### 队列模型

就像我们理解队列一样，消息中间件的队列模型就真的只是一个队列。画一张图给大家理解。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef3834ae653469.jpg)

在一开始我跟你提到了一个**广播**的概念，也就是说如果我们此时我们需要将一个消息发送给多个消费者(比如此时我需要将信息发送给短信系统和邮件系统)，这个时候单个队列即不能满足需求了。

当然你可以让Producer生产消息放入多个队列中，然后每个队列去对应每一个消费者。问题是可以解决，创建多个队列并且复制多份消息是会很影响资源和性能的。而且，这样子就会导致生产者需要知道具体消费者个数然后去复制对应数量的消息队列，这就违背我们消息中间件的**解耦**这一原则。

###### 主题模型

那么有没有好的方法去解决这一个问题呢？有，那就是**主题模型**或者可以称为**发布订阅模型**。

> 感兴趣的同学可以去了解一下设计模式里面的观察者模式并且手动实现一下，我相信你会有所收获的。

在主题模型中，消息的生产者称为**发布者(Publisher)**，消息的消费者称为**订阅者(Subscriber)**，存放消息的容器称为**主题(Topic)**。

其中，发布者将消息发送到指定主题中，订阅者需要**提前订阅主题**才能接受特定主题的消息。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef3837887d9a54sds.jpg)

###### RocketMQ中的消息模型

RocketMQ中的消息模型就是按照**主题模型**所实现的。你可能会好奇这个**主题**到底是怎么实现的呢？你上面也没有讲到呀！

其实对于主题模型的实现来说每个消息中间件的底层设计都是不一样的，就比如Kafka中的**分区**，RocketMQ中的**队列**，RabbitMQ中的Exchange。我们可以理解为**主题模型/发布订阅模型**就是一个标准，那些中间件只不过照着这个标准去实现而已。

所以，RocketMQ中的**主题模型**到底是如何实现的呢？画一张图尝试着去理解一下。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef383d3e8c9788.jpg)

我们可以看到在整个图中有ProducerGroup、Topic、ConsumerGroup三个角色，我来分别介绍一下他们。

- ProducerGroup生产者组：代表某一类的生产者，比如我们有多个秒杀系统作为生产者，这多个合在一起就是一个ProducerGroup生产者组，它们一般生产相同的消息。
- ConsumerGroup消费者组：代表某一类的消费者，比如我们有多个短信系统作为消费者，这多个合在一起就是一个ConsumerGroup消费者组，它们一般消费相同的消息。
- Topic主题：代表一类消息，比如订单消息，物流消息等等。

你可以看到图中生产者组中的生产者会向主题发送消息，而**主题中存在多个队列**，生产者每次生产消息之后是指定主题中的某个队列发送消息的。

每个主题中都有多个队列(分布在不同的Broker中，如果是集群的话，Broker又分布在不同的服务器中)，集群消费模式下，一个消费者集群多台机器共同消费一个topic的多个队列，**一个队列只会被一个消费者消费**。如果某个消费者挂掉，分组内其它消费者会接替挂掉的消费者继续消费。就像上图中Consumer1和Consumer2分别对应着两个队列，而Consumer3是没有队列对应的，所以一般来讲要控制**消费者组中的消费者个数和主题中队列个数相同**。

当然也可以消费者个数小于队列个数，只不过不太建议。如下图。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef3850c808d707.jpg)

**每个消费组在每个队列上维护一个消费位置**，为什么呢？

因为我们刚刚画的仅仅是一个消费者组，我们知道在发布订阅模式中一般会涉及到多个消费者组，而每个消费者组在每个队列中的消费位置都是不同的。如果此时有多个消费者组，那么消息被一个消费者组消费完之后是不会删除的(因为其它消费者组也需要呀)，它仅仅是为每个消费者组维护一个**消费位移(offset)**，每次消费者组消费完会返回一个成功的响应，然后队列再把维护的消费位移加一，这样就不会出现刚刚消费过的消息再一次被消费了。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef3857fefaa079.jpg)

可能你还有一个问题，**为什么一个主题中需要维护多个队列**？

答案是**提高并发能力**。的确，每个主题中只存在一个队列也是可行的。你想一下，如果每个主题中只存在一个队列，这个队列中也维护着每个消费者组的消费位置，这样也可以做到**发布订阅模式**。如下图。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef38600cdb6d4b.jpg)

但是，这样我生产者是不是只能向一个队列发送消息？又因为需要维护消费位置所以一个队列只能对应一个消费者组中的消费者，这样是不是其他的Consumer就没有用武之地了？从这两个角度来讲，并发度一下子就小了很多。

所以总结来说，RocketMQ通过**使用在一个Topic中配置多个队列并且每个队列维护每个消费者组的消费位置**实现了**主题模式/发布订阅模式**。

##### RocketMQ的架构图

讲完了消息模型，我们理解起RocketMQ的技术架构起来就容易多了。

RocketMQ技术架构中有四大角色NameServer、Broker、Producer、Consumer。

- Broker：主要负责消息的存储、投递和查询以及服务高可用保证。说白了就是消息队列服务器嘛，生产者生产消息到Broker，消费者从Broker拉取消息并消费。

  这里，我还得普及一下关于Broker、Topic和队列的关系。上面我讲解了Topic和队列的关系——一个Topic中存在多个队列，那么这个Topic和队列存放在哪呢？

  **一个Topic分布在多个Broker上，一个Broker可以配置多个Topic，它们是多对多的关系**。

  如果某个Topic消息量很大，应该给它多配置几个队列(上文中提到了提高并发能力)，并且**尽量多分布在不同Broker上，以减轻某个Broker的压力**。

  Topic消息量都比较均匀的情况下，如果某个broker上的队列越多，则该broker压力越大。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef38687488a5a4.jpg)

  > 所以说我们需要配置多个Broker。

- NameServer：不知道你们有没有接触过ZooKeeper和SpringCloud中的Eureka，它其实也是一个**注册中心**，主要提供两个功能：**Broker管理**和**路由信息管理**。说白了就是Broker会将自己的信息注册到NameServer中，此时NameServer就存放了很多Broker的信息(Broker的路由表)，消费者和生产者就从NameServer中获取路由表然后照着路由表的信息和对应的Broker进行通信(生产者和消费者定期会向NameServer去查询相关的Broker的信息)。

- Producer：消息发布的角色，支持分布式集群方式部署。说白了就是生产者。

- Consumer：消息消费的角色，支持分布式集群方式部署。支持以push推，pull拉两种模式对消息进行消费。同时也支持集群方式和广播方式的消费，它提供实时消息订阅机制。说白了就是消费者。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef386c6d1e8bdb.jpg)

嗯？你可能会发现一个问题，这老家伙NameServer干啥用的，这不多余吗？直接Producer、Consumer和Broker直接进行生产消息，消费消息不就好了么？

但是，我们上文提到过Broker是需要保证高可用的，如果整个系统仅仅靠着一个Broker来维持的话，那么这个Broker的压力会不会很大？所以我们需要使用多个Broker来保证**负载均衡**。

如果说，我们的消费者和生产者直接和多个Broker相连，那么当Broker修改的时候必定会牵连着每个生产者和消费者，这样就会产生耦合问题，而NameServer注册中心就是用来解决这个问题的。

> 如果还不是很理解的话，可以去看我介绍SpringCloud的那篇文章，其中介绍了Eureka注册中心。

当然，RocketMQ中的技术架构肯定不止前面那么简单，因为上面图中的四个角色都是需要做集群的。给出一张官网的架构图，尝试理解一下。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef386fa3be1e53.jpg)

第一、我们的Broker**做了集群并且还进行了主从部署**，由于消息分布在各个Broker上，一旦某个Broker宕机，则该Broker上的消息读写都会受到影响。所以Rocketmq提供了master/slave的结构，salve定时从master同步数据(同步刷盘或者异步刷盘)，如果master宕机，**则slave提供消费服务，但是不能写入消息**(后面我还会提到哦)。

第二、为了保证HA，我们的NameServer也做了集群部署，但是请注意它是**去中心化**的。也就意味着它没有主节点，你可以很明显地看出NameServer的所有节点是没有进行InfoReplicate的，在RocketMQ中是通过**单个Broker和所有NameServer保持长连接**，并且在每隔30秒Broker会向所有Nameserver发送心跳，心跳包含了自身的Topic配置信息，这个步骤就对应这上面的RoutingInfo。

第三、在生产者需要向Broker发送消息的时候，**需要先从NameServer获取关于Broker的路由信息**，然后通过**轮询**的方法去向每个队列中生产数据以达到**负载均衡**的效果。

第四、消费者通过NameServer获取所有Broker的路由信息后，向Broker发送Pull请求来获取消息数据。Consumer可以以两种模式启动——**广播（Broadcast）和集群（Cluster）**。广播模式下，一条消息会发送给**同一个消费组中的所有消费者**，集群模式下消息只会发送给一个消费者。

##### 如何解决顺序消费、重复消费

其实，这些东西都是我在介绍消息队列带来的一些副作用的时候提到的，也就是说，这些问题不仅仅挂钩于RocketMQ，而是应该每个消息中间件都需要去解决的。

在上面我介绍RocketMQ的技术架构的时候我已经向你展示了**它是如何保证高可用的**，这里不涉及运维方面的搭建，如果你感兴趣可以自己去官网上照着例子搭建属于你自己的RocketMQ集群。

> 其实Kafka的架构基本和RocketMQ类似，只是它注册中心使用了Zookeeper、它的**分区**就相当于RocketMQ中的**队列**。还有一些小细节不同会在后面提到。

###### 顺序消费

在上面的技术架构介绍中，我们已经知道了**RocketMQ在主题上是无序的、它只有在队列层面才是保证有序**的。

这又扯到两个概念——**普通顺序**和**严格顺序**。

所谓普通顺序是指消费者通过**同一个消费队列收到的消息是有顺序的**，不同消息队列收到的消息则可能是无顺序的。普通顺序消息在Broker**重启情况下不会保证消息顺序性**(短暂时间)。

所谓严格顺序是指消费者收到的**所有消息**均是有顺序的。严格顺序消息**即使在异常情况下也会保证消息的顺序性**。

但是，严格顺序看起来虽好，实现它可会付出巨大的代价。如果你使用严格顺序模式，Broker集群中只要有一台机器不可用，则整个集群都不可用。你还用啥？现在主要场景也就在binlog同步。

一般而言，我们的MQ都是能容忍短暂的乱序，所以推荐使用普通顺序模式。

那么，我们现在使用了**普通顺序模式**，我们从上面学习知道了在Producer生产消息的时候会进行轮询(取决你的负载均衡策略)来向同一主题的不同消息队列发送消息。那么如果此时我有几个消息分别是同一个订单的创建、支付、发货，在轮询的策略下这**三个消息会被发送到不同队列**，因为在不同的队列此时就无法使用RocketMQ带来的队列有序特性来保证消息有序性了。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef3874585e096e.jpg)

那么，怎么解决呢？

其实很简单，我们需要处理的仅仅是将同一语义下的消息放入同一个队列(比如这里是同一个订单)，那我们就可以使用**Hash取模法**来保证同一个订单在同一个队列中就行了。

###### 重复消费

就两个字——**幂等**。在编程中一个*幂等*操作的特点是其任意多次执行所产生的影响均与一次执行的影响相同。比如说，这个时候我们有一个订单的处理积分的系统，每当来一个消息的时候它就负责为创建这个订单的用户的积分加上相应的数值。可是有一次，消息队列发送给订单系统FrancisQ的订单信息，其要求是给FrancisQ的积分加上500。但是积分系统在收到FrancisQ的订单信息处理完成之后返回给消息队列处理成功的信息的时候出现了网络波动(当然还有很多种情况，比如Broker意外重启等等)，这条回应没有发送成功。

那么，消息队列没收到积分系统的回应会不会尝试重发这个消息？问题就来了，我再发这个消息，万一它又给FrancisQ的账户加上500积分怎么办呢？

所以我们需要给我们的消费者实现**幂等**，也就是对同一个消息的处理结果，执行多少次都不变。

那么如何给业务实现幂等呢？这个还是需要结合具体的业务的。你可以使用**写入Redis**来保证，因为Redis的key和value就是天然支持幂等的。当然还有使用**数据库插入法**，基于数据库的唯一键来保证重复数据不会被插入多条。

不过最主要的还是需要**根据特定场景使用特定的解决方案**，你要知道你的消息消费是否是完全不可重复消费还是可以忍受重复消费的，然后再选择强校验和弱校验的方式。毕竟在CS领域还是很少有技术银弹的说法。

而在整个互联网领域，幂等不仅仅适用于消息队列的重复消费问题，这些实现幂等的方法，也同样适用于，**在其他场景中来解决重复请求或者重复调用的问题**。比如将HTTP服务设计成幂等的，**解决前端或者APP重复提交表单数据的问题**，也可以将一个微服务设计成幂等的，解决RPC框架自动重试导致的**重复调用问题**。

##### 分布式事务

如何解释分布式事务呢？事务大家都知道吧？**要么都执行要么都不执行**。在同一个系统中我们可以轻松地实现事务，但是在分布式架构中，我们有很多服务是部署在不同系统之间的，而不同服务之间又需要进行调用。比如此时我下订单然后增加积分，如果保证不了分布式事务的话，就会出现A系统下了订单，但是B系统增加积分失败或者A系统没有下订单，B系统却增加了积分。前者对用户不友好，后者对运营商不利，这是我们都不愿意见到的。

那么，如何去解决这个问题呢？

如今比较常见的分布式事务实现有2PC、TCC和事务消息(half半消息机制)。每一种实现都有其特定的使用场景，但是也有各自的问题，**都不是完美的解决方案**。

在RocketMQ中使用的是**事务消息加上事务反查机制**来解决分布式事务问题的。我画了张图，大家可以对照着图进行理解。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef38798d7a987f.png)

在第一步发送的half消息，它的意思是**在事务提交之前，对于消费者来说，这个消息是不可见的**。

> 那么，如何做到写入消息但是对用户不可见呢？RocketMQ事务消息的做法是：如果消息是half消息，将备份原消息的主题与消息消费队列，然后**改变主题**为RMQ_SYS_TRANS_HALF_TOPIC。由于消费组未订阅该主题，故消费端无法消费half类型的消息，**然后RocketMQ会开启一个定时任务，从Topic为RMQ_SYS_TRANS_HALF_TOPIC中拉取消息进行消费**，根据生产者组获取一个服务提供者发送回查事务状态请求，根据事务状态来决定是提交或回滚消息。

你可以试想一下，如果没有从第5步开始的**事务反查机制**，如果出现网路波动第4步没有发送成功，这样就会产生MQ不知道是不是需要给消费者消费的问题，他就像一个无头苍蝇一样。在RocketMQ中就是使用的上述的事务反查来解决的，而在Kafka中通常是直接抛出一个异常让用户来自行解决。

你还需要注意的是，在MQServer指向系统B的操作已经和系统A不相关了，也就是说在消息队列中的分布式事务是——**本地事务和存储消息到消息队列才是同一个事务**。这样也就产生了事务的**最终一致性**，因为整个过程是异步的，**每个系统只要保证它自己那一部分的事务就行了**。

##### 消息堆积问题

在上面我们提到了消息队列一个很重要的功能——**削峰**。那么如果这个峰值太大了导致消息堆积在队列中怎么办呢？

其实这个问题可以将它广义化，因为产生消息堆积的根源其实就只有两个——生产者生产太快或者消费者消费太慢。

我们可以从多个角度去思考解决这个问题，当流量到峰值的时候是因为生产者生产太快，我们可以使用一些**限流降级**的方法，当然你也可以增加多个消费者实例去水平扩展增加消费能力来匹配生产的激增。如果消费者消费过慢的话，我们可以先检查**是否是消费者出现了大量的消费错误**，或者打印一下日志查看是否是哪一个线程卡死，出现了锁资源不释放等等的问题。

> 当然，最快速解决消息堆积问题的方法还是增加消费者实例，不过**同时你还需要增加每个主题的队列数量**。
> 
> 别忘了在RocketMQ中，**一个队列只会被一个消费者消费**，如果你仅仅是增加消费者实例就会出现我一开始给你画架构图的那种情况。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef387d939ab66d.jpg)

##### 回溯消费

回溯消费是指Consumer已经消费成功的消息，由于业务上需求需要重新消费，在RocketMQ中，Broker在向Consumer投递成功消息后，**消息仍然需要保留**。并且重新消费一般是按照时间维度，例如由于Consumer系统故障，恢复后需要重新消费1小时前的数据，那么Broker要提供一种机制，可以按照时间维度来回退消费进度。RocketMQ支持按照时间回溯消费，时间维度精确到毫秒。

这是官方文档的解释，我直接照搬过来就当科普了😁😁😁。

##### RocketMQ的刷盘机制

上面我讲了那么多的RocketMQ的架构和设计原理，你有没有好奇

在Topic中的**队列是以什么样的形式存在的？**

**队列中的消息又是如何进行存储持久化的呢？**

我在上文中提到的**同步刷盘**和**异步刷盘**又是什么呢？它们会给持久化带来什么样的影响呢？

下面我将给你们一一解释。

###### 同步刷盘和异步刷盘

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef387fba311cda.jpg)

如上图所示，在同步刷盘中需要等待一个刷盘成功的ACK，同步刷盘对MQ消息可靠性来说是一种不错的保障，但是**性能上会有较大影响**，一般地适用于金融等特定业务场景。

而异步刷盘往往是开启一个线程去异步地执行刷盘操作。消息刷盘采用后台异步线程提交的方式进行，**降低了读写延迟**，提高了MQ的性能和吞吐量，一般适用于如发验证码等对于消息保证要求不太高的业务场景。

一般地，**异步刷盘只有在Broker意外宕机的时候会丢失部分数据**，你可以设置Broker的参数FlushDiskType来调整你的刷盘策略(ASYNC_FLUSH或者SYNC_FLUSH)。

###### 同步复制和异步复制

上面的同步刷盘和异步刷盘是在单个结点层面的，而同步复制和异步复制主要是指的Borker主从模式下，主节点返回消息给客户端的时候是否需要同步从节点。

- 同步复制：也叫“同步双写”，也就是说，**只有消息同步双写到主从节点上时才返回写入成功**。
- 异步复制：**消息写入主节点之后就直接返回写入成功**。

然而，很多事情是没有完美的方案的，就比如我们进行消息写入的节点越多就更能保证消息的可靠性，但是随之的性能也会下降，所以需要程序员根据特定业务场景去选择适应的主从复制方案。

那么，**异步复制会不会也像异步刷盘那样影响消息的可靠性呢？**

答案是不会的，因为两者就是不同的概念，对于消息可靠性是通过不同的刷盘策略保证的，而像异步同步复制策略仅仅是影响到了**可用性**。为什么呢？其主要原因**是RocketMQ是不支持自动主从切换的，当主节点挂掉之后，生产者就不能再给这个主节点生产消息了**。

比如这个时候采用异步复制的方式，在主节点还未发送完需要同步的消息的时候主节点挂掉了，这个时候从节点就少了一部分消息。但是此时生产者无法再给主节点生产消息了，**消费者可以自动切换到从节点进行消费**(仅仅是消费)，所以在主节点挂掉的时间只会产生主从结点短暂的消息不一致的情况，降低了可用性，而当主节点重启之后，从节点那部分未来得及复制的消息还会继续复制。

在单主从架构中，如果一个主节点挂掉了，那么也就意味着整个系统不能再生产了。那么这个可用性的问题能否解决呢？**一个主从不行那就多个主从的呗**，别忘了在我们最初的架构图中，每个Topic是分布在不同Broker中的。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef38687488a5a4.jpg)

但是这种复制方式同样也会带来一个问题，那就是无法保证**严格顺序**。在上文中我们提到了如何保证的消息顺序性是通过将一个语义的消息发送在同一个队列中，使用Topic下的队列来保证顺序性的。如果此时我们主节点A负责的是订单A的一系列语义消息，然后它挂了，这样其他节点是无法代替主节点A的，如果我们任意节点都可以存入任何消息，那就没有顺序性可言了。

而在RocketMQ中采用了Dledger解决这个问题。他要求在写入消息的时候，要求**至少消息复制到半数以上的节点之后**，才给客⼾端返回写⼊成功，并且它是⽀持通过选举来动态切换主节点的。这里我就不展开说明了，读者可以自己去了解。

> 也不是说Dledger是个完美的方案，至少在Dledger选举过程中是无法提供服务的，而且他必须要使用三个节点或以上，如果多数节点同时挂掉他也是无法保证可用性的，而且要求消息复制半数以上节点的效率和直接异步复制还是有一定的差距的。

###### 存储机制

还记得上面我们一开始的三个问题吗？到这里第三个问题已经解决了。

但是，在Topic中的**队列是以什么样的形式存在的？队列中的消息又是如何进行存储持久化的呢？**还未解决，其实这里涉及到了RocketMQ是如何设计它的存储结构了。我首先想大家介绍RocketMQ消息存储架构中的三大角色——CommitLog、ConsumeQueue和IndexFile。

- CommitLog：**消息主体以及元数据的存储主体**，存储Producer端写入的消息主体内容,消息内容不是定长的。单个文件大小默认1G，文件名长度为20位，左边补零，剩余为起始偏移量，比如00000000000000000000代表了第一个文件，起始偏移量为0，文件大小为1G=1073741824；当第一个文件写满了，第二个文件为00000000001073741824，起始偏移量为1073741824，以此类推。消息主要是**顺序写入日志文件**，当文件满了，写入下一个文件。
- ConsumeQueue：消息消费队列，**引入的目的主要是提高消息消费的性能**(我们再前面也讲了)，由于RocketMQ是基于主题Topic的订阅模式，消息消费是针对主题进行的，如果要遍历commitlog文件中根据Topic检索消息是非常低效的。Consumer即可根据ConsumeQueue来查找待消费的消息。其中，ConsumeQueue（逻辑消费队列）**作为消费消息的索引**，保存了指定Topic下的队列消息在CommitLog中的**起始物理偏移量offset，消息大小size和消息Tag的HashCode值。consumequeue文件可以看成是基于topic的commitlog索引文件**，故consumequeue文件夹的组织方式如下：topic/queue/file三层组织结构，具体存储路径为：$HOME/store/consumequeue/{topic}/{queueId}/{fileName}。同样consumequeue文件采取定长设计，每一个条目共20个字节，分别为8字节的commitlog物理偏移量、4字节的消息长度、8字节taghashcode，单个文件由30W个条目组成，可以像数组一样随机访问每一个条目，每个ConsumeQueue文件大小约5.72M；
- IndexFile：IndexFile（索引文件）提供了一种可以通过key或时间区间来查询消息的方法。这里只做科普不做详细介绍。

总结来说，整个消息存储的结构，最主要的就是CommitLoq和ConsumeQueue。而ConsumeQueue你可以大概理解为Topic中的队列。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef3884c02acc72.png)

RocketMQ采用的是**混合型的存储结构**，即为Broker单个实例下所有的队列共用一个日志数据文件来存储消息。有意思的是在同样高并发的Kafka中会为每个Topic分配一个存储文件。这就有点类似于我们有一大堆书需要装上书架，RockeMQ是不分书的种类直接成批的塞上去的，而Kafka是将书本放入指定的分类区域的。

而RocketMQ为什么要这么做呢？原因是**提高数据的写入效率**，不分Topic意味着我们有更大的几率获取**成批**的消息进行数据写入，但也会带来一个麻烦就是读取消息的时候需要遍历整个大文件，这是非常耗时的。

所以，在RocketMQ中又使用了ConsumeQueue作为每个队列的索引文件来**提升读取消息的效率**。我们可以直接根据队列的消息序号，计算出索引的全局位置（索引序号*索引固定⻓度20），然后直接读取这条索引，再根据索引中记录的消息的全局位置，找到消息。

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/16ef388763c25c62.jpg)

首先，在最上面的那一块就是我刚刚讲的你现在可以直接**把ConsumerQueue理解为Queue**。

在图中最左边说明了红色方块代表被写入的消息，虚线方块代表等待被写入的。左边的生产者发送消息会指定Topic、QueueId和具体消息内容，而在Broker中管你是哪门子消息，他直接**全部顺序存储到了CommitLog**。而根据生产者指定的Topic和QueueId将这条消息本身在CommitLog的偏移(offset)，消息本身大小，和tag的hash值存入对应的ConsumeQueue索引文件中。而在每个队列中都保存了ConsumeOffset即每个消费者组的消费位置，而消费者拉取消息进行消费的时候只需要根据ConsumeOffset获取下一个未被消费的消息就行了。

为什么CommitLog文件要设计成固定大小的长度呢？提醒：**内存映射机制**

[RocketMQ高级功能代码实现](https://mp.weixin.qq.com/s/WVZRVMUACXuYwuJLjGIUHA)

#### RocketMQ常见面试题总结

##### 单机版消息中心

一个消息中心，最基本的需要支持多生产者、多消费者，例如下：


```java
class Scratch {
    public static void main(String[] args) {
        // 实际中会有nameserver服务来找到broker,具体位置以及broker主从信息
        Broker broker = new Broker();
        Producer producer1 = new Producer();
        producer1.connectBroker(broker);
        Producer producer2 = new Producer();
        producer2.connectBroker(broker);

        Consumer consumer1 = new Consumer();
        consumer1.connectBroker(broker);
        Consumer consumer2 = new Consumer();
        consumer2.connectBroker(broker);

        for (int i = 0; i < 2; i++) {
            producer1.asyncSendMsg("producer1 send msg" + i);
            producer2.asyncSendMsg("producer2 send msg" + i);
        }
        System.out.println("broker has msg:" + broker.getAllMagByDisk());

        for (int i = 0; i < 1; i++) {
            System.out.println("consumer1 consume msg：" + consumer1.syncPullMsg());
        }
        for (int i = 0; i < 3; i++) {
            System.out.println("consumer2 consume msg：" + consumer2.syncPullMsg());
        }
    }

}

class Producer {

    private Broker broker;

    public void connectBroker(Broker broker) {
        this.broker = broker;
    }

    public void asyncSendMsg(String msg) {
        if (broker == null) {
            throw new RuntimeException("please connect broker first");
        }
        new Thread(() -> {
            broker.sendMsg(msg);
        }).start();
    }
}

class Consumer {
    private Broker broker;

    public void connectBroker(Broker broker) {
        this.broker = broker;
    }

    public String syncPullMsg() {
        return broker.getMsg();
    }

}

class Broker {

    // 对应RocketMQ中MessageQueue，默认情况下1个Topic包含4个MessageQueue
    private LinkedBlockingQueue<String> messageQueue = new LinkedBlockingQueue(Integer.MAX_VALUE);

    // 实际发送消息到broker服务器使用Netty发送
    public void sendMsg(String msg) {
        try {
            messageQueue.put(msg);
            // 实际会同步或异步落盘，异步落盘使用的定时任务定时扫描落盘
        } catch (InterruptedException e) {

        }
    }

    public String getMsg() {
        try {
            return messageQueue.take();
        } catch (InterruptedException e) {

        }
        return null;
    }

    public String getAllMagByDisk() {
        StringBuilder sb = new StringBuilder("\n");
        messageQueue.iterator().forEachRemaining((msg) -> {
            sb.append(msg + "\n");
        });
        return sb.toString();
    }
}
```

问题：

1. 没有实现真正执行消息存储落盘
2. 没有实现NameServer去作为注册中心，定位服务
3. 使用LinkedBlockingQueue作为消息队列，注意，参数是无限大，在真正RocketMQ也是如此是无限大，理论上不会出现对进来的数据进行抛弃，但是会有内存泄漏问题（阿里巴巴开发手册也因为这个问题，建议我们使用自制线程池）
4. 没有使用多个队列（即多个LinkedBlockingQueue），RocketMQ的顺序消息是通过生产者和消费者同时使用同一个MessageQueue来实现，但是如果我们只有一个MessageQueue，那我们天然就支持顺序消息
5. 没有使用MappedByteBuffer来实现文件映射从而使消息数据落盘非常的快（实际RocketMQ使用的是FileChannel+DirectBuffer）

##### 分布式消息中心

###### 问题与解决

**消息丢失的问题**

1. 当你系统需要保证百分百消息不丢失，你可以使用生产者每发送一个消息，Broker同步返回一个消息发送成功的反馈消息
2. 即每发送一个消息，同步落盘后才返回生产者消息发送成功，这样只要生产者得到了消息发送生成的返回，事后除了硬盘损坏，都可以保证不会消息丢失
3. 但是这同时引入了一个问题，同步落盘怎么才能快？

**同步落盘怎么才能快**

1. 使用FileChannel+DirectBuffer池，使用堆外内存，加快内存拷贝
2. 使用数据和索引分离，当消息需要写入时，使用commitlog文件顺序写，当需要定位某个消息时，查询index文件来定位，从而减少文件IO随机读写的性能损耗

**消息堆积的问题**

1. 后台定时任务每隔72小时，删除旧的没有使用过的消息信息
2. 根据不同的业务实现不同的丢弃任务，具体参考线程池的AbortPolicy，例如FIFO/LRU等（RocketMQ没有此策略）
3. 消息定时转移，或者对某些重要的TAG型（支付型）消息真正落库

**定时消息的实现**

1. 实际RocketMQ没有实现任意精度的定时消息，它只支持某些特定的时间精度的定时消息
2. 实现定时消息的原理是：创建特定时间精度的MessageQueue，例如生产者需要定时1s之后被消费者消费，你只需要将此消息发送到特定的Topic，例如：MessageQueue-1表示这个MessageQueue里面的消息都会延迟一秒被消费，然后Broker会在1s后发送到消费者消费此消息，使用newSingleThreadScheduledExecutor实现

**顺序消息的实现**

1. 与定时消息同原理，生产者生产消息时指定特定的MessageQueue，消费者消费消息时，消费特定的MessageQueue，其实单机版的消息中心在一个MessageQueue就天然支持了顺序消息
2. 注意：同一个MessageQueue保证里面的消息是顺序消费的前提是：消费者是串行的消费该MessageQueue，因为就算MessageQueue是顺序的，但是当并行消费时，还是会有顺序问题，但是串行消费也同时引入了两个问题：

> 1. 引入锁来实现串行
> 2. 前一个消费阻塞时后面都会被阻塞

**分布式消息的实现**

1. 需要前置知识：2PC
2. RocketMQ4.3起支持，原理为2PC，即两阶段提交，prepared->commit/rollback
3. 生产者发送事务消息，假设该事务消息Topic为Topic1-Trans，Broker得到后首先更改该消息的Topic为Topic1-Prepared，该Topic1-Prepared对消费者不可见。然后定时回调生产者的本地事务A执行状态，根据本地事务A执行状态，来是否将该消息修改为Topic1-Commit或Topic1-Rollback，消费者就可以正常找到该事务消息或者不执行等

> 注意，就算是事务消息最后回滚了也不会物理删除，只会逻辑删除该消息

**消息的push实现**

1. 注意，RocketMQ已经说了自己会有低延迟问题，其中就包括这个消息的push延迟问题
2. 因为这并不是真正的将消息主动的推送到消费者，而是Broker定时任务每5s将消息推送到消费者
3. pull模式需要我们手动调用consumer拉消息，而push模式则只需要我们提供一个listener即可实现对消息的监听，而实际上，RocketMQ的push模式是基于pull模式实现的，它没有实现真正的push。
4. push方式里，consumer把轮询过程封装了，并注册MessageListener监听器，取到消息后，唤醒MessageListener的consumeMessage()来消费，对用户而言，感觉消息是被推送过来的。

**消息重复发送的避免**

1. RocketMQ会出现消息重复发送的问题，因为在网络延迟的情况下，这种问题不可避免的发生，如果非要实现消息不可重复发送，那基本太难，因为网络环境无法预知，还会使程序复杂度加大，因此默认允许消息重复发送
2. RocketMQ让使用者在消费者端去解决该问题，即需要消费者端在消费消息时支持幂等性的去消费消息
3. 最简单的解决方案是每条消费记录有个消费状态字段，根据这个消费状态字段来判断是否消费或者使用一个集中式的表，来存储所有消息的消费状态，从而避免重复消费
4. 具体实现可以查询关于消息幂等消费的解决方案

**广播消费与集群消费**

1. 消息消费区别：广播消费，订阅该Topic的消息者们都会消费**每个**消息。集群消费，订阅该Topic的消息者们只会有一个去消费**某个**消息
2. 消息落盘区别：具体表现在消息消费进度的保存上。广播消费，由于每个消费者都独立的去消费每个消息，因此每个消费者各自保存自己的消息消费进度。而集群消费下，订阅了某个Topic，而旗下又有多个MessageQueue，每个消费者都可能会去消费不同的MessageQueue，因此总体的消费进度保存在Broker上集中的管理

**RocketMQ不使用ZooKeeper作为注册中心的原因，以及自制的NameServer优缺点？**

1. ZooKeeper作为支持顺序一致性的中间件，在某些情况下，它为了满足一致性，会丢失一定时间内的可用性，RocketMQ需要注册中心只是为了发现组件地址，在某些情况下，RocketMQ的注册中心可以出现数据不一致性，这同时也是NameServer的缺点，因为NameServer集群间互不通信，它们之间的注册信息可能会不一致
2. 另外，当有新的服务器加入时，NameServer并不会立马通知到Producer，而是由Producer定时去请求NameServer获取最新的Broker/Consumer信息（这种情况是通过Producer发送消息时，负载均衡解决）

**其它**

![img](https://leran2deeplearnjavawebtech.oss-cn-beijing.aliyuncs.com/somephoto/RocketMQ流程.png)

加分项

1. 包括组件通信间使用Netty的自定义协议
2. 消息重试负载均衡策略（具体参考Dubbo负载均衡策略）
3. 消息过滤器（Producer发送消息到Broker，Broker存储消息信息，Consumer消费时请求Broker端从磁盘文件查询消息文件时,在Broker端就使用过滤服务器进行过滤）
4. Broker同步双写和异步双写中Master和Slave的交互
5. Broker在4.5.0版本更新中引入了基于Raft协议的多副本选举，之前这是商业版才有的特性[ISSUE-1046](http://rocketmq.apache.org/release_notes/release-notes-4.5.0/)

#### 相关文章

- [阿里二面：RocketMQ消息积压了，增加消费者有用吗？](https://mp.weixin.qq.com/s/804KWy4Gf9EIo9GiNRg5Fg)
- [如何基于RocketMQ设计一套全链路消息不丢失方案？](https://mp.weixin.qq.com/s/_Gl8MqcSa5gASLXSxyhHCg)
- [通过这三个文件彻底搞懂rocketmq的存储原理](https://mp.weixin.qq.com/s/4k_g85aktaBhwSl9iu6r5A)
- [RocketMQ集群Broker挂了，会造成什么影响？](https://mp.weixin.qq.com/s/vIqOEeNBCSQJDB-qNYukTQ)
- [RocketMQ高级功能代码实现](https://mp.weixin.qq.com/s/WVZRVMUACXuYwuJLjGIUHA)

### RabbitMQ

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/high-performance/message-queue/rabbitmq-logo.png)

#### RabbitMQ基础知识总结

##### 简介

RabbitMQ是采用Erlang语言实现AMQP(AdvancedMessageQueuingProtocol，高级消息队列协议)的消息中间件，它最初起源于金融系统，用于在分布式系统中存储转发消息。

RabbitMQ发展到今天，被越来越多的人认可，这和它在易用性、扩展性、可靠性和高可用性等方面的卓著表现是分不开的。RabbitMQ的具体特点可以概括为以下几点：

- **可靠性：**RabbitMQ使用一些机制来保证消息的可靠性，如持久化、传输确认及发布确认等。
- **灵活的路由：**在消息进入队列之前，通过交换器来路由消息。对于典型的路由功能，RabbitMQ己经提供了一些内置的交换器来实现。针对更复杂的路由功能，可以将多个交换器绑定在一起，也可以通过插件机制来实现自己的交换器。这个后面会在我们讲RabbitMQ核心概念的时候详细介绍到。
- **扩展性：**多个RabbitMQ节点可以组成一个集群，也可以根据实际业务情况动态地扩展集群中节点。
- **高可用性：**队列可以在集群中的机器上设置镜像，使得在部分节点出现问题的情况下队列仍然可用。
- **支持多种协议：**RabbitMQ除了原生支持AMQP协议，还支持STOMP、MQTT等多种消息中间件协议。
- **多语言客户端：**RabbitMQ几乎支持所有常用语言，比如Java、Python、Ruby、PHP、C#、JavaScript等。
- **易用的管理界面：**RabbitMQ提供了一个易用的用户界面，使得用户可以监控和管理消息、集群中的节点等。在安装RabbitMQ的时候会介绍到，安装好RabbitMQ就自带管理界面。
- **插件机制：**RabbitMQ提供了许多插件，以实现从多方面进行扩展，当然也可以编写自己的插件。感觉这个有点类似Dubbo的SPI机制

[RabbitMQ官网](https://www.rabbitmq.com/)

[RabbitMQ更新记录](https://www.rabbitmq.com/news.html)

##### RabbitMQ核心概念

RabbitMQ整体上是一个生产者与消费者模型，主要负责接收、存储和转发消息。可以把消息传递的过程想象成：当你将一个包裹送到邮局，邮局会暂存并最终将邮件通过邮递员送到收件人的手上，RabbitMQ就好比由邮局、邮箱和邮递员组成的一个系统。从计算机术语层面来说，RabbitMQ模型更像是一种交换机模型。

下面再来看看图1——RabbitMQ的整体模型架构。

![图1-RabbitMQ的整体模型架构](https://oss.javaguide.cn/github/javaguide/rabbitmq/96388546.jpg)


**Producer(生产者)和Consumer(消费者)**

- **Producer(生产者)**:生产消息的一方（邮件投递者）
- **Consumer(消费者)**:消费消息的一方（邮件收件人）

消息一般由2部分组成：**消息头**（或者说是标签Label）和**消息体**。消息体也可以称为payLoad,消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。生产者把消息交由RabbitMQ后，RabbitMQ会根据消息头把消息发送给感兴趣的Consumer(消费者)。

**Exchange(交换器)**

在RabbitMQ中，消息并不是直接被投递到**Queue(消息队列)**中的，中间还必须经过**Exchange(交换器)**这一层，**Exchange(交换器)**会把我们的消息分配到对应的**Queue(消息队列)**中。

**Exchange(交换器)**用来接收生产者发送的消息并将这些消息路由给服务器中的队列中，如果路由不到，或许会返回给**Producer(生产者)**，或许会被直接丢弃掉。这里可以将RabbitMQ中的交换器看作一个简单的实体。

**RabbitMQ的Exchange(交换器)有4种类型，不同的类型对应着不同的路由策略**：**direct(默认)**，**fanout**,**topic**,和**headers**，不同类型的Exchange转发消息的策略有所区别。这个会在介绍**ExchangeTypes(交换器类型)**的时候介绍到。

Exchange(交换器)示意图如下：

![Exchange(交换器)示意图](https://oss.javaguide.cn/github/javaguide/rabbitmq/24007899.jpg)

生产者将消息发给交换器的时候，一般会指定一个**RoutingKey(路由键)**，用来指定这个消息的路由规则，而这个**RoutingKey需要与交换器类型和绑定键(BindingKey)联合使用才能最终生效**。

RabbitMQ中通过**Binding(绑定)**将**Exchange(交换器)**与**Queue(消息队列)**关联起来，在绑定的时候一般会指定一个**BindingKey(绑定建)**,这样RabbitMQ就知道如何正确将消息路由到队列了,如下图所示。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。Exchange和Queue的绑定可以是多对多的关系。

Binding(绑定)示意图：

![Binding(绑定)示意图](https://oss.javaguide.cn/github/javaguide/rabbitmq/70553134.jpg)

生产者将消息发送给交换器时，需要一个RoutingKey,当BindingKey和RoutingKey相匹配时，消息会被路由到对应的队列中。在绑定多个队列到同一个交换器的时候，这些绑定允许使用相同的BindingKey。BindingKey并不是在所有的情况下都生效，它依赖于交换器类型，比如fanout类型的交换器就会无视，而是将消息路由到所有绑定到该交换器的队列中。

**Queue(消息队列)**

**Queue(消息队列)**用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

**RabbitMQ**中消息只能存储在**队列**中，这一点和**Kafka**这种消息中间件相反。Kafka将消息存储在**topic（主题）**这个逻辑层面，而相对应的队列逻辑只是topic实际存储文件中的位移标识。RabbitMQ的生产者生产消息并最终投递到队列中，消费者可以从队列中获取消息并消费。

**多个消费者可以订阅同一个队列**，这时队列中的消息会被平均分摊（Round-Robin，即轮询）给多个消费者进行处理，而不是每个消费者都收到所有的消息并处理，这样避免消息被重复消费。

**RabbitMQ**不支持队列层面的广播消费,如果有广播消费的需求，需要在其上进行二次开发,这样会很麻烦，不建议这样做。

**Broker（消息中间件的服务节点）**

对于RabbitMQ来说，一个RabbitMQBroker可以简单地看作一个RabbitMQ服务节点，或者RabbitMQ服务实例。大多数情况下也可以将一个RabbitMQBroker看作一台RabbitMQ服务器。

下图展示了生产者将消息存入RabbitMQBroker,以及消费者从Broker中消费数据的整个流程。

![消息队列的运转过程](https://oss.javaguide.cn/github/javaguide/rabbitmq/67952922.jpg)

**ExchangeTypes(交换器类型)**

RabbitMQ常用的ExchangeType有**fanout**、**direct**、**topic**、**headers**这四种（AMQP规范里还提到两种ExchangeType，分别为system与自定义，这里不予以描述）。

**①fanout**

fanout类型的Exchange路由规则非常简单，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中，不需要做任何判断操作，所以fanout类型是所有的交换机类型里面速度最快的。fanout类型常用来广播消息。

**②direct**

direct类型的Exchange路由规则也很简单，它会把消息路由到那些Bindingkey与RoutingKey完全匹配的Queue中。

![direct类型交换器](https://oss.javaguide.cn/github/javaguide/rabbitmq/37008021.jpg)

以上图为例，如果发送消息的时候设置路由键为“warning”,那么消息会路由到Queue1和Queue2。如果在发送消息的时候设置路由键为"Info”或者"debug”，消息只会路由到Queue2。如果以其他的路由键发送消息，则消息不会路由到这两个队列中。

direct类型常用在处理有优先级的任务，根据任务的优先级把消息发送到对应的队列，这样可以指派更多的资源去处理高优先级的队列。

**③topic**

前面讲到direct类型的交换器路由规则是完全匹配BindingKey和RoutingKey，但是这种严格的匹配方式在很多情况下不能满足实际业务的需求。topic类型的交换器在匹配规则上进行了扩展，它与direct类型的交换器相似，也是将消息路由到BindingKey和RoutingKey相匹配的队列中，但这里的匹配规则有些不同，它约定：

- RoutingKey为一个点号“．”分隔的字符串（被点号“．”分隔开的每一段独立的字符串称为一个单词），如“com.rabbitmq.client”、“java.util.concurrent”、“com.hidden.client”;
- BindingKey和RoutingKey一样也是点号“．”分隔的字符串；
- BindingKey中可以存在两种特殊字符串“*”和“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词(可以是零个)。

![topic类型交换器](https://oss.javaguide.cn/github/javaguide/rabbitmq/73843.jpg)

以上图为例：

- 路由键为“com.rabbitmq.client”的消息会同时路由到Queue1和Queue2;
- 路由键为“com.hidden.client”的消息只会路由到Queue2中；
- 路由键为“com.hidden.demo”的消息只会路由到Queue2中；
- 路由键为“java.rabbitmq.demo”的消息只会路由到Queue1中；
- 路由键为“java.util.concurrent”的消息将会被丢弃或者返回给生产者（需要设置mandatory参数），因为它没有匹配任何路由键。

**④headers(不推荐)**

headers类型的交换器不依赖于路由键的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配。在绑定队列和交换器时指定一组键值对，当发送消息到交换器时，RabbitMQ会获取到该消息的headers(也是一个键值对的形式)，对比其中的键值对是否完全匹配队列和交换器绑定时指定的键值对，如果完全匹配则消息会路由到该队列，否则不会路由到该队列。headers类型的交换器性能会很差，而且也不实用，基本上不会看到它的存在。

##### 安装RabbitMQ

通过Docker安装非常方便，只需要几条命令就好了，我这里是只说一下常规安装方法。

前面提到了RabbitMQ是由Erlang语言编写的，也正因如此，在安装RabbitMQ之前需要安装Erlang。

注意：在安装RabbitMQ的时候需要注意RabbitMQ和Erlang的版本关系，如果不注意的话会导致出错，两者对应关系如下:

![RabbitMQ和Erlang的版本关系](https://oss.javaguide.cn/github/javaguide/rabbitmq/RabbitMQ-Erlang.png)

###### 安装erlang

**1. 下载erlang安装包**

在官网下载然后上传到Linux上或者直接使用下面的命令下载对应的版本。

```bash
[root@SnailClimb local]#wget https://erlang.org/download/otp_src_19.3.tar.gz
```

erlang官网下载：[https://www.erlang.org/downloads](https://www.erlang.org/downloads)

**2. 解压erlang安装包**


```bash
[root@SnailClimb local]#tar -xvzf otp_src_19.3.tar.gz
```

**3. 删除erlang安装包**

```bash
[root@SnailClimb local]#rm -rf otp_src_19.3.tar.gz
```

**4. 安装erlang的依赖工具**

```bash
[root@SnailClimb local]#yum -y install make gcc gcc-c++ kernel-devel m4 ncurses-devel openssl-devel unixODBC-devel
```

**5. 进入erlang安装包解压文件对erlang进行安装环境的配置**

新建一个文件夹


```bash
[root@SnailClimb local]# mkdir erlang
```

对erlang进行安装环境的配置

```bash
[root@SnailClimb otp_src_19.3]# 
./configure --prefix=/usr/local/erlang --without-javac
```

**6. 编译安装**

```bash
[root@SnailClimb otp_src_19.3]# 
make && make install
```

**7. 验证一下erlang是否安装成功了**

```bash
[root@SnailClimb otp_src_19.3]# ./bin/erl
```

运行下面的语句输出“hello world”

```erlang
 io:format("hello world~n", []).
```

![输出“hello world”](https://oss.javaguide.cn/github/javaguide/rabbitmq/49570541.jpg)

大功告成，我们的erlang已经安装完成。

**8. 配置erlang环境变量**

```bash
[root@SnailClimb etc]# vim profile
```

追加下列环境变量到文件末尾

```bash
#erlang
ERL_HOME=/usr/local/erlang
PATH=$ERL_HOME/bin:$PATH
export ERL_HOME PATH
```

运行下列命令使配置文件`profile`生效

```bash
[root@SnailClimb etc]# source /etc/profile
```

输入erl查看erlang环境变量是否配置正确

```bash
[root@SnailClimb etc]# erl
```

![输入erl查看erlang环境变量是否配置正确](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-12-12/62504246.jpg)

###### 安装RabbitMQ

**1. 下载rpm**


```bash
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.8/rabbitmq-server-3.6.8-1.el7.noarch.rpm
```

或者直接在官网下载

[https://www.rabbitmq.com/install-rpm.html](https://www.rabbitmq.com/install-rpm.html)

**2. 安装rpm**

```bash
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
```

紧接着执行：

```bash
yum install rabbitmq-server-3.6.8-1.el7.noarch.rpm
```

中途需要你输入"y"才能继续安装。

**3. 开启web管理插件**

```bash
rabbitmq-plugins enable rabbitmq_management
```

**4. 设置开机启动**

```bash
chkconfig rabbitmq-server on
```

**5. 启动服务**

```bash
service rabbitmq-server start
```

**6. 查看服务状态**

```bash
service rabbitmq-server status
```

**7. 访问RabbitMQ控制台**

浏览器访问：http://你的ip地址:15672/

默认用户名和密码：guest/guest;但是需要注意的是：guest用户只是被容许从localhost访问。官网文档描述如下：

```bash
“guest” user can only connect via localhost
```

**解决远程访问RabbitMQ远程访问密码错误**

新建用户并授权

```bash
[root@SnailClimb rabbitmq]# rabbitmqctl add_user root root
Creating user "root" ...
[root@SnailClimb rabbitmq]# rabbitmqctl set_user_tags root administrator

Setting tags for user "root" to [administrator] ...
[root@SnailClimb rabbitmq]# 
[root@SnailClimb rabbitmq]# rabbitmqctl set_permissions -p / root ".*" ".*" ".*"
Setting permissions for user "root" in vhost "/" ...
```

再次访问:http://你的ip地址:15672/,输入用户名和密码：root root

![RabbitMQ控制台](https://oss.javaguide.cn/github/javaguide/rabbitmq/45835332.jpg)


#### RabbitMQ常见面试题总结

##### RabbitMQ是什么？

RabbitMQ是一个在AMQP（AdvancedMessageQueuingProtocol）基础上实现的，可复用的企业消息系统。它可以用于大型软件系统各个模块之间的高效通信，支持高并发，支持可扩展。它支持多种客户端如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX，持久化，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。

RabbitMQ是使用Erlang编写的一个开源的消息队列，本身支持很多的协议：AMQP，XMPP,SMTP,STOMP，也正是如此，使的它变的非常重量级，更适合于企业级的开发。它同时实现了一个Broker构架，这意味着消息在发送给客户端时先在中心队列排队，对路由(Routing)、负载均衡(Loadbalance)或者数据持久化都有很好的支持。

PS:也可能直接问什么是消息队列？消息队列就是一个使用队列来通信的组件。

##### RabbitMQ特点?

- **可靠性**:RabbitMQ使用一些机制来保证可靠性，如持久化、传输确认及发布确认等。
- **灵活的路由**:在消息进入队列之前，通过交换器来路由消息。对于典型的路由功能，RabbitMQ己经提供了一些内置的交换器来实现。针对更复杂的路由功能，可以将多个交换器绑定在一起，也可以通过插件机制来实现自己的交换器。
- **扩展性**:多个RabbitMQ节点可以组成一个集群，也可以根据实际业务情况动态地扩展集群中节点。
- **高可用性**:队列可以在集群中的机器上设置镜像，使得在部分节点出现问题的情况下队列仍然可用。
- **多种协议**:RabbitMQ除了原生支持AMQP协议，还支持STOMP，MQTT等多种消息中间件协议。
- **多语言客户端**:RabbitMQ几乎支持所有常用语言，比如Java、Python、Ruby、PHP、C#、JavaScript等。
- **管理界面**:RabbitMQ提供了一个易用的用户界面，使得用户可以监控和管理消息、集群中的节点等。
- **插件机制**:RabbitMQ提供了许多插件，以实现从多方面进行扩展，当然也可以编写自己的插件。

##### AMQP是什么?

RabbitMQ就是AMQP协议的Erlang的实现(当然RabbitMQ还支持STOMP2、MQTT3等协议)AMQP的模型架构和RabbitMQ的模型架构是一样的，生产者将消息发送给交换器，交换器和队列绑定。

RabbitMQ中的交换器、交换器类型、队列、绑定、路由键等都是遵循的AMQP协议中相应的概念。目前RabbitMQ最新版本默认支持的是AMQP0-9-1。

**AMQP协议的三层**：

- **ModuleLayer**:协议最高层，主要定义了一些客户端调用的命令，客户端可以用这些命令实现自己的业务逻辑。
- **SessionLayer**:中间层，主要负责客户端命令发送给服务器，再将服务端应答返回客户端，提供可靠性同步机制和错误处理。
- **TransportLayer**:最底层，主要传输二进制数据流，提供帧的处理、信道服用、错误检测和数据表示等。

**AMQP模型的三大组件**：

- **交换器(Exchange)**：消息代理服务器中用于把消息路由到队列的组件。
- **队列(Queue)**：用来存储消息的数据结构，位于硬盘或内存中。
- **绑定(Binding)**：一套规则，告知交换器消息应该将消息投递给哪个队列。

##### 说说生产者Producer和消费者Consumer?

**生产者**:

-消息生产者，就是投递消息的一方。
-消息一般包含两个部分：消息体(`payload`)和标签(`Label`)。

**消费者**：

-消费消息，也就是接收消息的一方。
-消费者连接到RabbitMQ服务器，并订阅到队列上。消费消息时只消费消息体，丢弃标签。

##### 说说Broker服务节点、Queue队列、Exchange交换器？

- **Broker**：可以看做RabbitMQ的服务节点。一般请下一个Broker可以看做一个RabbitMQ服务器。
- **Queue**:RabbitMQ的内部对象，用于存储消息。多个消费者可以订阅同一队列，这时队列中的消息会被平摊（轮询）给多个消费者进行处理。
- **Exchange**:生产者将消息发送到交换器，由交换器将消息路由到一个或者多个队列中。当路由不到时，或返回给生产者或直接丢弃。

##### 什么是死信队列？如何导致的？

DLX，全称为Dead-Letter-Exchange，死信交换器，死信邮箱。当消息在一个队列中变成死信(deadmessage)之后，它能被重新被发送到另一个交换器中，这个交换器就是DLX，绑定DLX的队列就称之为死信队列。

**导致的死信的几种原因**：

- 消息被拒(`Basic.Reject/Basic.Nack`)且`requeue=false`。
- 消息TTL过期。
- 队列满了，无法再添加。

##### 什么是延迟队列？RabbitMQ怎么实现延迟队列？

延迟队列指的是存储对应的延迟消息，消息被发送以后，并不想让消费者立刻拿到消息，而是等待特定时间后，消费者才能拿到这个消息进行消费。

RabbitMQ本身是没有延迟队列的，要实现延迟消息，一般有两种方式：

1. 通过RabbitMQ本身队列的特性来实现，需要使用RabbitMQ的死信交换机（Exchange）和消息的存活时间TTL（TimeToLive）。
2. 在RabbitMQ3.5.7及以上的版本提供了一个插件（rabbitmq-delayed-message-exchange）来实现延迟队列功能。同时，插件依赖Erlang/OPT18.0及以上。

也就是说，AMQP协议以及RabbitMQ本身没有直接支持延迟队列的功能，但是可以通过TTL和DLX模拟出延迟队列的功能。

##### 什么是优先级队列？

RabbitMQ自V3.5.0有优先级队列实现，优先级高的队列会先被消费。

可以通过`x-max-priority`参数来实现优先级队列。不过，当消费速度大于生产速度且Broker没有堆积的情况下，优先级显得没有意义。

##### RabbitMQ有哪些工作模式？

- 简单模式
- work工作模式
- pub/sub发布订阅模式
- Routing路由模式
- Topic主题模式

##### RabbitMQ消息怎么传输？

由于TCP链接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈，所以RabbitMQ使用信道的方式来传输数据。信道（Channel）是生产者、消费者与RabbitMQ通信的渠道，信道是建立在TCP链接上的虚拟链接，且每条TCP链接上的信道数量没有限制。就是说RabbitMQ在一条TCP链接上建立成百上千个信道来达到多个线程处理，这个TCP被多个线程共享，每个信道在RabbitMQ都有唯一的ID，保证了信道私有性，每个信道对应一个线程使用。

##### 如何保证消息的可靠性？

消息到MQ的过程中搞丢，MQ自己搞丢，MQ到消费过程中搞丢。

- 生产者到RabbitMQ：事务机制和Confirm机制，注意：事务机制和Confirm机制是互斥的，两者不能共存，会导致RabbitMQ报错。
- RabbitMQ自身：持久化、集群、普通模式、镜像模式。
- RabbitMQ到消费者：basicAck机制、死信队列、消息补偿机制。

##### 如何保证RabbitMQ消息的顺序性？

- 拆分多个queue(消息队列)，每个queue(消息队列)一个consumer(消费者)，就是多一些queue(消息队列)而已，确实是麻烦点；
- 或者就一个queue(消息队列)但是对应一个consumer(消费者)，然后这个consumer(消费者)内部用内存队列做排队，然后分发给底层不同的worker来处理。

##### 如何保证RabbitMQ高可用的？

RabbitMQ是比较有代表性的，因为是基于主从（非分布式）做高可用性的，我们就以RabbitMQ为例子讲解第一种MQ的高可用性怎么实现。RabbitMQ有三种模式：单机模式、普通集群模式、镜像集群模式。

**单机模式**

Demo级别的，一般就是你本地启动了玩玩儿的?，没人生产用单机模式。

**普通集群模式**

意思就是在多台机器上启动多个RabbitMQ实例，每个机器启动一个。你创建的queue，只会放在一个RabbitMQ实例上，但是每个实例都同步queue的元数据（元数据可以认为是queue的一些配置信息，通过元数据，可以找到queue所在实例）。

你消费的时候，实际上如果连接到了另外一个实例，那么那个实例会从queue所在实例上拉取数据过来。这方案主要是提高吞吐量的，就是说让集群中多个节点来服务某个queue的读写操作。

**镜像集群模式**

这种模式，才是所谓的RabbitMQ的高可用模式。跟普通集群模式不一样的是，在镜像集群模式下，你创建的queue，无论元数据还是queue里的消息都会存在于多个实例上，就是说，每个RabbitMQ节点都有这个queue的一个完整镜像，包含queue的全部数据的意思。然后每次你写消息到queue的时候，都会自动把消息同步到多个实例的queue上。RabbitMQ有很好的管理控制台，就是在后台新增一个策略，这个策略是镜像集群模式的策略，指定的时候是可以要求数据同步到所有节点的，也可以要求同步到指定数量的节点，再次创建queue的时候，应用这个策略，就会自动将数据同步到其他的节点上去了。

这样的好处在于，你任何一个机器宕机了，没事儿，其它机器（节点）还包含了这个queue的完整数据，别的consumer都可以到其它节点上去消费数据。坏处在于，第一，这个性能开销也太大了吧，消息需要同步到所有机器上，导致网络带宽压力和消耗很重！RabbitMQ一个queue的数据都是放在一个节点里的，镜像集群下，也是每个节点都放这个queue的完整数据。

##### 如何解决消息挤压问题？

**临时紧急扩容**。先修复consumer的问题，确保其恢复消费速度，然后将现有consumer都停掉。新建一个topic，partition是原来的10倍，临时建立好原先10倍的queue数量。然后写一个临时的分发数据的consumer程序，这个程序部署上去消费积压的数据，消费之后不做耗时的处理，直接均匀轮询写入临时建立好的10倍数量的queue。接着临时征用10倍的机器来部署consumer，每一批consumer消费一个临时queue的数据。这种做法相当于是临时将queue资源和consumer资源扩大10倍，以正常的10倍速度来消费数据。等快速消费完积压数据之后，得恢复原先部署的架构，重新用原先的consumer机器来消费消息。

##### 如何解决消息队列的延时以及过期失效问题？

RabbtiMQ是可以设置过期时间的，也就是TTL。如果消息在queue中积压超过一定的时间就会被RabbitMQ给清理掉，这个数据就没了。那这就是第二个坑了。这就不是说数据会大量积压在mq里，而是大量的数据会直接搞丢。我们可以采取一个方案，就是批量重导，这个我们之前线上也有类似的场景干过。就是大量积压的时候，我们当时就直接丢弃数据了，然后等过了高峰期以后，比如大家一起喝咖啡熬夜到晚上12点以后，用户都睡觉了。这个时候我们就开始写程序，将丢失的那批数据，写个临时程序，一点一点的查出来，然后重新灌入mq里面去，把白天丢的数据给他补回来。也只能是这样了。假设1万个订单积压在mq里面，没有处理，其中1000个订单都丢了，你只能手动写程序把那1000个订单给查出来，手动发到mq里去再补一次。

#### 相关文章

- [四种策略确保RabbitMQ消息发送可靠性！你用哪种？](https://mp.weixin.qq.com/s/hj8iqASSOk2AgdtkuLPCCQ)
- [新来个技术总监，把RabbitMQ讲的那叫一个透彻，佩服！](https://mp.weixin.qq.com/s/RzxiXBQk3zjHt9PVI3JSQw)
- [RabbitMQ高可用之如何确保消息成功消费](https://mp.weixin.qq.com/s/5szA0KBpFn9G3DeS9C0U3w)
- [RabbitMQ中的消息会过期吗](https://mp.weixin.qq.com/s/fFqzuN2AnLazoCyahL3EFw)
- [RabbitMQ七种消息传递形式](https://mp.weixin.qq.com/s/inhSj-nIiDCfUf24TrE2XA)
- [使用rabbitmq延时队列实现超时取消订单](https://mp.weixin.qq.com/s/V_XZ2vC2jZmzxm-0ThhiTg)
- [面试官：引入RabbitMQ后，你如何保证全链路数据100%不丢失？](https://mp.weixin.qq.com/s/38XaTeHdgiH5GfCIQiWlnQ)
- [RabbitMQ详解](https://mp.weixin.qq.com/s/_G_hd8OX9D76Pd5TmM3_BQ)
- [RabbitMQ的AMQP协议都是些什么内容呢](https://mp.weixin.qq.com/s/JDkIP7Kzi2uWcWoy9k7SQA)


### Pulsar

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/high-performance/message-queue/pulsar-logo.png)

Pulsar是下一代云原生分布式消息流平台，最初由Yahoo开发，已经成为Apache顶级项目。

Pulsar集消息、存储、轻量化函数式计算为一体，采用计算与存储分离架构设计，支持多租户、持久化存储、多机房跨区域数据复制，具有强一致性、高吞吐、低延时及高可扩展性等流数据存储特性，被看作是云原生时代实时消息流传输、存储和计算最佳解决方案。

Pulsar的关键特性如下（摘自官网）：

- 是下一代云原生分布式消息流平台。
- Pulsar的单个实例原生支持多个集群，可跨机房在集群间无缝地完成消息复制。
- 极低的发布延迟和端到端延迟。
- 可无缝扩展到超过一百万个topic。
- 简单的客户端API，支持Java、Go、Python和C++。
- 主题的多种订阅模式（独占、共享和故障转移）。
- 通过ApacheBookKeeper提供的持久化消息存储机制保证消息传递。
- 由轻量级的serverless计算框架PulsarFunctions实现流原生的数据处理。
- 基于PulsarFunctions的serverlessconnector框架PulsarIO使得数据更易移入、移出ApachePulsar。
- 分层式存储可在数据陈旧时，将数据从热存储卸载到冷/长期存储（如S3、GCS）中。

[Pulsar官网](https://pulsar.apache.org/)

[Pulsar更新记录](https://github.com/apache/pulsar/releases)

### ActiveMQ

目前已经被淘汰，不推荐使用，不建议学习。

### 如何选择？

> 参考《Java工程师面试突击第1季-中华石杉老师》

| 对比方向 | 概要                                                         |
| -------- | ------------------------------------------------------------ |
| 吞吐量   | 万级的ActiveMQ和RabbitMQ的吞吐量（ActiveMQ的性能最差）要比十万级甚至是百万级的RocketMQ和Kafka低一个数量级。 |
| 可用性   | 都可以实现高可用。ActiveMQ和RabbitMQ都是基于主从架构实现高可用性。RocketMQ基于分布式架构。Kafka也是分布式的，一个数据多个副本，少数机器宕机，不会丢失数据，不会导致不可用 |
| 时效性   |RabbitMQ基于Erlang开发，所以并发能力很强，性能极其好，延时很低，达到微秒级，其他几个都是ms级。 |
| 功能支持 | Pulsar的功能更全面，支持多租户、多种消费模式和持久性模式等功能，是下一代云原生分布式消息流平台。 |
| 消息丢失 | ActiveMQ和RabbitMQ丢失的可能性非常低，Kafka、RocketMQ和Pulsar理论上可以做到0丢失。 |

**总结：**

- ActiveMQ的社区算是比较成熟，但是较目前来说，ActiveMQ的性能比较差，而且版本迭代很慢，不推荐使用，已经被淘汰了。
- RabbitMQ在吞吐量方面虽然稍逊于Kafka、RocketMQ和Pulsar，但是由于它基于Erlang开发，所以并发能力很强，性能极其好，延时很低，达到微秒级。但是也因为RabbitMQ基于Erlang开发，所以国内很少有公司有实力做Erlang源码级别的研究和定制。如果业务场景对并发量要求不是太高（十万级、百万级），那这几种消息队列中，RabbitMQ或许是你的首选。
- RocketMQ和Pulsar支持强一致性，对消息一致性要求比较高的场景可以使用。
- RocketMQ阿里出品，Java系开源项目，源代码我们可以直接阅读，然后可以定制自己公司的MQ，并且RocketMQ有阿里巴巴的实际业务场景的实战考验。
- Kafka的特点其实很明显，就是仅仅提供较少的核心功能，但是提供超高的吞吐量，ms级的延迟，极高的可用性以及可靠性，而且分布式可以任意扩展。同时Kafka最好是支撑较少的topic数量即可，保证其超高吞吐量。Kafka唯一的一点劣势是有可能消息重复消费，那么对数据准确性会造成极其轻微的影响，在大数据领域中以及日志采集中，这点轻微影响可以忽略这个特性天然适合大数据实时计算以及日志收集。如果是大数据领域的实时计算、日志采集等场景，用Kafka是业内标准的，绝对没问题，社区活跃度很高，绝对不会黄，何况几乎是全世界这个领域的事实性规范。
- [7个方面综合对比Kafka、RabbitMQ、RocketMQ、ActiveMQ四个分布式消息队列](https://mp.weixin.qq.com/s/7PDuuEChls6MDUCiI8_FaA)


## 经典面试题

### 为什么使用消息队列
**解耦**
传统模式下系统间的耦合性太强。怎么说呢，举个例子：系统A通过接口调用发送数据到B、C、D三个系统，如果将来E系统接入或者B系统不需要接入了，那么系统A还需要修改代码，非常麻烦。如果系统A产生了一条比较关键的数据，那么它就要时时刻刻考虑B、C、D、E四个系统如果挂了该咋办？这条数据它们是否都收到了？显然，系统A跟其它系统严重耦合。而如果我们将数据（消息）写入消息队列，需要消息的系统直接自己从消息队列中消费。这样下来，系统A就不需要去考虑要给谁发送数据，不需要去维护这个代码，也不需要考虑其他系统是否调用成功、失败超时等情况，反正我只负责生产，别的我不管。

**异步**
先来看传统同步的情况，举个例子：系统A接收一个用户请求，需要进行写库操作，还需要同样的在B、C、D三个系统中进行写库操作。如果A自己本地写库只要1ms，而B、C、D三个系统写库分别要100ms、200ms、300ms,最终请求总延时是1 + 100 + 200 + 300 = 601ms，用户体验大打折扣。如果使用消息队列，那么系统A就只需要发送3条消息到消息队列中就行了，假如耗时5ms，A系统从接受一个请求到返回响应给用户，总时长是1 + 5 = 6ms，对于用户而言，体验好感度直接拉满。

**消峰**
如果没有使用缓存或者消息队列，那么系统就是直接基于数据库MySQL的，如果有那么一个高峰期，产生了大量的请求涌入MySQL，毫无疑问，系统将会直接崩溃。那如果我们使用消息队列，假设MySQL每秒钟最多处理1k条数据，而高峰期瞬间涌入了5k条数据，不过，这5k条数据涌入了消息队列。这样，我们的系统就可以从消息队列中根据数据库的能力慢慢的来拉取请求，不要超过自己每秒能处理的最大请求数量就行。也就是说消息队列每秒钟5k个请求进来，1k个请求出去，假设高峰期1个小时，那么这段时间就可能有几十万甚至几百万的请求积压在消息队列中。不过这个短暂的高峰期积压是完全可以的，因为高峰期过了之后，每秒钟就没有那么多的请求进入消息队列了，但是数据库依然会按照每秒1k个请求的速度处理。所以只要高峰期一过，系统就会快速的将积压的消息给处理掉。

### MQ怎么保证消息可靠性？MQ怎么保证消息100%消费？如何保证消息不会丢失？
我们需要保障MQ消息的可靠性，需要从三个层面/维度解决：生产者100%投递、MQ持久化、消费者100%消费，这里的100%消费指的是消息不少消费，也不多消费。

**消息生产阶段**：从消息被生产出来，然后提交给MQ的过程中，只要能正常收到MQ Broker的ack确认响应，就表示发送成功，所以只要处理好返回值和异常，这个阶段是不会出现消息丢失的。
**消息存储阶段**：这个阶段一般会直接交给MQ消息中间件来保证，但是你要了解它的原理，比如Broker会做副本，保证一条消息至少同步两个节点再返回ack。
**消息消费阶段**：消费端从Broker上拉取消息，只要消费端在收到消息后，不立即发送消费确认给Broker，而是等到执行完业务逻辑后，再发送消费确认，也能保证消息的不丢失。

**以kafka为例**
一、生产者端

1. Kafka消息发送端有个ACK机制。设置ack参数：ack=0，表示不重试，Kafka不需要返回ack，极有可能各种原因造成丢失；ack=1，表示Leader写入成功就返回ack了，Follower不一定同步成功；ack=all或ack=-1，表示ISR列表中的所有Follower同步完成再返回ack。只要能正常收到MQ Broker的ack确认响应，就表示发送成功，所以只要处理好返回值和异常，这个阶段是不会出现消息丢失的。
2. 设置参数unclean.leader.election.enable:false，禁止选举ISR以外的Follower为Leader，只能从ISR列表中的节点中选举Leader；可能会牺牲Kafka的可用性，但是能够提高消息的可靠性。
3. 重试机制，设置tries>1，表示消息重发次数。
4. 设置最小同步副本数min.insync.replicas>1，没满足该值前，Kafka不提供读写服务，写操作会异常。
总结：生产消息时通过设置最小同步副本数和ACK机制，可以让MQ在性能与可靠性上达到平衡。

二、消费者端
手工提交offset（偏移量）

**以RocketMQ为例**
需要确保生产者、broker、消费者三者都不丢失数据。
1. 生产者不丢失消息
方案1：confirm消息确认机制(同步发送消息) + 失败重试；
方案2：事务消息机制；
2. broker不丢失消息，开启同步刷盘策略 + 主从架构同步机制。
只要让一个master Broker收到消息之后同步写入磁盘，同时同步复制给其他slave Broker，再返回成功响应给生产者，此时就可以保证MQ自己不会弄丢消息
3. 消费者不丢失消息，采用RocketMQ的消费者天然就可以保证你处理完消息之后，才会提交消息的offset到broker去，不过别采用多线程异步处理消息的方式。

### 如何保证mq有序且只消费一次?怎么解决消息被重复消费的问题？

最简单的实现方案，就是在数据库(redis或者关系库)中建一张消息日志表，这个表有两个字段：消息ID和消息执行状态

### 消息积压
增加消费端数量

### MQ高可用的相关文章

- [消息队列消息丢失和消息重复发送的处理策略](https://mp.weixin.qq.com/s/Yu95odAgq7VVzCa5ttkwow)
- [消息队列如何保证消息不丢失，且只被消费一次，这篇就教会你](https://mp.weixin.qq.com/s/wzYvFEowBHB-MJNQU6N3OQ)
- [面试官：如何保障消息100%投递成功、消息幂等性？](https://mp.weixin.qq.com/s/ON4avaV8pOjmJVll3K90RQ)
- [如何保证MQ消息是有序的？](https://mp.weixin.qq.com/s/tNyJhqzOyQFE2fIEygFyTw)
- [消息被重复消费，怎么避免？有什么好的解决方案？](https://mp.weixin.qq.com/s/T3aEpibbP1u48LTat1KxHQ)
- [如何保证消息队列里的数据顺序执行？](https://mp.weixin.qq.com/s/b_2OZkTIViA4IbfsfwwbxA)
- [面试基操：MQ怎么保障消息可靠性？](https://mp.weixin.qq.com/s/F6JsNbLfeo_63kCzcBgogA)
- [使用MQ的时候，怎么确保消息100%不丢失](https://mp.weixin.qq.com/s/yVatpQzVd--dlbR5ImBkWg)



### other

- [什么是MQ？](https://mp.weixin.qq.com/s/mXX2hjRQ7qlPs0wK98Kb4Q)
- [消息队列原理和选型：Kafka、RocketMQ、RabbitMQ和ActiveMQ](https://mp.weixin.qq.com/s/upKoeiT2EvMBBlqyRKiv8g)
- [一篇文章把RabbitMQ、RocketMQ、Kafka三元归一](https://mp.weixin.qq.com/s/pXjLoPkyLNj9AHFoHI-GdA)
- [这个队列的思路真的好，现在它是我简历上的亮点了。](https://mp.weixin.qq.com/s/mtEdb-vJGF7irOJ26QbnvA)
- [面试官必问的3道MQ面试题，还有谁不会？？](https://mp.weixin.qq.com/s/8-zvNtXlad0pACg7Yhh4TA)
