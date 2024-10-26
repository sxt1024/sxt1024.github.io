解析ELF文件格式

## ELF格式

ELF文件是linux默认使用的执行文件的格式。
分为32位版本和64位版本：ELF32和ELF64。
我机器一直使用的是64位的机器，所以简单起见，32位不去管他了，用64位的进行。


首先是ELF定义的数据类型：


### 文件头elf header

### 程序头program header

### 字段section header

## 查看文件头


### 文件头elf header

类型及其所占的字节数大小如下：

	Elf64_Addr 		8 	8 Unsigned program address
	Elf64_Off 		8 	8 Unsigned ﬁle offset
	Elf64_Half 		2 	2 Unsigned medium integer
	Elf64_Word 		4 	4 Unsigned integer
	Elf64_Sword 	4 	4 Signed integer
	Elf64_Xword 	8 	8 Unsigned long integer
	Elf64_Sxword 	8 	8 Signed long integer
	unsigned char 	1 	1 Unsigned small integer


然后是文件头格式

	typedef struct
	{
	unsigned char e_ident[16]; /* ELF identification */ 16
	Elf64_Half e_type; /* Object file type */ 18
	Elf64_Half e_machine; /* Machine type */  20
	Elf64_Word e_version; /* Object file version */ 24
	Elf64_Addr e_entry; /* Entry point address */	32
	Elf64_Off e_phoff; /* Program header offset */	40
	Elf64_Off e_shoff; /* Section header offset */	48
	Elf64_Word e_flags; /* Processor-specific flags */
	Elf64_Half e_ehsize; /* ELF header size */
	Elf64_Half e_phentsize; /* Size of program header entry */
	Elf64_Half e_phnum; /* Number of program header entries */
	Elf64_Half e_shentsize; /* Size of section header entry */
	Elf64_Half e_shnum; /* Number of section header entries */
	Elf64_Half e_shstrndx; /* Section name string table index */
	} Elf64_Ehdr;


1）e_ident
e_ident是一个字节数组，定义个文件格式：
e_ident[4] 代表文件类型，1为32位2为64位
e_ident[5] 代表文件编码方式，1为小端法，2为大端法
e_ident[6] 代表文件版本
e_ident[7] 代表操作系统OS/ABI identiﬁcation：0为System V ABI，1为HP-UX operating system
				255为Standalone (embedded) application
e_ident[8] 代表ABI版本
e_ident[9] Start of padding bytes


2）e_type
e_type代表文件类型
0	ET_NONE  No ﬁle type
1 	ET_REL  Relocatable object ﬁle
2 	ET_EXEC  Executable ﬁle
3 	ET_DYN  Shared object ﬁle
4 	ET_CORE  Core ﬁle

### 程序头program header

程序头

	typedef struct
	{
	Elf64_Word p_type; /* Type of segment */       4
	Elf64_Word p_flags; /* Segment attributes */   8
	Elf64_Off p_offset; /* Offset in file */       16
	Elf64_Addr p_vaddr; /* Virtual address in memory */ 32
	Elf64_Addr p_paddr; /* Reserved */					40
	Elf64_Xword p_filesz; /* Size of segment in file */
	Elf64_Xword p_memsz; /* Size of segment in memory */
	Elf64_Xword p_align; /* Alignment of segment */
	}

### 字段section header

字段头格式

	typedef struct
	{
	Elf64_Word sh_name; /* Section name */
	Elf64_Word sh_type; /* Section type */
	Elf64_Xword sh_flags; /* Section attributes */
	Elf64_Addr sh_addr; /* Virtual address in memory */
	Elf64_Off sh_offset; /* Offset in file */
	Elf64_Xword sh_size; /* Size of section */
	Elf64_Word sh_link; /* Link to other section */
	Elf64_Word sh_info; /* Miscellaneous information */
	Elf64_Xword sh_addralign; /* Address alignment boundary */
	Elf64_Xword sh_entsize; /* Size of entries, if section has table */
	} Elf64_Shdr

Elf64_Phdr;


## 查看文件头

安装mingw后，可使用readelf命令查看elf文件格式：
查看readelf帮助可知：
查看文件头：	readelf -h 文件名
查看程序头：	readelf -l 文件名
查看字段头：	readelf -l 文件名


## 使用go解析

struct.go

	package main

	//ELF文件头
	type Elf64_Header struct {
		E_ident   [16]byte
		E_type    int16  //目标文件类型
		E_machine uint16 //硬件平台
		E_version uint32 //ELF头部版本
		E_entry   uint64 //程序进入点
		E_phoff   uint64 //程序头表偏移量
		E_shoff   uint64 //节头表偏移量
		E_flags   uint32 //处理器特定标志
		E_ehsize  uint16 //ELF头部长度

		E_phentsize uint16 //程序头表中一个条目的长度
		E_phnum     uint16 //程序头表条目数目

		E_shentsize uint16 //节头表中一个条目的长度
		E_shnum     uint16 //节头表条目个数
		E_shstrmdx  uint16 //节头表字符索引

	}

main.go

	package main

	import (
		"bytes"
		"encoding/binary"
		"fmt"
		"io"
		"io/ioutil"
	)

	type Elf64_Info struct {
		Filename    string
		Order       binary.ByteOrder
		Rd          io.Reader
		Elf64Header Elf64_Header
	}

	func (p *Elf64_Info) paser_Elf64Header() {

		elf64_Header := &Elf64_Header{}
		p.BgRead(&elf64_Header.E_ident)
		if elf64_Header.E_ident[4] == 0x1 {
			fmt.Println("64位版本")
		}
		if elf64_Header.E_ident[4] == 0x2 {
			fmt.Println("64位版本")
		}
		if elf64_Header.E_ident[5] == 0x1 {
			p.Order = binary.LittleEndian
			fmt.Println("小端法字节顺序")
		}
		if elf64_Header.E_ident[5] == 0x2 {
			p.Order = binary.BigEndian
			fmt.Println("大端法字节顺序")
		}

		p.BgRead(&elf64_Header.E_type)
		p.BgRead(&elf64_Header.E_machine)
		p.BgRead(&elf64_Header.E_version)
		p.BgRead(&elf64_Header.E_entry)
		p.BgRead(&elf64_Header.E_phoff)
		p.BgRead(&elf64_Header.E_shoff)
		p.BgRead(&elf64_Header.E_flags)
		p.BgRead(&elf64_Header.E_ehsize)

		p.BgRead(&elf64_Header.E_phentsize)
		p.BgRead(&elf64_Header.E_phnum)

		p.BgRead(&elf64_Header.E_shentsize)
		p.BgRead(&elf64_Header.E_shnum)
		p.BgRead(&elf64_Header.E_shstrmdx)

		fmt.Printf("目标文件格式 E_ident:%s\n", elf64_Header.E_ident)
		fmt.Printf("目标文件类型 E_type:%x\n", elf64_Header.E_type)
		fmt.Printf("硬件平台 E_machine:%x\n", elf64_Header.E_machine)
		fmt.Printf("ELF头部版本 E_version:%x\n", elf64_Header.E_version)
		fmt.Printf("程序进入点 E_entry:%x\n", elf64_Header.E_entry)
		fmt.Printf("程序头表偏移量 E_phoff:%x\n", elf64_Header.E_phoff)
		fmt.Printf("节头表偏移量 E_shoff:%x\n", elf64_Header.E_shoff)
		fmt.Printf("处理器特定标志 E_flags:%x\n", elf64_Header.E_flags)
		fmt.Printf("ELF头部长度 E_ehsize:%d\n", elf64_Header.E_ehsize)

		fmt.Printf("程序头表中一个条目的长度 E_phentsize:%d\n", elf64_Header.E_phentsize)
		fmt.Printf("程序头表条目数目 E_phnum:%d\n", elf64_Header.E_phnum)

		fmt.Printf("节头表中一个条目的长度 E_shentsize:%d\n", elf64_Header.E_shentsize)
		fmt.Printf("节头表条目个数 E_shnum:%d\n", elf64_Header.E_shnum)
		fmt.Printf("节头表字符索引 E_shstrmdx:%d\n", elf64_Header.E_shstrmdx)
	}
	func parse(filename string) {

		elf64_Info := &Elf64_Info{Filename: filename}
		buf, err := ioutil.ReadFile(elf64_Info.Filename)
		prd := bytes.NewReader(buf)
		elf64_Info.Rd = prd
		if err != nil {
			fmt.Println("read class file error:", err)
		}

		elf64_Info.paser_Elf64Header()
	}

	func (p *Elf64_Info) BgRead(inf interface{}) {
		if err := binary.Read(p.Rd, p.Order, inf); err != nil {
			fmt.Println("read err:", err)
			panic("read error：文件格式解析失败")
		}
	}

	func main() {
		parse("src")
	}
