----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的Parallel NOR启动**。  

　　上一篇讲i.MXRT从Raw NAND启动的文章 [飞思卡尔i.MX RT系列微控制器启动篇（8）- 从Raw NAND启动](https://www.cnblogs.com/henjay724/p/9173425.html) 一经放出，深入广大网友喜爱，短时间内阅读量飙升，这让痞子衡深入鼓舞，所以趁热打铁继续把从Parallel NOR启动也顺便一起讲了，为什么说是顺便呢？因为Parallel NOR与Raw NAND都是并行接口，属于同一门派，且这两种外存设备在i.MXRT内部是通过同一IP（SEMC）实现底层接口通信的，所以了解了Raw NAND启动，再来看Parallel NOR启动会觉得简单很多。话不多说，开讲。  

### 一、支持的Parallel NOR
　　依旧开门见山，<font color="Blue">i.MXRT支持加载启动的主要是兼容CFI标准且内置EPSCD命令集的ADM SLC Parallel NOR，至于数据线宽度，x8,x16都支持；关于时钟模式，i.MXRT105x/i.MXRT102x仅支持Asynchronous，i.MXRT106x既支持Asynchronous也支持Synchronous。</font>关于Parallel NOR基本知识请先看一下痞子衡的另一篇文章 [并行接口NOR标准(CFI)及SLC Parallel NOR简介](https://www.cnblogs.com/henjay724/p/9152535.html)。    
　　Parallel NOR厂商非常多，对应Parallel NOR芯片型号也很多，如果你在选型时不确定到底该为i.MXRT选择哪一款Parallel NOR时，可选用下面三款芯片，痞子衡均实测过：  

```text
Micron MT28EW128ABA1LPC-0SIT    （x8/x16 bits, 32B Page/128KB Block/128Mb Device,   Non-ADM, Asynchronous）
Winbond W29GL128CH9T            （x8/x16 bits, 64B Page/128KB Sector/128Mb Device,  Non-ADM, Asynchronous）
Spansion S29GL128S90TFI020      （x16 bits,    512B Page/128KB Sector/128Mb Device, Non-ADM, Asynchronous）
```

> Note: ADM即地址线与数据线复用，为了减少pin脚，有些Parallel NOR芯片会将低bit地址线与数据线复用，但目前市面上主流Parallel NOR芯片还是Non-ADM（即地址与数据是不复用的）居多，i.MXRT虽不能直接连接Non-ADM NOR芯片，但在i.MXRT与Non-ADM NOR芯片之间使用一片74系列锁存器桥接一下便能正常工作。  

### 二、Parallel NOR硬件连接
　　确定了Parallel NOR芯片选型后，底下便进入Parallel NOR硬件电路设计及与i.MXRT的信号连接环节：  

　　i.MXRT对于Parallel NOR的底层接口支持是通过内部SEMC这个IP实现的，SEMC最多能支持五种设备（SDRAM, NAND, NOR, SRAM, 8080 Display），但SEMC接口信号是复用的，所以同一时刻仅能支持一种设备。下表是SEMC接口复用表，关于NOR接口信号，需要特别说一下的是CE#信号和地址线宽度，从表中我们可以看到<font color="Blue">NOR的CE4#信号有6个，即有6种配置选择，但i.MXRT BootROM固定选择的是SEMC_CSX[0]</font>，至于地址线宽度，<font color="Blue">SEMC本身最大可支持28bits地址宽度，但是i.MXRT BootROM里最大只支持24bits地址宽度（即A0-A23，最大128Mb）</font>，这2点在设计NOR硬件连接时需要特别注意。  

> Note: 对于小容量Paralle NOR芯片（比如512KB，地址线A0-A18），i.MXRT当然也可以支持，SEMC未用的地址线（此处为A19-A23）可不用管。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ParallelNOR_semc_io_mux.PNG" style="zoom:100%" />

　　如下是典型的NOR硬件连接设计，示例NOR芯片是MT28EW128ABA1LPC-0SIT，该NOR芯片为Non-ADM，所以我们使用了一片74ALVT16373锁存器桥接了一下，当WEIM_ADV_B信号为高电平时，锁存器Dx会输出给Qx，即此时WEIM_DATA[15:0]作为地址线输出给A[15:0]，而WEIM_ADV_B信号为低电平时，WEIM_DATA[15:0]就是数据线（即此处WEIM_ADV_B作为ADV#信号是高有效，这在后续配置NOR eFUSE时会涉及到）。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ParallelNOR_mt28ew128aba_pcb.PNG" style="zoom:100%" />


### 三、Parallel NOR加载启动过程
　　确保Parallel NOR硬件相关设计无误之后，底下便是下载更新Bootable Image进Parallel NOR以供BootROM加载启动了，在下载Bootable image之前有必要先了解Parallel NOR的加载启动过程：  

　　痞子衡在启动系列文章的第六篇 [飞思卡尔i.MX RT系列微控制器启动篇（6）- Bootable image格式与加载(elftosb/.bd)](https://www.cnblogs.com/henjay724/p/9125869.html) 里的最后已经介绍过non-XIP image加载启动过程，但实际上那个过程主要适用于存储在外部NAND Flash中Bootable image加载启动，对于存储在外部Parallel NOR的Bootable image而言有一些区别，我们知道NOR Flash是支持XIP执行的，所以从NOR启动有两种选择，一种是Non-XIP，另一种是XIP。  
　　对于Non-XIP启动而言，其基本流程与第六篇里介绍的non-XIP image加载启动过程类似，只有两点区别，<font color="Blue">第一个区别是存储在NOR Flash里的Bootable image中IVT偏移地址是固定在0x1000（对于NAND Flash，偏移固定是0x400）；第二个区别是BootROM加载initial image的大小为12KB（对于NAND Flash，initial image是4KB），且这个initial image的加载不需要经过OCRAM缓存，BootROM是直接从NOR对应的SEMC map region去获取的</font>。  
　　对于XIP启动而言，其基本流程与non-XIP image加载启动过程差异就比较大了，因为整个Bootable image都不需要搬运，BootROM直接从NOR对应的SEMC map region去获取IVT,BootData,Application，而<font color="Blue">BootROM中分配给SEMC NOR的XIP空间为0x90000000 - 0x90FFFFFF，所以XIP执行的Application需要链接在这个空间里。</font>  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ParallelNOR_image_layout.PNG" style="zoom:100%" />

　　有了前面的背景知识，NOR的加载启动过程便是上电之后，BootROM先从NOR起始地址处加载initial image数据（12KB），再根据initial image里的IVT获取Application起始地址，如Application地址是链接在SRAM里，便认为这是个Non-XIP Application，然后再将Application拷贝到相应SRAM里去启动；如Application地址是链接在SEMC NOR XIP空间里，则不需要拷贝，直接原地XIP执行启动。  

### 四、下载Application进Parallel NOR
　　理解了Parallel NOR加载启动过程，我们便可以开始使用Flashloader下载Application进Parallel NOR芯片中：  

　　痞子衡在启动系列文章的第四篇 [飞思卡尔i.MX RT系列微控制器启动篇（4）- Flashloader初体验(blhost)](https://www.cnblogs.com/henjay724/p/9098577.html) 和第六篇 [飞思卡尔i.MX RT系列微控制器启动篇（6）- Bootable image格式与加载(elftosb/.bd)](https://www.cnblogs.com/henjay724/p/9125869.html) 里分别介绍了Flashloader的基本使用以及如何将你的Application制作成Bootable image，但那里面制作的Bootable image主要是用于NAND启动，而对于NOR启动，其用于生成Bootable image的BD文件稍有不同。  
　　先来看Non-XIP的情况，下面是一个Non-XIP的BD文件示例，ivtOffset必须设0x1000，因为startAddress = 0x8000, initialLoadSize = 0x3000，所以Application只读段应从0xb000处开始链接：   

```text
options {
    flags = 0x00;
    # Note: This is an example address, it can be any non-zero address in ITCM region
    startAddress = 0x8000;
    ivtOffset = 0x1000;
    initialLoadSize = 0x3000;
    # Note: This is required if the default entrypoint is not the Reset_Handler 
    #       Please set the entryPointAddress to Reset_Handler address 
    // entryPointAddress = 0xd531;
}

sources {
    elfFile = extern(0);
}

section (0)
{
}
```

　　再来看XIP的情况，下面是一个XIP的BD文件示例，ivtOffset也必须设0x1000，因为startAddress = 0x90000000, initialLoadSize = 0x3000，所以Application只读段应从0x90003000处开始链接：   

```text
options {
    flags = 0x00;
    startAddress = 0x90000000;
    ivtOffset = 0x1000;
    initialLoadSize = 0x3000;
    # Note: This is required if the default entrypoint is not the Reset_Handler 
    #       Please set the entryPointAddress to Reset_Handler address 
    // entryPointAddress = 0x90005531;
}

sources {
    elfFile = extern(0);
}

section (0)
{
}
```

　　假定你已经制作好Bootable image并且使用blhost工具与Flashloader建立了基本通信，正要开始将Bootable image下载进Parallel NOR。  
　　与Raw NAND启动一样，Parallel NOR也支持configuration block，只不过configuration block对于BootROM启动而言不是必需的，configuration block必须放在NOR Flash起始地址处，下面是其结构原型（如果你还有印象的话，你会发现它跟Raw NAND的semc_nand_config_t很像），如果想使能configuration block，你需要手动创建这256bytes数据，并且用其覆盖bootable image的前256bytes，在本文里暂不使能configuration block。  

```C
#define SEMC_NOR_INIT_IMG_SIZE (12u * 1024)
#define SEMC_NOR_MAX_SIZE (16U * 1024 * 1024)

#define SEMC_MEM0_BASE (0x80000000u)
#define SEMC_MEM1_BASE (0x90000000u)
#define SEMC_MEM2_BASE (0xA0000000u)
#define SEMC_MEM3_BASE (0xC0000000u)
#define SEMC_MEM_NOR_AXI_BASE SEMC_MEM1_BASE

typedef struct __semc_nor_config
{
    semc_mem_config_t memConfig; //!< [0x000-0x04f]
    uint8_t vendorType;          //!< [0x050-0x050]
    uint8_t acTimingMode;        //!< [0x051-0x051]
    uint8_t deviceCommandSet;    //!< [0x052-0x052]
    uint8_t reserved0[77];       //!< [0x053-0x09f]
    uint32_t pageSizeInBytes;    //!< [0x0a0-0x0a3]
    uint32_t blockSizeInBytes;   //!< [0x0a4-0x0a7]
    uint32_t blockCount;         //!< [0x0a8-0x0ab]
    uint32_t reserved1[21];      //!< [0x0ac-0x0ff]
} semc_nor_config_t;
```

　　前面铺垫了这么多，终于来到关键地方了，到底怎么样将Bootable image数据下载进Parallel NOR中呢？当然还是靠Flashloader工具，我们只需要提供简化的4byte配置数据即可。下面是一种Application下载更新示例（该示例适用于第二节里介绍的NOR硬件连接）：  

```text
// 在SRAM里临时存储Parallel NOR配置数据
blhost -p COMx -- fill-memory 0x2000 0x4 0xD0000600 (Configure to CSX0, ADV high active, 16bits IO, safe AC timing mode)

// 使用Parallel NOR配置数据去配置Parallel NOR接口
blhost -p COMx -- configure-memory 0x8 0x2000
```

　　在上述示例里痞子衡首先使用了fill-memory命令在0x2000地址处暂存了4byte配置数据，然后通过config-memory将这4byte数据里的信息配置到Flashloader的Parallel NOR接口中，实际上这2个命令成功执行后，你就可以开始使用Flashloader下载Bootable image了。那么这4byte配置数据到底是怎么组织的？详见下表：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ParallelNOR_img_option2.PNG" style="zoom:100%" />

　　从上表我们可以知道，其实这4byte数据提供的配置信息主要是NOR配置，这4byte里真正需要注意的只有两个地方（ADV# Polarity、Data Port Size），其余可用固定配置。  
　　configure-memory命令执行成功之后，底下image的下载很简单，只需要将Bootable image从SEMC NOR起始map地址开始下载即可，具体步骤如下：    

```text
// 擦除Parallel NOR并将image下载进Parallel NOR
blhost -p COMx -- flash-erase-region 0x90000000 0x20000
blhost -p COMx -- write-memory 0x90000000 ivt_image.bin
```

> <font color="Red">Note: 实测发现，RT1050 Flashloader 1.1里使用USB接口去下载Parallel NOR会报kStatus_SemcNOR_ProgramVerifyFailure错误，而使用UART接口下载则正常，应该是USB下载对NOR的支持有缺陷，期望在后续版本的Flashloader里修复这个问题。</font>

　　Bootable image下载成功之后，我们可以试着用read-memory从NOR芯片里读回IVT,BootData,Application确认一下，Bootable image起始地址在0x90000000，那么IVT,BootData应该在0x90001000，Application应该在0x90003000：  
　　Non-XIP Bootable image读回情况如下，检查初始PC可知其链接在SRAM空间  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ParallelNOR_readback_img_nonxip.PNG" style="zoom:100%" />

　　XIP Bootable image读回情况如下，检查初始PC可知其链接在SEMC NOR map空间  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ParallelNOR_readback_img_xip.PNG" style="zoom:100%" />

> <font color="Red">Note: 如果Application是XIP在SEMC NOR空间，其时钟初始化代码不能覆盖BootROM里对于SEMC的相关配置，否则XIP可能会失败。</font>

　　至此，Application的下载工作便结束了。  

### 五、进入Parallel NOR启动模式
　　Application已经被成功下载进Parallel NOR芯片之后，此时我们便可以开始设置芯片从Parallel NOR启动：  

　　在进入Boot Device选择之前，你首先需要确定BOOT_MODE[1:0]=2'b00，即芯片处于Boot From Fuses模式，并且将BT_FUSE_SEL（eFUSE偏移0x460处的32bit配置数据的bit4）烧写为1'b1，这里看不懂的朋友请温习痞子衡前面的文章 [飞思卡尔i.MX RT系列微控制器启动篇（2）- Boot配置(BOOT Pin/eFUSE)](http://www.cnblogs.com/henjay724/p/9034563.html)。  
　　设置好正确Boot模式后，再来选择Boot Device，，你还需要将BOOT_CFG1[7:4]（eFUSE偏移0x450处的32bit配置数据的bit7:4）烧写成4'b0001，此时便进入了从SEMC NOR启动模式。  
　　如果想确保i.MXRT芯片一定正在从Parallel NOR启动，可在芯片上电时使用Jlink调试器或者借助Flashloader读取芯片内部2个寄存器的值，这2个寄存器分别是SRC_SBMR1/2, 我们设的关于启动模式的BOOT_MODE pins/BOOT_CFG pin/eFUSE偏移0x450配置值在上电时会自动加载到SRC_SBMR1/2寄存器里，BootROM主要是根据SRC_SBMR1/2寄存器的值来判断启动模式的。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ParallelNOR_readback_ocotp_src_regs.PNG" style="zoom:100%" />

　　PS: BOOT_MODE[1:0]也可以设为2'b10，即芯片处于Internal Boot模式，此时需要确保BT_FUSE_SEL（eFUSE偏移0x460处的32bit配置数据的bit4）为1'b0和BOOT_CFG1[7:4]这四个pin的输入状态设为4'b0001。  

### 六、配置eFUSE启动Parallel NOR
　　设置好芯片启动模式是从Parallel NOR启动之后，我们还需要最后关注一下与Parallel NOR相关的具体特性配置：  

　　你应该记得我们在使用Flashloader下载Application的时候提供过4bytes的NOR配置数据，这4bytes的NOR配置数据是为了让Flashloader能够正确初始化Parallel NOR接口去访问NOR芯片（主要是写Bootable image），同样BootROM上电也需要初始化Parallel NOR接口去访问NOR芯片（主要是读Bootable image），所以BootROM也需要类似这4bytes NOR配置数据，而BootROM的NOR配置便放在如下的eFUSE区域里：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ParallelNOR_1050a1_1020_fusemap.PNG" style="zoom:100%" />

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_ParallelNOR_1060_fusemap.PNG" style="zoom:100%" />

### 七、几个注意事项
> 1. 市面上Parallel NOR从内置命令集角度分为两大类，一类是以Micron MT28EW系列为代表的EPSCD命令集，另一类是以Micron MT28GU为代表的SFMCD命令集，BootROM本身对于这两类NOR芯片都是支持的，但Flashloader目前只支持内置EPSCD命令集的NOR芯片。

　　至此，飞思卡尔i.MX RT系列MCU的Parallel NOR启动痞子衡便介绍完毕了，掌声在哪里~~~ 

