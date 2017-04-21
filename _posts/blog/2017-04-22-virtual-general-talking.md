---
layout: post
title: 虚拟化技术漫谈
categories: 7-虚拟化
description: 虚拟化技术漫谈
keywords: 虚拟化, virtual
---



### 1、虚拟化技术简介

* **什么是虚拟化**

虚拟化（Virtualization）技术最早出现在 20 世纪 60 年代的 IBM 大型机系统，在70年代的 System 370 系列中逐渐流行起来，这些机器通过一种叫虚拟机监控器（Virtual Machine Monitor，VMM）的程序在物理硬件之上生成许多可以运行独立操作系统软件的虚拟机（Virtual Machine）实例。随着近年多核系统、集群、网格甚至云计算的广泛部署，虚拟化技术在商业应用上的优势日益体现，不仅降低了 IT 成本，而且还增强了系统安全性和可靠性，虚拟化的概念也逐渐深入到人们日常的工作与生活中。

虚拟化是一个广义的术语，对于不同的人来说可能意味着不同的东西，这要取决他们所处的环境。在计算机科学领域中，虚拟化代表着对计算资源的抽象，而不仅仅局限于虚拟机的概念。例如对物理内存的抽象，产生了虚拟内存技术，使得应用程序认为其自身拥有连续可用的地址空间（Address Space），而实际上，应用程序的代码和数据可能是被分隔成多个碎片页或段），甚至被交换到磁盘、闪存等外部存储器上，即使物理内存不足，应用程序也能顺利执行。

### 2、虚拟化技术的分类

虚拟化技术主要分为以下几个大类：
* 平台虚拟化（Platform Virtualization），针对计算机和操作系统的虚拟化。
* 资源虚拟化（Resource Virtualization），针对特定的系统资源的虚拟化，比如内存、存储、网络资源等。
* 应用程序虚拟化（Application Virtualization），包括仿真、模拟、解释技术等。


> 我们通常所说的虚拟化主要是指平台虚拟化技术，通过使用控制程序（Control Program，也被称为 Virtual Machine Monitor 或 Hypervisor），隐藏特定计算平台的实际物理特性，为用户提供抽象的、统一的、模拟的计算环境（称为虚拟机）。虚拟机中运行的操作系统被称为客户机操作系统（Guest OS），运行虚拟机监控器的操作系统被称为主机操作系统（Host OS），当然某些虚拟机监控器可以脱离操作系统直接运行在硬件之上（如 VMWARE 的 ESX 产品）。运行虚拟机的真实系统我们称之为主机系统。

```
cpu虚拟化：
    模拟：emulation
    虚拟：virtulization
         完全虚拟化（full-virtulization）
               BT: 二进制翻译 （软件）
               HVM：硬件辅助的虚拟化 （硬件）
         半虚拟化(para-virtulization)
Memory: 
    进程：线性地址空间
    内核：物理地址空间
    MMU Virtulization
          Intel: EPT, Extended Page Table
          AMD: NTP, Nested Page Table
    TLB virtulization
          tagged TLB
I/O虚拟化的方式：
         模拟: 完全使用软件来模拟真实硬件
         半虚拟化化: 
         IO-through: IO透传
Intel: VT-d
         基于北桥的硬件辅助的虚拟化技术；
		  
```

平台虚拟化技术又可以细分为如下几个子类：
#### 1、完全虚拟化（Full Virtualization）
    完全虚拟化是指虚拟机模拟了完整的底层硬件，包括处理器、物理内存、时钟、外设等，使得为原始硬件设计的操作系统或其它系统软件完全不做任何修改就可以在虚拟机中运行。操作系统与真实硬件之间的交互可以看成是通过一个预先规定的硬件接口进行的。完全虚拟化 VMM 以完整模拟硬件的方式提供全部接口（同时还必须模拟特权指令的执行过程）。
    举例而言，x86 体系结构中，对于操作系统切换进程页表的操作，真实硬件通过提供一个特权 CR3 寄存器来实现该接口，操作系统只需执行 "mov pgtable,%%cr3" 汇编指令即可。完全虚拟化 VMM 必须完整地模拟该接口执行的全过程。如果硬件不提供虚拟化的特殊支持，那么这个模拟过程将会十分复杂：一般而言，VMM 必须运行在最高优先级来完全控制主机系统，而 Guest OS 需要降级运行，从而不能执行特权操作。当 Guest OS 执行前面的特权汇编指令时，主机系统产生异常（General Protection Exception），执行控制权重新从 Guest OS 转到 VMM 手中。VMM 事先分配一个变量作为影子 CR3 寄存器给 Guest OS，将 pgtable 代表的客户机物理地址（Guest Physical Address）填入影子 CR3 寄存器，然后 VMM 还需要 pgtable 翻译成主机物理地址（Host Physical Address）并填入物理 CR3 寄存器，最后返回到 Guest OS中。随后 VMM 还将处理复杂的 Guest OS 缺页异常（Page Fault）。
    比较著名的完全虚拟化 VMM 有 Microsoft Virtual PC、VMware Workstation、Sun Virtual Box、Parallels Desktop for Mac 和 QEMU。
	
![](http://s3.51cto.com/wyfs02/M00/78/E0/wKioL1aEmgfzmpiCAAFA9bEA4Gc653.jpg)	

#### 2、半虚拟化（Para Virtualization）
    这是一种修改 Guest OS 部分访问特权状态的代码以便直接与 VMM 交互的技术。在半虚拟化虚拟机中，部分硬件接口以软件的形式提供给客户机操作系统，这可以通过 Hypercall（VMM 提供给 Guest OS 的直接调用，与系统调用类似）的方式来提供。
    例如，Guest OS 把切换页表的代码修改为调用 Hypercall 来直接完成修改影子 CR3 寄存器和翻译地址的工作。由于不需要产生额外的异常和模拟部分硬件执行流程，半虚拟化可以大幅度提高性能。
    比较著名的 半虚拟化VMM 有 Denali、Xen。
	
	
![](http://s5.51cto.com/wyfs02/M02/78/E2/wKioL1aEpLXQHMAQAAEgQ_1xTOE214.jpg)


#### 3、硬件辅助虚拟化（Hardware-Assisted Virtualization Machine）
    硬件辅助虚拟化是指借助硬件（主要是主机处理器）的支持来实现高效的全虚拟化。例如有了 Intel-VT 技术的支持，Guest OS 和 VMM 的执行环境自动地完全隔离开来，Guest OS 有自己的“全套寄存器”，可以直接运行在最高级别。因此在上面的例子中，Guest OS 能够执行修改页表的汇编指令。Intel-VT 和 AMD-V 是目前 x86 体系结构上可用的两种硬件辅助虚拟化技术。

	
![](http://s4.51cto.com/wyfs02/M00/78/E4/wKiom1aErpzg8Yr4AAG0lNxWepc987.jpg)	
	
#### 4、操作系统级虚拟化（Operating System Level Virtualization）
    在传统操作系统中，所有用户的进程本质上是在同一个操作系统的实例中运行，因此内核或应用程序的缺陷可能影响到其它进程。操作系统级虚拟化是一种在服务器操作系统中使用的轻量级的虚拟化技术，内核通过创建多个虚拟的操作系统实例（内核和库）来隔离不同的进程，不同实例中的进程完全不了解对方的存在。
    比较著名的有 Solaris Container [2]，FreeBSD Jail 和 OpenVZ 等。

    这种分类并不是绝对的，一个优秀的虚拟化软件往往融合了多项技术。例如 VMware Workstation 是一个著名的全虚拟化的 VMM，但是它使用了一种被称为动态二进制（BT）翻译的技术把对特权状态的访问转换成对影子状态的操作，从而避免了低效的 Trap-And-Emulate 的处理方式，这与完全虚拟化相似，只不过完全虚拟化是静态地修改程序代码。对于超虚拟化而言，如果能利用硬件特性，那么虚拟机的管理将会大大简化，同时还能保持较高的性能。
    本文讨论的虚拟化技术只针对 x86 平台（含 AMD 64），并假定虚拟机中运行的 Guest OS 也是为 x86 平台设计的。

* 内存的虚拟化

![](http://s3.51cto.com/wyfs02/M01/78/E4/wKiom1aEr_SBEcnfAAF9zCOcQtI927.jpg)

	
	
	

    上图中的左半部分是正常情况下内存的使用情况，线性地址到物理地址的一个转换，通常应用程序使用的内存地址是线性地址，需要通过MMU地址转换的一个硬件来实现线性地址到物理地址的一个转换。上图的右半部分是虚拟化情况下内存是如何实现线性地址到物理地址的转换,虚拟机中的OS获得是有Host OS分配的线性地址，Guest OS中的应用程序获取的是相对于Guest os来说的线性地址，那么对于这个线性地址来说最终只能转换成线性地址，无法转换成正在的物理地址，虚拟化中就提供了一个虚拟的MMU直接将Guest OS中的线性地址直接转换成真正的物理地址，但是这个虚拟的MMU毕竟是虚拟出来的没有真实的硬件MMU性能好而且虚拟MMU的开发也很复杂，每一个Guest都要有一个MMU，这对内存消耗太大。对于这种技术我们称作是影子页表技术。
    此后Intel开创了EPT(Extended Page Tables)技术,相对于影子页表技术来说EPT技术降低了内存虚拟化的难度，EPT技术是直接在硬件上实现了Guest OS中的线性地址到Guest OS中的物理地址再到Host OS中的物理地址的两次转换。


* IO的虚拟化	

![](http://s1.51cto.com/wyfs02/M00/78/E5/wKiom1aEs32BZrV3AAHwoyBH4ao025.jpg)


最左边IO虚拟化的使用的是仿真技术:
    仿真(emulation)是一个完全通过软件程序来模拟硬件的技术。早期虚拟化都才采用这种方案来虚拟网络设备。常见仿真软件有QEMU、VMware WorkStation、VirtualBox。Emulation网
中间的IO虚拟化是Para-virtualization:
    Para-virtualization又称半虚拟化，最早由Citrix的Xen提出使用。在半虚拟化模型中，物理硬件资源统一由 Hypervisor管理，由Hypervisor提供资源调用接口。虚拟子机通过特定的调用接口与Hypervisor通信，然后完整I/O资源控制操.
最右边的是直接IO
    Hypervisor将一个PCI设备(可以是网卡、USB、光驱)直接分配给指定虚拟子机单独访问。为了安全和稳定性考虑，，pass-through使用通常结合intel VT-D(AMD也有类似技术)来使用，通过iommu保证虚拟子机之间内存访问不冲突。这种技术在VMware上叫VMDirectPath I/O，其他方案中没有找到相关专门名词。

	
### 3、Intel-VT 硬件辅助虚拟化技术详解

2005 年冬天，英特尔带来了业内首个面向台式机的硬件辅助虚拟化技术 Intel-VT 及相关的处理器产品，从而拉开了 IA 架构虚拟化技术应用的新时代大幕。支持虚拟化技术的处理器带有特别优化过的指令集来自动控制虚拟化过程，从而极大简化 VMM 的设计，VMM 的性能也能得到很大提高。其中 IA-32 处理器的虚拟化技术称为 VT-x，安腾处理器的虚拟化技术称为 VT-i。AMD 公司也推出了自己的虚拟化解决方案，称为 AMD-V。尽管 Intel-VT 和 AMD-V 并不完全相同，但是基本思想和数据结构却是相似的，本文只讨论 Intel-VT-x 技术。

* 新增的两种操作模式

VT-x 为 IA 32 处理器增加了两种操作模式：VMX root operation 和 VMX non-root operation。VMM 自己运行在 VMX root operation 模式，VMX non-root operation 模式则由 Guest OS 使用。两种操作模式都支持 Ring 0 ~ Ring 3 这 4 个特权级，因此 VMM 和 Guest OS 都可以自由选择它们所期望的运行级别。

这两种操作模式可以互相转换。运行在 VMX root operation 模式下的 VMM 通过显式调用 VMLAUNCH 或 VMRESUME 指令切换到 VMX non-root operation 模式，硬件自动加载 Guest OS的上下文，于是 Guest OS 获得运行，这种转换称为 VM entry。Guest OS 运行过程中遇到需要 VMM 处理的事件，例如外部中断或缺页异常，或者主动调用 VMCALL 指令调用 VMM 的服务的时候（与系统调用类似），硬件自动挂起 Guest OS，切换到 VMX root operation 模式，恢复 VMM 的运行，这种转换称为 VM exit。VMX root operation 模式下软件的行为与在没有 VT-x 技术的处理器上的行为基本一致；而VMX non-root operation 模式则有很大不同，最主要的区别是此时运行某些指令或遇到某些事件时，发生 VM exit。

* 虚拟机控制块

VMM 和 Guest OS 共享底层的处理器资源，因此硬件需要一个物理内存区域来自动保存或恢复彼此执行的上下文。这个区域称为虚拟机控制块（VMCS），包括客户机状态区（Guest State Area），主机状态区（Host State Area）和执行控制区。VM entry 时，硬件自动从客户机状态区加载 Guest OS 的上下文。并不需要保存 VMM 的上下文，原因与中断处理程序类似，因为 VMM 如果开始运行，就不会受到 Guest OS的干扰，只有 VMM 将工作彻底处理完毕才可能自行切换到 Guest OS。而 VMM 的下次运行必然是处理一个新的事件，因此每次 VMM entry 时， VMM 都从一个通用事件处理函数开始执行；VM exit 时，硬件自动将 Guest OS 的上下文保存在客户机状态区，从主机状态区中加载 VMM 的通用事件处理函数的地址，VMM 开始执行。而执行控制区存放的则是可以操控 VM entry 和 exit 的标志位，例如标记哪些事件可以导致 VM exit，VM entry 时准备自动给 Guest OS “塞”入哪种中断等等。

客户机状态区和主机状态区都应该包含部分物理寄存器的信息，例如控制寄存器 CR0，CR3，CR4；ESP 和 EIP（如果处理器支持 64 位扩展，则为 RSP，RIP）；CS，SS，DS，ES，FS，GS 等段寄存器及其描述项；TR，GDTR，IDTR 寄存器；IA32_SYSENTER_CS，IA32_SYSENTER_ESP，IA32_SYSENTER_EIP 和 IA32_PERF_GLOBAL_CTRL 等 MSR 寄存器。客户机状态区并不包括通用寄存器的内容，VMM 自行决定是否在 VM exit 的时候保存它们，从而提高了系统性能。客户机状态区还包括非物理寄存器的内容，比如一个 32 位的 Active State 值表明 Guest OS 执行时处理器所处的活跃状态，如果正常执行指令就是处于 Active 状态，如果触发了三重故障（Triple Fault）或其它严重错误就处于 Shutdown 状态，等等。

* 解决纯软件虚拟化技术面临的挑战

首先，由于新的操作模式的引入，VMM 和 Guest OS 的执行由硬件自动隔离开来，任何关键的事件都可以将系统控制权自动转移到 VMM，因此 VMM 能够完全控制系统的全部资源。

其次，Guest OS 可以运行在它所期望的最高特权级别，因此特权级压缩和特权级别名的问题迎刃而解，而且 Guest OS 中的系统调用也不会触发 VM exit。

硬件使用物理地址访问虚拟机控制块（VMCS），而 VMCS 保存了 VMM 和 Guest OS 各自的 IDTR 和 CR3 寄存器，因此 VMM 可以拥有独立的地址空间，Guest OS 能够完全控制自己的地址空间，地址空间压缩的问题也不存在了。

中断和异常虚拟化的问题也得到了很好的解决。VMM 只用简单地设置需要转发的虚拟中断或异常，在 VM entry 时，硬件自动调用 Guest OS 的中断和异常处理程序，大大简化 VMM 的设计。同时，Guest OS 对中断的屏蔽及解除可以不触发 VM exit，从而提高了性能。而且 VMM 还可以设置当 Guest OS 解除中断屏蔽时触发 VM exit，因此能够及时地转发积累的虚拟中断和异常。	
	
	
### 4、纯软件虚拟化技术的原理及面临的挑战

**虚拟机监控器应当具备的条件**

1974 年，Popek 和 Goldberg 在《Formal Requirements for Virtualizable Third Generation Architectures》[3] 论文中提出了一组称为虚拟化准则的充分条件，满足这些条件的控制程序可以被称为虚拟机监控器（Virtual Machine Monitor，简称 VMM）：
资源控制。控制程序必须能够管理所有的系统资源。
等价性。在控制程序管理下运行的程序（包括操作系统），除时序和资源可用性之外的行为应该与没有控制程序时的完全一致，且预先编写的特权指令可以自由地执行。
效率性。绝大多数的客户机指令应该由主机硬件直接执行而无需控制程序的参与。

尽管基于简化的假设，但上述条件仍为评判一个计算机体系结构是否能够有效支持虚拟化提供了一个便利方法，也为设计可虚拟化计算机架构给出了指导原则。

1、原理简介

我们知道，传统的 x86 体系结构缺乏必要的硬件支持，任何虚拟机监控器都无法直接满足上述条件，所以不是一个可虚拟化架构，但是我们可以使用纯软件实现的方式构造虚拟机监控器。

虚拟机是对真实计算环境的抽象和模拟，VMM 需要为每个虚拟机分配一套数据结构来管理它们状态，包括虚拟处理器的全套寄存器，物理内存的使用情况，虚拟设备的状态等等。VMM 调度虚拟机时，将其部分状态恢复到主机系统中。并非所有的状态都需要恢复，例如主机 CR3 寄存器中存放的是 VMM 设置的页表物理地址，而不是 Guest OS 设置的值。主机处理器直接运行 Guest OS 的机器指令，由于 Guest OS运行在低特权级别，当访问主机系统的特权状态（如写 GDT 寄存器）时，权限不足导致主机处理器产生异常，将运行权自动交还给 VMM。此外，外部中断的到来也会导致 VMM 的运行。VMM 可能需要先将 该虚拟机的当前状态写回到状态数据结构中，分析虚拟机被挂起的原因，然后代表 Guest OS 执行相应的特权操作。最简单的情况，如Guest OS 对 CR3 寄存器的修改，只需要更新虚拟机的状态数据结构即可。一般而言，大部分情况下，VMM 需要经过复杂的流程才能完成原本简单的操作。最后 VMM 将运行权还给 Guest OS，Guest OS 从上次被中断的地方继续执行，或处理 VMM “塞”入的虚拟中断和异常。这种经典的虚拟机运行方式被称为 Trap-And-Emulate，虚拟机对于 Guest OS 完全透明，Guest OS 不需要任何修改，但是 VMM 的设计会比较复杂，系统整体性能受到明显的损害。

2、面临的挑战

在设计纯软件 VMM 的时候，需要解决如下挑战：

确保 VMM 控制所有的系统资源。

x86 处理器有 4 个特权级别，Ring 0 ~ Ring 3，只有运行在 Ring 0 ~ 2 级时，处理器才可以访问特权资源或执行特权指令；运行在 Ring 0 级时，处理器可以访问所有的特权状态。x86 平台上的操作系统一般只使用 Ring 0 和 Ring 3 这两个级别，操作系统运行在 Ring 0 级，用户进程运行在 Ring 3 级。为了满足上面的第一个充分条件-资源控制，VMM 自己必须运行在 Ring 0 级，同时为了避免 Guest OS 控制系统资源，Guest OS 不得不降低自身的运行级别，运行在 Ring 1 或 Ring 3 级（Ring 2 不使用）。

**特权级压缩（Ring Compression）**

VMM 使用分页或段限制的方式保护物理内存的访问，但是 64 位模式下段限制不起作用，而分页又不区分 Ring 0, 1, 2。为了统一和简化 VMM的设计，Guest OS 只能和 Guest 进程一样运行在 Ring 3 级。VMM 必须监视 Guest OS 对 GDT、IDT 等特权资源的设置，防止 Guest OS 运行在 Ring 0级，同时又要保护降级后的 Guest OS 不受 Guest 进程的主动攻击或无意破坏。

	
**特权级别名（Ring Alias）**

特权级别名是指 Guest OS 在虚拟机中运行的级别并不是它所期望的。VMM 必须保证 Guest OS 不能获知正在虚拟机中运行这一事实，否则可能打破等价性条件。例如，x86 处理器的特权级别存放在 CS 代码段寄存器内，Guest OS 可以使用非特权 push 指令将 CS 寄存器压栈，然后 pop 出来检查该值。又如，Guest OS 在低特权级别时读取特权寄存器 GDT、LDT、IDT 和 TR，并不发生异常，从而可能发现这些值与自己期望的不一样。为了解决这个挑战，VMM 可以使用动态二进制翻译的技术，例如预先把 “push %%cs” 指令替换，在栈上存放一个影子 CS 寄存器值；又如，可以把读取 GDT 寄存器的操作“sgdt dest”改为“movl fake_gdt, dest”。

	
	
**地址空间压缩（Address Space Compression）**

地址空间压缩是指 VMM 必须在Guest OS 的地址空间中保留一部分供其使用。例如，中断描述表寄存器（IDT Register）中存放的是中断描述表的线性地址，如果 Guest OS 运行过程中来了外部中断或触发处理器异常，必须保证运行权马上转移到 VMM 中，因此 VMM 需要将 Guest OS 的一部分线性地址空间映射成自己的中断描述表的主机物理地址。VMM 可以完全运行在 Guest OS 的地址空间中，也可以拥有独立的地址空间，后者的话，VMM 只占用 Guest OS 很少的地址空间，用于存放中断描述表和全局描述符表（GDT）等重要的特权状态。无论如何哪种情况，VMM 应该防止 Guest OS 直接读取和修改这部分地址空间。

**处理 Guest OS 的缺页异常**

内存是一种非常重要的系统资源，VMM 必须全权管理，Guest OS 理解的物理地址只是客户机物理地址（Guest Physical Address），并不是最终的主机物理地址（Host Physical Address）。当 Guest OS 发生缺页异常时，VMM 需要知道缺页异常的原因，是 Guest 进程试图访问没有权限的地址，或是客户机线性地址（Guest Linear Address）尚未翻译成 Guest Physical Address，还是客户机物理地址尚未翻译成主机物理地址。一种可行的解决方法是 VMM 为 Guest OS 的每个进程的页表构造一个影子页表，维护 Guest Linear Address 到 Host Physical Address 的映射，主机 CR3 寄存器存放这个影子页表的物理内存地址。VMM 同时维护一个 Guest OS 全局的 Guest Physical Address 到 Host Physical Address 的映射表。发生缺页异常的地址总是Guest Linear Address，VMM 先去 Guest OS 中的页表检查原因，如果页表项已经建立，即对应的Guest Physical Address 存在，说明尚未建立到 Host Physical Address的映射，那么 VMM 分配一页物理内存，将影子页表和映射表更新；否则，VMM 返回到 Guest OS，由 Guest OS 自己处理该异常。

	
**处理 Guest OS 中的系统调用**

系统调用是操作系统提供给用户的服务例程，使用非常频繁。最新的操作系统一般使用 SYSENTER/SYSEXIT 指令对来实现快速系统调用。SYSENTER 指令通过IA32_SYSENTER_CS，IA32_SYSENTER_EIP 和 IA32_SYSENTER_ESP 这 3 个 MSR（Model Specific Register）寄存器直接转到 Ring 0级；而 SYSEXIT 指令不在 Ring 0 级执行的话将触发异常。因此，如果 VMM 只能采取 Trap-And-Emulate 的方式处理这 2 条指令的话，整体性能将会受到极大损害。

	
**转发虚拟的中断和异常**

所有的外部中断和主机处理器的异常直接由 VMM 接管，VMM 构造必需的虚拟中断和异常，然后转发给 Guest OS。VMM 需要模拟硬件和操作系统对中断和异常的完整处理流程，例如 VMM 先要在 Guest OS 当前的内核栈上压入一些信息，然后找到 Guest OS 相应处理例程的地址，并跳转过去。VMM 必须对不同的 Guest OS 的内部工作流程比较清楚，这增加了 VMM 的实现难度。同时，Guest OS 可能频繁地屏蔽中断和启用中断，这两个操作访问特权寄存器 EFLAGS，必须由 VMM 模拟完成，性能因此会受到损害。 Guest OS 重新启用中断时，VMM 需要及时地获知这一情况，并将积累的虚拟中断转发。

**Guest OS 频繁访问特权资源**

Guest OS对特权资源的每次访问都会触发处理器异常，然后由 VMM 模拟执行，如果访问过于频繁，则系统整体性能将会受到极大损害。比如对中断的屏蔽和启用，cli（Clear Interrupts）指令在 Pentium 4 处理器上需要花费 60 个时钟周期（cycle）。又如，处理器本地高级可编程中断处理器（Local APIC）上有一个操作系统可修改的任务优先级寄存器（Task-Priority Register），IO-APIC 将外部中断转发到 TPR 值最低的处理器上（期望该处理器正在执行低优先级的线程），从而优化中断的处理。TPR 是一个特权寄存器，某些操作系统会频繁设置（Linux Kernel只在初始化阶段为每个处理器的 TPR 设置相同的值）。

软件 VMM 所遇到的以上挑战从本质上来说是因为 Guest OS 无法运行在它所期望的最高特权级，传统的 Trap-And-Emulate 处理方式虽然以透明的方式基本解决上述挑战，但是带来极大的设计复杂性和性能下降。当前比较先进的虚拟化软件结合使用二进制翻译和超虚拟化的技术，核心思想是动态或静态地改变 Guest OS 对特权状态访问的操作，尽量减少产生不必要的硬件异常，同时简化 VMM 的设计。


	
	
### 参考

虚拟化技术漫谈：<http://wzlinux.blog.51cto.com/8021085/1730261>
