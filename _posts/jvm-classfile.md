---
name: classfile
title: ClassFile
date: 2024-09-13
tags: 
- 字节码 
categories: jvm
---

# ClassFile 字节码

Java 字节码是 Java 编译器将 Java 源代码编译成的 **中间表示格式**，它运行在 **Java 虚拟机（JVM）** 上。Java 字节码文件的扩展名为 `.class`，可以通过 `javap -c` 命令查看其内容。

任何一个Class文件都对应着唯一的一个类或接口的定义信息[1]，但是反过来说，类或 接口并不一定都得定义在文件里（譬如类或接口也可以动态生成，直接送入类加载器中）。本章中， 笔者只是通俗地将任意一个有效的类或接口所应当满足的格式称为“Class文件格式”，实际上它完全不 需要以磁盘文件的形式存在。

Class文件是一组以8个字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在文 件之中，中间没有添加任何分隔符，这使得整个Class文件中存储的内容几乎全部是程序运行的必要数 据，没有空隙存在。当遇到需要占用8个字节以上空间的数据项时，则会按照高位在前[2]的方式分割 成若干个8个字节进行存储。

<!-- more -->

## 字节码指令

**示例代码**

```java
public class BytecodeExample {
    public static int add(int a, int b) {
        return a + b;
    }

    public static void main(String[] args) {
        int result = add(2, 3);
        System.out.println(result);
    }
}
```

**编译和查看字节码指令** 

```bash
javac BytecodeExample.java
javap -c BytecodeExample
# 打印类中每个方法的反汇编代码，例如组成 Java 字节码的指令。
```

```java
public class ByteCodeExample {
  int a;

  int b;

  public static final double MAX;

  public ByteCodeExample();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static int add(int, int);
    Code:
       0: iload_0
       1: iload_1
       2: iadd
       3: ireturn

  public static void main(java.lang.String[]);
    Code:
       0: iconst_2
       1: iconst_3
       2: invokestatic  #7                  // Method add:(II)I
       5: istore_1
       6: getstatic     #13                 // Field java/lang/System.out:Ljava/io/PrintStream;
       9: iload_1
      10: invokevirtual #19                 // Method java/io/PrintStream.println:(I)V
      13: return
}
```

### <span id="code">字节码指令解释</span>

#### 构造方法：BytecodeExample()

```java
public ByteCodeExample();
    Code:
       0: aload_0           // 将 this 引用加载到操作数栈
       1: invokespecial #1  // Method java/lang/Object."<init>":()V
       4: return			// 返回
```

**说明**：自动生成的默认构造函数，调用 `Object` 的构造函数。

#### add 方法：add(int, int)

```java
public static int add(int, int);
    Code:
       0: iload_0              // 加载第一个参数 a
       1: iload_1              // 加载第二个参数 b
       2: iadd                 // 执行整数相加
       3: ireturn              // 返回结果
```

**说明**：

1. `iload_0`、`iload_1`：将参数加载到操作数栈。
2. `iadd`：弹出栈顶两个整数并执行加法运算，再将结果压入栈顶。
3. `ireturn`：将栈顶的结果返回给调用者。

#### main 方法：main(String[])

```java
public static void main(java.lang.String[]);
    Code:
       0: iconst_2           // 将常量 2 压入栈
       1: iconst_3           // 将常量 3 压入栈
       2: invokestatic  #7   // 调用静态方法 add(2, 3)
       5: istore_1           // 将结果保存到局部变量表中的索引 1
       6: getstatic     #13  // java/lang/System.out:Ljava/io/PrintStream;
       9: iload_1            // 加载变量 result
      10: invokevirtual #19  // java/io/PrintStream.println:(I)V
      13: return             // 结束 main 方法
```

**说明：**

- `iconst_2` 和 `iconst_3`：将常量 2 和 3 压入栈顶。
- `invokestatic`：调用静态方法 `add`，返回结果并压入栈顶。
- `istore_1`：将结果存入局部变量表的索引 1（变量 result）。
- `getstatic`：加载 `System.out` 对象到栈顶，用于后续方法调用。
- `invokevirtual`：调用 `println` 方法打印结果。

### 常见字节码指令表

| 指令              | 描述                            |
| ----------------- | ------------------------------- |
| **aload_x**       | 将引用变量加载到操作数栈        |
| **iload_x**       | 将 int 型变量加载到操作数栈     |
| **istore_x**      | 将 int 型值从栈顶存入局部变量表 |
| **iconst_x**      | 将常量 x 压入操作数栈           |
| **iadd**          | 执行整数加法                    |
| **isub**          | 执行整数减法                    |
| **invokestatic**  | 调用静态方法                    |
| **invokevirtual** | 调用对象的实例方法              |
| **return**        | 方法结束，返回 void             |
| **ireturn**       | 方法结束，返回 int 值           |

## .class 文件结构(ClassFile)

ClassFile 结构定义：

```java
ClassFile {
    u4             magic;// first 4 bytes of the file must be 0xCAFEBABE
    u2             minor_version;// 次版本号                        2 bytes
    u2             major_version;// 主版本号 比如 JDK 21 对应 65.0   2 bytes 
    
    u2             constant_pool_count;//常量池的数量 2 bytes
    cp_info        constant_pool[constant_pool_count-1];//常量池
    
    u2             access_flags;//Class 的访问标记   
    u2             this_class;//当前类
    u2             super_class;//父类
    
    u2             interfaces_count;//接口数量
    u2             interfaces[interfaces_count];//一个类可以实现多个接口
    u2             fields_count;//字段数量
    field_info     fields[fields_count];//一个类可以有多个字段
    u2             methods_count;//方法数量
    method_info    methods[methods_count];//一个类可以有个多个方法
    u2             attributes_count;//此类的属性表中的属性数
    attribute_info attributes[attributes_count];//属性表集合
}

```

![ClassFile 内容分析](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/16d5ec47609818fc.jpeg)

### 常量池 `cp_info constant_pool[constant_pool_count-1]` 

常量池是 ClassFile 的核心，存储类的常量信息，如字符串、字段名、方法名和方法描述符等。

计数器 constant_pool_count 表示常量池项的数量（从 1 开始计数）。（**常量池计数器是从 1 开始计数的，将第 0 项常量空出来是有特殊考虑的，索引值为 0 代表“不引用任何一个常量池项”**）。

```json
cp_info {
    u1 tag;                  // 常量类型标志      1 byte
    u1 info[];               // 常量值或引用      1 byte
}
```

常量池主要存放两大常量：字面量和符号引用。字面量比较接近于 Java 语言层面的的常量概念，如文本字符串、声明为 final 的常量值等。而符号引用则属于编译原理方面的概念。包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

常量池中每一项常量都是一个表，这 14 种表有一个共同的特点：**开始的第一位是一个 u1 类型的标志位 -tag 来标识常量的类型，代表当前这个常量属于哪种常量类型．**

|               类型               | 标志（tag） |          描述          |
| :------------------------------: | :---------: | :--------------------: |
|        CONSTANT_utf8_info        |      1      |   UTF-8 编码的字符串   |
|      CONSTANT_Integer_info       |      3      |       整形字面量       |
|       CONSTANT_Float_info        |      4      |      浮点型字面量      |
|        CONSTANT_Long_info        |      5      |      长整型字面量      |
|       CONSTANT_Double_info       |      6      |   双精度浮点型字面量   |
|       CONSTANT_Class_info        |      7      |   类或接口的符号引用   |
|       CONSTANT_String_info       |      8      |    字符串类型字面量    |
|      CONSTANT_FieldRef_info      |      9      |     字段的符号引用     |
|     CONSTANT_MethodRef_info      |     10      |   类中方法的符号引用   |
| CONSTANT_InterfaceMethodRef_info |     11      |  接口中方法的符号引用  |
|    CONSTANT_NameAndType_info     |     12      |  字段或方法的符号引用  |
|     CONSTANT_MethodType_info     |     16      |      标志方法类型      |
|    CONSTANT_MethodHandle_info    |     15      |      表示方法句柄      |
|   CONSTANT_InvokeDynamic_info    |     18      | 表示一个动态方法调用点 |

### 类访问标志 `access flags`

定义类或接口的修饰符：

| 标志值                  | 含义                       |
| ----------------------- | -------------------------- |
| 0x0001 (ACC_PUBLIC)     | 公共类。                   |
| 0x0010 (ACC_FINAL)      | 不可继承（final）。        |
| 0x0020 (ACC_SUPER)      | 支持 invokespecial。       |
| 0x0200 (ACC_INTERFACE)  | 接口。                     |
| 0x0400 (ACC_ABSTRACT)   | 抽象类或接口。             |
| 0x1000 (ACC_SYNTHETIC)  | 编译器自动生成的类或方法。 |
| 0x2000 (ACC_ANNOTATION) | 注解类。                   |
| 0x4000 (ACC_ENUM)       | 枚举类。                   |

### 当前类和父类的索引 `this_class` `super_class`

- **this_class** 指向当前类在常量池中的索引，描述类名。
- super_class 指向父类的索引。
  - 如果父类是 `java.lang.Object`，其值为 0。

#### 接口索引集合 `interfaces[interface_count]`

接口索引集合用来描述这个类实现了哪些接口，这些被实现的接口将按 `implements` (如果这个类本身是接口的话则是`extends`) 后的接口顺序从左到右排列在接口索引集合中。

### 类成员

#### 字段集合 `field_info fields[fields_count]`

```json
field_info {
    u2 access_flags;       // 访问标志字段的作用域（public ,private,protected修饰符），是实例变量还是类变量（static修饰符）,可否被序列化（transient 修饰符）,可变性（final）,可见性（volatile 修饰符，是否强制从主内存读写）。
    u2 name_index;         // 字段名称 对常量池的引用
    u2 descriptor_index;   // 描述符 对常量池的引用
    u2 attributes_count;   // 属性计数器
    attribute_info attributes[]; // 属性表
}

```

上述这些信息中，各个修饰符都是布尔值，要么有某个修饰符，要么没有，很适合使用标志位来表示。而字段叫什么名字、字段被定义为什么数据类型这些都是无法固定的，只能引用常量池中常量来描述。

字段的访问标志位：

![字段的 access_flag 的取值](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20201031084342859.png)

#### 方法表 `method_info methods[methods_count]`

```json
method_info {
    u2 access_flags;       // 
    u2 name_index;         // 方法名 对常量池的引用
    u2 descriptor_index;   // 描述符 对常量池的引用
    u2 attributes_count;   // 属性计数器
    attribute_info attributes[]; // 属性表
}
```

注意：因为`volatile`修饰符和`transient`修饰符不可以修饰方法，所以方法表的访问标志中没有这两个对应的标志，但是增加了`synchronized`、`native`、`abstract`等关键字修饰方法，所以也就多了这些关键字对应的标志。

方法的访问标志位：

![方法表的 access_flag 取值](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20201031084248965.png)

#### 属性表 `attribute_info attributes[attributes_count]`

存储与类、字段或方法相关的附加信息，如注解、调试信息等。

| 属性名             | 说明                                 |
| ------------------ | ------------------------------------ |
| Code               | 存储方法的字节码指令。               |
| ConstantValue      | 常量值属性（如 static final 常量）。 |
| LineNumberTable    | 行号表，用于调试信息映射源代码行。   |
| SourceFile         | 源文件名属性，用于调试信息。         |
| Exceptions         | 方法抛出的异常信息。                 |
| LocalVariableTable | 方法中局部变量的信息（调试用途）。   |
| Deprecated         | 标记类、方法或字段为废弃。           |

在 Class 文件，字段表，方法表中都可以携带自己的属性表集合，以用于描述某些场景专有的信息。与 Class 文件中其它的数据项目要求的顺序、长度和内容不同，属性表集合的限制稍微宽松一些，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写 入自己定义的属性信息，Java 虚拟机运行时会忽略掉它不认识的属性。

比如在上文[字节码指令解释](#code)的部分，方法的Code属性就是字节码指令

## 输出示例

```json
javap -verbose BytecodeExample // 打印有关所选类别的附加信息

Classfile /C:/Users/Lenovo/Desktop/coding/JavaSingle/BytecodeExample.class
  Last modified 2025年1月3日; size 555 bytes
  SHA-256 checksum 2ade518b0d7194939efe9f3eefc0d055e21b70bab639130f050bc35282fa976b
  Compiled from "ByteCodeExample.java"
public class ByteCodeExample
  minor version: 0
  major version: 65
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #8                          // ByteCodeExample
  super_class: #2                         // java/lang/Object
  interfaces: 0, // 实现了0个接口
  fields: 3,    // 有3个字段 （成员变量）
  methods: 3,  // 有3个方法
  attributes: 1  
Constant pool:
   #1 = Methodref          #2.#3          // java/lang/Object."<init>":()V
   #2 = Class              #4             // java/lang/Object
   #3 = NameAndType        #5:#6          // "<init>":()V
   #4 = Utf8               java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Methodref          #8.#9          // ByteCodeExample.add:(II)I
   #8 = Class              #10            // ByteCodeExample
   #9 = NameAndType        #11:#12        // add:(II)I
  #10 = Utf8               ByteCodeExample
  #11 = Utf8               add
  #12 = Utf8               (II)I
  #13 = Fieldref           #14.#15        // java/lang/System.out:Ljava/io/PrintStream;
  #14 = Class              #16            // java/lang/System
  #15 = NameAndType        #17:#18        // out:Ljava/io/PrintStream;
  #16 = Utf8               java/lang/System
  #17 = Utf8               out
  #18 = Utf8               Ljava/io/PrintStream;
  #19 = Methodref          #20.#21        // java/io/PrintStream.println:(I)V
  #20 = Class              #22            // java/io/PrintStream
  #21 = NameAndType        #23:#24        // println:(I)V
  #22 = Utf8               java/io/PrintStream
  #23 = Utf8               println
  #24 = Utf8               (I)V
  #25 = Utf8               a
  #26 = Utf8               I
  #27 = Utf8               b
  #28 = Utf8               MAX
  #29 = Utf8               D
  #30 = Utf8               ConstantValue
  #31 = Double             12.0d
  #33 = Utf8               Code
  #34 = Utf8               LineNumberTable
  #35 = Utf8               main
  #36 = Utf8               ([Ljava/lang/String;)V
  #37 = Utf8               SourceFile
  #38 = Utf8               ByteCodeExample.java
{
  int a;
    descriptor: I
    flags: (0x0000)

  int b;
    descriptor: I
    flags: (0x0000)

  public static final double MAX;
    descriptor: D
    flags: (0x0019) ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    ConstantValue: double 12.0d

  public ByteCodeExample();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static int add(int, int);
    descriptor: (II)I
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=2
         0: iload_0
         1: iload_1
         2: iadd
         3: ireturn
      LineNumberTable:
        line 6: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: iconst_2
         1: iconst_3
         2: invokestatic  #7                  // Method add:(II)I
         5: istore_1
         6: getstatic     #13                 // Field java/lang/System.out:Ljava/io/PrintStream;
         9: iload_1
        10: invokevirtual #19                 // Method java/io/PrintStream.println:(I)V
        13: return
      LineNumberTable:
        line 10: 0
        line 11: 6
        line 12: 13
}
SourceFile: "ByteCodeExample.java"
```

# ClassLoading

## One main to one JVM process

当我们启动一个Java程序，即启动一个main方法时，都将启动一个Java虚拟机进程，不管这个进程有多么复杂。而不同的JVM进程之间是不会相互影响的。这也就是为什么说，Java程序只有一个入口——main方法，让虚拟机调用。而两个main方法，对应的是2个JVM进程，启动的是两个不同的类加载器，操作的实际上是不同的类。故而不会互相影响。

## ClassLoading Workflow

当我们使用一个类，如果这个类还未加载到内存中，系统会通过加载、连接、初始化对类进行初始化。完成后可以使用Using和卸载Unloading。

### 加载(Loading)

**目的**：将类的字节码文件从持久存储加载到内存的方法区，并生成对应的 **Class 对象**。

**具体步骤**：

1. 通过类名查找 `.class` 文件，并将其二进制字节码读入内存
2. 将字节码中的静态存储结构转换为方法区中的 **运行时数据结构**。
3. 在堆内存中创建一个 **java.lang.Class** 对象，作为对方法区载入数据的访问入口。

类加载器有很多种，当我们想要加载一个类的时候，具体是哪个类加载器加载由 **双亲委派模型** 决定（不过，我们也能打破由双亲委派模型）。

每个 Java 类都有一个引用指向加载它的 `ClassLoader`。不过，数组类不是通过 `ClassLoader` 创建的，而是 JVM 在需要的时候自动创建的，数组类通过`getClassLoader()`方法获取 `ClassLoader` 的时候和该数组的元素类型的 `ClassLoader` 是一致的。

一个非数组类的加载阶段（加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，这一步我们可以去完成还可以自定义类加载器去控制字节流的获取方式（重写一个类加载器的 `loadClass()` 方法）。默认的loadClass方法最后就是Link的

### 连接(Linking)

类连接：指的是把类的二进制数据合并到 JRE 中，这又分为 3 个阶段：

#### 验证(Verification)

**目的**：检查载入Class文件数据的正确性，确保字节码文件符合 JVM 要求，不会危害虚拟机安全。

![验证阶段示意图](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/class-loading-process-verification.png)

**具体检查**：

1. **文件格式验证**：是否符合 Class 文件格式规范。
2. **元数据验证**：类继承、接口实现是否正确。
3. **字节码验证**：指令流是否合法，如变量初始化和栈操作正确。
4. **符号引用验证**：类、字段、方法等是否存在。

**结果**：不合法的字节码会抛出 **VerifyError**。

文件格式验证这一阶段是基于该类的二进制字节流进行的，主要目的是保证输入的字节流能正确地解析并存储于方法区之内，格式上符合描述一个 Java 类型信息的要求。除了这一阶段之外，其余三个验证阶段都是基于方法区的存储结构上进行的，不会再直接读取、操作字节流了。符号引用验证发生在类加载过程中的解析阶段，具体点说是 JVM 将符号引用转化为直接引用的时候（解析阶段会介绍符号引用和直接引用）。

符号引用验证的主要目的是确保解析阶段能正常执行，如果无法通过符号引用验证，JVM 会抛出异常NoSuchFieldError NoSuchMethodError IllegalAccessError。

#### 准备(Preparation)

给类的<mark>静态变量<mark>分配存储空间，并进行<mark>默认初始化，赋零值<mark>。

![基本数据类型的零值](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/基本数据类型的零值.png)

**目的**：为类的静态变量分配内存，并设置默认值（不会执行静态初始化）。

**示例**：

```java
class Test {
    static int a = 10;
}
```

**执行阶段**：

- 分配内存，并将 `a` 的初始值设为 `0`（默认值）。
- 注意：这里不会执行 `= 10`，赋值在初始化阶段完成。

从概念上讲，类变量所使用的内存都应当在 **方法区** 中进行分配。不过有一点需要注意的是：JDK 7 之前，HotSpot 使用永久代来实现方法区的时候，实现是完全符合这种逻辑概念的。 而在 JDK 7 及之后，HotSpot 已经把原本放在永久代的字符串常量池、静态变量等移动到堆中，这个时候类变量则会随着 Class 对象一起存放在 Java 堆中。

#### 解析(Resolution)

**目的**：将类的二进制数据中 常量池的 **符号引用** 替换为 **直接引用**。

- **符号引用**：类、方法、字段等以字符串形式存在于常量池中。
  - 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符 7 类符号引用进行。
  - 符号引用以一组符号来描述所引用的目标，符号可以是任何 形式的字面量，只要使用时能无歧义地定位到目标即可。符号引用与虚拟机实现的内存布局无关，引 用的目标并不一定是已经加载到虚拟机内存当中的内容。各种虚拟机实现的内存布局可以各不相同， 但是它们能接受的符号引用必须都是一致的，因为符号引用的字面量形式明确定义在《Java虚拟机规 范》的Class文件格式中。
- **直接引用**：实际内存地址或偏移量。
  - 直接引用是可以直接指向目标的指针、相对偏移量或者是一个能 间接定位到目标的句柄。直接引用是和虚拟机实现的内存布局直接相关的，同一个符号引用在不同虚 拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那引用的目标必定已经在虚拟机 的内存中存在。
  - 在程序执行方法时，系统需要明确知道这个方法所在的位置。Java 虚拟机为每个类都准备了一张方法表来存放类中所有的方法。当需要调用一个类的方法的时候，只要知道这个方法在方法表中的偏移量就可以直接调用该方法了。通过解析操作符号引用就可以直接转变为目标方法在类中方法表的位置，从而使得方法可以被调用。

### 初始化(Initialization)

对类的静态变量、静态初始化块进行初始化，因此不是必须的。**这一步 JVM 才开始真正执行类中定义的 Java 程序代码(字节码)。** 

初始化阶段就是执行类构造器`<clinit> ()`方法的过程。`<clinit> ()`并不是程序员在Java代码中直接编写 的方法，它是Javac编译器的自动生成物

对于`<clinit> ()` 方法的调用，虚拟机会自己确保其在多线程环境中的安全性。因为 `<clinit> ()` 方法是带锁线程安全，所以在多线程环境下进行类初始化的话可能会引起多个线程阻塞，并且这种阻塞很难被发现。

由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中的 语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序决定的，静态语句块中只能**访问到定义在静态语句块之前的变量，** 在前面的静态语句块可以赋值定义在其后的变量，但是不能访问它

```java
public class Test {
     static {
         i = 0; // 给变量复制可以正常编译通过
         System.out.print(i); // 这句编译器会提示“非法向前引用”
     }
     static int i = 1;
}

```

`<clinit> ()`方法与类的构造函数（即在虚拟机视角中的实例构造器`<init> ()`方法）不同，它不需要显 式地调用父类构造器，Java虚拟机会保证在子类的`<clinit> ()`方法执行前，父类的`<init> ()`方法已经执行 完毕。因此在Java虚拟机中第一个被执行的()方法的类型肯定是java.lang.Object

![clinit](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250104142317079.png)

**具体步骤**：

1. 按照声明顺序依次执行静态变量赋值和静态代码块。
2. 若父类未初始化，会先初始化父类。

**示例**：

```java
class Test {
    static int a = 10;
    static { a = 20; }
}
```

**执行结果**：`a = 20`（因为静态代码块会覆盖前面的赋值）。

**注意**：

- 初始化是类加载的最后阶段，只有在首次使用类时触发。
- 使用场景：实例化对象、调用静态方法、访问静态变量等。

#### 初始化的触发条件(主动)

关于在什么情况下需要开始类加载过程的第一个阶段“加载”，《Java虚拟机规范》中并没有进行 强制约束，这点可以交给虚拟机的具体实现来自由把握。但是对于初始化阶段，《Java虚拟机规范》 则是严格规定了有且只有六种情况必须立即对类进行“初始化”（而加载、验证、准备自然需要在此之 前开始）：

1. 遇到new、getstatic、putstatic或invokestatic这四条字节码指令时，如果类型没有进行过初始 化，则需要先触发其初始化阶段。能够生成这四条指令的典型Java代码场景有：
   1. 创建该类的实例对象
   2. 访问static变量
   3. 调用static方法
2. 反射调用时如 `Class.forName("...")`, `newInstance()` 等
   - `MethodHandle` 和 `VarHandle` 可以看作是轻量级的反射调用机制，而要想使用这 2 个调用，就必须先使用 `findStaticVarHandle` 来初始化要调用的类
3. 初始化子类会触发父类的初始化
4. 有 `default` 方法的接口实现类初始化，接口也要初始化
5. 当虚拟机启动时，用户需要定义一个要执行的主类 (包含 `main` 方法的那个类)，虚拟机会先初始化这个类。

被动访问不会触发初始化：

1. 访问类的 **常量**（`static final` 修饰）因为常量位于 运行时常量池
   - 在编译阶段通过常量传播优化，已经将常量的值“hello world”直接存储在NotInitialization类的常量池中，以后NotInitialization对常量 ConstClass.HELLOWORLD的引用，实际都被转化为NotInitialization类对自身常量池的引用了。也就是说，实际上NotInitialization的Class文件之中并没有ConstClass类的符号引用入口，这两个类在编译成 Class文件后就已不存在任何联系了。
2. 通过 **数组定义类引用**（如 `Test[] arr`）。
   - 运行之后发现没有输出“SuperClass init！”，说明并没有触发类org.fenixsoft.classloading.SuperClass的初始化阶段。但是这段代码里面触发了 另一个名为“[Lorg.fenixsoft.classloading.SuperClass”的类的初始化阶段，对于用户代码来说，这并不是 一个合法的类型名称，它是一个由虚拟机自动生成的、直接继承于java.lang.Object的子类，创建动作由 字节码指令newarray触发。
3. 对于静态字段， 只有直接定义这个字段的类才会被初始化，因此通过其子类来引用父类中定义的静态字段，只会触发 父类的初始化而不会触发子类的初始化。

（注意：一个final类型的静态属性，如果在编译时已经得到了属性值，那么调用该属性时，不会导致该类初始化，因为这个相当于使用常量；使用ClassLoader()方法，只是加载该类，并未初始化。）

### 使用(Using)

类加载完成后，可以使用类创建实例、调用方法或访问字段。

**关键点**：

- 使用过程中 JVM 可能会进行 **动态绑定**（如多态方法调用）和 **反射机制**。 

### 卸载(Unloading)

**目的**：当某些类不再被使用时，将其从内存中移除。

**条件**：

- 该类的所有实例已被回收。
- 该类的 ClassLoader 实例已被回收。
- JVM 中没有该类的任何引用。

**注意**：

- 卸载阶段只针对用户的使用自定义类加载器加载的类，Bootstrap 引导加载器加载的类不会被卸载。
- GC 会回收 **Class 对象** 和相关的元数据。

所以，在 JVM 生命周期内，由 jvm 自带的类加载器加载的类是不会被卸载的。但是由我们自定义的类加载器加载的类是可能被卸载的。

只要想通一点就好了，JDK 自带的 `BootstrapClassLoader`, `ExtClassLoader`, `AppClassLoader` 负责加载 JDK 提供的类，所以它们(类加载器的实例)肯定不会被回收。而我们自定义的类加载器的实例是可以被回收的，所以使用我们自定义加载器加载的类是可以被卸载掉的。

## ClassLoader

类加载器是一个负责加载类的对象。`ClassLoader` 是一个抽象类。给定类的二进制名称，类加载器应尝试定位或生成构成类定义的数据。典型的策略是将名称转换为文件名，然后从文件系统中读取该名称的“类文件”。负责将.class文件加载到内存中，并为之生成对应的java.lang.Class对象。

在 Java 中，一个类用其全限定类名（即包名+类名）作为标识。

而在 JVM 中，一个类用其全限定类名和其类加载器作为标识。

### dynamic lazy loading, only once

JVM 启动的时候，并不会一次性加载所有的类，而是根据需要去动态加载。也就是说，大部分类在具体用到的时候才会去加载，这样对内存更加友好。加载时机并没有明确的要求。

对于已经加载的类会被放在 `ClassLoader` 中的 `classes` 字段，这是一个存放Class对象的容器。在类加载的时候，首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。也就是说，对于一个类加载器来说，相同二进制名称的类只会被加载一次。

每个 Java 类都有一个引用指向加载它的 `ClassLoader`。不过，数组类不是通过 `ClassLoader` 创建的，而是 JVM 在需要的时候自动创建的，数组类通过`getClassLoader()`方法获取 `ClassLoader` 的时候和该数组的元素类型的 `ClassLoader` 是一致的。

```java
public abstract class ClassLoader {
  ...
  private final ClassLoader parent;
  // 由这个类加载器加载的类。
  private final Vector<Class<?>> classes = new Vector<>();
  // 由VM调用，用此类加载器记录每个已加载类。
  void addClass(Class<?> c) {
        classes.addElement(c);
   }
  ...
}
```

### 分类

- **Bootstrap ClassLoader**（引导类加载器）：最顶层的加载类，由 C++实现，通常表示为 null，并且没有父级。加载核心类库（ `%JAVA_HOME%/lib`目录下的 `rt.jar`、`resources.jar`、`charsets.jar`等 jar 包和类）以及被 `-Xbootclasspath`参数指定的路径下的所有类）。
  - `rt.jar`：rt 代表“RunTime”，`rt.jar`是 Java 基础类库，包含 Java doc 里面看到的所有的类的类文件。也就是说，我们常用内置库 `java.xxx.*`都在里面，比如`java.util.*`、`java.io.*`、`java.nio.*`、`java.lang.*`、`java.sql.*`、`java.math.*`。
- **Extension ClassLoader**（扩展类加载器）：主要负责加载 `%JRE_HOME%/lib/ext` 目录下的 jar 包和类以及被 `java.ext.dirs` 系统变量所指定的路径下的所有类。
  - Java 9 引入了模块系统，并且略微更改了上述的类加载器。Extension ClassLoader被改名为平台类加载器（Platform Classloader）。Java SE 中除了少数几个关键模块，比如说 `java.base` 是由 Bootstrap ClassLoader 加载之外，其他的模块均由 Platform Classloader 所加载。
- **Application ClassLoader**（应用类加载器）：加载应用程序的 `classpath` 下的类。
- **自定义类加载器**：用户实现的特殊需求类加载器。

除了 `BootstrapClassLoader` 是 JVM 自身的一部分之外，其他所有的类加载器都是在 JVM 外部实现的，并且全都继承自 `ClassLoader`抽象类。这样做的好处是用户可以自定义类加载器，以便让应用程序自己决定如何去获取所需的类。

### 加载器调用顺序

每个 `ClassLoader` 可以通过`getParent()`获取其父 `ClassLoader`，如果获取到 `ClassLoader` 为`null`的话，那么该类是通过 `BootstrapClassLoader` 加载的。

- 我们编写的 Java 类 `PrintClassLoaderTree` 的 `ClassLoader` 是`AppClassLoader`；
- `AppClassLoader`的父 `ClassLoader` 是 `ExtClassLoader`；
- `ExtClassLoader`的父 `ClassLoader` 是 `Bootstrap ClassLoader` 

其中，`BootstrapClassLoader`负责加载JRE的核心类库，它不是`ClassLoader`的子类，使用C++编写，因此我们在Java中看不到它，通过其子类的`getParent()`方法获取时，将返回`null`。

`ExtClassLoader`和`AppClassLoader`为`ClassLoader`的子类。在 API 中看不到它们，他们位于 rt.jar 文件中，因此由`BootstrapClassLoader`进行加载，全限定类名分别为：

- `sun.misc.Launcher$AppClassLoader` 
  - `jdk.internal.loader.ClassLoaders$AppClassLoader` 

- `sun.misc.Launcher$ExtClassLoader` 
  - `jdk.internal.loader.ClassLoaders$PlatformClassLoader` 

```java
package com.stopTalking.crazy;
public class TestClassLoader {
    public static void main(String[] args) { 
        //获取当前线程的类装载器 
    ClassLoader loader = Thread.currentThread().getContextClassLoader(); 
        //获取System类的类装载器
        ClassLoader loader1 = System.class.getClassLoader(); 
        //获取本类TestClassLoader的类加载器loader2 
        ClassLoader loader2 = TestClassLoader.class.getClassLoader(); 
        //获取loader2的父加载器 
        ClassLoader loader3 = loader2.getParent(); 
        //获取loader2的父加载器的父加载器 
        ClassLoader loader4 = loader3.getParent(); 
        System.out.println(loader); 
        System.out.println(loader1); 
        System.out.println(loader2); 
        System.out.println(loader3); 
        System.out.println(loader4); 
    } 
}
```

```java
//当前线程类获取的类加载器是AppClassLoader
jdk.internal.loader.ClassLoaders$AppClassLoader@4aa298b7
//System类为根装载器加载，java中访问不到，所以为null
null 
//本类的类加载器当然也是AppClassLoader    
jdk.internal.loader.ClassLoaders$AppClassLoader@4aa298b7
jdk.internal.loader.ClassLoaders$PlatformClassLoader@5caf905d
null
```

### 自定义 ClassLoader

需要继承 `ClassLoader`抽象类。`ClassLoader` 类有两个关键方法：

- `protected Class loadClass(String name, boolean resolve)`：加载指定二进制名称的类，实现了双亲委派机制 。`name` 为类的二进制名称，`resolve` 如果为 true，在加载时调用 `resolveClass(Class<?> c)` 方法解析该类。
- `protected Class findClass(String name)`：根据类的二进制名称来查找类，默认实现是空方法。

> 建议 `ClassLoader`的子类重写 `findClass(String name)`方法而不是`loadClass(String name, boolean resolve)` 方法。

如果我们不想打破双亲委派模型，就重写 `ClassLoader` 类中的 `findClass()` 方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。但是，如果想打破双亲委派模型则需要重写 `loadClass()` 方法。

## Parents Delegation Model

`ClassLoader` 类使用委托模型来搜索类和资源。

父辈委派模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。

`ClassLoader` 实例会在试图亲自查找类或资源之前，将搜索类或资源的任务委托给其父类加载器。

父辈委派模型并不是一种强制性的约束，只是 JDK 官方推荐的一种方式。如果我们因为某些特殊需求想要打破双亲委派模型，也是可以的，后文会介绍具体的方法。

![类加载器层次关系图](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/class-loader-parents-delegation-model-1735908352601-36.png)

另外，类加载器之间的父子关系一般不是以继承的关系来实现的，而是通常使用组合关系来复用父加载器的代码。

```java
public abstract class ClassLoader {
  ...
  // 组合
  private final ClassLoader parent;
  protected ClassLoader(ClassLoader parent) {
       this(checkCreateClassLoader(), parent);
  }
  ...
}
```

在面向对象编程中，有一条非常经典的设计原则：**组合优于继承，多用组合少用继承。**

### `java.lang.ClassLoader.loadClass()`

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        //首先，检查该类是否已经加载过，自底向上
        Class c = findLoadedClass(name);
        if (c == null) {
            //如果 c 为 null，则说明该类没有被加载过
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    //当父类的加载器不为空，则通过父类的loadClass来加载该类
                    c = parent.loadClass(name, false);
                } else {
                    //当父类的加载器为空，则调用启动类加载器来加载该类
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                //非空父类的类加载器无法找到相应的类，则抛出异常
            }

            if (c == null) {
                //当父类加载器无法加载时，则调用findClass方法来加载该类
                //用户可通过覆写该方法，来自定义类加载器
                long t1 = System.nanoTime();
                c = findClass(name);

        		//用于统计类加载器相关的信息
                PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                PerfCounter.getFindClasses().increment();
            }
        }
        
        if (resolve) {
            //对类进行连接操作
            resolveClass(c);
        }
        return c;
    }
}
```

- 在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载（每个父类加载器都会走一遍这个流程）。
- 类加载器在进行类加载的时候，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成（调用父加载器 `loadClass()`方法来加载类）。这样的话，所有的请求最终都会传送到顶层的启动类加载器 `BootstrapClassLoader` 中。
- 只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载（调用自己的 `findClass()` 方法来加载类）。
- 如果子类加载器也无法加载这个类，那么它会抛出一个 `ClassNotFoundException` 异常。

**JVM 判定两个 Java 类是否相同的具体规则**：JVM 不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样。只有两者都相同的情况，才认为两个类是相同的。即使两个类来源于同一个 `Class` 文件，被同一个虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相同。

### 优点

双亲委派模型保证了 Java 程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改。

双亲委派很好地解决了各个类 加载器协作时基础类型的一致性问题（越基础的类由越上层的加载器进行加载），基础类型之所以被 称为“基础”，是因为它们总是作为被用户代码继承、调用的API存在，但程序设计往往没有绝对不变 的完美规则，如果有基础类型又要调用回用户的代码，那该怎么办呢？

> 这并非是不可能出现的事情，一个典型的例子便是JNDI服务，JNDI现在已经是Java的标准服务， 它的代码由启动类加载器来完成加载（在JDK 1.3时加入到rt.jar的），肯定属于Java中很基础的类型 了。但JNDI存在的目的就是对资源进行查找和集中管理，它需要调用由其他厂商实现并部署在应用程 序的ClassPath下的JNDI服务提供者接口（Service Provider Interface，SPI）的代码，现在问题来了，启 动类加载器是绝不可能认识、加载这些代码的，那该怎么办？ 为了解决这个困境，Java的设计团队只好引入了一个不太优雅的设计：线程上下文类加载器 （Thread Context ClassLoader）。这个类加载器可以通过java.lang.Thread类的setContext-ClassLoader()方 法进行设置，如果创建线程时还未设置，它将会从父线程中继承一个，如果在应用程序的全局范围内 都没有设置过的话，那这个类加载器默认就是应用程序类加载器。 有了线程上下文类加载器，程序就可以做一些“舞弊”的事情了。JNDI服务使用这个线程上下文类 加载器去加载所需的SPI服务代码，这是一种父类加载器去请求子类加载器完成类加载的行为，这种行 为实际上是打通了双亲委派模型的层次结构来逆向使用类加载器，已经违背了双亲委派模型的一般性 原则，但也是无可奈何的事情。Java中涉及SPI的加载基本上都采用这种方式来完成，例如JNDI、 JDBC、JCE、JAXB和JBI等。不过，当SPI的服务提供者多于一个的时候，代码就只能根据具体提供 者的类型来硬编码判断，为了消除这种极不优雅的实现方式，在JDK 6时，JDK提供了 java.util.ServiceLoader类，以META-INF/services中的配置信息，辅以责任链模式，这才算是给SPI的加 载提供了一种相对合理的解决方案。

如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现两个不同的 `Object` 类。双亲委派模型可以保证加载的是 JRE 里的那个 `Object` 类，而不是你写的 `Object` 类。这是因为 `AppClassLoader` 在加载你的 `Object` 类时，会委托给 `ExtClassLoader` 去加载，而 `ExtClassLoader` 又会委托给 `BootstrapClassLoader`，`BootstrapClassLoader` 发现自己已经加载过了 `Object` 类，会直接返回，不会去加载你写的 `Object` 类。

自定义加载器的话，需要继承 `ClassLoader` 。如果我们不想打破双亲委派模型，就重写 `ClassLoader` 类中的 `findClass()` 方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。

但是，如果想打破双亲委派模型则需要重写 `loadClass()` 方法。重写 `loadClass()`方法之后，我们就可以改变传统双亲委派模型的执行流程。例如，子类加载器可以在委派给父类加载器之前，先自己尝试加载这个类，或者在父类加载器返回之后，再尝试从其他地方加载这个类。具体的规则由我们自己实现，根据项目需求定制化。另外，仅仅自定义加载器也不能够满足全部的要求。

[打破双亲委派模型方法 | JavaGuide](https://javaguide.cn/java/jvm/classloader.html#打破双亲委派模型方法) 



## Classpath

**Classpath** 是 Java 应用程序运行时用来查找 **类文件（.class）** 和 **资源文件** 的路径。它定义了 JVM 加载类和资源的搜索目录。**Classpath** 指定了 JVM 在加载类时搜索的目录或 JAR 包路径。

**Classpath** 是 JVM 加载类和资源的搜索路径，通常包括：

1. 当前目录 (`.`)。
2. 指定的文件夹（包含 `.class` 文件或 JAR 包）。
3. 第三方库文件（如 `lib/*.jar`）。

在开发和运行 Java 程序时，可以通过命令行、环境变量或 IDE 设置 Classpath，以确保依赖文件和类可以正确加载。

### 设置方式

1. **命令行设置：**
   使用 `-classpath` 或 `-cp` 参数指定路径：

   ```shell
   java -cp .;lib/* com.example.Main
   ```

   - **“.”** 表示当前目录。
   - **“lib/\*”** 表示 `lib` 文件夹下的所有 JAR 文件。

2. **环境变量设置：**
   设置全局环境变量：

   ```shell
   export CLASSPATH=/usr/local/app/classes:/usr/local/app/lib/*
   ```

   或 Windows 下设置：

   ```
   set CLASSPATH=.;lib\*
   ```

3. **IDE 设置：**
   在 IDE（如 IntelliJ IDEA、Eclipse）中，classpath 默认包含 **src/main/java** 和 **target/classes**，以及项目引用的依赖项。

### Classpath 路径内容

Classpath 支持以下类型的路径：

1. **目录路径：**

   - 包含编译好的类文件，如：`/home/user/classes`。

   - 示例：

     ```
     java -cp /home/user/classes Test
     ```

2. **JAR 文件路径：**

   - 支持直接引用 JAR 包：

     ```
     java -cp lib/example.jar Test
     ```

3. **通配符路径：**

   - 可使用 `*` 引用多个 JAR 包：

     ```
     java -cp lib/* Test
     ```

4. **相对路径或绝对路径：**

   - 相对路径：`./lib`（当前目录）。
   - 绝对路径：`/usr/lib/java/`.

### 默认 Classpath 设置

1. 如果未显式设置 `-classpath` 或 `CLASSPATH` 环境变量，JVM 默认搜索 **当前目录（.）**。

2. 示例：

   ```shell
   javac Test.java
   java Test
   ```

   默认会从当前目录加载 `Test.class` 文件。

### 示例 1：单个类文件

假设有以下目录结构：

```
/project/
  Main.java
```

编译与运行：

```shell
javac Main.java
java Main
```

### 示例 2：JAR 包依赖

```
/project/
  lib/
    gson.jar
  Main.java
```

编译与运行：

```shell
javac -cp lib/gson.jar Main.java
java -cp lib/gson.jar:. Main
```

### 示例 3：多个 JAR 包

```shell
java -cp "lib/*" com.example.Main
```

## 加载顺序demo

这其实是去年校招时我遇到的一道阿里巴巴的笔试题(承认有点久远了-。-)，嗯，如果我没记错的话，当时是作为java方向的一道选做大题。当然题意没有这么直白，题目只要求你写出程序运行后所有system.out.println的输出结果，其中程序是题目给的，而各个system.out.println的执行顺序不同会导致最后程序输出的结果也不同。

具体的题目我肯定记不清，不过我们可以换个直接的问法，如果类A和类B中有静态变量，静态语句块，非静态变量，非静态语句块，构造函数，静态方法，非静态方法，同时类A继承类B，请问当实例化A时，类内部的加载顺序是什么?

当时我也是一头雾水，事后我就自己写了一个小Demo，这才知道了类内部的实际加载顺，测试代码如下：

Class B:

```java
public class B{
    //静态变量 
    static int i=1;
    //静态语句块
    static {  System.out.println("Class B1:static blocks"+i);}
    //非静态变量
    int j=1;
    //静态语句块
    static{  i++;  System.out.println("Class B2:static blocks"+i);}
    //构造函数
    public B(){  
        i++;  j++;  
        System.out.println("constructor B: "+"i="+i+",j="+j);
    }
    //非静态语句块
    { 
        i++; j++; 
        System.out.println("Class B:common blocks"+"i="+i+",j="+j);
    }
    //非静态方法
    public void bDisplay(){  
        i++;  
        System.out.println("Class B:static void bDisplay(): "+"i="+i+",j="+j);  
        return ;
    }
    //静态方法
    public static void bTest(){ 
        i++;  
        System.out.println("Class B:static void bTest():  "+"i="+i);  
        return ;
    }
}
```

Class A:

```java
public class A extends B{
    //静态变量 
    static int i=1;
    //静态语句块
    static {  System.out.println("Class A1:static blocks"+i);}
    //非静态变量
    int j=1;
    //静态语句块
    static{  i++;  System.out.println("Class A2:static blocks"+i);}
    //构造函数
    public A(){  
        super();  i++;  j++;  
        System.out.println("constructor A: "+"i="+i+",j="+j);
    }
    //非静态语句块
    { 
        i++; j++; 
        System.out.println("Class A:common blocks"+"i="+i+",j="+j);
    }
    //非静态方法
    public void aDisplay(){  
        i++;  
        System.out.println("Class A:static void aDisplay(): "+"i="+i+",j="+j);  
        return ;
    }
    //静态方法
    public static void aTest(){  
        i++;  
        System.out.println("Class A:static void aTest():  "+"i="+i);  
        return ;
    }
}
```

Class ClassLoading :

```java
public class ClassLoading {      
    public static void main (String args[]) {    
        A a=new A();    
        a.aDisplay();  
    }
}
```

程序运行结果如图：

![Java中类的加载顺序剖析（常用于面试题）](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/1484270652220239.jpg)

通过上述示图，我们可以比较清晰的看出java类的整个加载过程。

1. 若要加载类A，则先加载执行其父类B(Object)的静态变量以及静态语句块(执行先后顺序按排列的先后顺序)。
2. 然后再加载执行类A的静态变量以及静态语句块。(并且1、2步骤只会执行1次)
3. 若需实例化类A，则先调用其父类B的构造函数,并且在调用其父类B的构造函数前,依次先调用父类B中的非静态变量及非静态语句块.最后再调用父类B中的构造函数初始化。
4. 然后再依次调用类A中的非静态变量及非静态语句块.最后调用A中的构造函数初始化。( 并且3、4步骤可以重复执行)
5. 而对于静态方法和非静态方法都是被动调用,即系统不会自动调用执行,所以用户没有调用时都不执行,主要区别在于静态方法可以直接用类名直接调用(实例化对象也可以),而非静态方法只能先实例化对象后才能调用。

```java
父类--静态变量
父类--静态初始化块
子类--静态变量
子类--静态初始化块
*************in main***************
父类--变量
父类--初始化块
父类--构造器
子类--变量
子类--初始化块
子类--构造器
*************second subclass***************
父类--变量
父类--初始化块
父类--构造器
子类--变量
子类--初始化块
子类--构造器
```

结果分析：

很显然在加载main方法后，静态变量不管父类还是子类的都执行了，然后才是父类和子类的的普通变量和构造器。这是因为，当要创建子类这个对象时，发现这个类需要一个父类，所以把父类的.class加载进来，然后依次初始化其普通变量和初始化代码块，最后其构造器，然后可以开始子类的工作，把子类的.class加载进来，在做子类的工作。

另外在 Java 中子类中都会有默认的调用父类的默认构造函数即super() 如果父类声明了有参构造函数，那么如果没有显式声明无参构造，子类就会爆出语法错误，无法调用父类的无参构造。

# NoClassDefFoundError vs ClassNotFoundException

`LinkageError` 表示 JVM 在加载类时，发现类与已有的类结构存在冲突或不兼容的情况。**类在链接（Linking）过程中发生了问题**。 

`NoClassDefError`: 类在编译时存在于 classpath，但是运行时却找不到这个类了。

> 排查时应检查运行环境的 classpath 设置、jar 包完整性和版本、类加载器可见性，以及静态初始化代码是否异常

> 该错误通常发生在类的隐式加载过程中，如使用 new 关键字创建实例或调用方法时触发类加载，或者在执行静态代码块或初始化静态字段时失败（例如静态初始化块抛出异常），触发 ExceptionInInitializerError，进而引发 NoClassDefFoundError；

```java
public class SomePanel extends Panel {
    static int CALC_VALUE = ValueCalcUtils.calcValue(); // calcValue 抛异常
    ... // new 对象就会发生 NoClassDefError 的问题
```

- classpath 配置错误或缺失，导致运行时找不到对应的类或 jar 包；
- jar 包冲突或版本不一致，导致加载了错误版本的类或类文件损坏；
- 可能程序的启动脚本覆盖了原来的 classpath 环境变量；
- 某些类对某些类加载器不可见

另外一种就是编译完之后，手动删除了 class 字节码，再运行肯定就找不到了

`ClassNotFoundException`: 受检异常，JVM 尝试去加载一个实际在 classpath 并不存在的类。通常由显式调用 Class.forName() 等方法加载类时抛出。

