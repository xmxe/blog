---
title: 线程-Thread
sticky: 90
categories: Java 
index_img: /assert/thread.jpg
img: http://qungz.photo.store.qq.com/qun-qungz/V51g1j7W00cGjy3O92fU33gLE919gaa6/V5bCQAyOTM0MDQ3NThEvERjyTHWAQ!!/800?w5=1346&h5=664&rf=viewer_421
coverImg: https://s1.ax1x.com/2022/10/11/xN1njs.jpg
cover: true
summary: 线程+线程池
---

### 线程

#### 创建线程三种方式

1. 继承Thread类，重写run()方法。然后直接new这个对象的实例，再调用start()方法启动线程。其实本质上Thread是实现了Runnable接口的一个实例：public class Thread implements Runnable
2. 实现Runnable接口，重写run()方法。然后调用new Thread（runnable）的方式创建一个线程，再调用start()方法启动线程。
3. 实现Callable接口，重写call()方法。Callable是类似于Runnable的接口，是属于Executor框架中的功能类。具有返回值，并且可以对异常进行声明和抛出
- [【图解】透彻Java线程状态转换](https://mp.weixin.qq.com/s/G-X82-Fp7zShTTnkWg1N5A)
- [Thread, Runable, Callable 还傻傻分不清？](https://mp.weixin.qq.com/s/cNJQNzoqFpvgeDEbcWmI6A)

#### 线程相关方法

##### yield()

它让掉当前线程 CPU 的时间片，使正在运行中的线程重新变成就绪状态，并重新竞争 CPU 的调度权。它可能会获取到，也有可能被其他线程获取到。使当前线程从执行状态（运行状态）变为可执行态（就绪状态）。cpu会从众多的可执行态里选择。 也就是说，当前也就是刚刚的那个线程还是有可能会被再次执行到的

##### join()

并行变串行，当前线程等待另一个调用join()方法的线程执行结束后再往下执行, 哪个线程调用join()哪个线程优先执行（前提必须调用start()方法启动线程）

##### setDaemon()

设置是否为守护线程，线程分为用户线程和守护线程，当用户线程都退出时，无论当jvm里面的守护线程有没有执行完，jvm都会退出，使用setDaemon()必须在thread.start()之前，否则会抛出异常。守护线程服务于用户线程,当用户线程结束后守护线程也会结束,当所有线程都运行结束时，JVM退出，进程结束。
例如有一种线程的目的就是无限循环
```java
class TimerThread extends Thread { 
  @Override
  public void run() {
	while (true) {
		System.out.println(LocalTime.now());
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			break
		}
	}
  }
}
```
如果这个线程不结束，JVM进程就无法结束。问题是，由谁负责结束这个线程？ 然而这类线程经常没有负责人来负责结束它们。但是，当其他线程结束时，JVM进程又必须要结束，怎么办？ 答案是将这个线程设置成守护线程（Daemon Thread）。守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。因此，JVM退出时，不必关心守护线程是否已结束。在守护线程中，编写代码要注意：守护线程不能持有任何需要关闭的资源，例如打开文件等，因为虚拟机退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失

##### Thread.interrupted()

检测当前线程是否被中断，并且中断状态会被清除（即重置为false）；它是静态方法，即使是线程对象去调用，底层使用的也是判断当前线程的中断状态，而不是被调用线程的中断状态。如果连续两次调用该方法，则第二次调用将返回 false（在第一次调用已清除了其中断状态之后，且第二次调用检验完中断状态前，当前线程再次中断的情况除外）。

- this.isInterrupted()
检测调用该方法的线程是否被中断，中断状态不会被清除。线程一旦被中断，该方法返回true，而一旦sleep等方法抛出异常，它将清除中断状态，此时方法将返回false。

- this.interrupt()

中断调用该方法的线程,中断被阻塞的线程，会抛出一个InterruptedException，把线程从阻塞状态中解救出来，会清除中断标志位

如果当前线程没有中断它自己（这在任何情况下都是允许的），则该线程的 checkAccess 方法就会被调用，这可能抛出 SecurityException。如果线程在调用 Object 类的 wait()、wait(long) 或 wait(long, int) 方法，或者该类的 join()、join(long)、join(long, int)、sleep(long) 或 sleep(long, int) 方法过程中受阻，则其中断状态将被清除，它还将收到一个InterruptedException。如果该线程在可中断的通道上的 I/O 操作中受阻，则该通道将被关闭，该线程的中断状态将被设置并且该线程将收到ClosedByInterruptException。如果该线程在一个 Selector 中受阻，则该线程的中断状态将被设置，它将立即从选择操作返回，并可能带有一个非零值，就好像调用了选择器的 wakeup 方法一样。如果以前的条件都没有保存，则该线程的中断状态将被设置。

中断一个不处于活动状态的线程不需要任何作用。
[如何停止一个正在运行的线程？](https://mp.weixin.qq.com/s/J8Acb1FBPhqb1Z7Vur0erQ)

##### 捕获异常

- Thread.setDefaultUncaughtExceptionHandler()
相当于一个全局的捕获异常。用于记录当程序发生你未捕获的异常的时候,调用一个你默认的handler来进行某些操作

- Thread.getDefaultUncaughtExceptionHandler()
返回当线程由于未捕获的异常而突然终止时调用的默认处理程序。 如果返回的值为null，则没有默认值

- setUncaughtExceptionHandler
用来获取线程中产生的异常,建议使用该方法为线程设置异常捕获方法，主线程无法捕获子线程异常，当子线程异常时，可以使用这个方法处理异常

- getUncaughtExceptionHandler
返回该线程由于未捕获的异常而突然终止时调用的处理程序。

##### 线程的优先级

- Thread.MAX_PRIORITY：10
- Thread.MIN _PRIORITY：1
- Thread.NORM_PRIORITY：5 -->默认优先级 
- getPriority():获取线程的优先级 
- setPriority(int p):设置线程的优先级

说明：⾼优先级的线程要抢占低优先级线程cpu的执⾏权。但是只是从概率上讲，⾼优先级的线程⾼概率的情况下被执⾏。并不意味着只当⾼优先级的线程执⾏完以后，低优先级的线程才执行

##### checkAccess

确定当前运行的线程是否具有修改此线程的权限

##### countStackFrames

计算此线程中的堆栈帧数，当前线程必须被挂起

##### getThreadGroup()

获取线程所在的线程组

##### Thread.activeCount()

返回当前线程的线程组中活动线程的数量。返回的值只是一个估计值，因为当此方法遍历内部数据结构时，线程数可能会动态更改

##### Thread.dumpStack()

打印当前线程的堆栈跟踪到标准错误流。此方法仅用于调试。

##### Thread.enumerate(****Thread[] tarray****)

用于将每个活动线程的线程组及其子组复制到指定的数组中。 此方法使用tarray参数调用enumerate方法。此方法使用activeCount方法来估计数组应该有多大。 如果数组的长度太短而无法容纳所有线程，则会以静默方式忽略额外的线程。tarray ：此方法是要复制到的Thread对象数组。返回此方法返回放入数组的线程数。

##### Thread.getAllStackTraces()

返回所有活动线程的堆栈跟踪的一个映射

##### Thread.holdsLock()

当且仅当当前线程在指定的对象上保持监视器锁方法返回true

[多线程基础知识、线程相关方法](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493938&idx=1&sn=125990919a15c7dd3c4ed4c36451d34b&source=41#wechat_redirect)


#### 线程同步

1. synchronized
2. 使用特殊域变量(volatile)实现线程同步
3. 使用重入锁实现线程同步（ReentrantLock）
4. ThreadLocal与同步机制


#### 8种保证线程安全的技术

1. 无状态
我们都知道只有多个线程访问公共资源的时候，才可能出现数据安全问题，那么如果我们没有公共资源，是不是就没有这个问题呢？
2. 不可变 （final）
如果多个线程访问公共资源是不可变的，也不会出现数据的安全性问题
3. 安全的发布 （private）
如果类中有公共资源，但是没有对外开放访问权限，即对外安全发布，也没有线程安全问题
4. volatile
如果有些公共资源只是一个开关，只要求可见性，不要求原子性，这样可以用volidate关键字定义来解决问题。
5. synchronized
使用JDK内部提供的同步机制，这也是使用比较多的手段，分为：方法同步 和 代码块同步，我们优先使用代码块同步，因为方法同步的范围更大，更消耗性能。每个对象内部都又一把锁，只有抢答那把锁的线程，才能进入代码块里，代码块执行完之后，会自动释放锁
6. lock
除了使用synchronized关键字实现同步功能之外，JDK还提供了lock显示锁的方式。它包含：可重入锁、读写锁 等更多更强大的功能，有个小问题就是需要手动释放锁，不过在编码时提供了更多的灵活性
7. cas
JDK除了使用锁的机制解决多线程情况下数据安全问题之外，还提供了cas机制。这种机制是使用CPU中比较和交换指令的原子性，JDK里面是通过Unsafe类实现的。cas需要四个值：旧数据、期望数据、新数据 和 地址，比较旧数据 和 期望的数据如果一样的话，就把旧数据改成新数据，当前线程不断自旋，一直到成功为止。不过可能会出现aba问题，需要使用AtomicStampedReference增加版本号解决。其实，实际工作中很少直接使用Unsafe类的，一般用atomic包下面的类即可。
8. threadlocal
除了上面几种解决思路之外，JDK还提供了另外一种用空间换时间的新思路：threadlocal。它的核心思想是：共享变量在每个线程都有一个副本，每个线程操作的都是自己的副本，对另外的线程没有影响。特别注意，使用threadlocal时，使用完之后，要记得调用remove方法，不然可能会出现内存泄露问题
[聊聊保证线程安全的 10 个小技巧](https://mp.weixin.qq.com/s/5q130nkGk23CPlqbRK01OA)
[4种方法，实现多线程按着指定顺序执行](https://mp.weixin.qq.com/s/HZBWUijsI8CA1KKUhHuXKw)


#### 线程通信（例如：A线程操作到某一步通知B线程）

1. thread.join(),
2. object.wait(), object.notify()
3. CountdownLatch
4. 使用 volatile 关键字
5. 使用 ReentrantLock 结合 Condition
6. LockSupport 是一种非常灵活的实现线程间阻塞和唤醒的工具，使用它不用关注是等待线程先进行还是唤醒线程先运行，但是得知道线程的名字

[Java 如何线程间通信，面试被问哭](https://mp.weixin.qq.com/s/NUJL_mEfXSo0e-nf2UUNJQ)
[线程通信的5中方式](https://mp.weixin.qq.com/s/47UlDrzbH9cKeQ1g3DHQeQ)

#### 最佳线程数

QPS=每秒钟request数量
TPS=每秒钟事务数量
RT=一般取平均响应时间
QPS=并发数/RT 或者 并发数=QPS * RT
最佳线程数=RT/CPU Time * CPU核心数 * CPU利用率
最大QPS=最佳线程数 * 单线程QPS=（RT/CPU Time * CPU核心数 * CPU利用率）\*（1/RT) = CPU核心数 * CPU利用率/CPU time

最佳线程经验值：
IO密集型配置线程数经验值是：2N，其中N代表CPU核数。
CPU密集型配置线程数经验值是：N + 1，其中N代表CPU核数。
如果获取N的值？
```java
int availableProcessors = Runtime.getRuntime().availableProcessors()
```
最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 * CPU数目
数据库连接池连接数 = ((核心数 * 2) + 有效磁盘数)
[如何设置线程池参数？美团给出了一个让面试官虎躯一震的回答。](https://mp.weixin.qq.com/s/D73p4StaBA1sXEoZQZrEaQ)
[面试官问：高并发下，你都怎么选择最优的线程数？](https://mp.weixin.qq.com/s/isjUUOSK64XMiveRO_-3rA)
[线程池最佳线程数量到底要如何配置？](https://mp.weixin.qq.com/s/MPsUd0sy0yojcUoSYl8B2A)


#### 线程顺序执行

1. 使用线程的join方法
2. 使用主线程的join方法
3. 使用线程的wait方法
4. 使用线程的线程池方法
5. 使用线程的Condition(条件变量)方法
6. 使用线程的CountDownLatch(倒计数)方法
7. 使用线程的CyclicBarrier(回环栅栏)方法
8. 使用线程的Semaphore(信号量)方法
[线程顺序执行的 8 种方法](https://mp.weixin.qq.com/s/NotprQ-F-ivnmVusELkDhg)


#### 线程相关文章
- [Java 多线程与并发高频面试题解析](https://mp.weixin.qq.com/s/DIbxSun2NcD-wbqKGQpRLg)
- [超赞，大牛总结的多线程的问题及答案](https://mp.weixin.qq.com/s/0KmWOLNqhck85WECC9uQ-g)
- [99 道 Java 多线程面试题，看完我跪了！](https://mp.weixin.qq.com/s/dRqLZG7eev87hda9ohlJrA)
- [2 万字长文详解 10 大多线程面试题](https://mp.weixin.qq.com/s/hq5GbYBe98YsBDNA3u2s5Q)
- [两万字！多线程硬核50问！](https://mp.weixin.qq.com/s/wGJsOWAGUhlE4QlZsNpMXg)
- [进程间的通信(操作系统间)](https://mp.weixin.qq.com/s/tDfMAshpUr2N6absPR7Dug)
- [106道Java并发和多线程基础面试题大集合（2w字）](https://mp.weixin.qq.com/s/3istnkJ3MMYQnvbtTsdxmA)

### 线程池

#### 介绍

```java
ExecutorService threadPool = Executors.newCachedThreadPool();

// newCachedThreadPool: 创建一个可缓存线程池，可以无限扩大，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。适用于服务器负载较轻，执行很多短期异步任务。

// newFixedThreadPool: 创建一个定长、固定大小的线程池，可控制线程最大并发数，超出的线程会在队列中等待，表示同一时刻只能有这么大的并发数，实际线程数量永远不会变化，适用于可以预测线程数量的业务中，或者服务器负载较重，对当前线程数量进行限制。

// newScheduledThreadPool: 创建一个定长线程池，支持定时及周期性任务执行。可以延时启动，定时启动，适用于需要多个后台线程执行周期任务的场景。

// newSingleThreadExecutor: 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。是一个单线程的线程池，适用于需要保证顺序执行各个任务，并且在任意时间点，不会有多个线程是活动的场景

// newWorkStealingPool: 创建一个拥有多个任务队列的线程池，可以减少连接数，创建当前可用cpu数量的线程来并行执行，适用于大耗时的操作，可以并行来执行

// newSingleThreadScheduledExecutor: 只有一个线程，该线程池可用于定时或周期性任务的执行，类似于Timer，但比Timer要更安全
```
[为什么阿里巴巴要禁用Executors创建线程池？](https://mp.weixin.qq.com/s/EheN1I84uo1zk6ptSqsqcQ)

```java
ExecutorService threadPool = new ThreadPoolExecutor(
int corePoolSize, 
int maximumPoolSize, 
long keepAliveTime,
TimeUnit unit, 
BlockingQueue<Runnable> workQueue,
ThreadFactory threadFactory,
RejectedExecutionHandler handler)

// corePoolSize：线程池的核心线程数(最小线程数)，不管它们创建以后是不是空闲的。线程池需要保持corePoolSize数量的线程，除非设置了allowCoreThreadTimeOut

// maximumPoolSize：线程池的最大线程数；

// keepAliveTime：线程池空闲时线程的存活时长；如果经过keepAliveTime时间后，超过核心线程数的线程还没有接受到新的任务，那就销毁，超出线程池核心线程数 小于线程池最大线程数的线程都是借的，没有用了,超时就销毁

// unit：keepAliveTime时长单位；

// workQueue：当提交的任务数超过核心线程数大小后，再提交的任务就存放在这里。它仅仅用来存放被execute方法提交的Runnable任务。存放任务的队列，上面提到的线程数超过corePoolSize存放任务的地方；
// new ArrayBlockingQueue<Runnable>(10)：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
// new LinkedBlockingQueue<Runnable>(10)：一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，也可以不传参数，默认是Integer.MAX_VALUE
// new SynchronousQueue<Runnable>()：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，使用SynchronousQueue阻塞队列一般要求maximumPoolSizes为无界(Integer.MAX_VALUE)，避免线程拒绝执行操作。
// PriorityBlockingQueue：一个具有优先级的无限阻塞队列。
// DelayQueue: DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。

// threadFactory：线程工厂，可以自己重写一下，为每个线程赋予一个名字，便于排查问题
class MyThreadFactory implements ThreadFactory{
	@Override
	public Thread newThread(Runnable r){
		return new Thread(r,"thread_name");
	}  
}

// handler：当队列里面放满了任务、最大线程数的线程都在工作时，这时继续提交的任务线程池就处理不了，应该执行怎么样的拒绝策略。
//在队列（workQueue）和线程池达到最大线程数（maximumPoolSize）均满时仍有任务的情况下的处理方式即当任务数大于最大线程数并且队列已满时，采用的拒绝策略，分4种，
new ThreadPoolExecutor.AbortPolicy // 丢弃任务并抛出RejectedExecutionException异常
// AbortPolicy策略：默认策略，如果线程池队列满了丢掉这个任务并且抛出RejectedExecutionException异常。
new ThreadPoolExecutor.DiscardPolicy //丢弃任务，但是不抛出异常
// DiscardPolicy策略：如果线程池队列满了，会直接丢掉这个任务并且不会有任何异常。
new ThreadPoolExecutor.CallerRunsPolicy//（调用者运行）:如果线程池的线程数量达到上限，该策略会把任务队列中的任务放在调用者线程当中运行 由调用线程处理该任务
// CallerRunsPolicy策略：如果添加到线程池失败，那么主线程会自己去执行该任务，不会等待线程池中的线程去执行。
new ThreadPoolExecutor.DiscardOldestPolicy // 抛弃最旧的 丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
// DiscardOldestPolicy策略：如果队列满了，会将最早进入队列的任务删掉腾出空间，再尝试加入队列。

// demo：

ThreadPoolExecutor exec = new ThreadPoolExecutor(
10, 30, 300, TimeUnit.SECONDS, 
new ArrayBlockingQueue<Runnable>(10000), 
Executors.defaultThreadFactory(), 
new ThreadPoolExecutor.CallerRunsPolicy());

// 核心线程数10，最大线程数30，keepAliveTime是3秒,随着任务数量不断上升，线程池会不断的创建线程，直到到达核心线程数10，就不创建线程了， 这时多余的任务通过加入阻塞队列来运行， 当超出阻塞队列长度+核心线程数时， 这时不得不扩大线程个数来满足当前任务的运行，这时就需要创建新的线程了（最大线程数起作用），上限是最大线程数30 那么超出核心线程数10并小于最大线程数30的可能新创建的这20个线程相当于是“借”的，如果这20个线程空闲时间超过keepAliveTime，就会被退出

```
**ArrayBlockingQueue、LinkedBlockingQueue区别**
![](/images/ArrayBlockingQueueLinkedBlockingQueue区别.png)


#### submit与execute区别

1. submit在执行过程中与execute不一样，submit不会抛出异常而是把异常保存在成员变量中，在FutureTask.get阻塞获取的时候再把异常抛出来。
2. submit有返回值Future，execute无返回值
3. execute会抛出异常，sumbit方法不会抛出异常。除非你调用Future.get()。execute直接抛出异常之后线程就死掉了，submit保存异常线程没有死掉，因此execute的线程池可能会出现没有意义的情况，因为线程没有得到重用。而submit不会出现这种情况。
```java
// ①execute方法
threadPool.execute(new Runnable() {
  public void run() {
   try {
		System.out.println(index);Thread.sleep(2000);
   } catch (InterruptedException e) {
		e.printStackTrace();
   }
  }
});

// ②submit方法
Future<String> f = threadPool.submit(new Callable<String>(){
    @Override
    public String call(){
		return "e";
    }
}); 

String str = f.get();
FutureTask<String> futureTask = new FutureTask<String>(new Callable(){
    @Override
    public String call(){
    	return "e";
    }
});

new Thread(futureTask).start();// executor.submit(futureTask);
String result = futureTask.get(2000, TimeUnit.MILLISECONDS)// 如果在指定时间内，还没获取到结果，就直接返回null
```

#### 线程池相关文章

- [面试官：Java并发-Executor给我讲讲？](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494021&idx=2&sn=af90e629487ffb76d795fb1f2ff3ccfb&source=41#wechat_redirect)
- [10问10答：你真的了解线程池吗？](https://mp.weixin.qq.com/s/oKCJmQt9egsy5RCMgf9xpw)
- [如何优雅的自定义 ThreadPoolExecutor 线程池？](https://mp.weixin.qq.com/s/01MnCUVRxWoqUrCC_xmtEw)
- [如何优雅的使用线程池！！！](https://mp.weixin.qq.com/s/UScLWJbiMhWig5doo-coxA)
- [深入线程池的问题连环炮](https://mp.weixin.qq.com/s/N76EM8SEjZzZHdheS4jF2Q)
