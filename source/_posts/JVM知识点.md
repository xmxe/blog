---
title: JVM知识点
categories: Java
index_img: /assert/jvm.jpg
img: https://pic1.zhimg.com/v2-716588f96320420369eb5a5734852aa5_1440w.jpg
coverImg: https://pic1.zhimg.com/v2-7382fb7654c33a5e843f51ec139b1da0_r.jpg
cover: true
summary: JVM内存区域、垃圾回收、调优、JVM工具、参数等相关命令
top: true

---

## JVM内存区域

- Java虚拟机栈（栈区）
- 本地方法栈
- Java堆（堆区）
- 方法区
- 程序计数器

Java虚拟机在执行Java程序的过程中会把它管理的内存划分成若干个不同的数据区域。JDK1.8和之前的版本略有不同，下面会介绍到。

**JDK1.8之前**：

![Java运行时数据区域（JDK1.8之前）](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/jvm/java-runtime-data-areas-jdk1.7.png)

**JDK1.8之后**：

![Java运行时数据区域（JDK1.8之后）](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/jvm/java-runtime-data-areas-jdk1.8.png)

**线程私有的**：

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的**：

- 堆
- 方法区
- 直接内存(非运行时数据区的一部分)


Java虚拟机规范对于运行时数据区域的规定是相当宽松的。以堆为例：堆可以是连续空间，也可以不连续。堆的大小可以固定，也可以在运行时按需扩展。虚拟机实现者可以使用任何垃圾回收算法管理堆，甚至完全不进行垃圾收集也是可以的


### Java堆

GC堆是java虚拟机所管理的内存中最大的一块内存区域，也是被各个线程共享的内存区域，在JVM启动时创建。其大小通过-Xms(最小值)和-Xmx(最大值)参数设置，-Xms为JVM启动时申请的最小内存，-Xmx为JVM可申请的最大内存。由于现在收集器都是采用分代收集算法，堆被划分为新生代和老年代。新生代可通过-Xmn参数来指定新生代的大小。对象刚创建的时候，会被创建在新生代，到一定阶段之后会移送至老年代，如果创建了一个新生代无法容纳的新对象，那么这个新对象也可以创建到老年代
**所有对象实例以及数组都在堆上分配**。堆内存用来存放由new创建的对象实例和数组,堆中不存放基本数据类型和对象引用，只存放对象本身(包括属性,即成员变量),jvm只有一个堆区(heap)被所有线程共享

- **新生代(Young Gen)** ：新生代主要存放新创建的对象，内存大小相对会比较小，垃圾回收会比较频繁。新生代分为1个Eden区和2个S区，S代表Survivor。当对象在堆创建时，将进入Eden Space。垃圾回收器进行垃圾回收时，它的策略是会把没有引用的对象直接给回收掉，还有引用的对象会被移送到Survivor区。Survivor区有S0和S1两个内存空间，每次进行YGC的时候，会将存活的对象复制到未使用的那块内存空间，然后将当前正在使用的空间完全清除掉，再交换两个空间的使用状况。如果YGC要移送的对象Survivor区无法容纳，那么就会将该对象直接移交给老年代。上面说了，到一定阶段的对象会移送到老年区，这是什么意思呢？每一个对象都有一个计数器，当每次进行YGC的时候，都会+1。通过-XX:MAXTenuringThrehold参数可以配置当计数器的值到达某个阈值时，对象就会从新生代移送至老年代。该参数的默认值为15，也就是说对象在Survivor区中的S0和S1内存空间交换的次数累加到15次之后，就会移送至老年代。如果参数配置为1，那么创建的对象就会直接移送至老年代。扫描完毕后，JVM将Eden Space和A Suvivor Space清空，然后交换A和B的角色，即下次垃圾回收时会扫描Eden Space和B Suvivor Space。这么做主要是为了减少内存碎片的产生。我们可以看到：Young Gen垃圾回收时，采用将存活对象复制到到空的Suvivor Space的方式来确保尽量不存在内存碎片，采用空间换时间的方式来加速内存中不再被持有的对象尽快能够得到回收。
- **老年代(Tenured Gen)** ：老年代主要存放JVM认为生命周期比较长的对象（经过几次的Young Gen的垃圾回收后仍然存在），内存大小相对会比较大，垃圾回收也相对没有那么频繁（譬如可能几个小时一次）。老年代主要采用压缩的方式来避免内存碎片（将存活对象移动到内存片的一边，也就是内存整理）。当然，有些垃圾回收器（譬如CMS垃圾回收器）出于效率的原因，可能会不进行压缩。


堆是Java虚拟机所管理的内存中最大的一块，Java堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存**。

Java世界中“几乎”所有的对象都在堆中分配，但是，随着JIT编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。从JDK1.7开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存。

Java堆是垃圾收集器管理的主要区域，因此也被称作**GC堆（Garbage Collected Heap）**。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以Java堆还可以细分为：新生代和老年代；再细致一点有：Eden、Survivor、Old等空间。进一步划分的目的是更好地回收内存，或者更快地分配内存。

在JDK7版本及JDK7版本之前，堆内存被通常分为下面三部分：

1. 新生代内存(Young Generation)
2. 老生代(Old Generation)
3. 永久代(Permanent Generation)

Eden区、两个Survivor区S0和S1都属于新生代，中间一层属于老年代，最下面一层属于永久代。

**JDK8版本之后PermGen(永久)已被Metaspace(元空间)取代，元空间使用的是本地内存**。

大部分情况，对象都会首先在Eden区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入S0或者S1，并且对象的年龄还会加1(Eden区->Survivor区后对象的初始年龄变为1)，当它的年龄增加到一定程度（默认为15岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数-XX:MaxTenuringThreshold来设置。

> **🐛修正（参见：[issue552](https://github.com/Snailclimb/JavaGuide/issues/552)）**：“Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了survivor区的一半时，取这个年龄和MaxTenuringThreshold中更小的一个值，作为新的晋升年龄阈值”。
>
> **动态年龄计算的代码如下**
>
> ```c++
>uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> 	//survivor_capacity是survivor空间的大小
> size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
> size_t total = 0;
> uint age = 1;
> while (age < table_size) {
> total += sizes[age];//sizes数组是每个年龄段对象大小
> if (total > desired_survivor_size) break;
> age++;
> }
> uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> 	...
> }
> ```

堆这里最容易出现的就是OutOfMemoryError错误，并且出现这种错误之后的表现形式还会有几种，比如：

1. **java.lang.OutOfMemoryError: GC Overhead Limit Exceeded**：当JVM花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误。
2. **java.lang.OutOfMemoryError: Java heap space**：假如在创建新的对象时,堆内存中的空间不足以存放新创建的对象,就会引发此错误。(和配置的最大堆内存有关，且受制于物理内存大小。最大堆内存可通过`-Xmx`参数配置，若没有特别配置，将会使用默认值，详见：[Default Java 8 max heap size](https://stackoverflow.com/questions/28272923/default-xmxsize-in-java-8-max-heap-size))

### Java虚拟机栈

每个线程包含一个栈区，栈中只保存基础数据类型的对象和自定义对象的引用(不是对象)，对象都存放在堆区中,每个栈中的数据(原始类型和对象引用)都是私有的，其他栈不能访问。定义的局部变量也在栈内存中,线程私有,FILO(先进后出)

与程序计数器一样，Java虚拟机栈也是线程私有的，它的生命周期与线程相同,随着线程的创建而创建，随着线程的死亡而死亡。栈绝对算的上是JVM运行时数据区域的一个核心，除了一些Native方法调用是通过本地方法栈实现的(后面会提到)，其他所有的Java方法调用都是通过栈来实现的（也需要和其他运行时数据区域比如程序计数器配合）。每个方法被执行的时候都会创建一个"栈帧",用于存储局部变量表(包括参数)、操作数栈、动态链接、方法出口等信息。每个方法被调用到执行完的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。局部变量表存放各种基本数据类型boolean、byte、char、short等。方法调用的数据需要通过栈进行传递，每一次方法调用都会有一个对应的栈帧被压入栈中，每一个方法调用结束后，都会有一个栈帧被弹出。

栈由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法返回地址。和数据结构上的栈类似，两者都是先进后出的数据结构，只支持出栈和入栈两种操作。

![Java虚拟机栈](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/jvm/stack-area.png)

**局部变量表**主要存放了编译期可知的各种数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

![局部变量表](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/jvm/local-variables-table.png)

**操作数栈**主要作为方法调用的中转站使用，用于存放方法执行过程中产生的中间计算结果。另外，计算过程中产生的临时变量也会放在操作数栈中。

**动态链接**主要服务一个方法需要调用其他方法的场景。Class 文件的常量池里保存有大量的符号引用比如方法引用的符号引用。当一个方法要调用其他方法，需要将常量池中指向方法的符号引用转化为其在内存地址中的直接引用。动态链接的作用就是为了将符号引用转换为调用方法的直接引用，这个过程也被称为**动态连接**。

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/jvmimage-20220331175738692.png)

栈空间虽然不是无限的，但一般正常调用的情况下是不会出现问题的。不过，如果函数调用陷入无限循环的话，就会导致栈中被压入太多栈帧而占用太多空间，导致栈空间过深。那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度的时候，就抛出StackOverFlowError错误。

Java方法有两种返回方式，一种是return语句正常返回，一种是抛出异常。不管哪种返回方式，都会导致栈帧被弹出。也就是说，**栈帧随着方法调用而创建，随着方法结束而销毁。无论方法正常完成还是异常完成都算作方法结束**。

除了StackOverFlowError错误之外，栈还可能会出现OutOfMemoryError错误，这是因为如果栈的内存大小可以动态扩展，如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常。

简单总结一下程序运行中栈可能会出现两种错误：

- **StackOverFlowError**：若栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度的时候，就抛出StackOverFlowError错误。
- **OutOfMemoryError**：如果栈的内存大小可以动态扩展，如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常。

### 本地方法栈

线程私有FILO(先进后出)
和虚拟机栈所发挥的作用非常相似，区别是：**虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务**。在HotSpot虚拟机中和Java虚拟机栈合二为一。本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现StackOverFlowError和OutOfMemoryError两种错误。


### 方法区
jdk1.7及以前方法区也被称为永久代，1.8之后移除，取而代之的为metaspace元空间,注意：元空间，永久代都是方法区的一种实现,永久代主要存放类定义、字节码和常量等很少会变更的信息。永久带是方法区的一种实现，可以理解为就是方法区，类似于java的接口和实现类

方法区是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、静态变量、final类型的常量、属性和方法信息，即时编译器编译后的代码等数据,也称”永久代”，它用于存储虚拟机加载的类信息、常量、静态变量、是各个线程共享的内存区域。可以通过-XX:PermSize和-XX:MaxPermSize参数限制方法区的大小。

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


方法区属于是JVM运行时数据区域的一块逻辑区域，是各个线程共享的内存区域。《Java虚拟机规范》只是规定了有方法区这么个概念和它的作用，方法区到底要如何实现那就是虚拟机自己要考虑的事情了。也就是说，在不同的虚拟机实现上，方法区的实现是不同的。当虚拟机要使用一个类时，它需要读取并解析Class文件获取相关信息，再将信息存入到方法区。方法区会存储已被虚拟机加载的**类信息、字段信息、方法信息、常量、静态变量、即时编译器编译后的代码缓存等数据**。

**方法区和永久代以及元空间是什么关系呢**？方法区和永久代以及元空间的关系很像Java中接口和类的关系，类实现了接口，这里的类就可以看作是永久代和元空间，接口可以看作是方法区，也就是说永久代以及元空间是HotSpot虚拟机对虚拟机规范中方法区的两种实现方式。并且，永久代是JDK1.8之前的方法区实现，JDK1.8及以后方法区的实现变成了元空间。

![HotSpot虚拟机方法区的两种实现](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/jvm/method-area-implementation.png)

**为什么要将永久代(PermGen)替换为元空间(MetaSpace)呢?**

下图来自《深入理解Java虚拟机》第3版2.2.5

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/jvm/20210425134508117.png)

1、整个永久代有一个JVM本身设置的固定大小上限，无法进行调整，而元空间使用的是本地内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

> 当元空间溢出时会得到如下错误：java.lang.OutOfMemoryError:MetaSpace

你可以使用`-XX：MaxMetaspaceSize`标志设置最大元空间大小，默认值为unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize`调整标志定义元空间的初始大小如果未指定此标志，则Metaspace将根据运行时的应用程序需求动态地重新调整大小。

2、元空间里面存放的是类的元数据，这样加载多少类的元数据就不由`MaxPermSize`控制了,而由系统的实际可用空间来控制，这样能加载的类就更多了。

3、在JDK8，合并HotSpot和JRockit的代码时,JRockit从来没有一个叫永久代的东西,合并之后就没有必要额外的设置这么一个永久代的地方了。

**方法区常用参数有哪些？**

JDK1.8之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小。


```java
-XX:PermSize=N //方法区(永久代)初始大小
-XX:MaxPermSize=N //方法区(永久代)最大大小,超过这个值将会抛出OutOfMemoryError异常:java.lang.OutOfMemoryError:PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

JDK1.8的时候，方法区（HotSpot的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是本地内存。下面是一些常用参数：

```java
-XX:MetaspaceSize=N //设置Metaspace的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置Metaspace的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

#### 运行时常量池

Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有用于存放编译期生成的各种字面量（Literal）和符号引用（Symbolic Reference）的常量池表(Constant Pool Table)。

字面量是源代码中的固定值的表示法，即通过字面我们就能知道其值的含义。字面量包括整数、浮点数和字符串字面量。常见的符号引用包括类符号引用、字段符号引用、方法符号引用、接口方法符号。

《深入理解Java虚拟机》7.34节第三版对符号引用和直接引用的解释如下：

![符号引用和直接引用](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/jvm/symbol-reference-and-direct-reference.png)

常量池表会在类加载后存放到方法区的运行时常量池中。运行时常量池的功能类似于传统编程语言的符号表，尽管它包含了比典型符号表更广泛的数据。既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出`OutOfMemoryError`错误。

#### 字符串常量池

**字符串常量池**是JVM为了提升性能和减少内存消耗针对字符串（String类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

```java
// 在堆中创建字符串对象”ab“
// 将字符串对象”ab“的引用保存在字符串常量池中
String aa = "ab";
// 直接返回字符串常量池中字符串对象”ab“的引用
String bb = "ab";
System.out.println(aa==bb);// true
```

HotSpot虚拟机中字符串常量池的实现是`src/hotspot/share/classfile/stringTable.cpp`,`StringTable`本质上就是一个`HashSet<String>`,容量为`StringTableSize`（可以通过`-XX:StringTableSize`参数来设置）。

**StringTable中保存的是字符串对象的引用，字符串对象的引用指向堆中的字符串对象**。

JDK1.7之前，字符串常量池存放在永久代。JDK1.7字符串常量池和静态变量从永久代移动了Java堆中。

![method-area-jdk1.6](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/jvm/method-area-jdk1.6.png)

![method-area-jdk1.7](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/jvm/method-area-jdk1.7.png)

**JDK 1.7为什么要将字符串常量池移动到堆中？**

主要是因为永久代（方法区实现）的GC回收效率太低，只有在整堆收集(Full GC)的时候才会被执行GC。Java程序中通常会有大量的被创建的字符串等待回收，将字符串常量池放到堆中，能够更高效及时地回收字符串内存。

> 相关问题：[JVM常量池中存储的是对象还是引用呢？](https://www.zhihu.com/question/57109429/answer/151717241)
> 最后再来分享一段周志明老师在[《深入理解Java虚拟机（第3版）》样例代码&勘误](https://github.com/fenixsoft/jvm_book)Github仓库的[issue#112](https://github.com/fenixsoft/jvm_book/issues/112)中说过的话：
> **运行时常量池、方法区、字符串常量池这些都是不随虚拟机实现而改变的逻辑概念，是公共且抽象的，Metaspace、Heap是与具体某种虚拟机实现相关的物理概念，是私有且具体的。**

### 程序计数器

存放下一条指令所在单元地址的地方
程序计数器是一块较小的内存空间，可以看作当前线程所执行的字节码的行号指示器。在虚拟机的模型里，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、异常处理、线程恢复等基础功能都需要依赖计数器完成。另外，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。

从上面的介绍中我们知道了程序计数器主要有两个作用：

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
- 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

⚠️ 注意：程序计数器是唯一一个不会出现OutOfMemoryError的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡

### 直接内存

直接内存是一种特殊的内存缓冲区，并不在Java堆或方法区中分配的，而是通过JNI的方式在本地内存上分配的。

直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致`OutOfMemoryError`错误出现。

JDK1.4中新加入的**NIO(New Input/Output)类**，引入了一种基于通道（Channel）与缓存区（Buffer）的I/O方式，它可以直接使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为避免了在Java堆和Native堆之间来回复制数据。

直接内存的分配不会受到Java堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

类似的概念还有**堆外内存**。在一些文章中将直接内存等价于堆外内，个人觉得不是特别准确。

堆外内存就是把内存对象分配在堆（新生代+老年代+永久代）以外的内存，这些内存直接受操作系统管理（而不是虚拟机），这样做的结果就是能够在一定程度上减少垃圾回收对应用程序造成的影响

### 示例

```java
Foo foo = new Foo();
foo.f();
```
以上代码的内存实现原理为：
1. Foo类首先被装载到JVM的方法区，其中包括类的信息，包括方法和构造等。
2. 在栈内存中分配引用变量foo。
3. 在堆内存中按照Foo类型信息分配实例变量内存空间；然后，将栈中引用foo指向foo对象堆内存的首地址。
4. 使用引用foo调用方法，根据foo引用的类型Foo调用f方法。

![](/images/jvmdemo.png)
![](/images/jvmdemo2.jpg)

> [JVM底层原理最全知识总结--GitHub](https://github.com/doocs/jvm)


### 相关文章

- [2万字长文包教包会JVM内存结构](https://mp.weixin.qq.com/s/VDZNpS4Qk0jvv_MctVXhww)
- [图文并茂，傻瓜都能看懂的JVM内存布局](https://mp.weixin.qq.com/s/oDeO8Td-SJn9g4mRygqtSw)
- [小白都能看得懂的java虚拟机内存模型](https://mp.weixin.qq.com/s/m2dp6jv8lfmy-S2gmQDHvA)
- [深入理解堆外内存Metaspace](https://mp.weixin.qq.com/s/xB-uiqy4eVsNovEknO9_5w)
- [求你了，别再说Java对象都是在堆内存上分配空间的了](https://mp.weixin.qq.com/s/Owlhu5IFpDAyu0WYcK1EhQ)
- [终于搞懂了Java 8的内存结构，再也不纠结方法区和常量池了！！](https://mp.weixin.qq.com/s/8uGOt1OJloZMHQl43vrbyQ)
- [个人笔记，深入理解JVM，很全！](https://mp.weixin.qq.com/s/1rxmi3lg02I7nHwWbldvLA)
- [JVM内存布局详解，图文并茂，写得太好了！](https://mp.weixin.qq.com/s/RySwVg9SYvSM-ONdB9cOVw)

## JVM垃圾回收

> 常见面试题：
> - 如何判断对象是否死亡（两种方法）。
> - 简单的介绍一下强引用、软引用、弱引用、虚引用（虚引用与软引用和弱引用的区别、使用软引用能带来的好处）。
> - 如何判断一个常量是废弃常量
> - 如何判断一个类是无用的类
> - 垃圾收集有哪些算法，各自的特点？
> - HotSpot为什么要分为新生代和老年代？
> - 常见的垃圾回收器有哪些？
> - 介绍一下CMS,G1收集器。
> - MinorGc和FullGC有什么不同呢？

### 堆空间的基本结构

Java的自动内存管理主要是针对对象内存的回收和对象内存的分配。同时，Java自动内存管理最核心的功能是堆内存中对象的分配与回收。

Java堆是垃圾收集器管理的主要区域，因此也被称作**GC堆（Garbage Collected Heap）**。

从垃圾回收的角度来说，由于现在收集器基本都采用分代垃圾收集算法，所以Java堆被划分为了几个不同的区域，这样我们就可以根据各个区域的特点选择合适的垃圾收集算法。

在JDK7版本及JDK7版本之前，堆内存被通常分为下面三部分：

1. 新生代内存(Young Generation)
2. 老生代(Old Generation)
3. 永久代(Permanent Generation)

Eden区、两个Survivor区S0和S1都属于新生代，中间一层属于老年代，最下面一层属于永久代。

**JDK8版本之后PermGen(永久)已被Metaspace(元空间)取代，元空间使用的是直接内存**。

> 关于堆空间结构更详细的介绍，可以回过头看看[Java内存区域详解](https://javaguide.cn/java/jvm/memory-area.html)这篇文章。

### 内存分配和回收原则

#### 对象优先在Eden区分配

大多数情况下，对象在新生代中Eden区分配。当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC。下面我们来进行实际测试以下。

测试代码：

```java
public class GCTest {
	public static void main(String[] args) {
		byte[] allocation1, allocation2;
		allocation1 = new byte[30900*1024];
	}
}
```


运行结果(红色字体描述有误，应该是对应于JDK1.7的永久代)：

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-8-26/28954286.jpg)

从上图我们可以看出Eden区内存几乎已经被分配完全（即使程序什么也不做，新生代也会使用2000多k内存）。假如我们再为allocation2分配内存会出现什么情况呢？


```java
allocation2 = new byte[900*1024];
```

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-8-26/28128785.jpg)

给allocation2分配内存的时候Eden区内存几乎已经被分配完了

当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC。GC期间虚拟机又发现allocation1无法存入Survivor空间，所以只好通过**分配担保机制**把新生代的对象提前转移到老年代中去，老年代上的空间足够存放allocation1，所以不会出现Full GC。执行Minor GC后，后面分配的对象如果能够存在Eden区的话，还是会在Eden区分配内存。可以执行如下代码验证：

```java
public class GCTest {
	public static void main(String[] args) {
		byte[] allocation1, allocation2,allocation3,allocation4,allocation5;
		allocation1 = new byte[32000*1024];
		allocation2 = new byte[1000*1024];
		allocation3 = new byte[1000*1024];
		allocation4 = new byte[1000*1024];
		allocation5 = new byte[1000*1024];
	}
}
```

#### 大对象直接进入老年代

大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。大对象直接进入老年代主要是为了避免为大对象分配内存时由于分配担保机制带来的复制而降低效率。

#### 长期存活的对象将进入老年代

既然虚拟机采用了分代收集的思想来管理内存，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了做到这一点，虚拟机给每个对象一个对象年龄（Age）计数器。

大部分情况，对象都会首先在Eden区域分配。如果对象在Eden出生并经过第一次Minor GC后仍然能够存活，并且能被Survivor容纳的话，将被移动到Survivor空间（s0或者s1）中，并将对象年龄设为1(Eden区->Survivor区后对象的初始年龄变为1)。

对象在Survivor中每熬过一次MinorGC,年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数`-XX:MaxTenuringThreshold`来设置。

> 修正（[issue552](https://github.com/Snailclimb/JavaGuide/issues/552)）：“Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了survivor区的50%时（默认值是50%，可以通过`-XX:TargetSurvivorRatio=percent`来设置，参见[issue1199](https://github.com/Snailclimb/JavaGuide/issues/1199)），取这个年龄和MaxTenuringThreshold中更小的一个值，作为新的晋升年龄阈值”。
>
> jdk8官方文档引用：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html。
>
> ![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/java-guide-blog/image-20210523201742303.png)
>
> **动态年龄计算的代码如下：**
>
>
> ```c++
> uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> //survivor_capacity是survivor空间的大小
> size_t desired_survivor_size = (size_t)((((double)survivor_capacity)*TargetSurvivorRatio)/100);
> size_t total = 0;
> uint age = 1;
> while (age < table_size) {
> //sizes数组是每个年龄段对象大小
> total += sizes[age];
> if (total > desired_survivor_size) {
> break;
> }
> age++;
> }
> uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> ...
> }
> ```
>
> 额外补充说明([issue672](https://github.com/Snailclimb/JavaGuide/issues/672))：**关于默认的晋升年龄是15，这个说法的来源大部分都是《深入理解Java虚拟机》这本书**。如果你去Oracle的官网阅读[相关的虚拟机参数](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)，你会发现`-XX:MaxTenuringThreshold=threshold`这里有个说明
>
> **Sets the maximum tenuring threshold for use in adaptive GC sizing. The largest value is 15. The default value is 15 for the parallel (throughput) collector, and 6 for the CMS collector.默认晋升年龄并不都是15，这个是要区分垃圾收集器的，CMS就是6.**

#### 主要进行gc的区域

周志明先生在《深入理解Java虚拟机》第二版中P92如是写道：

> ~~*“老年代GC（Major GC/Full GC），指发生在老年代的GC……”*~~
> 上面的说法已经在《深入理解Java虚拟机》第三版中被改正过来了。

**总结：**

针对HotSpot VM的实现，它里面的GC其实准确分类只有两大种：

部分收集(Partial GC)：

- 新生代收集（Minor GC/Young GC）：只对新生代进行垃圾收集；
- 老年代收集（Major GC/Old GC）：只对老年代进行垃圾收集。需要注意的是Major GC在有的语境中也用于指代整堆收集；
- 混合收集（Mixed GC）：对整个新生代和部分老年代进行垃圾收集。

整堆收集(Full GC)：收集整个Java堆和方法区。

#### 空间分配担保

空间分配担保是为了确保在Minor GC之前老年代本身还有容纳新生代所有对象的剩余空间。《深入理解Java虚拟机》第三章对于空间分配担保的描述如下：

> JDK6 Update 24之前，在发生Minor GC之前，虚拟机必须先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那这一次Minor GC可以确保是安全的。如果不成立，则虚拟机会先查看`-XX:HandlePromotionFailure`参数的设置值是否允许担保失败(Handle Promotion Failure);如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试进行一次Minor GC，尽管这次Minor GC是有风险的;如果小于，或者`-XX:HandlePromotionFailure`设置不允许冒险，那这时就要改为进行一次Full GC。
>
> JDK6 Update 24之后的规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小，就会进行Minor GC，否则将进行Full GC。

### 死亡对象判断方法

堆中几乎放着所有的对象实例，对堆垃圾回收前的第一步就是要判断哪些对象已经死亡（即不能再被任何途径使用的对象）。

#### 引用计数法

给对象中添加一个引用计数器：

- 每当有一个地方引用它，计数器就加1；
- 当引用失效，计数器就减1；
- 任何时候计数器为0的对象就是不可能再被使用的。

**这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题**。

所谓对象之间的相互引用问题，如下面代码所示：除了对象`objA`和`objB`相互引用着对方之外，这两个对象之间再无任何引用。但是他们因为互相引用对方，导致它们的引用计数器都不为0，于是引用计数算法无法通知GC回收器回收他们。

```java
public class ReferenceCountingGc {
    Object instance = null;
    public static void main(String[] args) {
        ReferenceCountingGc objA = new ReferenceCountingGc();
        ReferenceCountingGc objB = new ReferenceCountingGc();
        objA.instance = objB;
        objB.instance = objA;
        objA = null;
        objB = null;
    }
}
```

#### 可达性分析算法

这个算法的基本思想就是通过一系列的称为GC Roots的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连的话，则证明此对象是不可用的，需要被回收。下图中的Object 6~Object 10之间虽有引用关系，但它们到GC Roots不可达，因此为需要被回收的对象。

**哪些对象可以作为GC Roots呢？**

- 虚拟机栈(栈帧中的本地变量表)中引用的对象
- 本地方法栈(Native方法)中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 所有被同步锁持有的对象

**对象可以被回收，就代表一定会被回收吗**？

即使在可达性分析法中不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑阶段”，要真正宣告一个对象死亡，至少要经历两次标记过程；可达性分析法中不可达的对象被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行`finalize`方法。当对象没有覆盖`finalize`方法，或`finalize`方法已经被虚拟机调用过时，虚拟机将这两种情况视为没有必要执行。

被判定为需要执行的对象将会被放在一个队列中进行第二次标记，除非这个对象与引用链上的任何一个对象建立关联，否则就会被真的回收。

> Object类中的finalize方法一直被认为是一个糟糕的设计，成为了Java语言的负担，影响了Java语言的安全和GC的性能。JDK9版本及后续版本中各个类中的finalize方法会被逐渐弃用移除。忘掉它的存在吧！
>
> 参考：
>
> - [JEP 421: Deprecate Finalization for Removal](https://openjdk.java.net/jeps/421)
> - [是时候忘掉finalize方法了](https://mp.weixin.qq.com/s/LW-paZAMD08DP_3-XCUxmg)

#### 引用类型总结

无论是通过引用计数法判断对象引用数量，还是通过可达性分析法判断对象的引用链是否可达，判定对象的存活都与“引用”有关。JDK1.2之前，Java中引用的定义很传统：如果reference类型的数据存储的数值代表的是另一块内存的起始地址，就称这块内存代表一个引用。JDK1.2以后，Java对引用的概念进行了扩充，将引用分为强引用、软引用、弱引用、虚引用四种（引用强度逐渐减弱）

**1．强引用（StrongReference）**

以前我们使用的大部分引用实际上都是强引用，这是使用最普遍的引用。如果一个对象具有强引用，那就类似于**必不可少的生活用品**，垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。

**2．软引用（SoftReference）**

如果一个对象只具有软引用，那就类似于**可有可无的生活用品**。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，JAVA虚拟机就会把这个软引用加入到与之关联的引用队列中。

**3．弱引用（WeakReference）**

如果一个对象只具有弱引用，那就类似于**可有可无的生活用品**。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象。

弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。

**4．虚引用（PhantomReference）**

"虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收。

**虚引用主要用来跟踪对象被垃圾回收的活动**。

**虚引用与软引用和弱引用的一个区别在于**：虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

特别注意，在程序设计中一般很少使用弱引用与虚引用，使用软引用的情况较多，这是因为**软引用可以加速JVM对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出（OutOfMemory）等问题的产生**。

#### 如何判断一个常量是废弃常量？

运行时常量池主要回收的是废弃的常量。那么，我们如何判断一个常量是废弃常量呢？

**JDK1.7及之后版本的JVM已经将运行时常量池从方法区中移了出来，在Java堆（Heap）中开辟了一块区域存放运行时常量池**。

> **🐛修正（参见：[issue747](https://github.com/Snailclimb/JavaGuide/issues/747)，[reference](https://blog.csdn.net/q5706503/article/details/84640762)）**：
>
> 1. **JDK1.7之前运行时常量池逻辑包含字符串常量池存放在方法区,此时hotspot虚拟机对方法区的实现为永久代**
> 2. **JDK1.7字符串常量池被从方法区拿到了堆中,这里没有提到运行时常量池,也就是说字符串常量池被单独拿到堆,运行时常量池剩下的东西还在方法区,也就是hotspot中的永久代**。
> 3. **JDK1.8hotspot移除了永久代用元空间(Metaspace)取而代之,这时候字符串常量池还在堆,运行时常量池还在方法区,只不过方法区的实现从永久代变成了元空间(Metaspace)**

假如在字符串常量池中存在字符串"abc"，如果当前没有任何String对象引用该字符串常量的话，就说明常量"abc"就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc"就会被系统清理出常量池了。

#### 如何判断一个类是无用的类

方法区主要回收的是无用的类，那么如何判断一个类是无用的类的呢？

判定一个常量是否是“废弃常量”比较简单，而要判定一个类是否是“无用的类”的条件则相对苛刻许多。类需要同时满足下面3个条件才能算是无用的类：

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例。
- 加载该类的ClassLoader已经被回收。
- 该类对应的`java.lang.Class`对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

虚拟机可以对满足上述3个条件的无用类进行回收，这里说的仅仅是“可以”，而并不是和对象一样不使用了就会必然被回收。

### 垃圾收集算法

#### 标记-清除算法

该算法分为“标记”和“清除”阶段：首先标记出所有不需要回收的对象，在标记完成后统一回收掉所有没有被标记的对象。它是最基础的收集算法，后续的算法都是对其不足进行改进得到。这种垃圾收集算法会带来两个明显的问题：

1. **效率问题**
2. **空间问题（标记清除后会产生大量不连续的碎片）**

#### 标记-复制算法

为了解决效率问题，“标记-复制”收集算法出现了。它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

#### 标记-整理算法

根据老年代的特点提出的一种标记算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。

#### 分代收集算法

当前虚拟机的垃圾收集都采用分代收集算法，这种算法没有什么新的思想，只是根据对象存活周期的不同将内存分为几块。一般将java堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

比如在新生代中，每次收集都会有大量对象死去，所以可以选择”标记-复制“算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清除”或“标记-整理”算法进行垃圾收集。

> **延伸面试问题**：HotSpot为什么要分为新生代和老年代？
> 根据上面的对分代收集算法的介绍回答。

### 垃圾收集器

**如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现**。

虽然我们对各个收集器进行比较，但并非要挑选出一个最好的收集器。因为直到现在为止还没有最好的垃圾收集器出现，更加没有万能的垃圾收集器，我们能做的就是根据具体应用场景选择适合自己的垃圾收集器。试想一下：如果有一种四海之内、任何场景下都适用的完美收集器存在，那么我们的HotSpot虚拟机就不会实现那么多不同的垃圾收集器了。

#### Serial收集器

Serial（串行）收集器是最基本、历史最悠久的垃圾收集器了。大家看名字就知道这个收集器是一个单线程收集器了。它的单线程的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（Stop The World），直到它收集结束。

**新生代采用标记-复制算法，老年代采用标记-整理算法**。

虚拟机的设计者们当然知道Stop The World带来的不良用户体验，所以在后续的垃圾收集器设计中停顿时间在不断缩短（仍然还有停顿，寻找最优秀的垃圾收集器的过程仍然在继续）。

但是Serial收集器有没有优于其他垃圾收集器的地方呢？当然有，它简单而高效（与其他收集器的单线程相比）。Serial收集器由于没有线程交互的开销，自然可以获得很高的单线程收集效率。Serial收集器对于运行在Client模式下的虚拟机来说是个不错的选择。

#### ParNew收集器

ParNew收集器其实就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和Serial收集器完全一样。

新生代采用标记-复制算法，老年代采用标记-整理算法。

它是许多运行在Server模式下的虚拟机的首要选择，除了Serial收集器外，只有它能与CMS收集器（真正意义上的并发收集器，后面会介绍到）配合工作。

并行和并发概念补充：

- **并行（Parallel）**：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。
- **并发（Concurrent）**：指用户线程与垃圾收集线程同时执行（但不一定是并行，可能会交替执行），用户程序在继续运行，而垃圾收集器运行在另一个CPU上。

#### Parallel Scavenge收集器

Parallel Scavenge收集器也是使用标记-复制算法的多线程收集器，它看上去几乎和ParNew都一样。那么它有什么特别之处呢？


```text
-XX:+UseParallelGC 使用Parallel收集器+老年代串行

-XX:+UseParallelOldGC 使用Parallel收集器+老年代并行
```

Parallel Scavenge收集器关注点是吞吐量（高效率的利用CPU）。CMS等垃圾收集器的关注点更多的是用户线程的停顿时间（提高用户体验）。所谓吞吐量就是CPU中用于运行用户代码的时间与CPU总消耗时间的比值。Parallel Scavenge收集器提供了很多参数供用户找到最合适的停顿时间或最大吞吐量，如果对于收集器运作不太了解，手工优化存在困难的时候，使用Parallel Scavenge收集器配合自适应调节策略，把内存管理优化交给虚拟机去完成也是一个不错的选择。

使用`java -XX:+PrintCommandLineFlags -version`命令查看


```bash
-XX:InitialHeapSize=262921408 -XX:MaxHeapSize=4206742528 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
```

JDK1.8默认使用的是Parallel Scavenge+Parallel Old，如果指定了-XX:+UseParallelGC参数，则默认指定了-XX:+UseParallelOldGC，可以使用-XX:-UseParallelOldGC来禁用该功能

#### SerialOld收集器

Serial收集器的老年代版本，它同样是一个单线程收集器。它主要有两大用途：一种用途是在JDK1.5以及以前的版本中与Parallel Scavenge收集器搭配使用，另一种用途是作为CMS收集器的后备方案。

#### Parallel Old收集器

Parallel Scavenge收集器的老年代版本。使用多线程和“标记-整理”算法。在注重吞吐量以及CPU资源的场合，都可以优先考虑Parallel Scavenge收集器和Parallel Old收集器。

#### CMS收集器

CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。它非常符合在注重用户体验的应用上使用。CMS（Concurrent Mark Sweep）收集器是HotSpot虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。从名字中的Mark Sweep这两个词可以看出，CMS收集器是一种标记-清除算法实现的，它的运作过程相比于前面几种垃圾收集器来说更加复杂一些。整个过程分为四个步骤：

- **初始标记**:暂停所有的其他线程，并记录下直接与root相连的对象，速度很快；
- **并发标记**:同时开启GC和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以GC线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
- **重新标记**:重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
- **并发清除**:开启用户线程，同时GC线程开始对未标记的区域做清扫。

从它的名字就可以看出它是一款优秀的垃圾收集器，主要优点：**并发收集、低停顿**。但是它有下面三个明显的缺点：

- **对CPU资源敏感；**
- **无法处理浮动垃圾；**
- **它使用的回收算法-“标记-清除”算法会导致收集结束时会有大量空间碎片产生。**

#### G1收集器

**G1(Garbage-First)是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器.以极高概率满足GC停顿时间要求的同时,还具备高吞吐量性能特征**。

被视为JDK1.7中HotSpot虚拟机的一个重要进化特征。它具备以下特点：

- **并行与并发**：G1能充分利用CPU、多核环境下的硬件优势，使用多个CPU（CPU或者CPU核心）来缩短Stop-The-World停顿时间。部分其他收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方式让java程序继续执行。
- **分代收集**：虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但是还是保留了分代的概念。
- **空间整合**：与CMS的“标记-清除”算法不同，G1从整体来看是基于“标记-整理”算法实现的收集器；从局部上来看是基于“标记-复制”算法实现的。
- **可预测的停顿**：这是G1相对于CMS的另一个大优势，降低停顿时间是G1和CMS共同的关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内。

G1收集器的运作大致分为以下几个步骤：

- **初始标记**
- **并发标记**
- **最终标记**
- **筛选回收**

G1收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的Region(这也就是它的名字Garbage-First的由来)。这种使用Region划分内存空间以及有优先级的区域回收方式，保证了G1收集器在有限时间内可以尽可能高的收集效率（把内存化整为零）。

#### ZGC收集器

与CMS中的ParNew和G1类似，ZGC也采用标记-复制算法，不过ZGC对该算法做了重大改进。在ZGC中出现Stop The World的情况会更少！

> 详情可以看：[《新一代垃圾回收器ZGC的探索与实践》](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)
> [原文链接](https://javaguide.cn/java/jvm/jvm-garbage-collection.html)


### 相关文章

- [7种JVM垃圾回收器及垃圾回收流程](https://mp.weixin.qq.com/s/fyorrpT5-hFpIS5aEDNjZA)
- [深度解析ZGC](https://mp.weixin.qq.com/s/NYrUEYmuWQw6RrN1VVILpg)
- [从历代GC算法角度刨析ZGC](https://mp.weixin.qq.com/s/VrjjPUm-p6h1YmU8olN6-Q)
- [深度揭秘垃圾回收底层，这次让你彻底弄懂它](https://mp.weixin.qq.com/s/pb7h9ROzr5LOLA0gMnquqQ)
- [看完这篇垃圾回收，和面试官扯皮没问题了](https://mp.weixin.qq.com/s/EcTwdaxP-Z4jG7PDq50nSA)
- [详解Java性能优化和JVM GC（垃圾回收机制）](https://mp.weixin.qq.com/s/Bfz7sqaI4iJuGe_i5dUXgA)
- [图文并茂，万字详解，带你掌握JVM垃圾回收！](https://mp.weixin.qq.com/s/EYOD4mQ7ErB11xMrGgzfCw)
- [一篇文章搞懂GC垃圾回收](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494090&idx=1&sn=0cb942ee592e277014b6cd16059c13c7&source=41#wechat_redirect)
- [垃圾回收策略和算法，看这篇就够了](https://mp.weixin.qq.com/s/Anj6PRc9UPiWGDnpKahxUQ)
- [一口气问了我18个JVM问题](https://mp.weixin.qq.com/s/U8uzm3YjdqoYCtLPPosSrg)
- [一文了解Java 8-18，垃圾回收的十次进化](https://mp.weixin.qq.com/s/nF489_nmrFAWcw1IfFF9lg)
- [CMS和G1改用三色标记法，可达性分析到底做错了什么](https://mp.weixin.qq.com/s/LfGrLo0fVrR85qOXAZxnvw)

## JVM调优

根据刚刚涉及的jvm的知识点，我们可以尝试对JVM进行调优，主要就是堆内存那块。

所有线程共享数据区大小=新生代大小+年老代大小+持久代大小。持久代一般固定大小为64m。所以java堆中增大年轻代后，将会减小年老代大小（因为老年代的清理是使用fullgc，所以老年代过小的话反而是会增多fullgc的）。此值对系统性能影响较大，Sun官方推荐配置为java堆的3/8。

### 调整最大堆内存和最小堆内存

-Xmx –Xms：指定java堆最大值（默认值是物理内存的1/4(<1GB)）和初始java堆最小值（默认值是物理内存的1/64(<1GB)。默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制。简单点来说，你不停地往堆内存里面丢数据，等它剩余大小小于40%了，JVM就会动态申请内存空间不过会小于-Xmx，如果剩余大小大于70%，又会动态缩小不过不会小于–Xms。就这么简单

开发过程中，通常会将-Xms与-Xmx两个参数配置成相同的值，其目的是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小而浪费资源。我们执行下面的代码


```java
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");  //系统的最大空间
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //系统的空闲空间
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  //当前可用的总空间
```

注意：此处设置的是Java堆大小，也就是新生代大小+老年代大小

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/5e7b352c16d74c789c665af46d3a2509-new-imagedd645dae-307d-4572-b6e2-b5a9925a46cd.png)

设置一个VMoptions的参数

```
-Xmx20m -Xms5m -XX:+PrintGCDetails
```

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/fe99e355f4754fa4be7427cb65261f3d-new-imagebb5cf485-99f8-43eb-8809-2a89e6a1768e.png)

再次启动main方法

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/300539f6560043dd8a3fe085d28420e6-new-image3c581a2e-196f-4b01-90f1-c27731b4610b.png)

这里GC弹出了一个Allocation Failure分配失败，这个事情发生在PSYoungGen，也就是年轻代中，这时候申请到的内存为18M，空闲内存为4.214195251464844M，我们此时创建一个字节数组看看，执行下面的代码

```java
byte[] b = new byte[1 * 1024 * 1024];
System.out.println("分配了1M空间给数组");
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");  //系统的最大空间
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //系统的空闲空间
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");
```

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/bdd717d0a3394be7a733760052773374-new-image371b5d59-0020-4091-9874-603c0ab0073d.png)

此时free memory就又缩水了，不过total memory是没有变化的。Java会尽可能将total mem的值维持在最小堆内存大小。

```java
byte[] b = new byte[10 * 1024 * 1024];
System.out.println("分配了10M空间给数组");
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");  //系统的最大空间
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //系统的空闲空间
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  //当前可用的总空间
```

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/0fd7550ae2144adca8ed2ede12d5fb96-new-image0c31ff20-289d-4088-8c67-a846d0c5d1e0.png)

这时候我们创建了一个10M的字节数据，这时候最小堆内存是顶不住的。我们会发现现在的total memory已经变成了15M，这就是已经申请了一次内存的结果。此时我们再跑一下这个代码

```java
System.gc();
System.out.println("Xmx=" + Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");    //系统的最大空间
System.out.println("free mem=" + Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");  //系统的空闲空间
System.out.println("total mem=" + Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");  //当前可用的总空间
```

![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/4cc44b5d5d1c40c48640ece6a296b1ac-new-image4b57baf6-085b-4150-9c60-ac51b0f815d7.png)

此时我们手动执行了一次fullgc，此时total memory的内存空间又变回5.5M了，此时又是把申请的内存释放掉的结果。

### 调整新生代和老年代的比值

-XX:NewRatio---新生代（eden+2*Survivor）和老年代（不包含永久区）的比值。例如：-XX:NewRatio=4，表示新生代:老年代=1:4，即新生代占整个堆的1/5。在Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置。

### 调整Survivor区和Eden区的比值

-XX:SurvivorRatio（幸存代）---设置两个Survivor区和eden的比值。例如：8，表示两个Survivor:eden=2:8，即一个Survivor占年轻代的1/10

### 设置年轻代和老年代的大小

-XX:NewSize---设置年轻代大小

-XX:MaxNewSize---设置年轻代最大值

可以通过设置不同参数来测试不同的情况，反正最优解当然就是官方的Eden和Survivor的占比为8:1:1，然后在刚刚介绍这些参数的时候都已经附带了一些说明，感兴趣的也可以看看。反正最大堆内存和最小堆内存如果数值不同会导致多次的gc，需要注意。

### 小总结

根据实际事情调整新生代和幸存代的大小，官方推荐新生代占java堆的3/8，幸存代占新生代的1/10，在OOM时，记得Dump出堆，确保可以排查现场问题，通过下面命令你可以输出一个.dump文件，这个文件可以使用VisualVM或者Java自带的Java VisualVM工具。

```
-Xmx20m-Xms5m-XX:+HeapDumpOnOutOfMemoryError-XX:HeapDumpPath=你要输出的日志路径
```

一般我们也可以通过编写脚本的方式来让OOM出现时给我们报个信，可以通过发送邮件或者重启程序等来解决。

### 永久区的设置

```
-XX:PermSize -XX:MaxPermSize
```

初始空间（默认为物理内存的1/64）和最大空间（默认为物理内存的1/4）。也就是说，jvm启动时，永久区一开始就占用了PermSize大小的空间，如果空间还不够，可以继续扩展，但是不能超过MaxPermSize，否则会OOM。

> tips：如果堆空间没有用完也抛出了OOM，有可能是永久区导致的。堆空间实际占用非常少，但是永久区溢出一样抛出OOM。

### JVM的栈参数调优

#### 调整每个线程栈空间的大小

可以通过-Xss：调整每个线程栈空间的大小，JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。在相同物理内存下,减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右

#### 设置线程栈的大小

```
-XXThreadStackSize： 设置线程栈的大小(0 means use default stack size)
```

这些参数都是可以通过自己编写程序去简单测试的，这里碍于篇幅问题就不再提供demo了

### JVM其他参数介绍

#### 设置内存页的大小

```
-XXThreadStackSize：
    设置内存页的大小，不可设置过大，会影响Perm的大小
```

#### 设置原始类型的快速优化

```
-XX:+UseFastAccessorMethods：
    设置原始类型的快速优化
```

#### 设置关闭手动GC

```
-XX:+DisableExplicitGC：
    设置关闭System.gc()(这个参数需要严格的测试)
```

#### 设置垃圾最大年龄

```
-XX:MaxTenuringThreshold
    设置垃圾最大年龄。如果设置为0的话,则年轻代对象不经过Survivor区,直接进入年老代.
    对于年老代比较多的应用,可以提高效率。如果将此值设置为一个较大值,
    则年轻代对象会在Survivor区进行多次复制,这样可以增加对象再年轻代的存活时间,
    增加在年轻代即被回收的概率。该参数只有在串行GC时才有效.
```

#### 加快编译速度

```
-XX:+AggressiveOpts
```

加快编译速度

#### 改善锁机制性能

```
-XX:+UseBiasedLocking
```

#### 禁用垃圾回收

```
-Xnoclassgc
```

#### 设置堆空间存活时间

```
-XX:SoftRefLRUPolicyMSPerMB
    设置每兆堆空闲空间中SoftReference的存活时间，默认值是1s。
```

#### 设置对象直接分配在老年代

```
-XX:PretenureSizeThreshold
    设置对象超过多大时直接在老年代分配，默认值是0。
```

#### 设置TLAB占eden区的比例

```
-XX:TLABWasteTargetPercent
    设置TLAB占eden区的百分比，默认值是1%。
```

#### 设置是否优先YGC

```
-XX:+CollectGen0First
    设置FullGC时是否先YGC，默认值是false。
```
> [原文链接](https://javaguide.cn/java/jvm/jvm-intro.html)

## JVM工具

- **jps**(JVM Process Status）:类似UNIX的ps命令。用于查看所有Java进程的启动类、传入参数和Java虚拟机参数等信息；
- **jstat**（JVM Statistics Monitoring Tool）:用于收集HotSpot虚拟机各方面的运行数据;
- **jinfo** (Configuration Info for Java):Configuration Info for Java,显示虚拟机配置信息;
- **jmap** (Memory Map for Java):生成堆转储快照;
- **jhat**(JVM Heap Dump Browser):用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结果;
- **jstack**(Stack Trace for Java):生成虚拟机当前时刻的线程快照，线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合。

### jps:查看所有Java进程

jps(JVM Process Status)命令类似UNIX的ps命令。

`jps`：显示虚拟机执行主类名称以及这些进程的本地虚拟机唯一ID（Local Virtual Machine Identifier,LVMID）。
`jps -q`：只输出进程的本地虚拟机唯一ID。

```powershell
C:\Users\SnailClimb>jps
7360 NettyClient2
17396
7972 Launcher
16504 Jps
17340 NettyServer
```

`jps -l`:输出主类的全名，如果进程执行的是Jar包，输出Jar路径。

```powershell
C:\Users\SnailClimb>jps -l
7360 firstNettyDemo.NettyClient2
17396
7972 org.jetbrains.jps.cmdline.Launcher
16492 sun.tools.jps.Jps
17340 firstNettyDemo.NettyServer
```

`jps -v`：输出虚拟机进程启动时JVM参数。

`jps -m`：输出传递给Java进程main()函数的参数。

### jstat:监视虚拟机各种运行状态信息

jstat（JVM Statistics Monitoring Tool）使用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程（需要远程主机提供RMI支持）虚拟机进程中的类信息、内存、垃圾收集、JIT编译等运行数据，在没有GUI，只提供了纯文本控制台环境的服务器上，它将是运行期间定位虚拟机性能问题的首选工具。

**jstat命令使用格式：**

```powershell
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

比如`jstat -gc -h3 31736 1000 10`表示分析进程id为31736的gc情况，每隔1000ms打印一次记录，打印10次停止，每3行后打印指标头部。

**常见的option如下：**

- jstat -class vmid：显示ClassLoader的相关信息；
- jstat -compiler vmid：显示JIT编译的相关信息；
- jstat -gc vmid：显示与GC相关的堆信息；
- jstat -gccapacity vmid：显示各个代的容量及使用情况；
- jstat -gcnew vmid ：显示新生代信息；
- jstat -gcnewcapcacity vmid：显示新生代大小与使用情况；
- jstat -gcold vmid ：显示老年代和永久代的行为统计，从jdk1.8开始,该选项仅表示老年代，因为永久代被移除了；
- jstat -gcoldcapacity vmid：显示老年代的大小；
- jstat -gcpermcapacity vmid：显示永久代大小，从jdk1.8开始,该选项不存在了，因为永久代被移除了；
- jstat -gcutil vmid：显示垃圾收集信息；

另外，加上`-t`参数可以在输出信息上加一个Timestamp列，显示程序的运行时间。

主要是对java应用程序的资源和性能进行实时的命令行监控，包括了对heap size和垃圾回收状况的监控

```shell
jstat -class pid # 输出加载类的数量及所占空间信息。
jstat -gc pid # 输出gc信息，包括gc次数和时间，内存使用状况（可带时间和显示条目参数）
```

> [你了解Java的jstat命令吗？](https://mp.weixin.qq.com/s/4vUmSsPoEI-MLVKc2NBvnw)

### jinfo:实时地查看和调整虚拟机各项参数

jinfo vmid:输出当前jvm进程的全部参数和系统属性(第一部分是系统的属性，第二部分是JVM的参数)。

jinfo -flag name vmid:输出对应名称的参数的具体值。比如输出MaxHeapSize、查看当前jvm进程是否开启打印GC日志(-XX:PrintGCDetails:详细GC日志模式，这两个都是默认关闭的)。

```powershell
C:\Users\SnailClimb>jinfo  -flag MaxHeapSize 17340
-XX:MaxHeapSize=2124414976
C:\Users\SnailClimb>jinfo  -flag PrintGC 17340
-XX:-PrintGC
```

使用jinfo可以在不重启虚拟机的情况下，可以动态的修改jvm的参数。尤其在线上的环境特别有用,请看下面的例子：

jinfo -flag [+|-]name vmid开启或者关闭对应名称的参数。

```powershell
C:\Users\SnailClimb>jinfo  -flag  PrintGC 17340
-XX:-PrintGC

C:\Users\SnailClimb>jinfo  -flag  +PrintGC 17340

C:\Users\SnailClimb>jinfo  -flag  PrintGC 17340
-XX:+PrintGC
```

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

### jmap:生成堆转储快照

jmap（MemoryMapforJava）命令用于生成堆转储快照。如果不使用jmap命令，要想获取Java堆转储，可以使用“-XX:+HeapDumpOnOutOfMemoryError”参数，可以让虚拟机在OOM异常出现之后自动生成dump文件，Linux命令下可以通过kill-3发送进程退出信号也能拿到dump文件。

jmap的作用并不仅仅是为了获取dump文件，它还可以查询finalizer执行队列、Java堆和永久代的详细信息，如空间使用率、当前使用的是哪种收集器等。和jinfo一样，jmap有不少功能在Windows平台下也是受限制的。

示例：将指定应用程序的堆快照输出到桌面。后面，可以通过jhat、Visual VM等工具分析该堆文件。

```powershell
C:\Users\SnailClimb>jmap -dump:format=b,file=C:\Users\SnailClimb\Desktop\heap.hprof 17340
Dumping heap to C:\Users\SnailClimb\Desktop\heap.hprof ...
Heap dump file created
```

```shell
jmap -heap pid # 输出堆内存设置和使用情况（JDK11使用jhsdb jmap --heap --pid pid）
jmap -histo pid # 查看堆中对象数量和大小，包括类名，对象数量，对象占用大小
jmap -histo:live pid # 同上，只输出存活对象信息
jmap -clstats pid # 输出加载类信息
jmap -help # jmap命令帮助信息
```

**生成dump文件**

方式一、jmap -dump:live,format=b,file=heap-dump.bin (pid)
方式二、使用JConsole的dumpHeap按钮生成Heap Dump文件
方式三、在JVM的配置参数中可以添加
-XX:+HeapDumpOnOutOfMemoryError参数，当应用抛出OutOfMemoryError时自动生成dump文件；
-XX:HeapDumpPath=/home/liuke/jvmlogs/：生成堆文件地址
在JVM的配置参数中添加-Xrunhprof:head=site参数，会生成java.hprof.txt文件，不过这样会影响JVM的运行效率，不建议在生产环境中使用（未亲测）

### jhat:分析heapdump文件

jhat用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结果。

```powershell
C:\Users\SnailClimb>jhat C:\Users\SnailClimb\Desktop\heap.hprof
Reading from C:\Users\SnailClimb\Desktop\heap.hprof...
Dump file created Sat May 04 12:30:31 CST 2019
Snapshot read, resolving...
Resolving 131419 objects...
Chasing references, expect 26 dots..........................
Eliminating duplicate references..........................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

访问[http://localhost:7000/](http://localhost:7000/)

### jstack:生成虚拟机当前时刻的线程快照

jstack（Stack Trace for Java）命令用于生成虚拟机当前时刻的线程快照。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合.

生成线程快照的目的主要是定位线程长时间出现停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的原因。线程出现停顿的时候通过`jstack`来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者在等待些什么资源。

**下面是一个线程死锁的代码。我们下面会通过jstack命令进行死锁检查，输出死锁信息，找到发生死锁的线程**。


```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//资源1
    private static Object resource2 = new Object();//资源2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                System.out.println(Thread.currentThread() + "get resource1");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource2");
                synchronized (resource2) {
                    System.out.println(Thread.currentThread() + "get resource2");
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                System.out.println(Thread.currentThread() + "get resource2");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread() + "waiting get resource1");
                synchronized (resource1) {
                    System.out.println(Thread.currentThread() + "get resource1");
                }
            }
        }, "线程 2").start();
    }
}
```

Output


```text
Thread[线程 1,5,main]get resource1
Thread[线程 2,5,main]get resource2
Thread[线程 1,5,main]waiting get resource2
Thread[线程 2,5,main]waiting get resource1
```

线程A通过synchronized(resource1)获得resource1的监视器锁，然后通过Thread.sleep(1000);让线程A休眠1s为的是让线程B得到执行然后获取到resource2的监视器锁。线程A和线程B休眠结束了都开始企图请求获取对方的资源，然后这两个线程就会陷入互相等待的状态，这也就产生了死锁。

**通过jstack命令分析：**


```powershell
C:\Users\SnailClimb>jps
13792 KotlinCompileDaemon
7360 NettyClient2
17396
7972 Launcher
8932 Launcher
9256 DeadLockDemo
10764 Jps
17340 NettyServer

C:\Users\SnailClimb>jstack 9256
```

输出的部分内容如下：

```powershell
Found one Java-level deadlock:
=============================
"线程2":
  waiting to lock monitor 0x000000000333e668 (object 0x00000000d5efe1c0, a java.lang.Object),
  which is held by "线程1"
"线程1":
  waiting to lock monitor 0x000000000333be88 (object 0x00000000d5efe1d0, a java.lang.Object),
  which is held by "线程2"

Java stack information for the threads listed above:
===================================================
"线程2":
        at DeadLockDemo.lambda$main$1(DeadLockDemo.java:31)
        - waiting to lock <0x00000000d5efe1c0> (a java.lang.Object)
        - locked <0x00000000d5efe1d0> (a java.lang.Object)
        at DeadLockDemo$$Lambda$2/1078694789.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"线程1":
        at DeadLockDemo.lambda$main$0(DeadLockDemo.java:16)
        - waiting to lock <0x00000000d5efe1d0> (a java.lang.Object)
        - locked <0x00000000d5efe1c0> (a java.lang.Object)
        at DeadLockDemo$$Lambda$1/1324119927.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

可以看到jstack命令已经帮我们找到发生死锁的线程的具体信息。

### JDK可视化分析工具

### JConsole:Java监视与管理控制台

JConsole是基于JMX的可视化监视、管理工具。可以很方便的监视本地及远程服务器的java进程的内存使用情况。你可以在控制台输出`console`命令启动或者在JDK目录下的bin目录找到`jconsole.exe`然后双击启动。

#### 连接Jconsole

如果需要使用JConsole连接远程进程，可以在远程Java程序启动时加上下面这些参数:

```properties
-Djava.rmi.server.hostname=外网访问ip地址
-Dcom.sun.management.jmxremote.port=60001 //监控的端口号
-Dcom.sun.management.jmxremote.authenticate=false //关闭认证
-Dcom.sun.management.jmxremote.ssl=false
```

在使用JConsole连接时，远程进程地址如下：


```text
外网访问ip地址:60001
```

#### 内存监控

JConsole可以显示当前内存的详细信息。不仅包括堆内存/非堆内存的整体信息，还可以细化到eden区、survivor区等的使用情况，如下图所示。点击右边的“执行GC(G)”按钮可以强制应用程序执行一个FullGC。

> - **新生代 GC（Minor GC）**:指发生新生代的的垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快。
> - **老年代 GC（Major GC/Full GC）**:指发生在老年代的GC，出现了Major GC经常会伴随至少一次的Minor GC（并非绝对），Major GC的速度一般会比Minor GC的慢10倍以上。

#### 线程监控

类似我们前面讲的`jstack`命令，不过这个是可视化的。最下面有一个"检测死锁(D)"按钮，点击这个按钮可以自动为你找到发生死锁的线程以及它们的详细信息。

### Visual VM:多合一故障处理工具

VisualVM提供在Java虚拟机(Java Virutal Machine,JVM)上运行的Java应用程序的详细信息。在VisualVM的图形用户界面中，您可以方便、快捷地查看多个Java应用程序的相关信息。[VisualVM官网](https://visualvm.github.io/)、[VisualVM中文文档](https://visualvm.github.io/documentation.html)。

下面这段话摘自《深入理解Java虚拟机》。

> VisualVM（All-in-One Java Troubleshooting Tool）是到目前为止随JDK发布的功能最强大的运行监视和故障处理程序，官方在VisualVM的软件说明中写上了“All-in-One”的描述字样，预示着他除了运行监视、故障处理外，还提供了很多其他方面的功能，如性能分析（Profiling）。VisualVM的性能分析功能甚至比起JProfiler、YourKit等专业且收费的Profiling工具都不会逊色多少，而且VisualVM还有一个很大的优点：不需要被监视的程序基于特殊Agent运行，因此他对应用程序的实际性能的影响很小，使得他可以直接应用在生产环境中。这个优点是JProfiler、YourKit等工具无法与之媲美的。

VisualVM基于NetBeans平台开发，因此他一开始就具备了插件扩展功能的特性，通过插件扩展支持，VisualVM可以做到：

- 显示虚拟机进程以及进程的配置、环境信息（jps、jinfo）。
- 监视应用程序的CPU、GC、堆、方法区以及线程的信息（jstat、jstack）。
- dump以及分析堆转储快照（jmap、jhat）。
- 方法级的程序运行性能分析，找到被调用最多、运行时间最长的方法。
- 离线程序快照：收集程序的运行时配置、线程dump、内存dump等信息建立一个快照，可以将快照发送开发者处进行Bug反馈。
- 其他plugins的无限的可能性......

> 这里就不具体介绍VisualVM的使用，如果想了解的话可以看:
> - [https://visualvm.github.io/documentation.html](https://visualvm.github.io/documentation.html)
> - https://www.ibm.com/developerworks/cn/java/j-lo-visualvm/index.html
> 
> [原文链接](https://javaguide.cn/java/jvm/jdk-monitoring-and-troubleshooting-tools.html)

###  jstack

主要用于生成指定进程当前时刻的线程快照，线程快照是当前java虚拟机每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致长时间等待。

### jcmd

在JDK1.7之后，新增了一个命令行工具jcmd。它是一个多功能工具，可以用来导出堆，查看java进程，导出线程信息，执行GC等。jcmd拥有jmap的大部分功能，Oracle官方建议使用jcmd代替jmap。

### 相关文章

- [这几款JVM故障诊断处理工具，你还不会？](https://mp.weixin.qq.com/s/b5pnoqWNMIp7JnfUWyWLoA)
- [JVM性能调优监控工具jps、jstack、jmap、jhat、jstat、hprof使用详解](https://mp.weixin.qq.com/s/72X1qvTsCLneFffzIpKoaA)
- [有了这款可视化工具，Java应用性能调优so easy！(JVisualVM简介)](https://mp.weixin.qq.com/s/rLhkbpe0Uo3Nsy8Xs68Z7A)
- [死锁的4种排查工具！](https://mp.weixin.qq.com/s/5l6q1-jcfZNegpUsriaj5g)
- [6款Java8自带工具，轻松分析定位JVM问题！](https://mp.weixin.qq.com/s/phD24wUysUDQkx5pAjp5tg)


## JVM参数

### 示例
```
-Xms:初始堆大小,默认值:物理内存的1/64(<1GB)
默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.

-Xmx:最大堆大小,默认值:物理内存的1/4(<1GB)
默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制

-Xmn:年轻代大小(1.4or lator)
此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的。整个堆大小=年轻代大小+老年代大小+持久代（永久代）大小.增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8

-XX:NewSize:年轻代大小(for 1.3/1.4)

-XX:MaxNewSize:年轻代最大值(for 1.3/1.4)

-XX:PermSize:设置持久代(perm gen)初始值,默认值:物理内存的1/64

-XX:MaxPermSize:设置持久带最大值

**注意：jdk8取消持久带用元空间代替**

-XX:MetaspaceSize代替-XX:PermSize

-XX:MaxMetaspaceSize代替-XX:MaxPermSize

-Xss:每个线程的堆栈大小

JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行调整,在相同物理内存下,减小这个值能生成更多的线程,但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右一般小的应用,如果栈不是很深,应该是128k够用的大的应用建议使用256k.这个选项对性能影响比较大，需要严格的测试

-XX:NewRatio:年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代)
-XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆栈的1/5Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置

-XX:SurvivorRatio:Eden区与Survivor区的大小比值
-XX:SurvivorRatio=8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10

-XX:MaxDirectMemorySize：设置New I/O(java.nio) direct-buffer allocations的最大大小
当Direct ByteBuffer分配的堆外内存到达指定大小后，即触发Full GC。注意该值是有上限的，默认是64M，最大sun.misc.VM.maxDirectMemory()，在程序中中可以获得-XX:MaxDirectMemorySize的设置的值。

-XX:+DisableExplicitGC:关闭System.gc()

-XX:PretenureSizeThreshold:对象超过多大是直接在旧生代分配	0
单位字节,新生代采用Parallel ScavengeGC时无效另一种直接在旧生代分配的情况是大的数组对象,且数组中无外部引用对象.

-XX:ParallelGCThreads:并行收集器的线程数
此值最好配置与处理器数目相等,同样适用于CMS

-XX:MaxGCPauseMillis:每次年轻代垃圾回收的最长时间(最大暂停时间)
如果无法满足此时间,JVM会自动调整年轻代大小,以满足此值.
```

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

  -XX:ReservedCodeCacheSize=8m # 这是由 JIT（即时）编译器编译为本地代码的本机代码（如JNI）或Java方法的空间

  -XX:MaxDirectMemorySize=16m

  -XX:NativeMemoryTracking=summary #开启内存追踪

  -jar app.jar
```
### 堆内存相关

> Java虚拟机所管理的内存中最大的一块，Java堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存**。

#### 显式指定堆内存–Xms和-Xmx

与性能有关的最常见实践之一是根据应用程序要求初始化堆内存。如果我们需要指定最小和最大堆大小（推荐显示指定大小），以下参数可以帮助你实现：

```bash
-Xms<heap size>[unit]
-Xmx<heap size>[unit]
```

- **heap size**表示要初始化内存的具体大小。
- **unit**表示要初始化内存的单位。单位为**g**(GB)、**m**（MB）、**k**（KB）。

举个栗子🌰，如果我们要为JVM分配最小2GB和最大5GB的堆内存大小，我们的参数应该这样来写：

```bash
-Xms2G -Xmx5G
```

#### 显式新生代内存(Young Generation)

根据[Oracle官方文档](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html)，在堆总可用内存配置完成之后，第二大影响因素是为Young Generation在堆内存所占的比例。默认情况下，YG的最小大小为1310*MB*，最大大小为无限制。

一共有两种指定新生代内存(Young Ceneration)大小的方法：

1.通过-XX:NewSize和-XX:MaxNewSize指定

```bash
-XX:NewSize=<young size>[unit]
-XX:MaxNewSize=<young size>[unit]
```

举个栗子🌰，如果我们要为新生代分配最小256m的内存，最大1024m的内存我们的参数应该这样来写：


```bash
-XX:NewSize=256m
-XX:MaxNewSize=1024m
```

2.通过-Xmn<young size\>[unit]指定

举个栗子🌰，如果我们要为新生代分配256m的内存（NewSize与MaxNewSize设为一致），我们的参数应该这样来写：

```bash
-Xmn256m
```

GC调优策略中很重要的一条经验总结是这样说的：

> 将新对象预留在新生代，由于Full GC的成本远高于Minor GC，因此尽可能将对象分配在新生代是明智的做法，实际项目中根据GC日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况。

另外，你还可以通过-XX:NewRatio=<int\>来设置老年代与新生代内存的比值。

比如下面的参数就是设置老年代与新生代内存的比值为1。也就是说老年代和新生代所占比值为1：1，新生代占整个堆栈的1/2。

```text
-XX:NewRatio=1
```

#### 显式指定永久代/元空间的大小

**从Java8开始，如果我们没有指定Metaspace的大小，随着更多类的创建，虚拟机会耗尽所有可用的系统内存（永久代并不会出现这种情况）**。

JDK1.8之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```bash
# 方法区(永久代)初始大小
-XX:PermSize=N
# 方法区(永久代)最大大小,超过这个值将会抛出OutOfMemoryError异常:java.lang.OutOfMemoryError:PermGen
-XX:MaxPermSize=N
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

**JDK1.8的时候，方法区（HotSpot的永久代）被彻底移除了（JDK1.7就已经开始了），取而代之是元空间，元空间使用的是本地内存**。下面是一些常用参数：

```bash
# 设置Metaspace的初始（和最小大小）
-XX:MetaspaceSize=N
# 设置Metaspace的最大大小，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。
-XX:MaxMetaspaceSize=N
```

### 垃圾收集相关

#### 垃圾回收器

为了提高应用程序的稳定性，选择正确的[垃圾收集](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)算法至关重要。

JVM具有四种类型的GC实现：

- 串行垃圾收集器
- 并行垃圾收集器
- CMS垃圾收集器
- G1垃圾收集器

可以使用以下参数声明这些实现：

```bash
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+UseParNewGC
-XX:+UseG1GC
```

> 有关*垃圾回收*实施的更多详细信息，请参见[此处](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/jvm/JVM垃圾回收.md)。

#### GC日志记录

生产环境上，或者其他要测试GC问题的环境上，一定会配置上打印GC日志的参数，便于分析GC相关的问题。


```bash
# 必选
# 打印基本GC信息
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
# 打印对象分布
-XX:+PrintTenuringDistribution
# 打印堆数据
-XX:+PrintHeapAtGC
# 打印Reference处理信息
# 强引用/弱引用/软引用/虚引用/finalize相关的方法
-XX:+PrintReferenceGC
# 打印STW时间
-XX:+PrintGCApplicationStoppedTime

# 可选
# 打印safepoint信息，进入STW阶段之前，需要要找到一个合适的safepoint
-XX:+PrintSafepointStatistics
-XX:PrintSafepointStatisticsCount=1

# GC日志输出的文件路径
-Xloggc:/path/to/gc-%t.log
# 开启日志文件分割
-XX:+UseGCLogFileRotation
# 最多分割几个文件，超过之后从头文件开始写
-XX:NumberOfGCLogFiles=14
# 每个文件上限大小，超过就触发分割
-XX:GCLogFileSize=50M
```

### 处理OOM

对于大型应用程序来说，面对内存不足错误是非常常见的，这反过来会导致应用程序崩溃。这是一个非常关键的场景，很难通过复制来解决这个问题。这就是为什么JVM提供了一些参数，这些参数将堆内存转储到一个物理文件中，以后可以用来查找泄漏:

```bash
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./java_pid<pid>.hprof
-XX:OnOutOfMemoryError="< cmd args >;< cmd args >"
-XX:+UseGCOverheadLimit
```

这里有几点需要注意:

- **HeapDumpOnOutOfMemoryError**指示JVM在遇到**OutOfMemoryError**错误时将heap转储到物理文件中。
- **HeapDumpPath**表示要写入文件的路径;可以给出任何文件名;但是，如果JVM在名称中找到一个`<pid>`标记，则当前进程的进程id将附加到文件名中，并使用`.hprof`格式
- **OnOutOfMemoryError**用于发出紧急命令，以便在内存不足的情况下执行;应该在`cmd args`空间中使用适当的命令。例如，如果我们想在内存不足时重启服务器，我们可以设置参数:`-XX:OnOutOfMemoryError="shutdown -r"`。
- **UseGCOverheadLimit**是一种策略，它限制在抛出OutOfMemory错误之前在GC中花费的VM时间的比例

### 其他

- `-server`:启用“Server Hotspot VM”;此参数默认用于64位JVM
- `-XX:+UseStringDeduplication`:*Java 8u20*引入了这个JVM参数，通过创建太多相同String的实例来减少不必要的内存使用;这通过将重复String值减少为单个全局`char []`数组来优化堆内存。
- `-XX:+UseLWPSynchronization`:设置基于LWP(轻量级进程)的同步策略，而不是基于线程的同步。
- `-XX:LargePageSizeInBytes`:设置用于Java堆的较大页面大小;它采用GB/MB/KB的参数;页面大小越大，我们可以更好地利用虚拟内存硬件资源;然而，这可能会导致PermGen的空间大小更大，这反过来又会迫使Java堆空间的大小减小。
- `-XX:MaxHeapFreeRatio`:设置GC后,堆空闲的最大百分比，以避免收缩。
- `-XX:SurvivorRatio`:eden/survivor空间的比例,例如`-XX:SurvivorRatio=6`设置每个survivor和eden之间的比例为1:6。
- `-XX:+UseLargePages`:如果系统支持，则使用大页面内存;请注意，如果使用这个JVM参数，OpenJDK 7可能会崩溃。
- `-XX:+UseStringCache`:启用String池中可用的常用分配字符串的缓存。
- `-XX:+UseCompressedStrings`:对String对象使用`byte []`类型，该类型可以用纯ASCII格式表示。
- `-XX:+OptimizeStringConcat`:它尽可能优化字符串串联操作。

### 文章推荐

- [JVM参数配置说明-阿里云官方文档-2022](https://help.aliyun.com/document_detail/148851.html)
- [JVM内存配置最佳实践-阿里云官方文档-2022](https://help.aliyun.com/document_detail/383255.html)
- [求你了，GC日志打印别再瞎配置了](https://segmentfault.com/a/1190000039806436)
- [一次大量JVM Native内存泄露的排查分析（64M问题）](https://juejin.cn/post/7078624931826794503)
- [一次线上JVM调优实践，FullGC40次/天到10天一次的优化过程](https://heapdump.cn/article/1859160)
- [听说JVM性能优化很难？今天我小试了一把](https://shuyi.tech/archives/have-a-try-in-jvm-combat)
- [你们要的线上GC问题案例来啦](https://mp.weixin.qq.com/s/df1uxHWUXzhErxW1sZ6OvQ)
- [Java中9种常见的CMS GC问题分析与解决](https://tech.meituan.com/2020/11/12/java-9-cms-gc.html)
- [从实际案例聊聊Java应用的GC优化](https://tech.meituan.com/2017/12/29/jvm-optimize.html)
- [常用的J?VM参数，你现在就记好！](https://mp.weixin.qq.com/s/3yvA2m66--Nxsf_xQDJZog)
- [每天100万次登陆请求，8G内存该如何设置JVM参数？](https://mp.weixin.qq.com/s/vZtg3XolIYJuiqJIeM2j3A)


## 相关文章

### JVM知识点

- [JVM知识点全面梳理！](https://mp.weixin.qq.com/s/IUw8SroGMJdvqPFQ1xEHxg)
- [《深入理解Java虚拟机》把这个知识点讲错了？](https://mp.weixin.qq.com/s/4qZLipxi-zjrdI-_6Ae11A)
- [JVM史上最最最完整知识总结！](https://mp.weixin.qq.com/s/GAXLr0cIcLnGaTXVstHKvA)
- [JVM面试的30个知识点](https://mp.weixin.qq.com/s/PM_Q3718Du81ZFEx0ARrkw)
- [满满的一整篇，全是JVM核心知识点！](https://mp.weixin.qq.com/s/GfAfffbF_uiphN0Zd3YouQ)
- [大白话带你认识JVM、JVM的常用参数](https://mp.weixin.qq.com/s/CZtW-rDXgRl9E_3N2ou5TA)
- [JVM夺命连环10问](https://mp.weixin.qq.com/s/_0IANOvyP_UNezDm0bxXmg)
- [【干货】JVM完整深入解析！](https://mp.weixin.qq.com/s/vd72d6wqc_fq7YUgA7d1fA)
- [如何在面试时搞定Java虚拟机](https://mp.weixin.qq.com/s/6MSezM6g4JPoML0kDtx64Q)
- [JAVA虚拟机(JVM)面试题](https://mp.weixin.qq.com/s/eNpHYG9T5DIdh7QNpLg4dQ)
- [JVM史上最最最完整深入解析，不看后悔一百次！](https://mp.weixin.qq.com/s/EYUNHXLP9jjTTA1Ig6vCYQ)
- [【秒懂！】JVM虚拟机图文详解！一点都不难！](https://mp.weixin.qq.com/s/A4BbKDmy8oe3uBfHfrgGQA)
- [从JMM透析volatile与synchronized原理](https://mp.weixin.qq.com/s/s2nnXDY7phqKX07nwZiHyA)
- [JVM锁优化和逃逸分析详解](https://mp.weixin.qq.com/s/guITssdS4aYXzDJD3xl4HA)


### 内存溢出分析

- [Java内存泄漏了，怎么排查？](https://mp.weixin.qq.com/s/OtSoZWIl4XuIzqTayfORHw)
- [常见OOM异常分析](https://mp.weixin.qq.com/s/DssuTRyN1RVWhQgMClP_ng)
- [大厂的OOM优化和监控方案](https://mp.weixin.qq.com/s/-pZDUcW2ljBSIxZV0w0iSw)
- [如何排查Java内存泄漏?看完我给跪了!](https://mp.weixin.qq.com/s/fH6Go6_1TcvHfxDPnyZvhg)
- [常见的OOM异常分析（硬核干货）](https://mp.weixin.qq.com/s/oNWwPZ56yLCp1KQyir17AA)
- [深入剖析线上内存溢出的原因](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494348&idx=1&sn=0096ff574aace3d725bcbb3323a67d01&source=41#wechat_redirect)
- [一套完整的Java线上故障排查技巧，建议收藏](https://mp.weixin.qq.com/s/03qoBFLpvkRVhaorwCNJ5Q)
- [GC垃圾收集器&JVM调优汇总](https://mp.weixin.qq.com/s/UzGeDeuZORaJkzFr_V6-5A)
- [图解Java虚拟机中GC的复制算法和“标记-整理”算法](https://mp.weixin.qq.com/s/1_Dz7vl9k01FtyYHIn107w)
- [Java项目线上故障排查：从CPU、磁盘、内存、网络、GC一条龙完整套路！](https://mp.weixin.qq.com/s/ujqwk7pdz5a-UpR6q5ogFQ)