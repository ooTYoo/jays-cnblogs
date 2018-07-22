----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔Kinetis系列MCU的KBOOT配置**。  

　　KBOOT是支持配置功能的，配置功能可分为两方面：一、芯片系统的启动配置；二、KBOOT特性配置；痞子衡在前一篇文章里介绍了 [KBOOT形态(ROM/Bootloader/Flashloader)](https://www.cnblogs.com/henjay724/p/9322963.html)，虽然KBOOT有三种形态，但实际上只有2种类型的芯片载体，即含ROM空间的芯片（比如Kinetis K80）和不含ROM空间的芯片（比如Kinetis KL25），KBOOT配置在这两种载体上是有区别的，下面痞子衡为大家详解KBOOT配置：  

### 一、启动配置：FTFx_FOPT, BOOT Pin, RCM_FM
　　芯片系统的启动配置主要决定的是芯片上电从哪里（ROM/Flash）开始启动，所以这个启动配置对于含ROM空间的芯片特别重要，而不适用于不含ROM空间的芯片。  
#### 1.1 启动方式选择Flash Configuration Field - FOPT
　　熟悉Kinetis芯片的朋友肯定知道，Kinetis芯片都是含内部Flash的，内部Flash起始地址一般是0x00000000，Flash操作是通过FTFx这个IP模块实现的，如果你对FTFx模块了解的话，这个IP模块内部其实有一些寄存器属性是readonly的，并且从手册里看这些readonly寄存器的初值是undefined，截取K80芯片FTFA模块中这些readonly寄存器如下：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_CFG_k80_ftfa_mem_map.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_CFG_k80_ftfa_mem_map1.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_CFG_k80_ftfa_mem_map2.PNG" style="zoom:100%" />

　　既然这些寄存器是readonly属性并且初值又是undefined的，那么其初值到底取决于什么？这里就涉及到<font color="Blue">Kinetis芯片中比较特别的FCF加载机制，FCF即Flash Configuration Field，其区域地址为Flash偏移0x400 - 0x40f，一共16个bytes</font>，这16bytes内容组织如下：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_CFG_FCF_description.PNG" style="zoom:100%" />

　　<font color="Blue">任何一次热启动后，芯片系统会自动从FCF区域加载初值进FTFx相应寄存器中</font>，我们主要关注的是跟启动配置相关的FTFx_FOPT寄存器（特别注意，当FCF中对应FOPT的值是无效值0x00时，在加载过程中芯片自动会给FOPT赋值0xFF），下面是FTFx_FOPT寄存器的bit定义，其中BOOTSRC_SEL和BOOTPIN_OPT位是关键（注意这两个位在不含ROM空间的芯片上是reserved的）。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_CFG_FOPT_bit_definitions.PNG" style="zoom:100%" />

　　BOOTSRC_SEL和BOOTPIN_OPT的值共同决定了芯片的启动位置（ROM/Flash）：  
> * BOOTPIN_OPT = 1: 启动位置完全由BOOTSRC_SEL决定。
> * BOOTPIN_OPT = 0: 启动位置由BOOTSRC_SEL和BOOTCFG0 pin共同决定。

　　因此当在FCF里指定FOPT为0xFF时，芯片上电永远从ROM启动；当在FCF里指定FOPT为0x3F时，芯片上电永远从Flash启动。  

#### 1.2 启动位置切换BOOT Pin
　　在1.1节的最后痞子衡提到了BOOTCFG0 pin，其实BOOTCFG0 pin对于含ROM空间芯片而言就是BOOT Pin，这个BOOT Pin是芯片系统直接指定的，与NMI pin复用（在上电以及ROM执行过程中，NMI pin原本中断功能是被屏蔽的）。  
　　你一定会疑惑BOOT pin有什么用？让我们再回到1.1节的最后，0x3F和0xFF是两种比较典型的FOPT启动配置值，但是这种配置值指定的是固定启动位置，除非你擦除FCF重新烧写，不然无法轻易改变启动位置。但是有的时候我们想在不擦除FCF情况下自由切换启动位置ROM/Flash，这时候就得依靠BOOT Pin，此时我们需要在FCF里指定FOPT为0x3D，让我们结合下面的TWR-K80F150M原理图来说明：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_CFG_k80_sch_nmi_rst.PNG" style="zoom:100%" />

　　在上述TWR-K80F150M原理图中，我们可以看到两个按键开关（SW2,SW1）分别连到了K80芯片的NMI_b pin和RESET_b pin，<font color="Blue">当我们配置FOPT为0x3D时，即启动位置由BOOTSRC_SEL（2'b00，即从Flash启动）和BOOTCFG0（NMI）共同决定，如果在RESET_b pin（SW1）按下复位过程中，BOOTCFG0 pin（SW2）一直被按下，那么芯片会从ROM启动（并且超时也不会跳转到Application）；而如果BOOTCFG0 pin（SW2）没有被按下，那么芯片会从Flash启动。</font>是不是瞬间觉得这样切换启动位置很方便！  
　　其实BOOT Pin设计不仅仅只在含ROM空间的芯片上存在，在不含ROM空间的芯片上也支持，只不过在不含ROM空间的芯片上，BOOT Pin是由Bootloader代码指定的（需要查看芯片手册Bootloader章节或源代码），我们知道当芯片不含ROM时，上电默认从Flash起始地址处启动，而Flash起始地址已被Flash-Resident Bootloader占据，所以上电永远执行Flash-Resident Bootloader，此时BOOT Pin的意义主要是决定是否要超时跳转到Application，如果BOOT Pin在RESET_b pin按下复位过程中一直被按下，那么芯片将会一直停留在Bootloader中；如果BOOT Pin没有被按下，那么芯片在执行Bootloader超时时间到了之后会跳转到Application。  

#### 1.3 强制从ROM热启动RCM_FM
　　我们知道芯片复位启动分为冷启动（POR Pin）和热启动（RESET_b Pin），冷启动是最为彻底的启动（所有寄存器初值全部重置），而热启动并不是彻底启动（有些寄存器初值不会重置），RCM模块里有1个寄存器（RCM_FM）就只有冷启动才能被重置，而且这个寄存器与从ROM启动息息相关，不得不提。下面是RCM_FM和RCM_MR寄存器的bit定义：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_CFG_k80_rcm_fm.PNG" style="zoom:100%" />

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Kinetis_Boot_CFG_k80_rcm_mr.PNG" style="zoom:100%" />

　　上述两个寄存器只在含ROM空间的芯片上存在，其作用是为了保证ROM在执行期间即使不小心发生热启动，下一次还是会强制执行ROM程序，而不受FOPT, BOOT Pin状态变化影响。ROM程序里操作RCM_FM/MR寄存器使能了这一强制ROM启动功能，具体代码如下：  

```C
// ROM statrup过程中调用的函数
void SystemInit (void)
{
    // ...

    // Set Force ROM bits in RCM. We only set bit 2, so the RCM_MR register doesn't
    // falsely show that the ROM was booted via boot pin assertion.
    RCM->FM = RCM_FM_FORCEROM(2);

    // ...
}

// ROM跳转到Application之前调用的函数
void shutdown_cleanup(bool isShutdown)
{
    // ...

    // Disable force ROM.
	RCM->FM = RCM_FM_FORCEROM(0);

    // Clear status register (bits are w1c).
	RCM->MR = RCM_MR_BOOTROM(3);

    // ...
}
```

　　因为ROM里有了上述代码，所以只要芯片上电执行过ROM程序，除非是ROM主动跳转到了Application或者发生了冷启动，否则任何与ROM有关的配置修改操作都不会影响到下一次启动ROM的执行，这种机制可以确保Application一定会被ROM下载进Flash。

### 二、特性配置：BCA
　　除了启动配置外，KBOOT还支持特性配置，我们知道KBOOT提供的特性功能非常多，比如支持的外设种类丰富、超时时间可设、Application完整性校验、USB ID可设、运行时钟可配、加密特性支持、QSPI启动支持，这些特性可以通过BCA来配置，BCA是Bootloader Configuration Area的简称，KBOOT通过从BCA区域加载用户配置数据完成这些特性配置。BCA配置结构体原型如下（以K80芯片为例）：  

```C
//! @brief Format of bootloader configuration data on Flash.
typedef struct BootloaderConfigurationData
{
    uint32_t tag;                          //!< [00:03] Tag value used to validate the BCA data. Must be set to 'kcfg'.
    uint32_t crcStartAddress;              //!< [04:07]
    uint32_t crcByteCount;                 //!< [08:0b]
    uint32_t crcExpectedValue;             //!< [0c:0f]
    uint8_t enabledPeripherals;            //!< [10:10]
    uint8_t i2cSlaveAddress;               //!< [11:11]
    uint16_t peripheralDetectionTimeoutMs; //!< [12:13] Timeout in milliseconds for peripheral detection before jumping to application code
    uint16_t usbVid;                       //!< [14:15]
    uint16_t usbPid;                       //!< [16:17]
    uint32_t usbStringsPointer;            //!< [18:1b]
    uint8_t clockFlags;                    //!< [1c:1c] High Speed and other clock options
    uint8_t clockDivider;                  //!< [1d:1d] One's complement of clock divider, zero divider is divide by 1
    uint8_t bootFlags;                     //!< [1e:1e] One's complemnt of direct boot flag, 0xFE represents direct boot
    uint8_t pad0;                          //!< [1f:1f] One's complemnt of direct boot flag, 0xFE represents direct boot
    uint32_t mmcauConfigPointer;           //!< [20:23] Holds a pointer value to the MMCAU configuration
    uint32_t keyBlobPointer;               //!< [24:27] Holds a pointer value to the key blob array used to configure OTFAD
    uint8_t reserved[8];                   //!< [28:2f] Reserved.
    uint32_t qspi_config_block_pointer;    //!< [30:33] QSPI config block pointer.
} bootloader_configuration_data_t;
```

　　如果你想配置KBOOT的特性，必须按上述结构体格式准备好配置数据，具体数据值所代表含义请查看芯片手册Bootloader章节，痞子衡在后续文章里也会慢慢讲到。此处假设你已经准备好了BCA数据，那么这个BCA数据应该放在哪里呢？其实KBOOT已经指定好了BCA位置，见如下代码，BCA起始地址固定在APP_VECTOR_TABLE地址偏移0x3c0处，对于ROM Bootloader而言，BCA地址就是0x3c0，因为APP_VECTOR_TABLE=0；而对于Flash-Resident Bootloader而言，BCA地址是Bootloader指定的Application起始地址偏移0x3c0处。  

```C
//! @brief Flash constants.
enum _flash_constants
{
    //! @brief The bootloader configuration data location .
    //!
    //! A User Application should populate a BootloaderConfigurationData
    //! struct at 0x3c0 from the beginning of the application image which must
    //! be the User Application vector table for the flash-resident bootloader
    //! collaboration.
    kBootloaderConfigAreaAddress = (uint32_t)(APP_VECTOR_TABLE) + 0x3c0
};
```

　　最后再解释一下BCA地址为何是APP_VECTOR_TABLE + 0x3c0，我们知道ARM Cortex-M系统规定Application前1KB（0x0 - 0x3FF）应放中断向量表，Cortex-M最大支持256个中断，其中前16个是系统中断，后240个是外设中断，而Cortex-M厂商生产的芯片一般用不满240个外设中断，所以其实中断向量表后半部分其实是reserved的，因此我们可以把reserved区域里的0x3C0 - 0x3FF这64bytes用作BCA配置。  

　　至此，飞思卡尔Kinetis系列MCU的KBOOT形态痞子衡便介绍完毕了，掌声在哪里~~~ 

