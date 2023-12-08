---
title: Spring相关
categories: Spring
index_img: /assert/spring.jpg
img: https://pic2.zhimg.com/v2-a576f4f763928a2187ec87f161b3f5bc_1440w.jpg
coverImg: https://pic4.zhimg.com/v2-13cd8daa0a4eba009b68785cfbbd5007_r.jpg
cover: true
summary: Spring注解、循环依赖、Spring MVC、XMl配置、Spring Boot
top: true

---

> 我写的关于Spring扩展插件的一个[示例](https://github.com/xmxe/springboot/tree/3.x/springboot-lifecycle)，里面有很多Spring扩展的测试。
> 其他相关文章:[Spring中的Bean对象](https://xmxe.gitee.io/blog/posts/b435885d7cf1/)

## Spring循环依赖

1. 如果是原型bean的循环依赖，Spring无法解决
2. 如果是构造参数注入的循环依赖，Spring无法解决

容器为了缓存这些单例的Bean需要一个数据结构来存储，比如Map {k:name; v:bean}。
而我们创建一个Bean就可以往Map中存入一个Bean。这时候我们仅需要一个Map就可以满足创建+缓存的需求。
但是创建Bean过程中可能会遇到循环依赖问题，比如A对象依赖了一个B对象，而B对象内部又依赖了一个A:

```java
public class A {
    B b;
}
public class B {
    A a;
}
```
假设A和B我都定义为单例的对象，并且需要在项目启动过程中自动注入，如下：
```java
@Component
public class A {
  @Autowired
  B b;
}
@Component
public class B {
  @Autowired
  A a;
}
```

> 一般来说，如果我们的代码中出现了循环依赖，则说明我们的代码在设计的过程中可能存在问题，我们应该尽量避免循环依赖的发生。不过一旦发生了循环依赖，Spring默认也帮我们处理好了，当然这并不能说明循环依赖这种代码就没问题。实际上在目前最新版的Spring中，循环依赖是要额外开启的，如果不额外配置，发生了循环依赖就直接报错了

### 一级缓存

1. 实例化A对象。
2. 填充A的属性阶段时需要去填充B对象，而此时B对象还没有创建，所以这里为了完成A的填充就必须要先去创建B对象；
3. 实例化B对象。
4. 执行到B对象的填充属性阶段，又会需要去获取A对象，而此时Map中没有A，因为A还没有创建完成，导致又需要去创建A对象。
这样，就会循环往复，一直创建下去，只到堆栈溢出。

**为什么不能在实例化A之后就放入Map**？因为此时A尚未创建完整，所有属性都是默认值，并不是一个完整的对象，在执行业务时可能会抛出未知的异常。所以必须要在A创建完成之后才能放入Map。

### 二级缓存

此时我们引入二级缓存用另外一个Map2 {k:name;v:earlybean}来存储尚未已经开始创建但是尚未完整创建的对象。

1. 实例化A对象之后，将A对象放入Map2中。
2. 在填充A的属性阶段需要去填充B对象，而此时B对象还没有创建，所以这里为了完成A的填充就必须要先去创建B对象。
3. 创建B对象的过程中，实例化B对象之后，将B对象放入Map2中。
4. 执行到B对象填充属性阶段，又会需要去获取A对象，而此时Map中没有A，因为A还没有创建完成，但是我们继续从Map2中拿到尚未创建完毕的A的引用赋值给a字段。这样B对象其实就已经创建完整了，尽管B.a对象是一个还未创建完成的对象。
5. 此时将B放入Map并且从Map2中删除。
6. 这时候B创建完成，A继续执行b的属性填充可以拿到B对象，这样A也完成了创建。B.a也完整了
7. 此时将A对象放入Map并从Map2中删除。

**二级缓存已然解决了循环依赖问题，为什么还需要三级缓存**？

从上面的流程中我们可以看到使用两级缓存可以完美解决循环依赖的问题，但是Spring中还有另外一个问题需要解决，这就是初始化过程中的AOP实现。AOP是Spring的重要功能，实现方式就是使用代理模式动态增强类的功能。动态单例目前有两种技术可以实现，一种是JDK自带的基于接口的动态Proxy技术，一种是CGlib基于字节码动态生成的Proxy技术，这两种技术都是需要原始对象创建完毕，之后基于原始对象生成代理对象的。那么我们发现，在二级缓存的设计下，我们需要在放入缓存Map之前将代理对象生成好。
将流程改为：

1. 实例化Bean对象，为Bean对象在内存中分配空间，各属性赋值为默认值
2. 如果有动态代理，生成Bean对象的代理Proxy对象
3. 初始化Proxy对象，为Bean对象填充属性
4. 将Proxy放入缓存
这样虽然也可以解决，AOP的问题，但是我们知道Spring中AOP的实现是通过后置处理器BeanPostProcessor机制来实现的，而后置处理器是在填充属性结束后才执行的。流程如下：
1. 实例化对象
2. 对象填充属性
3. BeanPostProcessor doBefore
4. init-method
5. BeanPostProcessor doAfter --AOP是在这个阶段实现的
所以要实现上面的方案，势必需要将BeanPostProcessor阶段提前或者侵入到填充属性的流程中，那么从程序设计上来说，这样做肯定是不美的

> 面试官会问：为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？
> 答：如果要使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理，这样违背了Spring设计的原则，Spring在设计之初就是通过AnnotationAwareAspectJAutoProxyCreator这个后置处理器来在Bean生命周期的最后一步来完成AOP代理，而不是在实例化后就立马进行AOP代理

### 三级缓存

Spring引入了第三级缓存来解决这个问题，Map3 {k:name v:ObjectFactory}，这个缓存的value就不是Bean对象了，而是一个接口对象由一段lamda表达式实现。在这段lamda表达式中去完成一些BeanPostProcessor的执行。
1. 实例化A对象之后，将A的ObjectFactory对象放入Map3中。
2. 在填充A的属性阶段需要去填充B对象，而此时B对象还没有创建，所以这里为了完成A的填充就必须要先去创建B对象。
3. 创建B对象的过程中，实例化B的ObjectFactory对象之后，将B对象放入Map2中。
4. 执行到B对象填充属性阶段，又会需要去获取A对象，而此时Map1中没有A，因为A还没有创建完成，但是我们继续从Map2中也拿不到，到Map3中获取了A的ObjectFactory对象，通过ObjectFactory对象获取A的早期对象，并将这个早期对象放入Map2中，同时删除Map3中的A，将尚未创建完毕的A的引用赋值给a字段。这样B对象其实就已经创建完整了，尽管B.a对象是一个还未创建完成的对象。
5. 此时将B放入Map并且从Map3中删除。
6. 这时候B创建完成，A继续执行b的属性填充可以拿到B对象，这样A也完成了创建。
7. 此时将A对象放入Map并从Map2中删除。

### 带图解析

![](https://mmbiz.qpic.cn/sz_mmbiz_png/GvtDGKK4uYl7Vdb6FD5gpKdIVy6ibibEbgSFMjDAgsG3lvnhB8NBlP5fts8ZiaT2fKk4pOmVGhDk5Hjwh0T38xvDg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

我们在这里引入了一个缓存池。当我们需要创建AService的实例的时候，会首先通过Java反射创建出来一个原始的AService，这个原始AService可以简单理解为刚刚new出来（实际是刚刚通过反射创建出来）还没设置任何属性的AService，此时，我们把这个AService先存入到一个缓存池中。

接下来我们就需要给AService的属性设置值了，同时还要处理AService的依赖，这时我们发现AService依赖BService，那么就去创建BService对象，结果创建BService的时候，发现BService依赖AService，那么此时就先从缓存池中取出来AService先用着，然后继续BService创建的后续流程，直到BService创建完成后，将之赋值给AService，此时AService和BService就都创建完成了。

可能有小伙伴会说，BService从缓存池中拿到的AService是一个半成品，并不是真正的最终的AService，但是小伙伴们要知道，Java是引用传递（也可以认为是值传递，只不过这个值是内存地址），BService当时拿到的是AService的引用，说白了就是一块内存地址而已，根据这个地址找到的就是AService，所以，后续如果AService创建完成后，BService所拿到的AService就是完整的AService了。

那么上面提到的这个缓存池，在Spring容器中有一个专门的名字，就叫做**earlySingletonObjects**，这是Spring三级缓存中的**二级缓存**，这里保存的是刚刚通过反射创建出来的Bean，这些Bean还没有经历过完整生命周期，Bean的属性可能都还没有设置，Bean需要的依赖都还没有注入进来。另外两级缓存分别是：

**singletonObjects**：这是**一级缓存**，一级缓存中保存的是所有经历了完整生命周期的Bean，即一个Bean从创建、到属性赋值、到各种处理器的执行等等，都经历过了，就存到singletonObjects中，当我们需要获取一个Bean的时候，首先会去一级缓存中查找，当一级缓存中没有的时候，才会考虑去二级缓存。
**singletonFactories**：这是**三级缓存**。在一级缓存和二级缓存中，缓存的key是beanName，缓存的value则是一个Bean对象，但是在三级缓存中，缓存的value是一个Lambda表达式，通过这个Lambda表达式可以创建出来目标对象的一个代理对象。
有的小伙伴可能会觉得奇怪，按照上文的介绍，一级缓存和二级缓存就足以解决循环依赖了，为什么还冒出来一个三级缓存？那就得考虑AOP的情况了！

**AOP的创建流程**

正常来说是我们首先通过反射获取到一个Bean的实例，然后就是给这个Bean填充属性，属性填充完毕之后，接下来就是执行各种BeanPostProcessor了，如果这个Bean中有需要代理的方法，那么系统就会自动配置对应的后置处理器，举一个简单例子，假设我有如下一个Service：
```java
@Service
public class UserService {
    @Async
    public void hello() {
        System.out.println("hello>>>"+Thread.currentThread().getName());
    }
}
```
那么系统就会自动提供一个名为AsyncAnnotationBeanPostProcessor的处理器，在这个处理器中，系统会生成一个代理的UserService对象，并用这个对象代替原本的UserService。那么小伙伴们要搞清楚的是，原本的UserService和新生成的代理的UserService是两个不同的对象，占两块不同的内存地址！！！

我们再来回顾下面这张图：
![](https://mmbiz.qpic.cn/sz_mmbiz_png/GvtDGKK4uYl7Vdb6FD5gpKdIVy6ibibEbgSFMjDAgsG3lvnhB8NBlP5fts8ZiaT2fKk4pOmVGhDk5Hjwh0T38xvDg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如果AService最终是要生成一个代理对象的话，那么AService存到缓存池的其实还是原本的AService，因为此时还没到处理AOP那一步（要先给各个属性赋值，然后才是AOP处理），这就导致BService从缓存池里拿到的AService是原本的AService，等到BService创建完毕之后，AService的属性赋值才完成，接下来在AService后续的创建流程中，AService会变成了一个代理对象了，不是缓存池里的AService了，最终就导致BService所依赖的AService和最终创建出来的AService不是同一个。

为了解决这个问题，Spring引入了三级缓存`singletonFactories`。`singletonFactories`的工作机制是这样的（假设AService最终是一个代理对象）：当我们创建一个AService的时候，通过反射刚把原始的AService创建出来之后，先去判断当前一级缓存中是否存在当前Bean，如果不存在，则首先向三级缓存中添加一条记录，记录的key就是当前Bean的beanName，value则是一个Lambda表达式ObjectFactory，通过执行这个Lambda可以给当前AService生成代理对象。然后如果二级缓存中存在当前AService Bean，则移除掉。现在继续去给AService各个属性赋值，结果发现AService需要BService，然后就去创建BService，创建BService的时候，发现BService又需要用到AService，于是就先去一级缓存中查找是否有AService，如果有，就使用，如果没有，则去二级缓存中查找是否有AService，如果有，就使用，如果没有，则去三级缓存中找出来那个ObjectFactory，然后执行这里的getObject方法，这个方法在执行的过程中，会去判断是否需要生成一个代理对象，如果需要就生成代理对象返回，如果不需要生成代理对象，则将原始对象返回即可。最后，把拿到手的对象存入到二级缓存中以备下次使用，同时删除掉三级缓存中对应的数据。这样AService所依赖的BService就创建好了。接下来继续去完善AService，去执行各种后置的处理器，此时，有的后置处理器想给AService生成代理对象，发现AService已经是代理对象了，就不用生成了，直接用已有的代理对象去代替AService即可。至此，AService和BService都搞定。本质上，singletonFactories是把AOP的过程提前了。

总的来说，Spring解决循环依赖把握住两个关键点：

- 提前暴露：刚刚创建好的对象还没有进行任何赋值的时候，将之暴露出来放到缓存中，供其他Bean提前引用（二级缓存）。
- 提前AOP：A依赖B的时候，去检查是否发生了循环依赖（检查的方式就是将正在创建的A标记出来，然后B需要A，B去创建A的时候，发现A正在创建，就说明发生了循环依赖），如果发生了循环依赖，就提前进行AOP处理，处理完成后再使用（三级缓存）。

原本AOP这个过程是属性赋完值之后，再由各种后置处理器去处理AOP的（AbstractAutoProxyCreator），但是如果发生了循环依赖，就先AOP，然后属性赋值，最后等到后置处理器执行的时候，就不再做AOP的处理了。不过需要注意，三级缓存并不能解决所有的循环依赖

> [如何通过三级缓存解决Spring循环依赖※](https://mp.weixin.qq.com/s/ig22T20Ie3jmTLhuPVPmdA)
> [Spring能解决所有循环依赖吗？](https://mp.weixin.qq.com/s/Un8pyET2XDXpDY4FnRbwXw)

### 相关文章

- [Spring三级缓存解决循环依赖](https://mp.weixin.qq.com/s/ns9JEpvMt7U-nsMZzEUIUQ)
- [终于有人把Spring循环依赖讲清楚了！](https://mp.weixin.qq.com/s/L1PJ-cikoS8sOORszEYnfw)
- [烂大街的Spring循环依赖该如何回答？](https://mp.weixin.qq.com/s/5VHU2qRQMPL0IOZuEOPmQA)
- [spring：我是如何解决循环依赖的？](https://mp.weixin.qq.com/s/7S9wVOVJyoHiC_RnhZrJTw)
- [Spring为何需要三级缓存解决循环依赖，而不是二级缓存？](https://mp.weixin.qq.com/s/BaRlMNo0HlPP9Vz4x9bUaA)
- [图解Spring循环依赖，写得太好了！](https://mp.weixin.qq.com/s/JuS6aewMXSp22zjwWCKt6g)
- [Spring是如何解决循环依赖的](https://mp.weixin.qq.com/s/e7e-Pct5CcMrHBCsdjsLSQ)
- [Spring面试题之循环依赖的理解](https://mp.weixin.qq.com/s/amgsB3MMvcA2pw9N2wECDw)
- [Spring的循环依赖，到底是什么样的](https://mp.weixin.qq.com/s/qWWjWpIbghj5v6-KCd8xxA)
- [面试官:SpringBoot循环依赖，如何解决？](https://mp.weixin.qq.com/s/YSXfAn8n313TMFIKM92LPw)
- [透过源码，捋清楚循环依赖到底是如何解决的！](https://mp.weixin.qq.com/s/YIokfCvLKLhcsEpO734Qtg)

## Spring相关注解

### @RequestBody

主要用来接收前端传递给后端的json字符串中的数据的(请求体中的数据的),GET方式无请求体，所以使用@RequestBody接收数据时，前端不能使用GET方式提交数据，而是用POST方式进行提交,在方法里面标记，可以作为一个对象接收,也可以作为字符串接收，关键在于Spring中对json的解析配置.
```js
// jquery使用下面的请求时可以使用@RequestBody注解接收参数
var data =  {"id" : $("#id").val(),"userId" : $("#userId").val()}
$.ajax({
	url : "/api/updateFeedback",
	async : false,
	type : "POST",
	contentType : 'application/json',
	dataType : 'json',
	data :JSON.stringify(data),
	success : function(data) {}
});
```
> [@RequestBody接收数组、List参数、@Deprecated标记废弃方法](https://mp.weixin.qq.com/s/iyubfxmV_8KU8v4cg1A7tg)

### @PostConstruct和@PreDestroy

@PostConstruct该注解被用来修饰一个非静态的void()方法,被@PostConstruct修饰的方法会当bean创建完成的时候，会后置执行@PostConstruct修饰的方法。PostConstruct在构造函数之后执行，bean的init()方法之前执行,相当于init-method,使用在方法上，当Bean初始化时执行,Constructor(构造方法) -> @Autowired(依赖注入) -> @PostConstruct(注释的方法)

@PreDestroy类似于destory-method在servlet destory()方法之后执行


### @Configuration和@Component的区别
一句话概括就是@Configuration中所有带@Bean注解的方法都会被动态代理，因此调用该方法返回的都是同一个实例。
理解：调用@Configuration类中的@Bean注解的方法，返回的是同一个实例；而调用@Component类中的@Bean注解的方法，返回的是一个新的实例。

> [终于搞懂了@Configuration和@Component的区别](https://mp.weixin.qq.com/s/-_h5Hz6MOBb8TK3qm9gBog)
> [@Configuration和@Component有何区别？](https://mp.weixin.qq.com/s/D84pWlXs7wbHFYvCE5TAVQ)

### 条件注解
> [Spring Boot中条件注解底层如何实现的？](https://mp.weixin.qq.com/s/XhNTfz6nw-rfP2avh0owAQ)

## Spring设计模式

### 工厂设计模式

Spring使用工厂模式可以通过BeanFactory或ApplicationContext创建bean对象。

**两者对比**：

- BeanFactory：延迟注入(使用到某个bean的时候才会注入),相比于ApplicationContext来说会占用更少的内存，程序启动速度更快。
- ApplicationContext：容器启动的时候，不管你用没用到，一次性创建所有bean。BeanFactory仅提供了最基本的依赖注入支持，ApplicationContext扩展了BeanFactory,除了有BeanFactory的功能还有额外更多功能，所以一般开发人员使用ApplicationContext会更多。

ApplicationContext的三个实现类：

1. ClassPathXmlApplication：把上下文文件当成类路径资源。
2. FileSystemXmlApplication：从文件系统中的XML文件载入上下文定义信息。
3. XmlWebApplicationContext：从Web系统中的XML文件载入上下文定义信息。

Example:

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;

public class App {
	public static void main(String[] args) {
		ApplicationContext context = new FileSystemXmlApplicationContext(
				"C:/work/IOC Containers/springframework.applicationcontext/src/main/resources/bean-factory-config.xml");

		HelloApplicationContext obj = (HelloApplicationContext) context.getBean("helloApplicationContext");
		obj.getMsg();
	}
}
```

### 单例设计模式

在我们的系统中，有一些对象其实我们只需要一个，比如说：线程池、缓存、对话框、注册表、日志对象、充当打印机、显卡等设备驱动程序的对象。事实上，这一类对象只能有一个实例，如果制造出多个实例就可能会导致一些问题的产生，比如：程序的行为异常、资源使用过量、或者不一致性的结果。

**使用单例模式的好处**:

- 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
- 由于new操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻GC压力，缩短GC停顿时间。

**Spring中bean的默认作用域就是singleton(单例)的**。除了singleton作用域，Spring中bean还有下面几种作用域：

- **prototype**：每次获取都会创建一个新的bean实例。也就是说，连续getBean()两次，得到的是不同的Bean实例。
- **request（仅Web应用可用）**:每一次HTTP请求都会产生一个新的bean（请求bean），该bean仅在当前HTTPrequest内有效。
- **session（仅Web应用可用）**:每一次来自新session的HTTP请求都会产生一个新的bean（会话bean），该bean仅在当前HTTPsession内有效。
- **application/global-session（仅Web应用可用）**：每个Web应用在启动时创建一个Bean（应用Bean），该bean仅在当前应用启动时间内有效。
- **websocket（仅Web应用可用）**：每一次WebSocket会话产生一个新的bean。

> [详解Spring的6种内置作用域及其应用场景](https://mp.weixin.qq.com/s/FB-eiiYIaGUY1Y20qBxanw)

Spring通过ConcurrentHashMap实现单例注册表的特殊方式实现单例模式。Spring实现单例的核心代码如下：

```java
// 通过ConcurrentHashMap（线程安全）实现单例注册表
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            // 检查缓存中是否存在实例
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                // ...省略了很多代码
                try {
                    singletonObject = singletonFactory.getObject();
                }
                // ...省略了很多代码
                // 如果实例对象在不存在，我们注册到单例注册表中。
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }
    // 将对象添加到单例注册表
    protected void addSingleton(String beanName, Object singletonObject) {
            synchronized (this.singletonObjects) {
                this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

            }
        }
}
```

**单例Bean存在线程安全问题吗？**

大部分时候我们并没有在项目中使用多线程，所以很少有人会关注这个问题。单例Bean存在线程问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的。常见的有两种解决办法：

1. 在Bean中尽量避免定义可变的成员变量。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中（推荐的一种方式）。

不过，大部分Bean实际都是无状态（没有实例变量）的（比如Dao、Service），这种情况下，Bean是线程安全的。

### 代理设计模式

**代理模式在AOP中的应用**

**AOP(Aspect-Oriented Programming，面向切面编程)**,能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。**Spring AOP就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDKProxy**去创建代理对象，而对于没有实现接口的对象，就无法使用JDKProxy去进行代理了，这时候Spring AOP会使用**Cglib**生成一个被代理对象的子类来作为代理，如下图所示：

![SpringAOPProcess](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/SpringAOPProcess.jpg)

当然，你也可以使用AspectJ,SpringAOP已经集成了AspectJ，AspectJ应该算的上是Java生态系统中最完整的AOP框架了。使用AOP之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了AOP。

#### Spring AOP和AspectJ AOP有什么区别?

**Spring AOP属于运行时增强，而AspectJ是编译时增强**。Spring AOP基于代理(Proxying)，而AspectJ基于字节码操作(Bytecode Manipulation)。Spring AOP已经集成了AspectJ，AspectJ应该算的上是Java生态系统中最完整的AOP框架了。AspectJ相比于Spring AOP功能更加强大，但是Spring AOP相对来说更简单，如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择AspectJ，它比Spring AOP快很多。

### 模板方法模式

模板方法模式是一种行为设计模式，它定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤的实现方式。

```java
public abstract class Template {
    //这是我们的模板方法
    public final void TemplateMethod(){
        PrimitiveOperation1();
        PrimitiveOperation2();
        PrimitiveOperation3();
    }

    protected void  PrimitiveOperation1(){
        //当前类实现
    }

    //被子类实现的方法
    protected abstract void PrimitiveOperation2();
    protected abstract void PrimitiveOperation3();

}
public class TemplateImpl extends Template {

    @Override
    public void PrimitiveOperation2() {
        //当前类实现
    }

    @Override
    public void PrimitiveOperation3() {
        //当前类实现
    }
}
```

Spring中JdbcTemplate、HibernateTemplate等以Template结尾的对数据库操作的类，它们就使用到了模板模式。一般情况下，我们都是使用继承的方式来实现模板模式，但是Spring并没有使用这种方式，而是使用Callback模式与模板方法模式配合，既达到了代码复用的效果，同时增加了灵活性。

### 观察者模式

观察者模式是一种对象行为型模式。它表示的是一种对象与对象之间具有依赖关系，当一个对象发生改变的时候，这个对象所依赖的对象也会做出反应。Spring事件驱动模型就是观察者模式很经典的一个应用。Spring事件驱动模型非常有用，在很多场景都可以解耦我们的代码。比如我们每次添加商品的时候都需要重新更新商品索引，这个时候就可以利用观察者模式来解决这个问题。

#### Spring事件驱动模型中的三种角色

##### 事件角色

`ApplicationEvent`(`org.springframework.context`包下)充当事件的角色,这是一个抽象类，它继承了`java.util.EventObject`并实现了`java.io.Serializable`接口。

Spring中默认存在以下事件，他们都是对`ApplicationContextEvent`的实现(继承自`ApplicationContextEvent`)：

- ContextStartedEvent：ApplicationContext启动后触发的事件;
- ContextStoppedEvent：ApplicationContext停止后触发的事件;
- ContextRefreshedEvent：ApplicationContext初始化或刷新完成后触发的事件;
- ContextClosedEvent：ApplicationContext关闭后触发的事件。

![ApplicationEvent-Subclass](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/ApplicationEvent-Subclass.png)

##### 事件监听者角色

`ApplicationListener`充当了事件监听者角色，它是一个接口，里面只定义了一个`onApplicationEvent()`方法来处理`ApplicationEvent`。`ApplicationListener`接口类源码如下，可以看出接口定义看出接口中的事件只要实现了`ApplicationEvent`就可以了。所以，在Spring中我们只要实现`ApplicationListener`接口的`onApplicationEvent()`方法即可完成监听事件

```java
package org.springframework.context;
import java.util.EventListener;
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
    void onApplicationEvent(E var1);
}
```

##### 事件发布者角色

`ApplicationEventPublisher`充当了事件的发布者，它也是一个接口。

```java
@FunctionalInterface
public interface ApplicationEventPublisher {
    default void publishEvent(ApplicationEvent event) {
        this.publishEvent((Object)event);
    }

    void publishEvent(Object var1);
}
```

`ApplicationEventPublisher`接口的`publishEvent()`这个方法在`AbstractApplicationContext`类中被实现，阅读这个方法的实现，你会发现实际上事件真正是通过`ApplicationEventMulticaster`来广播出去的。具体内容过多，就不在这里分析了，后面可能会单独写一篇文章提到。

#### Spring的事件流程总结

1. 定义一个事件:实现一个继承自ApplicationEvent，并且写相应的构造函数；
2. 定义一个事件监听者：实现ApplicationListener接口，重写onApplicationEvent()方法；
3. 使用事件发布者发布消息:可以通过ApplicationEventPublisher的publishEvent()方法发布消息。

Example:


```java
// 定义一个事件,继承自ApplicationEvent并且写相应的构造函数
public class DemoEvent extends ApplicationEvent{
    private static final long serialVersionUID = 1L;

    private String message;

    public DemoEvent(Object source,String message){
        super(source);
        this.message = message;
    }

    public String getMessage() {
         return message;
          }


// 定义一个事件监听者,实现ApplicationListener接口，重写onApplicationEvent()方法；
@Component
public class DemoListener implements ApplicationListener<DemoEvent>{

    //使用onApplicationEvent接收消息
    @Override
    public void onApplicationEvent(DemoEvent event) {
        String msg = event.getMessage();
        System.out.println("接收到的信息是："+msg);
    }

}
// 发布事件，可以通过ApplicationEventPublisher的publishEvent()方法发布消息。
@Component
public class DemoPublisher {

    @Autowired
    ApplicationContext applicationContext;

    public void publish(String message){
        //发布事件
        applicationContext.publishEvent(new DemoEvent(this, message));
    }
}
```

当调用DemoPublisher的publish()方法的时候，比如demoPublisher.publish("你好")，控制台就会打印出:接收到的信息是：你好。

### 适配器模式

适配器模式(Adapter Pattern)将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作。

#### Spring AOP中的适配器模式

我们知道Spring AOP的实现是基于代理模式，但是Spring AOP的增强或通知(Advice)使用到了适配器模式，与之相关的接口是AdvisorAdapter。Advice常用的类型有：BeforeAdvice（目标方法调用前,前置通知）、AfterAdvice（目标方法调用后,后置通知）、AfterReturningAdvice(目标方法执行结束后，return之前)等等。每个类型Advice（通知）都有对应的拦截器:MethodBeforeAdviceInterceptor、AfterReturningAdviceInterceptor、ThrowsAdviceInterceptor等等。Spring预定义的通知要通过对应的适配器，适配成MethodInterceptor接口(方法拦截器)类型的对象（如：MethodBeforeAdviceAdapter通过调用getInterceptor方法，将MethodBeforeAdvice适配成MethodBeforeAdviceInterceptor）。

#### Spring MVC中的适配器模式

在SpringMVC中，DispatcherServlet根据请求信息调用HandlerMapping，解析请求对应的Handler。解析到对应的Handler（也就是我们平常说的Controller控制器）后，开始由HandlerAdapter适配器处理。HandlerAdapter作为期望接口，具体的适配器实现类用于对目标类进行适配，Controller作为需要适配的类。

**为什么要在SpringMVC中使用适配器模式？**

SpringMVC中的Controller种类众多，不同类型的Controller通过不同的方法来对请求进行处理。如果不利用适配器模式的话，DispatcherServlet直接获取对应类型的Controller，需要的自行来判断，像下面这段代码一样：


```java
if(mappedHandler.getHandler() instanceof MultiActionController){
   ((MultiActionController)mappedHandler.getHandler()).xxx
}else if(mappedHandler.getHandler() instanceof XXX){
    ...
}else if(...){
   ...
}
```

假如我们再增加一个Controller类型就要在上面代码中再加入一行判断语句，这种形式就使得程序难以维护，也违反了设计模式中的开闭原则–对扩展开放，对修改关闭。

### 装饰者模式

装饰者模式可以动态地给对象添加一些额外的属性或行为。相比于使用继承，装饰者模式更加灵活。简单点儿说就是当我们需要修改原有的功能，但我们又不愿直接去修改原有的代码时，设计一个Decorator套在原有代码外面。其实在JDK中就有很多地方用到了装饰者模式，比如InputStream家族，InputStream类下有FileInputStream(读取文件)、BufferedInputStream(增加缓存,使读取文件速度大大提升)等子类都在不修改InputStream代码的情况下扩展了它的功能。

![装饰者模式示意图](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/Decorator.jpg)

Spring中配置DataSource的时候，DataSource可能是不同的数据库和数据源。我们能否根据客户的需求在少修改原有类的代码下动态切换不同的数据源？这个时候就要用到装饰者模式(这一点我自己还没太理解具体原理)。Spring中用到的包装器模式在类名上含有Wrapper或者Decorator。这些类基本上都是动态地给一个对象添加一些额外的职责

### 总结

Spring框架中用到了哪些设计模式？

- **工厂设计模式**:Spring使用工厂模式通过BeanFactory、ApplicationContext创建bean对象。
- **代理设计模式**:Spring AOP功能的实现。
- **单例设计模式**:Spring中的Bean默认都是单例的。
- **模板方法模式**:Spring中jdbcTemplate、hibernateTemplate等以Template结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式**:我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式**:Spring事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式**:Spring AOP的增强或通知(Advice)使用到了适配器模式、Spring MVC中也是用到了适配器模式适配Controller。
- .....

> [原文链接](https://javaguide.cn/system-design/framework/spring/spring-design-patterns-summary.html)
> [Spring用到了哪些设计模式](https://mp.weixin.qq.com/s/O-gUDExJc5AKwb-t4ZDkNA)
> [Spring中经典的9种设计模式](https://mp.weixin.qq.com/s/gz2-izPrgW1AGbqqovT0cA)

## Spring MVC

### 组件

**Handle**:Handler是一个Controller的对象和请求方式的组合的一个Object对象
**HandleExcutionChains**：是HandleMapping返回的一个处理执行链，它是对Handle的二次封装，将拦截器关联到一起。然后，在DispatcherServlert中完成了拦截器链对handler的过滤。**DispatcherServlet**要将一个请求交给哪个特定的Controller，它需要咨询一个Bean——这个Bean为“HandlerMapping”。HandlerMapping是把一个URL指定到一个Controller上，（就像应用系统的web.xml文件使用<servlet-mapping\>将URL映射到servlet）。
**DispatcherServlet**：作为前端控制器，整个流程控制的中心，控制其它组件执行，统一调度，降低组件之间的耦合性，提高每个组件的扩展性。
作用：接收请求，响应结果，相当于转发器，中央处理器。有了dispatcherServlet减少了其它组件之间的耦合度。用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请,dispatcherServlet的存在降低了组件之间的耦合性
**HandlerMapping**：通过扩展处理器映射器实现不同的映射方式，作用:根据请求的url查找Handler,HandlerMapping负责根据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等
**HandlAdapter**：通过扩展处理器适配器，支持更多类型的处理器。
作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler，通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。
**ViewResolver**：通过扩展视图解析器，支持更多类型的视图解析，例如：jsp、freemarker、pdf、excel等。作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）
作用：View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

### DispatcherServlet的工作流程

![](/images/DispatcherServlet的工作流程.png)

1. 向服务器发送HTTP请求，请求被前端控制器DispatcherServlet捕获。
2. DispatcherServlet根据-servlet.xml中的配置对请求的URL进行解析，得到请求资源标识符（URI）。然后根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回。
3. DispatcherServlet根据获得的Handler，选择一个合适的HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(…)方法）。
4. 提取Request中的模型数据，填充Handler入参，开始执行Handler(Controller)。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：HttpMessageConveter：将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息。
数据转换：对请求消息进行数据转换。如String转换成Integer、Double等。数据根式化：对请求消息进行数据格式化。如将字符串转换成格式化数字或格式化日期等。数据验证：验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中。
5. Handler(Controller)执行完成后，向DispatcherServlet返回一个ModelAndView对象；
6. 根据返回的ModelAndView，选择一个适合的ViewResolver(必须是已经注册到Spring容器中的ViewResolver)返回给DispatcherServlet。
7. ViewResolver结合Model和View，来渲染视图。
8. 视图负责将渲染结果返回给客户端。

again
1. 用户发送请求至前端控制器DispatcherServlet。
2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。
3. 处理器映射器找到具体的处理器（controller或者handle）(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
4. DispatcherServlet调用HandlerAdapter处理器适配器。
5. HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。
6. Controller执行完成返回ModelAndView。
7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。
8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器。
ViewReslover解析后返回具体View。
9. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。
10. DispatcherServlet响应用户。

核心架构的具体流程步骤如下：
1. 首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；
2. DispatcherServlet——>HandlerMapping，HandlerMapping将会把请求映射为HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象，通过这种策略模式，很容易添加新的映射策略；
3. DispatcherServlet——>HandlerAdapter，HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；
4. HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个ModelAndView对象（包含模型数据、逻辑视图名）；
5. ModelAndView的逻辑视图名——>ViewResolver，ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；
6. View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技术；
7. 返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。
下边两个组件通常情况下需要开发：
Handler：处理器，即后端控制器用controller表示。
View：视图，即展示给用户的界面，视图中通常需要标签语言展示模型数据。

### Spring MVC相关文章

- [SpringMVC执行过程解析](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&amp;mid=2247494012&amp;idx=2&amp;sn=1c5052c5b4a5547a21449412f619327b&amp;source=41#wechat_redirect)
- [Spring MVC请求处理过程不是两张流程图就能讲清楚的](https://mp.weixin.qq.com/s/klCAv0TM2tvgy4xqoC1hQw)
- [Spring MVC初始化流程分析](https://mp.weixin.qq.com/s/IeMOfnXhOX5RCf4i5Xsdzw)
- [Spring MVC源码分析之DispatcherServlet](https://mp.weixin.qq.com/s/F0QZ-Ukgtn3oC6a4loM9Vg)
- [Spring MVC九大组件之HandlerMapping深入分析](https://mp.weixin.qq.com/s/0x7_OXPDFX5BqF0jGxN2Vg)
- [Spring MVC九大组件之HandlerAdapter深入分析](https://mp.weixin.qq.com/s/NCnawbIaLUQJNZiDi2yeCw)
- [Spring MVC九大组件之ViewResolver深入分析](https://mp.weixin.qq.com/s/rn-6QyuYIsM_P5b4B1OIrg)
- [编写Spring MVC控制器的14个技巧！涨知识了！](https://mp.weixin.qq.com/s/685jUKqg6I6r0RbmirsRxg)
- [SpringMVC异常处理体系深入分析](https://mp.weixin.qq.com/s/ZKBQSCMPV7T9yNcMQ5_pQQ)
- [使用Spring MVC的14个顶级技巧！](https://mp.weixin.qq.com/s/DAWdH_0VWZp3oaws_leJhQ)
- [Spring5里边的新玩法！这种URL请求让我涨见识了](https://mp.weixin.qq.com/s/uzI-ilR-cv5iX3HWFUOmXA)


## [Spring Boot](https://github.com/xmxe/springboot)

**配置文件默认的查找路径如下**：
```
file:./config/
file:./
classpath:/config/
classpath:/
```
配置⽂件名可以通过`spring.config.name`修改，最简单的⽅法是放置⼀个配置⽂件到jar包同层⽬录下，或是同层⽬录下的config⼦⽬录下，启动jar包即可加载配置⽂件实现配置项的覆盖。spring boot指定外部的配置⽂件,可以通过修改启动参数的值来指定加载⽬录或是加载⽂件:`spring.config.location`

```shell
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties.
```
> Spring Cloud项目加载bootstrap.yml使用`--spring.cloud.bootstrap.location`

这样不会去默认位置加载配置⽂件，⽽是加载类路径下default.properties和override.properties的⽂件，override.properties中的同名配置会覆盖default.properties,如果指定的路径是以/结尾则是⽬录配置，会去⽬录下找配置⽂件。

> [如何不重新编译让Spring Boot配置文件生效](https://mp.weixin.qq.com/s/pNAU_w6RQIjzxfaadjV_pA)

**特定配置**
在开发、测试、发布过程中，这三个场景⽐较固定，通常会定义三份不同的配置application-{profile}.yml，在使⽤时通过profile参数来切换。applicaiton-dev.yml，applicaiton-test.yml，applicaiton-prd.yml启动时，通过指定spring.profiles.active参数来切换配置⽂件

springboot项⽬启动的时候可以直接使⽤java -jar xxxjar这样。
1. -DpropName=propValue的形式携带，要放在-jar参数前⾯,`java -Dxxx=test -DprocessType=1 -jar xxx.jar`,取值:System.getProperty("propName")
2. 参数直接跟在命令后⾯,`java -jar xxx jar processType=1 processType2=2`,取值:参数就是jar包⾥主启动类中main⽅法的args参数，按顺序来
3. springboot的⽅式，--key=value⽅式,`java -jar xxx.jar --xxx=test`,取值:spring的@value("$(xxx)”)

区别：

1. -D参数为jvm参数，项⽬启动完后可通过`System.getProperty("nacos.standalone")`进⾏读取,也可以通过这个⽅式`Integer.getInteger("nacos.http.timeout",5000);`获取jvm参数
2. --参数，是通过main的args传⼊进去的，args参数最后会放⼊env环境变量⾥，所以配置bean（@ConfigurationProperties被注解修饰的）的配置值也被覆盖。
3. spring boot修改配置参数时命令行优先级最高,其次环境变量,最后是配置文件，使用命令行时--优先级最高,其次是-D(VM options)

**@Value失效的情况**
1. 使用static或final修饰
2. 类没有注册为bean
3. 构造方法调用该注解修饰的字段也会失效

```java
@ConfigurationProperties(prefix="user")
@PropertySource(value = {"demo/props/demo.properties"})
// @PropertySource(value = {"classpath:user.yml"}, factory = PropertySourceFactory.class)
// @Profile:指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件

@Autowired
Environment environmen
environmen.getProperty("propName")
```
Springboot中默认的静态资源路径有4个，分别是：
```
classpath:/METAINF/resources/，classpath:/resources/，classpath:/static/，classpath:/public/
优先级顺序为:META-INF/resources>resources>static>public
```


## XML配置

### <mvc:annotation-driven />
Spring 3.0.x中使用了<mvc:annotation-driven />后，默认会帮我们注册默认处理请求，参数和返回值的类，其中最主要的两个类：DefaultAnnotationHandlerMapping和AnnotationMethodHandlerAdapter，分别为HandlerMapping的实现类和HandlerAdapter的实现类。从3.1.x版本开始对应实现类改为了RequestMappingHandlerMapping和RequestMappingHandlerAdapter。

### <context:component-scan />
当配置了<mvc:annotation-driven />后，Spring就知道了我们启用注解驱动。然后Spring通过<context:component-scan />标签的配置，会自动为我们将扫描到的@Component,@Controller,@Service,@Repository等注解标记的组件注册到工厂中，来处理我们的请求.
<context:component-scan />标签是告诉Spring来扫描指定包下的类，并注册被@Component，@Controller，@Service，@Repository等注解标记的组件。而<mvc:annotation-driven />是告知Spring，我们启用注解驱动

### <mvc:default-servlet-handler />
当在web.xml中将前端控制器的映射请求设置为"/"时所有的请求包括静态资源的请求都提交到DispatcherServlet进行处理，所以访问静态资源会404，SpringM VC在全局配置文件中提供了一个<mvc:default-servlet-handler/>标签。在WEB容器启动的时候会在上下文中定义一个DefaultServletHttpRequestHandler，它会对DispatcherServlet的请求进行处理，如果该请求已经作了映射，那么会接着交给后台对应的处理程序，如果没有作映射，就交给WEB应用服务器默认的Servlet处理，从而找到对应的静态资源，只有再找不到资源时才会报错。
一般WEB应用服务器默认的Servlet都是default。如果默认Servlet用不同名称自定义配置，或者在缺省Servlet名称未知的情况下使用了不同的Servlet容器，则必须显式提供默认Servlet的名称，如下：
```xml
<mvc:default-servlet-handler default-servlet-name="myCustomDefaultServlet"/>
```
相当于在web.xml里这样配置

```xml
<servlet-mapping>
	<servlet-name>default</servlet-name>
	<url-pattern>*.js</url-pattern>
</servlet-mapping>
```

### <context:annotation-config />
< context:annotation-config>是用于激活那些已经在spring容器里注册过的bean上面的注解，也就是显示的向Spring注册AutowiredAnnotationBeanPostProcessor，CommonAnnotationBeanPostProcessor，PersistenceAnnotationBeanPostProcessor，RequiredAnnotationBeanPostProcessor这四个Processor，注册这4个BeanPostProcessor的作用，就是为了你的系统能够识别相应的注解。BeanPostProcessor就是处理注解的处理器。
一般来说，这些注解我们还是比较常用，尤其是@Autowired的注解，比如我们要使用@Autowired注解，那么就必须事先在Spring容器中声明AutowiredAnnotationBeanPostProcessor Bean。传统声明方式如下
```xml
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
```
在自动注入的时候更是经常使用，所以如果总是需要按照传统的方式一条一条配置显得有些繁琐和没有必要，于是spring给我们提供<context:annotation-config />的简化配置方式，自动帮你完成声明。

### <context:component-scan base-package=”XX.XX”/>
该配置项其实也包含了自动注入上述processor的功能，因此**当使用<context:component-scan />后，就可以将<context:annotation-config />移除了**。
<context:annotation-config />：仅能够在已经在已经注册过的bean上面起作用。对于没有在spring容器中注册的bean，它并不能执行任何操作。
<context:component-scan base-package="XX.XX"/>除了具有上面的功能之外，还具有自动将带有@component,@service,@Repository等注解的对象注册到spring容器中的功能。
如果同时使用这两个配置会不会出现重复注入的情况呢？
<context:annotation-config />和<context:component-scan />同时存在的时候，前者会被忽略。如@Autowire，@Resource等注解只会被注入一次！

### <context:spring-configured />
在没有注入ioc容器的类里面进行依赖注入,例如当一个类没有被Spring注册为bean,却想要在这个类里面使用@Autowired注解需要在XML里配置此标签

### 注册拦截器
```xml
<!-- 自定义拦截器，拦截所有请求，验证是否登录 -->
<mvc:interceptors>
	<mvc:interceptor>
	<mvc:mapping path="/**"/>
	<bean class="com.example.interceptor.CommonInterceptor"></bean>
	</mvc:interceptor>
</mvc:interceptors>
```

### 注册自定义参数解析器
```xml
<!--注册自定义参数解析器-->
 <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <bean class="com.liyao.pre.UserIdArgumentResolver"/>
        </mvc:argument-resolvers>
  </mvc:annotation-driven>
```

### 属性文件读取
```xml
<!-- 属性文件读取-->
<context:property-placeholder location="classpath:jdbc.properties" />
<!--或者-->
<bean id="propertyPlaceholderConfigurer" class="org.springframework,beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <list>
            <value>jdbc.properties<value/>
        </list>
    </property>
</bean>
```
### 有关classpath和classpath\*
Spring可以通过指定classpath\*:或classpath:前缀加路径的方式从classpath下加载文件。**classpath\*:可以从多个jar文件中加载相同的文件。classpath:只能加载找到的第一个文件。**而使用classpath加载一般的优先级为：当前classes > jar包中的classes

### 自定义消息转换器
```xml
<!--自定义消息转换器-->
<mvc:annotation-driven >
	<mvc:message-converters register-defaults="true">
        <!--字符串转换器-->
        <bean class="org.springframework.http.converter.StringHttpMessageConverter" >
            <property name = "supportedMediaTypes">
                <list>
                    <value>application/json;charset=utf-8</value>
                    <value>text/html;charset=utf-8</value>
                </list>
            </property>
        </bean>
        <!--json转换器-->
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter" />
        <!--自己定义的消息转换器-->
        <bean class ="com.dzf.converter.MyMessageConverter"> 
            <property name = "supportedMediaTypes">
                <list>
                    <value>application/json;charset=utf-8</value>
                    <value>application/x-result;charset=utf-8</value>
                    <value>text/html;charset=utf-8</value>
                </list>
            </property>
        </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

### 扫描
```xml
<context:include-filter> <!--只扫描某个目录-->
<context:exclude-filter> <!--不扫描某个目录-->

```
### Spring会自动发现基础的JTA实现
```xml
<tx:jta-transaction-manager /> <!--Spring会自动发现基础的JTA实现-->

<!-- 自动为spring容器中那些配置@aspectJ切面的bean创建代理，织入切面。当然，spring
在内部依旧采用AnnotationAwareAspectJAutoProxyCreator进行自动代理的创建工作，但具体实现的细节已经被<aop:aspectj-autoproxy />隐藏起来了-->
<aop:aspectj-autoproxy />
```

### <aop:aspectj-autoproxy />
```xml
<!-- 有一个proxy-target-class属性，默认为false，表示使用jdk动态代理织入增强，当配为<aop:aspectj-autoproxy  poxy-target-class="true"/>时，表示使用CGLib动态代理技术织入增强。不过即使proxy-target-class设置为false，如果目标类没有声明接口，则spring将自动使用CGLib动态代理。-->
<aop:aspectj-autoproxy />

```

### 扫描@Scheduled
```xml
<task:annotation-driven /> <!--扫描@Scheduled-->
```

### org.springframework.web.filter.HiddenHttpMethodFilter
浏览器form表单只支持GET与POST请求，而DELETE、PUT等method并不支持，spring3.0添加了一个过滤器，可以将这些请求转换为标准的http方法，使得支持GET、POST、PUT与DELETE请求，该过滤器为HiddenHttpMethodFilter,需要注意的是，由于doFilterInternal方法只对method为post的表单进行过滤，所以在页面中必须如下设置：
```xml
<form action="..." method="post">
   <input type="hidden" name="_method" value="put" />
        ......
</form>
  而不是使用：
<form action="..." method="put">
        ......
</form>
```
HiddenHttpMethodFilter必须作用于dispatcher前

### org.springframework.web.context.ContextLoaderListener
ContextLoaderListener的作用就是启动Web容器时，自动装配ApplicationContext.xml的配置信息。
ContextLoaderListener继承自ContextLoader，实现的是ServletContextListener接口。在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法。ContextLoaderListener可以指定在Web应用程序启动时载入Ioc容器，正是通过ContextLoader来实现的，ContextLoader来完成实际的WebApplicationContext，也就是Ioc容器的初始化工作。如果没有显式声明，则系统默认在WEB-INF/applicationContext.xml。

### org.springframework.web.util.IntrospectorCleanupListener
JDK中的java.beans.Introspector类的用途是发现Java类是否符合JavaBean规范,如果有的框架或程序用到了Introspector类,那么就会启用一个系统级别的缓存,此缓存会存放一些曾加载并分析过的JavaBean的引用。当Web服务器关闭时,由于此缓存中存放着这些JavaBean的引用,所以垃圾回收器无法回收Web容器中的JavaBean对象,最后导致内存变大。而org.springframework.web.util.IntrospectorCleanupListener就是专门用来处理Introspector内存泄漏问题的辅助类。IntrospectorCleanupListener会在Web服务器停止时清理Introspector缓存,使那些Javabean能被垃圾回收器正确回收。Spring自身不会出现这种问题，因为Spring在加载并分析完一个类之后会马上刷新JavaBeans Introspector缓存,这就保证Spring中不会出现这种内存泄漏的问题。但有些程序和框架在使用了JavaBeans Introspector之后,没有进行清理工作(如Quartz,Struts),最后导致内存泄漏


## 其他

- [Java面试中常问的Spring问题，你都会吗？](https://zhuanlan.zhihu.com/p/42092555)
- [如果我是面试官，我会问你Spring这些问题](https://mp.weixin.qq.com/s/SqsAO3dBF3d5iQ5TDqmSRQ)
- [推荐收藏：Spring面试63问！](https://mp.weixin.qq.com/s/txmK9ui20aXTewNOG17ZxQ)
- [spring中那些让你爱不释手的代码技巧](https://mp.weixin.qq.com/s/Aet8wzzzxGGfAAJAnBcLng)
- [《轻松读懂spring》之IOC的主干流程（上）](https://mp.weixin.qq.com/s/SZn9WRZjOuGXo2sX6TU0Uw)
- [我们到底为什么要用IoC和AOP](https://mp.weixin.qq.com/s/LjMV4TAng0kldVoubk8T-Q)
- [@Conditional的强大之处](https://mp.weixin.qq.com/s/rONWZ1YcnGc7QDW4-XOKlA)
- [SpringBatch批处理框架，真心强啊！！](https://mp.weixin.qq.com/s/M14kvrWMT_4MRZDZ1TpTYA)
- [手写Spring框架](https://mp.weixin.qq.com/s/YfS9xtaXWnt42xkk-kk4WA)
- [Spring的Bean明明设置了Scope为Prototype，为什么还是只能获取到单例对象？](https://mp.weixin.qq.com/s/_j_0fZKTX6YUhgytWXRKEw)
- [聊聊Spring核心](https://mp.weixin.qq.com/s/xqs0Q8zRKsOp-LdsW-uKeQ)
- [揭秘Spring依赖注入和SpEL表达式](https://mp.weixin.qq.com/s/uBLKOXiwOsaa5za_grlMZw)