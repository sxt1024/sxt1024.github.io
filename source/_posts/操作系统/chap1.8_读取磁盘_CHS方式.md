---
title: chap1.8_读取磁盘_CHS方式
date: 2018-11-13T11:27:28
lastmod: 2018-11-13
timezone: UTC+8
tags:
  - 操作系统
categories:
  - 操作系统
draft: false
---


### 软盘和硬盘

目前主流的硬盘分为两种：

- 机械硬盘

- 固态硬盘

硬盘最早分为两种接口方式：

- 并行接口（PATA），目前已经淘汰。

- 串行接口（SATA）。



### 磁盘寻址：CHS方式

磁盘的三层定位结构分别为

- 柱面(Cylinder)
- 磁头(Head)
- 扇区(Sector)

磁盘寻址，是指根据给出的方式找到扇区的位置。

按顺序根据柱面，磁头，扇区，这种方式称之为CHS方式。

例如: 磁盘的第一个扇区，位于 1柱面，1磁头，0扇区



**CHS方式表示的容量上限**

这种模式及下，支持的最大柱面数为1024，最大磁头数为16，最大扇区数为63，（每个扇区字节数为512Bit）因此最大访问硬盘容量为：

> 1024 x 16 x 64 x 512 = 528MB

虽然后面又拓展了large模式读取，但是没有解决根本问题。

所以后来有LBA的读取方式，可以描述更大容量。





## 读取磁盘_CHS方式

### BIOS读取磁盘

读取磁盘也是调用BIOS：


> 中断命令: INT 13H

入口参数

| 寄存器 | 说明              | 值   |
| ------ | ----------------- | ---- |
| AH     | 功能：02H读取扇区 | 02H  |
| AL     | 扇区数            | ---  |
| CH     | 柱面数            | ---  |
| CL     |  扇区  | ---  |
| DH     |  磁头  | ---  |
| DL     |驱动器号   | 00H-7FH：软盘驱动器号；80H-0FFH：硬盘驱动器号 |
| ES:BX  |缓冲区的地址   | 00H-7FH：软盘驱动器号；80H-0FFH：硬盘驱动器号 |




出口参数

读取成功：

| 寄存器 | 说明              | 值   |
| ------ | ----------------- | ---- |
| AH    |  00H            | ---  |
| AL    |  输的扇区数            | ---  |

读取失败：

| 寄存器 | 说明              | 值   |
| ------ | ----------------- | ---- |
| AH    |  状态代码            | ---  |




###  定义磁盘读取函数

**1. 读取一个扇区**

```assembly
.read_one_sector			
	MOV	AH,0x02				;参数：读扇区
	MOV	AL,1				;参数：读取扇区数
	MOV	BX,0				;参数：
	MOV	DL,0x00				;参数：设置读取驱动器为软盘
	INT	0x13				;调用BIOS中断操作磁盘：读取扇区
```



```assembly
; ------------------------------------------------------------------------
; 读取一个扇区函数:ReadDisk0
; ------------------------------------------------------------------------
; 参数:ES:BX 缓冲区地址，CH柱面，DH磁头，CL扇区
; ------------------------------------------------------------------------
ReadDisk0:
	MOV	SI,0				;初始化读取失败次数，用于循环计数
	
    ;为了防止读取错误，循环读取5次
    ;调用BIOS读取一个扇区
    ReadFiveLoop:
			MOV	AH,0x02				;BIOS中断参数：读扇区
			MOV	AL,1				;BIOS中断参数：读取扇区数
			MOV	BX,0
			MOV	DL,0x00				;BIOS中断参数：设置读取驱动器为软盘
			INT	0x13				;调用BIOS中断操作磁盘：读取扇区
			JNC	ReadEnd				;条件跳转，操作成功进位标志=0。则跳转执行ReadNextSector
	
			inc	si				;循环读取次数递增+1
			CMP	SI,5				;判断是否已经读取超过5次
			JAE	LoadError			;上面cmp判断(>=)结果为true则跳转到DisplayError
	
			MOV	AH,0x00				;BIOS中断参数：磁盘系统复位
			MOV	DL,0x00				;BIOS中断参数：设置读取驱动器为软盘
			INT	0x13				;调用BIOS中断操作磁盘：磁盘系统复位
			JMP	ReadFiveLoop
	;扇区读取完成
	ReadEnd:
			RET
```



**2. 读取多个扇区**

读取时要根据 扇区<磁头<柱面 的方式来读取。继续添加代码，读取18个扇区：（即完整的读取了一个柱面）。

代码如下：

​	

**3. 读取多个柱面**

继续添加代码，读取10个柱面。

​	

然后调用函数

```assembly
	;读取磁盘初始化
		MOV	AX,DISC_ADDR/0x10	;设置磁盘读取的缓冲区基本地址为ES=0x820。[ES:BX]=ES*0x10+BX
		MOV	ES,AX				;BIOS中断参数：ES:BX＝缓冲区的地址

		MOV	CH,0				;设置柱面为0
		MOV	DH,0				;设置磁头为0
		MOV	CL,2				;设置扇区为2

	ReadSectorLoop:
		CALL ReadDisk0;			;读取一个扇区
```



### CHS方式读取（使用LBA值转换后读取）



```assembly
; ==============================================
; 读取磁盘:Func_readCHS2LBA
; 参数:
; ebx 扇区逻辑号
; cx 读入的扇区数,8位
; es 读取到内存单元的地址
; ==============================================
.reset
    MOV	AH,0x00				;BIOS中断参数：磁盘系统复位
    MOV	DL,0x00				;BIOS中断参数：设置读取驱动器为软盘
    INT	0x13				;调用BIOS中断操作磁盘：磁盘系统复位
    ret
    
.readdisk
				;初始化读取失败次数，用于循环计数
	push cx
	MOV	cx,5	
	
    ;为了防止读取错误，循环读取5次
    ;调用BIOS读取一个扇区
    ReadFiveLoop:
			MOV	AH,0x02				;BIOS中断参数：读扇区
			MOV	AL,1				;BIOS中断参数：读取扇区数
			MOV	BX,0
			MOV	DL,0x00				;BIOS中断参数：设置读取驱动器为软盘
			INT	0x13				;调用BIOS中断操作磁盘：读取扇区
			JNC	.readok				;条件跳转，操作成功进位标志=0。则跳转执行ReadNextSector
	
			inc	si				;循环读取次数递增+1
			CMP	SI,5				;判断是否已经读取超过5次
			JAE	.readfail			;上面cmp判断(>=)结果为true则跳转到DisplayError
	
			call .reset
			loop ReadFiveLoop
			
    ;准备下一个扇区
    .readok:
    		
            MOV AX,ES
            ADD AX,0x0020			
            MOV	EX,AX				;内存单元基址后移0x20。[EX+0x20:]
            ADD CL,1				;读取扇区数递增+1
            CMP CL,18				;判断是否读取到18扇区
            JBE readdisk		;上面cmp判断(<=)结果为true则跳转到DisplayError	
            
     .readxx       
     ;读取另一面磁头。循环读取柱面
		MOV CL,1				;设置柱面为0
		ADD DH,1				;设置磁头递增+1:读取下一个磁头
		CMP DH,2				;判断磁头是否读取完毕
		JB ReadSectorLoop		;上面cmp判断(<)结果为true则跳转到DisplayError

		MOV DH,0				;设置磁头为0
		ADD CH,1				;设置柱面递增+1;读取下一柱面
		CMP	CH,10				;判断是否已经读取10个柱面
		JB	readdisk		;上面cmp判断(<)结果为true则跳转到DisplayError       
```




### 完整代码

最后，完整的boot.asm文件代码如下：

```assembly
; GloxOS BOOT
[bits 16]

; ----------- loader const ------------------
LOADER_CYLINDER_NUM			equ 10              ; 读取10个柱面
LOADER_SECTOR_CAP		equ 18                 ; 读取18个扇区
LOADER_SECTOR_CHS 		equ 0x1              ; 第2个逻辑扇区开始
LOADER_BASE_ADDR 		equ 0x9000            ; 内存地址0x9000
; -------------------------------------------


org     0x7c00                           ; 指明程序的偏移的基地址

                                         ; 启动程序
jmp     Entry
db      0x90
db      "GLOXBOOT"                       ; 启动区的名称可以是任意的字符串（8字节）

                                         ; 程序核心内容
Entry:

    ; ---------------------------
    ; 初始化寄存器
    ; ---------------------------
    mov ax,0
    mov ss,ax
    mov ds,ax
    mov es,ax
    mov sp,0x7c00

    ; ---------------------------
    ; 清除屏幕
    ; ---------------------------
    mov ah,0x06
    mov al,0
    mov cx,0
    mov dx,0xffff
    mov bh,0x17                          ; 属性为蓝底白字
    int 0x10

    ; ---------------------------
    ; 光标位置初始化
    ; ---------------------------
    mov ah,0x02
    mov dx,0
    mov bh,0
    mov dh,0x0
    mov dl,0x0
    int 0x10

    ; ---------------------------
    ; 输出字符串
    ; ---------------------------
    mov  si,BootMsg                      ; 将BootMsg的地址放入si
    mov  dh,0                            ; 设置显示行
    mov  dl,0
    call PrintString                       ; 调用函数

    ; ---------------------------
    ; 读取磁盘
    mov	ax,LOADER_BASE_ADDR/0x10         ; 设置磁盘读取的缓冲区基本地址为ES=0x900。[ES:BX]=ES*0x10+BX
    mov	es,ax                            ; BIOS中断参数：ES:BX＝缓冲区的地址 [0x900:0x0]

    mov	ch,0                             ; 设置柱面为0
    mov	dh,0                             ; 设置磁头为0
    mov	cl,1                             ; 设置扇区为2

; ---------------------------
; 循环读取扇区，读取10个柱面
; ---------------------------
ReadSectorLoop:
    call ReadDiskSector                  ; ;读取一个扇区

    ; 准备下一个扇区
    .readNext:
        mov	ax,es
        add	ax,0x0020
        mov	es,ax                        ; 内存单元基址后移0x20(512字节)。[ES+0x20:]
        add	cl,1                         ; 读取扇区数递增+1
        cmp	cl,LOADER_SECTOR_CAP   		 ; 判断是否读取到18扇区
        jbe	ReadSectorLoop               ; 上面cmp判断(<=)结果为true则跳转到DisplayError

        ; 读取另一面磁头。循环读取柱面
        mov	cl,1                         ; 设置柱面为0
        add	dh,1                         ; 设置磁头递增+1:读取下一个磁头
        cmp	dh,2                         ; 判断磁头是否读取完毕
        jb	ReadSectorLoop                ; 上面cmp判断(<)结果为true则跳转到DisplayError

        mov	dh,0                         ; 设置磁头为0
        add	ch,1                         ; 设置柱面递增+1;读取下一柱面
        cmp	ch,LOADER_CYLINDER_NUM       ; 判断是否已经读取10个柱面
        jb	.readFin                      ; 上面cmp判断(<)结果为true则跳转到DisplayError
        jmp     .readNext

    ; 读取完成
    .readFin:
            mov si,SuccessMsg
            mov dh,1
            mov dl,0
            call PrintString               ; 如果加载失败显示加载错误
            ret


; ------------------------------------------------------------------------
; 读取一个扇区函数:ReadDiskSector
; 参数:
; es:bx = 缓冲区地址，
; ch = 柱面
; dh = 磁头
; cl = 扇区
; ------------------------------------------------------------------------
ReadDiskSector:
    mov	si,0                             ; 初始化读取失败次数，用于循环计数

    ; 为了防止读取错误，循环读取5次
    ; 调用BIOS读取一个扇区
    .readFiveLoop:
            mov	ah,0x02                  ; BIOS中断参数：读扇区
            mov	al,1                     ; BIOS中断参数：读取扇区数
            mov	bx,0
            mov	dl,0x00                  ; BIOS中断参数：设置读取驱动器为软盘
            int	0x13                     ; 调用BIOS中断操作磁盘：读取扇区
            jnc	.readEnd                 ; 条件跳转，操作成功进位标志=0。则跳转执行ReadNextSector

            add	si,1                     ; 循环读取次数递增+1
            cmp	si,5                     ; 判断是否已经读取超过5次
            jae	.readError               ; 上面cmp判断(>=)结果为true则跳转到readError

            mov	ah,0x00                  ; BIOS中断参数：磁盘系统复位
            mov	dl,0x00                  ; BIOS中断参数：设置读取驱动器为软盘
            int	0x13                     ; 调用BIOS中断操作磁盘：磁盘系统复位
            jmp	.readFiveLoop

    ; 读取完成
    .readEnd:
            ret

    ; 读取错误
    .readError:
            mov si,ErrorMsg
            mov dh,1
            mov dl,0
            call PrintString               ; 如果加载失败显示加载错误
            ret

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
; 准备显示字符串
BootMsg: 	db 	"GloxOS is booting!",0
SuccessMsg:     db	"load disk Sector success.",0
ErrorMsg: 	db      "load disk Sector error.",0


Fill0:
    resb    510-($-$$)                   ; 处理当前行$至结束(1FE)填充0
    db      0x55, 0xaa

```



**运行**



创建build.sh脚本

```shell
#!/bin/bash

NASM=nasm
$NASM -f bin -o build/boot.bin boot/boot.asm
dd if=/dev/zero of=build/gloxos.img bs=512 count=2880
dd if=build/boot.bin  of=build/gloxos.img bs=512 count=1  conv=notrunc
```

创建run.sh脚本

```shell
#!/bin/bash

BOCHS=bochs
$BOCHS -f bochsrc.me
```



运行结果如下：

![2_1_1.png](2_1_1.png)

## 上面代码的作用
首先boot.asm会被加载到内存并且执行.后面开始读取磁盘的10个柱面(10*18个扇区).
读取的扇区数据复制到 内存 0x8000 开始的位置.
然后打印输出"hello,rats os start"

