---
name: net-safety
title: HTTPS(TLS)
date: 2025-05-12
tags:
- tls
- https
- 加密
categories:
- 计网

---



# 计算机面临的安全威胁

## 攻击类型

[网络攻击常见手段总结 | JavaGuide](https://javaguide.cn/cs-basics/network/network-attack-means.html) 

**被动攻击**：也叫截获，流量分析。

**主动攻击**：

- **篡改报文**。

- **恶意程序**（病毒、蠕虫、木马、流氓软件、漏洞入侵、逻辑炸弹）
- **拒绝服务**（DoS, Denial of Service）：从互联网的一个服务器A发送大量信息到另一个服务器B，使得服务器B崩溃，拒绝服务。从多个服务器发分组又叫做分布式拒绝服务（DDoS, Distributed DoS）。负载均衡（anycast...）
- **交换机中毒**：发送大量伪造MAC帧，迅速填满交换机的交换表，使交换机无法正常提供服务。

## 安全措施

- **机密性**：密码技术，加密报文信息；对称加密
- **信息完整性**：防止报文信息被篡改；非对称加密，签名-鉴别
- **端点鉴别**（Authentication）：鉴别对方的真实身份；



- **访问控制**（Authorization）：确认用户的权限级别，能够进行什么样的操作。

### Authentication（身份验证）

确认用户的身份，确保用户是他声称的那个人。

验证用户的凭证（如用户名/密码、指纹、OTP 等）。

#### 例子

- 登录网站时输入用户名和密码。
- 使用指纹、面部识别或其他生物识别方式解锁设备。
- 输入短信验证码进行二次验证。

#### 重点

- “你是谁？”
- 解决的是**身份问题**。

### Authorization（授权）

确定已经通过身份验证的用户是否有权访问某资源或执行某操作。

控制用户能访问的资源或功能。

#### 例子

- 一个普通用户登录后不能访问管理员控制面板。
- 文件系统中，只有特定权限的用户可以修改文件。
- 云服务中，用户可能只能管理自己的项目，不能访问其他人的数据。

#### 重点

- “你可以做什么？”
- 解决的是**权限问题**。

### 区别与联系

| **对比项**   | **Authentication（身份验证）** | **Authorization（授权）**            |
| ------------ | ------------------------------ | ------------------------------------ |
| **目的**     | 确认用户身份                   | 确定用户访问权限                     |
| **处理对象** | 用户的身份                     | 用户的权限                           |
| **执行顺序** | 先进行身份验证                 | 只有通过身份验证后才能进行授权       |
| **核心问题** | 用户是否真实有效？             | 用户能访问哪些资源或执行哪些操作？   |
| **技术示例** | 密码验证、2FA、OAuth 登录      | 角色权限分配（RBAC）、ACL、API Token |

`Authentication` 是 `Authorization` 的前提。
 一个用户必须先证明“自己是谁”，然后才能被系统允许或限制访问某些资源或执行操作。

# 加密算法

## 两类密钥体制

### [对称加密](https://zh.wikipedia.org/wiki/%E5%B0%8D%E7%A8%B1%E5%AF%86%E9%91%B0%E5%8A%A0%E5%AF%86)

双方使用同一把密钥，不对外公开，使用密钥将报文信息进行加密或者进行解密。

- **DES**：密钥共64位，8位用于奇偶校验，实际为56位有效。
- **3DES**：使用两个DES密钥，先用K1加密，再用K2解密，再用K1加密

- **AES**：使用分组加密，密钥长度为128 192 256三种
- **一个传统保管箱，开门和关门都是使用同一条钥匙，这是对称加密** 
- 只支持一对一通信。

### [公钥加密（非对称加密）](https://zh.wikipedia.org/wiki/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86)

- 使用密钥生成器生成密钥对，一个是公钥（对外公开），一个是私钥（自己保留）。
- 已知其中一个密钥的情况下，几乎不可能知道另外一个密钥。
- 私钥加密的数据，只能用公钥解密。公钥加密的数据，只能用私钥解密。
  - 其实不应该叫做加密和解密，应该称作D运算和E运算这两个互逆的运算，没有规定他们谁先谁后。
  - 更准确的说法是私钥参与D运算，公钥参与E运算。

- **一个公开的邮箱，投递口是任何人都可以寄信进去的，这可视为公钥；**
- **而只有信箱主人拥有钥匙可以打开信箱，这就视为私钥。**
- 支持多对一的通信。

## 鉴别（Authentication）

### 报文鉴别

主要是鉴别报文是否被篡改。

#### 数字签名

decryption and encryption

A要发出信息，并进行数字签名，将报文用私钥SK~A~进行D运算（签名），其他人只能用公钥PK~A~进行E运算（鉴别）。

- **防止篡改报文**：没人知道A的私钥SK~A~，窃听者使用公钥PK~A~进行E运算得出明文，篡改报文之后再D运算，另一边使用公钥PK~A~解析出的明文将会是不可读的，因此不会被欺骗。
- **不可否认**：既然信息明文可读，就说明没有被篡改过，而拥有私钥的只有A一个人，A就不能抵赖自己曾经的签名。

上述方法是对报文本身进行了私钥加密，公钥直接可以解出报文具体内容，如果要用非对称加密实现报文内容的保密，可以在A用私钥D运算之后，再用B的公钥E运算，这样就彻底加密了报文内容，B收到以后先用B的私钥D运算，再用A的公钥E运算。

#### 哈希函数

是一种摘要算法，将任意长度的报文经过哈希函数可以的到固定长度的字符串。

#### 结合报文鉴别码（MAC）实现数字签名报文鉴别

Message Authentication Code, MAC

H(X) 代表 X 的哈希值；D(X) 代表 X 经过私钥的D运算以后的结果；E(X) 代表 X 经过 公钥的E运算之后的结果

$D(E(X)) = X$          $E(D(X)) = X$ 

扩展报文实现的数字签名可以分为三类：

$A + H(A)$ ：只篡改报文A或者H(A)可以鉴别出来，但是直接换一个 X + H(X) 直接会蒙混过关，由此引出了使用密钥K来进行哈希运算的方法。

$A + H(A,K)$： 使用对称密钥$K$，如果报文$A$被篡改成$X$，但是攻击者不知道密钥，因此无法伪造出哈希值$H(X,K)$，另一边根据密钥计算出$H(X,K)$，就能发现报文被篡改过。这样将密钥拼接在正文之后，得出的散列值就是**HMAC**。JWT(Json Web Token)采用的数字签发就可以采用这样的方式 （HS256 代表 HMAC 散列函数为SHA-256）**对密钥K<u>严格要求保密</u>**。[JWT 签名算法 HS256、RS256 及 ES256 及密钥生成 - This Cute World](https://ryan4yin.space/posts/jwt-algorithm-key-generation/) 

$A + D(H(A))$：使用私钥对报文的哈希进行D运算，其他人使用公钥对密文进行E运算能够得出哈希值，将这个哈希值和报文段的哈希值比较即可。如果$A$被篡改为$X$，攻击者因为不知道发送者的私钥，因此无法得出一个正确的$D(H(X))$，这时候发给接收者，接受者再对攻击者伪造的哈希值密文进行E运算，得出哈希值肯定和真正的$H(X)$不同，这就是RS256和ES256的原理。**由此方法扩展出的报文<u>不可伪造，也不可否认</u>**，同时可以运用到分布式架构中，一个服务签发，其他服务验证只需要公钥即可。ES256（ECDSA）使用椭圆曲线计算公钥。

[JWT 签名算法 HS256、RS256 及 ES256 及密钥生成 - 於清樂 - 博客园](https://www.cnblogs.com/kirito-c/p/12402066.html) 

### 实体鉴别

#### 对称密钥

主要是验证来访者的确是访问者，也就是鉴别身份。

相比于报文鉴别，实体鉴别只需要鉴别一次发送者即可，后续传输不需要鉴别，使用**共享对称密钥**，如果第三方C截获了A发给B的验证信息，并未破译报文，而是直接伪装成A给B发消息，这样B就会错误地与C建立起联系。把以前窃听到的数据原封不动地重新发送给接收方，这就叫 **重放攻击**。

> [重放攻击_百度百科](https://baike.baidu.com/item/重放攻击/2229240) 
>
> 重放攻击的基本原理就是把以前[窃听](https://baike.baidu.com/item/窃听/1624599?fromModule=lemma_inlink)到的数据原封不动地重新发送给接收方。很多时候，网络上传输的数据是[加密](https://baike.baidu.com/item/加密/752748?fromModule=lemma_inlink)过的，此时窃听者无法得到数据的准确意义。但如果他知道这些数据的作用，就可以在不知道数据内容的情况下通过再次发送这些数据达到愚弄接收端的目的。例如，有的系统会将鉴别信息进行简单加密后进行传输，这时攻击者虽然无法窃听[密码](https://baike.baidu.com/item/密码/65553?fromModule=lemma_inlink)，但他们却可以首先截取加密后的口令然后将其重放，从而利用这种方式进行有效的攻击。再比如，假设网上存款系统中，一条消息表示用户支取了一笔存款，攻击者完全可以多次发送这条消息而偷窃存款。

因此采取的措施就是尽量能够让B端验证清楚A的身份。K(X)表示用密钥加密过的X

A先发身份信息 + 一个大随机不重复数`RA`，B收到后响应一个`RB`和`K(RA)`，因此对于A端而言B端是可信的；A端再发送`K(RB)`，因此对于B端而言A端是可信的。需要注意的是每次会话都都需要重新验证，并且需要有足够的空间存储不同会话的不同随机数。

#### 非对称密钥

对于上文的不重复随机数，B可以用B自己的私钥加密RA，A用B的公钥解密得出RA；A用自己的私钥加密RB，B用A的公钥解密RB。这里C可以生成自己的私钥和公钥，分别截获RA和RB，仍然能够冒充A，与B进行通信。不过A与B根本没有进行通信，A端很容易察觉到。

**中间人攻击**：假设AB通信双方并没有提前知道对方的公钥是什么。

![image-20241129155621051](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241129155621051.png)

C截获A的身份验证信息和RB，用自己的私钥加密RB得到D~C~(RB)返回给B，同时把RB转发给A，A向对方索取公钥，C就把自己的公钥发给A，但是仍然把公钥的请求转发给A，A发出了自己的D~A~(RB)和公钥，但均被C所截获。

A跟B开始建立通信，B就把发送的数据用C的公钥加密，C直接可以用自己的私钥解密，解密出来以后再用A的公钥加密，发给A，看似A和B建立了保密通信，实际上C能够神不知鬼不觉地窃取信息。

> SSL劫持
>
> 当今绝大部分网站采用HTTPS方式进行访问，也就是用户与网站服务器间建立[SSL](https://info.support.huawei.com/info-finder/encyclopedia/zh/SSL.html)连接，基于SSL证书进行数据验证和加密。HTTPS可以在一定程度上减少中间人攻击，但是攻击者还是会使用各种技术尝试破坏HTTPS，SSL劫持就是其中的一种。SSL劫持也称为SSL证书欺骗，攻击者伪造网站服务器证书，公钥替换为自己的公钥，然后将虚假证书发给用户。此时用户浏览器会提示不安全，但是如果用户安全意识不强继续浏览，攻击者就可以控制用户和服务器之间的通信，解密流量，窃取甚至篡改数据。



##  密钥分配与管理

### 对称密钥交换

#### RSA

##### 难点：大数因式分解

公开的数字：公钥D，N

![image-20241129230424076](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241129230424076-1732894652386-1.png)

难点：算出T的值，也就是要分解N。

![image-20241129230521315](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241129230521315-1732944264022-8.png)

##### 使用场景

- 使用场景较广泛，适合需要加密数据、验证签名的场合。
- 不适合直接交换大数据（效率较低）。

##### RSA 的缺陷

没有前向保密性，所有的数字（质数乘积N，公钥E）必须提前沟通好，依赖于长期的静态密钥对（公钥和私钥）来交换对称密钥，而没有使用每次会话生成的临时密钥对。在 RSA 密钥交换过程中，如果服务器的私钥被泄露，攻击者可以解密所有使用该私钥加密的对称密钥，进而恢复以前的会话数据，从而破坏了前向保密性。

#### Diffle-Hellman 密钥交换

[一文搞懂Diffie-Hellman密钥交换协议 - 知乎](https://zhuanlan.zhihu.com/p/599518034) 

##### 难点：离散对数

公开的数字：P、G、双方的公钥。

- 正向计算简单，逆向计算难（即在有限域上计算  $g^x \mod p$ 的逆运算很难）

- 依赖于离散对数的计算难度，使用的素数位数越长，安全性越高。

- 易受中间人攻击，因此通常结合认证机制（如数字签名）使用。

##### 使用场景

- 专用于安全密钥交换，通常与对称加密算法结合使用。
- 不支持数据加密或签名。

由8和19无法逆推回x与y，由$19^x \mod 23 = 8^y \mod 23$ 也无法推出x和y的值

![image-20241129230002490](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241129230002490.png)

##### ECDHE

static DH算法 -- > DHE算法 -- > ECDHE算法

**static DH** 算法里有一方的私钥是静态的，也就说每次密钥协商的时候有一方的私钥都是一样的，一般是服务器方固定，即 a 不变，客户端的私钥则是随机生成的。 于是，DH 交换密钥时就只有客户端的公钥是变化，而服务端公钥是不变的，那么随着时间延长，黑客就会截获海量的密钥协商过程的数据，因为密钥协商的过程有些数据是公开的，黑客就可以依据这些数据暴力破解出服务器的私钥，然后就可以计算出会话密钥了，于是之前截获的加密数据会被破解，所以 static DH 算法**不具备前向安全性**。

**DHE** 

- **Ephemeral（临时密钥）**：在 DHE 中，"Ephemeral" 表示每次会话都生成新的密钥对（临时密钥）。因此，即使密钥在某一时刻被泄露，它也无法用于恢复以前的通信，增强了前向保密性（forward secrecy）。

G、P可以固定，DHE 每次需要传递的数据就是计算出来的公钥，不过大数乘除性能不好，于是有了ECDHE，用ECC提高性能。

**ECDHE**（Elliptic Curve Diffie-Hellman Ephemeral） 是 Diffie-Hellman 协议的椭圆曲线版本，使用椭圆曲线加密（ECC，Elliptic Curve Cryptography）来代替传统的基于大数的计算。椭圆曲线加密在相同的安全级别下，所需的密钥长度比传统的 DH 小得多，从而提高了计算效率。

#### 密钥分发中心（KDC）

Key Distribution Center

### 公钥体制：CA

[Certificate Authority 证书权威机构](https://zh.wikipedia.org/wiki/%E5%85%AC%E9%96%8B%E9%87%91%E9%91%B0%E8%AA%8D%E8%AD%89)

除非A与B线下沟通好公钥，不然总是有可能会被中间人攻击（即中间人会截获请求者的公钥，替换成自己的公钥），那么如何保证B的公钥能准确无误传到A手上不被篡改呢？需要第三方进行数字签名，CA用自己的私钥将报文信息数字签名，也就是MAC报文鉴别码。最终报文鉴别码附在报文后面$B + D(H(B))$

这里的报文B包含：

- B的身份验证信息（我是B）
- B的公钥（我的公钥是xxx）

因此，B需要提供证书，A需要把证书的报文部分进行哈希运算，然后用报文中B提供的公钥进行E运算，看得出的结果是否相同，如果不同说明证书被篡改，不可信。一切的基础是建立在CA上面，CA是一个大家都认可的权威机构

#### CA 标准：X.509（PKI）

[公开密钥认证 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/公開金鑰認證) Public Key Infrastructure

**数字证书标准**：

- X.509 版本
- 数字证书名称、序列号
- 本数字签名证书使用的签名算法（CA进行数字签名的非对称算法）
- 数字签名的公钥（私钥由签发者保存）
- 签发者的唯一标识符
- 数字证书的有效期
- 数字证书的主体名（数字证书拥有者及其拥有的公钥的唯一标识符）

#### CA 信任链：多级认证系统

万一中级证书不能信任了，还可以让根证书再找一个中级证书，因为信任根证书，也自然信任这个新的中级证书，但如果根证书直接信任某个网站的证书，万一根证书被攻破不能信任了，那就找不到可以信任的了。

![image-20241129222025691](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241129222025691.png)

[根证书](https://zh.wikipedia.org/wiki/根证书)：所有信任链的起点，使用私钥自签。根证书获得广泛认可，通常已预先安装在各种软件（包括[操作系统](https://zh.wikipedia.org/wiki/操作系统)、[浏览器](https://zh.wikipedia.org/wiki/网页浏览器)、[电邮软件](https://zh.wikipedia.org/wiki/電子郵件用戶端)等），作为[信任链](https://zh.wikipedia.org/wiki/信任鏈)的起点，来自于公认可靠的[政府机关](https://zh.wikipedia.org/wiki/政府机关)（如[香港邮政](https://zh.wikipedia.org/wiki/香港郵政)、[台湾网络信息中心](https://zh.wikipedia.org/wiki/台灣網路資訊中心)）、[证书颁发机构](https://zh.wikipedia.org/wiki/证书颁发机构)公司（如[DigiCert](https://zh.wikipedia.org/wiki/DigiCert)、[Google](https://zh.wikipedia.org/wiki/Google)）、非营利组织（如[Let's Encrypt](https://zh.wikipedia.org/wiki/Let's_Encrypt)）等，与各大软件商透过严谨的核认程序才在不同的软件广泛部署。由于部署程序复杂费时，需要行政人员的授权及机构法人身份的核认，一张根证书有效期可能长达二十年以上。在某些企业，也可能会在内部电脑自行安装企业自签的根证书，以支持[内部网](https://zh.wikipedia.org/wiki/内部网)的[企业级软件](https://zh.wikipedia.org/wiki/企业级软件)；但是这些证书可能未被广泛认可，只在企业内部适用。

**中介证书**：认证机构的一个重要任务就是为客户签发证书，虽然广泛认可的认证机构都已拥有根证书，相对应的私钥可用以签署其他证书，但因为[密钥管理](https://zh.wikipedia.org/wiki/密钥管理)和行政考虑，一般会先行签发中介证书，才为客户作数字签署。中介证书的有效期会较根证书为短，并可能对不同类别的客户有不同的中介证书作分工。

**TLS 服务器证书**：服务器通常以域名形式在互联网上提供服务，服务器证书上**主体**的**通用名称**就会是相应的域名，相关机构名称则写在**组织**或**单位**一栏上。服务器证书（包括公钥）和私钥会安装于服务器（例如[Apache](https://zh.wikipedia.org/wiki/Apache_HTTP_Server)），等待客户端连接时[协议加密细节](https://zh.wikipedia.org/wiki/握手_(技术))。客户端的软件（如浏览器）会执行[认证路径验证算法](https://zh.wikipedia.org/w/index.php?title=認證路徑驗證算法&action=edit&redlink=1)以确保安全，如果未能肯定加密通道是否安全（例如证书上的主体名称不对应网站域名、服务器使用了自签证书、或加密算法不够强），可能会警告用户。

**TLS 客户端证书**：有时候，某些TLS服务器可能会在建立加密通道时，要求客户端提供客户端证书，以验证[身份](https://zh.wikipedia.org/wiki/数字身份)及[控制访问权限](https://zh.wikipedia.org/wiki/存取控制)。客户端证书包含电子邮件地址或个人姓名，而不是主机名。但客户端证书比较不常见，因为考虑到技术门槛及成本因素，通常都是由服务提供者验证客户身份，而不是依赖第三方认证机构。通常，需要使用到客户端证书的服务都是内部网的企业级软件，他们会设立自己的内部根证书，由企业的技术人员在企业内部的电脑安装相关客户端证书以便使用。在公开的互联网，大多数网站都是使用[登录密码](https://zh.wikipedia.org/wiki/密碼_(認證))和[Cookie](https://zh.wikipedia.org/wiki/Cookie)来验证用户，而不是客户端证书。客户端证书在[RPC系统](https://zh.wikipedia.org/wiki/遠程過程調用)中更常见，用于验证连接设备的许可授权。

[自签证书](https://zh.wikipedia.org/wiki/自签证书)：在用于小范围测试等目的的时候，用户也可以自己生成数字证书，但没有任何可信赖的人签名，这种自签名证书通常不会被广泛信任，使用时可能会遇到电脑软件的安全警告。

#### CA 撤销名单

CA 撤销名单：尚未到期就被[证书颁发机构](https://zh.wikipedia.org/wiki/证书颁发机构)吊销的[数字证书](https://zh.wikipedia.org/wiki/数字证书)的名单。

- 吊销：该证书被不可逆的吊销。例如，它被不当的[证书颁发机构](https://zh.wikipedia.org/wiki/证书颁发机构)颁发了证书，或者私钥被认为已经破坏。被吊销的最常见的原因是用户不再独有私钥，而私钥则被窃取。
- 吊扣：这个状态是可逆的。

每个CA都有对应的CA证书撤销名单，里面包含着证书的序列号。有 [OCSP](https://zh.wikipedia.org/wiki/%E5%9C%A8%E7%BA%BF%E8%AF%81%E4%B9%A6%E7%8A%B6%E6%80%81%E5%8D%8F%E8%AE%AE) 和 [CRL](https://zh.wikipedia.org/wiki/证书吊销列表) 两种形式

> 非关键 CA 颁发者: URI: http://secure.globalsign.com/cacert/gsrsaovsslca2018.crt 
>
> OCSP 响应者: URI: http://ocsp.globalsign.com/gsrsaovsslca2018

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/crl.png)

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/ocsp.png)

# HTTPS (SSL/TLS)

[HTTPS  HTTP](https://xiaolincoding.com/network/2_http/http_interview.html#http-%E4%B8%8E-https) 

## 传输层安全（TLS）

Transport Layer Security

### TLS 握手（TLS 1.2）

在TCP threeway handshake之后，就会开始TLS handshake

两种交换会话密钥的算法：[RSA 握手](https://xiaolincoding.com/network/2_http/https_rsa.html#rsa-%E6%8F%A1%E6%89%8B%E8%BF%87%E7%A8%8B) 和 [ECDHE 握手](https://xiaolincoding.com/network/2_http/https_ecdhe.html) 后者支持前向安全，现在使用更加广泛

#### RSA 握手

![image-20241130161720294](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241130161720294.png)

1. **ClientHello**: 客户端向服务器发送随机数`client random`，TLS版本，支持的加密套件列表。
2. **ServerHello**: 服务器响应随机数`server random`，确认好加密套件，下发服务器证书(`Certificate`)。
   - 证书里有用于`premaster`加密的服务器公钥
3. **Client Key Exchange**: 客户端证书验证通过后，生成另一个随机数`premaster secret`，通过**服务器证书的公钥**加密
4. 服务器用私钥解密获取`premaster secret`，双方根据`premaster secret`和两个随机数生成`session key` 
5. **Change Cipher Spec**: 客户端通知接下来要使用会话密钥进行通信了，之前都是明文通信。切换加密标准
6. **Finishd**: 客户端计算之前发出的明文消息的摘要（Hash），再用`session key`加密后发给服务端
7. 服务端重复5,6步，双方认证加解密无问题，则可以开始正式发送用`session key`加密后的`application data` 

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/e6zkf4qwi7.jpeg)

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/https_rsa.png) 

##### 加密套件（Cipher Suite）

[TLS 各种加密套件_tls加密套件-CSDN博客](https://blog.csdn.net/weixin_43408952/article/details/124715927) 

**TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256** 

- **密钥交换策略**（Key Exchange/Arrangement）：用于交换对称密钥。**RSA**、DH、DHE、**ECDHE**、PSK
- **数字签名算法**（Authentication）：用于验证证书。**RSA**、[**DSA**](https://zh.wikipedia.org/wiki/数字签名算法) 、ECDSA
- **对称加密算法**（Block/stream ciphers）：用于加密消息流。**ChaCha20**、**AES**、DES等
- **MAC 算法**（Message authentication）：用于创建报文鉴别码，例如**SHA-256**，MD5，消息流每个[数据块](https://zh.wikipedia.org/wiki/数据块)的[加密散列](https://zh.wikipedia.org/wiki/加密散列)。(分块加密信息以后的报文摘要)

##### 前向安全性（Forward Secrecy）

共享密钥被攻破不会导致之前的会话信息全部泄露。Perfect Forward Secrecy

[如何理解前向安全性？和完美前向保密（perfect forward secrecy）区别？ - 知乎](https://www.zhihu.com/question/45203206) 

[有了共享密钥为什么还需要会话密钥？ - 知乎](https://www.zhihu.com/question/348420897) 

**有了premaster key为什么要session key？** 

RSA握手中不把premaster key直接当做对称密钥，单次session key泄露不会造成之前的会话信息都泄露。

前向安全性问题出在共享密钥上，RSA握手的共享公-私密钥对是长期不变的，也就是说如果服务端用于RSA加密的私钥泄露会导致之前的会话信息全部暴露。如果对每次会话都生成一对RSA密钥对，理论可行，但是性能不如后面要介绍的ECDHE。

 双**RSA**虽然也能实现**PFS**，但是效率太差，没有公司会采用， 基本都是**RSA + ECDHE**。 

#### ECDHE 握手

椭圆曲线最重要的参数是**椭圆曲线类型**（基点G）[RSA 算法的替代品：X25519/Ed25519 使用记录 | 存在感消失的地方|ω•`)](https://akarin.dev/2021/09/16/a-taste-of-curve25519/)  

![image-20241130165130228](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241130165130228.png)

1. **ClientHello**: 客户端向服务器发送**随机数**client random，TLS版本，支持的加密套件列表
2. **ServerHello**: 服务器响应**随机数**server random，确认好双方都支持的加密套件，同时下发服务器**证书**(`certificate`)
   - 证书中的公钥用于鉴别自己发出的签名有效。
3. **Server Key Exchange**: 生成随机数作为**临时私钥**，保留在本地。公开**椭圆曲线**基点G，一般是X25519，根据G和临时生成的私钥，算出**公钥**发给客户端。为了保证公钥不被篡改，同时会使用RSA进行数字签名。
4. **Client Key Exchange**: 客户端验证通过后，生成自己临时私钥，根据基点G算出**公钥**，发送给服务器。
5. 这样，双方知道了对方的公钥，就可以开始算共享密钥了。
6. 之后的步骤就是互相发送change cipher spec+finished，ECDHE可以在客户端发完信息之后可以直接开始发送`application data` 

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/0mhr8kq63w.jpeg)

==抓包实战== 

<u>搭建https server（springboot）</u>

1. **生成自签名证书**，使用jdk的keytool生成证书

```vbnet
keytool -genkey -alias wxl -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore wxl-ssl-key.p12 -validity 3650
```

- -genkey：表示要创建一个新的密钥
- -alias：keystore别名
- -keyalg：加密算法
- -keysize：密钥长度
- -storetype：密钥类型
- -keystore：文件存放位置
- -validity：密钥有效期，单位为天

2. **springboot配置https**

```yml
server:
  port: 8443
  ssl:
    enabled: true
    key-store: classpath:wxl-ssl-key.p12
    key-store-password: 123456
    key-store-type: PKCS12
    enabled-protocols:
      - TLSv1.2
```

- enabled-protocols表示支持启用的TLS版本，这里配置仅TLS1.2

<u>请求并抓包</u>

```bash
curl https://localhost:8443/hello -k
```

tcp.port == 8443

##### TLS False Start

[https - TLS False Start究竟是如何加速网站的 - 野狗科技官方专栏 - SegmentFault 思否](https://segmentfault.com/a/1190000004003319) 

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/0mhr8kq63w.jpeg)

> The recommended whitelists are such that if cryptographic algorithms suitable for forward secrecy would possibly be negotiated, no False Start will take place if the current handshake fails to provide forward secrecy.
>
> [RFC 7918 - Transport Layer Security (TLS) False Start](https://datatracker.ietf.org/doc/html/rfc7918) 

ECDHE和DHE支持前向保密，所以可以使用TLS抢跑。

[90%的人都不懂的TLS握手优化-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1420297) 

### TLS 记录

TLS 握手主要用来解决服务器的可信度问题，TLS 记录可以解决报文的压缩、加密和数据认证的问题。

![image-20241130224715135](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241130224715135.png)

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/%E8%AE%B0%E5%BD%95%E5%8D%8F%E8%AE%AE.png)

### TLS 漏洞

------

<u>**协议设计上的漏洞**</u>：一些漏洞源于TLS协议本身的设计缺陷，通常会在新版本中修复：

**BEAST（Browser Exploit Against SSL/TLS）**

- **描述**：BEAST 攻击利用 TLS 1.0 中的 CBC 模式实现的设计缺陷，通过中间人攻击窃取加密数据。
- **修复**：TLS 1.1 和更高版本已经修复了该问题，同时推荐使用 AES-GCM 等替代的加密模式。

**CRIME（Compression Ratio Info-leak Made Easy）**

- **描述**：通过利用TLS压缩功能，攻击者可以猜测敏感数据（如会话Cookie）。
- **修复**：禁用TLS压缩功能。

**POODLE（Padding Oracle On Downgraded Legacy Encryption）**

- **描述**：POODLE 利用 SSL 3.0 中的填充漏洞进行攻击，针对使用CBC模式的实现。
- **修复**：废弃 SSL 3.0 并采用更安全的协议版本（如 TLS 1.2+）。

**Downgrade Attacks**

- **描述**：攻击者通过中间人攻击强制客户端和服务器降级到不安全的协议版本（如 SSL 3.0 或早期 TLS 版本）。
- **修复**：使用 **TLS_FALLBACK_SCSV** 标记防止协议降级攻击。

**Logjam**

- **描述**：Logjam 攻击利用 Diffie-Hellman 密钥交换中的弱参数（512位素数），允许攻击者破解加密。
- **修复**：升级到更强的密钥（2048位或以上），并禁用弱算法。

------

<u>**实现上的漏洞**</u>：许多漏洞是由于TLS库实现中存在的错误或疏漏，而不是协议本身的问题。

**Heartbleed**

- **描述**：这是 OpenSSL 的实现漏洞，允许攻击者通过 Heartbeat 扩展读取服务器内存中的敏感数据（如私钥）。
- **修复**：修复受影响的 OpenSSL 版本，更新到无漏洞的版本。

**ROBOT（Return Of Bleichenbacher’s Oracle Threat）**

- **描述**：利用某些TLS实现中的 RSA 加密模式缺陷，通过构造恶意数据包破解私钥。
- **修复**：修复实现并采用更安全的加密模式。

**Zero-Length Padding**

- **描述**：某些TLS实现接受零长度的填充，可能被攻击者利用进行漏洞利用。
- **修复**：严格遵循协议规范，确保填充字段符合要求。

------

<u>**配置和使用上的问题**</u>：即使TLS协议和实现没有漏洞，不当的配置或使用也可能导致安全隐患。

**弱密码套件**

- 使用已知不安全的加密算法或过短的密钥长度（如RC4或1024位RSA）。
- **解决方法**：禁用弱密码套件，使用推荐的安全算法（如 AES-GCM、ChaCha20-Poly1305）。

**过期或伪造的证书**

- 如果服务器使用过期、伪造或不可信的证书，攻击者可以进行中间人攻击。
- **解决方法**：确保证书可信且未过期，并使用 Certificate Transparency 来检测伪造证书。

**主机名验证问题**

- 一些TLS实现未正确验证证书中的主机名，可能导致攻击者伪装成合法服务器。
- **解决方法**：强制严格的主机名验证。

**会话恢复漏洞**

- TLS会话恢复功能（如会话ID或会话票据）在设计或实现上可能存在漏洞，导致会话劫持。
- **解决方法**：确保会话恢复机制的实现安全，并定期刷新密钥。

------

<u>**环境相关漏洞**</u>

**硬件漏洞**

- 硬件加速器或HSM（硬件安全模块）可能存在漏洞，攻击者可以利用其生成弱密钥或泄露数据。

**随机数生成问题**

- 如果随机数生成器的质量不足，攻击者可能预测到密钥。
- **解决方法**：使用高质量的随机数生成器（如 /dev/urandom 或硬件随机数生成器）。

**中间人攻击**

- 攻击者通过篡改DNS或ARP欺骗迫使客户端连接到恶意服务器，伪装成合法的TLS服务。
- **解决方法**：使用 HSTS（HTTP Strict Transport Security）和证书锁定（Certificate Pinning）。

------

<u>**社会工程和弱安全操作**</u>：即使TLS协议本身非常安全，攻击者可能利用社会工程或操作疏漏来绕过安全机制

- 攻击者诱骗用户接受不可信的证书。
- 管理员错误配置服务器，允许使用过时的协议版本。

------

#### 如何应对TLS漏洞？

1. **升级协议和实现**
   - 确保使用最新版本的TLS（推荐TLS 1.2或TLS 1.3）。
   - 定期更新TLS库（如OpenSSL、BoringSSL、GnuTLS）。
2. **禁用弱配置**
   - 禁用SSL 2.0、SSL 3.0和TLS 1.0。
   - 禁用RC4和其他已知不安全的密码套件。
3. **安全的服务器配置**
   - 强制使用强加密算法。
   - 使用2048位以上的密钥和推荐的椭圆曲线（如 P-256）。
4. **监控和检测**
   - 定期扫描服务器的TLS配置，使用工具如 SSL Labs 的服务器测试工具。
   - 监控网络流量中的潜在攻击行为。

### TLS 1.3

![图片](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/0877fe78380bf34ad3b28768e59fb53a.png)

[TLS 1.3科普——新特性与协议实现 - 知乎](https://zhuanlan.zhihu.com/p/28850798)

[TLS 1.3 进行时 - 知乎](https://zhuanlan.zhihu.com/p/187262056) 

#### 2-RTT（TLS 1.2）

[TLS1.2握手流程分析（RSA，ECDHE），和TLS1.3区别 - wuworker - 博客园](https://www.cnblogs.com/wusanga/p/17386098.html) 

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-485b564d2209a3108575a1b13a52d715_1440w.jpg)

#### 1-RTT

TLS1.2 为了考虑各种兼容性，保留了许多加密套件，这就使得客户端必须提前和服务端协商好用哪一种加密套件，这也导致了必须空出一个RTT专门协商，因此变成2-RTT。1.3只剩下5种，省去协商步骤，变成了1-RTT。

```
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
TLS_AES_128_GCM_SHA256
TLS_AES_128_CCM_8_SHA256
TLS_AES_128_CCM_SHA256
```

TLS 1.3 在之前版本的基础上删除了那些不安全的加密算法，这些加密算法包括：

- RSA 密钥传输 —— 不支持前向安全性
- CBC 模式密码 —— 易受 BEAST 和 Lucky 13 攻击
- RC4 流密码 —— 在 HTTPS 中使用并不安全
- SHA-1 哈希函数 —— 建议以 SHA-2 取而代之
- 任意 Diffie-Hellman 组—— CVE-2016-0701 漏洞
- 输出密码 —— 易受 FREAK 和 LogJam 攻击

TLS 1.3 只支持[AEAD认证模式](https://zh.wikipedia.org/wiki/%E8%AE%A4%E8%AF%81%E5%8A%A0%E5%AF%86) 同时完成加密和完整性校验，不再允许对加密报文进行压缩、不再允许双方发起重协商，密钥的改变不再需要发送change_cipher_spec报文给对方。对称加密算法只有AES，Chacha20，摘要算法只有 SHA，密钥交换算法只有ECDHE。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-91669b8728eb5b0fa2d88730425f9391_1440w.jpg)

#### 0-RTT

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/v2-083c00146d71e75adbcab401e57c90e1_1440w.jpg)

![图片](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/59539201f006d7dc0a06333617e5ea85.png)

## [HTTPS 优化（TLS 性能优化）](https://cloud.tencent.com/developer/article/1420297) 

上文的TLS漏洞已经说明了提高安全性的措施，下列措施为提高TLS性能的措施

### 硬件优化

- 计算密集型任务，升级的是**CPU**而不是网卡等IO设备，如果CPU有针对 **AES-NI** 的特性，可以使用AES，否则可以选择chacha20

### 软件优化

- 升级Linux，升级OpenSSL

### 协议优化：节省RTT

（整体的握手过程还是需要的）

- **密钥交换算法**：使用`ECDHE`而不是`RSA`，`ECDHE`支持前向安全，因此也支持 `TLS False Start`，发送ClientKeyExchange的同时可以开始发送`application data`节省 `1 RTT`。
- **升级TLS版本**：TLS 1.3 相比于 TLS 1.2 废除了不安全的加密套件，总数变少，因此不需要协商套件的过程，可以节省`1 RTT`。

### 证书优化：节省客户端验证时间

- **证书类型**：在相同的安全强度下，`ECDSA` 椭圆曲线证书相比于 `RSA` 证书的密钥长度减少很多，减少了验证证书完整性的时间。
- **证书验证流程**：验证证书信任链的时候，`OCSP` 相比于 `CRL` 实时性更高，不用逐行读取文件；
  - 进一步提升性能可以启用`OCSP Stapling`，服务器周期性地向 CA 获取证书状态，CA会在状态上签名防止篡改，在发出ServerHello的同时也把有效信息发给客户端。

### 会话复用：复用会话密钥

[TLS/SSL 协议详解 (22)会话复用_ssl会话复用-CSDN博客](https://blog.csdn.net/mrpre/article/details/77868669) 抓包

**以下技术都是建立在已经建立过一次连接的基础上的，可以节省第一次以后会话的RTT，是用来免去握手过程的**

- **Session ID**：将与每个客户端的会话密钥缓存在服务器内存里，形成 id-key的键值对形式，ClientHello带上session id，节省`1 RTT`，缺点是不支持分布式和消耗服务器内存。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/sessionid.png)

- **Session Ticket**：服务器可以将会话密钥再次用**只有自己知道的密钥**加密，然后附上有效期进行签发，以session ticket的形式交给客户端进行缓存，ClientHello带上ticket，服务器验证有效期（类似JWT）也能节省 `1 RTT` ，分布式之间需要共享这个密钥。

![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/ticket.png)

- **Pre-shared Key**：TLS 1.3 引入的新特性，将 Session Ticket 和 早期的 application data 一并发送给服务器 直接变成`0-RTT` ，因为不用协商套件。

![image-20241130172210036](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241130172210036.png)

会话复用会影响前向安全性，还可能会受到重放攻击，解决方案：

- 只对幂等请求（GET）开放0-RTT；
- 对session ticket添加有效期；

# SSH

**[SSH](https://zh.wikipedia.org/wiki/Secure_Shell) （Secure Shell）** 基于 TCP 协议，通过加密和认证机制实现安全的访问和文件传输等业务。

[SSH 握手详解 - bilibili 技术蛋老师](https://www.bilibili.com/video/BV13P4y1o76u) 

SSH 的经典用途是登录到远程电脑中执行命令。除此之外，SSH 也支持隧道协议、端口映射和 X11 连接（允许用户在本地运行远程服务器上的图形应用程序）。借助 SFTP（SSH File Transfer Protocol） 或 SCP（Secure Copy Protocol） 协议，SSH 还可以安全传输文件。

SSH 使用客户端-服务器模型，默认端口是 22。SSH 是一个守护进程，负责实时监听客户端请求，并进行处理。大多数现代操作系统都提供了 SSH。

如下图所示，SSH Client（SSH 客户端）和 SSH Server（SSH 服务器）通过公钥交换生成共享的对称加密密钥，用于后续的加密通信。

![SSH:安全的网络传输协议](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/ssh-client-server.png)

SSH以[非对称加密](https://zh.wikipedia.org/wiki/非对称加密)实现[身份验证](https://zh.wikipedia.org/wiki/身份验证)：

身份验证有多种途径，例如其中一种方法是使用自动生成的公钥-私钥对来简单地加密网络连接，随后使用密码认证进行登录；另一种方法是人工生成一对公钥和私钥，通过生成的密钥进行认证，这样就可以在不输入密码的情况下登录。任何人都可以自行生成密钥。公钥需要放在待访问的电脑之中，而对应的私钥需要由用户自行保管。认证过程基于生成出来的私钥，但整个认证过程中私钥本身不会传输到网络中。



# IPsec

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/download" alt="IPsec加密验证过程" style="zoom:150%;" />

