----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的Boot配置**。  

　　在上一篇文章 [飞思卡尔i.MX RT系列微控制器启动篇（1）- Boot简介](http://www.cnblogs.com/henjay724/p/9031655.html) 里痞子衡为大家介绍了Boot基本原理以及i.MXRT Boot方式简介。今天痞子衡就来重点聊一聊i.MXRT Boot方式具体由哪些配置决定的。  

　　无论是什么芯片里的BootROM，其最核心的功能无非两个：一、从存放Application的存储器中加载执行；二、通过支持的通信接口接收来自Host的Application数据完成更新，所以Boot配置也主要围绕这两个核心功能。  

### 一、Boot行为模式选择
　　BOOT_MODE[1:0] pin是决定i.MXRT Boot行为的最顶层配置，但是与上一篇文章里介绍的Kinetis/LPC/STM32 Boot Mode配置不同的是，i.MXRT上电永远是从ROM里开始启动，此处的BOOT_MODE[1:0]决定的仅是BootROM程序的不同行为模式（执行代码分支），而Kinetis/LPC/STM32 Boot Mode侧重的是决定CPU从ROM还是FLASH里启动。  
#### 1.1 BOOT_MODE[1:0] Pinout
　　下表是BOOT_MODE相关pinout信息，可在参考手册的External Signals and Pin Multiplexing章节中找到。  
　　i.MXRT105x pinout：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxIndex.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxSRC_1050.PNG" style="zoom:100%" />

　　i.MXRT102x pinout：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxIndex.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxSRC_1020.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxSRC_1020_1.PNG" style="zoom:100%" />

#### 1.2 BMOD[1:0]三种模式
　　BOOT_MODE[1:0] pin状态是在POR_B pin上沿时被自动采样存储在芯片内部的寄存器SRC->SBMR2[25:24]中的，这两个bit也叫BMOD[1:0]，BootROM其实是根据BMOD[1:0]的值来决定Boot行为的。（注意：如果改变了BOOT_MODE[1:0] pins的输入状态而使用ONOFF pin（RESET_B）去软复位，Boot行为并不会改变，因为BMOD[1:0]值并未改变）。  
<img src="http://odox9r8vg.bkt.clouddn.com/i.MXRT_Boot_BOOT_MODE_pins_setting.PNG" style="zoom:100%" />
　　从上述Boot MODE pin settings表中我们可以知道，除了BMOD[1:0]=2'b11这种情况是reserved之外，其余三种情况对应三种Boot行为，痞子衡为大家逐一分析，先从最简单的行为模式（Serial Downloader）说起。  

##### 1.2.1 Serial Downloader模式
　　Serial Downloader模式顾名思义即串行下载模式，在这种模式下，BootROM通过指定的USB或者UART口来接收来自Host（恩智浦提供了上位机工具sdphost.exe或者mfgtool）的Application数据，并将数据存储在SRAM中执行，这种模式其实就是从SRAM启动，但是如果用这种模式去Boot Application缺点很明显，每次上电都需要将Application重新下载进SRAM，无法做到脱机自动Boot，所以显然这种模式的主要目的并不是从SRAM启动Application，那它到底有什么用？  
　　其实Serial Downloader模式主要是用来从SRAM中启动Flashloader，恩智浦官方提供了Flashloader程序，Flashloader程序可以用来将你的Application下载进i.MXRT支持的所有外部非易失性存储器中，为后续从外部存储器启动做准备。除此以外Serial Downloader模式还可以用来查看Fuse值。  
　　关于Serial Downloader模式以及Flashloader具体如何应用，痞子衡会在下一篇文章里进一步介绍。  

##### 1.2.2 Boot From Fuses模式
　　Boot From Fuses模式从名字来看其实会让人误解，这个模式并不是从Fuse里加载Application启动的意思，而是根据Fuse里的一些Boot配置值来决定从哪个外部存储器Boot。Fuse是i.MXRT里一块特殊的存储区域，用于存放全部芯片配置信息，其中有一部分配置信息和Boot相关。  
　　在参考手册Fusemap这一章节，可以看到完整的Fuse Map表，其中偏移0x460处的32bit配置数据的bit7是BT_FUSE_SEL，这个bit至关重要，决定了Boot From Fuses模式的主要行为，具体表现如下：  
> * BT_FUSE_SEL=0：表明所有外部存储器中均没有Application，此时Boot From Fuses模式等同于Serial Downloader模式。
> * BT_FUSE_SEL=1：表明有外部存储器中存在有效Application，此时BootROM会根据Fuse中其他Boot配置信息进一步选择指定的外部存储器（Boot Device）去Boot。

　　关于Fuse中其他Boot配置信息，文章后面痞子衡会继续讲。  

##### 1.2.3 Internal Boot模式
　　Internal Boot模式其实跟Boot From Fuses模式（BT_FUSE_SEL=1时）很类似，只是这个模式下BT_FUSE_SEL的意义有点不同，具体表现如下：  
> * BT_FUSE_SEL=0：BootROM完全根据Fuse中Boot配置信息选择指定的Boot Device去Boot。
> * BT_FUSE_SEL=1：BootROM根据BOOT_CFG[x:0] pins和Fuse中Boot配置综合决定Boot Device，其中BOOT_CFG[x:0] pins的配置会覆盖Fuse中意义相同的Boot配置信息。

　　我们可以通过更改BOOT_CFG[x:0] pins输入状态来切换Boot配置，这部分Boot配置在Fuse里也同样存在，但是使用BOOT_CFG[x:0]来更改Boot配置显然比烧写Fuse更方便快捷（也可以认为BOOT_CFG[x:0]主要用于产品开发过程中，待产品开发结束后，应直接用Fuse来锁定Boot配置）。关于BOOT_CFG[x:0] pin具体提供哪些Boot配置，文章后面痞子衡会继续讲。  

### 二、Boot Device类型选择
　　前面痞子衡讲了，无论是Boot From Fuses模式还是Internal Boot模式，最终目的都是为了选择合适的Boot Device加载启动，那么Boot Device到底如何确定？主要取决于BOOT_CFG[x:0] pin以及Fuse里的BOOT_CFG1位。  
　　BOOT_CFG[x:0] pin（在RT105x上是12bit，在RT102x上是10bit）跟Boot Device选择相关的是BOOT_CFG[7:4]这4个bit，对应Fuse里是偏移0x450处32bit配置数据里的bit4-7(也叫BOOT_CFG1[7:4])，本小节主要介绍的是BOOT_CFG1[7:4]。  
#### 2.1 BOOT_CFG[x:0] Pinout
　　下表是BOOT_CFG相关pinout信息，可在参考手册的External Signals and Pin Multiplexing章节中找到。  
　　i.MXRT105x pinout：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxIndex.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxSRC_1050_1.PNG" style="zoom:100%" />

　　i.MXRT102x pinout：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxIndex.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxSRC_1020_2.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_PinMuxSRC_1020_3.PNG" style="zoom:100%" />

#### 2.2 BOOT_CFG1[7:4]六种Device
　　BOOT_CFG1[7:4]用于选择具体Boot Device，无论是RT105x还是RT102x，它们支持的外部存储器种类是一样的，但是这里的具体配置值两个芯片上稍稍有些区别（主要是SD和SEMC NAND不一样）。  
　　下表是BOOT_CFG1[7:4]具体配置值信息，可在参考手册的Fusemap章节中找到。
　　i.MXRT105x device selection：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_BOOTCFG1_1050.PNG" style="zoom:100%" />

　　i.MXRT102x device selection：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_BOOTCFG1_1020.PNG" style="zoom:100%" />

### 三、Boot Device具体配置
　　当Boot Device已经确定之后，底下就是配置该Device具体属性了。假设我们选择了Serial NOR FLASH，但是Serial NOR只是一类FLASH的统称，市面上有非常多的Serial NOR芯片，每个芯片特性可能不完全一样，那么BootROM怎么知道这些不同的Serial NOR芯片的特性呢？还是通过BOOT_CFG[x:0] pin和Fuse来指定。  

#### 3.1 BOOT_CFG[x:0]
　　BOOT_CFG[x:0] pin其实是跟Fuse里是偏移0x450处32bit配置数据里的bit0-x是对应的。  
> * BOOT_CFG[7:0] pin对应的是Fuse BOOT_CFG1[7:0]
> * BOOT_CFG[x:8] pin对应的是Fuse BOOT_CFG2[x-8:0]

　　前面讲过BOOT_CFG1[7:4]是用来确定Boot Device类型的，其余bit就是用来配置当前选择的Boot Device的具体信息，因此不同的Boot Device，这些bit的意义是不一样的。具体在Fuse里统一介绍。  

#### 3.2 eFUSE map
　　前面讲过，Fuse是i.MXRT里一块特殊的存储区域，用于存放全部芯片配置信息，其中有一部分区域分配给Boot。参考手册的Fusemap章节中可见所有bit具体定义，这里痞子衡仅贴出一部分用于示例：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_Fusemap_1050.PNG" style="zoom:100%" />
　　从上表中我们可以看到i.MXRT105x上BOOT_CFG1，BOOT_CFG2共16bit的完整定义，除了BOOT_CFG1[7:4]前面已经介绍过之外，从其余bit的定义来看，确实是与具体Boot Device属性相关的。  
　　这些Boot相关的Fuse定义，在这里逐一解释意义不大，需要结合具体Boot Device一起来看，痞子衡后续会在介绍每个Boot Device启动的文章里再进一步分析。  

　　至此，飞思卡尔i.MX RT系列MCU的Boot配置痞子衡便介绍完毕了，掌声在哪里~~~ 

