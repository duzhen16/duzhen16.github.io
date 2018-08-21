---
layout:     post             # layout, do not alter
title:      校招准备          # title
subtitle:   C/C++基础（2）    # subtitle
date:       2018-8-20         # time
author:     xSun             # author
header-img: img/DSC_4446.jpg  #bg image
catalog: true                # catalog or not
mermaid: true		     # support mermaid
tags:                        #tags
    - 校招
    - C/C++
---

> 这几个月来，一直在准备校招，零零碎碎的从各个渠道也总结了很多知识点，现在进行一下整理，一方面是对自己的交代，另一方面也希望可以给别人提供参考。

## 基础

### class & struct 
这两个关键字都可以用来定义一个类，唯一的区别在于默认的**访问方式不同**。

类别|默认访问方式
:---:|:---:
class |private;
struct|public

他们定义的类按照其成员变量的顺序，在内存中按字节对齐，
并且总的大小要被最大成员的大小整除

---

### 构造函数相关

1.如果类的成员变量有const、引用，那么必须使用构造函数初始化的形式为其提供初始值。

```
class F  {
	const int k;
	int p;
	int &r;
	
	F(const kk, int pp) : k(kk), p(pp), r(p) { }
}; 这种初始化的方式效率更高
```
构造函数初始化时必须采用初始化列表一共有三种情况:

1. 需要初始化的数据成员是对象(继承时调用基类构造函数)
2. 需要初始化const修饰的类成员
3. 需要初始化引用成员数据


2.成员变量初始化的顺序与他们在类定义中的顺序相同，与构造函数时写的顺序无关。

3.委托构造函数： F() : F(0,1) { } 将F()此构造函数的初始化工作委托给F(const kk, int pp)

4.拷贝构造函数：当用同类型的另一个对象初始化此对象，第一个参数必须引用，不能是值传递的方式。
Foo f1 = f2; 调用拷贝构造函数生成f1.
f1 = f3;
	
5.拷贝构造函数和赋值运算符的区别：关键在于有没有新的对象生成，拷贝构造函数导致的结果是将会产生一个新的对象f1,而赋值运算符是将一个已存在的对象f3赋值给另一个已近存在的对象f1。

---

### 析构函数

一个类只有一个析构函数，析构函数在对象最后一次使用之后调用，执行一些收尾工作，如释放资源等。

当一个对象的引用或者指针离开作用域时，**析构函数不会执行**，只有由程序员显示的delete时，才会调用析构函数。

析构函数可以实现为虚函数的方式。

---

### 只在堆 or 栈上创建对象

#### 只在堆上创建对象
类对象只能建立在堆上，就是编译器不能自动地静态地建立类对象。将析构函数定义为private，就可以使得此类定义的对象只在堆上生成。

```
class OnlyHeapClass
{
public:
    OnlyHeapClass()
    {
        cout << "construct\n";
    }
private:
    OnlyHeapClass()
	
    {
        cout << "deconstruct\n";
    }
};
```

#### 只在栈上创建对象
只有使用new运算符，对象才会建立在堆上，因此，只要禁用new运算符就可以实现类对象只能建立在栈上。将operator new()设为私有即可。

```
class OnlyStackClass
{
private:
    void* operator new(size_t t){ return nullptr;}
    // 函数的第一个参数和返回值都是固定的
    void operator delete(void* ptr){}
    // 重载了new就需要重载delete
};
```
---

### 静态成员

类的静态成员不属于某个具体的对象，这意味着静态成员的初始化不是由构造函数完成的，析构函数也不能销毁类的静态成员。

类的静态成员处于**静态区**。

类的静态成员不属于该对象，因此**不能使用this指针**。

```
  1 #include <iostream>
  2 using namespace std;
  3 class Cstatic {
  4 public:
  5     static int a;
  6     Cstatic() { cout << "static init " << a++ << endl; }
  7 };
  8 class A {
  9 public:
 10     int a;
 11     static Cstatic c1;
 12     A() { a = 1; cout << "A " << endl; }
 13 };
 14 class B : public A {
 15 public:
 16     int b;
 17     static Cstatic c2;
 18     B() { b = 2; cout << "B" << endl;}
 19 };
 20 int Cstatic::a = 0;
 21 B *b = new B();
 22
 23 int main()
 24 {
 25     return 0;
 26 }
 
 output：
 A
 B
```

---
### 常量对象

C++为了防止对对象的恶意修改，可以在定义某个类的对象时将其定义为常量对象：
const Foo foo(xxx);
常量对象不能调用类的普通成员函数，只能调用常量成员函数，例如`void func () const {}`,常量成员函数不能修改调用它的对象的内容。

因此，构造函数和析构函数不能被声明为const
非常量对象除了调用普通成员函数外，也可以调用常量成员函数。

但是如果将某个数据成员声明为`mutable`，那么此成员即使在常量对象中也可以被修改。

---

### 继承

1. 每个类控制它自己的构造函数，子类的构造函数执行时，先会按照生命顺序调用它父类的构造函数，初始化它由父类那里继承来的成员
2. 析构函数会先调用子类的析构，再调用父类的析构函数。
3. 使用final关键字，可以使得某个类不能被继承，即就是它不能被当做基类。
4. 不能使用子类的指针指向父类对象，因为有可能会访问到父类不存在的成员。
5. 将一个类的构造和析构设置为private，则此类不能被继承。
6. 多继承

![菱形继承](https://xsun24images.oss-cn-hangzhou.aliyuncs.com/images/%E8%8F%B1%E5%BD%A2%E7%BB%A7%E6%89%BF.jpg)

虚继承是一种机制，类通过虚继承指出它希望共享虚基类的状态。对给定的虚基类，无论该类在派生层次中作为虚基类出现多少次，只继承一个共享的基类子对象，共享基类子对象称为虚基类。虚基类用`virtual`声明继承关系就行了。这样，D就只有A的一份拷贝。

#### 继承 重载 隐藏

- 成员函数被重载的特征：
  - 相同的范围（在同一个类中）；
  - 函数名字相同；
  - 参数不同；
  - `virtual`关键字可有可无。

- 覆盖是指派生类函数覆盖基类函数，特征是：
  - 不同的范围（分别位于派生类与基类）；
  - 函数名字相同；
  - 参数相同；
  - 基类函数必须有`virtual`关键字。

- 隐藏是指派生类的函数屏蔽了与其同名的基类函数，规则如下：
  - 如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无virtual关键字，基类的函数将被隐藏（注意别与重载混淆）。
  - 如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有virtual 关键字。此时，基类的函数被隐藏（注意别与覆盖混淆)

[参考链接][2]



[^_^]: refs here:

[1]:http://www.xsun24.top/
[2]:https://www.nowcoder.com/test/question/done?tid=15425945&amp;qid=15106#summary
