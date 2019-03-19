# lec7:  lab2 SPOC思考题

NOTICE

* 有"w4l1"标记的题是助教要提交到学堂在线上的。
* 有"w4l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
* 有"hard"标记的题有一定难度，鼓励实现。
* 有"easy"标记的题很容易实现，鼓励实现。
* 有"midd"标记的题是一般水平，鼓励实现。

## 提前准备

* 完成lec7\(lab2\)的视频学习和提交对应的在线练习
* git pull ucore\_os\_lab, v9\_cpu, os\_course\_spoc\_exercises 　in github repos。这样可以在本机上完成课堂练习。
* 理解如何实现建立页表（主要自学和网上查找）

## 第七讲视频相关思考题

### 7.1 了解x86保护模式中的特权级

1. X86有几个特权级？
* 4个特权级。在Ucore中一般只需要用到两个特权级（Ring0&Ring3）。

2. 不同特权级有什么区别？
* 指令：部分特权指令只能在高特权级下使用，否则会产生异常。
* 访存：不同特权级下允许访问的数据范围可能不同。
* 行为：不同特权级下相同指令行为可能不同。

3. 请说明CPL、DPL和RPL在中断响应、函数调用和指令执行时的作用。
* 访问门（Gate）时，从低优先级访问高优先级服务。CPL <= DPL门，CPL >= DPL段。
* 访问段（Segment）时，从高优先级访问低优先级数据。MAX(CPL,RPL) <= DPL段，即较高优先级的若比访问段的优先级要高，则允许访问。

4. 写一个示例程序，完成4个特权级间的函数调用和数据访问时特权级控制的作用。
* ？

### 7.2 了解特权级切换过程

1. 一条指令在执行时会有哪些可能的特权级判断？
* 访问数据时，需要确认CPL与DPL的关系。
* 发生中断和异常时，需要确认CPL与DPL门、DPL段的关系。

2. 在什么情况下会出现特权级切换？
* 在发生系统调用时，中断发生，中断返回时。

3. int指令在ring0和ring3的执行行为有什么不同？
* 在ring0，无需发生栈的切换，无需压入ss和esp。
* 在ring3，需要切换到ring0的栈，并压入ss和esp。

4. 如何利用int和iret指令完成不同特权级的切换？
* ring3->0，int后，修改栈中的CPL为0，并删去ss和esp，这样在iret后不会回到ring3。
* ring0->3，int后，修改栈中的CPL为3，并添加合适的ss和esp，这样在iret后就能切换到ring0。

5. TSS和Task Register的作用是什么？
* TSS在任务切换中起到任务的挂起和恢复的作用。在切换时，TSS能帮助保存现场的寄存器，而在恢复时将其复原到原有的寄存器中。
* Task Register是一个十六位的寄存器，用于保存TSS的段选择子。可以通过LTR来将其内容取出。

 > [Task state segment](https://en.wikipedia.org/wiki/Task_state_segment)

 > Reference: [Intel® 64 and IA-32 Architectures Software Developer Manuals](http://os.cs.tsinghua.edu.cn/oscourse/OS2017spring/lecture04?action=AttachFile&do=view&target=325462-sdm-vol-1-2abcd-3abcd.pdf) Page 2897/4684: 7.2.1 Task-State Segment (TSS)

### 7.3 了解段/页表

1. 一条指令执行时最多会出现多少次地址转换？
* 什么指令？和指令集、指令本身是有关系的。例如若只涉及到一次访存操作，则会出现2次地址转换（在页机制开启的情况下）：由逻辑地址转换到线性地址，再由线性地址转换到物理地址。

2. 描述X86-32的MMU地址转换过程；
* 由逻辑地址转换到线性地址，再由线性地址转换到物理地址。

### 7.4 了解UCORE建立段/页表

1. 分析MMU的使能过程，尽可能详细地分析在执行进入保护械的代码“movl %eax, %cr0 ; ljmp $CODE_SEL, $0x0”时，CPU的状态和寄存器内容的变化。
* `movl %eax, %cr0`这一句是设置CR0寄存器的PE标志位，让处理器处于保护模式。
* `ljmp $CODE_SEL, $0x0`这一句远跳指令会清空预取指令序列，然后移动到CS段描述符处，开始执行此处描述的地址0处的代码。

2. 分析页表的建立过程；
* 为页表分配一个页，然后清空它；
* 在页表和页目录表里填好相应的项，建立好映射关系。在Ucore中，在0xC0000000 - 0xF8000000映射到0x00000000 - 0x38000000。
* 将页机制使能。
* 更新GDT，将段机制从之前的不对等映射变为对等映射。

## 个人思考题

---

x86保护模式中权限管理无处不在，下面哪些时候要检查访问权限\(\)  \(w4l1\)

* [x] 内存寻址过程中
* [x] 代码跳转过程中
* [x] 中断处理过程中
* [ ] ALU计算过程中

请描述ucore OS建立页机制的准备工作包括哪些步骤？ \(w4l1\)


## 小组思考题

---

（1）（spoc）请用lab1实验的基准代码（即没有修改的需要填空的源代码）来做如下实验： 执行`make qemu`，会得到一个输出结果，请给出合理的解释：为何qemu退出了？【提示】需要对qemu增加一些用于显示执行过程的参数，重点是分析其执行的指令和产生的中断或异常。 可试试"qemu-system-i386  -d help"


（2）\(spoc\)假定你已经完成了lab1的实验,接下来是对lab1的中断处理的回顾：请把你的学号对37\(十进制\)取模，得到一个数x（x的范围是-1&lt;x&lt;37），然后在你的答案的基础上，修init.c中的kern\_init函数，在大约36行处，即

```
    intr_enable();              // enable irq interrupt
```

语句之后，加入如下语句\(把x替换为你学号 mod 37得的值\)：

```
    asm volatile ("int $x");
```

然后，请回答加入这条语句后，执行`make qemu`的输出结果与你没有加入这条语句后执行`make qemu`的输出结果的差异，并解释为什么有差异或没差异？


（3）对于lab2的输出信息，请说明数字的含义

```
e820map:
  memory: 0009fc00, [00000000, 0009fbff], type = 1.
  memory: 00000400, [0009fc00, 0009ffff], type = 2.
  memory: 00010000, [000f0000, 000fffff], type = 2.
  memory: 07ee0000, [00100000, 07fdffff], type = 1.
  memory: 00020000, [07fe0000, 07ffffff], type = 2.
  memory: 00040000, [fffc0000, ffffffff], type = 2.
```

修改lab2，让其显示`type="some string"` 让人能够读懂，而不是不好理解的数字1,2  \(easy\)

* [x]
>

（4）\(spoc\)有一台只有页机制的简化80386的32bit计算机，有地址范围位0~256MB的物理内存空间（physical memory），可表示大小为256MB，范围为0xC0000000~0xD0000000的虚拟地址空间（virtual address space）,页大小（page size）为4KB，采用二级页表，一个页目录项（page directory entry ，PDE）大小为4B,一个页表项（page-table entries PTEs）大小为4B，1个页目录表大小为4KB，1个页表大小为4KB。

```
PTE格式（32 bit） :
  PFN19 ... PFN0|NOUSE9 ... NOUSE0|WRITABLE|VALID
PDE格式（32 bit） :
  PT19 ... PT0|NOUSE9 ... NOUSE0|WRITABLE|VALID

其中：
NOUSE9 ... NOUSE0为保留位，要求固定为0
WRITABLE：1表示可写，0表示只读
VLAID：1表示有效，0表示无效
```

假设ucore OS已经为此机器设置好了针对如下虚拟地址&lt;--&gt;物理地址映射的二级页表，设置了页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐），其值为0。ucore OS在物理内存空间（0x1000~0x41000）已经建立和配置好了整个二级页表（包括页目录表和页表），且页目录表的index为0x300~0x363的页目录项的\(PT19 ... PT0\)的值=\(index-0x300+1\)。  
请写出一个translation程序（可基于python、ruby、C、C++、LISP、JavaScript等），输入是一个虚拟地址和一个物理地址，能够自动计算出对应的页目录项的index值,页目录项内容的值，页表项的index值，页表项内容的值。即\(pde\_idx, pde\_ctx, pte\_idx, pte\_cxt\)

请用如下值来验证你写的程序的正确性：

```
va 0xc2265b1f, pa 0x0d8f1b1f
va 0xcc386bbc, pa 0x0414cbbc
va 0xc7ed4d57, pa 0x07311d57
va 0xca6cecc0, pa 0x0c9e9cc0
va 0xc18072e8, pa 0x007412e8
va 0xcd5f4b3a, pa 0x06ec9b3a
va 0xcc324c99, pa 0x0008ac99
va 0xc7204e52, pa 0x0b8b6e52
va 0xc3a90293, pa 0x0f1fd293
va 0xce6c3f32, pa 0x007d4f32
```

参考的输出格式为：

```
va 0xcd82c07c, pa 0x0c20907c, pde_idx 0x00000336, pde_ctx  0x00037003, pte_idx 0x0000002c, pte_ctx  0x0000c20b
```
>> 注意：上述参考输出只是表示了正确的格式，其数值并不正确。
---
(5) 尝试在内存为256字节的OP-CPU机器上，设计一个支持自映射的内存空间部局。说明其页表起始逻辑地址、一级和二级虚拟地址计算公式。

## 开放思考题

---

（1）请简要分析Intel的x64 64bit体系结构下的分页机制是如何实现的


（2）Intel8086不支持页机制，但有hacker设计过包含未做任何改动的8086CPU的分页系统。猜想一下，hacker是如何做到这一点的？提示：想想MMU的逻辑位置

## 页表自映射机制思考题

1. (easy) 自映射的目的是什么？它相比线性映射的好处、不足是什么？
* 自映射的目的是为了节省4KB的空间。好处是节省空间，不足是页表的结构固定，对于X86，它的32位、两级页表已经定死了，不能作灵活调整。

2. (easy) Linux和Windows分别采用哪种映射机制（线性映射or自映射）？它们的选择背后有什么原因吗？
* Windows选择自映射，原因是其模块化做得很好，每段空间的作用有很好的规定，自映射便于一致性的管理。
* Linux选择线性映射，是因为Linux内核空间只提供了映射和分配规则，没有规定各个空间的用途，故不选择自映射。

以下为optional：

1. (midd) 在x86-32中，假设页目录的第R个页表项是自映射的，那么：

   1. 可以通过哪个虚拟地址访问到页目录？
   2. 控制虚地址`vaddr`所在页的页表项地址是多少？（假设在`vaddr`发生了缺页异常，根据你给出的答案，就可以快速地修改相应的页表项）
   3. 当你试图访问上述页表项时，又发生了缺页异常。这可能是什么原因？

2. (midd) 如何合理设置自映射页表，使得用户程序可以看到自己的页表，但不能修改或是看到内核部分？（参考ucore lab中的`sys_pgdir`）

3. (hard) 修改自映射页表：现在有两个自映射页表A和B，其中A是活动的（CR3指向pgdir），B是非活动的。如何在不切换页表（不修改CR3）的条件下，通过自映射区域修改页表B？

4. (challenge) RISCV中的自映射机制：

   如果照搬x86的方式，在RISCV中配置自映射页表，那么会在访问页目录时触发PageFault。原因是，根据规范，RISCV中的页表项不能被同时解读为【指向页表】和【指向数据页】（前者的flags是V，后者是VRW）。如果设置为V，那么在解读最后一级时会触发异常，如果设置为VRW，则会在第一级被认为是一个大数据页。而在x86中，同一个页表项在最后一级被解读为指向数据页，在中间级则被解读为指向页表，因此方便实现自映射。

   请你在x86自映射的基础上进行一些修改，为RISCV32设计一种自映射的实现方案。并描述一次修改页表项的完整过程。

参考阅读：[Advanced Paging](https://os.phil-opp.com/advanced-paging/)

## v9-cpu相关

\[challenge\]在v9-cpu上，设定物理内存为64MB。在os2.c和os4.c的基础上实现页机制管理，内核空间的映射关系： kernel\_virt\_addr=0xc00000000+phy\_addr，内核空间大小为64MB，虚拟空间范围为0xc0000000--x0xc4000000, 物理空间范围为0x00000000--x0x04000000；用户空间的映射关系：user\_virt\_addr=0x40000000+usr\_phy\_addr，用户空间可用大小为1MB，虚拟空间范围为0x40000000--0x40100000，物理空间范围为0x02000000--x0x02100000。可参考v9-cpu git repo的testing分支中的os.c和mem.h。修改代码为[os5.c](https://github.com/chyyuu/v9-cpu/blob/master/root/usr/os/os5.c)

* \(1\)在内核态可正确访问这两个空间
* \(2\)在用户态可正确访问这两个空间


在使能页机制的前一条指令和后一条指令的内存地址的访问会出现什么不同/变化？


如果在建立页表过程中，使能页机制前，如果不加上`pg_dir[0]=....`，为何v9-cpu模拟器会出现"kernel stack fault"的fatal error并推出？



* 请比较一下os在v9-cpu和x86上建立页表过程的不同之处，需要考虑硬件设计上的差异
* [x]