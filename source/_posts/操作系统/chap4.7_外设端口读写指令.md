---
title: "4.7_外设端口读写指令"
date: 2019-02-15T11:27:28
lastmod: 2019-02-15
timezone: UTC+8
tags: ["操作系统"]
categories: ["操作系统"]
draft: false
---





# 外设端口读写指令

cpu可以直接读写以下三个地方的数据，读写三个地方的指令都是不同的，他们的空间也是分开的，这点要注意。

- 内存

- 寄存器

- 端口



## 端口读写指令

CPU对外设通过端口读写指令来进行读写数据。

in cpu寄存器，端口地址 ： 从端口中读取数据到CPU寄存器

out 端口地址 , cpu寄存器:   写入CPU寄存器的数据到端口中

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
typedef unsigned int uint;
typedef unsigned long long uint64;

#endif
```


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



lib/io.asm

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



