---
title: chap3.2_显示点阵字
date: 2019-02-20T11:27:28
lastmod: 2019-02-20
timezone: UTC+8
tags:
  - 操作系统
categories:
  - 操作系统
draft: false
---






## VGA图形模式



include/stdint.h

```c
#ifndef __LIB_STDINT_H
#define __LIB_STDINT_H

typedef signed char int8;
typedef signed short int16;
typedef signed int int32;
typedef signed long long int64;

typedef unsigned char uint8;
typedef unsigned short uint16;
typedef unsigned int uint32;
typedef unsigned int uint;
typedef unsigned long long uint64;

#endif
```



include/console.h

```c

#ifndef __LIB_CONSOLE_H
#define __LIB_CONSOLE_H

#include "stdint.h"

uint16 get_cursor();

void set_cursor(uint16 pos);

void put_char(char ch);

void put_string(char* chs);

void put_pointer(uint32 addr);

#endif

```





lib/console.c

```c

#include "../include/stdint.h"
#include "../include/io.h"
#include "../include/console.h"

#define VGA_BASE 0xB8000
#define DATA_BASE 0x60000
#define ROW 25
#define COL 80

uint16 get_cursor()
{   
    //get high cursor value
    io_out8(0x3d4, 0x0e);
    uint8 cursor_high = io_in8(0x3d5);
    //get low cursor value
    io_out8(0x3d4, 0x0f);
    uint8 cursor_low = io_in8(0x3d5);
    //high + low
    return (cursor_high << 8) + (cursor_low & 0xff);
}

void set_cursor(uint16 pos)
{
    uint8 cursor_high = pos >> 8;
    uint8 cursor_low = pos & 0xff;
    //set high cursor value
    io_out8(0x3d4, 0x0e);
    io_out8(0x3d5, cursor_high);
    //set low cursor value
    io_out8(0x3d4, 0x0f);
    io_out8(0x3d5, cursor_low);
}

void put_char(char ch)
{

    uint16 pos = get_cursor();
    char *pvga = (char *)VGA_BASE;

    //字符
    switch (ch)
    {
        case 0x0d: //RETURN
            pos = (pos / COL) * COL;
            break;
        case 0x0a: //NEW LINE
            pos = pos + COL;
            pos = (pos / COL) * COL;
            break;
        case 0x09: //TAB
            pos=pos+4;
            break;
        case 0x08: //BACKSPACE
            pos--;
            *(pvga + pos * 2) = 0x00;
            break;
        default:
            *(pvga + pos * 2) = ch;
            pos++;
    }
    //字符超出时, 滚屏
    if (pos + 1 > ROW * COL )
    {
        int display = (ROW -1) * COL;
        pos = pos - COL;
        for (int i = 0; i < display; i++)
        {
            *(pvga + 0x00 + i * 2) = *(pvga + 0xa0 + i * 2);
        }
        for (int i = 0; i < COL ; i++)
        {
            *(pvga + display * 2 + i * 2) = 0x0;
        }
    }
    set_cursor(pos);
}

void put_string(char *chs)
{
    int i = 0;
    while (*(chs + i) != 0)
    {
        put_char(*(chs + i));
        i++;
    }
}

void put_pointer(uint32 addr)
{   
    put_char('0');
    put_char('x');
    int i = 0;
    char *chs = (char *)DATA_BASE; //need point mem alloc
    do
    {
        *(chs + i) = (char)(addr % 10 + '0'); //取下一个数字
        i++;

    } while ((addr /= 10) > 0); //删除该数字
    for (int j = i; j >= 0; j--)
    { //生成的数字是逆序的，所以要逆序输出
        put_char(*(chs + j));
        *(chs + j) = 0;
    }
}

```

main.c

```c
#include "../include/io.h"
#include "../include/console.h"

int _start(){
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



Makefile

```makefile
# tools
PLATFORM=Linux
NASM=nasm
BOCHS=bochs
BXIMAGE=bximage


# args
boot=boot
asm=asm
lib=lib
kernel=kernel
build=build
ENTRY_POINT =  0x70000
CFLAGS = -m32 -c  # --Wall -W  fno-builtin -Wstrict-prototypes -Wmissing-prototypes 
LDFLAGS = -m elf_i386 -e _start -Ttext $(ENTRY_POINT) 

target: prepare $(build)/gloxos.img	
	@echo "build img completed"

$(build)/gloxos.img: $(build)/boot.bin $(build)/loader.bin $(build)/kernel.bin
	$(BXIMAGE) -mode=create   -imgmode=flat  -hd=16M  -q $(build)/gloxos.img 
	sleep 1
	dd if=$(build)/boot.bin of=$(build)/gloxos.img bs=512 count=1  conv=notrunc
	dd if=$(build)/loader.bin of=$(build)/gloxos.img bs=512 count=1 seek=1 conv=notrunc
	dd if=$(build)/kernel.bin of=$(build)/gloxos.img bs=512 count=25 seek=5 conv=notrunc


$(build)/%.bin: $(boot)/%.asm
	$(NASM) -f bin -o $(build)/$*.bin $(boot)/$*.asm	

$(build)/kernel.bin: $(build)/kernel.o  $(build)/io.o $(build)/console.o
	$(LD) -o $@  $^  $(LDFLAGS) 

$(build)/kernel.o: 
	$(CC) -o $(build)/kernel.o  $(kernel)/main.c -I include $(CFLAGS) 

$(build)/console.o:
	$(CC) -o $(build)/console.o  $(lib)/console.c -I include $(CFLAGS) 

$(build)/io.o: 
	$(NASM) -f elf -o $(build)/io.o $(asm)/io.asm


prepare:
	mkdir -p $(build)

clean:
	@echo "clean dir $(build)"
	rm -rf $(build)/*

platform:
	@echo $(PLATFORM)


```


附：ASCII字符编码表


| 0x01 | SOH(start of headline) | 标题开始 |
| ---- | ---------------------- | -------- |
|0x00 |NUL  |空字符|
|0x08|BS (backspace)|退格|
|0x09|HT (horizontal tab)|水平制表符|
| 0x0A| LF (NL line feed, new line)| 换行键|
| 0x0B | VT (vertical tab) | 垂直制表符 |
|0x0C|FF (NP form feed, new page)|换页键|
|0x0D|CR (carriage return)|回车键|
|0x20|(space)|空格|
|0x7F|DEL (delete)|删除|

