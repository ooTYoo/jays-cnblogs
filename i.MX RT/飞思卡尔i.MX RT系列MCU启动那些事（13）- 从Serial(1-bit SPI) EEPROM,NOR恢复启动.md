----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的Serial EEPROM/NOR恢复启动**。  

　　在前几篇里痞子衡介绍的Boot Device都属于主动启动的Primary Boot Device（Serial NOR/NAND, Parallel NOR/NAND， SD/eMMC），试想一下如果遇到这样的情况，你选择启动的某个Primary Boot Device正常工作一段时间后某次开机突然因为某种未知原因无法启动了，此时系统无法正常工作，但如果你希望系统能够有一定的容错/鲁棒能力，即使这种场合下也能够保证基本工作，那应该怎么做？别担心，i.MXRT BootROM提供了一种解决方案，即Recovery Boot机制，BootROM支持Serial EEPROM/NOR作为Recovery Boot Device，你只需要将备份application事先放进Recovery Boot Device即可，任何主动启动的Primary Boot Device启动失败，BootROM会自动启动Recovery Boot Device中的备份application保证系统能正常工作，是不是觉得recovery boot很贴心？今天痞子衡就为大家介绍Recovery Boot：

### 一、支持的Serial EEPROM/NOR
　　<font color="Blue">i.MXRT支持加载恢复启动的主要是1-bit SPI接口的EEPROM，除此以外市面上有些Serial(QSPI) NOR也兼容EEPROM命令集（即1bit read/normal read模式），所以这些QSPI NOR也能被i.MXRT用作恢复启动。</font>关于Serial EEPROM基本知识请先看一下痞子衡的另一篇文章 [EEPROM接口事实标准及Serial EEPROM简介](https://www.cnblogs.com/henjay724/p/9251620.html)。    
　　Serial EEPROM/NOR厂商非常多，对应Serial EEPROM/NOR芯片型号也很多，如果你在选型时不确定到底该为i.MXRT选择哪一款Serial EEPROM/NOR时，可选用下面三款芯片，痞子衡均实测过：  

```text
Onsemi CAT25512HU5I-GT3         （EEPROM,    1-bit SPI,    20MHz,      128B Page/512Kb Device）
Micron MT25QL128ABA1ESE-OSIT    （NOR Flash, Multiple I/O, 133MHz-STR, 64B Page/4-32-64KB Sector/128Mb Device）
Spansion S25FL129P              （NOR Flash, Multiple I/O, 80MHz,      256B Page/4-8-64-256KB Sector/128Mb Device）
```

> Note1: BootROM固定使用SPI Mode(0,0)（即CPOL=0, CPHA=0）去访问外部EEPROM/NOR。
> Note2: BootROM主要支持2bytes（存储范围为4Kb - 512Kb） / 3bytes（存储范围为1Mb - 128Mb）地址的外部EEPROM/NOR。

### 二、Serial EEPROM/NOR硬件连接
　　确定了Serial EEPROM/NOR芯片选型后，底下便进入Serial EEPROM/NOR硬件电路设计及与i.MXRT的信号连接环节：  

　　i.MXRT对于Serial EEPROM/NOR的底层接口支持是通过内部LPSPI这个IP实现的，i.MXRT内部一共有4个LPSPI，BootROM对这4组LPSPI都支持，具体pinmux如下：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_SerialEEPROM_lpspi_io_pinmux.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_SerialEEPROM_lpspi_io_pinmux2.PNG" style="zoom:100%" />

　　如下是典型的SPI EEPROM硬件连接设计，示例EEPROM芯片是CAT25512HU5I-GT3，标准1-bit SPI，直接连接即可:  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_SerialEEPROM_cat25512hu5i_pcb1.PNG" style="zoom:100%" />

　　如下是典型的QSPI NOR硬件连接设计，示例NOR芯片是MT25QL128ABA1ESE-OSIT，该NOR芯片为Multiple I/O，数据线为DQ[3:0]，当用作1-bit SPI模式时，仅需连接DQ[1:0]：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_SerialEEPROM_mt25ql128aba_pcb.PNG" style="zoom:100%" />

### 三、Serial EEPROM/NOR加载启动过程
　　确保Serial EEPROM/NOR硬件相关设计无误之后，底下便是下载更新Bootable Image进Serial EEPROM/NOR以供BootROM加载启动了，在下载Bootable image之前有必要先了解Serial EEPROM/NOR的加载启动过程：  

　　痞子衡在启动系列文章的第六篇 [飞思卡尔i.MX RT系列微控制器启动篇（6）- Bootable image格式与加载(elftosb/.bd)](https://www.cnblogs.com/henjay724/p/9125869.html) 里的最后已经介绍过non-XIP image加载启动过程，这个过程其实已经充分地描述了Serial EEPROM/NOR的加载启动过程。  
　　有了non-XIP image加载启动的背景知识，Serial EEPROM/NOR的加载启动过程便是上电之后，在主动选择的Primary Boot Device启动失败之后，BootROM会从Serial EEPROM/NOR起始地址处加载initial image数据（4KB），再根据initial image里的IVT,Boot Data获取Application起始地址以及总长度，然后再将Application全部拷贝到相应SRAM里去启动，其过程如下图所示：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_SerialEEPROM_image_layout.PNG" style="zoom:100%" />

### 四、下载Application进Serial EEPROM/NOR
　　理解了Serial EEPROM/NOR加载启动过程，我们便可以开始使用Flashloader下载Application进Serial EEPROM/NOR芯片中：  

　　痞子衡在启动系列文章的第四篇 [飞思卡尔i.MX RT系列微控制器启动篇（4）- Flashloader初体验(blhost)](https://www.cnblogs.com/henjay724/p/9098577.html) 和第六篇 [飞思卡尔i.MX RT系列微控制器启动篇（6）- Bootable image格式与加载(elftosb/.bd)](https://www.cnblogs.com/henjay724/p/9125869.html) 里分别介绍了Flashloader的基本使用以及如何将你的Application制作成Bootable image，后续内容假定你已经制作好一个Bootable image并且使用blhost工具与Flashloader建立了基本通信，正要开始将Bootable image下载进Serial EEPROM/NOR。  

　　Serial EEPROM/NOR也支持configuration block，只不过configuration block对于BootROM恢复启动而言不是必需的，configuration block结构原型是下面的spi_nor_eeprom_config_t，在本文里暂不使能configuration block。  

```C
//! @brief Serial NOR/EEPROM peripheral Config block structure
typedef struct __spi_nor_eeprom_peripheral_config
{
    uint8_t spiIndex;     //!< Index of SPI module
    uint8_t spiPcsx;      //!< PCS instance of SPI module
    uint8_t reserved0[2]; //!< Reserved0
    uint32_t spiSpeed_Hz; //!< SPI transfer speed to connected NOR/EEPROM
} spi_nor_eeprom_peripheral_config_t;

//! @brief Serial NOR/EEPROM Config block structure
typedef struct __serial_nor_eeprom_config
{
    uint8_t memoryType;                            //!< Determines the connected memory type
    uint8_t addressLengthInBits;                   //!< Nor/Eeprom address length
    uint8_t waitTime;                              //!< Wait time before read
    uint8_t reserved0;                             //!< Reserved0
    uint32_t memorySizeInBytes;                    //!< Nor/Eeprom memory size
    uint32_t pageSizeInBytes;                      //!< Nor/Eeprom page size
    uint32_t sectorSizeInBytes;                    //!< Nor/Eeprom sector size
    serial_nor_eeprom_command_set commandSet;      //!< Nor/Eeprom command set
} serial_nor_eeprom_config_t;

//! @brief Serial NOR/EEPROM Config block structure
typedef struct __spi_nor_eeprom_config
{
    uint32_t tag;                                 //!< [0x000-0x003]
    uint32_t version;                             //!< [0x004-0x007]
    spi_nor_eeprom_peripheral_config_t spiConfig; //!< [0x008-0x00f]
    serial_nor_eeprom_config_t memoryConfig;      //!< [0x010-0x02b]
} spi_nor_eeprom_config_t;
```

　　前面扯了些没用的，那么到底怎么样将Bootable image数据下载进Serial EEPROM/NOR中呢？当然还是靠Flashloader工具，我们只需要提供简化的8byte配置数据即可。下面是一种Application下载更新示例（该示例适用于 Serial NOR芯片MT25QL128ABA1ESE-OSIT）：  

```text
// 在SRAM里临时存储Serial EEPROM/NOR配置数据
blhost -p COMx -- fill-memory 0x2000 0x4 0xC1100500 // SPI1, PCS0, NOR Flash, 256B Page, 4KB Sector, 16MB Device
blhost -p COMx -- fill-memory 0x2004 0x4 0x00000000 // 20MHz SPI Speed

// 使用Serial EEPROM/NOR配置数据去配置SPI接口
blhost -p COMx -- configure-memory 0x110 0x2000
```

　　在上述示例里痞子衡首先使用了fill-memory命令在0x2000地址处暂存了8byte配置数据，然后通过config-memory将这8byte数据里的信息配置到Flashloader的Serial EEPROM/NOR接口中，实际上这2个命令成功执行后，你就可以开始使用Flashloader下载Bootable image了。那么这8byte配置数据到底是怎么组织的？详见下表：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_SerialEEPROM_img_option.PNG" style="zoom:100%" />

　　从上表我们可以知道，其实这8byte数据提供的配置信息主要是SPI连接以及EEPROM/NOR Device属性配置。configure-memory命令执行成功之后，底下image的下载很简单，只需要将Bootable image从Serial EEPROM/NOR起始地址开始下载即可，具体步骤如下：    

```text
// 擦除Serial EEPROM/NOR并将image下载进Serial EEPROM/NOR
blhost -p COMx -- flash-erase-region 0x0 0x20000 0x110
blhost -p COMx -- write-memory 0x0 ivt_image.bin 0x110
```

　　Bootable image下载成功之后，我们可以试着用read-memory从Serial EEPROM/NOR芯片里读回IVT,BootData,Application确认一下，Bootable image起始地址在0x0，那么IVT,BootData应该在0x400，Application应该在0x2000：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_SerialEEPROM_readback_img.PNG" style="zoom:100%" />

　　至此，Application的下载工作便结束了。  

### 五、进入Serial EEPROM/NOR备份启动模式
　　Application已经被成功下载进Serial EEPROM/NOR芯片之后，此时我们便可以开始设置芯片从Serial EEPROM/NOR启动：  

　　在进入Boot Device选择之前，你首先需要设置BOOT_MODE[1:0]=2'b10，即芯片处于Internal Boot模式，并且确认BT_FUSE_SEL（eFUSE偏移0x460处的32bit配置数据的bit4）为1'b0，或者也可以设置BOOT_MODE[1:0]=2'b00，即芯片处于Boot From Fuses模式，并且将BT_FUSE_SEL（eFUSE偏移0x460处的32bit配置数据的bit4）烧写为1'b1，这里看不懂的朋友请温习痞子衡前面的文章 [飞思卡尔i.MX RT系列微控制器启动篇（2）- Boot配置(BOOT Pin/eFUSE)](http://www.cnblogs.com/henjay724/p/9034563.html)。  
　　设置好正确Boot模式后，再来选择Boot Device，Serial EEPROM/NOR属于Recovery Boot Device，并不是可以主动选择启动的Boot device，所以并没有BOOT_CFG pin或者eFUSE来配置选择直接从Serial EEPROM/NOR启动，如果想验证从Seril EEPROM/NOR启动是否能成功，你需要确保当前BOOT_CFG/eFUSE决定的Primary Boot Device中没有Bootable image。  

### 六、配置eFUSE启动Serial EEPROM/NOR
　　设置好芯片启动模式是从Serial EEPROM/NOR启动之后，我们还需要最后关注一下与Serial EEPROM/NOR相关的具体特性配置：  

　　Serial EEPROM/NOR的配置相对还是比较简单的，只有4部分：Recovery Boot Enable、SPI Speed、SPI addressing、SPI Port，其中Recovery Boot Enable是一定要开启的，SPI Port要根据板级线路设计而定，SPI addressing根据选用的Serial EEPROM/NOR型号而定，SPI Speed是唯一的可以自由配置的选项（当然不能超过所选Serial EEPROM/NOR的最高速度）。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_SerialEEPROM_fusemap.PNG" style="zoom:100%" />

### 七、几个注意事项
> 1. 市面上基本大于64KB的QSPI NOR均兼容EEPROM命令集（即1bit read/normal read模式）。
> 2. 虽然从Serial EEPROM/NOR启动的设计目的是用于recovery boot，但如果你硬要将Serial EEPROM/NOR作为系统里的唯一Boot Device，也并不是不可以，你需要在板级设计时考虑与Primary Boot Device连接的i.MXRT相关引脚在上电时的电平转换（BootROM总是会尝试先启动Primary Boot Device）的影响。

　　上述所有步骤全部完成之后，复位芯片你就应该能看到你放在Serial EEPROM/NOR里的Application已经正常地启动了。  

　　至此，飞思卡尔i.MX RT系列MCU的Serial EEPROM/NOR恢复启动痞子衡便介绍完毕了，掌声在哪里~~~ 

