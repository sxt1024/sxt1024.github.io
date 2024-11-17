---
title: chap3.4_光标寄存器
date: 2019-02-16T11:27:28
lastmod: 2019-02-16
timezone: UTC+8
tags:
  - 操作系统
categories:
  - 操作系统
draft: false
---



## 光标寄存器



光标位置信息位于显卡的2个光标寄存器中，总共16位，分为高位和低位存储。

例如。

标准VGA文本模式为 25 行 80 列

例如：
pos = 0 ：表示 位于 1 行 0 列
pos = 27 ：表示 位于 2 行 2 列
pos =1999 ： 最右下角



读取光标值

光标寄存器的端口号是 0x3d4 和 0x3d5

0x3d4用来写入变量
0x3d5用来获取光标的值


| 端口    | 值      | 读写  |
| ----- | ------ | --- |
| 0x3d4 | 0xe0   | 写入  |
| 0x3d5 | 光标高8位值 | 读取  |
| 0x3d4 | 0xf0   | 写入  |
| 0x3d5 | 光标低8位值 | 读取  |


获取光标和设置光标



```c
uint16 get_cursor();

void set_cursor(uint16 pos);

```

实现：
```c
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

```

说明：
pos代表设置位置，标准VGA文本模式为 25 行 80 列，pos取值从0到1999. 通过get_cursor函数获取光标现在的位置，通过set_cursor设置光标在25 行 80 列文本模式的位置。