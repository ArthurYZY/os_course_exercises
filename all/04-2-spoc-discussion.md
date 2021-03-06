#lec9: 虚存置换算法spoc练习

## 视频相关思考题

### 9.1 页面置换算法的概念

1. 设计置换算法时需要考虑哪些影响因素？如何评判的好坏？
   * 考虑因素： 需要考虑算法的效率，是否能够尽量将最近可能会使用的页保留下来，并将最近不会使用的页面换出去。除此之外需要考虑算法是否在现有架构下算法是否可行，是否能够在当前环境下高效实现。
   * 评价标准：缺页率，平均访存时间等
2. 全局和局部置换算法的不同？
   * 局部置换算法是在当前进程中进行页面的置换
   * 全局置换算法考虑到不同进程对物理页的需求是不同的，它是在所有的可以换出的物理页范围内进行置换

### 9.2 最优算法、先进先出算法和最近最久未使用算法

1. 最优算法、先进先出算法和LRU算法的思路？
   * 最优算法：换出未来最长时间不被访问的页面
   * 先进先出：最先被换入内存的页面最先被换出（驻留最长的页面）
   * LRU：换出最长时间没有被访问的页面

### 9.3 时钟置换算法和最不常用算法

1. 时钟置换算法的思路？
   * 主要思路：结合FIFO与LRU的特点，通过设置访问位来帮助算法实现，比LRU实现更简单，而且又不想FIFO那样简易。
   * 具体实现：实现一个页的环形链表，并设置一个指针，在这个环上滑动，每次找第一个未被访问的页。每次指针扫过时会把页的访问位设置为0。而每次访问相应页面则会把页的访问位置1
2. 改进的时钟置换算法与时钟置换算法有什么不同？
   * 增加修改位，减少缺页处理的开销
3. LFU算法的思路？
   * 通过页面被访问的次数作为衡量标准，换出最近访问次数最少的页面


### 9.4 Belady现象和局部置换算法比较

1. 什么是Belady现象？如何判断一种置换算法是否存在Belady现象？
   * Belady：分配的物理页面增加但是缺页的次数也增加
   * 判断方法：构造反例或者证明算法是栈式置换算法
2. 请证明LRU算法不存在Belady现象。
   * 证明思路：证明LRU算法是栈式算法即可。容易证明，当物理页帧为n是存的n个页都是在一段时间内的页集合，当物理帧为n+1时，由于页的访问顺序是相同的，因此在相同的时间段内最近访问的n+1个页一定包含在n+1的帧中，显然这n+1个帧一定包含原来n个帧时存的页。故算法是栈式算法，因此不存在belady现象

### 9.5 工作集置换算法

1. CPU利用率与并发进程数的关系是什么？
   * 进程数少时：进程数的增加会提高CPU的利用率，利用起了CPU的等待时间
   * 进程数很多时：频繁的进程切换导致系统的局部性降低，会让CPU的利用率降低
2. 什么是工作集？
   * 一段时间内进程访问的所有逻辑页面的集合
3. 什么是常驻集？
   * 当前时刻进程实际驻留内存的页面集合
4. 工作集算法的思路？
   * 主要思路：将不在工作集的页面换出，缺页时换入需要的页面

### 9.6 缺页率置换算法

1. 缺页率算法的思路？
   * 保证适度的缺页率，在缺页率过高时增加常驻集，缺页率过低时减少常驻集

### 9.7 抖动和负载控制

1. 什么是虚拟内存管理的抖动现象？
   * 进程物理页面太少，导致换入换出频繁，从而使进程运行速度下降
2. 操作系统负载控制的最佳状态是什么状态？
   * 平均缺页间隔时间 = 缺页异常处理时间 
3. 局部置换算法（如FIFO, LRU等）是否能作为全局置换算法来使用？为什么？
   * 不能，局部置换算法会导致进程切换时进程占用的页面被全部换出

----

## 扩展思考题

1.  改进时钟置换算法的极端情况: 如果所有的页面都被修改过了，这时需要分配新的页面时，算法的performance会如何？能否改进在保证正确的前提下提高缺页中断的处理时间？

2.  如何设计改进时钟算法的写回策略?

3. （spoc）根据你的`学号 mod 4`的结果值，确定选择四种页面置换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试用例和测试结果。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)     

4. 请判断OPT、LRU、FIFO、Clock和LFU等各页面置换算法是否存在Belady现象？如果存在，给出实例；如果不存在，给出证明。

5. 了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！
	- 参考信息：
 	- [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
	 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
	 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
	 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
