---
name: jvm-in-One
title: JVM 介绍
date: 2024-09-11
tags: 
- gc
- 内存区域
categories: jvm
---

# JVM 是虚拟机吗？

## JVM 是什么？

**Java Virtual Machine (JVM)** 是 Java 平台的核心组件，它是一个<mark>**程序运行环境**<mark>，专门用来执行 Java 字节码（Bytecode）

**主要功能：**

1. **跨平台支持**：JVM 实现了 Java 的 "Write Once, Run Anywhere" 特性，使 Java 程序能够在不同操作系统上运行。
2. 字节码解释和编译：JVM 将 Java 编译器生成的字节码（平台无关）转换为平台相关的机器代码，并执行。
3. 内存管理：提供垃圾回收机制 (Garbage Collection, GC)。
4. 安全性：内置安全检查机制，确保 Java 程序在受控环境中运行，防止恶意代码执行。

## Hypervisor/VMM 是什么?

![undefined](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/2560px-Hyperviseur.svg.png)

Hypervisor 是一种一种运行在基础物理服务器和操作系统之间的中间软件层，可允许多个操作系统和应用共享硬件。也可叫做虚拟机监视器，即 VMM（ virtual machine monitor ）。

- 当服务器启动并执行 Hypervisor 时，它会加载所有虚拟机客户端的操作系统同时会分配给每一台虚拟机适量的内存，CPU，网络和磁盘。它允许多个操作系统（称为**Guest OS**）在同一台物理机器（称为**Host**）上同时运行，每个操作系统被隔离在独立的虚拟机中。
- Hypervisor 负责资源分配，如 CPU、内存和存储，协调着这些硬件资源的访问，而且在各个虚拟机之间施加防护，处理虚拟机之间的通信和安全隔离。

## JVM 与 Hypervisor/VMM 的对比

| 特性           | JVM                                     | Hypervisor/VMM                                     |
| -------------- | --------------------------------------- | -------------------------------------------------- |
| **用途**       | 运行 Java 程序的虚拟环境                | 管理和运行多个虚拟机，每个虚拟机可运行不同操作系统 |
| **虚拟化层级** | 应用程序级虚拟化（针对 Java 字节码）    | 操作系统级虚拟化（针对硬件资源）                   |
| **管理对象**   | **Java 字节码执行环境**                 | 虚拟机管理和**硬件资源虚拟化**                     |
| **依赖性**     | 必须安装在操作系统上，依赖操作系统环境  | Type 1 可以直接运行在硬件上，Type 2 依赖操作系统   |
| **隔离性**     | 程序之间隔离性较弱，依赖 JVM 的安全机制 | 每个虚拟机互相独立，硬件级别隔离                   |
| **示例**       | Oracle HotSpot VM、OpenJDK JVM          | VMware ESXi、VirtualBox、KVM、Hyper-V              |

1. 在 VMware ESXi 上创建一个虚拟机；
2. 在虚拟机内部运行一个 Guest OS，比如 Linux；
3. 在 Linux 中安装 Java 环境并运行 JVM。

<!-- more -->

# Java Memory Model(J.U.C)

# Java 内存区域



<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/java-runtime-data-areas-jdk1.8.png" alt="Java 运行时数据区域（JDK1.8 ）" style="zoom: 80%;" />

1.7之前， MetaSpace 在运行时内存区域，也是独立于堆、线程私有内存的一片线程共享内存

线程私有：**虚拟机栈**、**本地方法栈**、**程序计数器 PC**

线程共享：堆、Metaspace、直接内存

Java 虚拟机规范对于运行时数据区域的规定是相当宽松的。以堆为例：堆可以是连续空间，也可以不连续。堆的大小可以固定，也可以在运行时按需扩展 。虚拟机实现者可以使用任何垃圾回收算法管理堆，甚至完全不进行垃圾收集也是可以的。

## 运行时数据区域(Runtime)

![image-20250103144549650](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250103144549650.png)

### 线程私有(Local)

#### <mark>程序计数器<mark>(Program Counter)

- 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。	
- 在多线程的情况下，程序计数器用于记录当前线程执行的位置，当线程被切换回来时能够知道该线程上次运行的位置。
- 如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地 址；如果正在执行的是本地（Native）方法，这个计数器值则应为空（Undefined）。
- 唯一不会出现 `OutOufMemoryError` 的内存区域，与线程的生命周期相同

#### <mark>虚拟机栈<mark>(VM Stack)

- per thread：与线程的生命周期相同
- 所有的 Java 方法调用都是通过虚拟机栈来实现的，就是事实上的 Java 方法栈
- **LIFO**：只支持出栈和入栈两种操作

##### 栈帧(Stack Frame)

- 每一次方法调用都会有一个对应的栈帧被压入VM Stack，调用结束时弹出一个栈帧。

- 每个**栈帧**中都拥有：局部变量表、操作数栈、动态链接、方法返回地址

  <img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/stack-area.png" alt="Java 虚拟机栈" style="zoom:67%;" />

**<mark>局部变量表<mark>**：局部变量表存放了编译期可知的各种Java虚拟机**基本数据类型**（boolean、byte、char、short、int、 float、long、double）、**对象引用**（reference类型，它并不等同于对象本身，可能是一个指向对象起始 地址的引用指针，也可能是指向一个代表对象的句柄或者其他与此对象相关的位置）和**returnAddress 类型**（指向了一条字节码指令的地址）。

其存储空间以局部变量槽（Slot）来表示，其中64位长度的long和 double类型的数据会占用两个变量槽，其余的数据类型只占用一个。所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在栈帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。这里说的“大小”是指变量槽的数量， 虚拟机真正使用多大的内存空间（譬如按照1个变量槽占用32个比特、64个比特，或者更多）来实现一个变量槽，这是完全由具体的虚拟机实现自行决定的事情。

**<mark>操作数栈<mark>**：主要作为**方法调用的中转站**使用，用于存放方法执行过程中产生的中间计算结果。另外，计算过程中产生的**临时变量**也会放在操作数栈中。方法执行过程中通过出栈、入栈完成计算操作。

**<mark>动态链接<mark>**：主要用于**一个方法需要调用其他方法**的场景。Class 文件的常量池里保存有大量的符号引用比如方法引用的符号引用。当一个方法要调用其他方法，需要将常量池中指向方法的符号引用转化为实际内存地址。动态链接的作用就是为了将符号引用转换为调用方法的直接引用，这个过程也被称为 **动态链接**。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/jvmimage-20220331175738692-1735821102490-10.png)

##### StackOverFlow/OutOfMemory **Error**

Java 方法有两种返回方式，一种是 return 语句正常返回，一种是抛出异常。不管哪种返回方式，都会导致栈帧被弹出。也就是说， **栈帧随着方法调用而创建，随着方法结束而销毁。无论方法正常完成还是异常完成都算作方法结束。** 

栈空间虽然不是无限的，但一般正常调用的情况下是不会出现问题的。不过，如果函数调用陷入无限循环的话，就会导致栈中被压入太多栈帧而占用太多空间，导致栈空间过深。那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 `StackOverFlowError` 错误。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/《深入理解虚拟机》第三版的第2章-虚拟机栈.png)

**<mark>方法返回地址<mark>**：

- **记录调用点的字节码指令地址**，以便方法执行完毕后能返回到正确的位置继续执行程序。
  1. JVM 创建一个栈帧并保存调用点的字节码指令地址（`result = add(2, 3)`）。
  2. 计算 `a + b` 的结果，并将值返回给调用点。
  3. 执行结束后，当前栈帧弹出，并根据**返回地址**继续执行 `System.out.println(result)`
- 保证方法调用的**有序性**和**完整性**，支持嵌套调用和递归处理。



#### <mark>本地方法栈<mark>(Native Stack)

和虚拟机栈所发挥的作用非常相似，区别是：**虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。



### 线程共享(Global)

#### <mark>堆<mark>(Heap)

Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，<mark>几乎<mark>所有的<mark>对象实例<mark>以及<mark>数组<mark>都在这里分配内存。 [现在还可以在线程栈上分配](#threadlocal)

堆是垃圾收集器管理的主要区域，因此也被称作 **GC 堆（Garbage Collected Heap）**。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代；再细致一点有：Eden、Survivor、Old 等空间。进一步划分的目的是更好地回收内存，或者更快地分配内存。

##### 堆分代(HotSpot VM)

![堆内存结构](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/hotspot-heap-structure.png)

 **Eden** 区、两个 **Survivor** 区 S0 和 S1 都属于新生代，中间一层属于老年代，最下面一层属于永久代。**JDK 8 版本之后 PermGen(永久代) 已被 Metaspace(元空间) 取代，元空间使用本地内存。** 

大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 S0 或者 S1，并且对象的年龄还会加 1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。不过，设置的值应该在 0-15，否则会爆出以下错误：MaxTenuringThreshold of 20 is invalid; must be between 0 and 15

###### 为什么年龄只能是 0-15?

因为记录年龄的区域在对象头中，这个区域的大小通常是 4 位。这 4 位可以表示的最大二进制数字是 1111，即十进制的 15。因此，对象的年龄被限制为 0 到 15。

###### 堆内存一定分代管理吗?

在十年之前（以G1收集器的出现为分界），作为业界绝对主流的HotSpot虚拟机，它内部的垃圾收集器全部 都基于“经典分代” 来设计，需要新生代、老年代收集器搭配才能工作。但是随着GC技术的发展，HotSpot里面也出现了不采用分代设计的新GC。

如果从分配内存的角度看，所有线程共享的Java堆中可以划分出多个线程私有的分配缓冲区 （Thread Local Allocation Buffer，TLAB），以提升对象分配时的效率。不过无论从什么角度，无论如何划分，都不会改变Java堆中存储内容的共性，无论是哪个区域，存储的都只能是对象的实例，将 Java 堆细分的目的只是为了更好地回收内存，或者更快地分配内存。

##### <mark>字符串常量池<mark>(String Constant Pool)

**字符串常量池** 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

```
// 在字符串常量池中创建字符串对象 ”ab“
// 将字符串对象 ”ab“ 的引用赋值给给 aa
String aa = "ab";
// 直接返回字符串常量池中字符串对象 ”ab“，赋值给引用 bb
String bb = "ab";
System.out.println(aa==bb); // true
```

HotSpot 虚拟机中字符串常量池的实现是 `src/hotspot/share/classfile/stringTable.cpp` ,`StringTable` 可以简单理解为一个固定大小的`HashTable` ，容量为 `StringTableSize`（可以通过 `-XX:StringTableSize` 参数来设置），保存的是字符串（key）和 字符串对象的引用（value）的映射关系，字符串对象的引用指向堆中的字符串对象。

JDK1.7 之前，字符串常量池存放在永久代。JDK1.7 **字符串常量池**和**静态变量**从**永久代**移动到了 Java **堆**中。主要是因为永久代（方法区实现）的 GC 回收效率太低，只有在整堆收集 (Full GC)的时候才会被执行 GC。Java 程序中通常会有大量的被创建的字符串等待回收，将字符串常量池放到堆中，能够更高效及时地回收字符串内存。

> **运行时常量池、方法区、字符串常量池这些都是不随虚拟机实现而改变的逻辑概念，是公共且抽象的，Metaspace、Heap 是与具体某种虚拟机实现相关的物理概念，是私有且具体的。**



## 本地内存(Native Memory, 也是线程共享)

### <mark>方法区</mark>(Method Area)

方法区属于是 JVM 运行时数据区域的一块逻辑区域，是各个线程共享的内存区域。

《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，方法区到底要如何实现那就是虚拟机自己要考虑的事情了。也就是说，在不同的虚拟机实现上，方法区的实现是不同的。

当虚拟机要使用一个类时，它需要读取并解析 Class 文件获取相关信息，再将信息存入到方法区。方法区会存储已被虚拟机加载的 **类信息、字段信息、方法信息、常量、静态变量、即时编译器编译后的代码缓存等数据**。

#### 永久代(PermGen) 和 元空间(Metaspace)

![image-20250103142749020](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250103142749020.png)

**方法区和永久代以及元空间是什么关系呢？** 方法区和永久代以及元空间的关系很像 Java 中接口和类的关系，类实现了接口，这里的类就可以看作是**永久代**和**元空间**，接口可以看作是**方法区**，也就是说**永久代**以及**元空间**是 **HotSpot 虚拟机对虚拟机规范中方法区的两种实现方式**。并且，**永久代**（Permanent Gen）是 JDK 1.8 之前的方法区实现，JDK 1.8 及以后方法区的实现变成了**元空间**（Metaspace）。

- 整个**永久代**（PermGen）有一个 JVM 本身设置的固定大小上限，无法进行调整（也就是受到 JVM 内存的限制），而**元空间**（Metaspace）使用的是本地内存，受**本机可用内存**的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。
- 在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。
- 永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。
- **元空间**（Metaspace）里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。GC在元空间的回收主要由Full GC（如CMS或G1的Full GC）触发，JVM维护元空间的容量阈值（High Water Mark），当元空间的使用接近阈值时触发 Full GC。元空间的回收与堆的Full GC是同步的，元空间不会单独进行类似堆中Minor GC的分代回收。这种GC过程通过回收那些对应的`Class`对象和`ClassLoader`在堆中已经不可达的类元数据（主要就是类的卸载）来释放元空间的内存。如后面所说，GC结果很难令人满意，但是又不能完全忽视。

#### 常用参数

```java
-XX:MetaspaceSize=N /* 设置 Metaspace 的初始（和最小大小）
				        如果未指定，根据运行时需求动态调整 */
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小 默认unlimited
```

#### 运行时常量池(Runtime Constant Pool)

Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有用于存放编译期生成的各种字面量（Literal）和符号引用（Symbolic Reference）的 **常量池表(Constant Pool Table)** 。

字面量是源代码中的固定值的表示法，即通过字面我们就能知道其值的含义。字面量包括整数、浮点数和字符串字面量。常见的符号引用包括类符号引用、字段符号引用、方法符号引用、接口方法符号。

运行时常量池相对于Class文件常量池的另外一个重要特征是具备动态性，Java语言并不要求常量 一定只有编译期才能产生，也就是说，并非预置入Class文件中常量池的内容才能进入方法区运行时常 量池，运行期间也可以将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的 intern()方法。

《深入理解 Java 虚拟机》7.34 节第三版对符号引用和直接引用的解释如下：

![符号引用和直接引用](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/symbol-reference-and-direct-reference.png)

常量池表会在类加载后存放到方法区的**运行时常量池**中。

运行时常量池的功能类似于传统编程语言的符号表，尽管它包含了比典型符号表更广泛的数据。

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 `OutOfMemoryError` 错误。

#### GC?

> 《Java虚拟机规范》对方法区的约束是非常宽松的，除了和Java堆一样不需要连续的内存和可以选择固定大小或者可扩展外，甚至还可以选择不实现垃圾收集。相对而言，垃圾收集行为在这个区域的确是比较少出现的，但并非数据进入了方法区就如永久代的名字一样“永久”存在了。这区域的内存回收目标主要是针对运行时常量池的回收和对类型的卸载，一般来说这个区域的回收效果比较难令人满意，尤其是类型的卸载，条件相当苛刻，但是这部分区域的回收有时又确实是必要的。以前Sun公司的Bug列表中，曾出现过的若干个严重的Bug就是由于低版本的HotSpot虚拟机对此区域未完全回收而导致内存泄漏。

### <mark>直接内存</mark>(Direct Memory)

直接内存是一种特殊的内存缓冲区，并不在 Java 堆或方法区中分配的，而是通过 JNI 的方式在本地内存上分配的。它并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 `OutOfMemoryError` 错误出现。

JDK 1.4 中新加入的 **NIO（Non-Blocking I/O，也被称为 New I/O）**，引入了一种基于**通道（Channel）**与**缓存区（Buffer）**的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

显然，本机直接内存的分配不会受到Java堆大小的限制，但是，既然是内存，则肯定还是会受到 本机总内存（包括物理内存、SWAP分区或者分页文件）大小以及处理器寻址空间的限制，一般服务 器管理员配置虚拟机参数时，会根据实际内存去设置-Xmx等参数信息，但经常忽略掉直接内存，使得 各个内存区域总和大于物理内存限制（包括物理的和操作系统级的限制），从而导致动态扩展时出现 OutOfMemoryError。

## HotSpot VM 对象

### 对象创建的过程

#### 类加载检查 metadata check

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，必须先执行类加载过程，确保 class metadata 已经加载到内存中，并在方法区中创建类的相关信息（如字段、方法、常量池等）。

#### 分配内存 memory allocation

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，**选择哪种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定**。

##### 内存分配的两种方式

- **指针碰撞**(Bump the Pointer): 
  - 适用场合：堆内存规整（即没有内存碎片）的情况下。
  - 原理：用过的内存全部整合到一边，没有用过的内存放在另一边，中间有一个分界指针，只需要向着没用过的内存方向将该指针移动对象内存大小位置即可。
  - 使用该分配方式的 GC 收集器：Serial, ParNew, G1, ZGC
- **空闲列表**(Free List): 
  - 适用场合：堆内存不规整的情况下。
  - 原理：虚拟机会维护一个列表，该列表中会记录哪些内存块是可用的，在分配的时候，找一块儿足够大的内存块来划分给对象实例，最后更新列表记录。类似操作系统分配内存
  - 使用该分配方式的 GC 收集器：CMS

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 算法是"清除"，还是"压缩"，复制算法的内存也是规整的（Serial, Parallel, PCompact 对于年轻代的收集）。

##### 内存分配并发问题

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机可以采用以下方式来保证堆内存分配的线程安全：

- **CAS + 失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS + 失败重试的方式保证更新操作的原子性。** 
- **TLAB (Thread Local Allocation Buffer)：** 为每一个线程预先在堆的 Eden 区分配一小块内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配。

##### <span id="threadlocal">逃逸分析优化</span>

**逃逸分析优化**：将不逃逸的对象直接分配到栈或使用标量替换，避免堆内存分配。

随着 JIT 编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。

**栈上分配**：从 JDK 1.7 开始已经默认开启逃逸分析，如果**某些方法中的对象引用没有被返回或者未被外面使用**（也就是未逃逸出去），那么对象可以直接在栈上分配内存。

**标量替换**：将原本需要分配在堆上的对象拆解成若干个基础数据类型存储在栈上，进一步减少堆空间的使用。（**比如只有两个int成员变量xy的Point类，只是通过point.x point.y使用，就会将其优化成两xy两个int变量**）

**锁消除**：如果对象只在单线程中使用，那么同步锁（锁对象）可能会被消除，提高程序性能。

###### JIT 即时编译器——乐观优化

Java 程序首先被编译成 **字节码（bytecode）**，而不是直接编译为特定 CPU 的机器码，这是为了实现跨平台，字节码保留了较多结构信息，利于反编译、调试、监控。因此，解释执行字节码，肯定不如 C/C++ 那样的机器码效率高。

但是 JVM 也可以使用 **JIT 编译器**动态地将频繁执行的代码编译成本地机器码，提升性能。JIT 可以**根据真实的运行情况**进行动态优化，静态编译器很难做到这些，因为它不知道程序将来如何运行。

| 阶段     | JIT 在“想什么”                                               |
| -------- | ------------------------------------------------------------ |
| 启动阶段 | 程序刚开始，先解释执行，不急着编译                           |
| 热点检测 | 哪些代码被频繁使用？值得投入资源优化（监控方法调用次数等）   |
| 编译时机 | 什么时候编译？用 C1 还是 C2？用多少资源？                    |
| 编译策略 | 如何根据实际情况最大限度提升性能（逃逸分析、方法内联、无用代码消除） |
| 动态调整 | 实际运行过程假设不成立，可以取消优化，回退到解释执行字节码   |

#### 初始化零值 set to zero

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。（例如，整型为0，浮点型为0.0，引用类型为null等）

#### 设置对象头 set object header

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

- **Mark Word**: 存储对象的 hashcode, GC 年龄, 锁状态标志等
- **Klass Pointer**: 指向方法区中的 class metadata，确定对象的类型。
- 对于数组来说，还有数组的长度信息。

#### 执行 init 方法 execute \<init>

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

### 对象内存布局

#### 64 位 JVM 下的对象内存布局

![image.png](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/57e6e147dbe447d0b96b4d310f01846atplv-k3u1fbpfcp-zoom-in-crop-mark1512000.webp)

| 部分                 | 大小          | 描述                                                         |
| -------------------- | ------------- | ------------------------------------------------------------ |
| **Mark Word**        | 8 字节        | 存储对象状态信息，如哈希码、锁信息和 GC 标记。               |
| **Klass Pointer**    | 8 字节        | 指向对象所属类的类型元数据，便于方法调度和类型检查。         |
| **数组长度（可选）** | 4 字节 (可选) | 如果是数组对象，会额外包含存储数组长度的字段。               |
| **实例数据**         | 动态分配      | 存储实例变量的值，按照字段声明顺序对齐存储（按 4 字节或 8 字节对齐）。 |
| **对齐填充**         | 动态填充      | 保证对象大小是 8 字节的倍数，以便提高内存访问效率。          |

```java
public class Example {
    int a = 10;
    boolean flag = true;
    double value = 20.5;
}
```

在 64 位 JVM 上，内存布局如下：

| 部分                       | 大小    | 描述                                                         |
| -------------------------- | ------- | ------------------------------------------------------------ |
| **对象头 - Mark Word**     | 8 字节  | 哈希码、锁信息、GC 状态等。                                  |
| **对象头 - Klass Pointer** | 8 字节  | 指向类元数据的指针。                                         |
| **实例数据**               | 12 字节 | 包括 `a` (4 字节)、`flag` (1 字节)、填充 (3 字节) 和 `value` (8 字节)。 |
| **对齐填充**               | 4 字节  | 保证对象大小为 8 字节的倍数。                                |

总大小：**32 字节**（对齐填充确保 8 字节对齐）。

Java 对象头主要由 **Mark Word** 和 **Klass Pointer** 组成，存储对象状态信息和类型元数据指针。

- 支持 JVM 的垃圾回收、同步机制（锁）、偏向锁及哈希计算等操作。
- 数组对象还包含额外的长度字段以支持动态数组存储。
- 由于对象头是二进制存储格式，不是直接可读的文本信息，需要工具或命令解析其内容，如 `jol` 或 `javap -v`。

#### 对象头(Object Header)

Java 对象头（Object Header）是 **Java 对象在内存中的元数据信息**，主要用于支持 JVM 的对象管理和操作

##### Mark Word 的详细解析

用于存储对象的运行时状态信息，包括：

- **锁状态标记**（轻量级锁、重量级锁等）
- **哈希码**（HashCode）
- **GC 标记**（垃圾回收标记位）
- **年龄计数器**（GC 年龄，用于晋升到老年代的判断）

Mark Word 的具体结构会根据对象状态发生变化：

| 状态                   | 标记位 | 内容                                        |
| ---------------------- | ------ | ------------------------------------------- |
| **未锁定（默认状态）** | 01     | 对象哈希码、GC 年龄等信息。                 |
| **轻量级锁定**         | 00     | 指向栈中锁记录的指针（Lock Record）。       |
| **重量级锁定**         | 10     | 指向重量级锁的指针（Monitor 对象）。        |
| **GC 标记**            | 11     | 标记 GC 状态。                              |
| **偏向锁**             | 01     | 偏向线程 ID、时间戳等信息（启用偏向锁时）。 |

示例 1：Mark Word 示例（未锁定状态）

假设一个对象未加锁且未偏向：

| 偏向锁 | 锁状态 | GC 年龄 | 对象哈希码 (HashCode)            |
| ------ | ------ | ------- | -------------------------------- |
| 0      | 01     | 000000  | 00101011100011000111100011000101 |

- 偏向锁：0 表示未偏向线程。
- 锁状态：01 表示未加锁状态。
- GC 年龄：000000 表示对象的 GC 年龄为 0。
- 哈希码：对象哈希值经过位移和组合后存储在 Mark Word 中。

示例 2：轻量级锁

对象进入轻量级锁定状态：

| 偏向锁 | 锁状态 | 指针内容                                       |
| ------ | ------ | ---------------------------------------------- |
| 0      | 00     | 指向线程栈中 Lock Record（锁记录）的指针地址。 |

##### Klass Pointer（类型指针）

指向对象的**类元数据**（Class Metadata），用于标识该对象的具体类型及方法表等信息。

Klass Pointer 是一个指向方法区中 **类元数据** 的指针，包含以下信息：

- 对象所属类的名称和继承关系。
- 类字段和方法表指针（用于动态方法分派）。
- 类的内存布局及实例大小信息。

**作用：**帮助 JVM 在运行时支持**多态**和**类型检查**。

##### 数组对象的额外信息

如果对象是数组类型，则会额外存储**数组长度信息**。

| 部分             | 大小     | 描述                             |
| ---------------- | -------- | -------------------------------- |
| **数组长度字段** | 4 字节   | 存储数组元素的个数。             |
| **元素数据**     | 动态大小 | 实际存储数组元素的连续内存空间。 |

### 对象的访问定位

建立对象就是为了使用对象，我们的 Java 程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式由虚拟机实现而定，目前主流的访问方式有：**使用句柄**、**直接指针**。

#### 句柄

如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与对象类型数据各自的具体地址信息。

![对象的访问定位-使用句柄](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/access-location-of-object-handle.png)

使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄池地址，在对象被移动或者GC时只会改变句柄池中的实例数据指针，而 reference 本身不需要修改。缺点是多一次指针解析，性能略低。

#### 直接指针(HotSpot VM)

如果使用直接指针访问，reference 中存储的直接就是对象的地址。

![对象的访问定位-直接指针](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/access-location-of-object-handle-direct-pointer.png)

使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销，更适合现代高性能 JVM 和 GC，如 G1 和 ZGC，这些 GC 使用并发机制降低对象移动成本，同时提高访问速度。

### JOL 分析对象内存布局

Java Object Layout

- **JOL 的核心功能**
  1. **对象内存布局分析**
     检查 Java 对象的结构，包括对象头信息、实例数据以及填充字节（padding）。
  2. **对象大小计算**
     精确测量对象占用的内存空间，有助于优化内存使用。
  3. **对象对齐规则分析**
     研究 Java 虚拟机的内存对齐方式和效率。
  4. **偏向锁与轻量级锁的状态分析**
     观察锁状态以及对象头的变化，帮助理解 Java 并发机制的底层实现。

- JOL 是基于 JVM 实现的，具体表现可能因 JVM 版本和平台而异。
- 使用 JOL 分析锁状态时，需要结合 `-XX:+UseBiasedLocking` 等 JVM 参数。
- JOL 不支持所有类型的对象分析，例如直接内存分配的对象需要其他工具辅助。

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.16</version>
</dependency>

```

```java
import org.openjdk.jol.info.ClassLayout;

public class JOLExample {
    static class Example {
        int a;
        boolean b;
        long c;
    }

    public static void main(String[] args) {
        // 打印对象内存布局
        Example example = new Example();
        System.out.println(ClassLayout.parseInstance(example).toPrintable());
    }
}

```

```php
org.example.JOLExample$Example object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                     VALUE
      0     8        (object header)                 0x0000000000000000
      8     4    int JOLExample$Example.a            0
     12     1 boolean JOLExample$Example.b           false
     13     3        (alignment/padding gap)
     16     8   long JOLExample$Example.c            0
Instance size: 24 bytes
Space losses: 3 bytes internal + 0 bytes external = 3 bytes total
/*
 * OFFSET：字段或数据在对象中的偏移量（单位字节）。
 * SIZE：字段占用的字节数。
 * TYPE/DESCRIPTION：字段类型和名称。
 * VALUE：字段的当前值。
 * alignment/padding：由于内存对齐，填充的字节数，保证效率。
 * Instance size：对象的总大小（包括填充字节）。
*/
```



# [JVM 调参](https://javaguide.cn/java/jvm/jvm-parameters-intro.html) 

## JDK 分析工具

除了之前提到的 JOL 是运行在 JVM 上的分析工具，还有很多命令行和可视化工具，主要用于监控、性能分析、内存管理和线程死锁与阻塞分析。

| 工具         | 主要用途                     | 适用场景                               |
| ------------ | ---------------------------- | -------------------------------------- |
| **JConsole** | 实时监控 JVM 性能            | 本地或远程简单监控应用性能             |
| **VisualVM** | 高级性能分析和内存快照分析   | 找出性能瓶颈和内存泄漏问题             |
| **jstat**    | JVM 性能统计和 GC 监控       | 快速诊断内存和垃圾回收问题             |
| **jmap**     | 查看堆信息，生成堆快照       | 内存分析和排查内存泄漏                 |
| **jhat**     | 分析堆快照，与jmap配合       | 离线分析堆对象引用关系                 |
| **jstack**   | 查看线程堆栈信息             | 排查线程死锁和阻塞问题                 |
| **jinfo**    | 查看和调整 JVM 参数          | 动态调整配置，调试运行时环境           |
| **JMC/JFR**  | 高级分析工具，支持低开销记录 | 深入分析生产环境性能问题（低性能影响） |

除了 **JOL (Java Object Layout)** 之外，还有以下几款同类型的 Java 内存分析和对象布局工具，可以用于分析 Java 对象结构、内存占用及内存管理相关问题：

JDK 

| 工具       | 类型           | 主要用途                                  | 是否免费 |
| ---------- | -------------- | ----------------------------------------- | -------- |
| **JOL**    | 对象布局分析   | 内存对齐、对象头、填充字节分析            | 是       |
| **Arthas** | 在线诊断工具   | 实时分析线程、内存、锁状态，适合生产环境  | 是       |
| **MAT**    | 堆内存分析     | 内存泄漏、引用链分析                      | 是       |
| **javap**  | 字节码分析工具 | 查看编译后的字节码结构，研究 JVM 执行过程 | 是       |

# GC

## 强引用/软引用/弱引用/虚引用

Java 提供了 4 种不同强度的引用类型，用于控制对象的生命周期：

1. **强引用 (Strong Reference)**
   - 默认引用类型，不可回收。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不回收这种对象。
   - 示例：`String s = "Hello";`
   - 只有强引用断开，GC 才会考虑回收对象。
2. **软引用 (Soft Reference)**
   - 适合缓存数据，内存不足时才会回收。
   - 示例：`SoftReference<String> sr = new SoftReference<>(new String("Cache"));`
3. **弱引用 (Weak Reference)**
   - GC 时会立即回收，无论内存是否充足。
   - 示例：`WeakReference<String> wr = new WeakReference<>(new String("Temp"));` 这里wr对String就是一个弱引用，ThreadLocalMap 里面的 Entry 是对 ThreadLocal类型的弱引用。
4. **虚引用 (Phantom Reference)**
   - 不影响对象生命周期，仅用于检测对象何时被 GC 回收。
   - 示例：`PhantomReference<String> pr = new PhantomReference<>(new String("Test"), refQueue);`

## 失去引用

**失去引用**是指一个对象在程序中不再被任何变量或其他对象引用，从而变成**不可达对象**。这些对象无法通过代码访问，因此被认为是**垃圾**，可以由**垃圾回收器（GC）**回收并释放内存。对象失去引用意味着程序无法再访问它，它将成为垃圾回收的候选对象。当程序对对象的引用断开或超出作用域后，JVM 会在合适的时机将其回收以优化内存使用。

**引用的定义**：引用是指程序中的变量指向堆内存中的对象。例如：

```java
String str = new String("Hello");
```

这里变量 `str` 引用了字符串对象 `"Hello"`。

**变量重新赋值**

```java
String str = new String("Hello");
str = null; // 原来的 "Hello" 对象失去了引用
```

```java
String str1 = new String("Hello");
String str2 = new String("World");
str1 = str2; // "Hello" 对象失去了引用
```

**作用域结束**

```java
public void example() {
    String temp = "Temporary"; // 创建引用
} // 方法结束，temp 超出作用域，失去引用
```

**对象被移除出集合**

```java
List<String> list = new ArrayList<>();
list.add("A");
list.remove(0); // 元素 "A" 失去了引用
```

**对象之间的引用关系断开**

```java
class Node {
    Node next;
}
Node n1 = new Node();
Node n2 = new Node();
n1.next = n2;
n1.next = null; // n2 失去了引用
```

处理：

1. **垃圾回收**：JVM 会自动检测失去引用的对象，并在适当的时机进行垃圾回收（GC）。
2. **内存释放**：GC 回收这些对象占用的堆内存，使其可供新对象使用。

## 注意事项

1. 内存泄漏：如果对象仍然被引用，但实际上不再使用，就会导致垃圾回收器无法回收它们，形成内存泄漏。例如：

   ```java
   List<Object> list = new ArrayList<>();
   while (true) {
       list.add(new Object()); // 对象持续引用导致内存泄漏
   }
   ```

2. **显式释放引用**：可以通过将对象引用赋值为 `null` 来帮助垃圾回收器更快地释放内存，但通常不需要显式操作，JVM 会自动管理内存。

## GC 管理哪些对象？

1. **堆内存中的对象**
   - **新生代 (Young Generation)**：短生命周期对象，通常是临时变量和局部变量。
   - **老年代 (Old Generation)**：生命周期较长的对象，例如缓存对象或单例对象。
   - **元空间 (Metaspace)**：类的元数据和常量池。
2. **非 GC 管理的对象**
   - **栈上的局部变量**：存储在线程栈中，不由 GC 管理，方法结束后自动销毁。
   - **直接内存 (Direct Memory)**：例如 NIO 的直接缓冲区，需要显式释放。
   - **静态变量**：存储在方法区中（元空间），生命周期随类加载器而定，不属于 GC 管理对象。

## GC 回收哪些对象？

### 引用计数

每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。但是有一个弊端就是，可能会出现A 引用了 B，B 又引用了 A，这时候就算他们都不再使用了，但因为相互引用 计数器=1 永远无法被回收。

### 可达性分析

GC 使用**可达性分析算法**判断对象是否可达。如果从 GC Roots 无法找到路径引用某个对象，则它被视为垃圾对象。

#### 不可达对象

如果一个对象无法通过 **GC Roots** 直接或间接引用，则它被视为不可达对象，可以被 GC 回收。

#### GC Roots 可达

以下对象被视为 GC Roots，是垃圾回收的起始点：

1. **栈中的局部变量和方法参数**（线程栈帧中引用的对象）。
2. **静态变量引用的对象**。
3. **常量引用的对象**。
4. **类加载器引用的对象**。
5. **JNI 引用的本地代码对象**（如 Native 方法）。
6. **活动线程引用的对象**。

## 触发 GC 的条件

GC分为多种类型，如Minor GC（新生代回收）、Full GC（整个堆回收）、Mixed GC（G1收集器特有，回收新生代和部分老年代）

1. **内存不足**：年轻代(Eden/Survivor)或老年代空间耗尽。
   - 大多数对象初始分配在**新生代（Young Generation）**的Eden区。
   - 当Eden区空间不足时，会触发Minor GC。如果对象在经历一次或多次Minor GC后依然存活，并且Survivor区有足够空间，存活对象会被移动到Survivor区。
   - 长期存活的对象会被晋升（Promotion）到**老年代（Old Generation）**。当老年代空间不足时，会触发Full GC。
2. **类加载和卸载**：类元数据或方法区空间逼近 High Watermark 时触发 Full GC。
3. **对象失去引用**：对象不可达，一定 GC。
4. 现代GC器（如ZGC/Shenandoah）会在内存压力较小时**主动运行**，以：
   - 减少停顿时间（分散GC工作量）
   - 整理内存碎片
   - 维持预期的吞吐量
5. **手动释放资源**：如关闭文件流、网络连接等，减少内存压力。
6. **调用 System.gc()**：请求 JVM 进行 GC，但不保证执行。

### 为什么不等堆内存满再GC？

- **避免突发性停顿**：内存快满时GC会导致长时间StopTheWorld停顿。
- **局部性原理**：分代收集需要定期清理年轻代。
- **元空间/堆外内存**：这些区域的内存不足可能间接触发堆Full GC。

## GC 过程

- 对象创建后，GC持续监控其引用状态。
- 当对象变为不可达时，GC会标记它们为“死亡对象”。
- 对于覆盖了`finalize()`方法的对象，GC会先调用该方法，给对象一次“自救”机会，之后如果仍不可达，才会真正回收。
- JDK9及以后版本中，`finalize()`方法逐渐被废弃，因为它影响GC性能和安全性。

> `finalize()`
>
> Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的。它是在 Object 类中定义的，因此所有的类都继承了它。子类覆盖 finalize() 方法以整理系统资源或者执行其他清理工作。finalize() 方法是在垃圾收集器删除对象之前对这个对象调用的。 
>
> 当垃圾回收器(garbage colector)决定回收某对象时，就会运行该对象的finalize()方法。值得C++程序员注意的是，finalize()方法并不能等同与析构函数。Java中是没有析构函数的。C++的析构函数是在对象消亡时运行的。由于C++没有垃圾回收，对象空间手动回收，所以一旦对象用不到时，程序员就应当把它delete()掉。所以析构函数中经常做一些文件保存之类的收尾工作。但是在Java中很不幸，如果内存总是充足的，那么垃圾回收可能永远不会进行，也就是说filalize()可能永远不被执行，显然指望它做收尾工作是靠不住的。
>
> 它最主要的用途是回收特殊渠道申请的内存。Java程序有垃圾回收器，所以一般情况下内存问题不用程序员操心。但有一种JNI(Java Native Interface)调用native方法（C或C++），finalize()的工作就是回收这部分的内存。

```java
public class GCDemo {
    public static void main(String[] args) {
        Object obj1 = new Object(); // 强引用
        Object obj2 = obj1;        // 共享引用
        obj1 = null;               // obj1 失去引用，但 obj2 仍指向它
        System.gc();               // 提示 GC 执行
    }
}
```

**分析**：

- `obj1` 被赋值为 `null`，但 `obj2` 仍然引用对象，因此不会被回收。
- 若同时将 `obj2` 设置为 `null`，对象变为不可达对象，则 GC 会将其回收。

