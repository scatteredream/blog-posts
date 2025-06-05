---
name: rpc-interpretation
title: 基于 Netty 的 RPC 框架
date: 2025-02-03
tags: 
- netty
- 动态代理
- spring
- spring-boot
- 自动装配
- spi
- bean
- 注解开发
- 负载均衡
- 序列化
- netty
categories: 项目
---



# 基于 Netty 的 RPC 框架

[补充：压测、遇到的困难、瓶颈、为什么做RPC、容灾、调用方式、重试、幂等。](https://docs.qq.com/doc/p/f8200727e71c50b34456abd09cf70401ac4b0eb9)

项目地址：https://github.com/scatteredream/netty-rpc 

实现要点： 

- 实现了 Netty [心跳机制](#心跳机制)，保持连接。客户端[指数退避](#指数退避)重试连接，服务端用线程池处理请求。可选 HTTP 和 Socket。
- 实现了自定义 RPC [通信协议](#protocol)，[自定义编解码器和拆包解码器](#codec)解决粘包和半包，实现了 Kryo 等5种[序列化方式](#serialization)。
- [注册中心](#center)支持 Nacos 与 Zookeeper，服务发现支持本地缓存、实时监听。除利用健康检查机制外，下线服务还会主动通知注册中心注销，实现优雅下线。支持一致性哈希等3种[负载均衡](#loadbalance)算法。
- 集成 SpringBoot。通过[自定义注解](#annotation)，提供者自动扫描并注册服务 Bean，消费者自动注入[代理对象](#proxy)。自定义 starter 实现[自动装配](#autoconfig)。
- 参考 Dubbo 实现 [SPI](#spi)，支持[序列化](#spi1)、[服务发现](#spi2)等的动态扩展，实现与类型解耦的[单例缓存](#cache)，减少大量冗余的对象创建。

## 为什么做 RPC

1、首先，目前的应用大部分都是分布式/微服务架构，通常各个模块之间都是通过RPC来进行调用的，所以我认为自己写一个RPC项目可以有更加深入的理解RPC的一个原理；

2、其次的话，在写RPC项目的过程中，学会了对Zookeeper、Netty的使用，包括对动态代理、负载均衡、序列化算法的实现，以及对SpringBoot框架的扩展和自动配置的理解。

## RPC 优化瓶颈

1. 网络传输
2. 序列化：CPU密集型操作——复杂序列化格式（如XML）或大数据量会消耗大量CPU资源，增加延迟。二进制更加高效 <u>Kryo</u>
3. 客户端策略：熔断与重试引入额外开销，负载均衡不均匀。<u>一致性哈希</u>
4. 资源管理、内存泄漏：高并发内存分配导致频繁GC，停顿时间过长，<u>单例池</u>
5. 协议设计、编解码：底层可用 HTTP/2。低效的消息边界解析会增加CPU消耗。<u>lengthBasedFrameDecoder</u>

## Netty/NIO

Java NIO（New I/O）是 Java 1.4 引入的非阻塞 I/O API，提供了基于通道（Channel）、缓冲区（Buffer）和选择器（Selector）的异步非阻塞编程模型。它允许单线程处理多个连接，显著提升了高并发场景下的性能。

Netty 是基于 Java NIO 构建的高性能网络编程框架，它简化了 NIO 的复杂操作，提供了更易用的 API，同时解决了 NIO 编程中的许多陷阱（如断连重连、半包读写、心跳处理等）。Netty 广泛应用于各类高性能网络应用，如分布式系统中的 RPC 框架、游戏服务器等。

- **选择 Java NIO**：如果需要完全控制底层实现，应用场景简单，或想深入理解 NIO 原理。
- **选择 Netty**：如果需要快速开发高性能网络应用，避免处理 NIO 的复杂性和陷阱。

| 特性       | Java NIO                           | Netty                                 |
| ---------- | ---------------------------------- | ------------------------------------- |
| API 复杂度 | 复杂，需要深入理解 Selector 等机制 | 简单，封装了底层细节                  |
| 内存管理   | 需手动管理 ByteBuffer              | 自动内存管理，提供更高效的 ByteBuf    |
| 可靠性     | 需要自己处理断连、粘包等问题       | 内置多种编解码器和处理器              |
| 扩展性     | 需自行实现                         | 模块化设计，易于扩展                  |
| 性能       | 高，但需要优化                     | 更高，针对性能做了优化                |
| 应用场景   | 简单应用或学习 NIO 原理            | 复杂网络应用，如 RPC 框架、游戏服务器 |

### Java NIO 核心组件

1. **Channel**：数据的双向通道，类似传统 IO 中的 Stream，但更灵活，可以异步读写。
2. **Buffer**：数据的容器，所有数据必须通过 Buffer 处理。
3. **Selector**：单线程管理多个 Channel 的关键，实现多路复用。
4. **SelectionKey**：表示 Channel 在 Selector 中的注册关系及就绪状态。

### Netty 核心组件

1. **EventLoop**：处理 I/O 操作的多线程事件循环器，替代了原生 Selector。
2. **ChannelPipeline**：处理网络事件的责任链模式实现。
3. **ChannelHandler**：处理或拦截 I/O 事件的接口。
4. **ByteBuf**：更高效的字节缓冲区，替代了原生 ByteBuffer。



## 项目结构

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/项目架构图.png" alt="项目架构图" style="zoom:67%;" />

`consumer`模块：服务的消费者，依赖于 `rpc-client-spring-boot-starter` 模块；

`provider-api`模块：服务提供者暴露的API；

`provider`模块：服务的提供者，依赖于 `rpc-server-spring-boot-starter` 模块：

`rpc-client-spring-boot`模块：rpc 客户端模块，封装客户端发起的请求过程，提供服务发现、动态代理，网络通信等功能；

`rpc-client-spring-boot-stater`模块：是`rpc-client-spring-boot`的stater模块，负责引入相应依赖进行自动配置；

`rpc-framework-core`模块：是rpc核心依赖，提供负载均衡、服务注册发现、消息协议、消息编码解码、序列化算法；

`rpc-server-spring-boot`模块：rpc 服务端模块，负责启动服务，接受和处理RPC请求，提供服务发布、反射调用等功能；

`rpc-server-spring-boot-stater`模块：是`rpc-server-spring-boot`的stater模块，负责引入相应依赖进行自动配置；



<!-- more -->

## RPC

RPC 又称远程过程调用（Remote Procedure Call），用于解决分布式系统中服务之间的调用问题。通俗地讲，就是开发者能够像调用本地方法一样调用远程的服务。一个最基本的RPC框架的基本架构如下图所示：

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/简单RPC架构图.png" alt="简单RPC架构图" style="zoom: 45%;" />

RPC框架一般必须包含三个组件，分别是**客户端、服务端**以及**注册中心**，一次完整的 RPC 调用流程一般为：

1. 服务端启动服务后，将他提供的服务列表发布到注册中心（服务注册）；
2. 客户端会向注册中心订阅相关的服务地址（服务订阅）；
3. 客户端通常会利用本地代理模块 Proxy 向服务端发起远程过程调用，Proxy 负责将调用的方法、参数等数据转化为网络字节流；
4. 客户端从服务列表中根据负载均衡策略选择一个服务地址，并将数据通过网络发送给服务端；
5. 服务端得到数据后，调用对应的服务，然后将结果通过网络返回给客户端。

## 调用方式

成熟的 RPC 框架一般会提供四种调用方式，分别为同步 Sync、异步 Future、回调 Callback和单向 Oneway。RPC 框架的性能和吞吐量与合理使用调用方式是息息相关的，下面我们逐一介绍下四种调用方式的实现原理。

- Sync 同步调用。客户端线程发起 RPC 调用后，当前线程会一直阻塞，直至服务端返回结果或者处理超时异常。Sync 同步调用一般是 RPC 框架默认的调用方式，为了保证系统可用性，客户端设置合理的超时时间是非常重要的。虽说 Sync 是同步调用，但是客户端线程和服务端线程并不是同一个线程，实际在 RPC 框架内部还是异步处理的。Sync 同步调用的过程如下图所示。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/Sync同步调用.png" alt="Sync同步调用" style="zoom:67%;" />

- Future 异步调用。客户端发起调用后不会再阻塞等待，而是拿到 RPC 框架返回的 Future 对象，调用结果会被服务端缓存，客户端自行决定后续何时获取返回结果。当客户端主动获取结果时，该过程是阻塞等待的。Future 异步调用过程如下图所示。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/Future异步调用.png" alt="Future异步调用" style="zoom:67%;" />

- Callback 回调调用。如下图所示，客户端发起调用时，将 Callback 对象传递给 RPC 框架，无须同步等待返回结果，直接返回。当获取到服务端响应结果或者超时异常后，再执行用户注册的 Callback 回调。所以 Callback 接口一般包含 onResponse 和 onException 两个方法，分别对应成功返回和异常返回两种情况。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/Callback回调调用.png" alt="Callback回调调用" style="zoom:67%;" />

- Oneway 单向调用。客户端发起请求之后直接返回，忽略返回结果。Oneway 方式是最简单的，具体调用过程如下图所示。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/Onway单向调用.png" alt="Onway单向调用" style="zoom:67%;" />

四种调用方式都各有优缺点，很难说异步方式一定会比同步方式效果好，在不用的业务场景可以按需选取更合适的调用方式。

**这是一个同步调用的 RPC 框架。** 

## <mark>framework-core</mark>

**config** 线程池配置类 **factory** 单例工厂和线程池工厂

服务消费方包装好 `RpcRequest`，做完服务发现之后发送网络请求，使用 `Promise`来进行异步请求的同步等待 `await(timeout)`与收到 `RpcResponse`之后的 `setSuccess(rpcMessage)`。

服务提供方收到 `RpcRequest`，根据服务名称在本地注册中心找到实现类，执行调用，返回结果封装在 `RpcResponse`内。

![责任链](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250423123418951.png)

### <span id="protocol">common（消息体定义、注册信息）</span>

包括消息体的三种类型，以及注册中心提供的服务信息 `ServiceInfo`。

- `RpcRequest` 包含 服务名称（服务名+版本）、方法名、参数类型、参数
- `RpcResponse` 包含 返回值、发生异常时的异常信息
- `HeartBeatMessage` 包含字符串（PING/PONG）
- `ServiceInfo` 应用名称、服务名称、版本号、服务提供方主机地址、端口号。

### utils（ServiceUtil）

`String serviceKey(String serviceName, String version)` 根据服务名称和版本号生成注册服务的key

`Map toMap(ServiceInfo serviceInfo)` 将ServiceInfo转换成Map 用于nacos注册

`ServiceInfo toServiceInfo（Map map）` 将Map转换成ServiceInfo 用于nacos发现

> nacos instance 的metadata是一个map，键和值都是字符串。
>
> port是 int 类型，Gson 解析出 8080 会变成 Number 类型 8080.0 变成Double，因此要把8080转成字符串。
>
> `map.put("port", serviceInfo.getPort().toString());` 
>
> `map.put("port", Integer.parseInt(map.getOrDefault("port", "0").toString()))` 

### enums（消息头参数）

- `MessageStatus` `code`=01 代表成功/失败
- `MessageType` `type`=01表示正常的请求与响应 23表示心跳的ping 与pong
- `SerializationType`  `type` = 0-4 分别对应5种序列化算法，可以根据输入的序列化算法名称匹配对应的序列化方式，默认为Hessian，也可根据type匹配。

### protocol/constant（协议定义相关）

既然 RPC 是远程调用，必然离不开网络通信协议。客户端在向服务端发起调用之前，需要考虑采用何种方式将调用信息进行编码，并传输到服务端。因为 RPC 框架对性能有非常高的要求，所以通信协议应该越简单越好，这样可以减少编解码的性能损耗。RPC 框架可以基于不同的协议实现，大部分主流 RPC 框架会选择 TCP、HTTP 协议，出名的 gRPC 框架使用的则是 HTTP2。TCP、HTTP、HTTP2 都是稳定可靠的，但其实使用 UDP 协议也是可以的，具体看业务使用的场景。成熟的 RPC 框架能够支持多种协议，例如阿里开源的 Dubbo 框架被很多互联网公司广泛使用，其中可插拔的协议支持是 Dubbo 的一大特色，这样不仅可以给开发者提供多种不同的选择，而且为接入异构系统提供了便利。

- `RpcMessage` 封装好的 Rpc 协议信息
  - `MessageHeader header` 消息头
  - `Object body` 携带的消息内容（可能是 `RpcRequest`，`RpcResponse`，也可能是心跳 `HeartBeatMessage`）
- `MessageHeader`  消息头
  - 4B 魔数 1B 版本号  均有**默认填充** 
  - 1B `serializerType`
  - 1B `messageType` 
  - 1B `messageStatus` 
  - 4B `serialNumber` 有一个原子变量，每get一次就自增
  - 4B `length` 
  - **Builder 建造者模式** 链式构造 build

> 设计模式：建造者模式
>
> - **解决复杂对象的构造问题**：当一个对象需要多个参数（尤其包含大量可选参数）时，传统构造方法会变得臃肿且难以维护。
> - **避免“重叠构造器”反模式**：无需编写多个不同参数组合的构造函数。
> - **防止对象的不一致状态**：确保对象在构造完成后处于完整且一致的状态。
>
> | 优势                 | 说明                                                         |
> | :------------------- | :----------------------------------------------------------- |
> | **参数灵活**         | 可选参数可自由组合，避免编写多个重载构造函数。               |
> | **代码可读性高**     | 链式调用清晰表达参数含义（如 `.age(30).email("...")`）。     |
> | **对象状态一致性**   | 通过 `build()` 方法统一校验参数，确保对象构造完成后合法且完整。 |
> | **与不可变对象兼容** | 适合构建不可变（`final`）对象，所有参数在构造时一次性设置。  |

### <span id="serialization"><mark>serialization（序列化相关）</mark></span>

客户端和服务端在通信过程中需要传输哪些数据呢？这些数据又该如何编解码呢？如果采用 TCP 协议，你需要将调用的接口、方法、请求参数、调用属性等信息序列化成二进制字节流传递给服务提供方，服务端接收到数据后，再把二进制字节流反序列化得到调用信息，然后利用反射的原理调用对应方法，最后将返回结果、返回码、异常信息等返回给客户端。所谓序列化和反序列化就是将对象转换成二进制流以及将二进制流再转换成对象的过程。因为网络通信依赖于字节流，而且这些请求信息都是不确定的，所以一般会选用通用且高效的序列化算法。比较常用的序列化算法有 FastJson、Kryo、Hessian、Protobuf 等，这些第三方序列化算法都比 Java 原生的序列化操作都更加高效。Dubbo 支持多种序列化算法，并定义了 Serialization 接口规范，所有序列化算法扩展都必须实现该接口，其中默认使用的是 Hessian 序列化算法。

序列化对于远程调用的响应速度、吞吐量、网络带宽消耗等同样也起着至关重要的作用，是我们提升分布式系统性能的最关键因素之一。判断一个编码框架的优劣主要从以下几个方面：

```undefined
1. 是否支持跨语言，支持语种是否丰富
2. 编码后的码流
3. 编解码的性能
4. 类库是否小巧，API使用是否方便
5. 使用者开发的工作量和难度。
```

- `Serialization` 接口，定义了 `serialize()`和 `deserialize()` 两个方法 共有5个实现（Gson、Kryo、JDK、Protostuff、Hessian）使用 SPI 标注，表示这个类是可扩展的（可插拔）
- `SerializationFactory` 工厂模式，使用工厂方法创建序列化实例。

> 设计模式：工厂模式
>
> - 提供了一种创建对象的方式，使得创建对象的过程与使用对象的过程分离。
> - 工厂模式提供了一种创建对象的方式，而无需指定要创建的具体类。
> - 通过使用工厂模式，可以将对象的创建逻辑封装在一个工厂类中，而不是在客户端代码中直接实例化对象，这样可以提高代码的可维护性和可扩展性。

**Gson**：默认不支持class序列化，需要自己实现

**Kryo**：Kryo 线程不安全，所以使用 ThreadLocal 保存 kryo 对象

**Protostuff**：提前分配好 LinkedBuffer，避免每次进行序列化都需要重新分配 buffer 内存空间

序列化无需双方约定，只需要客户端规定好就可以，处理完消息头(消息头直接是以值的形式)之后，双方的codec自己会根据对应字段选择相同的序列化/反序列化方式将处理消息体。五种序列化算法的比较如下：

| 序列化算法     | **优点**                 | **缺点**         |
| -------------- | ------------------------ | ---------------- |
| **Kryo**       | 速度快，序列化后体积小   | 跨语言支持较复杂 |
| **Hessian**    | 默认支持跨语言           | 较慢             |
| **Protostuff** | 速度快，基于protobuf     | 需静态编译       |
| **Json**       | 使用方便                 | 性能一般         |
| **Jdk**        | 使用方便，可序列化所有类 | 速度慢，占空间   |

性能对比图，时间单位为 nanos：

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/序列化性能对比.png" alt="序列化性能对比" style="zoom:100%;" />

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2015/f615deb9.png)

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/序列化性能比较.png" alt="序列化性能比较图" style="zoom:100%;" />

空间占用如下

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2015/37cf0654.png)

### exception（异常）

定义了RPCException和SerializeException来封装框架的异常

### <span id="codec"><mark>codec（编解码、粘包处理）</mark></span>

`MessageToByteEncoder<T>` 负责编码 `ByteToMessageDecoder` 负责解码



- 在 Netty 的 Pipeline 中，通常将两者结合使用：

  1. `LengthFieldBasedFrameDecoder`：解决粘包/半包， 提取完整帧。
  1. `MessageToMessageCodec`：拆分后的数据帧（如`ByteBuf`）与业务对象转换。


```java
ch.pipeline()
  // 第一步：处理粘包/半包，输出完整ByteBuf帧
  .addLast(new LengthFieldBasedFrameDecoder(1024, 0, 4, 0, 4))
  // 第二步：将ByteBuf解码为Java对象
  .addLast(new MyMessageCodec())
  // 第三步：处理业务逻辑
  .addLast(new BusinessHandler());
```

- `SharableRpcMessageCodec`
  - 可共享的 Rpc 消息编码解码器，继承自 `MessageToMessageCodec<ByteBuf, RpcMessage>` 重写其 `encode(ctx, msg, out)` 与 `decode(ctx, msg, out)` 方法。使用此编解码器必须配合 `RpcFrameDecoder` 进行使用， 以保证得到完整的数据包。不同于 `io.netty.handler.codec.ByteToMessageCodec` 的编解码器，共享编解码器无需 保存 `ByteBuf` 的状态信息。
  - 为了支持多线程共享，编解码器 Handler 应该是 **无状态或线程安全** 的。`@Sharable` 是 **Netty 框架**中一个重要的注解，用于标记 `ChannelHandler` 是否可以被多个 `Channel`（连接）**安全共享**。（默认情况下会为每一个Channel创建一个新的Handler实例，不能复用）
    - 仅当 `Handler` **无状态**或**线程安全**时，才应添加 `@Sharable`：
    - **统计类 Handler**（如监控请求总数、连接数）。
    - **无状态 Handler**（不保存任何与特定 `Channel` 相关的数据）。
    - **工具类 Handler**（如日志记录、编解码器等）。
  - 入：`Bytebuf` -> `RpcMessage` （解码）：先读魔数，然后是读版本号，之后是其余的字段，根据信息构造出消息头，然后根据消息类型反序列化消息体，rpc请求和响应的消息分别对应的是 `RpcRequest` 和 `RpcResponse`，心跳对应的是 `String` ，将这些信息拼成一个 `RpcMessage`，传递到下一个出站处理器
  - 出：`RpcMessage`-> `Bytebuf` （编码）：消息头发出时根据 `RpcMessage` 的消息头直接发送协议的对应字段，发到 `length` 以后暂停，将 `body` 序列化成为`byte` 数组，之后设置消息头的长度字段，然后将长度字段也写入，最后将消息体发出，传递到下一个出站处理器
- `RpcFrameDecoder` 

- 粘包：发送 abc def，接收 abcdef


* 原因
  * 应用层：接收方 ByteBuf 设置太大（Netty 默认 1024）
  * 滑动窗口：假设发送方 256 bytes 表示一个完整报文，但由于接收方处理不及时且窗口大小足够大，这 256 bytes 字节就会缓冲在接收方的滑动窗口中，当滑动窗口中缓冲了多个报文就会粘包
  * Nagle 算法：会造成粘包

- 半包：发送 abcdef，接收 abc def


* 原因
  * 应用层：接收方 ByteBuf 小于实际发送数据量
  * 滑动窗口：假设接收方的窗口只剩了 128 bytes，发送方的报文大小是 256 bytes，这时放不下了，只能先发送前 128 bytes，等待 ack 后才能发送剩余部分，这就造成了半包
  * MSS 限制：当发送的数据超过 MSS 限制后，会将数据切分发送，就会造成半包

- 粘包拆包编码器，固定长度的帧解码器，通过约定用定长字节表示接下来数据的长度。非共享，保存 `ByteBuf` 的状态信息。继承自 `LengthFieldBasedFrameDecoder` 


```java
public RpcFrameDecoder() {
    this(1024, 12, 4);
}
/**
 * @param maxFrameLength    数据帧的最大长度
 * @param lengthFieldOffset 长度域的偏移字节数
 * @param lengthFieldLength 长度域所占的字节数
 */
public RpcFrameDecoder(int maxFrameLength, int lengthFieldOffset, int lengthFieldLength) {
    super(maxFrameLength, lengthFieldOffset, lengthFieldLength);
}
```

- 解决方案

  - 短连接：发一次数据包建立一次连接，这样连接建立到连接断开之间就是一次消息边界，缺点是效率低；
  - 固定长度：每一条消息采用固定长度，缺点是浪费空间；
  - 分隔符：每一条消息采用分隔符，例如 \n ，缺点是需要转义；
  - 消息长度+消息内容：每一条消息分为 header 和 body，header 中包含 body 的长度（推荐）；
- TCP 是无边界的字节流，考虑到 RPC 框架特性，排除了固定长度/分隔符/channel短连接，采用消息长度+消息体的模式。不同 `Channel`之间肯定会互相干扰，因此不能使用 `@Sharable` 注解

### <span id="loadbalance"><mark>loadbalance（负载均衡）</mark></span>

本项目参考 Dubbo 实现了 Random、RoundRobin、ConsistentHash 三种负载均衡算法

在分布式系统中，服务提供者和服务消费者都会有多台节点，如何保证服务提供者所有节点的负载均衡呢？客户端在发起调用之前，需要感知有多少服务端节点可用，然后从中选取一个进行调用。客户端需要拿到服务端节点的状态信息，并根据不同的策略实现负载均衡算法。负载均衡策略是影响 RPC 框架吞吐量很重要的一个因素，下面我们介绍几种最常用的负载均衡策略。

- [Random](https://cn.dubbo.apache.org/zh-cn/docsv2.7/dev/source/loadbalance/#21-randomloadbalance) 是一个简单，高效的负载均衡实现，因此 Dubbo 选择它作为缺省实现。
- Round-Robin 轮询。Round-Robin 是最简单有效的负载均衡策略，并没有考虑服务端节点的实际负载水平，而是依次轮询服务端节点。
- [Weighted Round-Robin 权重轮询](https://cn.dubbo.apache.org/zh-cn/docsv2.7/dev/source/loadbalance/#24-roundrobinloadbalance)。对不同负载水平的服务端节点增加权重系数，这样可以通过权重系数降低性能较差或者配置较低的节点流量。权重系数可以根据服务端负载水平实时进行调整，使集群达到相对均衡的状态。
- [Least Active 最少连接数](https://cn.dubbo.apache.org/zh-cn/docsv2.7/dev/source/loadbalance/#22-leastactiveloadbalance)。客户端根据服务端节点当前的连接数进行负载均衡，客户端会选择连接数最少的一台服务器进行调用。只是服务端其中一种维度，我们可以演化出最少请求数、CPU 利用率最低等其他维度的负载均衡方案。
- Fastest Response 最快响应
- (Consistent)Hash **相同参数会落到同一个节点上，这样做可以保证相同的请求被路由到同一个服务实例（比如为了缓存或者会话粘性）。** 

#### 一致性哈希实现

概括：缓存TreeMap哈希环、拼接参数实现负载均衡、通过计算40次拼接字符串md5+md5分为4段的方式做出了160个虚拟节点。

[Consistent Hash 一致性哈希](https://cn.dubbo.apache.org/zh-cn/docsv2.7/dev/source/loadbalance/#23-consistenthashloadbalance) 。目前主流推荐的负载均衡策略，Consistent Hash 是一种特殊的 Hash 算法，在服务端节点扩容或者下线时，尽可能保证客户端请求还是固定分配到同一台服务器节点。Consistent Hash 算法是采用哈希环来实现的，通过 Hash 函数将对象和服务器节点放置在哈希环上，一般来说服务器可以选择 IP + Port 进行 Hash，然后为对象选择对应的服务器节点，在哈希环中顺时针查找距离对象 Hash 值最近的服务器节点。**相同参数会落到同一个节点上，这样做可以保证相同的请求被路由到同一个服务实例（比如为了缓存或者会话粘性）。** 

- 实现细节：缓存 `consistentHashSelector`，如果 `invokers`的`hashCode`变化了，说明服务发生变化。创建新的`selector`。
- 一致性哈希能负载均衡的根本原因还是在key上，如果一直请求的是相同的服务，如果不做任何key的修饰，只使用类全限定名+方法名的key，所有的请求都会打到同一个节点上，这是我们不想看到的，因此不同参数的请求也应该路由到不同的节点，相同参数值的请求应该路由到同一节点，把`selectKey`后边拼接一下参数值。通过`selector.select(selectKey)`进行节点寻找。
- `TreeMap<Long,ServiceInfo> virtualInvokers` 哈希值和 invoker 信息要做映射，还要方便比较哈希的大小，TreeMap里面是[红黑树](https://javaguide.cn/cs-basics/data-structure/red-black-tree.html)，查询性能很好，并且提供了方便的比较api。
- 为什么选择md5？
  - MD5虽然安全性有些问题，但是作为一个散列函数，他拥有雪崩效应，输入1位的变化就能使结果发生50%的改变，并且每个十六进制字符分布的熵值接近4，MD5 分组的标准差远低于普通哈希。反观hashCode，若输入数据存在模式（如递增 ID），hashCode 可能呈现线性分布，导致分组不均匀。String.hashCode () 在处理大量字符串时，可能出现哈希冲突导致的分组倾斜。
  - 性能：md5 平均300ns左右 hashcode 30ns，相比之下网络IO开销才是瓶颈，因此md5值得使用。

- 默认有160个<mark>虚拟节点</mark>，对于每一个`invoker`，首先我们拿到其ip地址，做md5摘要得到一个128位16字节的字节数组，将其从前至后分为4段，可以算出4个不同的 32位哈希值（`long`），把哈希值->invoker放入 `virtualInvokers`里。那么为了得到更多的哈希值，我们可以将ip地址多次利用，既然一个md5可以生成4个哈希值，那么虚拟节点一共160个，通过40个循环，然后每个循环将对应的`i`拼接到ip地址就可以。
- `select(selectKey)`: 传进来的key做md5摘要，取其前4个字节的hash，在TreeMap中通过 `virtualInvokers.ceilingEntry(hash)` 找到比其大的最小Entry，如果hash位于hash环的最末端（很大），那么就通过`virtualInvokers.firstEntry(hash)` 返回最小的hash对应的 Entry，因此我们就找到了节点。

此外，负载均衡算法可以是多种多样的，客户端可以记录例如健康状态、连接数、内存、CPU、Load 等更加丰富的信息，根据综合因素进行更好地决策。

- `LoadBalance` 接口定义了 `ServiceInfo select(List<ServiceInfo> invokers, RpcRequest request)` ，SPI 注解表示可扩展。
- `AbstractLoadBalance` 实现了 `select` 考虑到 无invokers、单invokers 的情况，体现了 模版方法 的设计模式，让实现类自己实现 `doSelect` 方法。

> 设计模式：模板方法 （示例：HttpServlet 的 service() 方法）
>
> ```java
> @Override
> protected void service(HttpServletRequest req, HttpServletResponse resp)
>         throws ServletException, IOException {
>     String method = req.getMethod();
>     if (method.equals("GET")) {
>         doGet(req, resp);
>     } else if (method.equals("POST")) {
>         doPost(req, resp);
>     }
>     // 其他如 PUT、DELETE 等也可以类似处理
> }
> ```

### <span id="centor"><mark>discovery/registry</mark></span>

在分布式系统中，不同服务之间应该如何通信呢？传统的方式可以通过 HTTP 请求调用、保存服务端的服务列表等，这样做需要开发者主动感知到服务端暴露的信息，系统之间耦合严重。为了更好地将客户端和服务端解耦，以及实现服务优雅上线和下线，于是注册中心就出现了。

在 RPC 框架中，主要是使用注册中心来实现服务注册和发现的功能。服务端节点上线后自行向注册中心注册服务列表，节点下线时需要从注册中心将节点元数据信息移除。客户端向服务端发起调用时，自己负责从注册中心获取服务端的服务列表，然后在通过负载均衡算法选择其中一个服务节点进行调用。以上是最简单直接的服务端和客户端的发布和订阅模式，不需要再借助任何中间服务器，性能损耗也是最小的。

现在思考一个问题，服务在下线时需要从注册中心移除元数据，那么注册中心怎么才能感知到服务下线呢？我们最先想到的方法就是节点主动通知的实现方式，当节点需要下线时，向注册中心发送下线请求，让注册中心移除自己的元数据信息。但是如果节点异常退出，例如断网、进程崩溃等，那么注册中心将会一直残留异常节点的元数据，从而可能造成服务调用出现问题。

为了避免上述问题，实现服务优雅下线比较好的方式是采用主动通知 + 心跳检测的方案。除了主动通知注册中心下线外，还需要增加节点与注册中心的心跳检测功能，这个过程也叫作探活。心跳检测可以由节点或者注册中心负责，例如注册中心可以向服务节点每 60s 发送一次心跳包，如果 3 次心跳包都没有收到请求结果，可以任务该服务节点已经下线。

由此可见，采用注册中心的好处是可以解耦客户端和服务端之间错综复杂的关系，并且能够实现对服务的动态管理。服务配置可以支持动态修改，然后将更新后的配置推送到客户端和服务端，无须重启任何服务。

心跳检测：

Nacos: 客户端默认5秒发送一次心跳，服务端15s未收到心跳会将实例标记为不健康，超过30s就会删除实例

ZK: 临时节点（Ephemeral）本身就具备心跳检测特性，超时后节点自动删除（默认40s）

#### Nacos

> Nacos 是阿里开源的一站式服务发现、配置管理和服务管理平台。它可以作为注册中心使用，也支持配置中心功能，是服务治理的核心组件之一，类似于 Eureka、Consul、Zookeeper。*适合AP，高可用，健康检查机制更加可靠*

| 特性         | Nacos   | Eureka | Consul | Zookeeper |
| ------------ | ------- | ------ | ------ | --------- |
| CAP 理论倾向 | AP(+CP) | AP     | CP     | CP        |
| 支持健康检查 | ✅       | ❌      | ✅      | ✅         |
| 支持配置中心 | ✅       | ❌      | 部分   | ❌         |
| UI 管理界面  | ✅       | ✅      | ✅      | ❌         |
| 动态感知能力 | ✅       | ✅      | ✅      | ❌（延迟） |

**服务注册**（Service Registration）：一个服务启动后把自己的信息（如 IP、端口等）注册到 Nacos。

**服务发现**（Service Discovery）：别的服务通过服务名从 Nacos 查询到这个服务的实例信息，然后进行调用。

<mark>Nacos 注册中心的核心功能（命名服务 NamingService）</mark>

在微服务架构中，服务的实例通常是动态变化的（可能会横向扩展、下线），不能写死 IP 和端口。使用命名服务的好处是：

- 动态管理服务实例
- 实现服务的自动上下线
- 支持客户端负载均衡
- 解耦服务之间的调用

| 特性           | Zookeeper     | Nacos     | Eureka    | Consul |
| -------------- | ------------- | --------- | --------- | ------ |
| CAP 理论倾向   | **CP**        | AP        | AP        | CP     |
| 强一致性       | ✅             | ❌         | ❌         | ✅      |
| 服务宕机会下线 | ✅（临时节点） | ✅（心跳） | ❌（延迟） | ✅      |
| Watcher 机制   | ✅             | ❌         | ❌         | ✅      |
| 配置中心功能   | ❌             | ✅         | ❌         | 有限   |

| 名称                  | 含义                                                         |
| --------------------- | ------------------------------------------------------------ |
| 服务（Service）       | 一个服务的逻辑名，如 `user-service`                          |
| 实例（Instance）      | 服务的一个具体实现，如 `user-service 192.168.1.10:8080 healthy metadata` |
| 命名空间（Namespace） | 用于服务隔离，可以做多环境管理，比如开发、测试、生产环境     |
| 分组（Group）         | 用于对服务进行逻辑分组，默认是 `DEFAULT_GROUP`               |

```java
NamingService naming = NacosFactory.createNamingService("127.0.0.1:8848");
// 注册服务
naming.registerInstance("user-service", "192.168.1.100", 8080);
// 获取服务实例列表
List<Instance> instances = naming.getAllInstances("user-service");
```

```yaml
spring:
  application:
    name: user-service
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
```

#### Zookeeper

> Zookeeper 是一个开源的**分布式协调框架**，用于实现分布式系统中的统一配置管理、命名服务、分布式锁、主从选举等功能。在微服务体系中，它也可以作为注册中心使用，比如 Dubbo 体系就常用 ZK。*CP，强一致性，不过临时节点删除可能有一点延迟* 

| 功能点     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 服务注册   | 服务提供者在 ZooKeeper 的 `/services/{serviceName}/providers` 路径下创建<mark>临时节点</mark>（Ephemeral Node），节点内容为服务元数据。 |
| 服务发现   | 消费者监听 `/services/{serviceName}/providers` 的子节点变化（<mark>Watcher</mark> 机制），实时获取服务列表。 |
| 健康检查   | 依赖<mark>临时节点</mark>的机制，服务宕机/断连节点会自动消失。 |
| 动态感知   | 使用 Watcher 机制，监听节点变化自动感知上下线，通过 Watcher 通知消费者刷新本地缓存。 |
| 数据一致性 | 强一致性（CP系统），适合对一致性要求高的场景。               |

ZooKeeper 的 Watcher 机制保证消费者在节点增删时立即收到通知，触发本地缓存更新。

若 ZooKeeper 会话超时，消费者需重新注册 Watcher 并全量拉取数据。

缓存当前服务列表的快照，并在 Watcher 回调中对比差异，实现增量更新。

#### 服务发现

- `ServiceDiscovery` 接口
  - `ServiceInfo discover(RpcRequest request)` 服务发现
  - `default List<ServiceInfo> getServices(String serviceName)` 返回所有服务提供方
  - `void destroy()` 摧毁连接

- `NacosServiceDiscovery` 

  - `LoadBalance loadBalance` 负载均衡算法
  - `Map<String, List<ServiceInfo>> serviceMap` 本地缓存
  - 构造器：提供负载均衡算法，nacos 地址
  - `void destroy()` 关闭 命名服务

  - `ServiceInfo discover(RpcRequest request)` 调用`getServices()`得到服务信息传给负载均衡器返回结果

  - `List<ServiceInfo> getServices(String serverName)` 

    先看看缓存里有没有，没有就从NamingService里获取`List<Instances>`，通过stream映射到`List<ServiceInfo>`，将list作为服务名对应的值加到本地缓存里面。与此同时，订阅 NamingEvent，`subscribe(String serviceName, EventListener el)` 实现 `EventListener` 的 `onEvent(Event e)`，`Event` 转成 `NamingEvent`，从 NamingEvent 获取服务信息`getInstances`，更新至本地缓存。

- `ZookeeperServiceDiscovery` 

  - `ServiceDiscovery<ServiceInfo> serviceDiscovery` Curator 提供的服务发现组件

  - `Map<String, List<ServiceInfo>> serviceMap`

    将服务列表缓存到本地内存，当服务发生变化时，由 serviceCache 进行服务列表更新操作，当 zk 挂掉时，将保存当前服务列表以便继续提供服务。

  - `Map<String, ServiceCache<ServiceInfo>> serviceCacheMap`  

    将在zk中的服务数据缓存至本地，并监听服务变化，实时更新缓存
    服务本地缓存，将服务缓存到本地并增加 watch 事件，当远程服务发生改变时自动更新服务缓存


  - 构造器：提供负载均衡算法，zk地址，使用curator构建servicediscovery

  - `void destroy()` 关闭缓存，关闭服务发现中心，关闭客户端


  - `ServiceInfo discover(RpcRequest request)` 调用`getServices()`得到服务信息传给负载均衡器返回结果


  - `List<ServiceInfo> getServices(String serverName)` 

    先查本地缓存，没有就构建缓存 `ServiceCache<ServiceInfo>`（这个是实时更新的缓存），使用建造者模式赋值服务名称。添加缓存事件监听器 `ServiceCacheListener`：重写 `cacheChanged()`，一旦缓存发生改变，就`getInstances` 获取 `List<ServiceInstance<ServiceInfo>>` 通过 `getPayload()` 获取`ServiceInfo`。连接状态改变只打印信息。将<服务名，对应的实时缓存>存到 `ServiceCacheMap` 里面，也存到本地缓存里面。

#### 服务注册

- `ServiceRegistry`
  - `register(ServiceInfo info)` 注册
  - `unregister(ServiceInfo info)` 注销
  - `destroy()` 摧毁连接
- `NacosServiceRegistry` 
  - 注册：构建 `Instance`，将 `ServiceInfo` 转为 `Map` 作为其 metadata 进行注册
  - 注销：构建 `Instance`，将 `ServiceInfo` 转为 `Map` 作为其 metadata 进行注销
- `ZookeeperServiceRegistry`
  - 注册：将 `ServiceInfo` 构建为 `ServiceInstance<ServiceInfo>` ，然后进行注册
  - 注销：将 `ServiceInfo` 构建为 `ServiceInstance<ServiceInfo>` ，然后进行注销

#### 总结

Nacos 直接拿 `NamingService` 使用就可以，`Instance` 中的 **metadata** 是 `ServiceInfo` 转换成的 Map

- 服务发现的时候，数据来源就是 `NamingService` 本身。

ZK 需要手动用 Curator API 构造注册中心， `ServiceInstance<ServiceInfo>` 的 **payload** 是 `ServiceInfo` 

- 服务发现的时候，数据主要来源是本地的实时缓存 `ServiceCache<ServiceInfo>`。

### <span id="spi">extension （SPI）</span>

已实现，参考Dubbo部分源码，实现了自定义的SPI机制，目前仅支持根据接口类型加载配置文件中的所有具体的扩展实现类，并且可以根据指定的key获取特定的实现类，具体实现类逻辑在 `com.wxy.rpc.core.extension.ExtensionLoader` 中。

服务存储目录在 `resource/META-INF/extensions`

```config
protostuff=com.wxy.rpc.core.serialization.protostuff.ProtostuffSerialization
kryo=com.wxy.rpc.core.serialization.kryo.KryoSerialization
json=com.wxy.rpc.core.serialization.json.JsonSerialization
jdk=com.wxy.rpc.core.serialization.jdk.JdkSerialization
hessian=com.wxy.rpc.core.serialization.hessian.HessianSerialization
```

与类解耦的单例生成器 Holder。

#### <span id="cache">ExtensionLoader</span>

ExtensionLoader 通过一系列map缓存减少无用的对象创建：

`Map<Class<?>, ExtensionLoader<?>> classToLoaderMap`  loader缓存（接口，loader）

- 关联方法：`ExtensionLoader<T> getExtensionLoader(Class<T> type)` type必须有SPI注解才能返回对应的Loader

![单例缓存](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250426001316156.png)

#### ExtensionFactory

然后使用工厂类，先根据type获取loader，然后根据name得到对象。

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250426001516610.png" alt="Factory" style="zoom:50%;" />

#### AutoConfiguration

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250426001729767.png" alt="自动装配类" style="zoom:50%;" />

如图，单例工厂，从配置读取 `key`，然后让加载器加载 `key` 对应的实现类。

#### <span id="spi1">序列化</span>

序列化全程不需要以bean的形式出现，因为`framework-core`内部不好移动代码。

客户端需要从配置文件读取方式名来获取编号写入消息头（`key->type`）。编解码器需要按照消息头的编号获取方式名（`type->key`）然后借此拿到实例（`key->object`）。而`key->object`的映射就是SPI做的事情。

在工厂类的静态代码块中使用反射框架 Reflections 填充`key,type`的互相映射。

![image-20250502004542885](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250502004542885.png)

#### <span id="spi2">服务注册/发现</span>

原先的策略：registryAddr作为 ServiceRegistry 实现类的构造器参数，构造器里开启客户端。但是发现如果从ExtensionFactory里面拿对象，确实是解耦了，但是在bean方法里根本无法注入registryAddr给ServiceRegistry，而且spi只支持空参构造。因此改换思路，要实现完全解耦，就需要将接口改成抽象类，这样ServiceRegistry的子类（实现类）就能拿到参数开启客户端了，另开一个方法独自开启客户端。

![image-20250502004633513](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250502004633513.png)



## server-spring-boot

### <mark>store</mark>

`LocalServiceCache` 服务提供者的本地注册中心

从调用信息 RpcRequest 中解析出的服务名映射到本地实现类.class `ConcurrentHashMap`

`static addService(String serviceName, Object obj)`  添加服务

`static Object getService(String serviceName)`  获得服务

`static void removeService(Object obj)` 移除服务

### <mark>handler</mark>

`RpcRequestHandler` 执行调用的核心

`Object handleRpcRequest(RpcRequest request)`

从本地注册中心 `LocalServiceCache` 获取服务，通过反射执行调用，返回结果。

### transport

`RpcServer` 接口 定义了 `start(Integer port)` 方法，有三个实现（Http、Netty、Socket）

#### <span id="httpserver">http</span>

`HttpRpcRequestHandler` 处理器，单例模式，责任链模式，相当于回调函数 核心方法 `handler.handle(req, resp)` 将 `RpcRequestHandler` 的结果以及异常封装到 `RpcResponse` 中，发回到序列化流中。

`HttpRpcServer` 启动 tomcat ，主要是设置好 Servlet

`DispatcherServlet` 分发 Servlet，前端控制器模式，使用线程池处理，来一个请求就开一个线程，关于参数：IO密集型 为 cpuNum *2 ，cpu密集型应为 cpuNum + 1 

[http 客户端](#httpclient) 

#### <span id="socketserver">socket</span>

`SocketRpcServer` 启动 ServerSocket bind 端口，在死循环中 accept 还被阻塞，只要服务端socket不关闭（抛出 IOException），就会一直在循环里。除非关闭serversocket/网络问题。建立连接就在线程池开一个任务。

`SocketRpcRequestHandler` 实现了 `Runnable` 接口，将 `RpcRequestHandler` 的结果以及异常封装到 `RpcResponse` 中，发回到序列化流中。

[socket 客户端](#socketclient) 

#### <span id="nettyserver">netty</span> 

[netty 客户端](#nettyclient) 

`NettyRpcRequestHandler` 

> 继承自 `SimpleChannelInboundHandler<RpcMessage>` 表示入站处理器处理RpcMessage。
>
> 只接收你想要的类型（即泛型指定的类型）；
>
> 消息处理完之后，**自动释放 ByteBuf**，避免内存泄漏（比 `ChannelInboundHandlerAdapter` 更安全）。

重写了 `channelRead0` 方法。相当于进入一个消息的处理器。**新开一个任务提交到线程池**：先判断消息类型，如果是心跳请求PING，那么就设置回复的RpcMessage为心跳响应，然后设置好头部信息，消息体为 PONG。剩下的就是客户端的Rpc请求了，像其他方法一样将RpcRequest提取出来交给handler处理，设置好type status body 就发到下一个处理器（codec）。

`ctx.writeAndFlush(responseRpcMessage).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);`

`finally:  ReferenceCountUtil.release(msg);` 防止内存泄露

<mark>心跳检测</mark>：`userEventTriggered` 用户自定义事件，当触发读空闲(`IdleState.READER_IDLE`)时，也就是 30s 没有读的内容，就自动关闭【客户端channel】连接 



`NettyRpcServer`

- `EventLoopGroup` 每个 `EventLoopGroup` 默认使用多个线程（CPU 核数 * 2）

  > Netty 的线程模型是什么？为什么要用 boss/worker 分离？
  >
  > Netty 使用的是 **Reactor 多线程模型**，将连接处理和读写事件分离，避免阻塞。
  >
  > - `bossGroup` 负责处理 **accept（连接建立）** 事件 将接受的连接注册到 Worker Group 的某个线程上，通常1-2个线程即可
  > - `workerGroup` 负责处理 **read/write（数据通信）** 事件，执行实际的业务处理，通常 为 cpu core *2 
  >
  > 这样设计可以保证即使某个连接在业务处理时耗时较长，也不会阻塞新连接的接入，提高并发性能和系统的稳定性。

- `ServerBootstrap` 启动类 配置参数

```java
ServerBootstrap serverBootstrap = new ServerBootstrap();
serverBootstrap.group(boss, worker) // 指定线程组 group(parentGroup, childGroup)
.channel(NioServerSocketChannel.class) // 指定服务端 Channel 类型
.childOption(ChannelOption.TCP_NODELAY, true) // 关闭 Nagle 算法，更及时发送数据
.childOption(ChannelOption.SO_KEEPALIVE, true) // TCP 底层心跳机制（不是业务层心跳，2小时）
.option(ChannelOption.SO_BACKLOG, 128)/* 表示用于临时存放完成三次握手的请求的队列的最大长度,
    								   	若连接建立频繁，服务器处理创建新连接较慢，可适当调大 */
    
.handler(new LoggingHandler(LogLevel.DEBUG)) // ParentGroup 的处理器，记录日志
    
.childHandler(new ChannelInitializer<SocketChannel>() {// 客户端首次请求，为此连接分配一个
    @Override                                          // 新的SocketChannel (lazy init)
    protected void initChannel(SocketChannel ch) throws Exception {
    //  心跳机制，30s内没有收到客户端的请求就关闭连接，会触发一个 IdleState#READER_IDLE 事件
        ch.pipeline().addLast(new IdleStateHandler(30, 0, 0, TimeUnit.SECONDS));
        ch.pipeline().addLast(new RpcFrameDecoder());
        ch.pipeline().addLast(new SharableRpcMessageCodec());
        ch.pipeline().addLast(new NettyRpcRequestHandler());
    }
});
// 绑定端口，同步等待绑定成功 生命周期 bind 
ChannelFuture channelFuture = serverBootstrap.bind(inetAddress, port).sync();
log.debug("Rpc server add {} started on the port {}.", inetAddress, port);
// 等待服务端监听端口关闭，如果不写这句，主线程会直接退出，Netty 服务就被销毁了。
channelFuture.channel().closeFuture().sync();
// 这里 netty 已经封装好了异步接受逻辑，bind通常用sync等待一次，然后监听channel的关闭即可。
```

> `EventLoop` 事件处理单元 `ChannelHandler`  业务处理逻辑 `ChannelPipeline`  处理器链
>
> `ByteBuf`  高效字节容器
>
> `Channel`  网络连接通道
>
> `NioServerSocketChannel` 用于 boss 的Channel，`SocketChannel` 用于 worker 

Netty 的 `ChannelPipeline` 是一个基于责任链模式的事件处理链，每个请求或响应都会从一个 handler 流向下一个 handler，形成链式处理。

`[字节流] -> ([心跳检测]) -> [帧解码器] -> [消息解码器] -> [业务处理器]` 

<span id="心跳机制"></span>

| Handler 名称              | 作用说明                                                     | 方向 |
| ------------------------- | ------------------------------------------------------------ | ---- |
| `IdleStateHandler`        | 空闲连接检测（30 秒无读事件会触发 READER_IDLE）实现应用层心跳机制（比 TCP 更灵活）更适合业务层快速检测连接状态 | 入站 |
| `RpcFrameDecoder`         | 拆包/粘包处理，按协议规则拆出完整消息帧                      | 入站 |
| `SharableRpcMessageCodec` | 编解码器，将 ByteBuf ↔ 自定义消息（RpcMessage）相互转换      | 双向 |
| `NettyRpcRequestHandler`  | 业务逻辑处理器，处理请求、调用本地服务、写回响应             | 入站 |

![责任链](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250423123418951.png)

[netty 客户端](#nettyclient)



## client-spring-boot

### common

`RequestMetadata` 封装请求，有 RpcMessage serverAddr port timeout 字段。

### handler

`RpcResponseHandler` 继承自 `SimpleChannelInboundHandler<RpcMessage>` 

> **Netty 中的 Promise 机制** 
>
> RPC 是远程调用，发送请求和得到响应的过程属于异步操作，Promise 机制用来解决 “客户端怎么知道哪个响应对应哪个请求？怎么挂起等待，又怎么唤醒？”的问题。
>
> Netty 提供的 `DefaultPromise<V>` 是一个线程安全的对象，用来异步地接收并处理计算结果。你可以把它理解为一个 **可手动完成的 Future**。它有几个特点：
>
> - 在请求发送后立刻返回，不阻塞
> - 把结果放入 Promise 中，调用方通过 `promise.get()` 或回调拿到结果
> - 由响应处理器来手动调用 `promise.setSuccess(response)` 来填充结果
>
> 实践中通常会用一个 `Map<requestId, Promise<RpcResponse>>` 保存请求对应的 Promise。具体流程如下：
>
> 1. **发送请求之前：**
>    - 创建一个 `DefaultPromise<RpcResponse>` 对象
>    - 存入全局请求池（Map）中：`promises.put(requestId, promise)`
> 2. **接收响应时：**
>    - 从请求池里通过 `requestId` 找到对应的 `Promise`
>    - 执行 `promise.setSuccess(response)`，通知结果已经到了
> 3. **调用方阻塞等待结果（可选）：**
>    - `promise.get()` 或 `promise.await()`
>
> ```java
> // 1. 发请求之前保存 promise
> DefaultPromise<RpcResponse> promise = new DefaultPromise<>(eventExecutor);
> RpcResponsePool.put(requestId, promise);
> channel.writeAndFlush(rpcMessage);
> 
> // 2. 接收响应时设置结果
> public void channelRead0(ChannelHandlerContext ctx, RpcMessage msg) {
>     RpcResponse response = (RpcResponse) msg.getBody();
>     Promise<RpcResponse> promise = RpcResponsePool.remove(response.getRequestId()); // 防内存泄露
>     if (promise != null) {
>         promise.setSuccess(response);
>     }
> }
> 
> // 3. 获取结果
> RpcResponse result = promise.get();
> ```

`Map<Integer, Promise<RpcMessage>> UNPROCESSED_RPC_RESPONSES` 未处理的响应请求，跟 sequenceId 相对应

`channelRead0` 如果是 `RpcResponse`，从map中拿出对应的响应，如果响应中没有异常，代表 `Promise` 成功。如果是心跳回应，则记录日志。

`userEventTriggered(ctx, Object evt)` 用户自定义事件处理器，处理写空闲（`WRITE_IDLE`），当检测到写空闲15s 以后自动发送一个心跳检测数据包。如果 `evt` 是 `IdleStateEvent`，那么就构建一个 心跳检查的 RpcMessage，这里使用 Kryo 序列化以达到最好的效果。

### transport

`RpcClient` 

`RpcMessage sendRpcRequest(RequestMetadata requestMetadata)`  

#### <span id="httpclient">http</span>

`sendRpcRequest(RequestMetadata requestMetadata)` 

根据 RequestMetadata，拿出 RpcMessage，取出 RpcRequest 并通过 Http 连接序列化发送，阻塞等待读取 RpcResponse，读取之后封装为 RpcMessage返回。

`HttpRpcClient 发送和接受的数据为：RpcRequest，Response` 

[http 服务端](#httpserver) 

#### <span id="socketclient">socket</span>

`sendRpcRequest(RequestMetadata requestMetadata)`   socket connect 获取数据

`SocketRpcClient 发送和接受的数据为：RpcRequest, RpcResponse`

[socket 服务端](#socketserver) 

#### <span id="nettyclient">netty</span>

| 场景                     | 你的职责                                                     |
| ------------------------ | ------------------------------------------------------------ |
| Channel 断开             | 触发 `channelInactive()` 或 `exceptionCaught()` 时，尝试**重连** |
| 连接池中拿到无效 Channel | 检查 `channel.isActive()`，如果无效：**close 并重新连接**    |
| 心跳失败或无响应         | 在 `IdleStateHandler` 的事件中**关闭旧连接并重建**           |
| 重试次数限制             | 设置最大重试次数，避免死循环                                 |

> `ChannelProvider` 获取Channel的工具类
>
> - `Map<String, Channel> channels` 存储 channel 对象，key 为 ip:port
> - `get(host,port)` 取出 活跃的 channel，不活跃的/null 从map移除
> - `get(inetSocketAddress)` 同上
> - `set(host,port,channel)` 将 host:port 与 Channel 映射

`NettyRpcClient` 

`BootStrap` 启动类，注册 NioSocketChannel 超时时间，5s连接超时。handler 设置为 15s写空闲的IdleStateHandler，粘包拆包解码器、codec、ResponseHandler 。提供 ChannelProvider 工具类

`EventLoopGroup` 每个事件循环对象对应一个线程，维护一个 Selector，用来处理io事件

`getChannel(inetSocketAddress)`首先从map中获取，获取不到就主动去连接 `doConnect`

`doConnect(inetSocketAddress)` ：

方式1：

随后sync()同步阻塞等待 channel 的 closeFuture()。

```java
// connect 会返回一个 异步的 ChannelFuture
// sync() 同步等待异步connect连接成功 
Channel channel = bootstrap.connect(inetSocketAddress).sync().channel();
// 同步阻塞等待异步关闭完成
channel.closeFuture().sync();
```

方式2：

<span id="指数退避">指数退避重连</span>，使用 CompletetableFuture 阻塞获取 channel，future 成功时返回值。

```java
@SneakyThrows
public Channel doConnectWithRetry(InetSocketAddress inetSocketAddress, int remainingRetries){
    CompletableFuture<Channel> cf = new CompletableFuture<>();
    bootstrap.connect(inetSocketAddress).addListener((ChannelFutureListener)future -> {
        if (!future.isSuccess()) {
            if (remainingRetries > 0) {
                long delay = calculateRetryDelay(remainingRetries);
                log.info("Retrying connection to {} in {} ms", inetSocketAddress, delay);
                eventLoopGroup.schedule(() -> doConnectWithRetry(inetSocketAddress, remainingRetries - 1), delay, TimeUnit.MILLISECONDS); 
            } else {
                log.error("Failed to connect to {} after {} retries", inetSocketAddress, MAX_RETRIES);
            }
        }
        else{
            log.info("Connected to {} successfully", inetSocketAddress);
            cf.complete(future.channel());
        }
    });
    Channel channel = cf.get();
    channel.closeFuture().addListener(future -> {
        log.info("The client has been disconnected from server [{}].", inetSocketAddress.toString());
    });
    return channel;
}
private long calculateRetryDelay(int remainingRetries) {
    int attempt = MAX_RETRIES - remainingRetries + 1;
    return (long) (Math.pow(2, attempt) * 500); // 指数退避基础500ms
}
```

之后是

`sendRpcRequest()` ：

根据requestMetadata获取channel对象`getChannel()`。

`promise = DefaultPromise<>(channel.eventLoop());`

获取sequenceId，将promise存入map，把rpcMessage写入channel（异步的）addListener。

对于timeout，使用 promise 的 await 方法。isSuccess 则调用 getNow() 返回响应结果 RpcMessage。

[netty 服务端](#nettyserver) 

### <span id="proxy"><mark>proxy</mark></span>

代理对象生成过程：

1. 首先需要一个得到代理对象的工厂类，里面有一个工厂方法返回的就是代理对象。

2. 如何创建代理对象：

   JDK：`Proxy.newInstance(目标接口类加载器, 目标接口的数组, 实现InvocationHandler接口的invoke方法)`

   CGLIB: `Enhancer.create(目标类, 实现MethodInterceptor接口的intercept方法)`

3. 记住 `invoke` 和 `intercept` 就是拿着 `method`和 `args`为所欲为，当然也可以在实现类自己加参数，工厂类自己也要加参数

4. 如果你想得到代理对象，那么就调用工厂类的工厂方法。

RPC 框架怎么做到像调用本地接口一样调用远端服务呢？这必须依赖动态代理来实现。需要创建一个代理对象，在代理对象中完成数据报文编码，然后发起调用发送数据给服务提供方，以此屏蔽 RPC 框架的调用细节。因为代理类是在运行时生成的，所以代理类的生成速度、生成的字节码大小都会影响 RPC 框架整体的性能和资源消耗，所以需要慎重选择动态代理的实现方案。动态代理比较主流的实现方案有以下几种：JDK 动态代理、Cglib、Javassist、ASM、Byte Buddy，我们简单做一个对比和介绍。

- JDK 动态代理。在运行时可以动态创建代理类，但是 JDK 动态代理的功能比较局限，代理对象必须实现一个接口，否则抛出异常。因为代理类会继承 Proxy 类，然而 Java 是不支持多重继承的，只能通过接口实现多态。JDK 动态代理所生成的代理类是接口的实现类，不能代理接口中不存在的方法。JDK 动态代理是通过反射调用的形式代理类中的方法，比直接调用肯定是性能要慢的。
- Cglib 动态代理。Cglib 是基于 ASM 字节码生成框架实现的，通过字节码技术生成的代理类，所以代理类的类型是不受限制的。而且 Cglib 生成的代理类是继承于被代理类，所以可以提供更加灵活的功能。在代理方法方面，Cglib 是有优势的，它采用了 FastClass 机制，为代理类和被代理类各自创建一个 Class，这个 Class 会为代理类和被代理类的方法分配 index 索引，FastClass 就可以通过 index 直接定位要调用的方法，并直接调用，这是一种空间换时间的优化思路。
- Javassist 和 ASM。二者都是 Java 字节码操作框架，使用起来难度较大，需要开发者对 Class 文件结构以及 JVM 都有所了解，但是它们都比反射的性能要高。Byte Buddy 也是一个字节码生成和操作的类库，Byte Buddy 功能强大，相比于 Javassist 和 ASM，Byte Buddy 提供了更加便捷的 API，用于创建和修改 Java 类，无须理解字节码的格式，而且 Byte Buddy 更加轻量，性能更好。

#### 代理实例工厂

建立一个本地的代理对象缓存，不用每次都重新new代理对象。

```java
@AllArgsConstructor
public class ClientStubProxyFactory {
    private final ServiceDiscovery discovery;
    private final RpcClient rpcClient;
    private final RpcClientProperties properties;
    // 本地代理对象缓存
    private static final Map<String, Object> proxyMap = new ConcurrentHashMap<>();
    
    public <T> T getProxy(Class<T> clazz, String version) {
        return (T) proxyMap.computeIfAbsent(ServiceUtil.serviceKey(clazz.getName(), version), serviceName -> {
            // 如果目标类是一个接口或者 是 java.lang.reflect.Proxy 的子类 则默认使用 JDK 动态代理
            if (clazz.isInterface() || Proxy.isProxyClass(clazz)) {
                return Proxy.newProxyInstance(clazz.getClassLoader(),
                        new Class[]{clazz}, // 注意，这里的接口是 clazz 本身（即，要代理的实现类所实现的接口）
                        new ClientStubInvocationHandler(discovery, rpcClient, properties, serviceName));
            } else { // 使用 CGLIB 动态代理
                return Enhancer.create(clazz, 
                        new ClientStubMethodInterceptor(discovery, rpcClient, properties, serviceName));
            }
        });
    }
}
```

#### cglib

重写 MethodInterceptor 中的 intercept() 方法

```java
@AllArgsConstructor
public class ClientStubMethodInterceptor implements MethodInterceptor {
    private final ServiceDiscovery serviceDiscovery;
    private final RpcClient rpcClient;
    private final RpcClientProperties properties;
    private final String serviceName;
    
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        return RemoteMethodCall.remoteCall(serviceDiscovery, rpcClient, serviceName, properties, method, args);
    }
}
```

#### jdk

重写 InvocationHandler 中的 invoke() 方法

```java
@AllArgsConstructor
public class ClientStubInvocationHandler implements InvocationHandler {
    private final ServiceDiscovery serviceDiscovery;
    private final RpcClient rpcClient;
    private final RpcClientProperties properties;
    private final String serviceName;
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return RemoteMethodCall.remoteCall(serviceDiscovery, rpcClient, serviceName, properties, method, args);
    }
}
```

#### RemoteMethodCall 发起 RPC 的公共方法

```java
public static Object remoteCall(ServiceDiscovery serviceDiscovery, RpcClient rpcClient, String serviceName, 	
                                RpcClientProperties properties, Method method, Object[] args){
    MessageHeader header = new MessageHeader().build(properties.getSerialization());
    // 构建 RpcRequest: serviceName, methodName, parameterTypes, args
    // 构建 RpcMessage
    // 服务发现：得到负载均衡后返回的服务信息 serviceInfo
    // 根据 serviceInfo, properties, 与 rpcMessage 构建 RequestMetadata
    // client 调用 sendRpcRequest(rpcMetadata) 返回的 rpcMessage
    // rpcMessage 解包出 rpcResponse 提取 value 返回
}
```

# Spring 部分

服务端的Spring：

<mark>需要按照是否有@RpcService注解进行 Bean 的注册。谁能注册？谁来执行注册？（手动实现Scanner以BeanDefinitionRegistrar），然后把被注解的bean加入本地缓存map，准备执行 RequestHandler</mark> 

客户端的Spring：

<mark>所有的 bean 都已经注册并导入好了，唯一的问题是 RestController 里面，出现 HelloService 字段，需要将其进行替换为代理对象！</mark> 

## Review: Spring Bean 生命周期一览

### <mark>IoC 容器启动过程</mark> 

1. Spring IoC 启动入口：`AnnotationConfigApplicationContext` 或 Spring Boot 的 `SpringApplication.run(...)` 

2. 加载配置类（带 `@Configuration`、`@ComponentScan`、`@Import` 等注解）

3.  **扫描、注册阶段** （由 `ClassPathBeanDefinitionScanner` 完成）

   - 扫描被 `@ComponentScan` 指定的包
   - 找到带注解的类（如 `@Component`、`@Service`、`@Controller`）
   - 解析为 `BeanDefinition`，使用 `BeanDefinitionRegistry` 将其注册进`beanDefinitionMap`还未创建对象。
     - 修改Bean定义：执行所有 `BeanFactoryPostProcessor` 的实现类（如 `PropertySourcesPlaceholderConfigurer`），允许对 `BeanDefinition` 进行修改（例如替换占位符）。
     - **提前实例化处理器**：注册 `BeanPostProcessor` 实现类（如 `AutowiredAnnotationBeanPostProcessor`），这些处理器需在普通Bean之前初始化，以便后续处理其他Bean的创建。
     - 初始化消息源以及事件广播器

4. **实例化阶段**（容器对于单例且非懒加载的 Bean）

   对每个要使用的 Bean：

   - （主要是针对懒加载或者`Scope = prototype`）存在性检查：Scope判断（若为单例则检查到单例缓存）以及循环依赖判断（如果当前正在创建就从单例缓存获取原始对象）

   - **实例化**：从 `BeanDefinitionRegistry` 获取` BeanDefinition`，包含类名、作用域、初始化方法等元数据。检查是否存在未满足的依赖（如通过`@DependsOn`指定的前置依赖，或者`@Order`加载顺序）最后**实例化**对象（通过反射或者工厂方法调用`Constructor`创建原始对象）
   - **依赖注入**： `@Autowired/@Resource` 递归调用`getBean()`获取依赖bean，通过三级缓存（`singletonFactories`、`earlySingletonObjects`、`singletonObjects`）提前暴露对象引用，解决setter的循环依赖。设置好属性。

   - Aware 接口回调：如果实现了 `XXXAware` 接口，则通过 `setXXX` 注入容器底层信息。如名称，类加载器等

   - `BeanPostProcessor`: 每个bean在构建的过程中，Spring都会遍历所有的`BeanPostProcessor`的实现类，调用实现类中的方法，入参为构建好的bean。要实现无感的对bean的处理必须使用 `BeanPostProcessor`。

     | 方法（按照先后顺序）                          | 方法所属                     |
     | --------------------------------------------- | ---------------------------- |
     | 1. `Object postProcessBeforeInitialization()` | `BeanPostProcessor`          |
     | 2. `@PostConstruct` 标注的方法                | JSR-250 规定                 |
     | 3. `void afterPropertiesSet()`                | `InitializingBean`           |
     | 4. `init()`                                   | `@Bean (initMethod =  init)` |
     | 5. `Object postProcessAfterInitialization()`  | `BeanPostProcessor`          |

      **5 是 AOP 动态代理的关键阶段**：Spring 在这里可能会返回代理对象替代原对象

5. **完成容器启动**：触发 `ContextRefreshedEvent`，通知监听器容器已就绪。**此时可以通过 `getBean()` 获取单例 Bean**，如果是懒加载或者`Scope = prototype`的则会在主动调用 `getBean()` 的时候才实例化。

### Bean 生命周期

1. Bean 实例化(仅为构造出对象)

   - **触发条件**：①容器启动 ②首次请求 Bean 时 `getBean() 或者依赖注入`。
   - **方式**：通过构造函数或工厂方法创建 Bean 的实例。
   - **异常**：若依赖无法解析或构造函数抛出异常，Bean 创建失败。

2. 属性赋值 Populate Properties

   - **依赖注入**：通过 `@Autowired`、`@Resource`、XML 配置等方式注入属性。
   - 处理 `@Value`：解析并注入 SpEL 表达式或占位符的值。

3. Aware 接口回调 与 `BeanPostProcessor `前置处理、初始化、后置处理。

4. 就绪状态

   - Bean 完全初始化，可被应用程序使用。
   - **Singleton Bean** 会被缓存，后续请求直接获取。
   - **Prototype Bean** 每次请求创建新实例（无后续销毁步骤）。

5. Bean 对象销毁回调

   - `@PreDestroy` JSR-250

   - `destroy()->`  DisposableBean

   - `close()` @Bean (destroyMethod =  close)

6. Bean 对象销毁

   - **触发条件**：容器关闭时（如 `close()` 方法调用）。
   - **作用域影响**：仅 Singleton Bean 会执行销毁回调，Prototype Bean 需手动清理。

### SpringBoot 启动

```java
SpringApplication.run()
    ├── 创建 SpringApplication
    ├── prepareEnvironment
    ├── createApplicationContext // 创建应用上下文
    ├── refresh() // AbstractApplicationContext#refresh() 方法 启动 IOC 容器
        ├── postProcessBeanFactory(beanFactory)
        ├── prepareBeanFactory(beanFactory)
        ├── invokeBeanFactoryPostProcessors(beanFactory)
        ├── registerBeanPostProcessors, initMessageSource/initEventMulticaster 
        ├── onRefresh (Web容器启动)
        ├── registerListeners
        ├── finishBeanFactoryInitialization // 初始化所有非懒加载单例 Bean 
        └── finishRefresh (发布ContextRefreshedEvent)
    ├── 调用 CommandLineRunner.run(String... args) ApplicationRunner(String... args)
    └── ApplicationReady
```

[由此可见 CommandLineRunner 是在容器启动完成以后执行的。可以实现这个接口的 run 方法来注入参数。](#boot)

## <span id="annotation">Spring 扫描自定义注解</span>

### `@RpcComponentScan` ：扫描、注册`@RpcService`的类，实例化时将其加入本地缓存

扫描、注册的具体过程：

1. 扫描到 `@Configuration` 配置类，又扫到了 `@Import(RpcBeanDefinitionRegistrar.class)` 
2. 通过 `RpcBeanDefinitionRegistrar` 的 `registerBeanDefinitions(AnnotationMetadata data, BeanDefinitionRegistry registry)` 进行动态注册，其中 `data` 参数为被@Import注解 的类（这里是注解`@RpcComponentScan`）解析出要扫描的包路径，`registry` 为注册bean的核心。在里面 new 一个 `RpcClassPathBeanDefinitionScanner(registry, RpcService.class)` 在其构造函数里添加针对 RpcService.class 的 TypeFilter。
3. 设置好 scanner 的`resourceloader`，增强代码健壮性。调用 scanner 的 `scan(basePackages)` 方法，此方法会在内部调用 scanner 的 `registry` 进行 bean 的注册。

#### 定义 BeanDefinition 扫描器

> `ClassPathBeanDefinitionScanner` 是 Spring 框架中的一个类，用于从指定的类路径中扫描符合条件的类，并将其注册为 Spring 容器中的 Bean 定义。
>
> 主要功能：
>
> - 扫描类路径：根据指定的包路径，扫描类路径下的所有类。
> - 过滤条件：通过过滤器（如注解过滤器、类型过滤器等）筛选出符合条件的类。
> - 注册 Bean 定义：将符合条件的类的元信息（`BeanDefinition`）注册到 Spring 容器中。核心：`BeanDefinitionRegistry`
>
> 核心用途：
>
> - 用于实现自定义注解扫描和动态注册 Bean。
> - 常见于 Spring 的扩展机制中，例如自定义注解的扫描器。
>
> 工作流程：
>
> 1. 指定需要扫描的包路径`basePackages`。
>
> 2. 配置过滤器（如只扫描带有特定注解的类）。 `addIncludeFilter(TypeFilter tf)` 
>
> 3. > TypeFilter 接口需要实现`boolean match(MetadataReader reader, MetadataReaderFactory factory)` 
>
> 4. 调用 `scan(String... basePackages)` 方法，扫描并注册符合条件的类。

`RpcClassPathBeanDefinitionScanner` 继承了 `ClassPathBeanDefinitionScanner` 

构造函数调用 `this.addIncludeFilter(new AnnotationTypeFilter(this.annotationType));` 注册想要扫描的注解类。通过 父类的 scan 方法扫包

---

#### 使用 @Import 注解结合 Registrar 动态注册 BeanDefinition

> `@Import`注解
>
> - Allows for importing `@Configuration` classes, [`ImportSelector`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ImportSelector.html) and [`ImportBeanDefinitionRegistrar`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ImportBeanDefinitionRegistrar.html) implementations, as well as regular **component** classes
>
> **将一个类或多个类注入到 Spring 的 IOC 容器中**，等价于在配置类中手动用 `@Bean` 或 `@ComponentScan` 注册。
>
> <img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250423120651325.png" alt="三种等价的导入方式" style="zoom: 35%;" />
>
> `ImportBeanDefinitionRegistrar`   它通常与 `@Import` 注解一起使用，允许开发者在 Spring 容器启动时通过编程方式向容器中动态注册 Bean。
>
> - 动态注册 Bean：通过实现 `ImportBeanDefinitionRegistrar`接口，可以在运行时根据需要向 Spring 容器中注册 Bean。
> - 扩展 Spring 配置：允许开发者在 Spring 的配置阶段插入自定义逻辑，动态调整 Bean 的定义和注册。
>
> `ImportSelector` 例如 `AutoConfigurationImportSelector` 实现自动装配的核心
>
> ```java
> @Target(ElementType.TYPE)
> @Retention(RetentionPolicy.RUNTIME)
> @Import(MyRegistrar.class)
> public @interface EnableMyFeature {}
> ---------------------------------------------------------
> public class MyRegistrar implements ImportBeanDefinitionRegistrar {
>     @Override
>     public void registerBeanDefinitions(
>             AnnotationMetadata importingClassMetadata, // 被@Import注解的类，这里是 EnableMyFeature 接口的信息（关键）
>             BeanDefinitionRegistry registry  // 用于注册 Bean 定义的注册表，开发者可以通过它向容器中动态添加 Bean
>     ) {
> 
>         RootBeanDefinition beanDefinition = new RootBeanDefinition(MyService.class);
>         registry.registerBeanDefinition("myService", beanDefinition);
>     }
> }
> ---------------------------------------------------------
> @Configuration
> @EnableMyFeature 
> public class AppConfig{}
> 
> ```
>
> 如上，相当于在底层手动注册了一个类到IoC容器中。
>
> Spring 的 `ConfigurationClassPostProcessor` 在处理 `@Import` 时，**会检查被导入的类是否实现了 `Aware` 接口（如 `EnvironmentAware`、`ResourceLoaderAware`、`BeanFactoryAware` 等）**，并会调用对应的方法注入你需要的对象（resourceLoader environment beanFactoryAware）等。为了以防万一，让自定义的Registerar实现ResourceLoaderAware，重写其setLoader，设置好scanner以后再扫描
>

自定义 Rpc 组件扫描注解
`RpcComponentScan` 注解用 `@Import` 引入了 `RpcBeanDefinitionRegistrar` 类，而这个类是一个 `ImportBeanDefinitionRegistrar` 的实现类， Spring 容器在解析该类型的 Bean 时会调用其 `importBeanDefRegistrar.registerBeanDefinitions(AnnotationMetadata, BeanDefinitionRegistry)` 方法， 将 `@RpcComponentScan` 注解上的信息提取成 `AnnotationMetadata` 以及容器注册器对象作为此方法的参数，这个就是自定义注解式组件扫描的关键逻辑。

metadata.getAnnotationAttributes(RpcComponent.class.getName()) 返回 RpcComponent 注解的属性和值(basePackages)，是一个map。从attributes中提取basePackages的值。未指定则扫描被注解类所在的包。随后构建一个扫描器，添加针对自定义注解的过滤器，开始扫描（注册）

---

#### Bean 处理器

@RpcService 注解

`RpcServerBeanPostProcessor` 实现了 BeanPostProcessor

有服务注册、RpcServer、以及RpcServerProperties 3个字段。

`public Object postProcessAfterInitialization(Object bean, String beanName)` 重写此方法配置好服务 Bean： 如果当前的bean被RpcService注解标注（也就是服务类的实现类），获取其注解的值，得到其暴露的接口的对象，由此得到接口名字，和版本拼接形成服务名，再根据properties配置构建好 ServiceInfo，**然后进行服务注册，在本地缓存map加入服务bean**.

还实现了 CommandLineRunner：新开一个线程启动服务器，增加关闭勾子强制清除注册中心的连接。

```java
@Override
public void run(String... args) throws Exception {
    new Thread(() -> rpcServer.start(properties.getPort())).start();
    Runtime.getRuntime().addShutdownHook(new Thread(() -> {
        try {
            // 当服务关闭之后，将服务从 注册中心 上清除（关闭连接）
            serviceRegistry.destroy();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }));
}
```

### `@RpcReference`：利用反射把 Bean 的字段替换成代理对象

定义了**接口类型**、版本号、**负载均衡**、**超时时间**、mock，可以在字段、方法、注解上面使用

自动配置：除去负载均衡、服务发现、客户端类型，还有**代理工厂**、RpcClientBeanPostProcessor、RpcClientExitDisposableBean（这些都是需要注入 bean 的参数的）

客户端这边唯一的问题是 RestController 里面，出现 HelloService 字段，需要将其进行替换为代理对象。

如果字段被 @RpcReference 注解，那么就从注解中提取相关信息（接口全限定名+版本号），交给代理工厂生产代理对象。

```java
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    Field[] fields = bean.getClass().getDeclaredFields();
    // 遍历所有属性
    for (Field field : fields) {
        // 判断是否被 RpcReference 注解标注
        if (field.isAnnotationPresent(RpcReference.class)) {
            //...
            field.setAccessible(true);// 关闭安全检查 因为是 private
            field.set(bean, proxy); // 设置域的值为代理对象
        }
    }
    return bean;
}
```



## <span id="autoconfig">SpringBoot 自动装配：自动装配框架内部的 Bean</span>

自动装配基于自动装配类，所以要把所有的bean都用bean方法的形式注册到容器中。

包括之前讲的 服务发现、服务注册、代理工厂、BeanPostProcessor、ConfigurationProperties等。

Bean 方法不需要使用Autowired在参数上注解！！！！

### 介绍

[SpringBoot 自动装配原理详解](https://javaguide.cn/system-design/framework/spring/spring-boot-auto-assembly-principles.html) 

自动装配可以简单理解为：**通过注解或者一些简单的配置就能在 Spring Boot 的帮助下实现某块功能。**

- 没有 Spring Boot 的时候，我们写一个 RestFul Web 服务，还首先需要自己写 Configuration 配置类，写 Bean 方法。
- 但有了 SpringBoot，只需要引入依赖，启动 SpringBootApplication 即可。

> SpringBoot 定义了一套接口规范，这套规范规定：SpringBoot 在启动时会扫描外部引用 jar 包中的`META-INF/spring.factories`文件，将文件中配置的类型信息加载到 Spring 容器（此处涉及到 JVM 类加载机制与 Spring 的容器知识），并执行类中定义的各种操作。对于外部 jar 来说，只需要按照 SpringBoot 定义的标准，就能将自己的功能装置进 SpringBoot。
>
> ```properties
> # SpringBoot 2.x  在 META-INF/spring.factories 
> org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
> com.example.rpc.RpcServerAutoConfiguration
> 
> # SpringBoot 3 在 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
> com.example.rpc.RpcServerAutoConfiguration
> ```
>
> 没有 Spring Boot 的情况下，如果我们需要引入第三方依赖，需要手动配置，非常麻烦。但是，Spring Boot 中，我们直接引入一个 starter 即可。比如你想要在项目中使用 redis 的话，直接在项目中引入对应的 starter 即可。
>
> ```xml
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-starter-data-redis</artifactId>
> </dependency>
> ```
>
> 引入 starter 之后，我们通过少量注解和一些简单的配置就能使用第三方组件提供的功能了。

### 原理浅析

机制核心 @EnableAutoConfiguration (@SpringBootApplication 的一部分) 

底层通过 @Import(AutoConfigurationImportSelector.class) 加载所有自动配置类（通过 spring.factories 找到） 这个ImportSelector很重要，通过 selectImport

方法扫描获取所有符合条件的类的全限定类名，将这些类注册到 IoC 容器。核心调用路径如下：

`selectImport->getAutoConfigurationEntry->getCandidateConfigurations->SpringFactoriesLoader.loadFactoryNames->loadSpringFactories` 不光是这个依赖下的`META-INF/spring.factories`被读取到，所有 Spring Boot Starter 下的`META-INF/spring.factories`都会被读取到。后边会根据条件进行逐层筛选。

### 示例 创建 starter

> 引入 starter-validation
>
> @Validated注解加到类上，下面这些注解可以用到 字段、参数
>
> @NotBlank @Email  @Min(1)  @Max(91)
>
> @Pattern(regexp = "^[a-zA-Z0-9]{8,16}$",message = "用户名只能是长度在8至16"      + "之间的包含数字和大小写字母的字符串")

```java
@Validated
@Data
@ConfigurationProperties(prefix = "rpc.server")
public class RpcServerProperties {
    private String address;private Integer port;private String appName;
    @Pattern(regexp = "zookeeper|nacos", message = "必须是 nacos或者zookeeper")
    private String registry;
    private String transport;private String registryAddr;
    public RpcServerProperties() throws UnknownHostException {
        this.address = InetAddress.getLocalHost().getHostAddress();
        this.port = 8080;
        this.appName = "provider-1";
        this.registry = "zookeeper";
        this.transport = "netty";
        this.registryAddr = "127.0.0.1:2181";
    }
}
```

pojo 类 + @ConfigurationProperties注解，可在 application.yml 中按照前缀配置属性。

```properties
rpc.server.app-name=provider-1
rpc.server.port=9991
rpc.server.registry=zookeeper
rpc.server.registry-addr=39.108.66.202:2181
rpc.server.transport=netty
# 设置指定包下的日志显示级别 INFO/DEBUG/WARNING/OFF
logging.level.com.wxy.rpc=info
```

| 注解                        | 作用                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `@ConditionalOnProperty`    | 属性Property，满足一定的条件才生效                           |
| `@ConditionalOnMissingBean` | 只有没有这个类型的 Bean 时才生效 (用户自定义实现了Bean方法，可以替换这个自动装配的) |
| `@ConditionalOnClass`       | 类路径下有某个类才生效                                       |
| `@ConditionalOnBean`        | 依赖的 Bean 存在才生效                                       |
| `@Primary`                  | 多个同类的 Bean 存在时首选注入                               |

```java
@Configuration
@EnableConfigurationProperties(RpcServerProperties.class) // 绑定 pojo 作为 properties
public class RpcServerAutoConfiguration {
    
    @Autowired
    RpcServerProperties properties;

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(prefix = "rpc.server", name = "registry", havingValue = "zookeeper", matchIfMissing = true)// property 的 registry 字段的 value = zookeeper 才生效，如果配置项不存在也会生效
    public ServiceRegistry serviceRegistry() {
        if(properties.get)
        
        return new ZookeeperServiceRegistry(properties.getRegistryAddr());
    }
    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(prefix = "rpc.server", name = "registry", havingValue = "nacos")
    public ServiceRegistry nacosServiceRegistry() {
        return new NacosServiceRegistry(properties.getRegistryAddr());
    }

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnBean({ServiceRegistry.class, RpcServer.class})
    public RpcServerBeanPostProcessor rpcServerBeanPostProcessor(
        @Autowired ServiceRegistry serviceRegistry,
		@Autowired RpcServer rpcServer,
        @Autowired RpcServerProperties properties) 
    {
        return new RpcServerBeanPostProcessor(serviceRegistry, rpcServer, properties);
    }
}
```

例子：添加 `spring-boot-starter-data-redis` 后，可直接注入 `RedisTemplate`。

| 特征         | Starter 模块                                                 |
| :----------- | :----------------------------------------------------------- |
| **命名**     | 以 `-spring-boot-starter` 结尾                               |
| **依赖**     | 包含 `spring-boot-autoconfigure`                             |
| **自动配置** | 有 `@AutoConfiguration` 类，并注册到 `spring.factories` 或 `AutoConfiguration.imports` |
| **配置属性** | 包含 `@ConfigurationProperties` 类                           |
| **功能入口** | 提供开箱即用的 Bean，无需用户手动配置。可通过 `application.properties` 或 `@Bean` 覆盖 Starter 的默认配置。 |

1. 实现自动配置类 AutoConfiguration 

2. 按照 SpringBoot 版本将配置类的全限定名引入指定路径下。

3. 新建 starter 模块，添加依赖

   ```xml
   <dependencies>
       <!-- 必须依赖 -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-autoconfigure</artifactId>
           <version>${spring-boot.version}</version>
       </dependency>
       <!-- 可选：配置注解处理器 -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-configuration-processor</artifactId>
           <optional>true</optional>
       </dependency>
       <!-- 你的模块核心实现 -->
       <dependency>
           <groupId>com.example</groupId>
           <artifactId>rpc-server-spring-boot</artifactId>
           <version>1.0.0</version>
       </dependency>
   </dependencies>
   ```




# 项目使用

## 环境搭建

- 操作系统：Windows + Linux
- 集成开发工具：IntelliJ IDEA
- 项目技术栈：SpringBoot 2.5.2 + JDK 1.8 + Netty 4.1.65.Final
- 项目依赖管理工具：Maven 4.0.0
- 注册中心：Zookeeeper 3.7.1

## 项目测试

- 启动 Zookeeper 服务器：进入到zk的bin目录，输入命令 `./zkServer.sh`
- 启动 provider 模块 ProviderApplication
- 启动 consumer 模块 ConsumerApplication
- 测试：浏览器输入 http://localhost:8080/hello/zhangsan ，成功返回：`hello, zhangsan`，rpc 调用成功。
- 调用接口 100 次耗时 26ms，调用 10_0000 次耗时 25164 ms。

## 压力测试

**[JMH](https://zhuanlan.zhihu.com/p/434083702)**

`JMH`即`Java Microbenchmark Harness`，是`Java`用来做基准测试的一个工具，该工具由`OpenJDK`提供并维护，测试结果可信度高。

相对于 Jmeter、ab ，它通过编写代码的方式进行压测，在特定场景下会更能评估某项性能。

本次通过使用 JMH 来压测 RPC 的性能（官方也是使用JMH压测）

启动 10000 个线程同时访问 sayHello 接口，总共进行 3 轮测试，测试结果如下：

```
Benchmark                                          Mode     Cnt      Score       Error  Units
BenchmarkTest.testSayHello                        thrpt       3  29288.573 ± 20780.318  ops/s
BenchmarkTest.testSayHello                         avgt       3      0.532 ±     6.159   s/op
BenchmarkTest.testSayHello                       sample  395972      0.382 ±     0.002   s/op
BenchmarkTest.testSayHello:testSayHello·p0.00    sample              0.003               s/op
BenchmarkTest.testSayHello:testSayHello·p0.50    sample              0.318               s/op
BenchmarkTest.testSayHello:testSayHello·p0.90    sample              0.387               s/op
BenchmarkTest.testSayHello:testSayHello·p0.95    sample              0.840               s/op
BenchmarkTest.testSayHello:testSayHello·p0.99    sample              2.282               s/op
BenchmarkTest.testSayHello:testSayHello·p0.999   sample              2.470               s/op
BenchmarkTest.testSayHello:testSayHello·p0.9999  sample              2.496               s/op
BenchmarkTest.testSayHello:testSayHello·p1.00    sample              2.508               s/op
BenchmarkTest.testSayHello                           ss       3      0.118 ±     0.051   s/op
```

测试曲线图：

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/rpc10000并发测试结果.png">

同时，在同样的条件下，启动 5000（1w个电脑会卡死） 个线程同时对 **Dubbo2.7.14** 发起 RPC 调用，得到的结果如下：

```
Benchmark                                       Mode     Cnt      Score      Error  Units
StressTest.testSayHello                        thrpt       3  41549.866 ± 9703.455  ops/s
StressTest.testSayHello                         avgt       3      0.119 ±    0.034   s/op
StressTest.testSayHello                       sample  611821      0.123 ±    0.001   s/op
StressTest.testSayHello:testSayHello·p0.00    sample              0.042              s/op
StressTest.testSayHello:testSayHello·p0.50    sample              0.119              s/op
StressTest.testSayHello:testSayHello·p0.90    sample              0.129              s/op
StressTest.testSayHello:testSayHello·p0.95    sample              0.139              s/op
StressTest.testSayHello:testSayHello·p0.99    sample              0.195              s/op
StressTest.testSayHello:testSayHello·p0.999   sample              0.446              s/op
StressTest.testSayHello:testSayHello·p0.9999  sample              0.455              s/op
StressTest.testSayHello:testSayHello·p1.00    sample              0.456              s/op
StressTest.testSayHello                           ss       3      0.058 ±    0.135   s/op
```



**结果**：<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/dubbo5000并发测试结果.png">

|            | RPC     | RPC   | Dubbo2.7.14 |
| ---------- | ------- | ----- | ----------- |
| 并发数     | 10000   | 5000  | 5000        |
| TPS        | 29288   | 31675 | 41549       |
| RTT        | 95% 8ms | xxx   | 95% 50ms    |
| AVGTime/OP | 0.532   | 0.532 | 0.119       |
| OOM        | 无      | 无    | 无          |

对比了 jmeter、Apache-Benmark（ab）、jmh 这三个压测工具，个人比较推荐使用jmh，原因有：

- jmh压测简单，只需要引入依赖，声明注解
- 准确性高，目前大多数性能压测都是使用jmh
- 缺点就是代码入侵



# 压测优化

![image-20250605135734528](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250605135734528.png)

优化点1 hessian的 ByteArrayOutputStream 局部缓存（ThreadLocal），将会显著减少 GC 压力，提升吞吐。

优化点2 线程池调优，Dubbo内部进行了很多线程池调优。

优化点3 零拷贝，但是性能几乎不变。（WHY？）

# 问题的解决

## 序列化

1.     Kryo线程不安全，因此要放到 ThreadLocal，并且要提前把嵌套在序列化内部的类注册到Kryo上。[KryoSerialization.java](https://github.com/scatteredream/netty-rpc/blob/ec9b3e776668a05be14ed7fa4db583af176cb81c/rpc-framework-core/src/main/java/com/wxy/rpc/core/serialization/impl/KryoSerialization.java#L29)

2.     Gson原生不能序列化class对象，需要注册一个TypeAdapter完成 JsonElement 和 Class 对象的互转 [ClassCodec.java](https://github.com/scatteredream/netty-rpc/blob/ec9b3e776668a05be14ed7fa4db583af176cb81c/rpc-framework-core/src/main/java/com/wxy/rpc/core/serialization/impl/JsonSerialization.java#L24)

3.     Protostuff提前分配好Buffer，避免每次进行序列化都需要重新分配buffer内存空间。[ProtoStuffSerialization.java](https://github.com/scatteredream/netty-rpc/blob/ec9b3e776668a05be14ed7fa4db583af176cb81c/rpc-framework-core/src/main/java/com/wxy/rpc/core/serialization/impl/ProtostuffSerialization.java#L23)

4.     Hessian可以把ByteArrayOutputStream和BAIS放到ThreadLocal进行复用，不要每次都new。

## SPI

1.     建立双向的映射关系：[SerializationFactory.java](https://github.com/scatteredream/netty-rpc/blob/ec9b3e776668a05be14ed7fa4db583af176cb81c/rpc-framework-core/src/main/java/com/wxy/rpc/core/serialization/SerializationFactory.java#30)

**Serialization：**调用者发消息的时候需要根据Properties构建MessageHeader，在header里面序列化方式是一个byte(type)，因此需要构建一个**nameToTypeMap**。而在真正进行序列化的时候需要根据消息头的byte获取到其实例对象，实例对象通过SpiExtensionFactory进行获取，又需要name作为参数。因此需要**typeToNameMap**。最终的解决方案是在静态代码块中使用反射框架Reflections进行操作，方便地找到所有Serialization的子类。

2.     面向对象：[ServiceDiscovery.java](https://github.com/scatteredream/netty-rpc/blob/ec9b3e776668a05be14ed7fa4db583af176cb81c/rpc-framework-core/src/main/java/com/wxy/rpc/core/discovery/ServiceDiscovery.java) [ServiceRegistry.java](https://github.com/scatteredream/netty-rpc/blob/ec9b3e776668a05be14ed7fa4db583af176cb81c/rpc-framework-core/src/main/java/com/wxy/rpc/core/registry/ServiceRegistry.java)

**ServiceDiscovery/Registry从接口改换成抽象类**：两边都需要启动客户端，原来的想法是从实现类的有参构造注入ip，**注入的同时启动客户端**，这样确实可以直接根据Properties提供对应的注入好的对象，但是这样就不支持动态扩展了，因为SPI的ExtensionLoader只支持无参构造。

那么就想着用一个方法注入（没错就是setter），但是问题来了，接口只支持常量，如果你在接口里面定义一个setter肯定是没办法注入的，那在实现类里通过setter注入呢？可以是可以了，但是你在实际使用的时候肯定都是拿着ServiceDiscovery对象（多态），因此父类对象没法调用子类的setter。

所以说最后就利用抽象类的特性：抽象程度不如接口那么高，正好可以定义一个变量，父类里面实现setter的，子类就可以通过setter注入IP地址，同时不影响多态。

3.     SPI：ExtensionLoader的实现

有点绕：我们需要通过父类/接口的class来构造对应的ExtensionLoader，这应该是一个单例，所以需要使用我们构造的Holder进行获取（classToLoaderMap.computeIfAbsent）。

得到之后就根据key来获取对应实例，从**keyToInstanceMap**.computeIfAbsent(key,()->**createExtension(key)**)）里面拿取。这就走到了下一步，我们首先应该将key和对应的class做好映射，即**keyToClassMap**，那么这就需要我们从文件里读取对应的键值对，然后通过等号右边的全限定名，class.forName 获取class对象，装满keyToClassMap。

映射做好了，拿到class对象：

**classToInstanceMap**.computeIfAbsent(clazz,k->clazz.getDeclaredConstructor().newInstance()); （这个结果赋值给extension）

至此，**createExtension**返回了(T) extension，填充到**keyToInstanceMap**里面

## 负载均衡的实现

一致性哈希

## Spring自定义注解

bean定义的扫描：自定义registrar、里面用扫描器扫描。

## Arthas 

### 检测 CPU 飙高

thread 查看仪表板

thread 73 查看 tid=73的线程，查看调用栈，定位到死循环cpu飙高。

### 监控方法调用

`watch com.example.MyService doSomething '{params, returnObj, cost}' -x 3`

- 这条命令是在不打断程序运行的前提下，**偷偷看一下这个方法到底接收了什么参数、返回了什么结果、用了多久**。中间是个ognl表达式

`doSomething("张三", 18);`:

```yaml
@WatchCondition: class=com.example.MyService, method=doSomething
ts=2024-06-05 15:20:00; [cost=12ms]
params[0]: "张三"
params[1]: 18
returnObj: "Hello 张三, age: 18"
```

`trace com.example.MyService doSomething`

- 查看方法内部每一层的调用耗时，找性能瓶颈

`tt -t com.example.MyService doSomething`

- 记录并回放方法调用

ognl 表达式可以实时修改变量。

### 排查 JVM 

- dashboard (Thread GC 内存)
- jvm 查看 jvm 配置
- vmoption -l 查看jvm XX 参数

### 类加载情况

是否加载、类加载器是否冲突、加载了几个版本

`sc -d com.example.MyService`



# 差异化

在自定义 RPC 框架时，与 Dubbo 这样的成熟框架实现差异化需要从设计理念、功能特性、适用场景等多个维度进行创新。以下是一些可能的差异化方向，分为核心优化、扩展场景和新兴技术整合三类：

## 核心架构与性能优化

1. **更轻量的设计**
   - **去中心化架构**：Dubbo 依赖注册中心（如 Zookeeper、Nacos），可设计为无注册中心的点对点通信（如基于 DNS 或静态配置），降低运维复杂度。
   - **零依赖核心**：剥离非必要模块（如监控、配置中心），核心库仅保留序列化、网络通信和容错，适合嵌入式场景（如 IoT 设备）。
   - **GraalVM 原生镜像支持**：编译为原生二进制，启动速度提升 10 倍以上，内存占用降低 50%（对比 Dubbo 的 JVM 模式）。

2. **极致性能优化**
   - **基于 Rust/Go 重写通信层**：网络库（类似 Netty）用 Rust 实现，减少 GC 停顿，单机 QPS 提升 30%-50%（参考 Apache Dubbo Rust 的实验数据）。
   - **无锁化设计**：请求路由、线程模型避免锁竞争（如单线程事件循环 + 协程调度）。
   - **内存池化技术**：复用序列化缓冲区，减少堆内存分配（类似 gRPC 的 Arena 分配器）。

3. **协议与序列化创新**
   - **自定义二进制协议**：对比 Dubbo 的默认 Hessian2/JSON，可支持 FlatBuffers/Cap'n Proto 等零拷贝序列化，延迟降低 20%-40%。
   - **多协议混合路由**：根据请求特征动态选择协议（如大文件走 HTTP/2，高频小包走自定义二进制协议）。

## 垂直场景深度适配

1. **边缘计算与弱网络**
   - **断网自治模式**：服务节点离线时自动切换为本地存根（如基于 SQLite 缓存），适合移动端/边缘设备。
   - **低带宽优化**：差分序列化（仅传输变化字段），压缩算法动态选择（Zstd/Brotli）。

2. **大规模服务治理**
   - **流量染色与单元化**：基于请求标签的精细路由（如按用户 ID 分片），避免 Dubbo 的全局负载均衡短板。
   - **无损发布**：连接级灰度（长连接保持旧版本流量，新连接路由到新版本），避免 Dubbo 重启导致的流量闪断。

3. **云原生深度集成**
   - **Kubernetes Native**：直接基于 Service Account 做服务发现，替代独立的注册中心。
   - **Serverless 适配**：冷启动优化（预热请求池）、按调用计费（动态伸缩实例）。

## 开发体验与可观测性

1. **开发者友好设计**
   - **DSL 定义接口**：类似 Kotlin 的 suspend RPC 函数，生成客户端代码，避免手动定义 Stub。
   - **交互式调试**：内置请求重放、Mock 服务器（类似 Postman for RPC）。

2. **深度可观测性**
   - **分布式链路追踪**：内置 OpenTelemetry 支持，自动关联 RPC 调用与数据库访问。
   - **实时拓扑图**：动态展示服务依赖关系（类似 Dubbo Admin，但增加流量热力图）。

3. **多语言生态**
   - **WebAssembly 运行时**：客户端逻辑用 WASM 编写，跨语言一致性（对比 Dubbo 的多语言 SDK 维护成本）。
   - **IDE 插件**：IntelliJ/VSCode 插件直接生成接口 Mock 数据。

## 差异化对比表示例

| 特性           | Dubbo 3.x                 | 自定义 RPC 框架差异化点                |
| -------------- | ------------------------- | -------------------------------------- |
| **服务发现**   | 依赖 Nacos/Zookeeper      | 基于 Kubernetes Endpoints 或无中心 DNS |
| **协议扩展**   | 支持 Triple (gRPC)/Dubbo2 | 动态协议选择 + 自定义零拷贝协议        |
| **冷启动速度** | 1-3s (JVM)                | <100ms (GraalVM Native)                |
| **移动端支持** | 无官方优化                | 差分序列化 + 断网自治模式              |
| **调试工具**   | 依赖第三方工具            | 内置请求重放与 IDE 集成                |

## 实施建议

1. **优先聚焦痛点场景**：例如选择 Dubbo 在 IoT 领域资源占用高的短板，针对性优化。
2. **兼容性设计**：提供 Dubbo 协议适配层，降低迁移成本。
3. **开源社区策略**：差异化功能作为扩展插件（如 WASM 运行时），吸引特定场景用户。

通过以上方向，可以在不直接挑战 Dubbo 通用性的前提下，在特定领域（如边缘计算、极致性能、云原生深度集成）建立优势。关键是根据目标用户的实际需求做减法（如移除复杂治理功能）或创新（如内置新型序列化）。
