---
title: ThreadLocal
categories: Java
index_img: /assert/threadlocal.jpg
img: https://pic3.zhimg.com/v2-7807ea8fe415abd379c725aa449e814c_1440w.jpg?source=172ae18b
---

#### ThreadLocal使用场景
当需要存储线程私有变量的时候、当需要实现线程安全的变量时、当需要减少线程资源竞争的时候。

#### ThreadLocal原理

每个线程是一个Thread实例，其内部维护一个threadLocals的实例成员，其类型是ThreadLocal.ThreadLocalMap。通过实例化ThreadLocal实例，我们可以对当前运行的线程设置一些线程私有的变量，通过调用ThreadLocal的set和get方法存取。ThreadLocal本身并不是一个容器，我们存取的value实际上存储在ThreadLocalMap中，ThreadLocal只是作为TheadLocalMap的key。每个线程实例都对应一个TheadLocalMap实例，我们可以在同一个线程里实例化很多个ThreadLocal来存储很多种类型的值，这些ThreadLocal实例分别作为key，对应各自的value，最终存储在Entry table数组中。

当调用ThreadLocal的set/get进行赋值/取值操作时，首先获取当前线程的ThreadLocalMap实例，然后就像操作一个普通的map一样，进行put和get。


#### ThreadLocalMap为什么使用弱引用而不是强引用？


##### 强引用

一直活着：类似“Object obj=new Object()”这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象实例。

##### 弱引用

回收就会死亡：被弱引用关联的对象实例只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象实例。在JDK1.2之后，提供了WeakReference类来实现弱引用。

##### 软引用

有一次活的机会：软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象实例列进回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。在JDK1.2之后，提供了SoftReference类来实现软引用。

##### 虚引用

也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象实例是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象实例被收集器回收时收到一个系统通知。在JDK1.2之后，提供了PhantomReference类来实现虚引用。

##### 总结
关于为什么ThreadLocalMap使用弱引用而不是强引用分两种情况讨论：
1.key使用强引用
引用ThreadLocal的对象被回收了，但是ThreadLocalMap还持有ThreadLocal的强引用，如果没有手动删除，ThreadLocal不会被回收，导致Entry内存泄漏。
2.key使用弱引
引用ThreadLocal的对象被回收了，由于ThreadLocalMap持有ThreadLocal的弱引用，即使没有手动删除，ThreadLocal也会被回收。value在下一次ThreadLocalMap调用set、get、remove的时候会被清除。

比较两种情况，我们可以发现：由于ThreadLocalMap的生命周期跟Thread一样长，如果都没有手动删除对应key，都会导致内存泄漏，但是使用弱引用可以多一层保障：弱引用ThreadLocal被清理后key为null，对应的value在下一次ThreadLocalMap调用set、get、remove的时候可能会被清除。因此，ThreadLocal内存泄漏的根源是：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。

#### ThreadLocal为什么内存泄漏

因为ThreadLocal是基于ThreadLocalMap实现的，其中ThreadLocalMap的Entry继承了WeakReference，而Entry对象中的key使用了WeakReference封装，也就是说，Entry中的key是一个弱引用类型，对于弱引用来说，它只能存活到下次GC之前，如果此时一个线程调用了ThreadLocalMap的set设置变量，当前的ThreadLocalMap就会新增一条记录，但由于发生了一次垃圾回收，这样就会造成一个结果:key值被回收掉了，但是value值还在内存中，而且如果线程一直存在的话，那么它的value值就会一直存在,这样被垃圾回收掉的key就会一直存在一条引用链:Thread->ThreadLocalMap->Entry->Value:就是因为这条引用链的存在，就会导致如果Thread还在运行，那么Entry不会被回收，进而value也不会被回收掉，但是Entry里面的key值已经被回收掉了,这只是一个线程，如果再来一个线程，又来一个线程…多了之后就会造成内存泄漏

- [详细解读ThreadLocal的内存泄露](https://mp.weixin.qq.com/s/gasR16pjlN3WfuFj9mQxdQ)
- [ThreadLocal你怎么动不动就内存泄漏？](https://mp.weixin.qq.com/s/S0IwbXadRgZ86fFLSFObVQ)
- [细数ThreadLocal三大坑，内存泄露仅是小儿科](https://mp.weixin.qq.com/s/P2eiSHcyf0xMkQmyhTAfhg)
- [内存泄露的原因找到了，罪魁祸首居然是Java TheadLocal](https://mp.weixin.qq.com/s/0Hj4y5lO2Ha4483qluDJ0g)
- [线上系统因为一个ThreadLocal直接内存飙升](https://mp.weixin.qq.com/s/CQA-7FG1txi1pzUgdCV6ig)


#### 相关文章

- [手撕面试题ThreadLocal](https://mp.weixin.qq.com/s/SNLNJcap8qmJF9r4IuY8LA)
- [从头到尾再讲一遍ThreadLocal](https://mp.weixin.qq.com/s/BE6_wskWUaFTrTysHhF5Vw)
- [一文搞懂ThreadLocal原理](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494016&idx=1&sn=94452c28345b6ca8adc37f0879931bf0&source=41#wechat_redirect)
- [面试官：听说你看过ThreadLocal源码](https://mp.weixin.qq.com/s/7sR7okSS1_LGpUWudWtQNw)
- [Java并发之ThreadLocal](https://mp.weixin.qq.com/s/ntjmEHIj_aINhNmtwhMecA)
- [100%保证线程安全，还有这种黑科技](https://mp.weixin.qq.com/s/gfF3y-mnBrnW-VoGaQIn8g)
- [ThreadLocal夺命11连问](https://mp.weixin.qq.com/s/s6waqV8X7KPKip8zbOIDqQ)
