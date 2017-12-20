---
layout:       post
title:          2017-12-20-进程内核栈
subtitle:    内核栈相关概念
date:         2017-12-20
author:     xSun
header-img: img/home-gb-o.jpg
catalog:   true
tags: 
    - Kernel
---

#进程的栈们
    在进程创建时，内核会为进程创建一系列数据结构，其中最重要的就是task_struct结构，它是进程描述符，表征进程在其生命周期内的所有特征。同时，内核为进程创建两个栈，一个是用户栈，一个是内核栈，顾名思义，分别是进程处于用户态和内核态时使用的栈。
	当进程通过系统调用等事件进入内核时，此时用户栈和内核栈进行切换，由用户栈切换到内核栈。内核将用户态时的堆栈寄存器的值保存在内核栈中，以便从内核栈切换回进程栈时能找得到用户栈的地址。但是，从进程栈切换到内核栈时，内核是如何找到该进程的内核栈的位置的呢？
	实际上，当进程每次陷入内核时，内核栈总是空的，因为进程在用户态运行时，内核栈没有存在的意义。当内核要检索到某个进程的内核栈时，实际上是借助前面所述的task_struct进程描述符结构体。
	需要注意的是，内核栈仅用于内核例程，Linux内核另外为中断提供了单独的硬中断栈和软中断栈。

---------
#内核栈
    task_struct定义在 include/linux/sched.h中，这个结构体积庞大，详细内容参考另一篇文章。这里我们只关心和内核栈相关的数据项：
	
  ```
  struct task_struct {
      struct thread_info thread_info;
	  ...
      void * stack;
	  ...
  }
  ```
  其中，thread_info是一个体系相关的描述符，不同的硬件体系所需要记录的标志是不同的，因此内核将和特定硬件体系相关的标志定义在此结构体中，再将其作为一个数据项定义在task_struct中，以保持task_struct的通用性。
  
  还有另外一个联合体 thread_union也很重要，它也定义在include/linux/sched.h中，如下：
  ```
  union thread_unoin {
      struct thread_info thread_info;
	    unsigned long stack[THREAD_SIZE/sizeof(long)];
  }
  ```
  在x86_64中，thread_union的大小为16KB，每个页为4KB，它所占4个页。如下图所示：
                                                    ![enter description here][1]
													
    
  由此图可以看出，在4个页面内，thread_info处于最低端，栈从高地址开始扩展，esp指向栈顶，task_struct中的stack指向此区域的基地址，是低12位对齐的。
----
#内核栈的创建、销毁
    
 1. 创建：内核在创建进程时，通过[alloc_thread_info_node()][2]创建进程的内核栈。
 2. 销毁：内核通过[free_thread_stack()][3]销毁进程的内核栈。


  [1]: img/kernel-stack-layout.png
  [2]: http://elixir.free-electrons.com/linux/v4.10/source/kernel/fork.c#L172
  [3]: http://elixir.free-electrons.com/linux/v4.10/source/kernel/fork.c#L214
