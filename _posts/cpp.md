---
name: cpp
title: cppnotes
date: 2023-11-23
---



# C++ 笔记

## 错误处理

- 如果程序执行错误，就throw一个异常

  b是一个int


```c++
throw b 

try{
...
}catch (int b){

cerr <<b<<endl }
```

throw 一个字符串常量，catch的参数就是一个const char

- throw加在函数声明的后边 void fun() throw(A,B,C,D)

  表示函数能抛出ABCD四种类型的错误，不加throw表明能抛出任何类型的异常

```c++
#include <iostream>
using namespace std;
 
double division(int a, int b)
{
   if( b == 0 )
   {
      throw "Division by zero condition!";
   }
   return (a/b);
}
 
int main ()
{
   int x = 50;
   int y = 0;
   double z = 0;
   try {
     z = division(x, y);
     cout << z << endl;
   }catch (const char* msg) {
     cerr << msg << endl;
   }
   return 0;
}
```



## 命名空间

- using namespace XXX
  如果不写using namespace std的话
  用cout函数就得加std::cout
  尽量别用，容易污染
  xx::a 与 yy::a不是一个东西
  全局作用域符号，用来区分同名的的全局变量与局部变量
  ::不跟类名，表示全局的

## 动态数组（new&delete）

```c++
int *p=NULL;
p=new int;
*p=2;
delete p;

char *p=NULL;
p=new char[10];
delete []p;//对象数组是同样的处理方法
```

我们在用动态内存分配时，经常是用`new`来定义一块内存空间，比如说 `int* p = new int(1)；`这时会在堆上分配一块内存，当作int类型使用，内存中存储的值为1并将内存地址赋值给在栈中的int*类型的p。（注意：p只是一个变量，就像是int a=1中的a一样，不过a是整形变量，而p是指针变量）当我们不用p指针时，往往需要用delete p将其释放，我们需要注意的是释放一个指针p（delete p;）实际意思是删除了p所指的目标（变量或对象），释放了它所占的堆空间，而不是删除p本身（指针p本身并没有撤销，它自己仍然存在，该指针所占内存空间并未释放，指针p的真正释放是随着函数调用的结束而消失），释放堆空间后，p成了"空指针"。如果我们在delete p后没有进行指针p的制空（p=NULL)的话，其实指针p这时会成为野指针，为了使用的安全，我们一般在delete p之后还会加上p=NULL这一语句。具体怎么成为野指针的，这有一篇非常详细的介绍[点击打开链接](http://https//www.cnblogs.com/uniqueliu/archive/2011/07/18/2109778.html)

## 重载

- 重载声明是指一个与之前已经在该作用域内声明过的函数或方法具有相同名称的声明，但是它们的参数列表和定义（实现）不相同。 

- 当您调用一个重载函数或重载运算符时，编译器通过把您所使用的参数类型与定义中的参数类型进行比较，决定选用最合适的定义。选择最合适的重载函数或重载运算符的过程，称为重载决策。

## 运算符

位运算符（除去“~”都是双目运算符）

| &   与   | 只有同时为1才为1         |
| -------- | ------------------------ |
| \|    或 | 只有同时为0才为0         |
| ^  异或  | 一样就是0 不一样就是1    |
| ~  取反  | 变成负数，1变成0，0变成1 |
| << 左移  | 字面意思                 |
| >> 右移  | 字面意思                 |

​      A = 0011 1100

​      B = 0000 1101

A & B = 0000 1100

A  |  B = 0011 1101

A  ^ B = 0011 0001

   ~A   = 1100 0011

A<<2  = 1111 0000

A>>2  = 0000 1111

- 赋值运算符

  +=，-=，/=，%=，*=   不做赘述

  位运算<<=：`a<<=2`   a=a<<2

  任何数异或 **^0** 得到的值不变:`a^0 = a`

  任何数异或同一个数两次得到的值不变:`a^b^b = a`

## 类与对象

- 访问权限：public private protected

### 	构造函数

定义一个Line类，有一个length成员变量 初始化列表赋初值

```c++
Line::Line(int  num0):length (num0)
Line a(1)
```

如果声明了任何非默认构造函数、编译器不会提供默认构造函数。

构造函数在未指定参数或者提供了一个空初始化器列表，则会调用默认构造函数：`vector v1; vector v2{};`

引用和const必须被初始化

### 	复制构造函数（&引用、const的使用）

```c++
Line::Line(const Line&obj)
```

&是引用号作用类似于指针，引用时必须初始化值

```c++
a=1 ;   
int &ra =a ;   
```

​	相当于起了个别名ra，这样a在用作参数时候使用ra可以不用调用复制构造函数，因此不会造成无限复制构造函数的无限循环。如果引用对象参数为const类型，则不能通过ra修改a，反过来是可以的，但可以修改指针指向的地方。

​	如果对象中有指针，默认复制时指针的值是不会变的，因为他分配的内存是不在对象里面的，在析构	的时候，会出现两次析构相同位置的情况，所以==有指针的情况下最好自己构建一个复制构造函数==，并分配新的内存空间给新对象的指针。并且析构函数中也要额外释放指针，在系统默认的拷贝构造，**对指针的赋值时为==浅拷贝==**，可能会导致直接对位的赋值，从而导致出现野指针情况，**手动处理为==深拷贝==**。

- 用引用传递函数的参数，能保证参数传递中不产生副本，提高传递的效率，且通过const的使用，保证了引用传递的安全性。普通引用在声明时必须用其它的变量进行初始化

- 引用作为函数参数声明时不进行初始化

- 传递引用是传递原变量，不需要做变量拷贝，普通的变量做函数参数的时候会开辟内存拷贝数值，而传递引用则不需要开辟内存；

  C++ primer p406 ：拷贝构造函数是一种特殊的构造函数，具有单个形参，该形参（常用const修饰）是对该类类型的引用。当定义一个新对象并用一个同类型的对象对它进行初始化时，将显示使用拷贝构造函数。当该类型的对象传递给函数或从函数返回该类型的对象时，将隐式调用拷贝构造函数。

- C++支持两种初始化形式：

  - 拷贝初始化 int a = 5; 
- 直接初始化 int a(5); 
  - 对于其他类型没有什么区别，对于类类型直接初始化直接调用实参匹配的构造函数，<u>拷贝初始化总是调用拷贝构造函数</u>，也就是说：


```c++
A z;        //定义，调用无参构造函数
A x(2);　　 //直接初始化，调用有参构造函数
A y = x;　　//拷贝初始化，调用拷贝构造函数
```

- <u>必须定义拷贝构造函数的情况</u>：

  1. 只包含类类型成员或内置类型（但不是指针类型）成员的类，无须显式地定义拷贝构造函数也可以拷贝；
  2. 有的类有一个数据成员是指针，或者是有成员表示在构造函数中分配的其他资源。
- <u>类的对象需要拷贝时，拷贝构造函数将会被调用。以下情况都会调用拷贝构造函数：</u>

  1. 一个对象以值传递的方式传入函数体；

  2. 一个对象以值传递的方式从函数返回；

  3. 一个对象需要通过另外一个对象进行初始化。
- 实例

```c++
#include <iostream>
using namespace std;
class Line
{
   public:
      int getLength( void );
      Line( int len );             // 简单的有参构造函数
      Line( const Line &obj);      // 拷贝构造函数
      ~Line();                     // 析构函数
   private:
      int *ptr;
};
Line::Line(int len)
{
    cout << "调用构造函数" << endl;
    // 为指针分配内存
    ptr = new int;
    *ptr = len;
}
Line::Line(const Line &obj)
{
    cout << "调用拷贝构造函数并为指针 ptr 分配内存"          << endl;
    ptr = new int;
    *ptr = *obj.ptr; // 拷贝值
}
 
Line::~Line(void)
{
    cout << "释放内存" << endl;
    delete ptr;
}
int Line::getLength( void )
{
    return *ptr;
}
 
void display(Line obj)
{
   cout << "line 大小 : " << obj.getLength() 		         <<endl;
}
// 程序的主函数
int main( )
{
   Line line1(10);
   Line line2 = line1; // 这里也调用了拷贝构造函数
   display(line1);
   display(line2);
   return 0;
}
```

- 执行结果：

```
调用构造函数
调用拷贝构造函数并为指针 ptr 分配内存
调用拷贝构造函数并为指针 ptr 分配内存
line 大小 : 10
释放内存
调用拷贝构造函数并为指针 ptr 分配内存
line 大小 : 10
释放内存
释放内存
释放内存
```

### 	析构函数

- 必须与类同名 再在前部加一个～，删除对象的时候会自动调用

- 如果一个类中有指针，且在使用的过程中动态的申请了内存，那么最好在销毁类之前显式构造析构函数，释放掉申请的内存空间，避免内存泄漏。

- **类析构顺序：**

  1）派生类本身的析构函数；

  2）对象成员析构函数；

  3）基类析构函数。

### 	友元函数（友元类）

- 定义在类外部，但有权访问类的所有私有（private）成员和保护（protected）成员。
- 尽管友元函数的原型有在类的定义中出现过，但是友元函数并不是成员函数。
- 友元可以是一个函数，该函数被称为友元函数；友元也可以是一个类，该类被称为友元类，在这种情况下，整个类及其所有成员都是友元。如果要声明函数为一个类的友元，需要在类定义中该函数原型前使用关键字 friend，没有this指针，访问非static指针要引入对象做参数

### 	内联函数	

- 不允许使用switch 与loop 语句

- 定义必须出现在首次调用之前

- 较为短小的代码

## 继承、多态

#### 继承方式

##### 	按权限：

- ##### 		public  保持不变

- ##### 		protected  原public变成protected

- ##### 		private  全变成private

##### 	按父类个数：

- ##### 单继承：只有一个父类

- ##### 多继承：有多个父类

- ##### 链式继承：一条链，首尾相连

- ##### [虚继承](####虚继承)：也就是菱形继承

#### 重载(**静态多态**)

静态函数在编译的时候就已经确定运行时机

#### 	虚函数(动态多态)

- 函数只在 code 区存放一份，数据成员则每个对象一份，并按照声明顺序依次存放

- 在有虚函数的类中，类的最开始部分是一个虚函数表的指针，这个指针指向一个虚函数表，表中放了虚函数的地址，实际的虚函数在代码段(.text)中。当子类继承了父类的时候也会继承其虚函数表，当子类重写父类中虚函数时候，会将其继承到的虚函数表中的地址替换为重新写的函数地址。使用了虚函数，会增加访问内存开销，降低效率。

- 类A中有了虚函数就会再类的数据成员的最前面添加一个 vfptr 指针(void** vfptr)，这个指针用来指向一个 vtable 表（一个函数指针数组）（一个类只有一个该表），该表存储着当前类的所有 虚函数 的地址。这样 vfptr 就成为了一个类似成员变量的存在。访问虚函数的时候通过 vfptr 间址找到vtable 表，再间址进而找到要调用的函数。这样就在一定程度上摆脱了类型制约。

  ![people派生出student再派生出vtable](https://i.imgloc.com/2023/05/25/VDJKFq.png)

  在虚函数表中，基类的虚函数在 vtable 中的索引（下标）是固定的，不会随着继承层次的增加而改变，派生类新增的虚函数放在 vtable 的最后。如果派生类有同名的虚函数遮蔽（覆盖）了基类的虚函数，那么将使用派生类的虚函数替换基类的虚函数，这样具有遮蔽关系的虚函数在 vtable 中只会出现一次。

  只要vptr的值不同，那么访问函数成员的时候使用的vtable表就不同，就可能访问到不同类的函数成员。B类对象中的vptr指向B类自己的vtable。

  当B类继承A类的时候，因为A中有虚函数，编译器就自动的给B类添加vfprt指针和vtable表。也可以理解为B类继承来了A类中的那个vptr指针成员。

  当A类指针指向B类对象时，发生假切割。要知道这个过程只是切掉A类中没有的那些成员。（即当People类指针指向Student类对象时，切割掉m_score这个People类中没有的成员）
  由于vptr是从A类中继承来的，所以这个量仍将保留。而对于vptr的值则不会改变，仍然指向B类的vtable表。所以访问F1函数的时候是通过B类的vtable表去寻址的，自然就是使用子类的函数（拿图中的情况举例，子类的Student::display()函数已经覆盖了People::display()函数，此时A类指针访问虚函数display()时也是访问到子类的Student::display()函数）。

  当B类的指针指向A类的对象时（当B类存在新增数据成员时可能出错），同理。

  而对于普通函数则受类型的制约，（因为没有vptr指针）使用哪个类的指针调用函数，那么所调用的就是那个累的函数。
  总而言之，普通函数通过对象或指针的类型来找所调用的函数，而虚函数是通过一个指针来找到所要调用的函数的。

  ***==派生类指针指向基类对象==***，这里疑问会比较大。首先是为什么这里不会报错，为什么派生类指针指向基类对象可以成立？理论上指针的可访问范围一定大于对象的大小，会指向一些未知区域导致运行出错，但是要注意的是，**这个题目里面B类不存在新增数据成员，所以不会出错**。还有就是由于是基类对象，还没有发生虚函数掩盖

- 函数要修改数据必须要传入该数据的地址

  实现C++的多态，基类与派生类有同名的函数，派生类在调用这个函数的时候不知道调用哪个，因此就要用虚函数，在基类的这个函数加上virtual前缀。

  虚函数必须实现也就是定义，不然会报错，

  **纯虚函数:virtual xxx xxx()=0**

  声明纯虚函数就代表这个类成了**抽象类，不能进行实例化**

  这就是在提醒继承这个类的时候要再次定义这个函数，不然还是抽象类没法实例化

  定义他为虚函数是为了允许用基类的指针来调用子类的这个函数。

  定义一个函数为纯虚函数，才代表函数没有被实现。

  定义纯虚函数是为了实现一个接口，起到一个规范的作用，规范继承这个类的程序员必须实现这个函数，一个类函数的调用并不是在编译时刻被确定的，而是在运行时刻被确定的。由于编写代码的时候并不能确定被调用的是基类的函数还是哪个派生类的函数，所以被成为"虚"函数。

  虚函数只能借助于指针或者引用来达到多态的效果。

  纯虚函数是在基类中声明的虚函数，它在基类中没有定义，但要求任何派生类都要定义自己的实现方法。在基类中实现纯虚函数的方法是在函数原型后加 =0:

  那么此时就能[通过父类的一个指针来调用子类的方法](####通过父类指针调用子类对象的成员函数（同名同参）)

```c++
class A virtual void sow()
class B void sow()

A *pa=NULL;
B b;
pa=＆b;
pa->sow()
```


#### 虚继承

- ​    A

   /      \
  B      C             D继承了两个A，析构时会造成内存泄露
   \     /              所以BC在继承A的时候必须要 `virtual public A` 虚继承
      D                 
  
  ​		在一个派生类中保留间接基类的多份同名成员，虽然可以在不同的成员变量中分别存放不同的数据，但大多数情况下这是多余的：因为保留多份成员变量不仅占用较多的存储空间，还容易产生命名冲突。假如类 A 有一个成员变量 a，那么在类 D 中直接访问 a 就会产生歧义，编译器不知道它究竟来自 A ->B->D 这条路径，还是来自 A->C->D 这条路径。
  
  ​		为了解决多继承时的命名冲突和冗余数据问题，C++ 提出了虚继承，使得在派生类中只保留一份间接基类的成员。
  
  ​		虚继承的目的是让某个类做出声明，承诺愿意共享它的基类。其中，这个被共享的基类就称为**虚基类（*Virtual Base Class*）**，本例中的 A 就是一个虚基类。在这种机制下，不论虚基类在继承体系中出现了多少次，在派生类中都只包含一份虚基类的成员。

#### 子类从基类继承的成员限制

- 基类的构造函数、析构函数和拷贝构造函数。
- 基类的重载运算符。
- 基类的友元函数。
- ==子类对象中父类成员初始化必须调用父类的构造函数，初始化列表方式==
- 子类对象中其他类的成员初始化必须使用初始化列表方式

#### ==构造函数 析构函数调用顺序==

- 仅考虑实例化派生类对象时的情况
- 构造函数调用顺序：基类 > 派生类里的对象成员 > 派生类； 
- 多继承派生类： 基类的构造顺序依照基类继承顺序调用
- 对象成员[^2]：依照在派生类中对象成员的定义顺序 调用成员的构造函数 与初始化列表顺序无关

```c++
#include <iostream>
using namespace std;
class Shape // 基类 Shape
{   
public:
    Shape() {cout << "Shape" << endl;}
    ~Shape() {cout << "~Shape" << endl;}
};

class PaintCost // 基类 PaintCost
{   
public:
    PaintCost() {cout << "PaintCost" << endl;}
    ~PaintCost() {cout << "~PaintCost" << endl;}
};

// 派生类
class Rectangle : public Shape, public PaintCost  //基类构造顺序 依照 继承顺序
{
public:
    Rectangle() :b(), a(), Shape(), PaintCost()
    {cout << "Rectangle" << endl;}
    ~Rectangle() 
    {cout << "~Rectangle" << endl;}
    PaintCost b;        // 对象成员依照定义顺序
    Shape a; 
};

int main(void)
{
    Rectangle Rect;
    return 0;
}
```

执行结果：

```c++
Shape
PaintCost
PaintCost
Shape
Rectangle
~Rectangle
~Shape
~PaintCost
~PaintCost
~Shape
```

#### 通过父类指针调用子类对象的成员函数（同名同参）

- 基类指针指向基类对象，简单。只需要通过基类指针简单地调用基类的功能。

- 派生类指针指向派生类对象，简单。只需要通过派生类指针简单地调用派生类功能。

- 将基类指针指向派生类对象是安全的，因为派生类对象“是”它的基类的对象。

  但是要注意的是，这个指针只能用来调用基类的成员函数。

  如果试图通过基类指针调用派生类才有的成员函数，则编译器会报错。

  为了避免这种错误，必须将基类指针强制转化为派生类指针。然后派生类指针可以用来调用派生类的功能。这称为向下强制类型转换，这是一种潜在的危险操作。

注意：如果在基类和派生类中定义了虚函数（通过继承和重写），并通过基类指针在派生类对象上调用这个虚函数，则实际调用的是这个函数的派生类版本。

##### 出现同时有虚实函数的情况

1. 若全为虚函数，则调用子类的函数

2. 若全为实函数，则调用父类的函数

3. 若一实一虚，则调用他们中的实函数

   **父类没有定义为虚的时候，子类是没办法多态的，而父类定义为虚函数的时候，子类默认也是个虚函数，会根据指针指向的数据类型来选择函数调用**

   **派生类加不加virtual都是虚函数，只要派生类实现了虚函数就会覆盖基类的虚函数，基类指针pBase指向派生类对象basePlus时会调用派生类的虚函数**

##### **虚析构函数**（delete）

1. 可能通过基类指针删除派生类对象、
2. 如果你打算允许他人通过基类指针调用对象的析构函数（通过delete这样做是正常的），就需要让基类的析构函数变为虚函数，否则执行delete的结果是不确定的
3. 虚构造函数不合法(有了虚函数就要有虚函数表，调用构造函数就要去找vptr，此时vptr还没初始化)
4. 虚析构函数的实现原理：

   在父类中通过virtual 修饰析构函数后，通过 父类指针再去指向子类对象，然后通过delete 接父类指针，就可以 释放掉子类对象了

   有了这个前提，如果使用父类的指针通过 [delete](##动态数组（new&delete）) 的方式去释放子类的 对象，那么只要能够实现通过父类的指针执行到子类的析构函数即可

   如果子类中不写虚析构函数，计算机会默认给你定义一个虚析构函数， 前提是你在父类中得有virtual 来修饰父类的析构函数

   在使用时： 如果在main() 函数中通过父类指针指向子类对象，然后通过 delete 接父类指针释放子类对象 此时，虚函数表的工作： 如果在父类中定义了虚析构函数，那么在父类的虚函数表中就会 有一个父类析构函数的函数指针，指向父类的析构函数 而在子类的虚函数表中也会产生一个子类析构函数的函数指针， 指向子类的析构函数（注意：虚析构函数会覆盖） 当 父类的指针指向 子类的对象，通过 delete 接 父类的 指针时，就可以通过子类对象的 虚函数表指针 找到子类的 虚函数表，再通过子类 的虚函数表找到子类的析构函数，从而使得子类的析构函数得以执行，子类的析构函数执行完毕后， 系统会自动执行父类的析构函数（这句是重点）！

```c++
class A {//虚析构函数应用举例
public:
	A(int _a) :a(_a) {};
	virtual int getValue();
	virtual  ~A() {
		printf("A deleted.");
	}
private:
	int a;
};
int A::getValue() {
	return a;
}

class B:public A {
public:
	B(int _a, int _b) :A(_a), b(_b) {};
	virtual int getValue();
	~B() {
		printf("B deleted.");
	}
private:
	int b;
};
int B::getValue() {
	return b;
}
int main() {
	A* pa;//基类A指针
	pa = new B(1,2);//基类A指针指向派生类B
	printf("%d\n", pa->getValue());
	delete pa;
}
/*输出：
2
B deleted.A deleted.
*/
////delete 删除的是指针指向的空间，不代表指针置NULL
```

## 静态成员、常成员

### 	静态成员变量：

- 我们不能把静态成员的初始化放置在类的定义中，它是所有对象共有的

- 所以应该再类内声明 static int a;

- 在类外定义 int A::a=0;

### 	静态成员函数：

- 静态成员函数没有 this 指针，且只能访问静态成员（包括静态成员变量和静态成员函数）

- 静态成员函数即使在类对象不存在的情况下也能被调用

### 	类中特殊成员变量的初始化问题：

- 常量变量：必须通过构造函数参数列表进行初始化。
- 引用变量：必须通过构造函数参数列表进行初始化。
- 普通静态变量：要在类外通过"::"初始化。
- 静态整型常量：可以直接在定义的时候初始化。
- 静态非整型常量：不能直接在定义的时候初始化。要在类外通过"::"初始化。

### 	常成员变量

- 一经初始化就不能再改变,并且只能通过初始化列表初始

### 	常成员函数/常对象

- 常对象里面的成员变量都不能改变，所以只能用常成员函数
- 常成员函数只能修改常成员变量，调用同类的常成员函数
- 不要误认为常对象中的成员函数都是常成员函数，常对象只保证其所有数据成员的值不被修改。
- 声明时候要把const加在参数表后边，不能加在前边，否则就是返回值是const类型，实现的时候也要把const加上

### 	常成员变量的初始化

- 只能通过初始化列表，构造函数里面相当于赋值了

```C++
#pragma once
#include <iostream>
class CTestBasic
{
public:
	//常成员：默认初始化
	CTestBasic() :conNum(0) {
		value = -1;
		pValue = new int;
	}
	//常成员：重载初始化
	CTestBasic(int num ) :conNum(num) {}

	//常成员函数：又成为只读函数，不能改变成员变量的值
	int getsNum() const;
	int getcNum() const;
	int get_scNum() const;
	int getPointerValue() const;
	~CTestBasic();
private:
	//const成员变量不能在类定义处初始化，[ 只能通过构造函数初始化列表进行 ]，并且必须有构造函数
	const int conNum;
	
	//static成员变量不能在构造函数初始化列表中初始化，因为它不属于某个对象.
	static int sNum;
	
	const static  int scNum = 100;
	int value;
	int* pValue;
};

//静态成员类外初始化
int CTestBasic::sNum = 1;

//函数定义也必须包含const 关键字
int CTestBasic::getsNum() const {return sNum;}
int CTestBasic::getcNum() const {return conNum;}
int CTestBasic::get_scNum() const {return scNum;}
 
int CTestBasic::getPointerValue() const
{
	*pValue = 200;
	//pValue++; Error: pValue的值不能改变
	return *pValue;
}
 
CTestBasic::~CTestBasic(){}
```



## 一些常用的类

### 		string

#### 				库函数

​			append() -- 在字符串的末尾添加字符

​			find() -- 在字符串中查找字符串

​			insert() -- 插入字符

​			length() -- 返回字符串的长度

​			replace() -- 替换字符串

​			substr() -- 返回某个子字符串

​        	eg. 

```c++
#include <iostream>
#include <string>
using namespace std;
int main(){
    //定义一个string类对象
    string http = "www.runoob.com";
   //打印字符串长度
   cout<<http.length()<<endl;
    //拼接
    http.append("/C++");
    cout<<http<<endl; //打印结果为：www.runoob.com/C++
    //删除
    int pos = http.find("/C++"); //查找"C++"在字符串中的位置
    cout<<pos<<endl;
    http.replace(pos, 4, "");   //从位置pos开始，之后的4个字符替换为空，即删除
    cout<<http<<endl;
    //找子串runoob
    int first = http.find_first_of("."); //从头开始寻找字符'.'的位置
    int last = http.find_last_of(".");   //从尾开始寻找字符'.'的位置
    cout<<http.substr(first+1, last-first-1)<<endl; //提取"runoob"子串并打印
    return 0;
```

- 几种输入方法的不同
- `cin`:遇到tab 空格 enter都结束

- `cin.getline (str,x)` x个字符包括反斜杠0 可以有空格

需要<string\>头文件：

- `getline(cin,str )`可以有空格
- `gets(s)`可以有空格

### 	**vector**

- 动态数组——顺序容器——stack（栈）的上位替代

#### 	构造函数

- `vector<int\> obj`          创造了一个vector int 类的obj数组
- `vector<int\> obj(10)`    创建一个obj数组，最多容纳10个数据
- `vector<int\> obj(10,3)`  创建一个obj数组，初始化10个数据为3
- `vector< vector<int\> \>` 二维数组

#### 	成员函数

- `begin()`,`end()` 获取首地址与尾地址
- `push_back(m)` 在最后插入数据m
- `pop_back()` 移除最后的数据
- `back()` 返回最后一个数据
- `clear()` 清除数据但不回收空间
- `size()`,`capacity()` size是当前容量 capacity是真实最大容量 一般来说相等 但是clear后不等
- `empty()` 判断是否为空 空返回1
- `insert(\__position\_\_,x)`    在指定位置插入x
- `insert(\__position\_\_,N,x)` 从指定位置开始插入N个x
- `erase(\__position\_\_)`       删除指定位置的元素
- `erase(\_\_positionBegin\_\_,\__positionEnd\_\_)`  删除指定区间内的元素

#### 	迭代器 (iterator)

- 遍历方法除了用数组之外还可以用迭代器（iterator）类似于指针

  声明：vector<int\>::iterator it;

  具体方法：`for(it=obj.begin();it!=obj.end();it++)    `

  ​               `cout<<*it<<" ";` 

- 迭代器实际上是对“遍历容器”这一操作进行了封装。

  在编程中我们往往会用到各种各样的容器，但由于这些容器的底层实现各不相同，所以对他们进行遍历的方法也是不同的。例如，数组使用指针算数就可以遍历，但链表就要在不同节点直接进行跳转。

  这是非常不利于代码重用的。例如你有一个简单的查找容器中最小值的函数findMin，如果没有迭代器，那么你就必须定义适用于数组版本的findMin和适用于链表版本的findMin，如果以后有更多容器需要使用findMin，那就只好继续添加重载……而如果每个容器又需要更多的函数例如findMax，sort，那简直就是重载地狱……

  我们的救星就是迭代器啦！如果我们将这些遍历容器的操作都封装成迭代器，那么诸如findMin一类的算法就都可以针对迭代器编程而不是针对具体容器编程，工作量一下子就少了很多！

  至于指针，由于指针也可以用来遍历容器(数组)，所以指针也可是算是迭代器的一种。但是指针还有其他功能，并不只局限于遍历数组。因为使用指针变量数组的操作太深入人心，c++stl中的迭代器就是刻意仿照指针来设计接口的

 

### algorithm

[algorithm头文件函数全集——史上最全，最贴心_算法头文件_来老铁干了这碗代码的博客-CSDN博客](https://blog.csdn.net/weixin_43899069/article/details/104450000)



## 模板

### 		函数模板

```c++
#include <iostream>
#include <string>
using namespace std;
template <typename T>
inline T const& Max (T const& a, T const& b) { 
    return a < b ? b:a; 
}
int main ()
{
    int i = 39;
    int j = 20;
    cout << "Max(i, j): " << Max(i, j) << endl; 
    double f1 = 13.5; 
    double f2 = 20.7; 
    cout << "Max(f1, f2): " << Max(f1, f2) << endl; 
    string s1 = "Hello"; 
    string s2 = "World"; 
    cout << "Max(s1, s2): " << Max(s1, s2) << endl; 
    return 0;
}
```

```
输出结果：
Max(i, j): 39
Max(f1, f2): 20.7
Max(s1, s2): World
```



### 		类模板

```c++
#include <iostream>
using namespace std;
template <typename T>
class Op{
public:
    T peocess(T v)
    {
        return v * v;
    }
};
int main()
{
    Op<int> opInt;
    Op<double> opDouble;
    cout << "5 * 5 = " << opInt.peocess(5) <<endl;
    cout << "0.5 * 0.5 = " << opDouble.peocess(0.5) <<endl;
}
```













**函数模板可以重载，只要它们的形参表不同即可。**

### C++ 中 typename 和 class 的区别

在 C++ Template 中很多地方都用到了 typename 与 class 这两个关键字，而且好像可以替换，是不是这两个关键字完全一样呢?

相信学习 C++ 的人对 class 这个关键字都非常明白，class 用于定义类，在模板引入 c++ 后，最初定义模板的方法为：

```
template<class T>......
```

这里 class 关键字表明T是一个类型，后来为了避免 class 在这两个地方的使用可能给人带来混淆，所以引入了 typename 这个关键字，它的作用同 class 一样表明后面的符号为一个类型，这样在定义模板的时候就可以使用下面的方式了：

```
template<typename
T>......
```

在模板定义语法中关键字 class 与 typename 的作用完全一样。

typename 难道仅仅在模板定义中起作用吗？其实不是这样，typename 另外一个作用为：使用嵌套依赖类型(nested depended name)，如下所示：

```c++
class MyArray 
{ 
    public：
    typedef int LengthType;
}
template<class T>
void MyMethod( T myarr ) 
{ 
    typedef typename T::LengthType LengthType; 
    LengthType length = myarr.GetLength; 
}
```

这个时候 typename 的作用就是告诉 c++ 编译器，typename 后面的字符串为一个类型名称，而不是成员函数或者成员变量，这个时候<u>如果前面没有 typename，编译器没有任何办法知道 T::LengthType 是一个类型还是一个成员名称(静态数据成员或者静态函数)</u>，所以编译不能够通过。

## 指针

### 	类指针

- `Student *b = new Student；`在定义b这个指针变量的时候并没有分配[内存](https://so.csdn.net/so/search?q=内存&spm=1001.2101.3001.7020)，只有执行new后才会分配内存，且为内存堆。是个永久变量,除非你释放它
- 是一个内存地址值，指向内存中存放的类对象（包括一些成员变量赋值；类指针可以指向多个不同的对象，这就是多态）
- 指针变量是间接访问，但可实现多态（通过父类指针可调用子类对象），并且没有调用构造函数；

### 常指针

#### 指向==常对象的指针==，指向==对象的常指针==

```c++
class A{
    A *p0; 			 //指向对象的指针
    const A *p1; //指向常对象的指针，指向常对象的指针只能是这种指针,不能通过此类指针修改A的数据
    A *const p2; //指向对象的常指针，p是指针常量，不能改变p的值
		/*主要看const修饰谁，const修饰指针的指向那就是指向常量的指针
const修饰指针本身*/
}
```


## 向上造型（Upcast)

在C++中，把子类的对象当做父类对象看待，就称为”向上造型“  (upcast)。

```c++
class manager: pubilc employee
{
manager();
};

manager pett;

employee *ep = &pett;   //就是upcast

employee &ep = pett;   //也是upcast



把父类的对象当做子类来看待，称为 downcast.

employee mob;

manager *lowe = &mob;   //downcast, 将父类对象转换成子类对象
```

注意：向上造型是安全的，向下造型是有风险的。



[^2]: 一个类的对象可以作为另一个类的数据成员，此时把该对象称为类的对象成员。也叫类的组合
