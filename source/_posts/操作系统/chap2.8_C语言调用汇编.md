---
title: chap2.8_C语言调用汇编
date: 2019-02-07T11:27:28
lastmod: 2019-02-07
timezone: UTC+8
tags:
  - 操作系统
categories:
  - 操作系统
draft: false
---



# C语言调用汇编



**C语言头文件**

include/io.h

```assembly
#ifndef __LIB_IO_H
#define __LIB_IO_H

void io_hlt();

#endif
```





**汇编实现**

asm/io.asm

```assembly
;io.asm

[bits 32]
global io_hlt


[section .text]
io_hlt:       ; void io_hlt
    hlt
    ret
        
```



**代码调用**

main.c

```c
#include "include/io.h"

typedef unsigned char int8;

int _start(){
    int8 *pvga = (int8 *)0xa0000;	//填充到显示内存的初始地址	
    for(int i = 0;i <= 0xffff;i++){	
		  *(pvga + i) = i & 0x0F;     //显卡内存存填充颜色值	
    }
    io_hlt();
}
```





