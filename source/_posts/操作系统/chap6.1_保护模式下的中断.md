## 中断



### 中断分类



- 外部中断：由硬件发出，发出信号通知CPU

  - 可屏蔽中断：不需要马上处理，例如键盘事件
  - 不可屏蔽中断：必须及时相应，不然系统无法继续。例如内存读写错误等

- 内部中断：

  - 软中断：软件发起的中断
  - 异常：指令执行时出现的错误



### 异常分类

- Fault：故障
- Trap：陷阱
- Abort：终止







### 实模式中断                                                               

在实模式下，有BIOS进行中断控制。使用中断向量表（IVT），

在实模式下，中断向量表占据内存最低的1KB，共256个表项。

实模式下，中断的处理过程很单一。当中断被触发时，CPU保护现场，跳转中断处理程序，执行完毕后恢复现场，继续执行原程序。



### 保护模式下的中断

保护模式下，中断向量表可以在内存中自由浮动。就像GDT被GDTR指向一样，中断向量表被IDTR(Interrupt Descriptor Table Register，中断描述符表寄存器)指向。

在保护模式下，中断的处理过程呈现出了多样化。中断处理过程因IDT中的描述符而异，可以分为三类。

1） 如果IDT描述符是一个中断门描述符或者陷阱门描述符，则执行类似于实模式的中断跳转。但是中断门和陷阱门差别不大。

2） 如果IDT描述符是一个TSS描述符，那么中啊实打实的断会引发任务切换。这种机制使得多任务轮询调度成为可能。

保护模式下中断：

（1）发生中断操作

（2）保存指令信息

（3）根据IDTR寄存器信息，查询中断描述符表

（5）处理中断信息

（6）恢复指令信息   

> 保护模式下无法访问中断向量表,需要为IDTR重新分配中断向量号
>
> 因为前0-31位已经被占用, 从32位开始`  io_out8(PIC0_ICW2, 0x20);`





| Vector | Mne-monic | Description Type Error Code Source                           |
| ------ | --------- | ------------------------------------------------------------ |
| 0      | #DE       | Divide Error Fault No DIV and IDIV instructions.             |
| 1      | #DB       | Debug Exception Fault/ Trap No Instruction, data, and I/O breakpoints; single-step; and others. |
| 2      | —         | NMI Interrupt Interrupt No Nonmaskable external interrupt.   |
| 3      | #BP       | Breakpoint Trap No INT3 instruction.                         |
| 4      | #OF       | Overflow Trap No INTO instruction.                           |
| 5      | #BR       | BOUND Range Exceeded Fault No BOUND instruction.             |
| 6      | #UD       | Invalid Opcode (Undefined Opcode) Fault No UD instruction or reserved opcode. |
| 7      | #NM       | Device Not Available (No MathCoprocessor) Fault No Floating-point or WAIT/FWAIT instruction. |
| 8      | #DF       | Double Fault Abort Yes(zero) Any instruction that can generate anexception, an NMI, or an INTR. |
| 9      |           | Coprocessor Segment Overrun (reserved) Fault No Floating-point instruction. |
| 10     | #TS       | Invalid TSS Fault Yes Task switch or TSS access.             |
| 11     | #NP       | Segment Not Present Fault Yes Loading segment registers or accessingsystem segments. |
| 12     | #SS       | Stack-Segment Fault Fault Yes Stack operations and SS register loads. |
| 13     | #GP       | General Protection Fault Yes Any memory reference and otherprotection checks. |
| 14     | #PF       | Page Fault Fault Yes Any memory reference.                   |
| 15     |           | reserved                                                     |
| 16     | #MF       | x87 FPU Floating-Point Error                                 |
| 17     | #AC       | Alignment Check Exception                                    |
| 18     | #MC       | Machine Check Exception                                      |
| 19     | #XM       | SIMD Floating-Point Exception                                |
| 20     | #VE       | Virtualization Exception                                     |
| 21     | #CP       | Contorl Protection Exception                                 |
| 22-31  |           | Intel 保留使用                                               |
| 32-255 |           | 用户自定义                                                   |







### 中断指令                      



中断返回：iret                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       