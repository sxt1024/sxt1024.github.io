---
title: "3.4_GDT临时分段"
date: 2019-01-07T11:27:28
lastmod: 2019-01-07
timezone: UTC+8
tags: ["操作系统"]
categories: ["操作系统"]
draft: false
---





## GDT临时分段

### GDT临时段说明

现在已经进入了保护模式, 目前的改变

- 可以访问1M以上的内存了
- 可以使用32位的指令

问题：

由于以前的是实式下段寄存器寻址方式无法使用了，我们必须切换到使用GDT段方式来寻址

首要的任务就是先建立一个临时的GDT段，以便我们接下来的指令操作

目前准备建立3个段，如下：

> Base, Limit, Attr
>
> 代码段：0x00000000, 0xfffff,  1100_1001_1010B =  db 0x0000ffff, 0x00cf9a00 
>
> 数据段：0x00000000, 0xfffff,  1100_1001_0010B = db  0x0000ffff, 0x00cf9200
>
> vga显卡内存数据段：x000b8000, 0x07fff, 1100_1001_0010B



### GDT解析宏

首先创建一个nasm宏，可以进行GDT解析

参数为GDT的Base, Limit, Attr，也就是段基址(32位),段界限(20位),段描述符(12位)，然后生成GDT在内存中的数据

```assembly
;---------------------------------------------------------
; 描述符
; usage: Gdt_Descriptor Base, Limit, Attr ：  段基址(32位),段界限(20位),段描述符(12位)
;        Base:  dd
;        Limit: dd (low 20 bits available)
;        Attr:  dw (lower 4 bits of higher byte are always 0)
;---------------------------------------------------------
%macro Gdt_Descriptor  3
dw %2 & 0xFFFF
dw %1 & 0xFFFF
db (%1 >> 16) & 0xFF
db %3 & 0xFF
db ((%3 >> 4 ) & 0xF0 ) | ((%2 >> 16) & 0x0F )
db (%1 >> 24) & 0xFF
%endmacro
```



### GDT 描述符属性定义

```assembly
;--------------   gdt描述符属性  -------------
DESC_G_4K   equ	  1000_0000_0000b   
DESC_D_32   equ	  0100_0000_0000b
DESC_L	    equ	  0000_0000_0000b	;  64位代码标记，此处标记为0便可。
DESC_AVL    equ	  0000_0000_0000b	;  cpu不用此位，暂置为0  
DESC_P	    equ	  0000_1000_0000b
DESC_DPL_0  equ	        000_0000b
DESC_DPL_1  equ	        010_0000b
DESC_DPL_2  equ	        100_0000b
DESC_DPL_3  equ	        110_0000b
DESC_S_CODE equ	          1_0000b
DESC_S_DATA equ	          1_0000b
DESC_S_SYS  equ		      0_0000b
DESC_TYPE_CODE  equ	        1010b	;x=1可执行代码段,c=0普通,r=1可读,a=0已访问位a清0  
DESC_TYPE_DATA  equ	        0010b	;x=0数据段,e=0向高位扩展,w=1可写,a=0已访问位a清0.

;--------------   选择子属性  ---------------
RPL0    equ   00b
RPL1    equ   01b
RPL2    equ   10b
RPL3    equ   11b
TI_GDT	equ   000b
TI_LDT	equ   100b

    
DESC_CODE equ  DESC_G_4K + DESC_D_32 + DESC_L + DESC_AVL + DESC_P + DESC_DPL0 + DESC_S_CODE + DESC_TYPE_CODE 
DESC_DATA equ  DESC_G_4K + DESC_D_32 + DESC_L + DESC_AVL + DESC_P + DESC_DPL0 + DESC_S_DATA + DESC_TYPE_DATA 
```



### 定义GDT全局描述符表

```assembly
;---------------------------------
;定义GDT全局描述符表
;code: 0x0000ffff, 0x00cf9a00   
;data: 0x0000ffff, 0x00cf9200
;vga:
Gdt_Addr:
    dw		8*4-1						;指定段上限为4(GDT全局描述符表的大小)
    dd		gdt_table_addr				;GDT全局描述符表的地址
Gdt_Table_Addr:
    		    Gdt_Descriptor 0,0,0
Label_Sel_Code:	Gdt_Descriptor 0x00000000, 0xfffff, DESC_CODE  ;可以执行的段
Label_Sel_Data:	Gdt_Descriptor 0x00000000, 0xfffff, DESC_DATA  ;可以读写的段
Label_Sel_VGA:	Gdt_Descriptor 0x000b8000, 0x07fff, DESC_DATA  ;vga段
    dw		0	

;--------------------------------
;选择子
Selector_Code  equ	 Label_Sel_Code - Gdt_Table_Addr
Selector_Data  equ	 Label_Sel_Data - Gdt_Table_Addr
Selector_VGA  equ	 Label_Sel_VGA - Gdt_Table_Addr
```

加载gdt

```assembly
;---------------------------
;加载GDT
lgdt [Gdt_Addr]
```

### 代码 

创建常量头文件

创建 boot.inc文件。用来配置常量

```assembly
; boot.inc
;---------------------------------------------------------
; 描述符
; usage: Gdt_Descriptor Base, Limit, Attr ：  段基址(32位),段界限(20位),段描述符(12位)
;        Base:  dd
;        Limit: dd (low 20 bits available)
;        Attr:  dw (lower 4 bits of higher byte are always 0)
;---------------------------------------------------------
%macro Gdt_Descriptor  3
dw %2 & 0xFFFF
dw %1 & 0xFFFF
db (%1 >> 16) & 0xFF
db %3 & 0xFF
db ((%3 >> 4 ) & 0xF0 ) | ((%2 >> 16) & 0x0F )
db (%1 >> 24) & 0xFF
%endmacro



;--------------   gdt描述符属性  -------------
DESC_G_4K   equ	  1000_0000_0000b   
DESC_D_32   equ	  0100_0000_0000b
DESC_L	    equ	  0000_0000_0000b	;  64位代码标记，此处标记为0便可。
DESC_AVL    equ	  0000_0000_0000b	;  cpu不用此位，暂置为0  
DESC_P	    equ	  0000_1000_0000b
DESC_DPL_0  equ	        000_0000b
DESC_DPL_1  equ	        010_0000b
DESC_DPL_2  equ	        100_0000b
DESC_DPL_3  equ	        110_0000b
DESC_S_CODE equ	          1_0000b
DESC_S_DATA equ	          1_0000b
DESC_S_SYS  equ		      0_0000b
DESC_TYPE_CODE  equ	        1010b	;x=1可执行代码段,c=0普通,r=1可读,a=0已访问位a清0  
DESC_TYPE_DATA  equ	        0010b	;x=0数据段,e=0向高位扩展,w=1可写,a=0已访问位a清0.


;--------------   选择子属性  ---------------
RPL0    equ   00b
RPL1    equ   01b
RPL2    equ   10b
RPL3    equ   11b
TI_GDT	equ   000b
TI_LDT	equ   100b

DESC_CODE equ  DESC_G_4K + DESC_D_32 + DESC_L + DESC_AVL + DESC_P + DESC_DPL_0 + DESC_S_CODE + DESC_TYPE_CODE 
DESC_DATA equ  DESC_G_4K + DESC_D_32 + DESC_L + DESC_AVL + DESC_P + DESC_DPL_0 + DESC_S_DATA + DESC_TYPE_DATA 

;----------- loader const ------------------
LOADER_SECTOR_LBA  		equ 0x1		;第2个逻辑扇区开始
LOADER_SECTOR_NUM		equ 9		;读取9个扇区
LOADER_BASE_ADDR 		equ 0x9000  ;内存地址0x9200
LOADER1_BASE_ADDR       equ 0x9800  ;内存地址0x9200
;-------------------------------------------
```

loader.asm文件

```assembly
;GloxOS LOADER [0x9000]
;Tab=4
[bits 16]

%include "boot/boot.inc"

section loader vstart=LOADER_BASE_ADDR ;指明程序的偏移的基地址

jmp Entry;



;程序核心内容
Entry:
    
    ;----------------------
    ;禁止CPU级别的中断，进入保护模式时没有建立中断表
    ;----------------------
    cli	

    ;----------------------
    ;打开A20
    ;----------------------
    in 		al,0x92
    or 		al,0000_0010B       ;设置第1位为1
    out 	0x92,al
    
    ;----------------------
    ;加载GDT
    ;----------------------
    lgdt [Gdt_Addr]

	;----------------------
    ;进入保护模式
    ;----------------------
    mov 	eax,cr0
    or 		eax,0x1      ;设置第0位为1
    mov 	cr0,eax

;程序挂起
Fin:
    hlt 					;让CPU挂起，等待指令。
    jmp Fin
    
;---------------------------------
;定义GDT全局描述符表
;code: 0x0000ffff, 0x00cf9a00   
;data: 0x0000ffff, 0x00cf9200
;vga:
;---------------------------------
Gdt_Addr:
    dw		8*4-1						;指定段上限为4(GDT全局描述符表的大小)
    dd		Gdt_Table_Addr				;GDT全局描述符表的地址
Gdt_Table_Addr:
    Gdt_Descriptor 0,0,0
    Gdt_Descriptor 0x00000000, 0xfffff, DESC_CODE  ;可以执行的段
    Gdt_Descriptor 0x00000000, 0xfffff, DESC_DATA  ;可以读写的段
    Gdt_Descriptor 0x000b8000, 0x07fff, DESC_DATA  ;vga段
    dw		0	    

times	512-($-$$) db  0 ; 处理当前行$至结束(1FE)的填充
```



## 测试

使用bochs执行

打好断点后，执行并查看gtd描述符数据是否正确。

> info gdt

![images/001.png](images/001.png)



