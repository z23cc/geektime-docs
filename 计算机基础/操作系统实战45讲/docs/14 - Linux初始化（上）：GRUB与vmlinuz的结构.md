你好，我是LMOS。

在前面的课程中，我们建好了二级引导器，启动了我们的Cosmos，并进行了我们Cosmos的Hal层初始化。

我会用两节课带你领会Linux怎样做初始化。虽然我们自己具体实现过了初始化，不过我们也不妨看看Linux的初始化流程，借鉴一下Linux开发者的玩法。

这节课，我会先为你梳理启动的整体流程，重点为你解读Linux上GRUB是怎样启动，以及内核里的“实权人物”——vmlinuz内核文件是如何产生和运转的。下节课，我们从setup.bin文件的\_start函数入手，研究Linux初始化流程。

好，接下来我们从全局流程讲起，正式进入今天的学习。

## 全局流程

x86平台的启动流程，是非常复杂的。为了帮助你理解，我们先从全局粗略地看一看整体流程，然后一步步细化。

在机器加电后，BIOS会进行自检，然后由BIOS加载引导设备中引导扇区。在安装有Linux操作系统的情况下，在引导扇区里，通常是安装的GRUB的一小段程序（安装windows的情况则不同）。最后，GRUB会加载Linux的内核映像vmlinuz，如下图所示。

![](https://static001.geekbang.org/resource/image/f3/b3/f3d8b95e8c1563466d31f385bb42aab3.jpg?wh=3543%2A2005 "x86的全局启动流程示意图")

上图中的引导设备通常是机器中的硬盘，但也可以是U盘或者光盘甚至是软盘。BIOS会自动读取保存在CMOS中的引导设备信息。

## 从BIOS到GRUB

从前面的课程我们已经知道，CPU被设计成只能运行内存中的程序，没有办法直接运行储存在硬盘或者U盘中的操作系统程序。

如果想要运行硬盘或者U盘中的程序，就必须要先加载到内存（RAM）中才能运行。这是因为硬盘、U盘（外部储存器）并不和CPU直接相连，它们的访问机制和寻址方式与内存截然不同。

内存在断电后就没法保存数据了，那BIOS又是如何启动的呢？硬件工程师设计CPU时，硬性地规定在加电的瞬间，强制将CS寄存器的值设置为0XF000，IP寄存器的值设置为0XFFF0。

这样一来，CS:IP就指向了0XFFFF0这个物理地址。在这个物理地址上连接了主板上的一块小的ROM芯片。这种芯片的访问机制和寻址方式和内存一样，只是它在断电时不会丢失数据，在常规下也不能往这里写入数据，它是一种**只读内存**，BIOS程序就被固化在该ROM芯片里。

现在，CS:IP指向了0XFFFF0这个位置，正是BIOS程序的入口地址。这意味着BIOS正式开始启动。

BIOS一开始会初始化CPU，接着检查并初始化内存，然后将自己的一部分复制到内存，最后跳转到内存中运行。BIOS的下一步就是枚举本地设备进行初始化，并进行相关的检查，检查硬件是否损坏，这期间BIOS会调用其它设备上的固件程序，如显卡、网卡等设备上的固件程序。

当设备初始化和检查步骤完成之后，**BIOS会在内存中建立中断表和中断服务程序**，这是启动Linux至关重要的工作，因为Linux会用到它们。

具体是怎么操作的呢？BIOS会从内存地址（0x00000）开始用1KB的内存空间（0x00000~0x003FF）构建中断表，在紧接着中断表的位置，用256KB的内存空间构建BIOS数据区（0x00400~0x004FF），并在0x0e05b的地址加载了8KB大小的与中断表对应的中断服务程序。

中断表中有256个条目，每个条目占用4个字节，其中两个字节是CS寄存器的值，两个字节是IP寄存器的值。每个条目都指向一个具体的中断服务程序。

为了启动外部储存器中的程序，BIOS会搜索可引导的设备，搜索的顺序是由CMOS中的设置信息决定的（这也是我们平时讲的，所谓的在BIOS中设置的启动设备顺序）。一个是软驱，一个是光驱，一个是硬盘上，还可以是网络上的设备甚至是一个usb 接口的U盘，都可以作为一个启动设备。

当然，Linux通常是从硬盘中启动的。硬盘上的第1个扇区（每个扇区512字节空间），被称为**MBR（主启动记录）**，其中包含有基本的GRUB启动程序和分区表，安装GRUB时会自动写入到这个扇区，当MBR被BIOS装载到0x7c00地址开始的内存空间中后，BIOS就会将控制权转交给了MBR。在当前的情况下，其实是交给了GRUB。

到这里，BIOS到GRUB的过程结束。

## GRUB是如何启动的

根据前面内容可以发现，BIOS只会加载硬盘上的第1个扇区。不过这个扇区仅有512字节，这512字节中还有64字节的分区表加2字节的启动标志，很显然，剩下446字节的空间，是装不下GRUB这种大型通用引导器的。

于是，GRUB的加载分成了多个步骤，同时GRUB也分成了多个文件，其中有两个**重要的文件boot.img和core.img**，如下所示：

![](https://static001.geekbang.org/resource/image/b9/85/b92dyy0f686e730ffcb606ed17e5b785.jpg?wh=558%2A86 "GRUB核心文件")

其中，boot.img被GRUB的安装程序写入到硬盘的MBR中，同时在boot.img文件中的一个位置写入core.img文件占用的第一个扇区的扇区号。

而core.img文件是由GRUB安装程序根据安装时环境信息，用其它GRUB的模块文件动态生成。如下图所示：

![](https://static001.geekbang.org/resource/image/cb/4b/cb36d637ce0a0d7c38788102e139604b.jpg?wh=3180%2A1105 "GRUB-coreimg格式")

如果是从硬盘启动的话，core.img中的第一个扇区的内容就是diskboot.img文件。diskboot.img文件的作用是，**读取core.img中剩余的部分到内存中。**

由于这时diskboot.img文件还不识别文件系统，所以我们将core.img文件的全部位置，都用文件块列表的方式保存到diskboot.img文件中。这样就能确保diskboot.img文件找到core.img文件的剩余内容，最后将控制权交给kernel.img文件。

因为这时core.img文件中嵌入了足够多的功能模块，所以可以保证GRUB识别出硬盘分区上文件系统，能够访问/boot/grub目录，并且可以加载相关的配置文件和功能模块，来实现相关的功能，例如加载启动菜单、加载目标操作系统等。

正因为GRUB2大量使用了动态加载功能模块，这使得core.img文件的体积变得足够小。而GRUB的core.img文件一旦开始工作，就可以加载Linux系统的vmlinuz内核文件了。

## 详解vmlinuz文件结构

我们在/boot目录下会发现vmlinuz文件，这个文件是怎么来的呢？

其实它是由Linux编译生成的bzImage文件复制而来的，你自己可以下载最新的Linux代码.

我们一致把Linux源码解压到一个linux目录中，也就是说我们后面查找Linux源代码文件总是从linux目录开始的，切换到代码目录执行make ARCH=x86\_64，再执行make install，就会产生vmlinuz文件，你可以参考后面的makefile代码。

```
#linux/arch/x86/boot/Makefile
install:    sh $(srctree)/$(src)/install.sh $(KERNELRELEASE) $(obj)/bzImage \        System.map "$(INSTALL_PATH)"
```

install.sh脚本文件只是完成复制的功能，所以我们只要搞懂了bzImage文件结构，就等同于理解了vmlinuz文件结构。

那么bzImage文件又是怎么来的呢？我们只要研究bzImage文件在Makefile中的生成规则，就会恍然大悟，代码如下 ：

```
#linux/arch/x86/boot/Makefile
$(obj)/bzImage: $(obj)/setup.bin $(obj)/vmlinux.bin $(obj)/tools/build FORCE    $(call if_changed,image)    @$(kecho) 'Kernel: $@ is ready' ' (#'`cat .version`')'
```

从前面的代码可以知道，生成bzImage文件需要三个依赖文件：setup.bin、vmlinux.bin，linux/arch/x86/boot/tools目录下的build。让我们挨个来分析一下。

其实，build只是一个HOSTOS（正在使用的Linux）下的应用程序，它的作用就是将setup.bin、vmlinux.bin两个文件拼接成一个bzImage文件，如下图所示：

![](https://static001.geekbang.org/resource/image/22/30/22a83b33b4eededec109bda203133830.jpg?wh=2805%2A1805 "bzImage文件结构示意图")

剩下的就是搞清楚setup.bin、vmlinux.bin这两个文件的的结构，先来看看setup.bin文件，setup.bin文件是由objcopy命令根据setup.elf生成的。

setup.elf文件又怎么生成的呢？我们结合后面的代码来看看。

```
#这些目标文件正是由/arch/x86/boot/目录下对应的程序源代码文件编译产生
setup-y     += a20.o bioscall.o cmdline.o copy.o cpu.o cpuflags.o cpucheck.o
setup-y     += early_serial_console.o edd.o header.o main.o memory.o
setup-y     += pm.o pmjump.o printf.o regs.o string.o tty.o video.o
setup-y     += video-mode.o version.o

#……
SETUP_OBJS = $(addprefix $(obj)/,$(setup-y))
#……
LDFLAGS_setup.elf   := -m elf_i386 -T$(obj)/setup.elf: $(src)/setup.ld $(SETUP_OBJS) FORCE    $(call if_changed,ld)
#……
OBJCOPYFLAGS_setup.bin  := -O binary$(obj)/setup.bin: $(obj)/setup.elf FORCE    $(call if_changed,objcopy)
```

根据这段代码，不难发现setup.bin文件正是由/arch/x86/boot/目录下一系列对应的程序源代码文件编译链接产生，其中的**head.S文件**和**main.c文件**格外重要，别急，这个我之后会讲。

下面我们先看看vmlinux.bin是怎么产生的，构建vmlinux.bin的规则依然在linux/arch/x86/boot/目录下的Makefile文件中，如下所示：

```
#linux/arch/x86/boot/Makefile
OBJCOPYFLAGS_vmlinux.bin := -O binary -R .note -R .comment -S$(obj)/vmlinux.bin: $(obj)/compressed/vmlinux FORCE    $(call if_changed,objcopy)
```

这段代码的意思是，vmlinux.bin文件依赖于linux/arch/x86/boot/compressed/目录下的vmlinux目标，下面让我们切换到linux/arch/x86/boot/compressed/目录下继续追踪。打开该目录下的Makefile，会看到如下代码。

```
#linux/arch/x86/boot/compressed/Makefile
#……
#这些目标文件正是由/arch/x86/boot/compressed/目录下对应的程序源代码文件编译产生$(BITS)取值32或者64
vmlinux-objs-y := $(obj)/vmlinux.lds $(obj)/kernel_info.o $(obj)/head_$(BITS).o \    $(obj)/misc.o $(obj)/string.o $(obj)/cmdline.o $(obj)/error.o \    $(obj)/piggy.o $(obj)/cpuflags.o
vmlinux-objs-$(CONFIG_EARLY_PRINTK) += $(obj)/early_serial_console.o
vmlinux-objs-$(CONFIG_RANDOMIZE_BASE) += $(obj)/kaslr.o
ifdef CONFIG_X86_64    
vmlinux-objs-y += $(obj)/ident_map_64.o    
vmlinux-objs-y += $(obj)/idt_64.o $(obj)/idt_handlers_64.o    vmlinux-objs-y += $(obj)/mem_encrypt.o    
vmlinux-objs-y += $(obj)/pgtable_64.o    
vmlinux-objs-$(CONFIG_AMD_MEM_ENCRYPT) += $(obj)/sev-es.o
endif
#……
$(obj)/vmlinux: $(vmlinux-objs-y) $(efi-obj-y) FORCE  
$(call if_changed,ld)
```

结合这段代码我们发现，linux/arch/x86/boot/compressed目录下的vmlinux是由该目录下的head\_32.o或者head\_64.o、cpuflags.o、error.o、kernel.o、misc.o、string.o 、cmdline.o 、early\_serial\_console.o等文件以及piggy.o链接而成的。

其中，vmlinux.lds是链接脚本文件。在没做任何编译动作前，前面依赖列表中任何一个目标文件的源文件（除了piggy.o源码），我们几乎都可以在Linux内核源码里找到。

比如说，head\_64.o对应源文件head\_64.S、string.o对应源文件string.c、misc.o对应源文件misc.c等。

那么问题来了，为啥找不到piggy.o对应的源文件，比如piggy.c、piggy.S或其他文件呢？你需要在Makefile文件仔细观察一下，才能发现有个创建文件piggy.S的规则，代码如下所示：

```
#linux/arch/x86/boot/compressed/Makefile
#……
quiet_cmd_mkpiggy = MKPIGGY $@      
cmd_mkpiggy = $(obj)/mkpiggy $< > $@

targets += piggy.S
$(obj)/piggy.S: $(obj)/vmlinux.bin.$(suffix-y) $(obj)/mkpiggy FORCE    $(call if_changed,mkpiggy)
```

看到上面的规则，我们豁然开朗，原来piggy.o是由piggy.S汇编代码生成而来，而piggy.S是编译Linux内核时由mkpiggy工作（HOST OS下的应用程序）动态创建的，这就是我们找不到它的原因。

piggy.S的第一个依赖文件vmlinux.bin.$(suffix-y)中的suffix-y，它表示内核压缩方式对应的后缀。

```
#linux/arch/x86/boot/compressed/Makefile
#……
vmlinux.bin.all-y := $(obj)/vmlinux.bin
vmlinux.bin.all-$(CONFIG_X86_NEED_RELOCS) += $(obj)/vmlinux.relocs
$(obj)/vmlinux.bin.gz: $(vmlinux.bin.all-y) FORCE    
$(call if_changed,gzip)
$(obj)/vmlinux.bin.bz2: $(vmlinux.bin.all-y) FORCE    
$(call if_changed,bzip2)
$(obj)/vmlinux.bin.lzma: $(vmlinux.bin.all-y) FORCE    
$(call if_changed,lzma)
$(obj)/vmlinux.bin.xz: $(vmlinux.bin.all-y) FORCE   
$(call if_changed,xzkern)
$(obj)/vmlinux.bin.lzo: $(vmlinux.bin.all-y) FORCE    
$(call if_changed,lzo)
$(obj)/vmlinux.bin.lz4: $(vmlinux.bin.all-y) FORCE    
$(call if_changed,lz4)
$(obj)/vmlinux.bin.zst: $(vmlinux.bin.all-y) FORCE    
$(call if_changed,zstd22)
suffix-$(CONFIG_KERNEL_GZIP)    := gz
suffix-$(CONFIG_KERNEL_BZIP2)   := bz2
suffix-$(CONFIG_KERNEL_LZMA)    := lzma
suffix-$(CONFIG_KERNEL_XZ)  := xz
suffix-$(CONFIG_KERNEL_LZO)     := lzo
suffix-$(CONFIG_KERNEL_LZ4)     := lz4
suffix-$(CONFIG_KERNEL_ZSTD)    := zst
```

由前面内容可以发现，Linux内核可以被压缩成多种格式。虽然现在我们依然没有搞清楚vmlinux.bin文件是怎么来的，但是我们可以发现，linux/arch/x86/boot/compressed目录下的Makefile文件中，有下面这样的代码。

```
#linux/arch/x86/boot/compressed/Makefile
#……
OBJCOPYFLAGS_vmlinux.bin :=  -R .comment -S
$(obj)/vmlinux.bin: vmlinux FORCE 
$(call if_changed,objcopy)
```

也就是说，arch/x86/boot/compressed目录下的vmlinux.bin，它是由objcopy工具通过vmlinux目标生成。而vmlinux目标没有任何修饰前缀和依赖的目标，这说明它就是**最顶层目录下的一个vmlinux文件**。

我们继续深究一步就会发现，objcopy工具在处理过程中只是删除了vmlinux文件中“.comment”段，以及符号表和重定位表（通过参数-S指定），而vmlinux文件的格式依然是ELF格式的，如果不需要使用ELF格式的内核，这里添加“-O binary”选项就可以了。

我们现在来梳理一下，vmlinux文件是如何创建的。

其实，vmlinux文件就是编译整个Linux内核源代码文件生成的，Linux的代码分布在各个代码目录下，这些目录之下又存在目录，Linux的kbuild（内核编译）系统，会递归进入到每个目录，由该目录下的Makefile决定要编译哪些文件。

在编译完具体文件之后，就会在该目录下，把已经编译了的文件链接成一个该目录下的built-in.o文件，这个built-in.o文件也会与上层目录的built-in.o文件链接在一起。

再然后，层层目录返回到顶层目录，所有的built-in.o文件会链接生成一个vmlinux文件，这个vmlinux文件会通过前面的方法转换成vmlinux.bin文件。但是请注意，vmlinux.bin文件它依然是ELF格式的文件。

最后，工具软件会压缩成vmlinux.bin.gz文件，这里我们以gzip方式压缩。

让我们再次回到mkpiggy命令，其中mkpiggy是内核自带的一个工具程序，它把输出方式重定向到文件，从而产生piggy.S汇编文件，源码如下：

```
int main(int argc, char *argv[]){ 
    uint32_t olen;    
    long ilen;    
    FILE *f = NULL;    
    int retval = 1;
    f = fopen(argv[1], "r");    
    if (!f) {        
        perror(argv[1]);        
        goto bail;    
    }
    //……为节约篇幅略去部分代码
    printf(".section \".rodata..compressed\",\"a\",@progbits\n");
    printf(".globl z_input_len\n");    
    printf("z_input_len = %lu\n", ilen);    
    printf(".globl z_output_len\n");    
    printf("z_output_len = %lu\n", (unsigned long)olen);
    printf(".globl input_data, input_data_end\n");
    printf("input_data:\n");    
    printf(".incbin \"%s\"\n", argv[1]);    
    printf("input_data_end:\n");
    printf(".section \".rodata\",\"a\",@progbits\n");
    printf(".globl input_len\n");    
    printf("input_len:\n\t.long %lu\n", ilen);    
    printf(".globl output_len\n");    
    printf("output_len:\n\t.long %lu\n", (unsigned long)olen);
    retval = 0;
bail:    
    if (f)        
        fclose(f);    
    return retval;
}
//由上mkpiggy程序“写的”一个汇编程序piggy.S。
.section ".rodata..compressed","a",@progbits 
.globl z_input_len
 z_input_len = 1921557 
.globl z_output_len 
z_output_len = 3421472 
.globl input_data,input_data_end
.incbin "arch/x86/boot/compressed/vmlinux.bin.gz" 
input_data_end:
.section ".rodata","a",@progbits
.globl input_len
input_len:4421472
.globl output_len
output_len:4424772
```

根据上述代码不难发现，这个piggy.S非常简单，使用汇编指令incbin将压缩的vmlinux.bin.gz毫无修改地包含进来。

除了包含了压缩的vmlinux.bin.gz内核映像文件外，piggy.S中还定义了解压vmlinux.bin.gz时需要的各种信息，包括压缩内核映像的长度、解压后的长度等信息。

**这些信息和vmlinux.bin.gz文件，它们一起生成了piggy.o文件，然后piggy.o文件和$(vmlinux-objs-y)$(efi-obj-y)中的目标文件一起链接生成，最终生成了linux/arch/x86/boot/compressed目录下的vmlinux。**

说到这里，你是不是感觉，这和Linux的启动流程无关呢？有这种想法就大错特错了，要想搞明白Linux的启动流程，首先得搞懂它vmlinuz的文件结构。有了这些基础，才能知其然同时知其所以然。

## 重点回顾

又到了课程尾声，这节课的学习我们就告一段落了，我来给你做个总结。

今天我们首先从全局梳理了一遍x86平台的启动流程，掌握了BIOS加载GRUB的过程，又一起学习了BIOS是如何启动的，它又是如何加载引导设备的。

接着我们研究了GRUB的启动流程，BIOS加载了GRUB的第一个部分，这一部分加载了GRUB的其余部分。

最后，我们详细了解了Linux内核的启动文件vmlinuz的结构，搞清楚了它的生成过程。

## 思考题

请问，为什么要用C代码mkpiggy程序生成piggy.S文件，并包含vmlinux.bin.gz文件呢？

欢迎你在留言区记录你的收获和疑问，也欢迎你把这节课分享给有需要的朋友，跟他一起学习进步。

好，我是LMOS，我们下节课见！
<div><strong>精选留言（14）</strong></div><ul>
<li><span>neohope</span> 👍（57） 💬（7）<p>大体上整理了一下，有问题欢迎帮忙指出【上】：

操作系统的启动分为两个阶段：引导boot和启动startup，本节主要还是boot过程：
BIOS-&gt;GRUG1-&gt;GRUB1.5-&gt;GRUB2-&gt;Linux内核【环境硬盘引导、MBR分区】

1、按电源键，系统加电

2、主板通电
CPU加电时，会默认设置[CS:IP]为[0XF000:0XFFF0]，根据实模式下寻址规则，CPU指向0XFFFF0
这个地址正是BIOS启动程序位置，而BIOS访问方式与内存一致，所以CPU可以直接读取命令并执行

3、BIOS执行
3.1、BIOS首先执行POST自检，包括主板、内存、外设等，遇到问题则报警并停止引导

3.2、BIOS对设备执行简单的初始化工作

3.3、BIOS 会在内存中：
建立中断表（0x00000~0x003FF）
构建 BIOS 数据区（0x00400~0x004FF）
加载了中断服务程序（0x0e05b~0x1005A）

3.4、BIOS根据设备启动顺序，依次判断是否可以启动
比如先检查光驱能否启动
然后依次检查硬盘是否可以启动【硬盘分区的时候，设置为活动分区】

4、硬盘引导
4.1、先说下寻址方式，与扇区编号的事情
最传统的磁盘寻址方式为CHS，由三个参数决定读取哪个扇区：磁头（Heads）、柱面(Cylinder)、扇区(Sector)
磁头数【8位】，从0开始，最大255【微软DOS系统，只能用255个】，决定了读取哪个盘片的哪个面【一盘两面】
柱面数【10位】，从0开始，最大1023【决定了读取哪个磁道，磁道无论长短都会划分为相同扇区数】
扇区数【6位】，从1开始，最大数 63【CHS中扇区从1开始，而逻辑划分中扇区从0开始，经常会造成很多误解】
每个扇区为512字节

4.2、然后说下引导方式
BIOS在发现硬盘启动标志后，BIOS会引发INT 19H中断
这个操作，会将MBR【逻辑0扇区】，也就是磁盘CHS【磁头0，柱面0，扇区1】，读取到内存[0:7C00h]，然后执行其代码【GRUB1阶段】，至此BIOS把主动权交给了GRUB1阶段代码
MBR扇区为512字节，扇区最后分区表至少需要66字节【64字节DPT+2字节引导标志】，所以这段代码最多只能有446字节，grub中对应的就是引导镜像boot.img
boot.img的任务就是，定位，并通过BIOS INT13中断读取1.5阶段代码，并运行

5、Grub1.5阶段
5.1、先说一下MBR GAP
据说微软DOS系统原因，第一个分区的起始逻辑扇区是63扇区，在MBR【0扇区】和分布表之间【63扇区】，存在62个空白扇区，共 31KB。
Grub1.5阶段代码就安装在这里。

5.2、上面提到，boot.img主要功能就是找到并加载Grub1.5阶段代码，并切换执行。
Grub1.5阶段代码是core.img，其主要功能就是加载文件系统驱动，挂载文件系统， 位加载并运行GRUB2阶段代码。
core.img包括多个映像和模块：
diskboot.img【1.5阶段引导程序】，存在于MBR GAP第一个扇区；【这里是硬盘启动的情况，如果是cd启动就会是cdboot.img】
lzma_decompress.img【解压程序】
kernel.img【grub核心代码】，会【压缩存放】
biosdisk.mod【磁盘驱动】、Part_msdos.mod【MBR分区支持】、Ext2.mod【EXT文件系统】等，会【压缩存放】

其实boot.img只加载了core.img的第一个扇区【存放diskboot.img】，然后控制权就交出去了，grub阶段1代码使命结束。
diskboot.img知道后续每个文件的位置，会继续通过BIOS中断读取扇区，加载余下的部分并转交控制权，包括：
加载lzma_decompress.img，从而可以解压被压缩的模块
加载kernel.img，并转交控制权给kernel.img
kernel.img的grub_main函数会调用grub_load_modules函数加载各个mod模块
加载各个mod后，grub就支持文件系统了，访问磁盘不需要再依靠BIOS的中断以扇区为单位读取了，终于可以使用文件系统了</p>2021-06-10</li><br/><li><span>neohope</span> 👍（35） 💬（1）<p>大体上整理了一下，有问题欢迎帮忙指出【下】：
6、GRUB2阶段
现在grub就能访问boot&#47;grub及其子目录了
kernel.img接着调用grub_load_normal_mode加载normal模块
normal模块读取解析文件grub.cfg，查看有哪些命令，比如发现了linux、initrd这几个命令，需要linux模块
normal模块会根据command.lst，定位并加载用到的linux模块【一般在&#47;boot&#47;grub2&#47;i386-pc目录】
当然，同时需要完成初始化显示、载入字体等工作
接下来Grub就会给咱们展示启动菜单了

7、选择启动菜单
7.1、引导协议
引导程序加载内核，前提是确定好数据交换方式，叫做引导协议，内核中引导协议相关部分的代码在arch&#47;x86&#47;boot&#47;header.S中，内核会在这个文件中标明自己的对齐要求、是否可以重定位以及希望的加载地址等信息。同时也会预留空位，由引导加载程序在加载内核时填充，比如initramfs的加载位置和大小等信息。
引导加载程序和内核均为此定义了一个结构体linux_kernel_params，称为引导参数，用于参数设定。Grub会在把控制权移交给内核之前，填充好linux_kernel_params结构体。如果用户要通过grub向内核传递启动参数，即grub.cfg中linux后面的命令行参数。Grub也会把这部分信息关联到引导参数结构体中。

7.2、开始引导
Linux内核的相关文件位于&#47;boot 目录下，文件名均带有前缀 vmlinuz。
咱们选择对应的菜单后，Grub会开始执行对应命令，定位、加载、初始化内核，并移交到内核继续执行。
调用linux模块中的linux命令，加载linux内核
调用linux模块中的initrd命令，填充initramfs信息，然后Grub会把控制权移交给内核。
内核此时开始执行，同时也就可以读取linux_kernel_params结构体的数据了
boot阶段结束，开始进入startup阶段。</p>2021-06-10</li><br/><li><span>178</span> 👍（6） 💬（1）<p>vmlinuz是可引导的、压缩的内核。“vm”代表 “Virtual Memory”，linu代表“Linux”，z代表压缩。是个有故事的缩写</p>2021-11-10</li><br/><li><span>Fan</span> 👍（6） 💬（1）<p>请问，为什么要用 C 代码 mkpiggy 程序生成 piggy.S 文件，并包含 vmlinux.bin.gz 文件呢？

看了这下生成的这个piggy.S 文件
主要的就是这行代码
.incbin &quot;arch&#47;x86&#47;boot&#47;compressed&#47;vmlinux.bin.gz&quot;
通过C 代码的形式，可以传入不同的参数来设置不同的压缩包vmlinux.bin.xx  来生成piggy.S 。
</p>2021-06-09</li><br/><li><span>孤星可</span> 👍（5） 💬（1）<p>尝试实现了 lmoskrlimg 的逻辑（即 Cosmos.eki 的生成），有兴趣可以看看。

https:&#47;&#47;github.com&#47;guxingke&#47;demo&#47;blob&#47;master&#47;bytes-demo&#47;src&#47;main&#47;java&#47;com&#47;gxk&#47;demo&#47;Main.java</p>2021-06-10</li><br/><li><span>Qfeng</span> 👍（4） 💬（1）<p>一、简单总结：
1. CPU上电启动BIOS（ROM）
CPU硬件被设计成在加电的瞬间，强制将 CS 寄存器的值设置为 0XF000，IP 寄存器的值设置为 0XFFF0。
CS:IP 为 0XFFFF0 的这个物理地址上连接了主板上的一块小的 ROM 芯片，BIOS 程序就被固化在该 ROM 芯片里。
总结：CPU上电后硬件自动将 CS:IP 指向地址 0XFFFF0，这里存放了BIOS程序的入口地址，达到启动 BIOS 的目的。
（注：这个ROM芯片的访问机制和寻址方式和内存一样，只是它在断电时不会丢失数据，在常规下也不能往这里写入数据，它是一种只读内存。）

2. ROM BIOS
初始化CPU和内存，将自己拷贝到内存，执行环境跳转到内存（DDR）

3. DDR BIOS
1）设备初始化
2）在内存中建立中断服务程序表（0x00000~0x003FF，1KB，256个条目）, BIOS 数据区（0x00400~0x004FF）和中断服务程序（0x0E05B~1005B，8KB）
3）搜索可引导的外部存储器，并启动其中的程序。（包括：硬盘，U盘，软驱，光驱和网络设备等）
4）Linux从硬盘启动时，硬盘中名为MRB的第一个扇区包含的GRUB 启动程序（安装GRUB时自动写入）和分区表被 BIOS 装载到 0x7c00 地址开始的内存空间，
至此BIOS使命结束，控制权交给GRUB。

4. GRUB 启动
1）GRUB包含 boot.img和core.img两部分，硬盘MRB空间有限只包含boot.img，boot.img中包含core.img存放的硬盘扇区号
2）core.img 文件是由 GRUB 安装程序根据安装时环境信息，用其它 GRUB 的模块文件动态生成，主要包含diskboot.img，kernel.img和其他功能模块。
GRUB 的 core.img 文件一旦开始工作，就可以加载 Linux 系统的 vmlinuz 内核文件了。

5. Linux vmlinuz的生成
1）由Linux 编译生成的 bzImage 文件复制而来，存放在 &#47;boot 目录下
2）bzImage 由 arch&#47;x86&#47;boot&#47;Makefile 编译而来，依赖 setup.bin和vmlinux.bin两个。
3）setup.bin 由 &#47;arch&#47;x86&#47;boot&#47; 目录下的文件编译生成
4）arch&#47;x86&#47;boot&#47;compressed 目录 编译生成 vmlinux.bin过程总结：Linux 的 kbuild（内核编译）系统会递归进入到每个目录，由该目录下的 Makefile 决定要编译的文件。每个目录编译生成一个 built-in.o，所有的 built-in.o最终链接生成一个 vmlinux 文件最终压缩成 vmlinux.bin.gz，最终由mkpiggy 生成 vmlinux。

二、思考题
为什么要用 C 代码 mkpiggy 程序生成 piggy.S 文件，并包含 vmlinux.bin.gz 文件呢？
==》通过main函数传入的可变参数可以根据需要灵活扩展piggy.S的内容？</p>2022-06-19</li><br/><li><span>pedro</span> 👍（3） 💬（1）<p>工具生成方式灵活，支持多种配置，且能满足各种场景下的需求~</p>2021-06-09</li><br/><li><span>springXu</span> 👍（2） 💬（4）<p>程序来生成汇编代码，这个是有多种配置让使用者根据自己的需求来生成。比如vmlinuz.bin压缩的方式的不同又或者是cpu的指令不同。  产生的汇编代码也不同。 另外Cosmos.eki的生成过程能有个说明不？ </p>2021-06-09</li><br/><li><span>blentle</span> 👍（1） 💬（1）<p>不管肚子多少遍启动流程，还是记不住各种地址，除了0×7c00，哈哈</p>2021-06-10</li><br/><li><span>Fan</span> 👍（1） 💬（1）<p>又学习了一遍启动流程。😂</p>2021-06-09</li><br/><li><span>搬铁少年ai</span> 👍（0） 💬（1）<p>请问老师，boot.img 是 512B 大小，MBR 也是 512，难道 boot.img 安装到 MBR 的时候会把 MBR 全部覆盖掉吗？</p>2021-10-26</li><br/><li><span>搬铁少年ai</span> 👍（0） 💬（2）<p>老师好，我看别的书里面将上电后先跳转到 reset vector(0xFFFFFFF0)，然后 reset vector 会跳转到 BIOS 的入口（0xFFFF0），您这里为什么没有这个恢复向量呢</p>2021-10-24</li><br/><li><span>老王</span> 👍（0） 💬（1）<p>需要多分析各个脚本的功能，才能把握住脉络</p>2021-06-09</li><br/><li><span>WGJ</span> 👍（0） 💬（1）<p>老师，这里面好像piggy.s依赖vmlinux目标，然后vmlinux 目标又依赖piggy.s，这种互相依赖是怎么解决的啊</p>2024-02-02</li><br/>
</ul>