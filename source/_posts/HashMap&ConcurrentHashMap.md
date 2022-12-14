---
title: HashMap&ConcurrentHashMap
categories: Java
index_img: /assert/hashmap.jpg
img: https://pica.zhimg.com/v2-617196c5dc7927460c726d1477174464_1440w.jpg
---

### HashMap

#### 为什么hashmap容器长度是2的幂

HashMap的默认长度为16和规定数组长度为2的幂,是为了降低hash碰撞的几率
加载因子为什么0.75，而不是其他值？
答：可以说是一个经过考量的经验值。加载因子涉及扩容，下次扩容的阈值=数组桶的大小\*加载因子，如果加载因子太小，这就会导致阈值太小，这就会导致比较容易发生扩容。如果加载因子太大，那就会导致阈值太大，可能冲突会很多，导致查找效率下降。负载因子为什么是0.75，如果负载因子为0.5甚至更低的可能的话，最后得到的临时阈值明显会很小，这样的情况就会造成分配的内存的浪费，存在多余的没用的内存空间，也不满足了哈希表均匀分布的情况。如果负载因子达到了1的情况，也就是Entry数组存满了才发生扩容，这样会出现大量的哈希冲突的情况，出现链表过长，因此造成get查询数据的效率。因此选择了0.5~1的折中数也就是0.75，均衡解决了上面出现的情况。
[面试官竟然问我为啥HashMap的负载因子不设置成1！](https://mp.weixin.qq.com/s/kbLASf0lcF4PDJ3qBsFyUg)
[面试官：为什么HashMap的加载因子是0.75？](https://mp.weixin.qq.com/s/ZxwU2qSXvdEZVAIbY_5EPQ)

#### 为什么不能将实数作为HashMap的key？

答：java中浮点数的表示比较复杂，特别是牵涉到-0.0,NaN,正负无穷这种，所以不适宜用来作为Map的key


#### HaspMap为什么线程不安全

答：准确的讲应该是JDK1.7的HashMap链表会有死循环的可能，因为JDK1.7是采用的头插法，在多线程环境下有可能会使链表形成环状，从而导致死循环。JDK1.8做了改进，用的是尾插法，不会产生死循环。
我们从put()方法开始，最终找到线程不安全的那个方法。这里省略中间不重要的过程，我只把方法的跳转流程贴出来：
//添加元素方法->添加新节点方法->扩容方法->把原数组元素重新分配到新数组中
put()-->addEntry()-->resize()-->transfer()
现在，有两个线程都执行transfer方法。每个线程都会在它们自己的工作内存生成一个newTable的数组，用于存储变化后的链表，它们互不影响（这里互不影响，指的是两个新数组本身互不影响）。但是，需要注意的是，它们操作的数据却是同一份。一番扩容操作后出现环形链表，这时，有的同学可能就会问了，就算他们成环了，又怎样，跟死循环有什么关系？我们看下get()方法（最终调用getEntry方法），可以看到查找元素时，只要e不为空，就会一直循环查找下去。若有某个元素C的hash值也落在了和A，B元素同一个桶中，则会由于，A，B互相指向，e.next永远不为空，就会形成死循环。

#### 相关文章

- [Hashtable渐渐被人们遗忘了](https://mp.weixin.qq.com/s/lf15gkZ3woB_6Hw0wQWMJA)
- [阿里面试问HashMap，引发10连炮争吵](https://mp.weixin.qq.com/s/3ew-HiaPu0rDCjSpuyAhOQ)
- [图文并茂，HashMap经典详解](https://mp.weixin.qq.com/s/0106VVsbTa8GuLi5w0c4TA)
- [HashMap怎样解决hash冲突](https://mp.weixin.qq.com/s/gtiUaz810kI3RjemN9ZihQ)
- [HashMap是如何工作的？图文详解，一起来看看](https://mp.weixin.qq.com/s/zdvSs3J2hipGMIsJLe_CbQ)
- [高端的面试从来不会在HashMap的红黑树上纠缠太多](https://mp.weixin.qq.com/s/NTGC1PhhHbQz_h3-s4Y14A)
- [美团面试题：HashMap是如何形成死循环的？（最完整的配图讲解）](https://mp.weixin.qq.com/s/5FdDjDo5H-nDZhFxo7H6QQ)
- [多线程环境下，HashMap为什么会出现死循环？](https://mp.weixin.qq.com/s/gAw9K6yd-w9ZyP90xyvTwg)
- [面试官再问你HashMap底层原理，就把这篇文章甩给他看](https://mp.weixin.qq.com/s/8Nl9dv_ywubW7Wc45--pgw)
- [你好，面试官|你拿JavaMap考验老干部?](https://mp.weixin.qq.com/s/UrSOzclIq8-Cy2D_riBLaA)
- [HashMap面试21问，这次要跪了！](https://mp.weixin.qq.com/s/WyPnPAKZfA58eX7qSBcP8Q)
- [看完这篇HashMap，和面试官扯皮就没问题了](https://mp.weixin.qq.com/s/Ky4pWzdJtpdpJqKBxn71Nw)
- [HashMap和currentHashMap终于总结清楚了](https://mp.weixin.qq.com/s/AX9ZgiAtJ88rPmE3qzt1tA)
- [一文解读所有HashMap的面试题](https://mp.weixin.qq.com/s/TV5Oy6331IM5byzeO31R7w)


### ConcurrentHashMap

jdk1.7 ConcurrentHashMap类所采用的是分段锁的思想，将HashMap进行切割，把HashMap中的哈希数组切分成小数组，这个小数组名叫Segment,每个Segment有n个HashEntry组成，其中Segment继承自ReentrantLock（可重入锁）,在操作的时候给Segment赋予了一个对象锁，从而保证多线程环境下并发操作安全。

JDK1.8对HashMap做了改造，当冲突链表长度大于8时，会将链表转变成红黑树结构,JDK1.8中ConcurrentHashMap类取消了Segment分段锁，采用CAS+synchronized来保证并发安全，数据结构跟jdk1.8中HashMap结构类似，都是数组+链表（当链表长度大于8时，链表结构转为红黑二叉树）结构。ConcurrentHashMap中synchronized只锁定当前链表或红黑二叉树的首节点，只要节点hash不冲突，就不会产生并发，相比JDK1.7的ConcurrentHashMap效率又提升了N倍！

#### ConcurrentHashMap的哪些操作需要加锁？

答：只有写入操作才需要加锁，读取操作不需要加锁

#### ConcurrentHashMap的无锁读是如何实现的？

答：首先HashEntry中的value和next都是有volatile修饰的，其次在写入操作的时候通过调用UNSAFE库延迟同步了主存，保证了数据的一致性

#### 在多线程的场景下调用size（）方法获取ConcurrentHashMap的大小有什么挑战？ConcurrentHashMap是怎么解决的？

答：size()具有全局的语义，如何能保证在不加全局锁的情况下读取到全局状态的值是一个很大的挑战，ConcurrentHashMap通过查看两次无锁读中间是否发生了写入操作来决定读取到的size()是否可信，如果写入操作频繁，则再退化为全局加锁读取。

#### 在有Segment存在的前提下，是如何扩容的？

答：segment数组的大小在一开始初始化的时候就已经决定了，扩容主要扩的是HashEntry数组，基本的思路与HashTable一致，但这是一个线程不安全方法，调用之前需要加锁。

#### 为什么JDK8舍弃掉了分段锁呢？

1. 每个锁控制的是一段，当分段很多，并且加锁的分段不连续的时候，内存空间的浪费比较严重。
2. 如果某个分段特别的大，那么就会影响效率，耽误时间。

#### 相关文章
- [那些年你啃过的ConcurrentHashMap](https://mp.weixin.qq.com/s/ufoKhs4VRXhE8_PT2rXoeg)
- [面试必问之ConcurrentHashMap线程安全的具体实现方式](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491116&idx=1&sn=30ee6196266dab2cbf46cf7f98d99120&source=41#wechat_redirect)
- [ConcurrentHashMap是如何保证线程安全的，你知道么？](https://mp.weixin.qq.com/s/HrPtgYMPhw_DB9jx7u4Byw)
- [面试再被问到ConcurrentHashMap，把这篇文章甩给他](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493923&idx=1&sn=2cfec6fbcd3acb149b53855823838486&source=41#wechat_redirect)
- [ConcurrentHashMap核心原理，这次彻底给整明白了](https://mp.weixin.qq.com/s/bXUYP_0n0p9cXPTbEw0h3A)
- [ConcurrentHashMap中十个提升性能的细节](https://mp.weixin.qq.com/s/MWxKAkncwuqIqnqwJ_qfpA)
- [JDK8的ConcurrentHashMap为什么放弃了分段锁](https://mp.weixin.qq.com/s/nLy7z1emJAybRwetyGO4Sg)
- [一文澄清网上对ConcurrentHashMap的一个流传甚广的误解!](https://mp.weixin.qq.com/s/MN718Yqr2bWud-Rt_zcgAw)