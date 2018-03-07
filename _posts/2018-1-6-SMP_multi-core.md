---
layout:     post                    # layout, do not alter
title:      Linux SMP(1) # title
subtitle:   SMP相关概念辨析 # subtitle
date:       2018-01-06              # time
author:     xSun                    # author
header-img: img/home-bg-o.jpg    #bg image
catalog: true                       # catalog or not
tags:                               #tags
    - HardWare
---

>最近跟老板讨论问题，发现有些基础概念有些模糊，在此做以辨析。

## 多核 & SMP 
多核处理器是在一个物理CPU中，集成了多个核core，这些core通过总线进行通信，共享内存等资源。我们大多数接触到的电脑都是单个CPU多core结构。可以使用命令`#cat /proc/cpuinfo |grep "cores"| uniq `查看CPU的core 的个数。

SMP的英文全称是Symmetric multiprocessing，即为对称多处理器系统，有两个或更多的相同的处理机（处理器）共享同一主存，由一个操作系统控制。实际上，SMP是一个较为广谱的概念。

**可以将单CPU多core也看成是SMP结构**，对称多处理架构就是这些核，它把这些核当作不同的处理器。

---

## Linux查看CPU信息
Linux中，/proc/cpuinfo文件记录着本机的CPU信息。使用命令 `cat /proc/cpuinfo`可以进行查看。大致的简略内容如下：

``` stylus
processor	: 0
model name	: Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz
cpu MHz		: 1676.757
cache size	: 6144 KB
physical id	: 0
core id		: 0
cpu cores	: 2
flags		: ... vmx smx ...

processor	: 1
model name	: Intel(R) Core(TM) i5-3470 CPU @ 3.20GHz
cpu MHz		: 1676.757
cache size	: 6144 KB
physical id	: 0
core id		: 1
cpu cores	: 2
flags		: ... vmx smx ...

...
```
其中，model name 表示该CPU的名称。flag中的vmx smx表示支持虚拟化技术。
### 物理CPU

物理CPU就是指真正插在机器主板上的CPU，其数量等与cpuinfo文件中不同的physical id的个数，可以使用命令`cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l `查看。

### 逻辑CPU

逻辑CPU是对操作系统所讲的语言，逻辑CPU数量=物理cpu数量 x cpu cores 这个规格值 x 2(如果支持并开启HT 超线程技术)

可以使用命令`cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l `查看。

Linux下top查看的CPU也是逻辑CPU个数。


