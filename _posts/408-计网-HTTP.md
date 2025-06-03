---
name: http-in-one
title: HTTP
date: 2025-05-14
tags:
- quic
- http/3
- http/2
- http/1.1
- 缓存
categories: 计网
---

<blockquote class="blockquote-center"><a href="https://hpbn.co/">High Performance Browser Networking</a></blockquote>

# HTTP

## 超文本传输协议（HTTP）

**HyperText Transfer Protocol** based on TCP port 80

TCP/IP 协议四层架构的最上层 规定了服务器和浏览器之间传输数据的规则

[HTTP | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)

[超文本传输协议 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/超文本传输协议) 

### 超文本

**超文本**（英语：**Hypertext**）是一种可以显示在[电脑显示器](https://zh.wikipedia.org/wiki/電腦顯示器)或其他[电子设备](https://zh.wikipedia.org/wiki/電子裝置)上的文本，普遍以[电子文档](https://zh.wikipedia.org/wiki/電子檔)的方式存在，其中的[超文本](https://zh.wikipedia.org/wiki/超文字)包含有可以链接到其他[文件](https://zh.wikipedia.org/wiki/文件系统)/[文件页面](https://zh.wikipedia.org/wiki/頁面)的[超链接](https://zh.wikipedia.org/wiki/超链接)，允许从当前阅读的文字直接切换到超链接所指向的所有文字。

![](https://upload.wikimedia.org/wikipedia/commons/4/41/Sistema_hipertextual.jpg)

### 特点

- 基于TCP，面向连接，安全
- 基于请求-响应模型：1Request1Response
- HTTP是无状态协议，对事务处理没有记忆能力，每次RR都是独立的
  - 缺点：多次请求之间不能共享数据，用会话技术（cookie session）解决这个问题
  - 优点：速度快，没有存储状态的开销

HTTP 协议基于 TCP 协议，发送 HTTP 请求之前首先要建立 TCP 连接也就是要经历 3 次握手。目前使用的 HTTP 协议大部分都是 1.1。在 1.1 的协议里面，默认是开启了 Keep-Alive 的，这样的话建立的连接就可以在多次请求中被复用了。

### 请求数据

**请求行**：第一行，`GET`(请求方式) 后面的`/`表示请求资源的路径，HTTP/1.1表示协议版本

**请求头**：第二行开始 key: value形式

**请求体**：POST请求的最后一部分，存放请求参数

GET请求参数在请求行中，没有请求体，参数大小有限制(URL长度限制) POST请求的参数在请求体中，参数大小无限制

![image-20240926220614547](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926220614547.png)

![image-20240926221417729](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926221417729.png)

**Host**: 请求的主机名

**User-Agent**: 浏览器版本

**Accept**: 浏览器能接受的资源类型，如text/* image/* */\* 

**Accept-Language**: 浏览器的偏好语言

**Accept-Encoding**: 浏览器支持的压缩类型

### 响应数据

**响应行**：响应数据的第一行，HTTP/1.1表示协议版本，下一个是响应状态码，OK表示状态码描述

**响应头**：key value

**响应体**：最后一部分，存放响应数据

![image-20240926221630980](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926221630980.png)

**Content-Type**：响应内容类型，比如text/html image/jpeg

**Content-Length**：响应内容长度（bytes）

**Content-Encoding**：响应压缩算法 gzip等

**Cache-Control**: 指示客户端如何缓存，例如max-age=300 表示最多缓存300s

#### 状态码

[HTTP 常见状态码总结（应用层） | JavaGuide](https://javaguide.cn/cs-basics/network/http-status-codes.html) 

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/http-status-code.png" alt="常见 HTTP 状态码" style="zoom:67%;" />

##### 2xx Success

- <mark>200 OK</mark>：请求被成功处理。例如，发送一个查询用户数据的 HTTP 请求到服务端，服务端正确返回了用户数据。这个是我们平时最常见的一个 HTTP 状态码。
- **201 Created**：请求被成功处理并且在服务端创建了一个新的资源。例如，通过 POST 请求创建一个新的用户。
- **202 Accepted**：服务端已经接收到了请求，但是还未处理。例如，发送一个需要服务端花费较长时间处理的请求（如报告生成、Excel 导出），服务端接收了请求但尚未处理完毕。
- **204 No Content**：服务端已经成功处理了请求，但是没有返回任何内容。例如，发送请求删除一个用户，服务器成功处理了删除操作但没有返回任何内容。（也就是 true or false）

##### 3xx Redirection

- **301 Moved Permanently**：资源被永久重定向了。比如你的网站的网址更换了。
- **302 Found**：资源被临时重定向了。比如你的网站的某些资源被暂时转移到另外一个网址。
- **304 Not Modified**：表示资源在由请求头中的If-Modified-Since或If-None-Match参数指定的这一版本之后，未曾被修改。在这种情况下，由于客户端仍然具有以前下载的副本，因此不需要重新传输资源。

##### 4xx Client Error

- **400 Bad Request**：发送的 HTTP 请求存在问题。比如请求参数不合法、请求方法错误。
- **401 Unauthorized**：未认证却请求需要认证之后才能访问的资源。
- **403 Forbidden**：直接拒绝 HTTP 请求，不处理。一般用来针对非法请求。
- **404 Not Found**：你请求的资源未在服务端找到。比如你请求某个用户的信息，服务端并没有找到指定的用户。
- **409 Conflict**：表示请求的资源与服务端当前的状态存在冲突，请求无法被处理。

##### 5xx Server Error

- **500 Internal Error**：服务端出问题了（通常是服务端出 Bug 了）。比如你服务端处理请求的时候突然抛出异常，但是异常并未在服务端被正确处理。
- **503 Service Unavailable**：由于临时的服务器维护或者过载，服务器当前无法处理请求。这个状况是暂时的，并且将在一段时间以后恢复。如果能够预计延迟时间，那么响应中可以包含一个Retry-After头用以标明这个延迟时间。如果没有给出这个Retry-After信息，那么客户端应当以处理500响应的方式处理它。
- **502 Bad Gateway**：我们的网关将请求转发到服务端，但是服务端返回的却是一个错误的响应。

Java程序中，如果直接用自带的javawebsocketAPI 代码会变得异常繁琐，要注意请求和响应的格式要求，因此要用web服务器软件进行开发----Tomcat

### [HTTP 缓存](https://xiaolincoding.com/network/2_http/http_interview.html#http-缓存技术)  

#### 强制缓存

响应头里有 `Cache-control`：提供了多种指令，用于精确控制缓存行为（时间段）。如 `max-age`、`no-cache`、`no-store`、`must-revalidate` 等。示例：`Cache-Control: max-age=3600`（资源有效期 3600 秒），其优先级高于 `Expires`（绝对时间）。

- 当浏览器第一次请求访问服务器资源时，服务器会在返回这个资源的同时，在 Response 头部加上 Cache-Control，Cache-Control 中设置了过期时间大小； 
- 浏览器再次请求访问服务器中的该资源时，会先通过请求资源的时间与 Cache-Control 中设置的过期时间大小，来计算出该资源是否过期，如果没有，则使用本地缓存，否则重新请求服务器； 
- 服务器再次收到请求后，会再次更新 Response 头部的 Cache-Control。

可以看到响应的结果 (from disk cache)。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1cb6bc37597e4af8adfef412bfc57a42.png)

#### 协商缓存

**协商缓存只有在未命中强制缓存的时候才能使用，因此必须依赖于 之前 Cache-control 的缓存。**

条件请求 Etag 的优先级高于 Last-Modified，因为 Last-Modified 是秒级粒度，而 Etag 更能准确判断是否被修改

- 服务器响应中带上Last-Modified:x 
- 客户端下次请求就带上If-Modified-Since:x
- 如果服务器看鉴定没有过期，回复一个HTTP 304 允许使用缓存，过期就回复HTTP 200发送最新数据。



<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/network/http/http%E7%BC%93%E5%AD%98.png" alt="img" style="zoom: 50%;" />

许多[CDN](https://zh.wikipedia.org/wiki/內容傳遞網路)和网络设备制造商已经用动态缓存取代了这个标准的HTTP缓存控制。

### HTTPS

[HTTP vs HTTPS（应用层） | JavaGuide](https://javaguide.cn/cs-basics/network/http-vs-https.html) 

[HTTPS in one - scatteredream’s blog](https://scatteredream.github.io/2025/05/12/408-计网-HTTPS/) 

# HTTP/1.0 到 HTTP/3

[HTTP/1.1、HTTP/2、HTTP/3 演变 - 小林 coding](https://xiaolincoding.com/network/2_http/http_interview.html#http-1-1、http-2、http-3-演变) 

## 早期版本

### HTTP/0.9

只支持 GET 请求，不支持请求头。

### HTTP/1.0

基本成型，支持富文本、请求头、状态码、缓存。第一个在通讯协议中使用版本号的协议，状态码、请求方式比较少。

## HTTP/1.1

[HTTP 1.0 vs HTTP 1.1（应用层） | JavaGuide](https://javaguide.cn/cs-basics/network/http1.0-vs-http1.1.html) 

### 持久连接 (默认开启 `Connection: keep-alive`)

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/d5/HTTP_persistent_connection.svg/2560px-HTTP_persistent_connection.svg.png" style="zoom:25%;" />

在 HTTP 1.0 中，每次请求 - 响应完成后，连接就会被关闭。这意味着如果客户端需要再次请求服务器上的其他资源，就需要重新建立连接，而建立连接需要消耗额外的时间和资源，包括 DNS 查询、TCP 握手等过程。HTTP 1.0协议中并未定义持久连接的实现方式，但是一些服务端和客户端开始使用这种方式交互。如果浏览器支持keep-alive，它会在请求头添加`Connection: keep-alive` 服务器也会在响应头中添加相同的字段。

在HTTP 1.1中，除非任意一方在请求时明确声明不支持，否则所有的连接默认都是持续连接，即一次连接可以在多个请求 - 响应之间复用。这样可以显著减少连接建立的开销，提高性能，特别是对于包含多个资源的网页（如包含多个图片、脚本、样式表等），可以大大加快页面的加载速度。

### 管道化 (Pipelining)

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/fb/HTTP_pipelining.svg/2560px-HTTP_pipelining.svg.png" alt="undefined" style="zoom: 25%;" />

HTTP 1.0 那样必须等待前一个请求的响应返回后才能发送新的请求。

在持久连接的基础上，HTTP 1.1 还支持管道化。管道化允许客户端在未收到前一个请求的响应时，就可以发送下一个请求。这样可以进一步提高效率，减少整体的请求 - 响应时间。例如，客户端可以连续发送多个对不同资源的请求，服务器按照请求的顺序依次响应，客户端可以在接收响应的同时继续发送新的请求，充分利用连接的带宽。长连接会复用一个 TCP 连接。

- 服务端要遵循 HTTP/1.1 协议，必须按照客户端发送的请求顺序来回复请求。
- 一般又允许每个主机建立6个TCP连接，这样可以更加充分的利用带宽资源。
- 但每个连接中队头阻塞的问题还是存在的。这样整个连接还是先进先出的，[队头阻塞](https://zh.wikipedia.org/wiki/队头阻塞)（HOL blocking）可能会发生，造成延迟。

### 更好的缓存机制 (`Expires,Last-Modified->Cache-Control,ETag`)

```http
HTTP/1.1 200 OK
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Last-Modified: Wed, 15 Jun 2022 12:00:00 GMT
```

在 HTTP 1.0 中，主要通过`Expires`, `Last-Modified` , `If-Modified-Since` 实现缓存机制。服务器可以在响应头中设置`Expires`字段，指定资源的过期时间。客户端在请求资源时，会检查本地缓存中资源的过期时间。如果未过期，客户端就可以直接使用缓存中的资源，而不必再次向服务器发送请求，从而提高了性能和减少了网络带宽的占用。Expires 字段依赖于服务器和客户端的时钟同步，如果两者的时钟不一致，可能会导致缓存策略出现偏差。对于缓存的控制相对简单，不够灵活，无法满足复杂的缓存需求。

- 使用 `Last-Modified` 和 `If-Modified-Since` 实现条件请求。
- 服务器返回资源的最后修改时间（`Last-Modified`），客户端下次请求时通过 `If-Modified-Since` 询问资源是否有更新。



------



```http
HTTP/1.1 200 OK
Cache-Control: max-age=3600, public
ETag: "abc123"
Last-Modified: Wed, 15 Jun 2022 12:00:00 GMT
```

相比之下，HTTP 1.1 引入了更强大和灵活的缓存机制，如`Cache - Control`头字段，提供了更多的缓存控制选项，能够更精确地管理缓存行为。

- 新增 `ETag`（实体标签）和 `If-None-Match` 作为条件请求更可靠的验证方式。优先使用 `ETag`，其次才是 `Last-Modified`，提高验证准确性。
- `ETag` 是资源的唯一标识符（如哈希值），即使资源修改时间未变，内容变化也会导致 `ETag` 变化，避免误判。

### 其他

- 增加了更多的请求方法（如 PUT、DELETE 等）和状态码

- 支持断点续传（通过 Range 头字段，它允许只请求资源的某个部分，即返回码是 206（Partial Content），方便了开发者自由的选择以便于充分利用带宽和连接。
- **Host 头处理** : HTTP/1.1 在请求头中加入了`Host`字段。
- 支持分块发送（应用层的分块）

## [HTTP/2](https://hpbn.co/http2/)

> From *High Performance Browser Networking*

HTTP/2 [2015] 的前身是 SPDY[2009]

### 多路复用（Stream）

HTTP 1.1 对于连续多个请求可能会开启**并行的最多6个TCP连接**，用来提高带宽利用率。另外，**管道化**虽然也能实现同时发送多个请求，但是返回的时候是会阻塞的，**受限于其请求-响应的基本模型**，谁先到达，谁先返回，顺序绝对不能乱，因此造成了**队头阻塞**。HTTP 1.1还有另一个限制，只有幂等请求（get、head等）才能使用，大部分浏览器默认是关闭管道化的。

------

HTTP/2中，浏览器针对同一个域名的资源，只建立一个 TCP 通道，所有的针对这个域名的请求全部在这个通道中完成，并且引入了**流（Stream）**的机制，如下图：

![Figure 12-2. HTTP/2 streams, messages, and frames](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/8e6931bb40fc26c511ad15645e7b6113.svg)

- **流**（Stream）：在一个 TCP 连接中进行的一个独立的、双向的、按序的数据交换。换句话说，流就对应着一个 **请求-响应对**。所有的流共享同一个 TCP 连接。
- **流编号**（Stream ID）：是一个 31 位的无符号整数（最高位保留），用于唯一标识每个流。客户端发起的流编号是奇数，服务器发起的是偶数。
- **帧**（Frame）：协议将每个请求/响应（Message）分割为二进制头帧与数据帧。并且帧有所属的流编号，帧可以乱序传输，减轻**应用层**的**队头阻塞**。
- 举例：客户端发送 1 3 5 号请求，如下图，请求5的数据帧还在途中，服务端同时处理了请求1 3，可以看到是乱序的实现了多路复用：

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/47ba5b32e42cf5a06c3741d29ef9b94a.svg"  />



### 以二进制帧为单位的流

**帧是原子单位**：HTTP/2 协议规定，每个帧（包括 `HEADERS` 和 `DATA`）必须作为一个完整单元发送和接收。帧头（Frame Header）明确定义了帧的长度（Length 字段，占 3 字节），接收方必须读取完整的帧后才能处理。

- 例如：一个 `DATA` 帧若声明长度为 1000 字节，则必须完整传输这 1000 字节的数据。
- HTTP/2 Flow control只对`DATA`帧生效，`HEADERS`作为元数据始终处于高优先级

<img src="https://hpbn.co/assets/diagrams/ae09920e853bee0b21be83f8e770ba01.svg" alt="img"  />

#### DATA 帧：载荷分块传输

![Figure 12-9. DATA frame](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/8199bc14fc3e692d5ea83792822d5def.png)

`DATA`帧的的**载荷**（Payload）可能会因底层限制（如 TCP MSS、流控等）被**间接拆分或分片**

- **HTTP/1.1 的 `Transfer-Encoding: chunked`**：显式将响应体拆分为多个块（每个块包含长度前缀），是应用层行为。
- **HTTP/2 的 二进制帧**：天然支持分块传输，可以将负载拆到多个帧里。最后一帧将 `END_STREAM`字段设置为true，false 表明表明负载尚未完成传输，更多的数据帧即将到来。

#### HEADER 帧编码：HPACK 算法

双方共同维护字典（静态表、动态表及 Huffman 编码），将HEADER中的重复出现的常见部分进行编码，取代之前的ASCII文本。图中 1 号流和 3 号流分别对共享字典进行了更新。（path 更新到动态表）

![Figure 12-6. HPACK: Header Compression for HTTP/2](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/feb142f82737d148ed5bcefd91915276.svg)

![Figure 12-8. Decoded HEADERS frame in Wireshark](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/61dc2bae615536155a5af7203ad191fd.png)

### 服务器推送 (Server Push)

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/d759887277b266a42c526643285dd244.svg" alt="Figure 12-5. Server initiates new streams (promises) for push resources"  />

网站为了使请求数减少，通常采用对页面上的图片、脚本进行极简化处理。但是，这一举措十分不方便，也不高效，依然需要诸多HTTP链接来加载页面和页面资源。

HTTP/2引入了**服务器推送**，即服务端向客户端发送比客户端请求更多的数据。服务器根据HTML的内容解析需要发送哪些内容，直接提供浏览器渲染页面所需要的资源，而无须浏览器在收到、解析页面后再提起一轮请求，节约了加载时间。

### 其他

- HTTP/2禁用了诸多加密包
- **允许设置请求优先级**，服务器根据优先级响应

## HTTP/3 (QUIC)

HTTP2协议虽然大幅提升了 HTTP/1.1 的性能，然而，基于TCP实现的HTTP/2遗留下3个问题：

- **TCP** 的有序字节流导致丢包就会出现**传输层**队头阻塞（Head-of-line blocking），使得HTTP2的多路复用能力大打折扣
- TCP与TLS叠加握手时延
  - HTTPS 需要经过[三次TCP握手](https://scatteredream.github.io/2025/05/11/408-计网-TCP+UDP/)和[四次TLS握手（TLS v1.2）](https://scatteredream.github.io/2025/05/12/408-计网-HTTPS/)总共3个RTT才能发送数据。
  - TCP连接的慢启动拖延连接建立

- 基于TCP四元组确定一个连接，这种诞生于有线网络的设计，并不适合移动状态下的无线网络，这意味着IP地址的频繁变动会导致TCP连接、TLS会话反复握手，成本高昂。

<img src="https://miro.medium.com/v2/resize:fit:1875/1*uk5OZPL7gtUwqRLwaoGyFw.png" alt="img" style="zoom:67%;" />



HTTP/3 的 QUIC 实现了 **多个stream的拥塞控制与可靠传输**、**原生加密**、**连接迁移** 

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/quic-fig1.png" alt="A quick look at QUIC | APNIC Blog" style="zoom: 80%;" />

### HTTP/3 帧：简化帧头、升级压缩算法

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240105144457456.png" alt="image-20240105144457456" style="zoom: 33%;" />

HTTP/3 延续了 HTTP/2 的帧结构设计，仍通过 **DATA帧** 传输请求/响应的正文内容，通过 **HEADERS帧** 传输经过 **QPACK** (HTTP/2 为 HPACK) 压缩的 HTTP 头部信息，其他帧类型（如 SETTINGS、CANCEL_PUSH 等）也得到保留，但部分功能因 QUIC 的特性有所调整。

封装：HTTP/3 帧 → 封装到 QUIC STREAM 帧 → 封装到 QUIC 包 → 通过 UDP 发送。

- **HTTP/2**：基于 TCP，帧在流（Stream）上传输，流由 HTTP/2 自身管理，存在队头阻塞问题
- **HTTP/3**：基于 QUIC，直接复用 QUIC 的流机制，帧通过 QUIC 流传输。QUIC 的流相互独立，避免了 TCP 层的队头阻塞。
  1. HTTP/3 的**帧头仅包含类型和长度字段**，比 HTTP/2 更简洁，因为 Stream 管理转移给了 QUIC。
  2. HPACK 升级为 QPACK，增加静态表的同时改进动态表：

> 所谓的动态表，在首次请求-响应后，双方会将未包含在静态表中的 Header 项更新各自的动态表，接着后续传输时仅用 1 个数字表示，然后对方可以根据这 1 个数字从动态表查到对应的数据，就不必每次都传输长长的数据，大大提升了编码效率。动态表是具有**时序性**的，如果首次出现的请求**发生丢包**，后续的收到请求，对方就无法解码出 HPACK 头部，因为对方还没建立好动态表，因此后续的请求解码会阻塞直到首次请求中丢失的数据包重传过来。

HTTP/3 使用两个单向流，一个叫 **QPACK Encoder Stream**，用于将一个字典（Key-Value）传递给对方，比如面对不属于静态表的 HTTP 请求头部，客户端可以通过这个 Stream 发送字典；一个叫 **QPACK Decoder Stream**，用于响应对方，告诉它刚发的字典已经更新到自己的本地动态表了，后续就可以使用这个字典来编码了。这两个单向流同步双方的动态表，编码方收到解码方的确认通知以后才使用动态表编码头帧。解决了HTTP/2的HPACK队头阻塞（也是因为TCP）

### UDP 上实现基于 Stream 的可靠传输

[HTTP/3, QUIC, and How it Works. Though HTTP/3 is still in draft today… | by Carson | Medium](https://cabulous.medium.com/http-3-quic-and-how-it-works-c5ffdb6735b4) 

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-49a6be0d647f0f9241a122971166ea23_1440w.jpg" alt="金山视频云推出QUIC+ ，畅快直播再升级（技术篇） - 知乎" style="zoom:67%;" />

| 特性         | HTTP/2 的 stream                   | HTTP/3 (QUIC) 的 stream                          |
| ------------ | ---------------------------------- | ------------------------------------------------ |
| 属于哪一层   | 应用层                             | 传输层（QUIC）                                   |
| 是否独立传输 | 否，共用 TCP 的一个流              | 是， stream 为传输层的基本单位，天然支持多路复用 |
| 队头阻塞问题 | 有，TCP 一丢包，所有 stream 都阻塞 | 无，哪个流丢包只影响该流                         |
| 序号机制     | Stream ID（奇偶区分）              | Stream ID（可双向并发）                          |
| 对应         | 一个请求/响应对                    | 一个请求/响应对                                  |

#### QUIC Packet：传输层单元

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-60231adb6c7014c7f043712839f77ab5_1440w.jpg" alt="img" style="zoom: 50%;" />

- Packet 数据包是 QUIC 的传输单元，负责在 UDP 上可靠传输数据，一个 UDP 数据报里可以有一个或者多个 QUIC Packet。
- **作用**：用来可靠传输 Frame（包括加密、确认、重传）
- **构成**：包含 **头部（Header）** 和 **负载（Payload）**。
- **头部**：连接 ID（标识终点和起点，将 HTTP 连接和 UDP 四元组解耦）、QUIC 版本、Public Flags、**Packet Number**。明文。
- **负载**：可能包含一个或多个 **QUIC 帧**（QUIC Frames）。密文。

#### QUIC Frame：协议层单元

**作用**：QUIC 协议的最小数据单元，用于实现连接控制、流管理、ACK、和数据传输等。

**​关键帧类型​**​：

- **STREAM 帧**：传输应用层数据（如 HTTP/3 的帧）。
- **CRYPTO 帧**：传输 TLS 握手数据。
- **ACK 帧**：确认收到的包号（替代 TCP 的 ACK）。
- **MAX_DATA 帧**：流量控制（类似 TCP 的窗口机制）。

##### STREAM 帧：传输应用层数据

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-f1cb88ac186e851a724a85dbd4f3de01_r.jpg" alt="img" style="zoom: 40%;" />

- **Frame Type**：标识帧类型。
- **Stream ID**: 唯一标识 Stream，因此一个 QUIC Stream ID 就对应唯一一个 HTTP 请求-响应对。
- **offset**：偏移量，用于排序。
- **Data Length**：data长度。

##### ACK 帧：[可靠传输](https://zhuanlan.zhihu.com/p/405387352) 

**问题 1：发送端怎么知道发出的包是否被接收端收到了？**

解决方案：通过包号（PKN）和选择确认（Selective-ACK）

1. 客户端：发送 3 个数据包给服务器（PKN = 1,2,3）
2. 服务器：通过 SACK 告知客户端已经收到了 1 和 3，没有收到 2
3. 客户端：重传第 2 个数据包（PKN=4）

由此可以看出，QUIC 的PKN是单调递增的。也就是说，之前发送的数据包（PKN=2）和重传的数据包（PKN=4），虽然数据一样，但包号不同。

**问题 2：既然包号是单调递增的，那接收端怎么保证数据的有序性呢？**

解决方案：通过数据偏移量 offset

1. 每个数据包都有一个 offset 字段，表示在整个数据中的偏移量，比如 PKN 1,2,3 分别对应 offset 0,1,2，重传包 PKN=4, offset=1。
2. 接收端根据 offset 字段就可以对异步到达的数据包进行排序了。

**为什么 QUIC 要将 PKN 设计为单调递增？**

解决 TCP 的重传歧义问题：由于原始包和重传包的序列号是一样的，客户端不知道服务器返回的 ACK 包到底是原始包的，还是重传包的。但 QUIC 的原始包和重传包的序列号是不同的，也就能够判断 ACK 包的归属。

#### 可靠传输防止不同 Stream 之间的队头阻塞

QUIC 协议会保证数据包的可靠性，每个数据包都有一个 Packet Number 唯一标识。 **QUIC 的 packet 是以 frame 为单位封装的，frame 可跨 stream 独立发送**

当某个包丢失，里面帧所对应的流就会影响，即使该流的其他帧到达了，数据也无法被 HTTP/3 读取，流内部仍然会阻塞，直到 QUIC 重传丢失的报文，数据才会交给 HTTP/3。而其他跟这个包没关系的流完全不受影响，HTTP/3 可以读取到数据。

因此，QUIC 连接上的多个 Stream 之间并没有依赖，都是独立的，某个流所在的包发生丢包，只会影响该流，其他流不受影响。HTTP/2 就会互相影响，因为基于 TCP

### 建立连接速度快、原生加密

![QUIC-PICTURE-04-1024x553.jpg](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/QUIC-PICTURE-04-1024x553.jpg)

| 特性         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| 使用协议     | **TLS 1.3**（不是 TCP 上的 TLS，而是集成到 QUIC 内部）       |
| 加密内容     | 所有 **QUIC Packet 的 Payload（Frame）** 都会加密；只有 **部分 Header 字段明文** |
| 握手流程     | 连接建立阶段使用 **QUIC CRYPTO Frame** 承载 TLS 1.3 的握手数据 |
| 对称密钥生成 | 通过 TLS 1.3 完成密钥协商（ECDHE）                           |
| 后续通信加密 | 使用 **AEAD 算法**（如 AES-GCM 或 ChaCha20-Poly1305）加密 QUIC Packet 的 payload |
| Header 保护  | QUIC 还对部分 Header 字段进行加密混淆，防止中间人分析        |

| 握手步骤     | 内容                                 | QUIC 特点                     |
| ------------ | ------------------------------------ | ----------------------------- |
| Client Hello | 用 CRYPTO frame 承载 TLS ClientHello | 无需 TCP Handshake            |
| Server Hello | 返回证书、公钥等                     | 建立早期密钥                  |
| 完成握手     | 双方完成密钥协商                     | 建立 0-RTT/1-RTT 数据加密通道 |

对于 HTTP/1 和 HTTP/2 协议，TCP 和 TLS 是分层的，分别属于内核实现的传输层、OpenSSL 库实现的表示层，因此它们难以合并在一起，需要分批次来握手，先 TCP 握手，再 TLS 握手。HTTP/3 在传输数据前虽然需要 QUIC 协议握手，这个握手过程只需要 1 RTT，握手的目的是为确认双方的「连接 ID」，连接迁移就是基于连接 ID 实现的。

QUIC 默认包含并开启 TLS 1.3，仅需 1 RTT 即可完成连接的建立和密钥协商，第二次连接，在握手完成之前，客户端就能基于缓存密钥发送部分早期加密应用数据。达到 0 RTT 的效果。

### 连接迁移

QUIC 协议没有用四元组的方式来“绑定”连接，而是通过「**连接 ID** 」来标记通信的两个端点，客户端和服务器可以各自选择一组 ID 来标记自己，因此即使移动设备的网络变化后，导致 IP 地址变化了，只要仍保有**上下文信息（比如连接 ID、TLS 密钥等）**，就可以**无缝复用**原连接，消除重连成本。

------

# 服务器推送（Server Push）

<img src="https://hpbn.co/assets/diagrams/1a8db2948eb2aad0dd47470c6c011a42.svg" alt="Figure 17-2. Communication flow of XHR, SSE, and WebSocket"  />

方法主要有 长轮询、SSE、WebSocket三种

## 长轮询 (Long-polling)

<img src="https://websocket.org/_astro/long-polling.d7000adb_Vn4A9.webp" style="zoom:67%;" />

从本质上讲，长轮询是一种更有效的轮询形式，是一项服务器选择尽可能长时间保持客户连接的技术，服务器仅在请求的数据可用或达到超时门槛后才提供响应。收到服务器响应后，客户端通常立即发出另一个请求。本质还是客户端发起请求。

## 服务器发送事件 (Server-Sent Events)

SSE是一种通常用于将消息更新或连续数据流发送给浏览器客户端的服务器推送技术。 SSE旨在通过称为EventSource的JavaScript API来增强本地的、跨浏览器的、Server-to-Client的推送，是 HTML5 的一部分。本质是服务器这边专门开一条 HTTP 链接，这条链接里面服务器和客户端的角色互换。

```js
var source = new EventSource('URL_TO_EVENT_STREAM');
source.onopen = function () {
  console.log('connection to stream has been opened');
};
source.onerror = function (error) {
  console.log('An error has occurred while receiving stream', error);
};
source.onmessage = function (stream) {
  console.log('received stream', stream);
};
```



## [WebSocket](https://xiaolincoding.com/network/2_http/http_websocket.html) 

WebSocket 是一种基于 TCP 连接的**全双工通信协议**，即客户端和服务器可以同时发送和接收数据。WebSocket 协议在 2008 年诞生，2011 年成为国际标准，几乎所有主流较新版本的浏览器都支持该协议。不过，WebSocket 不只能在基于浏览器的应用程序中使用，很多编程语言、框架和服务器都提供了 WebSocket 支持。WebSocket 协议本质上是应用层的协议，用于弥补 HTTP 协议在持久通信能力上的不足。客户端和服务器仅需一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。 常见应用场景：视频弹幕、实时消息推送，详见[Web 实时消息推送详解](https://javaguide.cn/system-design/web-real-time-message-push.html)这篇文章、实时游戏对战、多用户协同编辑、社交聊天。

|                  | **WebSockets**                   | **HTTP/1.1**         |
| ---------------- | -------------------------------- | -------------------- |
| **通信形式**     | 全双工                           | 半双工               |
| **信息交换方式** | 双向通信                         | 基于请求-响应对      |
| **服务器推送**   | 核心特色                         | 原生不支持           |
| **开销**         | 连接建立有一定开销，信息开销极小 | 每条信息都有一定开销 |
| **状态**         | 有状态                           | 无状态               |

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/websocket-vs-http.be03ced1_sWP4f.webp" alt="Websocket 示意图" style="zoom: 67%;" />

WebSocket 的工作过程可以分为以下几个步骤：

1. 客户端向服务器发送一个 HTTP 请求，请求头中包含 `Upgrade: websocket` 和 `Sec-WebSocket-Key` 等字段，表示要求升级协议为 WebSocket；
2. 服务器收到这个请求后，会进行升级协议的操作，如果支持 WebSocket，它将回复一个 HTTP 101 状态码，响应头中包含 ，`Connection: Upgrade`和 `Sec-WebSocket-Accept: xxx` 等字段、表示成功升级到 WebSocket 协议。
3. 客户端和服务器之间建立了一个 WebSocket 连接，可以进行双向的数据传输。数据以帧（frames）的形式进行传送，WebSocket 的每条消息可能会被切分成多个数据帧（最小单位）。发送端会将消息切割成多个帧发送给接收端，接收端接收消息帧，并将关联的帧重新组装成完整的消息。
4. 客户端或服务器可以主动发送一个关闭帧，表示要断开连接。另一方收到后，也会回复一个关闭帧，然后双方关闭 TCP 连接。

另外，建立 WebSocket 连接之后，通过心跳机制来保持 WebSocket 连接的稳定性和活跃性。