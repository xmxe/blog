---
title: Java反射
categories: Java
tags: 代码实战
img: https://pica.zhimg.com/v2-710586d41488c451dd9dd70bc33eb121_1440w.jpg

---


## 何为反射？

如果说大家研究过框架的底层原理或者咱们自己写过框架的话，一定对反射这个概念不陌生。

反射之所以被称为框架的灵魂，主要是因为它赋予了我们在运行时分析类以及执行类中方法的能力。

通过反射你可以获取任意一个类的所有属性和方法，你还可以调用这些方法和属性。

## 反射的应用场景了解么？

像咱们平时大部分时候都是在写业务代码，很少会接触到直接使用反射机制的场景。

但是，这并不代表反射没有用。相反，正是因为反射，你才能这么轻松地使用各种框架。像Spring/SpringBoot、MyBatis等等框架中都大量使用了反射机制。

**这些框架中也大量使用了动态代理，而动态代理的实现也依赖反射。**

比如下面是通过JDK实现动态代理的示例代码，其中就使用了反射类`Method`来调用指定的方法。


```java
public class DebugInvocationHandler implements InvocationHandler {
    /**
     * 代理类中的真实对象
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }


    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("after method " + method.getName());
        return result;
    }
}
```

另外，像Java中的一大利器**注解**的实现也用到了反射。

为什么你使用Spring的时候，一个@Component注解就声明了一个类为Spring Bean呢？为什么你通过一个@Value注解就读取到配置文件中的值呢？究竟是怎么起作用的呢？

这些都是因为你可以基于反射分析类，然后获取到类/属性/方法/方法的参数上的注解。你获取到注解之后，就可以做进一步的处理。

## 谈谈反射机制的优缺点

**优点**：可以让咱们的代码更加灵活、为各种框架提供开箱即用的功能提供了便利

**缺点**：让我们在运行时有了分析操作类的能力，这同样也增加了安全问题。比如可以无视泛型参数的安全检查（泛型参数的安全检查发生在编译时）。另外，反射的性能也要稍差点，不过，对于框架来说实际是影响不大的。相关阅读：[Java Reflection:Why is it so slow?](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow)

## 反射实战

### 获取Class对象的四种方式

如果我们动态获取到这些信息，我们需要依靠Class对象。Class类对象将一个类的方法、变量等信息告诉运行的程序。Java提供了四种方式获取Class对象:

**1. 知道具体类的情况下可以使用：**

```java
Class alunbarClass = TargetObject.class;
```

但是我们一般是不知道具体类的，基本都是通过遍历包下面的类来获取Class对象，通过此方式获取Class对象不会进行初始化

**2. 通过Class.forName()传入类的全路径获取：**

```java
Class alunbarClass1 = Class.forName("cn.javaguide.TargetObject");
```

**3. 通过对象实例instance.getClass()获取：**

```java
TargetObject o = new TargetObject();
Class alunbarClass2 = o.getClass();
```

**4. 通过类加载器xxxClassLoader.loadClass()传入类路径获取:**

```java
ClassLoader.getSystemClassLoader().loadClass("cn.javaguide.TargetObject");
```

通过类加载器获取Class对象不会进行初始化，意味着不进行包括初始化等一系列步骤，静态代码块和静态对象不会得到执行

### 反射的一些基本操作

1. 创建一个我们要使用反射操作的类TargetObject。

```java
package cn.javaguide;

public class TargetObject {
    private String value;

    public TargetObject() {
        value = "JavaGuide";
    }

    public void publicMethod(String s) {
        System.out.println("I love " + s);
    }

    private void privateMethod() {
        System.out.println("value is " + value);
    }
}
```

2. 使用反射操作这个类的方法以及参数

```java
package cn.javaguide;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException, NoSuchFieldException {
        /**
         * 获取TargetObject类的Class对象并且创建TargetObject类实例
         */
        Class<?> targetClass = Class.forName("cn.javaguide.TargetObject");
        TargetObject targetObject = (TargetObject) targetClass.newInstance();
        /**
         * 获取TargetObject类中定义的所有方法
         */
        Method[] methods = targetClass.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method.getName());
        }

        /**
         * 获取指定方法并调用
         */
        Method publicMethod = targetClass.getDeclaredMethod("publicMethod",
                String.class);

        publicMethod.invoke(targetObject, "JavaGuide");

        /**
         * 获取指定参数并对参数进行修改
         */
        Field field = targetClass.getDeclaredField("value");
        //为了对类中的参数进行修改我们取消安全检查
        field.setAccessible(true);
        field.set(targetObject, "JavaGuide");

        /**
         * 调用private方法
         */
        Method privateMethod = targetClass.getDeclaredMethod("privateMethod");
        //为了调用private方法我们取消安全检查
        privateMethod.setAccessible(true);
        privateMethod.invoke(targetObject);
    }
}
```

输出内容：

```text
publicMethod
privateMethod
I love JavaGuide
value is JavaGuide
```

**注意**:有读者提到上面代码运行会抛出ClassNotFoundException异常,具体原因是你没有下面把这段代码的包名替换成自己创建的TargetObject所在的包。

```java
Class<?> targetClass = Class.forName("cn.javaguide.TargetObject");
```
> [原文链接](https://javaguide.cn/java/basis/reflection.html)

### 其他

```java
public class ReflectTest {
    public static void main(String[] args) throws Exception{
        Class<?> clazz = Class.forName("com.xmxe.study_demo.entity.Student");

        // 获取所有pubic修饰的成员变量
        Field[] fields = clazz.getFields();
        for (Field field : fields) {
            System.out.println("获取所有pubic修饰的成员变量==="+field);
        }
        // 获取指定的pulic成员变量
        Field field = clazz.getField("name");
        System.out.println("获取指定的pulic成员变量name===" + field);
        // 获取所有成员变量
        Field[] fields2 =  clazz.getDeclaredFields();
        for (Field field1 : fields2) {
            System.out.println("获取所有成员变量==="+field1);
        }
        // 获取指定成员变量，不考虑修饰符
        Field field3 = clazz.getDeclaredField("age");
        System.out.println("获取指定成员变量，不考虑修饰符==="+field3);

        // 获取所有public 修饰的构造方法，返回一个含有所有public修饰的构造函数对象的数组。
        Constructor<?>[] constructors = clazz.getConstructors();
        for(Constructor<?> constructor : constructors){
            System.out.println("获取所有public 修饰的构造方法==="+constructor);
        }
        // 获取所有构造函数，不考虑修饰符，参数是构造器中参数类型对应的Class对象。
        Constructor<?> con =  clazz.getDeclaredConstructor(String.class, Integer.class);
        System.out.println("获取所有构造函数，不考虑修饰符，参数是构造器中参数类型对应的Class对象==="+con);

        // 获取所有方法，不考虑修饰符
        Method[] methods = clazz.getDeclaredMethods();
        for(Method method : methods){
            System.out.println("获取所有方法，不考虑修饰符==="+method);
        }
        // 根据方法名获取
        Method method = clazz.getMethod("method1", String.class,Integer.class);
        System.out.println("根据方法名获取==="+method);

        // 获取类的全路径名
        String getName = clazz.getName();
        System.out.println("获取类的全路径名==="+getName);

        // 获取方法注解
        Annotation[] annotations = method.getAnnotations();
        for(Annotation annotation : annotations){
            System.out.println("获取方法注解==="+annotation);
        }

        OrderHandlerTypeAnnotation orderHandlerType = method.getAnnotation(OrderHandlerTypeAnnotation.class);
        System.out.println(orderHandlerType.source()+"---"+orderHandlerType.annotationType());
    }

}
```

## 相关文章

[Java反射机制你还不会？那怎么看Spring源码？](https://mp.weixin.qq.com/s/jV9kE2ajB40f3fOU_lT9ng)
[Java反射是什么？看这篇绝对会了！](https://mp.weixin.qq.com/s/QbacsQwTyvBJi12LYPNKJw)
[学会这篇反射，我就可以去吹牛逼了。](https://mp.weixin.qq.com/s/Dyg4qSqiyjSJTne8yvUYpQ)
[深入理解Java：类加载机制及反射](https://mp.weixin.qq.com/s/kTYLjg_FlKBdAAQQvSAF9g)
