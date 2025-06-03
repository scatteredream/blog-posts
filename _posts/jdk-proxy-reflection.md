---
name: proxy-reflection
title: 反射、动态代理
date: 2025-05-20
tags:
- 动态代理
- 反射
categories: 
- jdk
---



# Reflection

加载类，允许用编程的方式，解剖类中的各种成分（万物皆对象）比如ide中创建一个对象，对象引用后加一个点就能显示可以调用的方法，说明这个类实现的方法已经提前加载好了。

反射（Reflection）是计算机编程中的一种机制，允许程序在运行时动态地检查、修改或调用其自身的结构或行为。它打破了传统静态代码的编译时绑定限制，使程序能够获取类、方法、属性等元信息，并动态操作它们。

## load

1. 加载类，获取 `Class` 对象

   - `Class c = 类名.class` 
   - `Class.forName(全名)`  
   - `Class c = 对象.getClass()` 
   - `getName() 全名带包名 getSimpleName() 简名`
   - `getMethod() getConstructor() getField()` 

2. 获取类的构造器 `Constructor` 对象，主调是 `Class 对象`

   - `getConstructors` 只有 public **返回数组**
   - `getDeclaredConstructors` 存在就能拿到
   - `getConstructor(形参的类型对象)` 只有public 拿一个
   - `getDeclaredConstructor(形参的类型对象)` 存在就能拿到 拿1个

   

   - 下面的主调是构造器对象
   - `getParameterCount`几个参数
   - `newInstance(...参数)` 返回`object` 强转为对象，如果私有构造器会报错
   - `setAccessible(true)` 暴力反射，禁止检查访问权限

3. 获取类的成员变量 `Field` 对象

   - `getFields` `getDeclaredFields` 
   - `getField(name)` `getDeclaredField(name)`
   - 下面主调是`Field`对象
   - `set(对象, 值)` `get(对象)` 赋值 取值
   - `setAccessible` 暴力反射

4. 获取类的成员方法 `Method` 对象

   - `getMethods` `getDeclaredMethods`
   - `getMethod(name, String.class, int.class) `返回值
   - 下面主调是`method`
   - `getName getParameterConut getReturnType`
   - `invoke(对象, 参数)` 返回object 强转为返回值类型
   - `setAccesible`

## 作用、应用场景

- 得到类的全部成分
- 做框架、解耦代码、提高通用性
- 支持注解处理、AOP
- IDE代码分析、断点调试依赖反射

![image-20240920001226903](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240920001226903.png)

接收任意对象，接到对象，用反射获取class对象，获取全部成员变量，遍历他们，把他们的属性写出到文件中

劣势：

- 破坏封装性+反射代码需更高权限，可能引发安全漏洞（如通过反射调用`System.exit`）
- 性能开销大，反射调用涉及动态解析类型和方法，比直接调用慢数倍
- 代码可读性差，逻辑通常冗长且难以静态分析，增加维护成本。
- 编译时检查失效：错误（如方法名拼写错误）在运行时才暴露，增加调试难度。

# 代理模式（Proxy Pattern）

> 为什么要用代理模式

- **中介隔离作用**：在某些情况下，一个客户类**不想或者不能直接引用一个委托对象**，而代理类对象可以在客户类和委托对象之间起到中介的作用，**其特征是代理类和委托类实现相同的接口。**
- **开闭原则，增加功能**：**真正的业务功能还是由委托类来实现**，但是可以在业务功能执行的前后加入一些公共的服务。例如我们想**给项目加入缓存、日志**这些功能，我们就可以使用代理类来完成，而没必要打开已经封装好的委托类。

代理模式包含如下角色：

- Subject（抽象主题角色）：定义代理类和真实主题的公共对外方法，也是代理类代理真实主题的方法；
- RealSubject（真实主题角色）：真正实现业务逻辑的类；
- Proxy（代理主题角色）：用来代理和封装真实主题；

![image-20250529222121439](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250529222121439.png)

![image-20250529222209026](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20250529222209026.png)

> 装饰器模式**在 JDK 中通过包装对象实现功能扩展，典型如 I/O 流的多层装饰和集合的功能增强，具有灵活、可复用的特点。然而代理模式关注于控制对对象的访问。**

- 使用代理模式的时候，**我们常常在一个代理类中创建一个对象的实例**。
- 当我们使用装饰器模式的时候，我们通常的做法是**将原始对象作为一个参数传给装饰者的构造器**。
- 典型：IO流中的流装饰器(数据流、缓冲流)、Collections工具类中为集合添加额外功能、打印流提供格式化输出或者更加灵活的文本输出
- 宗旨：组合优于继承，避免类爆炸

> 适配器模式主要解决接口不兼容问题，如流转换、事件监听简化等，通过 “转换” 让不同接口协同工作。

- InputStreamReader 将 InputStream 转换为 Reader
- AWT中`MouseAdapter`、`KeyAdapter` 等抽象类实现了对应事件接口的所有方法（空实现），开发者只需要重写MouseAdaptor中需要的方法，不用重写所有方法（实现Listener接口所有方法）
- `Enumeration` 是早期 JDK 的迭代接口，`Iterator` 是更现代的接口。

## 静态代理

在程序运行前就已经存在代理类的字节码文件，代理类和真实主题角色的关系在运行前就确定了。

是**由程序员创建或特定工具自动生成源代码**，在对其编译。在程序员运行之前，代理类.class文件就已经被创建了。

# 动态代理

为什么类可以动态的生成？这涉及到Java虚拟机的**类加载机制** 

> Java虚拟机类加载过程主要分为五个阶段：加载、验证、准备、解析、初始化。其中加载阶段需要完成以下3件事情：
>
> 1. 通过一个类的全限定名来获取定义此类的二进制字节流
> 2. 将这个字节流所代表的静态存储结构（字节码）转化为方法区的运行时数据结构（运行时常量池）
> 3. 在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种元数据访问入口。
>
> 由于虚拟机规范对这3点要求并不具体，所以实际的实现是非常灵活的，关于第1点，获取类的二进制字节流（class字节码）就有很多途径：
>
> 1. 从ZIP包获取，这是JAR、EAR、WAR等格式的基础
> 2. 从网络中获取，典型的应用是 Applet
> 3. 运行时计算生成，这种场景使用最多的是动态代理技术，在 java.lang.reflect.Proxy 类中，就是用了 ProxyGenerator.generateProxyClass 来为特定接口生成形式为 *$Proxy 的代理类的二进制字节流
> 4. 由其它文件生成，典型应用是JSP，即由JSP文件生成对应的Class类
> 5. 从数据库中获取等等
>
> 所以，动态代理就是想办法，根据接口或目标对象，计算出代理类的字节码，然后再加载到JVM中使用。

**动态代理又有两种典型的实现方式：JDK动态代理和CGLib动态代理**

- **通过实现接口的方式** -> JDK动态代理
- **通过继承类的方式** -> CGLIB动态代理

框架的核心技术，一个类有很多方法，需要加载资源，而代理可以代替类执行这些操作。

Spring AOP技术使用了JDK动态代理和CGLIB动态代理两种方式，在不改变原始方法的前提下对功能进行增强。

## JDK 动态代理——反射

- **JDK动态代理**：反射调用有一定性能开销，但初始化较快

- JDK原生的实现方式，需要被代理的目标类必须实现接口。因为这个技术要求**代理对象和目标对象实现同样的接口**，目标对象和代理对象是平等地位的。

### 使用

对于一个UserServiceImpl，要生成它的代理对象，为了能创建一个跟UserServiceImpl拥有同名方法的代理Proxy，<mark>这个类必须实现一个接口UserService，并且拥有Impl的全部方法</mark>，然后把Impl传给生成代理的方法。

接口能将原来的实现对象的方法抽象化（或者部分抽象） 方便代理进行重写，代理重写完具体的执行逻辑，返回的还是这个接口的实现对象，相当于是把原来的实现对象包装了一下，完美地把对象的职责转移到了代理身上，业务对象。减少代码冗余和多余的资源调用

```java
public interface Subject {
    void request();
}

public class RealSubject implements Subject {
    public void request() {
        System.out.println("Real request");
    }
}

public class JdkProxy implements InvocationHandler {
    private Object target;
    
    public Object bind(Object target) {
        this.target = target;
        return Proxy.newProxyInstance(
            target.getClass().getClassLoader(),
            target.getClass().getInterfaces(),
            this);
    }
    // proxy 是动态生成的代理类的实例，method是对应方法，args是方法参数
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before JDK proxy");
        Object result = method.invoke(target, args);
        System.out.println("After JDK proxy");
        return result;
    }
    
    public static void main(String[] args){
        Subject proxy = new JdkProxy().bind(new RealSubject());
        proxy.request();
    }
}
```

### 原理

- 基于Java反射机制实现，内存中动态生成一个代理类
- 要求目标类必须实现至少一个接口
- 在运行时动态生成接口的实现类（匿名类）

流程：<mark>通过 Proxy.newProxyInstance() 创建代理对象</mark> 

- 动态生成的代理类特征：
  - 由JDK内部的 ProxyGenerator 生成（final），通过反射获取要实现的方法的 Method 对象。
  - 继承了 `java.lang.reflect.Proxy` 类，类名通常格式为`$ProxyN`(N为数字)：实现了指定接口的所有方法。方法的实现会调用 InvocationHandler 的`invoke(proxy,method,args)`，也就是AOP切面逻辑。
    - proxy 就是 this，代理类的实例
    - 在这个invoke内部跟被代理对象`target`相关的只有 `method.invoke(target,args)`，其他逻辑都是增强行为。但是 Proxy 类内部并不存在被代理对象，因此如果需要生成真正的代理对象，`target` 的注入应该从InvocationHandler的实现类入手，跟代理类没关系。
  - 内存中动态生成代理类的字节码。默认情况下，这些动态生成的代理类不会被保存到磁盘。但可以通过设置系统属性`sun.misc.ProxyGenerator.saveGeneratedFiles`为`true`来保存生成的.class文件。

```java
public final class $Proxy0 extends Proxy implements TargetInterface {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    
    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", ...);
            m2 = Class.forName("java.lang.Object").getMethod("toString", ...);
            m3 = Class.forName("com.dream.service.TargetInterface").getMethod("targetMethod", ...);
        } catch (...) {...}
    }
    
    public $Proxy0(InvocationHandler h) {
        super(h);
    } // Proxy 父类有一个 protected 的 InvocationHandler 成员变量 h
    
    public final boolean equals(Object var1) {
        try {
            return (Boolean) super.h.invoke(this, m1, new Object[]{var1}); 
            // 转发到InvocationHandler
        } catch (...) {...}
    }
    
    public final String toString() {
        try {
            return (String) super.h.invoke(this, m2, null); 
            // 转发到InvocationHandler
        } catch (...) {...}
    }
    
    public final void targetMethod(String name) {
        try {
            super.h.invoke(this, m3, new Object[]{name}); 
            // 转发到InvocationHandler
        } catch (...) {...}
    }
    
    // 其他方法...
}
```









## CGLIB 动态代理——ASM字节码框架

- **CGLIB**：生成字节码初始化较慢，但后续调用性能更好

- 通过**继承被代理的目标类**实现代理，所以不需要目标类实现接口。


```xml
<dependency>
	<groupId>cglib</groupId>
	<artifactId>cglib-nodeps</artifactId>
	<version>3.3.0</version>
</dependency>
```

### 使用

```java
public class RealService {
    public void request() {
        System.out.println("Real service");
    }
}

public class CglibProxy implements MethodInterceptor {
    public Object getProxy(Class clazz) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        return enhancer.create();
    }
    // obj 是代理类的实例对象, methodProxy 是核心的方法代理，可以看到这里并未使用反射的method调用
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Before CGLIB proxy");
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("After CGLIB proxy");
        return result;
    }
    public static void main(String[] args){
        RealService proxy = new CglibProxy().getProxy(RealService.class);
        proxy.request();
    }
}
```

### 原理

**CGLIB 通过动态生成一个需要被代理类的子类（即被代理类作为父类），该子类重写被代理类的所有不是 final 修饰的方法，并在子类中采用方法拦截的技术拦截父类所有的方法调用，进而织入横切逻辑，** 目标对象是代理对象的父类。

- 基于ASM字节码操作框架实现。
  - **ASM框架**：一个轻量级Java字节码操作和分析框架
  - **直接生成.class文件结构**：比JDK反射方式更底层，性能更好，不过初始化需要的时间更长
- 通过继承目标类来生成其子类，不需要目标类实现接口
  - **避免反射开销**：生成的方法调用是直接调用，而非反射调用。

> ASM从类文件中读入信息后，能够改变类行为，分析类信息，甚至能够根据用户要求生成新类。
>
> ASM相对于其他类似工具如BCEL、SERP、Javassist、CGLIB，它的最大的优势就在于其性能更高，其jar包仅30K。Hibernate和Spring都使用了cglib代理，而cglib底层使用的是ASM，可见ASM在各种开源框架都有广泛的应用。[【设计模式自习室】详解代理模式-阿里云开发者社区](https://developer.aliyun.com/article/935647#slide-12) 
>
> ASM 字节码框架动态生成字节码的步骤：[字节码实践 -- 使用 ASM 实现 AOP_jdk.internal.org.objectweb.asm?-CSDN博客](https://blog.csdn.net/xifeijian/article/details/83246240) 
>
> 1. **创建ClassWriter**：使用ASM的ClassWriter开始构建类结构
> 2. **定义类信息**：设置版本号、访问修饰符、类名(继承目标类)
> 3. **添加字段**：生成用于存储 MethodInterceptor 的字段
> 4. **方法生成**：
>    - 对于每个非final方法，生成两个版本：
>      - 覆盖，也就是重写的方法(调用拦截器)
>      - 直接调用父类方法的"fast"版本(避免反射)
> 5. **添加构造函数**：初始化MethodInterceptor字段
> 6. **类加载**：使用自定义的ClassLoader加载生成的字节码
>
> 

生成的代理类class如下特征：

- 由ASM字节码操作框架生成

- 类名格式：`OriginalClass$$EnhancerByCGLIB$$<随机字符>`
- 继承自目标类，包含一个MethodInterceptor类型字段
- 重写父类的同名方法，同时有一个**直接调用**父类方法的fast方法。

```java
// 生成的代理类
public class SampleService$$EnhancerByCGLIB$$1234abcd extends SampleService {
    private MethodInterceptor interceptor;
    private static final Method CGLIB$save$0$Method; 
    private static final MethodProxy CGLIB$save$0$Proxy;
    
    static {
        // 初始化方法引用和方法代理
        CGLIB$save$0$Method = ReflectUtils.findMethods(new String[]{"save","()V"}, ...)[0];
        CGLIB$save$0$Proxy = MethodProxy.create(SampleService.class, SampleService$$EnhancerByCGLIB$$1234abcd.class, "()V", "save", "CGLIB$save$0");
    }
    
    // 生成的fast方法
    final void CGLIB$save$0() {
        super.save(); // 直接调用父类方法
    }
    
    // 重写的方法
    public final void save() {
        MethodInterceptor tmp = interceptor;
        if (tmp != null) {
            // 调用拦截器
            tmp.intercept(this, CGLIB$save$0$Method, new Object[]{}, CGLIB$save$0$Proxy);
            // CGLIB$save$0$Method是 Method, Object result = method.invoke(obj, args) 使用反射调用
            // CGLIB$save$0$Proxy是 MethodProxy, Object result = proxy.invokeSuper(obj, args) 可以看出没有使用反射调用
        } else {
            super.save(); // 无拦截器时直接调用父类
        }
    }
}
```

MethodProxy：是 **CGLIB 提供的优化版代理对象**，类型是 `net.sf.cglib.proxy.MethodProxy`。

它的作用主要是：提供一个高效的、直接调用 **super 方法** 的方式。

- 它可以用 `proxy.invokeSuper(obj, args)` 来调用被代理类的 **父类方法**，避免了反射带来的性能损耗。
  - 主要是为代理类和目标类各自生成 FastClass，fastClass 通过方法索引直接定位方法，跳过了反射查找。[源码](https://github.com/cglib/cglib/blob/9d67875290d269c9b1ff5e4f4bc578a9f05c392e/cglib/src/main/java/net/sf/cglib/proxy/MethodProxy.java#L224)
- 它内部直接生成了字节码调用，所以比 `method.invoke` 快很多。

| 比较点         | Method                         | MethodProxy                             |
| -------------- | ------------------------------ | --------------------------------------- |
| 通用性         | 适用所有类、接口               | 仅适用 CGLIB 代理子类                   |
| 反射能力       | 可以操作注解、参数、返回值等   | 主要用作高效 super 调用，不提供反射功能 |
| 调用范围       | 任意方法（只要有 Method 对象） | 只能调用当前拦截方法对应的父类实现      |
| 性能           | 反射调用，较慢                 | 字节码生成，高效调用                    |
| 接口代理可用性 | 可以                           | 不适用（只在继承结构上用）              |



## 对比

### 静态代理优缺点

- 优点：可以做到**在符合开闭原则的情况下**对目标对象进行功能扩展。
- 缺点：当需要代理多个类的时候，由于代理对象要实现与目标对象一致的接口，有两种方式：

- 只维护一个代理类，由这个代理类实现多个接口，但是这样就**导致代理类过于庞大**
- 新建多个代理类，每个目标对象对应一个代理类，但是这样会**产生过多的代理类**



### JDK动态代理优缺点

- 优势：虽然相对于静态代理，动态代理大大减少了我们的开发任务，同时减少了对业务接口的依赖，降低了耦合度。
- 劣势：**只能对接口进行代理**，使用反射调用原方法，性能稍弱



### CGLIB动态代理优缺点

**CGLIB性能比JDK性能更高**，但是**CGLIB创建对象所花费的时间却比JDK多得多**。

- 所以**对于单例的对象，因为无需频繁创建对象，用CGLIB合适**，反之使用JDK方式要更为合适一些。
- 同时由于CGLib由于是采用动态创建子类的方法，对于final修饰的方法无法进行代理。



1. **构造函数限制**：代理类会调用父类的默认构造函数
2. **体积较大**：相比JDK代理生成的字节码更复杂
3. **初始化开销**：首次生成代理类时需要较多时间

CGLIB的字节码生成机制虽然复杂，但提供了比JDK动态代理更灵活和高效的代理方式，特别适合需要代理普通Java类(而非接口)的场景。

