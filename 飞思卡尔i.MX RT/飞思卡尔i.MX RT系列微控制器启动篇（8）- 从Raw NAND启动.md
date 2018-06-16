----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的Raw NAND启动**。  

　　前面铺垫了七篇启动系列文章，终于该讲具体Boot Device了，我们知道i.MXRT支持的外部Boot Device共有6种（Serial NOR&NAND、Parallel NOR&NAND、SD/eMMC、SPI NOR/EEPROM），其中最常用的是Serial NOR&NAND，目前各大社区里讨论最火的也是Serial NOR/NAND启动，有不少大神(硬汉eric2013, jicheng0622)已经写过关于Serial NOR&NAND启动的文章，写得非常好，这让痞子衡非常有压力，因此痞子衡决定第一篇Boot Device写较常用但还没有人写过的Raw(Parallel) NAND启动，大家是不是很期待？（请配合说“是”），好，话不多说，开讲。  

### 一、支持的Raw NAND
　　开门见山，<font color="Blue">i.MXRT支持加载启动的主要是兼容ONFI 1.0标准的Asynchronous SLC Raw NAND，至于数据线宽度，x8,x16都支持(一般x8应用比较多)。</font>关于Raw NAND基本知识请先看一下痞子衡的另一篇文章 [并行接口NAND标准(ONFI)及SLC Raw NAND简介](https://www.cnblogs.com/henjay724/p/9152535.html)，本文后续的很多内容均是基于充分了解Raw NAND的前提下开展的。  
　　Raw NAND厂商非常多，对应Raw NAND芯片型号也很多，如果你在选型时不确定到底该为i.MXRT选择哪一款Raw NAND时，可选用下面四款芯片，痞子衡均实测过：  

```text
Macronix MX30LF4GE8AB-TI        （x8 bits, 2KB Page/128KB Block/4Gb Device,  0bit ECC, 3.3V）
Micron MT29F4G08ABBDAWP         （x8 bits, 2KB Page/128KB Block/4Gb Device,  4bit ECC, 1.8V）
Micron MT29F4G08ABAFAWP:D       （x8 bits, 2KB Page/128KB Block/4Gb Device,  4bit ECC, 3.3V）
Micron MT29F16G08ABACAWP:C      （x8 bits, 4KB Page/512KB Block/16Gb Device, 4bit ECC, 3.3V）
```

### 二、Raw NAND硬件连接
　　确定了Raw NAND芯片选型后，底下便进入Raw NAND硬件电路设计及与i.MXRT的信号连接环节：  

　　i.MXRT对于Raw NAND的底层接口支持是通过内部SEMC这个IP实现的，如下是SEMC的内部模块图，从图中我们可以看到，从内部数据总线来看SEMC支持AXI Bus/IP Bus两种（此两种方式会在后续eFUSE配置里看到），而从外部接口来看SEMC最多能支持五种设备（SDRAM, NAND, NOR, SRAM, 8080 Display），NAND是其中一种。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_semc_block_diagram1.PNG" style="zoom:100%" />

　　虽然SEMC最多能支持五种设备，但并不是同时支持的，同一时刻仅能支持一种设备，因此SEMC接口信号必然是复用的，下表是SEMC接口复用表，关于NAND接口信号，需要特别说一下的是CE#信号，从表中我们可以看到<font color="Blue">NAND的CE6#信号有5个，即有5种配置选择，但i.MXRT BootROM固定选择的是SEMC_CSX[0]</font>，这点在设计NAND硬件连接时需要特别注意。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_semc_io_mux.PNG" style="zoom:100%" />

　　如下是典型的NAND硬件连接设计，示例NAND芯片是MX30<font color="Blue">L</font>F4GE8AB-TI（经典的TSOP-48封装，芯片丝印上的<font color="Blue">L</font>表明其是3.3V供电），其中WP#信号没有使能，并且供电选择同时支持3.3V和1.8V（通过R302选择，此处应连2-3），有朋友会疑问，为什么此处要留有2路不同供电电压？因为后期方便我们更换不同的供电输入的NAND芯片。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_mx30lf4g_pcb.png" style="zoom:100%" />

### 三、Raw NAND加载启动过程
　　确保Raw NAND硬件相关设计无误之后，底下便是下载更新Bootable Image进Raw NAND以供BootROM加载启动了，在下载Bootable image之前有必要先了解Raw NAND的加载启动过程：  

　　痞子衡在启动系列文章的第六篇 [飞思卡尔i.MX RT系列微控制器启动篇（6）- Bootable image格式与加载(elftosb/.bd)](https://www.cnblogs.com/henjay724/p/9125869.html) 里的最后已经介绍过non-XIP image加载启动过程，但实际上那个过程对于存储在外部NAND Flash中Bootable image而言还是介绍得不够全面，欠缺FCB/DBBT的处理流程，你肯定会疑问FCB/DBBT是什么？这得从NAND与NOR差异说起，我们知道NOR Flash中所有空间都必须是可用的（即不允许有坏块），这意味着NOR Flash中的Bootable image数据是可以按指定地址连续存放的（即所谓的线性存储），并且Application可以原地XIP执行；但是NAND Flash中常常是有坏块的（出厂坏块，使用中产生坏块），这就导致NAND可用空间地址不可预知并且有可能不连续，因此存储在NAND中的Bootable image数据极有可能并不是连续存放的，并且Bootable image实际存储的起始地址也不一定就是指定的起始地址（即所谓的非线性存储）。  
　　举例来说，如果NAND的block大小为128KB，Firmware（即Bootable Image）大小为260KB，我们指定从NAND地址0x40000处（即block index = 2）开始存储Firmware，但是很不幸的是index为2、4的block均是坏块，那么实际上Firmware被分散存储在了index为3、5、6三个block中，为了将来能正确地从NAND中读回Firmware，我们需要额外记录至少两个信息，一是指定的Firmware起始存储地址0x40000，二是NAND中坏块信息block index 2、4。FCB/DBBT就是用来记录这些额外的信息。  
　　FCB大小为1KB，其主要记录了Firmware信息（地址，长度，份数），以及DBBT地址信息。DBBT大小为1056bytes，其记录了NAND芯片中所有的坏块个数以及位置，DBBT即所谓的坏块表。FCB/DBBT最大可有两份，实际应用中一般只用一份即可，后面介绍均以一份FCB/DBBT为例讲解，<font color="Blue">FCB0永远从NAND地址0x0处（即index为0的block中的第1个Page）开始存放，DBBT0一般放在index为n的block里（其实n是可设的，这在后面使用Flashloader时会讲到，为求简单我们常常设n=1），Firmware 0一般放在index为n+1的block里（Firmware只允许从index为n+1的block及其之后开始存放，Firmware最大可以有8份）。</font>关于FCB/DBBT结构原型，后续会进一步介绍。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_image_layout1.PNG" style="zoom:100%" />

　　有了前面的背景知识，NAND的加载启动过程便是上电之后，BootROM先从NAND起始地址处获取FCB0数据，再根据FCB0里的信息获取DBBT0数据以及Firmware 0起始地址，底下便进入跟NOR Flash一样的加载过程，只不过在加载Firmware 0的过程中需要根据DBBT0坏块表信息自动跳过坏块。如果在读取Firmware 0时，发现部分Firmware数据所在的block是一个坏块，但是这个block没有被记录在DBBT0中，这说明该坏块是新产生的（该新坏块信息会在下一次下载Application时记录在新DBBT中），存在该坏块中的Firmware数据被破坏了，Firmware 0便失效了，BootROM便会尝试按同样的流程去加载Firmware 1、2...7，直到找到有效的Firmware，这就是为什么在NAND中存储多份Firmware的意义。  

### 四、下载Application进Raw NAND
　　理解了Raw NAND加载启动过程，我们便可以开始使用Flashloader下载Application进Raw NAND芯片中：  

　　痞子衡在启动系列文章的第四篇 [飞思卡尔i.MX RT系列微控制器启动篇（4）- Flashloader初体验(blhost)](https://www.cnblogs.com/henjay724/p/9098577.html) 和第六篇 [飞思卡尔i.MX RT系列微控制器启动篇（6）- Bootable image格式与加载(elftosb/.bd)](https://www.cnblogs.com/henjay724/p/9125869.html) 里分别介绍了Flashloader的基本使用以及如何将你的Application制作成Bootable image，后续内容假定你已经制作好一个Bootable image并且使用blhost工具与Flashloader建立了基本通信，正要开始将Bootable image下载进Raw NAND。  
　　前面讲过Raw NAND中除了要有Bootable image（Firmware）之外，还需要有FCB/DBBT，并且FCB/DBBT在Raw NAND中存储的位置是比Bootable image靠前的，因此你遇到的第一个问题便是如何下载FCB/DBBT进Raw NAND？  
　　首先来看FCB和DBBT的原型，如下semc_nand_fcb_t是FCB原型，semc_nand_dbbt_t是DBBT原型：
　　FCB/DBBT结构体开头都是12bytes的semc_bcb_header_t，这个bcb header由Tag、Version、CRC Checksum（CRC32-MPEG2）组成，用于验证FCB/DBBT的完整性。  
　　semc_nand_fcb_t.DBBTSerachAreaStartPage标明DBBT所在位置；semc_nand_fcb_t.searchStride和semc_nand_fcb_t.searchCount用于存在2份FCB/DBBT时标明第二份位置（此处我们仅用一份，所以searchCount设为1，searchStride的值不用管）；semc_nand_fcb_t.firmwareCopies记录Firmware总份数，semc_nand_fcb_t.firmwareTable标明所有Firmware具体位置；semc_nand_fcb_t.nandConfig是Raw NAND的configuration block，大小为256bytes，记录Raw NAND特性参数。  
　　semc_nand_dbbt_t.badBlockNumber记录坏块总个数，semc_nand_dbbt_t.badBlockTable标明所有坏块具体位置。  

```C
#define SEMC_NAND_BAD_BLOCKS_MAX_NUM 256
#define SEMC_NAND_FW_MAX_NUM 8

#define SEMC_NAND_FCB_TAG 0x4E464342U     //!< ASCII: "NFCB"
#define SEMC_NAND_FCB_VERSION 0x00000001  //!< Version: 1.0
#define SEMC_NAND_DBBT_TAG 0x44424254U    //!< ASCII: "DBBT"
#define SEMC_NAND_DBBT_VERSION 0x00000001 //!< Version: 1.0

typedef struct _nand_firmware_info
{
    uint32_t startPage;
    uint32_t pagesInFirmware;
} nand_firmware_info_t;

typedef struct _semc_bcb_header
{
    uint32_t crcChecksum; //!< [0x000-0x003]
    uint32_t fingerprint; //!< [0x004-0x007]
    uint32_t version;     //!< [0x008-0x00b]
} semc_bcb_header_t;

typedef struct __semc_nand_config
{
    semc_mem_config_t memConfig;      //!< [0x000-0x04f]
    uint8_t vendorType;               //!< [0x050-0x050]
    uint8_t cellTechnology;
    uint8_t onfiVersion;
    uint8_t acTimingTableIndex;
    uint8_t enableEccCheck;           //!< [0x054-0x054]
    uint8_t eccCheckType;
    uint8_t deviceEccStatus;
    uint8_t swEccAlgorithm;
    uint32_t swEccBlockBytes;         //!< [0x058-0x05b]
    uint8_t readyCheckOption;         //!< [0x05c-0x05c]
    uint8_t statusCommandType;        //!< [0x05d-0x05d]
    uint16_t readyCheckTimeoutInMs;   //!< [0x05e-0x05f]
    uint16_t readyCheckIntervalInUs;  //!< [0x060-0x061]
    uint8_t reserved0[30];            //!< [0x062-0x07f]
    uint8_t userOnfiAcTimingModeCode; //!< [0x080-0x080]
    uint8_t reserved1[31];            //!< [0x081-0x09f]
    uint32_t bytesInPageDataArea;     //!< [0x0a0-0x0a3]
    uint32_t bytesInPageSpareArea;
    uint32_t pagesInBlock;
    uint32_t blocksInPlane;           //!< [0x0ac-0x0af]
    uint32_t planesInDevice;          //!< [0x0b0-0x0b3]
    uint32_t reserved2[19];           //!< [0x0b4-0x0ff]
} semc_nand_config_t;

typedef struct _semc_nand_fcb
{
    semc_bcb_header_t bcbHeader;                              //!< [0x000-0x00b]
    uint32_t DBBTSerachAreaStartPage;                         //!< [0x00c-0x00f]
    uint16_t searchStride;                                    //!< [0x010-0x011]
    uint16_t searchCount;                                     //!< [0x012-0x013]
    uint32_t firmwareCopies;                                  //!< [0x014-0x017]
    uint32_t reserved0[10];                                   //!< [0x018-0x03f]
    nand_firmware_info_t firmwareTable[SEMC_NAND_FW_MAX_NUM]; //!< [0x040-0x07f]
    uint32_t reserved1[32];                                   //!< [0x080-0x0ff]
    semc_nand_config_t nandConfig;                            //!< [0x100-0x1ff]
    uint32_t reserved2[128];                                  //!< [0x200-0x3ff]
} semc_nand_fcb_t;

typedef struct _semc_nand_dbbt
{
    semc_bcb_header_t bcbHeader;                          //!< [0x000-0x00b]
    uint32_t reserved0;                                   //!< [0x00c-0x00f]
    uint32_t badBlockNumber;                              //!< [0x010-0x013]
    uint32_t reserved1[3];                                //!< [0x014-0x01f]
    uint32_t badBlockTable[SEMC_NAND_BAD_BLOCKS_MAX_NUM]; //!< [0x020-0x41f]
} semc_nand_dbbt_t;
```

　　知道了FCB/DBBT结构，那么怎么生成FCB/DBBT数据并且下载进Raw NAND什么地址处呢？当然我们可以手工创建FCB/DBBT并将其下载到Raw NAND中，但其实<font color="Blue">Flashloader工具会帮我们自动做好大部分工作（生成FCB/DBBT，将FCB/DBBT下载到Raw NAND中），而我们只需要提供简化的12byte配置数据即可。</font>如果你还有印象的话，痞子衡在启动系列文章的第四篇 [飞思卡尔i.MX RT系列微控制器启动篇（4）- Flashloader初体验(blhost)](https://www.cnblogs.com/henjay724/p/9098577.html) 的最后介绍过下载更新Application示例（该示例适用NAND芯片MX30LF4GE8AB-TI）：  

```text
// 在SRAM里临时存储Raw NAND配置数据
blhost -u -- fill-memory 0x2000 0x4 0xD0010101 // ONFI 1.0, non-EDO, Timing mode 0, 8bit IO, CSX0, HW ECC Check, inital HW ECC is enabled
blhost -u -- fill-memory 0x2004 0x4 0x00010101 // image copy = 1, search stride = 1, search count = 1
blhost -u -- fill-memory 0x2008 0x4 0x00020001 // Firmware block index = 2, block count = 1

// 使用Raw NAND配置数据去配置Raw NAND接口
blhost -u -- configure-memory 0x100 0x2000
```

　　在上述示例里痞子衡首先使用了fill-memory命令在0x2000地址处暂存了12byte配置数据，然后通过config-memory将这12byte数据里的信息配置到Flashloader的Raw NAND接口中，实际上这4个命令成功执行后，FCB/DBBT就已经被下载进Raw NAND里面了。那么这12byte配置数据到底是怎么组织的？详见下表：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_img_option1.PNG" style="zoom:100%" />

　　从上表我们可以知道，其实这<font color="Blue">12byte数据提供的配置信息还是比较多的，涵盖NAND配置、FCB配置、Image配置，但是与FCB/DBBT原本的数据结构相比已经大幅精简，我们还可以再进一步简化，这12byte里真正需要注意的只有四个地方（ECC status、ECC Type、IO Port Size、EDO mode），其余可用固定配置。</font>由于此处我们示例NAND芯片为MX30LF4GE8AB-TI，查看NAND芯片手册可知其是x8 IO且没有HW ECC，那么IO Port Size需设2'b01（即x8），ECC type、ECC status分别可设1'b1、1'b0（HW ECC Check，initial HW ECC is enabled，其实这样设在没有HW ECC的NAND芯片上的意思是不使能ECC Check），EDO mode可设1'b0（即non-EDO模式）。  
　　configure-memory命令执行成功之后，我们可以试着用read-memory从NAND芯片里读回FCB,DBBT确认一下，示例NAND芯片MX30LF4GE8AB-TI的page size为2KB，block size为128KB，那么FCB应该在0x0处，DBBT应该在0x20000处，从0x0处读回1KB数据发现其确实是有效的FCB，从FCB里找到其中DBBTSerachAreaStartPage = 0x40，即DBBT放在index为64的Page里（即index为1的Block起始Page里），再从0x20000处读回1056bytes数据发现其也确实是有效的DBBT。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_readback_fcb1.PNG" style="zoom:100%" />

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_readback_dbbt1.PNG" style="zoom:100%" />

　　解决了第一个问题即FCB/DBBT问题，还有另一个问题便是image应该如何下载进Raw NAND？  
　　其实image的下载很简单，只需要将Bootable image从index为2的block里（与FCB里的firmwareTable[0].startPage对应）开始下载即可，示例NAND芯片MX30LF4GE8AB-TI的block size为128KB，则下载地址应为0x40000，具体步骤如下：  

```text
// 擦除Raw NAND并将image下载进Raw NAND
blhost -u -- flash-erase-region 0x40000 0x20000 0x100    // Erase 1 block starting from block 2
blhost -u -- write-memory 0x40000 ivt_image.bin 0x100    // Program ivt_image.bin to block 2
```
　　Bootable image下载成功之后，同样我们可以试着用read-memory从NAND芯片里读回IVT,BootData,Application确认一下，Bootable image起始地址在0x40000，那么IVT,BootData应该在0x40400，Application应该在0x42000，查看数据发现确实是有效的Bootable image。你可能会疑问，NAND的读写操作一般都是按page的，为何我们使用read-memory命令提供的地址参数可以不按page对齐？其实Flashloader内部会有page缓存区，Flashloader底层对NAND的访问是按page进行的，并缓存在内部page缓存区，接口上层来读写NAND数据实际是在page缓存区进行的，所以不受page对齐限制。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_readback_img.PNG" style="zoom:100%" />

　　至此，Application的下载工作便结束了。  

### 五、进入Raw NAND启动模式
　　Application已经被成功下载进Raw NAND芯片之后，此时我们便可以开始设置芯片从Raw NAND启动：  

　　在进入Boot Device选择之前，你首先需要确定BOOT_MODE[1:0]=2'b10，即芯片处于Internal Boot模式，并且确认BT_FUSE_SEL（eFUSE偏移0x460处的32bit配置数据的bit4）为1'b0，这里看不懂的朋友请温习痞子衡前面的文章 [飞思卡尔i.MX RT系列微控制器启动篇（2）- Boot配置(BOOT Pin/eFUSE)](http://www.cnblogs.com/henjay724/p/9034563.html)。  
　　设置好正确Boot模式后，再来选择Boot Device，Boot Device由BOOT_CFG1[7:4]这四个pin的输入状态决定，下图是RT105x/RT106x硬件板的参考设计，拨码开关SW6应拨向SW_DIP-8的7,8,11，即设置BOOT_CFG[7:4]=4'b001x（4'b001x适用于i.MXRT105x/i.MXRT106x，对于i.MXRT102x此值应为4'b01xx），此时便进入了从SEMC NAND启动模式。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_bootpin_sel.PNG" style="zoom:100%" />

　　如果想确保i.MXRT芯片一定正在从Raw NAND启动，可在芯片上电时使用Jlink调试器或者借助Flashloader读取芯片内部2个寄存器的值，这2个寄存器分别是SRC_SBMR1/2, 我们设的关于启动模式的BOOT_MODE pins/BOOT_CFG pin/eFUSE偏移0x450配置值在上电时会自动加载到SRC_SBMR1/2寄存器里，BootROM主要是根据SRC_SBMR1/2寄存器的值来判断启动模式的。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_readback_ocotp_src_regs1.PNG" style="zoom:100%" />

　　PS: BOOT_MODE[1:0]也可以设为2'b00，即芯片处于Boot From Fuses模式，但此时稍微繁琐一点，需要将BT_FUSE_SEL（eFUSE偏移0x460处的32bit配置数据的bit4）烧写为1'b1和BOOT_CFG1[7:4]（eFUSE偏移0x450处的32bit配置数据的bit7:4）烧写成4'b001x（适用于i.MXRT105x/i.MXRT106x）。  

### 六、配置eFUSE启动Raw NAND
　　设置好芯片启动模式是从Raw NAND启动之后，我们还需要最后关注一下与Raw NAND相关的具体特性配置：  

　　你应该记得我们在使用Flashloader下载Application的时候提供过12bytes的NAND配置数据，这12bytes的NAND配置数据是为了让Flashloader能够正确初始化Raw NAND接口去访问NAND芯片（主要是写FCB,DBBT,Bootable image），同样BootROM上电也需要初始化Raw NAND接口去访问NAND芯片（主要是读FCB,DBBT,Bootable image），所以BootROM也需要类似这12bytes NAND配置数据，而BootROM的NAND配置便放在如下的eFUSE区域里（i.MXRT105x/i.MXRT102x是一样的，i.MXRT106x与i.MXRT105x比有细微调整），与Flashloader一样这部分配置里真正需要注意的也只有四个地方（ECC status、ECC Type、IO Port Size、EDO mode），其余可用初始配置（即0值）。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_1050a1_1020_fusemap.PNG" style="zoom:100%" />

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_RawNAND_1060_fusemap.PNG" style="zoom:100%" />

### 七、几个注意事项
> 1. i.MXRT105x A0版本与A1版本的BootROM有很大区别，本文内容仅适用于A1版本。
> 2. 实测发现i.MXRT105x需要使能EDO模式方可访问NAND（默认是AXI方式），而i.MXRT106x则不需要使能EDO模式。
> 3. 配置数据（Flashloader的12bytes/BootROM的eFUSE）里关于ECC的2处配置（ECC status、ECC Type）需根据实际连接的NAND芯片而定，不可随意设置。
>   3.1 对于没有硬件ECC的NAND芯片，可有两种设置组合：一、ECC Type=HW，ECC status=Enabled（即不用ECC check）；二、ECC Type=SW，ECC status=x（即使用SW ECC check）。
>   3.2 对于含有硬件ECC且默认是使能的NAND芯片，仅有一种设置组合：一、ECC Type=HW，ECC status=Enabled（即使用HW ECC check）。
>   3.3 对于含有硬件ECC且默认没使能的NAND芯片，可有两种设置组合：一、ECC Type=SW，ECC status=x（即使用SW ECC check）；二、ECC Type=HW，ECC status=Disabled（即使用HW ECC check，BootROM会自动使用set-feature命令开启硬件ECC，目前仅支持Micron的NAND芯片）

　　上述所有步骤全部完成之后，复位芯片你就应该能看到你放在Raw NAND里的Application已经正常地启动了。  

　　至此，飞思卡尔i.MX RT系列MCU的Raw NAND启动痞子衡便介绍完毕了，掌声在哪里~~~ 

