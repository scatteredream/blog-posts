---
name: se-java
title: Java SE
date: 2024-10-01
tags: 
- oop
- io
- 动态代理
- 反射
categories: jdk

---

# 面向对象、基本语法



一个java文件内只能有1个public class 且public class名字需与文件名相同

生成类文件的名字=class名，几个类几个名字

运行的时候是按照类运行的，因此一个java文件里不同的类可以有不同的main

## 导包

在刷算法题时，Java 的常用包可以帮助你解决各种问题。以下是一些常见的包：

1. **基础包**（默认无需导入，属于 `java.lang`，直接使用）：
   - `java.lang`：包含基础类如 `String`, `Math`, `Integer`, `System`，它们是自动导入的，不需要显式导入。

2. **集合框架**：
   - `java.util.*`：包括大多数数据结构和集合类，常用于算法题：
     - `ArrayList`, `LinkedList`：动态数组和链表。
     - `HashMap`, `TreeMap`, `LinkedHashMap`：哈希表和有序映射。
     - `HashSet`, `TreeSet`：哈希集合和有序集合。
     - `PriorityQueue`：优先队列（常用于堆排序、最小堆、最大堆）。
     - `Stack`：栈结构。
     - `Deque`, `ArrayDeque`：双端队列。

3. **并发包**：
   - `java.util.concurrent.*`：多线程工具和并发数据结构。

4. **数学计算**：
   - `java.math.BigInteger` 和 `java.math.BigDecimal`：用于处理大整数和高精度小数。
   - `java.util.Random`：生成随机数。

5. **输入输出**（用于处理大规模输入输出）：
   - `java.io.*`：包括 `BufferedReader`、`InputStreamReader` 等，常用于高效输入读取。
   - `java.util.Scanner`：常用于简单的控制台输入。

6. **时间处理**：
   - `java.time.*`：处理日期和时间类（如果题目涉及日期计算）。

### 常用导入：

```java
import java.util.*; // 通用集合框架
import java.math.*;  // 高精度数学计算
import java.io.*;    // 文件、输入输出
```

这些包涵盖了绝大多数算法题目需要的工具和数据结构。如果你有特定类型的题目或需求，也可以进一步讨论合适的导入包。



## 基本数据类型 2024.9.9

### 字符

char **2** byte

路径 \\\ 转义字符

### 整数类型

0x hex 、 0 oct    byte short int long 

整数全部是int型，数字超出int范围后面要加L `long number = 666666666666L  `

### 浮点数

4e3 ⇔ 4000 这是一个浮点数 默认为double 

float 32  double 64 

浮点数全部是double型，数字如果是float型后边必须要加上f

double类型不能给float类型赋值。 char 型可以兼容整型，也可以看成16位整型

| **数据类型**           | **默认值** |
| :--------------------- | :--------- |
| byte                   | 0          |
| short                  | 0          |
| int                    | 0          |
| long                   | 0L         |
| float                  | 0.0f       |
| double                 | 0.0d       |
| char                   | '\u0000'   |
| String (or any object) | null       |
| boolean                | false      |

### Transform

- 隐式(自动) 基本数据类型中除了boolean其他都兼容

  - byte, short, char ->int->long->float->double
  - byte+short 在表达式中参与运算自动提升为int 小类型自动转换成大类型
  - 

- 强制转换

  - 小数转换成整数，保留整数部分 `double a = 1.2 ` `int b = (int) a`  

  - 四舍五入

 ```java
      double pi = Math.PI;
      pi = pi * 10000 + 0.5;
      pi = (int)pi;
      pi = pi / 10000;
      double pi_2 = Math.PI;
      long i = Math.round(pi_2 * 10000);
      pi_2 = i / 10000.0;
 ```

### 运算符

- 比较运算符 H **instanceof** Human H是否属于Human类型


- **位运算符** （整数） >>> 

- **自增自减**  i++ 的值等于i      ++i的值等于i+1

- **取模** 值只跟被模数有关 

- **扩展赋值运算符**：包含强制类型转换

- **逻辑运算符** 

  - && 第一个false 不会判断 优先级较高

  - ||  第一个true不会判断剩下的
  - ^ 异或 相同false 不同true

## API from JDK

### Switch语句简化

```java
int day = 3;
String dayName = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 4 -> "Thursday";
    case 5 -> "Friday";
    case 6 -> "Saturday";
    case 7 -> "Sunday";
    default -> throw new IllegalArgumentException("Invalid day: " + day);
};
System.out.println(dayName); // 输出: Wednesday

```

JDK12开始引入的写法，可以避免贯穿效应，并能直接在后面执行一个语句或者返回一个值

### String

#### 构造器

```java
String str = "asd" //常量池
String str = new String(字符数组、字节数组)//一般的堆
```

#### 方法

`str.toCharArray()`  返回一个字符数组

`str.charAt(2)`   返回str[2]的字符

`str1.equals(str2)` s1与s2是否相同

`str1.equalsIgnoreCase(str2)` s1与s2是否相同 **忽略大小写** 

`str.substring(0,8)` 字符串截断

`str.replace(x,y)` 将x替换成y

`str.contains("Java")` 是否包含Java

`str.startsWith("A")` 是否以A开头

`str.split(',')` Split with PERIOD ‘,’

`int compareToIgnoreCase(String str)` 按照字典顺序比较两个字符串，忽略大小写

#### 注意事项

- 用str = “aaa” 的方式，会把字符串存在**字符常量池**，内容相同只存储一份

- 用str = new String("aaa")的方式，每一次都new一个新对象存在堆中

- **String是不可变对象**，不可变性（Immutability）的含义 
- 只要对String进行操作，就要<u>创建新的字符串对象</u>，而非修改原有字符串。

```java
String str = "Hello";
str = str + " World";
```

在这个例子中，`str + " World"` 实际上创建了一个新的 `String` 对象，包含 "Hello World"，而原来的 `"Hello"` 对象仍然存在于内存中，并没有被修改。`str` 变量此时指向了这个新的对象。为什么 `String` 是不可变的？

1. **安全性（Security）**：不可变的对象可以避免在多个线程之间共享数据时的并发问题。由于 `String` 的值不会被改变，因此可以安全地在多个地方或多个线程中使用，而不必担心被其他代码修改。

2. **性能优化（String Pool）**：Java 中有一个叫做 **字符串常量池（String Pool）** 的机制，当你创建一个 `String` 对象时，如果该字符串值已经存在于常量池中，那么不会重新创建对象，而是复用已有的对象。不可变性确保了这个机制的有效性，因为相同的 `String` 实例永远不会被修改。

   ```java
   String s1 = "Hello";
   String s2 = "Hello";
   ```

   在上述代码中，`s1` 和 `s2` 实际上引用了常量池中的同一个 `String` 对象。

   **3.  哈希值缓存**：由于 `String` 对象不可变，它的哈希值只需要计算一次，可以缓存下来以提高哈希表操作的效率（如在 `HashMap` 中用作键）。如果 `String` 是可变的，那么它的哈希值也会随之变化，影响哈希表的正确性。 

![image-20240910180312621](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240910180312621.png)







### ArrayList

大小可变的 **容器**，auto extension 

![image-20240910193135775](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240910193135775.png)

```java
ArrayList<String> list = new ArrayList<>();
```

add 方法可重载 `add(Object o);` 

注意 remove 方法会让整体左移，元素对应的索引会变化，会影响遍历的索引

### 随机数

```java
Random r = new Random();

int data = r.nextInt(10)  //0-9

int data = r.nextInt(10,31)  // [10,31) 不包含31
```

### 键盘输入

```java
Scanner sc = new Scanner(System.in);

int data = sc.nextInt();
```

## 数组 2024.9.10

### 静态数组

```java
int[] data1 = new int[]{1,2,3,4};

int[] data2 = {1,2,3,4}; //简化写法
```

引用数据类型 引用相当于是对象的地址

### 动态数组（非动态长度）

```java
int[] data1 = new int[4];
```

`double 0.0` `int 0 ` `boolean false ` `String null` (<u>引用数据类型</u>) 



new一个对象，就会在堆上创建空间，然后将空间的地址传给栈中的引用

### 引用数据类型做参数

引用相当于封装好的指针，只能指向对象，可以通过它对**对象**进行操作，引用数据类型做参数，实际上是一个引用的拷贝，通过它可以操作对象，但是不能通过修改它来让真实的引用指向一个新的对象。修改对象就行了，没事修改人家引用干啥呢！

```java
public class Personnel {
    public static void modifyPerson(Person person) {
        System.out.println(person);
        person = new Person();  // 修改引用，指向新的对象  不可以
        person.name = "Alice";
        System.out.println(person);
    }
    public static void main(String[] args) {
        Person p = new Person();
        p.name = "Bob";
        System.out.println(p);
        modifyPerson(p);
        System.out.println(p);  // 仍然输出 "Bob"
    }
}
```

Java的参数都是值传递，也就是副本传递。main 中 p的值始终不变， 进入modify方法之后，一开始person也是p的值，但是无法通过更改person的指向 来 更改p的指向



### **方法的重载**：

必须要有不同的**参数列表** 

```java
void function(String... args);
void function(String [] args);
```

这两种方法是等价的

## OOP

- 类名大写
- 不用赋初值
- xx.java 中只能有一个public class 且名字必须叫xx 可以有多个class
- 对象失去引用，将成为垃圾无法被操作

- this 用在**方法内** 用于拿到当前的对象 调用方法的时候this会自动接收当前对象的引用(防止对象的成员变量和方法内部的变量名称相同产生冲突)

### 构造器

构造函数，可重载

```java
public class Student{
	public Student(){}
	public Student(String name, double score){
        this.name = name;
        this.score = score;
    }
}
```

- 创建对象的同时完成初始化赋值
- 不写构造函数会自动生成无参构造
- 如果定义了有参构造，**不会自动生成**无参构造了

### 封装

合理隐藏，合理暴露，考虑安全性

public(any)>protected(继承类)>friendly(同一package的类)>private(只有当前类才有资格访问)

### JavaBean

- 变量私有，方法公开（get set 右键快捷生成）
- 必有公开的无参构造
- **实体类**负责数据存取，处理数据交给**业务类**来完成

```java
public class StudentOperator{
	private Student student;
	public StudentOperator(Student student) {
		this.student = student;
	}
}
```

### JavaBean vs. POJO

JavaBean 和 POJO 的区别主要在于它们的用途和规范化程度：

1. **JavaBean**：
   - **定义**：JavaBean 是一种特殊的 Java 类，通常遵循严格的规范。它主要用于开发可复用的组件，尤其是在 Java EE 应用程序中，JavaBean 被广泛用于数据传输对象（DTO）、表单数据和企业级应用中。
   - **规范**：
     - 必须有一个**无参构造函数**。
     - 所有的属性（成员变量）必须是**私有的**（`private`），并通过**getter** 和 **setter** 方法进行访问。
     - 必须实现 **Serializable** 接口（可选，但常见）。
   - **应用场景**：JavaBean 常用于 Java EE（例如 JSP、Servlet）中与视图层交互的数据封装，也可用于一些持久层框架如 Hibernate 和 Spring。

2. **POJO (Plain Old Java Object)**：
   - **定义**：POJO 是一个普通的 Java 对象，没有任何特殊的要求或规范。它是最普通的 Java 类，通常只用来封装数据，避免依赖特定的框架或库。
   - **规范**：没有严格的要求，可以有构造函数、任意修饰符的属性，甚至没有 getter 和 setter 方法。
   - **应用场景**：POJO 被广泛用于各种场景中，特别是作为轻量级的数据承载类。它不依赖于任何框架的 API，目的是使代码更加简洁和易于维护。

**总结**：

- **JavaBean** 是一种受规范约束的 POJO，适合组件开发和框架集成。
- **POJO** 是一个没有任何限制或依赖的 Java 类，更加灵活自由。



### case

局部变量：方法内 一般在栈中

成员变量：一般在类的声明中表现

![image-20240910144323152](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240910144323152.png)

out

## OOP Advanced 2024.9.11

### static 关键字

#### 修饰成员变量：类变量

```java
public class Student {
	static String name;//所有类共享 （类变量）
	int age;
}
```

直接通过类名访问：`Student.name = "袁华"` 只有一份 

- 应用场景：某个数据只要一份（记住自己创建了多少个用户对象了）希望能够被共享、修改

![image-20240911105822932](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240911105822932.png)

#### 类方法：属于类的方法

```java
public class Student{
	public static void printHelloworld(){
	}	
}
Student.printHelloworld();
```

![image-20240911110754007](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240911110754007.png)

- 应用场景：工具类 提高代码复 用率

![image-20240911111331657](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240911111331657.png)

#### 其他注意

- static 方法 可以直接访问 static 成员变量 ，<u>不能访</u>问实例变量，不能有**this** 
- 实例 方法 可以直接访问 static 成员变量和实例变量、方法，可以有**this** 

##### 应用：**静态代码块 实例代码块**

静态代码块：<mark>类加载的同时<mark>会加载静态代码块但只有一次，适用于静态变量的初始化

实例代码块：<mark>创建对象的时候<mark>执行，并在构造器<mark>之前<mark>执行，适用于非静态变量的初始化



#### 设计模式：单例

- 一个类有且只有一个对象，创建对象之前就自己有了11:112311

- 私有构造器

```java
  private static A a = new A();//自有，记住一个对象
  private A(){}//
  public static A getObject(){
      return a;
  }
```

  ![image-20240911114322070](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240911114322070.png) 上图为拿到一个**对象** **以后**才开始创建对象

### 继承 extends 

关键字：extends`public class B extends A` B能继承A的非私有成员和方法

#### 子类的访问

- 子类不能直接访问父类的私有成员，但是他们**仍然存在于子类对象中**，父类的私有成员子类用get set方法可以完成访问或修改 /  (或者用父类的构造器)
- 父亲的私人物品儿子不能直接动，但是儿子能通过父亲认可的方法接触
- 子类方法中同名变量\方法的处理：优先调用方法中声明 的变量，**this**可以调用子类在方法外声明的，**super**可以调用父类声明的

![image-20240911115852654](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240911115852654.png)



| 修饰符    | 本类 | 同一包的其他类(包括同一包的子类) | 其他包的子类 | 其他包的其他类 |
| --------- | ---- | -------------------------------- | ------------ | -------------- |
| private   | OK   | **NO**                           | **NO**       | **NO**         |
| 缺省      | OK   | OK                               | **NO**       | **NO**         |
| protected | OK   | OK                               | OK           | **NO**         |
| public    | OK   | OK                               | OK           | OK             |

- 注意这些都是在对应的类 **内部** 才起作用 
- java 单继承 ， 多层继承 ， 不支持多继承

- 所有类默认继承自`Object`类

#### 子类的方法重写

- 子类将父类的方法保持参数列表相同进行重写

- 用`@Override` 注解
- 子类的访问权限必须大于父类
- `private` 和 `static` 不能重写 
- 应用：println(A) 默认调用的是`A.toString()` 这是一个Object类的函数，返回地址信息，如果在A类重写toString函数就能改变,可以使用右键快捷生成toString函数

#### 子类的<u>构造器</u>

- 调用子类的构造器之前，自动调用父类的无参构造，默认存在`super()` 

- 如果父类没有无参构造（有了有参构造），子类的构造器无法调用`super()` 就会报错，所以要在子类的构造器中手动调用 `super(name)`有参构造 

- 原因：子类中虽然不能直接操作父类的private成员，但是同样也是要有这些成员的，而且要避免一直调用父类的get set方法，那么子类构造时就要用**父类的构造器** 对父类的private成员进行初始化，再回来把对象里包含子类这部分数据赋值。

- 有参构造的重载：本来有三个参数，如果只接受两个参数，对其余的一个参数进行缺省设定，`this()` 就能调用本类的无参构造

  ```java
  public Student(String name, int age){
  	this(name, age, "heima");
  }
  public Student(String name, int age, String schoolName){
  	super(name, age);
  	this.schoolName = schoolName;
  }
  ```

- this()和super()不能同时出现在构造器中，都必须放在第一行

### 多态 polymorphism

- **对象多态**：父类的引用类型变量 可以指向子类的对象

- **行为多态**：子类中和父类**同名(重写)的方法**，在多态调用时采用子类的方法(new的是Teacher的无参构造)

- 编译看的是左边引用的类型，但是实际运行起来看的是右边的构造函数，但是多态 **不包括** 成员变量 Person p = new Student() 
- 右边对象是解耦合的，比如前面用的是Student后边想换成Person可以直接换掉
- 用父类的引用形参能接受一切子类的对象
  - 多态下不能使用子类的 **独有功能**  
    - Method1 父类中定义一个抽象方法
    - Method2 instanceof判定ifelse

#### **多态的类型转换** 

**auto** : `People p = new Teacher();` 小的自动转成大的

**force**: `Teacher t = (Teacher)p;`大的必须强制才能转成小的 

编译阶段有继承或者实现关系就不会报错，但是运行会报`ClassCastException`

Teacher有`teach() `Student 有`test()` ， 一个父类的引用person是无法调用他们的，必须类型转换

- 但是如果p实际上是Student类，是无法强制转换成Teacher类的，引出 `instanceof` 运算符，结果是一个boolean类型的变量。

- 用父类的引用形参接受子类的对象，p instanceof Teacher = true 那么调用Teacher的独有功能，否则调用Student的独有功能

### final 关键字

final 加在class上 类不能被继承， final加在方法上不能被重写，加在变量上 变量必须且仅能赋值一次   

static final 修饰的成员变量 String SCHOOL_NAME = "HEIMA" 相当于是 **常量**

常量和直接用字面量性能一样

### 抽象类 abstract 关键字

- **抽象方法** 只有方法签名，不能有具体的方法实现
- **不能创建对象** 只能作为父类让子类继承
- 如果一个类从抽象父类继承而来，除非重写完所有上一代的抽象方法，否则这个类也必须是抽象类
- 更好支持多态，父类知道子类都要做某个行为，但每个子类做的情况又不一样，父类就定义抽象方法，交给子类重写实现
- ![image-20240911172104015](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240911172104015.png)

#### 设计模式：模版方法

解决了什么问题：两个类的方法有大量的重复代码，仅仅有部分不同。

老师和学生都要写同一片作文，开头结尾相同，正文部分不一样，A B都需要`write()`且大量重复，抽象父类Father可以在`write()`写好重复的部分，插入抽象方法`body()`  并且父类的`write()`可以加`final`关键字，保证不被继承



### 接口 interface 关键字

实现类 实现 接口  重写所有抽象方法

- 方法 默认 `public` `abstract` 

- 变量 默认 `public` `` static`` `final`（常量）

- 接口 不能 实例化，不能被类继承，可被类实现，可被接口继承

- 弥补单继承，可实现多接口(类似多继承)，一个类可以拥有更多的 能力(某专业方面的能力)

- 面向接口编程，业务实现的切换灵活，解耦，AB都实现了接口Driver Driver某一天想换人，直接new B就可以，AB都实现了drive方法。你是司机，你就必须会开车，这样的话想换司机只需要换人就可以，不需要额外增加方法 `Driver driver = new A();` `Driver driver = new B();` 

- 同一个功能的多套方案：建立一个接口，用接口的抽象方法表示要实现的功能，然后分别做多个接口的实现类，把抽象方法具体化

- JDK 8 new:  均非抽象方法，

  - 默认方法 default 接口 **实现类的对象** 才可以调用 用public修饰
  - 私有方法 private 只有在接口 **内部** 才能访问
  - 静态方法 static   只能通过 **接口名调用** 用public修饰 
  - ![image-20240911221630071](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240911221630071.png)

  外部软件包访问本包接口，需要将接口设置为public

### 重载和重写

| 区别点   | 重载方法(同一个类) | 重写方法                                       |
| :------- | :----------------- | :--------------------------------------------- |
| 参数列表 | 必须修改           | 一定不能修改                                   |
| 返回类型 | 可以修改           | 一定不能修改                                   |
| 异常     | 可以修改           | 可以减少或删除，一定不能抛出新的或者更广的异常 |
| 访问     | 可以修改           | 一定不能做更严格的限制（可以降低限制）         |



### 内部类 9.12

- 创建内部类对象：`Outer.Inner in = new Outer().new Inner();` 外部类.内部类
  - 内部类的方法访问内部类的成员变量 `this`
  - Outer.this 返回外部类对象

- JDK16开始 可以定义内部类的静态成员
- 静态内部类：`public static class`
  - `Outer.Inner in = new Outer.Inner();` 
  - 不需要创建外部对象就能获得内部类
  - 可以访问外部类的静态成员

#### 匿名内部类

- 本质上是一个子类，会立即创建出一个子类对象出来

```java
Animal a = new Animal(){
	public void cry(){}
};
a.cry();

abstract class Animal{
   	public void cry();
}
```

- Animal这个抽象类不能实例化，在后面加一个大括号，然后在其中实现抽象方法，这样就创建了一个子类对象

```java

public static void main(String[] args){
    Swimming s1 = new Swimming(){
        public void swim(){
            System.out.println("Swim!");
        }
	};
    go(s1); //输出 Swim！
    go(new Swimming(){
        public void swim(){
            System.out.println("Swim!2nd");
        }
	})//输出Swim！2nd
}

public void go(Swimming s){
	System.out.println("Go!");
	s.swim();
}

interface Swimming{
	void swim();
}
```

- 应用：快速创建子类对象，用于（interface）实现类对象作形参的情况 一般是被动去用
- JFrame API: `button.addActionListener(ActionListener act)` `ActionListener` is an interface

### 枚举 enum 关键字

-  特殊的class ，私有的构造器，只能创建固定数量的实例(对象)

- 如果有抽象方法，对象必须实现方法

- 遍历 for-each语句 没有索引，不能修改元素

  ```java
    for (int number: numbers){
    	System.out.prinln(number);
    }
    for (String name : names) {
                System.out.println(name);
    }
  ```

- ![image-20240912125835772](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240912125835772.png)

枚举变量实际是常对象，用public static final 修饰，创建了X,Y,Z三个常对象，调用的是无参的构造器，也可以自己写有参构造器， 

![image-20240912130046257](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240912130046257.png)

#### 设计模式：单例

```java
public enum C{X;}//单例
```

#### 信息标注

常量输入的信息不受约束，用枚举做信息标注更好

![image-20240912131648801](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240912131648801.png)

枚举做形参，方法内直接写出对应的枚举即可

### 泛型

- 相当于是类做参数

- `ArrayList` 没有指出泛型，默认是`Object`对象

- `ArrayList<String>` 只能接受`String`类型的数据

- `ArrayList<T extends Animal>` 只能接受`Animal`子类

- `ArrayList<T implements Driver>` 只能接受`Driver`实现类

- 把数据类型作为参数传递给类型变量，相当于缺省，然后对类的成员进行赋值

- *使用 Java 泛型的概念，我们可以写一个泛型方法来对一个对象数组(`Object[] arr`)排序。然后，调用该泛型方法来对整型数组、浮点数数组、字符串数组等进行排序。* 

```java
  public interface Data<T>{
  	void add(T t);
  	ArrayList<T> getByName(String name);
  }
  class Teacher implements Data<Teacher>{
      @Override
      void add(Teacher teacher){}
      @Override
      ArrayList<Teacher> getByName(String name){
          return null;
      }
  }
  }//泛型接口
```

- 泛型方法 用`<E>`修饰，表明方法中存在缺省**类型** class E，可能在参数列表中，也可能在返回值中，也可能在具体的实现中 

```java
public static < E > void printArray( E[] inputArray ){
      // 输出数组元素            
    for ( E element : inputArray ){     
            System.out.printf( "%s ", element );
    }
         System.out.println();
}
```

![image-20240912141719516](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240912141719516.png)

- 定义的时候在返回值前加泛型，单纯使用可以在ArrayList加通配符
  - `public static <T> void go (ArrayList<T> cars)` 
  - `public static void go (ArrayList<?> cars)` 
  - `<?>`表示能接收一切类型的ArrayList 
  - `<? extends Car>`表示能接收Car以及Car子类的ArrayList，也叫上限
  - `<? super Car>`表示能接收Car以及Car父类的ArrayList ，也叫下限

- **泛型擦除：** 编译阶段工作，class字节码文件中并不存在泛型，都是将 Object 对象 强转为 E 类型

- 泛型**不支持基本数据类型**，Integer Double 类型解决

## API from JDK 2024.9.12

### Object

- Java Object 类是所有类的父类，也就是说 Java 的所有类都继承了 Object，**子类可以使用 Object 的所有方法**。

- Object 类位于 java.lang 包中，编译时会自动导入，我们创建一个类时，如果没有明确继承一个父类，那么它就会自动继承 Object，成为 Object 的子类。

- `toString() 返回字符串形式` 可重写,以返回对象的内容

- `equals(Object o) 返回boolean `   判断对象是否相等 默认比较 **地址** 可重写成 比较两个的内容是否一样

  - 先判断地址是否一样
  - 再判断是不是null/是不是同一个类
  - 再判断变量是否相等 `return this.age == student.age && Objects.equals(this.name,student.name)`  

### Object 的 clone()

- `protected o clone()` 复制一个完全相同的Object对象，内容相同，属于**浅拷贝** 调用子类方法需要强转 。我们用的类跟Object源文件不在一个包下，所以子类要用`clone()`必须要重写，通过子类`super`间接调用父类Object的`clone()`。并且子类必须是接口`Cloneable`的实现类.

- 浅拷贝：**对象中包含的其他对象**，依然指向同一个对象(相当于直接把对象的地址也复制过去了) 

- 深拷贝：浅拷贝+对象中包含的对象单独浅拷贝![image-20240912230730643](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240912230730643.png)

```java
  protected Object clone() throws CloneNotSupportedException {
          Animal a = (Animal)super.clone();
          a.arr = a.arr.clone();//数组内部没有其他的对象，
          return o;
      }
```

#### 浅拷贝：实现 Cloneable 接口，重写 clone() 方法

clone(): protected->public  +  Cloneable

```java
public class Person implements Cloneable {
    String name;
    int age;
    
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();  // 浅拷贝
    }
    
}

```

其性能好，使用 native 方法复制对象。简单，适用于成员变量都是**<u>基本类型或不可变对象</u>**的情况。`Cloneable` 接口本身没有 `clone()` 方法，是一个标记接口，没有标记会抛出 `CloneNotSupportedException`。实际上就是调用的Object的clone()，但是你不能不重写，因为这个只有在子类里才能调用，必须将原来的 protected 改成 public。

#### 深拷贝：重写 clone() 方法，可变对象引用单独调用clone()

```java
public class Person implements Cloneable {
    String name;
    Address address;
	
    @Override
    public Object clone() throws CloneNotSupportedException {
        Person cloned = (Person) super.clone();
        cloned.address = (Address) address.clone();  // 手动深拷贝引用
        return cloned;
    }
}

```

#### 使用序列化实现深拷贝（**常用于对象图很复杂时**）

```java
ObjectMapper mapper = new ObjectMapper();
Person cloned = mapper.readValue(mapper.writeValueAsString(original), Person.class);
```

#### 深拷贝：手写构造器进行复制

```java
public class Person {
    String name;
    Address address;

    public Person(Person other) {
        this.name = other.name;
        this.address = new Address(other.address);  // 深拷贝
    }
}

```

### Objects

- ![image-20240912233304677](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240912233304677.png)

- equals方法是为了避免空指针异常，s1.equals()，s1 = null 会报NullPointerException


```java
public static boolean equals(Object a, Object b){
	return (a==b)||(a != null && a.equals(b))
}
```

### 包装类 

int -> Integer      char ->Character

泛型和集合不支持基本数据类型，用包装类替代，有自动装箱和自动拆箱机制

- `Integer.toString()` 将Integer对象转换成字符串 i.toString();
- `Integer.toString(int i)` 将23这个数字转换成字符串 静态方法 
  - arr[i] + "" 数字+空串也可以转换成字符串
- `Double.parseDouble(String str)` 将str转换成 double 静态方法
- `Double.valueOf(String str)` 将str转换成 double 静态方法
- println 能加则加，不能加就一起输出

### StringBuilder

![image-20240913000059749](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240913000059749.png)

- StringBuilder重写了toString()  
- append返回对象本身，可以链式调用，s.append(1).append(2).append("!23213");
- reverse、length
- 操作字符串建议使用StringBuilder 是可变对象 效率更高
- 线程不安全。

### StringBuffer

- 线程安全

### StringJoiner

- 格式化拼接，方便快捷

![image-20240913001740989](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240913001740989.png)

```java
public static String getArr(int[] arr){ //返回数组
        StringBuilder sb = new StringBuilder();
        sb.append("[");
        for(int i=0;i<arr.length;i++){
            sb.append(arr[i]);
            if(i!=arr.length-1){
                sb.append(",");
            }
        }
        sb.append("]");
        return sb.toString();
    }
public static String getArr(int[] arr){ //返回数组
        StringJoiner sj = new StringJoiner(",","[","]");
		for(int i=0;i<arr.length;i++){
            sj.add(arr[i]+"");
        }       
        return sj.toString();
    }
```

### Math 2024.9.13

工具类，静态方法

![image-20240913134448095](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240913134448095.png)

abs ceil floor round

### System

工具类 静态方法

`System.exit(0) ` 人为停机

`System.currentTimeMillis()` 返回毫秒值 统计程序时间

### Runtime

- java程序所在的运行环境
- **单例类** 构造器私有，声明一个public static final
- exit exec

### BigDecimal

- `public BigDecimal(String val)`构造器 将double转成String类型然后付给BigDecimal

- `public static BigDecimal valueOf(double val)` 直接接一个double，据此创建一个BigDecimal对象（推荐）

- divide(另一个BD对象，精确位数，舍入模式)

  - ![image-20240913140854393](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240913140854393.png)

  - | UP        | DOWN      | CEILING  | FLOOR    | HALF_UP  | HALF_DOWN |
    | --------- | --------- | -------- | -------- | -------- | --------- |
    | 远离0方向 | 接近0方向 | 向上取整 | 向下取整 | 四舍五入 | 五舍六入  |

  - HALF_EVEN 若（**舍入位大于**5）或者（**舍入位等于**5**且前一位为奇数**），则对舍入部分的前一位数字加1；若（**舍入位小于**5）或者（**舍入位等于**5且前一位为偶数），则直接舍弃。

- 转成`double d =  bigdecimal.doubleValue()`

#### Date

![image-20240913164633885](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240913164633885.png)

##### SimpleDateFormat

### Arrays

- `toString([] arr)` 返回一个数组字符串，每个元素调用`toString()`转换成字符串，用`[,,,]`拼接成一个总字符串

-  `copyOfRange([] arr, from, to)` 返回值为数组， 内容为arr中索引 **[from, to)** 的部分

- `copyOf([] arr, int newlength)` 返回值为数组，内容为arr的内容，长度为newlength，不够的用0补齐，索引超出的截断

- `setAll`  第二个参数是接口`IntToDoubleFunction`的实现对象，重写了`applyAsDouble(int value)`方法，value是数组的索引，返回对数组中内容进行操作的结果，图中是将数组内容乘以0.8。这个对象传进来以后，对数组进行遍历，都用`applyAsDouble`的方法进行操作。

  ![image-20240913170741004](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240913170741004.png)

- `sort([] arr)` 对arr升序排序(默认升序)

- `asList()`:只能加**引用类型**，基本数据类型需要改成包装类，并且返回的`List`是不能运用`add remove clear`方法的，这个`list`是`AbstractList`的子类，如果要访问`utils`只能将返回的这个`List`添加到`new ArrayList<>()`作为有参构造

#### 对象数组如何排序？ Comparable Comparator —匿名内部类实现函数式接口

**自己指定比较规则** 

- 方法1：对象类实现`Comparable<E>`这个泛型接口，类中重写`int compareTo(E e)` 方法，将E改为自己的对象类型。

  - 约定，左边大于右边 return 正整数，小于 return 负整数，等于 return 0;（升序排序）
  - `return this.age - o.age`  注意返回值是int double可以调用`Double.compare(o1.height, o2.height)`
  - 调用`Arrays.sort(students)`即可

- 方法2：实现`Comparator`这个泛型接口，运用

  - sort有一个重载函数，第一个参数是对象数组`T[] arr`，第二个参数是`Comparator<? super T> comparator`

  - 泛型接口做参数用匿名内部类实现，类中重写`int compare(T o1,T o2)`方法，同时注意返回的是int


```java
Arrays.sort(students, new Comparator<Students>(){
    @Override
    public int compare(Student o1, Student o2){
        return Double.compare(o1.height,o2.height);//升序
    }
})
```

- ArrayList有自己的sort方法，实现同Comparator，可以免去泛型的指定

#### **Lambda 表达式—简化匿名内部类**

- 简化匿名内部类的写法，并且只能简化函数式接口(只有一个抽象方法的接口)，表示该接口的一个实现
- @FunctionalInterface

```java
  Swimming s = new Swimming(){
  	@Override
      public void swim(String name){
  		System.out.println(name + "is a swimmer~");
      }
  }
  Swimming s = (name) -> {
          System.out.println(name + "is a swimmer~");
          }
  }
```

- ![image-20240913220402634](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240913220402634.png)

- ![image-20240913220414015](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240913220414015.png)

- 参数类型可省略不写，无参数也要空括号

- 如果只有一个参数，参数列表的括号可以省略不写

- 如果实现的抽象方法只有一行代码，可以省略大括号，同时也要省略`;`，若这一行是`return`语句，return也不能写

```java
  Arrays.sort(students, new Comparator<Student>(){
     @Override
      public int compare(Student o1, Student o2){
          return Double.compare(o1.age(),o2.age());
      }
  });
  Arrays.sort(students, (Student o1,Student o2)->{
      return Double.compare(o1.age(),o2.age());
  });
  Arrays.sort(students, (o1,o2)->{return Double.compare(o1.age(),o2.age());});
  Arrays.sort(students, (o1,o2)->Double.compare(o1.age(),o2.age());
```

  

#### 方法引用

[秒懂Java之方法引用（method reference）详解-CSDN博客](https://blog.csdn.net/ShuSheng0007/article/details/107562812)

Lambda的简化 对于参数列表和返回值相同的情况

##### 类的静态方法引用/类的实例方法引用（对象本身并不作为参数）

- 静态方法引用: 调用静态方法，不需要实例，前缀是类名

- 实例方法引用: 调用已经存在的对象的实例方法，对象本身并不是参数，方法引用前缀是具体的对象

```java
  interface Comparator<E>{
  	public int compare(E o1, E o2);
  }
  class CompareByData{
  	public static int compareByAge(Student o1, Student o2){
  		return Double.compare(o1.getAge(),o2.getAge());
  	}
      public int compareByAgeDesc(Student o1, Student o2){
  		return Double.compare(o2.getAge(),o1.getAge());
  	}
      public int compareBy
      
  }
  Arrays.sort(students, (o1,o2)->CompareByData.compareByAge(o1,o2));//lambda
  Arrays.sort(students, CompareByData::compareByAge);//静态方法引用
  
  CompareByData com = new CompareByData();
  Arrays.sort(students, (o1,o2)->com.compareByAgeDesc(o1,o2));//lambda
  Arrays.sort(students, com::compareByAgeDesc);//实例方法引用,com并不是参数
```

##### 类的实例方法引用(对象本身作为参数)

- 调用实例方法，对象本身作为参数传进来，方法引用前缀为类名

```java
  Arrays.sort(names, (String o1,String o2)->o1.compareToIgnoreCase(o2));
  Arrays.sort(names, String::compareToIgnoreCase);//compareToIgnoreCase本身是String的一个实例方法
      
  lqw.lt((User o1)->o1.getAge());
  lqw.lt(User::getAge, 10);//getAge是User的一个实例方法
  
```

- 传入的对象o1是实例方法的调用者，其余的参数都是这个方法的参数

##### 构造器引用

- Lambda表达式如果只是在创建一个对象

```java
  private Student getStudent(String name, int age, BiFunction<String, Integer, Student> biFunction) {
      return biFunction.apply(name, age);
  }
  BiFunction<String, Integer, Student> s1 = new BiFunction<String, Integer, Student>() {
          @Override
          public Student apply(String name, Integer age) {
              return new Student(name, age);
          }
  };
      
  	//lambda表达式
      BiFunction<String, Integer, Student> s2 = (name, age) -> new Student(name, age);
      
  	//对应的方法引用
      BiFunction<String, Integer, Student> s3 = Student::new;
      
```

##### 方法引用/Lambda作为参数，可以在前面加上实现的函数式接口形参名，增强可读性

```java
Arrays.sort(names, (Comparator<? super String>) String::compareToIgnoreCase);
Arrays.sort(names, String::compareToIgnoreCase);
```



## Regex 正则表达式

```java
String qqCode = "12812415";
System.out.println(qqCode.matches("[1-9]\\d{5-19}"));
```

## Exception 异常处理

`Error`属于严重异常

运行时异常`RuntimeException`

编译时异常，不解决是无法运行的

### 异常类

![image-20240914213155516](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240914213155516.png)

```java
//运行时异常
//自定义异常对象
class AgeIllegalRuntimeException extends RuntimeException{
    public AgeIllegalRuntimeException(){}
    public AgeIllegalRuntimeException(String message){
        super(message);
    }
}
public static void saveAge(int age){
    if(....) {...}
    else {
        throw new AgeIllegalRuntimeException("Your age is illegal!");
    }//抛出一个异常
}
public class Main(){
    try{
        saveAge(123);//函数执行
    }
    catch(Exception e){//catch 用来接受可能抛出的异常
        e.printStackTrace();
    }
    
}
```

```java
//编译时异常 强烈提醒检查
//自定义异常对象
class AgeIllegalException extends Exception{
    public AgeIllegalException(){}
    public AgeIllegalException(String message){
        super(message);
    }
}
public static void saveAge(int age) throws AgeIllegalException{//接收方法内部可能抛出的异常
    if(....) {...}
    else {
        throw new AgeIllegalException("Your age is illegal!");//一开始会报错,throws抛出
    }//抛出一个异常
}
public class Main(){
    try{
        saveAge(183);//函数执行
    }
    catch(Exception e){//catch 用来接受可能抛出的异常
        e.printStackTrace();
    }
    
}
```

### 异常处理

`throws`:用于方法的声明当方法内部抛出指定类型的异常时，该异常会被传递给**调用该方法的代码**，并在该代码中处理异常

`try-catch`:试图接收并处理异常

`throw`:用于在当前方法中抛出一个异常，当代码执行到某个条件下无法继续正常执行时，可以使用 **throw** 关键字抛出异常，以告知调用者当前代码的执行状态

- 底层异常往外抛,最外层接受并记录,响应给用户
- 最外层接收,并在底层尝试修复
  - `sc.nextDouble()`接收一个`double`类型,如果乱输aafads就会在运行时自动抛出错误给外层
  - 只需要将外层调用的`try-catch`块用`while(true)`围起来即可

# 流

## IO 流

### File 类

file&directory

- `File(String pathname)`:有参构造, `pathname`路径分隔符` / \\ File.separator` 
  - 绝对路径
  - 相对路径：默认起始位置是工程目录`(模块)file-io-app\\src\\itheima.txt`
- `f1.length()`:返回文件,文件夹大小 字节数 文件夹是存储文件夹内部的一些文件信息
- `f3.exists()`:是否存在
- `f3.isFile()`:是否是文件
- `f3.isDirectory()`:是否是文件夹
- `f3.getName()`:获取文件名称，包含后缀
- `f3.lastModified()`:返回long 最后修改的时间
- `f1.getPath()`获取创建对象时输入的路径
- `f1.getAbsolutePath()`获取文件的绝对路径



- `f1.createNewFile()`:不存在才创建，存在就创建失败，返回`false`  没找到路径会报错
- `f1.mkdir()`:不存在才创建，存在就创建失败，返回`false` 只能创建一级目录
- `f1.mkdirs()`:可以创建多级目录，返回结果
- `f1.delete()`:删除文件、空文件夹，返回结果



- `f1.list()`:当前**文件夹**内的**一级文件名称**返回`String[]` 也包含文件夹
- `f1.listFiles()`:返回当前**文件夹**内的**一级文件对象**，返回文件对象数组`File[]`，也包含文件夹
  - 文件、路径不存在、无权限访问的文件夹 `return null`
  - 空文件夹返回长度为0的File数组
  - 隐藏文件也显示
- `renameTo(new File(file.getParent(), newName))` 改名，file是目录下子文件,`substring(from,to)`是截取`[from,to)`,`substring(from)`是截取`[from,最后]`

#### 文件搜索、非空文件夹删除

递归，数学问题最好写出表达式

eg 猴子一天吃一半还要再吃一个，10天剩余1个。前9天各自吃了多少？

$f(x)$代表第x天的桃子数量，题干可得出$f(x)/2+1 = f(x+1), f(10)=1$ 直接写递归函数即可

```java
public static int getNumberOfPeach(int n) {
        if(n==10) return 1;
        return 2*getNumberOfPeach(n+1)+2;
    }
```

[文件搜索、非空文件夹删除](C:\Users\Lenovo\IdeaProjects\MyFirstProject\src\com\it\IOtest\FileTestDemo.java) 

[递归练习](C:\Users\Lenovo\IdeaProjects\MyFirstProject\src\com\it\IOtest\RecursiveDemo.java) 



### 字符集（前置）

ASCII: 首位是0，使用1个字节存储英文数字

GBK: 1个中文字符编码成2个字节，兼容ASCII，并且对汉字规定第一个字节的第一位必须是1，与ASCII字符做区分

Unicode: 

- UTF-32 4个字节表示一个字符
- UTF-8 
  - 可变长编码方案 可以有1B 2B 3B 4B 四种长度 
  - 兼容ASCII，汉字字符占3B
  - ![image-20240916165638204](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240916165638204.png)
- Encode Decode Identical

- String提供了一些API，为字符串提供了编码和解码操作，以字节数组的形式![image-20240916170250587](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240916170250587.png)



### IO流:字节流&字符流

- `InputStream`:字节输入流:以内存为基准，来自磁盘文件/网络中的数据以字节的形式读入到内存中去的流
- `OutputStream`:字节输出流:以内存为基准，把内存中的数据以字节写出到磁盘文件或者网络中去的流
- `Reader`:字符输入流:以内存为基准，来自磁盘文件/网络中的数据以字符的形式读入到内存中去的流，
- `Writer`:字符输出流:以内存为基准，把内存中的数据以字符写出到磁盘文件或者网络介质中去的流。

![image-20240916171717591](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240916171717591.png)

#### FileInputStream 字节流

- 从文件一个字节一个字节地读取数据到内存中
- `InputStream is = new FileInputStream("........")`
- `FileInputStream(String name)`:打开实际文件的连接创建文件字节输入流，也可用`File file`做形参，`String`重载会自动帮你转成file对象。
- 一次读取一个 `int read()` 读取一个字节并返回，无数据返回-1 
  - 改进：循环`(b=is.read())!=-1`
  - 调用系统硬件资源，性能较差
  - 一次读一个，无法解决非ASCII的乱码
  - 流使用系统资源，使用完记得关闭！`is.close()`
- 一次读取多个 `int read(byte[] buffer)` 用`buffer`字节数组装字节，字节数组的长度代表每次读取的字节数，返回每次读取的字节数（`buffer`可能会装不满）无数据返回-1
  - `buffer`字节数组要转换成字符串，转成`String`可以设定转的部分，`String(buffer, 0, len)`确保如实把**这次**读到的内容转为字符串 `len`为`read`的返回值 
  - 性能得到了提升，汉字仍然乱码，会强行截断汉字
- 一次读取全部
  - `int read(byte[] buffer)`buffer大小和文件的大小字节数相同
  - `byte[] readAllBytes()` `is.readAllBytes()` 
  - 文件过大会导致内存溢出，字节数组过大，所以字节流不适合读文本，更适合做数据转移，比如文件复制

#### FileOutputStream 字节流

- 从内存一个字节一个字节地输出到文件中，目标文件自动生成
- `OutputStream os = new FileOutputStream("........")` 写文件（覆盖）
- `OutputStream os = new FileOutputStream("........", true)` 写文件（追加）
  - 换行：`"\r\n"` 将其转换成`bytes[]` 再`write`
- `os.write(int b)`把b写入文件 还是以字节形式
- `os.write(byte[] b)`把字节数组b写入文件
- `os.write(byte[] b,int off,int len)`把b写入文件, `off`表示起始的字节数组索引，`len`表示要写入的字节长度，3个汉字一共9字节

##### case: 文件复制

总结：字节流适合拷贝一切文件，因为一切文件的内容都是以字节形式存储的，一字不漏地转移所有字节，就不会出现问题

#### 释放资源的方式

原来：中间出现异常，就无法释放资源（关闭字节流）

##### try-catch-finally

```java
try{
	...
}
catch(Exception e){
    ...
}
finally{
    
}
```

不论try是否正常执行，一定执行finally，除非JVM终止`System.exit(0)`

finally可以无视`return`语句 一定执行一次，如果finally是return，原函数接受的就一定是finally里边的值，所以最好不要在finally里面return

在程序执行完成后进行资源的释放操作，finally不能访问try块里面的内容

![image-20240916200213214](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240916200213214.png)先在try块外定义流，然后try块内对流进行操作，finally块内关闭流，关闭流因为系统不知道之前是否开流会报错，用try catch块包围起来即可![image-20240916200719319](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240916200719319.png)

**臃肿**

##### try-with-resource

```java
try（
	//资源的定义;
	//资源的定义;
）{
    .....
}
catch(Exception e){
    .......
}
```

只能放置资源对象, 资源都会实现AutoClosable接口，资源放进去会自动调用close方法

![image-20240916201402673](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240916201402673.png)

#### FileReader 字符流

- 把文件数据以字符形式读到内存中去
- 构造器与`FileInputStream`类似 `Reader fr = new FileReader(String filename)`
- `int read() `每次读取一个字符并返回
- `int read(char[] buffer) `每次读取多个字符并返回，返回此次读取字符的个数

#### FileWriter 字符流

- 把字符数据写到文件中去,目标文件自动生成
- 构造器与`FileOutputStream`类似 `Writer fw = new FileWriter(String filename)`可以参数后加true表示追加
- `void write(int c)`写一个字符出去
- `void write(String str)`写一个字符串出去
- `void write(String str,int pos,int len)` 写字符串从`pos`开始长度为`len`
- `void write(char[] buffer)`写一个字符数组
- `void write(char[] buffer,int pos,int len)`写字符数组从`pos`开始长度为`len`
- 注意事项
  - 字符输出流写出数据以后，必须刷新流或者关闭流（包含刷新）才能生效。`flush() close()` 因为对文件操作比较耗费系统资源，所以都是先从管道写到缓冲区，写完之后一次性写到文件中去。

### IO流:缓冲流

对原始流进行包装，提高原始流读写数据的性能

#### 字节缓冲流

开辟缓冲区，不用一次一次写 **8KB缓冲池** 

![image-20240916231524970](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240916231524970.png)

减少调用系统资源的次数

`InputStream bis = new BufferedInputStream(InputStream is)`

`OutputStream bos = new BufferedOutputStream(OutputStream os)`

方法和原始类一样，性能有所提高，有参构造可以自定义缓冲区大小，默认8192

#### 字符缓冲流

自带8K的字符缓冲池

![image-20240916232037970](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240916232037970.png)

`BufferedReader br = new BufferedReader(Reader r)` 

不使用多态写法：新增了独有的功能

新增功能 `String readLine()` 读一整行，直到换行符，返回这行的内容，没有返回`null`

`BufferedWriter bw = new BufferedWriter(Writer r)` 

新增功能：`void newLine()`换行

### 原始&缓冲流对比

- 低级字节流一个一个字节的赋值，慢的简直让人无法忍受，直接淘汰! X
- 低级字节流一个一个字节数组的形式复制，速度较慢! 
- 缓冲流按照一个一个字节的形式复制，速度较慢 X
- 缓冲流按照一个一个字节数组的形式复制，速度极快，推荐使用!
- 性能与字节数组的大小强相关，32MB最大，用空间换时间
- 与缓冲区的大小也有关

并行&多线程 异步IO 内存映射技术 减少系统调用 不要一个字节一个字节地输入

### 字符转换流

#### InputStreamReader

`Reader isr = new InputStreamReader(InputStream is, String charset)` 把**原始<mark>字节流<mark>is**按照charset设定转换成对应字符输入流

新的字符输入流可以继续用`BufferedReader`包装

#### OutputStreamWriter

控制写出去的字符集编码

- `str.getBytes("GBK")`
- 字符输出转换流

### 打印流

PrintStream：是**字节输出流**的实现类

PrintWriter：是**字符输出流**的实现类 内部包装缓冲流

**方便高效**，所见即所得

构造器：

- `PrintStream(OutputStream os/File file/String filename)` 直接链接+包装原始流
- `PrintStream(String filename, Charset charset)` `Charset.forName("GBK")`能返回GBK的Charset
- `PrintStream(OutputStream os, boolean autoFlush, String charset)` 字符集可有可无
- **`ps.println()`:打印一行东西**
- `ps.write()`写int字节 字节数组 字节数组的一部分

自身没有追加功能，只能包装`true输出流`:

`PrintStream ps = PrintStream(new FileOutputStream(filename,true))` 

`PrintWriter pw = PrintWriter(new FileWriter(filename,true))`  

#### 拓展：Redirecting PrintStream/PrintWriter

`System.out.println()` out实际上就是一个打印流，是指向控制台的

`System.setOut(PrintStream ps)` 把系统的out设定到指定的打印流ps中

![image-20240917153902713](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917153902713.png)

### 数据流

`DataInputStream`:字节输入流的实现

构造器：包装低级的字节输入流 

`dos.writeInt(int a)` 写出去a，包括数据类型

`dis.readUTF()` 读进来字符串并返回 UTF-8

![image-20240917154727844](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917154727844.png)

### 序列化流

序列化 见 serialization.md

 字节流实现类![image-20240917154853819](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917154853819.png)![image-20240917154908066](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917154908066.png)

**First Step** 创建对象字节输出流 包装原始字节输出流:

`ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream)`

**Second Step** 调用 writeObject(具体对象)方法 

对象的类要**`implements` `Serializable`** 接口！

**Third Step** 创建对象字节输入流 包装原始字节输入流

`ObjectInputStream ois = new ObjectInputStream(new FileInputStream)` 

#### 拓展：对象中某些变量不想参与序列化

`private ` `transient` ` String password;` transient关键字

#### 拓展：一次序列化多个对象

将对象存入ArrayList中，对ArrayList进行序列化

### IO框架

框架：把编写好的类和接口编译成class形式，压缩成.jar结尾的文件发行

![image-20240917162151321](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917162151321.png)

## 特殊文件

存储有关系的数据作为系统的配置文件

### .properties 属性文件

- 存储键值对数据
- 键不重复

### Properties

![image-20240917163605651](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917163605651.png)

```java
//创建属性对象，加载文件内容
Properties properties = new Properties();
properties.load(new FileReader(filename));

//创建字符串集合，接收属性对象返回的键集合
Set<String> keys = properties.stringPropertyNames();

//遍历键集合，通过属性对象和键找到value
for(String key : keys){
	String value = properties.getProperty(key);
	properties.forEach((k,v)->System.out.println(k + v));
}
```

![image-20240917164532330](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917164532330.png)

```java
Properties properties = new Properties();
properties.setProperty("asd","123");
properties.setProperty("dfg","213");
properties.setProperty("hjk","321");
String comments = "I have saved many users!!!";
//
properties.store(new FileWriter(filename), comments);
//
```



### XML

E**X**tensible **M**arkup **L**anguage

根标签 只有一个

![image-20240917170133400](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917170133400.png)![image-20240917170306289](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917170306289.png)![image-20240917170346202](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917170346202.png)![image-20240917170514649](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917170514649.png)![image-20240917185127560](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917185127560.png)![image-20240917185753893](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917185753893.png)好

### 日志

- 系统执行信息方便记录到指定的位置（控制台，文件，数据库）
- 随时用开关控制日志启停，不需要修改源代码

![image-20240917191534986](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917191534986.png)![image-20240917191744756](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917191744756.png)![image-20240917192232262](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917192232262.png)![image-20240917192328808](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917192328808.png)

![image-20240917192715641](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917192715641.png)

`<pattern> </pattern>` 日志格式

![image-20240917192956963](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917192956963.png)

![image-20240917193034517](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917193034517.png)

文件拆分规则，保证每个不超过1MB，过去的压缩成1MB的gz文件

![image-20240917193427486](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917193427486.png)

控制日志输出情况

日志级别 trace< **debug** < info < warn < error 

![image-20240917193721056](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240917193721056.png)

# 网络编程

- `java.net.*`

CS BS架构

![image-20240918155511067](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918155511067-1726751033487-55.png)

## 网络通信基本概念

### IP

设备在网络的地址，唯一标识

**IPv4:** 32bit 点分十进制表示法    **IPv6:** 128bit 冒号分16进制表示法

**域名**代表IP，**DNS服务器**会记录域名的真实IP

**公网IP**：链接互联网 **内网IP**：局域网，内部使用

**`localhost = 127.0.0.1`**：代表本机IP，只会寻找当前所在的主机

**`ping IP地址`** 

#### InetAddress

常用方法：

![image-20240918160540846](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918160540846-1726751033487-56.png)

`getByName()`: 根据主机名 返回IP地址对象

`isReachable(int ms)` 相当于ping命令

### port

端口，应用程序在设备的地址，唯一标识

16bit 0-65535

0-1023: 预定义占用，周知端口

1024-49151: 注册端口

49152-65535: 动态分配

同一设备不能有两个程序的端口号一样

### protocol

协议，应用程序之间进行通信的规则

OSI 网络参考模型

![image-20240918161423219](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918161423219-1726751033487-57.png)

TCP/IP 事实上的国际标准

#### UDP

**U**ser **D**atagram **P**rotocol

![image-20240918161816467](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918161816467-1726751033487-58.png)

#### TCP 

**T**ransmission **C**ontrol **P**rotocol

![image-20240918161923650](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918161923650-1726751033487-59.png)

#### TCP 三次握手

第一次握手：客户端发消息，服务器收到消息，服务器知道客户端发消息没问题。

第二次握手：服务端发消息，客户端收到消息，客户端知道服务端收消息没问题，发消息也没问题。

第三次握手：客户端根据上一次握手的内容再次发出确认信息，服务器端收到信息，说明客户端收消息没问题。

全双工：双方都要确认对方同时具备收发信息的能力

不可靠信道上实现可靠的传输

**B 收到了 A 发来的消息，B 因此判断 A 具备发送能力**：

   - 这个理解是正确的。当 B 收到 A 发送的 `SYN` 报文时，B 可以判断 A 具有发送消息的能力。A 通过发送 `SYN` 表示自己希望建立连接，并告诉 B 自己的初始序列号。
   - 但需要注意的是，B 仅能判断 A 能发送数据，尚无法确定 A 能正确接收 B 发送的消息（即 A 的接收能力）。

**A 收到了 B 发回的消息，A 判断 B 具备收发信息能力**：

   - **发送能力**：正确。当 A 收到 B 发回的 `SYN-ACK` 消息后，A 可以确认 B 具备发送能力，因为 B 能够发送 `SYN-ACK` 报文。
   - **接收能力**：A 也可以推断 B 具备接收能力，因为 B 不仅发送了 `SYN-ACK`，还包含了对 A 的 `SYN` 的确认（`ACK`），说明 B 成功接收了 A 的 `SYN` 报文。
   - 因此，通过 B 的 `SYN-ACK` 报文，A 可以确认 B 既能够发送，也能够接收消息。

**B 收到了 A 发回的信息，B 因此判断 A 具备接收能力，连接建立**：

   - 这个理解也基本正确。当 B 收到 A 的 `ACK` 报文后，B 可以确认 A 的接收能力，因为 A 收到了 B 的 `SYN-ACK` 并发回了 `ACK`。如果 A 无法接收数据，就无法正确回应 B 的 `SYN-ACK` 报文。
   - 至此，B 确定 A 既能发送也能接收，双方通信能力都得到确认，连接可以正式建立。



- 在 TCP 三次握手过程中，双方都通过序列号和确认号来确认彼此的发送和接收能力，确保连接是双向可靠的。
- 三次握手的过程确保了双方的**发送能力**和**接收能力**都正常，连接才会被建立。
- 你对各个步骤的理解是对的，只需要记住，每一步都不仅仅是确认对方的发送能力，还需要通过确认号和响应确认对方的接收能力。

#### TCP 四次挥手

1. 客户端发完数据，发出断连请求
2. 服务器还没处理完最后的数据，先发一个消息，让客户端稍等
3. 处理完数据后，服务器再发一个消息确认断开连接
4. 客户端正式断开连接

![image-20240918213236026](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918213236026-1726751033487-61.png)



## [Java UDP](jetbrains://idea/navigate/reference?project=MyFirstProject&fqn=com.it.netTest.UDP.Client)

- `java.net.DatagramSocket` `java.net.DatagramPacket` 
- ![image-20240918213602137](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240918213602137-1726751033487-60.png)
- 创建DS对象，创建DP对象用于封装数据，DP对象里有字节数组，字节数组长度，目标IP,目标端口，客户端调用send方法发送数据包。
- 创建DS对象(端口号)，创建DP对象用于接收数据，DP对象里的数组长度64KB，然后服务段调用receive方法接收数据包。数据包调用getLength方法获取实际接收数据包的大小，以便正确输出数据包内数组的内容。
- getAddress()可以拿到客户端IP地址
- getPort()可以拿到客户端端口

## [Java TCP](jetbrains://idea/navigate/reference?project=MyFirstProject&path=com/JavaSE/netTest/tcp/Server.java)

`java.net.Socket` `java.net.ServerSocket`

服务端和客户端通过`socket`对象之间建立的管道进行通信，数据通过管道也就是流进行传输，而UDP是通过发送单个`datagram`数据报的形式，不需要建立联系，不需要建立稳定的管道

- 客户端：	
  - 创建`Socket sk = new Socket(hostname, port)` 数据目的地的端口号
  - `OutputStream os = sk.getOutputStream();` 把`socket`的流拿到
  - `DataOutputStream dos = new DataOutputStream(os)` 对原始的流进行包装
  - `dos.writeUTF("String sth")` 
  - 关闭`dos` 关闭`socket` 

- 服务端
  - 创建 `ServerSocket ss = new ServerSocket( port)` 注册端口，跟客户端里面的端口一样
  - `Socket socket = ss.accept()` 等待客户端发来socket连接请求，端到端，这两个socket内容其实是一样的。
  - `DataInputStream dis = new DataInputStream(socket.getInputStream());`用数据流包装原始的输入流
  - `dis.readUTF().sout`



支持一发一收 多发多收，因为只能是一个socket对应一个socket，是端到端的，要实现跟其他socket的通信只能断开连接，所以引入多线程，接到socket就开一个新线程，继续监听socket，用while循环实现。

要实现群聊，服务器可以用一个集合储存socket，如果某个socket收到了消息，就遍历集合发给所有socket（同客户端的发送代码）客户端建立socket对象之后，在while循环内部出不来，要监听

BrowserServer架构 Server要提供的内容必须符合HTTP规范

# Debug

步入：进入查看递归调用栈

跳出：跳过查看之后的递归调用栈

mask 隐藏断点。

# 单元测试

![image-20240919211712463](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240919211712463.png)

检测抛出异常

## 断言机制

Assert.assertEquals() bug提示 **期望值** 实际值 

## 自动化测试

可以一键进行所有测试，一键自动化测试

## 常用注解

@Test 测试方法

@Before(BeforeEach) 每个测试前要跑的方法

@BeforeClass(All) 修饰静态方法，所有测试方法之前，最先 这两个是初始化

@After(AfterEach) 每个测试执行完要跑的方法

@AfterClass(All) 修饰静态方法，所有测试方法之后，最后 这两个是释放资源

如果是每一个测试方法都需要一个独立的资源，就用before

# 注解

## annotation

让其他程序根据注解来决定怎么执行该程序

`@Override`:让IDE判断方法是否重写成功 

`@Test`: 让测试框架知道这个是测试方法

## 自定义注解

```java
public @interface MyTest1{
    String aaa();
    boolean bbb() default true;
    String[] ccc();
}
//value 属于特殊属性，可以只写值
//有 default 可以不用赋值
//没 default 必须赋值，带变量名和等号
public interface MyTest1 extends Annotation{
    public abstract String aaa();
    public abstract boolean bbb(){default: return true;};
    public abstract String[] ccc();
}
```

本质是接口，都继承了Annotation接口

`@MyTest1(aaa="123",bbb=false, ccc={"str","asd"})` 是一个实现类对象

## 元注解

修饰注解的注解, 在自定义注解的上面 

![image-20240919214219300](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240919214219300.png)

## 解析注解 

![image-20240919214508991](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240919214508991.png)

## 应用场景

模拟框架 解析注解，然后执行 模拟[JUnitSimulator(Toolbox 引用)](jetbrains://idea/navigate/reference?project=MyFirstProject&fqn=org.JavaSE.AdvancedTest.Annotation.JUnitSimulator)识别并执行有特定注解的方法



# Hutool Usage

## BeanUtil

### bean和Map转换

```java
Map<String,Object>BeanUtil.beanToMap(userDTO, new HashMap<>(),
                 CopyOptions.create().setFieldValueEditor((fieldName,fieldValue)->fieldValue.toString()));

BeanUtil.fillBeanWithMap(map, new UserDTO(),false);
```

## JSONUtil

### JSON和对象互转

```java
String jsonStr = JSONUtil.toJsonStr(shop);

shop = JSONUtil.toBean(jsonStr,Shop.class);
```

### JSON和数组互转

```java
String jsonArrayStr = JSONUtil.toJsonStr(list);

JSONArray jsonArray = JSONUtil.parseArray(jsonArrayStr);
shopTypeList = JSONUtil.toList(jsonArray, ShopType.class);
```

