---
title: chap3.3_外设端口读写
date: 2019-02-15T11:27:28
lastmod: 2019-02-15
timezone: UTC+8
tags:
  - 操作系统
categories:
  - 操作系统
draft: false
---

## 外设端口

设备通常会提供一组寄存器来控制设备、读写设备和获取设备状态，即**控制寄存器、数据寄存器和状态寄存器。**

这些寄器可能位于I/O空间中，也可能位于内存空间中。当位于I/O空间时，通常被称为I/O端口；当位于内存空间时，对应的内存空间被称为I/O内存。

**每个外设都是通过读写其寄存器来控制的。外设寄存器也称为I/O端口**，通常包括：控制寄存器、状态寄存器和数据寄存器三大类。


## ## 外设I/O端口读写

X86的cpu可以直接读写以下三个地方的数据，读写三个地方的指令都是不同的，他们的空间也是分开的，这点要注意。


I/O端口是指设备控制器中**可被CPU直接访问的寄存器**,主要有以下三类寄存器。

- **数据寄存器**:用于缓存从设备送来的输入数据,或从CPU送来的输出数据。
- **状态寄存器**:保存设备的执行结果或状态信息,以供CPU读取。
- **控制寄存器**:由CPU写入,以便启动命令或更改设备模式。

I/O 端口要想能够被CPU访问,就要对各个端口进行编址,每个端口对应一个端口地址。而对 I/O端口的编址方式有与存储器独立编址和统一编址两种.

将外设的寄存器看成一个独立的地址空间，所以访问内存的指令不能用来访问这些寄存器，而要为对外设寄存器的读／写设置专用指令，如IN和OUT指令。这就是所谓的**“ I/O端口”**方式


## 端口读写指令

CPU对外设通过端口读写指令来进行读写数据。

端口读写指令

- in cpu寄存器，端口地址 ： 从端口中读取数据到CPU寄存器

- out 端口地址 , cpu寄存器:   写入CPU寄存器的数据到端口中

> - 此处的CPU寄存器，为一个字节（8位），字（16位）或双字（32位），根据CPU寄存器的大小，从端口地址处读取不同的字节数。


include/stdint.h

```assembly
#ifndef __LIB_STDINT_H
#define __LIB_STDINT_H

typedef signed char int8;
typedef signed short int16;
typedef signed int int32;
typedef signed long long int64;

typedef unsigned char uint8;
typedef unsigned short uint16;
typedef unsigned int uint32;
typedef unsigned long long uint64;

#endif
```
类型别名


## c语言调用汇编

定义外设端口读写方法，C语言调用汇编

1）首先定义一个C语言头文件

2）使用汇编实现C语言头文件方法

3）C语言和汇编编译链接在一起


include/io.h

```assembly
#ifndef __LIB_IO_H
#define __LIB_IO_H

#include "stdint.h"

void io_cli();

void io_sti();

void io_hlt();

uint8 io_in8(uint16 port);

void io_out8(uint16 port, uint8 data);

uint16 io_in16(uint16 port);

void io_out16(uint16 port, uint16 data);

uint32 io_in32(uint16 port);

void io_out32(uint16 port, uint32 data);

#endif
```




asm/io.asm

```assembly
;io.asm
[bits 32]

global io_cli, io_sti, io_hlt
global io_in8, io_out8
global io_in16, io_out16
global io_in32, io_out32

[section .text]

io_cli:     ;void io_cli
    cli
    ret

io_sti:     ;void io_sti
    sti
    ret

io_hlt:     ;void io_hlt
    hlt
    ret

io_in8:       ; uint8 io_in8(uint16 port)
    mov dx,[esp+4]
    mov al,0
    in  al,dx
    ret

io_out8:      ; void io_out8(uint16 port,uint8 data)
    mov dx,[esp+4]
    mov al,[esp+8]
    out dx, al
    ret

io_in16:      ; uint16 io_in16(uint16 port)
    mov dx,[esp+4]
    mov ax,0
    in  ax,dx
    ret

io_out16:     ; void io_out16(uint16 port,uint16 data)
    mov dx,[esp+4]
    mov ax,[esp+8]
    out dx, ax
    ret

io_in32:      ; uint32 io_in32(uint16 port)
    mov dx,[esp+4]
    mov eax,0
    in  eax,dx
    ret

io_out32:     ; void io_out32(uint16 port,uint32 data)
    mov dx,[esp+4]
    mov eax,[esp+8]
    out dx,eax
    ret



```



