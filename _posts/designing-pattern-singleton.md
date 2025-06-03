---
name: singleton-in-one
title: 单例模式
date: 2024-09-20
tags: 
- 单例
- 设计模式
categories: 设计模式
---

# Singleton

## 饿汉 

```java
public class Singleton {
    private static final Singleton instance = new Singleton();
    private Singleton() {}
    public static Singleton getInstance() {return instance;}
}
```

线程安全，但是可能造成资源的浪费

## <u>**懒汉 双重校验锁 DCL**</u>

1. 懒汉（延迟初始化） 要素：无参构造私有，进行判断**「线程不安全」**

```java
public class Singleton {
	private static Singleton instance;
	private Singleton() {}
	public static Singleton getInstance() {
		if (instance == null) {
			instance = new Singleton();
		}
		return instance;
	}
}
```

2. 原来的实现容易出现线程安全问题，改进：`getInstance()`变成 `synchronized` 方法。
   - **锁范围太大**：性能差，同步开销大。正常的执行路径不需要同步，只需要在创建实例的时候进行锁定即可。
   
     - 锁对象的选择：**两个不相关的方法不要使用同一把锁**。**并且要注意不能出现死锁互相等待**。
     - class：适用于保护静态变量或者类级别的共享资源，如果是为了保护某个对象的状态，就应该使用this。
     - this可能的问题：这边在内部使用this调用task.dosth()，另一边一个完全无关的业务使用 task做锁。
     - 死锁：
   
       ```java
       public class Friend {
           private final String name;
           public Friend(String name) {this.name = name;}
           public void bow(Friend other) {
               synchronized (this) {
                   System.out.println(this.name + " 正在向 " + other.name + " 鞠躬");
                   try {
                       Thread.sleep(100); // 模拟执行延迟
                   } catch (InterruptedException e) {}
                   other.bowBack(this);
               }
           }
           public void bowBack(Friend other) {
               synchronized (this) {
                   System.out.println(this.name + " 正在回礼 " + other.name);
               }
           }
       }
       // 死锁：a和b同时向对方bow，先都锁住了自己，bowBack的时候就会造成互相等待
       ```
   
   - **未二次判空**：线程 A 通过了条件判断，获取了锁，开始创建实例，就在同时，线程B也通过判断等待锁释放，在线程 A 出来之后线程B又进去创建了一次。
   - **指令重排序**：Java 允许指令重排序，创建对象可以分为 1. 分配内存地址 2. 在这块内存地址创建对象 3. 将 对象的引用 instance 指向这块地址。如果2 3的顺序颠倒，线程 A 进去之后在完全创建对象之前就将 instance 置为 非null，线程B就会以为别人已经创建好了，直接返回 instance 引用，线程B拿到以后就开始使用，其中就会随机发生空指针或者状态异常等其他难以复现调试的bug。引入 volatile，表示这个变量不允许指令重排，且线程对它的写入会立刻对其他线程可见。避免发生拿到错误对象的情况

```java
public class Singleton {
	private static volatile Singleton instance; // 1. 禁止指令重排序
	private Singleton() {}
	public static Singleton getInstance() {
		if (instance == null) {
			synchronized(Singleton.class){// 2. 锁的范围缩小
                if (instance == null){// 3. 二次判空
                    instance = new Singleton();
                }
			}			
		}
		return instance;
	}
}
```

适用场景：

- **单例模式**：延迟初始化全局唯一的对象。
- **高并发资源初始化**：如数据库连接池、线程池等。
- **框架中的扩展点加载**：如 Dubbo 的 `ExtensionLoader` 按需加载扩展类。

## <u>双重校验锁 (与类解耦)</u>

```java
public class Holder<T>{
	private volatile T instance;
    private Supplier<T> supplier;
    public Holder(Supplier<T> supplier){
        this.supplier = supplier;
    }
    public T get(){
        if(instance == null){
            synchronized(this){
                if(instance == null){
                    instance = supplier.get();
                }
            }
        }
        return instance;
    }
}
// 变种
public class Holder<T>{
    private T instance;
    public T get(){
        return instance;
    }
    public void set(T value){
        instance = value;
    }
}// 在外部实现
```



## <u>**枚举 enum**</u>

```java
public enum Singleton {
    INSTANCE;
    public void doSomething() { /* ... */ }
}
```

天然线程安全，并且能够防止反射攻击

## <u>**Holder 静态内部类 （利用类加载机制）**</u>

```java
public class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}

```

Holder 在 Singleton 加载时并不会被加载，只有第一次调用 `getInstance()` 才会加载 Holder，然后创建单例。

## CAS 自旋

```java
import java.util.concurrent.atomic.AtomicReference;

public class Singleton {
    private static final AtomicReference<Singleton> instance = new AtomicReference<>();
    private Singleton() {}
    public static Singleton getInstance() {
        for (;;) {
            Singleton current = instance.get();
            if (current != null) return current;
            current = new Singleton();
            if (instance.compareAndSet(null, current)) { // 期望值为 null，设置值为 current
                return current;
            }
        }
    }
}

```

无锁，非阻塞，线程安全。实现较为复杂，追求极致性能才考虑















