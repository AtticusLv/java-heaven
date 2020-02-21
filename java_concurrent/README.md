<!-- TOC -->

- [一、使用线程](#一使用线程)
    - [线程状态](#线程状态)
    - [实现Runnable接口](#实现runnable接口)
    - [实现Callable接口](#实现callable接口)
    - [继承Thread](#继承thread)
    - [对比实现接口与继承Thread](#对比实现接口与继承thread)
    - [Callable与Runnable对比](#callable与runnable对比)
- [二、基础线程机制](#二基础线程机制)
    - [Executor](#executor)
    - [Daemon](#daemon)
    - [sleep()](#sleep)
    - [yield()](#yield)
    - [join()](#join)
- [三、中断](#三中断)
    - [InterruptedException](#interruptedexception)
    - [interrupted()方法](#interrupted方法)
    - [Excecutor的中断操作](#excecutor的中断操作)
- [四、互斥同步](#四互斥同步)
    - [synchronized](#synchronized)
        - [1.同步一个代码块](#1同步一个代码块)
        - [2.同步一个方法](#2同步一个方法)
        - [3.同步一个类](#3同步一个类)
        - [4.同步一个静态方法](#4同步一个静态方法)
    - [ReentrantLock](#reentrantlock)
    - [比较](#比较)
        - [1. 锁的实现](#1-锁的实现)
        - [2. 性能](#2-性能)
        - [3. 等待可中断](#3-等待可中断)
        - [4. 公平锁](#4-公平锁)
        - [5. 锁绑定多个条件](#5-锁绑定多个条件)
    - [使用选择](#使用选择)
- [五、线程之间的协作](#五线程之间的协作)
    - [join()](#join-1)
    - [wait() notify() notifyAll()](#wait-notify-notifyall)
        - [wait() 和 sleep() 的区别](#wait-和-sleep-的区别)
    - [await() signal() signalAll()](#await-signal-signalall)
- [JVM线程状态](#jvm线程状态)
    - [NEW新建](#new新建)
    - [RUNNABLE可运行](#runnable可运行)
    - [BLOCKED阻塞](#blocked阻塞)
    - [WAITING无限期等待](#waiting无限期等待)
    - [TIMED_WAITING限期等待](#timed_waiting限期等待)
    - [TERMINATED死亡](#terminated死亡)
- [七、J.U.C - AQS](#七juc---aqs)
    - [CountDownLatch](#countdownlatch)
    - [CyclicBarrier](#cyclicbarrier)
    - [Semaphore](#semaphore)
- [八、J.U.C - 其他组件](#八juc---其他组件)
    - [FutureTask](#futuretask)
    - [BlockingQueue](#blockingqueue)
        - [使用 BlockingQueue 实现生产者消费者问题](#使用-blockingqueue-实现生产者消费者问题)
    - [ForkJoin](#forkjoin)
- [九、线程不安全](#九线程不安全)
- [十、Java内存模型](#十java内存模型)
    - [主内存与工作内存](#主内存与工作内存)
    - [内存模型三大特性](#内存模型三大特性)
        - [1.原子性](#1原子性)
        - [2.可见性](#2可见性)
        - [3.有序性](#3有序性)
    - [先行先发原则](#先行先发原则)
- [十一、线程安全](#十一线程安全)
    - [不可变](#不可变)
    - [互斥同步（悲观锁）](#互斥同步悲观锁)
    - [非阻塞同步（乐观锁）](#非阻塞同步乐观锁)
        - [1.CAS](#1cas)
        - [2.AtomicInteger](#2atomicinteger)
        - [3.ABA问题](#3aba问题)
    - [无同步方案](#无同步方案)
        - [1.栈封闭](#1栈封闭)
        - [2.线程本地存储](#2线程本地存储)
        - [3.可重入代码](#3可重入代码)
- [十二、锁优化](#十二锁优化)
    - [自旋锁](#自旋锁)
    - [锁消除](#锁消除)
    - [锁粗化](#锁粗化)
    - [偏向锁](#偏向锁)
    - [轻量级锁 CAS](#轻量级锁-cas)
    - [重量级锁](#重量级锁)
- [多线程开发良好习惯](#多线程开发良好习惯)

<!-- /TOC -->
# 一、使用线程
使用线程方法：
- 实现Runnable接口
- 实现Callable接口
- 继承Thread类

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以理解为任务是通过线程驱动从而执行的。
## 线程状态
- new 新建
- runnable 就绪
- running 运行
- dead 死亡
- blocked 阻塞
## 实现Runnable接口
需要实现接口中的run()方法
```
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        // ...
    }
}
```
在实际操作代码中创建一个Thread实例，然后调用Thread实例的start()方法来启动线程
```
public static void main(String[] args) {
    MyRunnable instance = new MyRunnable();
    Thread thread = new Thread(instance);
    thread.start();
}
```
## 实现Callable接口
与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。
```
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}
```
```
public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```
## 继承Thread
同样需要实现run()方法

当调用start()方法启动一个线程时，JVM会将改线程放入就绪队列中等待被带哦度，当一个线程被调度时会执行该线程的run()方法。

## 对比实现接口与继承Thread
实现接口会好一些
- Java不支持多重继承，因此继承了Thread类无法继承其他类，但是可以实现多个接口

- 类可能只要求课执行就行，继承整个Thread类开销太大

## Callable与Runnable对比
- Callable有返回值，Runnable没有
- Callable有
# 二、基础线程机制
## Executor
Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor：

- CachedThreadPool：一个任务创建一个线程；
- FixedThreadPool：所有任务只能使用固定大小的线程；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

## Daemon
守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。

在线程启动之前使用 setDaemon() 方法可以将一个线程设置为守护线程。
```
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```
## sleep()
Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。
```
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```
## yield()
对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。
```
public void run() {
    Thread.yield();
}
```
## join()
线程进入阻塞状态

当某个线程拥有cpu资源时，它决定把资源让给另一个特定的线程
# 三、中断
一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。
## InterruptedException
通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

## interrupted()方法
如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。
## Excecutor的中断操作
调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

如果只想中断 Executor 中的一个线程，可以通过使用 ExecutorService中的submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。

# 四、互斥同步
Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。
## synchronized
### 1.同步一个代码块
作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。
### 2.同步一个方法
作用于同一个对象
### 3.同步一个类
作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。
### 4.同步一个静态方法
```
public synchronized static void staticMethod() {
    // do sth
}
```
## ReentrantLock
ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

[ReentrantLock分析](https://www.jianshu.com/p/4e54802c965f)

## 比较
### 1. 锁的实现

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

### 2. 性能

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

### 3. 等待可中断

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

### 4. 公平锁

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

### 5. 锁绑定多个条件

一个 ReentrantLock 可以同时绑定多个 Condition 对象。
## 使用选择
除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

JDK1.8对synchronized做了升级锁，在最开始获取锁时是偏向锁，然后再次获取锁失败就省纪委CAS轻量级锁，再次失败就会短暂自旋，防止线程别系统挂起，最后如果都失败就升级为重量级锁

# 五、线程之间的协作
## join()
在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束后才再继续执行
## wait() notify() notifyAll()
调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

### wait() 和 sleep() 的区别
- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

## await() signal() signalAll()
java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。
```
public class AwaitSignalExample {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```
```
public static void main(String[] args) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    AwaitSignalExample example = new AwaitSignalExample();
    executorService.execute(() -> example.after());
    executorService.execute(() -> example.before());
}
```
```
before
after
```
# JVM线程状态
## NEW新建
创建后未启动
## RUNNABLE可运行
正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。
## BLOCKED阻塞
请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其它线程已经占用了该 monitor lock，所以出于阻塞状态。要结束该状态进入从而 RUNABLE 需要其他线程释放 monitor lock。
## WAITING无限期等待
等待其它线程显式地唤醒。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取 monitor lock。而等待是主动的，通过调用 Object.wait() 等方法进入。
## TIMED_WAITING限期等待
无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。
## TERMINATED死亡
可以是线程结束任务之后自己结束，或者产生了异常而结束。

# 七、J.U.C - AQS
[AQS详解](https://blog.csdn.net/mulinsen77/article/details/84583716?depth_1.utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)

AQS是将每一条请求的共享资源的线程封装成一个先进先出的双向队列（仅存在节点之间的关联关系），用volatile修饰共享变量state，线程通过CAS去改变状态符，成功则获取锁，失败则进入等待队列等待被唤醒（采用自旋的方式，【AQS是自旋锁】，不停地尝试获取锁）
## CountDownLatch
维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，那些因为调用 await() 方法而在等待的线程就会被唤醒。

## CyclicBarrier
和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

## Semaphore
Semaphore 类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。
# 八、J.U.C - 其他组件
## FutureTask
在介绍 Callable 时我们知道它可以有返回值，返回值通过 Future 进行封装。FutureTask 实现了 RunnableFuture 接口，该接口继承自 Runnable 和 Future 接口，这使得 FutureTask 既可以当做一个任务执行，也可以有返回值。
```
public class FutureTask<V> implements RunnableFuture<V>
```
```
public interface RunnableFuture<V> extends Runnable, Future<V>
```
FutureTask 可用于异步获取执行结果或取消执行任务的场景。当一个计算任务需要执行很长时间，那么就可以用 FutureTask 来封装这个任务，主线程在完成自己的任务之后再去获取结果。

## BlockingQueue
BlockingQueue有以下阻塞队列实现：
- FIFO 队列 ：LinkedBlockingQueue、ArrayBlockingQueue（固定长度）
- 优先级队列 ：PriorityBlockingQueue

提供了阻塞的 take() 和 put() 方法：如果队列为空 take() 将阻塞，直到队列中有内容；如果队列为满 put() 将阻塞，直到队列有空闲位置。
### 使用 BlockingQueue 实现生产者消费者问题

## ForkJoin
主要用于并行计算中，和 MapReduce 原理类似，都是把大的计算任务拆分成多个小任务并行计算。

# 九、线程不安全

# 十、Java内存模型
[深入理解Java内存模型](https://www.jianshu.com/p/15106e9c4bf3)
Java 内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。
## 主内存与工作内存
## 内存模型三大特性
### 1.原子性
Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性


### 2.可见性
可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有三种实现可见性的方式：

- volatile
- synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。

- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

volatile 并不能保证操作的原子性。

### 3.有序性
有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

## 先行先发原则

# 十一、线程安全
## 不可变
Immutable对象一定是线程安全的
- final关键字修饰的基本数据类型
- String
- 枚举类型
- Number部分子类，如Long和Double等数值包装类型，BigInteger和BigDecimal等大数据类型。但是注意，AtomicInteger和AtomicLong是可变的。

对于集合类型可以通过```Collecions.unmodifiableXXX()```方法来获取不可变集合。当对此类集合进行改变操作时，会抛出```UnsupportedOpertationException```

## 互斥同步（悲观锁）
synchronized和ReentrantLock
## 非阻塞同步（乐观锁）
先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。
### 1.CAS
乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。
### 2.AtomicInteger
J.U.C 包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

### 3.ABA问题
如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。

J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。

简单来说就是使用了一个版本号来保证。

## 无同步方案
### 1.栈封闭
因为局部变量存储在虚拟机栈中，属于线程私有的，多个线程访问同一个方法的局部变量时不会出现线程安全问题。

### 2.线程本地存储
如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如“生产者-消费者”模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。

### 3.可重入代码
这种代码也叫做纯代码（Pure Code），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

# 十二、锁优化
## 自旋锁
互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。

在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。
## 锁消除
锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。

## 锁粗化
如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

上一节的示例代码中连续的 append() 方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

## 偏向锁
## 轻量级锁 CAS
## 重量级锁
# 多线程开发良好习惯
给线程起个有意义的名字，这样可以方便找 Bug。

缩小同步范围，从而减少锁争用。例如对于 synchronized，应该尽量使用同步块而不是同步方法。

多用同步工具少用 wait() 和 notify()。首先，CountDownLatch, CyclicBarrier, Semaphore 和 Exchanger 这些同步类简化了编码操作，而用 wait() 和 notify() 很难实现复杂控制流；其次，这些同步类是由最好的企业编写和维护，在后续的 JDK 中还会不断优化和完善。

使用 BlockingQueue 实现生产者消费者问题。

多用并发集合少用同步集合，例如应该使用 ConcurrentHashMap 而不是 Hashtable。

使用本地变量和不可变类来保证线程安全。

使用线程池而不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地利用有限的线程来启动任务。
