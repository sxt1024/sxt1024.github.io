## PIC中断控制器



## PIC简介

PIC：可编程中断控制器，英文：Programmable Interrupt Controller，简称PIC。



作用：

由于中断的种类繁多，情况复杂，所以CPU并不会直接进行中断调用处理，为了节省CPU资源，CPU与外被设备的通信是通过PIC设备进行转接。

PIC，是外部设备和CPU进行中断通信的桥梁。CPU发送中断请求给外部设备，或者CPU接受外部设备发送中断请求时，都要通过PIC控制器进行中转。

1. PIC 分为主从两部分，每部分有11个寄存器，每个寄存器大小8位
2. CPU对外设的操作使用`in， out`指令完成。





## 8259A中断处理芯片

以目前常见的8259A方案为例：

PIC为8259A芯片组成在一起：

- 1. 主PIC: master PIC，8位

- 2. 从PIC: slave PIC，8位

主PIC负责第`0~7号`中断，从PIC负责第`8~15`号中断。

每个PIC包含以下寄存器。

- IMR
- IRR
- ISR
- PR
- ICW1，ICW2，ICW3，ICW4（寄存器大小为8位）
- OCW1，OCW2，OCW3（寄存器大小为8位）



## PIC指令：

PIC使用

`in  al, PIC端口地址 `

发送PIC数据到CPU寄存器

`out PIC端口地址, al`

写入CPU寄存器数据到PIC



  缺图。



### 初始化PIC_8259A



io.asm

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

io.h

```
#ifndef __LIB_IO_H
#define __LIB_IO_H

#include "stdint.h"

#define PIC0_ICW1  0x20
#define PIC0_ICW2  0x21
#define PIC0_ICW3  0x21
#define PIC0_ICW4  0x21
#define PIC0_IMR   0x21

#define PIC1_ICW1  0xa0
#define PIC1_ICW2  0xa1
#define PIC1_ICW3  0xa1
#define PIC1_ICW4  0xa1
#define PIC1_IMR   0xa1


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



idt.h

```


#ifndef __LIB_INTERRUPT_H
#define __LIB_INTERRUPT_H

#include "stdint.h"

void init_pic();

void init_idt();

#endif
```



idt.c

```c
#include "../include/io.h"
#include "../include/interrupt.h"

 //初始化PIC
void init_pic()
{   
    io_out8(PIC0_IMR, 0xff); 
    io_out8(PIC1_IMR, 0xff);

    //master
    io_out8(PIC0_ICW1, 0x11);
    io_out8(PIC0_ICW2, 0x20);
    io_out8(PIC0_ICW3, 0x04);
    io_out8(PIC0_ICW4, 0x01);

    //slaver
    io_out8(PIC1_ICW1, 0x11);
    io_out8(PIC1_ICW2, 0x28);
    io_out8(PIC1_ICW3, 0x02);
    io_out8(PIC1_ICW4, 0x01);

     /* 打开主片上IR0,也就是目前只接受时钟产生的中断 */
    io_out8(PIC0_IMR, 0xfb); 
    io_out8(PIC1_IMR, 0xff);
}


void init_idt()
{
	
    init_pic();
    
    io_sti();   //允许中断

    /** 打开键盘中断处理 **/
    io_out8(PIC0_IMR, 0xf9);
    io_out8(PIC1_IMR, 0xef); 
}
```



main.c文件

```c
#include "../include/io.h"
#include "../include/console.h"
#include "../include/idt.h"

int _start(){

   
    init_idt();

    put_char('h');
    put_char('e');
    put_char('l');
    put_char('l');
    put_char('o');
    put_char(',');
    put_char('g');
    put_char('l');
    put_char('o');
    put_char('x');
    io_hlt();
}
```

