---
title: Java中的volatile关键字
categories: Java
index_img: /assert/volatile.jpg
img: https://pic2.zhimg.com/v2-1858f3d5d46af69688092c4c0a10285a_1440w.jpg

---

> [VolatileTest.java](https://github.com/xmxe/demo/blob/master/study-demo/src/main/java/com/xmxe/jdkfeature/thread/VolatileTest.java)

### 如何保证变量的可见性？

在Java中，volatile关键字可以保证变量的可见性，如果我们将变量声明为**volatile**，这就指示JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/concurrent/jmm.png)

![JMM(Java内存模型)强制在主存中进行读取](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/concurrent/jmm2.png)

volatile关键字其实并非是Java语言特有的，在C语言里也有，它最原始的意义就是禁用CPU缓存。如果我们将一个变量使用volatile修饰，这就指示编译器，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

volatile关键字能保证数据的可见性，但不能保证数据的原子性。synchronized关键字两者都能保证。

### 如何禁止指令重排序？

在Java中，volatile关键字除了可以保证变量的可见性，还有一个重要的作用就是防止JVM的指令重排序。如果我们将变量声明为volatile，在对这个变量进行读写操作的时候，会通过插入特定的内存屏障的方式来禁止指令重排序。

在Java中，Unsafe类提供了三个开箱即用的内存屏障相关的方法，屏蔽了操作系统底层的差异：

```java
public native void loadFence();
public native void storeFence();
public native void fullFence();
```

理论上来说，你通过这个三个方法也可以实现和volatile禁止重排序一样的效果，只是会麻烦一些。面试中面试官经常会说：“单例模式了解吗？来给我手写一下！给我解释一下双重检验锁方式实现单例模式的原理呗！”

**双重校验锁实现对象单例（线程安全）**：

```java
public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public  static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

uniqueInstance采用volatile关键字修饰也是很有必要的，`uniqueInstance = new Singleton();`这段代码其实是分为三步执行：

1. 为`uniqueInstance`分配内存空间
2. 初始化`uniqueInstance`
3. 将`uniqueInstance`指向分配的内存地址

但是由于JVM具有指令重排的特性，执行顺序有可能变成1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程T1执行了1和3，此时T2调用getUniqueInstance()后发现uniqueInstance不为空，因此返回uniqueInstance，但此时uniqueInstance还未被初始化。

### volatile可以保证原子性么？

**volatile关键字能保证变量的可见性，但不能保证对变量的操作是原子性的**。

我们通过下面的代码即可证明：

```java

public class VolatoleAtomicityDemo {
    public volatile static int inc = 0;

    public void increase() {
        inc++;
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
        VolatoleAtomicityDemo volatoleAtomicityDemo = new VolatoleAtomicityDemo();
        for (int i = 0; i < 5; i++) {
            threadPool.execute(() -> {
                for (int j = 0; j < 500; j++) {
                    volatoleAtomicityDemo.increase();
                }
            });
        }
        // 等待1.5秒，保证上面程序执行完成
        Thread.sleep(1500);
        System.out.println(inc);
        threadPool.shutdown();
    }
}
```

正常情况下，运行上面的代码理应输出`2500`。但你真正运行了上面的代码之后，你会发现每次输出结果都小于`2500`。为什么会出现这种情况呢？不是说好了，volatile可以保证变量的可见性嘛！也就是说，如果volatile能保证inc++操作的原子性的话。每个线程中对inc变量自增完之后，其他线程可以立即看到修改后的值。5个线程分别进行了500次操作，那么最终inc的值应该是5*500=2500。

很多人会误认为自增操作`inc++`是原子性的，实际上，`inc++`其实是一个复合操作，包括三步：

1. 读取inc的值。
2. 对inc加1。
3. 将inc的值写回内存。

volatile是无法保证这三个操作是具有原子性的，有可能导致下面这种情况出现：

1. 线程1对inc进行读取操作之后，还未对其进行修改。线程2又读取了inc的值并对其进行修改（+1），再将inc的值写回内存。
2. 线程2操作完毕后，线程1对inc的值进行修改（+1），再将inc的值写回内存。

这也就导致两个线程分别对inc进行了一次自增操作后，inc实际上只增加了1。其实，如果想要保证上面的代码运行正确也非常简单，利用synchronized、Lock或者AtomicInteger都可以。

使用**synchronized**改进：

```java
public synchronized void increase() {
    inc++;
}
```

使用**AtomicInteger**改进：

```java
public AtomicInteger inc = new AtomicInteger();

public void increase() {
    inc.getAndIncrement();
}
```

使用**ReentrantLock**改进：

```java
Lock lock = new ReentrantLock();
public void increase() {
    lock.lock();
    try {
        inc++;
    } finally {
        lock.unlock();
    }
}
```

### 原理

并发编程中的三个概念:原子性问题，可见性问题，有序性问题
**原子性**：即一个操作或者多个操作,要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。例如银行转账
**可见性**：是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值
**有序性**：即程序执行的顺序按照代码的先后顺序执行
要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。

cpu很快,但是cpu从内存里读取数据和写入数据是很慢，为了提高读写内存速度，硬件工程师在cpu上加了一块高速缓存(工作内存),cpu从主内存中读取数据，在高速缓存(工作内存)中建立副本变量，在操作完副本变量后将变量更新到主内存，而volatile关键字则是告诉jvm，它所修饰的变量不保留拷贝，直接访问主内存中的数据，这样就保证了可见性，其实就是如果一个变量声明成是volatile的，那么当我读变量时，总是能读到它的最新值，这里最新值是指不管其它哪个线程对该变量做了写操作，都会立刻被更新到主存里，我也能从主存里读到这个刚写入的值。也就是说volatile关键字可以保证可见性以及有序性，但不保证原子性

JMM（java内存模型）允许编译器和处理器对指令重排序的，但是规定了as-if-serial语义，即不管怎么重排序，程序的执行结果不能改变
**volatile禁止指令重排(指令重排序)，不支持原子性操作**
volatile变量规则：对一个volatile域的写，happens-before（发生前）于后续对这个volatile域的读,即写在读之前

### 使用场景

您只能在有限的一些情形下使用volatile变量替代锁。要使volatile变量提供理想的线程安全，必须同时满足下面两个条件：
1. 对变量的写操作不依赖于当前值。
2. 该变量没有包含在具有其他变量的不变式中。

volatile最适用一个线程写，多个线程读的场合。如果有多个线程并发写操作，仍然需要使用锁或者线程安全的容器或者原子变量来代替

JMM具备一些先天的有序性,即不需要通过任何手段就可以保证的有序性，通常称为happens-before原则。<<JSR-133：Java Memory Model and Thread Specification>>定义了如下happens-before规则
1. 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作
2. 监视器锁规则：对一个线程的解锁，happens-before于随后对这个线程的加锁
3. volatile变量规则：对一个volatile域的写，happens-before于后续对这个volatile域的读
4. 传递性：如果A happens-before B,且B happens-before C,那么A happens-before C
5. start()规则：如果线程A执行操作ThreadB_start()(启动线程B),那么A线程的ThreadB_start()happens-before于B中的任意操作
6. join()原则：如果A执行ThreadB.join()并且成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。
7. interrupt()原则：对线程interrupt()方法的调用先行发生于被中断线程代码检测到中断事件的发生，可以通过Thread.interrupted()方法检测是否有中断发生
8. finalize()原则：一个对象的初始化完成先行发生于它的finalize()方法的开始

**System.out.println()会影响volatile可见性**
因为println()使用synchronized修饰的，JMM关于synchronized的两条规定:

1. 线程解锁前，必须把共享变量的最新值刷新到主内存中
2. 线程加锁时，将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新获取最新的值

### 相关文章

| [面试：说说Java中的volatile关键词？](https://mp.weixin.qq.com/s/0a-74XQagO0TkN5b5RMD3A) | [volatile和synchronized到底啥区别？多图文讲解告诉你](https://mp.weixin.qq.com/s/U8WoqH1YRdh1GyFiPDw0KQ) | [面试:Java并发之volatile我彻底懂了](https://mp.weixin.qq.com/s/2srKmbZlgmUuuT7kCrL0RQ) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [面试官最爱的volatile关键字](https://mp.weixin.qq.com/s?__biz=MzI4MDYwMDc3MQ==&mid=2247486266&idx=1&sn=7beaca0358914b3606cde78bfcdc8da3&chksm=ebb74296dcc0cb805a45ca9c0501b7c2c37e8f2586295210896d18e3a0c72b01bea765924ce5&mpshare=1&scene=24&srcid=&key=c8fbfa031bd0c4166acd110fd54b85e9b3568f80a3f4c2d80add2f4add0ced46d1d3a0cf139c0ca64877a98635727a7fc593b850f8082d1fcf77a5ebf067fc1476285146d13d691f80b64b930006a341&ascene=0&uin=MjYwNzAzMzYzNw%3D%3D&devicetype=iMac+MacBookAir6%2C2+OSX+OSX+10.14.2+build(18C54)&version=12020810&nettype=WIFI&lang=zh_CN&fontScale=100&pass_ticket=hbg9AwR77rok2jxxdwyHyTHBDzwwC7lR8aEfF6HfW4KgJwsj0ruOpw8iNsUK%2B5kK) | [26张图带你彻底搞懂volatile关键字！](https://mp.weixin.qq.com/s/ShZwxqQb2Y-it27QawB9PQ) | [掉了两根头发，可算是把volatile整明白了](https://mp.weixin.qq.com/s/qX-IxNw86kw5Lk_6D07x1w) |
| [面试官：谈谈CPU Cache工作原理，Cache一致性？我懵了。。](https://mp.weixin.qq.com/s/Md1s1t_V1gPNZr0JsoLebQ) |                                                              |                                                              |