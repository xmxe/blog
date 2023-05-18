---
title: 分布式ID
img: https://pic1.zhimg.com/v2-a813d6ce08aea23e86f1f269a1760810_1440w.jpg
categories: Java

---


## 分布式ID介绍

### 什么是ID？

日常开发中，我们需要对系统中的各种数据使用ID唯一表示，比如用户ID对应且仅对应一个人，商品ID对应且仅对应一件商品，订单ID对应且仅对应一个订单。

我们现实生活中也有各种ID，比如身份证ID对应且仅对应一个人、地址ID对应且仅对应

简单来说，**ID就是数据的唯一标识**。

### 什么是分布式ID？

分布式ID是分布式系统下的ID。分布式ID不存在与现实生活中，属于计算机系统中的一个概念。

我简单举一个分库分表的例子。

我司的一个项目，使用的是单机MySQL。但是，没想到的是，项目上线一个月之后，随着使用人数越来越多，整个系统的数据量将越来越大。单机MySQL已经没办法支撑了，需要进行分库分表（推荐Sharding-JDBC）。

在分库之后，数据遍布在不同服务器上的数据库，数据库的自增主键已经没办法满足生成的主键唯一了。**我们如何为不同的数据节点生成全局唯一主键呢？**

这个时候就需要生成**分布式ID**了。

![img](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/id-after-the-sub-table-not-conflict.png)

### 分布式ID需要满足哪些要求?

![img](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/distributed-id-requirements.png)

分布式ID作为分布式系统中必不可少的一环，很多地方都要用到分布式ID。

一个最基本的分布式ID需要满足下面这些要求：

- **全局唯一**：ID的全局唯一性肯定是首先要满足的！
- **高性能**：分布式ID的生成速度要快，对本地资源消耗要小。
- **高可用**：生成分布式ID的服务要保证可用性无限接近于100%。
- **方便易用**：拿来即用，使用方便，快速接入！

除了这些之外，一个比较好的分布式ID还应保证：

- **安全**：ID中不包含敏感信息。
- **有序递增**：如果要把ID存放在数据库的话，ID的有序性可以提升数据库写入速度。并且，很多时候，我们还很有可能会直接通过ID来进行排序。
- **有具体的业务含义**：生成的ID如果能有具体的业务含义，可以让定位问题以及开发更透明化（通过ID就能确定是哪个业务）。
- **独立部署**：也就是分布式系统单独有一个发号器服务，专门用来生成分布式ID。这样就生成ID的服务可以和业务相关的服务解耦。不过，这样同样带来了网络调用消耗增加的问题。总的来说，如果需要用到分布式ID的场景比较多的话，独立部署的发号器服务还是很有必要的。

## 分布式ID常见解决方案

### 数据库

#### 数据库主键自增

这种方式就比较简单直白了，就是通过关系型数据库的自增主键产生来唯一的ID。

![数据库主键自增](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/the-primary-key-of-the-database-increases-automatically.png)

以MySQL举例，我们通过下面的方式即可。

**1.创建一个数据库表。**

```sql
CREATE TABLE sequence_id (
  id bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  stub char(10) NOT NULL DEFAULT '',
  PRIMARY KEY (id),
  UNIQUE KEY stub (stub)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

stub字段无意义，只是为了占位，便于我们插入或者修改数据。并且，给stub字段创建了唯一索引，保证其唯一性。

**2.通过replace into来插入数据。**

```java
BEGIN;
REPLACE INTO sequence_id (stub) VALUES ('stub');
SELECT LAST_INSERT_ID();
COMMIT;
```

插入数据这里，我们没有使用insert into而是使用replace into来插入数据，具体步骤是这样的：

1. 第一步：尝试把数据插入到表中。

2. 第二步：如果主键或唯一索引字段出现重复数据错误而插入失败时，先从表中删除含有重复关键字值的冲突行，然后再次尝试把数据插入到表中。

这种方式的优缺点也比较明显：

- **优点**：实现起来比较简单、ID有序递增、存储消耗空间小
- **缺点**：支持的并发量不大、存在数据库单点问题（可以使用数据库集群解决，不过增加了复杂度）、ID没有具体业务含义、安全问题（比如根据订单ID的递增规律就能推算出每天的订单量，商业机密啊！）、每次获取ID都要访问一次数据库（增加了对数据库的压力，获取速度也慢）

#### 数据库号段模式

数据库主键自增这种模式，每次获取ID都要访问一次数据库，ID需求比较大的时候，肯定是不行的。

如果我们可以批量获取，然后存在在内存里面，需要用到的时候，直接从内存里面拿就舒服了！这也就是我们说的**基于数据库的号段模式来生成分布式ID。**

数据库的号段模式也是目前比较主流的一种分布式ID生成方式。像滴滴开源的[Tinyid](https://github.com/didi/tinyid/wiki/tinyid原理介绍)就是基于这种方式来做的。不过，TinyId使用了双号段缓存、增加多db支持等方式来进一步优化。

以MySQL举例，我们通过下面的方式即可。

**1.创建一个数据库表。**

```sql
CREATE TABLE sequence_id_generator (
  id int(10) NOT NULL,
  current_max_id bigint(20) NOT NULL COMMENT '当前最大id',
  step int(10) NOT NULL COMMENT '号段的长度',
  version int(20) NOT NULL COMMENT '版本号',
  biz_type int(20) NOT NULL COMMENT '业务类型',
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

current_max_id字段和step字段主要用于获取批量ID，获取的批量id为：current_max_id~current_max_id+step。

![数据库号段模式](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/database-number-segment-mode.png)

version字段主要用于解决并发问题（乐观锁）,biz_type主要用于表示业务类型。

**2.先插入一行数据。**

```sql
INSERT INTO sequence_id_generator (id, current_max_id, step, version, biz_type)
VALUES(1, 0, 100, 0, 101);
```

**3.通过SELECT获取指定业务下的批量唯一ID**

```sql
SELECT current_max_id, step,version FROM sequence_id_generator where biz_type = 101
```

结果：

```text
id	current_max_id	step	version	biz_type
1	0	100	0	101
```

**4.不够用的话，更新之后重新SELECT即可。**

```sql
UPDATE sequence_id_generator SET current_max_id = 0+100, version=version+1 WHERE version = 0  AND biz_type = 101
SELECT current_max_id, step,version FROM sequence_id_generator where biz_type = 101
```

结果：

```text
id	current_max_id	step	version	biz_type
1	100	100	1	101
```

相比于数据库主键自增的方式，**数据库的号段模式对于数据库的访问次数更少，数据库压力更小。**

另外，为了避免单点问题，你可以从使用主从模式来提高可用性。

**数据库号段模式的优缺点:**

- **优点**：ID有序递增、存储消耗空间小
- **缺点**：存在数据库单点问题（可以使用数据库集群解决，不过增加了复杂度）、ID没有具体业务含义、安全问题（比如根据订单ID的递增规律就能推算出每天的订单量，商业机密啊！）

#### NoSQL

![img](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/nosql-distributed-id.png)

一般情况下，NoSQL方案使用Redis多一些。我们通过Redis的incr命令即可实现对id原子顺序递增。

```bash
127.0.0.1:6379> set sequence_id_biz_type 1
OK
127.0.0.1:6379> incr sequence_id_biz_type
(integer) 2
127.0.0.1:6379> get sequence_id_biz_type
"2"
```

为了提高可用性和并发，我们可以使用Redis Cluster。Redis Cluster是Redis官方提供的Redis集群解决方案（3.0+版本）。

除了Redis Cluster之外，你也可以使用开源的Redis集群方案[Codis](https://github.com/CodisLabs/codis)（大规模集群比如上百个节点的时候比较推荐）。

除了高可用和并发之外，我们知道Redis基于内存，我们需要持久化数据，避免重启机器或者机器故障后数据丢失。Redis支持两种不同的持久化方式：**快照（snapshotting，RDB）**、**只追加文件（append-onlyfile,AOF）**。并且，Redis4.0开始支持**RDB和AOF的混合持久化**（默认关闭，可以通过配置项aof-use-rdb-preamble开启）。


**Redis方案的优缺点：**

- **优点**：性能不错并且生成的ID是有序递增的
- **缺点**：和数据库主键自增方案的缺点类似

除了Redis之外，MongoDB ObjectId经常也会被拿来当做分布式ID的解决方案。

![img](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/mongodb9-objectId-distributed-id.png)

MongoDB ObjectId一共需要12个字节存储：

- 0~3：时间戳
- 3~6：代表机器ID
- 7~8：机器进程ID
- 9~11：自增值

**MongoDB方案的优缺点：**

- **优点**：性能不错并且生成的ID是有序递增的
- **缺点**：需要解决重复ID问题（当机器时间不对的情况下，可能导致会产生重复ID）、有安全性问题（ID生成有规律性）

### 算法

#### UUID

UUID是Universally Unique Identifier（通用唯一标识符）的缩写。UUID包含32个16进制数字（8-4-4-4-12）。

JDK就提供了现成的生成UUID的方法，一行代码就行了。

```java
//输出示例：cb4a9ede-fa5e-4585-b9bb-d60bce986eaa
UUID.randomUUID()
```

[RFC 4122](https://tools.ietf.org/html/rfc4122)中关于UUID的示例是这样的：

![img](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/rfc-4122-uuid.png)

我们这里重点关注一下这个Version(版本)，不同的版本对应的UUID的生成规则是不同的。

5种不同的Version(版本)值分别对应的含义（参考[维基百科对于UUID的介绍](https://zh.wikipedia.org/wiki/通用唯一识别码)）：

- **版本1**:UUID是根据时间和节点ID（通常是MAC地址）生成；
- **版本2**:UUID是根据标识符（通常是组或用户ID）、时间和节点ID生成；
- **版本3、版本5**:版本5-确定性UUID通过散列（hashing）名字空间（namespace）标识符和名称生成；
- **版本4**:UUID使用[随机性](https://zh.wikipedia.org/wiki/随机性)或[伪随机性](https://zh.wikipedia.org/wiki/伪随机性)生成。

下面是Version1版本下生成的UUID的示例：

![Version1版本下生成的UUID的示例](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/version1-uuid.png)

JDK中通过UUID的randomUUID()方法生成的UUID的版本默认为4。

```java
UUID uuid = UUID.randomUUID();
int version = uuid.version();//4
```

另外，Variant(变体)也有4种不同的值，这种值分别对应不同的含义。这里就不介绍了，貌似平时也不怎么需要关注。

需要用到的时候，去看看维基百科对于UUID的Variant(变体)相关的介绍即可。

从上面的介绍中可以看出，UUID可以保证唯一性，因为其生成规则包括MAC地址、时间戳、名字空间（Namespace）、随机或伪随机数、时序等元素，计算机基于这些规则生成的UUID是肯定不会重复的。

虽然，UUID可以做到全局唯一性，但是，我们一般很少会使用它。

比如使用UUID作为MySQL数据库主键的时候就非常不合适：

- 数据库主键要尽量越短越好，而UUID的消耗的存储空间比较大（32个字符串，128位）。
- UUID是无顺序的，InnoDB引擎下，数据库主键的无序性会严重影响数据库性能。

最后，我们再简单分析一下**UUID的优缺点**（面试的时候可能会被问到的哦！）:

- **优点**：生成速度比较快、简单易用
- **缺点**：存储消耗空间大（32个字符串，128位）、不安全(基于MAC地址生成UUID的算法会造成MAC地址泄露)、无序（非自增）、没有具体业务含义、需要解决重复ID问题（当机器时间不对的情况下，可能导致会产生重复ID）

#### Snowflake(雪花算法)

Snowflake是Twitter开源的分布式ID生成算法。Snowflake由64bit的二进制数字组成，这64bit的二进制被分成了几部分，每一部分存储的数据都有特定的含义：

- **第0位**：符号位（标识正负），始终为0，没有用，不用管。
- **第1~41位**：一共41位，用来表示时间戳，单位是毫秒，可以支撑2^41毫秒（约69年）
- **第42~52位**：一共10位，一般来说，前5位表示机房ID，后5位表示机器ID（实际项目中可以根据实际情况调整）。这样就可以区分不同集群/机房的节点。
- **第53~64位**：一共12位，用来表示序列号。序列号为自增值，代表单台机器每毫秒能够产生的最大ID数(2^12=4096),也就是说单台机器每毫秒最多可以生成4096个唯一ID。

![Snowflake示意图](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/snowflake-distributed-id-schematic-diagram.png)

如果你想要使用Snowflake算法的话，一般不需要你自己再造轮子。有很多基于Snowflake算法的开源实现比如美团的Leaf、百度的UidGenerator，并且这些开源实现对原有的Snowflake算法进行了优化。

另外，在实际项目中，我们一般也会对Snowflake算法进行改造，最常见的就是在Snowflake算法生成的ID中加入业务类型信息。

我们再来看看Snowflake算法的优缺点：

- **优点**：生成速度比较快、生成的ID有序递增、比较灵活（可以对Snowflake算法进行简单的改造比如加入业务ID）
- **缺点**：需要解决重复ID问题（依赖时间，当机器时间不对的情况下，可能导致会产生重复ID）。

##### 雪花算法生成分布式ID

```java
public class SnowFlake {

    /**
     * 起始的时间戳
     */
    private final static long START_STMP = 1480166465631L;

    /**
     * 每一部分占用的位数
     */
    private final static long SEQUENCE_BIT = 12; // 序列号占用的位数
    private final static long MACHINE_BIT = 5; // 机器标识占用的位数
    private final static long DATACENTER_BIT = 5;// 数据中心占用的位数

    /**
     * 每一部分的最大值
     */
    private final static long MAX_DATACENTER_NUM = -1L ^ (-1L << DATACENTER_BIT);
    private final static long MAX_MACHINE_NUM = -1L ^ (-1L << MACHINE_BIT);
    private final static long MAX_SEQUENCE = -1L ^ (-1L << SEQUENCE_BIT);

    /**
     * 每一部分向左的位移
     */
    private final static long MACHINE_LEFT = SEQUENCE_BIT;
    private final static long DATACENTER_LEFT = SEQUENCE_BIT + MACHINE_BIT;
    private final static long TIMESTMP_LEFT = DATACENTER_LEFT + DATACENTER_BIT;

    private long datacenterId; // 数据中心
    private long machineId; // 机器标识
    private long sequence = 0L; // 序列号
    private long lastStmp = -1L;// 上一次时间戳

    public SnowFlake(long datacenterId, long machineId) {
        if (datacenterId > MAX_DATACENTER_NUM || datacenterId < 0) {
            throw new IllegalArgumentException("datacenterId can't be greater than MAX_DATACENTER_NUM or less than 0");
        }
        if (machineId > MAX_MACHINE_NUM || machineId < 0) {
            throw new IllegalArgumentException("machineId can't be greater than MAX_MACHINE_NUM or less than 0");
        }
        this.datacenterId = datacenterId;
        this.machineId = machineId;
    }

    /**
     * 产生下一个ID
     *
     * @return
     */
    public synchronized long nextId() {
        long currStmp = getNewstmp();
        if (currStmp < lastStmp) {
            throw new RuntimeException("Clock moved backwards.  Refusing to generate id");
        }

        if (currStmp == lastStmp) {
            // 相同毫秒内，序列号自增
            sequence = (sequence + 1) & MAX_SEQUENCE;
            // 同一毫秒的序列数已经达到最大
            if (sequence == 0L) {
                currStmp = getNextMill();
            }
        } else {
            // 不同毫秒内，序列号置为0
            sequence = 0L;
        }

        lastStmp = currStmp;

        return (currStmp - START_STMP) << TIMESTMP_LEFT // 时间戳部分
                | datacenterId << DATACENTER_LEFT // 数据中心部分
                | machineId << MACHINE_LEFT // 机器标识部分
                | sequence; // 序列号部分
    }

    private long getNextMill() {
        long mill = getNewstmp();
        while (mill <= lastStmp) {
            mill = getNewstmp();
        }
        return mill;
    }

    private long getNewstmp() {
        return System.currentTimeMillis();
    }

    public static void main(String[] args) {
        SnowFlake snowFlake = new SnowFlake(2, 3);

        for (int i = 0; i < (1 << 12); i++) {
            System.out.println(snowFlake.nextId());
        }

    }
}

/**
 * 饿汉式单例模式实现雪花算法
 **/
class SnowFlakeSingleE {

    private static SnowFlakeSingleE snowFlake = new SnowFlakeSingleE();

    private SnowFlakeSingleE() {}

    public static SnowFlakeSingleE getInstance() {
        return snowFlake;
    }

    // 序列号，同一毫秒内用此参数来控制并发
    private long sequence = 0L;
    // 上一次生成编号的时间串，格式：yyMMddHHmmssSSS
    private String lastTime = "";

    public synchronized String getNum() {
        String nowTime = getTime(); // 获取当前时间串，格式：yyMMddHHmmssSSS
        String machineId = "01"; // 机器编号，这里假装获取到的机器编号是2。实际项目中可从配置文件中读取
        // 本次和上次不是同一毫秒，直接生成编号返回
        if (!lastTime.equals(nowTime)) {
            sequence = 0L; // 重置序列号，方便下次使用
            lastTime = nowTime; // 更新时间串，方便下次使用
            return new StringBuilder(nowTime).append(machineId).append(sequence).toString();
        }
        // 本次和上次在同一个毫秒内，需要用序列号控制并发
        if (sequence < 99) { // 序列号没有达到最大值，直接生成编号返回
            sequence = sequence + 1;
            return new StringBuilder(nowTime).append(machineId).append(sequence).toString();
        }
        // 序列号达到最大值，需要等待下一毫秒的到来
        while (lastTime.equals(nowTime)) {
            nowTime = getTime();
        }
        sequence = 0L; // 重置序列号，方便下次使用
        lastTime = nowTime; // 更新时间串，方便下次使用
        return new StringBuilder(nowTime).append(machineId).append(sequence).toString();
    }

    private String getTime() {
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyMMddHHmmssSSS"));
    }
}


/**
 * 懒汉式单例模式实现雪花算法
 **/
class SnowFlakeSingleL {

    private static SnowFlakeSingleL snowFlake = null;

    private SnowFlakeSingleL() {}

    public static SnowFlakeSingleL getInstance() {
        if (snowFlake == null) {
            synchronized (SnowFlakeSingleL.class) {
                if (snowFlake == null) {
                    snowFlake = new SnowFlakeSingleL();
                }
                return snowFlake;
            }
        }
        return snowFlake;
    }

    // 序列号，同一毫秒内用此参数来控制并发
    private long sequence = 0L;
    // 上一次生成编号的时间串，格式：yyMMddHHmmssSSS
    private String lastTime = "";

    public synchronized String getNum() {
        String nowTime = getTime(); // 获取当前时间串，格式：yyMMddHHmmssSSS
        String machineId = "01"; // 机器编号，这里假装获取到的机器编号是2。实际项目中可从配置文件中读取
        // 本次和上次不是同一毫秒，直接生成编号返回
        if (!lastTime.equals(nowTime)) {
            sequence = 0L; // 重置序列号，方便下次使用
            lastTime = nowTime; // 更新时间串，方便下次使用
            return new StringBuilder(nowTime).append(machineId).append(sequence).toString();
        }
        // 本次和上次在同一个毫秒内，需要用序列号控制并发
        if (sequence < 99) { // 序列号没有达到最大值，直接生成编号返回
            sequence = sequence + 1;
            return new StringBuilder(nowTime).append(machineId).append(sequence).toString();
        }
        // 序列号达到最大值，需要等待下一毫秒的到来
        while (lastTime.equals(nowTime)) {
            nowTime = getTime();
        }
        sequence = 0L; // 重置序列号，方便下次使用
        lastTime = nowTime; // 更新时间串，方便下次使用
        return new StringBuilder(nowTime).append(machineId).append(sequence).toString();
    }

    private String getTime() {
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyMMddHHmmssSSS"));
    }
}
```


### 开源框架

#### UidGenerator(百度)

[UidGenerator](https://github.com/baidu/uid-generator)是百度开源的一款基于Snowflake(雪花算法)的唯一ID生成器。

不过，UidGenerator对Snowflake(雪花算法)进行了改进，生成的唯一ID组成如下。

![img](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/uidgenerator-distributed-id-schematic-diagram.png)

可以看出，和原始Snowflake(雪花算法)生成的唯一ID的组成不太一样。并且，上面这些参数我们都可以自定义。

UidGenerator官方文档中的介绍如下：

![img](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/uidgenerator-introduction-official-documents.png)

自18年后，UidGenerator就基本没有再维护了，我这里也不过多介绍。想要进一步了解的朋友，可以看看[UidGenerator的官方介绍](https://github.com/baidu/uid-generator/blob/master/README.zh_cn.md)。

#### Leaf(美团)

**[Leaf](https://github.com/Meituan-Dianping/Leaf)**是美团开源的一个分布式ID解决方案。这个项目的名字Leaf（树叶）起源于德国哲学家、数学家莱布尼茨的一句话：“There are no two identical leaves in the world”（世界上没有两片相同的树叶）。这名字起得真心挺不错的，有点文艺青年那味了！

Leaf提供了**号段模式**和**Snowflake(雪花算法)**这两种模式来生成分布式ID。并且，它支持双号段，还解决了雪花ID系统时钟回拨问题。不过，时钟问题的解决需要弱依赖于Zookeeper。

Leaf的诞生主要是为了解决美团各个业务线生成分布式ID的方法多种多样以及不可靠的问题。

Leaf对原有的号段模式进行改进，比如它这里增加了双号段避免获取DB在获取号段的时候阻塞请求获取ID的线程。简单来说，就是我一个号段还没用完之前，我自己就主动提前去获取下一个号段（图片来自于美团官方文章：[《Leaf—美团点评分布式ID生成系统》openinnewwindow](https://tech.meituan.com/2017/04/21/mt-leaf.html)）。

![img](https://oscimg.oschina.net/oscnet/up-5c152efed042a8fe7e13692e0339d577f5c.png)

根据项目README介绍，在4C8GVM基础上，通过公司RPC方式调用，QPS压测结果近5w/s，TP9991ms。

#### Tinyid(滴滴)

[Tinyid](https://github.com/didi/tinyid)是滴滴开源的一款基于数据库号段模式的唯一ID生成器。

数据库号段模式的原理我们在上面已经介绍过了。**Tinyid有哪些亮点呢？**

为了搞清楚这个问题，我们先来看看基于数据库号段模式的简单架构方案。（图片来自于Tinyid的官方wiki:[《Tinyid原理介绍》](https://github.com/didi/tinyid/wiki/tinyid原理介绍)）

![img](https://oscimg.oschina.net/oscnet/up-4afc0e45c0c86ba5ad645d023dce11e53c2.png)

在这种架构模式下，我们通过HTTP请求向发号器服务申请唯一ID。负载均衡router会把我们的请求送往其中的一台tinyid-server。

这种方案有什么问题呢？在我看来（Tinyid官方wiki也有介绍到），主要由下面这2个问题：

- 获取新号段的情况下，程序获取唯一ID的速度比较慢。
- 需要保证DB高可用，这个是比较麻烦且耗费资源的。

除此之外，HTTP调用也存在网络开销。

Tinyid的原理比较简单，其架构如下图所示：

![img](https://oscimg.oschina.net/oscnet/up-53f74cd615178046d6c04fe50513fee74ce.png)

相比于基于数据库号段模式的简单架构方案，Tinyid方案主要做了下面这些优化：

- **双号段缓存**：为了避免在获取新号段的情况下，程序获取唯一ID的速度比较慢。Tinyid中的号段在用到一定程度的时候，就会去异步加载下一个号段，保证内存中始终有可用号段。
- **增加多db支持**：支持多个DB，并且，每个DB都能生成唯一ID，提高了可用性。
- **增加tinyid-client**：纯本地操作，无HTTP请求消耗，性能和可用性都有很大提升。

Tinyid的优缺点这里就不分析了，结合数据库号段模式的优缺点和Tinyid的原理就能知道。

## 总结

通过这篇文章，我基本上已经把最常见的分布式ID生成方案都总结了一波。

除了上面介绍的方式之外，像ZooKeeper这类中间件也可以帮助我们生成唯一ID。**没有银弹，一定要结合实际项目来选择最适合自己的方案。**

> [原文链接](https://javaguide.cn/distributed-system/distributed-id.html)


#### 相关文章

[深度介绍分布式系统原理与设计](https://mp.weixin.qq.com/s/-30WmwoYHg0oWZSWedE5MQ)
[一口气说出9种分布式ID生成方式](https://mp.weixin.qq.com/s/zXchd2SEjLkHftCe9-2_-Q)
[七种分布式全局ID生成策略，你更爱哪种](https://mp.weixin.qq.com/s/jGq7SvVggZ7gNqM2SZ320Q)
[一起学习下一线大厂的分布式唯一ID生成方案](https://mp.weixin.qq.com/s/dEkkSCbQzfhH3NuXsbbY0w)