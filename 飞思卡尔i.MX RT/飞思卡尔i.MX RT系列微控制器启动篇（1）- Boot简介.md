----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的BootROM功能简介**。  

　　截止目前为止i.MX RT系列已公布的芯片有三款i.MXRT105x, i.MXRT102x, i.MXRT106x，所以本文的研究对象便是这三款芯片，从参考手册来看，这三款芯片的BootROM功能差别不大，所以一篇文章可以概括这三款芯片的BootROM特性。  

### 一、Boot基本原理
#### 1.1 从内部FLASH启动
　　Boot是任何一款MCU都有的特性。<font color="Blue">提及Boot，首先应该联想到的是FLASH，通常Cortex-M微控制器芯片内部一般都会集成FLASH（从FLASH分类上来看应该属于Parallel NOR FLASH），你的Application代码都是保存在FLASH里，每次上电CPU会自动从FLASH里获取Application代码并执行，这个行为就是Boot</font>。  
　　大家都知道，ARM Cortex-M内存使用的是统一编址，32bit总线的地址空间是4GB (0x00000000 - 0xFFFFFFFF)。打开最新的Arm®v6/7/8-M Architecture Reference Manual手册找到如下system address map表，你会发现ARM已经将这4GB空间内容给初步规划好了，各ARM Cortex-M微控制器厂商在设计芯片时一般都会遵守ARM规定。  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ArmMemMap.PNG" style="zoom:100%" />
　　从上述system address map表中我们可以知道，ARM 4GB空间的前512MB(0x00000000 - 0x1FFFFFFF)规划为非易失性存储器空间。看到这，你是不是明白了为啥各大厂商生产的Cortex-M芯片内部FLASH地址总是从0x0开始，因为仅含FLASH的芯片上电启动默认都是从0x0地址开始获取Application的初始PC和SP开始Boot。  

#### 1.2 BootROM是什么
　　大家是不是也会经常在芯片参考手册里看到BootROM的字眼，BootROM是什么？<font color="Blue">BootROM其实是芯片在出厂前固化在ROM里的一段Bootloader程序。这个Bootloader程序可以帮助你完成FLASH里的Application的更新，而不需要使用额外的外部编程/调试器（比如JLink），除了Application更新之外，当然BootROM也可以完成Application的启动</font>，Bootloader一般提供UART/SPI/I2C/USB接口与上位机进行通信，与Bootloader配套使用的还有一个上位机软件，当芯片从BootROM启动后，通过这个上位机软件与BootROM建立连接，然后可以将你的Application代码（bin/s19/hex格式）下载进芯片FLASH。  
　　BootROM并不是每一款MCU都有的。以飞思卡尔Kinetis系列MCU为例，早期的Kinetis产品比如MKL25并不含ROM，第一款支持ROM的Kinetis芯片是2014年推出的MKL03，而恩智浦的LPC系列以及意法半导体的STM32系列MCU一般都是含ROM的。不同厂商芯片的ROM起始地址可能不一样（Kinetis ROM一般从0x1c000000开始，LPC ROM一般从0x03000000开始，STM32 ROM结束地址是0x1FFFFFFF）。  

#### 1.3 Boot Mode选择
　　<font color="Blue">当芯片既有ROM也有FLASH的时候，便会出现Boot位置选择问题，标准术语称为Boot Mode。芯片上电CPU到底是先从FLASH启动还是先从ROM启动？关于这个问题，各芯片厂商的解决方案不一样</font>。  
　　Kinetis的Boot Mode由FLASH偏移地址0x40d处的值（上电系统会自动将这个值加载到FTFx_FOPT寄存器中）以及NMI pin共同决定。LPC的Boot Mode由ISP[1:0]以及VBUS pins决定。STM32的Boot Mode由BOOT[1:0] pins决定。  
　　下图为MK80的具体Boot Mode：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ModeK80_1.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ModeK80_2.PNG" style="zoom:100%" />
　　下图为LPC54114的具体Boot Mode：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ModeLPC54114_1.PNG" style="zoom:100%" />
　　下图为STM32F407的具体Boot Mode：  
<img src="http://odox9r8vg.bkt.clouddn.com/i.MXRT_Boot_ModeSTM32F407_1.PNG" style="zoom:100%" />

#### 1.4 从内部SRAM启动
　　SRAM存在于任何一款MCU中，它除了可以保存Application数据变量外，当然也可以存放Application代码以供CPU执行。但是SRAM是易失性存储器，存放的数据断电会丢失，所以从SRAM启动跟从FLASH/ROM启动性质不一样。
　　<font color="Blue">从FLASH/ROM启动属于一级启动，不依赖除了Boot Mode选择之外的条件；而从SRAM启动属于二级启动，其需要外部引导一下才能完成</font>。外部引导的方式有两种：一是借助于外部调试器，直接将Application下载进SRAM并将PC指向Application开始执行，其实这就是所谓的在SRAM调试；二是借助于FLASH/ROM中的Bootloader程序，Bootloader会将存放在FLASH（或其他非易失性存储器，或者从上位机直接接收）中的Application先加载到SRAM里然后Jump过去执行。  

#### 1.5 从外部存储器启动
　　有些MCU并没有内部FLASH，所以会支持外接存储器，常见的外部存储器有QSPI NOR/NAND, SD/eMMC, SDRAM, Parallel NOR/NAND, SPI/I2C EEPROM等，MCU内部集成相应的存储器接口控制器，通过接口控制器可以轻松访问这些外部存储器。<font color="Blue">一个没有内部FLASH的MCU肯定会有ROM（BootROM），因为必须要借助BootROM才能Boot存储在外部存储器的Application，所以从外部存储器启动也属于二级启动</font>。  
　　那么怎么理解从外部存储器启动？需要弄明白以下几个问题：  
　　第一个问题：从外部NOR FLASH存储器启动（比如QSPI NOR/Parallel NOR/EEPROM）跟从内部FLASH启动有什么区别？最大的区别是从外部NOR FLASH启动本质上属于二级启动，其无法像内部FLASH那样直接启动，需要由Bootloader引导。即使技术上可以做到存储在外部NOR FLASH里的Application能够原地执行（XIP），但也需要Bootloader完成外部NOR FLASH的初始化以及XIP相关配置。  
　　第二个问题：从外部NAND FLASH存储器启动（比如QSPI NAND/Parallel NAND/SD/eMMC）跟从NOR FLASH启动有什么区别？最大的区别是NAND FLASH无法像NOR FLASH那样可以XIP执行，这是由NAND FLASH原理决定的，因为NAND FLASH是按Page访问的并且允许坏块的存在，这意味着CPU无法直接从NAND FLASH取指和执行，必须先由Bootloader将存放在NAND FLASH中的Application先全部拷贝到内部SRAM中，然后从SRAM启动执行。  
　　第三个问题：从外部SDRAM存储器启动跟从内部SRAM启动有什么区别？这里其实区别倒不大，两个都是易失性存储器，都无法直接启动，不过SRAM是直接挂在系统bus上，而SDRAM是挂在存储器接口控制器上，而后者需要Bootloader去做初始化。  

### 二、i.MXRT Boot
　　在第一部分里讲了Boot基本原理以及各种Boot方式，那么i.MXRT Boot到底属于哪一种？在回答这个问题之前我们先看一下i.MXRT102x的system memory map（i.MXRT105x也类似，区别是ITCM/DTCM/OCRAM的size是512KB）。  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_1020MemMap.PNG" style="zoom:100%" />
　　从memory map里可以看到，i.MXRT支持存储类型一共有三种：一是96KB的ROM（即BootROM）、二是总容量3*256KB的RAM（OCRAM/DTCM/ITCM）、三是分配给外部存储器接口控制器（SEMC/QSPI）的2GB区域。看到这里你应该明白了，<font color="Blue">i.MXRT Boot方式主要是借助BootROM从外部存储器加载Application到内部SRAM/外部SDRAM/原地XIP执行</font>。  
　　那么i.MXRT到底支持从哪些外部存储器加载启动呢？翻看i.MXRT的参考手册里的System Boot章节，可以看到i.MXRT启动支持以下6种外部存储器：  
> * Serial NOR Flash via FlexSPI
> * Serial NAND Flash via FlexSPI
> * Parallel NOR Flash via SEMC
> * RAW NAND Flash via SEMC
> * SD/MMC via uSDHC
> * SPI NOR/EEPROM via LPSPI

　　其中Serial/Parallel NOR这两种Device可以XIP，其他4种Device无法XIP，需要拷贝到内部RAM或外接SDRAM里运行。关于具体如何从这6种Device启动，痞子衡下篇文章接着聊。  

　　至此，飞思卡尔i.MX RT系列MCU的BootROM功能痞子衡便介绍完毕了，掌声在哪里~~~ 

