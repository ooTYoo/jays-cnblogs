----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔Kinetis系列MCU的KBOOT之完整性检测（Integrity Check）特性**。  

　　Application完整性检测是非常重要的，想象一下如果你的系统中Application被人为破坏了一部分并注入异常代码，而系统在启动过程中不能检测出Application的异常直接跳转执行，那么将会导致不可预测的结果，这种情况肯定是要避免的，KBOOT在设计时考虑到这种情况的存在，特意引入了Integrity Check这一特性来解决这个问题。今天痞子衡就为大家介绍KBOOT中的Integrity Check特性：  

### 一、使能完整性检测
　　KBOOT中使用了CRC32算法来检查Application的Integrity，痞子衡在前面文章 [KBOOT配置(FOPT/BOOT Pin/BCA)](https://www.cnblogs.com/henjay724/p/9350462.html) 里介绍过KBOOT的BCA配置，其中跟CRC32相关的是offset为0x04 - 0x0f的12bytes：  

```C
typedef struct BootloaderConfigurationData
{
    uint32_t tag;                          //!< [00:03] Tag value used to validate the BCA data. Must be set to 'kcfg'.
    uint32_t crcStartAddress;              //!< [04:07]
    uint32_t crcByteCount;                 //!< [08:0b]
    uint32_t crcExpectedValue;             //!< [0c:0f]
    // ...
} bootloader_configuration_data_t;
```

　　要想使能Integrity Check功能，Application中必须包含有效的BCA配置数据（即tag需为'kcfg'），且crcStartAddress、crcByteCount、crcExpectedValue不能全为0xFFFFFFFF，而实际使用中crcStartAddress一般指向Application起始地址，crcByteCount为Application总长度（假定Application在内存中是连续的），crcExpectedValue为Application的CRC32 checksum。  
　　知道了怎么使能Integrity Check，那有没有方便的工具为Application添加有效的Integrity Check数据呢？当然有！在\NXP_Kinetis_Bootloader_2_0_0\bin\Tools\KinetisFlashTool\win目录下面有一个KinetisFlashTool.exe工具，打开这个工具，选择【BCA Utilities】，并在Image File下面指定你的Application binary文件（此处以\NXP_Kinetis_Bootloader_2_0_0\apps\led_demo\MK80F25615\iar\binaries\led_demo_tower_0000.bin为例），然后点击【Config】便会弹出一个Kinetis Bootloader Configuration界面，确保Tag已被勾选，且在Crc check里勾选Enable并填写正确的Image Address，点击【ok】之后，回到主界面再点击【Save】，恭喜你，你的Application已经被添加了有效的Integrity Check数据。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_ImageCRC_tool_to_add_crc_data.PNG" style="zoom:100%" />

　　你可以用二进制编辑器打开你的Application binary文件查看Integrity Check数据是否添加成功，下面是痞子衡生成的新Application文件，可以看到crcStartAddress = 0, crcByteCount = 0x7A8, crcExpectedValue = 0x4779A38E。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_ImageCRC_img_with_crc.PNG" style="zoom:100%" />

> Note 1: BCA里指定的CRC计算范围如果包含crcExpectedValue这4bytes的话，在计算CRC时会自动跳过这4bytes。  
> Note 2: BCA里指定的CRC计算长度如果不是4字节对齐，在计算CRC时会自动补0。  

### 二、完整性检测流程
　　如果你使能了Integrity Check功能并把含有效Integrity Check配置的Application数据下载进了Flash，下一次芯片复位启动时会进入KBOOT执行，KBOOT会按照下面流程对Application进行完整性检测：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_ImageCRC_boot_flow.PNG" style="zoom:100%" />

　　KBOOT首先会从Flash里加载BCA数据并检测BCA数据是否有效，当发现BCA和integrity check数据都是有效的，便会将Integrity Checker状态置为inactive，否则则会置为invalid；  
　　当KBOOT对Application完成常规的有效性检查（初始PC地址是否合法）之后开始尝试跳转到Application，在跳转到Application之前会启动Integrity Checker流程，Integrity Checker正常工作的前提是其状态为inactive，Integrity Checker会分别检查CRC计算范围是否合法以及CRC结果是否匹配，仅当这些检查都通过时，Integrity Checker会返回passed状态通知KBOOT可以跳转Application。  

### 三、完整性检测算法CRC32-MPEG2
　　关于CRC32算法的具体实现有很多分支，KBOOT中使用的比较主流的MPEG2分支，其具体参数如下表所示：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_ImageCRC_mpeg2_characteristics.PNG" style="zoom:100%" />

　　上表中最重要的参数是多项式系数polynomial = 0x04C11BD7，这个参数值代表的是如下多项式：  

> x<sup>32</sup>+x<sup>26</sup>+x<sup>23</sup>+x<sup>22</sup>+x<sup>16</sup>+x<sup>12</sup>+x<sup>11</sup>+x<sup>10</sup>+x<sup>8</sup>+x<sup>7</sup>+x<sup>5</sup>+x<sup>4</sup>+x<sup>2</sup>+x+1

#### 3.1 硬件模块CRC实现
　　Kinetis芯片中都是有硬件CRC模块的，KBOOT中CRC32-MPEG2算法可以通过硬件CRC模块来实现，恩智浦官方SDK里提供了CRC模块的驱动，下面是KBOOT里对CRC驱动的封装：  

```C
#include "fsl_crc.h"

/* Table of base addresses for crc instances. */
static CRC_Type *const g_crcBase[1] = CRC_BASE_PTRS;

// initialize the members of the allocated crc32_data_t struct
void crc32_init(crc32_data_t *crc32Config)
{
    crc32Config->currentCrc = 0xffffffffU;
    crc32Config->byteCountCrc = 0;
}

// "running" crc32 calculation
void crc32_update(crc32_data_t *crc32Config, const uint8_t *src, uint32_t lengthInBytes)
{
    crc_config_t crcUserConfigPtr;
    CRC_GetDefaultConfig(&crcUserConfigPtr);

    crcUserConfigPtr.crcBits = kCrcBits32;
    crcUserConfigPtr.seed = crc32Config->currentCrc;
    crcUserConfigPtr.polynomial = 0x04c11db7U;
    crcUserConfigPtr.complementChecksum = false;
    crcUserConfigPtr.reflectIn = false;
    crcUserConfigPtr.reflectOut = false;

    // Init CRC module and then run it
    //! Note: We must init CRC module here, As we may seperate one crc calculation into several times
    //! Note: It is better to use lock to ensure the integrity of current updating operation of crc calculation
    //        in case crc module is shared by multiple crc updating requests at the same time
    if (lengthInBytes)
    {
        lock_acquire();
        CRC_Init(g_crcBase[0], &crcUserConfigPtr);
        CRC_WriteData(g_crcBase[0], src, lengthInBytes);
        crcUserConfigPtr.seed = CRC_Get32bitResult(g_crcBase[0]);
        lock_release();
    }
    crc32Config->currentCrc = crcUserConfigPtr.seed;
    crc32Config->byteCountCrc += lengthInBytes;
}

// finalize the crc32 calculation for non-word-aligned counts
void crc32_finalize(crc32_data_t *crc32Config, uint32_t *hash)
{
    uint32_t extraBytes = crc32Config->byteCountCrc % 4;
    // pad with zeroes
    if (extraBytes)
    {
        uint8_t temp[3] = { 0, 0, 0 };
        crc32_update(crc32Config, temp, 4 - extraBytes);
    }
    *hash = crc32Config->currentCrc;
    // De-init CRC module when we complete a full crc calculation
    CRC_Deinit(g_crcBase[0]);
}
```

#### 3.2 查表法软件CRC实现
　　当然CRC32-MPEG2算法也可以通过软件来实现，下面是查表法实现CRC32-MPEG2的参考：  

```C
////////////////////////////////////////////////////////////////////////////////
// Definitions
////////////////////////////////////////////////////////////////////////////////
//! @brief State information for the CRC32 algorithm.
typedef struct Crc32Data
{
    uint32_t currentCrc;   //!< Current CRC value.
    uint32_t byteCountCrc; //!< Number of bytes processed.
} crc32_data_t;

////////////////////////////////////////////////////////////////////////////////
// Variables
////////////////////////////////////////////////////////////////////////////////
//! Table of CRC-32's of all single byte values. The values in
//! this table are those used in the Ethernet CRC algorithm.
static const uint32_t s_crc32Table[] = {
    0x00000000, 0x04c11db7, 0x09823b6e, 0x0d4326d9, 0x130476dc, 0x17c56b6b, 0x1a864db2, 0x1e475005,
	0x2608edb8, 0x22c9f00f, 0x2f8ad6d6, 0x2b4bcb61, 0x350c9b64, 0x31cd86d3, 0x3c8ea00a, 0x384fbdbd,
	0x4c11db70, 0x48d0c6c7, 0x4593e01e, 0x4152fda9, 0x5f15adac, 0x5bd4b01b, 0x569796c2, 0x52568b75,
	0x6a1936c8, 0x6ed82b7f, 0x639b0da6, 0x675a1011, 0x791d4014, 0x7ddc5da3, 0x709f7b7a, 0x745e66cd,
	0x9823b6e0, 0x9ce2ab57, 0x91a18d8e, 0x95609039, 0x8b27c03c, 0x8fe6dd8b, 0x82a5fb52, 0x8664e6e5,
	0xbe2b5b58, 0xbaea46ef, 0xb7a96036, 0xb3687d81, 0xad2f2d84, 0xa9ee3033, 0xa4ad16ea, 0xa06c0b5d,
	0xd4326d90, 0xd0f37027, 0xddb056fe, 0xd9714b49, 0xc7361b4c, 0xc3f706fb, 0xceb42022, 0xca753d95,
	0xf23a8028, 0xf6fb9d9f, 0xfbb8bb46, 0xff79a6f1, 0xe13ef6f4, 0xe5ffeb43, 0xe8bccd9a, 0xec7dd02d,
	0x34867077, 0x30476dc0, 0x3d044b19, 0x39c556ae, 0x278206ab, 0x23431b1c, 0x2e003dc5, 0x2ac12072,
    0x128e9dcf, 0x164f8078, 0x1b0ca6a1, 0x1fcdbb16, 0x018aeb13, 0x054bf6a4, 0x0808d07d, 0x0cc9cdca,
	0x7897ab07, 0x7c56b6b0, 0x71159069, 0x75d48dde, 0x6b93dddb, 0x6f52c06c, 0x6211e6b5, 0x66d0fb02,
	0x5e9f46bf, 0x5a5e5b08, 0x571d7dd1, 0x53dc6066, 0x4d9b3063, 0x495a2dd4, 0x44190b0d, 0x40d816ba,
	0xaca5c697, 0xa864db20, 0xa527fdf9, 0xa1e6e04e, 0xbfa1b04b, 0xbb60adfc, 0xb6238b25, 0xb2e29692,
	0x8aad2b2f, 0x8e6c3698, 0x832f1041, 0x87ee0df6, 0x99a95df3, 0x9d684044, 0x902b669d, 0x94ea7b2a,
	0xe0b41de7, 0xe4750050, 0xe9362689, 0xedf73b3e, 0xf3b06b3b, 0xf771768c, 0xfa325055, 0xfef34de2,
	0xc6bcf05f, 0xc27dede8, 0xcf3ecb31, 0xcbffd686, 0xd5b88683, 0xd1799b34, 0xdc3abded, 0xd8fba05a,
	0x690ce0ee, 0x6dcdfd59, 0x608edb80, 0x644fc637, 0x7a089632, 0x7ec98b85, 0x738aad5c, 0x774bb0eb,
	0x4f040d56, 0x4bc510e1, 0x46863638, 0x42472b8f, 0x5c007b8a, 0x58c1663d, 0x558240e4, 0x51435d53,
	0x251d3b9e, 0x21dc2629, 0x2c9f00f0, 0x285e1d47, 0x36194d42, 0x32d850f5, 0x3f9b762c, 0x3b5a6b9b,
	0x0315d626, 0x07d4cb91, 0x0a97ed48, 0x0e56f0ff, 0x1011a0fa, 0x14d0bd4d, 0x19939b94, 0x1d528623,
	0xf12f560e, 0xf5ee4bb9, 0xf8ad6d60, 0xfc6c70d7, 0xe22b20d2, 0xe6ea3d65, 0xeba91bbc, 0xef68060b,
	0xd727bbb6, 0xd3e6a601, 0xdea580d8, 0xda649d6f, 0xc423cd6a, 0xc0e2d0dd, 0xcda1f604, 0xc960ebb3,
	0xbd3e8d7e, 0xb9ff90c9, 0xb4bcb610, 0xb07daba7, 0xae3afba2, 0xaafbe615, 0xa7b8c0cc, 0xa379dd7b,
	0x9b3660c6, 0x9ff77d71, 0x92b45ba8, 0x9675461f, 0x8832161a, 0x8cf30bad, 0x81b02d74, 0x857130c3,
	0x5d8a9099, 0x594b8d2e, 0x5408abf7, 0x50c9b640, 0x4e8ee645, 0x4a4ffbf2, 0x470cdd2b, 0x43cdc09c,
	0x7b827d21, 0x7f436096, 0x7200464f, 0x76c15bf8, 0x68860bfd, 0x6c47164a, 0x61043093, 0x65c52d24,
	0x119b4be9, 0x155a565e, 0x18197087, 0x1cd86d30, 0x029f3d35, 0x065e2082, 0x0b1d065b, 0x0fdc1bec,
    0x3793a651, 0x3352bbe6, 0x3e119d3f, 0x3ad08088, 0x2497d08d, 0x2056cd3a, 0x2d15ebe3, 0x29d4f654,
	0xc5a92679, 0xc1683bce, 0xcc2b1d17, 0xc8ea00a0, 0xd6ad50a5, 0xd26c4d12, 0xdf2f6bcb, 0xdbee767c,
	0xe3a1cbc1, 0xe760d676, 0xea23f0af, 0xeee2ed18, 0xf0a5bd1d, 0xf464a0aa, 0xf9278673, 0xfde69bc4,
	0x89b8fd09, 0x8d79e0be, 0x803ac667, 0x84fbdbd0, 0x9abc8bd5, 0x9e7d9662, 0x933eb0bb, 0x97ffad0c,
	0xafb010b1, 0xab710d06, 0xa6322bdf, 0xa2f33668, 0xbcb4666d, 0xb8757bda, 0xb5365d03, 0xb1f740b4
};

////////////////////////////////////////////////////////////////////////////////
// Code
////////////////////////////////////////////////////////////////////////////////
// initialize the members of the allocated crc32_data_t struct
void crc32_init(crc32_data_t *crc32Config)
{
    // initialize running crc and byte count
    crc32Config->currentCrc = 0xFFFFFFFF;
    crc32Config->byteCountCrc = 0;
}

// "running" crc32 calculation
void crc32_update(crc32_data_t *crc32Config, const uint8_t *src, uint32_t lengthInBytes)
{
    uint32_t crc = crc32Config->currentCrc;
    crc32Config->byteCountCrc += lengthInBytes;
    while (lengthInBytes--)
    {
        uint8_t c = *src++ & 0xff;
        crc = (crc << 8) ^ s_crc32Table[(crc >> 24) ^ c];
    }
    crc32Config->currentCrc = crc;
}

// finalize the crc32 calculation for non-word-aligned counts
void crc32_finalize(crc32_data_t *crc32Config, uint32_t *hash)
{
    uint32_t crc = crc32Config->currentCrc;
    uint32_t byteCount = crc32Config->byteCountCrc;
    // pad with zeroes
    if (byteCount % 4)
    {
        uint32_t i;
        for (i = byteCount % 4; i < 4; i++)
        {
            crc = (crc << 8) ^ s_crc32Table[(crc >> 24) ^ 0];
        }
    }
    crc32Config->currentCrc = crc;
    *hash = crc32Config->currentCrc;
}
```

　　至此，飞思卡尔Kinetis系列MCU的KBOOT之完整性检测（Integrity Check）痞子衡便介绍完毕了，掌声在哪里~~~  

