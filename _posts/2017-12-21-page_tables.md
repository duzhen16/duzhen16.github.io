---
layout:     post                                
title:         页表们            
subtitle:   进程页表，内核页表  
date:       2017-12-21                    
author:    xSun                  
header-img:   img/post-bg-unix-linux.jpg   
catalog: true                    
tags:                              
    - Kernel
---

>Linux系统中的内存管理和页表映射关系一直是比较繁琐的，因此在此处做以小结，主要内容是内核页表、进程页表、内核地址空间映射及缺页处理的部分相关内容。此文随时更新。为便于描述，均使用32位环境做解释。

---
## **内核页表**

内核维持一组自己使用的页表，即就是驻留内存的主内核页全局目录，地址在swapper_pg_dir变量中。它是在系统启动初始化时建立。系统初始化后，此组页表并未被任何进程或是内核线程直接使用，它的最高目录项部分是作为参考模型，为系统中每个普通进程对应的页全局目录项提供参考。

### 主内核页表

Linux在初始化时，第0个进程是idle进程，或叫swapper。它是Linux的初始化阶段从无到有所创建的第0个内核线程，是所有进程的祖先。它的数据结构都是内核在初始化时静态分配的，此处我们关心[init_mm][1]，是由INIT_MM宏初始化的，是一个内存描述符。

``` stylus
struct mm_struct init_mm = {
	.mm_rb		= RB_ROOT,
	.pgd		= swapper_pg_dir,
	.mm_users	= ATOMIC_INIT(2),
	.mm_count	= ATOMIC_INIT(1),
	.mmap_sem	= __RWSEM_INITIALIZER(init_mm.mmap_sem),
	.page_table_lock =  __SPIN_LOCK_UNLOCKED(init_mm.page_table_lock),
	.mmlist		= LIST_HEAD_INIT(init_mm.mmlist),
	.user_ns	= &init_user_ns,
	INIT_MM_CONTEXT(init_mm)
};
```
其中，init_mm.pgd是页全局目录，主内核页表就是指页全局目录和它的子页表。

---

## **非连续内存区**

### 32位内核地址空间布局

32位下，内核地址空间有1G，即就是4G中最高的1G空间，逻辑地址范围是：0xC000,0000<—>0xFFFF,FFFF。在此1G的地址空间中，划分为几个区域，如下图所示：
![img](http://p194hb5ge.bkt.clouddn.com/内核地址空间布局.jpg)

 - 内存区开始部分包含的是对896M RAM进行映射的线性地址。直接映射末尾的地址保存在high_memory变量中。（参考ULK P74.）
 
 - 固定映射区包含固定映射的线性地址，类似于第一种直接映射，不过此区域线性地址可以映射到任意RAM区域。（参考ULK P78.）
 
  - 永久映射允许内核建立高端页框到内核地址空间的长期映射。（参考ULK P307）
 
 - VMALLOC区，也就是非连续内存区，可以使用连续的线性地址空间访问非连续的页框。图中整个VMALLOC区域实际上是划分为若干个小的vmalloc区，使用[vm_struct][2] 描述符对其进行管理。它可以用在为活动的交换区分配数据结构，为模块分配空间等场景。（参考ULK P343.）

### 分配非连续内存区

[vmalloc(long size)][3]函数为内核分配一非连续内存区，大致流程为：首先调用[get_vm_area()][4]在VMALLOC区查找一个合适的空闲区域，然后调用[kmalloc()][5]请求一组连续的页框，再然后调用[map_vm_area()][6]对内核页表进行修改，对二者建立映射关系。

需要注意的是，[map_vm_area()][7]并不触及进程的页表，**它只是修改内核的主内核页表**。因此当内核态进程访问非连续内存区时，发生缺页，是因为其页表内相应表项为空，此时需要用内核页表相应的部分对其进行同步，具体的过程在后面解释。

内核还可以使用[vmalloc_32()][8]分配非连续内存区，不过它只从ZONE_DMA和ZONE_NORMAL物理内存区分配页框。

内核还可以使用[vmap()][9]函数将非连续内存区映射到已经非配的页框。

### 释放非连续内存区

[vfree()][10]函数释放[vmalloc()][11]函数和[vmalloc_32()][12]创建的非连续内存区，使用[vunmap()][13]释放[vmap()][14]创建的内存区。逻辑上操作过程和分配相反，最后会调用unmap_vm_area()清除相关的映射关系。同样它修改的也是主内核页表。

---

## **进程页表**

32位下，进程可寻址的地址空间是4G，范围是0x0000,0000<—>0xFFFF,FFFF。其中，低3G是用户空间，高1G是内核空间。当进程运行在用户态时，它产生的地址是小于0xC000,0000，此时使用进程的用户态的页表。当进程运行在内核态时，使用内核态的页表。

进程内核态的页表是我们所关心的。在进程描述符task_struct中的struct mm_struct mm 字段就是该进程的内存描述符，其结构如上，就可以按图索骥，找到该进程的页目录的基地址。它的页目录中属于内核态部分，是从内核主页全局目录同步而来。实际上，所有进程的内核态页表都是从内核的主内核页表同步而来的。

---
## **缺页处理（部分）**

接下来，我们来看内核如何将自己的主内核页表同步到进程的内核态页表。前面我们提到，内核初始化后，其他进程并不直接使用主内核页表。当内核态进程第一次访问非连续性内存区地址时，去查询它自己的内核页表，其中相应的表项为空，产生page fault，缺页处理程序[do_page_fault()][15]被调用，缺页处理的逻辑十分复杂，因为当时的情况可能会比较复杂，详细的处理逻辑可以查阅其源码。我们此处只关心其中的一条通路。如下图：
![img](http://p194hb5ge.bkt.clouddn.com/do_page_fault.gif)

当进程处于内核态，对内核地址空间访问时，发生缺页，此时产生vmalloc_fault：

``` stylus
if (unlikely(fault_in_kernel_space(address))) {
		if (!(error_code & (PF_RSVD | PF_USER | PF_PROT))) {
			if (vmalloc_fault(address) >= 0)
				return;
```


[vmalloc_fault()][16]函数对此进行处理，其内部逻辑基本上是根据发生fault的地址，找到其应对应的pgd表项，然后将主内核页表中相应的表项同步过来，并做一系列的检查，包括此页目录项下属的各级页表都是否正常。如下代码。


``` stylus
static noinline int vmalloc_fault(unsigned long address)
{
    ...
	pgd = (pgd_t *)__va(read_cr3()) + pgd_index(address);
	pgd_ref = pgd_offset_k(address);
	if (pgd_none(*pgd_ref))
		return -1;

	if (pgd_none(*pgd)) {
		set_pgd(pgd, *pgd_ref);
	    ...
    ...
	if (pud_none(*pud_ref))
		return -1;

	if (pud_none(*pud) || pud_pfn(*pud) != pud_pfn(*pud_ref))
		BUG();
	...
	if (pmd_none(*pmd_ref))
		return -1;

	if (pmd_none(*pmd) || pmd_pfn(*pmd) != pmd_pfn(*pmd_ref))
		BUG();

    ...
	pte_ref = pte_offset_kernel(pmd_ref, address);
	if (!pte_present(*pte_ref))
		return -1;
    ...
	return 0;
}
```

如果一切正常，则函数返回0，处理完毕。否则，那就是内核本身出现了问题，但是内核对自己是很有信心的。

---

## **结束**

此处将内核页表和进程内核态页表相关做了小结，清理了自己以前混乱的逻辑。也应及时更新。


  [1]: http://elixir.free-electrons.com/linux/v4.10/source/mm/init-mm.c#L17
  [2]: http://elixir.free-electrons.com/linux/v4.10/source/include/linux/vmalloc.h#L32
  [3]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1776
  [4]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1396
  [5]: http://elixir.free-electrons.com/linux/v4.10/source/include/linux/slab.h#L478
  [6]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1301
  [7]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1301
  [8]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1898
  [9]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1898
  [10]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1545
  [11]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1776
  [12]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1898
  [13]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1569
  [14]: http://elixir.free-electrons.com/linux/v4.10/source/mm/vmalloc.c#L1898
  [15]: http://elixir.free-electrons.com/linux/v4.10/source/arch/x86/mm/fault.c#L1444
  [16]: http://elixir.free-electrons.com/linux/v4.10/source/arch/x86/mm/fault.c#L424