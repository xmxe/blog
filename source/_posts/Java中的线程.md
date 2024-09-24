---
title: Java中的线程
categories: Java
index_img: /assert/thread.jpg
img: https://pic1.zhimg.com/06a1751875c26f6409149f9380a7899c_r.jpg
coverImg: https://pica.zhimg.com/v2-72839d9aecf00f3185303c7f76c597ee_r.jpg
cover: true
summary: 线程+线程池+虚拟线程
top: true

---

### 线程

#### 创建线程三种方式

1. 继承Thread类，重写run()方法。然后直接new这个对象的实例，再调用start()方法启动线程。其实本质上Thread是实现了Runnable接口的一个实例：`public class Thread implements Runnable`
2. 实现Runnable接口，重写run()方法。然后调用new Thread（runnable）的方式创建一个线程，再调用start()方法启动线程。
3. 实现Callable接口，重写call()方法。Callable是类似于Runnable的接口，是属于Executor框架中的功能类。具有返回值，并且可以对异常进行声明和抛出

> [【图解】透彻Java线程状态转换](https://mp.weixin.qq.com/s/G-X82-Fp7zShTTnkWg1N5A)

#### 线程相关方法

##### yield()

它让掉当前线程CPU的时间片，使正在运行中的线程重新变成就绪状态，并重新竞争CPU的调度权。它可能会获取到，也有可能被其他线程获取到。使当前线程从执行状态（运行状态）变为可执行态（就绪状态）。cpu会从众多的可执行态里选择。也就是说，当前也就是刚刚的那个线程还是有可能会被再次执行到的

##### join()

并行变串行，当前线程等待另一个调用join()方法的线程执行结束后再往下执行,哪个线程调用join()哪个线程优先执行（前提必须调用start()方法启动线程）

##### setDaemon()

设置是否为守护线程，线程分为用户线程和守护线程，当用户线程都退出时，无论当jvm里面的守护线程有没有执行完，jvm都会退出，使用setDaemon()必须在thread.start()之前，否则会抛出异常。守护线程服务于用户线程,当用户线程结束后守护线程也会结束,当所有线程都运行结束时，JVM退出，进程结束。例如有一种线程的目的就是无限循环

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
如果这个线程不结束，JVM进程就无法结束。问题是，由谁负责结束这个线程？然而这类线程经常没有负责人来负责结束它们。但是，当其他线程结束时，JVM进程又必须要结束，怎么办？答案是将这个线程设置成守护线程（Daemon Thread）。守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。因此，JVM退出时，不必关心守护线程是否已结束。在守护线程中，编写代码要注意：守护线程不能持有任何需要关闭的资源，例如打开文件等，因为虚拟机退出时，守护线程没有任何机会来关闭文件，这会导致数据丢失

##### Thread.interrupted()

检测当前线程是否被中断，并且中断状态会被清除（即重置为false）；它是静态方法，即使是线程对象去调用，底层使用的也是判断当前线程的中断状态，而不是被调用线程的中断状态。如果连续两次调用该方法，则第二次调用将返回false（在第一次调用已清除了其中断状态之后，且第二次调用检验完中断状态前，当前线程再次中断的情况除外）。

- this.isInterrupted()
检测调用该方法的线程是否被中断，中断状态不会被清除。线程一旦被中断，该方法返回true，而一旦sleep等方法抛出异常，它将清除中断状态，此时方法将返回false。

- this.interrupt()

中断调用该方法的线程,中断被阻塞的线程，会抛出一个InterruptedException，把线程从阻塞状态中解救出来，会清除中断标志位。如果当前线程没有中断它自己（这在任何情况下都是允许的），则该线程的checkAccess方法就会被调用，这可能抛出Security Exception。如果线程在调用Object类的wait()、wait(long)或wait(long,int)方法，或者该类的join()、join(long)、join(long,int)、sleep(long)或sleep(long,int)方法过程中受阻，则其中断状态将被清除，它还将收到一个Interrupted Exception。如果该线程在可中断的通道上的I/O操作中受阻，则该通道将被关闭，该线程的中断状态将被设置并且该线程将收到ClosedByInterrupt Exception。如果该线程在一个Selector中受阻，则该线程的中断状态将被设置，它将立即从选择操作返回，并可能带有一个非零值，就好像调用了选择器的wakeup方法一样。如果以前的条件都没有保存，则该线程的中断状态将被设置。中断一个不处于活动状态的线程不需要任何作用。

> [如何停止一个正在运行的线程？](https://mp.weixin.qq.com/s/J8Acb1FBPhqb1Z7Vur0erQ)

##### 捕获异常

- Thread.setDefaultUncaughtExceptionHandler()
相当于一个全局的捕获异常。用于记录当程序发生你未捕获的异常的时候,调用一个你默认的handler来进行某些操作

- Thread.getDefaultUncaughtExceptionHandler()
返回当线程由于未捕获的异常而突然终止时调用的默认处理程序。如果返回的值为null，则没有默认值

- setUncaughtExceptionHandler
用来获取线程中产生的异常,建议使用该方法为线程设置异常捕获方法，主线程无法捕获子线程异常，当子线程异常时，可以使用这个方法处理异常

- getUncaughtExceptionHandler
返回该线程由于未捕获的异常而突然终止时调用的处理程序。

##### 线程的优先级

- Thread.MAX_PRIORITY：10
- Thread.MIN_PRIORITY：1
- Thread.NORM_PRIORITY：5-->默认优先级
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

##### Thread.enumerate(Thread[] tarray)

用于将每个活动线程的线程组及其子组复制到指定的数组中。此方法使用tarray参数调用enumerate方法。此方法使用activeCount方法来估计数组应该有多大。如果数组的长度太短而无法容纳所有线程，则会以静默方式忽略额外的线程。tarray：此方法是要复制到的Thread对象数组。返回此方法返回放入数组的线程数。

##### Thread.getAllStackTraces()

返回所有活动线程的堆栈跟踪的一个映射

##### Thread.holdsLock()

当且仅当当前线程在指定的对象上保持监视器锁方法返回true

> [多线程基础知识、线程相关方法](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493938&idx=1&sn=125990919a15c7dd3c4ed4c36451d34b&source=41#wechat_redirect)


#### 线程同步

1. synchronized
2. 使用特殊域变量(volatile)实现线程同步
3. 使用重入锁实现线程同步（ReentrantLock）
4. ThreadLocal与同步机制


#### 8种保证线程安全的技术

1. 无状态
我们都知道只有多个线程访问公共资源的时候，才可能出现数据安全问题，那么如果我们没有公共资源，是不是就没有这个问题呢？
2. 不可变（final）
如果多个线程访问公共资源是不可变的，也不会出现数据的安全性问题
3. 安全的发布（private）
如果类中有公共资源，但是没有对外开放访问权限，即对外安全发布，也没有线程安全问题
4. volatile
如果有些公共资源只是一个开关，只要求可见性，不要求原子性，这样可以用volidate关键字定义来解决问题。
5. synchronized
使用JDK内部提供的同步机制，这也是使用比较多的手段，分为：方法同步和代码块同步，我们优先使用代码块同步，因为方法同步的范围更大，更消耗性能。每个对象内部都又一把锁，只有抢答那把锁的线程，才能进入代码块里，代码块执行完之后，会自动释放锁
6. lock
除了使用synchronized关键字实现同步功能之外，JDK还提供了lock显示锁的方式。它包含：可重入锁、读写锁等更多更强大的功能，有个小问题就是需要手动释放锁，不过在编码时提供了更多的灵活性
7. cas
JDK除了使用锁的机制解决多线程情况下数据安全问题之外，还提供了cas机制。这种机制是使用CPU中比较和交换指令的原子性，JDK里面是通过Unsafe类实现的。cas需要四个值：旧数据、期望数据、新数据和地址，比较旧数据和期望的数据如果一样的话，就把旧数据改成新数据，当前线程不断自旋，一直到成功为止。不过可能会出现aba问题，需要使用AtomicStampedReference增加版本号解决。其实，实际工作中很少直接使用Unsafe类的，一般用atomic包下面的类即可。
8. threadlocal
除了上面几种解决思路之外，JDK还提供了另外一种用空间换时间的新思路：threadlocal。它的核心思想是：共享变量在每个线程都有一个副本，每个线程操作的都是自己的副本，对另外的线程没有影响。特别注意，使用threadlocal时，使用完之后，要记得调用remove方法，不然可能会出现内存泄露问题

#### 线程通信（例如：A线程操作到某一步通知B线程）

1. thread.join(),
2. object.wait(),object.notify()
3. CountdownLatch
4. 使用volatile关键字
5. 使用ReentrantLock结合Condition
6. LockSupport是一种非常灵活的实现线程间阻塞和唤醒的工具，使用它不用关注是等待线程先进行还是唤醒线程先运行，但是得知道线程的名字

#### 最佳线程数

QPS=每秒钟request数量
TPS=每秒钟事务数量
RT=一般取平均响应时间
QPS=并发数/RT或者并发数=QPS * RT
最佳线程数=RT/CPU Time * CPU核心数 * CPU利用率
最大QPS=最佳线程数 * 单线程QPS=（RT/CPU Time * CPU核心数 * CPU利用率）\*（1/RT) = CPU核心数 * CPU利用率/CPU time

最佳线程经验值：
IO密集型配置线程数经验值是：2N，其中N代表CPU核数。
CPU密集型配置线程数经验值是：N + 1，其中N代表CPU核数。
如果获取N的值？
```java
int availableProcessors = Runtime.getRuntime().availableProcessors()
```
最佳线程数目 = （线程等待时间+线程CPU时间）/线程CPU时间 * CPU数目
数据库连接池连接数 = ((核心数 * 2) + 有效磁盘数)

#### 线程顺序执行

1. 使用线程的join方法
2. 使用主线程的join方法
3. 使用线程的wait方法
4. 使用线程的线程池方法
5. 使用线程的Condition(条件变量)方法
6. 使用线程的CountDownLatch(倒计数)方法
7. 使用线程的CyclicBarrier(回环栅栏)方法
8. 使用线程的Semaphore(信号量)方法

#### 线程相关文章

| [Java多线程与并发高频面试题解析](https://mp.weixin.qq.com/s/DIbxSun2NcD-wbqKGQpRLg) | [超赞，大牛总结的多线程的问题及答案](https://mp.weixin.qq.com/s/0KmWOLNqhck85WECC9uQ-g) | [99道Java多线程面试题，看完我跪了！](https://mp.weixin.qq.com/s/dRqLZG7eev87hda9ohlJrA) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [2万字长文详解10大多线程面试题](https://mp.weixin.qq.com/s/hq5GbYBe98YsBDNA3u2s5Q) | [两万字！多线程硬核50问！](https://mp.weixin.qq.com/s/wGJsOWAGUhlE4QlZsNpMXg) | [面试官：线程池中多余的线程是如何回收的？](https://mp.weixin.qq.com/s/Ts2DGoUJ6SOhdRuDaLa8UQ) |
| [你真的了解Thread线程类吗](https://mp.weixin.qq.com/s/PNHueqqUhKPihiJm3Hqmvw) | [面试官提问：线程中的wait和notify方法有啥作用？](https://mp.weixin.qq.com/s/ZY65yfzxVaWn-WFNKy1v4A) | [什么是线程组？](https://mp.weixin.qq.com/s/fbdJyn1Roa3Vxn1q1BPa6w) |

### 线程池

> ```java
> ThreadPoolExecutor extends AbstractExecutorService,
> AbstractExecutorService implements ExecutorService,
> ExecutorService extends Executor
> ```

#### Executors

```java
ExecutorService threadPool = Executors.newCachedThreadPool();

// newCachedThreadPool:创建一个可缓存线程池，可以无限扩大，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。适用于服务器负载较轻，执行很多短期异步任务。
// newFixedThreadPool:创建一个定长、固定大小的线程池，可控制线程最大并发数，超出的线程会在队列中等待，表示同一时刻只能有这么大的并发数，实际线程数量永远不会变化，适用于可以预测线程数量的业务中，或者服务器负载较重，对当前线程数量进行限制。
// newScheduledThreadPool:创建一个定长线程池，支持定时及周期性任务执行。可以延时启动，定时启动，适用于需要多个后台线程执行周期任务的场景。
// newSingleThreadExecutor:创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO,LIFO,优先级)执行。是一个单线程的线程池，适用于需要保证顺序执行各个任务，并且在任意时间点，不会有多个线程是活动的场景
// newWorkStealingPool:创建一个拥有多个任务队列的线程池，可以减少连接数，创建当前可用cpu数量的线程来并行执行，适用于大耗时的操作，可以并行来执行
// newSingleThreadScheduledExecutor:只有一个线程，该线程池可用于定时或周期性任务的执行，类似于Timer，但比Timer要更安全
```


#### 为什么阿里巴巴要禁用Executors创建线程池

《阿里巴巴Java开发手册》中强制线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor构造函数的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险

Executors返回线程池对象的弊端如下：

- **FixedThreadPool和SingleThreadExecutor**：使用的是无界的LinkedBlockingQueue，任务队列最大长度为Integer.MAX_VALUE,可能堆积大量的请求，从而导致OOM。
- **CachedThreadPool**：使用的是同步队列SynchronousQueue,允许创建的线程数量为Integer.MAX_VALUE，可能会创建大量线程，从而导致OOM。
- **ScheduledThreadPool和SingleThreadScheduledExecutor**：使用的无界的延迟阻塞队列DelayedWorkQueue，任务队列最大长度为Integer.MAX_VALUE,可能堆积大量的请求，从而导致OOM。

```java
// 无界队列LinkedBlockingQueue
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>());
}

// 无界队列LinkedBlockingQueue
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService (new ThreadPoolExecutor(1,1,0L,TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>()));
}

// 同步队列SynchronousQueue，没有容量，最大线程数是Integer.MAX_VALUE
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>());
}

// DelayedWorkQueue（延迟阻塞队列）
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```


#### ThreadPoolExecutor
```java
ExecutorService threadPool = new ThreadPoolExecutor(
    int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler
)

// corePoolSize：线程池的核心线程数(最小线程数)，不管它们创建以后是不是空闲的。线程池需要保持corePoolSize数量的线程，除非设置了allowCoreThreadTimeOut

// maximumPoolSize：线程池的最大线程数；

// keepAliveTime：线程池空闲时线程的存活时长；如果经过keepAliveTime时间后，超过核心线程数的线程还没有接受到新的任务，那就销毁，超出线程池核心线程数小于线程池最大线程数的线程都是借的，没有用了,超时就销毁

// unit：keepAliveTime时长单位；

// workQueue：当提交的任务数超过核心线程数大小后，再提交的任务就存放在这里。它仅仅用来存放被execute方法提交的Runnable任务。存放任务的队列，上面提到的线程数超过corePoolSize存放任务的地方；
// new ArrayBlockingQueue<Runnable>(10)：是一个基于数组结构的有界阻塞队列，此队列按FIFO（先进先出）原则对元素进行排序。
// new LinkedBlockingQueue<Runnable>(10)：一个基于链表结构的阻塞队列，此队列按FIFO（先进先出）排序元素，也可以不传参数，默认是Integer.MAX_VALUE
// new SynchronousQueue<Runnable>()：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，使用SynchronousQueue阻塞队列一般要求maximumPoolSizes为无界(Integer.MAX_VALUE)，避免线程拒绝执行操作。
// PriorityBlockingQueue：一个具有优先级的无限阻塞队列。
// DelayQueue:DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。

// threadFactory：线程工厂，可以自己重写一下，为每个线程赋予一个名字，便于排查问题
class MyThreadFactory implements ThreadFactory{
	@Override
	public Thread newThread(Runnable r){
		return new Thread(r,"thread_name");
	}
}

// handler：当队列里面放满了任务、最大线程数的线程都在工作时，这时继续提交的任务线程池就处理不了，应该执行怎么样的拒绝策略。
//在队列（workQueue）和线程池达到最大线程数（maximumPoolSize）均满时仍有任务的情况下的处理方式即当任务数大于最大线程数并且队列已满时，采用的拒绝策略，分4种，
new ThreadPoolExecutor.AbortPolicy //丢弃任务并抛出RejectedExecutionException异常
// AbortPolicy策略：默认策略，如果线程池队列满了丢掉这个任务并且抛出RejectedExecutionException异常。
new ThreadPoolExecutor.DiscardPolicy //丢弃任务，但是不抛出异常
// DiscardPolicy策略：如果线程池队列满了，会直接丢掉这个任务并且不会有任何异常。
new ThreadPoolExecutor.CallerRunsPolicy//（调用者运行）:如果线程池的线程数量达到上限，该策略会把任务队列中的任务放在调用者线程当中运行由调用线程处理该任务
// CallerRunsPolicy策略：如果添加到线程池失败，那么主线程会自己去执行该任务，不会等待线程池中的线程去执行。
new ThreadPoolExecutor.DiscardOldestPolicy //抛弃最旧的丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
// DiscardOldestPolicy策略：如果队列满了，会将最早进入队列的任务删掉腾出空间，再尝试加入队列。

// demo
ThreadPoolExecutor exec = new ThreadPoolExecutor(
10, 30, 300, TimeUnit.SECONDS, 
new ArrayBlockingQueue<Runnable>(10000),
Executors.defaultThreadFactory(), 
new ThreadPoolExecutor.CallerRunsPolicy());

// 核心线程数10，最大线程数30，keepAliveTime是3秒,随着任务数量不断上升，线程池会不断的创建线程，直到到达核心线程数10，就不创建线程了，这时多余的任务通过加入阻塞队列来运行，当超出阻塞队列长度+核心线程数时，这时不得不扩大线程个数来满足当前任务的运行，这时就需要创建新的线程了（最大线程数起作用），上限是最大线程数30那么超出核心线程数10并小于最大线程数30的可能新创建的这20个线程相当于是“借”的，如果这20个线程空闲时间超过keepAliveTime，就会被退出

```
**ArrayBlockingQueue、LinkedBlockingQueue区别**
![](/images/ArrayBlockingQueueLinkedBlockingQueue区别.png)

> [ThreadPoolExecutor深入解析](https://mp.weixin.qq.com/s/QEur_4cwOSc_4AHWAPFhJQ)

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
   } catch (InterruptedException e){
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
String result = futureTask.get(2000,TimeUnit.MILLISECONDS)// 如果在指定时间内，还没获取到结果，就直接返回null
```
> [线程池中的线程抛出了异常，该如何处理](https://mp.weixin.qq.com/s/LVAv7PKVvg7prW9ayD1eZA)
> [3分钟带你秒懂线程池设计机制](https://mp.weixin.qq.com/s/nKzjIOR_rMrPFw6PB7WUdA)


#### 为何先入队列再增加线程数？

- 资源管理与节约成本：Java线程池的设计目标之一是高效地利用系统资源。当任务到来时，如果当前线程数未达到最大线程数限制，优先将任务放入队列等待执行，而不是立即创建新线程。这样可以避免频繁地创建和销毁线程，节约了系统资源和开销。
- 避免线程爆炸：如果任务到来速度过快，直接增加线程数可能会导致线程数爆炸式增长，从而消耗过多的系统资源和内存。通过先将任务入队列，可以平滑地控制线程数量的增长，避免线程数量不受控制地增加。
- 防止资源竞争：在多线程环境下，线程之间可能会因为竞争资源而导致性能下降甚至死锁。通过将任务先放入队列，可以避免线程之间过度竞争共享资源，减少了竞争的可能性，提高了系统的稳定性和可靠性。
- 任务处理的优先级：在任务队列中，可以通过不同的调度策略对任务进行优先级排序，根据任务的重要性和紧急程度来决定执行顺序。这样可以更灵活地控制任务的执行顺序，提高系统的响应速度和效率。


### 虚拟线程

> JDK 19新推出的虚拟线程，或者叫协程，主要是为了解决在读书操作系统中线程需要依赖内核线程的实现，导致有很多额外开销的问题。通过在Java语言层面引入虚拟线程，通过JVM进行调度管理，从而减少上下文切换的成本。
虚拟线程是守护线程，所以有可能会没等他执行完虚拟机就会shutdown掉。

#### 虚拟线程与平台线程的区别
1. **虚拟线程总是守护线程**。setDaemon(false)方法不能将虚拟线程更改为非守护线程。所以，需要注意的是，**当所有启动的非守护进程线程都终止时，JVM将终止。这意味着JVM不会等待虚拟线程完成后才退出**。
2. 即使使用setPriority()方法，**虚拟线程始终具有normal的优先级**，且不能更改优先级。在虚拟线程上调用此方法没有效果。
3. 虚拟线程是不支持stop()、suspend()或resume()等方法。这些方法在虚拟线程上调用时会抛出UnsupportedOperationException异常。

#### 如何使用虚拟线程

首先，通过Thread.startVirtualThread()可以运行一个虚拟线程：
```java
Thread.startVirtualThread(() -> {
    System.out.println("虚拟线程执行中...");
});
```
其次，通过Thread.Builder也可以创建虚拟线程，Thread类提供了ofPlatform()来创建一个平台线程、ofVirtual()来创建虚拟现场。
```java
Thread.Builder platformBuilder = Thread.ofPlatform().name("平台线程");
Thread.Builder virtualBuilder = Thread.ofVirtual().name("虚拟线程");

Thread t1 = platformBuilder .start(() -> {...});
Thread t2 = virtualBuilder.start(() -> {...});
```

另外，线程池也支持了虚拟线程，可以通过Executors.newVirtualThreadPerTaskExecutor()来创建虚拟线程：
```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10000).forEach(i -> {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return i;
        });
    });
}
```
但是，其实并不建议虚拟线程和线程池一起使用，因为Java线程池的设计是为了避免创建新的操作系统线程的开销，但是创建虚拟线程的开销并不大，所以其实没必要放到线程池中。

#### 相关文章
- [科技与狠活？JDK19中的虚拟线程到底什么鬼？](https://mp.weixin.qq.com/s/1AuTVrBJmONKEku403BHhQ)
- [虚拟线程将会深刻影响大规模Java应用的并发机制](https://mp.weixin.qq.com/s/UkZjAcsYWncFBHc8DhnGcA)

