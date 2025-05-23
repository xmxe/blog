---
title: Java类加载
categories: Java
img: https://pic1.zhimg.com/v2-2bc1a9883960ccf43a0f4c643178c4c3_1440w.jpg

---

## 类文件结构详解

**回顾一下字节码**

在Java中，JVM可以理解的代码就叫做字节码（即扩展名为.class的文件），它不面向任何特定的处理器，只面向虚拟机。Java语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点。所以Java程序运行时比较高效，而且，由于字节码并不针对一种特定的机器，因此，Java程序无须重新编译便可在多种不同操作系统的计算机上运行。

Clojure（Lisp语言的一种方言）、Groovy、Scala等语言都是运行在Java虚拟机之上。下图展示了不同的语言被不同的编译器编译成.class文件最终运行在Java虚拟机之上。.class文件的二进制格式可以使用[WinHex](https://www.x-ways.net/winhex/)查看。

![java虚拟机](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/bg/desktop类文件结构概览.png)

可以说.class文件是不同的语言在Java虚拟机之间的重要桥梁，同时也是支持Java跨平台很重要的一个原因。

**Class文件结构总结**

根据Java虚拟机规范，Class文件通过ClassFile定义，有点类似C语言的结构体。ClassFile的结构如下：

```java
ClassFile {
    u4             magic; //Class文件的标志
    u2             minor_version;//Class的小版本号
    u2             major_version;//Class的大版本号
    u2             constant_pool_count;//常量池的数量
    cp_info        constant_pool[constant_pool_count-1];//常量池
    u2             access_flags;//Class的访问标记
    u2             this_class;//当前类
    u2             super_class;//父类
    u2             interfaces_count;//接口
    u2             interfaces[interfaces_count];//一个类可以实现多个接口
    u2             fields_count;//Class文件的字段属性
    field_info     fields[fields_count];//一个类可以有多个字段
    u2             methods_count;//Class文件的方法数量
    method_info    methods[methods_count];//一个类可以有个多个方法
    u2             attributes_count;//此类的属性表中的属性数
    attribute_info attributes[attributes_count];//属性表集合
}
```

通过分析ClassFile的内容，我们便可以知道class文件的组成。

![ClassFile内容分析](https://oss.javaguide.cn/java-guide-blog/16d5ec47609818fc.jpeg)

下面这张图是通过IDEA插件jclasslib查看的，你可以更直观看到Class文件结构。

![img](https://oss.javaguide.cn/java-guide-blog/image-20210401170711475.png)

使用jclasslib不光可以直观地查看某个类对应的字节码文件，还可以查看类的基本信息、常量池、接口、属性、函数等信息。下面详细介绍一下Class文件结构涉及到的一些组件。

**魔数（MagicNumber）**


```java
u4             magic; //Class文件的标志
```

每个Class文件的头4个字节称为魔数（MagicNumber）,它的唯一作用是**确定这个文件是否为一个能被虚拟机接收的Class文件**。程序设计者很多时候都喜欢用一些特殊的数字表示固定的文件类型或者其它特殊的含义。

**Class文件版本号（Minor&MajorVersion）**


```java
u2             minor_version;//Class的小版本号
u2             major_version;//Class的大版本号
```

紧接着魔数的四个字节存储的是Class文件的版本号：第5和第6位是次版本号，第7和第8位是主版本号。每当Java发布大版本（比如Java8，Java9）的时候，主版本号都会加1。你可以使用`javap -v`命令来快速查看Class文件的版本号信息。高版本的Java虚拟机可以执行低版本编译器生成的Class文件，但是低版本的Java虚拟机不能执行高版本编译器生成的Class文件。所以，我们在实际开发的时候要确保开发的的JDK版本和生产环境的JDK版本保持一致。

**常量池（Constant Pool）**

```java
u2             constant_pool_count;//常量池的数量
cp_info        constant_pool[constant_pool_count-1];//常量池
```

紧接着主次版本号之后的是常量池，常量池的数量是`constant_pool_count-1`（常量池计数器是从1开始计数的，将第0项常量空出来是有特殊考虑的，索引值为0代表“不引用任何一个常量池项”）。常量池主要存放两大常量：字面量和符号引用。字面量比较接近于Java语言层面的的常量概念，如文本字符串、声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

常量池中每一项常量都是一个表，这14种表有一个共同的特点：开始的第一位是一个u1类型的标志位-tag来标识常量的类型，代表当前这个常量属于哪种常量类型。

|               类型               | 标志（tag） |          描述          |
| :------------------------------: | :---------: | :--------------------: |
|        CONSTANT_utf8_info        |      1      |   UTF-8 编码的字符串   |
|      CONSTANT_Integer_info       |      3      |       整形字面量       |
|       CONSTANT_Float_info        |      4      |      浮点型字面量      |
|        CONSTANT_Long_info        |     ５      |      长整型字面量      |
|       CONSTANT_Double_info       |     ６      |   双精度浮点型字面量   |
|       CONSTANT_Class_info        |     ７      |   类或接口的符号引用   |
|       CONSTANT_String_info       |     ８      |    字符串类型字面量    |
|      CONSTANT_Fieldref_info      |     ９      |     字段的符号引用     |
|     CONSTANT_Methodref_info      |     10      |   类中方法的符号引用   |
| CONSTANT_InterfaceMethodref_info |     11      |  接口中方法的符号引用  |
|    CONSTANT_NameAndType_info     |     12      |  字段或方法的符号引用  |
|     CONSTANT_MothodType_info     |     16      |      标志方法类型      |
|    CONSTANT_MethodHandle_info    |     15      |      表示方法句柄      |
|   CONSTANT_InvokeDynamic_info    |     18      | 表示一个动态方法调用点 |

.class文件可以通过javap -v class类名指令来看一下其常量池中的信息(javap -v class类名 -> temp.txt：将结果输出到temp.txt文件)。

**访问标志(Access Flags)**


```java
u2             access_flags;//Class的访问标记
```

在常量池结束之后，紧接着的两个字节代表访问标志，这个标志用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口，是否为public或者abstract类型，如果是类的话是否声明为final等等。

类访问和属性修饰符:

![类访问和属性修饰符](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/访问标志.png)

我们定义了一个Employee类

```java
package top.snailclimb.bean;
public class Employee {
   ...
}
```

通过`javap -v class类名`指令来看一下类的访问标志。

![查看类的访问标志](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/查看类的访问标志.png)

**当前类（This Class）、父类（Super Class）、接口（Interfaces）索引集合**

```java
u2             this_class;//当前类
u2             super_class;//父类
u2             interfaces_count;//接口
u2             interfaces[interfaces_count];//一个类可以实现多个接口
```

Java类的继承关系由类索引、父类索引和接口索引集合三项确定。类索引、父类索引和接口索引集合按照顺序排在访问标志之后，类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名，由于Java语言的单继承，所以父类索引只有一个，除了`java.lang.Object`之外，所有的Java类都有父类，因此除了`java.lang.Object`外，所有Java类的父类索引都不为0。接口索引集合用来描述这个类实现了那些接口，这些被实现的接口将按`implements`(如果这个类本身是接口的话则是`extends`)后的接口顺序从左到右排列在接口索引集合中。

**字段表集合（Fields）**

```java
u2             fields_count;//Class文件的字段的个数
field_info     fields[fields_count];//一个类会可以有个字段
```

字段表（field info）用于描述接口或类中声明的变量。字段包括类级变量以及实例变量，但不包括在方法内部声明的局部变量。

**field info(字段表)的结构:**

![字段表的结构](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/字段表的结构.png)

- **access_flags**:字段的作用域（public,private,protected修饰符），是实例变量还是类变量（static修饰符）,可否被序列化（transient修饰符）,可变性（final）,可见性（volatile修饰符，是否强制从主内存读写）。
- **name_index**:对常量池的引用，表示的字段的名称；
- **descriptor_index**:对常量池的引用，表示字段和方法的描述符；
- **attributes_count**:一个字段还会拥有一些额外的属性，attributes_count存放属性的个数；
- **attributes[attributes_count]**:存放具体属性具体内容。

上述这些信息中，各个修饰符都是布尔值，要么有某个修饰符，要么没有，很适合使用标志位来表示。而字段叫什么名字、字段被定义为什么数据类型这些都是无法固定的，只能引用常量池中常量来描述。

**字段的access_flag的取值:**

![字段的access_flag的取值](https://oss.javaguide.cn/JVM/image-20201031084342859.png)

**方法表集合（Methods）**

```java
u2             methods_count;//Class文件的方法的数量
method_info    methods[methods_count];//一个类可以有个多个方法
```

methods_count表示方法的数量，而method_info表示方法表。

Class文件存储格式中对方法的描述与对字段的描述几乎采用了完全一致的方式。方法表的结构如同字段表一样，依次包括了访问标志、名称索引、描述符索引、属性表集合几项。

**method_info(方法表的)结构:**

![方法表的结构](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/方法表的结构.png)

**方法表的access_flag取值：**

![方法表的access_flag取值](https://oss.javaguide.cn/JVM/image-20201031084248965.png)

注意：因为volatile修饰符和transient修饰符不可以修饰方法，所以方法表的访问标志中没有这两个对应的标志，但是增加了synchronized、native、abstract等关键字修饰方法，所以也就多了这些关键字对应的标志。

**属性表集合（Attributes）**


```java
u2             attributes_count;//此类的属性表中的属性数
attribute_info attributes[attributes_count];//属性表集合
```

在Class文件，字段表，方法表中都可以携带自己的属性表集合，以用于描述某些场景专有的信息。与Class文件中其它的数据项目要求的顺序、长度和内容不同，属性表集合的限制稍微宽松一些，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写入自己定义的属性信息，Java虚拟机运行时会忽略掉它不认识的属性

> [原文链接](https://javaguide.cn/java/jvm/class-file-structure.html)

## 类加载过程详解


### 类的生命周期

类从被加载到虚拟机内存中开始到卸载出内存为止，它的整个生命周期可以简单概括为7个阶段：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）。其中，前三个阶段可以统称为连接（Linking）。这7个阶段的顺序如下图所示：

![一个类的完整生命周期](https://oss.javaguide.cn/github/javaguide/java/jvm/lifecycle-of-a-class.png)

### 类加载过程

Class文件需要加载到虚拟机中之后才能运行和使用，那么虚拟机是如何加载这些Class文件呢？

系统加载Class类型的文件主要三步：**加载->连接->初始化**。连接过程又可分为三步：**验证->准备->解析**。

![类加载过程](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loading-procedure.png)

> 详见[Java Virtual Machine Specification - 5.3. Creation and Loading](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html#jvms-5.3)。

#### 加载

类加载过程的第一步，主要完成下面3件事情：

1. 通过全类名获取定义此类的二进制字节流。
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构。
3. 在内存中生成一个代表该类的Class对象，作为方法区这些数据的访问入口。

虚拟机规范上面这3点并不具体，因此是非常灵活的。比如："通过全类名获取定义此类的二进制字节流"并没有指明具体从哪里获取（ZIP、JAR、EAR、WAR、网络、动态代理技术运行时动态生成、其他文件生成比如JSP...）、怎样获取。加载这一步主要是通过我们后面要讲到的**类加载器**完成的。类加载器有很多种，当我们想要加载一个类的时候，具体是哪个类加载器加载由**双亲委派模型**决定（不过，我们也能打破由双亲委派模型）。

> 类加载器、双亲委派模型也是非常重要的知识点，这部分内容在[类加载器详解](https://javaguide.cn/java/jvm/classloader.html)这篇文章中有详细介绍到

每个Java类都有一个引用指向加载它的ClassLoader。不过，数组类不是通过ClassLoader创建的，而是JVM在需要的时候自动创建的，数组类通过getClassLoader()方法获取ClassLoader的时候和该数组的元素类型的ClassLoader是一致的。一个非数组类的加载阶段（加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，这一步我们可以去完成还可以自定义类加载器去控制字节流的获取方式（重写一个类加载器的loadClass()方法）。加载阶段与连接阶段的部分动作(如一部分字节码文件格式验证动作)是交叉进行的，加载阶段尚未结束，连接阶段可能就已经开始了。

#### 验证

**验证是连接阶段的第一步，这一阶段的目的是确保Class文件的字节流中包含的信息符合《Java虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。**

验证阶段这一步在整个类加载过程中耗费的资源还是相对较多的，但很有必要，可以有效防止恶意代码的执行。任何时候，程序安全都是第一位。不过，验证阶段也不是必须要执行的阶段。如果程序运行的全部代码(包括自己编写的、第三方包中的、从外部加载的、动态生成的等所有代码)都已经被反复使用和验证过，在生产环境的实施阶段就可以考虑使用`-Xverify:none`参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。验证阶段主要由四个检验阶段组成：

1. 文件格式验证（Class文件格式检查）
2. 元数据验证（字节码语义检查）
3. 字节码验证（程序语义检查）
4. 符号引用验证（类的正确性检查）

![验证阶段示意图](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loading-process-verification.png)

文件格式验证这一阶段是基于该类的二进制字节流进行的，主要目的是保证输入的字节流能正确地解析并存储于方法区之内，格式上符合描述一个Java类型信息的要求。除了这一阶段之外，其余三个验证阶段都是基于方法区的存储结构上进行的，不会再直接读取、操作字节流了。

> 方法区属于是JVM运行时数据区域的一块逻辑区域，是各个线程共享的内存区域。当虚拟机要使用一个类时，它需要读取并解析Class文件获取相关信息，再将信息存入到方法区。方法区会存储已被虚拟机加载的**类信息、字段信息、方法信息、常量、静态变量、即时编译器编译后的代码缓存等数据**。
>
> 关于方法区的详细介绍，推荐阅读[Java内存区域详解](https://javaguide.cn/java/jvm/memory-area.html)这篇文章。

符号引用验证发生在类加载过程中的解析阶段，具体点说是JVM将符号引用转化为直接引用的时候（解析阶段会介绍符号引用和直接引用）。符号引用验证的主要目的是确保解析阶段能正常执行，如果无法通过符号引用验证，JVM会抛出异常，比如：

- java.lang.IllegalAccessError：当类试图访问或修改它没有权限访问的字段，或调用它没有权限访问的方法时，抛出该异常。
- java.lang.NoSuchFieldError：当类试图访问或修改一个指定的对象字段，而该对象不再包含该字段时，抛出该异常。
- java.lang.NoSuchMethodError：当类试图访问一个指定的方法，而该方法不存在时，抛出该异常。
- ......

#### 准备

**准备阶段是正式为类变量分配内存并设置类变量初始值的阶段**，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

1. 这时候进行内存分配的仅包括类变量（ClassVariables，即静态变量，被static关键字修饰的变量，只与类相关，因此被称为类变量），而不包括实例变量。实例变量会在对象实例化时随着对象一块分配在Java堆中。

2. 从概念上讲，类变量所使用的内存都应当在**方法区**中进行分配。不过有一点需要注意的是：JDK7之前，HotSpot使用永久代来实现方法区的时候，实现是完全符合这种逻辑概念的。而在JDK7及之后，HotSpot已经把原本放在永久代的字符串常量池、静态变量等移动到堆中，这个时候类变量则会随着Class对象一起存放在Java堆中。

   > 相关阅读：[《深入理解Java虚拟机（第3版）》勘误#75](https://github.com/fenixsoft/jvm_book/issues/75)

3. 这里所设置的初始值"通常情况"下是数据类型默认的零值（如0、0L、null、false等），比如我们定义了publicstaticintvalue=111，那么value变量在准备阶段的初始值就是0而不是111（初始化阶段才会赋值）。特殊情况：比如给value变量加上了final关键字`public static final int value=111`，那么准备阶段value的值就被赋值为111。

**基本数据类型的零值**：(图片来自《深入理解Java虚拟机》第3版7.33)

![基本数据类型的零值](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/基本数据类型的零值.png)

#### 解析

**解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程**。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符7类符号引用进行。

《深入理解Java虚拟机》7.34节第三版对符号引用和直接引用的解释如下：

![符号引用和直接引用](https://oss.javaguide.cn/github/javaguide/java/jvm/symbol-reference-and-direct-reference.png)

举个例子：在程序执行方法时，系统需要明确知道这个方法所在的位置。Java虚拟机为每个类都准备了一张方法表来存放类中所有的方法。当需要调用一个类的方法的时候，只要知道这个方法在方法表中的偏移量就可以直接调用该方法了。通过解析操作符号引用就可以直接转变为目标方法在类中方法表的位置，从而使得方法可以被调用。

综上，解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，也就是得到类或者字段、方法在内存中的指针或者偏移量。

#### 初始化

初始化阶段是执行初始化方法<clinit\>()方法的过程，是类加载的最后一步，这一步JVM才开始真正执行类中定义的Java程序代码(字节码)。

> 说明：<clinit\>()方法是编译之后自动生成的。

对于<clinit\>()方法的调用，虚拟机会自己确保其在多线程环境中的安全性。因为<clinit\>()方法是带锁线程安全，所以在多线程环境下进行类初始化的话可能会引起多个线程阻塞，并且这种阻塞很难被发现。

对于初始化阶段，虚拟机严格规范了有且只有5种情况下，必须对类进行初始化(只有主动去使用类才会初始化类)：

1. 当遇到`new`、`getstatic`、`putstatic`或`invokestatic`这4条直接码指令时，比如new一个类，读取一个静态字段(未被final修饰)、或调用一个类的静态方法时。

- 当jvm执行`new`指令时会初始化类。即当程序创建一个类的实例对象。
- 当jvm执行`getstatic`指令时会初始化类。即程序访问类的静态变量(不是静态常量，常量会被加载到运行时常量池)。
- 当jvm执行`putstatic`指令时会初始化类。即程序给类的静态变量赋值。
- 当jvm执行`invokestatic`指令时会初始化类。即程序调用类的静态方法。

2. 使用java.lang.reflect包的方法对类进行反射调用时如Class.forname("..."),newInstance()等等。如果类没初始化，需要触发其初始化。
3. 初始化一个类，如果其父类还未初始化，则先触发该父类的初始化。
4. 当虚拟机启动时，用户需要定义一个要执行的主类(包含main方法的那个类)，虚拟机会先初始化这个类。
5. MethodHandle和VarHandle可以看作是轻量级的反射调用机制，而要想使用这2个调用，就必须先使用findStaticVarHandle来初始化要调用的类。
6. 补充，来自[issue745](https://github.com/Snailclimb/JavaGuide/issues/745) 当一个接口中定义了JDK8新加入的默认方法（被default关键字修饰的接口方法）时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化。

**类的引用（主动引用，一定会初始化）**

1. 创建类的实例（new操作、反射、cloning，反序列化）
2. 访问某个类或接口的静态变量，或者对该静态变量赋值
3. 调用类的静态方法
4. 反射（Class.forName(“com.lyj.load”)）
5. 初始化一个类的子类（会首先初始化子类的父类）
6. JVM启动时标明的启动类，即文件名和类名相同的那个类

**类的引用（被动引用）**

1. 当访问一个静态域时，只有真正声明这个域的类才会被初始化。例如：通过子类引用父类的静态变量，不会导致子类初始化。
2. 通过数组定义类引用，不会触发此类的初始化。
3. 引用常量不会触发此类的初始化（常量在编译阶段就存入调用类的常量池中了）

**类加载的顺序**

1. 类初始化顺序：静态字段->静态代码块,按顺序自上而下全部执行->成员变量->非静态代码块->构造器
2. 带有继承关系的，实例化子类时，加载顺序如下：
父类的静态字段->父类静态代码块,按顺序自上而下全部执行->子类静态字段->子类静态代码块,按顺序自上而下全部执行->父类成员变量（非静态字段）->父类非静态代码块->父类构造器->子类成员变量->子类非静态代码块->子类构造器

### 总结

Java在new一个对象的时候，会先查看对象所属的类有没有被加载到内存，如果没有的话，就会先通过类的全限定名（包名+类名）来加载。加载并初始化类完成后，再进行对象的创建工作。
类加载的生命周期：加载(loading)-连接(Linking)-初始化(initialization)
1. loading:使用类加载器对类进行加载（双亲委派机制）。使用双亲委托机制的好处是：能够有效确保一个类的全局唯一性，当程序中出现多个限定名相同的类时，类加载器在执行加载时，始终只会加载其中的某一个类。通过委派的方式，可以避免类的重复加载，当父加载器已经加载过某一个类时，子加载器就不会再重新加载这个类
2. linking:连接分为验证-准备-解析三个阶段，验证的目的是确保Class文件的字节流中包含的信息符合《Java虚拟机规范》的全部约束要求。准备阶段是正式为类中定义的变量（即静态变量，被static修饰的变量）分配内存并设置类变量初始值的阶段，当类变量被final修饰时，在准备阶段就直接会被复制，不是使用初始值。解析阶段是Java虚拟机将常量池内的符号引用替换为直接引用的过程
3. initialization:初始化，真正开始执行Java代码的阶段(比如给类属性赋真实的值)执行<clinit\>方法的过程

### 相关文章

| [在Java中new一个对象的流程是怎样的？](https://mp.weixin.qq.com/s/23nIQlguz0-jaEEyTwR16g) | [Java new一个对象时发生了什么？](https://mp.weixin.qq.com/s/X-LuMvirhSQkZahFHsRl8g) | [JVM的类初始化机制](https://mp.weixin.qq.com/s/i52wFjkksaAosUorPqT-1A) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [Java类加载器解析及常见类加载问题](https://mp.weixin.qq.com/s/4yPyhMc1MW2np0sjmuilog) | [深入理解JVM-类加载机制](https://mp.weixin.qq.com/s/VKu6U58F4vo6OCzKMF2zxA) | [自己手写一个热加载](https://mp.weixin.qq.com/s/dZA3UJqfqK76zwJdhMa14g) |
| [六种方法创建Java对象](https://mp.weixin.qq.com/s/2ZDNDvJeB-WG5t3uWXvn4Q) | [JVM是如何加载Java类的？](https://mp.weixin.qq.com/s/1AXWlNXz6OG3OCyZiFU2yA) | [IDEA的debug调试为什么这么强？我挖出了背后的技术。](https://mp.weixin.qq.com/s/JIt5WCBtdHHORjeOmX5KaA) |
| [Class类文件的结构](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&amp;mid=2247494349&amp;idx=1&amp;sn=3a297393ea4a38af4fd277f4fc83c284&amp;source=41#wechat_redirect) | [一把小刀，直插class文件的小心脏](https://mp.weixin.qq.com/s/mNDWN2P1mOBL8C5qLtXG8A) | [面试官：说说类加载的几个阶段](https://mp.weixin.qq.com/s/hV65V_fDoaBMA7xqcwy2TA) |
| [3分钟带你了解对象的创建过程](https://mp.weixin.qq.com/s/Xxp6rcnrEuJvQNfjiFe46Q) | [Java类是如何加载的？※](https://mp.weixin.qq.com/s/73ehlcfUdpG27TLhYdnTpA) |                                                              |

## 类卸载

> 卸载这部分内容来自[issue#662](https://github.com/Snailclimb/JavaGuide/issues/662)由 **[guang19](https://github.com/guang19)** 补充完善。

**卸载类即该类的Class对象被GC。**

卸载类需要满足3个要求:

1. 该类的所有的实例对象都已被GC，也就是说堆不存在该类的实例对象。
2. 该类没有在其他任何地方被引用
3. 该类的类加载器的实例已被GC

所以，在JVM生命周期内，由jvm自带的类加载器加载的类是不会被卸载的。但是由我们自定义的类加载器加载的类是可能被卸载的。

只要想通一点就好了，JDK自带的BootstrapClassLoader,ExtClassLoader,AppClassLoader负责加载JDK提供的类，所以它们(类加载器的实例)肯定不会被回收。而我们自定义的类加载器的实例是可以被回收的，所以使用我们自定义加载器加载的类是可以被卸载掉的

> [原文链接](https://javaguide.cn/java/jvm/class-loading-process.html)

## 类加载器详解

**回顾一下类加载过程**

开始介绍类加载器和双亲委派模型之前，简单回顾一下类加载过程。

- 类加载过程：**加载->连接->初始化**。
- 连接过程又可分为三步：**验证->准备->解析**。

![类加载过程](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loading-procedure.png)

加载是类加载过程的第一步，主要完成下面3件事情：

1. 通过全类名获取定义此类的二进制字节流
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
3. 在内存中生成一个代表该类的Class对象，作为方法区这些数据的访问入口

### 类加载器

#### 类加载器介绍

类加载器从JDK1.0就出现了，最初只是为了满足Java Applet（已经被淘汰）的需要。后来，慢慢成为Java程序中的一个重要组成部分，赋予了Java类可以被动态加载到JVM中并执行的能力。

根据官方API文档的介绍：

> A class loader is an object that is responsible for loading classes. The class ClassLoader is an abstract class. Given the binary name of a class, a class loader should attempt to locate or generate data that constitutes a definition for the class. A typical strategy is to transform the name into a file name and then read a "class file" of that name from a file system.
>
> Every Class object contains a reference to the ClassLoader that defined it.
>
> Class objects for array classes are not created by class loaders, but are created automatically as required by the Java runtime. The class loader for an array class, as returned by Class.getClassLoader() is the same as the class loader for its element type; if the element type is a primitive type, then the array class has no class loader.

翻译过来大概的意思是：

> 类加载器是一个负责加载类的对象。ClassLoader是一个抽象类。给定类的二进制名称，类加载器应尝试定位或生成构成类定义的数据。典型的策略是将名称转换为文件名，然后从文件系统中读取该名称的“类文件”。
>
> 每个Java类都有一个引用指向加载它的ClassLoader。不过，数组类不是通过ClassLoader创建的，而是JVM在需要的时候自动创建的，数组类通过getClassLoader()方法获取ClassLoader的时候和该数组的元素类型的ClassLoader是一致的。

从上面的介绍可以看出:

- 类加载器是一个负责加载类的对象，用于实现类加载过程中的加载这一步。
- 每个Java类都有一个引用指向加载它的ClassLoader。
- 数组类不是通过ClassLoader创建的（数组类没有对应的二进制字节流），是由JVM直接生成的。


```java
class Class<T> {
  ...
  private final ClassLoader classLoader;
  @CallerSensitive
  public ClassLoader getClassLoader() {
     //...
  }
  ...
}
```

简单来说，**类加载器的主要作用就是加载Java类的字节码（.class文件）到JVM中（在内存中生成一个代表该类的Class对象）**。字节码可以是Java源程序（.java文件）经过javac编译得来，也可以是通过工具动态生成或者通过网络下载得来。其实除了加载类之外，类加载器还可以加载Java应用所需的资源如文本、图像、配置文件、视频等等文件资源。本文只讨论其核心功能：加载类。

#### 类加载器加载规则

JVM启动的时候，并不会一次性加载所有的类，而是根据需要去动态加载。也就是说，大部分类在具体用到的时候才会去加载，这样对内存更加友好。对于已经加载的类会被放在ClassLoader中。在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。也就是说，对于一个类加载器来说，相同二进制名称的类只会被加载一次。

```java
public abstract class ClassLoader {
  ...
  private final ClassLoader parent;
  // 由这个类加载器加载的类。
  private final Vector<Class<?>> classes = new Vector<>();
  // 由VM调用，用此类加载器记录每个已加载类。
  void addClass(Class<?> c) {
        classes.addElement(c);
   }
  ...
}
```

#### 类加载器总结

JVM中内置了三个重要的ClassLoader：

1. **BootstrapClassLoader（启动类加载器）**：最顶层的加载类，由C++实现，通常表示为null，并且没有父级，主要用来加载JDK内部的核心类库（%JAVA_HOME%/lib目录下的rt.jar、resources.jar、charsets.jar等jar包和类）以及被-Xbootclasspath参数指定的路径下的所有类。
2. **ExtensionClassLoader（扩展类加载器）**：主要负责加载%JRE_HOME%/lib/ext目录下的jar包和类以及被java.ext.dirs系统变量所指定的路径下的所有类。
3. **AppClassLoader（应用程序类加载器）**：面向我们用户的加载器，负责加载当前应用classpath下的所有jar包和类。

> 🌈拓展一下：
>
> - rt.jar：rt代表“RunTime”，rt.jar是Java基础类库，包含Javadoc里面看到的所有的类的类文件。也就是说，我们常用内置库java.xxx.\*都在里面，比如java.util.\*、java.io.\*、java.nio.\*、java.lang.\*、java.sql.\*、java.math.\*。
> - Java9引入了模块系统，并且略微更改了上述的类加载器。扩展类加载器被改名为平台类加载器（platformclassloader）。JavaSE中除了少数几个关键模块，比如说java.base是由启动类加载器加载之外，其他的模块均由平台类加载器所加载。

除了这三种类加载器之外，用户还可以加入自定义的类加载器来进行拓展，以满足自己的特殊需求。就比如说，我们可以对Java类的字节码（.class文件）进行加密，加载时再利用自定义的类加载器对其解密。

![类加载器层次关系图](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loader-parents-delegation-model.png)

除了BootstrapClassLoader是JVM自身的一部分之外，其他所有的类加载器都是在JVM外部实现的，并且全都继承自ClassLoader抽象类。这样做的好处是用户可以自定义类加载器，以便让应用程序自己决定如何去获取所需的类。每个ClassLoader可以通过getParent()获取其父ClassLoader，如果获取到ClassLoader为null的话，那么该类是通过BootstrapClassLoader加载的。


```java
public abstract class ClassLoader {
  ...
  // 父加载器
  private final ClassLoader parent;
  @CallerSensitive
  public final ClassLoader getParent() {
     //...
  }
  ...
}
```

**为什么获取到ClassLoader为null就是BootstrapClassLoader加载的呢**？这是因为BootstrapClassLoader由C++实现，由于这个C++实现的类加载器在Java中是没有与之对应的类的，所以拿到的结果是null。

下面我们来看一个获取ClassLoader的小案例：

```java
public class PrintClassLoaderTree {

    public static void main(String[] args) {

        ClassLoader classLoader = PrintClassLoaderTree.class.getClassLoader();

        StringBuilder split = new StringBuilder("|--");
        boolean needContinue = true;
        while (needContinue){
            System.out.println(split.toString() + classLoader);
            if(classLoader == null){
                needContinue = false;
            }else{
                classLoader = classLoader.getParent();
                split.insert(0, "\t");
            }
        }
    }

}
```

输出结果(JDK 8)：

```text
|--sun.misc.Launcher$AppClassLoader@18b4aac2
    |--sun.misc.Launcher$ExtClassLoader@53bd815b
        |--null
```

从输出结果可以看出：

- 我们编写的Java类PrintClassLoaderTree的ClassLoader是AppClassLoader；
- AppClassLoader的父ClassLoader是ExtClassLoader；
- ExtClassLoader的父ClassLoader是BootstrapClassLoader，因此输出结果为null。

#### 自定义类加载器

我们前面也说说了，除了BootstrapClassLoader其他类加载器均由Java实现且全部继承自java.lang.ClassLoader。如果我们要自定义自己的类加载器，很明显需要继承ClassLoader抽象类。ClassLoader类有两个关键的方法：

- protectedClassloadClass(String name,boolean resolve)：加载指定二进制名称的类，实现了双亲委派机制。name为类的二进制名称，resove如果为true，在加载时调用`resolveClass(Class<?>c)`方法解析该类。
- protectedClassfindClass(String name)：根据类的二进制名称来查找类，默认实现是空方法。

官方API文档中写到：

> Subclasses of `ClassLoader` are encouraged to override `findClass(String name)`, rather than this method.
>
> 建议`ClassLoader`的子类重写`findClass(String name)`方法而不是`loadClass(String name, boolean resolve)`方法。

如果我们不想打破双亲委派模型，就重写ClassLoader类中的findClass()方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。但是，如果想打破双亲委派模型则需要重写loadClass()方法。

#### ClassLoader.loadClass()、Class.forName()区别

ClassLoader.loadClass()和Class.forName()都是用于加载Java类的方法，但它们之间存在一些关键的区别：

1. 初始化行为：
   - Class.forName(className)会初始化加载的类。这意味着如果类中包含静态初始化块或静态字段，这些静态内容会在加载时被初始化。
   - ClassLoader.loadClass(className)只是加载类，不会执行任何初始化操作。除非你随后通过反射调用`.newInstance()`、访问静态字段或方法，或者调用`Class.forName(className, true, classLoader)`（第二个参数为`true`）来显式地初始化类。

2. 类加载器的使用：
   - Class.forName(className)默认使用当前线程的上下文类加载器来加载类。
   - ClassLoader.loadClass(className)需要你提供一个具体的类加载器实例来加载类。这提供了更多的灵活性，允许你控制使用哪个类加载器来加载类。
   > Class.forName(className)默认使用当前线程的上下文类加载器来加载类。如果你的当前线程的上下文类加载器能够找到并加载这个类，那么这个方法就能正常工作。而ClassLoader.getSystemClassLoader().loadClass(className)则明确使用系统类加载器来加载类。

3. 异常处理：
   - Class.forName(className)会抛出ClassNotFoundException或LinkageError的子类。
   - ClassLoader.loadClass(className)除了抛出上述异常外，还可能抛出SecurityException，因为该方法涉及到安全相关的类加载。

4. 双亲委派模型：
   - Class.forName(className)和Thread.currentThread().getContextClassLoader().loadClass(className)通常遵循双亲委派模型，即先委托给父类加载器加载，只有当父类加载器无法加载时才由当前类加载器尝试加载。
   - ClassLoader.getSystemClassLoader().loadClass(className)直接使用系统类加载器加载，可能会跳过一些类加载器，这在某些情况下可能导致类找不到。
   > Class.forName(className)和Thread.currentThread().getContextClassLoader().loadClass(className)通常会遵循双亲委托机制，而直接使用ClassLoader.getSystemClassLoader().loadClass(className)可能会跳过一些类加载器，导致类找不到。因此，如果你的类是通过系统类加载器以外的其他类加载器加载的（比如自定义类加载器或者应用类加载器），那么直接使用ClassLoader.getSystemClassLoader().loadClass(className)可能无法找到这个类，而Class.forName(className)或者Thread.currentThread().getContextClassLoader().loadClass(className)可能能够正确加载。

5. 功能和使用场景：
   - Class.forName()常用于需要触发类初始化的场景，比如在JDBC编程中加载数据库驱动类。
   - ClassLoader.loadClass()则更多地用于更精细的类加载控制，特别是在涉及自定义类加载器或者需要避免类初始化的场景。

总的来说，选择使用哪个方法取决于你的具体需求，包括是否需要初始化类、希望使用哪个类加载器以及是否需要遵循双亲委派模型等。

为了更好地控制类加载过程和解决你的问题(**使用ClassLoad加载spring boot jar包提示ClassNotFoundException**)，你可以尝试以下方法：

-  使用Thread.currentThread().getContextClassLoader().loadClass(className)代替ClassLoader.getSystemClassLoader().loadClass(className)。
- 如果你在使用自定义类加载器，确保你的类加载器遵循双亲委派模型，并且正确地委托给父类加载器。
- 检查你的类路径和类名是否正确，确保类文件可以被正确的类加载器找到。

### 双亲委派模型

#### 双亲委派模型介绍

类加载器有很多种，当我们想要加载一个类的时候，具体是哪个类加载器加载呢？这就需要提到双亲委派模型了。

根据官网介绍：

> The ClassLoader class uses a delegation model to search for classes and resources. Each instance of ClassLoader has an associated parent class loader. When requested to find a class or resource, a ClassLoader instance will delegate the search for the class or resource to its parent class loader before attempting to find the class or resource itself. The virtual machine's built-in class loader, called the "bootstrap class loader", does not itself have a parent but may serve as the parent of a ClassLoader instance.

翻译过来大概的意思是：

> ClassLoader类使用委托模型来搜索类和资源。每个ClassLoader实例都有一个相关的父类加载器。需要查找类或资源时，ClassLoader实例会在试图亲自查找类或资源之前，将搜索类或资源的任务委托给其父类加载器。虚拟机中被称为"bootstrapclassloader"的内置类加载器本身没有父类加载器，但是可以作为ClassLoader实例的父类加载器。

从上面的介绍可以看出：

- ClassLoader类使用委托模型来搜索类和资源。
- 双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。
- ClassLoader实例会在试图亲自查找类或资源之前，将搜索类或资源的任务委托给其父类加载器。

下图展示的各种类加载器之间的层次关系被称为类加载器的**双亲委派模型（ParentsDelegationModel）**。

![类加载器层次关系图](https://oss.javaguide.cn/github/javaguide/java/jvm/class-loader-parents-delegation-model.png)

注意⚠️：双亲委派模型并不是一种强制性的约束，只是JDK官方推荐的一种方式。如果我们因为某些特殊需求想要打破双亲委派模型，也是可以的，后文会介绍具体的方法。

其实这个双亲翻译的容易让别人误解，我们一般理解的双亲都是父母，这里的双亲更多地表达的是“父母这一辈”的人而已，并不是说真的有一个MotherClassLoader和一个FatherClassLoader。个人觉得翻译成单亲委派模型更好一些，不过，国内既然翻译成了双亲委派模型并流传了，按照这个来也没问题，不要被误解了就好。

另外，类加载器之间的父子关系一般不是以继承的关系来实现的，而是通常使用组合关系来复用父加载器的代码。

```java
public abstract class ClassLoader {
  ...
  // 组合
  private final ClassLoader parent;
  protected ClassLoader(ClassLoader parent) {
       this(checkCreateClassLoader(), parent);
  }
  ...
}
```

在面向对象编程中，有一条非常经典的设计原则：**组合优于继承，多用组合少用继承**。

#### 双亲委派模型的执行流程

双亲委派模型的实现代码非常简单，逻辑非常清晰，都集中在java.lang.ClassLoader的loadClass()中，相关代码如下所示。

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        //首先，检查该类是否已经加载过
        Class c = findLoadedClass(name);
        if (c == null) {
            //如果c为null，则说明该类没有被加载过
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    //当父类的加载器不为空，则通过父类的loadClass来加载该类
                    c = parent.loadClass(name, false);
                } else {
                    //当父类的加载器为空，则调用启动类加载器来加载该类
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                //非空父类的类加载器无法找到相应的类，则抛出异常
            }

            if (c == null) {
                //当父类加载器无法加载时，则调用findClass方法来加载该类
                //用户可通过覆写该方法，来自定义类加载器
                long t1 = System.nanoTime();
                c = findClass(name);

                //用于统计类加载器相关的信息
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            //对类进行link操作
            resolveClass(c);
        }
        return c;
    }
}
```

每当一个类加载器接收到加载请求时，它会先将请求转发给父类加载器。在父类加载器没有找到所请求的类的情况下，该类加载器才会尝试去加载。结合上面的源码，简单总结一下双亲委派模型的执行流程：

- 在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载（每个父类加载器都会走一遍这个流程）。
- 类加载器在进行类加载的时候，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成（调用父加载器loadClass()方法来加载类）。这样的话，所有的请求最终都会传送到顶层的启动类加载器BootstrapClassLoader中。
- 只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载（调用自己的findClass()方法来加载类）。

🌈拓展一下：

**JVM判定两个Java类是否相同的具体规则**：JVM不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样。只有两者都相同的情况，才认为两个类是相同的。即使两个类来源于同一个Class文件，被同一个虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相同。

#### 双亲委派模型的好处

双亲委派模型保证了Java程序的稳定运行，可以避免类的重复加载（JVM区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了Java的核心API不被篡改。如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为java.lang.Object类的话，那么程序运行的时候，系统就会出现两个不同的Object类。双亲委派模型可以保证加载的是JRE里的那个Object类，而不是你写的Object类。这是因为AppClassLoader在加载你的Object类时，会委托给ExtClassLoader去加载，而ExtClassLoader又会委托给BootstrapClassLoader，BootstrapClassLoader发现自己已经加载过了Object类，会直接返回，不会去加载你写的Object类。

#### 打破双亲委派模型方法

为了避免双亲委托机制，我们可以自己定义一个类加载器，然后重写loadClass()即可。

> **🐛修正（参见：[issue871](https://github.com/Snailclimb/JavaGuide/issues/871)）**：自定义加载器的话，需要继承ClassLoader。如果我们不想打破双亲委派模型，就重写ClassLoader类中的findClass()方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。但是，如果想打破双亲委派模型则需要重写loadClass()方法。

为什么是重写loadClass()方法打破双亲委派模型呢？双亲委派模型的执行流程已经解释了：

> 类加载器在进行类加载的时候，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成（调用父加载器loadClass()方法来加载类）。

我们比较熟悉的Tomcat服务器为了能够优先加载Web应用目录下的类，然后再加载其他目录下的类，就自定义了类加载器WebAppClassLoader来打破双亲委托机制。这也是Tomcat下Web应用之间的类实现隔离的具体原理。Tomcat的类加载器的层次结构如下：

![Tomcat的类加载器的层次结构](https://oss.javaguide.cn/github/javaguide/java/jvm/tomcat-class-loader-parents-delegation-model.png)

感兴趣的小伙伴可以自行研究一下Tomcat类加载器的层次结构，这有助于我们搞懂Tomcat隔离Web应用的原理，推荐资料是[《深入拆解Tomcat&Jetty》](http://gk.link/a/10Egr)。

> [原文链接](https://javaguide.cn/java/jvm/classloader.html)
> [我竟然被“双亲委派”给虐了](https://mp.weixin.qq.com/s/Q0MqcvbeI7gAcJH5ZaQWgA)
> [Tomcat为什么要破坏Java双亲委派机制？](https://mp.weixin.qq.com/s/IGV4-y1cWWOUWOETZI2hsw)