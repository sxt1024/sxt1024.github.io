---
title: chap2.4_GDT全局描述符表
date: 2019-01-06T11:27:28
lastmod: 2019-01-06
timezone: UTC+8
tags:
  - 操作系统
categories:
  - 操作系统
draft: false
---



## GDT全局描述符表

### 什么是GDT全局描述符表

GDT全称为`Global Descriptor Table`,全局描述符表。

保护模式的寻址方式不在使用寄存器分段的方式直接寻址方式了。而采用的是使用GDT（全局分段描述表）来寻址。从而使用更多的内存地址。


创建GDT全局描述符表使用到一个48位的寄存器：GDTR寄存器。

1）首先，在内存中划分一些内存段，并且每个内存段赋予一个索引。

2）然后，使用lgdt指令，设置GDT的索引和表信息的内存地址到GDTR寄存器。

3）进入保护模式，指令跳转，从实模式分段方式寻址切换到使用GDT分段方式寻址。

4)  GDT可以被放在内存的任何地方,只要提供内存地址给GDTR寄存器就可以了。



### GDT格式

**GDT全局描述符表**

- 表基地址，表基地址位GDT段表在内存的地址，GDT段表是一个列表，存储了多个 GDT段描述符。
- 表界限：GDT段表的空间信息，以字节为单位。



![2_1_1.png](2_1_1.png)




> GDT全局描述符表 = GDT段表基地址 | 16位表界限
>
> GDT段表 =　 GDT段描述符　｜　 GDT段描述符　｜　GDT段描述符 ...
>
> 表界限 = GDT字节数 - 1  （表示 0 - 0x...）

  

**GDT段描述符**

![images/003.png](images/002.png)

GDT段描述符，用来描述在GDT方式在内存中分配的一个段信息，总共8字节64位。

GDT段描述符结构

为了兼容以前的CPU，GDT段描述符的信息被分割成几个部分，格式如下：

> GDT段描述符　＝　
>
> 段基址 （8位）| 段描述符（4位） |  段界限（4位） | 段描述符（8位）  段基址 （8位）
>
> 段基址 （16位） | 段界限（16位）



段描述符定义

- 段基址：规定段的起始地址,长度32位.
- 段界限：规定段的大小,长度20位。段界限可以是以`4KB`或者`1B`为单元大小
- 段属性：确定段的各种性质.长度(12位)



段属性：

- G 粒度位: 段界限的单位大小，G=1表示段界限以`4KB`为单元单位，G=0表示段界限以`1B`为单元单位
- D/B 表示操作数为多少位， 0表示16位操作数，1表示32位操作数
- L ： 0 表示非64位代码段，1表示64位代码段
- AVL ：可用字段，暂时没什么用
- P   段存在位:通常为1，表示段存在于内存中，0则此段为非法的，不能被用来实现地址转换
- DPL 特权级(2位): 用来实现保护机制
- S  为0表示系统段，为1表示非系统段
- type 类型(4位): 用于区别不同类型的描述符。内存段或者门的子类型



type值

| Type位 | 说明                     | 取值         |
| ------ | ------------------------ | ------------ |
| **代码段时** |  |  |
| X：3位 | 代码段值为1              | 0：为数据段<br>1：为代码段   |
| C：2位 | 访问位                   | 0：为普通段<br>1：为一致码段 |
| R：1位 | 是否可读                   | 0：只执行<br>1：可读           |
| A：0位 | 访问位. 该段是否被访问过 | 0 ：未访问<br>1：已访问 |
| **数据段时** |  |  |
|   X：3位 | 数据段值为1           |   0：为数据段 <br>1：为代码段           |
| E：2位 | 扩展方向                |    0：向高位扩展 <br>1：向低位扩展        |
| W：1位 | 是否可写                  |   0：只读<br>1：可写         |
| A：0位 | 访问位                   |  0： 未访问<br>1：已访问<td>          |


段界限:

段界限边界值 =  （描述符的段界限值 + 1） × （段界限颗粒读：4Kb 或者 1b） -1

反之：
  描述符的段界限值 = （段界限边界值 + 1） /（段界限颗粒读：4Kb 或者 1b）

例如：

  16MB的段界限值 =  0x1000000  /（段界限颗粒读：4Kb 或者 1b - 1）= 0x0fff



### 段选择子

段选择子包括三部分：描述符索引（index）、TI、请求特权级（RPL） 



**GDTR寄存器**

在内存中建立完成GDT信息后，CPU会将GDT的内存地址 和 段界限 数据加载入GDTR寄存器

GDTR寄存器数据(48位)：

> GDTR定义数据(48位) ＝ GDT全局描述符表的大小(16位)  +  GDT全局描述符表的地址(32位)

**lgdt指令**

> lgdt GDTR定义数据

其中GDT全局描述符表数据格式如下

> GDT全局描述符表 =  GDT段描述符(64位) |  GDT段描述符(64位) | GDT段描述符(64位) ...
>
> GDT段描述符 =  段基址 （8位）| 段描述符（4位） |  段界限（4位） | 段描述符（8位） | 段基址 （8位） | 段基址 （16位） | 段界限（16位）

其中，第一个GDT段的数据为空。



、

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

![2_1_1.png](2_1_1.png)



