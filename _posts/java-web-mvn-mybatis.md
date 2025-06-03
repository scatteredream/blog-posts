---
name: web-miscellaneous
title: JDBC MyBatis XML Maven
date: 2024-07-25
tags: 
- jdbc
- mybatis
- xml
- maven
categories: web development

---



# XML

标记语言 e**X**tensible **M**arkup **L**anguage

自闭合标签 ：标签不包含任何内容可以简化为`<br/>` 

[XML - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/XML) 

[XML 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/xml/xml-tutorial.html) 

**XML**（可扩展标记语言，Extensible Markup Language）的核心语法主要用于描述结构化数据。XML 语法非常严格，但也因此具有良好的可读性和可扩展性。以下是 **XML** 的一些重要核心语法规则和概念，它们是理解和正确编写 XML 文档的基础：

## 基本语法

---

**元素（Element）**

- **元素是 XML 文档的基本构建块**。每个元素有一个**开始标签**和一个**结束标签**，或是自闭合的标签。

语法：

```xml
<element>content</element>
```

- **开始标签**：`<element>`
- **结束标签**：`</element>`
- **自闭合标签**：`<element />`

元素之间可以嵌套：

```xml
<root>
    <child>content</child>
</root>
```

规则：

- 元素名称必须区分大小写。`<Element>` 与 `<element>` 是不同的标签。
- 标签名称不能以数字或特殊符号开头，通常使用字母、数字和某些符号（如 `_`、`-`）。
- 标签必须要有匹配的结束标签，或是用自闭合标签。

---

**属性（Attributes）**

- **属性**为元素提供额外的信息。属性位于开始标签的内部，使用键值对表示，且值必须使用双引号或单引号包裹。

语法：

```xml
<element attribute="value">content</element>
```

或：

```xml
<element attribute="value" />
```

规则：

- 一个元素可以有多个属性，属性的名称必须唯一。
- 属性值必须用引号包裹（双引号或单引号均可）。

示例：

```xml
<book title="XML Guide" author="John Doe">
    <price>29.99</price>
</book>
```

---

**声明（XML Declaration）**

- XML 文档通常以声明开头，定义了 XML 版本和编码格式。虽然声明是可选的，但建议在每个 XML 文件开头声明。

语法：

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

说明：

- `version`：指定 XML 版本，通常为 `1.0` 或 `1.1`。
- `encoding`：指定字符编码，通常为 `UTF-8`。

---

**注释（Comments）**

- **注释**用于对 XML 文档进行说明或标注，注释不会被解析器处理。

语法：

```xml
<!-- 这是一个注释 -->
```

规则：

- 注释不能嵌套，且不能出现在声明之前。
- 注释内容不能包含 `--` 连字符。

---

**CDATA（Character Data，字符数据）**

- **CDATA** 区域用于包含原样保留的字符数据。它告诉 XML 解析器不要处理其中的内容为标签或实体。通常用于嵌入包含特殊符号（如 `<`、`&`）的文本数据，例如 HTML 代码片段。

语法：

```xml
<![CDATA[
    <content> 这里的内容将不会被解析为标签 </content>
]]>
```

规则：

- CDATA 区块中的内容不会被解析器处理，即使它包含 `<`、`>`、`&` 等特殊字符。

---

**实体引用（Entities）**

- **实体**用于替代特殊字符或常用的短字符串。常见的实体包括：`&amp;`、`&lt;`、`&gt;`、`&apos;` 和 `&quot;`。

语法：

- **特殊符号的实体引用**：
  - `&lt;` 表示 `<`
  - `&gt;` 表示 `>`
  - `&amp;` 表示 `&`
  - `&apos;` 表示 `'`
  - `&quot;` 表示 `"`

示例：

```xml
<description>This is &lt;important&gt; content &amp; data</description>
```

此 XML 中的内容表示为 `This is <important> content & data`。

---

**命名空间（Namespaces）**

- **命名空间**用于避免不同 XML 文档中的元素或属性发生冲突，尤其在合并两个不同 XML 数据集时更为重要。

语法：

```xml
<element xmlns:prefix="namespaceURI">
    <prefix:child>content</prefix:child>
</element>
```

- **xmlns** 是定义命名空间的标准属性。
- `prefix` 是命名空间的前缀，`namespaceURI` 是对应的命名空间地址。

示例：

```xml
<bookstore xmlns:bk="http://example.org/book">
    <bk:book>
        <bk:title>XML Guide</bk:title>
    </bk:book>
</bookstore>
```

这里定义了一个命名空间 `bk`，用来区分不同命名空间下的元素。

---

**空元素（Empty Elements）**

- **空元素**是没有内容的元素。可以用以下两种方式表示：
  - `<element></element>`
  - `<element />`（自闭合）

示例：

```xml
<emptyElement />
```

---

**处理指令（Processing Instructions）**

- **处理指令**用于为应用程序提供处理 XML 文档的特定指令。

语法：

```xml
<?instruction content?>
```

示例：

```xml
<?xml-stylesheet type="text/xsl" href="style.xsl"?>
```

此指令告诉浏览器使用 `style.xsl` 样式表来展示 XML 数据。



## XML Schema 和 DTD

- **XML Schema** 和 **DTD（Document Type Definition）** 是定义 XML 文档结构和约束的工具，帮助确保 XML 数据的有效性和结构一致性。

### DTD

```xml
<!DOCTYPE note [
    <!ELEMENT note (to,from,heading,body)>
    <!ELEMENT to (#PCDATA)>
    <!ELEMENT from (#PCDATA)>
    <!ELEMENT heading (#PCDATA)>
    <!ELEMENT body (#PCDATA)>
]>
```

### XML Schema

```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="note">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="to" type="xs:string"/>
                <xs:element name="from" type="xs:string"/>
                <xs:element name="heading" type="xs:string"/>
                <xs:element name="body" type="xs:string"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
```

- **DTD**：早期用来定义 XML 文档结构，较为简单。
- **XML Schema**：更现代且功能更强大的 XML 定义方式，支持更精确的数据类型定义。

#### Spring maven中的命名空间是什么？

为了避免标签名称冲突创建了namespace的概念，对于beans这个标签：

- xmlns="https://springframework.org/schema/beans/" 默认命名空间的namespaceURI指向spring官网
- xmlns:context 创建名为context的命名空间
- context:property-placeholder 指明property-placeholder这个标签含义从context命名空间解析



XML schema(.xsd文件)用于定义xml文档的结构，要说明的是XML作为标记语言本身其实比较自由，如果要严格一些，比如作为框架的配置文件，就需要做出一些限制，XML schema正好是做这个的。

`xsi` 是一个业界默认的用于 XSD(（XML Schema Definition) 文件的命名空间。

如果你创建一个命名空间需要引用别人的schema，需要加上这个schema的namespaceURI，并且在xsi的schemaLocation中加入namespaceURI和xsd文件的准确URL

```xml
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd"
```

上面这行的语法其实是， `xsi:schemaLocation = "键" “值”`
即 xsi 命名空间下 schemaLocation 元素的值为一个由空格分开的URI pair。

> The first records the author's warrant with pairs of URI references (**one for the namespace name**, and **one for a hint as to the location of a schema document** defining names for that namespace name)

- 前一个“键” http://maven.apache.org/POM/4.0.0 指代 命名空间， 只是一个全局唯一字符串 URI ，用来表示这个命名空间是唯一的 
- 后一个值指代 XSD URL , 这个值指示了命名空间所对应的 <u>XSD 文件</u>的位置， xml parser 可以利用这个信息获取到 XSD 文件， 从而通过 XSD 文件对所有属于 URN http://maven.apache.org/POM/4.0.0 的元素结构进行校验， 因此这个值必然是可以访问的， 且访问到的内容是一个 XSD 文件的内容

#### URI & URL & URN

Identifier & Locator & Name

URI比URL更加抽象

URN确定了东西的身份，不提供找到资源的方法，URL提供了找到它的方式，点开一个URL就能直接找到资源。

> 用于标志唯一书目的[ISBN](https://zh.wikipedia.org/wiki/ISBN)系统是一个典型的URN使用范例。例如，`ISBN 0-486-27557-4`无二义性地标志出莎士比亚的戏剧《[罗密欧与朱丽叶](https://zh.wikipedia.org/wiki/罗密欧与朱丽叶)》的某一特定版本。为获得该资源并阅读该书，人们需要它的位置，也就是一个URL地址。在[类Unix](https://zh.wikipedia.org/wiki/类Unix)操作系统中，一个典型的URL地址可能是一个[文件目录](https://zh.wikipedia.org/w/index.php?title=文件目录&action=edit&redlink=1)，例如`file:///home/username/RomeoAndJuliet.pdf`。该URL标志出存储于本地硬盘中的电子书文件。因此，URL和URN有着互补的作用。

这里有两个URI，分别是maven和spring的配置xml中的namespaceURI：

http://springframework.org/schema/beans 官方做成了像网盘一样的页面，里面有xsd文件的存储超链接，可以方便地访问xsd文件

![image-20241019210205303](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019210205303.png)

http://maven.apache.org/POM/4.0.0 就是一个PageNotFound页面

![image-20241019210235645](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241019210235645.png)

对于schemaLocation中的URI pair，到底能不能通过第一个URI直接定位资源不重要，重要的是它是一个<u>唯一</u>标识这个东西身份的字符串，第二个URI一定是一个URL，能访问到.xsd文件本身

#### XML 拓展阅读

[统一资源标识符（URI） - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/zh-cn/统一资源标识符)

[URL与URI与URN，有联系有区别？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/38120321)

[XML中的xmlns、xmlns:xsi和xsi:sechemaLoacation的具体含义是什么？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/372022140)

[详解 xml 文件头部的 xmlns:xsi-CSDN博客](https://blog.csdn.net/lengxiao1993/article/details/77914155)

[How schema definitions are located on the Web - w3.org](https://www.w3.org/TR/xmlschema-1/#schema-loc) 

---

## 总结

1. **元素** 是 XML 的核心构建块，标签必须正确匹配或自闭合。
2. **属性** 为元素提供额外信息，值必须用引号包裹。
3. **XML 声明** 用于定义 XML 的版本和字符编码。
4. **注释** 用于在文档中添加说明性文字，不会被解析器执行。
5. **CDATA 区域** 用于保留未解析的原始文本数据。
6. **实体引用** 用于表示特殊字符。
7. **命名空间** 用于避免元素或属性名称冲突。
8. **空元素** 可以通过自闭合标签表示。
9. **DTD 和 XML Schema** 是用于验证 XML 文档结构的工具。
10. **处理指令** 可以传递给应用程序或解析器，提供特定处理逻辑。

这些核心语法确保了 XML 文档的规范性和可读性，帮助应用程序和解析器处理结构化数据。

# Maven

## 初级

### 项目结构

#### 项目 Project

IDEA项目结构：.idea文件代表整个项目（整个窗口）

如果删掉.idea文件，里面的模块都将无法运行

#### 模块 Modules

创建项目时自动创建一个和项目同名的module，这个module里包含项目的.idea以及自己的示例src等。每个module都有一个iml(information of module)文件，如果从外部导入其他路径的module，会和这个module并列。

模块套模块就是物理上的依赖关系，maven project有一个pom，其子模块也可以有pom，并且可以相互依赖。maven项目创建以后自动创建一个maven父模块，模块路径就是项目所在的路径，包含了iml和.idea，可以手动移除整个模块，但不能删除idea文件。

如果导入不同的modules之间有物理上的嵌套关系，自动变成父子模块关系，如果没有嵌套关系，就是并列的，相互之间不会影响。

项目管理工具

[Maven Repository: Search/Browse/Explore (mvnrepository.com)](https://mvnrepository.com/) 



### 构建流程

标准化统一的构建路程，编译测试打包发布

### 依赖范围

default: compile

编译：main 测试：test 运行：打包以后的

![image-20240924153605252](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924153605252-1727923499126-1.png)

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.1.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### POM 对象模型

![image-20240924151728949](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924151728949-1727923499126-2.png)

![image-20240924152425583](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924152425583-1727923499126-3.png)

### 常用命令

clean 清理项目下文件

compile 编译项目下文件生成target目录

test 执行项目下的文件

package 打包成jar包

install 把当前的项目打包jar 放到本地仓库 能对其配置依赖

### 生命周期

![image-20240924153131022](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924153131022-1727923499126-4.png)

![image-20241024201235892](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024201235892-1729772077345-1.png)

#### **clean**—清理target目录

清理当前项目的target目录（上一次构建生成的文件）

#### validate——验证项目元信息可用

#### **compile**—将src/main/java编译为字节码，输出到target目录

#### test——单元测试（可跳过）

#### **package**—打包(jar,war,pom等)

#### verify——检查打的包是否有效

#### **install**—将项目部署到本地仓库

#### **deploy**—将项目部署到远程仓库

需要在maven的setting.xml中配置私服的用户名和密码（<server><mirror>的id相同），在pom.xml配置distributionmanagement

![image-20241025184411633](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025184411633.png)

#### site——创建项目站点

不属于build

## 高级进阶

### 分模块开发

不同模块之间的依赖关系

添加依赖到pom.xml中的dependency

```xml
<groupId>com.itheima</groupId>
<artifactId>maven_03_pojo</artifactId>
<version>1.0-SNAPSHOT</version>
```

安装依赖到本地仓库：

<mark>maven-install<mark>

### 依赖传递

#### 依赖冲突

同一个pom.xml 后配置的覆盖先配置的

![image-20241024184013133](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024184013133-1729772371846-3.png)

### 可选依赖&排除依赖

#### 可选依赖——主动向别人隐藏，别人引用时看不到（被引用者）

`<optional>true<optional>`

![image-20241024184316546](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024184316546-1729772371847-4.png)

#### 排除依赖——引用时主动忽略某个元素（引用者） 

`<exclusion></exclusion>` 不用写版本号

![image-20241024184835202](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024184835202-1729772371847-7.png)

### 聚合工程与继承

#### 聚合——同时构建模块

某个模块变化了，依赖于它的还要一个一个重新构建。聚合工程可以同时重新构建他们

##### 创建空工程

![image-20241024185111570](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024185111570-1729772371847-5.png)

##### 改变打包方式——pom

![image-20241024185202904](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024185202904-1729772371847-6.png)

##### 添加聚合的模块

顺序会自动根据依赖关系处理

![image-20241024185331214](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024185331214-1729772371847-8.png)

#### 继承——简化子工程配置

多个模块有相同的依赖，更换版本比较繁琐。

简化子工程配置，减少版本冲突

##### 父工程打包方式——pom

![image-20241024190544704](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024190544704-1729772371847-9.png)

##### 父工程配置子工程共同依赖

![image-20241024190619020](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024190619020-1729772371847-10.png)

##### 子工程配置parent

![image-20241024185744377](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024185744377-1729772371847-11.png)

子类会继承父类的全部依赖，父类动一下，子类全部跟着动，

##### 子类可选的继承项

有些依赖并不是所有的子工程都需要

###### 父工程配置可选依赖管理

![image-20241024190202481](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024190202481-1729772371847-12.png)

######  子工程声明需要此依赖

不要加版本

![image-20241024190240611](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024190240611-1729772371848-13.png)

#### 继承 vs 聚合

![image-20241024190720813](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024190720813-1729772371848-14.png)



### 属性加载

#### 属性

![image-20241024190830292](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024190830292-1729772371848-15.png)

定义一个常量，需要的时候直接写`${}`，减少硬编码，降低耦合 

`<properties>`

![image-20241024191136401](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024191136401-1729772371848-17.png)

####   .properties配置文件 加载属性

![image-20241024191334633](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024191334633-1729772371848-16.png)

![image-20241024191340916](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024191340916-1729772371848-18.png)

[maven-resources-plugin详解 - 红尘过客2022 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hcgk/p/17600908.html)

##### 指定resources目录，纳入maven管理

![image-20241024195223511](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024195223511-1729772371848-19.png)

子项目会继承父项目的配置，将自己的resources作为资源

![image-20241024191550934](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024191550934-1729772371848-22.png)

##### 解决打war包强制要求WEB-INF/web.xml

![image-20241024200210773](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024200210773-1729772371848-21.png)

### 版本号

![image-20241024200423176](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024200423176-1729772371848-23.png)

GA General Availability

### 多环境

#### 配置多环境

##### 指定加载某一环境

![image-20241024200555818](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024200555818-1729772371848-25.png)

profiles id 

activeByDefault 默认环境

![image-20241024200720633](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024200720633-1729772371848-24.png)

##### 构建指令 -P 环境名

![image-20241024200902692](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024200902692-1729772371848-26.png)

#### 跳过测试

![image-20241024201215677](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024201215677-1729772371848-27.png)

![image-20241024201235892](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024201235892-1729772371848-28.png)

测试，但是忽略一些东西

![image-20241024201333235](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024201333235-1729772371848-29.png)

![image-20241024201411791](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024201411791-1729772371848-30.png)

### 私服

![image-20241024225654228](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024225654228.png)

#### 私服仓库分类

![image-20241024230834698](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024230834698.png)

![image-20241024230845592](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024230845592.png)

第三方资源：oracle的jdbcJar包

#### 资源上传与下载

![image-20241024231324939](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241024231324939.png)

这些都是本地仓库的配置

需要在maven的xml

# JDBC

![image-20240923155407582](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923155407582-1727923816030-9.png)

![image-20240923155451882](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923155451882-1727923816031-10.png)

jdbc 接口 

## 执行SQL步骤

驱动 实现接口

1. 注册Class `Class.forName("com.mysql.cj.jdbc.Driver");`
2. 建立`Connection`使用`DriverManager`的`getConnenction(url,username,password)` 创建连接
3. 建立`Statement`调用上一部连接对象的`createStatement()` 
4. 调用上一部`Statement`对象的`execute(sql)` 执行sql语句，这个函数返回的是操作的数据库的行数
5. 关闭资源，或者try with resource



## DriverManager

#### 注册驱动，连接数据库

registerDriver 驱动中自带静态代码块，forname加载之后自动执行（可选）

连接数据库 URL 统一资源定位符

[协议类型]: //服务器地址:端口号/资源层级UNIX文件路径文件名?查询#片段ID

`jdbc:mysql://localhost:3306/itcast` 

`protocol:` // `ip+port` / `databaseName`

DriverManager.getConnection(url, usr, pw)

## Connection

## 获取执行sql的对象

- `prepareStatement(sql) `  预编译
- `createStatement()` 普通执行sql对象

#### <mark>管理事务<mark>

![image-20240923195630325](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923195630325-1727923816031-12.png)

mysql自动提交事务，

setAutoCommit(false) 开启事务

commit() 提交事务 rollback() 回滚事务

rollback放到catch块中。

Atomic Consistency Isolational Durability

## Statement

### 执行SQL

`int executeUpdate(DML, DDL)`返回受到影响的行数 

`ResultSet executeQuery(DQL)` 执行DQL，返回结果集

## PreparedStatement

### 预防SQL注入

**SQL Injection**：脚本

**登录逻辑**：接收username和pswd, 然后在sql语句中插入usrname和pswd

```java
String sql = "select * from tb_user where username = '" + username + "'and password = '" + pswd + "'";
//结果集没有结果则返回失败
```

登录操作实际上就是查询数据库，pswd = ' or '1' ='1 拼接sql语句，改变原先的验证逻辑

**解决方法**

```java
sql = "select * from tb_user where username = ? and password = ?"
PreparedStatement pstmt = myConnection.prepareStatement(sql);
//给参数赋值
pstmt.setString(1,username);
pstmt.setString(2,pswd);
//1,2是问号的位置编号 start from 1

pstmt.executeQuery();
```

![image-20240924163227260](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924163227260-1727923816031-11.png)



```sql
SELECT * FROM users WHERE username = 'admin' AND password = '\' OR \'1\'=\'1'
#用户名为admin 密码为' OR '1'='1 
select * from users where username = 'admin' AND password = '' OR '1' = '1'
```

- 占位符**将敏感字符变成转义字符**，这样sql就会认为这是一个文本类的单引号而不是格式的单引号

- 自动进行类型转换，外边加一层引号

![image-20240924165014456](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924165014456-1727923816031-13.png)

### 预编译SQL语句，高性能

原来sql执行，要把检查语法和编译sql都交给数据库服务器，`?userServerPrepStmts=true` 串加到最后的参数后边，默认关闭

![image-20240924165716038](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924165716038-1727923816031-15.png)![image-20240924165801554](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924165801554-1727923816031-16.png)

 执行两次，只会预编译一次：

![image-20240924170025388](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924170025388-1727923816032-18.png)

- PreparedStatement 原理
  在获取PreparedStatement对象时，将sql语发送给mysqI服务器进行检查，编译(这些步骤很耗时)
- 执行时就不用再进行这些步骤了，速度更快
- 如果sql模板一样，则只需要进行一次检查、编译

## ResultSet

把查询结果（表）封装起来

`boolean next()` 将光标向下移动一行，判断是否有效

`int getInt("id")`获取id列的数据

`String getString()`获取字符串

```java
while(resultset.next()){
    resultset.getInt();
    ... ...
//可以将查询到的数据封装到自定义类中，然后list存储多个对象
}
//循环判断游标是否在最后一行的末尾 
```

## DataSource 数据库连接池

- 数据库连接池，sun定义的官方接口

- connection要调用系统资源，开启关闭都要耗费系统资源。

- 连接池：资源复用，提升响应速度，避免连接遗漏（连接池资源占满时，如果有新的请求，就会根据一定的算法从已占用资源中空出来一个资源）

![image-20240924170701651](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924170701651-1727923816032-19.png)

![image-20240924170726525](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924170726525-1727923816031-14.png)

### Druid

![image-20240924170924469](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240924170924469-1727923816031-17.png)

```java
Properties prop = new Properties();

prop.load(new FileInputStream("path"));

DataSource ds = DruidDataSourceFactory.createDataSource(prop);

Connection conn = ds.getConnection;

//System.getProperty("user.dir") 得到当前的相对路径起始点
```

# MyBatis

## 快速入门

MyBatis：**持久层框架**，可简化JDBC开发

**持久层**：数据保存到数据库的一层代码（JavaEE：表现层，业务层，持久层）

**框架**：可重用的通用的软件基础代码模型，在框架的基础上构建软件编写更加高效规范

使用XML配置文件进行简化，免除了几乎所有JDBC代码 设置参数 获取结果集，减少硬编码，免除了繁冗步骤。

mybatis-config.xml 记录 JDBC注册连接信息和下方的映射文件

![image-20240925145936781](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925145936781.png)

EmployeeMapper.xml 记录映射信息 用session  select 的时候用test.selectAll来执行对应sql语句，<u>resultType是POJO(Plain old java object)的类名（全限定名）</u>  

![image-20240925150035852](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925150035852.png)

```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = 
new SqlSessionFactoryBuilder().build(inputStream); 
SqlSession sqlSession = sqlSessionFactory.openSession();
List<Employee> list = sqlSession.selectList("test.selectAll");
```

## Mapper 代理

### 步骤

![image-20240925151447994](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925151447994.png)![image-20240925151453517](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925151453517.png)

### 注意事项

src/main下有java和resources 

相对路径：编译之后java下的内容和resources的内容放在同一个classes文件夹下,如果java和resources的目录结构有相同的地方，合并内容。 接口在java文件夹下面新建com.example.mapper 对应的映射文件在resources文件夹下新建com/example/mapper目录放入

对应的，mybatis-config文件中也得修改映射文件的名称。上图所示，可以用包名代替 mapper resource

![image-20240925152748288](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925152748288.png)



原来session执行select* 语句调用的是selectList方法，这句话的意思是返回一个表，所以返回值要用List来接，

执行sql语句本质上是调用jdbc的各种api方法，重复性较高，并且有相当数量的硬编码，为了减少硬编码，把参数放入配置文件中，运用了动态代理技术

### 源码分析（粗浅）

框架利用java反射机制和动态代理机制，比如上图的select id  = selectAll，重写了mapper的sql方法，并据此生成mapper接口的代理对象                          method.getName == selectAll 执行 sql方法并返回值      

![image-20240925155924934](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925155924934.png)

manager的有参构造器，创建了一个代理

![image-20240925160215649](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925160215649.png)

InvocationHandler 中 invoke 的 重写 此处代理的是session，替session执行sql语句

![image-20240925160058102](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925160058102-1727251872747-1.png)

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 处理默认方法（如接口的 `default` 方法）
        if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, args);
        } else {
            return this.sqlSession.selectOne(method.getName(), args); // 根据具体方法执行查询
        }
    }
```

#### JDK 动态代理的关键点

1. **接口代理**：
   - JDK 动态代理只能为实现了接口的类生成代理，因此 MyBatis 的 `Mapper` 必须是接口。
   - 在运行时生成的代理类实现了 `Mapper` 接口，并将方法调用委托给 `MapperProxy`。
2. **`InvocationHandler`**：
   - `MapperProxy` 作为 `InvocationHandler`，负责拦截 `Mapper` 接口的方法调用。
   - `invoke` 方法中的逻辑决定了如何将接口方法映射到数据库操作。
3. **灵活的 SQL 映射**：
   - 通过动态代理，MyBatis 可以将 `Mapper` 接口中的任意方法与对应的 SQL 映射，无需编写具体的实现代码。

## mybatis-config.xml

### environments

不同的environment对应不同的工作环境，通过default来切换

![image-20240925162843370](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925162843370.png)

transaction manage 事务管理 这里是JDBC

datasource  pooled

property xxx 对应数据库的连接信息

### mappers

标注mapper的配置文件的路径或者包名

### typeAliases

- `<typeAlias type="com.atguigu.mybatis.bean.Employee" alias="emp/>`

  为这个类起一个别名emp，不写alias默认是小写employee，这样可以在resultType项用emp来代替全限定名

- `<package name="com.atguigu.mybatis.bean"/>` 对这个包下所有的类起别名，是类本名的小写

```java
  @Alias("emp")
  public class Employee {
  // 类的其他代码
  }
  ```

XML配置要遵循如下约束

![image-20240925164103152](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925164103152.png)

![image-20241020124943974](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241020124943974.png)

## crud

插件：MyBatisX

三步走：

- 编写Mapper接口 定义抽象方法，注意参数和返回值，还有抽象方法的名字就是 select id，
- 编写SQL语句：mapper映射文件中的 id resultType sql要齐全
- session.getMapper    用 mapper调用

### 实体类字段和数据库名称

- 成员变量名字和数据库字段严格一致

- sql语句中，给要查询的列起别名 `brand_name as brandName, company_Name as companyName`
  - 每一次都要写一大段，把相同的部分看做**sql片段** 不灵活
  - `<sql id="brand_column"> 相同的部分 </sql>` 
  - sql语句中 select `<include refid="brand_colomn"/>` from xxxx

#### 最好解决方案：ResultMap

- 定义`resultMap`标签

- 把`resultType`改成`resultMap` 结果 column行对应的是数据库里的别名 property对应的是pojo的属性 

  ![image-20240925175306144](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925175306144.png)

```java
  @Mapper
  public interface UserMapper {
      @Select("SELECT id, name, email FROM users WHERE id = #{id}")
      @Results(id = "userResultMap", value = {
          @Result(property = "id", column = "id"),
          @Result(property = "name", column = "name"),
          @Result(property = "email", column = "email")
      })
      User getUserById(int id);
      
      @Select("SELECT * FROM users")
      @ResultMap("userResultMap")
      List<User> getAllUsers();
  }
  //select注解开发，Results注解映射结果，给自己起名userResultMap
  //getAllUsers方法就可用ResultMap注解来映射返回值
  ```

  

### 参数占位符

定义抽象方法selectById(int id) 假如这里定义形参名id，返回值是实体类Brand，sql语句用 where 修饰，`where id = #{id} ` 形参名字用大括号加井号括起来

${id} ->     直接拼接字符串，不能防止[SQL注入]()

#{id} -> ?  占位符，能将内容转义，防止sql注入

preparedstatement执行，#{id} 会将id视为参数，接收以后把参数填补进去

#{}表名或者列名不固定

parameterType可以省略

#### 特殊字符处理

- 转义字符 \&amp;  \&gt;  \&lt;  双引号 \&quot; 单引号 \&apos;
- `<![CDATA[ < ]]>` 

### 模糊查询 分页查询

补充见 web服务端-其他细节

### 条件查询<select></select>

![image-20240925182248759](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925182248759.png)

#### 参数接收

模糊条件查询

![image-20240925184318146](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925184318146.png)

1. 散装参数，多个参数需要加@Param注解 后面要和sql语句占位符一样，最好是pojo类的属性名称
2. 对象参数：对象的属性名称要和参数占位符名称一致
3. map参数：创建一个map，里面键是sql语句的占位符，值是对象的属性名成员变量

```java
map.put("status",status);
map.put("companyName",companyName);
map.put("brandName",brandName);
```

传参数进来就是一个填字游戏详见下方的param注解

#### 多条件 动态条件查询 if

用户查询可能不会把条件全部填满：动态SQL

##### if 条件判断

test：逻辑表达式

![image-20240925184740473](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925184740473.png)

`<if test = "companyName != null and companyName != ' ' ">` sql条件字段  有选择地拼接

如果map中不包含brandname或者brandname是空的，mybatis 就不会把那一部分拼接到sql语句中，由于是直接拼接，所以不可避免会出现格式问题, 第一个条件不需要逻辑运算符。

if条件后的test 后面是参数的键名

- 可以在条件前面加个and 再用恒等式 缓冲一下
- *where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。where标签，里面加上判断标签，判断标签里面用and开头 ，如果只有一个判断标签有效，还会自动去掉and![image-20240925185741878](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925185741878.png)

#### 单条件动态查询 choose

还有 choose(when otherwise) 类似 switch-case 

![image-20240925190242016](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925190242016.png)

用户输入完就是给对象赋值，可能有些字段是空的，这会导致不能正确解析，出现where后面空的情况，此时otherwise标签里要有一个默认保底的查询方法

另外，where标签也可以代替otherwise的功能，如果检测到语法错误他会自动删除where 

where标签注意事项

![image-20240925192533201](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925192533201.png)



trim(where set) foreach



### 添加

`<insert></insert>`

autocommit 默认 false 

`sqlSession.commit()` 提交事务

`sqlSessionFactory.openSession(true)`

#### 主键返回 

 ![image-20240925221746219](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925221746219.png)

`<insert> id="addOrder" useGeneratedKeys="true" keyProperty="id" </insert>`

自动设置主键id的值，即使对象的id没有设定，也能自动设置其ID

### 修改 

`<update></update>` 

#### 修改全部字段

![image-20240925225339176](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925225339176.png)

#### 动态修改字段

`<set>`关键字

![image-20240925225816604](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925225816604.png)

### 删除

`<delete>` 关键字

#### 批量删除 

复选框删除 根据ID数组 批量删除 

![image-20240925230216448](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240925230216448.png)

foreach能遍历collection或者array，item是给每个元素起的名字，sep分隔符，open close分别代表起始符号和结束符号, ids是数组参数的param注解，如果没有param注解，就要用array

### @Param 注解 

[解决mybatis不加@Param报错 org.apache.ibatis.binding.BindingException_51CTO博客_mybatis @Param](https://blog.51cto.com/u_12393361/5021343)

对于参数，mybatis先使用map集合进行封装，每个参数都有对应的键名，解析sql语句时，遇到占位符就用值填充键的对应位置，或者在**if标签的test**、**foreach的collection标签** 中  用键名来代指参数。如果不使用参数注解，map会为参数生成默认的键，@Param注解用来代替map默认的键arg0, (param1还能用)



(-parameters可以生成元数据用于方法参数的反射，**JDK8起**在反射包中引入了java.lang.reflect.Parameter 来获取参数相关的信息。IDE编译时自动加了参数 -parameters，所以能把方法参数名给解析出来作为key)







- 对于一个基本类型参数，没有指定param，随便都可以, 加了注解就只能用param1和注解的别名



- 对于多个基本类型参数，如果没有指定param，mybatis底层会自动给予param1+arg0键，param2+arg1键，占位符需要用param1（arg0）和param2（arg1）‘’。IDE会优化，参数的键就变成自己的形参名，但如果形参的名字变化，没有注解，而且对应的占位符没有及时更新，就会出现异常

- 对于POJO对象类型参数，mybatis底层会给予对象字段键名，键名就是自己的字段名称，只需要保证字段名能和占位符一一对应。**不需要**使用param注解
- 对于map类参数，**直接使用不需要**注解，只要保证map中键名能和占位符一一对应即可。

- 对于collection类参数，至少会有arg0和colleciton两个键 ，对于list还会多一个list。key:"list"  value: userList
  - 对于数组类参数，至少会有array和arg0键名



为了映射结果一致，最好参数全部使用param进行注解，便于后期维护

#### 思考

- 如果map集合中存储一个user对象(key="user")和address对象(key="address")，但是在sql语句中要使用他们的字段，占位符应该用user.id user.name进行引用。另外一个user对象(key="user2")就需要用user2.name 进行引用。

  ```xml
  Map<String, Object> paramMap = new HashMap<>();
  paramMap.put("user", user);
  paramMap.put("address", address);
      
  <insert id="insertUserWithAddress">
      INSERT INTO tb_user (id, name, email, address)
      VALUES (#{user.id}, #{user.name}, #{user.email}, #{address.street});
  </insert>
  ```

  

- 如果是collection存储user对象，就要用foreach遍历，item 如下

  ```xml
  <insert id="insertUsers">
      INSERT INTO tb_user (id, name, email)
      VALUES
      <foreach collection="users" item="user" separator=",">
          (#{user.id}, #{user.name}, #{user.email})
      </foreach>
  </insert>
  ```

- 总结：对象map传进来不需要遍历，mybatis会根据对象对应的键找值，用`.`进行字段引用；对象collection和array需要遍历

### 注解开发

![image-20240926100629592](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926100629592.png)

[Mybatis注解开发（超详细）-CSDN博客](https://blog.csdn.net/weixin_43883917/article/details/113830667)

































