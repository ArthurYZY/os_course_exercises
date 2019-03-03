# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

  首先CPU是完成上述功能必不可上的硬件。

  * 进程切换：硬件需要有时钟终端
  * 虚存：需要硬件支持地址映射，TLB、MMU等硬件
  * 文件系统：需要有稳定的，非电易失性的存储介质，如磁盘等

  对于特权指令，相应的应该有：

  * 提供中断使能，触发软中断
  * 控制内存的寻址模式，设置页表等指令
  * 文件系统的的I/O指令

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

  早期8086 CPU只有一种工作方式即实模式，数据总线16位，地址总线20位，实模式下所有寄存器都是16位，80286开始后有了保护模式，从80386开始CPU的数据和地址总线均为32位。但是为了向下兼容，intel的CPU中都保留了实模式。

  两种模式最大的区别是进程内存是否收到保护

     * 实模式：将整个物理内存看成分段的区域，代码和数据放在不同的区域，系统程序和用户程序没有区别，而且指针指向的是实际物理地址，应用程序可以修改系统应用的或者其他用户的应用程序数据，非常不安全。	
     * 保护模式：物理内存不能被直接访问，程序所用的虚拟地址需要OS进行地址的映射转换为地址，此过程对应用程序是透明的。

  三种地址的理解：

  * 物理地址：和硬件对应的地址，是CPU提交到总线上的“真实的地址”
  * 逻辑地址：物理地址虚拟化的结果，和实际的物理地址具有映射的关系。逻辑地址通常提供给应用程序来使用，让应用程序不需要考虑实际的物理地址使用情况。
  * 线性地址：逻辑地址到物理地址映射的中间层，CPU通过Segment机制下形成的地址空间


- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征?

  RISC-V的特权模式一共有三种，分别是：

  * 运行最可信代码的机器模式（machine mode）
  * 为linux，FreeBSD和windows等操作系统提供支持的监管者模式（supervisor mode）
  * 用户模式

  三种模式的主要不同在于权限的不同，机器模式权限最高，监管者模式其次，最后是用户模式

  * 机器模式：对内存，I/O和一些对于启动和配置系统来说必要的底层功能有着完全的使用权。其最重要的特性是拦截和处理异常。使用物理地址

  * 监管者模式：提供了基于页面的虚拟内存，需要负责维护分页系统。异常委托机制会选择性的将中断和同步异常交给监管者模式处理。

  * 用户模式：权限最低，使用虚拟地址。

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义

```C++
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

​	":"的后面的数字代表每一个成员所使用的使用的bit位数

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

> **intr = 0x20003**
>
> PS:结构体的成员占用的bit是先用低位。

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
