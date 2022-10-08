---
title: 消息队列-MQ
sticky: 54
categories: Java
tags: mq
index_img: /assert/mq.jpg
---

#### MQ怎么保证消息可靠性？MQ怎么保证消息100%消费？如何保证消息不会丢失？
我们需要保障MQ消息的可靠性，需要从三个层面/维度解决：生产者100%投递、MQ持久化、消费者100%消费，这里的100%消费指的是消息不少消费，也不多消费。

消息生产阶段： 从消息被生产出来，然后提交给 MQ 的过程中，只要能正常收到 MQ Broker 的 ack 确认响应，就表示发送成功，所以只要处理好返回值和异常，这个阶段是不会出现消息丢失的。
消息存储阶段： 这个阶段一般会直接交给 MQ 消息中间件来保证，但是你要了解它的原理，比如 Broker 会做副本，保证一条消息至少同步两个节点再返回 ack。
消息消费阶段： 消费端从 Broker 上拉取消息，只要消费端在收到消息后，不立即发送消费确认给 Broker，而是等到执行完业务逻辑后，再发送消费确认，也能保证消息的不丢失。

**以kafka为例**
一、生产者端
1. Kafka消息发送端有个ACK机制。设置ack参数：ack=0，表示不重试，Kafka不需要返回ack，极有可能各种原因造成丢失；ack=1，表示Leader写入成功就返回ack了，Follower不一定同步成功；ack=all或ack=-1，表示ISR列表中的所有Follower同步完成再返回ack。只要能正常收到 MQ Broker 的 ack 确认响应，就表示发送成功，所以只要处理好返回值和异常，这个阶段是不会出现消息丢失的。
2. 设置参数unclean.leader.election.enable: false，禁止选举ISR以外的Follower为Leader，只能从ISR列表中的节点中选举Leader；可能会牺牲Kafka的可用性，但是能够提高消息的可靠性。
3. 重试机制，设置tries > 1，表示消息重发次数。
4. 设置最小同步副本数min.insync.replicas > 1，没满足该值前，Kafka不提供读写服务，写操作会异常。
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

#### 如何保证mq有序且只消费一次，怎么解决消息被重复消费的问题？

最简单的实现方案，就是在数据库(redis或者关系库)中建一张消息日志表， 这个表有两个字段：消息 ID 和消息执行状态

#### 消息积压
增加消费端数量

#### MQ高可用的相关文章

- [消息队列消息丢失和消息重复发送的处理策略](https://mp.weixin.qq.com/s/Yu95odAgq7VVzCa5ttkwow)
- [消息队列如何保证消息不丢失，且只被消费一次，这篇就教会你](https://mp.weixin.qq.com/s/wzYvFEowBHB-MJNQU6N3OQ)
- [面试官：如何保障消息100%投递成功、消息幂等性？](https://mp.weixin.qq.com/s/ON4avaV8pOjmJVll3K90RQ)
- [如何保证 MQ消息是有序的？](https://mp.weixin.qq.com/s/tNyJhqzOyQFE2fIEygFyTw)
- [消息被重复消费，怎么避免？有什么好的解决方案？](https://mp.weixin.qq.com/s/T3aEpibbP1u48LTat1KxHQ)
- [如何保证消息队列里的数据顺序执行？](https://mp.weixin.qq.com/s/b_2OZkTIViA4IbfsfwwbxA)
- [面试基操：MQ怎么保障消息可靠性？](https://mp.weixin.qq.com/s/F6JsNbLfeo_63kCzcBgogA)
- [使用MQ的时候，怎么确保消息100%不丢失](https://mp.weixin.qq.com/s/yVatpQzVd--dlbR5ImBkWg)


#### Kafka

- [Kafka 安装及快速入门](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493725&idx=1&sn=96eb8c16aa8cfe8336334d93c4034ed2&source=41#wechat_redirect)
- [带你涨姿势的认识一下kafka](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491187&idx=1&sn=f0f092e675677d87ea0675f65bb1eb14&source=41#wechat_redirect)
- [Kafka Shell基本命令](https://www.cnblogs.com/xiaodf/p/6093261.html)
- [漫画：图解 Kafka，看本篇就足够啦](https://mp.weixin.qq.com/s/FP3z5JMRB9Oo5pr22kr3XA)
- [你能说出 Kafka 这些原理吗？](https://mp.weixin.qq.com/s/lXds_G5LLs7CArMLyTAF6g)
- [Kafka基本架构及原理](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493937&idx=1&sn=3b2710a62cbb51b3f7c607bfd97664c1&source=41#wechat_redirect)
- [9 个 Kafka 面试题，你会几个？](https://mp.weixin.qq.com/s/LAXrK6WT1bv7I63BmvpXQg)
- [Kafka中副本机制的设计和原理](https://mp.weixin.qq.com/s/yIPIABpAzaHJvGoJ6pv0kg)
- [从面试角度一文学完 Kafka](https://mp.weixin.qq.com/s/Ndwrlst-aVdUpcNz-inM3Q)
- [支持百万级TPS，Kafka是怎么做到的？](https://mp.weixin.qq.com/s/Fvd_RcdMHzcwm3fBlpT8vw)
- [Kafka深度剖析](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494362&idx=1&sn=f16c4f215dd83c81957b0b431b147902&source=41#wechat_redirect)
- [Kafka性能篇：为何Kafka这么"快"](https://mp.weixin.qq.com/s/2wGCEWisbynCIjRsO2gxDg)
- [Kafka为什么吞吐量大、速度快？](https://mp.weixin.qq.com/s/fOhlF6crYGXRNHjV0nLQLQ)
- [图解 kafka 架构与工作原理](https://mp.weixin.qq.com/s/V3BlPGOuGPnmT36a500o7g)
- [面试官：聊一聊 kafka 线上会遇到哪些问题？](https://mp.weixin.qq.com/s/cmDqGpTiHKRqcSaQVesJVA)
- [《面试八股文》之 Kafka 21卷](https://mp.weixin.qq.com/s/l75MIcG4cf8C59z1JcfygA)


#### RabbitMQ

- [四种策略确保 RabbitMQ 消息发送可靠性！你用哪种？](https://mp.weixin.qq.com/s/hj8iqASSOk2AgdtkuLPCCQ)
- [新来个技术总监，把 RabbitMQ 讲的那叫一个透彻，佩服！](https://mp.weixin.qq.com/s/RzxiXBQk3zjHt9PVI3JSQw)
- [RabbitMQ 高可用之如何确保消息成功消费](https://mp.weixin.qq.com/s/5szA0KBpFn9G3DeS9C0U3w)
- [RabbitMQ 中的消息会过期吗](https://mp.weixin.qq.com/s/fFqzuN2AnLazoCyahL3EFw)
- [RabbitMQ 七种消息传递形式](https://mp.weixin.qq.com/s/inhSj-nIiDCfUf24TrE2XA)
- [使用rabbitmq延时队列实现超时取消订单](https://mp.weixin.qq.com/s/V_XZ2vC2jZmzxm-0ThhiTg)
- [面试官：引入RabbitMQ后，你如何保证全链路数据100%不丢失？](https://mp.weixin.qq.com/s/38XaTeHdgiH5GfCIQiWlnQ)
- [RabbitMQ详解](https://mp.weixin.qq.com/s/_G_hd8OX9D76Pd5TmM3_BQ)

#### RocketMQ

- [阿里二面：RocketMQ 消息积压了，增加消费者有用吗？](https://mp.weixin.qq.com/s/804KWy4Gf9EIo9GiNRg5Fg)
- [如何基于RocketMQ设计一套全链路消息不丢失方案？](https://mp.weixin.qq.com/s/_Gl8MqcSa5gASLXSxyhHCg)
- [通过这三个文件彻底搞懂rocketmq的存储原理](https://mp.weixin.qq.com/s/4k_g85aktaBhwSl9iu6r5A)
- [RocketMQ 集群 Broker 挂了，会造成什么影响？](https://mp.weixin.qq.com/s/vIqOEeNBCSQJDB-qNYukTQ)

#### other

- [什么是MQ？](https://mp.weixin.qq.com/s/mXX2hjRQ7qlPs0wK98Kb4Q)
- [消息队列原理和选型：Kafka、RocketMQ 、RabbitMQ 和 ActiveMQ](https://mp.weixin.qq.com/s/upKoeiT2EvMBBlqyRKiv8g)
- [一篇文章把RabbitMQ、RocketMQ、Kafka三元归一](https://mp.weixin.qq.com/s/pXjLoPkyLNj9AHFoHI-GdA)
- [这个队列的思路真的好，现在它是我简历上的亮点了。](https://mp.weixin.qq.com/s/mtEdb-vJGF7irOJ26QbnvA)
- [面试官必问的 3 道 MQ 面试题，还有谁不会？？](https://mp.weixin.qq.com/s/8-zvNtXlad0pACg7Yhh4TA)
