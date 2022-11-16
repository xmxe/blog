---
title: 消息队列-MQ
categories: Java
index_img: /assert/mq.jpg
img: https://img1.baidu.com/it/u=1369410832,3000356447&fm=253&fmt=auto&app=138&f=PNG?w=500&h=539
---
#### 经典面试题

##### 为什么使用消息队列
**解耦**
传统模式下系统间的耦合性太强。怎么说呢，举个例子：系统A通过接口调用发送数据到B、C、D三个系统，如果将来E系统接入或者B系统不需要接入了，那么系统A还需要修改代码，非常麻烦。如果系统A产生了一条比较关键的数据，那么它就要时时刻刻考虑B、C、D、E四个系统如果挂了该咋办？这条数据它们是否都收到了？显然，系统A跟其它系统严重耦合。而如果我们将数据（消息）写入消息队列，需要消息的系统直接自己从消息队列中消费。这样下来，系统A就不需要去考虑要给谁发送数据，不需要去维护这个代码，也不需要考虑其他系统是否调用成功、失败超时等情况，反正我只负责生产，别的我不管。

**异步**
先来看传统同步的情况，举个例子：系统A接收一个用户请求，需要进行写库操作，还需要同样的在B、C、D三个系统中进行写库操作。如果A自己本地写库只要1ms，而B、C、D三个系统写库分别要100ms、200ms、300ms,最终请求总延时是1 + 100 + 200 + 300 = 601ms，用户体验大打折扣。如果使用消息队列，那么系统A就只需要发送3条消息到消息队列中就行了，假如耗时5ms，A系统从接受一个请求到返回响应给用户，总时长是1 + 5 = 6ms，对于用户而言，体验好感度直接拉满。

**消峰**
如果没有使用缓存或者消息队列，那么系统就是直接基于数据库MySQL的，如果有那么一个高峰期，产生了大量的请求涌入MySQL，毫无疑问，系统将会直接崩溃。那如果我们使用消息队列，假设MySQL每秒钟最多处理1k条数据，而高峰期瞬间涌入了5k条数据，不过，这5k条数据涌入了消息队列。这样，我们的系统就可以从消息队列中根据数据库的能力慢慢的来拉取请求，不要超过自己每秒能处理的最大请求数量就行。也就是说消息队列每秒钟5k个请求进来，1k个请求出去，假设高峰期1个小时，那么这段时间就可能有几十万甚至几百万的请求积压在消息队列中。不过这个短暂的高峰期积压是完全可以的，因为高峰期过了之后，每秒钟就没有那么多的请求进入消息队列了，但是数据库依然会按照每秒1k个请求的速度处理。所以只要高峰期一过，系统就会快速的将积压的消息给处理掉。

##### MQ怎么保证消息可靠性？MQ怎么保证消息100%消费？如何保证消息不会丢失？
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

##### 如何保证mq有序且只消费一次?怎么解决消息被重复消费的问题？

最简单的实现方案，就是在数据库(redis或者关系库)中建一张消息日志表，这个表有两个字段：消息ID和消息执行状态

##### 消息积压
增加消费端数量

#### MQ高可用的相关文章

- [消息队列消息丢失和消息重复发送的处理策略](https://mp.weixin.qq.com/s/Yu95odAgq7VVzCa5ttkwow)
- [消息队列如何保证消息不丢失，且只被消费一次，这篇就教会你](https://mp.weixin.qq.com/s/wzYvFEowBHB-MJNQU6N3OQ)
- [面试官：如何保障消息100%投递成功、消息幂等性？](https://mp.weixin.qq.com/s/ON4avaV8pOjmJVll3K90RQ)
- [如何保证MQ消息是有序的？](https://mp.weixin.qq.com/s/tNyJhqzOyQFE2fIEygFyTw)
- [消息被重复消费，怎么避免？有什么好的解决方案？](https://mp.weixin.qq.com/s/T3aEpibbP1u48LTat1KxHQ)
- [如何保证消息队列里的数据顺序执行？](https://mp.weixin.qq.com/s/b_2OZkTIViA4IbfsfwwbxA)
- [面试基操：MQ怎么保障消息可靠性？](https://mp.weixin.qq.com/s/F6JsNbLfeo_63kCzcBgogA)
- [使用MQ的时候，怎么确保消息100%不丢失](https://mp.weixin.qq.com/s/yVatpQzVd--dlbR5ImBkWg)


#### Kafka

**kafka配置**
```
consumer
 /*
 * 1.group.id 消费者所属消费组的唯一标识
 * 2.max.poll.records 一次拉取请求的最大消息数，默认500条
 * 3.max.poll.interval.ms 指定拉取消息线程最长空闲时间，默认300000ms
 * 4.session.timeout.ms 检测消费者是否失效的超时时间，默认10000ms
 * 5.heartbeat.interval.ms 消费者心跳时间，默认3000ms
 * 6.bootstrap.servers 连接集群broker地址
 * 7.enable.auto.commit 是否开启自动提交消费位移的功能，默认true
 * 8.auto.commit.interval.ms 自动提交消费位移的时间间隔，默认5000ms
 * 9.partition.assignment.strategy 消费者的分区配置策略,默认RangeAssignor
 * 10.auto.offset.reset 
 * 如果分区没有初始偏移量，或者当前偏移量服务器上不存在时，将使用的偏移量设置，earliest从头开始消费，latest从最近的开始消费，
 * none抛出异常
 * 如果存在已经提交的offest时,不管设置为earliest或者latest都会从已经提交的offest处开始消费
 * 如果不存在已经提交的offest时,earliest表示从头开始消费,latest表示从最新的数据消费,也就是新产生的数据.
 * none topic各分区都存在已提交的offset时，从提交的offest处开始消费；只要有一个分区不存在已提交的offset，则抛出异常
 * kafka-0.10.1.X版本之前: auto.offset.reset的值为smallest,和,largest.(offest保存在zk中)
 * kafka-0.10.1.X版本之后: auto.offset.reset的值更改为:earliest,latest,和none(offest保存在kafka的一个特殊的topic
 * 名为:__consumer_offsets里面)
 * 11.fetch.min.bytes 消费者客户端一次请求从Kafka拉取消息的最小数据量，如果Kafka返回的数据量小于该值，会一直等待，直到满足这个配置大小，默认1b
 * 12.fetch.max.bytes 消费者客户端一次请求从Kafka拉取消息的最大数据量，默认50MB
 * 13.fetch.max.wait.ms 从Kafka拉取消息时，在不满足fetch.min.bytes条件时，等待的最大时间，默认500ms
 * 14.metadata.max.age.ms 强制刷新元数据时间，毫秒，默认300000，5分钟
 * 15.max.partition.fetch.bytes 设置从每个分区里返回给消费者的最大数据量，区别于fetch.max.bytes，默认1MB
 * 16.send.buffer.bytes Socket发送缓冲区大小，默认128kb,-1将使用操作系统的设置
 * 17.receive.buffer.bytes Socket发送缓冲区大小，默认64kb,-1将使用操作系统的设置
 * 18.client.id 消费者客户端的id
 * 19.reconnect.backoff.ms 连接失败后，尝试连接Kafka的时间间隔，默认50ms
 * 20.reconnect.backoff.max.ms 尝试连接到Kafka，生产者客户端等待的最大时间，默认1000ms
 * 21.retry.backoff.ms 消息发送失败重试时间间隔，默认100ms
 * 22.metrics.sample.window.ms 样本计算时间窗口，默认30000ms
 * 23.metrics.num.samples 用于维护metrics的样本数量，默认2
 * 24.metrics.log.level metrics日志记录级别，默认info
 * 25.metric.reporters 类的列表，用于衡量指标，默认空list
 * 26.check.crcs 自动检查CRC32记录的消耗
 * 27.key.deserializer key反序列化方式
 * 28.value.deserializer value反序列化方式
 * 29.connections.max.idle.ms 设置多久之后关闭空闲连接，默认540000ms
 * 30.request.timeout.ms 客户端将等待请求的响应的最大时间,如果在这个时间内没有收到响应，客户端将重发请求，超过重试次数将抛异常，默认30000ms
 * 31.default.api.timeout.ms 设置消费者api超时时间，默认60000ms
 * 32.interceptor.classes 自定义拦截器
 * 33.exclude.internal.topics 内部的主题:一consumer_offsets和一transaction_state。该参数用来指定Kafka中的内部主题是否可以向消费者公开，默认值为true。如果设置为true，那么只能使用subscribe(Collection)的方式而不能使用subscribe(Pattern)的方式来订阅内部主题，设置为false则没有这个限制。
 * 34.isolation.level 用来配置消费者的事务隔离级别。如果设置为“read committed”，那么消费者就会忽略事务未提交的消息，即只能消费到LSO(LastStableOffset)的位置，默认情况下为“read_uncommitted”，即可以消费到HW(High Watermark)处的位置
 * key.deserializer = org.apache.kafka.common.serialization.StringDeserializer
 * value.deserializer = org.apache.kafka.common.serialization.StringDeserializer
 */
```

```
producer
/*
* # Snappy压缩技术是Google开发的，它可以在提供较好的压缩比的同时，减少对CPU的使用率并保证好的性能，所以建议在同时考虑性能和带宽的情况下使用。
* # Gzip压缩技术通常会使用更多的CPU和时间，但会产生更好的压缩比，所以建议在网络带宽更受限制的情况下使用
* # 默认不压缩，该参数可以设置成snappy、gzip或lz4对发送给broker的消息进行压缩
* compression.type=Gzip
* # 请求的最大字节数。这也是对最大消息大小的有效限制。注意：server具有自己对消息大小的限制，这些大小和这个设置不同。此项设置将会限制producer每次批量发送请求的数目，以防发出巨量的请求。
* max.request.size=1048576
* # TCP的接收缓存 SO_RCVBUF 空间大小，用于读取数据
* receive.buffer.bytes=32768
* # client等待请求响应的最大时间,如果在这个时间内没有收到响应，客户端将重发请求，超过重试次数发送失败
* request.timeout.ms=30000
* # TCP的发送缓存 SO_SNDBUF 空间大小，用于发送数据
* send.buffer.bytes=131072
* # 指定server等待来自followers的确认的最大时间，根据acks的设置，超时则返回error
* timeout.ms=30000
* # 在block前一个connection上允许最大未确认的requests数量。
* # 当设为1时，即是消息保证有序模式，注意：这里的消息保证有序是指对于单个Partition的消息有顺序，因此若要保证全局消息有序，可以只使用一个Partition，当然也会降低性能
* max.in.flight.requests.per.connection=5
* # 在第一次将数据发送到某topic时，需先fetch该topic的metadata，得知哪些服务器持有该topic的partition，该值为最长获取metadata时间
* metadata.fetch.timeout.ms=60000
* # 连接失败时，当我们重新连接时的等待时间
* reconnect.backoff.ms=50
* # 在重试发送失败的request前的等待时间，防止若目的Broker完全挂掉的情况下Producer一直陷入死循环发送，折中的方法
* retry.backoff.ms=100
* # metrics系统维护可配置的样本数量，在一个可修正的window size
* metrics.sample.window.ms=30000
* # 用于维护metrics的样本数
* metrics.num.samples=2
* # 类的列表，用于衡量指标。实现MetricReporter接口
* metric.reporters=[]
* # 强制刷新metadata的周期，即使leader没有变化
* metadata.max.age.ms=300000
* # 与broker会话协议，取值：LAINTEXT,SSL,SASL_PLAINTEXT,SASL_SSL
* security.protocol=PLAINTEXT
* # 分区类，实现Partitioner接口
* partitioner.class=class org.apache.kafka.clients.producer.internals.DefaultPartitioner
* # 控制block的时长，当buffer空间不够或者metadata丢失时产生block
* max.block.ms=60000
* # 关闭达到该时间的空闲连接
* connections.max.idle.ms=540000
* # 当向server发出请求时，这个字符串会发送给server，目的是能够追踪请求源
* client.id=""
* # acks=0 配置适用于实现非常高的吞吐量,acks=all这是最安全的模式。Server完成producer request前需要确认的数量。
* # acks=0时，producer不会等待确认，直接添加到socket等待发送；acks=1时，等待leader写到local log就行；acks=all或acks=-1
* # 时，等待isr中所有副本确认
* #（注意：确认都是broker接收到消息放入内存就直接返回确认，不是需要等待数据写入磁盘后才返回确认，这也是kafka快的原因）
* acks = all
* # 发生错误时，重传次数。当开启重传时，需要将`max.in.flight.requests.per.connection`设置为1，否则可能导致失序
* retries = 0
* # 发送到同一个partition的消息会被先存储在batch中，该参数指定一个batch可以使用的内存大小，单位是byte。不一定需要等到batch
* # 被填满才能发送Producer可以将发往同一个Partition的数据做成一个Produce Request发送请求，即Batch批处理，以减少请求次数，
* # 该值即为每次批处理的大小。另外每个Request请求包含多个Batch，每个Batch对应一个Partition，且一个Request发送的目的Broker
* # 均为这些partition的leader副本。若将该值设为0，则不会进行批处理
* batch.size = 16384
* # 生产者在发送消息前等待linger.ms，从而等待更多的消息加入到batch中。如果batch被填满或者linger.ms达到上限，就把batch中的消* # 息发送出去,Producer默认会把两次发送时间间隔内收集到的所有Requests进行一次聚合然后再发送，以此提高吞吐量，而linger.ms则更
* # 进一步，这个参数为每次发送增加一些delay，以此来聚合更多的Message。官网解释翻译：producer会将request传输之间到达的所有
* # records聚合到一个批请求。通常这个值发生在欠负载情况下，record到达速度快于发送。但是在某些场景下，client即使在正常负载下也
* # 期望减少请求数量。这个设置就是如此，通过人工添加少量时延，而不是立马发送一个record,producer会等待所给的时延，以让其他
* # records发送出去，这样就会被聚合在一起。这个类似于TCP的Nagle算法。该设置给了batch的时延上限：当我们获得一个partition的
* # batch.size大小的records，就会立即发送出去，而不管该设置；但是如果对于这个partition没有累积到足够的record，会linger指定
* # 的时间等待更多的records出现。该设置的默认值为0(无时延)。例如，设置linger.ms=5，会减少request发送的数量，但是在无负载下会
* # 增加5ms的发送时延。
* linger.ms = 1
* # Producer可以用来缓存数据的内存大小。该值实际为RecordAccumulator类中的BufferPool，即Producer所管理的最大内存。如果数据
* # 产生速度大于向broker发送的速度，producer会阻塞max.block.ms，超时则抛出异常
* buffer.memory = 33554432
* key.serializer = org.apache.kafka.common.serialization.StringSerializer
* value.serializer = org.apache.kafka.common.serialization.StringSerializer
*/
```

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


#### RabbitMQ

- [四种策略确保RabbitMQ消息发送可靠性！你用哪种？](https://mp.weixin.qq.com/s/hj8iqASSOk2AgdtkuLPCCQ)
- [新来个技术总监，把RabbitMQ讲的那叫一个透彻，佩服！](https://mp.weixin.qq.com/s/RzxiXBQk3zjHt9PVI3JSQw)
- [RabbitMQ高可用之如何确保消息成功消费](https://mp.weixin.qq.com/s/5szA0KBpFn9G3DeS9C0U3w)
- [RabbitMQ中的消息会过期吗](https://mp.weixin.qq.com/s/fFqzuN2AnLazoCyahL3EFw)
- [RabbitMQ七种消息传递形式](https://mp.weixin.qq.com/s/inhSj-nIiDCfUf24TrE2XA)
- [使用rabbitmq延时队列实现超时取消订单](https://mp.weixin.qq.com/s/V_XZ2vC2jZmzxm-0ThhiTg)
- [面试官：引入RabbitMQ后，你如何保证全链路数据100%不丢失？](https://mp.weixin.qq.com/s/38XaTeHdgiH5GfCIQiWlnQ)
- [RabbitMQ详解](https://mp.weixin.qq.com/s/_G_hd8OX9D76Pd5TmM3_BQ)
- [RabbitMQ的AMQP协议都是些什么内容呢](https://mp.weixin.qq.com/s/JDkIP7Kzi2uWcWoy9k7SQA)

#### RocketMQ

- [阿里二面：RocketMQ消息积压了，增加消费者有用吗？](https://mp.weixin.qq.com/s/804KWy4Gf9EIo9GiNRg5Fg)
- [如何基于RocketMQ设计一套全链路消息不丢失方案？](https://mp.weixin.qq.com/s/_Gl8MqcSa5gASLXSxyhHCg)
- [通过这三个文件彻底搞懂rocketmq的存储原理](https://mp.weixin.qq.com/s/4k_g85aktaBhwSl9iu6r5A)
- [RocketMQ集群Broker挂了，会造成什么影响？](https://mp.weixin.qq.com/s/vIqOEeNBCSQJDB-qNYukTQ)

#### other

- [什么是MQ？](https://mp.weixin.qq.com/s/mXX2hjRQ7qlPs0wK98Kb4Q)
- [消息队列原理和选型：Kafka、RocketMQ、RabbitMQ和ActiveMQ](https://mp.weixin.qq.com/s/upKoeiT2EvMBBlqyRKiv8g)
- [一篇文章把RabbitMQ、RocketMQ、Kafka三元归一](https://mp.weixin.qq.com/s/pXjLoPkyLNj9AHFoHI-GdA)
- [这个队列的思路真的好，现在它是我简历上的亮点了。](https://mp.weixin.qq.com/s/mtEdb-vJGF7irOJ26QbnvA)
- [面试官必问的3道MQ面试题，还有谁不会？？](https://mp.weixin.qq.com/s/8-zvNtXlad0pACg7Yhh4TA)
