---
name: juc
title: java.util.concurrent
date: 2024-11-30
tags: 
- 并发
categories: juc
---



# Java Concurrency Overview

## [java.lang.Thread](https://scatteredream.github.io/2024/11/20/juc-thread-in-one/) 

1. **线程创建与运行**:

   - **继承 `Thread` 类**: 继承 `Thread` 并重写 `run()` 方法
     - `new MyThread().start()` 
   - **实现 `Runnable` 接口**: 实现 `Runnable` 实现 `run()` 方法
     - `new Thread(runnable).start()` 

2. **线程生命周期**:

   ![Java 线程状态变迁图](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/640.png)

   - **New**: `new Thread()`, 还没有调用 `start()` 
   - **Runnable**: 线程已经调用了 `start()` / 获取锁等待 CPU 调度执行，操作系统层面属于 **Ready** 和 **Running** 状态
   - **Blocked**:  等待获取一个排它锁，如果其线程释放了锁就会结束此状态。
   - **Waiting**: 操作系统的 **Sleep** 状态，等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。需要等待其他线程做出一些特定动作（通知或中断）
   - **Timed Waiting**: 无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。获取锁之后调用 `wait(long ms)` `sleep(long ms)` 可以在指定的时间后自行返回而不是像 WAITING 那样一直等待。
   - **Terminated**: `run()` 执行完毕正常退出或者抛出了未处理的异常

3. **Thread Methods**:

   - **`start()`**: 线程转为 Runnable 可运行状态，相当于 `pthread_create()`
   - **`run()`**: 线程应该执行的方法，相当于 `pthread_create()` 传的函数指针
   - **`sleep(long millis)`**: 调用者睡眠一段时间
   - **`join()`**: 父线程等待子线程执行完毕再继续执行
   - **`interrupt()`**: 使线程中断
   - `currentThread()`: 获取当前执行的线程对象
   - `get/setName()`: 获取线程名称
   - `yield()`: 声明当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。只是对线程调度器的一个建议，只是建议具有相同优先级的其它线程可以运行。只保证当前线程放弃CPU占用而不能保证使其它线程一定能占用CPU，执行yield()的线程有可能在进入到暂停状态后马上又被执行。
   - `setDaemon()`: 将一个线程设置为守护线程。当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。main() 属于非守护线程。

4. 线程安全：

   1. 线程互斥同步：`synchronized` `ReentrantLock` 

   2. 线程非阻塞同步：`AtomicInteger` (CAS)

   3. 无同步：`ThreadLocal` 利用线程各自的栈(FutureTask，线程池)

5. 线程通信与协作：

   - `thread.join()` 父线程与子线程的通信
   - `object.wait()/notify()/notifyAll()` 可用于 synchronized 对象锁
   - `condition.await()/signal()/signalAll()` 可用于 Lock 的条件变量
   - 或者使用共享内存，volatile/while轮询 的形式隐式通信


## `java.util.concurrent`

**<mark>[理论基础](#jvm)<mark>**：

1. **JVM** 对并发的支持：
   - **JMM 内存模型** 解决由于 Cache、指令重排序导致的可见性、有序性问题
   - `synchronized` 用于解决 CPU 时分复用(操作系统调度)导致的原子性问题
     - 偏向锁、轻量级锁与重量级锁，理解锁升级和锁优化机制，如自旋锁与锁消除。
   - `volatile` 用于解决指令重排序与可见性问题
   - `final` 创建不可变对象或常量(线程安全)，内存可见性
   - happens-before 原则
2. 常见**并发设计模式**：

   - **生产者-消费者模式：** 使用阻塞队列（BlockingQueue）优化实现。

   - **读写分离模式：** 提高读写性能，适合数据库访问优化，读写锁、CoW 集合。

   - **线程池模式：** 使用线程池 ThreadPool 统一管理线程资源。

   - **Future 模式：** 提供任务执行结果的异步返回。
3. **死锁检测与避免策略**

**可选：**虚拟线程、Reactor、Disruptor

`java.util.concurrent` 给并发控制提供更多可用的操作:

![formatte](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/formatte.png)

1. **<mark>[Locks](#lock)<mark>**:（悲观锁）

   - **ReentrantLock**: 相同线程可以重复持有同一把锁
   - **ReentrantReadWriteLock**: 有读锁和写锁两部分组成，支持多线程读取和单个线程写入
   - **StampedLock**: 不可重入的读写锁
   - **LockSupport**: 提供线程阻塞同步原语
     - `park()` `unpark(thread)` 
   - **Condition**: 更细粒度的线程同步
   - **AbstractQueuedSynchronizer**: AQS 自定义同步器

2. **<mark>[Tools](#tools)<mark>**(Synchronizers): 和锁配合使用，线程安全工具类

   - **CountDownLatch**: Allows one or more threads to wait until a set of operations being performed in other threads completes. 闭锁是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待
   - **CyclicBarrier**: A barrier that all threads must reach before any thread can proceed.（栅栏） 是因为是一个同步辅助类，允许一组线程互相等待，直到到达某个公共屏障点 ，并且在释放等待线程后可以重用。
   - **Semaphore**: Controls access to a resource by multiple threads. 它的本质是一个“共享锁“。信号量维护了一个信号量许可集。线程可以通过调用 acquire()来获取信号量的许可；当信号量中有可用的许可时，线程能获取该许可；否则线程必须等待，直到有可用的许可为止。 线程可以通过release()来释放它所持有的信号量许可。

3. **<mark>[Atomic Variables](#atomic)<mark>**: CAS Lock-free(乐观锁)

   - **AtomicBoolean**, **AtomicInteger**, **AtomicLong**
   - **AtomicIntegerArray** These classes support lock-free thread-safe programming on single variables using low-level atomic operations.
   - CAS 构建自旋锁
   - ABA 问题与解决方案（如 AtomicStampedReference）。

4. **<mark>[Executor](#executor)<mark>**: 线程池及异步任务相关

   - **Callable**: 和 Runnable 类似，但是有返回值
   - **Future** **FutureTask** 
     - CompletableFuture 异步编程

   - **ExecutorService**: A flexible interface for managing and controlling thread execution.
     - **ThreadPoolExecutor**: 通常所说的线程池

   - **Fork/Join** 框架

5. **Concurrent Collections/Maps**: 线程安全的集合

   ![Concurrent Collections](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/java-concurrent-collections.png)

   ![image](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/java-thread-x-juc-overview-2.png)

   ![Collection Hierarchy concurrent](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/Collection-Hierarchy-concurrent-1735225309564-38.png)

   - **ConcurrentHashMap** 线程安全的哈希表
   - **CopyOnWriteArrayList** CoW List
   - **CopyOnWriteArraySet** CoW Set
   - **BlockingQueue** 阻塞队列

# <span id="jvm">JVM 支持</span>

## 并发问题的根源

### CPU 时分复用

<mark>原子性</mark>：一个过程要么完全执行并且执行的过程不会被任何因素打断，要么就完全不执行。

操作系统基于受限直接执行(Limited Direct Execution)来运行任务，基于 CPU 的时钟中断对任务进行调度，通过这种 CPU 虚拟化技术让应用程序认为是自己在独占 CPU。

```java
int i = 1;
i += 1;// 线程1执行
i += 1;// 线程2执行
/*注意：i += 1 需要三条 CPU 指令
1. 将变量 i 从内存读取到 CPU寄存器；
2. 在CPU寄存器中执行 i + 1 操作；
3. 将最后的结果 i 写入内存（缓存机制导致可能写入的是 CPU 缓存而不是内存）。*/
```

由于CPU分时复用（线程切换）的存在，线程 1 执行了第一条指令后，就切换到线程 2 执行，假如线程 2 执行了这三条指令后，再切换会线程 1 执行后续两条指令，将造成最后写到内存中的 `i` 是 2 而不是 3

> 在 Java 中，可以借助`synchronized`、各种 `Lock` 以及各种原子类实现原子性。
>
> `synchronized` 和各种 `Lock` 可以保证任一时刻只有一个线程访问该代码块，因此可以保障原子性。各种原子类是利用 CAS (compare and swap) 操作（可能也会用到 `volatile`或者`final`关键字）来保证原子操作。

### CPU Cache

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/cpu-cache-protocol.png" alt="缓存一致性协议" style="zoom:67%;" />

<mark>**可见性**<mark>：当一个线程对共享变量进行了修改，那么另外的线程都是立即可以看到修改后的最新值。

1. CPU Cache 缓存的是物理内存数据，用于解决 CPU 处理速度和物理内存不匹配的问题
   - 多核缓存与主内存交互时需要遵守的原则和规范叫做 **缓存一致协议**，如 MESI
2. 应用程序眼中是一片完整的虚拟内存，由操作系统提供内存的虚拟化，将虚拟内存地址映射到真正的物理内存空间中。
   - 操作系统也要解决缓存(比如 TLB)与内存(比如页表)的一致性问题

> 在 Java 中，可以借助`synchronized`、`volatile` 以及各种 `Lock` 实现可见性。
>
> 如果我们将变量声明为 `volatile` ，这就指示 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

### 指令重排序

**<mark>有序性<mark>**：即程序执行的顺序按照代码的先后顺序执行。

为了提升执行速度/性能，计算机在执行程序代码的时候，会对指令进行重排序。 简单来说就是系统在执行代码的时候并不一定是按照你写的代码的顺序依次执行。**指令重排序可以保证串行语义一致，但是没有义务保证多线程间的语义也一致** ，所以在多线程下，指令重排序可能会导致一些问题。

> 在 Java 中，`volatile` 关键字可以禁止指令进行重排序优化。

#### 编译器优化重排

编译器（包括 JVM、JIT 编译器等）在不改变单线程程序语义的前提下，重新安排语句的执行顺序。

对于编译器，禁止重排两句代码的指令，需要在它们之间插入 compiler fence。

#### CPU 优化重排

对于处理器，通过插入内存屏障（Memory Barrier，或有时叫做内存栅栏，Memory Fence，一种 CPU 指令）的方式来禁止特定类型的处理器重排序。另外，为了达到屏障的效果，它也会使处理器写入、读取值之前，将主内存的值写入高速缓存，清空无效队列，从而保障变量的可见性。

[并发编程：乱序执行的那些事儿 - 知乎](https://zhuanlan.zhihu.com/p/413889872)  [图解CPU为何要乱序执行 - last_coding - 博客园](https://www.cnblogs.com/maycap/p/15690751.html)  

##### 指令级并行重排/乱序执行(ILP)

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/2586739-20211214234820966-1713761498.png" alt="img" style="zoom: 80%;" />

RISC 架构的特点就是指令长度相等，执行时间恒定(通常为一个时钟周期)，因此处理器设计起来就很简单，可以通过深长的流水线达到很高的频率，IBM 的 Power6 就可以轻松地达到 4.7GHz 的起步频率。和 RISC 相反，CISC 指令的长度不固定，执行时间也不固定，因此 Intel 的 RISC/CISC 混合处理器架构就要通过 IFheID 将 x86 指令翻译为 μops，从而获得 RISC 架构的长处，提升内部执行效率。x86 指令大部分简单指令可以一对一翻译为 μops，复杂的可能 1 ~ 4 条 μops。解码器是按位数取指的，在经过译码，因此每次可能产生多条 μops。

计算机执行符合局部性原理，这里不仅指同个指令可能重复执行，也指内存访问。而内存访问显然是比较慢的，**对多条指令重新排序，把访存相关的指令放到一起，显然是可以提升效率的。**

##### 内存系统重排

另外，内存系统也会有“重排序”，但又不是真正意义上的重排序。在 JMM 里表现为主内存和线程的本地内存可能不一致，进而导致程序在多线程下执行可能出现问题。

## <mark>Java 内存模型<mark>

并发编程环境下，像 CPU 多级缓存和指令重排这类设计可能会导致程序运行出现一些问题。就比如说我们上面提到的指令重排序就可能会让多线程程序的执行出现问题。

JMM 说白了就是定义了一些规范来解决这些问题，例如 JMM 抽象了 happens-before 原则（后文会详细介绍到）来解决指令重排序问题。开发者可以利用 JMM 规范更方便地开发多线程程序。Java 开发者不需要了解底层原理，直接使用并发相关的一些关键字和类（比如 `volatile`、`synchronized`、各种 `Lock`）即可开发出并发安全的程序。

与 Java 内存区域要区分开：

- JVM 内存结构和 Java 虚拟机的运行时区域相关，定义了 JVM 在运行时如何分区存储程序数据，就比如说**堆主要用于存放对象实例**，栈用来存放局部变量。
- Java 内存模型和 Java 的并发编程相关，抽象了线程和主内存之间的关系就比如说线程之间的共享变量必须存储在主内存中，规定了从 Java 源代码到 CPU 可执行指令的这个转化过程要遵守哪些和并发相关的原则和规范，其主要目的是为了简化多线程编程，增强程序可移植性的。

### 线程与主内存

在当前的 Java 内存模型下，线程可以把变量保存 **本地内存** （比如机器的寄存器）中，而不是直接在主存中进行读写。这就可能造成一个线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成数据的不一致。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/jmm.png" alt="JMM(Java 内存模型)" style="zoom:60%;" />

- **主内存**：所有线程创建的实例对象都存放在主内存中，不管该实例对象是**成员变量**，还是**局部变量**，类信息、**常量**、**静态变量**都是放在主内存中。为了获取更好的运行速度，虚拟机及硬件系统可能会让工作内存优先存储于寄存器和高速缓存中。
- **本地内存**：每个线程都有一个私有的本地内存，本地内存存储了该线程以读 / 写共享变量的副本。每个线程只能操作自己本地内存中的变量，**无法直接访问其他线程的本地内存**。线程间通信必须通过主内存来进行。本地内存是 JMM 抽象出来的一个概念，并不真实存在，它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。图中的线程1想和线程2通信，线程1必须把自己的共享变量副本同步到住内存里，然后线程2需要从主内存读取，读取的共享变量是否是线程1修改过的，是不知道的，由此引发了线程安全问题。
- Java 内存模型定义了八种同步操作，规定了关于主内存与工作内存直接的具体交互协议，即一个变量如何从主内存拷贝到工作内存，如何从工作内存同步到主内存之间的实现细节。规定了一些同步规则来保证这些同步操作的正确执行 [详见 JavaGuide](https://javaguide.cn/java/concurrent/jmm.html#jmm-%E6%98%AF%E5%A6%82%E4%BD%95%E6%8A%BD%E8%B1%A1%E7%BA%BF%E7%A8%8B%E5%92%8C%E4%B8%BB%E5%86%85%E5%AD%98%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB) 

### happens-before 原则

JSR 133 引入了 happens-before 这个概念来描述**两个操作之间的内存可见性**。

happens-before 原则的诞生是为了程序员和编译器、处理器之间的平衡。程序员追求的是易于理解和编程的强内存模型，遵守既定规则编码即可。编译器和处理器追求的是较少约束的弱内存模型，让它们尽己所能地去优化性能，让性能最大化。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20220731155332375-1735372822645-10-1735372984006-14.png" alt="img" style="zoom:70%;" />

- 为了对编译器和处理器的约束尽可能少，只要**不改变程序的执行结果**（单线程程序和正确执行的多线程程序），编译器和处理器怎么进行重排序优化都行。两个操作之间存在 happens-before 关系，并不意味着 Java 平台的具体实现必须要按照 happens-before 关系指定的顺序来执行。如果重排序之后的执行结果，与按 happens-before 关系来执行的结果一致，那么 JMM 也允许这样的重排序。
- 对于会改变程序执行结果的重排序，JMM 要求<u>编译器</u>和<u>处理器</u>必须<mark>禁止<mark>这种重排序。

happens-before 原则表达的意义其实并不是一个操作发生在另外一个操作的前面，虽然这从程序员的角度上来说也并无大碍。更准确地来说，它更想表达的意义是**前一个操作的结果对于后一个操作<mark>是可见的<mark>**，无论这两个操作是否在同一个线程里。

具体规则：1. 按照代码先后顺序 2. 线程的 `start()` 先于其他所有动作 3. 传递性

4. Monitor 的解锁 happens- before 于随后对此 Monitor 的加锁 `synchronized`

5. 对 `volatile` 域的<mark>写<mark>，happens- before 于任意的后续对此 `volatile` 域的<mark>读<mark> 

   - `volatile` 仅保证变量读写操作的可见性和有序性，不保证复合操作（ `i++`）的原子性

   ```java
   int a = 0;
   volatile boolean flag = false;
   // 线程1
   a = 1;        // 普通写
   flag = true;  // volatile 写
   // 线程2
   if (flag) {   // volatile 读
       System.out.println(a); // 一定会输出 1
   }
   ```

   - 保证 `flag = true` 的写入之前，`a = 1` 已经执行完毕，并且对线程 2 可见。
   - 在 JDK 5 之前，由于没有禁止 volatile 指令重排序，`a = 1` 可能会被移动到 `flag = true` 之后执行，导致线程 2 看到 `flag` 为 true，但 `a` 的值仍然是 0。这种情况显然是违背直觉的，也无法确保程序正确性。
   - 在 JDK 5 及之后，`a = 1` 一定会在 `flag = true` 之前执行， `flag = true` 一定在 `if(flag)` 之前执行，从而保证了有序性和内存可见性。

**JMM 与 happens-before**

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20220731084604667.png" alt="happens-before 与 JMM 的关系" style="zoom:67%;" />

程序员在 happens-before 提供的内存可见性基础上编程，JMM 的实现：根据编译器和处理器的重排序规则，如果出现了重排序，除非没有影响执行结果，否则就禁止重排序：为了在不改变程序执行结果的前提下，尽可能地提高程序执行的并行度。实现细节对于程序员是透明的，只要保证程序执行时语义不改变即可。

## <mark>volatile 关键字</mark>

1. **可见性**：`volatile` 关键字并非 Java 语言特有，在 C 语言里也有，其最原始的意义就是禁用 CPU 缓存。如果我们将一个变量使用 `volatile` 修饰，这就告诉编译器这个变量是共享且不稳定的。
   - 当一个线程写入一个 `volatile` 变量时，**JMM 会强制把这个值刷新到主内存**；

   - 当另一个线程读取这个 `volatile` 变量时，**JMM 会强制它从主内存中重新读取值**；

2. **有序性**：在 Java 中，`volatile` 关键字除了可以保证变量的可见性，还有一个重要的作用就是**防止 JVM 的指令重排序**。 如果我们将变量声明为 **`volatile`** ，在对这个变量进行读写操作的时候，会通过插入特定的 **内存屏障** 的方式来禁止指令重排序。（Unsafe 类的内存屏障方法也可以实现 volatile 相同的效果）

3. `volatile` 无法保证原子性。

### 双重校验实现单例：`volatile` + `synchronized`

```java
public class Singleton {
    
    private volatile static Singleton unique;
    
    private Singleton() {}

    public static Singleton getunique() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (unique <mark> null) {
            //类对象加锁(对象锁)
            synchronized (Singleton.class) {
                if (unique <mark> null) {
                    unique = new Singleton();
                }
            }
        }
        return unique;
    }
}
```

对 `unique` 的写操作: `unique = new Singleton();` 可以分成如下三步

1. 为 `unique` 分配内存空间 malloc
2. 初始化 `unique` initialize
3. 将 `unique` 指向分配的引用地址(赋值)

对 `unique` 的读操作: `if (unique == null)` 

`volatile` 使 **写操作的第 3 步** 一定对读操作可见；

但是指令重排仍然会导致一些问题，在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 `getunique`() 后发现 `unique` 不为空，因此返回 `unique`，但此时 `unique` 还未被初始化。`volatile` 能够禁止这种重排。

## <mark>synchronized 关键字</mark>

`synchronized` 是 Java 中的一个关键字，也叫做对象锁，翻译成中文是同步的意思，主要解决的是多个线程之间访问资源的同步性，可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。它是 Java 内置的同步机制，在 JVM 中实现，隐式获取、自动释放。

在 Java 早期版本中，`synchronized` 属于 **重量级锁**，效率低下。这是因为监视器锁（Monitor）是依赖于底层的操作系统的互斥锁 `mutex` 和条件变量 `cond` 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高。

在 Java 6 之后， `synchronized` 引入了大量的优化如**自旋锁**、适应性自旋锁、**锁消除**、**锁粗化**、~~-偏向锁-~~、**轻量级锁**等技术来减少锁操作的开销，这些优化让 `synchronized` 锁的效率提升了很多。因此， `synchronized` 还是可以在实际项目中使用的，像 JDK 源码、很多开源框架都大量使用了 `synchronized` 。

> 由于偏向锁增加了 JVM 的复杂性，同时也并没有为所有应用都带来性能提升。因此，在 JDK 15 中，偏向锁被默认关闭（仍然可以使用 `-XX:+UseBiasedLocking` 启用偏向锁），在 JDK 18 中，偏向锁已经被彻底废弃（无法通过命令行打开）。

- 加在实例方法上，相当于`synchronized(this)`；
- 加在静态方法上，相当于`synchronized(Example.class)`；
- 尽量使用`this`作为对象锁，不要图方便使用字符串常量等

### 底层原理

#### 同步代码块与同步方法

**`synchronized` 同步语句块的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明同步代码块的结束位置。**

含有同步代码块的字节码中包含一个 `monitorenter` 指令以及两个 `monitorexit` 指令，这是为了保证锁在同步代码块代码正常执行以及出现异常的这两种情况下都能被正确释放。

当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 **对象监视器 `monitor`** 的持有权。

> 在 Java 虚拟机(HotSpot)中，Monitor 是基于 C++实现的。每个对象中都内置了一个 `ObjectMonitor` 对象。
>
> 另外，`wait/notify`等方法也依赖于`monitor`对象，这就是为什么只有在同步的块或者方法中才能调用`wait/notify`等方法，否则会抛出`java.lang.IllegalMonitorStateException`的异常的原因。

在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。

对象锁的的拥有者线程才可以执行 `monitorexit` 指令来释放锁。在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放，其他线程可以尝试获取锁。

**同步方法**：`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取而代之的是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。

#### 可重入性 (Reentrancy)

`synchronized` 的可重入性依赖于 **Monitor 对象** 的 **锁计数器** 和 **锁持有线程ID**。

同一个线程每进入一次同步方法或者对象锁相同的同步代码块，就会将锁计数器+1，退出时-1，减到0则释放锁

#### ObjectMonitor

synchronized 是基于管程实现的，核心的数据结构是 ObjectMonitor，AQS也基于MESA管程

**ObjectMonitor 的核心作用**

- 保证同一时刻只有一个线程能执行同步代码块（**互斥**）。
- 提供线程之间的等待和唤醒机制（**条件变量**）。

每个 Java 对象都与一个 对象监视器锁 关联，用于控制对该对象的访问权限。采用 Mesa 语义

**底层机制** 

- **互斥锁（Mutex）**：
  - Monitor 使用操作系统的互斥锁来实现互斥访问。
  - 重量级锁通过内核态的同步原语（如 `futex` 或 `pthread_mutex`）挂起和唤醒线程。
- **条件变量（Condition Variable）**：
  - 等待队列和条件变量用于管理线程状态。`pthread_cond`
  - 条件变量依赖于操作系统的 `wait()` 和 `signal()` 机制，控制线程等待和唤醒。
- **线程阻塞与唤醒**：
  - 当线程无法获取锁时，Monitor 会将其挂起，并调用操作系统的线程调度机制。
  - 被唤醒的线程通过抢占式调度重新竞争锁资源。

### 锁的升级

锁主要有四种状态：无锁、偏向锁、轻量级锁、重量级锁，他们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/3230688-20231101142724469-1226844103.png" alt="img" style="zoom: 80%;" />

**1. 无锁状态（Lock-Free）**

- **特点**：对象未被任何线程锁定，Mark Word 存储对象的哈希码、分代年龄等信息。
- **场景**：对象刚被创建，未被线程访问。

**2. 偏向锁（Biased Locking）**

- **设计初衷**：在大多数情况下，锁总是由同一个线程多次获取，不存在多线程竞争。偏向锁通过消除同步操作来提高性能。
- **加锁过程**：
  1. 当第一个线程访问对象并尝试获取锁时，会在 Mark Word 中记录该线程的 ID（通过 CAS 操作），并将偏向锁标志位设为 1。
  2. 此后该线程再次获取锁时，无需任何同步操作，直接检查 Mark Word 中的线程 ID 是否与自身相同。
- **解锁过程**：偏向锁不会主动释放，只有当其他线程尝试竞争该锁时，持有偏向锁的线程才会释放锁（通过暂停原持有线程，重置 Mark Word）。
- **撤销条件**：当有多个线程竞争同一把锁时，偏向锁会被撤销并升级为轻量级锁。

**3. 轻量级锁（Lightweight Lock）**

- **设计初衷**：在多线程交替执行同步块的情况下，避免重量级锁的性能开销。
- 加锁过程：
  1. 线程在进入同步块时，会在栈帧中创建一个锁记录（Lock Record），并将 Mark Word 复制到锁记录中。
  2. 线程尝试通过 CAS 将 Mark Word 的指针指向锁记录。如果成功，则获取轻量级锁；如果失败，表示有其他线程竞争，锁升级为重量级锁。
- **解锁过程**：通过 CAS 将锁记录中的 Mark Word 复制回对象头。如果失败，表示有竞争，需要唤醒等待的线程。
- **性能开销**：轻量级锁的竞争通过 CAS 操作解决，避免了线程阻塞和唤醒的开销，但如果竞争激烈，频繁的 CAS 操作会消耗 CPU 资源。

**4. 重量级锁（Heavyweight Lock）**

- **设计初衷**：在多线程竞争激烈的情况下，保证线程安全。
- **实现机制**：依赖操作系统的互斥量（Mutex）实现，涉及用户态和内核态的切换，性能开销大。
- **加锁过程**：当轻量级锁竞争失败时，锁会膨胀为重量级锁，线程被挂起进入阻塞状态，等待锁释放后被唤醒。
- **适用场景**：多线程频繁竞争同一把锁的场景。

**对象头中的 Mark Word**

Java 对象在内存中由以下部分组成：

| 内存布局         | 内容                                      |
| ---------------- | ----------------------------------------- |
| 对象头（Header） | 4 字节 Mark Word、4 字节 `.class` Pointer |
| 实例数据         | 实例变量存储的数据                        |
| 对齐填充         | 用于内存对齐，按照8字节填充               |

Mark Word 是对象头中的一部分，存储对象的状态和锁信息。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/443934-20201207134826598-1740849743.png)

**注：锁状态会根据竞争情况自动升级，从偏向锁到轻量级锁，再到重量级锁。** 

### `synchronized` 与 `volatile` 的区别

`synchronized` 关键字和 `volatile` 关键字是两个互补的存在，而不是对立的存在！

- `volatile` 关键字是线程同步的轻量级实现，所以 `volatile`性能肯定比`synchronized`关键字要好 。但是 `volatile` 关键字只能用于变量而 `synchronized` 关键字可以修饰方法以及代码块 。
- `volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。
- `volatile`关键字主要用于解决变量在多个线程之间的可见性，而 `synchronized` 关键字解决的是多个线程之间访问资源的同步性。

## `final` 关键字

[关键字: final详解 | Java 全栈知识体系](https://www.nenggz.com/md/java/thread/java-thread-x-key-final.html) 

用 final 修饰的类不可以被继承，用 final 修饰的方法不可以被覆写，用 final 修饰的属性一旦初始化以后不可以被修改。当然，我们不关心这些段子，这节，我们来看看 final 带来的内存可见性影响。

之前在说双重检查的单例模式的时候，提过了一句，如果所有的属性都使用了 final 修饰，那么 volatile 也是可以不要的，这就是 final 带来的可见性影响。

在对象的构造方法中设置 final 属性，**同时在对象初始化完成前，不要将此对象的引用写入到其他线程可以访问到的地方**（不要让引用在构造函数中逸出）。如果这个条件满足，当其他线程看到这个对象的时候，那个线程始终可以看到正确初始化后的对象的 final 属性。

上面说得很明白了，final 属性的写操作不会和此引用的赋值操作发生重排序，如：

```java
x.finalField = v; ...; sharedRef = x;
```

## 并发安全总结

原子性：atomic、锁

可见性：synchronized volatile final

有序性：锁、volatile、happens before原则

# <mark>ThreadLocal\<T></mark>

`ThreadLocal` 主要用于以下场景：

1. **线程安全的变量管理：** 比如事务管理、用户会话等。
2. **避免同步锁：** 替代传统的锁机制，提高并发性能。
3. **数据库连接或会话缓存：** 每个线程维护自己的数据库连接实例。

T 用于表示 ThreadLocal 存的值类型

ThreadLocal在Spring中发挥着巨大的作用，在管理request作用域中的bean、事务管理、任务调度、aop等模块都出现了它的身影。Spring中绝大部分Bean都可以声明成Singleton作用域，采用ThreadLocal进行封装，因此有状态的bean就能够以 Singleton的方式在多线程中正常工作了。

## 底层机制

**JVM 提供线程隔离：**

- 每个线程都有自己的栈空间和线程私有变量。
- JVM 将 `Thread` 对象与其局部变量绑定，保证变量不被其他线程访问。

**JDK 提供 `ThreadLocal`：** 

- `ThrealLocal` 类中可以通过`Thread.currentThread()`获取到当前线程对象后，直接通过`getMap(Thread t)`可以访问到该线程的`ThreadLocalMap`对象：
- `ThreadLocalMap` 
  - `ThreadLocalMap`存储以`ThreadLocal`为 key ，Object 对象为 value 的键值对。
  - 每次访问 `ThreadLocal` 时，都会从当前线程的 `ThreadLocalMap` 查找对应的value。
  - 不同线程之间的 `ThreadLocalMap` 互不影响，因此保证了变量的线程隔离性。
  - 哈希冲突解决：开放定址法（线性探测）「而 HashMap 采用的是链地址法（拉链法）」

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/2-CFHd4NU8.png)

```java
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;
        Entry(ThreadLocal<?> k, Object v) {
//super(k)：调用父类 WeakReference 的构造方法，把 k 包装成一个弱引用。
            super(k);
            value = v;
        }
    }
}
ThreadLocal<String> tl = new ThreadLocal<>();

ThreadLocal<String> tlWithInitial = ThreadLocal.withInitial(()->return "Hello");


public T get() {
    return get(Thread.currentThread()); // 入参: currentThread()
}
private T get(Thread t) {
    ThreadLocalMap map = getMap(t); // one map per thread
    if (map != null) {
        // 通过 key:tl 获取 entry
        ThreadLocalMap.Entry e = map.getEntry(this); 
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T) e.value; // 获取 value
            return result;
        }
    }
    return setInitialValue(t);
}
public void set(T value) {
    set(Thread.currentThread(), value);
    if (TRACE_VTHREAD_LOCALS) {
        dumpStackIfVirtualThread();
    }
}
private void set(Thread t, T value) {
    // 通过 currentThread 找到对应的map
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value); // key: tl
    } else {
        createMap(t, value);
    }
}
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        if (e.refersTo(key)) {
            e.clear(); // entry 弱引用了threadLocal, 找到就CLEAR, 
            expungeStaleEntry(i);
            return;
        }
    }
}
```



## 内存泄漏：entry 未及时删除

> ThreadLocal 通过访问本线程的 ThreadLocalMap.Entry 进行对 value 的各种操作，如果你在外部失去对 TL 的引用，但是线程生命周期又迟迟未结束（线程池），导致map不会被回收，`thread->map->entry->value`这条引用链一直存在，那么就会有内存泄露的风险（为什么是“风险”？且看后面强引用和弱引用的区别），因此应该养成使用完就及时remove删除条目的习惯，避免一系列的问题。

### 强引用和弱引用

![ThreadLocal各引用间的关系](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/ThreadLocal-01.png)

ENTRY：key 是 ThreadLocal ，弱引用；value 是 Object 对象，强引用。

那么在`ThreadLocal.get()`的时候，发生`GC`之后，`key`是否是`null`？

为了搞清楚这个问题，我们需要搞清楚`Java`的**四种引用类型**：

- **强引用**：我们常常 new 出来的对象就是强引用类型，只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足的时候，宁愿 OutofMemoryError 也不回收。
- **弱引用**：使用 WeakReference 修饰的对象被称为弱引用，只要发生垃圾回收，若这个对象只被弱引用指向，那么就会被回收
- **软引用**：使用 SoftReference 修饰的对象被称为软引用，软引用指向的对象在内存要溢出的时候被回收
- **虚引用**：虚引用是最弱的引用，在 Java 中使用 PhantomReference 进行定义。虚引用中唯一的作用就是用队列接收对象即将死亡的通知

> 什么时候会失去外部对于 TL 的强引用？

1. ThreadLocal 作为类的成员变量，实例对象被置null
2. ThreadLocal 作为类的静态变量，ThreadLocal被置null
3. 方法内部的局部变量，方法结束之后就失去了强引用，比如线程池里提交的任务里的tl局部变量。

### 为什么 Entry 对 TL 弱引用？

> 假设每次使用都忘记remove，那么在失去外部tl引用的情况下，会存在内存泄漏的风险，但是弱引用可以补救。

**假如 ThreadLocal 是强引用**：无法补救内存泄漏

- 如果失去外部强引用（tl是一个run方法里面的局部变量），map这边依然对tl是一个强引用，线程和map也一直存活，entry依然是(tl,value)，但此时已经无法从外部访问value了，那么整个条目也就变成了事实意义上的「强引用」的垃圾，JVM肯定对这些垃圾肯定是束手无策的，从而造成内存泄漏。

**弱引用**：只要线程继续访问tlmap，就能补救。

- 失去强引用，tl就会被自动回收，无用的entry就会变成`(null,value)`。此时map这里有一个helpGC：调用`set()`,`get()`,`remove()` 方法的时候会给当前`ThreadLocalMap`里面`(null,value)`的value置null，相当于还有补救机会
- 但是如果你不用这些方法，又不主动 remove，**依然会造成内存的泄漏。**

## 最佳实践：psf + 手动 remove + Holder工具类

```java
//定义 ThreadLocal 变量
private static final ThreadLocal<Integer> tl = new ThreadLocal<>();
// tl = ThreadLocal.withInitial(() -> 0) 可以设置初始值
tl.set(100); // 当前线程设置值
int value = threadLocal.get(); // 获取当前线程的值
tl.remove(); // 避免内存泄漏
// ------------------------------
public class UserHolder {
    private static final ThreadLocal<UserDTO> tl = new ThreadLocal<>();

    public static void saveUser(UserDTO user){
        tl.set(user);
    }

    public static UserDTO getUser(){
        return tl.get();
    }

    public static void removeUser(){
        tl.remove();
    }
}

```

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241230193230746.png" alt="image-20241230193230746" style="zoom: 80%;" />

### 为什么 TL 建议设置成 static final？

之前分析过，线程提交的任务内部new ThreadLocal但是忘记回收就可能会导致内存泄漏的问题，那我们不妨将其设置为static，这样就保留了ThreadLocal为强引用了，因为他是 GCRoot，永不可能被回收，key（ThreadLocal）是不是弱引用已经无所谓了。

因为这边ThreadLocal是静态变量，所以只存在一个ThreadLocal对象，即使是线程池的线程（假设不发生异常，永远存活），这个ThreadLocal对象也只会占用每个线程的ThreadLocalMap一个Entry，如果我不去主动remove，相比于局部tl泄漏程度不是那么的严重，当然最佳实践还是remove。

### 手动 remove 不仅为了防止内存泄漏

前面说过，remove能够通过删除条目有效防止内存泄漏，这里还有一个好处，如果线程池内的线程一直处于存活状态，并且tl设置成static final，虽然不会发生严重内存泄漏，但是线程执行之前任务留下的threadlocal脏值依然在，之后就可能读到这个脏值。

- Spring 拦截器：对于tomcat服务器的线程池可以采用 filter或者interceptor，在出站的方法里remove。
- try-finally: 手动释放。

## InheritableThreadLocal

这个和ThreadLocal最大的区别就是：可以将父线程的传到子线程（值传递）InheritableThreadLocal是在<mark>new Thread</mark>对象的时候<u>复制</u>父线程的local到子线程的。 因此子线程对自己这个local的remove不会影响到父线程。

复制方式是只管复制，基本数据类型还好，子线程和父线程的改变不互通，值传递最大的问题在于引用：复制了引用的值，这就使得父子仍然使用的是同一个对象，使用的时候应该注意这一点。我们需要重写这个类的childValue方法，把parentValue[深拷贝](https://scatteredream.github.io/2024/10/01/java-se/#Object-的-clone())（json之类的）。

## TransmittableThreadLocal(TTL)

TransmittableThreadLocal 是由阿里开发的一个线程变量传递工具包，解决了InheritableThreadLocal只能再new Thread的时候传递本地变量，之后父线程再对threadLocal做改变，子线程也看不见的问题。可以应用来作链路追踪，传递变量等用途。

# <span id="lock">锁</span>

`java.util.concurrent.locks`

![image-20241230173434255](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241230173434255-1735551283426-42.png)

![image-20241230173500715](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241230173500715.png)

## <mark>`ReentrantLock`<mark> 

`ReentrantLock` 实现了 `Lock` 接口，是一个<mark>可重入<mark>且<mark>独占式<mark>的锁，和 `synchronized` 关键字类似。不过，`ReentrantLock` 更灵活、更强大，增加了**非阻塞**、**超时**、**中断**、**公平锁和非公平锁**等高级功能。

继承关系：实现了 Lock 接口，有一个 Sync 内部类，Sync 继承了 AQS，加锁和释放锁基本在 Sync 中实现，Sync 有公平锁 FairSync 和非公平锁 NonfairSync 两个子类。

![Classes](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/aqs.png)

### 使用

```java
ReentrantLock lock = new ReentrantLock();

lock.lock(); // 获取锁
try {
    // 临界区代码
} finally {
    lock.unlock(); // 释放锁
}
```

基本如上，finally 逻辑一定要释放锁，防止死锁

### 与 `synchronized` 比较

|                 | `synchronized`                    | `ReentrantLock`                                          |
| --------------- | --------------------------------- | -------------------------------------------------------- |
| 实现            | **JVM** 底层关键字                | **JDK** API                                              |
| 公平锁          | 不支持                            | 可显式指定 `new Reentrantlock(true)`                     |
| 多条件/选择通信 | 只支持一个条件变量                | Condition 支持多个条件变量                               |
| 线程通信API     | `wait()` `notify()` `notifyAll()` | `await()` `signal()` `signalAll()` `lock.newCondition()` |
| 可重入          | 支持                              | 支持 `getHoldCount()` 查看重入次数                       |
| 超时等待        | 不支持                            | 支持 `tryLock(timeout)` 超时返回机制                     |
| 释放            | 进出代码块自动完成                | 手动 (`lock()` `unlock()`)                               |
| 中断            | 不可响应中断                      | `lock.lockInterruptibly()`                               |
| 阻塞获取        | 只能阻塞获取                      | 支持非阻塞获取`tryLock()` 失败直接返回                   |

超时等待：防止死锁， 防止线程无限期阻塞

等待可中断：获取锁的线程在阻塞等待的过程中，如果其他线程中断当前线程 `interrupt()` ，就会抛出 `InterruptedException` 异常，可以捕获该异常，做一些处理操作

## 读写锁

包含独占写锁和共享读锁，在读多写少的情况下性能很好，分为可重入`ReentrantReadWriteLock` 和不可重入 `StampedLock` [详见 JavaGuide](https://javaguide.cn/java/concurrent/java-concurrent-questions-02.html#reentrantreadwritelock-%E6%98%AF%E4%BB%80%E4%B9%88) 

ReentrantReadWriteLock 

1、读写锁接口ReadWriteLock接口的一个具体实现，实现了读写锁的分离， 

2、支持公平和非公平，底层也是基于AQS实现 

3、允许从写锁降级为读锁 流程：先获取写锁，然后获取读锁，最后释放写锁；但不能从读锁升级到写锁 

4、重入：读锁后还可以获取读锁；获取了写锁之后既可以再次获取写锁又可以获取读锁 核心：读锁是共享的，写锁是独占的。 读和读之间不会互斥，读和写、写和读、写和写之间才会互斥， 主要是提升了读写的性能

## <span id="tools"><mark>AQS</mark></span> 

[Java并发之AQS详解 - waterystone - 博客园](https://www.cnblogs.com/waterystone/p/4920797.html) 

**AQS（AbstractQueuedSynchronizer）** 是 **Java 并发包**（`java.util.concurrent`）提供的一个**底层同步框架**，是一个抽象类，用来实现锁和同步器。

一般来说，自定义同步器要么是独占方法，要么是共享方式，

- 独占：`tryAcquire-tryRelease`(ReentrantLock)
- 共享：`tryAcquireShared-tryReleaseShared`(Semaphore, CountDownLatch)
- AQS也支持同时实现独占和共享两种方式，如`ReentrantReadWriteLock` 

### 核心机制

#### 数据结构

1. **共享变量 (state)：** 

   - AQS 内部有一个整数 `volatile` 变量 `state`，用来表示当前锁的状态，比如 0 表示空闲，1 表示已占用。
   - 多个线程可以通过 **CAS 操作**来修改这个共享变量，从而实现并发控制。

2. **等待队列 (CLH FIFO队列)：** 

   `Node`: 含有`thread`对象

   ![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/40cb932a64694262993907ebda6a0bfetplv-k3u1fbpfcp-zoom-1.png)

   - 如果线程无法获取共享资源，就进入一个等待队列，这个队列是一个**双向链表**结构。

   `waitStatus`: 

   ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/785ca26c055b5be2761374af6a0c7bc5.webp?x-image-process=image/format,png#pic_center)

   - **CANCELLED**(1)：表示当前结点已取消调度。当timeout或被中断（响应中断的情况下），会触发变更为此状态，进入该状态后的结点将不会再变化。
   - **SIGNAL**(-1)：表示后继结点在等待当前结点唤醒。后继结点入队时，会将前继结点的状态更新为SIGNAL。
   - **CONDITION**(-2)：表示结点等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将**从等待队列转移到同步队列中**，等待获取同步锁。
   - **PROPAGATE**(-3)：共享模式下，前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点。
   - **0**：新结点入队时的默认状态。

   注意，**负值表示结点处于有效等待状态，而正值表示结点已被取消。所以源码中很多地方用>0、<0来判断结点的状态是否正常**。



#### 模板方法: 自定义需要实现的方法

自定义同步器在实现时只需要实现的<u>共享资源state的获取与释放方式</u>即可**，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经实现好了。自定义同步器实现时主要实现以下几种方法：**  

- `isHeldExclusively()`：是否正在独占资源。只有用到condition才需要去实现它。

- `tryAcquire(int i)`：独占方式。尝试获取资源，成功则返回true，失败则返回false。

- `tryRelease(int i`)：独占方式。尝试释放资源，成功则返回true，失败则返回false。

- `tryAcquireShared(int i)`：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。

- `tryReleaseShared(int i)`：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

- 参数 `i` 表示共享资源的个数，`tryAcquire` 就是在更改 `state`

  - 以`ReentrantLock`为例，`tryLock()`就是在非阻塞获取，`lock()`就是真正的获取过程，先`initialTryLock()` 一下，逻辑和`tryLock()`类似这里默认state就是1，因为是Lock自己内部的。如果失败了才真正`acquire(1)`，这里的1就代表资源的个数（锁的个数）只有1，这里才开始重写AQS的内容，`tryAcquire(1)` 开始...


```java
final void lock() {
    if (!initialTryLock())
        acquire(1);
}
```

#### 底层流程（以ReentranLock为例）

[Java并发之AQS详解 - waterystone - 博客园](https://www.cnblogs.com/waterystone/p/4920797.html) 

以 ReentrantLock 中的 Sync 内部类（继承了AQS并实现了模板方法）为例：

0. `lock.lock()` 实际调用 `sync.lock()`
   - 公平锁只能老老实实进入 `acquire(1)` 开始尝试获取。
   - 非公平锁可以在在进入 `acquire(1)` 之前就进行一次CAS操作，不成功再去尝试。

1. 尝试获取（tryAcquire）

   - CAS 尝试快速获取，这里公平锁需要队列为空才能尝试，非公平锁这里直接可以尝试CAS

   - 如果尝试失败，线程会创建一个节点加入队列尾部，并找一个安全点睡眠。

2. 唤醒
   - `lock.unlock()` 实际调用 `sync.release(1)`
   - 前一个线程通过 `tryRelease(1)`释放成功时，会通知队列中的下一个线程 `unparkSuccessor(h)`。
   - 通知机制依赖 `LockSupport.park()` 和 `LockSupport.unpark()` 实现线程通信。

3. 被唤醒的线程继续尝试获取锁，如果成功，则从队列中移除原队头，老二称为新的队头。

##### **CLR 节点**

```java
static final class Node {
    // 等待状态
    static final int SIGNAL = -1; // 通知下一个线程
    static final int CANCELLED = 1; // 已取消 已放弃
    int waitStatus; 

    Node prev; // 前驱节点
    Node next; // 后继节点
    Thread thread; // 当前线程引用
}
```

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/721070-20170504110246211-10684485-1735391533653-31-1735395150328-37.png)

##### [`acquire(int arg)`](https://scatteredream.github.io/2017/10/19/juc-source-code-AQS%20%E8%8E%B7%E5%8F%96%E9%87%8A%E6%94%BE%E8%B5%84%E6%BA%90/#%E7%BA%BF%E7%A8%8B%E6%8A%A2%E9%94%81)

> `acquire(int arg)` Sync从AQS继承而来，同步器想实现语义直接调用这个方法
>
> - `tryAcquire(arg)` 尝试CAS获取
> - `acquireQueued()` 获取失败去排队，

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/721070-20151102145743461-623794326-1735395145015-35-1735395190296-39.png)

如图所示，队头是当前共享资源占用者，AQS保证严格按照入队顺序唤醒，老二被`unpark`之后尝试获取，如果成功自己就是队头，之前的队头将来会自动回收。

```java
public final void acquire(int arg) {
    // 1. CAS  2. CAS 失败入队
    if (!tryAcquire(arg) && 
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

在 acquire() 方法中，当 if 语句的条件返回 true 后，就会调用 selfInterrupt() ，该方法会中断当前线程，为什么需要中断当前线程呢？当 if 判断为 true 时，需要 tryAcquire() 返回 false ，并且 acquireQueued() 返回 true 。其中 acquireQueued() 方法返回的是线程被唤醒之后的 中断状态 ，通过执行 Thread.interrupted() 来返回。

该方法在返回中断状态的同时，会清除线程的中断状态。因此如果 if 判断为 true ，表明线程的中断状态为 true ，但是调用 Thread.interrupted() 之后，线程的中断状态被清除为 false ，因此需要重新执行 selfInterrupt() 来重新设置线程的中断状态。

![Exclusive Acquire](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/aqs-acquire-exclusive-1735447666698-3-1735447685230-5-1735447698634-7.png)

###### CAS 尝试快速获取：`tryAcquire(int arg)`

> `tryAcquire(int arg)` 对status的CAS操作，应该由具体的自定义同步器实现

以ReentrantLock为例：

- 非公平锁(NotFairSync)：`tryAcquire()` 无需考虑队列中是否有前驱节点，前面有人也可以试着CAS抢一下。失败后再排队。

- 公平锁(FairSync)：`tryAcquire()` 只有 `!hasQueuedPredecessors()` 队列为空才能 CAS。

- 当然，除了 CAS 抢锁，如果 state 并不为0，说明是重入的，直接加 state 就好

> `setExclusiveOwnerThread(current);` 设置 AQS 的 exclusiveThread 属性为当前线程，用于判断条件变量。


```java
// static final class NonfairSync extends Sync:
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
// abstract static class Sync extends AQS: 
protected final boolean nonfairTryAcquire(int acquires) {
    if (getState() == 0 && compareAndSetState(0, acquires)) {
        setExclusiveOwnerThread(Thread.currentThread());
        return true;
    }
    return false;
}
//------------------------------------------

// static final class FairSync extends Sync:
protected final boolean tryAcquire(int acquires) {
   if (getState() == 0 && !hasQueuedPredecessors() &&
       compareAndSetState(0, acquires)) {
       setExclusiveOwnerThread(Thread.currentThread());
       return true;
   }
   return false;
}
```

###### <span id="enq">尝试失败，线程入队(先快速后自旋)</span>

> `addWaiter(Node)` 
>
> - 快速插入
>
> - `enq(Node)` 创建节点加入 CLH 队列，返回当前节点

有快速和自旋两个阶段：

- 快速插入就是队列已经初始化了，尝试一次CAS尾插
- 快速插入不成功，进行自旋CAS插入：`enq()` 如果未初始化会先初始化队列

```java
// AQS
private Node addWaiter(Node mode) {
    // 1、将当前线程封装为 Node 节点。
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    // 2、如果 pred ！= null，则证明 tail 节点已经被初始化，直接将 Node 节点加入队列即可。 CAS
    if (pred != null) {
        node.prev = pred;
        // 2.1、通过 CAS 控制并发安全。
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 3、初始化队列，并将新创建的 Node 节点加入队列。
    enq(node);
    return node;
}
// CAS 自旋尾插，返回值是队尾的前驱节点，后面要用到
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) {
            // 1、初始化队列, CAS 
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            // 2、与 addWaiter() 方法中节点入队的操作相同
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

###### 如果前驱是head尝试获取，不是head或者失败，找安全点睡眠

> `boolean acquireQueued(Node, int arg)` 令节点<u>自旋地</u>进行如下操作：前驱节点是head则可以尝试获取，不是则找一个安全点睡眠，只有找到安全点才能睡眠（睡了得有人唤醒），只有抢锁成功才有返回值，返回是否被中断过。
>
> - `shouldParkAfterFailedAcquire(p, node)` 判断是不是安全点：只有前驱节点是 SIGNAL 才算安全点
>
> - `parkAndCheckInterrupt()` 调用 LockSupport 的 park() 进行睡眠，被唤醒（可能是 unpark 也可能是 interrupt）之后检查中断状态，返回是否被中断。

```java
// 
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;//标记是否成功拿到资源
    try {
        boolean interrupted = false;
        for (;;) {
            // 1、如果已经是前驱节点是head，就可以再尝试CAS获取？
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node); // 自己成为新的 head
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 2、判断线程是否处于安全点，若true则park，睡眠...
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        // 3. 详见下文取消等待。
        if (failed)
            cancelAcquire(node); 
    }
}
// 判断当前线程节点是否处于一个安全点: 前驱节点必须是 SIGNAL
// 前面可能有已经cancelled的节点，应该挪到最后一个正常等待的节点后边
// 只有前驱节点 SIGNAL 才能睡，不是的话就把前面的状态应该改成 SIGNAL，返回false表明不是安全点，重新开始流程
// SIGNAL 表明这个节点的后继节点准备睡眠，SIGNAL节点担负着唤醒后继节点的责任
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    int ws = pred.waitStatus;
    // 1、前继节点状态正常，直接返回 true 即可。
    if (ws == Node.SIGNAL)
        return true;
    // 2、ws > 0 表示前继节点的状态异常，即为 CANCELLED 状态，需要跳过异常状态的节点。
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 3、如果前继节点的状态不是 SIGNAL，也不是 CANCELLED，就将状态设置为 SIGNAL。
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
// 如果确实找到了安全点，就在这个方法里面等待唤醒
private final boolean parkAndCheckInterrupt() {
    // 1、线程阻塞到这里 等待unpark()或interrupt()唤醒自己；
    LockSupport.park(this);
    // 2、线程被唤醒之后，返回线程中断状态，并且清除中断状态
    return Thread.interrupted();
}
```



###### <span id="cancel">被中断则取消等待：`cancelAcquire(node)`</span>

> 如果我们要取消一个线程的排队，我们需要在另外一个线程中对其进行中断。
>
> `lock()` 
>
> - `sync.lock() -> acquire(1)`  
>
> - 某线程调用 lock() 很久不返回，想中断它。一旦对其进行中断，此线程会从 `LockSupport.park(this)` 中唤醒，然后 `Thread.interrupted();` 返回 true，`parkAndCheckInterrupt` 返回true，设置 `interrputed = true`。
> - 我们发现一个问题：即使是中断唤醒了这个线程，也就只是设置了 `interrupted = true` 然后继续下一次循环。而且，由于 `Thread.interrupted()`  会清除中断状态，第二次进 `parkAndCheckInterrupt` 的时候，返回会是 false。
> - 所以，我们要看到，在`acquireQueued()`中，interrupted 只是用来记录是否发生了中断，然后用于方法返回值，其他没有做任何相关事情。你中断归中断，我继续抢我的，抢到锁的时候，返回值是 true， `acquire(int arg)`的条件判断完，然后再进行自我中断。
> - <mark>总得来说就是`acquire()` 并不会中断抢锁过程，抢到锁了再自我中断。</mark>
>
> `lockInterruptibly() throws InterruptedException` 
>
> - `sync.acquireInterruptibly(1)` 是AQS内置的可中断获取锁方法
>   - `tryAcquire` CAS 尝试
>   - `doAcquireInterruptibly(arg)` CAS 失败，与acquireQueued()类似，但是这个方法里面，`parkAndCheckInterrupt` 返回true以后会抛出一个 `InterruptedException`，这么一抛，就会走到finally逻辑里面，真正取消获取过程，然后异常会一直抛到最外层，需要 catch 一下。
>
> `cancelAcquire(node)`：抛出异常随后进入 finally 块执行此方法。
>
> [中断是什么？- JavaDoop](https://scatteredream.github.io/2017/10/20/juc-source-code-AQS%20Fair%20Condition%20%E4%B8%AD%E6%96%AD/#%E5%86%8D%E8%AF%B4-java-%E7%BA%BF%E7%A8%8B%E4%B8%AD%E6%96%AD%E5%92%8C-InterruptedException-%E5%BC%82%E5%B8%B8) 

```java
private void cancelAcquire(Node node) {
    // Ignore if node doesn't exist
    if (node == null)
        return;

    node.thread = null;

    // 跳过状态变成 cancelled 的前驱节点
    Node pred = node.prev;
    while (pred.waitStatus > 0)
        node.prev = pred = pred.prev;
    // 把自己的前驱节点变成前面离自己最近的一个正常节点

    // pred 是前面离自己最近的一个正常节点
    Node predNext = pred.next;

    // 这里可以使用无条件写入而不是CAS。
    // 在这个原子步骤之后，其他节点可以跳过我们。在此之前，我们不受其他线程的干扰。
    node.waitStatus = Node.CANCELLED;

    // 如果自己是队尾，就把 pred 设置为队尾
    if (node == tail && compareAndSetTail(node, pred)) {
        compareAndSetNext(pred, predNext, null);
    } else {
        // 如果自己的后继节点需要唤醒，把 pred 的 next 设置成设置成后继结点
        // 并且把 pred 的状态改成 signal 这样的话就算给这个后继找到安全点了
        // 注意上面这些只是CAS，不成功的话就不耗费时间了，直接唤醒他让他自己去找
        int ws;
        if (pred != head && 
            ((ws = pred.waitStatus) == Node.SIGNAL ||
             (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) {
            Node next = node.next;
            if (next != null && next.waitStatus <= 0)
                compareAndSetNext(pred, predNext, next);
        } else {
            unparkSuccessor(node);
        }

        node.next = node; // help GC
    }
}
```

##### `release(int arg)` 

> `release(int arg)` Sync从AQS继承，包括 CAS 资源和操作队列两部分。实现语义就要调用这个

![Release](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/aqs-release.png)

```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;//找到头结点
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);//唤醒等待队列里的下一个线程
        return true;
    }
    return false;
}
```

###### 检查锁是否完全释放：`tryRelease(int arg)`

> `tryRelease(arg)`：自定义同步器实现的方法，如果 state 归零说明是全部释放，锁目前是空闲状态，应该唤醒之后的线程来获取锁。如果不是那就说明是重入的，还不能唤醒。

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

###### 唤醒队头之后首个未放弃正常等待的节点（从后往前找）

> `unparkSuccessor()` AQS的方法，实现对队列的操作，唤醒队头之后第一个未放弃正常等待的节点，如果后继结点正好取消了，那就从后往前找。

```java
private void unparkSuccessor(Node node) {
    //这里，node一般为当前线程所在的结点。
    int ws = node.waitStatus;
    if (ws < 0)//置零当前线程所在的结点状态，允许失败。
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;//找到下一个需要唤醒的结点s
    if (s == null || s.waitStatus > 0) {//如果为空或已取消
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev) 
     		// 从后向前遍历,遍历到head就停止,得到的
            if (t.waitStatus <= 0)
            //从这里可以看出，<=0的结点，都是还有效的结点。
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);//唤醒
}
```

> 为什么是从后往前找？因为 prev 链更加稳定。**从前往后遍历** 可能会因为 `next` 指针失效而漏掉后续的有效节点。`prev` 指针更稳定

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/aqs-addWaiter.png)

- 一个CANCELLED节点，它的 `prev` 指针仍然保持有效，指向的必然是一个有效的节点，但是 `next` 指针会指向他自己（[cancelAcquire(node)#L40](#cancel)）。直到新节点入队才会设置这个节点的next，因此 next 链并不可靠。
- 入队（`addWaiter->enq`）的时候不会检查之前节点的状态是否为 CANCELLED，在CAStail之前，<mark>一定会将原tail 设置成 node.prev</mark> ，<u>只有CAStail成功才会将原tail的next设置为node</u>。这里的组合用法非常巧妙，能保证CAS之前的prev链强一致，但不能保证CAS后的next链强一致。（[enq#L28](#enq)）

#### 共享资源（如Semaphore）

##### `acquireShared(int arg)`

> `void acquireShared(int arg)` Sync从AQS继承，自定义同步器要实现语义要调用此方法。
>
> - `int tryAcquireShared(int acquires)`  用于资源CAS，自定义同步器需要实现这个方法，返回资源数量
> - `void doAcquireShared(int arg)` Sync从AQS继承，用于队列操作。逻辑类似，如果自己前驱不是 head，就找个安全点睡眠，如果是 head就尝试获取资源。
>   - 获取成功，资源有剩余，则`setHeadAndPropagate(node,r)`，继续唤醒之后的节点。

```java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0) // 失败会返回负数，成功返回资源剩余数
        doAcquireShared(arg); // 排队
}
// tryAcquireShared以 semaphore 的 fairsync 实现为例
protected int tryAcquireShared(int acquires) { // 成功返回资源剩余数
    for (;;) {
        if (hasQueuedPredecessors()) // Semaphore 公平锁
            return -1;
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED); // CLH 队列添加，模式为共享
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);  // remaining 
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head;
    // propagate 是资源的剩余量
    setHead(node);//head指向自己node
     //如果还有剩余量，继续唤醒下一个邻居线程
    if (propagate > 0 || h == null || h.waitStatus < 0) {
        Node s = node.next;
        if (s == null || s.isShared())
            doReleaseShared(); // 详见下文的 doReleaseShared
    }
}
```

##### `releaseShared(int arg)`

> `boolean releaseShared(int arg)` Sync从AQS继承，自定义同步器实现语义必须调用此方法
>
> - `boolean tryReleaseShared(int arg)` 用于资源CAS，自定义同步器需要实现这个方法
>
> - `void doReleaseShared()`  Sync从AQS继承，主要是对队列的操作，唤醒之后的节点。

独占模式下的tryRelease()在完全释放掉资源（state=0）后，才会返回true去唤醒其他线程，这主要是基于独占下可重入的考量；而共享模式下的releaseShared()则没有这种要求，共享模式实质就是控制一定量的线程并发执行，那么拥有资源的线程在释放掉部分资源时就可以唤醒后继等待结点。

```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
protected final boolean tryReleaseShared(int releases) {
    for (;;) {
        int current = getState();
        int next = current + releases;
        if (next < current) // overflow
            throw new Error("Maximum permit count exceeded");
        if (compareAndSetState(current, next))
            return true;
    }
}
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;
                unparkSuccessor(h);//唤醒后继
            }
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;
        }
        if (h == head)// head发生变化
            break;
    }
}
```



### AQS 的常见应用

![image](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/java-thread-x-juc-overview-lock.png)

AQS 本身不是直接用来加锁的，而是作为**其他锁工具的基础**。

- **ReentrantLock（可重入锁）：** 用于线程互斥。
  -  `state` 表示重入次数，每lock一次就+1，unlock一次就-1，因此获取和释放要一一对应，否则就会死锁，tryAcquire(1)
- **Semaphore（信号量）：** 控制并发访问的线程数。tryAcquire(n)
- **CountDownLatch（计数器）：** 等待多个线程完成任务。tryAcquire(n)
  -  `state = N`，N 个子线程执行任务，每个子线程执行完后`countDown()` 一次，也就是 CAS 减 1，`state` 归零之后，会`unpark(callerThread)` 主调用线程，主线程从`await` 返回，继续后面的动作。
- **ReadWriteLock（读写锁）：** 支持多个读线程和一个写线程。

### 并发工具（自定义同步器）

#### `Semaphore`

Semaphore 是 synchronized 的加强版，作用是控制线程的并发数量（允许 自定义多少线程同时访问）。就这一点而言，单纯的synchronized 关键字是实现不了的。

Semaphore有一个构造函数， 可以传入一个int型整数n，表示某段代码最多只有n个线程可以访问，如果超出了n，那么请等待， 等到某个线程执行完毕这段代码块，下一个线程再进入。由此可以看出如果Semaphore构造函数中 传入的int型整数n=1，相当于变成了一个synchronized了。

`synchronized` 和 `ReentrantLock` 都是一次只允许一个线程访问某个资源，而`Semaphore`(信号量)可以用来控制同时访问特定资源的线程数量。

Semaphore 的使用简单，我们这里假设有 N(N>5) 个线程来获取 `Semaphore` 中的共享资源，下面的代码表示同一时刻 N 个线程中只有 5 个线程能获取到共享资源，其他线程都会阻塞，只有获取到共享资源的线程才能执行。等到有线程释放了共享资源，其他阻塞的线程才能获取到。

当初始的资源个数为 1 的时候，`Semaphore` 退化为独占锁。

`Semaphore` 有两种模式：公平和非公平 

`Semaphore` 是共享锁的一种实现，它默认构造 AQS 的 `state` 值为 `permits`，你可以将 `permits` 的值理解为许可证的数量，只有拿到许可证的线程才能执行。

调用`semaphore.acquire()` ，线程尝试获取许可证，如果 `state >= 0` 的话，则表示可以获取成功。如果获取成功的话，使用 CAS 操作去修改 `state` 的值 `state=state-1`。如果 `state<0` 的话，则表示许可证数量不足。此时会创建一个 Node 节点加入阻塞队列，挂起当前线程。

调用`semaphore.release();` ，线程尝试释放许可证，并使用 CAS 操作去修改 `state` 的值 `state=state+1`。释放许可证成功之后，同时会唤醒同步队列中的一个线程。被唤醒的线程会重新尝试去修改 `state` 的值 `state=state-1` ，如果 `state>=0` 则获取令牌成功，否则重新进入阻塞队列，挂起线程。

```java
// 初始共享资源数量
final Semaphore semaphore = new Semaphore(5);
// 获取1个许可
semaphore.acquire();
// 释放1个许可
semaphore.release();


public class SemaphoreExample {
  // 请求的数量
  private static final int threadCount = 550;

  public static void main(String[] args) throws InterruptedException {
    // 创建一个具有固定线程数量的线程池对象（如果这里线程池的线程数量给太少的话你会发现执行的很慢）
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    // 初始许可证数量
    final Semaphore semaphore = new Semaphore(20);

    for (int i = 0; i < threadCount; i++) {
      final int threadnum = i;
      threadPool.execute(() -> {// Lambda 表达式的运用
        try {
          semaphore.acquire();// 获取一个许可，所以可运行线程数量为20/1=20
          test(threadnum);
          semaphore.release();// 释放一个许可
        } catch (InterruptedException e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }

      });
    }
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);// 模拟请求的耗时操作
    System.out.println("threadnum:" + threadnum);
    Thread.sleep(1000);// 模拟请求的耗时操作
  }
}
```

#### `CountDownLatch`

##### 原理

`CountDownLatch` 是共享锁的一种实现,它默认构造 AQS 的 `state` 值为 `count`。当线程使用 `countDown()` 方法时,其实使用了`releaseShared()`方法以 CAS 的操作来减少 `state`,直至 `state` 为 0 。

当主线程调用 `await()` 方法的时候，实际是`tryAcquireShared(1)和acquire(1)`: 如果 **state 不为 0**，表示计数器未归零，线程会被**封装成 Node 并加入 AQS 的等待队列**中，并进入 **阻塞状态**。`await()` 方法就会一直阻塞，也就是说 `await()` 方法之后的语句不会被执行。直到 `count` 个线程调用了`countDown()`使 state 值被减为 0，或者调用`await()`的线程被中断，该线程才会从阻塞中被唤醒，`await()` 方法之后的语句得到执行。

```java
public final void acquireSharedInterruptibly(int arg)
    throws InterruptedException { // AQS 继承而来
    if (Thread.interrupted() ||
        (tryAcquireShared(arg) < 0 &&
         acquire(null, arg, true, true, false, 0L) < 0))
        throw new InterruptedException();
}
protected int tryAcquireShared(int acquires) { // 这个是自定义
    return (getState() == 0) ? 1 : -1;
}
// 减-1
public final boolean releaseShared(int arg) { // AQS 继承而来
    if (tryReleaseShared(arg)) {
        doReleaseShared();// SIGNAL
        return true;
    }
    return false;
}
protected boolean tryReleaseShared(int releases) { // 这个是自定义
    // Decrement count; signal when transition to zero
    for (;;) {
        int c = getState();
        if (c == 0)
            return false;
        int nextc = c - 1;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
```

```java
public class CountDownLatchExample {
    // 处理文件的数量
    private static final int threadCount = 6;

    public static void main(String[] args) throws InterruptedException {
        // 创建一个具有固定线程数量的线程池对象（推荐使用构造方法创建）
        ExecutorService threadPool = Executors.newFixedThreadPool(10);
        final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            final int threadnum = i;
            threadPool.execute(() -> {
                try {
                    //处理文件的业务操作
                    //......
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //表示一个文件已经被完成
                    countDownLatch.countDown();
                }

            });
        }
        countDownLatch.await();
        threadPool.shutdown();
        System.out.println("finish");
    }
}
```

##### 用法及注意事项

1. 某一线程在开始运行前等待 n 个线程执行完毕 : 将 `CountDownLatch` 的计数器初始化为 n （`new CountDownLatch(n)`），每当一个任务线程执行完毕，就将计数器减 1 （`countdownlatch.countDown()`），当计数器的值变为 0 时，在 `CountDownLatch 上 await()` 的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。
2. 实现多个线程开始执行任务的最大并行性：注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的 `CountDownLatch` 对象，将其计数器初始化为 1 （`new CountDownLatch(1)`），多个线程在开始执行任务前首先 `coundownlatch.await()`，当主线程调用 `countDown()` 时，计数器变为 0，**多个线程同时被唤醒**。

与 `CountDownLatch` 的第一次交互是主线程等待其他线程。主线程必须在启动其他线程后立即调用 `CountDownLatch.await()` 方法。这样主线程的操作就会在这个方法上阻塞，直到其他线程完成各自的任务。

其他 N 个线程必须引用闭锁对象，因为他们需要通知 `CountDownLatch` 对象，他们已经完成了各自的任务。这种通知机制是通过 `CountDownLatch.countDown()`方法来完成的；每调用一次这个方法，在构造函数中初始化的 count 值就减 1。所以当 N 个线程都调 用了这个方法，count 的值等于 0，然后主线程就能通过 `await()`方法，恢复执行自己的任务。

再插一嘴：`CountDownLatch` 的 `await()` 方法使用不当很容易产生死锁，比如我们上面代码中的 for 循环改为：

```java
for (int i = 0; i < threadCount-1; i++) {
.......
}
```

这样就导致 `count` 的值没办法等于 0，然后就会导致一直等待。

##### `CyclicBarrier`

一个加强版的CountDownLatch。

作用就是会让所有线程都等待完成后才会继续下一步行 动。 CyclicBarrier初始化时规定一个数目，然后计算调用了CyclicBarrier.await()进入等待的线程数。 当线程数达到了这个数目时，所有进入等待状态的线程被唤醒并继续

CyclicBarrier初始时还可带一个Runnable的参数， 此Runnable任务在CyclicBarrier的数目达到 后，所有其它线程被唤醒前被执行。

### `ConditionObject` 条件变量

为什么条件变量需要跟锁绑定？唤醒特定线程，这个线程肯定是因为一个特定的条件休眠的，此时被另一个线程唤醒，一定要保证共享资源是互斥访问的，也就是说被唤醒说明条件确实发生了改变，否则**发生并发安全问题，导致很多的无效唤醒。**  `IllegalMonitorStateException`

> Mesa 管程里面，Thread2 通知完 T1 后，T2 还是会接着执行，T1 并不立即执行，仅仅是从条件变量的等待队列进到入口等待队列里面。这样做的好处是 notify() 不用放到代码的最后，T2 也没有多余的阻塞唤醒操作。但是也有个副作用，就是**当 T1 再次执行的时候，可能曾经满足的条件现在已经不满足了**，所以需要以while循环方式检验条件变量。
>
> Mesa 管程发信号只是一个状态改变的暗示，并不能保证他运行之前的状态一直是期望的情况，线程的 Ready 和 Run 之间的状态转换是由调度程序决定的，`signal` 以后，Run 之前可能状态会发生变化。

```java
ReentrantLock lock = new ReentrantLock();
Condition condition = lock.newCondition();

lock.lock();
try {
    while (!conditionMet()) { // Mesa管程唤醒副作用
        condition.await(); // 等待条件
    }
    // 执行条件满足后的逻辑
} finally {
    lock.unlock();
}

// 在其他线程中：
lock.lock();
try {
    updateCondition();
    condition.signal(); // 通知等待的线程
} finally {
    lock.unlock();
}

```

####  Condition 的等待队列

![Monitor](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/Monitor.png)

`ConditionObject` 维护一个**等待队列**（<mark>单向链表</mark>），其中每个节点是一个 CLH 队列的 `Node` ，节点的 `waitStatus` 被设置为 `COND`，表示属于Condition的等待队列节点。

![condition-2](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/aqs2-2.png)

`Node` 字段：

- `waitStatus`：`COND` 
- `nextWaiter`：指向下个节点。

`ConditionObject` 拥有等待队列的 **头尾指针**：

- `firstWaiter`：指向队列头部（第一个等待线程）。
- `lastWaiter`：指向队列尾部（最后一个等待线程）。

#### 流程

1. 线程调用 `await()` 进入条件队列，<u>同时释放锁</u>(AQS的独占锁实现，比如ReentrantLock)。
   1. 调用 `addConditionWaiter()` 将线程加入Condition条件队列。
   2. 调用 `fullyRelease()` 释放当前线程持有的锁。
   3. 判断线程是否进入 AQS同步队列，如果没有则睡眠线程。
2. 某个线程唤醒在某个条件变量上面等待的其他线程（`signal()` 或 `signalAll()`）。
   1. 检查这个线程是否持有锁。
   2. 调用 `doSignal()` 将条件队列中符合条件的节点移动到AQS同步队列。（从头开始）
3. 被唤醒后，要么是被SyncQueue的前驱节点唤醒了，要么是被中断，反正需要重新获取锁。之后从await 返回，重新判断条件是否符合。（Mesa Sementic）

#### `await()`的睡眠过程

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/979960-20180430095049880-331436771.png)

```java
//把当前线程的节点加入到等待队列中　
//调用await()的线程已经获取锁，所以在加入等待队列后，需要释放锁，并且唤醒后继节点线程
//挂起当前线程，当别的线程调用了signal（），并且是当前线程被唤醒的时候才从返回
//当被唤醒后，该线程会尝试去获取锁，只有获取到了才会从await()返回，否则挂起自己
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter(); // 添加节点到这个condition的等待队列
    int savedState = fullyRelease(node); // 释放当前线程持有的锁,返回重入次数
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) { // 如果不在阻塞队列中，注意了，是阻塞队列
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    // 被唤醒以后，继续去 AQS 同步队列排队，并且需要重新拿到之前的重入次数。
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // 如果等待过程被 cancelled, 清理cancel
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}

```

##### 往条件队列里添加节点

> `Node addConditionWaiter()` 

```java
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters(); // 详见下方
        t = lastWaiter;
    }
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

###### 清理取消的节点 

> <span id="unlinkCancelled">`unlinkCancelledWaiters(node)`</span>

**机制**：通过遍历队列节点检查状态标记，将已取消的节点从队列中断开引用。在入队或者await被cancel的时候调用。

主要是为了在没有signal的情况下能够清理cancel节点。尽管它可能需要一次完整的遍历，但只有在没有信号的情况下在等待发生超时或取消时，它才会发挥作用。它遍历所有节点，而不在一个特定的节点停止，以解除指向垃圾节点的所有指针的链接，而不需要在大规模取消期间进行多次重新遍历（cancellation storms）

```java
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null;
    while (t != null) {
        Node next = t.nextWaiter;
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}
```

##### 完全释放锁

> `int fullyRelease(node)`

```java
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        int savedState = getState();
        if (release(savedState)) {
            failed = false;
            return savedState;
        } else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
```

> 考虑一下这里的 savedState。如果在 condition1.await() 之前，假设线程先执行了 2 次 lock() 操作，那么 state 为 2，我们理解为该线程持有 2 把锁，这里 await() 方法必须将 state 设置为 0，然后再进入挂起状态，这样其他线程才能持有锁。当它被唤醒的时候，它需要重新持有 2 把锁，才能继续下去。
>
> 考虑一下，如果一个线程在不持有 lock 的基础上，就去调用 condition1.await() 方法，它能进入条件队列，但是在上面的这个方法中，由于它不持有锁，release(savedState) 这个方法肯定要返回 false，进入到异常分支，然后进入 finally 块设置 `node.waitStatus = Node.CANCELLED`，这个已经入队的节点之后会被后继的节点”请出去“。

##### 不在AQS队列则睡眠

> `boolean isOnSyncQueue(Node)`

```java
// 在节点入条件队列的时候，初始化时设置了 waitStatus = Node.CONDITION
// 前面我提到，signal 的时候需要将节点从条件队列移到阻塞队列，
// 这个方法就是判断 node 是否已经移动到阻塞队列了
final boolean isOnSyncQueue(Node node) {
    // 移动过去的时候，node 的 waitStatus 会置为 0，这个之后在说 signal 方法的时候会说到
    // 如果 waitStatus 还是 Node.CONDITION，也就是 -2，那肯定就是还在条件队列中
    // 如果 node 的前驱 prev 指向还是 null，说明肯定没有在 阻塞队列(prev是阻塞队列链表中使用的)
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    // 如果 node 已经有后继节点 next 的时候，那肯定是在阻塞队列了
    if (node.next != null) 
        return true;
    
    return findNodeFromTail(node);
}
	// 下面这个方法从阻塞队列的队尾开始从后往前遍历找，如果找到相等的，说明在阻塞队列，否则就是不在阻塞队列
    // 可以通过判断 node.prev() != null 来推断出 node 在阻塞队列吗？答案是：不能。
    // 这个可以看上篇 AQS 的入队方法，首先设置的是 node.prev 指向 tail，
    // 然后是 CAS 操作将自己设置为新的 tail，可是这次的 CAS 是可能失败的。
private boolean findNodeFromTail(Node node) {
    Node t = tail;
    for (;;) {
        if (t == node)
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}
```



#### `signal()`

> `void signal()`

```java
public final void signal() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```

> `boolean isHeldExclusively()` 用来判断当前线程是否持有锁。

##### 将首个正常等待的节点从条件队列移动到 AQS 同步队列

> `doSignal(first)` 将节点从等待队列移动到 AQS 同步队列
>
> - `transferForSignal(first)`  转移工作

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/979960-20180430101906927-1765442903.png)

```java
// 从条件队列队头往后遍历，找出第一个需要转移的 node
// 因为前面我们说过，有些线程会取消排队，但是可能还在队列中
private void doSignal(Node first) {
    do {
        // first的后继变成firstWaiter，如果后继是null, 那么把tail也置null
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        first.nextWaiter = null; // 自此之后就跟条件队列没关系了
        // 这里 while 循环，如果 first 转移不成功
        // 那么选择 first 后面的第一个节点进行转移，依此类推
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
// 将节点从条件队列转移到阻塞队列
// true 代表成功转移
// false 代表在 signal 之前，节点已经取消了
final boolean transferForSignal(Node node) {
    
    // CAS 如果失败，说明节点已经取消
    // 既然已经取消，也就不需要转移了，方法返回，转移后面一个节点
    // 否则，将 waitStatus 置为 0
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;
  
    // enq(node): 自旋尾插到AQS同步队列
    // 注意，这里的返回值 p 是 node 在AQS同步队列的前驱节点
    Node p = enq(node);
    int ws = p.waitStatus;
    // ws > 0 说明 node 在阻塞队列中的前驱节点取消了等待锁，直接唤醒 node 对应的线程。唤醒之后会怎么样，后面再解释
    // 如果 ws <= 0, 那么 compareAndSetWaitStatus 将会被调用，上篇介绍的时候说过，节点入队后，需要把前驱节点的状态设为 Node.SIGNAL(-1)
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        // 如果前驱节点取消或者 CAS 失败，会进到这里唤醒线程，之后的操作看下一节
        LockSupport.unpark(node.thread);
//正常情况下，ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL)
//ws <= 0，而且 compareAndSetWaitStatus(p, ws, Node.SIGNAL) 会返回 true，
//所以一般也不会进去 if 语句块中唤醒 node 对应的线程。
//然后这个方法返回 true，也就意味着 signal 方法结束了，节点进入了AQS同步队列。
    return true;
}
```

##### `signalAll()`

与 `signal()` 类似，只是将所有等待节点依次移动到同步队列并唤醒。`signalAll()` 会唤醒所有线程，但可能导致“惊群效应”（即多个线程争夺锁），需要根据场景合理选择 `signal()` 或 `signalAll()`。

```java
public final void signalAll() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignalAll(first);
}
```

#### `await()`被唤醒

##### 检查中断

[唤醒后检查中断状态)](https://scatteredream.github.io/2017/10/20/juc-source-code-AQS Fair Condition 中断/#唤醒后检查中断状态)

```java
int interruptMode = 0;
while (!isOnSyncQueue(node)) {
    // 线程挂起
    LockSupport.park(this);
    
    // 被唤醒
    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
        break;
}

```

##### 重新按照之前的重入次数获取独占锁

[获取独占锁](https://scatteredream.github.io/2017/10/20/juc-source-code-AQS Fair Condition 中断/#获取独占锁) 

[处理中断状态](https://scatteredream.github.io/2017/10/20/juc-source-code-AQS Fair Condition 中断/#处理中断状态) 

```java
// 被唤醒以后，继续去 AQS 同步队列排队，并且需要重新拿到之前的重入次数。
if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
    interruptMode = REINTERRUPT;
if (node.nextWaiter != null) // 如果等待过程被 cancelled, 清理cancel
    unlinkCancelledWaiters();
if (interruptMode != 0)
    reportInterruptAfterWait(interruptMode);
```

#### 超时等待 `await()` 

[带超时机制的-await](https://scatteredream.github.io/2017/10/20/juc-source-code-AQS Fair Condition 中断/#带超时机制的-await) 

## 死锁排查——`jstack`

jps进程状态工具 **jps.exe 工具是 jdk 自带的，在 %JAVA_HOME%/bin 目录下。**

第一步：打开idea提供terminal终端命令行，使用`jps -l`查看进程
![DeadLock1](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/3039212-20230105200845649-1781815144.png)
第二步：使用`jstack 进程号`查看堆栈信息
![DeadLock2](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/3039212-20230105201438568-23123605.png)
一般情况信息在最后面
![DeadLock3](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/3039212-20230105201125421-886160053.png)

# <span id="atomic">原子变量</span>

一个操作具有原子性，即该操作不可分割、不可中断。即使在多个线程同时执行时，该操作要么全部执行完成，要么不执行，不会被其他线程看到部分完成的状态。

`java.util.concurrent.atomic` 包中的 `Atomic` 原子类提供了一种线程安全的方式来操作单个变量。`Atomic` 类依赖于 CAS（Compare-And-Swap，比较并交换）乐观锁来保证其方法的原子性，而不需要使用传统的锁机制（如 `synchronized` 块或 `ReentrantLock`）。

## <mark>乐观锁与悲观锁<mark>

### 悲观锁

悲观锁总是假设最坏的情况，即一定会发生线程安全问题，因此必须串行访问临界区，`synchronized` 和 `ReentrantLock` 就是悲观锁，高并发的场景下，激烈的锁竞争会造成线程阻塞，大量阻塞线程会导致系统的上下文切换，增加系统的性能开销。并且，悲观锁还可能会存在死锁问题（线程获得锁的顺序不当时），影响代码的正常运行。

### 乐观锁

乐观锁总是假设最好的情况，认为共享资源每次被访问的时候不会出现问题，线程可以不停地执行，无需加锁也无需等待，只是在提交修改的时候去验证对应的资源（也就是数据）是否被其它线程修改了（具体方法可以使用版本号机制或 CAS 算法）。

高并发的场景下，乐观锁相比悲观锁来说，不存在锁竞争造成线程阻塞，也不会有死锁问题，在性能上往往会更胜一筹。但是，如果冲突频繁发生（写占比非常多的情况），会频繁失败并重试，会出现类似活锁的问题，这样同样会非常影响性能，导致 CPU 飙升。

不过，大量失败重试的问题也是可以解决的，像我们前面提到的 `LongAdder`以空间换时间的方式就解决了这个问题。

**理论上**：

- 悲观锁通常多用于写比较多的情况（多写场景，竞争激烈），这样可以避免频繁失败和重试影响性能，悲观锁的开销是固定的。不过，如果乐观锁解决了频繁失败和重试这个问题的话（比如`LongAdder`），也是可以考虑使用乐观锁的，要视实际情况而定。
- 乐观锁通常多用于写比较少的情况（多读场景，竞争较少），这样可以避免频繁加锁影响性能。不过，乐观锁主要针对的对象是单个共享变量（参考`java.util.concurrent.atomic`包下面的原子变量类）。

### 版本号

一般是在数据库表中加上一个数据版本号 `version` 字段，表示数据被修改的次数。当数据被修改时，`version` 值会加一。线程 A 更新数据时读入版本号为 1，此时插入一个线程 B 抢先操作完并提交使版本号更新为 2，线程 A 要提交的时候发现版本号不对，因此重新进行更新操作。

### CAS

#### 原理

CAS 是一条原子操作，依赖于 CPU 的一条指令，在 Java 中由 Unsafe 类(native本地方法类)实现，一共有三个参数：要更新的变量，变量的预期值（旧值），要赋给变量的新值；返回值为CAS是否成功。具体原理参见OSTEP Concurrency 部分，`do-while` 循环也是自旋锁的原理。

#### 问题

##### ABA 问题

如果一个变量 V 初次读取的时候是 A 值，并且在准备赋值的时候检查到它仍然是 A 值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回 A，那 CAS 操作就会误认为它从来没有被修改过。这个问题被称为 CAS 操作的 **"ABA"问题。** 

解决思路是在变量前面追加上**版本号或者时间戳**。JDK 1.5 以后的 `AtomicStampedReference` 类就是用来解决 ABA 问题的，其中的 `compareAndSet()` 方法就是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

##### 自旋开销大

CAS 经常会用到自旋操作来进行重试，也就是不成功就一直循环执行直到成功。如果长时间不成功，会给 CPU 带来非常大的执行开销。

##### 只对单个变量有效

CAS 操作仅能对单个共享变量有效。当需要操作多个共享变量时，CAS 就显得无能为力。不过，从 JDK 1.5 开始，Java 提供了`AtomicReference`类，这使得我们能够保证引用对象之间的原子性。通过将多个变量封装在一个对象中，我们可以使用`AtomicReference`来执行 CAS 操作。

除了 `AtomicReference` 这种方式之外，还可以利用加锁来保证。

## Atomic

基本类型：`AtomicInteger/Long/Boolean`

数组类型：`AtomicXXXArray`  XXX = `Integer/Long/Reference`

引用类型：`AtomicReference` `AtomicStampedReference` 引用类型解决ABA

基本方法就是 get getAndAdd getAndIncrement compareAndSet getAndSet

- `LongAdder`:  消耗内存更多，适合写多读少的情况
- `LongAccumulator`: generalized version of LongAdder, use `LongBinaryOperator` as operations

# <span id="executor">线程池</span>

## `Future<V>` 

![image-20241230173414548](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241230173414548.png)

`Future` 提供了一种轮询/阻塞的方式来获取异步任务的结果，但它不直接支持通知机制，而是需要主动检查任务是否完成。提供方法检查任务是否完成、等待任务完成和获取结果。

是 `submit()`  的返回值，   isDone() 检查是否完成，get() 阻塞等待返回结果

### `Callable`

Callable 接口类似于 Runnable，从名字就可以看出来了，会返回结果，并且可以抛出返回结果的异常，使用 **call()** 方法代替 **run()** 方法，适合需要结果的任务。可以直接提交到线程池

```java
public class FutureCallableExample {
    public static void main(String[] args) {
        // 创建线程池
        ExecutorService executor = Executors.newSingleThreadExecutor();

        // 提交任务
        Callable<Integer> task = () -> {
            Thread.sleep(2000); // 模拟耗时任务
            return 123;
        };

        Future<Integer> future = executor.submit(task);

        try {
            System.out.println("任务是否完成: " + future.isDone());
            Integer result = future.get(); // 阻塞等待结果
            System.out.println("任务结果: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            executor.shutdown();
        }
    }
}

```

### `FutureTask<V>` 

FutureTask 表示一个异步运算的任务，实现了 Runnable 和 Future 接口（RunnableFuture）

FutureTask 通过传入 Callable 的实现类，可以对这个异步运算的任务的结果进行等待获取、判断是否已经完成、取消任务等操作。当然也可以传入 Runnable 的实现类，但是 Runnable 没有返回值，因此需要额外传入一个指定的返回值。V 表示返回值的类型

只有当任务完成的时候结果才能取回，如果<mark>任务尚未完成 `get()` 方法将会阻塞</mark>。

FutureTask 可以 submit 到线程池，还可以可以直接作为 Thread 构造器的参数(Runnable)创建新线程（实践中不建议直接创建新线程）

```java
public class FutureTaskExample {
    public static void main(String[] args) throws Exception {
        Callable<String> callable = () -> {
            Thread.sleep(1000);
            return "任务完成!";
        };

        FutureTask<String> futureTask = new FutureTask<>(callable);
        Thread thread = new Thread(futureTask);
        thread.start();

        // 执行其他任务
        System.out.println("主线程正在执行其他任务...");

        // 等待结果
        String result = futureTask.get(); // 阻塞等待
        System.out.println("执行结果: " + result);
    }
}

```

## `ExecutorService` 

继承了 Executor 并进行一定程度扩展

`shutdown()` 关闭            `submit()` 提交任务

![Executors](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/java-concurrent-executors.png)

### 线程池工作流程：`execute(Runnable command)` (**Runnable ONLY**) 

定义在 **`Executor`** 接口中，只支持提交 **`Runnable`** 类型的任务。无返回值，无法获取任务执行结果。

![图解线程池实现原理](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/thread-pool-principle.png)

![threadPool](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/threadPool.png)

```java
/**********源码************/
/*
 * Proceed in 3 steps:
 *
 * 1. If fewer than corePoolSize threads are running, try to
 * start a new thread with the given command as its first
 * task.  The call to addWorker atomically checks runState and
 * workerCount, and so prevents false alarms that would add
 * threads when it shouldn't, by returning false.
 *
 * 2. If a task can be successfully queued, then we still need
 * to double-check whether we should have added a thread
 * (because existing ones died since last checking) or that
 * the pool shut down since entry into this method. So we
 * recheck state and if necessary roll back the enqueuing if
 * stopped, or start a new thread if there are none.
 *
 * 3. If we cannot queue task, then we try to add a new
 * thread.  If it fails, we know we are shut down or saturated
 * and so reject the task.
 */
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();

    int c = ctl.get();
    // 1. 当前线程数 < corePoolSize，创建核心线程
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 2. 如果corePoolSize已满，将任务放入阻塞队列
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (!isRunning(recheck) && remove(command))
            reject(command); // 检查是否需要拒绝
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 3. 任务队列已满，尝试扩展到 maximumPoolSize
    else if (!addWorker(command, false))
        reject(command); // 执行拒绝策略
}

```

```java
// 用例
ExecutorService executor = Executors.newFixedThreadPool(2);

executor.execute(() -> {
    System.out.println("任务1执行: " + Thread.currentThread().getName());
});
executor.shutdown();
```

#### `submit()` (supports Callable, Runnable)

可以提交 Callable 和 Runnable 任务，返回的是 Future 对象

1. **`Runnable`**：不返回结果的任务。
2. **`Callable<T>`**：<mark>可以返回结果或抛出异常的任务<mark>。
3. **`T result`**：指定任务完成后返回的结果。可以给Runnable人工指定返回值，不指定也行

- 返回 **`Future`** 对象，用于获取任务执行的结果或状态，并且可以方便异常处理。

定义在 **`ExecutorService`** 接口中，由`AbstractExecutorService`实现。支持 **`Runnable`** 和 **`Callable`** 两种任务类型。返回一个 **`Future<T>`** 对象，可以获取任务结果或判断任务状态。

主要的任务就是将 Runnable 或者 Callable 封装成 **FutureTask**，FutureTask 实现了 RunnableFuture（实现了 Runnable）可以直接作为 execute(Runnable command) 的参数，查看返回值本身跟线程池没关系，是 Future 的结果。

```java
Future<?> submit(Runnable task);
<T> Future<T> submit(Runnable task, T result);
Future<T> submit(Callable<T> task);


public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}
public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}

public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```

用例：

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Future<Integer> future = executor.submit(() -> {
    System.out.println("任务2执行: " + Thread.currentThread().getName());
    return 42; // 返回结果
});

try {
    System.out.println("任务结果: " + future.get());
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
executor.shutdown();
```

### Executors 工具类提供的默认线程池

| 线程池类型               | 核心线程数 | 最大线程数 | 特点                                                 |
| ------------------------ | ---------- | ---------- | ---------------------------------------------------- |
| **FixedThreadPool**      | 固定数量   | 固定数量   | 固定线程数，适合长期稳定的任务执行。积压任务排队等待 |
| **CachedThreadPool**     | 0          | 无限制     | 动态扩展线程，适合短期大量任务。60s 回收空闲位置     |
| **SingleThreadExecutor** | 1          | 1          | 单线程执行任务，保证顺序。                           |
| **ScheduledThreadPool**  | 固定数量   | 无限制     | 支持延迟和周期性任务调度。                           |
| **WorkStealingPool**     | CPU 核心数 | CPU 核心数 | 支持并行任务执行和任务窃取算法。                     |

```java
// 固定大小线程池
ExecutorService executor = Executors.newFixedThreadPool(5);
// 可缓存线程池
ExecutorService executor = Executors.newCachedThreadPool();
// 单线程线程池
ExecutorService executor = Executors.newSingleThreadExecutor();
// 定时任务线程池
ScheduledExecutorService pool = Executors.newScheduledThreadPool(3);
// 定时任务
pool.schedule(() -> System.out.println("delay"), 3, TimeUnit.SECONDS);
// 周期任务
pool.scheduleAtFixedRate(() -> System.out.println("周期"), 
                         0, 2, TimeUnit.SECONDS);
```

### 最佳实践

#### 任务提交

- **`execute()`**：只关注任务执行，不关注结果或异常处理。适合无需返回值的任务，只支持Runnable。
- **`submit()`**：既关注任务执行，也关注结果和异常处理，适合需要返回结果的任务，除了Runnable 也支持 Callable，返回 Future，对其调用get()会阻塞当前任务。

**最佳实践：** 优先使用 **`submit()`**，即使不需结果，也可以捕获潜在异常，避免线程池异常崩溃。

#### 异常处理

- **使用`execute()`提交任务**：当任务通过`execute()`提交到线程池并在执行过程中抛出异常时，如果这个异常没有在任务内被捕获，那么该异常会导致当前线程终止，并且异常会被打印到控制台或日志文件中。线程池会检测到这种线程终止，并创建一个新线程来替换它，从而保持配置的线程数不变。
- **使用`submit()`提交任务**：对于通过`submit()`提交的任务，如果在任务执行中发生异常，这个异常不会直接打印出来。相反，异常会被封装在由`submit()`返回的`Future`对象中。当调用`Future.get()`方法时，可以捕获到一个`ExecutionException`。在这种情况下，线程不会因为异常而终止，它会继续存在于线程池中，准备执行后续的任务。

简单来说：使用`execute()`时，未捕获异常导致线程终止，线程池创建新线程替代；使用`submit()`时，异常被封装在`Future`中，线程继续复用。

这种设计允许`submit()`提供更灵活的错误处理机制，因为它允许调用者决定如何处理异常，而`execute()`则适用于那些不需要关注执行结果的场景。

#### <mark>使用自定义线程池 `ThreadPoolExecutor`</mark>  

**避免使用 `Executors` 创建线程池：推荐直接使用`ThreadPoolExecutor` 自定义线程池：因为 **`Executors` 默认队列为 **无限队列**，可能导致内存溢出。除了避免 OOM 的原因之外，不推荐使用 `Executors`提供快捷的线程池的原因：

##### **确定合适的线程数目**

实际使用中需要根据自己机器的性能、业务场景来手动配置线程池的参数比如核心线程数、使用的任务队列、饱和策略等等。

假如 `int N = Runtime.getRuntime().availableProcessors()`

- CPU 密集型任务：N 个线程。任务主要瓶颈在于 CPU 计算能力，与核心数相等的线程数能够最大化 CPU 利用率，过多线程反而会导致竞争和上下文切换开销。
  - "N+1" 的初衷是希望预留线程处理突发暂停，但实际上，处理缺页中断等情况仍然需要占用 CPU 核心。CPU 密集场景下，CPU 始终是瓶颈，预留线程并不能凭空增加 CPU 处理能力，反而可能加剧竞争。
- IO 密集型任务下：M*N 个线程。M约为1.5到2。需要通过测试和监控找到最佳平衡点。

判断任务类型：CPU 密集型简单理解就是利用 CPU 计算能力的任务比如你在内存中对大量数据进行排序。但凡涉及到网络读取，文件读取这类都是 IO 密集型，这类任务的特点是 CPU 计算耗费时间相比于等待 IO 操作完成的时间来说很少，大部分时间都花在了等待 IO 操作完成上。

最佳的线程数目 BestThreads 计算公式为：

![image-20250516214821727](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250516214821727.png)

其中，WT为等待时间，ST为计算时间，他们的和就是计算时间。可以看到，对于CPU密集型来说，N就是结果。对于IO密集型来说，WT应该远大于ST，但是为了避免过于频繁的上下文切换，设置为 1.5-2 即可。WT ST 这些参数我们可以通过 VisualVM 等工具进行检测

###### 数据库连接池

**理想情况下，数据库连接池的大小应该能够满足线程池中同时进行数据库操作的线程的需求。**

**一个常用的经验法则是，将数据库连接池的最大连接数设置为与处理业务请求的线程池的**最大线程数 (`maximumPoolSize`)大致相同，或者至少与核心线程数 (`corePoolSize`)相同

**为什么是最大线程数？** 在高负载时，线程池的线程数可能扩展到最大线程数。如果此时大部分任务都需要数据库连接，那么连接池至少需要提供相应数量的连接才能不成为瓶颈。

**为什么也可能与核心线程数相关？** 如果你确定在高负载时，即使线程数达到 `maximumPoolSize`，也只有一部分线程会同时进行数据库操作，或者数据库操作非常快，连接能快速释放，那么连接池大小可以设置得接近核心线程数，甚至略小于最大线程数。但这需要精确的分析和测试。[详细](#connectpool)

##### 参数详解

建议实践中手动指定线程池配置，ThreadPoolExecutor的构造器共有七个参数：

1. `int corePoolSize`: 核心线程数。

2. `int maxPoolSize`: 最大线程数。

3. `long keepAliveTime`: 非核心线程闲置的最长时间，超时销毁

4. `TimeUnit unit`: 时间单位，枚举

5. `BlockingQueue<Runnable> workQueue`: 保存任务的阻塞队列。
- 如果运行的线程数少于 `corePoolSize`，则执行器始终倾向于添加新线程而不是排队。
  
- 如果 `corePoolSize` 或更多线程正在运行，对请求进行排队而不是添加新线程。
  
- 如果请求无法排队，则会创建一个新线程，如果超出了 `maxPoolSize`，拒绝策略

- `SynchronousQueue`不会保存任务，直接递交给线程，没有就创建，`maxPoolSize`很大，`CachedThreadPool` 就属于这种，创建无限个线程也会OOM。
- `LinkedBlockingQueue`为无界队列，默认最大容量为 `Integer.MAX_VALUE`，可能会导致任务积压 OOM，比如 `FixedThreadPool` 和 `SingleThreadExecutor`
  - 当所有 `corePoolSize` 线程都忙时，使用无界队列（例如没有预定义容量的 `LinkedBlockingQueue`）将导致新任务在队列中等待。
  - 线程最多只有`corePoolSize`，`maxPoolSize`无所谓。
  - 当每个任务完全独立于其他任务时，这可能是合适的，因此任务不会影响彼此的执行；例如，在网页服务器中。虽然这种排队方式对于平滑请求的瞬态突发很有用，但它当命令平均到达速度继续快于处理速度时，可能发生OOM
- `ArrayBlockingQueue`为有界队列，防止OOM：
  - 使用大队列和小池可以最大限度地减少 CPU 使用率、操作系统资源和上下文切换开销，但可能会导致人为降低吞吐量。
  - 使用小队列通常需要更大的池，提高了 CPU 的利用率，但遇到不可接受的调度开销时也会降低吞吐量。
- `DelayedWorkQueue` 用于定时的线程池，无界延迟阻塞队列，最大长度 `Integer.MAX_VALUE` 可能堆积大量的请求，从而导致 OOM。

**<u>Optional</u> Parameters**：

5. `ThreadFactory threadFactory`: 线程工厂，创建新线程，支持自定义线程名称等属性。

- 默认为 `Executors.defaultThreadFactory()` 

6. `RejectedExecutionHandler handler`: 线程池满负荷(队列满且maxPoolSize)的拒绝策略

- `AbortPolicy`(default): 抛出[`RejectedExecutionException`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/RejectedExecutionException.html) 异常
- `CallerRunsPolicy`: 由提交任务的线程直接执行任务，防止任务丢失。这提供了一个简单的反馈控制机制，将减慢新任务提交的速度。
- `DiscardOldestPolicy`: 丢弃最旧未执行的任务，工作队列头部的任务将被删除，然后重试执行（可能会再次失败，导致重复执行）。这种策略很少被接受
- `DiscardPolicy`: 直接丢弃，不抛出任何异常

```java
public class ThreadPoolExecutorDemo {

    private static final int CORE_POOL_SIZE = 5;
    private static final int MAX_POOL_SIZE = 10;
    private static final int QUEUE_CAPACITY = 100;// 队列长度
    private static final Long KEEP_ALIVE_TIME = 1L;
    public static void main(String[] args) {

        //使用阿里巴巴推荐的创建线程池的方式
        //通过ThreadPoolExecutor构造函数自定义参数创建
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                CORE_POOL_SIZE,
                MAX_POOL_SIZE,
                KEEP_ALIVE_TIME,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(QUEUE_CAPACITY),
                new ThreadPoolExecutor.CallerRunsPolicy());
        for (int i = 0; i < 10; i++) {
            //创建WorkerThread对象（WorkerThread类实现了Runnable 接口）
            Runnable worker = new MyRunnable("" + i);
            //执行Runnable
            executor.execute(worker);
        }
        //终止线程池
        executor.shutdown();
        while (!executor.isTerminated()) {
        }
        System.out.println("Finished all threads");
    }
}
```

- 线程池可以使用getter setter等方法进行参数的访问与更改

- 使用 Spring 内部线程池 ThreadPoolTaskExecutor 时，一定要手动自定义，合理配置参数，不然会出现生产问题（一个请求创建一个线程）。


```java
@Configuration
@EnableAsync
public class ThreadPoolExecutorConfig {
    @Bean(name="threadPoolExecutor")
    public Executor threadPoolExecutor(){
        ThreadPoolTaskExecutor threadPoolExecutor = new ThreadPoolTaskExecutor();
        // 返回可用处理器的Java虚拟机的数量
        int processNum = Runtime.getRuntime().availableProcessors(); 
        int corePoolSize = (int) (processNum / (1 - 0.2));
        int maxPoolSize = (int) (processNum / (1 - 0.5));
        threadPoolExecutor.setCorePoolSize(corePoolSize); // 核心池大小
        threadPoolExecutor.setMaxPoolSize(maxPoolSize); // 最大线程数
        threadPoolExecutor.setQueueCapacity(maxPoolSize * 1000); // 队长
        threadPoolExecutor.setThreadPriority(Thread.MAX_PRIORITY);
        threadPoolExecutor.setDaemon(false);
        threadPoolExecutor.setKeepAliveSeconds(300);// 线程空闲时间
        // 线程名字前缀
        threadPoolExecutor.setThreadNamePrefix("test-Executor-"); 
       
        return threadPoolExecutor;
    }
}
```

##### 命名

- 给线程池(实际上是ThreadFactory)命名，有助于定位问题。一般可以使用 Guava：

```java
ThreadFactory threadFactory = new ThreadFactoryBuilder()
                        .setNameFormat(threadNamePrefix + "-%d")
                        .setDaemon(true).build();
ExecutorService threadPool = new ThreadPoolExecutor(corePoolSize,
  maximumPoolSize, keepAliveTime, TimeUnit.MINUTES, workQueue,
 threadFactory);
```

#### 监控线程池状态

除了 SpringBoot-Acuator，还可以通过线程池提供的参数进行监控，以下属性可以通过getter得到：

1. `taskCount：`线程池需要执行的任务数量，包括已经执行完的、未执行的和正在执行的。
2. `completedTaskCount：`线程池在运行过程中**已完成的任务数量**，`completedTaskCount <= taskCount`。
3. `largestPoolSize：`线程池**曾经创建过的最大线程数量**，`通过这个数据可以知道线程池是否满过`。**如等于线程池的最大大小**，则表示`线程池曾经满了`。
4. `poolSize:` 线程池的线程数量。如果`线程池不销毁`的话，`池里的线程不会自动销毁`，所以**线程池的线程数量只增不减**。
5. `activeCount：`获取**活动的**线程数。

我们可以通过一个定时线程池定期监控：

```java
public static void printThreadPoolStatus(ThreadPoolExecutor threadPool) {
    ScheduledExecutorService scheduledExecutorService = new ScheduledThreadPoolExecutor(1, createThreadFactory("print-images/thread-pool-status", false));
    scheduledExecutorService.scheduleAtFixedRate(() -> {
        log.info("=========================");
        log.info("ThreadPool Size: [{}]", threadPool.getPoolSize());
        log.info("Active Threads: {}", threadPool.getActiveCount());
        log.info("Number of Tasks : {}", threadPool.getCompletedTaskCount());
        log.info("Number of Tasks in Queue: {}", threadPool.getQueue().size());
        log.info("=========================");
    }, 0, 1, TimeUnit.SECONDS);
}
```



- 通过**继承线程池**并**重写**线程池的 `beforeExecute`，`afterExecute` 和 `terminated` 方法，我们可以在`任务执行前`，`执行后`和`线程池关闭前`干一些事情。
- 如监控任务的`平均执行时间`，`最大执行时间`和`最小执行时间`等。**这几个方法在线程池里是空方法**，如：

```java
protected void beforeExecute(Thread t, Runnable r) { }
```

#### **适度**复用

- 要<mark>适当复用</mark>线程池，不要一个请求创建一个线程池，浪费资源且效率极低。

- 根据当前业务的情况对线程池进行配置，<mark>不同业务不要复用线程池</mark>：父任务占满线程池，导致子任务阻塞，但是父任务也同时被子任务阻塞，造成互相等待的死锁局面。

<img src="https://oss.javaguide.cn/github/javaguide/java/concurrent/production-accident-threadpool-sharing-deadlock.png" alt="线程池使用不当导致死锁" style="zoom:67%;" />

#### 正确关闭：线程池状态

![在这里插入图片描述](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/dd7189a1bd12f507858a9f3fb95cd2dc.png)

释放资源，shutdown(Now)不过只是异步通知，不会同步阻塞等待。

- **`void shutdown()`** 有序关闭，执行完先前提交的任务。对应 SHUTDOWN
- **`List<Runnable> shutdownNow()`** 停止所有正在执行的任务，停止正在等待的任务的处理，并返回正在等待执行的任务的列表。对应 STOP
- `awaitTermination` 同步阻塞等待。
- 实现了autoclosable，close方法可以阻塞等待，因此trywithresource。

#### 不要和 JDK 自带 ThreadLocal 共用

这是因为线程池会复用线程对象，与线程对象绑定的类的静态属性 `ThreadLocal` 变量也会被重用，这就导致一个线程可能获取到其他线程的`ThreadLocal` 值。

#### 不要放入耗时任务

线程池本身的目的是为了提高任务执行效率，避免因频繁创建和销毁线程而带来的性能开销。如果将耗时任务提交到线程池中执行，可能会导致线程池中的线程被长时间占用，无法及时响应其他任务，甚至会导致线程池崩溃或者程序假死。这些任务可以采用异步 `CompletableFuture` 完成

### <span id="connectpool"><mark>线程池参数调优</mark></span>

[Java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html) 

#### 基本原则

##### 基于 CPU 与 IO 比例的线程数公式（WT/ST、目标利用率）

对于 IO 密集型任务（如数据库调用），线程在等待 IO 完成时不会占用 CPU。Goetz 建议：

> 线程数 = N × 目标利用率 × (1 + WT/ ST)

其中 WT 是线程等待 IO 的时间，ST 是实际执行业务逻辑的时间。计算密集型任务（WT ≈ 0）时，线程数 ≈ CPU 核心数；IO 密集型任务时，线程数会大幅 > CPU 核心数，以充分利用等待期间的空闲 CPU。

> [别再纠结线程池大小/线程数量了，没有固定公式的](https://juejin.cn/post/6948034657321484318) 
>
> 如果我期望目标利用率为90%（多核90），那么需要的线程数为：
>
> 核心数12 * 利用率0.9 * (1 + 50(sleep时间)/50(循环50_000_000耗时)) ≈ 22 个线程
>
> 结果现在CPU利用率大概80+，和预期比较接近了，由于线程数过多，还有些上下文切换的开销，再加上测试用例不够严谨，所以实际利用率低一些也正常。
>
> 比如一个普通的，SpringBoot 为基础的业务系统，默认Tomcat容器+HikariCP连接池+G1回收器，如果此时项目中也需要一个业务场景的多线程（或者线程池）来异步/并行执行业务流程。
>
> 此时按照上面的公式来规划线程数的话，误差一定会很大。因为此时这台主机上，已经有很多运行中的线程了，Tomcat有自己的线程池，HikariCP也有自己的后台线程，JVM也有一些编译的线程，连G1都有自己的后台线程。这些线程也是运行在当前进程、当前主机上的，也会占用CPU的资源。
>
> 所以受环境干扰下，单靠公式很难准确的规划线程数，一定要通过测试来验证。
>
> 流程一般是这样：
>
> 1. 分析当前主机上，有没有其他进程干扰
> 2. 分析当前JVM进程上，有没有其他运行中或可能运行的线程
> 3. 设定目标
>    1. 目标CPU利用率 - 我最高能容忍我的CPU飙到多少？
>    2. 目标GC频率/暂停时间 - 多线程执行后，GC频率会增高，最大能容忍到什么频率，每次暂停时间多少？
>    3. 执行效率 - 比如批处理时，单位时间内要开多少线程才能及时处理完毕
>    4. ……
> 4. 梳理链路关键点，是否有卡脖子的点，因为如果线程数过多，链路上某些节点资源有限可能会导致大量的线程在等待资源（比如三方接口限流，连接池数量有限，中间件压力过大无法支撑等）
> 5. 不断的增加/减少线程数来测试，按最高的要求去测试，最终获得一个“满足要求”的线程数

##### Little’s Law（利特尔规则）

排队论中的 Little’s Law 给出了并行处理能力与系统延迟、吞吐率之间的关系：

> 并发线程数 L = 系统吞吐率 λ × 平均延迟 W
>
> – L：系统中并发处理的请求数（即线程数）
> – λ：系统的长时平均到达率（请求数/秒）
> – W：请求的平均处理时间（秒） ([Zalando Engineering Blog](https://engineering.zalando.com/posts/2019/04/how-to-set-an-ideal-thread-pool-size.html?utm_source=chatgpt.com))

例如，要保持 100 QPS (λ=100) 的吞吐并且响应时间 W=0.01 s 时，需要 L=1 个并发线程才能稳定达到目标 ([Zalando Engineering Blog](https://engineering.zalando.com/posts/2019/04/how-to-set-an-ideal-thread-pool-size.html?utm_source=chatgpt.com))

##### 数据库连接池的约束

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1911189-20200407003006504-308847843.jpg)

- Druid主要由数组，ReentrantLock和两个信号量empty和notEmpty组成。
- 数组是存储连接的容器，每一次客户端线程请求连接时，从数组最后一个位置取，创建连接，由专门的创建连接的线程负责，销毁连接由专门的销毁连接的线程负责。
- 客户端线程，创建连接线程，销毁线程通过lock和两个condition协调下共同协作工作。

连接池的本质是属于一个操作系统进程（process）的计数信号量（counting Semaphore），用于控制可以并行使用数据库连接的线程数量。在 Java SDK 有一个[Semaphore (Java Platform SE 8 )](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Semaphore.html) 可以用来管理各种有限数量的资源。连接池的核心管理功能是从池中分配一个数据库连接给需要的线程，线程用完后回收连接到池中。由于连接池有限，可以并行进行数据库访问的线程数量最多是连接池的最大尺寸。如果所有线程都在等待数据库响应，且连接池最大并发连接数为 N，则有效并发线程数不应超过 N，否则线程会在应用层排队，造成不必要的上下文切换和内存占用 ([红帽文档](https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/24.0/html/high_availability_guide/concepts-threads-?utm_source=chatgpt.com), [Oracle 文档](https://docs.oracle.com/cd/E19900-01/819-4742/abehg/index.html?utm_source=chatgpt.com))。

连接池的使用者是业务应用程序。通常有二种：一种是基于用户/服务请求的 HTTP 服务线程，通常采用线程池。特点是线程数目动态变化很大，数据库的访问模式比较多样，处理时间也有长有短，可能有很大差别。另一种是后台服务，其线程数目比较固定，数据库访问模式和处理时间也比较稳定。连接池只是给业务应用提供已建立的连接，所有的访问请求都通过连接转发到后台数据库服务器。数据库服务器通常也采用线程（PostgreSQL 每个连对应一个进程）池处理所有的访问请求。因此，线程池中的一个线程使用连接池中的一条连接和数据库的服务端的一个线程进行通信。具体配置可见：[如何配置数据库连接池](https://juejin.cn/post/6844903850630086669#heading-5) 

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1911189-20200407004610684-971276319.jpg)

**maxSize ：** 我们需要将系统部署到一个单节点上，然后对其进行压测，测试出系统最大负载是多少。通过压测，不断提高客户端并发数。例如初始客户端并发数为10，依次递增为20，30，40等，在没有到达性能瓶颈前，压测的TPS会随着客户端并发数的增加而增大，响应时间通常会随着客户端并发数增加而增加，但是增加的幅度并不明显。当客户端并发数到达某个阀值的时候，TPS不再增长，反而出现下降，响应时间也出现跳跃式增长，这个时候就是系统最大性能。这个时候就可以得到系统瓶颈时的活跃连接数m(事先将maxSize设置为一个大到应该不会达到的数值)。为了给连接池空出足够的大小，我们在为期分配一倍的余量，所以maxSize=n*2。另外，将n当做连接池告警阀值，配置告警。

**initialSize：**连接池初始化大小，一般设定为一个较小的值即可。但是如果线上流量很大，服务上线后，连接不够用，会短时间创建大量线程等待连接创建的问题，可酌情增大initialSize。
如果是已经在线上运行服务，则可以设置为生产环境平峰的活跃线程数。

**minIdle：**设置为系统在流量低峰时段时活跃连接数

扩展阅读：[数据库连接池数量设置为多少合适？](https://cloud.tencent.com/developer/article/1575670) 

> connections = ((core_count * 2) + effective_spindle_count)

#### 具体示例

##### 情况 A：数据库响应时间 1 s

- **假设**：CPU 核心数 8，业务逻辑处理时间（ST）≈10 ms，IO 等待时间（WT）≈1000 ms。

- **按 Goetz 公式**：`threads = 8 * (1 + 1000/10) ≈ 8 * 101 = 808`

  但数据库连接池仅 10 条，此时绝大多数线程会在 JDBC 层排队，反而降低性能 ([Zalando Engineering Blog](https://engineering.zalando.com/posts/2019/04/how-to-set-an-ideal-thread-pool-size.html?utm_source=chatgpt.com), [红帽文档](https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/24.0/html/high_availability_guide/concepts-threads-?utm_source=chatgpt.com))。
  
- **实践建议**：将线程池大小设为连接池大小的 1.1～1.25 倍，以应对瞬时小流量峰值。例如：

  `corePoolSize = 10 maximumPoolSize = 12  // 允许最多两条备用线程`
  
  这样既能充分利用 10 条连接，又能在 1 s 的处理窗口内处理偶发的短暂冲击 ([Stack Overflow](https://stackoverflow.com/questions/1208077/optimal-number-of-connections-in-connection-pool?utm_source=chatgpt.com), [Medium](https://medium.com/@gaborfarkasds/so-how-big-should-that-connection-pool-be-e5c69f2e15dd?utm_source=chatgpt.com))。

##### 情况 B：数据库响应时间 10 ms

- **假设**：WT=10 ms，ST=10 ms，CPU 核心数 8。

- **按 Goetz 公式**：`threads = 8 * (1 + 10/10) = 16`

- **受连接池约束**：由于同时只能打开 10 条连接，并发线程再多也无法并行地阻塞在 JDBC 层。此时最优线程池大小依然是 10～12 之间 ([红帽文档](https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/24.0/html/high_availability_guide/concepts-threads-?utm_source=chatgpt.com), [Stack Overflow](https://stackoverflow.com/questions/1208077/optimal-number-of-connections-in-connection-pool?utm_source=chatgpt.com))。

- **基于 Little’s Law**：若目标吞吐 1000 QPS，W=0.01 s，则`L = λ * W = 1000 * 0.01 = 10 threads`刚好匹配连接池大小 ([Zalando Engineering Blog](https://engineering.zalando.com/posts/2019/04/how-to-set-an-ideal-thread-pool-size.html?utm_source=chatgpt.com))。

#### 最佳实践与调优

1. **分离不同任务线程池**
   对于数据库调用、外部 HTTP、CPU 密集型计算等，最好使用多个专用线程池，分别根据各自的 WT/ST 及资源约束来调优 ([Stack Overflow](https://stackoverflow.com/questions/76684733/how-to-determine-corethreadsize-if-having-multiple-threadpools-in-application?utm_source=chatgpt.com))。
2. **监控与压测“Measure, Don’t Guess”**
   所有公式与经验值仅作起点，必须通过 A/B 测试和压测（Load Test）验证性能瓶颈，动态调整 core/max、队列长度等参数 ([Zalando Engineering Blog](https://engineering.zalando.com/posts/2019/04/how-to-set-an-ideal-thread-pool-size.html?utm_source=chatgpt.com))。
3. **考虑拒绝策略与队列类型**
   结合业务特性选择合适的队列（SynchronousQueue、LinkedBlockingQueue 等）及拒绝策略（CallerRunsPolicy、AbortPolicy 等），避免在高并发下无界队列导致 OOM。
4. **动态伸缩**
   在流量可预测且波动较大时，可结合 Kubernetes/HPA、Spring Cloud TaskExecutor 等方案，实现线程池在运行时动态伸缩，以更加经济地利用资源。

## `CompletableFuture`

`Future` 在实际使用过程中存在一些局限性比如不支持异步任务的编排组合、获取计算结果的 `get()` 方法为阻塞调用。Java 8 才被引入`CompletableFuture` 类可以解决`Future` 的这些缺陷。`CompletableFuture` 除了提供了更为好用和强大的 `Future` 特性之外，还提供了函数式编程、异步任务编排组合（可以将多个异步任务串联起来，组成一个完整的链式调用）等能力。对某些部分可以并行执行的异步任务支持比较好。

| 方法                               | 描述                                 |
| ---------------------------------- | ------------------------------------ |
| **`runAsync`**                     | 执行无返回值异步任务。               |
| **`supplyAsync`**                  | 执行有返回值异步任务。               |
| **`thenApply`**                    | 转换结果并返回新的结果。             |
| **`thenAccept`**                   | 消费结果但不返回新结果。             |
| **`thenCombine`**                  | 合并两个任务的结果。                 |
| **`allOf`** / **`anyOf`**          | 等待所有任务完成 / 任意任务完成。    |
| **`exceptionally`** / **`handle`** | 处理异常并提供默认值或继续处理结果。 |

以下是 **`CompletableFuture`** 的使用示例及详解：

### 任务创建

**创建异步任务：**`runAsync()`**：执行无返回值任务。**

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    System.out.println("执行异步任务：" + Thread.currentThread().getName());
});
future.join(); // 等待任务执行完成
```

- **`join()`**：等待任务执行完成（类似 **`get()`**，但不会抛出 checked 异常）。

**执行有返回值任务：**`supplyAsync()`**：执行有返回值任务。**

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    System.out.println("计算中...");
    return 42; // 返回结果
});
System.out.println("计算结果：" + future.join());
```

### 链式操作

`thenApply` - 对任务结果进行变换（同步执行）。

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 10)
        .thenApply(result -> result * 2); // 结果翻倍
System.out.println(future.join()); // 输出 20
```

`thenAccept` - 消费结果，不返回新的结果。

```java
CompletableFuture.supplyAsync(() -> "Hello World")
        .thenAccept(result -> System.out.println("结果: " + result));
```

`thenRun` - 不关心前一任务结果，直接执行下一步操作。

```java
CompletableFuture.supplyAsync(() -> "任务完成")
        .thenRun(() -> System.out.println("继续执行任务"));
```

### 组合多个任务

`thenCombine` - 合并两个任务结果，组合两个任务结果，并执行新任务。

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 20);

CompletableFuture<Integer> result = future1.thenCombine(future2, (a, b) -> a + b);
System.out.println("结果: " + result.join()); // 输出 30
```

`allOf` - 等待所有任务完成，但不返回结果。

```java
CompletableFuture<Void> all = CompletableFuture.allOf(
        CompletableFuture.runAsync(() -> System.out.println("任务1")),
        CompletableFuture.runAsync(() -> System.out.println("任务2")),
        CompletableFuture.runAsync(() -> System.out.println("任务3"))
);
all.join(); // 等待所有任务完成
```

`anyOf` - 任意任务完成即结束，返回第一个任务的结果

```java
CompletableFuture<Object> any = CompletableFuture.anyOf(
        CompletableFuture.supplyAsync(() -> "任务1"),
        CompletableFuture.supplyAsync(() -> "任务2"),
        CompletableFuture.supplyAsync(() -> "任务3")
);
System.out.println("最快完成的任务: " + any.join());
```

### 异常处理

`exceptionally` - 捕获异常并返回默认值，继续执行任务链。

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    if (true) throw new RuntimeException("任务异常");
    return 42;
}).exceptionally(ex -> {
    System.out.println("捕获异常: " + ex.getMessage());
    return -1; // 返回默认值
});
System.out.println("结果: " + future.join()); // 输出 -1
```

`handle`- 捕获异常并处理结果,同时处理正常结果和异常情况。

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    if (true) throw new RuntimeException("计算出错");
    return 10;
}).handle((result, ex) -> {
    if (ex != null) {
        System.out.println("处理异常: " + ex.getMessage());
        return -1; // 返回默认值
    }
    return result * 2;
});
System.out.println("最终结果: " + future.join());
```

### 线程池自定义

指定线程池：

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

CompletableFuture.runAsync(() -> {
    System.out.println("自定义线程池: " + Thread.currentThread().getName());
}, executor).join();

executor.shutdown();
```

**异步任务默认使用 **`ForkJoinPool.commonPool()`，但可以自定义线程池以控制资源。

### 例：批量任务处理

```java
List<CompletableFuture<Integer>> futures = IntStream.range(1, 6)
        .mapToObj(i -> CompletableFuture.supplyAsync(() -> i * 2))
        .collect(Collectors.toList());

CompletableFuture<Void> allDone = CompletableFuture.allOf(
        futures.toArray(new CompletableFuture[0])
);

allDone.join();

List<Integer> results = futures.stream()
        .map(CompletableFuture::join)
        .collect(Collectors.toList());

System.out.println("所有结果: " + results);
```

- 使用 **`allOf()`** 确保所有任务完成。
- 收集各任务结果并返回。

```
所有结果: [2, 4, 6, 8, 10]
```

### [虚拟线程](https://javaguide.cn/java/concurrent/virtual-thread.html)  

