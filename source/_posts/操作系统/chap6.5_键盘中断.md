## 键盘中断



键盘的中断信号连接主片IR1引脚.中断向量号为 0x21



## 键盘控制芯片

键盘8042寄存器

| 寄存器                      | 端口 | 读写 |
| --------------------------- | ---- | ---- |
| Output Buffer(输出缓冲区)   | 0x60 | 读   |
| Intput Buffer(输入缓冲区)   | 0x60 | 写   |
| Status Register(状态寄存器) | 0x64 | 读   |
| Control Register(控制寄存器) | 0x64 | 读写|






## 读取键盘缓冲区

### 读取键盘缓冲区

读取0x60端口，可以读取出键盘缓冲区的按键数据，然后键盘缓冲区就可以继续接收数据了。

如下：

```
in al,0x60
```

但是，我发现我这不行，必须往0x20端口发送0x20数据，告诉中断结束。才能进行数据读取。





###   重启中断控制

当执行一次中断返回后，下一次主断就无法继续监听了。必须使用指令，在触发中断，接受数据之后，发送指令表示重新监听中断事件。

```
   //EOI结束中断,以便开始下次中断
    io_out8(PIC0_ICW1,0x20);
    
    或者
    //  通知PIC已经受理完毕
    io_out8(PIC0_ICW2,0x61);
```







### 汇编

头文件：io.h

```c
void io_keyboard_handler();
```


汇编：io.asm

中断完需要调用 iret返回，使用汇编。

```assembly
global io_interrupt, io_keyboard_handler
extern keyboard_handler_callback

io_interrupt: ; void io_interrupt(void* func)
    pushad
    mov al,0x20
    out 0xa0, al
    out 0x20, al
    mov ebx,[esp+8]
    mov ecx,4
    mul ebx
    add ebx,[esp+4]
    call [ebx]
    popad
    iretd

io_keyboard_handler:  ;void io_keyboard_handler()
    push	es
    push	ds
    pushad
    mov		eax,esp
    push	eax
    mov		ax,ss
    mov		ds,ax
    mov		es,ax
    call	keyboard_handler_callback
    pop		eax
    popad
    pop		ds
    pop		es
    iret
```





idt.h

```
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

```


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



### 回调用c函数

keyboard.h

```c
#ifndef __LIB_KEYBORD_H
#define __LIB_KEYBORD_H

void keyboard_handler_callback();

void init_keyboard();

#endif
```

keyboard.c

```c
#include "../include/global.h"
#include "../include/stdint.h"
#include "../include/io.h"
#include "../include/keyboard.h"


/** KEYBORD */
#define PORT_KBD_BUF 0x60

void keyboard_handler_callback()
{

     //读取键盘缓冲器
    uint8 ch = io_in8(PORT_KBD_BUF);
    putchar(ch);

    //EOI结束中断,以便开始下次中断
    io_out8(PIC0_ICW1,0x20);

    putstring(" do keyboard_handler_callback!\n");
}

void init_keyboard() {
    register_handler(0x21, io_keyboard_handler);
    putstring("init_keyboard!\n");
}
```



注册键盘函数

```c
/**
 * 初始化IDT
 */
void init_idt()
{
    //init pic
    init_pic();

    //init idt interrupt
    init_interrupt_desc();
    
    //init keybord
    init_keybord();

    //load idt
    uint64 idt_info = ((sizeof(idt) - 1) | ((uint64)idt << 16));
    io_lidt(&idt_info);


    /* 打开主片上IR0键盘中断,打开键盘中断处理，其他中断全关闭,keybord: 0xfb */
    io_out8(PIC0_IMR, 0xfb);
    io_out8(PIC1_IMR, 0xff);

    io_sti();
    io_out8(PIC0_IMR, 0xf9); /* PIC1偲僉乕儃乕僪傪嫋壜(11111001) */
    io_out8(PIC1_IMR, 0xef); /* 儅僂僗傪嫋壜(11101111) */
}

```



启动Bochs，在键盘中随便输入，发现有输出`keybord_handler_callback` ，但是无论调用多少次，只会打印一次`keybord_handler_callback`。







### 修改回调用c函数



keyboard.c

```c
#define PORT_KBD_BUF 0x60

void keybord_handler_callback()
{
    //读取键盘缓冲器
    uint8 ch = io_in8(PORT_KBD_BUF);
    putchar(ch);

      //EOI结束中断,以便开始下次中断
    io_out8(PIC0_ICW1,0x20);

    putstring("do keybord_handler_callback!\n");
}

void init_keyboard()
{
    putstring("init_keyboard!\n");
    register_handler(0x21, io_keybord_handler);
}
```

