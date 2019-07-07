# BATJ都爱问的多线程面试题 #

下面最近发的一些并发编程的文章汇总，通过阅读这些文章大家再看大厂面试中的并发编程问题就没有那么头疼了。今天给大家总结一下，面试中出镜率很高的几个多线程面试题，希望对大家学习和面试都能有所帮助。备注：文中的代码自己实现一遍的话效果会更佳哦！

* [并发编程面试必备：synchronized 关键字使用、底层原理、JDK1.6 之后的底层优化以及 和ReenTrantLock 的对比]( https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU4NDQ4MzU5OA%3D%3D%26amp%3Bmid%3D2247484539%26amp%3Bidx%3D1%26amp%3Bsn%3D3500cdcd5188bdc253fb19a1bfa805e6%26amp%3Bchksm%3Dfd98521acaefdb0c5167247a1fa903a1a53bb4e050b558da574f894f9feda5378ec9d0fa1ac7%26amp%3Bscene%3D21%23wechat_redirect )
* [并发编程面试必备：JUC 中的 Atomic 原子类总结]( https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU4NDQ4MzU5OA%3D%3D%26amp%3Bmid%3D2247484553%26amp%3Bidx%3D1%26amp%3Bsn%3Daca9fa19f723206eff7e33a10973a887%26amp%3Bchksm%3Dfd9852e8caefdbfe7180c34f83bbb422a1a0bef1ed44b1e84f56924244ea3fd2da720f25c6dd%23rd )
* [并发编程面试必备：AQS 原理以及 AQS 同步组件总结]( https://link.juejin.im?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU4NDQ4MzU5OA%3D%3D%26amp%3Bmid%3D2247484559%26amp%3Bidx%3D1%26amp%3Bsn%3D28dae85c38c4c500201c39234d25d731%26amp%3Bchksm%3Dfd9852eecaefdbf80cc54a25204e7c7d81170ce659acf92b7fa4151799ca3d0d7df2225d4ff1%23rd )

> 
> 
> 
> 该文已加入开源文档：JavaGuide（一份涵盖大部分Java程序员所需要掌握的核心知识）。地址: [github.com/Snailclimb/…](
> https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2FSnailclimb%2FJavaGuide
> ).
> 
> 
> 
> 腾讯云热门云产品1折起，送13000元续费/升级大礼包： [cloud.tencent.com/redirect.ph…](
> https://link.juejin.im?target=https%3A%2F%2Fcloud.tencent.com%2Fredirect.php%3Fredirect%3D1034%26amp%3Bcps_key%3D2b96dd3b35e69197e2f3dfb779a6139b%26amp%3Bfrom%3Dconsole
> )
> 
> 
> 
> 腾讯云新用户大额代金券： [cloud.tencent.com/redirect.ph…](
> https://link.juejin.im?target=https%3A%2F%2Fcloud.tencent.com%2Fredirect.php%3Fredirect%3D1025%26amp%3Bcps_key%3D2b96dd3b35e69197e2f3dfb779a6139b%26amp%3Bfrom%3Dconsole
> )
> 
> 

# 一 面试中关于 synchronized 关键字的 5 连击 #

### 1.1 说一说自己对于 synchronized 关键字的了解 ###

synchronized关键字解决的是多个线程之间访问资源的同步性，synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。

另外，在 Java 早期版本中，synchronized属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。庆幸的是在 Java 6 之后 Java 官方对从 JVM 层面对synchronized 较大优化，所以现在的 synchronized 锁效率也优化得很不错了。JDK1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

### 1.2 说说自己是怎么使用 synchronized 关键字，在项目中用到了吗 ###

**synchronized关键字最主要的三种使用方式：**

* **修饰实例方法，作用于当前对象实例加锁，进入同步代码前要获得当前对象实例的锁**
* **修饰静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁** 。也就是给当前类加锁，会作用于类的所有对象实例，因为静态成员不属于任何一个实例对象，是类成员（ static 表明这是该类的一个静态资源，不管new了多少个对象，只有一份，所以对该类的所有对象都加了锁）。所以如果一个线程A调用一个实例对象的非静态 synchronized 方法，而线程B需要调用这个实例对象所属类的静态 synchronized 方法，是允许的，不会发生互斥现象， **因为访问静态 synchronized 方法占用的锁是当前类的锁，而访问非静态 synchronized 方法占用的锁是当前实例对象锁** 。
* **修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。** 和 synchronized 方法一样，synchronized(this)代码块也是锁定当前对象的。synchronized 关键字加到 static 静态方法和 synchronized(class)代码块上都是是给 Class 类上锁。这里再提一下：synchronized关键字加到非 static 静态方法上是给对象实例上锁。另外需要注意的是：尽量不要使用 synchronized(String a) 因为JVM中，字符串常量池具有缓冲功能！

下面我已一个常见的面试题为例讲解一下 synchronized 关键字的具体使用。

面试中面试官经常会说：“单例模式了解吗？来给我手写一下！给我解释一下双重检验锁方式实现单利模式的原理呗！”

**双重校验锁实现对象单例（线程安全）**

` public class Singleton { private volatile static Singleton uniqueInstance; private Singleton () { } public static Singleton getUniqueInstance () { //先判断对象是否已经实例过，没有实例化过才进入加锁代码 if (uniqueInstance == null ) { //类对象加锁 synchronized (Singleton.class) { if (uniqueInstance == null ) { uniqueInstance = new Singleton(); } } } return uniqueInstance; } } 复制代码`

另外，需要注意 uniqueInstance 采用 volatile 关键字修饰也是很有必要。

uniqueInstance 采用 volatile 关键字修饰也是很有必要的， uniqueInstance = new Singleton(); 这段代码其实是分为三步执行：

* 为 uniqueInstance 分配内存空间
* 初始化 uniqueInstance
* 将 uniqueInstance 指向分配的内存地址

但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出先问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。

使用 volatile 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。

### 1.3 讲一下 synchronized 关键字的底层原理 ###

**synchronized 关键字底层原理属于 JVM 层面。**

**① synchronized 同步语句块的情况**

` public class SynchronizedDemo { public void method () { synchronized ( this ) { System.out.println( "synchronized 代码块" ); } } } 复制代码`

通过 JDK 自带的 javap 命令查看 SynchronizedDemo 类的相关字节码信息：首先切换到类的对应目录执行 ` javac SynchronizedDemo.java` 命令生成编译后的 .class 文件，然后执行 ` javap -c -s -v -l SynchronizedDemo.class` 。

![synchronized 关键字原理](https://user-gold-cdn.xitu.io/2018/10/26/166add616a292bcf?imageView2/0/w/1280/h/960/ignore-error/1)

从上面我们可以看出：

**synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。** 当执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权.当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

**② synchronized 修饰方法的的情况**

` public class SynchronizedDemo2 { public synchronized void method () { System.out.println( "synchronized 方法" ); } } 复制代码`

![synchronized 关键字原理](https://user-gold-cdn.xitu.io/2018/10/26/166add6169fc206d?imageView2/0/w/1280/h/960/ignore-error/1)

synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

### 1.4 说说 JDK1.6 之后的synchronized 关键字底层做了哪些优化，可以详细介绍一下这些优化吗 ###

JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。

锁主要存在四中状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。

关于这几种优化的详细信息可以查看： [synchronized 关键字使用、底层原理、JDK1.6 之后的底层优化以及 和ReenTrantLock 的对比]( https://link.juejin.im?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU4NDQ4MzU5OA%3D%3D%26amp%3Bmid%3D2247484539%26amp%3Bidx%3D1%26amp%3Bsn%3D3500cdcd5188bdc253fb19a1bfa805e6%26amp%3Bchksm%3Dfd98521acaefdb0c5167247a1fa903a1a53bb4e050b558da574f894f9feda5378ec9d0fa1ac7%26amp%3Btoken%3D1604028915%26amp%3Blang%3Dzh_CN%23rd )

### 1.5 谈谈 synchronized和ReenTrantLock 的区别 ###

**① 两者都是可重入锁**

两者都是可重入锁。“可重入锁”概念是：自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。

**② synchronized 依赖于 JVM 而 ReenTrantLock 依赖于 API**

synchronized 是依赖于 JVM 实现的，前面我们也讲到了 虚拟机团队在 JDK1.6 为 synchronized 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。ReenTrantLock 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。

**③ ReenTrantLock 比 synchronized 增加了一些高级功能**

相比synchronized，ReenTrantLock增加了一些高级功能。主要来说主要有三点： **①等待可中断；②可实现公平锁；③可实现选择性通知（锁可以绑定多个条件）**

* **ReenTrantLock提供了一种能够中断等待锁的线程的机制** ，通过lock.lockInterruptibly()来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
* **ReenTrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。** ReenTrantLock默认情况是非公平的，可以通过 ReenTrantLock类的 ` ReentrantLock(boolean fair)` 构造方法来制定是否是公平的。
* synchronized关键字与wait()和notify/notifyAll()方法相结合可以实现等待/通知机制，ReentrantLock类当然也可以实现，但是需要借助于Condition接口与newCondition() 方法。Condition是JDK1.5之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition实例（即对象监视器）， **线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用notify/notifyAll()方法进行通知时，被通知的线程是由 JVM 选择的，用ReentrantLock类结合Condition实例可以实现“选择性通知”** ，这个功能非常重要，而且是Condition接口默认提供的。而synchronized关键字就相当于整个Lock对象中只有一个Condition实例，所有的线程都注册在它一个身上。如果执行notifyAll()方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题，而Condition实例的signalAll()方法 只会唤醒注册在该Condition实例中的所有等待线程。

如果你想使用上述功能，那么选择ReenTrantLock是一个不错的选择。

**④ 性能已不是选择标准**

# 二 面试中关于线程池的 4 连击 #

### 2.1 讲一下Java内存模型 ###

在 JDK1.2 之前，Java的内存模型实现总是从 **主存** （即共享内存）读取变量 ，是不需要进行特别的注意的。而在当前的 Java 内存模型下，线程可以把变量保存 **本地内存** （比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成 **数据的不一致** 。

![数据的不一致](https://user-gold-cdn.xitu.io/2018/10/30/166c46ede4423ba2?imageView2/0/w/1280/h/960/ignore-error/1)

要解决这个问题，就需要把变量声明为 **volatile** ，这就指示 JVM，这个变量是不稳定的，每次使用它都到主存中进行读取。

说白了， **volatile** 关键字的主要作用就是保证变量的可见性然后还有一个作用是防止指令重排序。

![volatile关键字的可见性](https://user-gold-cdn.xitu.io/2018/10/30/166c46ede4b9f501?imageView2/0/w/1280/h/960/ignore-error/1)

### 2.2 说说 synchronized 关键字和 volatile 关键字的区别 ###

synchronized关键字和volatile关键字比较

* **volatile关键字** 是线程同步的 **轻量级实现** ，所以 **volatile性能肯定比synchronized关键字要好** 。但是 **volatile关键字只能用于变量而synchronized关键字可以修饰方法以及代码块** 。synchronized关键字在JavaSE1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁以及其它各种优化之后执行效率有了显著提升， **实际开发中使用 synchronized 关键字的场景还是更多一些** 。
* **多线程访问volatile关键字不会发生阻塞，而synchronized关键字可能会发生阻塞**
* **volatile关键字能保证数据的可见性，但不能保证数据的原子性。synchronized关键字两者都能保证。**
* **volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized关键字解决的是多个线程之间访问资源的同步性。**

# 三 面试中关于 线程池的 2 连击 #

### 3.1 为什么要用线程池？ ###

线程池提供了一种限制和管理资源（包括执行一个任务）。 每个线程池还维护一些基本统计信息，例如已完成任务的数量。

这里借用《Java并发编程的艺术》提到的来说一下使用线程池的好处：

* **降低资源消耗。** 通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
* **提高响应速度。** 当任务到达时，任务可以不需要的等到线程创建就能立即执行。
* **提高线程的可管理性。** 线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

### 3.2 实现Runnable接口和Callable接口的区别 ###

如果想让线程池执行任务的话需要实现的Runnable接口或Callable接口。 Runnable接口或Callable接口实现类都可以被ThreadPoolExecutor或ScheduledThreadPoolExecutor执行。两者的区别在于 Runnable 接口不会返回结果但是 Callable 接口可以返回结果。

**备注：** 工具类 ` Executors` 可以实现 ` Runnable` 对象和 ` Callable` 对象之间的相互转换。（ ` Executors.callable（Runnable task）` 或 ` Executors.callable（Runnable task，Object resule）` ）。

### 3.3 执行execute()方法和submit()方法的区别是什么呢？ ###

1) **` execute()` 方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；**

2) **submit()方法用于提交需要返回值的任务。线程池会返回一个future类型的对象，通过这个future对象可以判断任务是否执行成功** ，并且可以通过future的get()方法来获取返回值，get()方法会阻塞当前线程直到任务完成，而使用 ` get（long timeout，TimeUnit unit）` 方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

### 3.4 如何创建线程池 ###

《阿里巴巴Java开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险**

> 
> 
> 
> Executors 返回线程池对象的弊端如下：
> 
> 
> 
> * **FixedThreadPool 和 SingleThreadExecutor** ： 允许请求的队列长度为
> Integer.MAX_VALUE,可能堆积大量的请求，从而导致OOM。
> * **CachedThreadPool 和 ScheduledThreadPool** ： 允许创建的线程数量为 Integer.MAX_VALUE
> ，可能会创建大量线程，从而导致OOM。
> 
> 
> 

**方式一：通过构造方法实现**

![通过构造方法实现](https://user-gold-cdn.xitu.io/2018/10/30/166c4a5baac923e9?imageView2/0/w/1280/h/960/ignore-error/1) **方式二：通过Executor 框架的工具类Executors来实现** 我们可以创建三种类型的ThreadPoolExecutor：

* **FixedThreadPool** ： 该方法返回一个固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。
* **SingleThreadExecutor：** 方法返回一个只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。
* **CachedThreadPool：** 该方法返回一个可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。

对应Executors工具类中的方法如图所示：

![通过Executor 框架的工具类Executors来实现](https://user-gold-cdn.xitu.io/2018/10/30/166c4a5baa9ca5e9?imageView2/0/w/1280/h/960/ignore-error/1)

# 四 面试中关于 Atomic 原子类的 4 连击 #

### 4.1 介绍一下Atomic 原子类 ###

Atomic 翻译成中文是原子的意思。在化学上，我们知道原子是构成一般物质的最小单位，在化学反应中是不可分割的。在我们这里 Atomic 是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。

所以，所谓原子类说简单点就是具有原子/原子操作特征的类。

并发包 ` java.util.concurrent` 的原子类都存放在 ` java.util.concurrent.atomic` 下,如下图所示。

![JUC 原子类概览](https://user-gold-cdn.xitu.io/2018/10/30/166c4ac08d4c5547?imageView2/0/w/1280/h/960/ignore-error/1)

### 4.2 JUC 包中的原子类是哪4类? ###

**基本类型**

使用原子的方式更新基本类型

* AtomicInteger：整形原子类
* AtomicLong：长整型原子类
* AtomicBoolean ：布尔型原子类

**数组类型**

使用原子的方式更新数组里的某个元素

* AtomicIntegerArray：整形数组原子类
* AtomicLongArray：长整形数组原子类
* AtomicReferenceArray ：引用类型数组原子类

**引用类型**

* AtomicReference：引用类型原子类
* AtomicStampedRerence：原子更新引用类型里的字段原子类
* AtomicMarkableReference ：原子更新带有标记位的引用类型

**对象的属性修改类型**

* AtomicIntegerFieldUpdater:原子更新整形字段的更新器
* AtomicLongFieldUpdater：原子更新长整形字段的更新器
* AtomicStampedReference ：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。

### 4.3 讲讲 AtomicInteger 的使用 ###

**AtomicInteger 类常用方法**

` public final int get () //获取当前的值 public final int getAndSet ( int newValue) //获取当前的值，并设置新的值 public final int getAndIncrement () //获取当前的值，并自增 public final int getAndDecrement () //获取当前的值，并自减 public final int getAndAdd ( int delta) //获取当前的值，并加上预期的值 boolean compareAndSet ( int expect, int update) //如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update） public final void lazySet ( int newValue) //最终设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。 复制代码`

**AtomicInteger 类的使用示例**

使用 AtomicInteger 之后，不用对 increment() 方法加锁也可以保证线程安全。

` class AtomicIntegerTest { private AtomicInteger count = new AtomicInteger(); //使用AtomicInteger之后，不需要对该方法加锁，也可以实现线程安全。 public void increment () { count.incrementAndGet(); } public int getCount () { return count.get(); } } 复制代码`

### 4.4 能不能给我简单介绍一下 AtomicInteger 类的原理 ###

AtomicInteger 线程安全原理简单分析

AtomicInteger 类的部分源码：

` // setup to use Unsafe.compareAndSwapInt for updates（更新操作时提供“比较并替换”的作用） private static final Unsafe unsafe = Unsafe.getUnsafe(); private static final long valueOffset; static { try { valueOffset = unsafe.objectFieldOffset (AtomicInteger.class.getDeclaredField( "value" )); } catch (Exception ex) { throw new Error(ex); } } private volatile int value; 复制代码`

AtomicInteger 类主要利用 CAS (compare and swap) + volatile 和 native 方法来保证原子操作，从而避免 synchronized 的高开销，执行效率大为提升。

CAS的原理是拿期望的值和原本的一个值作比较，如果相同则更新成新的值。UnSafe 类的 objectFieldOffset() 方法是一个本地方法，这个方法是用来拿到“原来的值”的内存地址，返回值是 valueOffset。另外 value 是一个volatile变量，在内存中可见，因此 JVM 可以保证任何时刻任何线程总能拿到该变量的最新值。

关于 Atomic 原子类这部分更多内容可以查看我的这篇文章：并发编程面试必备： [JUC 中的 Atomic 原子类总结]( https://link.juejin.im?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fjoa-yOiTrYF67bElj8xqvg )

# 五 AQS #

### 5.1 AQS 介绍 ###

AQS的全称为（AbstractQueuedSynchronizer），这个类在java.util.concurrent.locks包下面。

![enter image description here](https://user-gold-cdn.xitu.io/2018/10/30/166c4bb575d4a690?imageView2/0/w/1280/h/960/ignore-error/1)

AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。当然，我们自己也能利用AQS非常轻松容易地构造出符合我们自己需求的同步器。

### 5.2 AQS 原理分析 ###

AQS 原理这部分参考了部分博客，在5.2节末尾放了链接。

> 
> 
> 
> 在面试中被问到并发知识的时候，大多都会被问到“请你说一下自己对于AQS原理的理解”。下面给大家一个示例供大家参加，面试不是背题，大家一定要假如自己的思想，即使加入不了自己的思想也要保证自己能够通俗的讲出来而不是背出来。
> 
> 
> 

下面大部分内容其实在AQS类注释上已经给出了，不过是英语看着比较吃力一点，感兴趣的话可以看看源码。

#### 5.2.1 AQS 原理概览 ####

**AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

> 
> 
> 
> CLH(Craig,Landin,and
> Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。
> 
> 
> 

看个AQS(AbstractQueuedSynchronizer)原理图：

![enter image description here](https://user-gold-cdn.xitu.io/2018/10/30/166c4bbe4a9c5ae7?imageView2/0/w/1280/h/960/ignore-error/1)

AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

` private volatile int state; //共享变量，使用volatile修饰保证线程可见性 复制代码`

状态信息通过procted类型的getState，setState，compareAndSetState进行操作

` //返回同步状态的当前值 protected final int getState () { return state; } // 设置同步状态的值 protected final void setState ( int newState) { state = newState; } //原子地（CAS操作）将同步状态值设置为给定值update如果当前同步状态的值等于expect（期望值） protected final boolean compareAndSetState ( int expect, int update) { return unsafe.compareAndSwapInt( this , stateOffset, expect, update); } 复制代码`

#### 5.2.2 AQS 对资源的共享方式 ####

**AQS定义两种资源共享方式**

* **Exclusive** （独占）：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：

* 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
* 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的

* **Share** （共享）：多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。

ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在顶层实现好了。

#### 5.2.3 AQS底层使用了模板方法模式 ####

同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

* 使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
* 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

这和我们以往通过实现接口的方式有很大区别，这是模板方法模式很经典的一个运用。

**AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：**

` isHeldExclusively() //该线程是否正在独占资源。只有用到condition才需要去实现它。 tryAcquire( int ) //独占方式。尝试获取资源，成功则返回true，失败则返回false。 tryRelease( int ) //独占方式。尝试释放资源，成功则返回true，失败则返回false。 tryAcquireShared( int ) //共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。 tryReleaseShared( int ) //共享方式。尝试释放资源，成功则返回true，失败则返回false。 复制代码`

默认情况下，每个方法都抛出 ` UnsupportedOperationException` 。 这些方法的实现必须是内部线程安全的，并且通常应该简短而不是阻塞。AQS类中的其他方法都是final ，所以无法被其他类使用，只有这几个方法可以被其他类使用。

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

再以CountDownLatch以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS(Compare and Swap)减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现 ` tryAcquire-tryRelease` 、 ` tryAcquireShared-tryReleaseShared` 中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如 ` ReentrantReadWriteLock` 。

推荐两篇 AQS 原理和相关源码分析的文章：

* [www.cnblogs.com/waterystone…]( https://link.juejin.im?target=http%3A%2F%2Fwww.cnblogs.com%2Fwaterystone%2Fp%2F4920797.html )
* [www.cnblogs.com/chengxiao/a…]( https://link.juejin.im?target=https%3A%2F%2Fwww.cnblogs.com%2Fchengxiao%2Farchive%2F2017%2F07%2F24%2F7141160.html )

### 5.3 AQS 组件总结 ###

* **Semaphore(信号量)-允许多个线程同时访问：** synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore(信号量)可以指定多个线程同时访问某个资源。
* **CountDownLatch （倒计时器）：** CountDownLatch是一个同步工具类，用来协调多个线程之间的同步。这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。
* **CyclicBarrier(循环栅栏)：** CyclicBarrier 和 CountDownLatch 非常类似，它也可以实现线程间的技术等待，但是它的功能比 CountDownLatch 更加复杂和强大。主要应用场景和 CountDownLatch 类似。CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。CyclicBarrier默认的构造方法是 CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用await方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞。

关于AQS这部分的更多内容可以查看我的这篇文章: [并发编程面试必备：AQS 原理以及 AQS 同步组件总结]( https://link.juejin.im?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fjoa-yOiTrYF67bElj8xqvg )

# Reference #

* 《深入理解 Java 虚拟机》
* 《实战 Java 高并发程序设计》
* 《Java并发编程的艺术》
* [www.cnblogs.com/waterystone…]( https://link.juejin.im?target=http%3A%2F%2Fwww.cnblogs.com%2Fwaterystone%2Fp%2F4920797.html )
* [www.cnblogs.com/chengxiao/a…]( https://link.juejin.im?target=https%3A%2F%2Fwww.cnblogs.com%2Fchengxiao%2Farchive%2F2017%2F07%2F24%2F7141160.html )

【强烈推荐！非广告！】阿里云双11褥羊毛活动（10.29-11.12）： [m.aliyun.com/act/team111…]( https://link.juejin.im?target=https%3A%2F%2Fm.aliyun.com%2Fact%2Fteam1111%2F%23%2Fshare%3Fparams%3DN.FF7yxCciiM.hf47liqn ) 。一句话解析该次活动：新用户低至一折购买（1核2g服务器仅8.3/月，比学生机还便宜，真的强烈推荐屯3年）。老用户可以加入我的战队，然后分享自己的链接，可以获得红包和25%的返现，我们的战队目前300位新人，所以可以排进前100，后面可以瓜分百万现金（按拉新人数瓜分现金，拉的越多分的越多！不要自己重新开战队，后面不能参与瓜分现金）。

> 
> 
> 
> 你若盛开，清风自来。
> 欢迎关注我的微信公众号：“Java面试通关手册”，一个有温度的微信公众号。公众号后台回复关键字“1”，可以免费获取一份我精心准备的小礼物哦！
> 
> 

![](https://user-gold-cdn.xitu.io/2018/7/5/1646a3d308a8db1c?imageView2/0/w/1280/h/960/ignore-error/1)