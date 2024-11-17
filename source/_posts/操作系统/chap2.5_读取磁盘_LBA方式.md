---
title: chap2.5_读取磁盘_LBA方式
date: 2018-11-17T11:27:28
lastmod: 2018-11-17
timezone: UTC+8
tags:
  - 操作系统
categories:
  - 操作系统
draft: false
---



## BA简介


LBA方式访问使用了data寄存器，LBA寄存器（总共3个），device寄存器，command寄存器来完成的。

LBA28和LBA48方式

LBA28方式：使用28位来描述一个扇区地址，最大支持128GB的硬磁盘容量。

LBA48方式：使用48位来描述一个扇区地址，最大支持144,000,000 GB的硬磁盘容量
  

 **1. CHS方式:**


寄存器|端口|作用
--|---|---
data寄存器         |0x1F0|已经读取或写入的数据，大小为两个字节（16位数据)<br/>每次读取1个word,反复循环，直到读完所有数据
features寄存器     |0x1F1|0
sector count寄存器 |0x1F2|指定读取的扇区数
LBA low寄存器      |0x1F3|扇区号     
LBA mid寄存器      |0x1F4|柱面的低8位  
LBA high寄存器     |0x1F5|柱面的高8位
device寄存器       |0x1F6|第0-3位：磁头号<br/>第4位：主盘值为0，从盘值为1
第5位：值为1
第6位：读取方式,LBA模式为1，CHS模式为0
第7位:   值为1lba地址的前4位（占用device寄存器的低4位）<br>
command寄存器      |0x1F7|指定读取或写入的命令，返回磁盘状态 <br> 读取扇区:0x20  第4位为0表示读写完成，否则要一直循环等待<br> 写入扇区:0x30 <br> 磁盘识别:0xEC


 **LBA28方式：**



读取时参数：

寄存器|端口|作用
--|---|---
data寄存器         |0x1F0|已经读取的数据，大小为两个字节（16位数据)<br/>每次读取1个word,反复循环，直到读完所有数据
features寄存器     |0x1F1|读取时的错误信息
sector count寄存器 |0x1F2|指定读取的扇区数
LBA low寄存器      |0x1F3|lba地址的低8位（0~7位）     
LBA mid寄存器      |0x1F4|lba地址的中8位（0~7位）  
LBA high寄存器     |0x1F5|lba地址的高8位16~23位）
device寄存器       |0x1F6| 第0-3位： lba地址<br>第4位：主盘值为0，从盘值为1<br>第5位：值为1<br/>第6位：读取方式,LBA模式为1，CHS模式为0<br>第7位:   值为1 
command寄存器      |0x1F7|指定读取或写入的命令，返回磁盘状态 <br> 读取扇区:0x20 <br> 写入扇区:0x30 <br> 磁盘识别:0xEC

写入时参数

寄存器|端口|作用
--|---|---
data寄存器         |0x1F0|已经写入的数据，大小为两个字节（16位数据)<br/>每次读取1个word,反复循环，直到读完所有数据
features寄存器     |0x1F1| 写入时的额外参数                                             
sector count寄存器 |0x1F2|指定读写入的扇区数
LBA low寄存器      |0x1F3|lba地址的低8位（0~7位）     
LBA mid寄存器      |0x1F4|lba地址的中8位（0~7位）  
LBA high寄存器     |0x1F5|lba地址的高8位（16~23位）
device寄存器       |0x1F6|第0-3位： lba地址<br>第4位：主盘值为0，从盘值为1<br>第5位：值为1<br/>第6位：读取方式,LBA模式为1，CHS模式为0<br>第7位:   值为1
command寄存器      |0x1F7|指定读取或写入的命令，返回磁盘状态 <br> 读取扇区:0x20 <br> 写入扇区:0x30 <br> 磁盘识别:0xEC

 IDE通道1，读写0x1f0-0x1f7号端口

 IDE通道2，读写0x170-0x17f号端口



  **LBA48方式: **

 写两次0x1f1端口: 0

 写两次0x1f2端口: 第一次要读的扇区数的高8位,第二次低8位

 写0x1f3: LBA参数的24~31位

 写0x1f3: LBA参数的0~7位

 写0x1f4: LBA参数的32~39位

 写0x1f4: LBA参数的8~15位

 写0x1f5: LBA参数的40~47位

 写0x1f5: LBA参数的16~23位

 写0x1f6: 7~5位,010,第4位0表示主盘,1表示从盘,3~0位,0

 写0x1f7: 0x24为读, 0x34为写

  
### 磁盘寻址：LBA方式

LBA方式不考虑扇区的物理位置，而是根据逻辑位置来寻址的。

LBA方式不使用`柱面-磁头-扇区`方式定位，直接以从0开始，逐次递增的方式来定位逻辑扇区。

比如： 扇区0，扇区1，....   

**LAB寄存器**

LAB使用硬盘控制器存储扇区定位信息。每个寄存器8位大小。

- device寄存器: 

- LBA low寄存器

- LBA mid寄存器

- LBA high寄存器 


例如LAB28，使用一个28位来表示扇区位置。

 LAB28扇区地址 = device寄存器的低4位 + LBA high寄存器 + LBA mid寄存器 + LBA low寄存器

### 读取硬盘

1）sector count寄存器寄存器写入读取的扇区数
2）LBA low寄存器，LBA mid寄存器，LBA high寄存器写入lba地址
3）device寄存器写入lba地址和读取模式
4）command寄存器写入写入命令
5）读取两个字节数据，多次循环直到读取完扇区数据。


### 代码

boot.asm
引导文件，初始化屏幕后，读取硬盘并加载4个扇区到内存位置[0x90000]处。然后跳转到0x90000处执行指令。
```assembly
; GloxOS BOOT
;Tab=4
[bits 16]

    org     0x7c00 				;指明程序的偏移的基地址

;----------- loader const ------------------
LOADER_SECTOR_LBA  		equ 0x1		;第2个逻辑扇区开始
LOADER_SECTOR_COUNT		equ 9		;读取9个扇区
LOADER_BASE_ADDR 		equ 0x9000  ;内存地址0x9000
;-------------------------------------------

;引导扇区代码 
    jmp     Entry
    db      0x90
    db      "GLOXBOOT"     		;启动区的名称可以是任意的字符串（8字节）    

;程序核心内容
Entry:

    ; ---------------------------
    ;初始化寄存器
    ; ---------------------------
    mov ax,0				
    mov ss,ax
    mov ds,ax
    mov es,ax
    mov ss,ax
    mov fs,ax
    mov gs,ax
    mov sp,0x7c00

    ; ---------------------------
    ;清屏
    ; ---------------------------
    mov ah,0x06				;清除屏幕					
    mov al,0
    mov cx,0   
    mov dx,0xffff  
    mov bh,0x17				;属性为蓝底白字
    int 0x10
    
	; ---------------------------
    ; 光标位置初始化
    ; ---------------------------
    mov ah,0x02				;光标位置初始化
    mov dx,0
    mov bh,0
    mov dh,0x0
    mov dl,0x0
    int 0x10

    ;------------------
    ;读取硬盘1-10扇区
    ;------------------
    mov ebx,LOADER_SECTOR_LBA 		;LBA扇区号
    mov cx,LOADER_SECTOR_COUNT		;读取扇区数
    mov di,LOADER_BASE_ADDR			;写入内存地址
    call ReadDiskLBA16
    
    jmp LOADER_BASE_ADDR

; ------------------------------------------------------------------------
; 读取磁盘: ReadDiskLBA16
; 参数:
; ebx 扇区逻辑号
; cx 读入的扇区数,8位
; di 读取后的写入内存地址
; ------------------------------------------------------------------------	
ReadDiskLBA16:
    ;设置读取的扇区数
    mov al,cl
    mov dx,0x1F2
    out dx,al
    
    ;设置lba地址
    ;设置低8位
    mov al,bl
    mov dx,0x1F3
    out dx,al
    
    ;设置中8位
    shr ebx,8
    mov al,bl
    mov dx,0x1F4
    out dx,al
    
    ;设置高8位
    shr ebx,8
    mov al,bl
    mov dx,0x1F5
    out dx,al
    
    ;设置高4位和device
    shr ebx,8
    and bl,0x0F
    or bl,0xE0
    mov al,bl
    mov dx,0x1F6
    out dx,al
        
    ;设置commond
    mov al,0x20
    mov dx,0x1F7
    out dx,al

.check_status:;检查磁盘状态
    nop
    in al,dx
    and al,0x88			;第4位为1表示硬盘准备好数据传输，第7位为1表示硬盘忙
    cmp al,0x08
    jnz .check_status   ;磁盘数据没准备好，继续循环检查
            
    ;设置循环次数到cx
    mov ax,cx 			;乘法ax存放目标操作数
    mov dx,256
    mul dx
    mov cx,ax			;循环次数 = 扇区数 x 512 / 2 
    mov bx,di
    mov dx,0x1F0
    
.read_data: 				
    in ax,dx			;读取数据
    mov [bx],ax			;复制数据到内存
    add bx,2    		;读取完成，内存地址后移2个字节
    
    loop .read_data
    ret


Fill0:
    resb    510-($-$$)                   ; 处理当前行$至结束(1FE)填充0
    db      0x55, 0xaa
```
boot.asm
被引导扇区加载到0x90000位置，执行输出`hello in loader`文字

```asm
; GloxOS LOADER
;Tab=4
[bits 16]

section loader vstart=LOADER_BASE_ADDR ;指明程序的偏移的基地址

;----------- loader const ------------------
LOADER_BASE_ADDR 		equ 0x9000  ;内存地址0x9000
;---------------------------------------	
	jmp Entry
	
;程序核心内容
Entry:
	

	;---------------------------
    ;输出字符串
    mov si,HelloMsg			;将HelloMsg的地址放入si
    mov dh,1				;设置显示行
	mov dl,0				;设置显示列
    call PrintString			;调用函数

	
	jmp $			;让CPU挂起，等待指令


		
; ------------------------------------------------------------------------
; 显示字符串函数:PrintString
; 参数:
; si = 字符串开始地址,
; dh = 第N行，0开始
; dl = 第N列，0开始
; ------------------------------------------------------------------------
PrintString:
            mov cx,0     ; BIOS中断参数：显示字符串长度
            mov bx,si
    .s1:                 ; 获取字符串长度
            mov al,[bx]  ; 读取1个字节到al
            add bx,1     ; 读取下个字节
            cmp al,0     ; 是否以0结束
            je .s2
            inc	cx       ; 计数器
            jmp .s1
    .s2:                 ; 显示字符串
            mov bx,si
            mov bp,bx
            mov ax,ds
            mov es,ax    ; BIOS中断参数：计算[ES:BP]为显示字符串开始地址

            mov ah,0x13  ; BIOS中断参数：中断模式
            mov al,0x01  ; BIOS中断参数：输出方式
            mov bh,0x0   ; BIOS中断参数：指定分页为0
            mov bl,0x1F  ; BIOS中断参数：显示属性，指定白色文字
            int 0x10     ; 调用BIOS中断操作显卡。输出字符串
            ret
            
; ------------------------------------------------------------------------
;准备显示字符串
HelloMsg: db "load loader file success!",0

times	512-($-$$) db  0 ; 处理当前行$至结束(1FE)的填充	
```

### 运行



创建build.sh脚本

```shell
#!/bin/bash

NASM=nasm
$NASM -f bin -o build/boot.bin boot/boot.asm	
$NASM -f bin -o build/loader.bin boot/loader.asm
dd if=/dev/zero of=build/gloxos.img bs=512 count=2880
dd if=build/boot.bin  of=build/gloxos.img bs=512 count=1  conv=notrunc
dd if=build/loader.bin of=build/gloxos.img bs=512 count=1 seek=1 conv=notrunc
```

创建run.sh脚本

```shell
#!/bin/bash

BOCHS=bochs
$BOCHS -f bochsrc.me
```



运行结果

![2_5_1.png](2_5_1.png)