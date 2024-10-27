---
title: chap1.4_BIOS中断
date: 2018-10-14
timezone: UTC+8
tags:
  - 操作系统
categories: 操作系统
---


### 需要的工具

> qemu: [qemu](https://www.qemu.org/)
>
   qemu的windows版本：[QEMU for Windows – Installers (64 bit)](https://qemu.weilnetz.de/w64/2024/)

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



## 清屏

[TOC]






### BIOS中断清屏

清屏是通过BIOS中断，来滚动屏幕，达到清屏的效果。

**1. BIOS中断滚屏**

中断 int 10h，AH = 06H / 07H

| 寄存器 | 说明                             | 值                            |
| ------ | -------------------------------- | ----------------------------- |
| AH     | 功能编码                         | 向上滚屏：06H，向下滚屏 : 07H |
| BH     | 空白区域的缺省属性               |                               |
| AL     | 滚动行数                         | 0：清窗口                     |
| CH、CL | 滚动区域左上角位置：Y坐标，X坐标 |                               |
| DH、DL | 滚动区域右下角位置：Y坐标，X坐标 |                               |



例如：使用蓝底白字清屏

```assembly
;---------------------------
;清除屏幕	 
mov ah,0x06								
mov al,0
mov cx,0   
mov df,0xffff  
mov bh,0x17				;属性为蓝底白字
int 0x10
```



**2. BIOS中断设置光标位置：**

中断 int 10h

功能描述：用文本坐标下设置光标位置

入口参数：

| 寄存器 | 说明                  | 值                |
| ------ | --------------------- | ----------------- |
| AH     | 功能编码              | 设置光标位置：02H |
| BH     | 显示页码              |                   |
| DH，DL | 行，列 (Y坐标，X坐标) |                   |

例如：设置光标到第一行第一列

```assembly
    ;---------------------------			
    ;光标位置初始化
    mov ah,0x02				
    mov bh,0
    mov dx,0
    int 0x10
```



### 实现

**1. 代码**




boot.asm内容如下

```assembly
; ASTRAOS BOOT
[bits 16]

org     0x7c00         ; 指明程序的偏移的基地址

jmp     Entry          ; 跳转到程序入口
db 		0x90
db      "ASTRAOS"

; ----------------------------
; 程序入口
; ----------------------------
Entry:

    ; ---------------------------
    ; 清除屏幕
    ; ----------------------------
    mov ah,0x06
    mov bh,0x07
    mov al,0
    mov cx,0
    mov dx,0xffff
    mov bh,0x17        ; 属性为蓝底白字
    int 0x10

    ; ---------------------------
    ; 光标位置初始化
    ; ----------------------------
    mov ah,0x02
    mov bh,0
    mov dx,0
    int 0x10

Fin:
    hlt
    jmp Fin            ; 进入死循环，不再往下执行。

Fill_Sector:
    resb    510-($-$$)      ; 处理当前行$至结束(1FE)的填充
    db      0x55, 0xaa
```





### 创建build.sh脚本

```shell

#!/bin/bash

NASM=nasm
mkdir out
$NASM -f bin -o out/boot.bin boot/boot.asm
dd if=/dev/zero of=out/astraos.img bs=512 count=2880
dd if=out/boot.bin  of=out/astraos.img bs=512 count=1  conv=notrunc

```

编译成astraos.ima镜像文件。


**运行**

创建run.sh脚本

```
#!/bin/bash

QEMU=qemu-system-x86_64
$QEMU -m 128 -rtc base=localtime -fda build/astraos.img
```

window下创建run.bat

```
set qume="C:/Program Files/qemu/qemu-system-x86_64w.exe"
%qume% -m 128 -rtc base=localtime -fda build/astraos.img
```

执行run.sh脚本

结果如图

![images/1.8_1.png](1_4_1.png)


