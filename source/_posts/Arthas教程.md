---
title: Arthas教程
categories: 技术栈
img: https://pic1.zhimg.com/70/v2-25803fd579557b184ec9282d63167976_1440w.avis

---


## 常用命令

### [stack](https://arthas.aliyun.com/doc/stack.html)

输出当前方法被调用的调用路径。很多时候我们都知道一个方法被执行，但是有很多地方调用了它，你并不知道是谁调用了它，此时你需要的是stack命令。

```shell
stack com.baomidou.mybatisplus.extension.service.IService getOne
```
### [jad](https://arthas.aliyun.com/doc/jad.html)

反编译指定已加载类的源码。有时候，版本发布后，代码竟然没有执行，代码是最新的吗，这时可以使用jad反编译相应的class。

```shell
jad cn.test.mobile.controller.order.OrderController
```
仅编译指定的方法

```shell
jad cn.test.mobile.controller.order.OrderController getOrderInfo
```

### [sc](https://arthas.aliyun.com/doc/sc.html)

“Search-Class”的简写，查看JVM已加载的类信息有的时候，你只记得类的部分关键词，你可以用sc获取完整名称,当你碰到这个错的时候“ClassNotFoundException”或者“ClassDefNotFoundException”，你可以用这个命令验证下是否已加载。

```shell
sc *OrderController*
# 打印类的详细信息sc -d
sc -d cn.test.mobile.controller.order.OrderController
```

### [sm](https://arthas.aliyun.com/doc/sm.html)

sm(“Search-Method”)，查看已加载类的方法信息。

```shell
# 查看String里的方法
sm java.lang.String
# 查看String中toString的详细信息
sm -d java.lang.String toString
```

### [watch](https://arthas.aliyun.com/doc/watch.html)

可以监测一个方法的入参和返回值。使用方法：`watch 类名 方法名 "{params,returnObj,throwExp}" -e -b  -x 2`观察方法出参，返回值。
> -b在方法调用之前观察
> -e在方法异常之后观察
> -s在方法返回之后观察
> -f在方法结束之后(正常返回和异常返回)观察，默认选项
> -x指定输出结果的遍历深度,默认为1

```shell
# 观察getOrderInfo的出参和返回值，出参就是方法结束后的入参
watch cn.test.mobile.controller.order.OrderController getOrderInfo "{params,returnObj}" -x 2
# 观察getOrderInfo的入参和返回值
watch cn.test.mobile.controller.order.OrderController getOrderInfo "{params,returnObj}" -x 3 -b
```

### [trace](https://arthas.aliyun.com/doc/trace.html)

`类名 方法名 '#cost > 10'`查看某个类的某个方法执行多长时间(加后面参数只会展示耗时大于10ms的调用路径)。输出方法内部调用路径，和路径上每个节点的耗时，可以通过这个命令，查看哪些方法耗性能，从而找出导致性能缺陷的代码，这个耗时还包含了arthas执行的时间。

```shell
# 输出getOrderInfo的调用路径
trace -j cn.test.mobile.controller.order.OrderController getOrderInfo
# 输出getOrderInfo的调用路径，且cost大于10ms，-j是指过滤掉jdk中的方法，可以看到输出少了很多
trace -j cn.test.mobile.controller.order.OrderController getOrderInfo '#cost > 10'
```

### jobs

执行后台异步任务,线上有些问题是偶然发生的，这时就需要使用异步任务，把信息写入文件。使用&指定命令去后台运行，使用>将结果重写到日志文件，以trace为例

```shell
trace -j cn.test.mobile.controller.order.OrderController getOrderInfo > test.out &

# jobs 列出所有job
jobs
```
异步执行时间，默认为1天，如果要修改，使用options命令
```shell
# options可选参数1d,2h,3m,25s，分别代表天、小时、分、秒
options job-timeout 2d
```
kill强制终止任务
```shell
kill 76
```

### [logger](https://arthas.aliyun.com/doc/logger.html)

查看logger信息，更新logger level

```shell
# 查看
logger
# 更新日志级别
logger --name ROOT --level debug
# 如果执行这个命令时出错：update logger level fail.指定classLoaderHash重试一下试试
logger -c 18b4aac2 --name ROOT --level debug
```

### [dashboard](https://arthas.aliyun.com/doc/dashboard.html)

查看当前系统的实时数据面板,这个命令可以全局的查看jvm运行状态，比如内存和cpu占用情况

ID: Java级别的线程ID，注意这个ID不能跟jstack中的nativeID一一对应我们可以通过thread id查看线程的堆栈信息

NAME: 线程名

GROUP: 线程组名

PRIORITY: 线程优先级,1~10之间的数字，越大表示优先级越高

STATE: 线程的状态

CPU%: 线程消耗的cpu占比，采样100ms，将所有线程在这100ms内的cpu使用量求和，再算出每个线程的cpu使用占比。

TIME: 线程运行总时间，数据格式为分：秒

INTERRUPTED: 线程当前的中断位状态

DAEMON: 是否是daemon线程

### [redefine](https://arthas.aliyun.com/doc/redefine.html)

redefine jvm已加载的类，可以在不重启项目的情况下，热更新类。这个功能真的很强大，但是命令不一定会成功.下面我们来模拟：假设我想修改OrderController里的某几行代码，然后热更新至jvm：

1. 反编译OrderController，默认情况下，反编译结果里会带有ClassLoader信息，通过--source-only选项，可以只打印源代码。方便和mc/redefine命令结合使用
```shell
jad --source-only cn.test.mobile.controller.order.OrderController > OrderController.java
```
生成的OrderController.java在哪呢，执行pwd就知道在哪个目录了

2. 查找加载OrderController的ClassLoader
```shell
sc -d cn.test.mobile.controller.order.OrderController | grep classLoaderHash
# 18b4aac2
```

3. 修改保存好OrderController.java之后，使用mc(Memory Compiler)命令来编译成字节码，并且通过-c参数指定ClassLoader
```shell
mc -c 18b4aac2 OrderController.java -d ./
```

4. 热更新刚才修改后的代码
```shell
redefine -c 18b4aac2 OrderController.class
```
然后代码就更新成功了。

> 注意：
> redefine的class不能修改、添加、删除类的field和method，包括方法参数、方法名称及返回值
> redefine后的原来的类不能恢复，redefine有可能失败（比如增加了新的field），参考jdk本身的文档
> reset命令对redefine的类无效。如果想重置，需要redefine原始的字节码。
> redefine命令和jad/watch/trace/monitor/tt等命令会冲突。执行完redefine之后，如果再执行上面提到的命令，则会把redefine的字节码重置。原因是jdk本身redefine和Retransform是不同的机制，同时使用两种机制来更新字节码，只有最后修改的会生效。
> 正在跑的函数，没有退出不能生效
> 推荐使用`retransform`命令



### [retransform](https://arthas.aliyun.com/doc/retransform.html)

加载外部的.class文件，retransform jvm已加载的类。使用参考如下：

```sh
retransform /tmp/Test.class
retransform -l # 查看retransform entry
retransform -d 1                    # delete retransform entry
retransform --deleteAll             # delete all retransform entries
retransform --classPattern demo.*   # triger retransform classes
retransform -c 327a647b /tmp/Test.class /tmp/Test\$Inner.class
retransform --classLoaderClass 'sun.misc.Launcher$AppClassLoader' /tmp/Test.class
```

- retransform指定的.class文件

```sh
$ retransform /tmp/MathGame.class
retransform success, size: 1, classes:
demo.MathGame
```
加载指定的.class文件，然后解析出class name，再retransform jvm中已加载的对应的类。每加载一个.class文件，则会记录一个retransform entry.如果多次执行retransform加载同一个class文件，则会有多条retransform entry.

- 查看retransform entry

```shell
$ retransform -l
Id              ClassName       TransformCount  LoaderHash      LoaderClassName
1               demo.MathGame   1               null            null
```
ransformCount统计在ClassFileTransformer#transform函数里尝试返回entry对应的.class文件的次数，但并不表明transform一定成功

- 删除指定retransform entry需要指定id：

```sh
retransform -d 1
```

- 删除所有retransform entry

```sh
retransform --deleteAll
```

- 显式触发retransform

```sh
$ retransform --classPattern demo.MathGame
retransform success, size: 1, classes:
demo.MathGame
```

> 注意：对于同一个类，当存在多个retransform entry时，如果显式触发retransform，则最后添加的entry生效(id最大的)。

- 消除retransform的影响
  如果对某个类执行retransform之后，想消除影响，则需要：
  1. 删除这个类对应的retransform entry
  2. 重新触发retransform

  > 提示
  > 如果不清除掉所有的retransform entry，并重新触发retransform，则arthas stop时，retransform过的类仍然生效。

- 结合jad/mc命令使用

```sh
# jad命令反编译，然后可以用其它编译器，比如vim来修改源码
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java
# mc命令来内存编译修改过的代码
mc /tmp/UserController.java -d /tmp
# 用retransform命令加载新的字节码
retransform /tmp/com/example/demo/arthas/user/UserController.class
```

- 上传.class文件到服务器的技巧
  使用mc命令来编译jad的反编译的代码有可能失败。可以在本地修改代码，编译好后再上传到服务器上。有的服务器不允许直接上传文件，可以使用base64命令来绕过。

- retransform的限制
  1. 不允许新增加field/method
  2. 正在跑的函数，没有退出不能生效

### [thread](https://arthas.aliyun.com/doc/thread.html)

使用thread命令可以查看线程的状态，显示的结果实际就是dashboard结果的第一栏。thread命令可以追加参数
id：可以查看指定线程id的堆栈信息
-n value：找出最忙的value个线程，并打印堆栈信息
-b：找出当前正在阻塞其他线程的线程
-i value：指定采样cpu占比的时间间隔，默认为100ms。

### cls

清空当前屏幕内容

### 退出arthas
quit或exit退出当前连接，完全退出使用stop(所有客户端连接都会退出)

> [arthas官方文档](https://arthas.aliyun.com/doc/quick-start.html)

## java程序cpu飙高如何排查

### 方式一

1. 执行`top`命令，查看CPU占用情况，找到进程的pid
2. 使用`top -Hp <pid>`命令（pid为Java进程的id号）查看该Java进程内所有线程的资源占用情况,找出负载高的线程，记录tid(thread pid)
    > -H：这个选项会使top一行显示一个线程，而不是一行显示一个进程。
    > -p：这个选项后面通常会跟一个或多个进程ID（PID），表示只查看这些特定进程的信息。

3. `printf '%x\n' tid`命令（tid指线程的id号）将以上10进制的线程号转换为16进制nid
4. `jstack -l pid > ./jstack_result.log`采用jstack命令导出线程快照。
5. `cat jstack_result.log | grep -A 200 'nid=0x6a'`根据线程号定位具体代码
    > -A 200: 这个选项表示“after”，意思是显示匹配行后的200行

### 方式二

1. `top`命令找到占用CPU高的Java进程PID
2. 根据进程ID找到占用CPU高的线程`ps -T -p pid -o tid,cmd | sort -r`
3. 将指定的线程ID输出为16进制格式`printf “%x\n” tid`
4. 根据16进制格式的线程ID查找线程堆栈信息`jstack pid |grep tid -A 50`

> [CPU占用过高，死锁，内存泄漏处理方法](https://mp.weixin.qq.com/s/TrLDaaxGupXpCxiEsvEeRw)