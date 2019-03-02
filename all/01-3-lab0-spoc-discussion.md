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
  - **进程切换需要硬件支持时钟中断，进一步需要Count和Compare寄存器的协作；**
  - **虚存需要用到地址映射，硬件需要提供MMU；**
  - **文件系统需要硬件提供稳定的存储介质。**
- 你理解的x86的实模式和保护模式有什么区别？
  - **实模式：不进行地址转换，地址之间一视同仁，可能会出现危险的情况；**
  - **保护模式：进行地址转换，通过虚拟地址来对系统程序进行保护。**
- 物理地址、线性地址、逻辑地址的含义分别是什么？
  - **逻辑地址：程序员所关心的地址。由程序产生、和段相关的偏移地址。它是相对于当前进程数据段的地址。**
  - **线性地址：逻辑地址变换到物理地址的中间层地址。逻辑地址+段的基地址=线性地址。**
  - **物理地址：变换的最终结果，在总线上的寻址信号。**
- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？
  - **M-mode：最高权限。不受权限地访问整个机器；**
  - **U-mode：用户态，权限最低。对系统的其他部分进行保护，防止其被用户程序破坏；**
  - **S-mode：管理员权限。比U-mode权限高，可以对操作系统进行操作；**
  - **H-mode：Hypervisor，比U-mode权限高，为了支持虚拟机监视器。**
  - **低权限在访问高权限资源时，可能只能只读，或是会被忽略。**
- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）
- 对于如下的代码段，请说明":"后面的数字是什么含义
```
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

-  **‘:’后的数字代表的是每一个‘变量’在结构体中所占的位数，例如，gd_off_15_0占了gatedesc结构的前16位，而gd_ss占了随后的16位…，而gd_off_31_16占了最后的16位。**

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

- **intr是一个64位的数（从gatedesc位数总和可知），其十六进制表示为0x00020003。**

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

   ```assembly
   .text
   .globl switch_to
   switch_to:                      # switch_to(from, to)
   
       # save from's registers 将寄存器的值保存入栈
       movl 4(%esp), %eax          # eax points to from 将栈顶的位置保存在寄存器eax中，预留空间保存各个寄存器的值
       popl 0(%eax)                # save eip !popl 将eip寄存器即将读的地址放到指定位置
       movl %esp, 4(%eax)			# 顺序将各个寄存器的值保存入栈
       movl %ebx, 8(%eax)
       movl %ecx, 12(%eax)
       movl %edx, 16(%eax)
       movl %esi, 20(%eax)
       movl %edi, 24(%eax)
       movl %ebp, 28(%eax)
   
       # restore to's registers 将栈中的值恢复到寄存器中
       movl 4(%esp), %eax          # not 8(%esp): popped return address already 将栈顶寄存器的值恢复到eax寄存中
                                   # eax now points to to
       movl 28(%eax), %ebp			# 顺序恢复各个寄存器的值
       movl 24(%eax), %edi
       movl 20(%eax), %esi
       movl 16(%eax), %edx
       movl 12(%eax), %ecx
       movl 8(%eax), %ebx
       movl 4(%eax), %esp
   
       pushl 0(%eax)               # push eip 将eip寄存器入栈
   
       ret							# 返回
   ```

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

```c
// 第一个宏定义了执行path下的名为name的进程，后面是其所带的参数
#define __KERNEL_EXECVE(name, path, ...) ({                         \
const char *argv[] = {path, ##__VA_ARGS__, NULL};       \
                     cprintf("kernel_execve: pid = %d, name = \"%s\".\n",    \
                             current->pid, name);                            \
                     kernel_execve(name, argv);                              \
})

// 而接下来四个宏则是对第一个宏的调用，分别是规定了在不同的情况下，应该如何传入path和name等参数

#define KERNEL_EXECVE(x, ...)                   __KERNEL_EXECVE(#x, #x, ##__VA_ARGS__)

#define KERNEL_EXECVE2(x, ...)                  KERNEL_EXECVE(x, ##__VA_ARGS__)

#define __KERNEL_EXECVE3(x, s, ...)             KERNEL_EXECVE(x, #s, ##__VA_ARGS__)

#define KERNEL_EXECVE3(x, s, ...)               __KERNEL_EXECVE3(x, s, ##__VA_ARGS__)


// user_main - kernel thread used to exec a user program
// 这个是用户的主进程，可以看到，在不同的宏定义的情况下，TEST，TESTSCRIPT是否开启的情况下，调用的是三类不同的执行宏，而最终会回到__KERNEL_EXECVE这个宏上。这样做有便于调试、而且在增删时易于调错。
static int
user_main(void *arg) {
#ifdef TEST
#ifdef TESTSCRIPT
    KERNEL_EXECVE3(TEST, TESTSCRIPT);
#else
    KERNEL_EXECVE2(TEST);
#endif
#else
    KERNEL_EXECVE(sh);
#endif
    panic("user_main execve failed.\n");
}
```

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
