## 加载IDT

global.h

```assembly
#ifndef __LIB_GLOBAL_H
#define __LIB_GLOBAL_H

#define RPL0 0
#define RPL1 1
#define RPL2 2
#define RPL3 3

#define TI_GDT 0
#define TI_LDT 1

#define SELECTOR_CODE ((1 << 3) + (TI_GDT << 2) + RPL0)         
#define SELECTOR_DATA ((2 << 3) + (TI_GDT << 2) + RPL0)         
#define SELECTOR_STACK SELECTOR_DATA
#define SELECTOR_GS ((3 << 3) + (TI_GDT << 2) + RPL0)

#define IDT_DESC_P 1
#define IDT_DESC_DPL0 0
#define IDT_DESC_DPL3 3
#define IDT_DESC_TYPE16 0x6
#define IDT_DESC_TYPE32 0xe

#define IDT_DESC_ATTR_DPL0 ((IDT_DESC_P << 7) + (IDT_DESC_DPL0 << 5) + IDT_DESC_TYPE32)
#define IDT_DESC_ATTR_DPL3 ((IDT_DESC_P << 7) + (IDT_DESC_DPL3 << 5) + IDT_DESC_TYPE32)



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

#endif
```



io.h

```
void io_lgdt(uint64* p_gdt);

void io_lidt(uint64* p_gdt);
```

io.asm

```
[bits 32]
global io_lgdt, io_lidt


[section .text]

io_lgdt:      ; void io_lgdt(uint64 p_gdt)
    mov eax, [esp+4]
    lgdt [eax]
    ret

io_lidt:      ; void io_lidt(uint64 p_gdt)
    mov eax, [esp+4]
    lidt [eax]
    ret
```



idt.h

```assembly
#ifndef __LIB_IDT_H
#define __LIB_IDT_H

#include "stdint.h"

struct gate_desc
{
    uint16 handler_offset_low;
    uint16 selector;
    uint8 dw_count;
    uint8 attribute;
    uint16 handler_offset_high;
};

void init_pic();

void init_idt();

void init_interrupt_desc();

void set_interrupt_desc(struct gate_desc *p_gate_desc, uint8 attr, void *handler);

#endif
```



idt.c

```assembly


#include "../include/stdint.h"
#include "../include/io.h"
#include "../include/global.h"
#include "../include/interrupt.h"
#include "../include/console.h"

#define IDT_DESC_COUNT 0x21

struct gate_desc idt[IDT_DESC_COUNT];
void* handler_table[IDT_DESC_COUNT];

void set_interrupt_desc(struct gate_desc *p_gate_desc, uint8 attr, void* handler)
{
    p_gate_desc->handler_offset_low = (uint32)handler & 0x0000ffff;
    p_gate_desc->selector = SELECTOR_CODE;
    p_gate_desc->dw_count = 0;
    p_gate_desc->attribute = attr;
    p_gate_desc->handler_offset_high = ((uint32)handler & 0xffff0000) >> 16;
}

void init_interrupt_desc()
{
    for (int i = 0; i < IDT_DESC_COUNT; i++)
    {
        set_interrupt_desc(&idt[i], IDT_DESC_ATTR_DPL0, handler_table[i]);
    }
}

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

   	//初始化pic
    init_pic(); 		 

	//初始化idt表
    init_interrupt_desc();    
   	
    //加载idt表
    uint64 idt_info = ((sizeof(idt) - 1) |((uint64)idt << 16));
    io_lidt(&idt_info);     
    
    //打印消息
    char *str2 = "init_idt completed\n";
    putstring(str2);
}

```



![chap5.3_加载IDT/images/001.png](chap5.3_加载IDT/images/001.png)
