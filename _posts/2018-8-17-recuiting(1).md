---
layout:     post             # layout, do not alter
title:      校招准备          # title
subtitle:   C/C++基础（1）    # subtitle
date:       2018-8-17         # time
author:     xSun             # author
header-img: img/home-bg.jpg  #bg image
catalog: true                # catalog or not
tags:                        #tags
    - 校招
    - C/C++
---

> 这几个月来，一直在准备校招，零零碎碎的从各个渠道也总结了很多知识点，现在进行一下整理，一方面是对自己的交代，另一方面也希望可以给别人提供参考。

## 基础原理

### 进程地址空间布局
以32位为例，进程的用户地址空间布局大致如下图所示。
![process_space](http://p194hb5ge.bkt.clouddn.com/process_space.jpg)

段|说明
:----:|:------------------------------------:
BSS |	静态存储区的一部分，用来存放未初始化的全局变量
数据段|	静态存储区的一部分，用来存储已初始化的全局变量和static声明的变量
代码段|存放程序代码的内存区，为程序代码在内存中的映射
堆|	为变量动态分配内存的区域，堆区大小不固定
栈|存放程序的局部变量和形参，函数调用时栈用来传递参数和返回值
动态存储区|堆，栈
静态存储区|BSS，数据段

查看进程地址映射可以使用 **cat /proc/pid/maps**命令，即就是查看maps文件，可以看到此进程所关联的vma的情况，包括地址范围，读取权限，映射文件等。如下图：

![process_maps](http://p194hb5ge.bkt.clouddn.com/process_maps.png)

**由输出并结合上图可知，如果在程序里访问NULL指针，会产生core dump，因为在进程地址空间中[0, 400000]此段为未映射，不属于该进程地址空间内。**

静态变量（静态局部变量，静态全局变量）和全局变量都位于**静态存储区**，他们的共同特点是生存周期贯穿于真个程序执行过程。区别在于作用范围不同，全局变量可作用于所有的函数，静态变量只能用于所定义的函数。

**int 型全局变量，静态局部变量的初始值都为0。
int型的局部变量的初始值为随机值。
要在一个文件中使用与局部变量同名的全局变量，需要加上：：**



```
#include <stdio.h>
int BSS; //存储在BSS区
Static char * p; // BSS
int data = 10; //存储在数据段
Static char * q = "abc"; // q在数据区，“abc"在文本常量区，值不能修改
int main
{
	static int a = 20; //存储在数据段
	int x, y; // 存储在栈上
	char *p = (char *)malloc(20); //存储在堆上
	int b = foo(x,y); //用栈进行参数传递
	int data = 30;
	printf("%d\n",::data); //访问与局部变量同名全局变量
	free(p);
	return 0;
}
int foo( int x, int y)
{
	int b = x + y;
	return b; // 返回值存储在eax寄存器中
}

```
---

[^_^]: refs here:

[1]:http://www.xsun24.top/
