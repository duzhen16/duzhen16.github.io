---
layout:     post                    # layout, do not alter
title:      Linux SMP(2)           # title
subtitle:   SMP 启动  # subtitle
date:       2018-3-7              # time
author:     xSun                    # author
header-img: img/home-bg.jpg    #bg image
catalog: true                       # catalog or not
tags:                               #tags
    - SMP
    - Kernel
---
>最近总结Linux SMP相关原理，那就先从启动开始喽~~

## Linux内核启动

在`init/miain.c : start_kernel()`之前，需要经历BIOS等阶段，这里我们略去不谈，直接从start_kernel开始。[start_kernel()][1]函数几乎初始化内核的每个部件，例如初始化CPU设置、内存布局，建立主内核页目录等，0号进程也在此时由内核静态创建，它是唯一一个没有经过do_fork方式创建的进程，即就是init_task，从这时启，Linux才有了进程的概念。

其中，在[start_kernel()][2]的最后是[rest_init()][3]函数，此时，init_task创建两个内核线程，分别是kernel_init，和kthreadd，它们的pid分别为1和2.如下代码所示。再之后，init_task将自己的调度类型设置为idle_sched_class，此时，它就变成了idle进程。然后执行调度，kernel_init内核线程开始执行。到此为止，SMP环境下内核的启动和单核并没有什么本质的区别。
``` stylus
	kernel_thread(kernel_init, NULL, CLONE_FS);
	pid = kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES);
```

### SMP特殊情况
那么问题来了，SMP环境中不止有一个CPU，那么到底上述工作是在其中哪一个上完成呢？实际上，Linux对于此是这么解决的，在各个CPU中任意选择其中一个作为引导处理器，即BP，其他的称为AP。当init_task通过调用调度函数，使得kernel_init内核线程上BP运行时，将会执行[kernel_init][4]()函数。kernel_init内核线程将负责完成其他各个AP的初始化任务。有如下调用函数调用` kernel_init() --> kernel_init_freeable() --> smp_init();`。[kernel_init_freeable()][5]函数的主要任务有初始化SMP具体体系结构，设置全局CPU个数等，然后将会调用[smp_init()][6]，它的主要调用逻辑如下：

``` stylus
smp_init() 
		--> idle_threads_init();
			--> idle_init();
				--> fork_idle();
					--> init_idle();  /* init_idle - set up an idle thread for a given CPU */
		--> for_each_cpu()
				cpu_up(cpu);
					--> do_cpu_up(cpu, CPUHP_ONLINE)
						--> _cpu_up(cpu, 0, target);
							--> ret = cpuhp_kick_ap_work(cpu); // wake up idle

```
首先[smp_init()][7]调用[idle_threads_init()][8]，通过[for_each_possible_cpu][9]对各个AP执行[idle_init()][10]操作，[idle_init()][11]调用[fork_idle()][12]为此AP创建一个新的idle进程，然后调用[init_idle()][13]函数对新创建的idle进程进行相关的设置工作。
然后，[smp_init()][7]通过for_each_cpu唤醒各个AP上刚创建的idle进程。**由此可知，SMP环境中，并不是只有一个idle进程，而是对于每个CPU都会有一个idle进程，在合适的时机运行**。

kernel_init内核线程完成其他AP的初始化之后，再做一系列环境设置操作后，加载init进程，变成为用户空间的进程init，pid还是为1，成为所有用户空间进程的鼻祖。

## 小结
本文结合Linux中两个比较特殊的进程简要地描述了SMP环境中内核的启动过程，接下来要研究系统正常启动之后，内核的操作和单核有何不同。




  [1]: https://elixir.bootlin.com/linux/v4.10/source/init/main.c#L482
  [2]: https://elixir.bootlin.com/linux/v4.10/source/init/main.c#L482
  [3]: https://elixir.bootlin.com/linux/v4.10/source/init/main.c#L384
  [4]: https://elixir.bootlin.com/linux/v4.10/source/init/main.c#L952
  [5]: https://elixir.bootlin.com/linux/v4.10/source/init/main.c#L999
  [6]: https://elixir.bootlin.com/linux/v4.10/source/kernel/smp.c#L550
  [7]: https://elixir.bootlin.com/linux/v4.10/source/kernel/smp.c#L550
  [8]: https://elixir.bootlin.com/linux/v4.10/source/kernel/smpboot.c#L65
  [9]: https://elixir.bootlin.com/linux/v4.10/source/include/linux/cpumask.h#L716
  [10]: https://elixir.bootlin.com/linux/v4.10/source/kernel/smpboot.c#L49
  [11]: https://elixir.bootlin.com/linux/v4.10/source/kernel/smpboot.c#L49
  [12]: https://elixir.bootlin.com/linux/v4.10/source/kernel/fork.c#L1894
  [13]: https://elixir.bootlin.com/linux/v4.10/source/kernel/sched/core.c#L5272