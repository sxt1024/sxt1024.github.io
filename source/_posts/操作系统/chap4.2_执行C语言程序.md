---
title: "3.7_C语言程序"
date: 2019-01-13T11:27:28
lastmod: 2019-01-13
timezone: UTC+8
tags: ["操作系统"]
categories: ["操作系统"]
draft: false
---



# C语言程序

##  工具

首先，需要的工具软件列表：

- gcc编译器：



### 编译C语言程序


**1. 为什么没有main函数**

main函数链接时需要一些系统库文件。而我们的系统目前并没有任何的系统库可以用，会导致报错。

所以此处不能使用main函数。

那么我们使用默认的入口_start符号(不设置,则默认为0)

kernel/main.c

```c
typedef char * Pointer;
int  _start(){
    Pointer  pvga = (Pointer)0xb8000;	//填充到显示内存的初始地址	
    for(int i = 0;i <= 0xffff;){
         //char: 0x3 ,color: 0x104
        *(pvga + i) = 0x03;i++;		 //显卡内存存填充颜色值，红色心形	
        *(pvga + i)= 0x104;i++;  
    }
    fin:
    	goto fin;
}
```

编译成目标文件

> $ gcc  bootloader1.c  -m32 -c   -nostdinc -Iinclude
>
>  -o  build/loader1.o


### 链接C语言程序

**1. 为什么要指定程序入口**
由于在保护模式下，我们默认加载到 0x00001000处执行代码。所以，下载需要做的是
1）指定当前c语言程序的入口地址为0x00001000
2）复制程序段的执行程序段到0x00001000


链接并指定程序入口

> ld -m elf_i386 -s -Ttext  0x00001000 build/loader1 -o loader1.bin

使用 

查看文件信息

> file loader1.bin


查看纯二进制内容

> xxd loader1.bin

查看反编译内容

> objdump -S loade1.bin

查看文件信息

> readelf -e  loader1.bin

