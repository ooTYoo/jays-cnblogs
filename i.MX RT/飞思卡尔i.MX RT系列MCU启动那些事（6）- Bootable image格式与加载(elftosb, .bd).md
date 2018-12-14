----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的Bootable image格式与加载过程**。  

　　在i.MXRT启动系列第三篇文章 [Serial Downloader模式(sdphost, mfgtool)](http://www.cnblogs.com/henjay724/p/9034563.html) 里痞子衡在介绍使用sdphost引导启动Flashloader时使用过一个名叫ivt_flashloader.bin的image文件，其实这个image文件就是Bootable image的一种，虽然痞子衡简单分析过ivt_flashloader的组成，但介绍得并不详尽，今天痞子衡会为大家系统地讲解i.MXRT Bootable image。  

### 一、什么是Bootable image？
　　如果你是一个有经验的嵌入式开发者，肯定对image格式有所了解，我们通常开发的Application都是针对含内部FLASH的MCU而言的，比如Kinetis、LPC、STM32等MCU，其内部集成了一块Parallel NOR FLASH，且FLASH地址是映射在ARM 4GB system address内的（一般从0x0地址开始），FLASH里存储的直接就是我们编译链接后生成的原始Application binary(.bin)，没有任何多余的数据组成。或许你会说还有.hex, .srec等其他image格式，是的，但这些带地址信息的image格式是为编程器或下载器服务的，这些image格式经过编程器或者下载器解析后真正下载进MCU内部FLASH的数据还是原始Application binary。这类MCU上电后CPU能直接从内部FLASH获取Application代码并原地执行（XIP），所以对这类MCU而言，Bootable image就是存储在内部FLASH的Application binary(.bin)。  
　　但是以上经验在开发i.MXRT时遇到了问题，i.MXRT没有内部FLASH，需要外接FLASH存储器以存储image。<font color="Blue">众所周知，FLASH从结构上分为NOR和NAND，i.MXRT启动同时支持这两种FLASH，NOR FLASH可以实现XIP，NAND FLASH不可以XIP，为了兼容所有FLASH，在设计i.MXRT bootable image格式时必须以非XIP这种情况为基准。既然是非XIP执行，即意味着i.MXRT上电时会将image从外接FLASH拷贝到内部SRAM中去执行，在拷贝时必不可免要知道两个重要的数据：image链接起始地址（决定image被拷贝到SRAM哪个地址）、image总长度（决定要从外部FLASH拷贝多长的image数据进SRAM），实际上除了这两个最基本的数据外还有其他更高级的数据（配置、安全等特性），因此存储在外接FLASH的i.MXRT Bootable image除了含有Application binary数据之外还必须含有额外的信息，这些额外的信息数据与Application binary共同组成i.MXRT Bootable image。</font>至于这些额外的信息在Bootable image里是如何组织的，痞子衡在后面会继续聊。  

### 二、Bootable image链接空间
　　一个image的链接空间分两种，一种是只读段（readonly code,data）的链接空间，另一种是读写段（readwrite data, STACK）的链接空间，这两种链接空间要求的存储介质特性不一样，痞子衡逐一讲解：  
　　前面讲了i.MXRT同时支持外接NOR和NAND FLASH，其中NAND FLASH无法XIP，那么存储在NAND FLASH中的image只读段必须要链接在SRAM里。i.MXRT内部有三种SRAM，分别是ITCM, DTCM, OCRAM，是不是这三种SRAM都可以被随意链接呢？答案并不是！因为在Boot期间，BootROM也需要占用SRAM，用于存放BootROM的读写段，所以被BootROM占用的SRAM无法用于链接image的只读段，如果强行链接，会导致BootROM在拷贝image只读段时破坏自身读写段，从而发生不可预料的行为。下图是RT1050 BootROM的memory map，从图中可以得知BootROM占用的是0x20200000开始的OCRAM，并且看起来是整块OCRAM都被占用了，所以不推荐使用OCRAM去链接image只读段。  
　　黑科技：如果有朋友表示不服，RT1060/RT1050/RT1020的OCRAM是1MB/512KB/256KB，BootROM读写段不可能有这么大，是的，痞子衡告诉你，其实<font color="Blue">BootROM数据段只要32KB（0x20200000 - 0x20207FFF），另外还需要4KB用加载initial non-XIP image（0x20208000 - 0x20208FFF），所以对于存储在non-XIP FLASH的image你可以从0x20209000之后的空间里链接image只读段，而对于存储在XIP FLASH的image你可以从0x20208000之后的空间里链接image只读段</font>，这个秘密一般人痞子衡是不会告诉他的。  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_image_bootrom_mem_map.PNG" style="zoom:100%" />

　　前面讲了存储在NAND FLASH中的image只读段链接注意事项，而对于可以XIP的NOR FLASH，除了跟NAND一样可以将只读段链接在SRAM外，还可以链接在i.MXRT分配给外接存储器的XIP映射空间里，下表给出了Serial NOR（QSPI）和Parallel NOR（SEMC）各自的映射起始地址，需要注意的是<font color="Blue">Serial NOR支持的最大XIP空间为504MB，但是Parallel NOR支持的最大XIP空间只有16MB</font>，别问痞子衡是怎么知道的，痞子衡无所不知。  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_image_xip_space.PNG" style="zoom:100%" />

　　至于image的读写段，在链接时就不用区别Non-XIP/XIP FLASH了，都只能放在SRAM里，并且不用考虑BootROM对SRAM的占用问题（因为不在一个时间域里被使用），只要注意不和image自身只读段冲突就行。  
　　黑科技：有朋友注意到了SDRAM，是的i.MXRT也支持SDRAM，通过SEMC接口去实现SDRAM读写，所以如果外接了SDRAM并且使能的话，也可以将image只读段/读写段放入SDRAM，关于SDRAM的使用，痞子衡会在后面文章里介绍。  

### 三、Bootable image七大组成
　　Bootable image是由一些额外的信息数据与Application binary共同组成的，那些额外的信息数据按功能分有6类，但这6类信息数据并不都是必须的，其中有4类是可选的，因此一个Bootable image最多由7部分组成，最少由3部分组成。下面痞子衡按在FLASH里存储位置从低到高的顺序逐一介绍组成Bootable image的7大部分：  
#### 3.1 偏移0x0000: FDCB（Flash Device Configuration Block）
　　第一个组成部分叫FDCB，是个可选组成，目前只用于Serial/Parallel NOR FLASH，FDCB是从FLASH的起始地址处开始存放的，也是Bootable image最开始部分。FDCB最大4KB，其本身没有统一的与FLASH无关的structure，具体structure根据启动FLASH的接口类型（Serial/Parallel）而定，其一般是用来存储当前连接的FLASH的具体特性参数，BootROM上电会使用通用且可靠的FLASH接口控制器配置（即BootROM中默认参数配置，一般是比较低速的配置）去访问外接FLASH并获取FDCB，然后根据FDCB存储的参数去重新配置FLASH接口控制器再去进一步访问FLASH。下面的结构体是Serial NOR的FDCB原型，此处痞子衡不会展开介绍这个结构体，留到后续介绍Serial NOR启动再详细介绍。  

```C
typedef struct _flexspi_nor_config
{
    flexspi_mem_config_t memConfig; //!< Common memory configuration info via FlexSPI
    uint32_t pageSize;              //!< Page size of Serial NOR
    uint32_t sectorSize;            //!< Sector size of Serial NOR
    uint8_t ipcmdSerialClkFreq;     //!< Clock frequency for IP command
    uint8_t isUniformBlockSize;     //!< Sector/Block size is the same
    uint8_t reserved0[2];           //!< Reserved for future use
    uint8_t serialNorType;          //!< Serial NOR Flash type: 0/1/2/3
    uint8_t needExitNoCmdMode;      //!< Need to exit NoCmd mode before other IP command
    uint8_t halfClkForNonReadCmd;   //!< Half the Serial Clock for non-read command: true/false
    uint8_t needRestoreNoCmdMode;   //!< Need to Restore NoCmd mode after IP commmand execution
    uint32_t blockSize;             //!< Block size
    uint32_t reserve2[11];          //!< Reserved for future use
} flexspi_nor_config_t;
```

#### 3.2 偏移0x0400/0x1000: IVT（Image Vector Table）
　　第二个组成部分叫IVT，是个必备组成，也是6类信息数据里的最核心数据，IVT是一个统一的与FLASH无关的structure，其原型如下面结构体所示，从结构体定义我们得知，<font color="Blue">IVT中记录了Application、DCD、BD、CSF的位置信息，这些信息对BootROM加载启动至关重要。IVT大小固定为32byte，其在Bootable image中的偏移位置也是固定的（对于XIP FLASH而言偏移是0x1000，对于Non-XIP FLASH而言偏移是0x400）</font>。有朋友会疑问为何IVT偏移地址是固定的？其实答案很简单，因为BootROM必须要首先获取IVT才能进一步找到其他信息数据，而IVT本身的位置信息没有在其他地方被标明，所以只能在BootROM里用一个常量来记录。  

```C
#define HAB_TAG_IVT0 0xd1     /**< Image Vector Table V0 */

/** @ref hab_header structure */
typedef struct hab_hdr {
    uint8_t tag;              /**< Tag field */
    uint8_t len[2];           /**< Length field in bytes (big-endian) */
    uint8_t par;              /**< Parameters field */
} hab_hdr_t;

/** @ref ivt structure */
struct hab_ivt_v0 {
    /** @ref hdr with tag #HAB_TAG_IVT0, length and HAB version fields */
    hab_hdr_t hdr;
    /** Absolute address of the first instruction to execute from the image */
    uint32_t entry;
    /** Reserved in this version of HAB: should be NULL. */
    uint32_t reserved1;
    /** Absolute address of the image DCD: may be NULL. */
    uint32_t dcd;
    /** Absolute address of the Boot Data: may be NULL, but not interpreted any further by HAB */
    uint32_t boot_data;
    /** Absolute address of the IVT.*/
    uint32_t self;
    /** Absolute address of the image CSF.*/
    uint32_t csf;
    /** Reserved in this version of HAB: should be zero. */
    uint32_t reserved2;
};
```

#### 3.3 偏移0x0420/0x1020: BD（Boot Data）
　　第三个组成部分叫BD，是个必备组成，是仅次于IVT的核心数据，BD也是一个统一的与FLASH无关的structure，其原型如下面结构体所示，BD中记录了Bootable image的起始地址与总长度。BD大小固定为16byte，BD信息虽然记录在了IVT中，但其在Bootable image中的偏移位置并不是任意的，BD是紧挨着IVT的。  

```C
/** @ref boot_data structure */
typedef struct boot_data{
    uint32_t start;           /* Start address of the image */
    uint32_t size;            /* Size of the image */
    uint32_t plugin;          /* Plugin flag */
    uint32_t placeholder;     /* placehoder to make even 0x10 size */
} BOOT_DATA_T;
```

#### 3.4 DCD（Device Configuration Data）
　　第四个组成部分叫DCD，是个可选组成，目前主要用于SDRAM接口控制器（SEMC）的配置。由于i.MXRT内部SRAM size通常是够用的，且访问速度也很快，所以SDRAM并不一定要被使能，Bootable image常常不会包含DCD，所以痞子衡在这里先不做展开，后续有必要会再介绍。下面是SDK_2.3.1_EVKB-IMXRT1050包里hello_world工程（flexspi_nor）所使用DCD示例：  

```C
#define DCD_TAG_HEADER (0xD2)

const uint8_t dcd_data[] = {
    /*0000*/ DCD_TAG_HEADER,
    0x04,0x30,0x41,0xCC,0x03,0xAC,0x04,0x40,0x0F,0xC0,0x68,0xFF,0xFF,0xFF,0xFF,
    /*0010*/ 0x40,
    0x0F,0xC0,0x6C,0xFF,0xFF,0xFF,0xFF,0x40,0x0F,0xC0,0x70,0xFF,0xFF,0xFF,0xFF,

	...

    /*0420*/ 0x00,
    0x00,0x00,0x01,0xCC,0x00,0x0C,0x04,0x40,0x2F,0x00,0x4C,0x50,0x21,0x0A,0x09,
};
```

#### 3.5 偏移0x2000: Application Binary
　　第五个组成部分是你最熟悉的Application binary，当然是个必备组成，其在Bootable image中的偏移位置是固定的（0x2000），关于Application本身这里就不再赘述了。只特别提一点，那就是<font color="Blue">i.MXRT的Application只读段（主要指ARM中断向量表）并不可以从任意地址开始链接，有一个小小的限制，必须从选定的存储器地址空间偏移0x2000之后开始链接（如选中ITCM，则必须要链接在0x00002000之后；如选中DTCM，则必须链接在0x20002000之后...），因为要预留至少8KB空间给IVT、BD、DCD等数据，这个限制是BootROM自身决定的，务必要注意</font>。  

#### 3.6 CSF（Command Sequence File）
　　第六个组成部分叫CSF，是个特性组成，主要用于安全启动的认证相关特性，痞子衡会在安全启动里进一步介绍。  

#### 3.7 KeyBlob
　　第七个组成部分叫KeyBlob，是个特性组成，主要用于安全启动的加密相关特性，痞子衡会在安全启动里进一步介绍。  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_image_layout.PNG" style="zoom:100%" />

　　上图是包含IVT、BD、DCD、Application、CSF的Bootable image的layout，这张图很好地诠释了IVT的作用。  

### 四、Bootable image三种分类
　　前面介绍了Bootable image最多有7大组成，有些是必备，有些是可选，有的是特性。而在实际应用中，主要是必备+特性的组合形成如下三种常用分类：  

> * Unsigned Image: 这是最简单的image类型，由IVT+BD+Application组成，主要用于产品开发阶段。
> * Signed Image: 这是较复杂的image类型，由IVT+BD+Application+CSF组成，一般用于产品发布阶段。
> * Encrypted Image: 这是最复杂的image类型，由IVT+BD+Application+CSF+KeyBlob组成，主要用于对安全要求较高的产品中。

### 五、使用elftosb生成Bootable image
　　恩智浦官方提供了一个用于生成Bootable image的工具，名叫elftosb，这个工具就在\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\elftosb目录下，这个工具可以用来生成所有类型的Bootable image，命令格式固定如下：  

> <font color="Blue">elftosb.exe -f imx -V -c config_application.bd -o ivt_application.bin application.out</font>

　　其中ivt_application.bin就是最终生成的Bootable image，命令所需要的2个输入文件分别是application.out、config_application.bd，application.out就是你的Application工程编译链接生成的ELF文件，config_application.bd是用户配置文件，这个.bd文件主要是指示elftosb工具如何在Application binary基础上添加IVT、BD等其他信息数据从而形成Bootable image，所以编写.bd文件是关键步骤，bd文件有专门语法格式，但\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\bd_file\imx10xx目录下给了很多bd文件示例，我们只需要在某一个bd文件基础上修改即可  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_image_bd_folder.PNG" style="zoom:100%" />

　　如果你追过痞子衡博客文章，你应该知道痞子衡曾经实测过RT1052的coremark性能，coremark工程已经上传到痞子衡的github [https://github.com/JayHeng/cortex-m_app](https://github.com/JayHeng/cortex-m_app)，工程路径在\cortex-m_app\apps\coremark_imxrt1052\bsp\build\coremark.eww，编译此工程可得到coremark_a000.out和coremark_a000.bin文件，coremark程序只读段链接在ITCM地址（0x0000a000），我们来试着使用elftosb将coremark程序转换成bootable image，bd文件可参考imx-itcm-unsigned.bd，打开这个参考bd文件：  

```text
options {
    flags = 0x00;
    # Note: This is an example address, it can be any non-zero address in ITCM region
    startAddress = 0x8000;
    ivtOffset = 0x400;
    initialLoadSize = 0x2000;
    # Note: This is required if the default entrypoint is not the Reset_Handler 
    #       Please set the entryPointAddress to Reset_Handler address 
    // entryPointAddress = 0x60002411;
}

sources {
    elfFile = extern(0);
}

section (0)
{
}
```

　　ivtOffset和initialLoadSize不用改，分别代表IVT和Application在Bootable image中的偏移地址，startAddress即BOOT_DATA_T.start，这个是可以修改的，牢记下面公式：  

> <font color="Blue">startAddress + initialLoadSize = Application只读段起始链接地址</font>

　　coremark_a000.out是链接在0xa000地址处的，0x8000 + 0x2000 = 0xa000，所以此处startAddress也无需改，唯一需要确认的是entryPointAddress，保险起见统一将entryPointAddress设成Application的复位中断地址，即entryPointAddress = 0x0000ecd1。bd文件修改完成之后另存为config_coremark_a000.bd，让我们试着执行下面命令：  

```text
elftosb.exe -f imx -V -c config_coremark_a000.bd -o ivt_coremark_a000.bin coremark_a000.out
```

　　分别打开coremark_a000.bin和ivt_coremark_a000.bin，可以看到ivt_coremark_a000.bin比coremark_a000.bin多了前8KB的数据，这前8KB里包含了有效的IVT（偏移0x400）和BD（偏移0x420）。  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_image_binary.PNG" style="zoom:100%" />

### 六、Bootable image的加载过程
　　知道了Bootable image的构成，痞子衡最后再简要为大家介绍一下i.MXRT BootROM是如何从外部存储器中加载Bootable image进SRAM内存的。以non-XIP image加载为例（image链接在ITCM里），下图显示了i.MXRT加载image的四个阶段：  

> * 第一个阶段即加载前，此时Bootable image完全存储在外部Flash中，SRAM中没有任何image数据；
> * 第二阶段即初始加载，BootROM首先会从外部Flash读取Bootable image前4KB数据进SRAM临时缓存区（OCRAM：0x20208000 - 0x20208FFF），我们知道这4KB数据里包含了IVT和BD，BootROM从IVT和BD里获取到Bootable image的目标地址（BOOT_DATA_T.start）以及总长度（BOOT_DATA_T.size），此时便可以开始做进一步加载；
> * 第三阶段即内部转移，由于BootROM已经从外部Flash读取了4KB进SRAM临时缓存区，为了避免重复读取，BootROM会把这4KB数据首先复制到Bootable image的目标地址（ITCM）；
> * 第四阶段即加载完成，BootROM会接着将剩下的Bootable image（BOOT_DATA_T.size - 4KB）从外部Flash中全部读取出来存到目标区域（ITCM）完成全部加载。  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_image_sequence.PNG" style="zoom:100%" />

　　至此，飞思卡尔i.MX RT系列MCU的Bootable image格式与加载过程痞子衡便介绍完毕了，掌声在哪里~~~ 

