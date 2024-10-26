---
title: "2.2_文本显示模式"
date: 2018-11-05T11:27:28
lastmod: 2021-03-20
timezone: UTC+8
tags: ["操作系统"]
categories: ["操作系统"]
draft: false
---





#  文本显示模式

[TOC]


### 显示模式

计算机在加电自检之后，会将显示初始化为80 x 25的文本模式。此时，我们可以进行文本显示了。



而计算机的显示一般有2种模式，可以通过中断来修改显示模式

- 文本模式
- 图形模式

文本模式只能显示字符，一般通过BIOS中断修改。不过首先我们尝试修改显存的方式来显示字符。



而计算机的显示一般有2种模式：

* 文本模式：开始地址地址：0xB8000
* 图形模式:  开始地址地址：0xA0000

图形模式，存储的内容主要是颜色信息，而文字模式，存储的内容主要是文字信息。

比如，刚启动linux时，输出的一堆的启动信息就是，进入文本模式。而开启xwindows才会进入图形模式。

根据分辨率，可用的显示模式如下所列：





### 文本显示模式

**2. 80 x 25文本模式**

刚开始启动计算机时，系统默认进入文本模式。在计算机在加电自检完成之后，会默认将显示初始化为80 x 25的文本模式

在 80 x 25的文本模式，屏幕可以显示25行80列。显示地址段是位于0xB8000-oxBffff的地址段。

我们可以通过修改0xB8000-oxBffff地址段的值，来在屏幕上显示文本。

此模式下每2个内存地址为一组，32位代表一个文字输出： 高地址16位为颜色信息，低地址16位为文字信息

因此我们可以通过修改这段显示地址区域的值，从而来控制屏幕输出文字。



例如：80x25文本模式的颜色如下：

在 80 x 25的文本模式，显存地址是位于0xB8000-oxBffff。

背景色颜色（背景），4位分别为 KRGB，K为是否闪烁
|值|颜色|值|颜色|
|---|---|---|---|
|0|黑色|4|红色|
|1|蓝色|5|紫色|
|2|绿色|6|黄色|
|3|青色|7|白色|


前景色颜色（文字），4位，分别为 IRGB

|值|颜色|值|颜色|
|---|---|---|---|
|0|黑色|8|灰色|
|1|蓝色|9|淡蓝色|
|2|绿色|A|淡绿色|
|3|青色|B | 淡青色|
|4|红色|C | 淡红色|
|5|紫色|D |淡紫色|
|6|黄色|E | 淡黄色|
|7|白色|F | 亮白色|



## 显示字符

**1. 通过修改内存数据来显示字符**

启动后实模式下-文本模式下的初始显存地址范围为[0xB8000-0xBFFFFF]。

显存地址的值对应屏幕的显示数据，我们可以修改显存值来改变屏幕显示。

我们使用段和偏移来表示这段显存信息，段基本地址为0xB800，偏移为0x0000到0xFFFF。

代码如下：



```assembly
mov ax,0xb800
mov ds,ax                ;配置显存段地址
mov byte [0x00],'r'      ;输出字符，内存地址为 DS<<4 + 0x00
mov byte [0x01],0x17     ;设置颜色(背景色蓝，前景色白)
```



**2. 实现代码**



新建一个目录boot，新建文件boot/boot.asm,代码如下
```assembly
;GloxOS BOOT
;TAB=4
[bits 16]
	org     0x7c00 				;指明程序的偏移的基地址

;boot程序
	jmp     Entry

;程序核心内容
Entry:
	mov ax,0xb800
	mov gs,ax                   ;显存段地址
	mov byte [gs:0x00],'h'      ;输出字符
	mov byte [gs:0x01],0x1F     ;设置颜色(背景色蓝，前景色白)
	mov byte [gs:0x02],'e'
	mov byte [gs:0x03],0x1F
	mov byte [gs:0x04],'l'
	mov byte [gs:0x05],0x1F
	mov byte [gs:0x06],'l'
	mov byte [gs:0x07],0x1F
	mov byte [gs:0x08],'o'
	mov byte [gs:0x09],0x1F
	mov byte [gs:0x0a],','     
	mov byte [gs:0x0b],0x1F     
	mov byte [gs:0x0c],'g'		 ;输出字符
	mov byte [gs:0x0d],0x1D		 ;设置颜色(背景色蓝，前景淡紫)
	mov byte [gs:0x0e],'l'
	mov byte [gs:0x0f],0x1D
	mov byte [gs:0x10],'o'
	mov byte [gs:0x11],0x1D
	mov byte [gs:0x12],'x'
	mov byte [gs:0x13],0x1D
                 	
	jmp $						;进入死循环，不再往下执行。

Fill_Sector:
	resb    510-($-$$)       	;处理当前行$至结束(1FE)的填充
	db      0x55, 0xaa
```


编译成gloxos.img镜像文件。

总结，也可以构建完整的 build.sh 执行脚本如下

```nasm
#!/bin/bash

NASM=nasm
$NASM -f bin -o boot.bin boot/boot.asm
dd if=/dev/zero of=build/gloxos.img bs=512 count=2880
dd if=build/boot.bin  of=build/gloxos.img bs=512 count=1  conv=notrunc
```



**3. 在虚拟机运行**


击显示运行系统：

![images/1.8_1.png](/images/1.8_1.png)



