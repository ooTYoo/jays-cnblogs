----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔Kinetis MCU开发板板载OpenSDA调试器（上篇）**。  

　　众所周知，嵌入式软件开发几乎离不开调试器，因为写一个稍有代码规模（5K行以上）的嵌入式应用程序一般不可能一次性搞定，没有任何bug，出了bug并不可怕，只要我们能尽快定位bug并修复即可，调试器就是定位bug的利器，有了调试器我们便可以进入系统主控芯片内部一窥究竟，控制芯片执行代码的动作，实时查看芯片内部状态，辅以各种调试技巧让bug无处藏身。  
　　飞思卡尔Kinetis MCU属于ARM Cortex-M系列芯片，因此本文主要介绍的Cortex-M系列芯片调试器，目前市面上Cortex-M调试器种类（这里主要指的是硬件生产商）非常多，主要分为如下两大阵营：  

> * 第三方公司出品：SEGGER的J-Link、IAR的I-jet、ARM的ULINK、开源项目OpenJTAG等  
> * 半导体厂商出品：Freescale的OpenSDA、NXP的LPC-Link、ST的ST-LINK、TI的Stellaris ICDI、Nuvoton的Nu-Link等。

　　本文要讲的主角OpenSDA属于阵营里的后者，其一般不单独存在，都是附着在飞思卡尔Kinetis官方开发板上，你可以理解为OpenSDA调试器是专为Kinetis芯片设计的（但其实其firmware设计是可以做到通用于所有Cortex-M芯片的），今天痞子衡就为大家介绍OpenSDA项目的来龙去脉，让大家对OpenSDA项目有一个整体认识。  

### 一、OpenSDA项目背景
　　OpenSDA全称是Open-standard Serial and Debug Adapter，从名字上来看，这个项目是要做一个开放标准的串口与调试适配器，该项目主要是为了Kinetis芯片配套官方开发板而设计的，直接板载在Kinetis官方开发板上，用于取代传统的第三方仿真器（比如J-Link）+开发板的开发调试模式。  
　　众所周知，ARM公司于2010年2月正式发布Cortex-M4内核，飞思卡尔于2010年8月推出其第一款ARM Cortex-M内核芯片Kinetis K60，这也是业界一款Cortex-M4内核的芯片。OpenSDA项目并不是从Kinetis诞生之初便存在的，不信你可以查看最早的Kinetis Tower开发板 [TWR-K60D100M](https://www.nxp.com/cn/support/developer-resources/evaluation-and-development-boards/tower-development-boards/mcu-and-processor-modules/kinetis-modules/kinetis-k60-100-mhz-mcu-tower-system-module:TWR-K60D100M) 原理图（该板通用于Kinetis K10、K20、K60芯片），早期的Kinetis开发板板载调试器为OSBDM/OSJTAG，这是一种早已普遍应用于飞思卡尔HCS12、DSC、PowerPC系列芯片开发板上的调试器，OSBDM/OSJTAG调试器主芯片为MC9S08JM60CLD（S08JM系列）。  

<img src="http://henjay724.com/image/cnblogs/OpenSDA_App_TWR-K60_OSJTAG.PNG" style="zoom:100%" />

　　随着Kinetis芯片的不断推出，OpenSDA项目逐渐酝酿成熟，尤其是2012年随着飞思卡尔Cortex-M0+内核Kinetis L系列芯片的走红，OpenSDA调试器开始大面积应用于Kinetis L系列芯片配套的Freedom开发板上，从此取代OSBDM/OSJTAG成为Kinetis开发板上的标准调试器，大家可以看一下下面这款经典的 [FRDM-KL25Z](https://www.nxp.com/cn/support/developer-resources/evaluation-and-development-boards/freedom-development-boards/mcu-boards/freedom-development-platform-for-kinetis-kl14-kl15-kl24-kl25-mcus:FRDM-KL25Z) 原理图，其板载调试器即为OpenSDA调试器（V1），OpenSDA调试器主芯片选用的MK20DX128VFM5（Kinetis K20系列）。  

<img src="http://henjay724.com/image/cnblogs/OpenSDA_App_FRDM-KL25_OpenSDA.PNG" style="zoom:100%" />

### 二、OpenSDA硬件电路
　　概括来说，OpenSDA调试器主要由硬件和软件两部分组成，其中硬件主要分为主芯片和外围接口电路，下面痞子衡分别为大家介绍OpenSDA的主芯片和外围接口电路。  

#### 2.1 不可撼动的男主（MK20DX128VFM5）
　　对于一个调试器项目来说，主芯片的选用至关重要，这直接决定了调试器的性能。OpenSDA调试器主芯片选用的是MK20DX128VFM5，这是一款Cortex-M4内核，主频50MHz，16KB SRAM，160KB ROM（128KB Flash，32KB FlexNVM(最大2KB EEPROM)），内置1个USBFS控制器，QFN32封装的芯片。  
　　为何选择Kinetis芯片作为调试器主控？当然是为了进一步推广Kinetis系列的知名度，毕竟飞思卡尔的Cortex-M芯片推出较晚。那么为何选择Kinetis K2x系列芯片呢？这要从下面Kinetis K系列的特性表里才能找到答案，从下表可以我们看出K2x系列是支持USB接口的入门芯片，而调试器主控是必须要支持USB接口的。  

<img src="http://henjay724.com/image/cnblogs/OpenSDA_App_kinetis_k_series.PNG" style="zoom:100%" />

　　为何选择型号为MK20DX128VFM5的K2x芯片呢？虽然这款芯片属于K2x系列里最低配型号（考虑到调试器将随着开发板大面积推广，成本也是一个不可忽略的因素），但是其外设（USBFS, UART, SPI）与存储（16KB RAM，160KB ROM）方面是满足调试器项目要求的。有人可能会问，50MHz主频会不会有点低，从而影响调试器性能？痞子衡可以明确地告诉你，性能是够的，作为对比SEGGER J-Link V7/V8里的主控是Atmel AT91SAM7S64（ARM7TDMI内核, 55MHz主频）。  

<img src="http://henjay724.com/image/cnblogs/OpenSDA_App_kinetis_k2x_series.PNG" style="zoom:100%" />

#### 2.2 几经搬迁的行宫（V1/V2/V2.1/V2.2）
　　确定了调试器主芯片，下一步便是设计调试器外围接口电路，实际上外围电路发展至今不止一个版本，我们可以在飞思卡尔官网 [OpenSDA项目主页](https://www.nxp.com/support/developer-resources/run-time-software/kinetis-developer-resources/ides-for-kinetis-mcus/opensda-serial-and-debug-adapter:OPENSDA) 里的Download – OpenSDA Bootloader and Application下面找到如下各版本典型的开发板：  

<table><tbody>
    <tr>
        <th><img src="http://henjay724.com/image/cnblogs/OpenSDA_App_hw_version1.0_frdm-kl25z.PNG" style="zoom:70%" /></th>
        <th><img src="http://henjay724.com/image/cnblogs/OpenSDA_App_hw_version2.0_twr-k65f180m.PNG" style="zoom:70%" /></th>
    </tr>
    <tr>
        <th><img src="http://henjay724.com/image/cnblogs/OpenSDA_App_hw_version2.1_twr-k80f150m.PNG" style="zoom:70%" /></th>
        <td><img src="http://henjay724.com/image/cnblogs/OpenSDA_App_hw_version2.2_frdm-kw41z.PNG" style="zoom:70%" /></td>
    </tr>
</table>

　　不过这些版本主要都是硬件设计细节方面的小改动，而对于调试器软件开发而言其实并没有变化，因为接口Pinout并没有改变，痞子衡将上述各开发板原理图里的OpenSDA电路仔细查看对比整理出了如下通用的OpenSDA硬件原理简图：  

<img src="http://henjay724.com/image/cnblogs/OpenSDA_App_hw_block_diagram.PNG" style="zoom:100%" />

　　从上面原理简图我们可以看出，OpenSDA电路其实非常简洁，除了电源、晶振、主芯片K20自身的调试接口外，其他跟OpenSDA调试器功能相关的主要有五部分电路：
> 1. **蓝色LED**： 指示OpenSDA调试器工作状态，主要有两种状态，一种是常亮（正常运行模式），另一种是闪烁（自身Firmware更新模式）。
> 2. **Boot按键**： 用于辅助进入OpenSDA调试器自身Firmware更新模式（也叫MSD Bootloader模式），电路上电前按下Boot按键不放即可进入MSD Bootloader模式（可在PC里看到新增名为BOOTLOADER的磁盘），往磁盘里拷贝相应调试器Firmware文件即可完成更新。
> 3. **USB接口**： 与上位机PC连接，完成后续USB转串口（可到PC设备管理器看到新增串口设备）, SWD调试交互功能。
> 4. **UART接口**： 与目标MCU连接。
> 5. **SWD接口**： 与目标MCU连接。

### 三、OpenSDA软件设计
　　随着OpenSDA项目的不断迭代，目前（2018年9月）OpenSDA调试器版本已经更新到V2.2，飞思卡尔官网有 [OpenSDA项目主页](https://www.nxp.com/support/developer-resources/run-time-software/kinetis-developer-resources/ides-for-kinetis-mcus/opensda-serial-and-debug-adapter:OPENSDA)，在主页上我们可以看到如下OpenSDA项目版本对比：  

<img src="http://henjay724.com/image/cnblogs/OpenSDA_App_comparsion_table_of_opensda_version.PNG" style="zoom:100%" />

#### 3.1 软件架构
　　不管是哪个版本的OpenSDA，其软件架构是一致的，如下图所示，OpenSDA软件主要由两部分组成：MSD Bootloader、OpenSDA Application(Firmware)，其中MSD Bootloader占据调试器主控K20芯片内部Flash的上半区，K20芯片上电永远先执行MSD Bootloader程序，MSD Bootloader功能比较单一，就是用来更新OpenSDA Firmware，这样使得调试器软件升级比较容易。  
　　OpenSDA Firmware则是调试器的核心，调试器的主要功能（SWD调试、USB转串口）均是这个应用程序实现的，OpenSDA Firmware占据K20芯片内部Flash的下半区，其本身不能上电自动执行，必须由MSD Bootloader引导执行。  

<img src="http://henjay724.com/image/cnblogs/OpenSDA_App_sw_block_diagram.PNG" style="zoom:100%" />

#### 3.2 后宫佳丽有三个（软件服务商）
　　说到OpenSDA Firmware的供应商，主要有三个，分别是P&EMicro、ARM Mbed、SEGGER，且听痞子衡一一道来：  
##### 3.2.1 原配P&E Micro
　　P&EMicro公司是OpenSDA项目最早的Firmware软件服务商，其Firmware软件专为OpenSDA设计，我们可以从下面网站下载到其开发的Firmware和配套上位机驱动：  

> * P&EMicro OpenSDA主页: http://www.pemicro.com/opensda/

　　不过比较遗憾的是P&EMicro公司并不开源其开发的Firmware源码，而在OpenSDA这个立志要开源的调试器项目里，Firmware不开源是一件比较奇怪的事，因此P&EMicro版本的Firmware逐渐没落。

##### 3.2.2 新欢ARM Mbed（DAPLink）
　　ARM公司（Mbed）是OpenSDA项目第二个Firmware软件服务商，虽然ARM官方已经设计了自己的ULINK调试器（ULINK主控为Cypress增强8051内核芯片AN2131QC、ULINK2主控为NXP公司ARM7内核芯片LPC2148FB064，ULINKpro主控为Xilinx VIRTEX FPGA），但为了ARM芯片的开放生态，其也同时主导了DAPLink项目（早期叫CMSIS-DAP项目，后来更名为DAPLink），下面是其官网：  

> * ARMMbed CMSIS-DAP主页: https://os.mbed.com/handbook/cmsis-dap-interface-firmware
> * ARMMbed CMSIS-DAP源码: https://github.com/mbedmicro/CMSIS-DAP
> * ARMMbed DAPLink主页: https://os.mbed.com/handbook/DAPLink
> * ARMMbed DAPLink源码: https://github.com/ARMmbed/DAPLink

　　DAPLink项目是完全开源的，目前支持的主控芯片主要有5款：K20DX, KL26, LPC11U35, LPC4322, SAM3U2C，其中K20DX即是应用于OpenSDA的主控，另外4款芯片分别用于其他类型的调试器，此处不再展开。DAPLink版Firmware因其开放的生态使其大受欢迎，是目前的主流。  

##### 3.2.3 野花SEGGER（J-Link）
　　SEGGER公司是OpenSDA项目第三个Firmware软件服务商，虽然SEGGER设计已久的通用型J-Link调试器是目前市场上最流行的，但SEGGER并没有因此固步自封，SEGGER看到OpenSDA调试器在Kinetis开发板上大面积推广，想到了一个非常聪明的策略，那便是免费为OpenSDA设计一个与J-Link软件兼容的Firmware，使OpenSDA摇身一变成为一个简配版J-Link。  

> * SEGGER OpenSDA主页: https://www.segger.com/products/debug-probes/j-link/models/other-j-links/opensda-sda-v2/

　　由于SEGGER公司是个靠软件服务收费的商业公司，因此SEGGER版Firmware并不开源，并且调试功能上也受一定限制，但这并不影响SEGGER版Firmware的流行，因此J-Link调试器实在是太普及了，J-Link上位机软件功能（J-Link Commander等）也非常强大。  

　　至此，飞思卡尔Kinetis MCU开发板板载OpenSDA调试器（上篇）痞子衡便介绍完毕了，掌声在哪里~~~  

