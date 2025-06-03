---
name: socket-programming
title: Socket 编程
date: 2025-05-29
tags:
- select/poll/epoll
- socket
- io
categories: 计网

---

# Socket Programming

## 前置知识

### 用户空间和内核态空间

服务器大多都采用Linux系统，这里我们以Linux为例来讲解:

ubuntu和Centos 都是Linux的发行版，发行版可以看成对linux包了一层壳，任何Linux发行版，其系统内核都是Linux。我们的应用都需要通过Linux内核与硬件交互

![1653844970346](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653844970346.png)

用户的应用，比如redis，mysql等其实是没有办法去执行访问我们操作系统的硬件的，所以我们可以通过发行版的这个壳子去访问内核，再通过内核去访问计算机硬件

![1653845147190](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653845147190.png)

计算机硬件包括，如cpu，内存，网卡等等，内核（通过寻址空间）可以操作硬件的，但是内核需要不同设备的驱动，有了这些驱动之后，内核就可以去对计算机硬件去进行 内存管理，文件系统的管理，进程的管理等等

![1653896065386](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653896065386.png)



我们想要用户的应用来访问，计算机就必须要通过对外暴露的一些接口，才能访问到，从而简介的实现对内核的操控，但是内核本身上来说也是一个应用，所以他本身也需要一些内存，cpu等设备资源，用户应用本身也在消耗这些资源，如果不加任何限制，用户去操作随意的去操作我们的资源，就有可能导致一些冲突，甚至有可能导致我们的系统出现无法运行的问题，因此我们需要把用户和**内核隔离开**

进程的寻址空间划分成两部分：**内核空间、用户空间**

什么是寻址空间呢？我们的应用程序也好，还是内核空间也好，都是没有办法直接去物理内存的，而是通过分配一些虚拟内存映射到物理内存中，我们的内核和应用程序去访问虚拟内存的时候，就需要一个虚拟地址，这个地址是一个无符号的整数，比如一个32位的操作系统，他的带宽就是32，他的虚拟地址就是2的32次方，也就是说他寻址的范围就是0~2的32次方， 这片寻址空间对应的就是2的32个字节，就是4GB，这个4GB，会有3个GB分给用户空间，会有1GB给内核系统

![1653896377259](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653896377259.png)

在linux中，他们权限分成两个等级，0和3，用户空间只能执行受限的命令（Ring3），而且不能直接调用系统资源，必须通过内核提供的接口来访问内核空间可以执行特权命令（Ring0），调用一切系统资源，所以一般情况下，用户的操作是运行在用户空间，而内核运行的数据是在内核空间的，而有的情况下，一个应用程序需要去调用一些特权资源，去调用一些内核空间的操作，所以此时他俩需要在用户态和内核态之间进行切换。

比如：

Linux系统为了提高IO效率，会在用户空间和内核空间都加入缓冲区：

写数据时，要把用户缓冲数据拷贝到内核缓冲区，然后写入设备

读数据时，要从设备读取数据到内核缓冲区，然后拷贝到用户缓冲区

针对这个操作：我们的用户在写读数据时，会去向内核态申请，想要读取内核的数据，而内核数据要去等待驱动程序从硬件上读取数据，当从磁盘上加载到数据之后，内核会将数据写入到内核的缓冲区中，然后再将数据拷贝到用户态的buffer中，然后再返回给应用程序，整体而言，速度慢，就是这个原因，为了加速，我们希望read也好，还是wait for data也最好都不要等待，或者时间尽量的短。

![1653896687354](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653896687354.png)

### TCP Socket

[网络编程：Socket 是如何创建的？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/464268288)

#### 定义

Socket：应用程序通过socket提供的接口将网络传输的工作交给linux内核，内核通过驱动程序操作网卡，接受网卡发来的信息。

`socket` 是一个 **编程接口**（API），它本身并不属于网络协议栈的某一层，但它主要用于操作 **传输层**（如 TCP 和 UDP）以及网络层（如原始套接字）的通信，它本身不是传输层或应用层的一部分，而是一个编程抽象，用于简化应用程序访问网络的过程。

- **Socket API** 是操作系统提供的接口，用于应用程序与网络协议栈交互。
- 它允许开发者使用 **传输层协议（TCP/UDP）** 或更底层的协议（如 IP）进行网络通信。
- 因此，虽然 Socket 本身不属于传输层或应用层，但它主要作用于 **传输层协议**，并为应用层提供访问网络的工具。

**按照使用的协议可以分为**：

- **传输层：**
  - 使用 `TCP` 协议时，`TCP Socket` 提供可靠的面向连接的数据传输。
  - 使用 `UDP` 协议时，`UDP Socket` 提供无连接、不可靠的数据传输。

- **网络层：**
  - 使用原始套接字（Raw Socket）时，可以直接处理 IP 包，适用于网络工具如 ping 和 traceroute。

- 应用程序通过 Socket 来发送和接收数据，但具体的数据传输和可靠性保障是由 **传输层协议（如 TCP/UDP）** 实现的

#### 类型

- **主动 Socket** socket()默认创建的socket类型，客户端向服务端建立连接需要的socket
- **监听 Socket** listen()将主动socket转换成监听socket，用于监听客户端的连接请求。是服务端独有的，将伴随服务端的整个生命周期。
- **已连接 Socket** 通过系统库函数accept()获取的已建立连接的socket，该socket是用于客户端和服务端数据读写的通道，已连接socket是服务器独有的，生命周期为 客户端和服务端的维持的连接时长，当断开连接，生命周期结束。


一个服务器通常通常仅仅只创建一个监听socket描述字，它在该服务器的生命周期内一直存在。内核为每个由服务器进程接受的客户连接创建了一个已连接socket描述字，当服务器完成了对某个客户的服务，相应的已连接socket描述字就被关闭。

#### socket 的基本使用

socket, bind, connect, listen, accept....



> - `socket()`:创建socket，规定各项参数，不能随意组合，socket()创建的socket默认是一个主动类型的，参数如下：
>
>   ![image-20241115195114618](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241115195114618.png)
>
> - `bind`: 将IP和端口号绑定到socket上，如果不绑定，客户端会通过connect随机分配，服务端会通过listen随机分配，服务端需要的是一个明确的地址信息，所以必须提前绑定好端口。
>
> <img src="https://cdn.xiaolincoding.com//mysql/other/35bc3541c237686aa36e0a88f80592d4.png" alt="图片" style="zoom: 80%;" />
>
> - `connect()`: 客户端通过connect向服务端发起连接请求。从调用到返回：从`SYN_SENT`到`ESTABLISHED`，之后客户端会发送ACK报文，提示
>
> - `listen()`: 将socket变为被动类型（监听Socket）监听客户的连接请求。
>
> - `accept()`: 服务端监听到连接请求之后，在3次成功握手之后成功建立连接。1个**监听Socket**维护2个连接队列（全连接established和半连接syn_rcvd），accept会从全连接队列中拿出一个已连接的Socket进行处理，如果还没有完成，就要阻塞等待直到全链接队列有可用的socket。拿到已连接Socket之后就可以开始网络I/O，类同普通文件的读写I/O。当连接可用时，创建的套接字就可以从请求连接的进程中读取数据。
>   - accept() 调用创建一个与监听socket具有相同属性的新socketFD，并将其返回给调用者caller。如果队列没有挂起的连接请求，accept() 会阻塞调用者，除非socket处于非阻塞模式（Non-Blocking）。如果没有连接请求排队并且套接字处于非阻塞模式，则accept()返回-1并将错误代码设置为<mark>EWOULDBLOCK<mark>。新的socketFD不能用于接受新连接。原来的监听socket仍然可以接受更多的连接请求。
>   - accept的第一个参数为服务器的socketFD，是服务器开始调用socket()函数生成的，称为监听socketFD；
>     而accept函数返回的是已连接的socketFD。两个套接字不一样。
>
> ![image-20241115202937962](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241115202937962.png)
>
> 连接双方的IP地址和端口构成一个四元组，唯一标记一个客户端，将其作为Key，存到哈希表里，值就是sockfd，下次连接时重新从哈希表里取出来即可。		
>
> - `send/recv read/write`: 读写操作会先写到缓冲区中
>
> **怎么观察 socket 缓冲区** 
>
> 如果想要查看 socket 缓冲区，可以在linux环境下执行 `netstat -nt` 命令。
>
> ```go
> # netstat -nt
> Active Internet connections (w/o servers)
> Proto Recv-Q Send-Q Local Address           Foreign Address         State      
> tcp        0     60 172.22.66.69:22         122.14.220.252:59889    ESTABLISHED
> ```
>
> 这上面表明了，这里有一个协议（Proto）类型为 TCP 的连接，同时还有本地（Local Address）和远端（Foreign Address）的IP信息，状态（State）是已连接（Established） 还有**Send-Q 是发送缓冲区**，下面的数字60是指，当前还有60 Byte在发送缓冲区中未发送。而 **Recv-Q 代表接收缓冲区**，此时是空的，数据都被应用进程接收干净了。
>
> [动画图解 socket 缓冲区的那些事儿-CSDN博客](https://blog.csdn.net/qcrao/article/details/120278587) 
>
> ![image-20241116154258525](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241116154258525.png)

#### 与文件系统、内核的调用关系

![image-20241115193558676](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241115193558676.png)

![image-20241115193416185](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241115193416185.png)

#### Socket 就绪之前

从ESTABLISHED tcp connection 到 就绪 之前：

TCP是面向连接的协议，通过三次握手建立连接后，会进行一些必要的初始化：

**（1）TCP状态维护**

- **接收和发送缓冲区准备**：双方为这个连接分配缓冲区，准备接收或发送数据。
- **连接状态记录**：内核中的Socket结构会记录新连接的相关信息，如对端的IP地址、端口、协议状态等。

**（2）SSL/TLS握手（如适用）**

如果通信使用了加密（如HTTPS），在TCP连接建立后会进行SSL/TLS握手。此过程包括：

- 协商协议版本和加密算法。
- 交换密钥。
- 验证身份。

**（3）应用层协议的初始化**

在TCP连接建立后，通常需要按照应用层协议（如HTTP、FTP、WebSocket等）定义的逻辑进行数据交互的初始化。例如：

- HTTP/1.1会发送`GET`或`POST`请求。
- WebSocket会升级协议，通过`Upgrade`头进行握手。

**（4）延迟与阻塞等待**

如果一端发送了数据而另一端未及时处理，连接可能处于阻塞或等待状态。例如：

- **服务端等待客户端请求**。  
- **客户端等待服务端响应**。  

## 五种IO模型

### 阻塞IO (BIO)

在《UNIX网络编程》一书中，总结归纳了5种IO模型：

* 阻塞IO（Blocking IO）
* 非阻塞IO（Nonblocking IO）
* IO多路复用（IO Multiplexing）
* 信号驱动IO（Signal Driven IO）
* 异步IO（Asynchronous IO）

应用程序想要去读取数据，他是无法直接去读取磁盘/网卡数据的，他需要先到内核里边去等待内核操作硬件拿到数据，这个过程就是1，是需要等待的，等到内核从磁盘上把数据加载出来之后，再把这个数据写给用户的缓存区，这个过程是2，如果是阻塞IO，那么整个过程中，用户从发起读请求开始，一直到读取到数据，都是一个阻塞状态。

![1653897115346](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653897115346.png)

具体流程如下图：

用户去读取数据时，会去先发起recvform一个命令，去尝试从内核上加载数据，如果内核没有数据，那么用户就会等待，此时内核会去从硬件上读取数据，内核读取数据之后，会把数据拷贝到用户态，并且返回ok，整个过程，都是阻塞等待的，这就是阻塞IO

总结如下：

顾名思义，阻塞IO就是两个阶段都必须阻塞等待：

阶段一：receivefrom（阻塞）

- 用户进程尝试读取数据（比如网卡数据）
- 此时数据尚未到达，内核需要等待数据
- 此时用户进程也处于阻塞状态

阶段二：copy（阻塞）

* 数据到达并拷贝到内核缓冲区，代表已就绪
* 将内核数据拷贝到用户缓冲区
* 拷贝过程中，用户进程依然阻塞等待
* 拷贝完成，用户进程解除阻塞，处理数据

可以看到，阻塞IO模型中，用户进程在两个阶段都是阻塞状态。



![1653897270074](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653897270074.png)

### 非阻塞IO (NIO)

顾名思义，非阻塞IO的recvfrom操作会立即返回结果而不是阻塞用户进程。

阶段一：recvfrom（非阻塞）

* 用户进程尝试读取数据（比如网卡数据）
* 此时数据尚未到达，内核需要等待数据
* 返回异常给用户进程
* 用户进程拿到error后，再次尝试读取
* 循环往复，直到数据就绪

阶段二：copy（阻塞）

* 将内核数据拷贝到用户缓冲区
* 拷贝过程中，用户进程依然阻塞等待
* 拷贝完成，用户进程解除阻塞，处理数据
* 可以看到，非阻塞IO模型中，用户进程在第一个阶段是非阻塞，第二个阶段是阻塞状态。虽然是非阻塞，但性能并没有得到提高。而且忙等机制会导致CPU空转，CPU使用率暴增。

用户应用进行IO操作，调用监听socket的accept()获取已连接socket进行IO操作，如果accept获取不到已连接的socket则直接返回-1(EWOULDBLOCK)，获取成功则返回已连接的socket的socketFD

![1653897490116](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653897490116.png)

### IO多路复用 (IO Multiplexing)

无论是阻塞IO还是非阻塞IO，用户应用在一阶段都需要调用recvfrom来获取数据，差别在于无数据时的处理方案：

如果调用recvfrom时，恰好没有数据，阻塞IO会使CPU阻塞，非阻塞IO使CPU空转，都不能充分发挥CPU的作用。
如果调用recvfrom时，恰好有数据，则用户进程可以直接进入第二阶段，读取并处理数据

所以怎么看起来以上两种方式性能都不好

而在单线程情况下，只能依次处理IO事件，如果正在处理的IO事件恰好未就绪（数据不可读或不可写），线程就会被阻塞，所有IO事件都必须等待，性能自然会很差。

就比如服务员给顾客点餐，**分两步**：

* 顾客思考要吃什么（服务员这边等待数据就绪）
* 顾客想好了，开始点餐（服务员开始真正读取数据）

要提高效率有几种办法？

方案一：增加更多服务员（多线程）：上下文切换消耗资源 PASS
方案二：不排队，谁想好了吃什么（数据就绪了），服务员就给谁点餐（用户应用就去读取数据）OKAY

那么问题来了：用户进程如何知道内核中数据是否就绪呢？

所以接下来就需要详细的来解决多路复用模型是如何知道到底怎么知道内核数据是否就绪的问题了

这个问题的解决依赖于提出的：Socket FD

文件描述符（File Descriptor）：简称FD，是一个从0 开始的无符号整数，用来关联Linux中的一个文件。在Linux中，一切皆文件，例如常规文件、视频、硬件设备等，当然也包括网络套接字（Socket）。

> 一个进程对应一个`task_struct`，一个`task_struct`中有一个`file_struct`，一个`file_struct`中有一个`fdt`(file descriptor table文件描述符表) `fdt`中就有一个`fd_array`（fd数组）数组索引为fd，内容就是指向`file`的指针

通过FD，我们的网络模型可以利用一个线程监听多个FD，并在某个FD可读、可写时得到通知，从而避免无效的等待，充分利用CPU资源。

阶段一：

* 用户进程调用select，指定要监听的FD集合
* 核监听FD对应的多个socket
* 任意一个或多个socket数据就绪则返回readable（可读）
* 此过程中用户进程阻塞

阶段二：

* 用户进程找到就绪的socket
* 依次调用recvfrom读取数据
* 内核将数据拷贝到用户空间
* 用户进程处理数据。

当用户去读取数据的时候，不再去直接调用recvfrom了，而是调用select的函数，select函数会将需要监听的数据交给内核，由内核去检查这些数据是否就绪了，如果说这个数据就绪了，就会通知应用程序数据就绪，然后来读取数据，再从内核中把数据拷贝给用户态，完成数据处理，如果N多个FD一个都没处理完，此时就进行等待。

BIO模式只能查看一个socket，SocketA准备就绪了，但是用户进程在阻塞等待SocketB的数据，这就是无效的等待。

IO Multiplexing可以减少空等空转空轮询，提高性能

![1653898691736](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653898691736.png)

数据就绪返回readable，用户进程调用recvfrom处理数据。

IO多路复用是利用单个线程来同时监听多个FD，并在某个FD可读、可写时得到通知，从而避免无效的等待，充分利用CPU资源。不过监听FD的方式、通知的方式又有多种实现，常见的有：select，poll，epoll。其中select和pool相当于是当被监听的数据准备好之后，他会把你监听的FD数组整个数据都发给你，你需要到整个FD中去找，哪些是处理好了的，需要通过遍历的方式，所以性能也并不是那么好。而epoll，则相当于内核准备好了之后，他会把准备好的数据，直接发给你，省去了遍历的动作。

#### `select()`

[select/poll](https://xiaolincoding.com/os/8_network_system/selete_poll_epoll.html#select-poll) 

##### 流程

select是Linux最早是由的I/O多路复用技术：

简单说，就是我们把需要处理的数据封装成FD，然后在用户态时创建一个fd的集合（这个集合的大小是要监听的那个FD的最大值+1，但是大小整体是有限制的 ），这个集合的长度大小是有限制的，同时在这个集合中，标明出来我们要控制哪些数据，比如要监听的数据，是1,2,5三个数据，此时会执行select函数，**遍历数组把需要监听的FD置1**（直到nfds fd上限），然后<mark>将整个FDSet拷贝到内核态<mark>，内核态会去**遍历用户态传递过来的数据**，如果发现这里边都数据都没有就绪，就休眠。

直到有数据准备好时，就会被唤醒，唤醒之后，**再次遍历一遍**，看看谁准备好了，将没有准备好的数据置0，<mark>最后再次将这个FDSet拷贝回用户态<mark>，此时用户态就知道有人准备好了（readable），但对于用户态而言，并不知道谁处理好了，所以用户态**也需要去进行遍历**，然后找到被置1（准备就绪）的节点，再去发起receivefrom读写请求，我们会发现，这种模式下他虽然比阻塞IO和非阻塞IO好，但是依然有些麻烦的事情， 比如说频繁的传递fd集合，频繁的去遍历FD等问题

bitmap： __fd_mask = 32bits  FDSet共1024个bit位。

把fd的状态映射到单个bit位上面，很大程度上节省了内存空间，但是也导致用户态不知道谁准备好了

![1653900022580](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653900022580.png)

![image-20241114211029023](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241114211029023.png)

> select 返回值：
>
> - **大于0**：表示有文件描述符准备好了。返回值是就绪的文件描述符的数量，也就是有多少个文件描述符在监视的时间内发生了事件（例如：可以读、可以写、异常等）。
> - **0**：表示在指定的时间内没有文件描述符准备好，即超时。
> - -1：表示发生了错误，errno 中会设置为相应的错误码。常见的错误包括：
>   - `EBADF`：传递给 `select` 的某些文件描述符无效。
>   - `EINTR`：调用被信号中断。
>   - `EINVAL`：某个参数无效（例如 `nfds` 负值）。

##### 缺点

- 整个流程涉及到两次用户态与内核态之间的拷贝，频繁切换，一共需要2次「拷贝」fdSet
- 返回值只是就绪的节点个数，然而select并不知道是具体哪个节点就绪，还需要重新遍历，一共需要2次「遍历」
- fd_Set 最多监听1024个

#### `poll()`

poll模式对select模式做了简单改进，但性能提升不明显，部分关键代码如下：

IO流程：

* 创建pollfd数组，向其中添加关注的fd信息，数组大小自定义
* 调用poll函数，将pollfd数组拷贝到内核空间，转链表存储，无上限
* 内核遍历fd，判断是否就绪
* 数据就绪或超时后，拷贝pollfd数组到用户空间，返回就绪fd数量n
* 用户进程判断n是否大于0,大于0则遍历pollfd数组，找到就绪的fd

**与select对比：**

* select模式中的fd_set大小固定为1024，而pollfd在内核中采用链表，理论上无上限
* 监听FD越多，每次遍历消耗时间也越久，性能反而会下降

![1653900721427](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653900721427.png)

#### `epoll()`：event poll

[epoll](https://xiaolincoding.com/os/8_network_system/selete_poll_epoll.html#epoll) <mark>核心：事件轮询</mark>

##### epoll 的优势

select模式存在的三个问题：

* 能监听的FD<mark>最大不超过</mark>1024
* <mark>每次</mark>select都需要把<mark>所有</mark>要监听的FD都拷贝到内核空间
* 每次都要<mark>遍历所有</mark>FD来判断就绪状态



poll模式的问题：

* poll利用链表解决了select中监听FD上限的问题，但依然要遍历所有FD，如果监听较多，性能会下降



epoll 是解决 C10K 问题的利器，通过两个方面解决了 select/poll 的问题：

- **减少无效拷贝**：epoll 在内核里使用「红黑树」来关注进程所有待检测的 Socket，理论上无上限。红黑树是个高效的数据结构，增删改一般时间复杂度是 O(logn)，通过对这棵黑红树的管理，不需要像 select/poll 在每次操作时都**传入整个 Socket 集合**，减少了内核和用户空间大量的**数据拷贝**和内存分配。
- **减少无效遍历**：epoll 使用事件驱动（ep_poll_callback）的机制，内核里维护了一个「链表」来记录就绪事件，只将有事件发生的 fd 传递给应用程序，不需要像 select/poll 那样轮询扫描整个集合（包含有和无事件的 fd），大大提高了检测的效率。

##### 流程

创建eventpoll结构，主要包含以下三个结构：

0、`wait_queue_head_t wq` 用于 `epoll_wait` 阻塞等待事件发生。

1、`rb_root rb_root` 用于快速查找文件描述符，存储所有注册的 `epitem`。

2、`list_head rdllist`  存储已触发的事件，`epoll_wait` 通过它返回事件。

> `epitem`：表示 `epoll` 监视的 **单个文件描述符**，它既是 **红黑树的节点**，也是 **双向链表的节点**。
>
> ```c
> struct epitem {
>     struct rb_node rbn;         // 红黑树节点
>     struct list_head rdllink;   // 双向链表节点
>     struct epoll_event event;   // 存储用户注册的事件
>     int fd;                     // 关联的文件描述符
> };
> ```
>
> `epoll_event`：在 `epoll` 中，事件（event）是通过 `epoll_event` 结构体来定义和处理的。`epoll` 的事件分类主要基于 I/O 操作的类型（例如：可读、可写等），它们可以通过 `epoll_ctl` 函数注册，之后通过 `epoll_wait` 等函数来等待这些事件的发生。
>
> ```c
> struct epoll_event {
>     uint32_t events;  /* 事件类型 */
>     epoll_data_t data; /* 用户数据 */
> };
> typedef union epoll_data {
>     void *ptr;    /* 指针 */
>     int fd;       /* 文件描述符 */
>     uint32_t u32; /* 32 位整数 */
>     uint64_t u64; /* 64 位整数 */
> } epoll_data_t;
> // 使用例------
> 
> struct epoll_event ev;
> ev.events = EPOLLIN | EPOLLET;
> ev.data.fd = sockfd; // 关联文件描述符
> epoll_ctl(epfd, EPOLL_CTL_ADD, sockfd, &ev);
> ```

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241114221957536.png" alt="image-20241114221957536" style="zoom: 80%;" />

3、`epoll_ctl(epfd, EPOLL_CTL_ADD, sockfd, &ev)`：将要监听的 fd 以及要监听的事件类型封装成 `epitem`，作为节点添加到红黑树，设置 `ep_poll_callback`回调函数，这个函数会在监听到fd发生对应的事件时触发，并把 fd 对应的 `epitem`  添加到就绪链表中。

> 回调函数：将函数作为参数传入，Java中是函数式接口的实现类，C中是函数指针
>
> 事件驱动：以状态的转换作为事件发生的标志，事件发生会触发回调函数的执行。而事件是多种多样的，这就要求执行的函数不能写死，需要实现充分的解耦
>
> [C 语言回调函数详解 | 菜鸟教程 (runoob.com)](https://www.runoob.com/w3cnote/c-callback-function.html) 

4、`epoll_wait(epfd, events, 10, 5000)`：最多监听 10 个事件，超时时间 5 秒。

```c
struct epoll_event events[10]; // 存储触发的事件
int nfds = epoll_wait(epfd, events, 10, 5000); // 最多监听 10 个事件，超时时间 5 秒

if (nfds < 0) {
    perror("epoll_wait failed");
    exit(EXIT_FAILURE);
}

for (int i = 0; i < nfds; i++) {
    if (events[i].events & EPOLLIN) {
        printf("文件描述符 %d 可读\n", events[i].data.fd);
    }
}

```

在用户态需要创建一个空的`events`数组传给 `epoll_wait`，触发事件时回调函数会把 fd 对应的 `epitem` 添加到`list_head`中去。调用`epoll_wait`时检查`list_head`是否为空，当然这个过程需要参考配置的等待时间，可以等一定时间，也可以一直等。

根据内核的 poll 机制，epoll 需要为每个监听的 fd 构造一个 epoll entry(设置关心的事件以及回调函数)作为等待队列成员睡眠在每个 fd 的等待队列，以便 fd 上的事件就绪时可以通过回调函数通知。无事件的时候，多个进程 task 调用`epoll_wait`睡眠在 epoll 的 wq 睡眠队列上。

1. 这个时候一个请求RQ_1上来，listen fd 这个时候就绪，开始唤醒其对应的睡眠队列上的 epoll entry，并执行之前 epoll 注册的回调函数 `ep_poll_callback`。
2. `ep_poll_callback`主要做两件事情：(1) 发生的事件是 epoll entry 关心的，则将`epitem`挂载到epoll的就绪链表并进入(2)，否则结束。(2)如果当前等待队列`wq`不为空，则唤醒睡眠在`wq`上睡眠的task(唤醒一个还是多个，是区分epoll的ET模式还是LT模式)。
3. `epoll_wait`被唤醒继续执行，在`ep_poll`中调用`ep_send_events`将 fd 相关的`epoll_event`和数据拷贝到用户空间传进来的`events`数组里，这个时候就需要遍历 epoll 的**就绪链表**以便收集进程监控的多个fd的`epoll_event`和数据上报给用户进程（在`ep_scan_ready_list`中完成） 将数据放入到`events`数组中，并且返回就绪的fd数量，用户态的此时收到响应后，从`events`中拿到fd，去调用 `accept()`或者`read()`。

> `ep_scan_ready_list`
>
> - 所有的 `epitem` 都转移到 txlist , 而 eventpoll 结构体上的 rdllist 被清空了。
> - `ep_send_events_proc` 遍历 txlist，[详见下文ET/LT区别](#etlt)。
> - 没有处理完的 `epitem` 重新插入到 `rdllist`，如果此时的 `rdllist` 非空，直接唤醒。

总流程：

1. 遍历并清空epoll结构体的ready list：遍历过程中，对于每个epitem收集其返回的events，如果没收集到event，则继续处理其他epi，收集到event就将当前epi的事件和用户传入的数据都copy给用户空间。完成后判断如果是在LT模式，则将当前epi重新放回epoll结构体的ready list。
2. 唤醒工作：如果ready list不为空，则继续唤醒epoll睡眠队列wq上的其他task B。task B从epoll_wait醒来继续前行，重复上面的流程，继续唤醒wq上的其他task C，这样链式唤醒下去。

##### ET(边沿触发)和 LT(水平触发)

当FD有数据可读时，我们调用epoll_wait（或者select、poll）可以得到通知。但是事件通知的模式有两种：

* Level Triggered：简称 LT，也叫做水平触发。只要某个FD中有数据可读，每次调用`epoll_wait`都会被通知。
* Edge Triggered：简称 ET，也叫做边沿触发。只有在某个FD状态变化时调用`epoll_wait`才会被通知。

假设一个客户端socket对应的FD已经注册到了epoll实例中，客户端socket发送了2kb的数据，服务端调用epoll_wait，得到通知说FD就绪，服务端从FD读取了1kb数据回到步骤3（再次调用epoll_wait，形成循环）。如果采用LT模式，因为FD中仍有1kb数据，则回到第3步依然会返回结果，并且得到通知；如果采用ET模式，因为一开始已经消费了FD可读事件，此后FD状态并没有「变化」，因此epoll_wait也不会触发回调，数据无法读取，客户端响应就会超时。

通过 ep_poll，task 是 排他(带 `WQ_FLAG_EXCLUSIVE`标记)加入到 epoll 的等待队列 `wq` 的。也就是说，在 `ep_poll_callback`回调中，只会唤醒一个 task。

LT 的语义：如果事件来了，不管来了几个，只要仍然有未处理的事件，epoll都会通知。

epoll_wait刚刚取到事件的时候的时候，不可能马上就调用accept去处理，事实上，epoll_wait里面的ep_poll中还没返回，这个时候显然符合“仍然有未处理的事件”这个条件，为了实现 LT 语义，需要通知别的同样阻塞在同一个epoll句柄睡眠队列上的进程。

在一个 epoll 上睡眠的多个 task，如果在一个 LT 模式下的 fd 的事件上来，会唤醒 epoll 睡眠队列上的所有 task，而 ET 模式下，仅仅唤醒一个 task。

> LT 语义的实现需要依赖以下两个机制：

###### <span id="etlt">`ep_send_events_proc`: LT 将 fd 重新加到链表</span>

递归逐渐深入：`ep_send_events->ep_scan_ready_list->ep_send_events_proc`

1. 从链表里移除就绪的 `epitem` 并获取对应事件。睡眠 entry 的回调函数的触发只是通知有“事件”，具体需要通过`ep_item_poll(epi, &pt, 1)`获取，返回相应的事件掩码`revents`。
2. 将当前发生的事件`revents`和用户当时传入的event数据部分copy到用户空间`events`数组。如果拷贝过程未顺利完成 则终止遍历过程，将当前`epitem`重新加回就绪链表，剩下的也会在`ep_scan_ready_list`中重新放回就绪链表，直接返回。
3. copy 完成之后，如果是 LT 模式，还会将这个 `epitem` 重新加回到 eventpoll 结构体里的 rdllist 中去：

```c
if (!(epi->event.events & EPOLLET)) {
    list_add_tail(&epi->rdllink, &ep->rdllist);
}
```

###### `ep_scan_ready_list`: 遍历完若链表非空则唤醒等待者

ET 结合NIO能够确保一次性读完Socket中的数据，减少epoll_wait的调用次数，提高效率

LT 因为会保留就绪链表上的节点，因此多个进程阻塞在epoll_wait的系统调用时，会将他们全部唤醒，即惊群。

[深入浅出 Linux 惊群：现象、原因和解决方案 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/385410196)

[再谈 Linux epoll 惊群问题的原因和解决方案 (qq.com)](https://mp.weixin.qq.com/s/xxjCrFH1361iG-srfNL9_Q) 

```c
// ep_scan_ready_list 遍历完成之后...(这将是LT惊群的根源)
// 1. LT 始终会加到链表里  2. 拷贝过程不顺利也会有残留 [语义: 现在仍有未处理的事件]
// 如果“就绪链表”上仍有未处理的epi，且有进程睡眠在wq，则唤醒。
if (!list_empty(&ep->rdllist)) {
    if (waitqueue_active(&ep->wq))
        wake_up_locked(&ep->wq);
}
```

##### 惊群（thundering herd）

当有多个进程或线程在等待同一个事件（比如同一个 socket 上的 `accept`，或 `epoll`/`select` 事件），一旦事件发生，**所有等待的进程或线程都被唤醒**，但最终只有一个能真正处理这个事件，其余的唤醒都是**无效唤醒**，白白消耗了 CPU 资源。这样一来，唤醒和竞争的成本非常高，尤其在高并发场景下，**频繁唤醒多个进程/线程** 会导致性能大幅下降。这就是所谓的 **thundering herd（惊群效应）**：

- 多进程服务器（如 Nginx、Apache）使用 `accept` 等待客户端连接。
- 一个新连接到来时，所有阻塞在 `accept` 的进程都被唤醒。
- 但操作系统只允许一个进程成功 `accept`，其余进程会发现 `accept` 返回 `EAGAIN` 或直接失败，然后又继续休眠等待下次唤醒。

惊群出现的常见位置

- `accept`

- `epoll_wait`（旧内核，水平LT触发时容易）
- `select` / `poll`（本身也是 LT 触发）
- 多线程下等待互斥锁、条件变量

> 解决方案

1️⃣ **多进程 `accept`：**

- Linux 2.6 以后引入 `SO_REUSEPORT`，让每个进程拥有独立的监听 socket，避免惊群。
- 使用 `accept` 锁：自己用互斥锁mutex保护 `accept`，保证同一时刻只有一个进程调用。

2️⃣ **epoll 驱动：**

- 用 `EPOLLET`（边缘触发）代替 `EPOLLLT`（水平触发），减少重复通知。
- 用 `EPOLLONESHOT`，保证事件只通知一次，处理完后再重新注册。

3️⃣ **多线程竞争：**

- 优化锁设计，减少竞争区域或用条件变量精准唤醒。

##### epoll 事件分类

`epoll` 支持多种事件类型，主要有以下几种：

1. **`EPOLLIN` - 可读事件**

- **描述**：表示指定的文件描述符（如 socket）可供读取数据。

- **适用场景**：当一个 socket 有数据可读时，`epoll` 会触发此事件。

- **使用场景**：服务器通常使用此事件来检测客户端是否有数据发送过来。

  **例子**：

  - 对于 TCP socket，`EPOLLIN` 事件表示客户端发送了数据，服务器可以使用 `recv()` 函数读取数据。
  - 对于 监听 socket，`EPOLLIN` 表示有客户端发起了连接请求，服务器可以调用 `accept()` 来接受连接。

2. **`EPOLLOUT` - 可写事件** 

- **描述**：表示指定的文件描述符（如 socket）可以写入数据。

- **适用场景**：当 socket 可以安全地写入数据时，`epoll` 会触发此事件。

- **使用场景**：客户端在发送大量数据时，可能需要等待 socket 可写。此时，`epoll` 通过 `EPOLLOUT` 事件通知应用程序可以写入数据了。

  **例子**：

  - 如果一个 socket 处于阻塞状态，等待写缓冲区可用，`EPOLLOUT` 会被触发，表示可以开始写入数据。

3. **`EPOLLERR` - 错误事件**

- **描述**：表示文件描述符发生了错误。

- **适用场景**：如果连接发生了错误或出错事件（如网络中断），`epoll` 会触发此事件。

- **使用场景**：应用程序通常会检查 `EPOLLERR` 事件来处理错误情况，如关闭连接或执行错误恢复。

  **例子**：

  - 当 socket 发生网络错误，或者远程主机不可达时，`EPOLLERR` 会被触发。

4. **`EPOLLHUP` - 挂起事件（Hangup）**

- **描述**：表示文件描述符发生挂起（通常表示连接关闭）。

- **适用场景**：当连接被关闭，或者流中的另一端挂起时，`epoll` 会触发此事件。

- **使用场景**：客户端或服务器检测到连接关闭时，通常会处理此事件并清理资源。

  **例子**：

  - 当客户端断开连接时，服务器会收到 `EPOLLHUP` 事件。此时，服务器应关闭对应的 socket。

5. **`EPOLLRDHUP` - 远程挂起事件**

- **描述**：表示对端关闭了连接。该事件是对 `EPOLLHUP` 的补充，专门用于表示对端关闭连接时的事件。
- **适用场景**：它是为了处理 **TCP** 连接中远程关闭（例如客户端关闭连接）时的特定事件。
- **使用场景**：与 `EPOLLHUP` 类似，应用程序通常通过这个事件来检测到对端已关闭连接，并可以进行相应处理。

6. **`EPOLLET` - 边缘触发（Edge Triggered）**

- **描述**：`EPOLLET` 是边缘触发模式，意味着当文件描述符的状态发生变化时，事件会被触发一次。边缘触发模式相比于传统的水平触发（Level Triggered），能够更加高效地处理 I/O 操作。
- **适用场景**：在高性能场景中使用，可以避免对文件描述符的重复检查。应用程序需要保证不会丢失事件，并且需要轮询所有事件，直到事件处理完成。
- **使用场景**：通常与 `EPOLLIN`、`EPOLLOUT` 等事件一同使用。

7. **`EPOLLONESHOT` - 单次触发**

- **描述**：表示文件描述符只会被触发一次事件。处理完事件后，`epoll` 会自动停止监视该文件描述符，直到再次通过 `epoll_ctl` 显式注册为监听状态。
- **适用场景**：用于那些只需要处理一次事件的场景，如处理某个特定的请求，处理完后不再关心该文件描述符。
- **使用场景**：一般用于一个事件只处理一次的情况，可以减少事件触发的次数，提高效率。

8. **`EPOLLPRI` - 优先事件**

- **描述**：表示文件描述符上发生了高优先级的事件。通常这种事件的优先级高于常规的 I/O 事件。
- **适用场景**：通常用于处理信号量、优先级消息队列等场景。

9. **`EPOLLWAKEUP` - 唤醒事件**

- **描述**：表示 `epoll` 实例在事件等待期间需要被唤醒。
- **适用场景**：适用于在多线程环境中希望从 `epoll_wait()` 等待事件的线程外部唤醒的情况。

**组合使用的方式**

`epoll` 事件通常可以组合使用，以满足更复杂的场景。例如，应用程序可以同时监视 **`EPOLLIN`** 和 **`EPOLLOUT`**，这样就可以同时处理读和写事件。

```c
struct epoll_event ev;
ev.events = EPOLLIN | EPOLLOUT | EPOLLERR;  // 监听读、写和错误事件
ev.data.fd = fd;
epoll_ctl(epoll_fd, EPOLL_CTL_ADD, fd, &ev);
```

##### 基于epoll的服务端流程示例

[从内核看epoll的实现（基于5.9.9） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/384098769)

1. 服务器通过前文提到的listen()将普通的socket转换成专门用来监听的socket(Listened Socket)
2. 服务端会去调用`epoll_create()`，创建一个epoll实例，epoll实例中包含两个数据

- 红黑树（为空）：`rb_root` 用来去记录需要被监听的 FD(Socket)
- 链表（为空）：`list_head`，用来存放已就绪 FD
  - 创建好了之后，会去调用`epoll_ctl()`函数，此函数将需要监听的数据添加到`rb_root`中去，并且对当前这些存在于红黑树的节点设置回调函数，（同时设置要监听什么类型的事件）当这些被监听的数据一旦准备完成，就会被调用，而调用的结果就是将红黑树的fd添加到list_head中去(但是此时并没有完成)

3. 当第二步完成后，就会调用`epoll_wait()`函数，这个函数会去校验是否有数据准备完毕（因为数据一旦准备就绪，就会被回调函数添加到`list_head`中），wait一段时间后(可配置)，如果等待超时，则返回无数据，如果有，则进一步判断当前是什么事件：

- 如果是**监听Socket**的FD发生了`EPOLLIN`事件，则调用`accept()` 获取已建立的TCP连接，准备IO。
- 如果是**已连接Socket**的`EPOLLIN`事件，则正常读取`socket`的数据

**网络套接字**：在使用 `epoll` 时，最常见的就是网络套接字（如 TCP 套接字）的数据就绪。具体来说，数据就绪意味着：

- **可读事件（`EPOLLIN`）**：当套接字有数据可读时，`epoll_wait` 会返回，表明应用程序可以从套接字中读取数据。
- **可写事件（`EPOLLOUT`）**：当套接字可写时，即缓冲区有足够空间发送数据时，`epoll_wait` 会返回，表明应用程序可以向套接字写入数据。

![1653902845082](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653902845082.png)

### 信号驱动IO (Signal Driven IO)

信号驱动IO是与内核建立SIGIO的信号关联并设置回调，当内核有FD就绪时，会发出SIGIO信号通知用户，期间用户应用可以执行其它业务，无需阻塞等待。

阶段一：

* 用户进程调用sigaction，注册信号处理函数
* 内核返回成功，开始监听FD
* 用户进程不阻塞等待，可以执行其它业务
* 当内核数据就绪后，回调用户进程的SIGIO处理函数

阶段二：

* 收到SIGIO回调信号
* 调用recvfrom，读取
* 内核将数据拷贝到用户空间
* 用户进程处理数据

![1653911776583](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653911776583.png)

当有大量IO操作时，信号较多，SIGIO处理函数不能及时处理可能导致信号队列溢出，而且内核空间与用户空间的频繁信号交互性能也较低。

与NIO要做区分：NIO会一直空轮询，SIGIO只会询问一次。

### 异步IO (Asynchronous IO)

这种方式，不仅仅是用户态在试图读取数据后，不阻塞，而且当内核的数据准备完成后，也不会阻塞

他会由内核将所有数据处理完成后，由内核将数据写入到用户态中，然后才算完成，所以性能极高，不会有任何阻塞，全部都由内核完成，可以看到，异步IO模型中，用户进程在两个阶段都是非阻塞状态。

缺陷：高并发下，内核中积累的IO任务很多，消耗太多系统资源从而导致崩溃，所以要求限流机制。

![1653911877542](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653911877542.png)

### 五种IO模型对比

最后用一幅图，来说明他们之间的区别

同步IO or 异步IO 取决于从内核拷贝到用户空间时是否阻塞

![1653912219712](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1653912219712.png)

