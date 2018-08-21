---
layout:     post             # layout, do not alter
title:      校招准备          # title
subtitle:   C/C++基础（3）    # subtitle
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

# 基础

## 虚函数

继承类可以重写父类中的虚函数，可以使用父类的**指针或引用**，让其指向不同的子类对象，而调用不同子类的虚函数，实现多态。这种机制是叫动态绑定，具体执行哪个版本的函数，只有到运行时才知道。只有是使用指针或是引用的方式才可以实行这种机制。 

**构造函数之外的非静态函数**都可以被定义为虚函数，包括析构函数。

可以在虚函数声明上加上=0，使得其为纯虚函数，纯虚函数不能定义，只能声明，有纯虚函数的类为抽象类，不能实例化。它的作用就是提供统一的编程接口和函数规范。

如果一个类中声明了纯虚函数，其派生类中没有对该函数定义，那该函数在派生类中仍为纯虚函数，凡是包含纯虚函数的类都是抽象类。

在**编译阶段**，如果某个类含有虚函数，编译器在构造函数结尾处隐式地添加为vptr赋值的操作。

#### vtable,vptr

Foo fo时，会将fo对象定义在栈区，因为它只是一个局部变量，假设Foo类中定义了虚函数，那么如上述，查看构造函数的汇编，可以看到，会在.rodata区创建该类的vtable，fo对象的内存布局中的第一个字段就是该地址，即为vptr，指向该类的vtable，vtable中依次为该类虚函数的指针，指向虚函数的定义代码，位于.text区

[参考链接][2]

如果子类有多个父类，那么子类中就会有多个vptr，假设有m个父类，其中n个有虚函数，那么子类中也会有n个vptr，分别指向不同的vtable。

虚析构函数可以使得在释放一个对象时，找到合适对象的析构函数，是哪个子类的，还是父类的。

---

## 引用

### 引用的底层实现

```c++
10000115f:	48 8d 45 e8 	        leaq	-24(%rbp), %rax
...
; int a = 10;
10000116a:	c7 45 e8 0a 00 00 00 	movl	$10, -24(%rbp)
; int &b = a;
100001171:	48 89 45 e0 	        movq	%rax, -32(%rbp)
; b++;
100001175:	48 8b 45 e0 	        movq	-32(%rbp), %rax
100001179:	8b 08 	                movl	(%rax), %ecx
10000117b:	83 c1 01 	            addl	$1, %ecx
10000117e:	89 08 	                movl	%ecx, (%rax)
```

由此汇编码可以看出，编译器将引用实际上实现为指针的形式，会在当前函数栈帧内的另外一处内存中，存放所引用的变量的地址。并且对引用的操作实际上是采用间接寻址的方式，机理和指针是类似的。

### 引用和指针

- 相同点 

  都是地址的概念；

  指针指向一块内存，它的内容是所指内存的地址；引用是某块内存的别名。都是地址的概念；

- 不同点

  - 指针是一个实体，而引用仅是个别名；

  - 引用使用时无需解引用(*)，指针需要解引用； 
  - 引用只能在定义时被初始化一次，之后不可变；指针可变；
  -  引用没有 const，指针有 const，const 的指针不可变；
  -  引用不能为空，指针可以为空；
  -  “sizeof 引用”得到的是所指向的变量(对象)的大小，而“sizeof 指针”得到的是指针本身(所指向的变量或对象的地址)的大小；但是当引用作为成员时，其占用空间与指针相同（没找到标准的规定）。
  - 指针和引用的自增(++)运算意义不一样；

---

## new/delete和malloc/free的区别

new可以理解成两步:

1.  调用::operator new分配内存，如果内存不足失败，抛出异常； 

2. 调用构造函数构造对象。

delete可以理解成两步：

1. 调用析构函数将对象析构

2. 调用：：operator delete释放内存。

#### 区别

1. malloc和free是库函数，而new和delete是C++操作符；

2. new不需要提供需要的空间大小，比如’int * a = new，malloc需要指定大小，例如’int * a = malloc(sizeof(int))；

3. new在动态分配内存的时候可以初始化对象，调用其构造函数，delete在释放内存时调用对象的析构函数。而malloc只分配一段给定大小的内存，并返回该内存首地址指针，如果失败，返回NULL。

4. opeartor new /operator delete可以重载，而malloc不行

5. new可以调用malloc来实现，但是malloc不能调用new来实现
6. 对于数据C++定义new[]专门进行动态数组分配，用delete[]进行销毁。new[]会一次分配内存，然后多次调用构造函数；delete[]会先多次调用析构函数，然后一次性释放。

---

## STL容器的底层实现 

| STL                  |                                                       底层 |
|:--------------------: | :---------------------------------------------------------: |
| vector               |                                         数组，支持随机访问 |
| list                 |                                     双向链表，支持快速增删 |
| deque                | 中央控制器和多个连续缓冲区，支持首位快速增删，支持随机访问 |
| stack                |            deque、list封闭底部端口，不支持遍历，没有迭代器 |
| queue                |     dequel、list封闭底部出，顶部入，不支持遍历，没有迭代器 |
| proority_queue，heap |                                               一般为vector |
| set                  |                                   红黑树，键值不重复，有序 |
| multiset             |                                   红黑树，键值可重复，有序 |
| map                  |                                   红黑树，键值不重复，有序 |
| multimap             |                                   红黑树，键值可重复，有序 |
| hash_set             |                                       hash表，无序，不重复 |
| hash_multiset        |                                       hash表，无序，可重复 |
| hash_map             |                                       hash表，无序，不重复 |
| hash_multimap        |                                       hash表，无序，可重复 |
| unordered_set        |                                       hash表，无序，不重复 |
| unordered_multiset   |                                       hash表，无序，可重复 |
| unordered_map        |                                       hash表，无序，不重复 |
| unordered_multimap   |                                       hash表，无序，可重复 |

> > queue和stack为适配器，是对容器的进一步封装。

---

## STL相关

### vector

vector是动态空间，随着元素的加入，会自动扩充空间。其内部实际上有一个指针，指向动态申请的内存首部。

`v.size() = v.end() - v.begin(); `

`v.capacity = v.end_of_storage - v.begin();`

vector的迭代器就是普通指针。

当vector容量满时，再进行push操作，实际容量会自动扩充为原来的两倍。当pop时，不会再将实际容量缩小。

### list

list底层是双向链表，各个节点的内存空间不连续。list是双向环链表，因此只需要一个指针就能遍历所有节点。

list的迭代器为普通指针，不过需要对指针的操作做重载。如需要实现`++`，`--`操作，`++`实际就是`it = it->next`，`--`实际就是`it = it->prev`等。



### deque

deque是双端队列，再队列的任一端都可以进行入队/出队的操作，要保证时间复杂度为`O(1)`。deque的底层结构比较复杂。由一个中央控制器和多个连续的缓冲区组成。因此，迭代器的设计比较复杂，对deque进行排序也是现将deque复制到vector，对vector排序，再将结果复制回deque。

deque的中控器是一个连续空间，由map指针指向，map实际是一个`T**`，其中的每个元素都是一个指针，指向某一个缓冲区，而这些缓冲区才是存储数据的实际载体。

deque迭代器要满足迭代器的一般目标，它的设计大致如下：

```c++
struct iterator{
    ...
    T * cur;    		 //指向某缓冲区当前的某个元素
    
    T* first;    		 //指向某个缓冲区的开头
    
    T* last;  			 //指向某缓冲区的结尾（包含后备空间）
    
    map_pointer node;    //指向中控器map的某个元素，此元素指向当前缓冲区。
    
    ...
   };
```

因此deque的迭代器的`++`，`--`操作，不仅需要考虑在某个缓冲区内部的移动，还要考虑跨越缓冲区的移动。`++`时需要判断是否已经到了当前缓冲区的结尾，如果是，则需要将`node + 1`，然后`cur`指向下一个缓冲区的开头。

`--`是需要判断是否已经到了当前缓冲区的开头，如果是，则需要将`node - 1`，然后将`cur`指向上一个缓冲区的结尾。

`+=n, -=n, +n, -n`的操作都要判断是否有跨越缓冲区的情况。

deque除了要维护map指针外，还要维护指向一个缓冲区的第一个元素`start`，和最后一个缓冲区的最后一个元素`finish`，这两个都是前面定义的迭代器类型，这是它本身功能的需要。还需要记录当前map区域的大小，当map容量不足时，需要重新分配更大的内存区。

### RB-Tree

RB-tree的迭代器设计主要是要实现`++，--`，`++`是找到下一个大于当前节点的，`--`是下一个小于当前节点的。大于当前节点的就是右子树的最左边，小于当前节点的就是左子树的最右边。**另外要考虑特殊情况，没有子树的情况。**

### alloc

考虑到分配小型内存区可能造成内存碎片的问题，STL采用双层级配置器。

第一级配置器直接使用`malloc()`和`free()` .对应的接口为` __malloc_alloc_template`

第二级配置器视情况不同采用不同的策略：当申请内存超过128byte时，直接调用第一级配置器；否则使用内存池的方式。对应的接口为`__default_alloc_template`

可以使用`_USE_MALLOC`为选项进行配置。

无论alloc被定义为第一还是第二级配置器，都需要使用simple_alloc类再做以包装。STL所有容器都使用此作为内存分配的接口。

---



[^_^]: refs here:

[1]:http://www.xsun24.top/
[2]:http://blog.kongfy.com/2015/08/%E6%8E%A2%E7%B4%A2c%E8%99%9A%E5%87%BD%E6%95%B0%E5%9C%A8g%E4%B8%AD%E7%9A%84%E5%AE%9E%E7%8E%B0/