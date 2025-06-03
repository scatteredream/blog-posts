---
name: web-back-end
title: Web 后端 Java
date: 2024-09-05
tags: 
- http
- servlet
- tomcat
- 设计模式
- json
- jwt
- session&cookie
categories: web development
---



# HTTP

**H**yper**T**ext **T**ransfer **P**rotocol 

TCPIP协议四层架构的最上层 规定了服务器和浏览器之间传输数据的规则

[HTTP | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)

[超文本传输协议 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/超文本传输协议) 

## 特点

- 基于TCP，面向连接，安全
- 基于请求-响应模型：1Request 1Response
- HTTP是无状态协议，对事务处理没有记忆能力，每次RR都是独立的
  - 多次请求之间不能共享数据，java用会话技术（cookie session）解决这个问题
  - 优点：速度快

## <a href="#request">请求数据格式</a>

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

## 响应数据格式

**响应行**：响应数据的第一行，HTTP/1.1表示协议版本，下一个是响应状态码，OK表示状态码描述

**响应头**：keyvalue

**响应体**：最后一部分，存放响应数据

![image-20240926221630980](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926221630980.png)

**Content-Type**：响应内容类型，比如text/html image/jpeg

**Content-Length**：响应内容长度（bytes）

**Content-Encoding**：响应压缩算法 gzip等

**Cache-Control**: 指示客户端如何缓存，例如max-age=300 表示最多缓存300s

## 状态码

![image-20240926222005460](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926222005460.png)

![image-20240926222320894](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926222320894.png)

200 OK 404资源不存在 500 服务器异常



Java程序中，如果直接用自带的javawebsocketAPI 代码会变得异常繁琐，要注意请求和响应的格式要求，因此要用web服务器软件进行开发----Tomcat



# Tomcat

Apache Tomcat web服务器是一个应用程序，封装http协议，不用对协议进行直接操作。类似的还有jetty，weblogic，ibm webSphere，部署web项目到服务器中

[JavaEE的13种核心技术规范： - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/61596145)

[Apache Tomcat® - Welcome!](https://tomcat.apache.org/) 

## 创建Web项目

![image-20240927151450747](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927151450747.png)

```lua
Web->META-INF()
     WEB-INF ->classes(java文件夹和resources文件夹合并)
     	     ->lib(依赖jar包)
             ->web.xml(web项目的配置文件)
     webapp中除了WEB-INF的其他文件
```



using 骨架

![image-20240927151529348](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927151529348.png)

packaging 默认jar 改成web项目用的war 

# Servlet

动态资源web开发技术

 ![image-20240927162801770](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927162801770.png)



## 入门

![image-20240927163054997](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927163054997.png)

servlet对象，service方法由web服务器tomcat创建

WebServlet继承了Servlet接口

## 生命周期

![image-20240927174147654](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927174147654.png)

1. 默认情况 servlet对象第一次被访问就被创建，通过改变loadOnStartup参数可以改变优先级
2. 容器（tomcat）通过init方法初始化对象，只需要调用一次
3. 每次请求servlet 容器都会调用servlet的service方法

## Servlet 接口

定义了五个抽象方法：

1. `init(ServletConfig conf)`：默认情况下，servlet第一次被访问，容器创建servlet对象时，会调用init，只调用一次。改变WebServlet注解的参数loadOnStartup，可以控制在创建服务器的时候就创建servlet对象。

2. `service(ServletRequest req,ServletResponse res)`: 每一次访问servlet就调用一次

3. `destroy()`: 内存释放、服务器关闭时调用，只有一次

4. `ServletConfig getServletConfig()`: servletconfig是容器调用init方法传进来的参数,可以在demo类中声明一个config成员变量，在init中赋值，然后在getConfig方法中返回

5. `String getServletInfo() `: copyright information

## Servlet 体系

![image-20240927180225035](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927180225035.png)

Http doGet doPost

根据请求方式的不同分别处理，因为get的参数在请求行中，post的参数在请求体中。

httpservlet是servlet的实现类，实际上把service方法重写，接收请求参数req，如果req中是get方式，就执行doGet，如果是post就执行doPost  子类只需要重写doGetdoPost方法即可

![image-20240927181319844](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927181319844.png)



源码分析：原来的service方法重载，参数变成httpservletrequest和httpservletresponse ，原版的请求参数传进来，强制转换成httpservletrequest，然后吊用自己写好的重载service方法

![image-20240927181753064](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927181753064.png)

![image-20240927181825881](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927181825881.png)

## urlPattern

### 一个servlet可以配置多个访问路径

urlPatterns = {"",""}

### 配置规则

![image-20240927183113407](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927183113407.png)

1. 精确匹配 （优先级比目录匹配高）
2. 目录匹配，通配符
3. 扩展名匹配，`*.do` `aaa.do bbb.do`都可以访问，注意不能有斜杠
4. 任意匹配，`/`优先级低于`/*`
   - /是tomcat默认生成的一个servlet，启动以后自动创建，是用来访问静态资源的
   - 很危险，不要用

![image-20240927183710215](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927183710215.png)

### web.xml配置

![image-20240927183912074](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927183912074.png)

# Request&Response

![image-20240927184223066](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927184223066.png)

request 获取请求数据

response 设置响应时的数据

request中含有用户输入的参数，response可以根据这个参数设置响应的数据，这样就完成了和用户交互的基本过程

## Request 

### 继承体系

![image-20240927184637027](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927184637027.png)

ServletRequest和HttpServletRequest都是接口，不能实例化，定义了一些抽象方法作为规范。

我们的 servlet 重写了 service()方法的方法体，浏览器访问时，tomcat就要调用servlet的service方法。tomcat作为servlet容器，**要解析请求报文，将其封装成req对象**，送到servlet的service方法处作为参数。

```java
//这是tomcat的程序
//tomcat做的是解析报文封装请求的操作，具体如何利用请求做出什么样的响应，则是开发者的工作
MyServlet myServlet = new MyServlet();
myServlet.init()//开发者重写
myServlet.service(req,res)//开发者重写
```

所以Tomcat对接口进行了实现，查J2EE API

### 获取请求数据

#### 请求行

![image-20240927192106826](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927192106826.png)![image-20240927192310996](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927192310996.png)

#### 请求头

![image-20240927192340229](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927192340229.png)

getHeader根据name来获取对应的信息

getHeader("User-Agent") 输出Mozilla/5.0 Chrome/91.0.4472.106

#### 请求体

![image-20240927192426370](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927192426370.png)

统一获取请求参数的方式？从而统一doGet和doPost方法内的代码

![image-20240927195026912](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927195026912.png)

getParameterMap 获取参数map 结构如上

getParameter 根据名称获取参数值

getParameterValues 根据名称获取参数值

#### 获取参数中文乱码

##### POST

设置输入流的字符集

底层是获取字符输入流BufferReader，所以`setCharacterEncoding("UTF-8")`

##### GET

底层是字符串形式

浏览器发出请求的时候，会把中文字符转成URL编码，tomcat需要进行URL解码

##### URL编码

字符串按照编码方式转为二进制，每个字节转换为两个十六进制数，在前面加上%                                            

![image-20240927214608765](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240927214608765.png)

Tomcat底层将URL编码 解码为ISO-8859-1

### Forward 请求转发

服务器内部资源跳转方式，转发的资源之间共享数据

![image-20240928140710653](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928140710653.png)

![image-20240928140736371](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928140736371.png)

请求内部有参数（map形式），来源URL等信息

- `setAttribute(String name, Object o)`把数据o 存到request域中，以key为键	
- `removeAttribute(String name)` 根据key删除键值对
- `Object getAttribute(String name)`根据key获取数据



- 地址栏路径不发生变化；
- 只能转发到服务器内部的资源；
- 浏览器发送一次请求，多个资源共享request数据
- 高效率

## Response

![image-20240928142749615](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928142749615.png)

### 设置响应数据

1. 响应行：设置状态码
2. 响应头：设置键值对
3. 响应体：通过输出流输出数据

![image-20240928142944677](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928142944677.png)

### Redirect 重定向

资源跳转方式

![image-20240928143039892](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928143039892.png)

状态码 **302** 响应头：location: 虚拟目录/demo6

`sendRedirect(String url)`发送重定向响应

`setStatus(302)` 

`setHeader("location","https://www.google.com")` 

`setHeader("Content-type","text/html")` 

#### 特点：

- 地址栏路径发生变化；
- 转发任意资源；
- 浏览器发送两次请求，不能在多个资源用request共享数据
- 效率低

#### 路径问题 动态获取虚拟目录

如果浏览器使用，需要加虚拟目录

服务端使用就不需要加了

![image-20240928145151848](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928145151848.png)

虚拟目录可以动态变化，所以尽量减少硬编码，减少耦合性

可以用`request`的`getContextPath()` 获取虚拟目录

### 设置响应数据

#### 字符数据

`getWriter().write(String s) `写入数据到资源中

`setContentType("text/html;charset=utf-8")`   

细节：不用关闭流

乱码可以用响应头设置编码，tomcat8不乱码

#### 字节数据

ServletOutputStream = request.getOutputStream()

## SqlSessionFactory

用来创建与数据库的连接会话,只需要一个即可，所以运用单例的设计模式

```java
public class SqlSessionFactoryUtils{
    SqlSessionFactory factory;
    static{
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory =
                new SqlSessionFactoryBuilder().build(inputStream);
    }
    SqlSessionFactory getSqlSessionFactory(){
        return factory;
    }
}
```

# JSP

## 入门

Java Server Pages 静态的页面嵌入动态的代码 简化开发

![image-20240928175143005](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928175143005.png)

JSP本质是servlet，把写标签等繁琐的工作交给jsp技术



### JSP脚本

![image-20240928175642020](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928175642020.png)

1. service方法是访问到这个资源的时候调用

2. out.print() 是printWriter 调用的

3. 被生成的jsp类直接包含



截断java代码，中间插入html标签是可以的

![image-20240928180319108](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928180319108.png)

出现HTML标签的地方可以理解为java程序代替你输入这些标签，最简单的字面意义上的代替功能，因此截断也没什么关系

![image-20240928180738621](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928180738621.png)

## 缺点

![image-20240928180843640](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928180843640.png)

## EL表达式

![image-20240928181558853](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928181558853.png)

 

## JSTL 标签库

<mark>**#{}**<mark>

### if

![image-20240928182101658](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928182101658.png)

### forEach

![image-20240928182210315](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928182210315.png)

brand.id 不是访问成员变量，是要调用get方法

自动调用getId()

![image-20240928182800258](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928182800258.png)

varStatus 计数器

status.count是计数从1开始，status.index是从1开始

#### 普通for循环

![image-20240928183033816](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928183033816.png)

Java虚拟机负责存储变量信息，jsp只负责展示与变量相关的信息，因此不用重启服务器，

## Servlet+JSP开发

**业务逻辑层**本质上是对dao层的封装，包括创建session，创建mapper，mapper调用dao方法，事务等。

每个servlet都是资源，浏览器能够通过网址或者表单的形式发出请求，servlet根据request的参数进行一系列业务逻辑操作，将返回的结果转发给jsp页面。

jsp本质也是一个servlet，将请求中的参数打印出来的同时还能生成html标签，浏览器就能通过html标签解析出网页。

在修改页面改了一个数据，提交表单到updateServlet，updateServlet进行业务操作，完成后，把包含参数的请求转发到 浏览所有数据 的showAll.jsp页面，jsp本质是servlet，负责打印标签和数据。



# MVC 设计模式

**Model**：接受Controller发出的指令，与数据库交互，增删改查，返回数据给Controller

**View**：接受Controller发出的数据（Model给的）渲染页面，返回HTML页面给Controller

**Controller**：接受客户端的数据请求，返回给客户端HTML页面，同时与model和view交流，充当Model和View之间的桥梁。

Model和View之间一个是处理数据，一个是呈现数据，二者可以专注于各自的事情

![image-20240919202449018](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240919202449018-1727923726251-1.png)

Servlet controller 

JSP View 

JavaBean Model 

![image-20240928183538077](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928183538077-1727923726251-2.png)

三层架构

![image-20240928183903472](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928183903472-1727923726251-3.png)



![image-20240928183924963](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928183924963-1727923726251-4.png)





# Cookie&Session&JWT

## 会话跟踪技术

浏览器打开一个网站就是会话建立的过程，其中可以**包含多次请求和响应**，服务段需要区分不同的会话，判断多次请求是否来自统一浏览器，以便在同一次会话的**多次请求之间，共享数据。**

**HTTP协议是无状态**的，为了最佳的请求响应效率，牺牲了存储记忆数据的功能，每次请求都被视作新的请求，因此要跟踪回话实现会话内数据共享。

本质是将数据存储在一端

![image-20240928201532521](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928201532521.png)

客户端：**Cookie** 

服务端：**Session** 

## Cookie

客户端的会话技术，保存数据到客户端，每次请求都携带cookie数据进行访问。客户端的记忆

![image-20240928202227465](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928202227465.png)

响应的时候setcookie，请求的时候带着cookie

### 发送cookie

创建Cookie，设置键值对

response调用addCookie方法发送cookie

### 获取cookie

request对象调用getCookies 接收cookies

for循环遍历，getName和getValue

## cookie的原理

**基于HTTP协议**

响应的时候，做好cookie传回去，**响应头**setCookie:username=zs

![image-20240928204006874](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928204006874.png)

浏览器再次请求的时候，**请求头**中cookie:username=zs

![image-20240928204020606](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928204020606.png)

### cookie使用细节

#### 存活时间

**默认**存储在浏览器内存中，关闭浏览器会释放内存，销毁cookie

setMaxAge(int seconds) 正数：写入浏览器硬盘，到时间自动删除；负数：写入内存，自动销毁；零：删除对应cookie

30天内免登录

cookie是键值对 

#### cookie存储中文

可以把字符串用URL编码

## Session

![image-20240928205018701](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928205018701.png)

服务端的记忆功能

request.getSession();

set Attribute 存到session域中

get Attribute

session是键值对集合，存储在服务器

**Session基于Cookie实现** 

如何保证多个浏览器不是同一个session？发送一个sessionID的cookie，作为唯一标识，浏览器请求的时候会带着cookie。响应的时候创建一个session，把浏览器唯一对应的session对象id作为cookie发过去，再次请求的时候带着sessionid作为cookie就能找到对应的session对象去存储

![image-20240928210309806](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928210309806.png)

识别sessionid如果已经创建过了就不再创建

![image-20240928210431476](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928210431476.png)

![image-20240928210448553](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928210448553.png)

### 使用细节

#### session 钝化、活化

钝化：服务器正常关闭，tomcat自动把session存到硬盘

活化：服务器开启，从session文件读取

浏览器关闭后中断会话，session不是同一个

#### session 销毁

- 自动销毁web.xml sessionconfig 时间默认为30分钟
- 手动销毁：登出

## Cookie vs. Session



![image-20240928220229036](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928220229036.png)

安全性，长期存储

cookie保证用户在未登录情况下的身份识别

session存储用户登录以后的

![image-20240928225946421](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928225946421.png)

### 登录系统DEMO: 

login.jsp页面

登录成功存储用户信息，并且要重定向到另一个brand.jsp页面，两次会话共享信息，考虑安全性，session

登录失败，转发回登录页面，把错误 信息加进request域中，jsp登录页面显示的是错误信息

记住用户登录信息：登录成功并且勾选了复选框（发送复选框的value参数，Object.equals或是"1".equals（remember））创建username和password的cookie并发送到浏览器。修改login.jsp：拿到请求中的cookies，分别把响应的数据填到页面的username和password中。

![image-20240928223745507](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928223745507.png)

### 用户注册DEMO:

reg.jsp 

![image-20240928223916558](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928223916558.png)

if(布尔表达式){

}

return 布尔表达式

![image-20240928231442349](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928231442349.png)![image-20240928231455396](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928231455396.png)

展示验证码：servlet输出的验证码作为img src显示在HTML静态页面上，写js脚本把点击事件设置成重新请求一次，每次请求的路径不同 (?号后面加时间)，防止浏览器缓存

生成验证码和提交注册表单一共需要两次请求，是不同的servlet在处理，所以服务器要在生成的时候将验证码存到session中，提交注册表单的时候再次从session中访问数据看是否一致。存到cookie中会直接被抓取然后攻击，失去了验证码的功能

if else if直接return就不用else了

checkcodeServlet 生成code，输出到自己的输出流中  

## Token(<u>J</u>son <u>W</u>eb <u>T</u>oken)

[JWT详细讲解(保姆级教程)-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/995894#comment)

### token cookie session

#### Cookie

本质是键值对，客户端发起请求，服务端响应会把包含着用户信息的set-cookie 加入响应头，客户端收到set-cookie，下次发送请求，请求头中会带着包含相同内容的cookie，服务端只需要根据cookie响应对应用户的资源。

局限：数据直接存放在浏览器端内存，安全性差，

优化：把set-cookie内容除了正常的cookie内容再加一段报文鉴别码，使用服务器自己的私钥进行签名，

优点：存储期限长

#### Session

以cookie为基础，本质是一个对象，每个session可以通过唯一的sessionID进行访问，客户端发起请求，服务端会把包含着sessionID信息的set-cookie响应给客户端，客户端下次发送请求，请求头中会带着包含着sessionID信息的cookie，服务端根据sessionID找到对应的session，响应对应用户的资源

优点：数据完全存储在服务端内存，安全性很高，

缺点：

最重要的是，session只支持单体服务器，session拷贝效率低，

默认不支持跨域名，但是不同域名可能是会共享用户信息的，

因此在集群部署，分布式应用，前后端分离的背景下，session已经不再适用

#### JsonWebToken

base64: 将原来的字符串二进制化，然后重新分成每6位一组，6位对应有64个索引，分别对应0-9和所有大小写英文字母

用户信息保存在浏览器端内存，本质就是一条加密字符串。

##### 非对称加密算法

RSA—单向陷门函数：加密数字5，公钥是7,33 ，**5**^7^ mod 33 = <mark>14<mark>，解密使用公钥，x^7^ mod 33 = <mark>14<mark> 的数字有无数个，也就无法推算出具体的x，只能穷举。如果有了私钥3,33，<mark>14<mark>^3^ mod 33 = **5** 很容易就能算出原数字5

加密和解密都用同一种算法，但不是逆向。

**签名算法：** 

HS256：$A+H(A,K)$ 签发和验证都使用同一个密钥，只适用于单体应用。H表示密钥拼接在报文后进行哈希。S表示SHA256

RS256, ES256：$A+D(H(A))$ 签发用私钥，验证用公钥，适合分布式架构，安全性更高。R,E分别表示RSA与ECDSA，S表示SHA256。

##### 构成

header(base64-encoded).payload(base64-encoded).signature(HMACSHA256-encoded)

- header 用于定义token类型以及加密算法（非对称），用base64编码，相当于明文
- payload 用于装载要传输的用户数据，用base64编码，相当于明文
  - 附加一些预定义声明
  - iss: 签发者issuer
  - sub: jwt所面向的用户subject
  - aud: 接收jwt的一方audience
  - iat: jwt的签发时间 issued at
  - exp: jwt的过期时间，必须大于签发时间  expire
  - nbf: 定义在什么时间之前，该jwt都是不可用的. not valid before
  - jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击 jwtid

`注意:对于已签名的Token，这些信息虽然受到保护，不会被篡改，但任何人都可以阅读。除非加密，否则不要将机密信息放在 JWT 的有效负载或头元素中。` 

- signature = HS256...(header(base64)+payload(base64)+secret)私钥 ，用header指定的算法进行加密，鉴权核心

用户请求通过鉴权成功，**服务端通过私钥签发JWT字符串**，通过响应返回给用户，用户后续请求会在请求头中添加一个authorization:token的键值对。

再次请求，鉴权成功，然后将JWT根据secret进行解密，验证此JWT是否有效。

![image-20241030152447734](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241030152447734.png)

假如其他人偶然间拿到了JWT，然后篡改JWT，服务器拿私钥解密JWT，会发现信息被篡改。

# Filter&Listener

## Filter

拦截资源请求

![image-20240928234937677](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928234937677.png)

### 入门

![image-20240928235049041](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240928235049041.png)

### 执行流程

![image-20240929000311150](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929000311150.png)

先访问完资源，然后执行放行后的逻辑

放行前对request进行处理，放行后对response进行处理

### 使用细节

#### 拦截路径

![image-20240929000603427](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929000603427.png)

拦截的是具体的资源，不是说filter访问哪个路径

#### Filter链

![image-20240929000822547](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929000822547.png)

执行顺序：类名字符串自然排序

#### 案例：登录验证才可以访问

第一次请求，没有登录，跳转到登录页面

filter要看是否登录，登录成功就把username pswd存到客户端session中，下一次请求的时候就验证session是否有值，没有值就继续

从服务端获取用户的session，session存储登录密码，如果session

![image-20240929011626311](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929011626311.png)

针对某个资源设置filter，第一次访问被拒绝，**转发**到login页面，此时网址不会变化，就会把这个资源的网址缓存成login页面的样式，login成功以后如果再次访问这个资源，会展示login页面，只有刷新一下才能解决这个问题

## Listener

![image-20240929121700747](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929121700747.png)

# HTML+CSS+JavaScript

# <span id=ajax>AJAX</span>

异步JS和XML **A**synchronous **J**avaScript **A**nd **X**ML 

**AJAX作用1：与服务器交换数据，前后端分离**

- Servlet+JSP开发：HTML是静态的，要想展示动态的数据必须要让servlet根据请求中的参数来手动打印页面（JSP），服务端负担较重
- AJAX+HTML: 替换JSP页面 ，AJAX给服务器**发送请求**，**获取服务器响应**的数据，展示给浏览器

**AJAX作用2：异步交互**

不刷新**整个页**面也能与服务器交换数据，更新部分网页，如搜索联想，用户名是否可用校验

用户名按照一定的规则：直接本地编写js脚本即可，如果用户名不能和已有的重复，还应该发送请求，接收服务器响应回来的结果（数据库中是否重名）

## 同步、异步

![image-20240929161209624](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929161209624.png)

异步操作使 用户可以在等待响应的同时继续与页面互动，这使得应用程序更具响应性

不用刷新整个页面，只跟服务器请求需要的数据，而不是整个页面，AJAX可以减少服务器的负担和网络流量，提高响应速度。

## 使用方法

![image-20240929161455819](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929161455819.png)

URL 全路径，前后端完全分离

![image-20240929161832460](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929161832460.png)

## <span id="request">异步请求案例</span> 

现在有这样一个需求，在浏览器填完了一个用户名，要把用户名发送到服务器的某个servlet，servlet根据用户名查询是否重复，并把数据传回到浏览器。

首先应该让失焦事件绑定函数，函数中要查询用户名。

设置 提示重复字句的style属性为不可见（正常情况下不可见）ajax根据传回的数据为true or false，改变 提示重复字句的style属性，如果是，则设置可见，如果否，则设置不可见。



具体流程？可以通过以下步骤：

## 前端发送请求

使用 JavaScript（比如 `XMLHttpRequest` 或 Fetch API）发送请求：

- **GET 方法**（URL 参数字符串）：

```javascript
let username = document.getElementById("username").value;
xhttp.open("GET", "http://example.com/checkUsername?username=" + encodeURIComponent(username));
xhttp.send();
```

- **POST 方法**（请求体）：

```javascript
let username = document.getElementById("username").value;
xhttp.open("POST", "http://example.com/checkUsername");
xhttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhttp.send("username=" + encodeURIComponent(username));
```

### 服务器端处理

在 Servlet 中处理请求：

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String username = request.getParameter("username");
    // 查询数据库检查用户名是否重复
    // 返回结果到浏览器
    response.setPatameter...
}

protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String username = request.getParameter("username");
    doGet(request,response);
}
```

### GET 和 POST 的区别

- **GET**：参数通过 URL 传递，适合获取数据，但不适合传递敏感信息，因为 URL 可见，且请求长度有限。
- **POST**：参数通过请求体传递，适合发送大量数据或敏感信息。

要探究本质，就要解析他们的**<mark>报文<mark>**  

GET：

```assembly
GET /search?query=java&page=2 HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Connection: keep-alive


```

POST：

```assembly
POST /search HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Connection: keep-alive

?query=java&page=2 
```

这些都是一个个字符串而已，发送的时候设置URL和参数本质上都是拼字符串，然后把整段报文发给服务。Servlet 接收请求，解析报文，拿到参数，仅此而已。填写表单的时候也一样，form标签的action属性就是要发送请求的目标，输入参数，提交的时候，就相当于填写好了目标URL，既然目标确定了。浏览器会解析内容，写好报文，发送给目标，那么浏览器是如何确定要发送给谁呢？浏览器确定请求的目标地址（即请求的 URL）是通过 URL 来实现的。以下是这一过程的基本步骤：以GET请求为例：

### 目标URL的填写 

用户在浏览器中提交表单（例如，点击“提交”按钮），这会触发一个请求。表单的 `action` 属性指定了要发送请求的 URL。浏览器会将表单数据编码为查询字符串，并附加到 `action` URL 后面作为要访问的目标。

### 解析 URL

浏览器解析这个 URL，分解成几个部分：

- **协议**：`http`
- **主机名**：`www.example.com`
- **路径**：`/search`
- **查询字符串**：`?query=java`

### DNS 解析

浏览器会通过 DNS（域名系统）将主机名转换为相应的 IP 地址，以便找到目标服务器。例如，`www.example.com` 可能会被解析为 `192.0.2.1`。

### 建立 TCP 连接

浏览器与目标服务器建立 TCP 连接，通常使用 HTTP 端口（默认为 80，HTTPS 为 443）。

### 构造请求报文

一旦连接建立，浏览器会根据表单数据构造 HTTP 请求报文，包括请求行、请求头和请求体。

### 发送请求

浏览器通过已建立的 TCP 连接，将构造好的请求报文发送到服务器的指定 IP 地址。

### 服务器处理请求

服务器接收到请求后，会根据请求的路径和参数来处理请求，最终返回相应的响应数据。



发出请求实际上就是浏览器访问目标URL

表单：action就是目标URL，如果是get请求，浏览器会将目标url加上参数。随后解析url，得出目标IP，根据这些参数生成请求报文发送到目标IP。

## Axios

axios是对js 的封装

[Axios中文文档 | Axios中文网 (axios-http.cn)](https://www.axios-http.cn/)

![image-20240929192249373](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929192249373.png)

method url data

![image-20240929192732304](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929192732304.png)

链式编程

```javascript
axios.get(url).then(function(resp){
    //具体用响应干什么？
})
axios.post(url,data).then(function(resp){
    //具体用响应干什么？
})
```

resp：

①data :实际响应回来的数据

②headers :响应头信息

③status :响应状态码

④statusText:响应状态信息

特色：自动将data对象序列化为json字符串，再自动将响应数据中的json转回js自定义对象

![image-20240929210844535](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929210844535.png)

解构赋值：then({data}) 只取resp的data字段

# JSON

**J**ava**S**cript **O**bject **N**otation js对象表示法

![image-20240929194822398](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929194822398.png)

字段名要用双引号括起来，

```js
var json = {
    "key1":value1,
    "key2":value2
};
//json.key1 访问value1
var json = {
    key1:value1,
    key2:value2
};
```

这两个对象的主要区别在于属性名的引号使用。在第一个对象中，所有属性名都用双引号包围，而在第二个对象中，属性名没有引号。根据 JavaScript 的语法，属性名可以不加引号（如果是有效的标识符），但如果包含特殊字符或空格，就需要加引号。功能上，它们是等价的。



axios发送自定义对象会自动转成json的形式

## JSON数据和Java对象转换

Fastjson 高性能JSON库。

导入fastjson坐标

```java
String jsonStr = JSON.toJSONString(user);
//对象tostring
User user = JSON.parseObject(user, User.class);
//解析出对象
```

```kotlin
response.setContentType("text/json;charset=utf-8");
response.getWriter().write(jsonString)
```

## 案例：增删改查

**查询**：把axios发送请求接收响应数据并打印数据的过程 封装成一个函数，跟onload（brandSelect页面加载完成）绑定。

axios+html 接收servlet响应，打印表格

![image-20240929200621197](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929200621197.png)

axios这边接收到json，也就是resp.data 是对象的集合，所以用for循环遍历，由于是打印，所以可以用id锚定表格的标签，每遍历一次就累加字符串一次，最后一起写入表格标签的innerHTML中。



**新增品牌**：

表单提交的操作是一个同步请求，同步请求是直接发送参数字段，而且需要重新加载页面才能生效，利用不上js的异步高效性，所以提交按钮应该设置成普通button，进行异步操作

![image-20240929203719660](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929203719660.png)![image-20240929204058611](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929204058611.png)![image-20240929204235548](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240929204235548.png)

获取表单数据，字符串直接赋值，复选框的结果用checked表示，因为这是两个复选框，名字都叫status，所以返回的是一个元素数组，对这个数组进行便利，被选中的就把自己的value赋给对象。

axios发送自定义对象会自动转成json的形式，直接把封装好的自定义对象添加到axios的data参数中即可



函数绑定提交按钮的onclick事件，设定js函数把表单填入的内容封装成json对象（即为前面的操作），发送ajax请求给addServlet。



**addServlet**处，<u>getParameter不能接收json数据</u>，所以应该用<u>getReader.readLine读取字符串</u>，然后把json字符串转成pojo对象，执行添加操作，返回操作成功与否，作为响应数据发出。

axios接收响应数据，如果操作成功，就跳转到第一步做出来的加载html页面中



增删改用post 查用get 

# Vue+Servlet 开发

## 整体架构

### 客户端

- **Vue**：前端的JS骨架，model view双向绑定，渲染网页
- **axios**：AJAX请求发送
- **Element** **UI**：CSS组件库，基于Vue

### 服务端

- **Web层**：Servlet调用Service层的方法查询，结果转为JSON，响应JSON数据发给客户端
- **Service层**：BrandService定义selectAll方法，获取sqlSession对象，调用BrandMapper执行SQL语句
- **DAO层**：BrandMapper定义selectAll方法，方法体为MyBatis执行某条具体的SQL语句

![image-20241002150928236](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002150928236.png)

## 服务端优化

### Service 优化

#### Service 接口定义

- 定义 **BrandService** 接口：定义一些业务的抽象方法，实现类中实现业务方法，在servlet中创建好业务实现对象，这样就解除了service层和servlet层的耦合性

#### ServiceImpl 接口实现

- 在实现类中，先创建好唯一的的factory工厂，然后在方法中开启sqlSession，执行SQL语句

#### 优化结果

- **UserService:**
- ![image-20241002164617150](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002164617150.png)
- **UserServiceImpl:**
- ![image-20241002164523529](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002164523529.png)





### Servlet 优化

#### 业务实现类的创建

为了增强项目的组织度，会进行业务整合，在BrandServlet中，创建一个BrandService的对象

```java
class BrandServlet{
	BrandService brandService = new BrandServiceImpl1();
    //调用service的方法
    
}
```



#### 业务功能整合

一个实体类的一个功能就要新创建一个Servlet，不易管理，要把一个实体类的所有功能都放在一个servlet中。（BrandServlet，UserServlet）通过/brand/*   /user/*来访问上述两个servlet。

![image-20241002135939524](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002135939524.png)

- 如图，原先`HttpServlet`的`service`方法根据请求的方式（`getMethod`）进行方法的分发（`doGe`t或`doPost`）
- 现在要根据请求的路径来进行方法的分发，因此`BrandServlet`不能直接继承`HttpServlet`，要创建一个`BaseServlet`继承`HttpServlet`，重写其`service`方法，根据路径分发方法。同理`UserServlet`也直接继承`Base`
- 获取到请求路径的最后一部分（最后一个`/`之后的内容）就是请求的方法名
- 方法名称有了还要找`对应servlet`的字节码文件，`baseServlet`没有`@WebServlet`注解，也就不会直接访问了，到时候被访问的应该是`BrandServlet`和`UserServlet`这两个子类，子类继承父类的`service`方法，所以`service`方法中的`this.getClass`就能理所应当地拿到子类的字节码文件。
- 因为`BrandServlet`和`UserServlet `都是要先执行`service(req,resp)`方法，接收`request`参数和`response`参数，如果要执行具体的`selectAll`业务方法，就要在反射调用方法的时候把参数加上，同时在`子类servlet`中，业务方法接受的参数全部统一成`req`和`resp`。

如此一来，就能实现：

1. 访问`/brand/selectAll`路径，
2. 调用重写过后的`service(HttpServletRequest req,HttpServletResponse resp)` 能获取方法名和字节码文件
3. 根据方法名和参数类型（`methodName, HttpServletRequest req.class, HttpServletResponse.class`）获取Method对象，
4. `method.invoke(this, req, resp)`，实现业务整合



#### 优化结果

- **BaseServlet:**
- ![image-20241002164451103](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002164451103.png)
- **UserServlet:** 
- ![image-20241002164355800](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002164355800.png)



### 优化后的后端结构

后端的DAO, Service, Web层分开 各司其职，减少了耦合度

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002164240142.png" alt="image-20241002164240142" style="zoom: 50%;" />



## <span id="mybatis"> 其他细节 </span> 

### MyBatis 模糊查询

#### `'%#{password}%'`不行？`'%${password}%'`行？

![image-20241002225509824](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002225509824.png)

${password}就是最简单的文本替换，直接拼接字符串，也不会类型转换（输入参数`that`会直接拼接成`and password like that` 很显然少了引号）连SQL都无法注入`' OR '1' = '1` `and password like ' OR '1' = '1` （语法错误）自然，模糊匹配就变成`'%that%'`了

![image-20241002230031380](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002230031380.png)

预编译占位符#{password}会把整个password转换成字符串，输入参数`' OR '1' = '1`会帮你转义成`\' OR \'1\' = \'1` 还会贴心地给两边加上引号`and password like '\' OR \'1\' = \'1'`

因此，#{password}本身就自带引号，模糊匹配会解析成`'%'that'%'` 完全的语法错误。

#### 应该怎么用

既然返回的是带引号的字符串，可以用拼接字符串函数，也可以用空格把这三个字符串分开

![image-20241002231938685](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002231938685.png)



### MyBatis 分页查询

![image-20241002234502508](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241002234502508.png)

![image-20241003001119464](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241003001119464.png)

#### 后端

分页查询要两个参数，这一页从哪一行开始和每页显示的条数

前端传递给后台 当前页码和每页显示条数，(当前页码-1)*每行显示条数就是这一页开始的一行

PageBean封装 总条数 和 这一页的查询结果List<User\> 

list用于显示，总条数返回给前端

#### 前端优化

前端刷新表格的操作：发出自己的两个属性，收到PageBean中的rows和totalCount

![image-20241003003901994](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241003003901994.png)

前端点击页码的操作，设置自身的两个属性，同时刷新表格

![image-20241003004044018](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241003004044018.png)

### 插入重复的键——事务回滚

username是unique 且 not null 的，所以不能重复，在提交表单的时候，如果输入重复数据，就会导致事务提交失败，这是就会出现异常，**如果出现异常不处理**，会一直导致故障。

![image-20241003005454880](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241003005454880.png)

使用try catch 如果出现异常，就调用rollback，同时return false，响应

前端收到响应，会根据结果弹出提示，成功或者失败

![image-20241003005659475](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241003005659475.png)

### 优化后的前端结构

![image-20241003004606856](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241003004606856.png)

![image-20241003004711700](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241003004711700.png)

加入了表格loading动画，刷新按钮，以及删除和插入的结果提示









