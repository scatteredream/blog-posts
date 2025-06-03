---
name: redis-source-code
title: Redis 数据结构 网络模型
date: 2024-08-13
tags: 
- redis
- reactor
- nio
- IO多路复用
- epoll
- 内存淘汰
- 源码
categories: redis
---

# Redis-source-code

## Redis底层数据结构

### 动态字符串SDS

我们都知道Redis中保存的Key是字符串，value往往是字符串或者字符串的集合。可见字符串是Redis中最常用的一种数据结构。

不过Redis没有直接使用C语言中的字符串，因为C语言字符串存在很多问题：
获取字符串长度的需要通过运算
非二进制安全
不可修改
Redis构建了一种新的字符串结构，称为简单动态字符串（Simple Dynamic String），简称SDS。
例如，我们执行命令：

![1653984583289](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984583289.png)

那么Redis将在底层创建两个SDS，其中一个是包含“name”的SDS，另一个是包含“虎哥”的SDS。

Redis是C语言实现的，其中SDS是一个结构体，源码如下：

![1653984624671](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984624671.png)

例如，一个包含字符串“name”的sds结构如下：

![1653984648404](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984648404.png)

SDS之所以叫做动态字符串，是因为它具备动态扩容的能力，例如一个内容为“hi”的SDS：

![1653984787383](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984787383.png)

假如我们要给SDS追加一段字符串“,Amy”，这里首先会申请新内存空间：

- 如果新字符串小于1M，则新空间为扩展后字符串长度的两倍+1；

- 如果新字符串大于1M，则新空间为扩展后字符串长度+1M+1。

- 称为内存预分配，减少分配次数，且二进制安全。


![1653984822363](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984822363.png)

![1653984838306](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984838306.png)



### Intset

#### 复习C语言数据类型

> `uint32_t` 是一种数据类型定义，常用于 C 和 C++ 语言中。它表示一个**32位无符号整数**类型，具体含义如下：
>
> 1. **u**：表示“unsigned”，即无符号。
> 2. **int**：表示“整数”。
> 3. **32**：表示占用的位数，即32位。
> 4. **_t**：表示类型（type），是标准库中的约定后缀，用于区别基本数据类型的固定大小版本。
>
> **`uint32_t`的特点：**
>
> - **取值范围**：因为是无符号的，它可以存储从 0 到 \(2^{32} - 1\) 的整数，即 0 到 4,294,967,295。
> - **固定宽度**：`uint32_t` 由标准库 `stdint.h`（C99 标准）或 `cstdint`（C++11 标准）提供，确保跨平台的一致性。在不同平台和编译器上，它总是占用 32 位（4 字节）的存储空间，因此适用于需要精确控制数据大小的场景，如嵌入式编程和网络协议设计。
>
>
> **用法示例：**
>
> ```c
> #include <stdint.h>
> 
> uint32_t num = 4294967295; // 最大值 4,294,967,295
> printf("num = %u\n", num);
> ```
>
> 这种类型定义可以确保在不同硬件架构下，程序行为的一致性，是一种便携的写法。



> `int8_t` 是 C 和 C++ 语言中定义的一种数据类型，表示一个**8位有符号整数**。它也是在 `stdint.h`（C99 标准）或 `cstdint`（C++11 标准）中定义的类型，常用于需要精确控制整数大小的场景。下面是它的具体含义：
>
> 1. **int**：表示整数类型。
> 2. **8**：表示这个整数类型占用 8 位（1 字节）。
> 3. **_t**：是类型（type）的后缀，用于区别标准库中的固定宽度整数类型。
>
> **`int8_t` 的特点：**
>
> - **取值范围**：因为是有符号整数，它的取值范围是 -128 到 127。
>   - 负数范围：-128 到 -1
>   - 正数范围：0 到 127
> - **固定宽度**：`int8_t` 代表固定宽度的8位整数，不受平台影响，因此在不同编译器和硬件上始终占用 8 位（1 字节）。这种特性在嵌入式系统、网络协议和文件格式处理中很重要，因为它可以确保数据的大小和布局不变。
>
> **用法示例**：
>
> ```c
> #include <stdint.h>
> 
> int8_t temperature = -30; // 设置温度为 -30 摄氏度
> printf("Temperature: %d\n", temperature);
> ```
>
> `int8_t` 和 `uint8_t`（8 位无符号整数）都是用于表示小范围的整数类型，通常在内存有限的系统中或者精确到字节操作的场景中广泛使用。

#### IntSet实现

IntSet是Redis中set集合的一种实现方式，基于整数数组来实现，并且具备长度可变、有序等特征。
结构如下：

![1653984923322](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984923322.png)

其中的encoding包含三种模式，表示存储的整数大小不同：

![1653984942385](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984942385.png)

为了方便查找，Redis会将intset中所有的整数按照升序依次保存在contents数组中，结构如图：

![1653985149557](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985149557.png)

现在，数组中每个数字都在int16_t的范围内，因此采用的编码方式是INTSET_ENC_INT16，每部分占用的字节大小为：
encoding：4字节
length：4字节
contents：2字节 * 3  = 6字节

![1653985197214](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985197214.png)

##### 有序与唯一

我们向该其中添加一个数字：50000，这个数字超出了int16_t的范围，intset会自动升级编码方式到合适的大小。
以当前案例来说流程如下：

* 升级编码为INTSET_ENC_INT32, 每个整数占4字节，并按照新的编码方式及元素个数扩容数组
* 倒序依次将数组中的元素拷贝到扩容后的正确位置
* 将待添加的元素放入数组末尾
* 最后，将inset的encoding属性改为INTSET_ENC_INT32，将length属性改为4

![1653985276621](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985276621.png)

源码如下：

![1653985304075](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985304075.png)

普通插入：如果编码没有越界，先在set中查找，查找的过程中进行pos的赋值，大于max，pos=length，小于最小值，pos = 0，然后开始二分查找，pos的结果就是最后的mid/left ，查到就不插入，查不到就能进行插入

##### 升级

![1653985327653](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985327653.png)

升级：新元素肯定在队首或队尾，倒序遍历把旧元素整体搬运，最后将新元素插入

#### 复习C语言指针

> 指针: 本质就是一块连续内存区域的头地址。
>
> 字符数组: `char *p = {'b','r','a','i','n','\0'};` 指针变量p指向字符数组(的头地址)
>
> 字符串数组: `char *array[] = {"abandon","brain","certain"};` array是一个指针数组的头地址（指针的地址，也就是指针的指针——二级指针）
>
> 二级指针: `char **q = array;` q是指向指针的指针，也就是二级指针之间的直接赋值。

#### 总结

Intset可以看做是特殊的整数数组，具备一些特点：

* Redis会确保Intset中的元素唯一、有序
* 具备类型升级机制，可以节省内存空间
* 底层采用二分查找方式来查询

### Dict

我们知道Redis是一个键值型（Key-Value Pair）的数据库，我们可以根据键实现快速的增删改查。而键与值的映射关系正是通过Dict来实现的。
Dict由三部分组成，分别是：哈希表（DictHashTable）、哈希节点（DictEntry）、字典（Dict）

![1653985396560](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985396560.png) 

当我们向Dict添加键值对时，Redis首先根据key计算出hash值（h），然后利用 h & sizemask来计算元素应该存储到数组中的哪个索引位置。我们存储k1=v1，假设k1的哈希值h =1，则1&3 =1，因此k1=v1要存储到数组角标1位置。

![1653985497735](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985497735.png)

Dict由三部分组成，分别是：哈希表（DictHashTable）、哈希节点（DictEntry）、字典（Dict）

![1653985570612](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985570612.png)

#### size = 2^n^ 

<mark>size<mark>

- 求余操作实际上就是截取低位，十进制如果除以10的n次方就是直接截取低位，而对于计算机二进制明显更有效率，所以截取二进制的低n位

- size = 2^n^ sizemask = 2^n^ - 1
- Hash % size 相当于截取自己的低n位，而要想获取低n位，最简单的办法就是拿n个1跟Hash进行与操作
- Hash & sizemask  = Hash % size
- 当哈希表的大小是2的n次方时，哈希函数能够更好地将数据分布在哈希表的各个位置上，从而减少哈希冲突的概率。`capacity - 1` 的二进制表示全部为 1（如 15 为 1111），这样能让低位的哈希值充分参与运算，最大程度分散数据，降低冲突概率。
- **负载因子调整**： 在很多哈希表实现中（如Java的 `HashMap` 或 `Redis`），当负载因子超过一定阈值时，哈希表的大小会动态扩展。如果哈希表的大小是2的n次方，那么扩展时的大小也会是2的n次方（如从 16 扩展到 32），这使得扩展过程更加简单且高效。
- 最小是4 `DICT_HT_INITIAL_SIZE` 

#### Entry Table Dict 数据结构

<mark>dictEntry<mark>

- dictEntry是自定义的一个数据结构，dictEntry *p 表示一个指向dictEntry的指针，指向分配给一个entry的连续内存区域的头地址，dictEntry是最底层的键值对元素

<mark>dictTable<mark>

- dictEntry **table 就表示二级指针，这个指针指向entry指针，又因为指针指向的区域都是一片连续的内存区域，所以就是指针数组。dictTable本质是entry数组，根据key把键值对存到数组的对应索引处。

![1653985586543](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985586543.png)

- 发生冲突，采用链地址法解决

<mark>dict<mark> 

- dict里有两个hashTable 

![1653985640422](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985640422.png)

- type 哈希函数的类型
- ht 两张hashtable，多出的一张表用于rehash时数据的暂存
- rehashidx，pauserehash rehash过程标记

#### Dict的伸缩

Dict中的HashTable就是数组结合单向链表的实现，当集合中元素较多时，必然导致哈希冲突增多，链表过长，则查询效率会大大降低。

<mark>扩容<mark>：Dict在每次新增键值对时都会检查负载因子（LoadFactor = used/size） ，满足以下两种情况时会触发扩容：

- 哈希表的`LoadFactor>= 1`，并且服务器没有执行 BGSAVE(RDB持久化) 或者 BGREWRITEAOF(AOF持久化) 等后台进程；
- 哈希表的`LoadFactor > 5`； 

扩容实际上扩到比used+1大的第一个2^n^ 

![1653985716275](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985716275.png)

<mark>收缩<mark>：SIZE>4 && LoadFactor < 0.1 (used*100避免浮点运算 `HASHTABLE_MIN_FILL=10` )

实际上容量为比used大的第一个2^n^ （`used>=4`）

![image-20241113202920178](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241113202920178.png)



<mark>DICTEXPAND<mark>这些实际上是申请了一个新的数组，如果不是初始化，还要rehash将旧数据装到新的数组中

rehashidx = 0，表示这个dict开始rehash

![image-20241113205217079](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241113205217079.png)

#### **Dict的渐进式rehash** 

不管是扩容还是收缩，必定会创建新的哈希表，导致哈希表的size和sizemask变化，而key的查询与sizemask有关。因此必<mark>须对哈希表中的每一个key重新计算索引，插入新的哈希表，这个过程称为rehash<mark>。过程是这样的：

* 计算新hash表的`realSize`，值取决于当前要做的是扩容还是收缩：
  * 如果是扩容，则`realSize`为第一个大于等于dict.ht[0].used + 1的2^n
  * 如果是收缩，则`realSize`为第一个大于等于dict.ht[0].used的2^n （不得小于4）
* 按照新的`realSize`申请内存空间，创建dictht，并赋值给dict.ht[1]
* 设置dict.`rehashidx` = 0，标示开始rehash
* ~~将dict.ht[0]中的每一个dictEntry都rehash到dict.ht[1]~~ 
* <mark>在rehash过程中，增删改查操作都会检查dict是否处于rehash状态（rehashidx）<mark>新增操作，则直接写入ht[1]，查询、修改和删除则会在dict.ht[0]和dict.ht[1]依次查找并执行。这样可以确保ht[0]的数据只减不增，随着rehash逐渐进行慢慢变成空数组
* 将dict.ht[1]赋值给dict.ht[0]，
* 给dict.ht[1]初始化为空哈希表，释放原来的dict.ht[0]的内存，并将`rehashidx`赋值为-1，代表rehash结束

整个过程可以描述成：

![1653985824540](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985824540.png)

#### 总结

Dict的结构：

* 类似java的HashTable，底层是数组加链表来解决哈希冲突（链地址法）
* Dict包含两个哈希表，ht[0]平常用，ht[1]用来rehash

Dict的伸缩：

* 当LoadFactor大于5或者LoadFactor大于1并且没有子进程任务时，Dict扩容
* 当LoadFactor小于0.1时，Dict收缩
* 扩容大小为第一个大于等于 `used + 1` 的2^n^
* 收缩大小为第一个大于等于 `used` 的2^n^ 
* Dict采用**<mark>渐进式<mark>**rehash，每次访问Dict时执行一次rehash
* rehash时ht[0]只减不增，新增操作只在ht[1]执行，其它操作在两个哈希表

### ZipList

[深入分析redis之quicklist，不一样的ziplist使用方式？ - 掘金 (juejin.cn)](https://juejin.cn/post/7093145133368999943)

ZipList 是一种特殊的“双端链表” ，由一系列特殊编码的<mark>连续内存块<mark>（不需要通过指针寻址）组成。可以在任意一端进行压入/弹出操作, 并且该操作的时间复杂度为 O(1)。

![1653985987327](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653985987327.png)

![1653986020491](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986020491.png)

| **属性** | **类型** | **长度** | **用途**                                                     |
| -------- | -------- | -------- | ------------------------------------------------------------ |
| zlbytes  | uint32_t | 4 字节   | 记录整个压缩列表占用的内存字节数                             |
| zltail   | uint32_t | 4 字节   | 记录压缩列表表尾节点距离压缩列表的起始地址有多少字节，通过这个偏移量，可以确定表尾节点的地址。 |
| zllen    | uint16_t | 2 字节   | 记录了压缩列表包含的节点个数。 最大值为UINT16_MAX （65534），如果超过这个值，此处会记录为65535，但节点的真实数量需要遍历整个压缩列表才能计算得出。 |
| entry    | 列表节点 | 不定     | 压缩列表包含的各个节点，节点的长度由节点保存的内容决定。     |
| zlend    | uint8_t  | 1 字节   | 特殊值 0xFF （十进制 255 ），用于标记压缩列表的末端。        |

#### **ZipListEntry**

ZipList 中的Entry并不像普通链表那样记录前后节点的指针，因为记录两个指针要占用16个字节，浪费内存。而是采用了下面的结构：

![1653986055253](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986055253.png)

* previous_entry_length：前一节点的长度，占1个或5个字节。
  * 如果前一节点的长度小于254字节，则采用1个字节来保存这个长度值
  * 如果前一节点的长度大于254字节，则采用5个字节来保存这个长度值，第一个字节为0xfe，后四个字节才是真实长度数据

* encoding：编码属性，记录content的数据类型（字符串还是整数）以及长度，占用1个、2个或5个字节
* contents：负责保存节点的数据，可以是字符串或整数

**正向遍历**：current + sizeof(previous_entry_length) + sizeof(encoding) + encoding.contentLength

**逆向遍历**：current - previous_entry_length

##### 大小端存储

ZipList中所有<mark>存储长度的数值：tlbytes,tltail,tllen,previous_entry_length<mark> 均采用<mark>小端字节序<mark>即低位字节在前，高位字节在后。例如：数值0x1234，采用小端字节序后实际存储值为：0x3412 

> LSB（Least Significant Byte）和MSB（Most Significant Byte）分别表示数据的最低有效字节和最高有效字节。它们在数据存储和处理时起到重要作用，尤其在大端字节序和小端字节序的不同存储方式中。
>
> - **LSB（最低有效字节）**：存储数据时，代表数据的最低有效字节，即值最小的字节。通常对应数据的最低位。
> - **MSB（最高有效字节）**：存储数据时，代表数据的最高有效字节，即值最大的字节。通常对应数据的最高位。
>
> 假设我们有一个4字节（32位）整数`0x12345678`，它的二进制表示为：
>
> ```pascal
> 0001 0010 0011 0100 0101 0110 0111 1000
> ```
>
> 我们将这个值从内存地址`0x1000`开始存储，来看不同字节序下 LSB 和 MSB 的位置。
>
> **小端字节序存储**
>
> 在小端模式下，LSB 存放在最低地址，MSB 存放在最高地址。也就是说，低地址存放低位字节，高地址存放高位字节。从最低位开始存。
>
> | 地址(Hex) | 数据 |
> | --------- | ---- |
> | 0x1000    | 78   |
> | 0x1001    | 56   |
> | 0x1002    | 34   |
> | 0x1003    | 12   |
>
> 在这种情况下，存储顺序为 `78 56 34 12`。
>
> - **便于数值操作**：在小端模式下，最低有效字节存放在最低地址，因此读取数值时，从低地址开始逐字节读取即可，省去了对字节顺序的额外处理。对于需要频繁数值计算的处理器（如x86架构），这种字节序更高效。
> - **简化某些数据类型的转换**：例如，将16位的`short`扩展成32位的`int`，只需将高位填零，不需要移动低位数据。
>
> - **人类阅读不直观**：小端模式存储的数据不符合从高到低的阅读习惯，直接查看数据时可能显得混乱。
>
> **大端字节序存储**
>
> 在大端模式下，MSB 存放在最低地址，LSB 存放在最高地址。也就是说，低地址存放高位字节，高地址存放低位字节。从最高位开始存
>
> | 地址（Hex） | 数据 |
> | ----------- | ---- |
> | 0x1000      | 12   |
> | 0x1001      | 34   |
> | 0x1002      | 56   |
> | 0x1003      | 78   |
>
> 在这种情况下，存储顺序为 `12 34 56 78`。
>
> - **符合人类阅读习惯**：大端模式将最高有效字节放在低地址，类似于人类阅读从高位到低位的顺序，因此直接查看数据更直观。
> - **统一网络字节序**：大端字节序是网络协议的标准（网络字节序），在跨平台通信时无需转换，适用于网络应用。
> - **计算复杂度稍高**：对于低地址优先访问的处理器，大端模式的数值计算可能需要更多的字节重排操作，不如小端模式高效。

##### Encoding编码—记录content长度

ZipListEntry中的encoding编码分为字符串和整数两种：

- 字符串：如果encoding是以“00”、“01”或者“10”开头，则证明content是字符串，不同encoding表示不同的字符串长度

| **编码**（bit）                                              | **编码长度** | **字符串大小**      |
| ------------------------------------------------------------ | ------------ | ------------------- |
| \| 00pppppp \|                                               | 1 bytes      | <= 63 bytes         |
| \| 01pppppp \| qqqqqqqq \|                                   | 2 bytes      | <= 16383 bytes      |
| \| 10000000 \| qqqqqqqq \| rrrrrrrr \| ssssssss \| tttttttt \| | 5 bytes      | <= 4294967295 bytes |

例如，我们要保存字符串：“ab”和 “bc”

![1653986172002](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986172002.png)

tlbytes,tltail,tllen 

ZipListEntry中的encoding编码分为字符串和整数两种：

* 整数：如果encoding是以“11”开始，则证明content是整数，且encoding固定只占用1个字节,不同的encoding表示不同数据类型，也就知道了长度

| **编码** | **编码长度** | **整数类型**                                                 |
| -------- | ------------ | ------------------------------------------------------------ |
| 11000000 | 1            | int16_t（2 bytes）                                           |
| 11010000 | 1            | int32_t（4 bytes）                                           |
| 11100000 | 1            | int64_t（8 bytes）                                           |
| 11110000 | 1            | 24位有符整数(3 bytes)                                        |
| 11111110 | 1            | 8位有符整数(1 bytes)                                         |
| 1111xxxx | 1            | 直接在xxxx位置存数，范围从0001~1101，减1后结果为实际值（0到12）节约内存的极致 |

![1653986282879](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986282879.png)

![1653986217182](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986217182.png)



#### ZipList的连锁更新问题

ZipList的每个Entry都包含previous_entry_length来记录上一个节点的大小，长度是1个或5个字节：
如果前一节点的长度小于254字节，则采用1个字节来保存这个长度值
如果前一节点的长度大于等于254字节，则采用5个字节来保存这个长度值，第一个字节为0xfe，后四个字节才是真实长度数据
现在，假设我们有N个连续的、长度为250~253字节之间的entry，因此entry的previous_entry_length属性用1个字节即可表示，如图所示：正好插入一个254字节的entry导致后面的previousLen全部都变化了

![1653986328124](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986328124.png)

ZipList这种特殊情况下产生的连续多次空间扩展操作称之为连锁更新（Cascade Update）。新增、删除都可能导致连锁更新的发生。

频繁申请、销毁内存  性能开销很大

#### 总结

**ZipList特性：**

优点：

* 压缩列表的可以看做一种连续内存空间的"双向链表"，不使用指针，所以不是真正意义上的链表
* 列表的节点之间不是通过指针连接，而是记录上一节点和本节点长度来寻址，内存占用较低。

缺点：

* 逐个遍历，如果列表数据量tllen过多，导致链表过长，可能影响查询性能
* 可能会发生频繁的内存申请销毁导致频繁内核态切换，资源开销较大
* 增或删较大数据时有可能发生连续更新问题

**ziplist** 的不足主要在于当 ziplist 中元素个数过多，它的查找效率就会降低。而且如果在 ziplist 里新增或修改数据，ziplist 占用的内存空间还需要**重新分配**；更糟糕的是，ziplist 新增某个元素或修改某个元素时，可能会导致后续元素的 prevlen 占用空间都发生变化，从而引起**连锁更新**问题，导致每个元素的空间都要重新分配，这就会导致 ziplist 的访问性能下降。

### QuickList（双端链表+压缩列表）

[深入分析redis之quicklist，不一样的ziplist使用方式？ - 掘金 (juejin.cn)](https://juejin.cn/post/7093145133368999943) 

问题1：ZipList虽然节省内存，但申请内存必须是连续空间，如果内存占用较多，申请内存效率很低。怎么办？

​	答：为了缓解这个问题，我们必须限制ZipList的长度和entry大小。

问题2：但是我们要存储大量数据，超出了ZipList最佳的上限该怎么办？

​	答：我们可以创建多个ZipList来<mark>分片<mark>存储数据。

问题3：数据拆分后比较分散，不方便管理和查找，这多个ZipList如何建立联系？

​	答：Redis在3.2版本引入了新的数据结构QuickList，它是一个双端链表，只不过链表中的每个节点都是一个ZipList。

![1653986474927](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986474927.png)

为了避免QuickList中的每个ZipList中entry过多，Redis提供了一个配置项：list-max-ziplist-size来限制。
如果值为正，则代表ZipList的允许的entry个数的最大值
如果值为负，则代表ZipList的最大内存大小，分5种情况：

* -1：每个ZipList的内存占用不能超过4kb
* -2：每个ZipList的内存占用不能超过8kb
* -3：每个ZipList的内存占用不能超过16kb
* -4：每个ZipList的内存占用不能超过32kb
* -5：每个ZipList的内存占用不能超过64kb

其默认值为 -2：

![1653986642777](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986642777.png)

以下是QuickList的和QuickListNode的结构源码：

![1653986667228](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986667228.png)

我们接下来用一段流程图来描述当前的这个结构

![1653986718554](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986718554.png)



#### 总结

QuickList的特点：

* 是一个节点为ZipList的双端链表
* 节点采用ZipList，解决了传统链表的内存占用问题
* 控制了ZipList大小，解决连续内存空间申请效率问题
* 中间节点可以压缩，进一步节省了内存

### Listpack

[Redis7代码分析阅读总结一：listpack - 个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000041670843)

[深入分析redis之listpack，取代ziplist? - 掘金 (juejin.cn)](https://juejin.cn/post/7093530299866284045)

解决了ZipList的连锁更新问题

### SkipList（加强链表）

SkipList（跳表）首先是链表，但与传统链表相比有几点差异：

- 元素按照SCORE值升序排列存储
- 节点可能包含多个指针，指针跨度不同。

#### 结构

![1653986771309](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986771309.png)

![1653986813240](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986813240.png)

![1653986877620](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986877620.png)

#### 总结

SkipList的特点：

* 跳跃表是一个双向链表，每个节点都包含score和ele值（sds字符串）
* 节点按照score值排序，score值一样则按照ele字典排序。
* 每个节点都可以包含多层指针，<mark>层数是1到32之间的随机数<mark> 这种**随机**性避免了平衡树中频繁的旋转或重构操作。
  * **抛硬币法**：抛硬币法是一种经典的随机算法，假设每次抛硬币有 50% 的概率使当前节点新增一层，直到硬币正面朝上或达到最大层数。该算法实现简单且符合概率分布。
  * **概率分布法**：概率分布法采用伪随机数生成器，预先设置一个层数分布表，以确保生成的层数具有严格的概率性分布。相较于抛硬币法，概率分布法更为精准，能够进一步优化跳表的性能。

* 不同层指针到下一个节点的跨度不同，层级越高，跨度越大
* 增删改查效率与红黑树基本一致，实现却更简单



#### 跳表 vs Trees

##### AVL Tree vs SkipList

平衡树：AVL Tree 的插入、删除和查询的时间复杂度和跳表一样都是 **O(log n)** 。但是每一次插入或者删除操作都需要保证整颗树左右节点的绝对平衡，只要不平衡就要通过旋转操作来保持平衡，这个过程是比较耗时的。

而跳表是一种可以用来代替平衡树的数据结构。跳表使用概率平衡而不是严格强制的平衡，因此，跳表中的插入和删除算法比平衡树的等效算法简单得多，速度也快得多。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/202401222005312.png)

##### 红黑树 vs SkipList

红黑树：Red Black Tree 也是一种自平衡二叉查找树，它的查询性能略微逊色于 AVL 树，但插入和删除效率更高。红黑树的插入、删除和查询的时间复杂度和跳表一样都是 **O(log n)** 。红黑树是一个**黑平衡树**，即从任意节点到另外一个叶子叶子节点，它所经过的黑节点是一样的。当对它进行插入操作时，需要通过旋转和染色（红黑变换）来保证黑平衡。不过，相较于 AVL 树为了维持平衡的开销要小一些。关于红黑树的详细介绍，可以查看这篇文章：[红黑树](https://javaguide.cn/cs-basics/data-structure/red-black-tree.html)。

跳表的实现也更简单一些。并且，按照区间来查找数据这个操作，红黑树的效率没有跳表高。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/202401222005709.png)

##### B+ Tree vs SkipList

B+树更适合作为数据库和文件系统中常用的索引结构之一，它的核心思想是通过可能少的 IO 定位到尽可能多的索引来获得查询数据。Redis对于文件IO不敏感，只需按照概率进行随机维护即可，节约内存。而且使用跳表实现 zset 时相较前者来说更简单一些，在进行插入时只需通过索引将数据插入到链表中合适的位置再随机维护一定高度的索引即可，也不需要像 B+树那样插入时发现失衡时还需要对节点分裂与合并。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/202401222005649.png)

##### Redis作者的理由

1、它们不是很占用内存。这主要取决于你。改变节点拥有给定层数的概率的参数，会使它们比 B 树更节省内存。

2、有序集合经常是许多 ZRANGE 或 ZREVRANGE 操作的目标，也就是说，以链表的方式遍历跳表。通过这种操作，跳表的缓存局部性至少和其他类型的平衡树一样好。

3、它们更容易实现、调试等等。例如，由于跳表的简单性，我收到了一个补丁（已经在 Redis 主分支中），用增强的跳表实现了 O(log(N))的 ZRANK。它只需要对代码做很少的修改。





## Redis 数据类型

### RedisObject

Redis中的任意数据类型的键和值都会被封装为一个RedisObject，也叫做Redis对象，源码如下：

1、什么是redisObject：
从Redis的使用者的角度来看，⼀个Redis节点包含多个database（非cluster模式下默认是16个，cluster模式下只能是1个），而一个database维护了从key space到object space的映射关系。这个映射关系的key是string类型，⽽value可以是多种数据类型，比如：
string, list, hash、set、sorted set等。我们可以看到，**key的类型固定是string，而value可能的类型是多个**。
⽽从Redis内部实现的⾓度来看，database内的这个映射关系是**用⼀个dict**来维护的。dict的key固定用⼀种数据结构来表达就够了，这就是动态字符串sds。而value则比较复杂，为了在同⼀个dict内能够存储不同类型的value，这就需要⼀个通⽤的数据结构，这个通用的数据结构就是robj，全名是redisObject。

一个database对应一个dict，其中entry就是key(string)和value(五种基本数据结构类型)的对应关系

需要用RedisObject囊括value

![1653986956618](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653986956618.png)

Redis的编码方式

Redis中会根据存储的数据类型不同，选择不同的编码方式，共包含11种不同类型：

| **编号** | **编码方式**            | **说明**                   |
| -------- | ----------------------- | -------------------------- |
| 0        | OBJ_ENCODING_RAW        | raw编码动态<mark>字符串<mark>      |
| 1        | OBJ_ENCODING_INT        | long类型的整数的<mark>字符串<mark> |
| 2        | OBJ_ENCODING_HT         | hash表（<mark>字典dict<mark>）     |
| 3        | OBJ_ENCODING_ZIPMAP     | 已废弃                     |
| 4        | OBJ_ENCODING_LINKEDLIST | 双端链表                   |
| 5        | OBJ_ENCODING_ZIPLIST    | <mark>压缩列表<mark>               |
| 6        | OBJ_ENCODING_INTSET     | <mark>整数集合<mark>               |
| 7        | OBJ_ENCODING_SKIPLIST   | <mark>跳表<mark>                   |
| 8        | OBJ_ENCODING_EMBSTR     | embstr的动态<mark>字符串<mark>     |
| 9        | OBJ_ENCODING_QUICKLIST  | <mark>快速列表<mark>               |
| 10       | OBJ_ENCODING_STREAM     | Stream流                   |

五种数据结构：

Redis中会根据存储的数据类型不同，选择不同的编码方式。每种数据类型的使用的编码方式如下：

| **数据类型** | **编码方式**                                       |
| ------------ | -------------------------------------------------- |
| OBJ_STRING   | int、embstr、raw                                   |
| OBJ_LIST     | LinkedList和ZipList(3.2以前)、QuickList（3.2以后） |
| OBJ_SET      | intset、HT                                         |
| OBJ_ZSET     | ZipList、HT、SkipList                              |
| OBJ_HASH     | ZipList、HT                                        |

### String（RAW / EMBSTR / INT）

String是Redis中最常见的数据存储类型：

- 其基本编码方式是<mark>ENCODING_RAW<mark>，基于简单动态字符串（SDS）实现，存储上限为512mb。

- 如果存储的SDS长度<mark>小于44<mark>字节，则会采用<mark>ENCODING_EMBSTR<mark>(Embedded String)编码，此时object head与SDS是一段连续空间。申请内存时只需要调用一次内存分配函数，<mark>效率更高。<mark>
  - 底层采用Jemalloc分配内存，2^n^效率更高，4+8+44+4 = 64
  - ![image-20241114163342533](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241114163342533.png)

底层实现⽅式：动态字符串 SDS 或 long 

String的内部存储结构⼀般是SDS（Simple Dynamic String，可以动态扩展内存）

但是如果⼀个String类型的value的值是数字，那么Redis内部会把它转成long类型来存储，从⽽减少内存的使用。

![1653987103450](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653987103450.png)

如果存储的字符串是整数值，并且大小在LONG_MAX范围内，则会采用<mark>ENCODING_INT<mark>编码：直接将数据保存在RedisObject的ptr指针位置（刚好8字节），不再需要SDS了。

![1653987159575](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653987159575.png)

![1653987202522](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653987202522.png)

确切地说，String在Redis中是⽤⼀个robj来表示的。

用来表示String的robj可能编码成3种内部表⽰：OBJ_ENCODING_RAW，OBJ_ENCODING_EMBSTR，OBJ_ENCODING_INT。
其中前两种编码使⽤的是sds来存储，最后⼀种OBJ_ENCODING_INT编码直接把string存成了long型。
在对string进行incr, decr等操作的时候，如果它内部是OBJ_ENCODING_INT编码，那么可以直接行加减操作；如果它内部是OBJ_ENCODING_RAW或OBJ_ENCODING_EMBSTR编码，那么Redis会先试图把sds存储的字符串转成long型，如果能转成功，再进行加减操作。对⼀个内部表示成long型的string执行append, setbit, getrange这些命令，针对的仍然是string的值（即⼗进制表示的字符串），而不是针对内部表⽰的long型进⾏操作。比如字符串”32”，如果按照字符数组来解释，它包含两个字符，它们的ASCII码分别是0x33和0x32。当我们执行命令setbit key 7 0的时候，相当于把字符0x33变成了0x32，这样字符串的值就变成了”22”。⽽如果将字符串”32”按照内部的64位long型来解释，那么它是0x0000000000000020，在这个基础上执⾏setbit位操作，结果就完全不对了。因此，在这些命令的实现中，会把long型先转成字符串再进行相应的操作。

![image-20241114163612101](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241114163612101.png)

总结：字符串尽量控制在44字节以内

### List（Quicklist）

Redis的List类型可以从首、尾操作列表中的元素：

![1653987240622](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653987240622.png)

哪一个数据结构能满足上述特征？

* LinkedList ：普通链表，可以从双端访问，内存占用较高，内存碎片较多
* ZipList ：压缩列表，可以从双端访问，内存占用低，存储上限低
* QuickList：LinkedList + ZipList，可以从双端访问，内存占用较低，包含多个ZipList，存储上限高

#### RedisObject 结构

Redis的List结构类似一个双端链表，可以从首、尾操作列表中的元素：

![image-20241114164451953](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241114164451953.png)

在3.2版本之前，Redis采用ZipList和LinkedList来实现List，当元素数量小于512并且元素大小小于64字节时采用ZipList编码，超过则采用LinkedList编码。

在3.2版本之后，Redis统一采用QuickList来实现List：

![1653987313461](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653987313461.png)

#### 创建与插入

![image-20241114174543958](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241114174543958-1731774561497-3.png)

![image-20241114174533836](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241114174533836.png)

### Set（Intset / Hashtable）

Set是Redis中的单列集合，满足下列特点：

* 不保证有序性
* 保证元素唯一
* 求交集、并集、差集

![1653987342550](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653987342550.png)

可以看出，Set对查询元素的效率要求非常高，思考一下，什么样的数据结构可以满足？
HashTable，也就是Redis中的<mark>Dict<mark>，不过Dict是双列集合（可以存键、值对）

同一个key计算的索引肯定一样，所以也能确保key唯一

Set是Redis中的集合，不一定确保元素有序，可以满足元素唯一、查询效率要求极高。

- 为了查询效率和唯一性，set采用HT编码（Dict）。Dict中的key用来存储元素，value统一为null。
- 当存储的所有数据都是整数，并且元素数量不超过set-max-intset-entries时，Set会采用IntSet编码，以节省内存

#### RedisObject 结构

![1653987454403](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653987454403.png)

#### 创建或插入对编码的影响

<mark>根据创建set时添加的第一个元素判断使用哪种编码格式<mark> 如果是intset编码，需要根据插入的元素判断是否转换数据结构

![1653987388177](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653987388177.png)

![image-20241114180238855](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241114180238855.png)



### Zset（Skiplist+Hashtable）

ZSet也就是SortedSet，其中每一个元素都需要指定一个score值和member值：

* 可以根据score值排序后
* member必须唯一
* 可以根据member查询分数

![1653992091967](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653992091967.png)

#### RedisObject 结构

因此，zset底层数据结构必须满足键值存储、键必须唯一、可排序这几个需求。之前学习的哪种编码结构可以满足？

* SkipList：可以排序，并且可以同时存储score和ele值（member）
* HT（Dict）：可以键值存储，并且可以根据key找value(SCORE)

![1653992121692](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653992121692.png)

![1653992172526](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653992172526.png)

#### 数据量小的优化：Ziplist

当元素数量不多时，HT和SkipList的优势不明显，而且更耗内存。因此zset还会采用ZipList结构来节省内存，不过需要同时满足两个条件。

* 元素数量小于zset_max_ziplist_entries，默认值128
* 每个元素都小于zset_max_ziplist_value字节，默认值64

ziplist本身没有排序功能，而且没有键值对的概念，因此需要有zset通过编码实现：

* ZipList是连续内存，因此score和element是紧挨在一起的两个entry， element在前，score在后
* score越小越接近队首，score越大越接近队尾，按照score值升序排列

![1653992299740](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653992299740.png)



#### 创建或插入对编码的影响

<mark>创建redisobject时，就会根据参数判断采用哪种结构，之后每次插入都会做判断是否需要更改编码类型<mark>  

![1653992238097](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653992238097.png)

### Hash（Hashtable）

hash结构如下：

![1653992339937](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653992339937.png)

zset集合如下：

![1653992360355](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653992360355.png)

Hash结构与Redis中的Zset非常类似：

* 都是键值存储
* 都需求根据键获取值
* 键必须唯一

区别如下：

* zset的键是member，值是score；hash的键和值都是任意值
* zset要根据score排序；hash则无需排序

#### RedisObject 结构

因此，Hash底层采用的编码与Zset也基本一致，只需要把排序有关的SkipList去掉即可

![1653992413406](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653992413406.png)

#### 数据量小的优化：Ziplist

Redis 会用 **ziplist（压缩列表）** 存储，以节省空间。ziplist 是一种连续内存块结构，适合小数据量。

Redis 3.2 后 ziplist 被 **quicklist** 替代，到了 Redis 5.0+，ZSet 的底层小数据结构是 **listpack**（更加高效）。



Hash结构默认采用ZipList编码，用以节省内存。 ZipList中相邻的两个entry 分别保存field和value。

随着数据的增加，底层的ziplist就可能会转成dict，具体配置如下：

- hash-max-ziplist-entries 512 元素个数

- hash-max-ziplist-value 64 元素大小

当满足上面两个条件其中之⼀的时候，Redis就使⽤dict字典来实现hash。
Redis的hash之所以这样设计，是因为当ziplist变得很⼤的时候，它有如下几个缺点：

* 每次插⼊或修改引发的realloc操作会有更⼤的概率造成内存拷贝，从而降低性能。
* ⼀旦发生内存拷贝，内存拷贝的成本也相应增加，因为要拷贝更⼤的⼀块数据。
* 当ziplist数据项过多的时候，在它上⾯查找指定的数据项就会性能变得很低，因为ziplist上的查找需要进行遍历。

ziplist这种结构并不擅长做修改操作。⼀旦数据发⽣改动，就会引发内存realloc，可能导致内存拷贝。

#### 创建或插入对编码的影响 

<mark>创建redisobject默认采用ZIPLIST，之后根据元素大小做判断是否需要转换成DICT，在真正插入后判断长度是否需要转换成DICT<mark> 

![image-20241117001037075](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241117001037075.png)

![image-20241117003843384](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241117003843384.png)



当 ZSet 的元素数量很少（默认小于 128 个），Redis 会用 **ziplist（压缩列表）** 存储，以节省空间。ziplist 是一种连续内存块结构，适合小数据量。

Redis 3.2 后 ziplist 被 **quicklist** 替代，到了 Redis 5.0+，ZSet 的底层小数据结构是 **listpack**（更加高效）。

## Redis网络模型

> [Socket Programming](https://scatteredream.github.io/2025/05/29/net-io-socket/)

### Redis 单线程？

#### 到底是单线程还是多线程？

* 如果仅仅聊Redis的核心业务部分（命令处理），答案是单线程。
* 如果是聊整个Redis，那么答案就是多线程。

在Redis版本迭代过程中，在两个重要的时间节点上引入了多线程的支持：

* Redis v4.0：引入多线程异步处理一些耗时较长的任务，例如异步删除命令unlink(另开一个线程删除bigKey)
* Redis v6.0：在核心网络模型中引入多线程，进一步提高对于多核CPU的利用率

因此，对于Redis的核心网络模型，在Redis 6.0之前确实都是单线程。是利用epoll（Linux系统）这样的IO多路复用技术在事件循环中不断处理客户端情况。

#### **为什么Redis要选择单线程**/单线程为什么这么快？

* 基于内存：抛开持久化不谈，Redis是**纯内存**操作，执行速度非常快，它的**性能瓶颈是网络延迟而不是执行速度，因此多线程并不会带来巨大的性能提升**。

* 数据结构高效：SDS简单字符串、dict哈希结构、skiplist跳表、ziplist压缩列表

  * String：int、raw、embstr
  * List：quicklist
  * Hash：ht、ziplist
  * Set：intset、ht
  * Zset：skiplist、ziplist/ht

* 多线程的问题：

  * 多线程会导致过多的**上下文切换**，带来**不必要的开销**。
  * 引入多线程会面临**线程安全**问题，必然要引入线程锁这样的安全手段，实现复杂度增高，而且性能也会大打折扣。

* 高性能：IO多路复用，多个网络连接使用同一个线程来建立

* 虚拟内存机制：Redis直接自己构建了VM机制 ，不会像一般的系统会调用系统函数处理，会浪费一定的时间去移动和请求。

  > 虚拟内存机制就是暂时把不经常访问的数据(冷数据)从内存交换到磁盘中，从而腾出宝贵的内存空间用于其它需要访问的数据(热数据)。通过VM功能可以实现冷热数据分离，使热数据仍在内存中、冷数据保存到磁盘。这样就可以避免因为内存不足而造成访问速度下降的问题。

### Reactor: 事件驱动的 I/O 模型

#### AE 事件库

C/C++ 指针与引用

> | 运算符 | 用途                       | 使用对象                   | 示例                             |
> | ------ | -------------------------- | -------------------------- | -------------------------------- |
> | `.`    | 访问对象的成员 C++         | 非指针对象                 | `obj.value`, `obj.print()`       |
> | `->`   | 通过指针访问对象的成员C    | 指针对象                   | `ptr->value`, `ptr->print()`     |
> | `::`   | 访问特定作用域中的成员 C++ | 命名空间、类、全局作用域等 | `std::cout`, `ClassName::member` |
>
> `int& ref = x;`引用变量，得到x的引用，交给ref引用。（only in c++，弱化版指针）
>
> `int* ptr = &x;`指针变量，取x的地址，赋值给ptr指针。
>
> `int val = x;` 普通变量，将x的值复制到val中。
>
> 访问x： ref 或 *ptr

*A simple Event-driven programming library* 

[Redis源码分析（二十）--- ae事件驱动_aeCreateEventLoop setSize-CSDN博客](https://blog.csdn.net/androidlushangderen/article/details/40474815)

Redis做了跨平台整合，把不同OS的IO多路复用函数封装到统一的API中——AE

![image-20241116140955535](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241116140955535.png)

> ![image-20241116145235220](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241116145235220.png)
>
> 1. 创建监听socket，并开始监听这个socket，acceptTcpHandler用于监听通信socket的可读事件的回调函数
> 2. 设置`beforeSleepProcessor`，用于sleep之前的一些准备工作，一旦开始epoll_wait如果没有就绪就会sleep。
> 3. 在`aeProcessEvents`中处理事件，首先调用`beforeSleep`，然后开始`epoll_wait`等待就绪socketFD
> 4. 拿到就绪的socketFD集合之后，遍历处理，分别调用对应的不同的事件处理器(每个socket事件类型不同，处理器也不同)

#### + epoll + 命令处理模型

##### 处理器

> `acceptTcpHandler`：TCP连接建立处理器
>
> - 大名鼎鼎的accept函数返回已经建立连接的socket FD，
> - 然后会创建一个connection关联此socket，监听socketFD可读事件，把命令读取处理器（回调函数）绑定到socketFD上
> - 随后才能开始socket的IO操作
>
> ![image-20241116150329753](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241116150329753.png)
>
> `readQueryFromClient:`命令读取处理器
>
> - 获取命令：client（绑定了socketFD）具备读写缓冲区，从缓冲区中获取字节流，转换成SDS字符串并存入client->argv[] 
> - 处理命令：命令由多个SDS构成，从缓冲区中读取之后存入一个ARGV数组中，`set name Jack` lookUpCommand 先要读取命令的类型，然后通过查找`set -> setCommand(client *c)（指针）`的映射表来确定要执行命令的具体函数，随后proc执行回调函数。这里也体现出回调函数的优越性：充分解耦。
> - 返回命令：addReply将执行结果作为SDS写到缓冲区中，满则写入链表。
> - 写回命令：最后将客户端加入待写出的队列中。
>
> ![image-20241116153602252](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241116153602252.png)
>
> `sendReplyFromClient`: 执行结果写回处理器
>
> 之前的beforeSleep，在正式开始监听事件之前，会遍历上文的待写队列，
>
> 监听待写client的socketFD可写事件，然后把写回处理器（回调函数）绑定到socketFD上，
>
> ![image-20241116162517768](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241116162517768.png)

##### 事件类型

简单来说，在正式监听之前首先要**注册**不同的事件，将其绑定到特定socket上，事件真正发生以后会触发处理器回调函数

| 事件名称 |               监听socket 可读               |          已连接socket 可读           |    已连接socket 可写     |
| :------: | :-----------------------------------------: | :----------------------------------: | :----------------------: |
| 绑定时机 | 创建监听socket时(createSocketAcceptHandler) | 创建已连接socket时(acceptTcpHandler) | 监听开始前(beforeSleep)  |
| 绑定对象 |                 监听socket                  |             已连接socket             | 待写队列中的已连接socket |
|  处理器  |       acceptTcpHandler (创建TCP连接)        |  readQueryFromClient (读取处理命令)  | sendReplyToClient (写回) |

##### Redis 单线程网络模型

IO Multiplexing + Event Distributing = `eventLoop` -> `beforeSleep` -> `aeApiPoll`(`epoll_wait`)

![1653982278727](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653982278727.png)

当我们的客户端想要去连接我们服务器，会去先到IO多路复用模型去进行排队，会有一个连接应答处理器，他会去接受读请求，然后又把读请求注册到具体模型中去，此时这些建立起来的连接，如果是客户端请求处理器去进行执行命令时，他会去把数据读取出来，然后把数据放入到client中， client去解析当前的命令转化为redis认识的命令，接下来就开始处理这些命令，从redis中的command中找到这些命令，然后就真正的去操作对应的数据了，当数据操作完成后，会去找到命令回复处理器，再由他将数据写出。

简

#### 多线程网络模型——解决网络IO瓶颈

![image-20241116173420995](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241116173420995.png)

原来的性能瓶颈出现在网络IO，从IO流中读取信息比较耗时，因此将读取命令和写回结果的任务派发给子线程

但是真正执行命令的过程并不是瓶颈（基于内存已经很快了）所以命令执行依然是单线程 

[高性能网络编程之 Reactor 网络模型（彻底搞懂）_reactor网络模型-CSDN博客](https://blog.csdn.net/ldw201510803006/article/details/124365838#:~:text=我们先来看看，一个网络请求在服务端经历了哪些阶段：) 

[Redis中的Reactor模型介绍Reactor模型，并对比Redis 6.0之前的单线程模型 与 Redis 6.0 - 掘金 (juejin.cn)](https://juejin.cn/post/7124667316637270046#heading-11) 

## Redis通信协议

### RESP

Redis是一个C/S架构的软件，通信一般分两步（不包括pipeline和PubSub）：

1. 客户端（Client）向服务端（Server）发送一条命令
2. 服务端解析并执行命令，返回响应结果给客户端

因此客户端发送命令的格式、服务端响应结果的格式必须有一个规范，这个规范就是通信协议。(B/S架构为HTTP协议)

[FTP、TFTP、HTTP、SMTP、DHCP、Telnet、DNS、SNMP(网络协议：应用层协议）-CSDN博客](https://blog.csdn.net/freee12/article/details/114411950)

Redis采用RESP（Redis Serialization Protocol）协议：

- Redis 1.2版本引入了RESP协议

- Redis 2.0版本中成为与Redis服务端通信的标准，称为RESP2

- Redis 6.0版本中，从RESP2升级到了RESP3协议，增加了更多数据类型并且支持6.0的新特性--客户端缓存


但目前，默认使用的依然是RESP2协议，也是我们要学习的协议版本（以下简称RESP）。



在RESP中，通过首字节的字符来区分不同数据类型，常用的数据类型包括5种：

单行字符串：首字节是 ‘+’ ，后面跟上单行字符串，以CRLF（ "\r\n" ）结尾。例如返回"OK"： "+OK\r\n"

错误（Errors）：首字节是 ‘-’ ，与单行字符串格式一样，只是字符串是异常信息，例如："-Error message\r\n"

数值：首字节是 ‘:’ ，后面跟上数字格式的字符串，以CRLF结尾。例如：":10\r\n"

多行字符串：首字节是 ‘$’ ，表示二进制安全的字符串，最大支持512MB：

如果大小为0，则代表空字符串："$0\r\n\r\n"

如果大小为-1，则代表不存在："$-1\r\n"

数组：首字节是 ‘*’，后面跟上数组元素个数，再跟上元素，元素数据类型不限:

![1653982993020](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653982993020.png)

### 基于Socket实现客户端

Redis支持TCP通信，因此我们可以使用Socket来模拟客户端，与Redis服务端建立连接：

```java
public class Main {

    static Socket s;
    static PrintWriter writer;
    static BufferedReader reader;

    public static void main(String[] args) {
        try {
            // 1.建立连接
            String host = "192.168.150.101";
            int port = 6379;
            s = new Socket(host, port);
            // 2.获取输出流、输入流
            writer = new PrintWriter(new OutputStreamWriter(s.getOutputStream(), StandardCharsets.UTF_8));
            reader = new BufferedReader(new InputStreamReader(s.getInputStream(), StandardCharsets.UTF_8));

            // 3.发出请求
            // 3.1.获取授权 auth 123321
            sendRequest("auth", "123321");
            Object obj = handleResponse();
            System.out.println("obj = " + obj);

            // 3.2.set name 虎哥
            sendRequest("set", "name", "虎哥");
            // 4.解析响应
            obj = handleResponse();
            System.out.println("obj = " + obj);

            // 3.2.set name 虎哥
            sendRequest("get", "name");
            // 4.解析响应
            obj = handleResponse();
            System.out.println("obj = " + obj);

            // 3.2.set name 虎哥
            sendRequest("mget", "name", "num", "msg");
            // 4.解析响应
            obj = handleResponse();
            System.out.println("obj = " + obj);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 5.释放连接
            try {
                if (reader != null) reader.close();
                if (writer != null) writer.close();
                if (s != null) s.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private static Object handleResponse() throws IOException {
        // 读取首字节
        int prefix = reader.read();
        // 判断数据类型标示
        switch (prefix) {
            case '+': // 单行字符串，直接读一行
                return reader.readLine();
            case '-': // 异常，也读一行
                throw new RuntimeException(reader.readLine());
            case ':': // 数字
                return Long.parseLong(reader.readLine());
            case '$': // 多行字符串
                // 先读长度
                int len = Integer.parseInt(reader.readLine());
                if (len == -1) {
                    return null;
                }
                if (len == 0) {
                    return "";
                }
                // 再读数据,读len个字节。我们假设没有特殊字符，所以读一行（简化）
                return reader.readLine();
            case '*':
                return readBulkString();
            default:
                throw new RuntimeException("错误的数据格式！");
        }
    }

    private static Object readBulkString() throws IOException {
        // 获取数组大小
        int len = Integer.parseInt(reader.readLine());
        if (len <= 0) {
            return null;
        }
        // 定义集合，接收多个元素
        List<Object> list = new ArrayList<>(len);
        // 遍历，依次读取每个元素
        for (int i = 0; i < len; i++) {
            list.add(handleResponse());
        }
        return list;
    }

    // set name 虎哥
    private static void sendRequest(String ... args) {
        writer.println("*" + args.length);
        for (String arg : args) {
            writer.println("$" + arg.getBytes(StandardCharsets.UTF_8).length);
            writer.println(arg);
        }
        writer.flush();
    }
}

```

## Redis内存回收

### 过期key处理

Redis之所以性能强，最主要的原因就是基于内存存储。然而单节点的Redis其内存大小不宜过大，会影响持久化或主从同步性能。
我们可以通过修改配置文件来设置Redis的最大内存：

![1653983341150](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653983341150.png)

当内存使用达到上限时，就无法存储更多数据了。为了解决这个问题，Redis提供了一些策略实现内存回收：

内存过期策略

在学习Redis缓存的时候我们说过，可以通过expire命令给Redis的key设置TTL（存活时间）：

![1653983366243](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653983366243.png)

可以发现，当key的TTL到期以后，再次访问name返回的是nil，说明这个key已经不存在了，对应的内存也得到释放。从而起到内存回收的目的。

#### RedisDB 结构

Redis本身是一个典型的key-value内存存储数据库，因此所有的key、value都保存在之前学习过的Dict结构中。不过在其db结构体中，有两个Dict：一个用来记录key-value；另一个用来记录key-TTL。

![redisDb 结构体](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653983423128.png)



![1653983606531](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653983606531.png)

这里有两个问题需要我们思考：
Redis是如何知道一个key是否过期呢？

利用两个Dict分别记录key-value对及key-ttl对

是不是TTL到期就立即删除了呢？

#### **惰性删除**

惰性删除：顾明思议并不是在TTL到期后就立刻删除，而是在访问一个key的时候，检查该key的存活时间，如果已经过期才执行删除。

![1653983652865](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653983652865.png)

#### **周期删除**

周期删除：通过一个定时任务，周期性的抽样部分过期的key，然后执行删除。执行周期有两种：

- Redis服务初始化函数initServer()中设置定时任务，按照server.hz的频率来执行过期key清理，模式为SLOW
- Redis的每个事件循环前会调用beforeSleep()函数，执行过期key清理，模式为FAST

SLOW模式规则：低频 高时长

* 执行频率受server.hz影响，默认为10，即每秒执行10次，每个执行周期100ms。
* 执行清理耗时不超过一次执行周期的25%.默认slow模式耗时不超过25ms
* 逐个遍历db，逐个遍历db中的bucket，抽取20个key判断是否过期
* 如果没达到时间上限（25ms）并且过期key比例大于10%，再进行一次抽样，否则结束

FAST模式规则（过期key比例小于10%不执行 ）	高频 低时长

* 执行频率受beforeSleep()调用频率影响，但两次FAST模式间隔不低于2ms
* 执行清理耗时不超过1ms
* 逐个遍历db，逐个遍历db中的bucket，抽取20个key判断是否过期
  如果没达到时间上限（1ms）并且过期key比例大于10%，再进行一次抽样，否则结束

#### 总结

RedisKey的TTL记录方式：

- 在RedisDB中通过一个Dict记录每个Key的TTL时间


过期key的删除策略：

- 惰性清理：每次查找key时判断是否过期，如果过期则删除

- 定期清理：定期抽样部分key，判断是否过期，如果过期则删除。

定期清理的两种模式：

- SLOW模式执行频率默认为10，每次不超过25ms，低频高时长

- FAST模式执行频率不固定，但两次间隔不低于2ms，每次耗时不超过1ms，高频低时长


### 内存淘汰策略

内存淘汰：就是当Redis内存使用达到设置的上限时，主动挑选部分key删除以释放更多内存的流程。Redis会在处理客户端命令的方法processCommand()中尝试做内存淘汰：（时机：调用真正的 回调函数之前）

![1653983978671](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653983978671.png)

#### 淘汰策略 

Redis支持8种不同策略来选择要删除的key：

* noeviction： 不淘汰任何key，但是内存满时不允许写入新数据，默认就是这种策略。
* volatile-ttl： 对设置了TTL的key，比较key的剩余TTL值，TTL越小越先被淘汰

* allkeys-random：对全体key ，随机进行淘汰。也就是直接从db->dict中随机挑选。
* volatile-random：对设置了TTL的key ，随机进行淘汰。也就是从db->expires中随机挑选。
* allkeys-lru： 对全体key，基于LRU算法进行淘汰
* volatile-lru： 对设置了TTL的key，基于LRU算法进行淘汰
* allkeys-lfu： 对全体key，基于LFU算法进行淘汰
* volatile-lfu： 对设置了TTL的key，基于LFI算法进行淘汰

#### LRU LFU

比较容易混淆的有两个：

* LRU（Least Recently Used），最少最近使用。最后一次访问时间越小则淘汰优先级越高。
* LFU（Least Frequently Used），最少频率使用。会统计每个key的访问频率，值越小淘汰优先级越高。

Redis的数据都会被封装为RedisObject结构：

![1653984029506](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984029506.png)

LFU的访问次数之所以叫做逻辑访问次数，是因为并不是每次key被访问都计数，而是通过运算：

* 生成0~1之间的随机数R
* 计算 (旧次数 * lfu_log_factor + 1)，记录为P
* 如果 R < P ，则计数器 + 1，且最大不超过255
* 访问次数会随时间衰减，距离上一次访问时间每隔 lfu_decay_time 分钟，计数器 -1

- Redis 在键的元数据中为每个键维护一个 `LFU 信息`，它是一个 8 位的字段。
- 这个字段的两个部分：
  - **前 6 位**（称为 `log-based counter`）：表示键的访问频率。
  - **后 2 位**：存储时间相关信息，用于对计数的衰减。

Redis 的访问计数器设计为 **对数型增长**，原因是避免计数器线性增长带来的溢出问题，同时降低热点数据被频繁访问的影响。

Redis 定期对计数值进行衰减，确保长期未访问的键逐渐失去其高计数值。

- 衰减机制使用 Redis 的 `LFU_DECAY_TIME` 参数控制，默认值为 1 分钟。
- 每次访问键时，Redis 检查上次更新计数的时间。如果超过 `LFU_DECAY_TIME`，就会减少计数值。

![image-20250518192519001](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250518192519001.png)

**计算淘汰优先级**

在淘汰键时，Redis 根据 LFU 计数选择淘汰候选：

- 计数值越低，淘汰优先级越高。
- 如果计数值相同，Redis 会根据其他元数据（如键的创建时间）来辅助决定。

Redis 提供了以下两种与 LFU 相关的内存淘汰策略：

1. `volatile-lfu`：从设置了过期时间的键中使用 LFU 策略淘汰。
2. `allkeys-lfu`：从所有键中使用 LFU 策略淘汰。

可以在 `redis.conf` 文件中设置：

```bash
maxmemory-policy allkeys-lfu
```

Redis 的 LFU 实现基于紧凑的计数器设计，并通过概率增长与时间衰减结合，达到高效的淘汰效果。它是 Redis 用于管理内存淘汰的重要策略之一，适合高访问频率场景下的优化。

#### 总结

![1653984085095](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653984085095.png)

淘汰池子，加入抽样调查key，

淘汰规则：按照idleTime升序排序，值大的优先淘汰

LRU LFU TTL 都是值越小越应该淘汰，因此idleTime优先淘汰值大的
