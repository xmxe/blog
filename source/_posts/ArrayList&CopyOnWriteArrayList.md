---
title: ArrayList&CopyOnWriteArrayList
sticky: 10
categories: Java
index_img: /assert/list.jpg
img: 
---

#### ArrayList的扩容机制

当添加元素的时候数组是空的，则直接给一个10长度的数组。当需要长度的数组大于现在长度的数组的时候，通过新=旧+旧>>1(即新=1.5倍的旧)来扩容，当扩容的大小还是不够需要的长度的时候，则将数组大小直接置为需要的长度

[ArrayList的扩容机制了解吗？](https://mp.weixin.qq.com/s/GY7RLE-yIF7jPAjqu5V9Cg)
[学会这几行，还不懂ArrayList，你来怼我](https://mp.weixin.qq.com/s/GQgerZ79qKYukodqRKUo1g)

#### CopyOnWriteArrayList
是一个典型的读写分离的动态数组操作类！

在写入数据的时候，将旧数组内容复制一份出来，然后向新的数组写入数据，最后将新的数组内存地址返回给数组变量；移除操作也类似，只是方式是移除元素而不是添加元素；而查询方法，因为不涉及线程操作，所以并没有加锁出来！因为CopyOnWriteArrayList读取内容没有加锁，在写入数据的时候同时也可以进行读取数据操作，因此性能得到很大的提升，但是也有缺陷，对于边读边写的情况，不一定能实时的读到最新的数据

[CopyOnWriteArrayList你了解多少？](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491084&idx=2&sn=ba8f6c452a12b2bb6cfd5dca1b4d177c&source=41#wechat_redirect)
[学会了CopyOnWriteArrayList可以再多和面试官对线三分钟](https://mp.weixin.qq.com/s/sgiRiWzob1thgux7oU0RDw)

#### 集合

- [别再催我更新集合知识了](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491083&idx=1&sn=771339bd09071818f1156072fb507e41&source=41#wechat_redirect)
- [厉害！Java集合框架综述，这篇让你吃透！](https://mp.weixin.qq.com/s/-QGlcunN2pac0F9mrRq1DQ)
- [Java集合框架看这一篇就够了](https://mp.weixin.qq.com/s/TW3cRsFZwqUiOPC5cbSRRQ)