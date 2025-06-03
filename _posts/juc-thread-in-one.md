---
name: thread
title: java.lang.Thread
date: 2024-11-20
tags: 
- 并发
categories: juc
---



# 多线程

## 线程概念相关

### Thread

![image-20241119185155274](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119185155274-1735313112574-111.png)

**Parallel Unit 并行最小单位**

- 拥有自己的上下文
- 拥有调用堆栈
- 有自己的 PC
- 但是内存和同一个进程的其他线程共享（SHARED），发生竞态条件（RACE CONDITION）

#### Process

操作系统对执行中程序的一种抽象

进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

在 Java 中，当我们启动 main 函数时其实就是启动了一个 JVM 的进程，而 main 函数所在的线程就是这个进程中的一个线程，也称主线程。

每个程序都有自己的进程，互不干扰。即使它们都是同一份代码，但各自播放的内容和进度都可以不同。进程（可以看成只有一个线程的进程）同时只能做一件事，如果将一个进程分成多个线程，这样就不会浪费时间空等了进程间是完全独立的，互不干扰。而线程则共享同一个进程的资源，所以线程间交换数据更方便，几乎没有通讯损耗。但进程间交换数据就麻烦多了，得通过一些通讯机制，比如管道、消息队列之类的（Inter-Process Communication）

![image-20241119184759519](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241119184759519.png)

#### Coroutine

线程在执行加载视频片段时，必须等待结果返回才能再次执行解码操作，如果引入多线程：加载本身是IO行为，CPU在等待结果返回期间几乎是在空等，浪费了CPU资源。当然，你可以让它休眠以释放 CPU 时间，但创建线程本身就有开销，线程切换同样有开销。

相比之下，协程（Coroutine）非常轻量，创建和切换的开销极小——它并非操作系统层面的东西，就不涉及内核调度。一般是由编程语言来实现（比如 Python 的 asyncio 标准库），它属于用户态的东西。

资源共享问题：线程的执行时机由操作系统调度，程序员无法控制，这正是多线程容易出现资源覆盖的主要原因。而协程的执行时机由程序自身控制，不受操作系统调度影响，因此可以完全避免这类问题。同一个线程内的多个协程共享同一个线程的 CPU 时间片资源，它们在 CPU 上的执行是有先后顺序的，不能并行执行。而线程是可以并行执行的

协程（coroutine），其实是一种特殊的子程序（subroutine，比如普通函数）。普通函数一旦执行就会从头到尾运行，然后返回结果，中间不会暂停。而协程则可以在执行到一半时暂停。利用这一特性，我们可以在遇到 I/O 这类不消耗 CPU 资源的操作时，将其挂起，继续执行其他计算任务，充分利用 CPU 资源。等 I/O 操作结果返回时，再恢复执行。在一个线程内并发执行多个任务

### 为什么要有多线程？

先从总体上来说：

- **从计算机底层来说：** 线程可以比作是轻量级的进程，是程序执行的最小单位,线程间的切换和调度的成本远远小于进程。另外，多核 CPU 时代意味着多个线程可以同时运行，这减少了线程上下文切换的开销。
- **从当代互联网发展趋势来说：** 现在的系统动不动就要求百万级甚至千万级的并发量，而多线程并发编程正是开发高并发系统的基础，利用好多线程机制可以大大提高系统整体的并发能力以及性能。

再深入到计算机底层来探讨：

- **单核时代**：在单核时代多线程主要是为了提高单进程利用 CPU 和 IO 系统的效率。 假设只运行了一个 Java 进程的情况，当我们请求 IO 的时候，如果 Java 进程中只有一个线程，此线程被 IO 阻塞则整个进程被阻塞。CPU 和 IO 设备只有一个在运行，那么可以简单地说系统整体效率只有 50%。当使用多线程的时候，一个线程被 IO 阻塞，其他线程还可以继续使用 CPU。从而提高了 Java 进程利用系统资源的整体效率。
- **多核时代**: 多核时代多线程主要是为了提高进程利用多核 CPU 的能力。举个例子：假如我们要计算一个复杂的任务，我们只用一个线程的话，不论系统有几个 CPU 核心，都只会有一个 CPU 核心被利用到。而创建多个线程，这些线程可以被映射到底层多个 CPU 核心上执行，在任务中的多个线程没有资源竞争的情况下，任务执行的效率会有显著性的提高，约等于（单核时执行时间/CPU 核心数）

### 多线程效率一定高吗？

#### 线程安全问题

众所周知，CPU、内存、I/O 设备的速度是有极大差异的，为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为:

- CPU 增加了 Cache，以均衡与内存的速度差异，导致 `可见性`问题
- 操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异，导致 `原子性`问题
- 编译程序优化指令执行次序，使得 Cache 能够得到更加合理地利用，导致 `有序性`问题

#### 单核 CPU 多线程

对于 IO 密集型，效率明显提高，因为线程切换带来的收益可以抵消代价

对于计算密集型，频繁的上下文切换开销很大

### 易混淆

#### 并发/并行

- 运行的程序就是一个独立的进程
- **并发**：CPU轮询执行每个线程，切换速度快，感觉线程在同时和执行，这就是**并发**
- **并行**：同一时刻有多个线程在被CPU调度执行

#### 同步/异步

- **同步**：发出一个调用之后，在没有得到结果之前， 该调用就不可以返回，一直等待。
- **异步**：调用在发出之后，不用等待返回结果，该调用直接返回。

## Java 线程和 OS 线程

JDK 1.2 之前，Java 线程是基于绿色线程（Green Threads）实现的，这是一种用户级线程（用户线程），也就是说 JVM 自己模拟了多线程的运行，而不依赖于操作系统。由于绿色线程和原生线程比起来在使用时有一些限制（比如绿色线程不能直接使用操作系统提供的功能如异步 I/O、只能在一个内核线程上运行无法利用多核），在 JDK 1.2 及以后，Java 线程改为基于原生线程（Native Threads）实现，也就是说 JVM 直接使用操作系统原生的内核级线程（内核线程）来实现 Java 线程，由操作系统内核进行线程的调度和管理。

我们上面提到了用户线程和内核线程，考虑到很多读者不太了解二者的区别，这里简单介绍一下：

- 用户线程：由用户空间程序管理和调度的线程，运行在用户空间（专门给应用程序使用）。
- 内核线程：由操作系统内核管理和调度的线程，运行在内核空间（只有内核程序可以访问）。

顺便简单总结一下用户线程和内核线程的区别和特点：用户线程创建和切换成本低，但不可以利用多核。内核态线程，创建和切换成本高，可以利用多核。

一句话概括 Java 线程和操作系统线程的关系：**现在的 Java 线程的本质其实就是操作系统的线程**。

线程模型是用户线程和内核线程之间的关联方式，常见的线程模型有这三种：

1. 一对一（一个用户线程对应一个内核线程）
2. 多对一（多个用户线程映射到一个内核线程）
3. 多对多（多个用户线程映射到多个内核线程）

![常见的三种线程模型](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/three-types-of-thread-models.png)

在 Windows 和 Linux 等主流操作系统中，Java 线程采用的是一对一的线程模型，也就是一个 Java 线程对应一个系统内核线程。Solaris 系统是一个特例（Solaris 系统本身就支持多对多的线程模型），HotSpot VM 在 Solaris 上支持多对多和一对一。

## 线程创建、运行、中断

### 三种 Thread 构造器

**<mark>继承 Thread 抽象类<mark>**

1. 创建**子类**继承Thread类，重写 run 方法
2. 创建子类对象`Thread t = new MyThread()`
3. `t.start()` 

- 编码简单，不能继承其他类
- 不要把主线程任务放在启动子线程之前
- 必须调用start方法，调用run方法还是会顺序执行，先启动，启动后会自动调用
- 不能返回执行结果

**<mark>实现 Runnable 接口 任务对象<mark>**

1. 自己创建一个类实现Runnable接口，重写run方法
2. 创建任务对象`Runnable target = new MyRunnable()`
3. 创建线程对象`new Thread(target)`
4. 调用线程对象的start方法

- 只是实现接口，可以继承其他的类，实现其他接口，扩展性强
- 不能返回执行结果

匿名内部类：

```java
//create an annoymous inner class object
Runnable target = new Runnable(){
	@Override
	public void run(){
		...;
        ...;
	}
}
new Thread(target).start();

//Lambda expression
new Thread( ()->{
    ...;
    ...;
}).start();
```

**<mark>实现 Callable 接口<mark>**

1. 定义类实现`Callable<>`，重写call方法，封装要做的事情和返回的数据。`Callable`后边的泛型就是返回的数据类型
2. 创建`Callable`对象，把他封装成`FutureTask<String>(Callable<String> callable)` 对象，泛型要相同
3. `FutureTask`实现了`Runnable`接口，是任务对象
4. 在线程执行完毕之后，对`FutureTask`对象调用`get`方法获取线程执行结果。

```java
Callable<String> callable = new MyCallable(10); //创建 Callable对象

FutureTask<String> futureTask = new FutureTask<>(callable); //用Callable对象 创建 任务对象

new Thread(futureTask).start(); //用任务对象 创建 线程对象 并启动

System.out.println(futureTask.get());//获取返回值
/**
*	如果执行到get()线程还未执行完毕
*	代码会暂停，等待执行完毕才获取结果
*/
```

- 可以获取返回值，扩展性强
- 编码复杂一些

### `thread.start()`

new 一个 `Thread`，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了 Runnable 状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结：调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行。**

在别的平台，比如 C/C++ ，线程创建即运行

### `thread.interrupt()`

如果异常没有被捕获该线程将会停止执行。

1. Java 通过 **`interrupt()`** 发出线程中断信号，**不会强制停止线程**，而是依靠线程自行检查并响应。
2. 使用 **`isInterrupted()`** 查询中断状态，或 **`interrupted()`** 查询并重置状态。
3. 在阻塞操作中，抛出 **`InterruptedException`** 时需恢复中断标志位并及时退出。
4. 遵循协作式终止原则，确保线程安全和资源正确释放。

**推荐方式：**

- 在关键任务中轮询中断状态。
- 在阻塞状态下捕获异常并优雅退出线程。

**如何处理中断**？

1. **响应中断标志位**

```java
while (!Thread.currentThread().isInterrupted()) {
    System.out.println("任务执行中...");
}
System.out.println("检测到中断，退出线程！");
```

**2. 捕获异常并退出**

在阻塞状态下（如 `sleep()`、`wait()`、`join()` 等方法），线程会抛出 **`InterruptedException`**，此时中断标志位会被**自动清除**。示例：

```java
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    System.out.println("线程被中断！");
    Thread.currentThread().interrupt(); // 恢复中断标志位
}
```

**为什么中断不能强制终止线程**？

Java 提倡线程的**协作式终止**，而不是**强制终止**（如 `Thread.stop()` 已被废弃）。原因包括：

1. **强制终止会导致资源泄漏：** 线程可能在持有锁或打开文件等资源时被突然终止，导致资源无法正常释放。
2. **不安全的状态：** 数据可能处于不一致状态，从而引发数据损坏。
3. **可控性低：** 开发者无法自行控制线程的退出逻辑。

## 线程生命周期

![image-20240918105642868](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918105642868-1726751033487-52-1735555863361-53.png)

## Thread 常用方法

  ![image-20240917203637026](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917203637026-1726751033487-46.png)

![image-20240917203429253](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917203429253-1726751033487-47.png)

- **`start()`**: 线程转为 Runnable 可运行状态，相当于 `pthread_create()`
- **`run()`**: 线程应该执行的方法，相当于 `pthread_create()` 传的函数指针
- **`sleep(long millis)`**: 调用者睡眠一段时间
- **`join()`**: 父线程等待子线程执行完毕再继续执行
- **`interrupt()`**: 使线程中断
- `currentThread()`: 获取当前执行的线程对象
- `get/setName()`: 获取线程名称
- `yield()`: 声明当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。只是对线程调度器的一个建议，只是建议具有相同优先级的其它线程可以运行。只保证当前线程放弃CPU占用而不能保证使其它线程一定能占用CPU，执行yield()的线程有可能在进入到暂停状态后马上又被执行。
- `setDaemon()`: 将一个线程设置为守护线程。当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。main() 属于非守护线程。

![Java 线程状态变迁图](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/640-1735555856683-51.png)

 

![image-20240918105918176](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918105918176-1726751033487-54.png)

## 线程安全问题

如果多个线程对同一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的。

以下代码演示了 1000 个线程同时对 cnt 执行自增操作，操作结束之后它的值有可能小于 1000。

```java
public class ThreadUnsafeExample {

    private int cnt = 0;

    public void add() {
        cnt++;
    }

    public int get() {
        return cnt;
    }
}
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    ThreadUnsafeExample example = new ThreadUnsafeExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
// --- 997 // 结果总是小于1000
```

多个线程，同时修改同一个共享资源，可能出现业务安全问题 [Banksim](jetbrains://idea/navigate/reference?project=MyFirstProject&fqn=com.it.threadtest.BankSimulator)

![image-20240917205938036](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917205938036-1726751033487-48.png)

又是一个理论的问题，各式各样的答案有很多，我给出一个个人认为解释地最好的：如果你的代码 在多线程下执行和在单线程下执行永远都能获得一样的结果，那么你的代码就是线程安全的。 这个问题有值得一提的地方，就是线程安全也是有几个级别的：

### 不可变 Immutable

final 关键字修饰的基本数据类型，枚举类型，Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。任何一个线程都改变不了它们的值，要改变除 非新创建一个，因此这些不可变对象不需要任何同步手段就可以直接在多线程环境下使用

### 绝对线程安全

不管运行时环境如何，调用者都不需要任何额外的同步措施。

### 相对线程安全

相对线程安全需要保证对这个对象单独的操作是线程安全的，在调用的时候不需要做额外的保障措施。但是对于一些特定顺序的连续调用，就可能需要在调用端使用额外的同步手段来保证调用的正确性。也就是我们通常意义上所说的线程安全

ConcurrentHashMap, Vector, HashTable, StringBuffer

像 **Vector** 这种，add、remove方法都是原子 操作，不会被打断，但也仅限于此，如果有个线程在遍历某个Vector、有个线程同时在add这个 Vector，99%的情况下都会出现ConcurrentModificationException，也就是fail-fast机制。HashTable也同理。StringBuffer 自带缓冲区，线程安全，性能较低。

<mark>ConcurrentHashMap<mark> 的复合操作：`if(map.containsKey(x)) map.put(x,1);` 不能保证原子性，需要使用内置的原子操作方法，如 `putIfAbsent()`。另外批量操作也不一定线程安全

### 线程不安全 

ArrayList、LinkedList、HashMap等都是线程非安全的类 StringBuilder也不安全。完全不安全

## 线程互斥、同步(Mutex Synchronization)

互斥：多线程只能串行访问某个区域，这个区域就是互斥的区域，比如同步代码块以及lock、信号量

同步：线程之间进行通信，互相配合完成任务。比如notify，await、或者 CyclicBarrier、CountDownLatch

### 悲观、乐观锁

**乐观锁**：CAS算法，共享资源修改之后跟修改之前做对比，如果一样就确认修改，不一样就作废，重新进行修改

主线程里调用子线程的join方法，表示主线程等待次线程执行完毕，再继续执行[赠送礼物案例](jetbrains://idea/navigate/reference?project=MyFirstProject&fqn=com.it.threadtest.GiftSender) 

**悲观锁**：直接把核心代码锁住，只要线程开始执行就加锁，线程安全,但并发性能差

t1和t2两个子线程同时启动，调用start方法，t1，t2进入就绪状态，何时启动，启动顺序由调度器决定，启动以后，main,t1,t2各自执行互不影响，此时如果在子线程的start语句后调用子线程的join方法，t1.join表示主线程要在这一步暂停，直到t1执行完毕，但是t2不会受到任何影响，当t1执行完毕之后，再调用t2.join等待t2执行完毕，可能在t1执行完之前t2就完了，所以有时候t2不join也不会影响结果，但是最好还是加上

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。解决线程安全问题的方案，让多个线程先后依次访问共享资源

### `synchronized`

#### 同步代码块

- 作用：把访问共享资源的核心代码上锁，保证线程安全
- 原理：每次只允许一个线程加锁以后进入，执行完毕自动解锁，其他线程才能进来执行。
- 同步锁必须是同一个锁对象。

关键字 `synchronized` 后边括号里是锁对象，<mark>必须保证是同一个<mark>，最简单的方法就是唯一的 字符串常量池。



![image-20240917214332757](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917214332757-1726751033487-49.png)

不同类或模块中不相关的代码会因为使用相同的字符串常量作为锁而相互干扰，你取钱关我什么事？我的账号我为什么不能取！搜易

`synchronized(this)` 实例方法，锁对象代表正在操作的共享资源

静态方法：`synchronized(Account.class)` 

#### 同步方法

`public synchronized int steal()`

实例方法就是用this作为锁，静态方法的锁：类名.class

web应用的业务层对象service通常是同一个（spring容器的单例）

如果多个线程都要调用service对象的synchronized方法，此时线程A拿到了this对象开始执行方法，其他线程只能阻塞等待A执行完毕释放this对象锁才可继续执行。

锁必须是不同的线程都能访问到的对象，比如字符串常量，比如this

### `ReentrantLock`

`final Lock lk = new ReentrantLock();` 声明在共享资源的成员变量，

`lk.lock()` `lk.unlock()`

- final 关键字 不能修改锁
- 加锁之后，执行核心代码的时候如果遇到异常，后面程序无法执行，就变成了死锁，所以用 `try-catch-finally`

### which one?

**比较**

**1. 锁的实现**

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

**2. 性能**

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

**3. 等待可中断**

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

**4. 公平锁**

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**5. 锁绑定多个条件**

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

**使用选择**

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

### 避免死锁

- Hold and wait（持有并等待）:  一个进程因请求资源而阻塞时，对以获得的资源保持不放。
  - 原子性抢锁，把要抢的锁集中到一起，一块抢了

- No preemption: 进程已获得的资源，在使用完毕之前，不能被强行抢走。（那就只能自己释放）
  - `a.lock()`->`if(!b.tryLock()) then a.unlock()` 
  - 如果 b tryLock 失败就会被抢走。
  - Issue: Livelock，拒绝方

- Circular wait: 循环等待
  - 严格规定抢锁的顺序，fixed order

- Mutual exclusion: 互斥
  - Lock-free CAS
- 无关的业务不要使用相同的锁


## 线程通信(Inter-Thread Communication)

可以通过共享变量的方式实现线程间的通讯和协作：volatile、while 轮询

同时可以使用下面线程消息传递机制：

### `thread.join()`

**<mark>join()<mark>**

join 在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

线程之间互相告知对方自己的状态，相互协调。**避免无效资源争夺** 

![image-20240917222622519](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917222622519-1726751033487-50.png)

### `object.wait()`

**<mark>wait() notify() notifyAll()<mark>**

![image-20240917224325687](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917224325687-1726751033487-51.png)

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateExeception。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。上述方法应该使用当前同步锁对象进行调用，只有锁知道当前占用自己的是哪个线程

先notifyAll 后wait   

先wait 后notifyAll 当前线程等待，notifyAll又再次唤醒了自己

底层逻辑：如果没有通信机制，就会出现无效的资源争夺，我做好了包子，我就应该通知你们所有人，我的工作做完了，你们可以开始你们的工作了，不然我会一直占用桌子但是又做不了包子，你们想吃包子的也因为桌子被占用而吃不上，导致厨师和吃货之间没有必要的资源挤兑与争抢。

#### `wait()` 与 `sleep()`

`wait()` 是让获得对象锁的线程实现等待，会自动释放当前线程占有的对象锁。每个对象（`Object`）都拥有对象锁，既然要释放当前线程占有的对象锁并让其进入 WAITING 状态，自然是要操作对应的对象（`Object`）而非当前的线程（`Thread`）。

类似的问题：**为什么 `sleep()` 方法定义在 `Thread` 中？** 

因为 `sleep()` 是让当前线程暂停执行，不涉及到对象类，也不需要获得对象锁。

**区别**：

- **`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- `wait()` 通常被用于线程间交互/通信，`sleep()`通常被用于暂停执行。
- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()`或者 `notifyAll()` 方法。`sleep()`方法执行完成后，线程会自动苏醒，或者也可以使用 `wait(long timeout)` 超时后线程会自动苏醒。
- `sleep()` 是 `Thread` 类的静态native方法，`wait()` 则是 `Object` 类的native方法。

### `condition.await()`

<mark>**await() signal() signalAll()**<mark>

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。

### Producer/Consumer

**生产者-消费者模式**（Producer-Consumer Pattern）是一种常见的多线程设计模式，通常用于解决**不同速率的线程之间如何协调工作**的问题。它的主要目标是将“**生产数据**”和“**消费数据**”的任务解耦，使用一个共享的缓冲区（通常是队列）来协调生产者和消费者的工作。 

<mark>模式概述<mark>

1. **生产者（Producer）**：负责生成数据，并将数据放入缓冲区或队列中。生产者线程的工作是“生产”，即产生新数据。
2. **消费者（Consumer）**：负责从缓冲区中获取数据并进行处理。消费者线程的工作是“消费”，即处理已经由生产者产生的数据。
3. **共享缓冲区**：生产者和消费者通过共享一个缓冲区（通常是一个线程安全的队列）进行数据传递。生产者将数据放入缓冲区，消费者从缓冲区中取出数据。

在该模式下，生产者和消费者可以独立运行，而不需要直接相互依赖。两者通过共享缓冲区进行**松耦合的通信**。

<mark>关键问题<mark>

生产者和消费者的工作速率可能不同：

- 如果生产者比消费者快，消费者可能一时无法处理所有的数据，导致缓冲区满。
- 如果消费者比生产者快，消费者可能没有数据可消费，导致缓冲区空。

因此，需要有一种机制来处理这两种情况：

- **缓冲区满时**：生产者应该等待，直到有空间可以继续生产数据。
- **缓冲区空时**：消费者应该等待，直到有数据可以消费。

如果在生产者-消费者模式中没有合适的**等待机制**，而生产者和消费者直接对缓冲区进行操作，那么可能会出现以下问题：

1. **缓冲区满时的情况**

当缓冲区已经满了，**生产者**线程如果继续向缓冲区添加数据，而没有等待机制，可能会出现以下两种情况：

- **覆盖数据**：如果没有检查缓冲区是否已满，生产者可能会继续向缓冲区写入数据，从而覆盖掉之前的数据，导致丢失尚未被消费者读取的数据。
- **抛出异常**：在某些实现中，尝试向满的缓冲区写入数据可能会抛出异常，程序会因为未处理该异常而崩溃或出现不可预测的行为。

示例：没有等待机制的生产者向满缓冲区添加数据

```java
public class NoWaitBufferFull {
    private static final int CAPACITY = 5;
    private static int[] buffer = new int[CAPACITY];
    private static int count = 0;

    public static void main(String[] args) {
        // 简单模拟生产者行为
        Thread producer = new Thread(() -> {
            while (true) {
                if (count == CAPACITY) {
                    // 缓冲区已满，没有等待机制，可能出现问题
                    System.out.println("Buffer is full! Data may be lost or overwritten.");
                }
                buffer[count % CAPACITY] = count; // 可能覆盖数据
                count++;
                System.out.println("Produced: " + count);
            }
        });

        producer.start();
    }
}
```

**问题：**

- 在没有等待机制的情况下，生产者线程一旦缓冲区满了，就会继续写入新数据，覆盖掉尚未被消费者读取的数据，从而丢失数据。

2. **缓冲区空时的情况**

当缓冲区为空，**消费者**线程如果继续尝试从缓冲区读取数据，而没有等待机制，可能会出现以下问题：

- **读取无效数据**：消费者从空的缓冲区读取到无效的或未初始化的数据，导致消费错误。
- **抛出异常**：如果没有等待机制，当消费者试图从空的缓冲区获取数据时，可能会抛出空指针异常或数组越界异常，导致程序崩溃。

示例：没有等待机制的消费者从空缓冲区读取数据

```java
public class NoWaitBufferEmpty {
    private static final int CAPACITY = 5;
    private static int[] buffer = new int[CAPACITY];
    private static int count = 0;

    public static void main(String[] args) {
        // 简单模拟消费者行为
        Thread consumer = new Thread(() -> {
            while (true) {
                if (count == 0) {
                    // 缓冲区为空，没有等待机制，可能出现问题
                    System.out.println("Buffer is empty! Consumer may read invalid data.");
                }
                int data = buffer[(count - 1) % CAPACITY]; // 可能读取到无效数据
                System.out.println("Consumed: " + data);
                count--;
            }
        });

        consumer.start();
    }
}
```

**问题：**

- 如果消费者在缓冲区为空时继续尝试读取，可能会读到无效的数据，或者抛出异常。

3. **竞态条件（Race Condition）**

没有等待机制的生产者和消费者可能会产生竞争条件，导致**数据一致性问题**。

- **生产者和消费者竞争访问缓冲区**：生产者在写入数据时，消费者同时读取缓冲区，可能导致消费者读取到部分更新的数据，或者读取到不完整的数据。
- **缓冲区状态不一致**：由于生产者和消费者缺乏同步，缓冲区的状态可能变得不可预测，例如在消费者从空缓冲区读取数据时，生产者已经开始写入数据，导致状态错乱。

正确的等待机制

为了避免上述问题，生产者和消费者应该使用合适的**等待和唤醒机制**，例如：

- **使用阻塞队列（BlockingQueue）**：自动处理缓冲区满和空的情况。
- **使用 `wait()` 和 `notify()`**：手动实现线程间的等待和通知。

这些机制可以保证：

- 当缓冲区满时，生产者会自动等待，直到消费者消费了一些数据并腾出空间。
- 当缓冲区为空时，消费者会自动等待，直到生产者产生新的数据。

总结

如果没有等待机制：

- **缓冲区满时**，生产者可能会覆盖数据或抛出异常。
- **缓冲区空时**，消费者可能会读取到无效数据或抛出异常。
- **竞态条件**会导致数据不一致和程序异常行为。

等待机制可以确保生产者和消费者之间的正确同步，保证数据的完整性和线程安全性。

示例代码：使用 `BlockingQueue

`BlockingQueue` 是 Java 并发包中的线程安全队列，当队列为空时，消费者会阻塞等待；当队列满时，生产者会阻塞等待。

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class ProducerConsumerExample {
    private static final int CAPACITY = 5; // 缓冲区容量
    private static BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(CAPACITY);

    public static void main(String[] args) {
        // 生产者线程
        Thread producer = new Thread(() -> {
            int value = 0;
            while (true) {
                try {
                    System.out.println("Producing: " + value);
                    queue.put(value); // 队列满时生产者阻塞
                    value++;
                    Thread.sleep(1000); // 模拟生产耗时
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        // 消费者线程
        Thread consumer = new Thread(() -> {
            while (true) {
                try {
                    int value = queue.take(); // 队列空时消费者阻塞
                    System.out.println("Consuming: " + value);
                    Thread.sleep(1500); // 模拟消费耗时
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        producer.start();
        consumer.start();
    }
}
```

代码解释

- **`ArrayBlockingQueue`**：这是一个固定容量的阻塞队列，容量为 5。
- **生产者线程**：生产一个整数，将其放入队列中。如果队列满了，`put()` 方法会阻塞，直到队列有空余位置。
- **消费者线程**：从队列中取出一个整数进行消费。如果队列为空，`take()` 方法会阻塞，直到有数据可以消费。
- 生产者每秒生产一个数据，消费者每 1.5 秒消费一个数据，因此队列会周期性地满和空。

运行结果（部分）：

```makefile
Producing: 0
Consuming: 0
Producing: 1
Producing: 2
Consuming: 1
Producing: 3
Producing: 4
Producing: 5
Consuming: 2
...
```

实现的核心优势：

- **线程安全**：`BlockingQueue` 是线程安全的，它自动管理生产者和消费者之间的同步操作。
- **自动阻塞与唤醒**：`BlockingQueue` 的 `put()` 和 `take()` 方法在队列满或空时自动阻塞生产者或消费者线程，不需要手动使用 `wait()` 和 `notify()` 。
- **简单的并发控制**：相比于自己手动编写锁和条件变量，`BlockingQueue` 使得生产者-消费者模式的实现更加简洁和可靠。

使用场景

生产者-消费者模式广泛用于以下场景：

- **多线程任务处理**：例如在消息队列中，生产者产生消息并存放到队列中，消费者从队列中读取并处理消息。
- **流式数据处理**：例如网络服务器中，生产者线程从网络中读取数据，消费者线程对数据进行处理。
- **异步任务队列**：后台线程不断产生任务，工作线程从任务队列中取出任务并执行。

总结

生产者-消费者模式通过解耦生产和消费的过程，提高了系统的**并发性**和**响应速度**，并通过使用共享的缓冲区，确保了线程之间的协调和通信。在 Java 中，通过 `BlockingQueue` 等并发工具，可以非常简便地实现这一模式。

为了更好地理解生产者-消费者模式中缓冲区满、空以及等待机制的问题，可以通过几个生活中的例子来形象化解释。

<mark>案例1：餐厅的厨房和服务员<mark>

**背景**：我们把餐厅的厨房比作生产者，把服务员比作消费者，而“出菜窗口”就是缓冲区。厨师不断做菜（生产者），然后把做好的菜放到窗口。服务员从窗口取菜并送到顾客桌上（消费者）。窗口有固定的空间（比如只能放5盘菜）。

情况一：**没有等待机制，缓冲区满** 

假设窗口只能放5盘菜，但没有任何规则限制厨房什么时候该停止做菜。

- **缓冲区满的情况**：当窗口已经放满了5盘菜时，厨师继续做菜并将新菜强行放到窗口，结果是已经在窗口的菜被挤掉了（覆盖数据），导致有些菜从来没有被服务员送出去（数据丢失）。
- **没有等待的后果**：这样会让顾客拿不到部分菜，而厨师和服务员也浪费了时间。顾客会抱怨因为菜不够，服务员也因为混乱的出菜流程而忙得不可开交。

**现实中的问题**：如果生产者不等待缓冲区腾出空间继续生产，数据就可能会被覆盖，导致丢失。

情况二：**没有等待机制，缓冲区空**

假设服务员到窗口取菜时，没有任何规则限制什么时候该停止等菜。

- **缓冲区为空的情况**：当窗口里一盘菜也没有时，服务员会继续来回在窗口取菜，却发现没有菜可以送（读取无效数据）。服务员一次次白跑，最终浪费时间，不能及时给顾客送餐。
- **没有等待的后果**：服务员会忙碌但却无法送餐，而顾客等待时间过长，抱怨连连。

**现实中的问题**：如果消费者在没有数据可取时不等待，那么它可能会白忙一场，甚至读取无效数据或造成错误。

在这些生活案例中，缓冲区（窗口、传送带、货架）的满和空如果不通过合适的等待机制进行处理，会导致资源浪费、系统效率下降，甚至使整个系统无法正常工作。等待机制的引入可以确保：

- 当缓冲区满时，生产者等待，避免覆盖数据。
- 当缓冲区空时，消费者等待，避免无效操作或消费错误。

通过这些例子可以更直观地理解，**等待机制**在多线程程序中的重要性就在于确保生产者和消费者合理协同，防止资源浪费或错误行为。

## 线程调度 scheduling

Java 的线程调度完全依赖于操作系统。操作系统根据线程优先级、时间片和调度算法来控制线程执行顺序和时间。JVM 负责将 Java 线程映射到操作系统线程，确保线程的创建、调度和同步机制正常工作。Java 提供的优先级和调度方法只是“建议”，最终执行权由操作系统决定。在大多数现代操作系统（如 Linux）中，线程优先级被忽略，使用时间片轮转调度所有线程。

![常见的三种线程模型](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/three-types-of-thread-models-1735555167554-48.png)

### 基于内核线程

Java 中的线程调度依赖于操作系统提供的**原生线程（Native Thread）**支持。具体来说：

1. **Java 线程映射到操作系统线程：**
   - Java 使用 **1:1 模型**，即每个 Java 线程对应一个操作系统线程。
   - 线程的创建、管理和调度全部委托给底层操作系统的线程调度器（如 Windows 的 **线程调度器** 或 Linux 的 **CFS调度器**）。
2. **线程调度策略：**
   操作系统调度器通常采用 **时间片轮转（Round-Robin）** 或 **优先级调度（Priority Scheduling）**策略，为线程分配 CPU 时间。
   - Java 线程也有优先级，但其实际执行顺序依赖于操作系统的实现。
3. **时间片和抢占式调度：**
   - Java 虚拟机（JVM）并不直接管理线程的时间片分配，而是由操作系统决定何时挂起或恢复线程。
   - Java 线程默认是**抢占式调度**，即高优先级线程可能会抢占 CPU 资源，但操作系统仍有最终决定权。

Java 线程通过 JVM 的 **线程库（Thread API）** 调用底层 **操作系统线程库（如 POSIX pthreads 或 Windows API）**来实现线程调度。

Java 线程是基于**内核级线程（Kernel Thread）**，因为：

- Java 线程依赖操作系统的内核调度器进行管理。
- 内核线程直接由操作系统管理和调度，可以利用多核 CPU 并行执行多个线程。
- 用户级线程（如早期的绿色线程）已经被淘汰，Java 不再支持用户级线程。

**关键组件：**

1. **`java.lang.Thread`：** Java 提供的线程抽象类，依赖 JVM 的原生接口。
2. **`Thread.start()`：** 调用 JVM 本地方法 `start0()`，最终委托给操作系统创建线程。
3. **`Thread.yield()`：** 提示线程调度器让出 CPU，但具体是否让出由操作系统决定。
4. **`Thread.sleep()`：** 当前线程进入休眠状态，底层通过操作系统的 **计时器** 和 **休眠 API** 实现。
5. **`Thread.join()`：** 等待其他线程执行完成，依赖于操作系统提供的线程同步机制。

### OS 线程调度

**1) Windows：**

- 使用 **Windows API** 管理线程调度。
- 支持优先级和基于时间片的抢占式调度策略。

**2) Linux/Unix：**

- 使用 **POSIX Threads (pthreads)** 作为底层实现。
- 基于 **CFS（Completely Fair Scheduler）** 调度器执行线程调度，按“公平性”分配 CPU 时间片。

## 线程池 `Executor`

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

- 可以复用线程的技术
- 如果不使用线程池，后台要用新的线程，创建新线程开销很大，请求过多就会严重影响系统性能

- 创建线程池
  - ExecutorService的实现类ThreadPoolExecutor 自创建对象

  - Executors 工具类

- ![image-20240918001837124](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918001837124-1726751033487-53.png)

### 构造 `ThreadPoolExecutor`

```java
public ThreadPoolExecutor(
	int corePoolSize,//核心线程数量
	int maximumPoolSize,//核心线程数量+临时线程数量
	long keepAliveTime,//临时线程存活时间
	TimeUnit unit,  //临时线程存活时间的单位,是一个枚举enum
	BlockingQueue<Runnable> workQueue,//线程池的任务队列
	ThreadFactory threadFactory,//线程池的线程工厂,创建线程
	RejectedExecutionHandler handler//任务拒绝策略, 线程全在忙,任务队列也满了,新任务来了怎么处理
)
```

`workQueue`: `new ArrayBlockingQueue<>(4)` 声明一个新的任务队列,容量为4

`threadFactory`: `Executors.defaultThreadFactory()`

`handler`: `new ThreadPoolExecutor.AbortPolicy()` 丢弃任务抛出异常 默认策略

​		`DiscardPolicy` 丢弃任务不抛异常     `DiscardOldestPolicy` 抛弃队列中等待最久任务

​		`CallerRunsPolicy` 主线程调用`Runnable`任务的`run()`方法从而绕过线程池直接执行

### 处理 `Runnable` 任务

`void execute(Runnable command)`: 处理一个任务

`void shutdown()`: 停机, 但是等待所有任务完成

`List<Runnable> shutdownNow()`: 立即停机, 返回未完成任务的列表

核心线程占满, 再排任务队列, 任务队列占满, 就加临时线程。

### 处理 `Callable` 任务

`Future<T> submit(Callable<T> task)` 返回一个任务对象获取线程结果

```java
Future<String> f1 = pool.submit(new MyCallable(100));
Future<String> f2 = pool.submit(new MyCallable(200));
Future<String> f3 = pool.submit(new MyCallable(300));
Future<String> f4 = pool.submit(new MyCallable(400));
```

### 构造线程池：Executors 工具类

- `public static ExecutorService newFixedThreadPool(int nThreads)`: 创建固定线程数量的线程池，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程替代它。任务队列可达 `Integer.MAX_VALUE` [OOM](https://blog.csdn.net/qq_42447950/article/details/81435080)风险
- `public static ExecutorService newSingleThreadExecutor()`: 创建只有一个线程的线程池对象，如果该线程出现异常而结束，那么线程池会补充一个新线程。任务队列可达 `Integer.MAX_VALUE` [OOM](https://blog.csdn.net/qq_42447950/article/details/81435080)风险
- `public static ExecutorService newCachedThreadPool()`: 线程数量随着任务增加而增加，如果线程任务执行完毕且空闲了60s则会被回收掉。线程数可达 `Integer.MAX_VALUE` [OOM](https://blog.csdn.net/qq_42447950/article/details/81435080)风险
- `public static ScheduledExecutorService newScheduledThreadPool(int corePoolsize)`: 创建一个线程池，可以实现在给定的延迟后运行任务或者定期执行任务。

- 底层都是ThreadPoolExecutor创建的线程池对象

计算密集型：核心线程数量 = CPU核数 + 1

IO密集型：核心线程数量 = CPU核数*2

Executors工具类可能在大型并发系统有危险，内存溢出

## 虚拟线程

[虚拟线程常见问题总结 | JavaGuide](https://javaguide.cn/java/concurrent/virtual-thread.html)