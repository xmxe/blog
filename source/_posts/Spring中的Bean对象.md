
---
title: Spring中的Bean对象
categories: Spring
index_img: /assert/spring.jpg
img: https://picx.zhimg.com/v2-5668c094310c3d27b25ea3df98b9e43c_1440w.jpg
coverImg: https://pic1.zhimg.com/v2-fad63176fa79f25cd1b486baf508abc0_r.jpg
cover: true
summary: Spring Bean、生命周期、扩展接口
top: true

---

## Spring Bean

### 谈谈自己对于Spring IoC的了解


**IoC(Inversion of Control,控制反转)** 是Spring中一个非常非常重要的概念，它不是什么技术，而是一种解耦的设计思想。IoC的主要目的是借助于“第三方”(Spring中的IoC容器)实现具有依赖关系的对象之间的解耦(IOC容器管理对象，你只管使用即可)，从而降低代码之间的耦合度。**IoC是一个原则，而不是一个模式，以下模式（但不限于）实现了IoC原则**。

![ioc-patterns](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/ioc-patterns.png)

**Spring IoC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的**。IoC容器负责创建对象，将对象连接在一起，配置这些对象，并从创建中处理这些对象的整个生命周期，直到它们被完全销毁。在实际项目中一个Service类如果有几百甚至上千个类作为它的底层，我们需要实例化这个Service，你可能要每次都要搞清这个Service所有底层类的构造函数，这可能会把人逼疯。如果利用IOC的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

> 关于Spring IOC的理解，推荐看一下这个[知乎回答](https://www.zhihu.com/question/23277575/answer/169698662)，非常不错。

**控制反转怎么理解呢**？举个例子："对象a依赖了对象b，当对象a需要使用对象b的时候必须自己去创建。但是当系统引入了IOC容器后，对象a和对象b之前就失去了直接的联系。这个时候，当对象a需要使用对象b的时候，我们可以指定IOC容器去创建一个对象b注入到对象a中"。对象a获得依赖对象b的过程,由主动行为变为了被动行为，控制权反转，这就是控制反转名字的由来。**DI(Dependecy Inject,依赖注入)是实现控制反转的一种设计模式，依赖注入就是将实例变量传入到一个对象中去**。

**IoC（InversionofControl:控制反转）**，是一种设计思想，而不是一个具体的技术实现。IoC的思想就是将原本在程序中手动创建对象的控制权，交由Spring框架来管理。不过，IoC并非Spring特有，在其他语言中也有应用。**为什么叫控制反转**?

- **控制**：指的是对象创建（实例化、管理）的权力
- **反转**：控制权交给外部环境（Spring框架、IoC容器）

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/java-guide-blog/frc-365faceb5697f04f31399937c059c162.png)

将对象之间的相互依赖关系交给IoC容器来管理，并由IoC容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。IoC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。在实际项目中一个Service类可能依赖了很多其他的类，假如我们需要实例化这个Service，你可能要每次都要搞清这个Service所有底层类的构造函数，这可能会把人逼疯。如果利用IoC的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

在Spring中，IoC容器是Spring用来实现IoC的载体，IoC容器实际上就是个Map（key，value），Map中存放的是各种对象。Spring时代我们一般通过XML文件来配置Bean，后来开发人员觉得XML文件来配置不太好，于是SpringBoot注解配置就慢慢开始流行起来。

> 相关阅读：
>
> - [IoC源码阅读](https://javadoop.com/post/spring-ioc)
> -  [面试被问了几百遍的IoC和AOP，还在傻傻搞不清楚？](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486938&idx=1&sn=c99ef0233f39a5ffc1b98c81e02dfcd4&chksm=cea24211f9d5cb07fa901183ba4d96187820713a72387788408040822ffb2ed575d28e953ce7&token=1736772241&lang=zh_CN#rd)

### 什么是Spring Bean？

简单来说，Bean代指的就是那些被IoC容器所管理的对象。我们需要告诉IoC容器帮助我们管理哪些对象，这个是通过配置元数据来定义的。配置元数据可以是XML文件、注解或者Java配置类。

```xml
<!-- Constructor-arg with 'value' attribute -->
<bean id="..." class="...">
   <constructor-arg value="..."/>
</bean>
```

下图简单地展示了IoC容器如何使用配置元数据来管理对象。

![img](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/system-design/framework/spring/062b422bd7ac4d53afd28fb74b2bc94d.png)

**org.springframework.beans**和**org.springframework.context**这两个包是IoC实现的基础，如果想要研究IoC相关的源码的话，可以去看看

### 将一个类声明为Bean的注解有哪些?

- @Component：通用的注解，可标注任意类为Spring组件。如果一个Bean不知道属于哪个层，可以使用@Component注解标注。
- @Repository:对应持久层即Dao层，主要用于数据库相关操作。
- @Service:对应服务层，主要涉及一些复杂的逻辑，需要用到Dao层。
- @Controller:对应SpringMVC控制层，主要用于接受用户请求并调用Service层返回数据给前端页面。

### @Component和@Bean的区别是什么？

- @Component注解作用于类，而@Bean注解作用于方法。
- @Component通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用@ComponentScan注解定义要扫描的路径从中找出标识了需要装配的类自动装配到Spring的bean容器中）。@Bean注解通常是我们在标有该注解的方法中定义产生这个bean,@Bean告诉了Spring这是某个类的实例，当我需要用它的时候还给我。
- @Bean注解比@Component注解的自定义性更强，而且很多地方我们只能通过@Bean注解来注册bean。比如当我们引用第三方库中的类需要装配到Spring容器时，则只能通过@Bean来实现。

@Bean注解使用示例：

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```
上面的代码相当于下面的xml配置

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

下面这个例子是通过@Component无法实现的。

```java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```
> [@Bean与@Component用在同一个类上，会怎么样？](https://mp.weixin.qq.com/s/Hi0Tdr9DW4bJrOzzDINgCA)

### @Configuration和@Component的区别
一句话概括就是@Configuration中所有带@Bean注解的方法都会被动态代理，因此调用该方法返回的都是同一个实例。
理解：调用@Configuration类中的@Bean注解的方法，返回的是同一个实例；而调用@Component类中的@Bean注解的方法，返回的是一个新的实例。

> [终于搞懂了@Configuration和@Component的区别](https://mp.weixin.qq.com/s/-_h5Hz6MOBb8TK3qm9gBog)
> [@Configuration和@Component有何区别？](https://mp.weixin.qq.com/s/D84pWlXs7wbHFYvCE5TAVQ)

### 注入Bean的注解有哪些？

Spring内置的@Autowired以及JDK内置的@Resource和@Inject都可以用于注入Bean。

| Annotaion    | Package                            | Source       |
| ------------ | ---------------------------------- | ------------ |
| @Autowired | org.springframework.bean.factory | Spring 2.5+  |
| @Resource  | javax.annotation                 | Java JSR-250 |
| @Inject    | javax.inject                     | Java JSR-330 |

@Autowired和@Resource使用的比较多一些。

### @Autowired和@Resource的区别是什么？

Autowired属于Spring内置的注解，默认的注入方式为byType（根据类型进行匹配），也就是说会优先根据接口类型去匹配并注入Bean（接口的实现类）。这会有什么问题呢？当一个接口存在多个实现类的话，byType这种方式就无法正确注入对象了,因为这个时候Spring会同时找到多个满足条件的选择，默认情况下它自己不知道选择哪一个。这种情况下，注入方式会变为byName（根据名称进行匹配），这个名称通常就是类名（首字母小写）。就比如说下面代码中的smsService就是我这里所说的名称，这样应该比较好理解了吧。


```java
// smsService就是我们上面所说的名称
@Autowired
private SmsService smsService;
```

举个例子，SmsService接口有两个实现类:SmsServiceImpl1和SmsServiceImpl2，且它们都已经被Spring容器所管理。


```java
// 报错，byName和byType都无法匹配到bean
@Autowired
private SmsService smsService;

// 正确注入SmsServiceImpl1对象对应的bean
@Autowired
private SmsService smsServiceImpl1;

// 正确注入SmsServiceImpl1对象对应的bean
// smsServiceImpl1就是我们上面所说的名称
@Autowired
@Qualifier(value="smsServiceImpl1")
private SmsService smsService;
```

我们还是建议通过@Qualifier注解来显式指定名称而不是依赖变量的名称。

@Resource属于JDK提供的注解，默认注入方式为byName。如果无法通过名称匹配到对应的Bean的话，注入方式会变为byType。@Resource有两个比较重要且日常开发常用的属性：name（名称）、type（类型）。

```java
public @interface Resource {
    String name() default "";
    Class<?> type() default Object.class;
}
```

如果仅指定name属性则注入方式为byName，如果仅指定type属性则注入方式为byType，如果同时指定name和type属性（不建议这么做）则注入方式为byType+byName。

```java
// 报错，byName和byType都无法匹配到bean
@Resource
private SmsService smsService;

// 正确注入SmsServiceImpl1对象对应的bean
@Resource
private SmsService smsServiceImpl1;

// 正确注入SmsServiceImpl1对象对应的bean（比较推荐这种方式）
@Resource(name = "smsServiceImpl1")
private SmsService smsService;
```

简单总结一下：

- @Autowired是Spring提供的注解，@Resource是JDK提供的注解。
- @Autowired默认的注入方式为byType（根据类型进行匹配），@Resource默认注入方式为byName（根据名称进行匹配）。
- 当一个接口存在多个实现类的情况下，@Autowired和@Resource都需要通过名称才能正确匹配到对应的Bean。Autowired可以通过@Qualifier注解来显式指定名称，@Resource可以通过name属性来显式指定名称。

**@Autowired和@Resource区别**😊

- @Autowired与@Resource都可以用来装配bean,都可以写在字段上,或写在setter方法上。
- @Autowired默认按类型装配（这个注解是属于spring的）,默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，如：@Autowired(required=false)，如果我们想使用名称装配可以结合@Qualifier注解进行使用，如下：

```java
@Autowired ()
@Qualifier ("baseDao")
private BaseDao baseDao;
```

- @Resource（这个注解属于J2EE的），默认按照名称进行装配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名进行安装名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。
- @Autowired只按照byType注入,由Spring提供，@Resource默认按byName自动注入，也提供按照byType注入

> [Spring探索｜既生@Resource，何生@Autowired？](https://mp.weixin.qq.com/s/MZX97YKKmjuj7FxrjBQ1hg)
> [@Autowired注解是如何实现的？](https://mp.weixin.qq.com/s/gRqZwUV791RtCI1xCoV3Qw)
> [@Autowired到底是怎么把变量注入进来的？](https://mp.weixin.qq.com/s/Ecs4MTjFpCa6Rz75buTSNw)
> [你所不知道的Spring中@Autowired那些实现细节](https://mp.weixin.qq.com/s/n_syhEFrXykI7ySRtahEmg)
> [@Autowired的这些骚操作，你都知道吗？](https://mp.weixin.qq.com/s/2X5xv8I0b6TcXWVH-SC8Ug)

**@Inject**

1. @Inject是JSR330(Dependency Injection for Java)中的规范，需要导入javax.inject.Inject,实现注入。
2. @Inject是根据类型进行自动装配的，如果需要按名称进行装配，则需要配合@Named
3. @Inject可以作用在变量、setter方法、构造函数上。

```java
private Abc abc;
@Inject
public void setAbc(@Named("beanName") Abc abc){
	this.abc = abc;
}
```

> [@Autowired,@Resource,@Inject三个注解的区别](https://mp.weixin.qq.com/s/YLIsRBSiIjz3dCtSA9onDQ)

1. @Autowired是Spring自带的，@Inject和@Resource都是JDK提供的，其中@Inject是JSR330规范实现的，@Resource是JSR250规范实现的，而Spring通过BeanPostProcessor来提供对JDK规范的支持。
2. @Autowired、@Inject用法基本一样，不同之处为@Autowired有一个required属性，表示该注入是否是必须的，即如果为必须的，则如果找不到对应的bean，就无法注入，无法创建当前bean。
3. @Autowired、@Inject是默认按照类型匹配的，@Resource是按照名称匹配的。如在spring-boot-data项目中自动生成的redisTemplate的bean，是需要通过byName来注入的。如果需要注入该默认的，则需要使用@Resource来注入，而不是@Autowired。
4. 对于@Autowire和@Inject，如果同一类型存在多个bean实例，则需要指定注入的beanName。@Autowired和@Qualifier一起使用，@Inject和@Named一起使用。

### Bean的作用域有哪些?

Spring中Bean的作用域通常有下面几种：

- **singleton**：IoC容器中只有唯一的bean实例。Spring中的bean默认都是单例的，是对单例设计模式的应用。
- **prototype**：每次获取都会创建一个新的bean实例。也就是说，连续`getBean()`两次，得到的是不同的Bean实例。
- **request（仅Web应用可用）**:每一次HTTP请求都会产生一个新的bean（请求bean），该bean仅在当前HTTPrequest内有效。
- **session（仅Web应用可用）**:每一次来自新session的HTTP请求都会产生一个新的bean（会话bean），该bean仅在当前HTTPsession内有效。
- **application/global-session（仅Web应用可用）**：每个Web应用在启动时创建一个Bean（应用Bean），该bean仅在当前应用启动时间内有效。
- **websocket（仅Web应用可用）**：每一次WebSocket会话产生一个新的bean。

**如何配置bean的作用域呢？**

xml方式：

```xml
<bean id="..." class="..." scope="singleton"></bean>
```

注解方式：

```java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}
```

### 单例Bean的线程安全问题了解吗？

大部分时候我们并没有在项目中使用多线程，所以很少有人会关注这个问题。单例Bean存在线程问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的。常见的有两种解决办法：

1. 在Bean中尽量避免定义可变的成员变量。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中（推荐的一种方式）。

不过，大部分Bean实际都是无状态（没有实例变量）的（比如Dao、Service），这种情况下，Bean是线程安全的。


## Spring Bean生命周期

图示：
![Spring Bean生命周期](https://images.xiaozhuanlan.com/photo/2019/24bc2bad3ce28144d60d9e0a2edf6c7f.jpg)

与之比较类似的中文版本:

![Spring Bean生命周期](https://images.xiaozhuanlan.com/photo/2019/b5d264565657a5395c2781081a7483e1.jpg)

### 一、获取Bean

![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPYhVDaaP8cNKOLWfufL5rQXaMa7xPp4N8NAI2162lm2Rrwvl8sibVCjg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

#### 第一阶段获取Bean

这里的流程图的入口在AbstractBeanFactory类的doGetBean方法，这里可以配合前面的getBean方法分析文章进行阅读。主要流程就是
1. 先处理Bean的名称，因为如果以“&”开头的Bean名称表示获取的是对应的FactoryBean对象
2. 从缓存中获取单例Bean，有则进一步判断这个Bean是不是在创建中，如果是的就等待创建完毕，否则直接返回这个Bean对象
3. 如果不存在单例Bean缓存，则先进行循环依赖的解析
4. 解析完毕之后先获取父类BeanFactory，获取到了则调用父类的getBean方法，不存在则先合并然后创建Bean

### 二、创建Bean

#### 2.1 创建Bean之前

![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPicgybuOPvUicWBAxrM1rT0PhJeZ1ftRibJGWGYM7P0f5XMga9QCrSlFFQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**在真正创建Bean之前逻辑**
这个流程图对应的代码在AbstractAutowireCapableBeanFactory类的createBean方法中。

(1)这里会先获取RootBeanDefinition对象中的Class对象并确保已经关联了要创建的Bean的Class。
(2)这里会检查3个条件：

- Bean的属性中的beforeInstantiationResolved字段是否为true，默认是false。
- Bean是原生的Bean。
- Bean的hasInstantiationAwareBeanPostProcessors属性为true，这个属性在Spring准备刷新容器BeanPostProcessors的时候会设置，如果当前Bean实现了InstantiationAwareBeanPostProcessor则这个就会是true。

当三个条件都存在的时候，就会调用实现的InstantiationAwareBeanPostProcessor接口的postProcessBeforeInstantiation方法，然后获取返回的Bean，如果返回的Bean不是null还会调用实现的BeanPostProcessor接口的postProcessAfterInitialization方法，这里用代码说明：

```java
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
        Object bean = null;
        //条件1
        if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
            //条件2跟条件3
            if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
                Class<?> targetType = determineTargetType(beanName, mbd);
                if (targetType != null) {
                    //调用实现的postProcessBeforeInstantiation方法
                    bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                    if (bean != null) {
                       //调用实现的postProcessAfterInitialization方法
                        bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                    }
                }
            }
            //不满足2或者3的时候就会设置为false
            mbd.beforeInstantiationResolved = (bean != null);
        }
        return bean;
    }
```

(3)如果上面3个条件其中一个不满足就不会调用实现的方法。默认这里都不会调用的这些BeanPostProcessors的实现方法。然后继续执行后面的doCreateBean方法。

#### 2.2 真正的创建Bean，doCreateBean

![](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPIhTibribNrjwS7O5fH8doMAibkvl5icWLeq16ibP52JcxspfB8nDtyMhKQA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**doCreateBean方法逻辑**
这个代码的实现还是在AbstractAutowireCapableBeanFactory方法中。流程是
1. 先检查instanceWrapper变量是不是null，这里一般是null，除非当前正在创建的Bean在factoryBeanInstanceCache中存在这个是保存还没创建完成的FactoryBean的集合。
2. 调用createBeanInstance方法实例化Bean，这个方法在后面会讲解
3. 如果当前RootBeanDefinition对象还没有调用过实现了的MergedBeanDefinitionPostProcessor接口的方法，则会进行调用。
4. 当满足以下三点
（1）是单例Bean
（2）尝试解析bean之间的循环引用
（3）bean目前正在创建中
则会进一步检查是否实现了SmartInstantiationAwareBeanPostProcessor接口如果实现了则调用是实现的getEarlyBeanReference方法
5. 调用populateBean方法进行属性填充，这里后面会讲解
6. 调用initializeBean方法对Bean进行初始化，这里后面会讲解

##### 2.2.1 实例化Bean，createBeanInstance

![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpP4WkTkskaiaq1XKqJAEKWhLeNicuTJSsicuK7licC9doicxAbdr01YF0taQg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**实例化Bean**

这里的逻辑稍微有一点复杂，这个流程图已经是简化过后的了。简要根据代码说明一下流程

```java
    protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
        //步骤1
        Class<?> beanClass = resolveBeanClass(mbd, beanName);

        if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                    "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
        }
        //步骤2
        Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
        if (instanceSupplier != null) {
            return obtainFromSupplier(instanceSupplier, beanName);
        }
        //步骤3
        if (mbd.getFactoryMethodName() != null) {
            return instantiateUsingFactoryMethod(beanName, mbd, args);
        }
        boolean resolved = false;
        boolean autowireNecessary = false;
        if (args == null) {
            synchronized (mbd.constructorArgumentLock) {
                if (mbd.resolvedConstructorOrFactoryMethod != null) {
                    resolved = true;
                    autowireNecessary = mbd.constructorArgumentsResolved;
                }
            }
        }
        //步骤4.1
        if (resolved) {
            if (autowireNecessary) {
                return autowireConstructor(beanName, mbd, null, null);
            }
            else {
                return instantiateBean(beanName, mbd);
            }
        }

          //步骤4.2
        Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
        if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
                mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
            return autowireConstructor(beanName, mbd, ctors, args);
        }
        //步骤5
        ctors = mbd.getPreferredConstructors();
        if (ctors != null) {
            return autowireConstructor(beanName, mbd, ctors, null);
        }
        return instantiateBean(beanName, mbd);
    }
```

1. 先检查Class是否已经关联了，并且对应的修饰符是否是public的
2. 如果用户定义了Bean实例化的函数，则调用并返回
3. 如果当前Bean实现了FactoryBean接口则调用对应的FactoryBean接口的getObject方法
4. 根据getBean时候是否传入构造参数进行处理
4.1如果没有传入构造参数，则检查是否存在已经缓存的无参构造器，有则使用构造器直接创建，没有就会调用instantiateBean方法先获取实例化的策略默认是CglibSubclassingInstantiationStrategy，然后实例化Bean。最后返回
4.2如果传入了构造参数，则会先检查是否实现了SmartInstantiationAwareBeanPostProcessor接口，如果实现了会调用determineCandidateConstructors获取返回的候选构造器。
4.3检查4个条件是否满足一个
（1）构造器不为null，
（2）从RootBeanDefinition中获取到的关联的注入方式是构造器注入（没有构造参数就是setter注入，有则是构造器注入）
（3）含有构造参数
（4）getBean方法传入构造参数不是空
满足其中一个则会调用返回的候选构造器实例化Bean并返回，如果都不满足，则会根据构造参数选则合适的有参构造器然后实例化Bean并返回
5. 如果上面都没有合适的构造器，则直接使用无参构造器创建并返回Bean。

##### 2.2.2 填充Bean，populateBean

![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPRaNrWofKRqgPdvMFQn03uicb2NmqJCHcRzncyuoobJ7alPiaOpVPGR8g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**填充Bean**
这里还是根据代码来说一下流程

```java
    protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
        if (bw == null) {
            if (mbd.hasPropertyValues()) {
                throw new BeanCreationException(
                        mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
            }
            else {
                // Skip property population phase for null instance.
                return;
            }
        }
        boolean continueWithPropertyPopulation = true;
        //步骤1
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            for (BeanPostProcessor bp : getBeanPostProcessors()) {
                if (bp instanceof InstantiationAwareBeanPostProcessor) {
                    InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                    if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                        continueWithPropertyPopulation = false;
                        break;
                    }
                }
            }
        }

        if (!continueWithPropertyPopulation) {
            return;
        }
        //步骤2--------------------
        PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

        int resolvedAutowireMode = mbd.getResolvedAutowireMode();
        if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
            MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
            // Add property values based on autowire by name if applicable.
            if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
                autowireByName(beanName, mbd, bw, newPvs);
            }
            // Add property values based on autowire by type if applicable.
            if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
                autowireByType(beanName, mbd, bw, newPvs);
            }
            pvs = newPvs;
        }

        boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
        boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

        PropertyDescriptor[] filteredPds = null;
        //步骤3
        if (hasInstAwareBpps) {
            if (pvs == null) {
                pvs = mbd.getPropertyValues();
            }
            for (BeanPostProcessor bp : getBeanPostProcessors()) {
                if (bp instanceof InstantiationAwareBeanPostProcessor) {
                    InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
                    PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
                    if (pvsToUse == null) {
                        if (filteredPds == null) {
                            filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                        }
                        pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
                        if (pvsToUse == null) {
                            return;
                        }
                    }
                    pvs = pvsToUse;
                }
            }
        }
        if (needsDepCheck) {
            if (filteredPds == null) {
                filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
            }
            checkDependencies(beanName, mbd, filteredPds, pvs);
        }
        //步骤4
        if (pvs != null) {
            applyPropertyValues(beanName, mbd, bw, pvs);
        }
    }
```

1. 检查当前Bean是否实现了InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation方法则调用，并结束Bean的填充。
2. 将按照类型跟按照名称注入的Bean分开，如果注入的Bean还没有实例化的这里会实例化，然后放到PropertyValues对象中。
3. 如果实现了InstantiationAwareBeanPostProcessor类的postProcessProperties则调用这个方法并获取返回值，如果返回值是null，则有可能是实现了过期的postProcessPropertyValues方法，这里需要进一步调用postProcessPropertyValues方法
4. 进行参数填充

##### 2.2.3 初始化Bean，initializeBean

![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPZjJibwdUibfEibHoFzlWI6yFbIlaG2EvckACOCY5mneiaibpOZfZrtQICibw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**初始化Bean**
同时这里根据代码跟流程图来说明

- 如果Bean实现了BeanNameAware,BeanClassLoaderAware,BeanFactoryAware则调用对应实现的方法。
- Bean不为null并且bean不是合成的，如果实现了BeanPostProcessor的postProcessBeforeInitialization则会调用实现的postProcessBeforeInitialization方法。在ApplicationContextAwareProcessor类中实现了postProcessBeforeInitialization方法。而这个类会在Spring刷新容器准备beanFactory的时候会加进去，这里就会被调用，而调用里面会检查Bean是不是EnvironmentAware,EmbeddedValueResolverAware,ResourceLoaderAware,ApplicationEventPublisherAware,MessageSourceAware,ApplicationContextAware的实现类。这里就会调用对应的实现方法。代码如下

```java
    protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        .......
        beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
        .......
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (!(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
                bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
                bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)){
            return bean;
        }

        AccessControlContext acc = null;

        if (System.getSecurityManager() != null) {
            acc = this.applicationContext.getBeanFactory().getAccessControlContext();
        }

        if (acc != null) {
            AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
                invokeAwareInterfaces(bean);
                return null;
            }, acc);
        }
        else {
            invokeAwareInterfaces(bean);
        }

        return bean;
    }
```

- 实例化Bean然后，检查是否实现了InitializingBean的afterPropertiesSet方法，如果实现了就会调用
- Bean不为null并且bean不是合成的，如果实现了BeanPostProcessor的postProcessBeforeInitialization则会调用实现的postProcessAfterInitialization方法。

到此创建Bean的流程就没了，剩下的就是容器销毁的时候的了

### 三、destory方法跟销毁Bean

Bean在创建完毕之后会检查用户是否指定了destroyMethodName以及是否实现了DestructionAwareBeanPostProcessor接口的requiresDestruction方法，如果指定了会记录下来保存在DisposableBeanAdapter对象中并保存在bean的disposableBeans属性中。代码在AbstractBeanFactory的registerDisposableBeanIfNecessary中

```java
    protected void registerDisposableBeanIfNecessary(String beanName, Object bean, RootBeanDefinition mbd) {
          ......
                registerDisposableBean(beanName,
                        new DisposableBeanAdapter(bean, beanName, mbd, getBeanPostProcessors(), acc));
            ......
    }
public DisposableBeanAdapter(Object bean, String beanName, RootBeanDefinition beanDefinition,
            List<BeanPostProcessor> postProcessors, @Nullable AccessControlContext acc) {
          .......
        String destroyMethodName = inferDestroyMethodIfNecessary(bean, beanDefinition);
        if (destroyMethodName != null && !(this.invokeDisposableBean && "destroy".equals(destroyMethodName)) &&
                !beanDefinition.isExternallyManagedDestroyMethod(destroyMethodName)) {
            ......
            this.destroyMethod = destroyMethod;
        }
        this.beanPostProcessors = filterPostProcessors(postProcessors, bean);
    }
```

在销毁Bean的时候最后都会调用AbstractAutowireCapableBeanFactory的destroyBean方法。

```java
    public void destroyBean(Object existingBean) {
        new DisposableBeanAdapter(existingBean, getBeanPostProcessors(), getAccessControlContext()).destroy();
    }
```

这里是创建一个DisposableBeanAdapter对象，这个对象实现了Runnable接口，在实现的run方法中会调用实现的DisposableBean接口的destroy方法。并且在创建DisposableBeanAdapter对象的时候会根据传入的bean是否实现了DisposableBean接口来设置invokeDisposableBean变量，这个变量表实有没有实现DisposableBean接口

```java
    public DisposableBeanAdapter(Object bean, List<BeanPostProcessor> postProcessors,AccessControlContext acc) {
        Assert.notNull(bean, "Disposable bean must not be null");
        this.bean = bean;
        this.beanName = bean.getClass().getName();
        //根据传入的bean是否实现了`DisposableBean`接口来设置`invokeDisposableBean`变量
        this.invokeDisposableBean = (this.bean instanceof DisposableBean);
        this.nonPublicAccessAllowed = true;
        this.acc = acc;
        this.beanPostProcessors = filterPostProcessors(postProcessors, bean);
    }

    public void destroy() {
        ......
        //根据invokeDisposableBean决定是否调用destroy方法
        if (this.invokeDisposableBean) {
            if (logger.isTraceEnabled()) {
                logger.trace("Invoking destroy() on bean with name '" + this.beanName + "'");
            }
            try {
                if (System.getSecurityManager() != null) {
                    AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                        ((DisposableBean) this.bean).destroy();
                        return null;
                    }, this.acc);
                }
                else {
                    ((DisposableBean) this.bean).destroy();
                }
            }
            catch (Throwable ex) {
                String msg = "Invocation of destroy method failed on bean with name '" + this.beanName + "'";
                if (logger.isDebugEnabled()) {
                    logger.warn(msg, ex);
                }
                else {
                    logger.warn(msg + ": " + ex);
                }
            }
        }
    }
```

### 四、Bean的初始化和销毁的几种方式

#### 初始化

- 实现InitializingBean接口,覆盖其中的afterPropertiesSet()方法
- 增加@PostConstruct注解
- 自定义init方法(@Bean(initMethod = "initMethod"))
执行的顺序依次是postConstruct,afterPropertiesSet,initMethod

#### 销毁

- 实现org.springframework.beans.factory.DisposableBean接口，覆盖destroy()方法
- 自定义一个方法，在方法上面增加@PreDestroy注解
- 在InitServiceImpl中增加一个自定义销毁方法，然后在配置类中增加Bean的destoryMethod
执行的顺序依次是preDestroy,destroy,destroyMethod


### 五、总结


> [Bean的生命周期（五步、七步、十步法剖析）](https://blog.csdn.net/m0_61933976/article/details/128697003)
> 
> **五步分析法**：
> > 第一步：实例化Bean（调用无参数构造方法）。
> > 第二步：Bean属性赋值（调用set方法）。
> > 第三步：初始化Bean（会调用Bean的init方法。注意：这个init方法需要自己写）。
> > 第四步：使用Bean。
> > 第五步：销毁Bean（会调用Bean的destroy方法。注意：这个destroy方法需要自己写）。
>
> **七步分析法**：在以上的5步中，第3步是初始化Bean，如果你还想在初始化前和初始化后添加代码，可以加入“Bean后处理器”；需要编写一个类实现BeanPostproccessor接口，并重写里面的befor和after方法。
>
> > 第一步：实例化Bean。
> > 第二步：Bean属性赋值。
> > 第三步：执行“Bean后处理器”的before方法。
> > 第四步：初始化Bean。
> > 第五步：执行“Bean后处理器”的after方法。
> > 第六步：使用Bean。
> > 第七步：销毁Bean
>
> **十步分析法**：比七步添加的那三步在哪里？
>
> > （1）在“Bean后处理器”before方法之前干了什么事儿？检查Bean是否实现了Aware相关的接口，如果实现了接口则调用这些接口中的方法；调用这些方法的目的是为了给你传递一些数据，让你更加方便使用。
> > （2）在“Bean后处理器”before方法之后干了什么事儿？检查Bean是否实现了InitializingBean接口，如果实现了，则调用接口中的方法。
> > （3）使用Bean之后，或者说销毁Bean之前干了什么事儿？检查Bean是否实现了DisposableBean接口，如果实现了，则调用接口中的方法。总结：添加的这三个点位的特点，都是在检查你这个Bean是否实现了某些特定的接口，如果实现了这些接口，则Spring容器会调用这个接口中的方法！

最后来一个大的流程

**实例化前的准备阶段**
![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPQu4pzSyprviaBic07GicVGPvAUdAibkFqybnvOfgdzdw1M1iaMtm9qfBLDQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)


**实例化前**
![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPic9W1CbpBia73nS2WJAGKRMdW9LtwbxG30IqbNT8ibvH5DfqcHO2IueBw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)


**实例化后**
![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPp3ia71rKnC0FeypESdhAFYAqGicz9KP9LeBxaJHKmvMPUDIGrBdBkBiag/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**初始化前**
![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPjfQ5qaic2Ro6hoqhCdoicgiabmkibR518z7vSpXxmibq91FH1XxgHvdet8Q/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**初始化后&销毁**
![图片](https://mmbiz.qpic.cn/mmbiz_png/SJm51egHPPGPI5JCBzTotEAS720l5YpPABU277ApFU3EVr8iaHxtFEVvsawgghYyJd7WlJQFwEkQvXoDW2sQEYQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)


### 六、Bean的扩展接口

**顺序一**

- Bean容器找到配置文件中Spring Bean的定义。
- Bean容器利用Java Reflection API创建一个Bean的实例。
- 如果涉及到一些属性值利用set()方法设置一些属性值。
- 如果Bean实现了BeanNameAware接口，调用setBeanName()方法，传入Bean的名字。
- 如果Bean实现了BeanClassLoaderAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。
- 如果Bean实现了BeanFactoryAware接口，调用setBeanFactory()方法，传入BeanFactory对象的实例。
- 与上面的类似，如果实现了其他\*.Aware接口，就调用相应的方法。
- 如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessBeforeInitialization()方法
- 如果Bean实现了InitializingBean接口，执行afterPropertiesSet()方法。
- 如果Bean在配置文件中的定义包含init-method属性，执行指定的方法。
- 如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessAfterInitialization()方法
- 当要销毁Bean的时候，如果Bean实现了DisposableBean接口，执行destroy()方法。
- 当要销毁Bean的时候，如果Bean在配置文件中的定义包含destroy-method属性，执行指定的方法。

**顺序二**

- spring启动，加载类路径下配置文件，解析为BeanDefinition并装配到对应容器中
- 查找并加载spring管理的bean，进行bean的实例化
- Bean实例化后对Bean的引用和值进行属性注入
- 若Bean实现接口BeanNameAware，则执行setBeanName()方法，获取bean的名字
- 若Bean实现接口BeanFactoryAware，则执行setBeanFactory()方法，获取BeanFactory
- 若Bean实现接口ApplicationContextAware，则执行setApplicationContext()方法，获取应用上下文
- 若Bean实现BeanPostProcessor接口，则先执行postProcessBeforeInitialization()方法
- 若Bean实现InitializingBean接口，则执行afterPropertiesSet()方法
- 若Bean配置了init-method方法，则执行自定义方法
- 若Bean实现BeanPostProcessor接口，则先执行postProcessAfterInitialization()方法
- 如Bean实现了DisposableBean接口，则容器销毁时则执行destory()方法
- 如果Bean配置了destory-method，则容器销毁时则执行自定义方法。

**顺序三(自己测试的结果)**

在将一个Bean对象配置在IOC容器中之后，这个Bean的生命周期就会交由IOC容器进行管理。一般担当管理者的角色是BeanFactory或ApplicationContext。在将一个bean对象配置在ioc容器中之后，这个bean的生命周期就会交由ioc容器进行管理。一般担当管理者的角色是BeanFactory和ApplicationContext。

1. bean的创建
在解析ioc容器时，根据解析容器的工厂，决定bean的初始化时间
BeanFactory-getBean()方法调用时初始化bean
ApplicationContext-解析ioc容器时初始化bean
2. setter注入
根据bean子元素的配置实现bean之间的被动注入
3. BeanNameAware
如果bean实现了该接口，执行其setBeanName(String name)方法.参数name是bean在容器中的名称,即xml里面bean的id名称
4. BeanFactoryAware
如果实现了该接口，执行其setBeanFactory(BeanFactory factory)方法，参数是创建Bean的BeanFactory本身
5. ApplicationContextAware
如果这个Bean已经实现了该接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文（同样这个方式也可以实现步骤4的内容，但比4更好，因为ApplicationContext是BeanFactory的子接口，有更多的实现方法）
org.springframework.beans.context.ApplicationContextAware.当需要从spring容器中获取bean时一般使用这种方式获取:
```java
ApplicationContext appContext = new ClassPathXmlApplicationContext("applicationContext-common.xml");
AbcService abcService = (AbcService)appContext.getBean("abcService");
```
但是这样就会存在一个问题：因为它会重新装载applicationContext-common.xml并实例化上下文bean，如果有些线程配置类也是在这个配置文件中，那么会造成做相同工作的的线程会被启两次。一次是web容器初始化时启动，另一次是上述代码显示的实例化了一次。当于重新初始化一遍！这样就产生了冗余,所以可以通过实现ApplicationContextAware接口获取bean,当一个类实现了ApplicationContextAware之后，这个类就可以方便获得ApplicationContext中的所有bean。换句话说，就是这个类可以直接获取spring配置文件中所有有引用到的bean对象.
代码:

```java
private static ApplicationContext applicationContext;
@Override
public void setApplicationContext(ApplicationContext arg0) throws BeansException {
    applicationContext = arg0;
}
```
注意：从ApplicationContextAware获取ApplicationContext上下文的情况，仅仅适用于当前运行的代码和已启动的Spring代码处于同一个Spring上下文，否则获取到的ApplicationContext是空的
6. BeanPostProcessor(前置方法)
ioc容器中如果有bean实现了该接口，那所有的bean在初始化之前都会执行其实例的postProcessBeforeInitialization(Object bean, String beanName)前置方法，BeanPostProcessor经常被用作是Bean内容的更改,该方法最后返回bean
7. @PostConstruct修饰的非静态方法
8. InitializingBean
如果实现了该接口，则允许一个bean在它的所有必须属性被BeanFactory设置后，来执行初始化的工作，会自动调用afterPropertiesSet()方法对Bean进行初始化，实现此接口的话正常情况下配置文件就不用指定init-method属性了。
9. 如果Bean在Spring中配置了init-method属性，调用init-method属性指向的方法,此时完成bean的初始化
10. BeanPostProcessor(后置方法)
ioc容器中如果有bean实现了接口，那所有的bean在初始化之后都会执行其实例的postProcessAfterInitialization(Object bean, String beanName)后置方法
11. 实现SmartInitializingSingleton的接口后，当所有单例bean都初始化完成以后，Spring的IOC容器会回调该接口的afterSingletonsInstantiated()方法,主要应用场合就是在所有单例bean创建完成之后，可以在该回调中做一些事情。执行时机在ApplicationContextAware执行之后
12. @PreDestroy修饰的方法
13. ioc容器关闭时，如果bean实现了DisposableBean接口，则执行其destory()方法，在Bean生命周期结束前调用destory()方法做一些收尾工作,重写destroy()方法
14. 如果这个Bean在Spring配置了destroy-method属性，执行destory-method属性指向的方法

> [Spring Boot启动扩展点超详细总结，再也不怕面试官问了](https://mp.weixin.qq.com/s/l0O3C_UiO3CdfNE2V73qmA)


![](/images/bean.png)

![](https://img-blog.csdnimg.cn/500f240463544992ad05bab3408c56eb.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAS0vlsI_lk6U=,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

**简单来说一个Bean的加载顺序：类构造方法 - postProcessBeforeInitialization前置方法 - @PostConstruct注解的方法 - InitializingBean的afterPropertiesSet()方法- XML中定义的bean init-method方法 - postProcessAfterInitialization后置方法**

> [Spring Boot启动扩展点超详细总结，再也不怕面试官问了](https://mp.weixin.qq.com/s/l0O3C_UiO3CdfNE2V73qmA)

#### BeanFactoryPostProcessor、BeanPostProcessor区别

BeanFactoryPostProcessor：针对bean工厂，BeanFactory后置处理器，是对BeanDefinition对象进行修改，可以修改BeanDefinition对象中的属性。（BeanDefinition：存储bean标签的信息，用来生成bean实例）,BeanFactoryPostProcessor的实现类可以在当前BeanFactory初始化（spring容器加载bean定义文件）后，bean实例化之前修改bean的定义属性，达到影响之后实例化bean的效果。也就是说，Spring允许BeanFactoryPostProcessor在容器实例化任何其它bean之前读取配置元数据，并可以根据需要进行修改，例如可以把bean的scope从singleton改为prototype，也可以把property的值给修改掉。可以同时配置多个BeanFactoryPostProcessor，并通过设置’order’属性来控制各个BeanFactoryPostProcessor的执行次序.
BeanPostProcessor：针对bean,Bean后置处理器，是对生成的Bean对象进行修改。BeanPostProcessor能在spring容器实例化bean之后，在执行bean的初始化方法前后，添加一些自己的处理逻辑。初始化方法包括以下两种：
1. 实现InitializingBean接口的bean，对应方法为afterPropertiesSet
2. xml定义中，通过init-method设置的方法,BeanPostProcessor是BeanFactoryPostProcessor之后执行的。

> [BeanFactoryPostProcessor和BeanPostProcessor有什么区别？](https://mp.weixin.qq.com/s/ZjN1XPamDaYZmvFbyI1KTQ)

#### BeanFactroy、ApplicationContext区别

1. BeanFactroy采用的是延迟加载形式来注入Bean的，即只有在使用到某个Bean时(调用getBean())，才对该Bean进行加载实例化，这样，我们就不能发现一些存在的Spring的配置问题。而ApplicationContext则相反，它是在容器启动时，一次性创建了所有的Bean。这样，在容器启动时，我们就可以发现Spring中存在的配置错误。相对于基本的BeanFactory，ApplicationContext唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。BeanFacotry延迟加载,如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常；而ApplicationContext则在初始化自身是检验，这样有利于检查所依赖属性是否注入；所以通常情况下我们选择使用ApplicationContext。应用上下文则会在上下文启动后预载入所有的单实例Bean。通过预载入单实例bean,确保当你需要的时候，你就不用等待，因为它们已经创建好了。
2. BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。（Applicationcontext比beanFactory加入了一些更好使用的功能。而且beanFactory的许多功能需要通过编程实现而Applicationcontext可以通过配置实现。比如后处理bean，Applicationcontext直接配置在配置文件即可而beanFactory这要在代码中显示的写出来才可以被容器识别。）
3. beanFactory主要是面对与spring框架的基础设施，面对spring自己。而Applicationcontex主要面对与spring使用的开发者。基本都会使用Applicationcontex并非beanFactory。

> [Spring系列之beanFactory与ApplicationContext](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493943&idx=1&sn=9eaa46ed730874fce003c66f76fe9c7f&source=41#wechat_redirect)
> [简单把Spring容器分为了两大类！](https://mp.weixin.qq.com/s/aOOQiBmBmNy4ZjHtv1phdQ)

#### BeanFactory和FactoryBean的区别

BeanFactory是Spring容器的顶级接口，给具体的IOC容器的实现提供了规范。
FactoryBean也是接口，为IOC容器中Bean的实现提供了更加灵活的方式，FactoryBean在IOC容器的基础上给Bean的实现加上了⼀个简单工厂模式和装饰模式,我们可以在getObject()方法中灵活配置。其实在Spring源码中有很多FactoryBean的实现类。
区别：BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是个Bean。在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。但对FactoryBean而言，这个Bean不是简单的Bean，而是⼀个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似。

##### BeanFactory

BeanFactory，以Factory结尾，表示它是⼀个工厂类(接口)，它负责生产和管理bean的⼀个工厂。在Spring中，BeanFactory是IOC容器的核心接口，它的职责包括：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现，如DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等，其中XmlBeanFactory就是常用的⼀个，该实现将以XML方式描述组成应用的对象及对象间的依赖关系。XmlBeanFactory类将持有此XML配置元数据，并用它来构建⼀个完全可配置的系统或应用。都是附加了某种功能的实现。它为其他具体的IOC容器提供了最基本的规范，例如DefaultListableBeanFactory,XmlBeanFactory,ApplicationContext等具体的容器都是实现了BeanFactory，再在其基础之上附加了其他的功能。BeanFactory和ApplicationContext就是Spring框架的两个IOC容器，现在⼀般使用ApplicationnContext，其不但包含了BeanFactory的作用，同时还进行更多的扩展。BeanFacotry是Spring中比较原始的Factory。如XMLBeanFactory就是⼀种典型的BeanFactory。原始的BeanFactory无法⽀持Spring的许多插件，如AOP功能、Web应用等。ApplicationContext接口,它由BeanFactory接口派生而来，ApplicationContext包含BeanFactory的所有功能，通常建议比BeanFactory优先，ApplicationContext以⼀种更面向框架的方式工作以及对上下文进行分层和实现继承，ApplicationContext包还提供了以下的功能：MessageSource,提供国际化的消息访问
资源访问，如URL和⽂件，事件传播，载入多个（有继承关系）上下文，使得每⼀个上下文都专注于⼀个特定的层次，比如应⽤的web层;

BeanFactory提供的方法及其简单，仅提供了六种方法供客户调用：

```java
// 判断⼯⼚中是否包含给定名称的bean定义，若有则返回true
boolean containsBean(String beanName)
// 返回给定名称注册的bean实例。根据bean的配置情况，如果是singleton模式将返回⼀个共享实例，否则将返回⼀个新建的实例，如果没有找到指定bean,该⽅法可能会抛出异常
Object getBean(String)
// 返回以给定名称注册的bean实例，并转换为给定class类型
Object getBean(String, Class)
// 返回给定名称的bean的Class,如果没有找到指定的bean实例，则排除NoSuchBeanDefinitionException异常
Class getType(String name)
// 判断给定名称的bean定义是否为单例模式
boolean isSingleton(String)
// 返回给定bean名称的所有别名
String[] getAliases(String name)
```
##### FactoryBean

⼀般情况下，Spring通过反射机制利用<bean\><bean\>的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean\><bean\>中提供大量的配置信息。配置⽅式的灵活性是受限的，这时采用编码的方式可能会得到⼀个简单的方案。Spring为此提供了⼀个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接⼝定制实例化Bean的逻辑。FactoryBean接口对于Spring框架来说占重要的地位，Spring自身就提供了70多个FactoryBean的实现。它们隐藏了实例化⼀些复杂Bean的细节，给上层应用带来了便利。从Spring3.0开始，FactoryBean开始⽀持泛型，即接口声明改为FactoryBean<T\>的形式,以Bean结尾，表示它是⼀个Bean，不同于普通Bean的是：它是实现了FactoryBean<T\>接口的Bean，根据该Bean的ID从BeanFactory中获取的实际上是FactoryBean的getObject()返回的对象，而不是FactoryBean本身，如果要获取FactoryBean对象，请在id前面加⼀个&符号来获取。例如自己实现⼀个FactoryBean，功能：用来代理⼀个对象，对该对象的所有方法做⼀个拦截，在调用前后都输出⼀行LOG，模仿ProxyFactoryBean的功能。FactoryBean是⼀个接口，当在IOC容器中的Bean实现了FactoryBean后，通过getBean(StringBeanName)获取到的Bean对象并不是FactoryBean的实现类对象，而是这个实现类中的getObject()方法返回的对象。要想获取FactoryBean的实现类，就要getBean(&BeanName)，在BeanName之前加上&。
在该接口中还定义了以下3个⽅法：

```java
// 返回由FactoryBean创建的Bean实例，如果isSingleton()返回true，则该实例会放到Spring容器中单实例缓存池中；
T getObject() throw Exception;
// 返回由FactoryBean创建的Bean实例的作⽤域是singleton还是prototype；
boolean isSingleton();
// 返回FactoryBean创建的Bean类型。当配置⽂件中<bean>的class属性配置的实现类是FactoryBean时，通过getBean()⽅法返回的不是FactoryBean本身，⽽是FactoryBean#getObject()⽅法所返回的对象，相当于FactoryBean#getObject()代理了getBean()⽅法。
Class<?> getObjectType();
```
**总结**
BeanFactory是个Factory，也就是IOC容器或对象工厂，FactoryBean是个Bean。在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。但对FactoryBean而言，这个Bean不是简单的Bean，而是⼀个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似

> [Spring中BeanFactory和FactoryBean有何区别？](https://mp.weixin.qq.com/s/r3rnVhU8vr58Cw__UWOVLA)
> [FactoryBean和它的兄弟SmartFactoryBean！](https://mp.weixin.qq.com/s/zVtedq-kwlhqeQB7GZz6Qw)


## Bean的调用

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
> [Spring中获取Bean的八种方式](https://mp.weixin.qq.com/s/BW3khRkQwjBsXw7yJhCyXQ)


## 相关文章

- [11张流程图帮你搞定Spring Bean生命周期](https://mp.weixin.qq.com/s/I8tsf7cFXkHX1pUp7SPByw)
- [面试官：说说Spring Bean的实例化过程？面试必问的！](https://mp.weixin.qq.com/s/5hAt9_KyyqHy7zzOjZ9LyQ)
- [你知道Spring lazy-init懒加载的原理吗？](https://mp.weixin.qq.com/s/_je69-0J72X5YMCrS-92MQ)
- [如何自己实现一个简单的Spring Bean容器](https://mp.weixin.qq.com/s/brlEwyKhwhSkljHLL1zmBA)
- [实力总结四类Bean注入Spring的方式](https://mp.weixin.qq.com/s/AuTnuxIQDPFbuslDz9ffVg)
- [最全的Spring依赖注入方式，你都会了吗？](https://mp.weixin.qq.com/s/TIDKofzCPz6qg2vj16JRMA)
- [关于Spring注入方式的几道面试题，你能答上么](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&amp;mid=2247494432&amp;idx=1&amp;sn=3acc7e7bf31c6d1f56ad830d6eb1ec41&amp;source=41#wechat_redirect)
- [最全的Spring依赖注入方式，你都会了吗？](https://mp.weixin.qq.com/s/u1DcCsRrrHYFOVykwW4Dcg)
- [Spring官方为什么建议构造器注入？](https://mp.weixin.qq.com/s/fVV6dYh0DQOoDiXwLR5miw)
- [Bean放入Spring容器，你知道几种方式？](https://mp.weixin.qq.com/s/g9iRu1slTMx0dwYJiy2m7w)
- [Spring注入Bean的7种方式，还有谁不会？？](https://mp.weixin.qq.com/s/i0Y-p7mda5FJCWCMJ8msdg)
- [Spring注解@Bean和@Component的区别,你知道吗？](https://mp.weixin.qq.com/s/6CwABJAePAT6hzTmfk7Jjg)
- [@Bean与@Component用在同一个类上，会怎么样？](https://mp.weixin.qq.com/s/lyH72PRAGcR2-aQvMZ1jPA)
- [Bean异步初始化，让你的应用启动飞起来](https://mp.weixin.qq.com/s/aZCgJS3Uaj28UiKTtUFcmw)
- [Spring中的父子容器是咋回事？](https://mp.weixin.qq.com/s/06Mmgnhhu98lQtQ8X13QBA)
- [Spring容器原始Bean是如何创建的？](https://mp.weixin.qq.com/s/jB9Vzt-uAj6njg2ADVFmyw)