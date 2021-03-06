# lec5: SPOC思考题

##**提前准备**
（请在上课前完成）

- 完成lec５的视频学习和提交对应的在线练习
- git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises in github repos。这样可以在本机上完成课堂练习。
- 理解连续内存动态分配算法的实现（主要自学和网上查找）

NOTICE
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 思考题
---

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次

1.操作系统中存储管理的目标是什么？

* 抽象：逻辑地址空间
* 保护：独立地址空间
* 共享：访问相同的内存
* 虚拟化：更大的地址空间


### 5.2 地址空间和地址生成
1.描述编译、汇编、链接和加载的过程是什么？

* 编译：将高级语言写的代码转化为汇编代码
* 汇编：将汇编代码转化为机器码
* 链接：将多个二进制机器码进行整合，组合成一个可执行的程序
* 加载：将程序从外存中加载到内存中

2.动态链接如何使用？尝试在Linux平台上使用LD_DEBUG查看一个C语言Hello world的启动流程。  (optional)



### 5.3 连续内存分配
1.什么是内碎片、外碎片？

* 内碎片：内存块分配给一个进程时，进程可能无法完全这个内存块里的所有内存，因此会产生内部碎片
* 外碎片：两个进程分配到的内存块之间存在没有被分配利用的内存，叫做外碎片

2.最先匹配会越用越慢吗？请说明理由（可来源于猜想或具体的实验）？

一般来讲会变慢，因为一开始最先匹配会先利用低地址空间，但是到后期，低地址空间存在大量小的不连续的内存空间，一般都会找到较高地址的内存，因此时间长了之后，搜索的时间会增加

3.最差匹配的外碎片会比最优适配算法少吗？请说明理由（可来源于猜想或具体的实验）？

不一定，具体情况和分配的内存块的尺寸有关，一般来讲最差匹配应对中等大小的内存块分配时效果比较好，应为所拆分的内存块比较大，因此拆分完之后依旧比较大仍旧可以利用，这样一定程度减少了外碎片。但如果有后期很大的分区无法分配的话，可能有会有比较大的外碎片。对于最优匹配的话，对于小块的内存分配效果比较好的，但是最优分配容易产生比较多的小块的外碎片。

4.理解0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm算法中分区释放后的合并处理过程？ (optional)

查看空闲块的周围是否有其他的空闲块，如果符合了算法的要求则进行合并，并将空闲块管理数据插入链表中，更新空闲块的数据


### 5.4 碎片整理
1.对换和紧凑都是碎片整理技术，它们的主要区别是什么？为什么在早期的操作系统中采用对换技术？  

* 区别：对换是把内存暂时存放到外存中，而对换是把内存块的位置在内存中移动
* 原因：对换技术实现简单而且有效

2.一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

将进程从外存中读入内存，在读入过程中，操作系统将该进程标记为等待状态并继续调度其他进程

### 5.5 伙伴系统
1.伙伴系统的空闲块如何组织？

按照内存的大小有一系列的链表组织，将相同大小的内存区域首地址链接起来

在linux中主要数据结构如下

```C
struct zone {
	...
         ...	
	struct free_area	free_area[MAX_ORDER];			// 一个order使用一个free_area
	...
	...
}

struct free_area {
	struct list_head	free_list[MIGRATE_TYPES];		// 双向链表
	unsigned long		nr_free;						// 总共空闲块数量
};
```



2.伙伴系统的内存分配流程？伙伴系统的内存回收流程？

* 分配：由小到大在空闲块数组中找到最小的可用空闲块，如果空闲块过大（大小需要内存的二倍），则对空闲块进行二等分，直到满足要求为止
* 回收：把释放的块放入空闲数组，如果满足合并条件（大小相同，地址相邻，低地址空闲块起始地址为2^(i+1)的倍数则对相邻的空闲块进行合并

## 课堂实践

观察最先匹配、最佳匹配和最差匹配这几种动态分区分配算法的工作过程，并选择一个例子进行分析分析整个工作过程中的分配和释放操作对维护数据结构的影响和原因。

  * [算法演示脚本的使用说明](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.md)
  * [算法演示脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.py)

例如：
```
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p BEST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p FIRST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p WORST -n 5 -c
```

### 扩展思考题 (optional)

1. 请参考xv6（umalloc.c），ucore lab2代码，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在Linux应用程序/库层面，用C、C++或python来实现malloc/free，给出你的设计思路，并给出可以在Linux上运行的malloc/free实现和测试用例。


2. 阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。
