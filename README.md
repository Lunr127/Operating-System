<div align = "center" style="font-size:40px"style="font-weight:bold" >目录</div>

[TOC]

## 1 实验环境的简介与搭建

### 1.1 VMware Workstation

​	VMware Workstation是一款虚拟机软件，可以帮助我们在Windows操作系统之下运行一个Linux操作系统。

​	在《ORANGE’S：一个操作系统的实现》一书的实验中，我们选择在ubuntu环境下运行bochs、nasm、gcc等工具，因此我们需要下载vmware运行ubuntu虚拟机。

### 1.2 Ubuntu

​	Ubuntu 是一个以桌面应用为主的 Linux 操作系统。

​	我首先在官网下载了在下载了相应的镜像文件ubuntu-22.04-desktop-amd64.iso后，在vmware中配置生成相应Ubuntu 64位虚拟机

​	在配置完资源之后，在创建的虚拟机界面中点击“开启此虚拟机”，进入系统设置界面。完成初始设置以及相关安装包下载后，进入Ubuntu主界面。

### 1.3 Bochs

​	在安装完虚拟机和操作系统之后，为了能够对编写的源代码进行编译和仿真，还必须下载相应的程序。

​	对于仿真程序，《ORANGE’S：一个操作系统的实现》一书中选择使用虚拟计算机Bochs进行仿真。

​	Bochs是一个x86硬件平台的开源模拟器。它可以模拟各种硬件的配置。Bochs模拟的是整个PC平台，包括I/O设备、内存和BIOS。

安装过程参考博客https://blog.csdn.net/Sunnil/article/details/79243192的教程：

​	首先，在终端安装如下几个包：

```
sudo apt-get install build-essential nasm
sudo apt-get install libx11-dev
sudo apt-get install xorg-dev
sudo apt-get install libgtk2.0-dev
sudo apt-get install bison
```

​	第二步：解压下载的bochs安装包：tar zxvf bochs-2.7.tar.gz

​	第三步：进入解压后的目录：cd bochs-2.7

​	第四步：再执行： ./configure --enable-debugger --enable-disasm

​	第五步：进行编译：sudo make

​	最后一步：安装，输入命令：sudo make install

​	安装完成后，在终端输入命令：bochs，显示如图1-1所示。

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143119272.png" alt="image-20230320143119272" style="zoom:50%;" />

<center>图 1-1 bochs安装成功后的显示</center>

### 1.4 NASM、GCC和GNU MAKE

​	在《ORANGE’S：一个操作系统的实现》一书中，所编写的操作系统是由汇编语言和C语言共同完成的，为了编译汇编语言，必须安装NASM程序，为了编译C语言，必须安装NASM程序，同时，还必须安装GNU Make，用于自动化编译和链接。

​	在Ubuntu中，已经预安装了GCC和NASM这两个程序，其余部分包，已经在1.3步骤中的第一步安装完成。

------





## 2 马上动手写一个最小的“操作系统”

### 2.1 实验1 十分钟完成操作系统

#### 2.1.1 关键代码

```
org	07c00h			
	mov	ax, cs
	mov	ds, ax
	mov	es, ax
	call	DispStr			
	jmp	$			
DispStr:
	mov	ax, BootMessage
	mov	bp, ax			
	mov	cx, 16			
	mov	ax, 01301h		
	mov	bx, 000ch		
	mov	dl, 0
	int	10h			
	ret
BootMessage:		db	"Hello, OS world!"
times 	510-($-$$)	db	0	
dw 	0xaa55				
```

#### 2.1.2 主要代码结构

代码主体框架：

告诉编译器程序加载到 07c00 处

使 ds 和 es 两个段寄存器指向与 cs 相同的段

调用 DispStr 子程序显示字符串

无限循环

 

DispStr 子程序：

设置 ES:BP = 串地址

设置 CX = 串长度

设置 AH = 13, AL = 01h

设置页号为 0(BH = 0) 黑底红字(BL = 0Ch,高亮)

10h 号中断

#### 2.1.3 代码执行流程图

```flow
st=>start: 代码主体
op1=>operation: 程序加载到 07c00 处
op2=>operation: 调用 DispStr 子程序显示字符串
op3=>operation: 显示字符串"Hello, OS world!"
e=>end: 无限循环

st->op1->op2->op3->e
```

#### 2.1.4 函数调用关系图

```mermaid
sequenceDiagram
	主程序->>主程序:程序加载到 07c00 处
	主程序->>DispStr():调用函数DispStr
	DispStr()-->>主程序:显示字符串"Hello, LiXinghan OS"
```



#### 2.1.5 调试过程

​	想要进行操作系统的调试，首先必须创建一个软盘映像，为了达成这个目标，必须使用bximage软件在打开bximgae后，依次输入1->fd->回车->回车，即可生成一个名为a.img的软盘映像，如下图2-1所示。

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143215611.png" alt="image-20230320143215611" style="zoom:50%;" />

<center>图2-1 bximage操作界面</center>

创建完软盘映像后，需要编译源代码，输入代码：
`nasm boot.asm -o boot.bin`
即可完成编译。
完成编译之后，我们需要使用软盘绝对扇区读写工具将这个文件写到一张空白软盘的第一个扇区，输入代码：
`dd if=boot.bin of=a.img bs=512 count=1 conv=notrunc`
即可完成读入。
一切准备就绪，紧接着需要编写配置文件，即告诉Bochs，你希望你的虚拟机是什么样子的，如内存多大、硬盘映像和软盘映像都是那些文件的内容，以下为Bochs配置文件bochsrc的示例：

```
###############################################################
# Configuration file for Bochs
###############################################################
# how much memory the emulated machine will have
megs: 32
# filename of ROM images
romimage:file=$BXSHARE/BIOS-bochs-latest
vgaromimage: file=$BXSHARE/VGABIOS-lgpl-latest
# what disk images will be used
floppya: 1_44=a.img, status=inserted
# choose the boot disk.
boot: a
# where do we send log messages?
# log: bochsout.txt
# disable the mouse
mouse: enabled=0
# enable key mapping, using US layout as default.
keyboard: keymap=$BXSHARE/keymaps/x11-pc-us.map
```

​	需要注意的是，书本上的配置文件是错误的，需要在romimage、vgaromimage、keyboard这三个地方进行修改。将romimage、vgaromimage改成自己虚拟机中文件的地址，keyboard的地方可以直接删去。
​	修改后的bochsrc配置文件如下所示：

```
megs:128

romimage:file=/home/zhaohan/bochs-2.7/bios/BIOS-bochs-latest

vgaromimage:file=/home/zhaohan/bochs-2.7/bios/VGABIOS-lgpl-latest  

floppya:1_44=a.img,status=inserted 

boot:floppy  
```

​	完成上述三个步骤之后，一切准备就绪，可以正式调试，输入命令：
`bochs -f bochsrc`
​	即可开始调试过程，按下回车键，会进入启动界面。
​	这时，需要再输入c（代表开始调试），再按下回车键，即会进入bochs页面，这时，在我们屏幕的左上角会出现一行红色的“Hello, OS world!”，即代表调试成功，调试结果如下图所示。

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143244527.png" alt="image-20230320143244527" style="zoom:50%;" />

#### 2.1.6 实验分析

在本实验中，我们使用汇编语言编写了一个最简单的操作系统，并且使用Bochs软件进行仿真。

但计算机电源打开时，它首先会加电自检（POST），然后寻找启动盘，如果是选择从软盘启动，计算机就会检查软盘的0面0磁道1扇区，如果发现它以**0xAA55**结束，那么BIOS就会认为它是一个引导扇区。当然，一个正确的引导扇区除了以**0xAA55**结束之外，还应该包含一段少于512字节的执行码。

而一旦BIOS发现了引导扇区，就会将这512字节的内容装载到内存地址**0000:7c00**处，然后跳转到0000:7c00处，**将控制权彻底交给这段引导代码**。至此为止，计算机不再有BIOS中固有的程序来控制，而是变为由操作系统的一部分来控制。

所以本实验的关键在于引导扇区的创建。即代码的最后部分：

```
times 	510-($-$$)	db	0	; 填充剩下的空间，使生成的二进制代码恰好为512字节
dw 	0xaa55				; 结束标志
```

------





## 3 保护模式

### 3.1 实验1 进入保护模式

#### 3.1.1 关键代码

​	**代码段1**

```
; ==========================================
; pmtest1.asm
; 编译方法：nasm pmtest1.asm -o pmtest1.bin
; ==========================================

%include	"pm.inc"	; 常量, 宏, 以及一些说明

org	07c00h
	jmp	LABEL_BEGIN

[SECTION .gdt]
; GDT
;                              段基址,       段界限     , 属性
LABEL_GDT:	   Descriptor       0,                0, 0           ; 空描述符
LABEL_DESC_CODE32: Descriptor       0, SegCode32Len - 1, DA_C + DA_32; 非一致代码段
LABEL_DESC_VIDEO:  Descriptor 0B8000h,           0ffffh, DA_DRW	     ; 显存首地址
; GDT 结束

GdtLen		equ	$ - LABEL_GDT	; GDT长度
GdtPtr		dw	GdtLen - 1	; GDT界限
		dd	0		; GDT基地址

; GDT 选择子
SelectorCode32		equ	LABEL_DESC_CODE32	- LABEL_GDT
SelectorVideo		equ	LABEL_DESC_VIDEO	- LABEL_GDT
; END of [SECTION .gdt]

[SECTION .s16]
[BITS	16]
LABEL_BEGIN:
	mov	ax, cs
	mov	ds, ax
	mov	es, ax
	mov	ss, ax
	mov	sp, 0100h

	; 初始化 32 位代码段描述符
	xor	eax, eax
	mov	ax, cs
	shl	eax, 4
	add	eax, LABEL_SEG_CODE32
	mov	word [LABEL_DESC_CODE32 + 2], ax
	shr	eax, 16
	mov	byte [LABEL_DESC_CODE32 + 4], al
	mov	byte [LABEL_DESC_CODE32 + 7], ah

	; 为加载 GDTR 作准备
	xor	eax, eax
	mov	ax, ds
	shl	eax, 4
	add	eax, LABEL_GDT		; eax <- gdt 基地址
	mov	dword [GdtPtr + 2], eax	; [GdtPtr + 2] <- gdt 基地址

	; 加载 GDTR
	lgdt	[GdtPtr]

	; 关中断
	cli

	; 打开地址线A20
	in	al, 92h
	or	al, 00000010b
	out	92h, al

	; 准备切换到保护模式
	mov	eax, cr0
	or	eax, 1
	mov	cr0, eax

	; 真正进入保护模式
	jmp	dword SelectorCode32:0	; 执行这一句会把 SelectorCode32 装入 cs,
					; 并跳转到 Code32Selector:0  处
; END of [SECTION .s16]


[SECTION .s32]; 32 位代码段. 由实模式跳入.
[BITS	32]

LABEL_SEG_CODE32:
	mov	ax, SelectorVideo
	mov	gs, ax			; 视频段选择子(目的)

	mov	edi, (80 * 11 + 79) * 2	; 屏幕第 11 行, 第 79 列。
	mov	ah, 0Ch			; 0000: 黑底    1100: 红字
	mov	al, 'P'
	mov	[gs:edi], ax

	; 到此停止
	jmp	$

SegCode32Len	equ	$ - LABEL_SEG_CODE32
; END of [SECTION .s32]


```

​	**代码段2**

```
; 描述符
; usage: Descriptor Base, Limit, Attr
;        Base:  dd
;        Limit: dd (low 20 bits available)
;        Attr:  dw (lower 4 bits of higher byte are always 0)
%macro Descriptor 3
	dw	%2 & 0FFFFh				; 段界限1
	dw	%1 & 0FFFFh				; 段基址1
	db	(%1 >> 16) & 0FFh			; 段基址2
	dw	((%2 >> 8) & 0F00h) | (%3 & 0F0FFh)	; 属性1 + 段界限2 + 属性2
	db	(%1 >> 24) & 0FFh			; 段基址3
%endmacro ; 共 8 字节
```

#### 3.1.2主要代码结构的流程图

```flow
st=>start: 实模式
op1=>operation: 初始化 32 位代码段描述符
op2=>operation: 将 GDT 的物理地址填充到 GdtPtr 中
op3=>operation: 加载 GDT并关中断
op4=>operation: 打开 A20 地址线
op5=>operation: 把寄存器 cr0 的第 0 位置为 1
e=>end: Jump进入保护模式

st->op1->op2->op3->op4->op5->e
```

<center>流程图-1 从实模式跳到保护模式</center>



```flow
st=>start: 实模式
op1=>operation: 跳入32位代码段
op2=>operation: 将视频段选择子 SelectorVideo 的地址赋给 gs
op3=>operation: 设置输出位置为屏幕 11 行 79 列
op4=>operation: 设置输出为黑底红字
op5=>operation: 往显存中 edi 偏移处（[gs:edi]地址处）写入 P
e=>end: 无限循环

st->op1->op2->op3->op4->op5->e
```

<center>流程图-2 实模式跳入32位代码段并在屏幕上输出</center>

#### 3.1.3 调试过程和实验结果

​	在之前的实验过程中，我们将文件写到了引导扇区运行，这样比较方便，但引导扇区的空间有限，只有512个字节，为了使我们的程序可以扩大可以选择先借用其他操作系统的引导扇区，这里《ORANGE’S：一个操作系统的实现》一书选择借用DOS操作系统，因此我们必须将程序编译成COM文件，然后用DOS来执行它。

​	为了这么做，第一步我们需要到Bochs官方网站下载一个FreeDos，解压后将其中的a.img复制到我们的工作目录中，并改名为freedos.img。

​	紧接着，我们需要使用bximage生成一个软盘映像，取名为pm.img。

​	首先我们需要一个软盘映像，我的选择是直接复制实验2的a.img，相关理由我会在最后的常见错误中详细解释。

​	再其次，我们需要修改我们的bochsrc，将其修改为一下内容，与实验2里的bochsrc相同。

​	创建完软盘映像后，需要编译源代码，输入代码：

`	nasm pmtest1.asm -o pmtest1.bin`

​	即可完成编译。

​	完成编译之后，我们需要使用软盘绝对扇区读写工具将这个文件写到一张空白软盘的第一个扇区，输入代码：

`dd if=pmtest1.bin of=a.img bs=512 count=1 conv=notrunc`

​	即可完成读入。

​	接下来，启动Bochs，按下回车键开始仿真，再输入c并且按下回车键开始调试，运行成功，如图3-1所示。 

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143317572.png" alt="image-20230320143317572" style="zoom:50%;" />

<center>图3-1 实验结果</center>

​	可以注意到，一个红色的字母“P”出现在了屏幕的右侧的中部，这代表调试成功，程序已经进入了保护模式。

#### 3.1.4 实验分析

​	在IA32下，CPU有两种工作模式：实模式和保护模式，但我们打开计算机时，开始时CPU是工作在实模式下的，但我们需要让其进入保护模式，发挥其巨大的寻址能力，并未强大的32位系提供硬件保障。

  在本实验中，我们在pm.inc文件中定义了全局描述表GDT，定义了相应的段描述符，并且通过代码由实模式进入了保护模式，最后在保护模式中，通过代码在屏幕右边中央打印了一个红色的“P”，代表实验成功。



### 3.2 实验2 保护模式进阶——从保护模式返回实模式

#### 3.2.1 关键代码

​	**代码段1 GDT、数据段和堆栈段**

```
%include	"pm.inc"	; 常量, 宏, 以及一些说明
org	0100h
	jmp	LABEL_BEGIN
[SECTION .gdt]
; GDT
;                            段基址,        段界限 , 属性
LABEL_GDT:         Descriptor    0,              0, 0         ; 空描述符
LABEL_DESC_NORMAL: Descriptor    0,         0ffffh, DA_DRW    ; Normal 描述符
LABEL_DESC_CODE32: Descriptor    0, SegCode32Len-1, DA_C+DA_32; 非一致代码段, 32
LABEL_DESC_CODE16: Descriptor    0,         0ffffh, DA_C      ; 非一致代码段, 16
LABEL_DESC_DATA:   Descriptor    0,      DataLen-1, DA_DRW    ; Data
LABEL_DESC_STACK:  Descriptor    0,     TopOfStack, DA_DRWA+DA_32; Stack, 32 位
LABEL_DESC_TEST:   Descriptor 0500000h,     0ffffh, DA_DRW
LABEL_DESC_VIDEO:  Descriptor  0B8000h,     0ffffh, DA_DRW    ; 显存首地址
; GDT 结束

GdtLen		equ	$ - LABEL_GDT	; GDT长度
GdtPtr		dw	GdtLen - 1	; GDT界限
		dd	0		; GDT基地址
; GDT 选择子
SelectorNormal		equ	LABEL_DESC_NORMAL	- LABEL_GDT
SelectorCode32		equ	LABEL_DESC_CODE32	- LABEL_GDT
SelectorCode16		equ	LABEL_DESC_CODE16	- LABEL_GDT
SelectorData		equ	LABEL_DESC_DATA		- LABEL_GDT
SelectorStack		equ	LABEL_DESC_STACK	- LABEL_GDT
SelectorTest		equ	LABEL_DESC_TEST		- LABEL_GDT
SelectorVideo		equ	LABEL_DESC_VIDEO	- LABEL_GDT
; END of [SECTION .gdt]
[SECTION .data1]	 ; 数据段
ALIGN	32
[BITS	32]
LABEL_DATA:
SPValueInRealMode	dw	0
; 字符串
PMMessage:		db	"In Protect Mode now. ^-^", 0	; 在保护模式中显示
OffsetPMMessage		equ	PMMessage - $$
StrTest:		db	"ABCDEFGHIJKLMNOPQRSTUVWXYZ", 0
OffsetStrTest		equ	StrTest - $$
DataLen			equ	$ - LABEL_DATA
; END of [SECTION .data1]
; 全局堆栈段
[SECTION .gs]
ALIGN	32
[BITS	32]
LABEL_STACK:
	times 512 db 0
TopOfStack	equ	$ - LABEL_STACK - 1
; END of [SECTION .gs]

32位代码段
LABEL_SEG_CODE32:
	mov	ax, SelectorData
	mov	ds, ax			; 数据段选择子
	mov	ax, SelectorTest
	mov	es, ax			; 测试段选择子
	mov	ax, SelectorVideo
	mov	gs, ax			; 视频段选择子
	mov	ax, SelectorStack
	mov	ss, ax			; 堆栈段选择子
	mov	esp, TopOfStack
	; 下面显示一个字符串
	mov	ah, 0Ch			; 0000: 黑底    1100: 红字
	xor	esi, esi
	xor	edi, edi
	mov	esi, OffsetPMMessage	; 源数据偏移
	mov	edi, (80 * 10 + 0) * 2	; 目的数据偏移。屏幕第 10 行, 第 0 列。
	cld
.1:
	lodsb
	test	al, al
	jz	.2
	mov	[gs:edi], ax
	add	edi, 2
	jmp	.1
.2:	; 显示完毕
	call	DispReturn
	call	TestRead
	call	TestWrite
	call	TestRead

	; 到此停止
	jmp	SelectorCode16:0
```

​	**代码段2 保护模式到实模式**

```
; 16 位代码段. 由 32 位代码段跳入, 跳出后到实模式
[SECTION .s16code]
ALIGN	32
[BITS	16]
LABEL_SEG_CODE16:
	; 跳回实模式:
	mov	ax, SelectorNormal
	mov	ds, ax
	mov	es, ax
	mov	fs, ax
	mov	gs, ax
	mov	ss, ax
	mov	eax, cr0
	and	al, 11111110b
	mov	cr0, eax
LABEL_GO_BACK_TO_REAL:
	jmp	0:LABEL_REAL_ENTRY	; 段地址会在程序开始处被设置成正确的值
Code16Len	equ	$ - LABEL_SEG_CODE16

; END of [SECTION .s16code]
```

​	**代码段3 保护模式到实模式的准备工作**

```
	mov	ax, cs
	mov	ds, ax
	mov	es, ax
	mov	ss, ax
	mov	sp, 0100h
	mov	[LABEL_GO_BACK_TO_REAL+3], ax
```

​	**代码段4 回到实模式**

```
LABEL_REAL_ENTRY:		; 从保护模式跳回到实模式就到了这里
	mov	ax, cs
	mov	ds, ax
	mov	es, ax
	mov	ss, ax
	mov	sp, [SPValueInRealMode]
	in	al, 92h		; `.
	and	al, 11111101b	;  | 关闭 A20 地址线
	out	92h, al		; /
	sti			; 开中断
	mov	ax, 4c00h	; `.
	int	21h		; /  回到 DOS
```

#### 3.2.2 主要代码结构流程图

```flow
st=>start: 保护模式
op1=>operation: 跳入16位代码段[SECTION .s16code]
op2=>operation: 把选择子 SelectorNormal 加载到 ds、es、ss
op3=>operation: 清 cr0 的 PE 位，跳回实模式
op4=>operation: 跳转到 LABEL_REAL_ENTRY
op5=>operation: 程序重设各个段寄存器值，使 ds、es、ss 指向 as
op6=>operation: 恢复 sp 的值
op7=>operation: 打开中断
e=>end: 实模式状态下运行

st->op1->op2->op3->op4->op5->op6->op7->e
```

<center>流程图-1 保护模式跳回到实模式</center>

#### 3.2.3 调试过程

​	在工作目录中打开终端，输入bochs
​	等到freedos加载成功后，格式化B盘，输入format b：
​	格式化完成后，将pmtest2.asm编译为com文件：
​	`nasm pmtest2.asm -o pmtest2.com`
​	紧接着，我们需要输入一下三条指令，将pmtest1.com复制到虚拟软盘pm.img中，所需要的指令如下：

```
sudo mount -o looop pm.img /mnt/floppy
sudo cp pmtest2.com /mnt/floppy
sudp umount /mnt/floppy
```

​	输入以下三条指令后，下一步我们需要重新回到FreeDos中（即原先打开的Bochs虚拟机），并且输入如下命令：
​	`B:\pmtest2.com`
​	这样子，pmtest2.com运行成功，如下图所示。

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143350428.png" alt="image-20230320143350428" style="zoom:67%;" />

可以看到，程序打印出了两行数字，第一行全部是0，说明开始内存5MB处都是0，而下一行已经变成了41、42、43...，说明写操作成功（十六进制的41、42、43...48正是A、B、C...H）。

同时可以看到，程序执行结束后没有进入死循环，而是重新出现了DOS提示符，这说明我们已经重新回到了实模式下的DOS。

#### 3.2.4 实验分析

在上一个实验中，我们成功的进入保护模式，但只打印了一个红色的P，而没有体验到保护模式的便利。

在保护模式中，CPU的寻址空间可以达到4GB，因此在本实验中，我们试验了读写大地址内存。

具体的做法如下，在前面程序的基础上，我们重新建立了一个以5MB为基址的段（远远超过实模式下1MB的界限），随后先读出开始处8字节的内容，然后写入一个字符串，再从中读出8字节，如果读写成功的话，两次读出的内容不同，而且第二次读出的内容应该是我们写进的字符串——事实的确如此。

最后，则是返回实模式，为了这么做，我们必须加载一个合适的描述符选择自到有关段寄存器，以使对应的段描述符高速缓存寄存器中含有合适的段界限和属性。而且，我们不能从32位代码段返回实模式，只能从16位代码段中返回，这是因为无法实现从32位代码段返回时cs高速缓存寄存器中的属性符合实模式的要求（实模式不能改变段属性）。



### 3.3 实验3 局部描述符表LDT

#### 3.3.1 关键代码

```
[SECTION .gdt]
...
LABEL_DESC_LDT:    Descriptor       0,        LDTLen - 1, DA_LDT	; LDT
...
SelectorLDT		equ	LABEL_DESC_LDT		- LABEL_GDT
...
[SECTION .s16]
...
; 初始化 LDT 在 GDT 中的描述符
	xor	eax, eax
	mov	ax, ds
	shl	eax, 4
	add	eax, LABEL_LDT
	mov	word [LABEL_DESC_LDT + 2], ax
	shr	eax, 16
	mov	byte [LABEL_DESC_LDT + 4], al
	mov	byte [LABEL_DESC_LDT + 7], ah
	; 初始化 LDT 中的描述符
	xor	eax, eax
	mov	ax, ds
	shl	eax, 4
	add	eax, LABEL_CODE_A
	mov	word [LABEL_LDT_DESC_CODEA + 2], ax
	shr	eax, 16
	mov	byte [LABEL_LDT_DESC_CODEA + 4], al
	mov	byte [LABEL_LDT_DESC_CODEA + 7], ah
...
[SECTION .s32]; 32 位代码段. 由实模式跳入.
...
; Load LDT
	mov	ax, SelectorLDT
	lldt	ax
	jmp	SelectorLDTCodeA:0	; 跳入局部任务
...
; LDT
[SECTION .ldt]
ALIGN	32
LABEL_LDT:
;                            段基址       段界限      属性
LABEL_LDT_DESC_CODEA: Descriptor 0, CodeALen - 1, DA_C + DA_32 ; Code, 32 位

LDTLen		equ	$ - LABEL_LDT

; LDT 选择子
SelectorLDTCodeA	equ	LABEL_LDT_DESC_CODEA	- LABEL_LDT + SA_TIL
; END of [SECTION .ldt]


; CodeA (LDT, 32 位代码段)
[SECTION .la]
ALIGN	32
[BITS	32]
LABEL_CODE_A:
	mov	ax, SelectorVideo
	mov	gs, ax			; 视频段选择子(目的)
	mov	edi, (80 * 12 + 0) * 2	; 屏幕第 10 行, 第 0 列。
	mov	ah, 0Ch			; 0000: 黑底    1100: 红字
	mov	al, 'L'
	mov	[gs:edi], ax

	; 准备经由16位代码段跳回实模式
	jmp	SelectorCode16:0
CodeALen	equ	$ - LABEL_CODE_A
; END of [SECTION .la]
```

#### 3.3.2 代码主要结构流程图

```flow
st=>start: 实模式
op1=>operation: 跳入32位代码段[SECTION .s32code]
op2=>operation: 初始化 ds、gs、ss
op3=>operation: 显示一个字符串
op4=>operation: 显示回车
op5=>operation: 加载 LDT
e=>end: 跳入局部任务

st->op1->op2->op3->op4->op5->e
```

<center>流程图-1 实模式加载LDT跳入局部任务</center>

#### 3.3.4 函数调用图

```mermaid
sequenceDiagram
	实模式->>实模式: 跳入32位代码段[SECTION .s32code]
	实模式->>DispStr(): 调用函数DispStr()
	DispStr()-->>实模式: 显示字符'L'
	实模式->>DisReturn(): 调用函数DispReturn()
	DisReturn()-->>实模式: 模拟回车显示
	实模式->>Load LDT(): 调用函数Load LDT()
	Load LDT()->>实模式: 加载LDT
	实模式->>实模式: 跳入局部任务
```

#### 3.3.5 实验结果

本实验LDT中的代码段非常的简单，只是打印一个字符L，因此，在[SECTION .s32]中打印完“In Protect Mode Now.”这个字符串之后，一个红色的字符L将会出现。

可以看到，在下图中，的确出现了“In Protect Mode Now.”字符串和一个红色的L，这说明我们的程序是成功与正确的。

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143419421.png" alt="image-20230320143419421" style="zoom:67%;" />

#### 3.3.6 实验分析

在本实验中，我们学习并了解了LDT。简单地说，它是一种描述符表，与GDT差不多，只不过它的段选择子的T1位必须置为1。在运用它时，必须先用lldt指令加载ldtr，ldtr的操作数是GDT中用来描述LDT的段描述符。

------





## 4 让操作系统走进保护模式

### 4.1 实验1 DOS可以识别的引导盘

#### 4.1.1 关键代码

​	**代码段1 BPB**

```
	jmp short LABEL_START		; Start to boot.
	nop				; 这个 nop 不可少
	; 下面是 FAT12 磁盘的头
	BS_OEMName	DB 'ForrestY'	; OEM String, 必须 8 个字节
	BPB_BytsPerSec	DW 512		; 每扇区字节数
	BPB_SecPerClus	DB 1		; 每簇多少扇区
	BPB_RsvdSecCnt	DW 1		; Boot 记录占用多少扇区
	BPB_NumFATs	DB 2		; 共有多少 FAT 表
	BPB_RootEntCnt	DW 224		; 根目录文件数最大值
	BPB_TotSec16	DW 2880		; 逻辑扇区总数
	BPB_Media	DB 0xF0		; 媒体描述符
	BPB_FATSz16	DW 9		; 每FAT扇区数
	BPB_SecPerTrk	DW 18		; 每磁道扇区数
	BPB_NumHeads	DW 2		; 磁头数(面数)
	BPB_HiddSec	DD 0		; 隐藏扇区数
	BPB_TotSec32	DD 0		; wTotalSectorCount为0时这个值记录扇区数
	BS_DrvNum	DB 0		; 中断 13 的驱动器号
	BS_Reserved1	DB 0		; 未使用
	BS_BootSig	DB 29h		; 扩展引导标记 (29h)
	BS_VolID	DD 0		; 卷序列号
	BS_VolLab	DB 'OrangeS0.02'; 卷标, 必须 11 个字节
	BS_FileSysType	DB 'FAT12   '	; 文件系统类型, 必须 8个字节
```

​	**代码段2 代码主体**

```
LABEL_START:
	mov	ax, cs
	mov	ds, ax
	mov	es, ax
	Call	DispStr			; 调用显示字符串例程
	jmp	$			; 无限循环
DispStr:
	mov	ax, BootMessage
	mov	bp, ax			; ES:BP = 串地址
	mov	cx, 16			; CX = 串长度
	mov	ax, 01301h		; AH = 13,  AL = 01h
	mov	bx, 000ch		; 页号为0(BH = 0) 黑底红字(BL = 0Ch,高亮)
	mov	dl, 0
	int	10h			; int 10h
	ret
BootMessage:		db	"Hello, OS world!"
times 	510-($-$$)	db	0	; 填充剩下的空间，使生成的二进制代码恰好为512字节
dw 	0xaa55				; 结束标志
```

#### 4.1.2 代码主要结构流程图

```flow
st=>start: 代码主体
op1=>operation: 引导扇区加入BPB等头信息
op2=>operation: 显示一个字符串
e=>end: 结束任务

st->op1->op2->e
```

#### 4.1.3 函数调用图

```mermaid
sequenceDiagram
	代码主体->>SetBPB(): 调用函数SetBPB()
	SetBPB()-->>代码主体: 为引导扇区加入BPB等头信息
	代码主体->>DispStr(): 调用函数DispStr()
	DispStr()-->>代码主体: 显示字符串"Hello, LiXinghan OS"
```

#### 4.1.4 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143439464.png" alt="image-20230320143439464" style="zoom:67%;" />![image-20230320143505893](E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143505893.png)

​	可以看到，调试结果同实验2.1，只不过这时我们已经为我们的引导扇区加入了BPB等头信息，使其可以被微软识别。

#### 4.1.5 实验分析

本实验主要涉及的分析如下：

**1）FAT 12**

FAT12是DOS时代就开始使用的文件系统（File System），直到现在仍然在软盘上使用。几乎所有的文件系统都会讲磁盘分为若干层次以方便组织和管理，这些层次包括：

扇区（Sector）：磁盘上的最小数据单元。

簇（Cluster）：一个或多个扇区。

分区（Partition）：通常指整个文件系统。

同时，FAT12格式分为若干个扇区。引导扇区是整个磁盘的第0个扇区，在这个扇区中有一个很重要的数据结构叫做BPB（BIOS Parameter Block），说明FAT的内容，之后则依次是FAT1、FAT2、根目录区及数据区。

**2） BPB**

  引导扇区需要有BPB等头信息才能被识别，在我们的程序开头必须加上它。



### 4.2 实验2 一个最简单的Loader

#### 4.2.1 关键代码

​	**代码1 最简单的Loader**

```
	org	0100h
	mov	ax, 0B800h
	mov	gs, ax
	mov	ah, 0Fh				; 0000: 黑底    1111: 白字
	mov	al, 'L'
	mov	[gs:((80 * 0 + 39) * 2)], ax	; 屏幕第 0 行, 第 39 列。
	jmp	$				; 到此停住
```

​	**代码2 读软盘扇区**

```
ReadSector:
	; ----------------------------------------------------------------------
	; 怎样由扇区号求扇区在磁盘中的位置 (扇区号 -> 柱面号, 起始扇区, 磁头号)
	; ----------------------------------------------------------------------
	; 设扇区号为 x
	;                           ┌ 柱面号 = y >> 1
	;       x           ┌ 商 y ┤
	; -------------- => ┤      └ 磁头号 = y & 1
	;  每磁道扇区数     │
	;                   └ 余 z => 起始扇区号 = z + 1
	push	bp
	mov	bp, sp
	sub	esp, 2 ; 辟出两个字节的堆栈区域保存要读的扇区数: byte [bp-2]
	mov	byte [bp-2], cl
	push	bx			; 保存 bx
	mov	bl, [BPB_SecPerTrk]	; bl: 除数
	div	bl			; y 在 al 中, z 在 ah 中
	inc	ah			; z ++
	mov	cl, ah			; cl <- 起始扇区号
	mov	dh, al			; dh <- y
	shr	al, 1			; y >> 1 (y/BPB_NumHeads)
	mov	ch, al			; ch <- 柱面号
	and	dh, 1			; dh & 1 = 磁头号
	pop	bx			; 恢复 bx
	; 至此, "柱面号, 起始扇区, 磁头号" 全部得到
	mov	dl, [BS_DrvNum]		; 驱动器号 (0 表示 A 盘)
.GoOnReading:
	mov	ah, 2			; 读
	mov	al, byte [bp-2]		; 读 al 个扇区
	int	13h
	jc	.GoOnReading		; 如果读取错误 CF 会被置为 1,这时就不停地读, 直到正确为止
	add	esp, 2
	pop	bp
	ret
```

​	**代码3 寻找Loader**

```
xor	ah, ah	; `.
	xor	dl, dl	;  |  软驱复位
	int	13h	; /
; 下面在 A 盘的根目录寻找 LOADER.BIN
	mov	word [wSectorNo], SectorNoOfRootDirectory
LABEL_SEARCH_IN_ROOT_DIR_BEGIN:
	cmp	word [wRootDirSizeForLoop], 0	;  `. 判断根目录区是不是已经读完
	jz	LABEL_NO_LOADERBIN		;  /  如果读完表示没有找到 LOADER.BIN
	dec	word [wRootDirSizeForLoop]	; /
	mov	ax, BaseOfLoader
	mov	es, ax			; es <- BaseOfLoader
	mov	bx, OffsetOfLoader	; bx <- OffsetOfLoader
	mov	ax, [wSectorNo]		; ax <- Root Directory 中的某 Sector 号
	mov	cl, 1
	call	ReadSector
	mov	si, LoaderFileName	; ds:si -> "LOADER  BIN"
	mov	di, OffsetOfLoader	; es:di -> BaseOfLoader:0100
	cld
	mov	dx, 10h
LABEL_SEARCH_FOR_LOADERBIN:
	cmp	dx, 0				   ; `. 循环次数控制,
	jz	LABEL_GOTO_NEXT_SECTOR_IN_ROOT_DIR ;  / 如果已经读完了一个 Sector,
	dec	dx				   ; /  就跳到下一个 Sector
	mov	cx, 11
LABEL_CMP_FILENAME:
	cmp	cx, 0
	jz	LABEL_FILENAME_FOUND	; 如果比较了 11 个字符都相等, 表示找到
	dec	cx
	lodsb				; ds:si -> al
	cmp	al, byte [es:di]
	jz	LABEL_GO_ON
	jmp	LABEL_DIFFERENT		; 只要发现不一样的字符就表明本 DirectoryEntry
					; 不是我们要找的 LOADER.BIN
LABEL_GO_ON:
	inc	di
	jmp	LABEL_CMP_FILENAME	; 继续循环
LABEL_DIFFERENT:
	and	di, 0FFE0h		; else `. di &= E0 为了让它指向本条目开头
	add	di, 20h			;       |
	mov	si, LoaderFileName	;       | di += 20h  下一个目录条目
	jmp	LABEL_SEARCH_FOR_LOADERBIN;    /
LABEL_GOTO_NEXT_SECTOR_IN_ROOT_DIR:
	add	word [wSectorNo], 1
	jmp	LABEL_SEARCH_IN_ROOT_DIR_BEGIN
LABEL_NO_LOADERBIN:
	mov	dh, 2			; "No LOADER."
	call	DispStr			; 显示字符串
%ifdef	_BOOT_DEBUG_
	mov	ax, 4c00h		; `.
	int	21h			; /  没有找到 LOADER.BIN, 回到 DOS
%else
	jmp	$			; 没有找到 LOADER.BIN, 死循环在这里
%endif
LABEL_FILENAME_FOUND:			; 找到 LOADER.BIN 后便来到这里继续
	jmp	$			; 代码暂时停在这里
```

#### 4.2.2 代码主要结构流程图

```flow
st=>start: 代码主体
op1=>operation: 初始化堆栈
op2=>operation: 在 A 盘的根目录寻找 LOADER.BIN
op3=>operation: 加载 LOADER.BIN
e=>end: 结束任务

st->op1->op2->op3->e
```

<center>流程图-1 加载LOADER</center>



```flow
st=>start: LOADER.BIN
op1=>operation: 令 gs 指向 0B8h00h 处
op2=>operation: 设置字符输出结构
op3=>operation: 显示一个字符串
e=>end: 结束任务

st->op1->op2->op3->e
```

<center>流程图-2 LOADER.BIN执行</center>

#### 4.2.3 调试过程

​	第一步，编译文件生成二进制文件：

```
nasm boot.asm -o boot.bin
nasm loader.asm -o loader.bin
```

​	第二步，先用bximage生成一个软盘印象，然后输入代码将loader写入软盘：

```
dd if=boot.bin of=a.img bs=512 count=1 conv=notrunc
sudo mount -o loop a.img /mnt/floppy
sudo cp loader.bin /mnt/floppy -v
sudo umount /mnt/floppy
```

​	第三步，运行bochs，但我们看不到任何现象，因为我们仅仅是找到Loader.bin就停在在那里。

#### 4.2.4 实验结果

![image-20230320143505893](E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143505893.png)

#### 4.2.5 实验分析

​	在本实验中，我们实现了将Loader加载进入内核，这用到了BIOS中断中的int 13h，并且编写了读扇区和寻找Loader.bin的代码。

下一步我们就需要将Loader.bin加载如内存，并且移交统治权。



### 4.3 实验3 向Loader交出控制权

#### 4.3.1 关键代码

​	**代码1 由扇区号求FAT项的值**

```
GetFATEntry:
	push	es
	push	bx
	push	ax
	mov	ax, BaseOfLoader; `.
	sub	ax, 0100h	;  | 在 BaseOfLoader 后面留出 4K 空间用于存放 FAT
	mov	es, ax		; /
	pop	ax
	mov	byte [bOdd], 0
	mov	bx, 3
	mul	bx			; dx:ax = ax * 3
	mov	bx, 2
	div	bx			; dx:ax / 2  ==>  ax <- 商, dx <- 余数
	cmp	dx, 0
	jz	LABEL_EVEN
	mov	byte [bOdd], 1
LABEL_EVEN:;偶数
	; 现在 ax 中是 FATEntry 在 FAT 中的偏移量,下面来
	; 计算 FATEntry 在哪个扇区中(FAT占用不止一个扇区)
	xor	dx, dx			
	mov	bx, [BPB_BytsPerSec]
	div	bx ; dx:ax / BPB_BytsPerSec
		   ;  ax <- 商 (FATEntry 所在的扇区相对于 FAT 的扇区号)
		   ;  dx <- 余数 (FATEntry 在扇区内的偏移)。
	push	dx
	mov	bx, 0 ; bx <- 0 于是, es:bx = (BaseOfLoader - 100):00
	add	ax, SectorNoOfFAT1 ; 此句之后的 ax 就是 FATEntry 所在的扇区号
	mov	cl, 2
	call	ReadSector ; 读取 FATEntry 所在的扇区, 一次读两个, 避免在边界
			   ; 发生错误, 因为一个 FATEntry 可能跨越两个扇区
	pop	dx
	add	bx, dx
	mov	ax, [es:bx]
	cmp	byte [bOdd], 1
	jnz	LABEL_EVEN_2
	shr	ax, 4
LABEL_EVEN_2:
	and	ax, 0FFFh

LABEL_GET_FAT_ENRY_OK:

	pop	bx
	pop	es
	ret
```

​	**代码2 加载Loader**

```
LABEL_FILENAME_FOUND:			; 找到 LOADER.BIN 后便来到这里继续
	mov	ax, RootDirSectors
	and	di, 0FFE0h		; di -> 当前条目的开始
	add	di, 01Ah		; di -> 首 Sector
	mov	cx, word [es:di]
	push	cx			; 保存此 Sector 在 FAT 中的序号
	add	cx, ax
	add	cx, DeltaSectorNo	; cl <- LOADER.BIN的起始扇区号(0-based)
	mov	ax, BaseOfLoader
	mov	es, ax			; es <- BaseOfLoader
	mov	bx, OffsetOfLoader	; bx <- OffsetOfLoader
	mov	ax, cx			; ax <- Sector 号

LABEL_GOON_LOADING_FILE:
	push	ax			; `.
	push	bx			;  |
	mov	ah, 0Eh		;  | 每读一个扇区就在 "Booting  " 后面
	mov	al, '.'		;  | 打一个点, 形成这样的效果:
	mov	bl, 0Fh		;  | Booting ......
	int	10h			   ;  |
	pop	bx			   ;  |
	pop	ax			   ; /
	mov	cl, 1
	call	ReadSector
	pop	ax			; 取出此 Sector 在 FAT 中的序号
	call	GetFATEntry
	cmp	ax, 0FFFh
	jz	LABEL_FILE_LOADED
	push	ax			; 保存 Sector 在 FAT 中的序号
	mov	dx, RootDirSectors
	add	ax, dx
	add	ax, DeltaSectorNo
	add	bx, [BPB_BytsPerSec]
	jmp	LABEL_GOON_LOADING_FILE
LABEL_FILE_LOADED:
```

​	**代码3 跳入Loader之前显示字符串**

```
	mov	dh, 1			; "Ready."
	call	DispStr			; 显示字符串
```

#### 4.3.2 代码主要结构流程图

```flow
st=>start: Boot代码主体
op1=>operation: 初始化堆栈
op2=>operation: 遍历根目录取所有的扇区，将每一个扇区加载到内存
op3=>operation: 每读一个扇区就在 "Booting " 后面打一个点
op4=>operation: 寻找文件名为 Loader.bin 的条目
op5=>operation: 显示字符串“Ready”
op6=>operation: 跳转到已加载到内存中的 LOADER.BIN 的开始处
op7=>operation: 显示字符'L'
e=>end: 结束任务

st->op1->op2->op3->op4->op5->op6->op7->e
```

<center>流程图-1 Boot从扇区找到Loader并交出控制权</center>

#### 4.3.3 函数调用图

```mermaid
sequenceDiagram
	Boot->>ReadSector(): 调用函数ReadSector()
	ReadSector()-->>Boot: 将扇区加载到内存
	Boot->>GetFATEntry(): 调用函数GetFATEntry()
	GetFATEntry()-->>Boot: 加载包含Loader.bin条目的扇区
	Boot->>Boot: 向Loader交出控制权
```

#### 4.3.4 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143530648.png" alt="image-20230320143530648" style="zoom:67%;" />

​	可以看到，屏幕上出现了Booting和Ready的字样，代表程序进入引导扇区和进入引导扇区结束，紧接着，程序跳转入Loader，在屏幕最上方的正中间打印了一个白色的“L”字，调试结果说明实验成功。

#### 4.3.5 实验分析

​	在本实验中，我们实现了Boot的代码，使其可以从软盘中读出Loader.bim文件，并且加载入内核，同时移交控制权。
​	在目前，只要一个.COM文件中不含有DOS系统调用，我们就可以将它当成Loader使用，也就是说，我们现在的程序已经可以被看做是一个在保护模式下执行的“操作系统”了。
​	但是，我们目前的Loader仅仅只是一个Loader，它不是操作系统内核，也不能当做操作系统内核，因为我们希望我们的操作系统内核至少可以在Linux下用GCC编译链接，摆脱汇编语言，而Loader则至少要做两件事：将内核Kernel加载入内核、跳入保护模式，这也是下一章的内容之一。

------





## 5 内核雏形

### 5.1 实验1 在Linux下用汇编写Hello World

#### 5.1.1 关键代码

```
; 编译链接方法
; (ld 的‘-s’选项意为“strip all”)
; $ nasm -f elf hello.asm -o hello.o
; $ ld -s hello.o -o hello
; $ ./hello
; Hello, world!
; $
[section .data]	; 数据在此
strHello	db	"Hello, world!", 0Ah
STRLEN		equ	$ - strHello
[section .text]	; 代码在此
global _start	; 我们必须导出 _start 这个入口，以便让链接器识别
_start:
	mov	edx, STRLEN
	mov	ecx, strHello
	mov	ebx, 1
	mov	eax, 4		; sys_write
	int	0x80		; 系统调用
	mov	ebx, 0
	mov	eax, 1		; sys_exit
	int	0x80		; 系统调用
```

#### 5.1.2 代码主要结构流程图

```flow
st=>start: _start函数
op1=>operation: 设置字符串长度
op2=>operation: 设置要显示的字符串
op3=>operation: 设置文件描述符(stdout)
op4=>operation: 系统调用输出字符串(sys_write)
e=>end: 退出函数

st->op1->op2->op3->op4->e
```

<center>流程图-1 汇编代码输出字符串</center>

#### 5.1.3 调试过程

​	依次输入以下指令，完成调试：

```
nasm -f elf hello.asm -o hello.o
ld -m elf_i386 -s hello.o -o hello
./hello
```

#### 5.1.4 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143549283.png" alt="image-20230320143549283" style="zoom:67%;" />

#### 5.1.5 实验分析

​	在本实验中，我们的程序定义了两个节（Section），一个放数据，一个放代码。在代码中值得注意的一点是，入口点默认是“_start”，我们不但要定义它，而且要通过global这个关键字将它导出，这样连接程序才能找到它，至于代码本身，则利用了两个系统调用。



### 5.2 实验2 再进一步，汇编和C同步使用

#### 5.2.1 关键代码

​	**代码1 汇编调用函数**

```
extern choose	; int choose(int a, int b);
[section .data]	; 数据在此
num1st		dd	3
num2nd		dd	4
[section .text]	; 代码在此
global _start	; 我们必须导出 _start 这个入口，以便让链接器识别。
global myprint	; 导出这个函数为了让 bar.c 使用
_start:
	push	dword [num2nd]	; `.
	push	dword [num1st]	;  |
	call	choose		;  | choose(num1st, num2nd);
	add	esp, 8		; /
	mov	ebx, 0
	mov	eax, 1		; sys_exit
	int	0x80		; 系统调用
; void myprint(char* msg, int len)
myprint:
	mov	edx, [esp + 8]	; len
	mov	ecx, [esp + 4]	; msg
	mov	ebx, 1
	mov	eax, 4		; sys_write
	int	0x80		; 系统调用
	ret
```

​	**代码2 C语言函数**

```
void myprint(char* msg, int len);
int choose(int a, int b)
{
	if(a >= b)
{
myprint("the 1st one\n", 13);
}
	else
{
myprint("the 2nd one\n", 13);
}
	return 0;
```

#### 5.2.2 代码主要结构流程图

```flow
st=>start: 汇编主体
op1=>operation: 把 num2、num1 先后压入栈
op2=>operation: 栈指针加 8
op3=>operation: 调用外部C函数 choose
e=>end: 退出代码

st->op1->op2->op3->e
```

<center>流程图-1 汇编代码调用C函数</center>

#### 5.2.3 函数调用图

```mermaid
sequenceDiagram
	汇编主体->>_start(): 调用内部函数_start()
	_start()-->>choose(): 调用外部C函数choose()
	choose()->>myprint(): 调用函数myprint()
	myprint()-->>sys_write(): 系统调用sys_write()
	sys_write()->>汇编主体: 屏幕显示字符串
```

#### 5.2.4 调试过程

输入以下指令以进行编译链接和执行：

```
nasm -f elf -o foo.o foo.asm
gcc -m32 -c -o bar.o bar.c
ld  -m elf_i386 -s -o foobar foo.o bar.o
./foobar
```

#### 5.2.5 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143610688.png" alt="image-20230320143610688" style="zoom:67%;" />

​	在本实验中，定义了num1=3，num2=4，程序输出大的那个数的结果，可以看到，程序输出了“the 2nd one”的字样，成功的完成了任务。

#### 5.2.6 实验分析

​	在本实验中，我们实现了汇编语言与c语言的共同使用，将它们编译为elf格式的文件并最终执行。
​	其中，我们的源代码包含两个文件：foo.asm和bar.c。程序入口_start在foo.asm中，一开始程序会调用bar.c中的函数choose，choose()会比较传入的两个参数，根据比较结果的不同打印出不同的字符串。打印字符串的工作是有foo.asm中的函数myprint()来完成的。
​	其中，在c语言中调用汇编语言的函数，需要在汇编语言中把该函数定义为global，而如果汇编语言要用到C语言中的函数，需要将相应的函数定义为extern，同时参数遵循C调用约定，后面的参数先入栈，并且由调用者清理堆栈。
​	下面再谈ELF（Executable and Linkable Format）文件格式，ELF文件由四个部分组成：ELF头（ELF header）、程序头表（Progarm header table）、节（Sections）和节头标（Section header table），定义了有关ELF格式的有关信息，并且可以使汇编语言和C语言串联起来。



### 5.3 实验3 从Loader到内核

#### 5.3.1 关键代码

​	**代码1 共享常量所在的文件fat12hdr.inc**

```
; FAT12 磁盘的头
; ----------------------------------------------------------------------
BS_OEMName	DB 'ForrestY'	; OEM String, 必须 8 个字节
BPB_BytsPerSec	DW 512		; 每扇区字节数
BPB_SecPerClus	DB 1		; 每簇多少扇区
BPB_RsvdSecCnt	DW 1		; Boot 记录占用多少扇区
BPB_NumFATs	DB 2		; 共有多少 FAT 表
BPB_RootEntCnt	DW 224		; 根目录文件数最大值
BPB_TotSec16	DW 2880		; 逻辑扇区总数
BPB_Media	DB 0xF0		; 媒体描述符
BPB_FATSz16	DW 9		; 每FAT扇区数
BPB_SecPerTrk	DW 18		; 每磁道扇区数
BPB_NumHeads	DW 2		; 磁头数(面数)
BPB_HiddSec	DD 0		; 隐藏扇区数
BPB_TotSec32	DD 0		; 如果 wTotalSectorCount 是 0 由这个值记录扇区数
BS_DrvNum	DB 0		; 中断 13 的驱动器号
BS_Reserved1	DB 0		; 未使用
BS_BootSig	DB 29h		; 扩展引导标记 (29h)
BS_VolID	DD 0		; 卷序列号
BS_VolLab	DB 'OrangeS0.02'; 卷标, 必须 11 个字节
BS_FileSysType	DB 'FAT12   '	; 文件系统类型, 必须 8个字节  
;------------------------------------------------------------------------
; -------------------------------------------------------------------------
; 基于 FAT12 头的一些常量定义，如果头信息改变，下面的常量可能也要做相应改变
; -------------------------------------------------------------------------
; BPB_FATSz16
FATSz			equ	9
; 根目录占用空间:
; RootDirSectors = ((BPB_RootEntCnt*32)+(BPB_BytsPerSec–1))/BPB_BytsPerSec
; 但如果按照此公式代码过长，故定义此宏
RootDirSectors		equ	14
; Root Directory 的第一个扇区号	= BPB_RsvdSecCnt + (BPB_NumFATs * FATSz)
SectorNoOfRootDirectory	equ	19
; FAT1 的第一个扇区号	= BPB_RsvdSecCnt
SectorNoOfFAT1		equ	1
; DeltaSectorNo = BPB_RsvdSecCnt + (BPB_NumFATs * FATSz) - 2
; 文件的开始Sector号 = DirEntry中的开始Sector号 + 根目录占用Sector数目
;                      + DeltaSectorNo
DeltaSectorNo		equ	17
```

​	**代码2 使用fat12hdr.inc(boot.asm中)**

```
	jmp short LABEL_START		; Start to boot.
	nop				; 这个 nop 不可少
; 下面是 FAT12 磁盘的头, 之所以包含它是因为下面用到了磁盘的一些信息
%include	"fat12hdr.inc"
LABEL_START:
```

​	**代码3 使用fat12hdr.inc(loader.asm中)**

```
org  0100h
BaseOfStack		equ	0100h
BaseOfKernelFile	equ	 08000h	; KERNEL.BIN 被加载到的位置 ----  段地址
OffsetOfKernelFile	equ	     0h	; KERNEL.BIN 被加载到的位置 ---- 偏移地址
	jmp	LABEL_START		; Start
; 下面是 FAT12 磁盘的头, 之所以包含它是因为下面用到了磁盘的一些信息
%include	"fat12hdr.inc"
LABEL_START:			; <--- 从这里开始 *************
	mov	ax, cs
	mov	ds, ax
	mov	es, ax
	mov	ss, ax
	mov	sp, BaseOfStack
	mov	dh, 0			; "Loading  "
	call	DispStr			; 显示字符串
	; 下面在 A 盘的根目录寻找 KERNEL.BIN
	mov	word [wSectorNo], SectorNoOfRootDirectory	
	xor	ah, ah	; `.
	xor	dl, dl	;  | 软驱复位
	int	13h	; /
LABEL_SEARCH_IN_ROOT_DIR_BEGIN:
	cmp	word [wRootDirSizeForLoop], 0	; `.
	jz	LABEL_NO_KERNELBIN		;  | 判断根目录区是不是已经读完,
	dec	word [wRootDirSizeForLoop]	; /  读完表示没有找到 KERNEL.BIN
	mov	ax, BaseOfKernelFile
	mov	es, ax			; es <- BaseOfKernelFile
	mov	bx, OffsetOfKernelFile	; bx <- OffsetOfKernelFile
	mov	ax, [wSectorNo]		; ax <- Root Directory 中的某 Sector 号
	mov	cl, 1
	call	ReadSector
	mov	si, KernelFileName	; ds:si -> "KERNEL  BIN"
	mov	di, OffsetOfKernelFile
	cld
	mov	dx, 10h
LABEL_SEARCH_FOR_KERNELBIN:
	cmp	dx, 0				  ; `.
	jz	LABEL_GOTO_NEXT_SECTOR_IN_ROOT_DIR;  | 循环次数控制, 如果已经读完
	dec	dx				  ; /  了一个 Sector, 就跳到下一个
	mov	cx, 11
LABEL_CMP_FILENAME:
	cmp	cx, 0			; `.
	jz	LABEL_FILENAME_FOUND	;  | 循环次数控制, 如果比较了 11 个字符都
	dec	cx			; /  相等, 表示找到
	lodsb				; ds:si -> al
	cmp	al, byte [es:di]	; if al == es:di
	jz	LABEL_GO_ON
	jmp	LABEL_DIFFERENT
LABEL_GO_ON:
	inc	di
	jmp	LABEL_CMP_FILENAME	;	继续循环
LABEL_DIFFERENT:
	and	di, 0FFE0h		; else`. 让 di 是 20h 的倍数
	add	di, 20h			;      |
	mov	si, KernelFileName	;      | di += 20h  下一个目录条目
	jmp	LABEL_SEARCH_FOR_KERNELBIN;   /
LABEL_GOTO_NEXT_SECTOR_IN_ROOT_DIR:
	add	word [wSectorNo], 1
	jmp	LABEL_SEARCH_IN_ROOT_DIR_BEGIN
LABEL_NO_KERNELBIN:
	mov	dh, 2			; "No KERNEL."
	call	DispStr			; 显示字符串
%ifdef	_LOADER_DEBUG_
	mov	ax, 4c00h		; `.
	int	21h			; / 没有找到 KERNEL.BIN, 回到 DOS
%else
	jmp	$			; 没有找到 KERNEL.BIN, 死循环在这里
%endif
LABEL_FILENAME_FOUND:			; 找到 KERNEL.BIN 后便来到这里继续
	mov	ax, RootDirSectors
	and	di, 0FFF0h		; di -> 当前条目的开始
	push	eax
	mov	eax, [es : di + 01Ch]		; `.
	mov	dword [dwKernelSize], eax	; / 保存 KERNEL.BIN 文件大小
	pop	eax
	add	di, 01Ah		; di -> 首 Sector
	mov	cx, word [es:di]
	push	cx			; 保存此 Sector 在 FAT 中的序号
	add	cx, ax
	add	cx, DeltaSectorNo	; cl <- LOADER.BIN 的起始扇区号(0-based)
	mov	ax, BaseOfKernelFile
	mov	es, ax			; es <- BaseOfKernelFile
	mov	bx, OffsetOfKernelFile	; bx <- OffsetOfKernelFile
	mov	ax, cx			; ax <- Sector 号
LABEL_GOON_LOADING_FILE:
	push	ax			; `.
	push	bx			;  |
	mov	ah, 0Eh			;  | 每读一个扇区就在 "Loading  " 后面
	mov	al, '.'			;  | 打一个点, 形成这样的效果:
	mov	bl, 0Fh			;  | Loading ......
	int	10h			;  |
	pop	bx			;  |
	pop	ax			; /
	mov	cl, 1
	call	ReadSector
	pop	ax			; 取出此 Sector 在 FAT 中的序号
	call	GetFATEntry
	cmp	ax, 0FFFh
	jz	LABEL_FILE_LOADED
	push	ax			; 保存 Sector 在 FAT 中的序号
	mov	dx, RootDirSectors
	add	ax, dx
	add	ax, DeltaSectorNo
	add	bx, [BPB_BytsPerSec]
	jmp	LABEL_GOON_LOADING_FILE
LABEL_FILE_LOADED:
	call	KillMotor		; 关闭软驱马达
	mov	dh, 1			; "Ready."
	call	DispStr			; 显示字符串
	jmp	$
```

​	**代码4 还算不上内核的内核雏形kernel.asm**

```
; 编译链接方法
; $ nasm -f elf kernel.asm -o kernel.o
; $ ld -s kernel.o -o kernel.bin    #‘-s’选项意为“strip all”
[section .text]	; 代码在此
global _start	; 导出 _start
_start:	; 跳到这里来的时候，我们假设 gs 指向显存
	mov	ah, 0Fh				; 0000: 黑底    1111: 白字
	mov	al, 'K'
	mov	[gs:((80 * 1 + 39) * 2)], ax	; 屏幕第 1 行, 第 39 列。
	jmp	$
```

#### 5.3.2 代码主要结构流程图

```flow
st=>start: Loader
op1=>operation: 初始化堆栈
op2=>operation: 遍历根目录取所有的扇区，将每一个扇区加载到内存
op3=>operation: 每读一个扇区打一个点
op4=>operation: 寻找文件名为 KERNEL.BIN 的条目
op5=>operation: 把 KERNEL.BIN 加载到内存
op6=>operation: 显示字符串“Ready”
e=>end: 结束任务

st->op1->op2->op3->op4->op5->op6->e
```

<center>流程图-1 Loader将Kernel加载入内存</center>

#### 5.3.3 调试过程

​	第一步，输入以下代码进行编译：

```
nasm boot.asm -o boot.bin
nasm loader.asm -o loader.bin
nasm -f elf -o kernel.o kernel.asm
ld -m elf_i386 -s -o kernel.bin kernel.o
```

​	第二步，输入以下代码进行调试：

```
dd if=boot.bin of=a.img bs=512 count=1 conv=notrunc
sudo mount -o loop a.img /mnt/floppy/
sudo cp loader.bin /mnt/floppy/ -v
sudo cp kernel.bin /mnt/floppy/ -v
sudo umount /mnt/floppy/
```

#### 5.3.4 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143635720.png" alt="image-20230320143635720" style="zoom:67%;" />

​	可以看到，在上一个实验的基础上，这次的实验结果多出了“Loading.........”及“Ready.”这样的两行，说明我们已经载入了内核，并且由Loader读取了一个扇区。不过，由于目前我们除了把内核加载到内存之外没有做其他任何工作，所以除了能看到“Ready.”的字样之外，并没有其他现象出现。

#### 5.3.5 实验分析

​	在本实验中，我们成功的加载了内核Kernel，虽然这是Kernel中还没有其他内容，但已经是一个内核的雏形，之后对于内核，还可以脱离汇编语言，使用C语言进行编程，是一个非常大的进步。



### 5.4 实验4 进入保护模式，并显示内存使用状况

#### 5.4.1 关键代码

​	**代码1 Loader的32位代码段**

```
[SECTION .s32]
ALIGN	32
[BITS	32]
LABEL_PM_START:
	mov	ax, SelectorVideo
	mov	gs, ax
	mov	ax, SelectorFlatRW
	mov	ds, ax
	mov	es, ax
	mov	fs, ax
	mov	ss, ax
	mov	esp, TopOfStack

	push	szMemChkTitle
	call	DispStr
	add	esp, 4

	call	DispMemInfo
	call	SetupPaging

	mov	ah, 0Fh				; 0000: 黑底    1111: 白字
	mov	al, 'P'
	mov	[gs:((80 * 0 + 39) * 2)], ax	; 屏幕第 0 行, 第 39 列
	jmp	$
```

​	**代码2 Loader进入保护模式**

```
LABEL_FILE_LOADED:
	call	KillMotor		; 关闭软驱马达
	mov	dh, 1			; "Ready."
	call	DispStrRealMode		; 显示字符串
	; 下面准备跳入保护模式
	; 加载 GDTR
	lgdt	[GdtPtr]
	; 关中断
	cli
	; 打开地址线A20
	in	al, 92h
	or	al, 00000010b
	out	92h, al
	; 准备切换到保护模式
	mov	eax, cr0
	or	eax, 1
	mov	cr0, eax
	; 真正进入保护模式
	jmp	dword SelectorFlatC:(BaseOfLoaderPhyAddr+LABEL_PM_START)
```

​	**代码3 得到内存信息**

```
; 得到内存数
	mov	ebx, 0			; ebx = 后续值, 开始时需为 0
	mov	di, _MemChkBuf		; es:di 指向一个地址范围描述符结构(ARDS)
.MemChkLoop:
	mov	eax, 0E820h		; eax = 0000E820h
	mov	ecx, 20			; ecx = 地址范围描述符结构的大小
	mov	edx, 0534D4150h		; edx = 'SMAP'
	int	15h			; int 15h
	jc	.MemChkFail
	add	di, 20
	inc	dword [_dwMCRNumber]	; dwMCRNumber = ARDS 的个数
	cmp	ebx, 0
	jne	.MemChkLoop
	jmp	.MemChkOK
.MemChkFail:
	mov	dword [_dwMCRNumber], 0
.MemChkOK:
```

​	**代码4 显示内存信息**

```
DispMemInfo:
	push	esi
	push	edi
	push	ecx

	mov	esi, MemChkBuf
	mov	ecx, [dwMCRNumber];for(int i=0;i<[MCRNumber];i++)//每次得到一个ARDS
.loop:				  ;{
	mov	edx, 5		  ;  for(int j=0;j<5;j++)//每次得到一个ARDS中的成员
	mov	edi, ARDStruct	  ;  {//依次显示:BaseAddrLow,BaseAddrHigh,LengthLow
.1:				  ;               LengthHigh,Type
	push	dword [esi]	  ;
	call	DispInt		  ;    DispInt(MemChkBuf[j*4]); // 显示一个成员
	pop	eax		  ;
	stosd			  ;    ARDStruct[j*4] = MemChkBuf[j*4];
	add	esi, 4		  ;
	dec	edx		  ;
	cmp	edx, 0		  ;
	jnz	.1		  ;  }
	call	DispReturn	  ;  printf("\n");
	cmp	dword [dwType], 1 ;  if(Type == AddressRangeMemory)
	jne	.2		  ;  {
	mov	eax, [dwBaseAddrLow];
	add	eax, [dwLengthLow];
	cmp	eax, [dwMemSize]  ;    if(BaseAddrLow + LengthLow > MemSize)
	jb	.2		  ;
	mov	[dwMemSize], eax  ;    MemSize = BaseAddrLow + LengthLow;
.2:				  ;  }
	loop	.loop		  ;}
				  ;
	call	DispReturn	  ;printf("\n");
	push	szRAMSize	  ;
	call	DispStr		  ;printf("RAM size:");
	add	esp, 4		  ;
				  ;
	push	dword [dwMemSize] ;
	call	DispInt		  ;DispInt(MemSize);
	add	esp, 4		  ;

	pop	ecx
	pop	edi
	pop	esi
	ret
```

​	**代码5 启动分页函数SetupPaging**

```
SetupPaging:
	; 根据内存大小计算应初始化多少PDE以及多少页表
	xor	edx, edx
	mov	eax, [dwMemSize]
	mov	ebx, 400000h	; 400000h = 4M = 4096 * 1024, 一个页表对应的内存大小
	div	ebx
	mov	ecx, eax	; 此时 ecx 为页表的个数，也即 PDE 应该的个数
	test	edx, edx
	jz	.no_remainder
	inc	ecx		; 如果余数不为 0 就需增加一个页表
.no_remainder:
	push	ecx		; 暂存页表个数

	; 为简化处理, 所有线性地址对应相等的物理地址. 并且不考虑内存空洞.

	; 首先初始化页目录
	mov	ax, SelectorFlatRW
	mov	es, ax
	mov	edi, PageDirBase	; 此段首地址为 PageDirBase
	xor	eax, eax
	mov	eax, PageTblBase | PG_P  | PG_USU | PG_RWW
.1:
	stosd
	add	eax, 4096		; 为了简化, 所有页表在内存中是连续的
	loop	.1

	; 再初始化所有页表
	pop	eax			; 页表个数
	mov	ebx, 1024		; 每个页表 1024 个 PTE
	mul	ebx
	mov	ecx, eax		; PTE个数 = 页表个数 * 1024
	mov	edi, PageTblBase	; 此段首地址为 PageTblBase
	xor	eax, eax
	mov	eax, PG_P  | PG_USU | PG_RWW
.2:
	stosd
	add	eax, 4096		; 每一页指向 4K 的空间
	loop	.2

	mov	eax, PageDirBase
	mov	cr3, eax
	mov	eax, cr0
	or	eax, 80000000h
	mov	cr0, eax
	jmp	short .3
.3:
	nop

	ret
```

#### 5.4.2 代码主要结构流程图

```flow
st=>start: Loader
op1=>operation: 进入保护模式
op2=>operation: 得到内存信息
op3=>operation: 启动分页函数SetupPaging
op4=>operation: 显示内存信息
e=>end: 结束任务

st->op1->op2->op3->op4->e
```

<center>流程图-1 Loader显示内存使用情况</center>

#### 5.4.3 函数调用图

```mermaid
sequenceDiagram
	Loader->>LABEL_FILE_LOADED(): 调用函数LABEL_FILE_LOADED()进入保护模式
	Loader->>MemChkLoop(): 调用函数MemChkLoop()得到内存信息
	Loader->>DispMemInfo(): 调用函数myprint()
	DispMemInfo()-->>Loader: 屏幕显示内存基本信息
	Loader->>SetupPaging(): 调用函数SetupPaging()启动分页
	SetupPaging()-->>Loader: 屏幕显示内存分页信息
```

#### 5.4.4 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143657532.png" alt="image-20230320143657532" style="zoom:67%;" />

​	看到字母“P”，说明我们已经成功地进入保护模式，下方显示的RAM size列出了内存使用状况。



### 5.5 实验 5 向内核交出控制权

#### 5.5.1 关键代码

​	**代码1 向内核跳转**

```
	;***************************************************************
	jmp	SelectorFlatC:KernelEntryPointPhyAddr	; 正式进入内核 *
	;***************************************************************
```

#### 5.5.2 调试过程

​	第一步，输入以下代码进行编译：

```
nasm boot.asm -o boot.bin
nasm loader.asm -o loader.bin
nasm -f elf -o kernel.o kernel.asm
ld -m elf_i386 -s -o kernel.bin kernel.o
```


​	第二步，输入以下代码进行调试：

```
dd if=boot.bin of=a.img bs=512 count=1 conv=notrunc
sudo mount -o loop a.img /mnt/floppy/
sudo cp loader.bin /mnt/floppy/ -v
sudo cp kernel.bin /mnt/floppy/ -v
sudo umount /mnt/floppy/
```

​	第三步，输入c并回车开始调试。

#### 5.5.3 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143713797.png" alt="image-20230320143713797" style="zoom:67%;" />

​	第一行中央出现字符”k”，这表明我们的内核正在执行了。

------





## 6 进程

### 6.1 实验1 最简单的进程

#### 6.1.1 关键代码

​	**代码1 时钟中断程序**

```
ALIGN	16
hwint00:		; Interrupt routine for irq 0 (the clock).
	iretd
```

​	**代码2 一个小的进程体**

```
void TestA()
{
	int i = 0;
	while(1){
		disp_str("A");
		disp_int(i++);
		disp_str(".");
		delay(1);
	}
}
```

​	**代码3 函数kernel_main**

```
PUBLIC int kernel_main()
{
	disp_str("-----\"kernel_main\" begins-----\n");
	PROCESS* p_proc	= proc_table;
	p_proc->ldt_sel	= SELECTOR_LDT_FIRST;
	memcpy(&p_proc->ldts[0], &gdt[SELECTOR_KERNEL_CS>>3], sizeof(DESCRIPTOR));
	p_proc->ldts[0].attr1 = DA_C | PRIVILEGE_TASK << 5;	// change the DPL
	memcpy(&p_proc->ldts[1], &gdt[SELECTOR_KERNEL_DS>>3], sizeof(DESCRIPTOR));
	p_proc->ldts[1].attr1 = DA_DRW | PRIVILEGE_TASK << 5;	// change the DPL
	p_proc->regs.cs	= (0 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | RPL_TASK;
	p_proc->regs.ds	= (8 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | RPL_TASK;
	p_proc->regs.es	= (8 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | RPL_TASK;
	p_proc->regs.fs	= (8 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | RPL_TASK;
	p_proc->regs.ss	= (8 & SA_RPL_MASK & SA_TI_MASK) | SA_TIL | RPL_TASK;
	p_proc->regs.gs	= (SELECTOR_KERNEL_GS & SA_RPL_MASK) | RPL_TASK;
	p_proc->regs.eip= (u32)TestA;
	p_proc->regs.esp= (u32) task_stack + STACK_SIZE_TOTAL;
	p_proc->regs.eflags = 0x1202;	// IF=1, IOPL=1, bit 2 is always 1.
	p_proc_ready	= proc_table;
	restart();
	while(1){}
}
```

​	**代码4 delay函数**

```
PUBLIC void delay(int time)
{
	int i, j, k;
	for (k = 0; k < time; k++) {
		for (i = 0; i < 10; i++) {
			for (j = 0; j < 10000; j++) {}
		}
	}
}
```

​	**代码5 restart函数**

```
restart:
	mov	esp, [p_proc_ready]
	lldt	[esp + P_LDT_SEL] 
	lea	eax, [esp + P_STACKTOP]
	mov	dword [tss + TSS3_S_SP0], eax
	pop	gs
	pop	fs
	pop	es
	pop	ds
	popad
	add	esp, 4
	iretd
```

#### 6.1.2 代码主要结构流程图

```flow
st=>start: Kernel
op1=>operation: 准备好进程体 TestA
op2=>operation: 对进程表、TSS进行初始化
op3=>operation: 准备进程表
op4=>operation: 时钟中断处理实现 ring0 到 ring1 的跳转
op5=>operation: 开始进程
e=>end: 结束任务

st->op1->op2->op3->op4->op5->e
```

<center>流程图-1 Kernel启动进程</center>

#### 6.1.3 函数调用图

```mermaid
sequenceDiagram
	Kernel->>init_descriptor(): 调用函数init_descriptor()
	init_descriptor()-->>Kernel: 初始化GDT中的描述符
	Kernel->>init_port(): 调用函数init_port()
	init_port()-->>Kernel: 初始化TSS
	Kernel->>kernel_main(): 调用函数myprint()准备进程表
	kernel_main()-->>Kernel: 准备进程表
	Kernel->>restart(): 通过restart()跳入进程
	restart()->>iretd(): 调用时钟中断iretd()实现ring0->ring1
	restart()-->>Kernel: 开始进程
```

#### 6.1.4 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143734465.png" alt="image-20230320143734465" style="zoom:67%;" />

​	由调试结果，我们看到了不断出现的字符“A”和不断增加的数字，这代表我们的进程已经开始运行，我们的调试成功了。

#### 6.1.5 实验分析

​	在本实验中，我们为我们的操作系统添加了进程，而为了实现进程功能，我们还实现了以下的部分：
​	时钟中断处理实现，用以实现ring0到ring1的跳转；
​	全新的进程表、进程体、TSS结构和更新过的GDT结构；
​	准备一个小的进程体；
​	对进程表、TSS进行初始化......
​	在一切准备完毕后，我们便可通过restart跳入进程，最终可以看到运行起来的进程。
​	而回顾整个程序，我们第一个进程的启动过程主要分为以下四步：
​	第一步，准备好进程体（本例中为TestA()）；
​	第二步，初始化GDT中的TSS和LDT两个描述符，并且初始化TSS（在init_port()中完成）；
​	第三步，准备进程表（在kernel_main()中完成）；
​	第四步，完成跳转，实现ring0->ring1.开始进程（kernel.asm中的restart）。
​	不过，我们的进程和进程表仍然非常简单，第一是我们的进程表不能保存并回复任务状态，第二是我们的进程体开启后便不能停止，因为我们并没有开启时钟中断，不能胜任多进程的任务，因此我们会在下面的实验2及实验3对其改进与完善。
​	让操作系统走进保护模式，另外建立了一个文件，将其通过引导扇区加载如内存， 引导扇区负责把 Loader 加载进内存，其他工作：跳入保护模式、开始执行内核 等由 Loader 模块去做，这样突破 512 字节的限制，灵活很多。



### 6.2 实验2 丰富中断处理程序

#### 6.2.1 关键代码

​	**代码1 打开时钟中断**

```
	out_byte(INT_M_CTLMASK,	0xFE);	// Master 8259, OCW1.
	out_byte(INT_S_CTLMASK,	0xFF);	// Slave  8259, OCW1. 
```

​	**代码2 设置EOI**

```
	hwint00:		; Interrupt routine for irq 0 (the clock).
	mov	al, EOI		; `. reenable
	out	INT_M_CTL, al	; / master 8259
	iretd
```

​	**代码3 EOI和INT_M_CTL**

```
INT_M_CTL	equ	0x20 ; I/O port for interrupt controller        <Master>
INT_M_CTLMASK	equ	0x21 ; setting bits in this port disables ints  <Master>
INT_S_CTL	equ	0xA0 ; I/O port for second interrupt controller <Slave>
INT_S_CTLMASK	equ	0xA1 ; setting bits in this port disables ints  <Slave>
EOI		equ	0x20
```

​	**代码4 时钟中断处理程序**

```
extern	disp_str
...
[SECTION .data]
clock_int_msg		db	"^", 0
...
ALIGN	16
hwint00:		; Interrupt routine for irq 0 (the clock).
	sub	esp, 4
	pushad		; `.
	push	ds	;  |
	push	es	;  | 保存原寄存器值
	push	fs	;  |
	push	gs	; /
	mov	dx, ss
	mov	ds, dx
	mov	es, dx
	mov	esp, StackTop		; 切到内核栈
	inc	byte [gs:0]		; 改变屏幕第 0 行, 第 0 列的字符
	mov	al, EOI			; `. reenable
	out	INT_M_CTL, al		; /  master 8259
	push	clock_int_msg
	call	disp_str
	add	esp, 4
	mov	esp, [p_proc_ready]	; 离开内核栈
	lea	eax, [esp + P_STACKTOP]
	mov	dword [tss + TSS3_S_SP0], eax
	pop	gs	; `.
	pop	fs	;  |
	pop	es	;  | 恢复原寄存器值
	pop	ds	;  |
	popad		; /
	add	esp, 4
	iretd
```

#### 6.2.2 代码主要结构流程图

```flow
st=>start: 时钟中断处理程序
op1=>operation: 保存原寄存器值
op2=>operation: 切到内核栈
op3=>operation: 设置EOI
op4=>operation: 改变屏幕第 0 行, 第 0 列的字符
op5=>operation: 离开内核栈
op6=>operation: 恢复原寄存器值
e=>end: 结束中断

st->op1->op2->op3->op4->op5->op6->e
```

<center>流程图-1 时钟中断改变屏幕字符</center>

#### 6.2.3 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143759806.png" alt="image-20230320143759806" style="zoom:67%;" />

#### 6.2.4 实验分析

​	从调试结果我们可以看到不断出现的字符“^”，这说明函数disp_str运行正常，而且没有影响到中断处理的其他部分以及进程A。之所以在两次字符A的中间出现不止一个“^”，是因为我们在进程的执行体中加入了delay()函数，而再次函数的执行过程中发生了多次中断。



### 6.3 实验3 中断重入

#### 6.3.1 关键代码

​	代码1 修改后的时钟中断处理

```
hwint00:		; Interrupt routine for irq 0 (the clock).
	sub	esp, 4
	pushad		; `.
	push	ds	;  |
	push	es	;  | 保存原寄存器值
	push	fs	;  |
	push	gs	; /
	mov	dx, ss
	mov	ds, dx
	mov	es, dx
	inc	byte [gs:0]		; 改变屏幕第 0 行, 第 0 列的字符
	mov	al, EOI			; `. reenable
	out	INT_M_CTL, al		; /  master 8259
	inc	dword [k_reenter]
	cmp	dword [k_reenter], 0
	jne	.re_enter	
	mov	esp, StackTop		; 切到内核栈
	sti	
	push	clock_int_msg
	call	disp_str
	add	esp, 4
;;; 	push	1
;;; 	call	delay
;;; 	add	esp, 4	
	cli	
	mov	esp, [p_proc_ready]	; 离开内核栈
	lea	eax, [esp + P_STACKTOP]
	mov	dword [tss + TSS3_S_SP0], eax
.re_enter:	; 如果(k_reenter != 0)，会跳转到这里
	dec	dword [k_reenter]
	pop	gs	; `.
	pop	fs	;  |
	pop	es	;  | 恢复原寄存器值
	pop	ds	;  |
	popad		; /
	add	esp, 4
	iretd
```

#### 6.3.2 代码主要结构流程图

```flow
st=>start: 时钟中断处理程序
op1=>operation: 保存原寄存器值
op2=>operation: 改变屏幕第 0 行, 第 0 列的字符
op3=>operation: 设置EOI
cond=>condition: 是否处于中断重入状态?
op4=>operation: 打印字符^
op5=>operation: 恢复原寄存器值
e=>end: 跳出中断程序

st->op1->op2->op3->cond
cond(yes)->op5->e
cond(no)->op4->op5->e
```

#### 6.3.3 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143819577.png" alt="image-20230320143819577" style="zoom:67%;" />

#### 6.3.4 实验分析

​	中断重入就是在一个中断程序执行过程中又被另一个中断打断，转而又去执行另一个中断程序。我们是应该允许这种情况的，因为在时钟中断中进行进程调度时我们还是希望接受键盘中断的，但我们又不希望因为中断重入而引起程序的错误。
​	由于我们的进程被打断进入中断程序时会自动关中断，所以我们先开中断。我们想让中断程序运行得足够久以至于能够被时钟中断再次打断，所以我们调用Delay函数，这样重入的度不断增加，控制权永远不会回到我们的进程中，而且内核堆栈迟早会溢出，产生异常进入13号中断处理程序，运行得知，左上角字符的速度明显比进程打印的字符的速度快，原因是inc byte [gs:0]的执行次数比进程体的打印语句要频繁得多。如果注释掉调用Delay的代码，我们看到两者的速度又会一样了。

我们可以看到，字符A和相应的数字在不断出现，并且可以发现屏幕左上角的字母跳动速度快，而字符“^”打印速度慢，说明很多时候程序在执行了inc byte [gs :0]之后并没有执行disp_str，这也说明中断重入的确发生了。

------





## 7 输入/输入系统

### 7.1 实验1 从中断开始——键盘初体验

#### 7.1.1 关键代码

​	代码1 键盘中断处理程序

```
PUBLIC void keyboard_hanlder(int irq)
{
   disp_str(＂str＂);
}
```

​	代码2 打开键盘中断

```
PUBLIC void init_keyboard()
{
   put_irq_handler(KEYBOARD_IRQ, keyboard_handler);/*设定键盘中断处理程序*/
   enable_irq(KEYBPARD_IRQ);/*开启键盘中断*/
}
```

​	代码3 调用 init_keyboard

```
PUBLIC int kerbel_main()
{
...
   init_keyboard();
...
}
```

#### 7.1.2 代码主要结构流程图

```flow
st=>start: Kernel
op1=>operation: 调用键盘中断
op2=>operation: 打开键盘中断
op3=>operation: 接受键盘敲击后输出字符
e=>end: 结束调用

st->op1->op2->op3->e
```

#### 7.1.3 函数调用图

```mermaid
sequenceDiagram
	Kernel->>kerbel_main(): 调用函数kerbel_main()
	kerbel_main()->>init_keyboard(): 调用函数init_keyboard()打开键盘中断
	init_keyboard()->>keyboard_hanlder(): 调用函数keyboard_hanlder()键盘中断处理
	keyboard_hanlder()-->>Kernel: 输出字符
```

#### 7.1.4 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143838177.png" alt="image-20230320143838177" style="zoom:67%;" />

​	可以看到，当敲击键盘之后，程序中出现了一个“\*”的字符，但再次敲击键盘后，键盘却不会再次相应，即屏幕上不会出现第二个“\*”字符。

#### 7.1.5 实验分析

在本实验中，我们调用键盘中断，在敲击键盘后出现了一个“\*”字符，代表我们的键盘中断是准确无误的，但它目前只能相应一次键盘敲击，显然是有问题的，需要进一步的改进。



### 7.2 实验2 从缓冲区读取信息并打印读取的值

#### 7.2.1 关键代码

​	**代码1 修改后的键盘中断**

```
PUBLIC void keyboard_handler(int irq)
{
	/* disp_str("*"); */
	u8 scan_code = in_byte(0x60);
	disp_int(scan_code);
}
```

#### 7.2.2 代码主要结构流程图

```flow
st=>start: Kernel
op1=>operation: 调用键盘中断
op2=>operation: 打开键盘中断
op3=>operation: 接受键盘敲击送入缓冲区
op4=>operation: 读取键盘缓冲区并且打印返回值
op5=>operation: 关闭键盘中断
e=>end: 结束调用

st->op1->op2->op3->op4->op5->e
```

#### 7.2.3 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143856229.png" alt="image-20230320143856229" style="zoom:67%;" />

​	在上图中，我们连续连续敲击了四个键，而实验结果则一共出现了8组代码：0x26、0xA6、0x17、0x97、0x2D、0xAD、0x2A、0xAA，分别对应于字符的Make Code和Break Code。

#### 7.2.4 实验分析

​	在本实验中，我们对键盘中断的处理程序进行了改进，不再是简单的打印一个“\*”，而是读取键盘缓冲区并且打印返回值，可以看到，我们目前的程序已经不会再卡死，而是可以相应多次键盘敲击过程，并且打印出扫描码。
​	对于这一点，则涉及到键盘控制器8042芯片和键盘编码器8048芯片，它们会监视键盘的输入，并把适当的数据传给计算机。
​	而对于敲击键盘的动作，则会产生被称为扫描码（Scan Code）的扫描码，分为按下一个按键或保持一个按键的Make Code和键盘弹起时的Break Code，这些扫描码可以通过in al,60h读取，并且只有我们将扫描码从缓冲区中读出来后，8042才能继续响应新的按键，这也解释了我们在实验7.1中只打印一个“\*”的原因。



### 7.3 实验3 扫描码解析数组、键盘输入缓冲区、与使用新加的任务处理键盘操作

#### 7.3.1 关键代码

​	**代码1 键盘缓冲区**

```
/* Keyboard structure, 1 per console. */
typedef struct s_kb {
	char*	p_head;			/* 指向缓冲区中下一个空闲位置 */
	char*	p_tail;			/* 指向键盘任务应处理的字节 */
	int	count;			/* 缓冲区中共有多少字节 */
	char	buf[KB_IN_BYTES];	/* 缓冲区 */
}KB_INPUT;
```

​	**代码2 修改后的keyboard_handler**

```
PRIVATE KB_INPUT	kb_in;
/*keyboard_handler*/
PUBLIC void keyboard_handler(int irq)
{
	u8 scan_code = in_byte(KB_DATA);
	if (kb_in.count < KB_IN_BYTES) {
		*(kb_in.p_head) = scan_code;
		kb_in.p_head++;
		if (kb_in.p_head == kb_in.buf + KB_IN_BYTES) {
			kb_in.p_head = kb_in.buf;
		}
		kb_in.count++;
	}
}
```

​	**代码3 修改后的init_keyboard**

```
PUBLIC void init_keyboard()
{
	kb_in.count = 0;
	kb_in.p_head = kb_in.p_tail = kb_in.buf;

        put_irq_handler(KEYBOARD_IRQ, keyboard_handler);/*设定键盘中断处理程序*/
        enable_irq(KEYBOARD_IRQ);                       /*开键盘中断*/
}
```

​	**代码4 关于键盘读取的tty任务**

```
PUBLIC void task_tty()
{
	while (1) {
		keyboard_read();
	}
}
```

​	**代码5 键盘读取函数keyboard_read()**

```
PUBLIC void keyboard_read()
{
	u8 scan_code;
	if(kb_in.count > 0){
		disable_int();
		scan_code = *(kb_in.p_tail);
		kb_in.p_tail++;
		if (kb_in.p_tail == kb_in.buf + KB_IN_BYTES) {
			kb_in.p_tail = kb_in.buf;
		}
		kb_in.count--;
		enable_int();
		disp_int(scan_code);
	}
}
```

#### 7.3.2 代码主要结构流程图

```flow
st=>start: Kernel
op1=>operation: 调用键盘中断
op2=>operation: 打开键盘中断
op3=>operation: 读取键盘输入送入缓冲区
op4=>operation: 读取缓冲区字符
op5=>operation: 通过任务tty处理扫描码
op6=>operation: 打印返回值
e=>end: 结束调用

st->op1->op2->op3->op4->op5->op6->e
```

#### 7.3.3 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143915501.png" alt="image-20230320143915501" style="zoom:67%;" />

​	可以看到，调试结果同前一个实验相比并没有出现变化，因为我们仅仅只是通过任务来处理扫描码，但还没有对扫描码进行解析。

#### 7.3.4 实验分析

​	在本实验中，我们首先是建立了一个扫描码的解析数组，其次则是建立了一个结构体，用来承担键盘缓冲区的工作，在最后，我们则是添加了一个任务，用来处理键盘操作，而这个任务将会构成未来中端任务的一部分。



### 7.4 实验4 解析扫描码——让字符显示出来

#### 7.4.1 关键代码

​	代码1 扫描解析码

```
PUBLIC void keyboard_read()
{
	u8	scan_code;
	char	output[2];
	int	make;	/* TRUE: make;  FALSE: break. */
	memset(output, 0, 2);
	if(kb_in.count > 0){
		disable_int();
		scan_code = *(kb_in.p_tail);
		kb_in.p_tail++;
		if (kb_in.p_tail == kb_in.buf + KB_IN_BYTES) {
			kb_in.p_tail = kb_in.buf;
		}
		kb_in.count--;
		enable_int();
		/* 下面开始解析扫描码 */
		if (scan_code == 0xE1) {
			/* 暂时不做任何操作 */
		}
		else if (scan_code == 0xE0) {
/* 暂时不做任何操作 */
}
		else {	/* 下面处理可打印字符 */
			/* 首先判断Make Code 还是 Break Code */
			make = (scan_code & FLAG_BREAK ? FALSE : TRUE);
			/* 如果是Make Code 就打印，是 Break Code 则不做处理 */
			if(make) {
				output[0] = keymap[(scan_code&0x7F)*MAP_COLS];
				disp_str(output);
			}
		}
		/* disp_int(scan_code); */
	}
```

#### 7.4.2 代码主要结构流程图

```flow
st=>start: Kernel
op1=>operation: 调用键盘中断
op2=>operation: 打开键盘中断
op3=>operation: 读取键盘输入送入缓冲区
op4=>operation: 读取缓冲区字符
op5=>operation: 通过任务tty处理扫描码
op6=>operation: 扫描解析数组
op7=>operation: 打印Make Code的解析码
e=>end: 结束调用

st->op1->op2->op3->op4->op5->op6->op7->e
```

#### 7.4.3 实验结果

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143935665.png" alt="image-20230320143935665" style="zoom:67%;" />

​	可以看到，目前我们的程序已经可以对小写a-z以及0-9进行正常的响应，不过仍然无法打印出大写字母，对shift、alt、ctrl、fn等按键则会输出意义不明的字符。

#### 7.4.4 实验分析

​	在本实验中我们通过扫描码解析数组keymap[]对扫描码进行了初步的解析，可以对输入的简单字符和数字进行解析。

------





## 8 实验总结

### 8.1实验中遇到的错误

#### 8.1.1 bochs配置文件bochsrc有关的问题

​	对于书本给出的bochsrc配置文件，由于版本不同，有一些部分的路径是错误的，必须换成本机路径。Bochsrc 中 romimage、vgaromimage 对应真实机器的 BIOS 和 VGA BIOS，应用实际路径
​	对于以下配置：

```
romimage: file=/usr/share/bochs/BIOS-bochs-latest
vgaromimage: /usr/share/vgabios/vgabios.bin
keyboard_mapping: enabled=1, map=/usr/share/bochs/keymaps/x11-pc-us.map
```

​	分别改为：

```
romimage:file=/home/zhaohan/bochs-2.7/bios/BIOS-bochs-latest
vgaromimage:file=/home/zhaohan/bochs-2.7/bios/VGABIOS-lgpl-latest
```

​	并且删去keyboard相应部分即可顺利的进行调试.

#### 8.1.2 __stack_chk_fail错误

​	在使用make命令进行编译连接时，会出现如下图所示的错误

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320143951671.png" alt="image-20230320143951671" style="zoom:67%;" />

​	原因是因为gcc编译器会自动进行堆栈越界检查，想要避免错误，可以将Makefile文件中的配置“$(CC) $(CFLAGS) -o $@ $<”修改为“$(CC) $(CFLAGS) -fno-stack-protector -o $@ $<”，即可解决问题，

#### 8.1.3 出现挂载点/mnt/floppy不存在的提示

​	该问题出现的原因是因为/mnt目录下没有对应的/floppy文件夹。解决办法有两个，最简单的办法是将所有的/mnt/floppy更换为/mnt；第二个则是在/mnt目录下建立一个floppy文件夹，需要输入以下指令：
`sudo mkdir /mnt/floppy`

#### 8.1.4  gcc指令相关问题

​	实验5.2中参考指令gcc -c bar.c -o bar.o应该改为gcc -m32 -c bar.c bar.o ,这时编译C文件产生的为32位代码，再使用ld -m elf_i386指令连接，操作成功.

#### 8.1.5  makefile相关问题

在实验6 7中需要用到makefile文件辅助编译和连接，以实验6.1为例，直接在终端输入make指令会出现如下错误

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320144007995.png" alt="image-20230320144007995" style="zoom:67%;" />

​	这是由于两个问题：一个是64位ubuntu中gcc ld指令的报错问题，还有一个是使用堆栈引发的冲突。
​	这就需要我们修改makefile文件，将makefile文件中的

```
CC		= gcc
LD		= ld  
```

改为：

```
CC		= gcc -m32
LD		= ld -m elf_i386 
```


​	同时将“$(CC) $(CFLAGS) -o $@ $<”修改为“$(CC) $(CFLAGS) -fno-stack-protector -o $@ $<”，再次在终端输入make指令编译，问题即可解决。从实验6.1往后的所有实验，都需要对makefile文件做出同样修改。

#### 8.1.6 找不到引导扇区的问题

在实验3.1中，初次编译运行bochs出现了如下问题，如图所示：

<img src="E:\大学作业\大二作业\操作系统课程设计\git repo\assets\image-20230320144021338.png" alt="image-20230320144021338" style="zoom:67%;" />

这是因为实验2中“hello os World”的那个a.img的最后两个字节在boot.asm中已经被填充为0xaa55,而我一开始是新创建的a.img，因此不能作为引导扇区使用。
解决办法：直接把实验2文件夹的a.img复制过来，也可以使用参考书光盘文件里面提供的a.img。

#### 8.1.7  ld指令出现incompatible with i386:x86-64 output

​	在第五章内核雏形实验5.1中，输入ld -s hello.o -o hello出现如图8-3所示错误，这是由于安装的Ubuntu是64位，默认产生64位的目标代码，但此处应编译32位目标代码，所以ld连接指令应改为ld -m elf_i386 -s -o hello hello.o,操作成功。



### 8.2 实验总结与心得

​	在本次操作系统复现实验中，我参考了哈工大、清华的操作系统实验课，了解操作系统内核的具体实现方式，选择《ORANGE’S：一个操作系统的实现》一书，实现了其中部分的源代码，并对其他相关的实验进行了调试。最后，就实验结果而言，实现了一个具有引导boot和Loader以及内核Kernel的操作系统，并且完成实模式到保护模式的转换以及实现中断和输入输出以及多进程功能。
​	通过本次实验，我了解操作系统开发实验环境，熟悉命令行方式的编译、调试工程，掌握基于硬件模拟器的调试技术，熟悉C语言编程和指针的概念，了解X86汇编语言，完成从理解操作系统原理到实践操作系统设计与实现的探索过程。更进一步地掌握了操作系统相应的底层原理，了解了引导扇区、保护模式、实模式、内核、进程、I/O等原理，对OS的体会更加深入。
​	通过本次操作系统实验，我的实践能力和创新能力有了极大的提高。另一方面，因为《ORANGE’S：一个操作系统的实现》一书年代较早，很多细节方面的异常在实验过程中时有出现，也提升了我通过各种方式解决问题的能力。
​	最后，我要感谢李蓉蓉老师对我的耐心教导，感谢老师在我实验过程中的指导和帮助，也对在实验过程中和我一起探讨问题的同学们表示感谢。

------





## 9 参考文献

[1]于渊. 《ORANGE’S：一个操作系统的实现》[M] 第二版 北京：电子工业出版社，2010年：P1-P278