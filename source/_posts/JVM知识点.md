---
title: JVM知识点
sticky: 41
categories: Java 
index_img: /assert/jvm.jpg
img: https://pic1.zhimg.com/v2-7382fb7654c33a5e843f51ec139b1da0_r.jpg
coverImg: https://pic1.zhimg.com/v2-7382fb7654c33a5e843f51ec139b1da0_r.jpg
cover: true
summary: JVM内存结构、共享区域、相关命令
top: true
---

#### JVM5大内存区域

- Java虚拟机栈（栈区）
- 本地方法栈
- Java堆（堆区）
- 方法区
- 程序计数器

##### Java堆

GC堆是java虚拟机所管理的内存中最大的一块内存区域，也是被各个线程共享的内存区域，在JVM启动时创建。其大小通过-Xms(最小值)和-Xmx(最大值)参数设置，-Xms为JVM启动时申请的最小内存，-Xmx为JVM可申请的最大内存。由于现在收集器都是采用分代收集算法，堆被划分为新生代和老年代。新生代可通过-Xmn参数来指定新生代的大小。对象刚创建的时候，会被创建在新生代，到一定阶段之后会移送至老年代 ，如果创建了一个新生代无法容纳的新对象，那么这个新对象也可以创建到老年代
**所有对象实例以及数组都在堆上分配**。 堆内存用来存放由new创建的对象实例和数组 堆中不存放基本数据类型和对象引用，只存放对象本身 (包括属性,即成员变量) jvm只有一个堆区(heap)被所有线程共享

- **新生代(Young Gen)** ：新生代主要存放新创建的对象，内存大小相对会比较小，垃圾回收会比较频繁。新生代分为1个Eden区和2个S区，S代表Survivor。当对象在堆创建时，将进入Eden Space。垃圾回收器进行垃圾回收时，它的策略是会把没有引用的对象直接给回收掉，还有引用的对象会被移送到Survivor区。Survivor区有S0和S1两个内存空间，每次进行YGC的时候，会将存活的对象复制到未使用的那块内存空间，然后将当前正在使用的空间完全清除掉，再交换两个空间的使用状况。如果YGC要移送的对象Survivor区无法容纳，那么就会将该对象直接移交给老年代。上面说了，到一定阶段的对象会移送到老年区，这是什么意思呢？每一个对象都有一个计数器，当每次进行YGC的时候，都会 +1。通过-XX:MAXTenuringThrehold参数可以配置当计数器的值到达某个阈值时，对象就会从新生代移送至老年代。 该参数的默认值为15，也就是说对象在Survivor区中的S0和S1内存空间交换的次数累加到15次之后，就会移送至老年代。如果参数配置为1，那么创建的对象就会直接移送至老年代。扫描完毕后，JVM将Eden Space和A Suvivor Space清空，然后交换A和B的角色，即下次垃圾回收时会扫描Eden Space和B Suvivor Space。这么做主要是为了减少内存碎片的产生。我们可以看到：Young Gen垃圾回收时，采用将存活对象复制到到空的Suvivor Space的方式来确保尽量不存在内存碎片，采用空间换时间的方式来加速内存中不再被持有的对象尽快能够得到回收。

- **老年代(Tenured Gen)** ：老年代主要存放JVM认为生命周期比较长的对象（经过几次的Young Gen的垃圾回收后仍然存在），内存大小相对会比较大，垃圾回收也相对没有那么频繁（譬如可能几个小时一次）。老年代主要采用压缩的方式来避免内存碎片（将存活对象移动到内存片的一边，也就是内存整理）。当然，有些垃圾回收器（譬如CMS垃圾回收器）出于效率的原因，可能会不进行压缩。

##### 虚拟机栈

每个线程包含一个栈区，栈中只保存基础数据类型的对象和自定义对象的引用(不是对象)，对象都存放在堆区中,每个栈中的数据(原始类型和对象引用)都是私有的，其他栈不能访问。定义的局部变量也在栈内存中 线程私有,FILO(先进后出)
与程序计数器一样，Java虚拟机栈也是线程私有的，它的生命周期与线程相同,每个方法被执行的时候都会创建一个"栈帧",用于存储局部变量表(包括参数)、操作数栈、动态链接、方法出口等信息。每个方法被调用到执行完的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。
局部变量表存放各种基本数据类型boolean、byte、char、short等

##### 本地方法栈

本地方法栈和JVM栈非常相似，它们之间的区别不过是jvm栈是为执行java方法服务，而本地方法栈是为jvm使用到对的本地方法服务。HotSpot虚拟机中直接把本地方法栈和JVM栈合二为一了。
线程私有 FILO(先进后出)
与虚拟机栈基本类似，区别在于虚拟机栈为虚拟机执行的java方法服务，而本地方法栈则是为Native方法服务。

##### 方法区
jdk1.7及以前方法区也被称为永久代，1.8之后移除，取而代之的为metaspace 元空间,注意：元空间，永久代都是方法区的一种实现,永久代主要存放类定义、字节码和常量等很少会变更的信息。永久带是方法区的一种实现，可以理解为就是方法区，类似于java的接口和实现类

方法区是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、静态变量、final类型的常量、属性和方法信息，即时编译器编译后的代码等数据 也称”永久代” ，它用于存储虚拟机加载的类信息、常量、静态变量、是各个线程共享的内存区域。可以通过-XX:PermSize 和 -XX:MaxPermSize 参数限制方法区的大小。

运行时常量池：是方法区的一部分，其中的主要内容来自于JVM对Class的加载。
Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池，用于存放编译器生成的各种符号引用，这部分内容将在类加载后放到方法区的运行时常量池中。

Java1.7之前，常量池是存放在方法区中的，运行常量池是方法区的一部分。方法区是堆的一个逻辑部分，他有一个名字叫做非堆。
> jdk1.7之前
> ![](/images/jvm1.7.png)


Java1.7，把字符串常量池放到了堆中。JVM已经将运行时常量池从方法区中移了出来，在JVM堆开辟了一块区域存放常量池。
> jdk1.7
> ![](images/jvm7.png)

Java8之后，取消了整个永久代区域，取而代之的是元空间。方法区概念保留，方法区的实现改为了元空间，元空间也是方法区的一种实现,常量池还是在堆中,没有再对常量池进行调整。元空间占用本地内存，不再占用jvm内存
> jdk1.8
> ![](/images/jvm1.8.png)


##### 程序计数器

存放下一条指令所在单元地址的地方
程序计数器是一块较小的内存空间，可以看作当前线程所执行的字节码的行号指示器。在虚拟机的模型里，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、异常处理、线程恢复等基础功能都需要依赖计数器完成


##### 线程共享区域
- java堆
- 方法区

##### 线程私有区域

- JVM栈
- 本地方法栈
- 程序计数器

##### 示例
Foo foo = new Foo(); 
foo.f();
以上代码的内存实现原理为： 
1. Foo类首先被装载到JVM的方法区，其中包括类的信息，包括方法和构造等。 
2. 在栈内存中分配引用变量foo。 
3. 在堆内存中按照Foo类型信息分配实例变量内存空间；然后，将栈中引用foo指向foo对象堆内存的首地址。 
4. 使用引用foo调用方法，根据foo引用的类型Foo调用f方法。

![](/images/jvmdemo.png)
![](/images/jvmdemo2.jpg)

- [JVM 底层原理最全知识总结--GitHub](https://github.com/doocs/jvm)


#### 相关文章

##### JVM内存

- [2万字长文包教包会 JVM 内存结构](https://mp.weixin.qq.com/s/VDZNpS4Qk0jvv_MctVXhww)
- [13 张图解 Java 中的内存模型](https://mp.weixin.qq.com/s/T5R_uxTMxo5WBv2Zvb4H7A)
- [面试官：说说什么是Java内存模型？](https://mp.weixin.qq.com/s/UomhyS-oZK4zjOg9M1F4_Q)
- [图文并茂，傻瓜都能看懂的 JVM 内存布局](https://mp.weixin.qq.com/s/oDeO8Td-SJn9g4mRygqtSw)
- [JVM 内存划分](https://mp.weixin.qq.com/s/TxKEYPoTcudoep6EBd-rRA)
- [小白都能看得懂的java虚拟机内存模型](https://mp.weixin.qq.com/s/m2dp6jv8lfmy-S2gmQDHvA)
- [深入理解堆外内存 Metaspace](https://mp.weixin.qq.com/s/xB-uiqy4eVsNovEknO9_5w)
- [求你了，别再说Java对象都是在堆内存上分配空间的了](https://mp.weixin.qq.com/s/Owlhu5IFpDAyu0WYcK1EhQ)
- [终于搞懂了Java 8 的内存结构，再也不纠结方法区和常量池了！！](https://mp.weixin.qq.com/s/8uGOt1OJloZMHQl43vrbyQ)
- [个人笔记，深入理解 JVM，很全！](https://mp.weixin.qq.com/s/1rxmi3lg02I7nHwWbldvLA)
- [JVM 内存布局详解，图文并茂，写得太好了！](https://mp.weixin.qq.com/s/RySwVg9SYvSM-ONdB9cOVw)
- [为啥一问JVM就懵B？？？](https://mp.weixin.qq.com/s/x7YNbT5a2DZRg2qe5jyWPA)

##### JVM垃圾回收

- [7种JVM 垃圾回收器及垃圾回收流程](https://mp.weixin.qq.com/s/fyorrpT5-hFpIS5aEDNjZA)
- [深度解析ZGC](https://mp.weixin.qq.com/s/NYrUEYmuWQw6RrN1VVILpg)
- [深度揭秘垃圾回收底层，这次让你彻底弄懂它](https://mp.weixin.qq.com/s/pb7h9ROzr5LOLA0gMnquqQ)
- [看完这篇垃圾回收，和面试官扯皮没问题了](https://mp.weixin.qq.com/s/EcTwdaxP-Z4jG7PDq50nSA)
- [详解 Java性能优化和JVM GC（垃圾回收机制）](https://mp.weixin.qq.com/s/Bfz7sqaI4iJuGe_i5dUXgA)
- [图文并茂，万字详解，带你掌握 JVM 垃圾回收！](https://mp.weixin.qq.com/s/EYOD4mQ7ErB11xMrGgzfCw)
- [垃圾回收-实战篇](https://mp.weixin.qq.com/s/k6NNxnlYGxOS3HiR_jEn_A)
- [面试的时候按照这个套路回答 Java GC 的相关问题一定能过！](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247490925&idx=1&sn=e44e924135109d93eb2a34afd6bf280b&source=41#wechat_redirect)
- [一篇文章搞懂GC垃圾回收](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494090&idx=1&sn=0cb942ee592e277014b6cd16059c13c7&source=41#wechat_redirect)
- [垃圾回收策略和算法，看这篇就够了](https://mp.weixin.qq.com/s/Anj6PRc9UPiWGDnpKahxUQ)
- [炸了！一口气问了我18个JVM问题](https://mp.weixin.qq.com/s/U8uzm3YjdqoYCtLPPosSrg)
- [面试官问我JVM的GC分代收集算法为什么这么设计](https://mp.weixin.qq.com/s/4xM_sg5vH8uxXJeaOQpraw)
- [一文了解 Java 8 - 18，垃圾回收的十次进化](https://mp.weixin.qq.com/s/nF489_nmrFAWcw1IfFF9lg)


##### JVM知识点

- [JVM 知识点全面梳理！](https://mp.weixin.qq.com/s/IUw8SroGMJdvqPFQ1xEHxg)
- [《深入理解 Java 虚拟机》把这个知识点讲错了？](https://mp.weixin.qq.com/s/4qZLipxi-zjrdI-_6Ae11A)
- [JVM 史上最最最完整知识总结！](https://mp.weixin.qq.com/s/GAXLr0cIcLnGaTXVstHKvA)
- [JVM面试的30个知识点](https://mp.weixin.qq.com/s/PM_Q3718Du81ZFEx0ARrkw)
- [满满的一整篇，全是 JVM 核心知识点！](https://mp.weixin.qq.com/s/GfAfffbF_uiphN0Zd3YouQ)
- [大白话带你认识JVM、JVM的常用参数](https://mp.weixin.qq.com/s/CZtW-rDXgRl9E_3N2ou5TA)
- [JVM夺命连环10问](https://mp.weixin.qq.com/s/_0IANOvyP_UNezDm0bxXmg)
- [【干货】JVM 完整深入解析！](https://mp.weixin.qq.com/s/vd72d6wqc_fq7YUgA7d1fA)
- [如何在面试时搞定 Java 虚拟机](https://mp.weixin.qq.com/s/6MSezM6g4JPoML0kDtx64Q)
- [JAVA虚拟机(JVM)面试题](https://mp.weixin.qq.com/s/eNpHYG9T5DIdh7QNpLg4dQ)
- [JVM史上最最最完整深入解析，不看后悔一百次！](https://mp.weixin.qq.com/s/EYUNHXLP9jjTTA1Ig6vCYQ)
- [【秒懂！】JVM虚拟机图文详解！一点都不难！](https://mp.weixin.qq.com/s/A4BbKDmy8oe3uBfHfrgGQA)
- [从 JMM 透析 volatile 与 synchronized 原理](https://mp.weixin.qq.com/s/s2nnXDY7phqKX07nwZiHyA)


##### 内存溢出分析

- [常见OOM异常分析](https://mp.weixin.qq.com/s/DssuTRyN1RVWhQgMClP_ng)
- [大厂的OOM优化和监控方案](https://mp.weixin.qq.com/s/-pZDUcW2ljBSIxZV0w0iSw)
- [如何排查Java内存泄漏?看完我给跪了!](https://mp.weixin.qq.com/s/fH6Go6_1TcvHfxDPnyZvhg)
- [常见的 OOM 异常分析（硬核干货）](https://mp.weixin.qq.com/s/oNWwPZ56yLCp1KQyir17AA)
- [深入剖析线上内存溢出的原因](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494348&idx=1&sn=0096ff574aace3d725bcbb3323a67d01&source=41#wechat_redirect)
- [一套完整的 Java 线上故障排查技巧，建议收藏](https://mp.weixin.qq.com/s/03qoBFLpvkRVhaorwCNJ5Q)
- [GC垃圾收集器&JVM调优汇总](https://mp.weixin.qq.com/s/UzGeDeuZORaJkzFr_V6-5A)
- [图解Java虚拟机中GC的复制算法和“标记-整理”算法](https://mp.weixin.qq.com/s/1_Dz7vl9k01FtyYHIn107w)


##### JVM分析工具

- [这几款 JVM 故障诊断处理工具，你还不会？](https://mp.weixin.qq.com/s/b5pnoqWNMIp7JnfUWyWLoA)
- [JVM 性能调优监控工具 jps、jstack、jmap、jhat、jstat、hprof 使用详解](https://mp.weixin.qq.com/s/72X1qvTsCLneFffzIpKoaA)
- [有了这款可视化工具，Java 应用性能调优 so easy！(JVisualVM 简介)](https://mp.weixin.qq.com/s/rLhkbpe0Uo3Nsy8Xs68Z7A)
- [死锁的 4 种排查工具 ！](https://mp.weixin.qq.com/s/5l6q1-jcfZNegpUsriaj5g)
- [6 款 Java 8 自带工具，轻松分析定位 JVM 问题！](https://mp.weixin.qq.com/s/phD24wUysUDQkx5pAjp5tg)

#### jvm相关命令

##### jps

显示当前所有java进程pid的命令，我们可以通过这个命令来查看到底启动了几个java进程

```shell
jps # 查看本地正在运行的java进程和进程ID（pid）
jps -l # 输出应用程序main.class的完整package名或者应用程序jar文件完整路径名
jps -v # 输出传递给JVM的参数
```

#####  jstack

主要用于生成指定进程当前时刻的线程快照，线程快照是当前java虚拟机每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致长时间等待。

##### jmap

主要用于打印指定java进程的共享对象内存映射或堆内存细节。

```shell
jmap -heap pid # 输出堆内存设置和使用情况（JDK11使用jhsdb jmap --heap --pid pid）
jmap -histo pid # 查看堆中对象数量和大小，包括类名，对象数量，对象占用大小
jmap -histo:live pid # 同上，只输出存活对象信息
jmap -clstats pid # 输出加载类信息
jmap -help # jmap命令帮助信息
```

#####  生成dump文件

方式一、jmap -dump:live,format=b,file=heap-dump.bin  (pid)
方式二、使用JConsole的dumpHeap 按钮生成 Heap Dump文件
方式三、在JVM的配置参数中可以添加
-XX:+HeapDumpOnOutOfMemoryError 参数，当应用抛出 OutOfMemoryError 时自动生成dump文件；
-XX:HeapDumpPath=/home/liuke/jvmlogs/ ：生成堆文件地址
在JVM的配置参数中添加 -Xrunhprof:head=site 参数，会生成java.hprof.txt 文件，不过这样会影响JVM的运行效率，不建议在生产环境中使用（未亲测）

#####  jinfo

可以用来查看正在运行的java运用程序的扩展参数，甚至支持在运行时动态地更改部分参数。
```shell
jinfo pid # 查看指定pid的所有JVM信息
jinfo -flags pid # 查看设置过值的参数
jinfo -flag InitialHeapSize pid # 查看初始堆内存
jinfo -flag MaxHeapSize pid # 查看最大堆内存
jinfo -flag PermSize pid # 查看初始分配的非堆内存
jinfo -flag MaxPermSize pid # 查看最大允许分配的非堆内存
jinfo -flag NewSize pid # 查看年轻代初始内存
jinfo -flag MaxNewSize pid # 查看年轻代最大内存
jinfo -flag NewRatio pid # 查看年轻代与年老代的比值
jinfo -flag SurvivorRatio pid # 查看年轻代中Eden区与Survivor区的比值
jinfo -flag MaxTenuringThreshold pid # 查看对象如果在Survivor区移动了N次还没有被垃圾回收就进入年老代
jinfo -flag UseSerialGC pid # 查看串行收集器
jinfo -flag UseParallelGC pid # 查看并行收集器
jinfo -flag UseParNewGC pid # 查看并行收集器
jinfo -flag UseParallelOldGC pid # 查看并行收集器
jinfo -flag UseConcMarkSweepGC pid # 查看CMS回收器
jinfo -flag UseG1GC pid # 查看G1回收器
jinfo -flag PrintGCDetails pid # 查看是否打印GC日志
```

##### jstat

主要是对java应用程序的资源和性能进行实时的命令行监控，包括了对heap size和垃圾回收状况的监控
```shell
jstat -class pid # 输出加载类的数量及所占空间信息。
jstat -gc pid # 输出gc信息，包括gc次数和时间，内存使用状况（可带时间和显示条目参数）
```
[你了解 Java 的 jstat 命令吗？](https://mp.weixin.qq.com/s/4vUmSsPoEI-MLVKc2NBvnw)

##### jhat

主要用来解析java堆dump并启动一个web服务器，然后就可以在浏览器中查看堆的dump文件了。
jhat 'heap-dump-file' 执行成功后显示
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
此时访问localhost:7000就可以看到结果了
或者使用Eclipse插件Memory Analyzer Tool打开dump文件

##### jcmd

在JDK 1.7之后，新增了一个命令行工具jcmd。它是一个多功能工具，可以用来导出堆，查看java进程，导出线程信息，执行GC等。jcmd拥有jmap的大部分功能，Oracle官方建议使用jcmd代替jmap。

##### jvm启动参数

**-Xms**:初始堆大小, 默认值:物理内存的1/64(<1GB)
默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.

**-Xmx**:最大堆大小, 默认值:物理内存的1/4(<1GB)
默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制

**-Xmn**:年轻代大小(1.4or lator)
此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的。整个堆大小=年轻代大小 + 老年代大小 + 持久代（永久代）大小.增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8

**-XX:NewSize**:年轻代大小(for 1.3/1.4)	

**-XX:MaxNewSize**:年轻代最大值(for 1.3/1.4)	

**-XX:PermSize**:设置持久代(perm gen)初始值, 默认值:物理内存的1/64	

**-XX:MaxPermSize**:设置持久带最大值

**注意：jdk8取消持久带用元空间代替**

**-XX:MetaspaceSize代替-XX:PermSize**

**-XX:MaxMetaspaceSize代替****-XX:MaxPermSize**

**-Xss**:每个线程的堆栈大小

JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行调整,在相同物理内存下,减小这个值能生成更多的线程,但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右一般小的应用,如果栈不是很深,应该是128k够用的大的应用建议使用256k.这个选项对性能影响比较大，需要严格的测试

**-XX:NewRatio**:年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代)	
-XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆栈的1/5Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置

**-XX:SurvivorRatio**:Eden区与Survivor区的大小比值	
-XX:SurvivorRatio**=**8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10

**-XX:MaxDirectMemorySize**：设置New I/O(java.nio) direct-buffer allocations的最大大小
当Direct ByteBuffer分配的堆外内存到达指定大小后，即触发Full GC。注意该值是有上限的，默认是64M，最大sun.misc.VM.maxDirectMemory()，在程序中中可以获得-XX:MaxDirectMemorySize的设置的值。

**-XX:+DisableExplicitGC**:关闭System.gc()	

**-XX:PretenureSizeThreshold**:对象超过多大是直接在旧生代分配	0
单位字节 新生代采用Parallel ScavengeGC时无效另一种直接在旧生代分配的情况是大的数组对象,且数组中无外部引用对象.

**-XX:ParallelGCThreads**:并行收集器的线程数	
此值最好配置与处理器数目相等 同样适用于CMS

**-XX:MaxGCPauseMillis**:每次年轻代垃圾回收的最长时间(最大暂停时间)
如果无法满足此时间,JVM会自动调整年轻代大小,以满足此值.

例：
```shell
java

  -Xms64m #JVM启动时的初始堆大小

  -Xmx128m #最大堆大小

  -Xmn64m #年轻代的大小，其余的空间是老年代

  -XX:MaxMetaspaceSize=128m #

  -XX:CompressedClassSpaceSize=64m #使用 -XX：CompressedClassSpaceSize 设置为压缩类空间保留的最大内存。

  -Xss256k #线程

  -XX:InitialCodeCacheSize=4m # 代码缓存区域设定值

  -XX:ReservedCodeCacheSize=8m # 这是由 JIT（即时）编译器编译为本地代码的本机代码（如JNI）或 Java 方法的空间  

  -XX:MaxDirectMemorySize=16m

  -XX:NativeMemoryTracking=summary #开启内存追踪

  -jar app.jar
  
```

[常用的J?VM参数，你现在就记好！](https://mp.weixin.qq.com/s/3yvA2m66--Nxsf_xQDJZog)

#### 内存泄漏、内存溢出

- 内存泄漏:是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄漏似乎不会有大的影响，但内存泄漏堆积后的后果就是内存溢出。 

- 内存溢出:指程序申请内存时，没有足够的内存供申请者使用