---
title: Spring相关
sticky: 100
categories: Spring
index_img: /assert/spring.jpg
img: 
coverImg: https://pic4.zhimg.com/v2-13cd8daa0a4eba009b68785cfbbd5007_r.jpg
cover: true
summary: Build the apps that make the world run
---

### Spring

#### 相关注解

##### @RequestBody

主要用来接收前端传递给后端的json字符串中的数据的(请求体中的数据的)；GET方式无请求体，所以使用@RequestBody接收数据时，前端不能使用GET方式提交数据，而是用POST方式进行提交,在方法里面标记，可以作为一个对象接收,也可以作为字符串接收，关键在于Spring中对json的解析配置
```js
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
[@RequestBody 接收数组、List 参数、@Deprecated 标记废弃方法](https://mp.weixin.qq.com/s/iyubfxmV_8KU8v4cg1A7tg)

##### @PostConstruct和@PreDestroy

@PostConstruct该注解被用来修饰一个非静态的void()方法。被@PostConstruct修饰的方法会当bean创建完成的时候，会后置执行@PostConstruct修饰的方法。PostConstruct在构造函数之后执行，bean的init()方法之前执行。相当于init-method,使用在方法上，当Bean初始化时执行。Constructor(构造方法) -> @Autowired(依赖注入) -> @PostConstruct(注释的方法)
@PreDestroy类似于destory-method 在servlet destory()方法之后执行

##### @Autowired和@Resource区别

1. @Autowired与@Resource都可以用来装配bean. 都可以写在字段上,或写在setter方法上。
2. @Autowired默认按类型装配（这个注解是属业spring的），默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用，如下：
```java
@Autowired () 
@Qualifier ( "baseDao" )
private BaseDao baseDao;
```
3. @Resource（这个注解属于J2EE的），默认按照名称进行装配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名进行安装名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。
4. @Autowired 只按照byType 注入,由Spring提供，@Resource @Resource默认按byName自动注入，也提供按照byType 注入

- [@Autowired 注解是如何实现的？](https://mp.weixin.qq.com/s/gRqZwUV791RtCI1xCoV3Qw)
- [你所不知道的 Spring 中 @Autowired 那些实现细节](https://mp.weixin.qq.com/s/n_syhEFrXykI7ySRtahEmg)
- [@Autowired的这些骚操作，你都知道吗？](https://mp.weixin.qq.com/s/2X5xv8I0b6TcXWVH-SC8Ug)

##### @Inject

1. @Inject是JSR330 (Dependency Injection for Java)中的规范，需要导入javax.inject.Inject;实现注入。
2. @Inject是根据类型进行自动装配的，如果需要按名称进行装配，则需要配合@Named；
3. @Inject可以作用在变量、setter方法、构造函数上。
```java
private Abc abc;
@Inject
public void setAbc(@Named("beanName") Abc abc){
	this.abc = abc;
}
```

[@Autowired, @Resource, @Inject 三个注解的区别](https://mp.weixin.qq.com/s/YLIsRBSiIjz3dCtSA9onDQ)

1. @Autowired是Spring自带的，@Inject和@Resource都是JDK提供的，其中@Inject是JSR330规范实现的，@Resource是JSR250规范实现的，而Spring通过BeanPostProcessor来提供对JDK规范的支持。
2. @Autowired、@Inject用法基本一样，不同之处为@Autowired有一个required属性，表示该注入是否是必须的，即如果为必须的，则如果找不到对应的bean，就无法注入，无法创建当前bean。
3. @Autowired、@Inject是默认按照类型匹配的，@Resource是按照名称匹配的。如在spring-boot-data项目中自动生成的redisTemplate的bean，是需要通过byName来注入的。如果需要注入该默认的，则需要使用@Resource来注入，而不是@Autowired。
4. 对于@Autowire和@Inject，如果同一类型存在多个bean实例，则需要指定注入的beanName。@Autowired和@Qualifier一起使用，@Inject和@Named一起使用。


- [关于Spring注入方式的几道面试题，你能答上么](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&amp;mid=2247494432&amp;idx=1&amp;sn=3acc7e7bf31c6d1f56ad830d6eb1ec41&amp;source=41#wechat_redirect)
- [最全的 Spring 依赖注入方式，你都会了吗？](https://mp.weixin.qq.com/s/u1DcCsRrrHYFOVykwW4Dcg)
- [Spring官方为什么建议构造器注入？](https://mp.weixin.qq.com/s/fVV6dYh0DQOoDiXwLR5miw)
- [Bean放入Spring容器，你知道几种方式？](https://mp.weixin.qq.com/s/g9iRu1slTMx0dwYJiy2m7w)
- [Spring 注入 Bean 的 7 种方式，还有谁不会？？](https://mp.weixin.qq.com/s/i0Y-p7mda5FJCWCMJ8msdg)
- [Spring 注解 @Bean 和 @Component 的区别, 你知道吗？](https://mp.weixin.qq.com/s/6CwABJAePAT6hzTmfk7Jjg)
- [@Bean 与 @Component 用在同一个类上，会怎么样？](https://mp.weixin.qq.com/s/lyH72PRAGcR2-aQvMZ1jPA)


#### Spring中的Bean对象

**Bean的生命周期**
在将一个bean对象配置在ioc容器中之后，这个bean的生命周期就会交由ioc容器进行管理。一般担当管理者的角色是BeanFactory和ApplicationContext。
1. bean的创建
在解析ioc容器时，根据解析容器的工厂，决定bean的初始化时间 
BeanFactory - getBean()方法调用时 初始化bean
ApplicationContext - 解析ioc容器时 初始化bean
2. 注入
根据bean子元素的配置实现bean之间的被动注入
3. BeanNameAware
如果bean实现了该接口，执行其setBeanName(String name)方法.参数name是bean在容器中的名称,即xml里面bean的id名称
4. BeanFactoryAware     
如果实现了该接口，执行其setBeanFactory(BeanFactory factory)方法，参数是创建Bean的BeanFactory本身
5. ApplicationContextAware 
如果这个Bean已经实现了该接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文（同样这个方式也可以实现步骤4的内容，但比4更好，因为ApplicationContext是BeanFactory的子接口，有更多的实现方法）
```java
import org.springframework.beans.context.ApplicationContextAware
// 当需要从spring容器中获取bean时一般使用这种方式获取
ApplicationContext appContext = new ClassPathXmlApplicationContext("applicationContext-common.xml");  
AbcService abcService = (AbcService)appContext.getBean("abcService"); 
// 但是这样就会存在一个问题：因为它会重新装载applicationContext-common.xml并实例化上下文bean，如果有些线程配置类也是在这个配置文件中，那么会造成做相同工作的的线程会被启两次。一次是web容器初始化时启动，另一次是上述代码显示的实例化了一次。当于重新初始化一遍！这样就产生了冗余,所以可以通过实现ApplicationContextAware接口获取bean,当一个类实现了这个接口（ApplicationContextAware）之后，这个类就可以方便获得ApplicationContext中的所有bean。换句话说，就是这个类可以直接获取spring配置文件中，所有有引用到的bean对象

private static ApplicationContext applicationContext;
 	@Override
    public void setApplicationContext(ApplicationContext arg0) throws BeansException {
        applicationContext = arg0;
    }
}
```
注意：从ApplicationContextAware获取ApplicationContext上下文的情况，仅仅适用于当前运行的代码和已启动的Spring代码处于同一个Spring上下文，否则获取到的ApplicationContext是空的
6. BeanPostProcessor (前置方法)
ioc容器中如果有bean实现了该接口，那所有的bean在初始化之前都会执行其实例的postProcessBeforeInitialization(Object bean, String beanName)前置方法，BeanPostProcessor经常被用作是Bean内容的更改,该方法最后返回bean
7. @PostConstruct修饰的非静态方法
8. InitializingBean
如果实现了该接口，则允许一个bean在它的所有必须属性被BeanFactory设置后，来执行初始化的工作，会自动调用afterPropertiesSet()方法对Bean进行初始化，实现此接口的话正常情况下配置文件就不用指定 init-method属性了。
9. 如果Bean在Spring中配置了init-method属性，调用init-method属性指向的方法 此时完成bean的初始化
10. BeanPostProcessor(后置方法)
ioc容器中如果有bean实现了接口，那所有的bean在初始化之后都会执行其实例的postProcessAfterInitialization(Object bean, String beanName)后置方法
11. 实现SmartInitializingSingleton的接口后，当所有单例 bean 都初始化完成以后， Spring的IOC容器会回调该接口的afterSingletonsInstantiated()方法,主要应用场合就是在所有单例 bean 创建完成之后，可以在该回调中做一些事情。执行时机在ApplicationContextAware执行之后
12. @PreDestroy修饰的方法
13. ioc容器关闭时，如果bean实现了DisposableBean接口，则执行其destory()方法，在Bean生命周期结束前调用destory()方法做一些收尾工作,重写destroy()方法
14. 如果这个Bean在Spring配置了destroy-method属性，执行destory-method属性指向的方法

![](/images/bean.png)

类构造方法 - postProcessBeforeInitialization前置方法 - @PostConstruct注解的方法 - InitializingBean的afterPropertiesSet()方法- XML中定义的bean init-method方法 - postProcessAfterInitialization后置方法

- [11张流程图帮你搞定 Spring Bean 生命周期](https://mp.weixin.qq.com/s/I8tsf7cFXkHX1pUp7SPByw)
- [面试官：兄弟你来阐述一下Spring框架中Bean的生命周期](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&amp;mid=2247490923&amp;idx=1&amp;sn=2b3488b33d806c4d59564244aa65eb0b&amp;source=41#wechat_redirect)
- [面试官：说说 Spring Bean 的实例化过程？面试必问的！](https://mp.weixin.qq.com/s/5hAt9_KyyqHy7zzOjZ9LyQ)
- [你能说说Spring框架中Bean的生命周期吗？](https://mp.weixin.qq.com/s/Hl8YtTlyX1KADWDZkNkMaw)
- [一文搞定 Spring Bean 的创建全过程！](https://mp.weixin.qq.com/s/pEKQk-ckeQbcRGPs2mCpQw)
- [Spring Boot 启动扩展点超详细总结，再也不怕面试官问了](https://mp.weixin.qq.com/s/l0O3C_UiO3CdfNE2V73qmA)
- [Spring系列之beanFactory与ApplicationContext](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&amp;mid=2247493943&amp;idx=1&amp;sn=9eaa46ed730874fce003c66f76fe9c7f&amp;source=41#wechat_redirect)

**BeanFactoryPostProcessor、BeanPostProcessor区别**
BeanFactoryPostProcessor：针对bean工厂，BeanFactory后置处理器，是对BeanDefinition对象进行修改。（BeanDefinition：存储bean标签的信息，用来生成bean实例）,BeanFactoryPostProcessor接口是针对bean容器的，它的实现类可以在当前BeanFactory初始化（spring容器加载bean定义文件）后，bean实例化之前修改bean的定义属性，达到影响之后实例化bean的效果。也就是说，Spring允许BeanFactoryPostProcessor在容器实例化任何其它bean之前读取配置元数据，并可以根据需要进行修改，例如可以把bean的scope从singleton改为prototype，也可以把property的值给修改掉。可以同时配置多个BeanFactoryPostProcessor，并通过设置’order’属性来控制各个BeanFactoryPostProcessor的执行次序.
BeanPostProcessor：针对bean,Bean后置处理器，是对生成的Bean对象进行修改。BeanPostProcessor能在spring容器实例化bean之后，在执行bean的初始化方法前后，添加一些自己的处理逻辑。初始化方法包括以下两种：
1. 实现InitializingBean接口的bean，对应方法为afterPropertiesSet
2. xml定义中，通过init-method设置的方法,BeanPostProcessor是BeanFactoryPostProcessor之后执行的。

**Bean的调用**
```java
// 1、使用BeanWrapper
HelloWorld hw = new HelloWorld();
BeanWrapper bw = new BeanWrapperImpl(hw);
bw.setPropertyvalue("msg","HelloWorld");
System.out.println(bw.getPropertyCalue("msg"));
// 2、使用BeanFactory
InputStream is = new FileInputStream("config.xml");
XmlBeanFactory factory = new XmlBeanFactory(is);
HelloWorld hw = (HelloWorld) factory.getBean("HelloWorld");
System.out.println(hw.getMsg());
// 3、使用ApplicationContext
ApplicationContext actx = new FleSystemXmlApplicationContext("config.xml");
HelloWorld hw = (HelloWorld) actx.getBean("HelloWorld");
System.out.println(hw.getMsg());
```

#### Spring循环依赖

1. 如果是原型 bean的循环依赖，Spring无法解决
2. 如果是构造参数注入的循环依赖，Spring无法解决
容器为了缓存这些单例的Bean需要一个数据结构来存储，比如Map {k:name; v:bean}。
而我们创建一个Bean就可以往Map中存入一个Bean。这时候我们仅需要一个Map就可以满足创建+缓存的需求.
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

**一级缓存** 
1. 实例化A对象。
2. 填充A的属性阶段时需要去填充B对象，而此时B对象还没有创建，所以这里为了完成A的填充就必须要先去创建B对象； 
3. 实例化B对象。 
4. 执行到B对象的填充属性阶段，又会需要去获取A对象，而此时Map中没有A，因为A还没有创建完成，导致又需要去创建A对象。 
这样，就会循环往复，一直创建下去，只到堆栈溢出。 

为什么不能在实例化A之后就放入Map？
因为此时A尚未创建完整，所有属性都是默认值，并不是一个完整的对象，在执行业务时可能会抛出未知的异常。所以必须要在A创建完成之后才能放入Map。

**二级缓存**
此时我们引入二级缓存用另外一个Map2 {k:name; v:earlybean} 来存储尚未已经开始创建但是尚未完整创建的对象。
1. 实例化A对象之后，将A对象放入Map2中。
2. 在填充A的属性阶段需要去填充B对象，而此时B对象还没有创建，所以这里为了完成A的填充就必须要先去创建B对象。 
3. 创建B对象的过程中，实例化B对象之后，将B对象放入Map2中。 
4. 执行到B对象填充属性阶段，又会需要去获取A对象，而此时Map中没有A，因为A还没有创建完成，但是我们继续从Map2中拿到尚未创建完毕的A的引用赋值给a字段。这样B对象其实就已经创建完整了，尽管B.a对象是一个还未创建完成的对象。 
5. 此时将B放入Map并且从Map2中删除。 
6. 这时候B创建完成，A继续执行b的属性填充可以拿到B对象，这样A也完成了创建。 B.a也完整了
7. 此时将A对象放入Map并从Map2中删除。

**二级缓存已然解决了循环依赖问题，为什么还需要三级缓存？**
从上面的流程中我们可以看到使用两级缓存可以完美解决循环依赖的问题，但是Spring中还有另外一个问题需要解决，这就是初始化过程中的AOP实现。 AOP是Spring的重要功能，实现方式就是使用代理模式动态增强类的功能。 动态单例目前有两种技术可以实现，一种是JDK自带的基于接口的动态Proxy技术，一种是CGlib基于字节码动态生成的Proxy技术，这两种技术都是需要原始对象创建完毕，之后基于原始对象生成代理对象的。 那么我们发现，在二级缓存的设计下，我们需要在放入缓存Map之前将代理对象生成好。 
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
5. BeanPostProcessor doAfter -- AOP是在这个阶段实现的 
所以要实现上面的方案，势必需要将BeanPostProcessor阶段提前或者侵入到填充属性的流程中，那么从程序设计上来说，这样做肯定是不美的

面试官会问：为什么要使用三级缓存呢？二级缓存能解决循环依赖吗？
答：如果要使用二级缓存解决循环依赖，意味着所有Bean在实例化后就要完成AOP代理，这样违背了Spring设计的原则，Spring在设计之初就是通过AnnotationAwareAspectJAutoProxyCreator这个后置处理器来在Bean生命周期的最后一步来完成AOP代理，而不是在实例化后就立马进行AOP代理

**三级缓存**
Spring引入了第三级缓存来解决这个问题， Map3 {k:name v:ObjectFactory} ，这个缓存的value就不是Bean对象了，而是一个接口对象由一段lamda表达式实现。在这段lamda表达式中去完成一些BeanPostProcessor的执行。
1. 实例化A对象之后，将A的ObjectFactory对象放入Map3中。 
2. 在填充A的属性阶段需要去填充B对象，而此时B对象还没有创建，所以这里为了完成A的填充就必须要先去创建B对象。 
3. 创建B对象的过程中，实例化B的ObjectFactory对象之后，将B对象放入Map2中。 
4. 执行到B对象填充属性阶段，又会需要去获取A对象，而此时Map1中没有A，因为A还没有创建完成，但是我们继续从Map2中也拿不到，到Map3中获取了A的ObjectFactory对象，通过ObjectFactory对象获取A的早期对象，并将这个早期对象放入Map2中，同时删除Map3中的A，将尚未创建完毕的A的引用赋值给a字段。这样B对象其实就已经创建完整了，尽管B.a对象是一个还未创建完成的对象。 
5. 此时将B放入Map并且从Map3中删除。 
6. 这时候B创建完成，A继续执行b的属性填充可以拿到B对象，这样A也完成了创建。 
7. 此时将A对象放入Map并从Map2中删除。


- [Spring 三级缓存解决循环依赖](https://mp.weixin.qq.com/s/ns9JEpvMt7U-nsMZzEUIUQ)
- [终于有人把 Spring 循环依赖讲清楚了！](https://mp.weixin.qq.com/s/L1PJ-cikoS8sOORszEYnfw)
- [烂大街的Spring循环依赖该如何回答？](https://mp.weixin.qq.com/s/5VHU2qRQMPL0IOZuEOPmQA)
- [spring：我是如何解决循环依赖的？](https://mp.weixin.qq.com/s/7S9wVOVJyoHiC_RnhZrJTw)
- [Spring 为何需要三级缓存解决循环依赖，而不是二级缓存？](https://mp.weixin.qq.com/s/BaRlMNo0HlPP9Vz4x9bUaA)
- [图解 Spring 循环依赖，写得太好了！](https://mp.weixin.qq.com/s/JuS6aewMXSp22zjwWCKt6g)
- [Spring 是如何解决循环依赖的](https://mp.weixin.qq.com/s/e7e-Pct5CcMrHBCsdjsLSQ)
- [Spring面试题之循环依赖的理解](https://mp.weixin.qq.com/s/amgsB3MMvcA2pw9N2wECDw)
- [Spring的循环依赖，到底是什么样的](https://mp.weixin.qq.com/s/qWWjWpIbghj5v6-KCd8xxA)

#### 相关文章
- [Spring中涉及的设计模式总结](https://mp.weixin.qq.com/s/ktNs4T_OZ-neWWvtBmC-cA)
- [Spring中经典的9种设计模式](https://mp.weixin.qq.com/s/gz2-izPrgW1AGbqqovT0cA)
- [Java面试中常问的Spring问题，你都会吗？](https://zhuanlan.zhihu.com/p/42092555)
- [如果我是面试官，我会问你 Spring 这些问题](https://mp.weixin.qq.com/s/SqsAO3dBF3d5iQ5TDqmSRQ)
- [推荐收藏：Spring 面试 63 问！](https://mp.weixin.qq.com/s/txmK9ui20aXTewNOG17ZxQ)
- [spring中那些让你爱不释手的代码技巧](https://mp.weixin.qq.com/s/Aet8wzzzxGGfAAJAnBcLng)
- [《轻松读懂spring》之 IOC的主干流程（上）](https://mp.weixin.qq.com/s/SZn9WRZjOuGXo2sX6TU0Uw)
- [我们到底为什么要用 IoC 和 AOP](https://mp.weixin.qq.com/s/LjMV4TAng0kldVoubk8T-Q)
- [@Conditional的强大之处](https://mp.weixin.qq.com/s/rONWZ1YcnGc7QDW4-XOKlA)
- [Spring Batch 批处理框架，真心强啊！！](https://mp.weixin.qq.com/s/M14kvrWMT_4MRZDZ1TpTYA)
- [手写Spring框架](https://mp.weixin.qq.com/s/YfS9xtaXWnt42xkk-kk4WA)
- [Spring 的 Bean 明明设置了 Scope 为 Prototype，为什么还是只能获取到单例对象？](https://mp.weixin.qq.com/s/_j_0fZKTX6YUhgytWXRKEw)

### Spring MVC

**Handle：** Handler是一个Controller的对象和请求方式的组合的一个Object对象
**HandleExcutionChains**是HandleMapping返回的一个处理执行链，它是对Handle的二次封装，将拦截器关联到一起。然后，在DispatcherServlert中完成了拦截器链对handler的过滤。**DispatcherServlet**要将一个请求交给哪个特定的Controller，它需要咨询一个Bean——这个Bean为“HandlerMapping”。HandlerMapping是把一个URL指定到一个Controller上，（就像应用系统的web.xml文件使用<servlet-mapping\>将URL映射到servlet）。
**DispatcherServlet**：作为前端控制器，整个流程控制的中心，控制其它组件执行，统一调度，降低组件之间的耦合性，提高每个组件的扩展性。
作用：接收请求，响应结果，相当于转发器，中央处理器。有了dispatcherServlet减少了其它组件之间的耦合度。用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请,dispatcherServlet的存在降低了组件之间的耦合性

**HandlerMapping：** 通过扩展处理器映射器实现不同的映射方式，作用:根据请求的url查找Handler,HandlerMapping负责根据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等

**HandlAdapter：**通过扩展处理器适配器，支持更多类型的处理器。
作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler，通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

**ViewResolver：**通过扩展视图解析器，支持更多类型的视图解析，例如：jsp、freemarker、pdf、excel等。作用：进行视图解析，根据逻辑视图名解析成真正的视图（view）
作用：View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。 springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

**DispatcherServlet 的工作流程**
![](/images/DispatcherServlet的工作流程.png)

1. 向服务器发送 HTTP 请求，请求被前端控制器 DispatcherServlet 捕获。
2. DispatcherServlet 根据 -servlet.xml 中的配置对请求的 URL 进行解析，得到请求资源标识符（URI）。然后根据该 URI，调用 HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 Handler 对象以及 Handler 对象对应的拦截器），最后以HandlerExecutionChain 对象的形式返回。
3. DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的 preHandler(…)方法）。
4. 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。在填充Handler的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作：HttpMessageConveter：将请求消息（如 Json、xml 等数据）转换成一个对象，将对象转换为指定的响应信息。
数据转换：对请求消息进行数据转换。如String转换成Integer、Double等。数据根式化：对请求消息进行数据格式化。如将字符串转换成格式化数字或格式化日期等。数据验证：验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中。
5. Handler(Controller)执行完成后，向 DispatcherServlet 返回一个 ModelAndView 对象；
6. 根据返回的ModelAndView，选择一个适合的 ViewResolver（必须是已经注册到 Spring 容器中的ViewResolver)返回给DispatcherServlet。
7. ViewResolver 结合Model和View，来渲染视图。
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
2. DispatcherServlet——>HandlerMapping， HandlerMapping 将会把请求映射为HandlerExecutionChain 对象（包含一个Handler 处理器（页面控制器）对象、多个HandlerInterceptor 拦截器）对象，通过这种策略模式，很容易添加新的映射策略；
3. DispatcherServlet——>HandlerAdapter，HandlerAdapter 将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；
4. HandlerAdapter——>处理器功能处理方法的调用，HandlerAdapter 将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个ModelAndView 对象（包含模型数据、逻辑视图名）；
5. ModelAndView的逻辑视图名——> ViewResolver， ViewResolver 将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；
6. View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，因此很容易支持其他视图技术；
7. 返回控制权给DispatcherServlet，由DispatcherServlet返回响应给用户，到此一个流程结束。
下边两个组件通常情况下需要开发：
Handler：处理器，即后端控制器用controller表示。
View：视图，即展示给用户的界面，视图中通常需要标签语言展示模型数据。

- [SpringMVC执行过程解析](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&amp;mid=2247494012&amp;idx=2&amp;sn=1c5052c5b4a5547a21449412f619327b&amp;source=41#wechat_redirect)
- [Spring MVC请求处理过程不是两张流程图就能讲清楚的](https://mp.weixin.qq.com/s/klCAv0TM2tvgy4xqoC1hQw)
- [SpringMVC 初始化流程分析](https://mp.weixin.qq.com/s/IeMOfnXhOX5RCf4i5Xsdzw)
- [SpringMVC 源码分析之 DispatcherServlet](https://mp.weixin.qq.com/s/F0QZ-Ukgtn3oC6a4loM9Vg)
- [SpringMVC 九大组件之 HandlerMapping 深入分析](https://mp.weixin.qq.com/s/0x7_OXPDFX5BqF0jGxN2Vg)
- [SpringMVC 九大组件之 HandlerAdapter 深入分析](https://mp.weixin.qq.com/s/NCnawbIaLUQJNZiDi2yeCw)
- [SpringMVC 九大组件之 ViewResolver 深入分析](https://mp.weixin.qq.com/s/rn-6QyuYIsM_P5b4B1OIrg)
- [编写Spring MVC控制器的14个技巧！涨知识了！](https://mp.weixin.qq.com/s/685jUKqg6I6r0RbmirsRxg)
- [SpringMVC 异常处理体系深入分析](https://mp.weixin.qq.com/s/ZKBQSCMPV7T9yNcMQ5_pQQ)



### [Spring Boot](https://github.com/xmxe/springboot)

**配置文件默认的查找路径如下：**
1. file:./config/
2. file:./ 
3. classpath:/config/
4. classpath:/
配置⽂件名可以通过 spring.config.name修改，最简单的⽅法是放置⼀个配置⽂件到jar包同层⽬录下，或是同层⽬录下的config⼦⽬录下，启动jar包即可加载配置⽂件实现配置项的覆盖

spring boot指定外部的配置⽂件,可以通过修改启动参数的值来指定加载⽬录或是加载⽂件:spring.config.location

```shell
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties.
```
这样不会去默认位置加载配置⽂件，⽽是加载类路径下default.properties和override.properties的⽂件，override.properties中的同名配置会覆盖default.properties,如果指定的路径是以/结尾则是⽬录配置，会去⽬录下找配置⽂件。

**特定配置**
在开发、测试、发布过程中，这三个场景⽐较固定，通常会定义三份不同的配置application-{profile}.yml，在使⽤时通过profile参数来切换。applicaiton-dev.yml，applicaiton-test.yml，applicaiton-prd.yml启动时，通过指定spring.profiles.active参数来切换配置⽂件

springboot项⽬启动的时候可以直接使⽤java jar xxxjar这样。下⾯说说参数的⼀些讲究
1. -DpropName=propValue的形式携带，要放在-jar参数前⾯
java -Dxxx=test -DprocessType=1 -jar xxx.jar
取值:System.getProperty("propName")
2. 参数直接跟在命令后⾯
java -jar xxx jar processType=1 processType2=2
取值:参数就是jar包⾥主启动类中main⽅法的args参数，按顺序来
3. springboot的⽅式，--key=value⽅式
java -jar xxx.jar --xxx=test
取值:spring的@value("$(xxx)”)
区别：
⼀、-D 参数为jvm参数， 项⽬启动完后可通过System.getProperty("nacos.standalone")进⾏读取
也可以通过这个⽅式Integer.getInteger("nacos.http.timeout", 5000);获取jvm参数
⼆、- -参数，是通过main的args传⼊进去的
args参数最后会放⼊env环境变量⾥，所以配置bean（@ConfigurationProperties被注解修饰的）的配置值也被覆盖。
spring boot修改配置参数时命令行优先级最高 其次环境变量 最后是配置文件
使用命令行时- -优先级最高 其次是-D(VM options) 

**@Value失效的情况**
1. 使用static或final修饰
2. 类没有注册为bean
3. 构造方法调用该注解修饰的字段也会失效

```java
@ConfigurationProperties(prefix="user")
@PropertySource(value = {"demo/props/demo.properties"})
//@PropertySource(value = {"classpath:user.yml"}, factory = PropertySourceFactory.class)
// @Profile:指定组件在哪个环境的情况下才能被注册到容器中，不指定，任何环境下都能注册这个组件

@Autowired
Environment environmen
environmen.getProperty("propName")
```
Springboot中默认的静态资源路径有4个，分别是：
```
classpath:/METAINF/resources/，classpath:/resources/，classpath:/static/，classpath:/public/
优先级顺序为：META-INF/resources>resources>static>public

```


### XML配置

**<mvc:annotation-driven />**
Spring 3.0.x中使用了<mvc:annotation-driven />后，默认会帮我们注册默认处理请求，参数和返回值的类，其中最主要的两个类：DefaultAnnotationHandlerMapping 和AnnotationMethodHandlerAdapter ，分别为HandlerMapping的实现类和HandlerAdapter的实现类。从3.1.x版本开始对应实现类改为了RequestMappingHandlerMapping和RequestMappingHandlerAdapter。
当配置了<mvc:annotation-driven />后，Spring就知道了我们启用注解驱动。然后Spring通过**<context:component-scan />**标签的配置，会自动为我们将扫描到的@Component,@Controller,@Service,@Repository等注解标记的组件注册到工厂中，来处理我们的请求.
<context:component-scan />标签是告诉Spring 来扫描指定包下的类，并注册被@Component，@Controller，@Service，@Repository等注解标记的组件。而<mvc:annotation-driven />是告知Spring，我们启用注解驱动

**<mvc:default-servlet-handler />**
当在web.xml中将前端控制器的映射请求设置为"/"时所有的请求包括静态资源的请求都提交到DispatcherServlet进行处理，所以访问静态资源会404，Spring MVC 在全局配置文件中提供了一个<mvc:default-servlet-handler />标签。在 WEB 容器启动的时候会在上下文中定义一个DefaultServletHttpRequestHandler，它会对DispatcherServlet的请求进行处理，如果该请求已经作了映射，那么会接着交给后台对应的处理程序，如果没有作映射，就交给 WEB 应用服务器默认的 Servlet 处理，从而找到对应的静态资源，只有再找不到资源时才会报错。
一般 WEB 应用服务器默认的 Servlet 都是 default。如果默认 Servlet 用不同名称自定义配置，或者在缺省 Servlet 名称未知的情况下使用了不同的 Servlet 容器，则必须显式提供默认 Servlet 的名称，如下：
<mvc:default-servlet-handler default-servlet-name="myCustomDefaultServlet"/>
相当于在web.xml里这样配置

```xml
<servlet-mapping>
	<servlet-name>default</servlet-name>
	<url-pattern>*.js</url-pattern>
</servlet-mapping>
```

**<context:annotation-config />**
< context:annotation-config> 是用于激活那些已经在spring容器里注册过的bean上面的注解，也就是显示的向Spring注册AutowiredAnnotationBeanPostProcessor，CommonAnnotationBeanPostProcessor，PersistenceAnnotationBeanPostProcessor，RequiredAnnotationBeanPostProcessor这四个Processor，注册这4个BeanPostProcessor的作用，就是为了你的系统能够识别相应的注解。BeanPostProcessor就是处理注解的处理器。
一般来说，这些注解我们还是比较常用，尤其是@Autowired的注解，比如我们要使用@Autowired注解，那么就必须事先在 Spring 容器中声明 AutowiredAnnotationBeanPostProcessor Bean。传统声明方式如下
```xml
<bean class="org.springframework.beans.factory.annotation. AutowiredAnnotationBeanPostProcessor "/>
```
在自动注入的时候更是经常使用，所以如果总是需要按照传统的方式一条一条配置显得有些繁琐和没有必要，于是spring给我们提供<context:annotation-config />的简化配置方式，自动帮你完成声明。

**<context:component-scan base-package=”XX.XX”/>**
该配置项其实也包含了自动注入上述processor的功能，因此**当使用 <context:component-scan /> 后，就可以将 <context:annotation-config /> 移除了。**
<context:annotation-config />：仅能够在已经在已经注册过的bean上面起作用。对于没有在spring容器中注册的bean，它并不能执行任何操作。 
<context:component-scan base-package="XX.XX"/>除了具有上面的功能之外，还具有自动将带有@component,@service,@Repository等注解的对象注册到spring容器中的功能。 
如果同时使用这两个配置会不会出现重复注入的情况呢？
<context:annotation-config />和 <context:component-scan />同时存在的时候，前者会被忽略。如@Autowire，@Resource等注解只会被注入一次！

**<context:spring-configured />**
在没有注入ioc容器的类里面进行依赖注入 例如当一个类没有被Spring注册为bean,却想要在这个类里面使用@Autowired注解 需要在XML里配置此标签

**注册拦截器**
```xml
<!-- 自定义拦截器，拦截所有请求，验证是否登录 -->
<mvc:interceptors>
	<mvc:interceptor>
	 <mvc:mapping path="/**"/> 
	<bean class="com.example.interceptor.CommonInterceptor"></bean>
	</mvc:interceptor>
</mvc:interceptors>
```

**注册自定义参数解析器**
```xml
<!--注册自定义参数解析器-->
 <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <bean class="com.liyao.pre.UserIdArgumentResolver"/>
        </mvc:argument-resolvers>
  </mvc:annotation-driven>

```

**属性文件读取**
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
**有关classpath和classpath\***
Spring可以通过指定classpath\*:或classpath:前缀加路径的方式从classpath下加载文件。
classpath\*:可以从多个jar文件中加载相同的文件。
classpath:只能加载找到的第一个文件。
而使用classpath加载一般的优先级为：当前classes > jar包中的classes

**自定义消息转换器**
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

**扫描**
```xml
<context:include-filter> <!--只扫描某个目录-->
<context:exclude-filter> <!--不扫描某个目录-->

```
**Spring会自动发现基础的JTA实现**
```xml
<tx:jta-transaction-manager /> <!--Spring会自动发现基础的JTA实现-->

<!-- 自动为spring容器中那些配置@aspectJ切面的bean创建代理，织入切面。当然，spring
在内部依旧采用AnnotationAwareAspectJAutoProxyCreator进行自动代理的创建工作，但具体实现的细节已经被<aop:aspectj-autoproxy />隐藏起来了-->
<aop:aspectj-autoproxy />
```

**<aop:aspectj-autoproxy />**
```xml
<!-- 有一个proxy-target-class属性，默认为false，表示使用jdk动态代理织入增强，当配为<aop:aspectj-autoproxy  poxy-target-class="true"/>时，表示使用CGLib动态代理技术织入增强。不过即使proxy-target-class设置为false，如果目标类没有声明接口，则spring将自动使用CGLib动态代理。-->
<aop:aspectj-autoproxy />

```

**扫描@Scheduled**
```xml
<task:annotation-driven /> <!--扫描@Scheduled-->
```

**org.springframework.web.filter.HiddenHttpMethodFilter**
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

**org.springframework.web.context.ContextLoaderListener**
ContextLoaderListener的作用就是启动Web容器时，自动装配ApplicationContext.xml的配置信息。
ContextLoaderListener继承自ContextLoader，实现的是ServletContextListener接口。在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法。ContextLoaderListener可以指定在Web应用程序启动时载入Ioc容器，正是通过ContextLoader来实现的，ContextLoader来完成实际的WebApplicationContext，也就是Ioc容器的初始化工作。如果没有显式声明，则 系统默认 在WEB-INF/applicationContext.xml。

**org.springframework.web.util.IntrospectorCleanupListener**
JDK中的java.beans.Introspector类的用途是发现Java类是否符合JavaBean规范,如果有的框架或程序用到了Introspector类,那么就会启用一个系统级别的缓存,此缓存会存放一些曾加载并分析过的JavaBean的引用。当Web服务器关闭时,由于此缓存中存放着这些JavaBean的引用,所以垃圾回收器无法回收Web容器中的JavaBean对象,最后导致内存变大。而org.springframework.web.util.IntrospectorCleanupListener就是专门用来处理Introspector内存泄漏问题的辅助类。IntrospectorCleanupListener会在Web服务器停止时清理Introspector缓存,使那些Javabean能被垃圾回收器正确回收。Spring自身不会出现这种问题，因为Spring在加载并分析完一个类之后会马上刷新JavaBeans Introspector缓存,这就保证Spring中不会出现这种内存泄漏的问题。但有些程序和框架在使用了JavaBeans Introspector之后,没有进行清理工作(如Quartz,Struts),最后导致内存泄漏