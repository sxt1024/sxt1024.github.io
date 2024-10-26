---
title: "1.7_BIOS中断"
date: 2018-10-14
timezone: UTC+8
tags: ["操作系统"]
categories: ["操作系统"]
---



## BIOS中断

### BIOS中断简介

计算机刚启动时，进入实模式下，此时操作系统跟硬件（例如键盘鼠标显卡等）交互通过BIOS进行的。通过调用中BIOS中断的方式来访问硬件设备。

BIOS中断就不详细介绍了。

### BIOS中断大全

查询相应的中断API可以根据BIOS中断大全：[BIOS中断大全](https://www.cnblogs.com/onroad/archive/2009/07/13/1522662.html))



## 指令

```
int  中断号
```



### BIOS的中断向量表

中断向量表位置

中断向量表位于BIOS的 0x0000 - 0x03FF 地方，大小为 1k。

| 中断号 |                | 说明: int 中断号 |
| ------ | ------------------ | ---- |
| 0x00   |DIVIDE ERROR  |       |
| 0x01   |SINGLE STEP  |       |
| 0x02   |NON-MASKABLE INTERRUPT   |       |
| 0x03   |BREAKPOINT  |       |
| 0x04   |INT0 DETECTED OVERFLOW |       |
| 0x05   |BOUND RANGE EXCEED  |       |
| 0x06   |INVALID OPCODE |       |
| 0x07   |PROCESSOR EXTENSION NOT AVAILABLE |       |
| 0x08   |IRQ0  |       |
| 0x09   |IRQ1 |       |
| 0x0a   |IRQ2  |       |
| 0x0b   |IRQ3  |       |
| 0x0c   |IRQ4  |       |
| 0x0d   |IRQ5  |       |
| 0x0e   |IRQ6  |       |
| 0x0e   |IRQ7  |       |
| 0x10   | VIDEO |     显示  |
| 0x11   | GET EQUIPMENT LIST |   设备列表    |
| 0x12   |  GET MEMORY SIZE |   内存大小   |
| 0x13   | DISK |     磁盘  |
| 0x14   | SERIAL |    串行口服务  |
| 0x15  | SYSTEM | 系统 |
| 0x16   | KEYBOARD | 键盘 |
| 0x17   | PRINTER | 打印机 |
| 0x18   |  CASETTE BASIC |     |
| 0x19   | BOOTSTRAP LOADER | 时钟 |
| 0x1a   | TIME ||
| 0x1b   | KEYBOARD - CONTROL-BREAK HANDLER ||
| 0x1c   | TIME - SYSTEM TIMER TICK  ||
| 0x1d   | SYSTEM DATA - VIDEO PARAMETER TABLES  |   |
| 0x1e   | SYSTEM DATA - DISKETTE PARAMETERS |   |
| 0x1f   |SYSTEM DATA - 8x8 GRAPHICS FONT |   |
| 0x70   |IRQ8 - CMOS REAL-TIME CLOCK |   |
| 0x71   | IRQ9 - REDIRECTED TO INT 0A BY BIOS|   |
| 0x72   | IRQ10 - RESERVED |   |
| 0x73   |IRQ11 - RESERVED |   |
| 0x74   |IRQ12 - POINTING DEVICE|   |
| 0x75   | IRQ13 - MATH COPROCESSOR EXCEPTION |   |
| 0x76   | IRQ14 - HARD DISK CONTROLLER OPERATION COMPLETE|   |
| 0x77   |IRQ15 - SECONDARY IDE CONTROLLER OPERATION  |   |



