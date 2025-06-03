---
name: collections-framework-in-one
title: Java 集合框架
date: 2024-07-01
tags: 
- 集合
- 源码
categories: jdk
---



# 集合框架

并发集合见 JUC

![Java 集合框架概览](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/java-collection-hierarchy.png)

![](https://img2020.cnblogs.com/blog/2178658/202103/2178658-20210301212057213-1371525375.jpg)

## Collecion 单列

![Collection](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240915133525868-1735477273565-35-1735477280164-37.png)

- `List`系列：添加元素有序，可重复，有索引
  - `ArrayList`
  - `LinkedList`
- `Set`系列：添加元素无序，不重复，无索引
  - `HashSet` 无序不重复无索引
    - `LinkedHashSet` 有序不重复无索引
  - `TreeSet` 按照大小默认升序排序 不重复 无索引

### Collection Methods

- `boolean add(E e)`  `boolean isEmpty()` `boolean remove(E e)` `boolean contains(Object o)` 
- `void clear()` `int size()` 
- `Object[] toArray()`:集合colletion转换成对象**数组** （返回Object数组是为了防止添加不同类型的对象）重载的`toArray(String[] strs)` 方法能够返回一个String数组![toArray](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240914230120510.png)
- `void addAll(Collections<E> c2)` `c1.addAll(c2)` 把c2的数据全部加入c1中

### Collection 遍历

#### 迭代器

```java
Iterator<String> it = collection.iterator();//默认在第一个对象
while(it.hasNext()){//判断迭代器是否能继续指向下一个
 System.out.println(it.next()); //迭代器返回现在指向的对象，之后指向下一个对象
}
```

 最好是一次`hasNext()`对应一次`next()`

#### for-Each增强循环

```java
Collection<String> colle = new ArrayList<>();
for(String name:colle){
}//colle代表要遍历的集合名，name代表集合中每个元素的名字
String[] names = new String[]{"1","2"};
for(String name:names){
    
}
```

效果等同于迭代器Iterator

#### Lambda 表达式

```java
colle.forEach(new Consumer<String>(){
	@Override
	public void accept(String s){
	System.out.println(s);
}
})
colle.forEach(s->System.out.println(s));
colle.forEach(System.out::println);//前后参数一样
```

action已经实现了Consumer接口的accept方法

内部实现还是增强for循环，将colle集合中的元素t送到action的accept()处，相当于用元素t执行accept()方法

![forEach](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240914234023283.png)

### List 支持索引

`List<String> list = new ArrayList<>();` 

- `void add(int index, E e)`(add()重载)
- `E remove(int index)` 返回remove的数据
- `E get(int index)`返回指定索引的数据
- `E set(int index, E e)` 修改指定索引数据，返回原来的数据
- `List<E> sublist(int from, int to)` 返回一个list里面装有[from,to)部分的list

#### List 遍历

- for-Each-Loop Lambda Iterator
- for循环（支持索引）

#### ArrayList 基于数组

- 基于<u>**数组**</u>实现 **对象数组** 
- 查询速度快 (索引) O(1) 集合末端元素有时可以达到 O(1)
- 删除效率低，添加效率极低，基本都需要整体移动甚至扩容 都是 O(n) 
- 有参构造：指定长度，不够再添

适用场景：索引查询，数据量不大

数据量大还要频繁进行增删操作，不适合！

1. 动态扩容
2. 创建指定大小
3. 指定泛型，确保元素安全
4. 线程不安全

##### ArrayList **扩容** 源码分析：`grow(int minCapacity)`

三种创建方式：默认容量为 10

1. 空参：首先创建的是一个<mark>空</mark>数组 *懒加载的运用*   
2. 参数为 n：创建<mark>容量为n<mark>的对象数组，0则创建空数组
3. 参数为 collection：将 collection 的内容复制进入 新的 ArrayList 中



1. <mark>无参构造，先使用一个长度为0的对象数组<mark> 
   - `Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};`
2. <mark>添加首个元素，创建长度为10的对象数组<mark>  
   - `elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];`
3. 存满后，再添加时创建扩容1.5倍的数组，原内容加进去
   - `newCap = oldCap + oldCap >> 1`
   -  `Arrays.copyof(elementData,newCap)`
4. 一次加多个元素，addAll，1.5 倍或者10个放不下，新创建数组长度以实际为准`minCapacity`

```java
public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}
```

加入一个元素，此时元素个数size = 0，存放元素的数组容量 length = 0，因此正好符合扩容条件：`grow(int minCapacity)` 对于普通的add，此处 `minCapacity = size + 1`，也就是现元素个数+1

**<mark>扩容逻辑<mark>**

空数组扩容到`max`[ <mark>10<mark> , `minCapacity` ], `minCapacity`是用来应对`addAll()`的

非空数组扩容到<mark>原来的1.5倍<mark>，当然1.5倍导致溢出则扩容到minCapacity即可

```java
private Object[] grow() {
    return grow(size + 1);
}
private Object[] grow(int minCapacity) {
    // 最小容量应为size+1, length为数组的大小
    int oldCapacity = elementData.length;//当前大小
    // 1. 如果是无参构造就是空数组: DEFAULTCAPACITY_EMPTY_ELEMENTDATA
    if (oldCapacity > 0 || 
    	elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth  */
                oldCapacity >> 1  /* preferred GROWTH 0.5倍 原长度*/);
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        // 1.1 上面是空数组，那就是默认容量 10 和 size+1 进行比较 创建比较大的那个
        return elementData = new 
            Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
public static int newLength(int oldLength,int minGrowth,int prefGrowth) {
    // preconditions not checked because of inlining
    // assert oldLength >= 0
    // assert minGrowth > 0

    int prefLength = oldLength + Math.max(minGrowth, prefGrowth); // might overflow
    if (0 < prefLength && prefLength <= SOFT_MAX_ARRAY_LENGTH) {
        return prefLength;
    } else {
        // put code cold in a separate method
        return hugeLength(oldLength, minGrowth);
    }
}
```

理论上来说，最好在向 `ArrayList` 添加大量元素之前用 `ensureCapacity(minCapacity)` 方法，以减少增量重新分配的次数

##### ~~Vector(Deprecated)~~

`ArrayList` 是 `List` 的主要实现类，底层使用 `Object[]`存储，适用于频繁的查找工作，线程不安全 。

`Vector` 是 `List` 的古老实现类，底层使用`Object[]` 存储，线程安全，但是并发性能较差。

##### ~~Stack(Deprecated)~~

- `Vector` 和 `Stack` 两者都是线程安全的，都是使用 `synchronized` 关键字进行同步处理。
- `Stack` 继承自 `Vector`，是一个后进先出的栈，而 `Vector` 是一个列表。

随着 Java 并发编程的发展，`Vector` 和 `Stack` 已经被淘汰，推荐使用并发集合类（例如 `ConcurrentHashMap`、`CopyOnWriteArrayList` 等）或者手动实现线程安全的方法来提供安全的多线程操作支持。

#### LinkedList 基于链表

- 基于<u>双向链表</u>实现，比单链表快
- 查询速度慢O(n)，**对首尾元素操作极快** O(1)
- 添加和删除不需要扩容，位移 不过还是O(n)的时间复杂度
- 线程不安全

![unlink 方法逻辑](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/linkedlist-unlink.jpg)

新增双链尾首尾特有方法：

![LinkedList](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240915001950393.png)

适用场景：

- 对首尾的操作性能很高，LinkedList可以用来实现先进先出(FIFO)的 **队列**

  - `LinkedList queue = new LinkedList<>();`

  - `Enqueue`⇔`addLast` `Dequeue`⇔`removeFirst`

- 可以实现**Stack** 栈

  - `LinkedList stack = new LinkedList<>();`

  - `push`⇔`addFirst` `Pop`⇔`removeFirst`
  - `push` `pop`方法已经由官方写入API可直接调用

### Queue：FIFO

#### 常用方法

add/remove实际上是对offer/poll的封装

1. **添加元素的方法**

| 方法             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| **`add(E e)`**   | 将指定元素插入队列，如果队列已满，则抛出 `IllegalStateException` 异常。 |
| **`offer(E e)`** | 将指定元素插入队列，如果队列已满，则返回 `false` 而不抛异常。 |

**2. 移除元素的方法**

| 方法           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| **`remove()`** | 移除队列头部的元素，如果队列为空，则抛出 `NoSuchElementException` 异常。 |
| **`poll()`**   | 移除队列头部的元素，如果队列为空，则返回 `null`。            |

**3. 查看元素的方法**

| 方法            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| **`element()`** | 查看队列头部的元素，不移除。如果队列为空，则抛出 `NoSuchElementException` 异常。 |
| **`peek()`**    | 查看队列头部的元素，不移除。如果队列为空，则返回 `null`。    |

#### Deque

- `Queue` 是单端队列，只能从一端插入元素，另一端删除元素，实现上一般遵循FIFO
- `Deque` 是双端队列，在队列的两端均可以插入或删除元素，LinkedList就实现了Deque，因此可以用来模拟栈和队列
  - Deque 常用的方法就是对头尾元素的 `add/remove/get` `offer/poll/peek`，前者会抛异常，后者不会抛异常只会返回 false(offer) 或者 null(poll/peek) 。
  - pop = removeFirst   push = addFirst。


##### ArrayDeque

基于可变长的数组和双指针来实现，`ArrayDeque` 插入时可能存在扩容过程, 不过均摊后的插入操作依然为 O(1)。虽然 `LinkedList` 不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢。

#### PriorityQueue

`PriorityQueue` 是在 JDK1.5 中被引入的, 其与 `Queue` 的区别在于元素出队顺序是与优先级相关的，即总是优先级最高的元素先出队。

这里列举其相关的一些要点：

- `PriorityQueue` 利用了二叉堆的数据结构来实现的，底层使用可变长的数组来存储数据
- `PriorityQueue` 通过堆元素的上浮和下沉，实现了在 O(logn) 的时间复杂度内插入元素和删除堆顶元素。
- `PriorityQueue` 是非线程安全的，且不支持存储 `NULL` 和 `non-comparable` 的对象。
- `PriorityQueue` 默认是小顶堆，但可以接收一个 `Comparator` 作为构造参数，从而来自定义元素优先级的先后。

`PriorityQueue` 在面试中可能更多的会出现在手撕算法的时候，典型例题包括堆排序、求第 K 大的数、带权图的遍历等，所以需要会熟练使用才行。

#### BlockingQueue

详见 JUC

### Set 不重复

`Set<Integer> set = new HashSet();`//无序

`Set<Integer> set = new LinkedHashSet();` //有序

`Set<Integer> set = new TreeSet()` //排序

#### HashSet 无序

- 每个对象都有哈希值，int类型，通过`hashCode()`返回
- 也可能相同，大部分情况下是相同的
- 增删改查性能较好，类比查字典，只要看到偏旁就能定位大概的位置
- ![JDK 8 之前的 HashSet](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240915124422018.png)数据过多会导致链表过长，查询性能降低，然后就扩容，加载因子0.75*16=12，占到12个数据就开始扩容，2倍大小
- ![HashSet 底层结构](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240915125104423.png)
- 二叉搜索树：小的存左边，大的存右边，一样的不存
- 平衡二叉树：左右高度差不超过1
- [红黑树](https://javaguide.cn/cs-basics/data-structure/red-black-tree.html)：自平衡的二叉搜索树 
- 无序，不重复，无索引！
  - 内容一样的两个对象s1s2，HashSet认为他们不一样
  - 对于HashSet可以重写对象类的`equals()`方法，比较对象的内容而不是地址，重写`hashCode()`方法根据对象的内容计算哈希值。![equals hashcode](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240915125944598.png)

##### 去重原理：Hashmap put(k,v)

```java
// Returns: true if this set did not already contain the specified element
// 返回值：当 set 中没有包含 add 的元素时返回真
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
}

```

而 map 的 put ，只有在key不存在时返回null，其他时候返回旧值，因此key不存在正好能够去重

#### LinkedHashSet 有序

- 在HashSet基础上，每个元素多了一个双链表机制记录前后位置，原来链表依然存在，双链表仅用来记录**<mark>添加<mark>先后顺序**
- 占用内存相对多

#### TreeSet 可自定义排序

- 不重复无索引，**可排序**。底层红黑树
- 对数值类型按照大小升序排序，对字符串类型按照首字符编号升序排序
- 自定义`Student`对象无法直接排序
  - 1. 让`Student`类实现`Comparable`接口，重写`int compareTo()`方法
  - 2. `TreeSet`有参构造，用`Comparator`实现对象指定比较规则，2规则优先
  - 如果指定排序规则是年龄，年龄相等的是不会存的

### <span id="concurrentmodificaiton">ConcurrentModificationException</span> 

- 遍历集合并删除集合中的元素时，会导致元素位置移动但索引没有及时更新导致的漏操作

- 迭代器会报错，`fori` 循环会正常执行但返回结果错误

- `fori` 循环：i-- 、倒着遍历

- 迭代器：不能调用集合自己的删除，要调用迭代器自己的删除，相当于也是做了i--的操作

- 0     1    2    3
  a     b    c    d

  删除b以后，索引为1，下一步是i++，中间插一个i--让索引不变(正序遍历)
  删除b以后，索引为1，下一步是i--，不影响正常的遍历（倒序遍历）

迭代器遍历的是开始遍历那一刻拿到的集合拷贝，遍历期间原集合发生的修改迭代器不知道。

**不要<mark>在 forEach 循环里进行元素的 `remove/add` 操作<mark>。remove 元素请使用 `Iterator` 方式**

```java
List<String> userNames = new ArrayList<String>() {{
    add("Hollis");
    add("H");
}};

Iterator iterator = userNames.iterator();

while (iterator.hasNext()) {
    if (iterator.next().equals("Hollis")) {
        iterator.remove();
    }
}
System.out.println(userNames);

```

removeIf：遍历并删除。

```java
List<Integer> list = new ArrayList<>();
for (int i = 1; i <= 10; ++i) {
    list.add(i);
}
list.removeIf(filter -> filter % 2 == 0); /* 删除list中的所有偶数 */
System.out.println(list); /* [1, 3, 5, 7, 9] */
```

**<mark>如果并发操作<mark>，在使用iterator迭代的时候使用synchronized或者Lock进行同步，或者使用JUC** 

并发情况下使用juc的并发集合，这样的集合容器在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发ConcurrentModificationException。

### 可变参数

- 可以不传参数，也可以传一个，两个多个，也可以传数组，接收数据比较灵活
- 对外是灵活接收数据，对内就是一个数组
- 一个参数列表只有一个可变参数，而且要放在最后

### Collections 工具类

![Collections](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240915140018933.png)

- `addAll(Collection<? super T> c, T...elements)`为集合批量添加数据
  - 泛型属于多态写法，`Animal`的`List`可以接收猫`Cat`和狗`Dog`作为可变参数
- `shuffle(List<?> list)` 打乱 <mark>list<mark> 的顺序
- `sort(List<?> list)` 帮助list排好序，自定义类要实现`Comparable`接口
- `sort(List<T> list, Comparator<? super T> c)` 帮助list排好序,自定义排序规则 `Animal` 的`Comparator`可以给`Cat`排序

重写T的toString方法，控制println的输出

## Map 双列 (K-V)

- 键值对集合 `key-value`
- `key`不允许重复 `value`允许重复

1. `HashMap`无序、不重复、无索引,键相同的会覆盖值
2. `LinkedHashMap`有序(添加顺序)、不重复、无索引
3. `TreeMap`大小默认升序、不重复、无索引

`Map<String,Integer> map = new HashMap<>();`

### Map 常用方法

- `put(K key, V value)`把键值对加入Map
- `void clear()` `int size()` `boolean isEmpty()` 
- `boolean containsKey(Object key)` 是否有某个键
- `boolean containsValue(V val)`是否有值`val` 
- `V get(Object key)` 根据键获取值 不存在返回`null`
- `V remove(Object key)`根据键获取值, 删除
- `Set<K> keySet()` 获取包含所有键的集合，无序不重复无索引
- `Collection<V> values()` 获取所有值的集合，**可重复** 
- `map1.putAll(Map<E> map2)`map2所有元素加入map1，能覆盖的覆盖

### Map 遍历方式

1. 键找值

   1. `keySet()` 获取所有键
   2. `V get(Object key)` 根据键找值

2. 键值对

   1. `Map.Entry<K,V>` API自带的Entry类型把Key-Value看做一个整体

   2. `Set<Map.Entry<K,V>> entrySet()` 返回一个Set，包含所有Entry对象

   3. 增强for循环遍历`Set<Map.Entry<String,Integer>> entryset` 

      ```java
      for(Map.Entry<String,Integer> entry: entryset){
          System.out.println(entry.getKey() + "=" + entry.getValue());
      }
      ```

3. Lambda表达式(**Most Simple**)

   - `map.forEach((k,v)->{System.out.println(k + "+" + v)})`
   - `forEach`方法的参数是`BiConsumer`接口的实现对象，要求重写`action`函数(遍历的时候要做的事情)
   - `forEach`方法具体的实现：用增强for循环遍历键值对组成的的Set

### HashMap 无序

#### [HashMap 和 HashSet 区别](#hashmap-和-hashset-区别)

如果你看过 `HashSet` 源码的话就应该知道：`HashSet` 底层就是基于 `HashMap` 实现的。（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外，其他方法都是直接调用 `HashMap` 中的方法。

|               `HashMap`                |                          `HashSet`                           |
| :------------------------------------: | :----------------------------------------------------------: |
|           实现了 `Map` 接口            |                       实现 `Set` 接口                        |
|               存储键值对               |                          仅存储对象                          |
|     调用 `put()`向 map 中添加元素      |             调用 `add()`方法向 `Set` 中添加元素              |
| `HashMap` 使用键（Key）计算 `hashcode` | `HashSet` 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以`equals()`方法用来判断对象的相等性 |

- 增删改查数据，性能都较好的集合
- 无序不重复无索引
- Key依赖hashCode和equals保证键的唯一性
- 如果存储自定义对象，重写上述方法即可
- HashSet实际上就是HashMap实现的，只关注键
- 线程不安全

![JDK 1.8 之后的内部结构-HashMap](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/jdk1.8_hashmap.png)



#### ~~HashTable(Deprecated)~~

`Hashtable`：数组+链表组成的，数组是 `Hashtable` 的主体，链表则是主要为了解决哈希冲突而存在的。基本都是直接在方法中加`synchronized`，性能比ConcurrentHashMap弱很多。与 Hashmap相比线程安全，不支持null

#### HashMap 遍历

`EntrySet` 之所以比 `KeySet` 的性能高是因为，`KeySet` 在循环时使用了 `map.get(key)`，而 `map.get(key)` 相当于又遍历了一遍 Map 集合去查询 `key` 所对应的值。为什么要用“又”这个词？那是因为**在使用迭代器或者 for 循环时，其实已经遍历了一遍 Map 集合了，因此再使用 `map.get(key)` 查询时，相当于遍历了两遍**。

而 `EntrySet` 只遍历了一遍 Map 集合，之后通过代码“Entry<Integer, String> entry = iterator.next()”把对象的 `key` 和 `value` 值都放入到了 `Entry` 对象中，因此再获取 `key` 和 `value` 值时就无需再遍历 Map 集合，只需要从 `Entry` 对象中取值就可以了。

所以，**`EntrySet` 的性能比 `KeySet` 的性能高出了一倍，因为 `KeySet` 相当于循环了两遍 Map 集合，而 `EntrySet` 只循环了一遍**。

#### 数组扩容

默认的**数组长度**为 16，也就是**容量 or 桶数量（capacity / number of buckets）**

数据过多会导致链表过长，查询性能降低

**加载因子（loadfactor）**为 0.75

元素个数达到**阈值（threshold）**: capacity*loadfactor = 16\*0.75 = 12 ，扩容到原来的两倍

扩容后，需要将原数组中的所有元素重新计算哈希值，并放入新的桶中，这个过程称为**rehash**，会有性能损耗，因此要尽量减少扩容次数。

##### 为什么容量必须是 2^n^ 

1. 容量cap 参与 hash % cap 运算，相当于截取低位，cap 如果是2的幂次方，cap-1就是全1，hash % cap = hash & (cap-1)，通过位运算提高了效率。
2. 还有一方面，因为cap-1是全1，因此hash的每一位都能充分参与运算，降低了哈希冲突的风险。
3. 扩容后只需检查哈希值高位的变化来决定元素的新位置，要么位置不变（高位为 0），要么就是移动到新位置（高位为 1，原索引位置+原容量）。详见下文的[rehashing](#rehash)

#### 链表转红黑树

JDK1.8 之前 `HashMap` 由数组+链表组成的。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，**如果当前数组的长度小于 64**，那么会选择**先进行数组扩容**，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

#### 源码分析

##### FIELDs 属性字段

```java
/**
 * 默认初始化容量（数组长度），必须是2的幂次方
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * 最大容量，必须小于2的30次方
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认加载因子 LoadFactor
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * 链表转换成树的阈值（链表长度）
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * 树转换回链表的阈值（链表长度）
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * 链表转换成树的阈值（数组容量）
 * 应该至少为TREEIFY_THRESHOLD的4倍
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```

##### 节点 Node<K,V>

```java
static class Node<K,V> implements Map.Entry<K,V> {
    // 哈希值
    final int hash;
    final K key;
    V value;
    // 下一个节点
    Node<K,V> next;

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }
	// 设置新的返回旧的
    public final V setValue(V newValue) {
        V oldValue = value;value = newValue;return oldValue;}

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        return o instanceof Map.Entry<?, ?> e
                && Objects.equals(key, e.getKey())
                && Objects.equals(value, e.getValue());
    }
}
Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
    return new Node<>(hash, key, value, next);
}
```

##### Contructor

```java
/* ---------------- Fields -------------- */

/**
 * Holds cached entrySet(). Note that AbstractMap fields are used
 * for keySet() and values().
 */
transient Set<Map.Entry<K,V>> entrySet;
/**
 * The table, initialized on first use, and resized as
 * necessary. When allocated, length is always a power of two.
 * (We also tolerate length zero in some operations to allow
 * bootstrapping mechanics that are currently not needed.)
 * 数组 table 每个 Entry 都是一个 节点Node
 */
transient Node<K,V>[] table;


// Constructors
public HashMap() {
	this.loadFactor = DEFAULT_LOAD_FACTOR; // all   other fields defaulted
}

// 包含另一个“Map”的构造函数
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);//下面会分析到这个方法
}

// 指定“容量大小”的构造函数
public HashMap(int initialCapacity) {
	this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
// 指定“容量大小”和“负载因子”的构造函数
    public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
    	throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
    	initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
    	throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
// 初始容量暂时存放到 threshold ，在resize中再赋值给 newCap 进行table初始化
    this.threshold = tableSizeFor(initialCapacity);
}
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
   return (n < 0)? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

```

四个构造方法中，都初始化了负载因子 loadFactor，由于 HashMap 中没有 capacity 这样的字段，即使指定了初始化容量 initialCapacity ，也只是通过 tableSizeFor() 扩容到与 initialCapacity 最接近的 2 的幂次方大小，然后**暂时赋值给 threshold ，后续通过 resize 方法将 threshold 赋值给 newCap 进行 table 的初始化。** 

###### 使用另一个map构造 `putMapEntries(map)` 

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            // s 是实际个数，dt 就是添加 s 个元素的最小容量 ceil 向上取整
            double dt = Math.ceil(s / (double)loadFactor);
            int t = ((dt < (double)MAXIMUM_CAPACITY) ?
                     (int)dt : MAXIMUM_CAPACITY);
            // 如果超过
            if (t > threshold)
                threshold = tableSizeFor(t);
        } else {
            // 已经初始化，只要超过阈值就要 resize
            // Because of linked-list bucket constraints, we cannot
            // expand all at once, but can reduce total resize
            // effort by repeated doubling now vs later
            while (s > threshold && table.length < MAXIMUM_CAPACITY)
                resize();
        }

        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

##### `getNode()`

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(key)) == null ? null : e.value;
}

final Node<K,V> getNode(Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n, hash; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & (hash = hash(key))]) != null) {
        if (first.hash == hash && // 快速检查头节点
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash && 
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e; // 从头节点开始遍历。
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

##### `putVal()`

体现了懒加载的思想，只有真正put的时候才初始化资源：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
public V putIfAbsent(K key, V value) {
    return putVal(hash(key), key, value, true, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 1. 如果没有初始化，先调用resize初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 2.1 没有哈希冲突的情况
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 2.2 出现了哈希冲突/值重复，在else逻辑里return
    else { 
      // .......
    }
    // 3. 没有哈希冲突或者值重复，元素自增与扩容策略
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);// 用于LinkedHashMap支持LRU实现,HashMap无用
    // 4. key值不存在,则返回值是null
    return null;
}
```

###### 哈希冲突/值重复

```java
// p 是链表/树的第一个节点
Node<K,V> e; K k;
/* 1. 快速判断第一个节点table[i]的key是否与插入的key一样 先判断 hash 
				若相同就将现在的节点p赋给e，然后在4中处理。     */ 		
if (p.hash == hash && // key 的 hash 
// 1.5 短路逻辑：先用 == 比较地址，地址不同再用equals比较内容（K需要重写equals）
	((k = p.key) == key || (key != null && key.equals(k))))
    e = p;
// 2. 树节点去执行树的逻辑
else if (p instanceof TreeNode)
    e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
// 3. 链表节点
else {
    for (int binCount = 0; ; ++binCount) {
        // 3.0 边遍历边比较是否出现了重复key, binCount+1 能得出链表长度
        if ((e = p.next) == null) {
            // 3.1 一直遍历到了尾部，说明肯定没有重复的，在链表尾端创建新 Node
            p.next = newNode(hash, key, value, null);
            // 3.2 如果到了 TREEIFY_THRESHOLD 就触发 treeifyBin 扩容或
            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                treeifyBin(tab, hash);
            // 3.3 跳出循环 e == null, 不会走重复的逻辑
            break;
        }
        // 3.5 判断key是否重复 重复就直接跳出，去4处理key的重复情况 e!=null
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            break;
        
        p = e;// 遍历
    }
}
// 4. key 重复 应对1和3.5的重复情况 
// 	在putIfAbsent中，onlyIfAbsent = true, 在put中，onlyIfAbsent = false
if (e != null) {
    V oldValue = e.value;
    // putIfAbsent只有原值null才能赋值
    if (!onlyIfAbsent || oldValue == null) 
        e.value = value;
    afterNodeAccess(e);//回调方法，只在LinkedHashMap中实现,可维护访问顺序(如LRU)
    // 4.5 key值如果存在,则会返回原先被替换掉的value值
    return oldValue;
}
```

##### `removeNode()` 

remove主要有两个：一个是remove(key)，返回值为被删除的value，如果节点不存在则返回null

另一个是remove(key,value) 用来表示只有key对应的值为value时才移除，返回值为boolean

```java
@Override
public boolean remove(Object key, Object value) {
    return removeNode(hash(key), key, value, true, true) != null;
}
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
//
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    /**  1. 寻找匹配到key的节点
      *  短路条件1: table 不为空(已初始化过)
      *  短路条件2: key 对应的桶不为空
      *  同时满足上述两个条件才会进入正式判断，否则直接返回 null
      *  p 现在是桶的第一个节点。
      */
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        // 1.1 快速检查: 先检查桶的第一个节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        // 1.2 开始遍历， e 相当于 tmp
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)// 树节点
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {// 链表节点
                do { 
                    // 1.2.1 找到了节点，直接break，将e赋给node
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;//  p是e的前驱
                } while ((e = e.next) != null);
            }
        }
        /** 2. 正式开始删除节点
          * node 为将要删除的节点
          * remove(k,v)->matchValue=true | remove(k)->matchValue = false
          */
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p) // 对应 1.1 快速检查, 将node.next赋值给tab[i]
                tab[index] = node.next;
            else //对应 1.2.1, p 是 node 的前驱节点， 直接将node.next赋给p.next
                p.next = node.next;
            ++modCount;
            --size;// 减小容量
            afterNodeRemoval(node);
            return node;// 返回被删除的节点
        }
    }
    return null;
}
```

##### `resize()`

用于初始化或扩容，初始化就调用属性字段里面的 threshold 初始化

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    //—————————扩容——————————
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 扩容两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // threshold * 2
    }
    //————————初始化——————————
    else if (oldThr > 0) 
    // 创建时指定了初始化容量或者负载因子 就会把算出的容量暂时存放到threshold中
    // 			在这里进行新容量的初始化
        newCap = oldThr;
    else {  
    // 创建时无参构造，就使用默认的 capacity 和 threshold 对容量和阈值进行初始化
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        // 创建时指定了初始化容量或者负载因子，在这里进行新阈值的初始化，
        // 或者扩容前的旧容量小于16，在这里计算新的resize上限
        float ft = (float)newCap * loadFactor;
    newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr; // 新的阈值。
    
    
    @SuppressWarnings({"rawtypes","unchecked"})
    // 正式创建新的数组, 可以看到分配大小为 newCap
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // oldTab 的 rehash
    }
    // 返回新的 Table 数组;
    return newTab;
}
```

###### <span id="rehash">rehashing oldTab</span> 

索引本质还是哈希值对容量取余。

HashMap 扩容时采用的容量是 **2 的幂次方**，它的二进制特性使得新容量只在**高位多出一位 1**。

元素 A B 的哈希值分别为 2 和 6，容量从 4 扩容到 8：

- `0010 & 0011 = 0010` `0110 & 0011 = 0010`  旧索引均为 2
- `0010 & 0111 = 0010` `0110 & 0111 = 0110`  新索引分别为 2 和 6

**二者的区别仅在于高位是否为 1**：避免了复杂的哈希重算，仅通过简单的位运算就完成了分配。

本质上还是取hash值的低位，原来只取低两位，只有这两位参与运算，新的需要取低三位，那么此时直接和原容量 `0100` 相与，看看第三位是不是0，如果第三位是0，索引当然不变，如果第三位是1，新的索引就是原索引+原容量

```java
for (int j = 0; j < oldCap; ++j) {
    Node<K,V> e;// tmp
    if ((e = oldTab[j]) != null) {
        oldTab[j] = null;
        if (e.next == null)// 如果没有后继节点，直接映射到新的哈希位即可
            newTab[e.hash & (newCap - 1)] = e;
		// e.hash & (newCap - 1) 等价于 e.hash % newCap
        else if (e instanceof TreeNode)// 树：
            ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
        
        else { // preserve order 保证原来的顺序
            Node<K,V> loHead = null, loTail = null; // 低索引，索引不变
            Node<K,V> hiHead = null, hiTail = null; // 高索引，索引变化
            Node<K,V> next;
            // 链表尾插法
            do {
                next = e.next;
                if ((e.hash & oldCap) == 0) {
                    // 还在原桶
                    if (loTail == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                }
                else {
                    // 换到新桶
                    if (hiTail == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                }
            } while ((e = next) != null);
            if (loTail != null) {
                // 在旧桶的，索引不变
                loTail.next = null;
                newTab[j] = loHead;
            }
            if (hiTail != null) {
                // 在新桶的，新索引为原索引+原容量
                hiTail.next = null;
                newTab[j + oldCap] = hiHead;
            }
        }
    }
}
return newTab;
```

###### **链表尾插法** 防止多线程死循环

JDK1.7 及之前版本的 `HashMap` 在多线程环境下扩容操作可能存在死循环问题，这是由于当一个桶位中有多个元素需要进行扩容时，多个线程同时对链表进行操作，头插法可能会导致链表中的节点指向错误的位置，从而形成一个环形链表，进而使得查询元素的操作陷入死循环无法结束。

为了解决这个问题，JDK1.8 版本的 HashMap 采用了尾插法而不是头插法来避免链表倒置，使得插入的节点永远都是放在链表的末尾，避免了链表中的环形结构。但是还是不建议在多线程下使用 `HashMap`，因为多线程下使用 `HashMap` 还是会存在数据覆盖的问题。并发环境下，推荐使用 `ConcurrentHashMap` 。

> `if (tail == null)     head = e; `
>
> `else tail.next = e;`    
>
> `tail = e;`    

##### `treeifyBin()` 

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 如果容量没有超过阈值64，优先扩容！
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

### LinkedHashMap 有序

- 有序（添加顺序） 不重复 无索引
- HashMap加了双链表机制记录添加顺序
- LinkedHashSet实际上就是LinkedHashMap实行的

继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，`LinkedHashMap` 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。详细可以查看

### TreeMap 可自定义排序 定向搜索

- 基于红黑树，TreeSet跟TreeMap原理一样
- 排序：自定义排序规则
  - 自定义的类实现`Comparable`接口，重写 `int compareTo(Object o)`方法
  - TreeMap的有参构造 参数是`Comparator`的实现对象，重写了`int compare()`方法

#### 集合嵌套

`Map<String,List<String>> cityMap = new HashMap<>();`

![嵌套](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240915170039061.png)

## Arrays 数组操作

- `toString(Object[] arr)` 返回一个数组字符串，每个元素调用`toString()`转换成字符串，用`[,,,]`拼接成一个总字符串

-  `copyOfRange(Object[] arr, from, to)` 返回值为数组， 内容为arr中索引 **[from, to)** 的部分

- `copyOf(Object[] arr, int newlength)` 返回值为数组，内容为arr的内容，长度为newlength，不够的用0补齐，索引超出的截断

- `setAll`  第二个参数是接口`IntToDoubleFunction`的实现对象，重写了`applyAsDouble(int value)`方法，value是数组的索引，返回对数组中内容进行操作的结果，图中是将数组内容乘以0.8。这个对象传进来以后，对数组进行遍历，都用`applyAsDouble`的方法进行操作。

  ![applyAsDouble](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240913170741004.png)

- `sort([] arr)` 对arr升序排序(默认升序)

- `asList()`:只能加**引用类型**，基本数据类型需要改成包装类，并且返回的`List`是不能运用`add remove clear`方法的，这个`list`是`AbstractList`的子类，如果要访问`utils`只能将返回的这个`List`添加到`new ArrayList<>()`作为有参构造

### 对象数组排序

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

# Stream流

- 需要有**数据源**，集合/数组等
- 调用流水线的方法对集合处理、计算
- 支持链式方法

![stream 流](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240915171033434-1735476275516-27.png)

得到流，filter forEach

## 常见方法

### 获取Stream流

- 流的泛型就是集合中元素的类型
- **集合**：`set.stream()` 
- **数组**：
  - `Arrays.stream(T[] array)`
  - `Stream<T>.of(T...values)` 
- Map：处理键用`keySet`，处理值用`values` Map.Entry用`entrySet` ![entrySet](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240915171810424-1735476275516-28.png)

### 中间方法

- 返回新的Stream流支持链式编程
- `filter`:接口的实现`filter(s->s.getAge>=23 && s.getAge<=30)`重写`boolean test()`方法返回值是一个布尔变量。**筛选条件** s就代表集合中的元素
- `sorted`:无参数默认根据值升序排序，有参数（实现`Comparator`接口并重写`int compare(o1,o2)`）自定义排序规则。**排序**
- `limit(long maxSize)`:取前3个对象 
- `skip(long n)`:跳过前n个对象，“指针”移动到对应位置，可以实现逻辑分页
- `distinct()`:去重
- `map(mapper)`:把集合中元素映射，mapper `map(Student::getName)` `map(s->s.getName())` 把集合中的元素 通过映射方法`mapper` 转换成对应元素
- `distinct()`:去重复，**自定义**类型对象如果希望对比内容，应该重写`hashCode()` `equals()`方法
- `Stream.concat(st1,st2)`:合并两个流内容,返回新的流
- `boxed()` 基本数据类型装箱操作

### 终结方法 void

- （没有返回值）
- `void forEach(action)`: `forEach(s->System.out.println(s))` 元素s -> 指定s想做的事情
- `long count()`: 返回经过前面处理以后集合剩下的元素个数
- `max((o1,o2)->Double.compare(o1.getScore(),o2.getScore()))` 
- `min()`同`max()` 实现`Comparator`
- `get()`:用于在`min max`后接收对象
- `collect(Collectors.toList())` `collect(Collectors.toSet())` 把流收集起来转换成集合
- `collect(Collectors.toMap(a->a.getName() , a->a.getHeight()))` 两个接口做参数，Lambda表达式。如果遇到`key`冲突，`toMap`需要调用重载函数，启用第三个参数，`(o1,o2)->o2`表示前后`key`冲突时，后添加的`value`会覆盖之前的`value`
- `collect(Collectors.groupingBy(Shop::getTypeId))`把流收集起来，并按照typeId分组，返回一个typeId:集合Map
- `toArray()`将流中的元素收集到一个`Object`**数组中** 
- `toArray(len -> new Student[len])`将流中的元素收集到一个指定`Student`类型的数组中 方法引用`toArray(Student[]::new)` 此处参数是`IntFunction<A[]>`接口的实现对象`generator` 重写函数需要`return`一个对应类型的数组，故可以用此写法

### toList() & collect(Collectors.toList())

- 确定其是一个不再被set/add/remove的list 可使用 Stream `toList`; 如果使用`collect(Collectors.toList())` ,sonar或idea自带以及第三方的一些code checker会爆warning, 以本人经验，可以使用`collect(Collectors.toCollection(ArrayList::new))`来代替

另外ListOf也是返回的不可增删改的List


Lambda表达式省略规则 REVIEW

- 只有一个参数可以省略`()` ，没有参数不能省略

- `->`后是具体的函数重写，多条语句需要`{}` `;` 单条语句可以省略分号

  - 单条语句分为有返回值和无返回值，只有一行`return`的可以省略`return`关键字，没有返回值的比如输出`System.out.println(s)` 就要注意了
  - `a->a.getName()` 要么getName()的返回值 matters 要么没有返回值 

- 方法引用

```java
  Function<Item, String> getNameFunction = item -> item.getName();
  
  Item item = new Item("Apple");
  String name = getNameFunction.apply(item);  // 返回 "Apple"
  
  R apply(T t);//抽象方法 给一个T，返回R类型
  String apply(Item item);//泛型将接口具体化
  
  /*
  键：return 键 
  值：return 值 
  转换成 Lambda 表达式 调用的函数必须有返回值的
  */
  
```

操作数组：最终的目的仍然是数组、集合

# 集合最佳实践

（1）根据需要选择正确的集合类型。比如，如果指定了大小，我们会选用Array而非ArrayList。如果我们想根据插入顺序遍历一个Map，我们需要使用TreeMap。如果我们不想重复，我们应该使用Set。

（2）一些集合类允许指定初始容量，所以如果我们能够估计到存储元素的数量，我们可以使用它，就避免了重新哈希或大小调整。

（3）基于接口编程，而非基于实现编程，它允许我们后来轻易地改变实现。

（4）总是使用类型安全的泛型，避免在运行时出现ClassCastException。

（5）使用JDK提供的不可变类作为Map的key，可以避免自己实现hashCode()和equals()。

（6）尽可能使用Collections工具类，或者获取只读、同步或空的集合，而非编写自己的实现。它将会提供代码重用性，它有着更好的稳定性和可维护性。

## 集合判空

这是因为 `isEmpty()` 方法的可读性更好，并且时间复杂度为 `O(1)`。

绝大部分我们使用的集合的 `size()` 方法的时间复杂度也是 `O(1)`，不过，也有很多复杂度不是 `O(1)` 的，比如 `java.util.concurrent` 包下的 `ConcurrentLinkedQueue`。`ConcurrentLinkedQueue` 的 `isEmpty()` 方法通过 `first()` 方法进行判断，其中 `first()` 方法返回的是队列中第一个值不为 `null` 的节点（节点值为`null`的原因是在迭代器中使用的逻辑删除）

## [集合遍历 iterator 并发修改异常](#concurrentmodificaiton)

## 集合转 Map

**在使用 `java.util.stream.Collectors` 类的 `toMap()` 方法转为 `Map` 集合时，一定要注意当 value 为 null 时会抛 NPE 异常。 **

## 集合转数组

`toArray(T[] array)` 方法的参数是一个泛型数组，如果 `toArray` 方法中没有传递任何参数的话返回的是 `Object`类 型数组。

```java
String [] s= new String[]{
    "dog", "lazy", "a", "over", "jumps", "fox", "brown", "quick", "A"
};
List<String> list = Arrays.asList(s);
Collections.reverse(list);
//没有指定类型的话会报错
s=list.toArray(new String[0]);
```

由于 JVM 优化，`new String[0]`作为`Collection.toArray()`方法的参数现在使用更好，`new String[0]`就是起一个模板的作用，指定了返回数组的类型

## 数组转集合

**`Arrays.asList()`是泛型方法，传递的数组必须是对象数组，而不是基本类型。** 

toList() & collect(Collectors.toList())

`Arrays.asList()` 或者流的`toList()`或者`List.of()`，得到的List只能读，不能进行修改操作，因为这个list是AbstarctList的实现类，并没有实现修改的方法

```java
Integer [] myArray = { 1, 2, 3 };
List myList = Arrays.stream(myArray).collect(Collectors.toList());
//基本类型也可以实现转换（依赖boxed的装箱操作）
int [] myArray2 = { 1, 2, 3 };
List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
```

`List list = new ArrayList<>(Arrays.asList("a", "b", "c"))` 也可以

