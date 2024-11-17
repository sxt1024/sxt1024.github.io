---
title: chap2.3_Makefile
date: 2018-11-21T11:27:28
lastmod: 2018-11-21
timezone: UTC+8
tags:
  - 操作系统
categories: 操作系统
draft: false
---



## Makefile


### 安装make

安装make

> sudo apt-get install make
>
> make -v


创建Makefile文件，并执行make命令

```makefile
# tools
PLATFORM=Linux
NASM=nasm
BOCHS=bochs
BXIMG=bximage

# args
boot=boot
build=build

target: prepare img  
	$(BOCHS) -f bochsrc.me


img: $(build)/astraos.img	
	@echo "build img completed"

$(build)/astraos.img:$(build)/boot.bin $(build)/loader.bin 
	$(BXIMAGE) -mode=create  -imgmode=flat -hd=16M   -q $(build)/astraos.img 
	sleep 1
	dd if=$(build)/boot.bin of=$(build)/astraos.img bs=512 count=1  conv=notrunc
	dd if=$(build)/loader.bin of=$(build)/astraos.img bs=512 count=1 seek=1 conv=notrunc

$(build)/%.bin: $(boot)/%.asm
	$(NASM) -f bin -o $(build)/$*.bin $(boot)/$*.asm	

prepare: $(build)
	@echo "prepare dir $(build)"
    ifeq ($(build), $(wildcard $(build)))
		@echo "build directory exist..."
    else
		mkdir -p $(build)
    endif

clean:
	@echo "clean dir $(build)"
	rm -rf $(build)/*

platform:
	@echo $(PLATFORM)

```

