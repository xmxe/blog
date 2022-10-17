---
title: Java中的volatile关键字
sticky: 50
categories: Java 
index_img: /assert/volatile.jpg
img: 
---

##### 原理

并发编程中的三个概念: 原子性问题，可见性问题，有序性问题
**原子性**：即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。例如银行转账
**可见性**：是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值
**有序性**：即程序执行的顺序按照代码的先后顺序执行
要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。

cpu很快 但是cpu从内存里读取数据和写入数据是很慢，为了提高读写内存速度，硬件工程师在cpu上加了一块高速缓存(工作内存),cpu从主内存中读取数据，在高速缓存(工作内存)中建立副本变量，在操作完副本变量后将变量更新到主内存，而volatile关键字则是告诉jvm ，它所修饰的变量不保留拷贝，直接访问主内存中的数据，这样就保证了可见性，其实就是如果一个变量声明成是volatile的，那么当我读变量时，总是能读到它的最新值，这里最新值是指不管其它哪个线程对该变量做了写操作，都会立刻被更新到主存里，我也能从主存里读到这个刚写入的值。也就是说volatile关键字可以保证可见性以及有序性，但不保证原子性

JMM（java内存模型）允许编译器和处理器对指令重排序的，但是规定了as-if-serial语义，即不管怎么重排序，程序的执行结果不能改变
**volatile禁止指令重排(指令重排序)，不支持原子性操作**
volatile变量规则： 对一个volatile域的写，happens-before（发生前）于后续对这个volatile域的读 即写在读之前

##### 使用场景

您只能在有限的一些情形下使用 volatile 变量替代锁。要使 volatile 变量提供理想的线程安全，必须同时满足下面两个条件：
1. 对变量的写操作不依赖于当前值。
2. 该变量没有包含在具有其他变量的不变式中。

volatile最适用一个线程写，多个线程读的场合。如果有多个线程并发写操作，仍然需要使用锁或者线程安全的容器或者原子变量来代替

JMM具备一些先天的有序性,即不需要通过任何手段就可以保证的有序性，通常称为happens-before原则。<<JSR-133：Java Memory Model and Thread Specification>>定义了如下happens-before规则
1. 程序顺序规则： 一个线程中的每个操作，happens-before于该线程中的任意后续操作
2. 监视器锁规则：对一个线程的解锁，happens-before于随后对这个线程的加锁
3. volatile变量规则： 对一个volatile域的写，happens-before于后续对这个volatile域的读
4. 传递性：如果A happens-before B ,且 B happens-before C, 那么 A happens-before C
5. start()规则： 如果线程A执行操作ThreadB_start()(启动线程B) , 那么A线程的ThreadB_start()happens-before 于B中的任意操作
6. join()原则： 如果A执行ThreadB.join()并且成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。
7. interrupt()原则： 对线程interrupt()方法的调用先行发生于被中断线程代码检测到中断事件的发生，可以通过Thread.interrupted()方法检测是否有中断发生
8. finalize()原则：一个对象的初始化完成先行发生于它的finalize()方法的开始

**System.out.println()会影响volatile可见性**
因为println()使用synchronized修饰的，JMM关于synchronized的两条规定:

1. 线程解锁前，必须把共享变量的最新值刷新到主内存中
2. 线程加锁时，将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新获取最新的值

##### 相关文章

- [面试：说说Java中的 volatile 关键词？](https://mp.weixin.qq.com/s/0a-74XQagO0TkN5b5RMD3A)
- [volatile和synchronized到底啥区别？多图文讲解告诉你](https://mp.weixin.qq.com/s/U8WoqH1YRdh1GyFiPDw0KQ)
- [面试:Java并发之volatile我彻底懂了](https://mp.weixin.qq.com/s/2srKmbZlgmUuuT7kCrL0RQ)
- [面试官最爱的volatile关键字](https://mp.weixin.qq.com/s?__biz=MzI4MDYwMDc3MQ==&mid=2247486266&idx=1&sn=7beaca0358914b3606cde78bfcdc8da3&chksm=ebb74296dcc0cb805a45ca9c0501b7c2c37e8f2586295210896d18e3a0c72b01bea765924ce5&mpshare=1&scene=24&srcid=&key=c8fbfa031bd0c4166acd110fd54b85e9b3568f80a3f4c2d80add2f4add0ced46d1d3a0cf139c0ca64877a98635727a7fc593b850f8082d1fcf77a5ebf067fc1476285146d13d691f80b64b930006a341&ascene=0&uin=MjYwNzAzMzYzNw%3D%3D&devicetype=iMac+MacBookAir6%2C2+OSX+OSX+10.14.2+build(18C54)&version=12020810&nettype=WIFI&lang=zh_CN&fontScale=100&pass_ticket=hbg9AwR77rok2jxxdwyHyTHBDzwwC7lR8aEfF6HfW4KgJwsj0ruOpw8iNsUK%2B5kK)
- [26 张图带你彻底搞懂 volatile 关键字！](https://mp.weixin.qq.com/s/ShZwxqQb2Y-it27QawB9PQ)
- [掉了两根头发，可算是把volatile整明白了](https://mp.weixin.qq.com/s/qX-IxNw86kw5Lk_6D07x1w)
- [面试官：谈谈 CPU Cache 工作原理，Cache 一致性？我懵了。。](https://mp.weixin.qq.com/s/Md1s1t_V1gPNZr0JsoLebQ)