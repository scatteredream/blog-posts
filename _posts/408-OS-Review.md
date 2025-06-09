---
name: os-concepts
title: OS 期末复习
date: 2025-06-08
categories: OS
---



> [操作系统启动 CPU模式 | scatteredream's blog](https://scatteredream.github.io/2025/01/06/408-OS-Boot/)
>
> [虚拟机 | scatteredream's blog](https://scatteredream.github.io/2025/01/20/408-OS-虚拟机/)
>
> [Linux 基本命令 | scatteredream's blog](https://scatteredream.github.io/2024/11/30/408-OS-Linux/)

# 进程、线程管理

[虚拟化#CPU-Virtualization | scatteredream's blog](https://scatteredream.github.io/2025/01/18/408-OS-虚拟化/#CPU-Virtualization)

进程的组成：code，data(static, heap, stack)，PCB

特征：动态性、独立性、异步性。（不共享）

PCB：和进程的生命周期相同

- 上下文（寄存器信息，主要是SP(栈)和IP(指令)，以及一些通用寄存器）、
- 进程当前状态
- 进程的地址空间、打开的文件、PID、内核栈、中断信息、父进程。

PCB 不采用静态分配(固定大小数组)灵活性差。难以应对进程动态创建/销毁，现代系统普遍采用**动态链表**或**索引表**。

PID 定位 PCB Hash表

| 原语名称 | 英文名 / 表示           | 作用说明                         |
| -------- | ----------------------- | -------------------------------- |
| 创建进程 | `create` / `fork()`     | 创建一个新进程                   |
| 撤销进程 | `exit` / `terminate`    | 结束当前进程                     |
| 阻塞进程 | `block`                 | 使当前进程进入等待（不参与调度） |
| 唤醒进程 | `wakeup`                | 使某个等待的进程重新进入就绪状态 |
| 进程切换 | `dispatch` / `schedule` | 把 CPU 切换给另一个进程          |

OS 对进程的操作：

- **fork**：复制一个和父进程一样的子进程（子进程直接从fork返回然后继续执行）子进程的内存空间和父进程是独立的，并且变量的值大部分一样。遵循 COW 原则，推迟甚至避免了大量复制
- **wait**：子进程创建后，根据OS调度（schedule）决定先后顺序，wait可以使父进程等子进程执行完再开始运行
- **exec**：当前进程不想运行和之前一样的代码，可以调用exec加参数运行其他代码，新的程序会替代原来进程的所有信息，因此exec后边的代码是不会被执行的。

![image-20250608215235926](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608215235926.png)

READY：新建时进程所需资源已全部分配 或者 阻塞时事件发生。

![image-20250608214700023](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608214700023.png)

Redirecting：

- shell的基本原理就是主进程fork wait 子进程这边exec 运行其他程序，运行完成主进程wait结束，继续进行其他操作
- **输出重定向**（redirecting）：默认输出就是标准输出流，如果你想重定向到一个文件，应当关闭stdout然后重新打开一个你想要的文件描述符。
- **管道**（pipe）：也类似与输出重定向，上一个的输出无缝作为下一个的输入

线程 THREAD：同一进程中的各线程可共享该进程所拥有的资源。如具有相同的进程地址空间、访问进程所拥有的已打开文件等。

- Thread 作为 CPU 调度单位，也是并行的最小单位。Process 作为资源分配的单位。
- 同一个进程的线程共享大部分资源，可以访问进程的code和data
- thread-local 只有 tcb、寄存器、栈等
- 线程的创建和切换更加轻量
- IPC 必须通过内核保证，线程通信因为有很多共享的所以可以不通过内核

![image-20250608221712079](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608221712079.png)

# 进程的交互

## 互斥与同步

[并发 | scatteredream's blog](https://scatteredream.github.io/2025/01/24/408-OS-并发/) 

Semaphore 的 P V 操作

```c
P(Semaphore  s)
{
   --s.count;          //表示申请一个资源;
   if (s.count < 0)   //表示没有空闲资源;
    {
        // 调用进程进入等待队列s.queue;
        // 阻塞调用进程;
    }
}

V(Semaphore s )
{
 ++s.count;	             //表示释放一个资源；
 if (s.count <= 0)    //表示有进程处于阻塞状态；
 {
     // 从等待队列s.queue中取出一个进程p;
     // 进程p进入就绪队列;
 }
}
```

## IPC

> 管道（Pipe）

1. 它是半双工的（即数据只能在一个方向上流动），具有固定的读端和写端。
2. 它只能用于具有亲缘关系的进程之间的通信（也是父子进程或者兄弟进程之间）。
3. 它可以看成是一种特殊的文件，对于它的读写也可以使用普通的read、write 等函数。但是它不是普通的文件，并不属于其他任何文件系统，并且只存在于内存中。

当一个管道建立时，它会创建两个文件描述符：`fd[0]`为读而打开，`fd[1]`为写而打开。

调用 pipe 的进程接着调用 fork，这样就创建了父进程与子进程之间的 IPC 通道。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/abeda97ed8d909980e1183629f516dee.png)

Bash: `cat story.txt | wc` `cat *.txt | grep hello | wc -l`

```c
fork(); exec("cat")
fork(); exec("wc")
forl(); exec("grep")
pipe(cat_output_fd, grep_input_fd);
pipe(grep_output_fd, wc_input_fd);
```

```c
pid_t pid = fork();
if (pid == 0) {
    // 子进程中，执行新程序
    execl("/bin/ls", "ls", "-l", (char*) NULL);
    // 如果 exec 执行成功，下面不会被执行
    perror("exec failed");
    exit(1);
} else {
    // 父进程中，等待子进程
    wait(NULL);
}
```



> 信号（SIGNAL）

操作系统通过向进程发送信号，使其异步地执行指定的处理函数（或默认操作）。

| 信号名    | 数字 | 含义                      | 默认操作    |
| --------- | ---- | ------------------------- | ----------- |
| `SIGINT`  | 2    | 来自终端的中断（Ctrl+C）  | 终止        |
| `SIGTERM` | 15   | 请求终止进程（kill 默认） | 终止        |
| `SIGKILL` | 9    | 强制终止进程（不可捕获）  | 终止        |
| `SIGSTOP` | 19   | 暂停进程（不可捕获）      | 暂停        |
| `SIGCONT` | 18   | 继续执行一个暂停的进程    | 继续执行    |
| `SIGCHLD` | 17   | 子进程状态改变            | 忽略 / 捕获 |
| `SIGHUP`  | 1    | 终端挂起                  | 终止        |
| `SIGSEGV` | 11   | 段错误（非法内存访问）    | 终止并core  |

```bash
kill -SIGINT <pid>      # 向某个进程发送 SIGINT
kill -9 <pid>           # 强制终止进程（SIGKILL）
```

```c
kill(pid, SIGTERM);     // 向指定进程发送信号
raise(SIGINT);          // 向自己发送信号
```

信号是异步的，随时可能打断程序执行。每个进程对信号有三种处理方式：`SIGKILL` 和 `SIGSTOP` 不能被捕获、阻塞或忽略。

1. **默认处理**（默认动作，如终止）
2. **捕获信号**（注册一个处理函数）
3. **忽略信号**（某些信号不能被忽略）



> [消息队列](https://blog.csdn.net/qq_34144916/article/details/81184434#t11)



> [信号量](https://blog.csdn.net/qq_34144916/article/details/81184434#t15)



> [共享内存（Shared Memory）](https://blog.csdn.net/qq_34144916/article/details/81184434#t19)



> [Socket 编程 | scatteredream's blog](https://scatteredream.github.io/2025/05/29/net-io-socket/#定义)

## 死锁

![image-20250608234254047](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608234254047.png)

死锁检测——资源分配图



# Trade-off

**空间换时间**：磁盘缓冲区、PCB FCB inode、prefetch、Spooling、缓冲池。

**时间换空间**：虚拟存储器通过消耗一定的额外时间（**缺页中断**处理时间、等请求分页系统中独有操作的时间），来达到扩大内存的逻辑空间的目的，从而给用户提供超过内存大小的进程逻辑空间。

# 调度策略

## 作业调度

周转时间 = 完成时间 - 到达时间

带权周转时间 = 周转时间 ÷ 执行时间

- **FCFS**：先到先得，计算密集型会阻塞io密集型，降低效率
- **SJF**：Shortest Job First 最短工作优先，<u>同时到达</u>，先进行最短的工作
- **STCF**：Shortest Time-to-Complete First 最短完成时间优先，针对<u>随时到达</u>的情况，到达时比较里完成还有多少时间，首次出现了任务切换的概念。
- **RR**：Round-Robin 轮转，运行一个任务到时间片就切换到下一个任务（context switch）
- **MLFQ**：Multi-Level Feedback Queue，多级反馈队列
  - 设置不同的优先级，每个任务刚到达都是最高级。
  - 级别低的任务必须先让级别高的执行完。
  - 相同级别的任务轮转执行（RR）。
  - 在同一个优先级执行时间达到阈值就降低优先级：防止高优先级一直占据CPU
    - 如果采用对每次执行单独计时的方法可能会有恶意占据CPU的情况发生
  - 每隔一段时间就重置所有任务的优先级为最高：防止低优先级的任务饥饿。

## 磁盘调度

- SSTF：最短寻道时间。驱动程序无法看到具体的磁道只能看到块地址；饥饿
- NBF：最近块。饥饿
- SCAN：双向扫描。
- Circular-SCAN：只进行单向扫描的SCAN
- LOOK：SCAN 如果扫描方向上没有了访问请求，立即掉头往回扫描。
- Circular-LOOK：C-SCAN 改进，原理同上，只不过是立即从头开始扫描

# 内存管理

[虚拟化#Memory Virtualization | scatteredream's blog](https://scatteredream.github.io/2025/01/18/408-OS-虚拟化/#Memory-Virtualization) 

- 虚拟地址的作用
- 内存分配:
  - 机制：空闲空间链表节点的分割与合并
  - 物理：Buddy, SLAB「内存池」
    - 空闲空间的管理：内存紧缩，<u>free list</u>(best fit、first fit、next fit、bitmap，内存池)
  - 虚拟：mmap malloc brk
- 虚拟地址的翻译（重定位）
  - 段式 base+bound, bound varies from each other
  - 页式 fixed bound，
  - 段页式 
  - 多级页表 fill one page with one table, hi-level table points to low-level table
  - TLB：翻译缓存
- Swap：将物理内存看作虚拟内存的缓存
  - 机制：Page Fault & Disk I/O 
  - 策略：
    - 是否需要SWAP？物理内存充足就没必要开启
    - 具体换**出**哪一页？LRU, FIFO, Random, Second Chance, LRU-K, 2Q, Clock
    - 何时换**出**？被动watermark、主动swappiness>0
    - 一次 I/O 换**出**多少页？ clustering
    - 何时换**入**？lazy aka. <u>demand paging(请求调入)</u>
    - 一次 I/O 只换**入**一页**吗**？prefetching(预取)





# 文件系统

> 文件系统 

- 定义：OS中与管理文件有关的软件和数据结构称为文件系统。它负责为用户建立、撤销、读写、修改和复制文件，还负责完成文件的按名存取和存取控制等。
- 文件系统功能：
  1. 定义文件逻辑结构：实现文件的按名存取；
  2. 定义文件的物理结构：文件信息在存储设备上的存放方式
  3. 文件信息的描述和管理：文件目录管理
  4. 合理地存放文件：对磁盘等辅助存储空间进行统一管理
  5. 实现文件信息的查找
  6. 完成文件的共享与保护

## 文件、记录和数据项

- 对记录式文件，操作系统为用户存取文件信息的最小单位是（记录）
- 由字符序列组成，文件内的信息不再划分结构，这是指（流式文件）
- 数据库文件的逻辑结构形式是（记录式文件）
- 逻辑文件是（从用户观点看）的文件组织形式。
- 文件的存储方法依赖于（文件的物理结构、存放文件的存储设备的特性）

![image-20250608231625487](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608231625487.png)

![image-20250608231633336](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608231633336.png)

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608230151762.png" alt="image-20250608230151762" style="zoom:50%;" />

## 层次结构

![image-20250608231527443](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608231527443.png)

![image-20250608230354877](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608230354877.png)

![image-20250608230413290](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608230413290.png)

![image-20250608230429042](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608230429042.png)

## 分配策略

### 文件块分配（物理结构）

文件物理结构（文件块分配）

- 连续（Contiguous Allocation）：磁盘上**连续分配**。记录文件只需起始块号和长度。「老式系统、CD-ROM 文件系统」

- 链接（Linked Allocation）：「FAT 文件系统（变种）」

  - FAT 表（File Allocation Table）：**不需要在每个块中嵌入指针**，可以将 FAT 缓存在内存中，加快速度；
  - FAT 表中集中记录每个块的指针，FAT 表中块号对应的内容是下一个块号。5-9-14 对应的内容分别是 9-14-0(EOF)
  - 起始块记录在目录项中，系统顺着 FAT 表查找整个文件。
  - 文件碎片严重（不连续）FAT 表越大，加载越耗内存；

- **索引结构（Indexed Allocation）**：每个文件分配一个**索引块**（或多个索引块），块中记录文件数据块地址。

  - UNIX（如 ext2、ext3、ext4）使用 inode，Windows NT 的 NTFS 使用 MFT
  - **单级索引**：索引块直接记录所有数据块。

  - **多级索引**（例如 Unix inode）：
    - 实现对大文件的支持。
    - 直接指针 + 间接指针（一级、二级、三级等）如下为 ext2 结构

| 指针类型         | 指向内容                              |
| ---------------- | ------------------------------------- |
| 12 个直接块指针  | 直接指向数据块                        |
| 1 个一级间接指针 | 指向一个块，块中存放数据块地址        |
| 1 个二级间接指针 | 指向一个块，块中是一级指针块地址      |
| 1 个三级间接指针 | 指向块 → 二级块 → 一级块 → 数据块地址 |

|          | inode                    | FAT                        |
| -------- | ------------------------ | -------------------------- |
| 类型     | 索引结构（混合索引）     | 链式结构（集中表）         |
| 访问方式 | 多级指针，支持随机访问   | 顺链访问，随机访问慢       |
| 碎片控制 | 更好（可配合预分配策略） | 容易碎片化                 |
| 稳定性   | 更强（结构分散）         | FAT 表损坏易导致全盘不可用 |

### 空闲分配

- 连续（Bitmap）：容易寻找连续空闲块。占空间（对于大磁盘，位图也大），并且需要加载整个bitmap
- 链接：每个空闲块中记录下一个空闲块的地址。查找连续空闲块效率低；
- 空闲表：用一张表记录每一个空闲块号或空闲块段。可以快速定位，但是表会占空间。

- 成组链接：空闲块分组管理，一个块记录多个空闲块地址 + 下一组入口地址。
  - 综合了“链接法”的简单与“表法”的批量管理优势。
  - 假设每组记录 4 个空闲块地址 + 下一个组的地址（总共可以在一个块中记录 5 个地址）
  - 每次取空闲块：
    - 优先用当前组的 4 个空闲块；
    - 当前组用完后，读取“下一组地址”所指向的块，加载新的一组空闲块信息。

```
[ 块100 ]：{ 下一个组地址 = 200, 空闲块地址 = 101, 102, 103, 104 }  
[ 块200 ]：{ 下一个组地址 = 300, 空闲块地址 = 201, 202, 203, 204 }  
[ 块300 ]：{ 下一个组地址 = 0（无下一组）, 空闲块地址 = 301, 302, 303, 304 }
```

- 空闲区间法（Extent-based Free Space Management）
  - 记录空闲区域的起始地址 + 长度（适合连续空闲块）
  - 适合大块文件系统，空间利用率高。但是不适合高度碎片化的场景。

## 文件共享

静态文件共享

- 绕弯路法：
- 连访法：
- 基于基本文件目录共享法：
- 基于索引节点共享：

![image-20250608231332461](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608231332461.png)

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608230514727.png" alt="image-20250608230514727" style="zoom:67%;" />



## 文件保护

访问类型：Read、Write、eXecute、修改访问权限

访问控制：

- 用户范围类型：指定用户、用户组、任意用户
- 访问类型和用户范围的组合：
  - 访问矩阵
  - 存取控制表

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608231226427.png" alt="image-20250608231226427" style="zoom: 33%;" />



# 设备管理

IO控制技术：

- 轮询、中断、DMA、通道



## 缓冲

- 目的：缓解CPU和外设的速度不匹配、实现并行、减少中断次数、提高效率
- 方式：单缓冲、双缓冲、环形缓冲、缓冲池

![image-20250608224100649](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608224100649.png)

## 设备分配

### 原则

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608224304923.png" alt="image-20250608224304923" style="zoom:80%;" />

![image-20250608224501939](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608224501939.png)

### SPOOLing

假脱机（Simultaneous Peripheral Operations On-Line）将独占设备虚拟成共享设备。

**SPOOLing 是一种“以磁盘作为中介缓冲”的技术**，用于协调低速外设（如打印机、CD机、磁带机）与高速 CPU 之间的速度差异。

通俗解释：你在电脑上点了**多份文档打印**，这些文档不是立即送到打印机，而是**先被缓存在磁盘的“打印队列”中**。打印机慢慢地、一份一份地取出来打印，CPU 不必等待外设。

| 应用场景   | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 打印系统   | 用户打印文档时，任务先存入“打印队列”，再由打印机按序执行，打印机这种独占设备变成共享设备 |
| 批处理系统 | 作业先存储在磁盘等待调度再运行                               |
| 多用户系统 | 多用户提交任务，SPOOLing 使多个用户“看起来像同时在打印”      |

> ⚠️注意：SPOOLing 是**磁盘级缓冲机制**，区别于内存级的缓存和缓冲。

SPOOLing 的运行流程

1. 用户程序提交 I/O 请求（如打印）；
2. 操作系统将请求信息写入**磁盘上的缓冲区（SPOOL区）**；
3. 外设空闲时，从磁盘中取出任务执行；
4. 任务完成后反馈状态（如打印完成、打印失败等）。

SPOOLing 利用了 **多道程序设计的思想**：多个程序同时驻留内存，通过**I/O 与 CPU 并行执行**来提高资源利用率，实现设备的请求排队与管理，支持多用户共享外设。

## 磁盘

![image-20250608225011060](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250608225011060.png)

磁盘调度见之前的。