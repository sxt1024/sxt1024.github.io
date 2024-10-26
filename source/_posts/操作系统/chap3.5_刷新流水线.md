---
title: "3.5_刷新流水线"
date: 2019-01-08T11:27:28
lastmod: 2019-01-08
timezone: UTC+8
tags: ["操作系统"]
categories: ["操作系统"]
draft: false
---



## 刷新流水线

### 为什么需要刷新流水线





进入保护模式下，需要尽快刷新CS，SS等段寄存器。



### 立即跳转到32位模式，刷新流水线



进入保护模式后，需要马上跳转并刷新流水

定义代码段和数据段的选择子常量

>CODE选择子： selector_code =  0x1<<3  + 000B
>
>DATA 选择子：selector_data =  0x2<<3  + 000B
>
>VGA 选择子：  selector_vga =  0x3 <<3  + 000B



boot.inc文件定义选择子

```
;-------------    选择子 --------------------------
SELECTOR_CODE equ 0x1<<3
SELECTOR_DATA equ 0x2<<3
SELECTOR_VGA equ 0x3<<3
```

​            



loader.asm文件 跳转并刷新流水，由16位模式进入32位代码模式：

```assembly
;------------------------    
;刷新流水线，进入32位模式
[bits 16]

     jmp	 dword   SELECTOR_CODE:FlushPipeline       


[bits 32]
;------------------    
;刷新流水线
FlushPipeline:

    mov		ax,SELECTOR_DATA			;  可读写的32bit
    mov		ds,ax
    mov		es,ax
    mov		fs,ax
    mov     ss,ax
    mov     ax,SELECTOR_VGA
    mov     gs,ax

```





