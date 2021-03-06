---
layout:     post             # layout, do not alter
title:      校招准备          # title
subtitle:   数据结构，数据库		    # subtitle
date:       2018-8-21         # time
author:     xSun             # author
header-img: img/DSC_4446.jpg  #bg image
catalog: true                # catalog or not
tags:                        #tags
    - 校招
    - 数据结构
    - 数据库
 
---

> 这几个月来，一直在准备校招，零零碎碎的从各个渠道也总结了很多知识点，现在进行一下整理，一方面是对自己的交代，另一方面也希望可以给别人提供参考。
 
# 数据结构

## 二叉树公式

n0,n1,n2分别表示度为0,1,2的节点，**二叉树中的度定义为出度**

- 非空二叉树 `1 + n2 = n0 `  
- 完全二叉树中第一个叶子节点的编号为`n / 2 + 1`, 编号从1开始
- 在二叉树中，第i层的结点总数不超过`2 ^ (i - 1)`
- 具有n个结点的完全二叉树的深度为`int（logn）+ 1`  
- 完全二叉树 `n1 == 0 或者 1`
- 一个具有n个节点的完全二叉树,其叶子节点的个数n0为: `n / 2 向上取整`，或者`(n + 1) / 2 向下取整`
- 某二叉树的前序后续遍历结果恰好相反，则它的`高度 == 节点数`

---

## B树，B+树

### B树
#### 定义
1. 有一个根节点，根节点只有一个记录和两个孩子或者根节点为空；
2. 每个节点记录中的key和指针相互间隔，指针指向孩子节点； 
3. d是表示树的宽度，除叶子节点之外，其它每个节点有[d/2,d-1]条记录，并且些记录中的key都是从左到右按大小排列的，有[d/2+1,d]个孩子； 
4. 在一个节点中，第n个子树中的所有key，小于这个节点中第n个key，大于第n-1个key，比如下图中B节点的第2个子节点E中的所有key都小于B中的第2个key 9，大于第1个key 3; 
5. 所有的叶子节点必须在同一层次，也就是它们具有相同的深度。
![B树](https://site-images-1256908946.cos.ap-shanghai.myqcloud.com/B%E6%A0%91.jpg)

#### 性质
一个度为d的B-Tree，设其索引N个key，则其树高h的上限为logd((N+1)/2)，
检索一个key，其查找节点个数的渐进复杂度为O(logdN)。
从这点可以看出，B-Tree是一个非常有效率的索引数据结构。
B树中的key值就是检索的索引值，首先查找到某个key，在找到此key对应的data。

### B+树
#### 定义
B+树是应文件系统所需而产生的一种B-树的变形树。一棵m阶的B+树和m阶的B-树的差异在于：
1. 有n 棵子树的结点中含有n 个关键码；
2. 所有的叶子结点中包含了全部关键码的信息，及指向含有这些关键码记录的指针，且叶子结点本身依关键码的大小自小而大的顺序链接。
3. 所有的非终端结点可以看成是索引部分，结点中仅含有其子树根结点中最大（或最小）关键码。

![B+树](https://site-images-1256908946.cos.ap-shanghai.myqcloud.com/B%2B%E6%A0%91.jpg)

做这个优化的目的是为了提高区间访问的性能，例如图4中如果要查询key为从18到49的所有数据记录，当找到18后，只需顺着节点和指针顺序遍历就可以一次性访问到所有数据节点，极大提到了区间查询效率。

---

## 散列表冲突处理
### 开放地址法

#### 线性探测发
线性再散列法是形式最简单的处理冲突的方法。插入元素时，如果发生冲突，算法会简单的从该槽位置向后循环遍历hash表，直到找到表中的下一个空槽，并将该元素放入该槽中（会导致相同hash值的元素挨在一起和其他hash值对应的槽被占用）

- 缺点：
	1. 处理溢出需要另写程序
	2. 删除非常困难。将某个元素删除后，产生空槽，再查找时认为该对应的元素不存在，产生错误
	3. 容易产生堆聚现象

#### 线性补偿探测法
将上述线性探测法的步长从1改为Q，即 `hash ＝ (hash ＋ 1) % m` 改为：`hash ＝ (hash ＋ Q) % m = hash % m + Q % m`，而且要求 `Q 与 m 互质`，以便能探测到哈希表中的所有单元。

#### 伪随机法
将上述线性探测法的步长从1改为改为随机数。这样可以避免或减少堆聚现象。

### 链地址法 hashmap
当发生冲突时，将冲突的向链接到同一链表中。

- 优点
  - 处理方式简单，且无堆聚现象，非同义词不会发生冲突
  - 节点的存储空间是动态申请，适合于构造之前无法确定表长的情况
  - 删除节点的操作容易实现，不会影响到其他节点
- 缺点
  - 指针需要额外的内存空间

### 再散列法
当发生冲突时，使用第二个或第三个hash函数，多次进行计算。缺点是增加了计算的时间。

### 建立公共缓冲区
建立一个公共的缓冲区，当发生冲突时，将冲突想存储在公共缓冲区中。

---

## 排序算法总结
### 内排序
以下是各种排序算法的时间复杂度和空间复杂度的列表总结。

![sort](https://site-images-1256908946.cos.ap-shanghai.myqcloud.com/sort.jpg)
[代码实现][2]

>>算法的稳定性以及初始输入是否会影响复杂度。
>>
- **选堆快基不稳**
- **选堆归基不变**

### 外排序
所谓外排序，顾名思义，即是在内存外面的排序，因为当要处理的数据量很大，而不能一次装入内存时，此时只能放在读写较慢的外存储器（通常是硬盘）上。

外排序通常采用的是一种**“排序-归并”**的策略。在排序阶段，先读入能放在内存中的数据量，调用某个内排序算法进行排序，将其结果输出到一个临时文件，依此进行，将待排序数据组织为多个有序的临时文件；而后在归并阶段将这些临时文件组合为一个大的有序文件，也即排序结果。

一般采用**败者树**的算法，将各个已排好序的部分结果进行整个。

##### 败者树
叶子节点记录k个段中的最小数据，然后两两进行比赛。败者树是在双亲节点中记录下刚刚进行完的这场比赛的败者，让胜者去参加更高一层的比赛。决赛，根节点记录输者，所以需要重建一个新的根节点，记录胜者(如下图节点0）

![败者树1](https://site-images-1256908946.cos.ap-shanghai.myqcloud.com/%E8%B4%A5%E8%80%85%E6%A0%911.jpg)

每路的第一个元素为胜利树的叶子节点，（5,7）比较出5胜出7失败成为其根节点，（29,9）比较9胜出29失败成为其根节点，胜者（5,9）进行下次的比赛9失败成为其根节点5胜出输出到输出缓冲区。由第一路归并段输出，所有将第一路归并段的第二个元素加到叶子节点如下图：

![败者树2](https://site-images-1256908946.cos.ap-shanghai.myqcloud.com/%E8%B4%A5%E8%80%85%E6%A0%912.jpg)

>> 资料来源于网络，侵删。

---

# 数据库

## ACID
数据库的四个特性：

1. 原子性（Atomicity） 
事务被视为不可分割的最小单元，事务的所有操作要么全部提交成功，要么全部失败回滚。 
回滚可以用日志来实现，日志记录着事务所执行的修改操作，在回滚时反向执行这些修改操作即可。 

2. 一致性（Consistency） 
数据库在事务执行前后都保持一致性状态。 
在一致性状态下，所有事务对一个数据的读取结果都是相同的。 

3. 隔离性（Isolation） 
一个事务所做的修改在最终提交以前，对其它事务是不可见的。 

4. 持久性（Durability） 
一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失。 可以通过数据库备份和恢复来实现，在系统发生奔溃时，使用备份的数据库进行数据恢复。

---

## MySQL两种索引引擎

### 局部性原理

为了提高效率，要尽量减少磁盘I/O。为了达到这个目的，磁盘往往不是严格按需读取，而是每次都会**预读**，这样做的理论依据是计算机科学中著名的**局部性原理**：当一个数据被用到时，其附近的数据也通常会马上被使用。程序运行期间所需要的数据通常比较集中。

**预读的长度一般为页（page）的整倍数。**当程序要读取的数据不在主存中时，会触发一个缺页异常，此时系统会向磁盘发出读盘信号，磁盘会找到数据的起始位置并向后连续读取一页或几页载入内存中，然后异常返回，程序继续运行。

根据B-Tree的定义，可知检索一次最多需要访问`h-1`个节点（根节点常驻内存）。数据库系统的设计者巧妙**利用了磁盘预读原理，将一个节点的大小设为等于一个页，这样每个节点只需要一次I/O就可以完全载入。**为了达到这个目的，在实际实现B-Tree还需要使用如下技巧：每次新建节点时，直接申请一个页的空间，这样就保证一个节点物理上也存储在一个页里，加之计算机存储分配都是按页对齐的，就实现了一个node只需一次I/O。

B-Tree中一次检索最多需要`h-1`次I/O（根节点常驻内存），渐进复杂度为`O(h)=O(logdN)`。一般实际应用中，出度d是非常大的数字，通常超过100，因此h非常小（通常不超过3）

综上所述，如果我们采用B-Tree存储结构，搜索时I/O次数一般不会超过3次，所以用B-Tree作为索引结构效率是非常高的。

### MySQL索引实现--B+树
两种搜索引擎都依托于B+树。

#### MyISAM索引实现
MyISAM引擎使用B+Tree作为索引结构，叶节点的data域存放的是数据记录的地址。MyISAM的索引方式也叫做“非聚集”的，之所以这么称呼是为了与InnoDB的聚集索引区分。
主索引和辅助索引的data值都是对应记录的地址。

![MyISAM](https://site-images-1256908946.cos.ap-shanghai.myqcloud.com/my.jpg)

#### InnoDB索引实现

虽然InnoDB也使用B+Tree作为索引结构，但具体实现方式却与MyISAM截然不同。
第一个重大区别是InnoDB的数据文件本身就是索引文件。在InnoDB中，表数据文件本身就是按B+Tree组织的一个索引结构，这棵树的叶节点data域保存了完整的数据记录。这个索引的key是数据表的主键，因此InnoDB表数据文件本身就是主索引。

![InnoDB](https://site-images-1256908946.cos.ap-shanghai.myqcloud.com/in1.jpg)

第二个与MyISAM索引的不同是InnoDB的辅助索引data域存储相应记录主键的值而不是地址。换句话说，InnoDB的所有辅助索引都引用主键作为data域。例如，下图为定义在Col3上的一个辅助索引：

![InnoDB2](https://site-images-1256908946.cos.ap-shanghai.myqcloud.com/In2.jpg)

另外，我们也就很容易明白为什么不建议使用过长的字段作为主键，因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大。

>> 资料来源于网络，侵删。

---


[^_^]: refs here:

[1]:http://www.xsun24.top/
[2]:https://github.com/duzhen16/algo/blob/master/sort.cpp
