----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的性能**。  

　　在前面的文章 [i.MXRT微控制器概览](http://www.cnblogs.com/henjay724/p/8556171.html) 里，痞子衡给大家简介过恩智浦半导体在2017年推出的新一代跨界微控制器i.MX RT系列，该系列第一款芯片i.MXRT105x性能完爆同时期市面上所有的微控制器，官方公布的CoreMark跑分高达3020，有人可能不明白这个数字意味着什么，作为对比，我们再来看看意法半导体最流行的芯片STM32F103RB，它的CoreMark是108（此处应该有类似My God的尖叫），真是没有对比就没有伤害，那么i.MXRT105x性能到底有没有这么强？今天痞子衡为大家实测一下。  

　　关于CoreMark标准的基本知识，痞子衡之前专门写过一篇文章 [微控制器CPU性能测试基准(EEMBC-CoreMark)](http://www.cnblogs.com/henjay724/p/8729364.html) ，本篇就是基于了解CoreMark基本知识之后的一次实践。来，让我们开始吧。  

### 一、准备工作
#### 1.1 硬件平台NXP i.MX RT1050 EVK
　　要开始实测i.MXRT105x的CoreMark，首先你得有一块开发板，恩智浦官网上有i.MXRT105x配套的评估板，如下图所示，痞子衡今天用的就是这块板子，板载主芯片型号为iMXRT1052DVL6。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/CoreMark_i.MXRT105xEVK_min.png" style="zoom:100%" />  

#### 1.2 集成开发环境IAR EWARM
　　ARM Cortex-M微控制器的集成开发环境有很多，其中IAR EWARM凭借优良的特性备受广大工程师青睐，且痞子衡在CoreMark标准基本知识介绍文章的最后贴出了一张iMXRT1050与STM32H743性能对比图，其中CoreMark跑分就是在IAR下得到的，所以今天痞子衡也选用IAR作为软件环境，具体版本为IAR EWARM v8.20.2。  

#### 1.3 官方软件开发包NXP MCUXpresso SDK
　　在开始移植CoreMark程序到i.MXRT1052上之前，我们需要先有一个i.MXRT1052的基本hello world的例程，当然我们可以对着数据手册自己从头写一个，但是这里痞子衡推荐使用官方软件开发包。  
　　注册并登录恩智浦官网，来到 [MCUXpresso SDK Builder](https://mcuxpresso.nxp.com/en/select) 页面，在"Select Development Board"里选择EVKB-IMXRT1050后点击Build MCUXpresso SDK后跳转到下一个页面，在"Developer Environment Settings"里选择IAR并点击Download SDK后便可得到SDK_2.3.1_EVKB-IMXRT1050.zip，下面是痞子衡下载的开发包具体版本信息：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/CoreMark_i.MXRT105x_SDK.PNG" style="zoom:100%" />  

### 二、开始实测
#### 2.1 跑通hello world
　　给i.MXRT1050 EVK板子供电（J2口接5V输出的电源），并且使用USB线连接电脑与板子的J28 USB口，此时在设备管理器应该可以看到USB虚拟的串口（EVK板载OpenSDA调试器内含USB转串口功能，如果看不到串口，请自行安装PE Micro驱动）。  
　　打开前一步下载的开发包里的\SDK_2.3.1_EVKB-IMXRT1050\boards\evkbimxrt1050\demo_apps\hello_world\iar\hello_world.eww工程，确认工程option里linker文件选择的是MIMXRT1052xxxxx_ram.icf，然后J21 JTAG口连接上Jlink Plus直接将工程下载进主芯片的RAM运行。  
　　如果工程运行正常，你在串口调试助手（115200，8N1）里应该能看到"hello world."打印输出。  

#### 2.2 移植CoreMark程序
　　以hello_world工程为基础，将从EEMBC官网下载到的coremark_v1.0.tgz包解压，将\coremark_v1.0\以及\coremark_v1.0\barebones\路径下的如下所有源文件(.c或.h)全部拷贝到hello world工程目录下  
```text
\coremark_v1.0
              \barebones            --移植到裸机下需要修改的文件
			            \core_portme.h    -- 移植平台工程具体配置信息
						\core_portme.c    -- 计时以及板级初始化实现  
						\cvt.c
						\ee_printf.c      -- 打印函数串口发送实现
              core_main.c           --主程序入口
              core_state.c          --状态机控制子程序
              core_list_join.c      --列表操作子程序
              core_matrix.c         --矩阵运算子程序
              core_util.c           --CRC计算子程序
              coremark.h            --工程配置与数据结构定义
```

　　将上面的所有coremark源文件全部添加进hello_world工程，并从工程中移除原主函数入口文件hello_world.c，此时基本coremark工程就完成了。但注意此时工程无法编译，因为从\coremark_v1.0\barebones\下拷贝的源文件还需要进一步修改。  

##### 2.2.1 板级初始化
　　程序移植的第一步是实现板载初始化，编译上一步建好的coremark工程会报这样的错误error "Call board initialization routines in portable init (if needed), in particular initialize UART!\n"。其实这是coremark作者故意在core_portme.c里的portable_init()函数里留下的提示，我们需要用i.MXRT1052的板级初始化函数替代这个#error，打开之前移除的hello_world.c文件，将其中板级初始化函数调用拷贝到portable_init函数中：  
```C
void portable_init(core_portable *p, int *argc, char *argv[])
{
    // 删除#error提示
	//#error "Call board initialization routines in portable init (if needed), in particular initialize UART!\n"

    // 添加板级初始化调用
    /* Init board hardware. */
    BOARD_InitHardware();
	//...
}
```

　　原hello_world里的BOARD_InitHardware已经包含了系统时钟以及UART初始化，所以这里的移植工作就算结束了。  

##### 2.2.2 计时功能
　　程序移植的第二步是实现计时功能，由于CoreMark得分其实是单位时间内跑了多少次CoreMark程序，所以CoreMark工程必须要有计时功能。编译coremark工程会报这样的错误error "You must implement a method to measure time in barebones_clock()! This function should return current time.\n"，这也是coremark作者故意在core_portme.barebones_clock()函数里留下的提示。  
　　关于计时器，第一个想到的自然是Cortex-M内核里的SysTick，不过考虑到coremark程序是要跑在600MHz的主频下，而SysTick计时器只有24bit，也就是说即使SysTick->LOAD设最大的reload值0xFFFFFF，也将每隔0.02796s(0x1000000/600MHz)产生一次SysTick中断，而CoreMark程序至少要跑10s以上，在coremark运行的10s内会产生357次（10/0.02796）SysTick中断，这无疑会降低coremark得分，所以SysTick直接被pass。(备注：SysTick->CTRL[2]用于选择SysTick的时钟源，默认1'b0为Core Clock，1'b1为外部clock，暂未研究这里的外部clock在i.MXRT105x上是否能用)。  
　　翻看i.MXRT1052的参考手册，其支持的计时器种类很多，有GPT、PIT、TMR、WDOG，就选择比较常用的32bit计时器PIT吧。  
　　之前下载的软件包里也有PIT的例程\SDK_2.3.1_EVKB-IMXRT1050\boards\evkbimxrt1050\driver_examples\pit，打开pit.c文件将其中PIT初始化相关代码放入新定义的timer_pit_init()函数中并在portable_init()中调用timer_pit_init()一次以完成PIT初始化。  
　　barebones_clock()函数中的#error也需要被PIT_GetCurrentTimerCount()函数替换，此外还需要添加定义#define CLOCKS_PER_SEC (24000000)，因为我们会在clock_config.c文件里的BOARD_BootClockRUN()函数最后添加如下语句以选择OSC作为PIT计时时钟，而EVK板载外部XTALOSC晶振是24MHz。  
```C
    /* Configure PIT divider */   
    CLOCK_SetMux(kCLOCK_PerclkMux, 1U); /* Set PERCLK_CLK source to OSC_CLK*/
    CLOCK_SetDiv(kCLOCK_PerclkDiv, 0U); /* Set PERCLK_CLK divider to 1 */
```
　　有朋友可能会好奇，我们似乎没有使能和实现PIT中断，其实在当前coremark工程下没有必要，因为PIT中断每隔178s(0x100000000/24MHz)才会触发一次，而我们只需要执行几十秒就够了，所以即使使能和实现PIT中断也用不到，当然如果能实现PIT中断更好，万一我们让coremark程序执行超过178s，不实现PIT中断就会导致计时出错。  
```C
#define CLOCKS_PER_SEC (24000000)

void timer_pit_init(void)
{
    /* Structure of initialize PIT */
    pit_config_t pitConfig;
    PIT_GetDefaultConfig(&pitConfig);
    /* Init pit module */
    PIT_Init(PIT, &pitConfig);
    /* Set max timer period for channel 0 */
    PIT_SetTimerPeriod(PIT, kPIT_Chnl_0, (uint32_t)~0);
    /* Start channel 0 */
    PIT_StartTimer(PIT, kPIT_Chnl_0);
}

CORETIMETYPE barebones_clock() {
    // 删除#error提示
	#error "You must implement a method to measure time in barebones_clock()! This function should return current time.\n"

    // 添加PIT计数值返回语句
    return (~PIT_GetCurrentTimerCount(PIT, kPIT_Chnl_0));
}

void portable_init(core_portable *p, int *argc, char *argv[])
{
    // 删除#error提示
	//#error "Call board initialization routines in portable init (if needed), in particular initialize UART!\n"

    // 添加板级初始化调用
    /* Init board hardware. */
    BOARD_InitHardware();

    // 添加PIT计时器初始化调用
    /* Init timer for microsecond function. */
    timer_pit_init();
	//...
}
```

##### 2.2.3 串口打印功能
　　程序移植的第三步是实现串口打印功能，由于hello_world工程已经包含串口打印功能，所以这里的修改比较简单。  

```C
void uart_send_char(char c) {
    // 删除#error提示
    //#error "You must implement the method uart_send_char to use this file!\n";

    // 添加UART发送函数调用
    if (c == '\n')
    {
        char tmp = '\r';
        LPUART_WriteBlocking((LPUART_Type *)BOARD_DEBUG_UART_BASEADDR, (const uint8_t *)&tmp, 1);
    }
    LPUART_WriteBlocking((LPUART_Type *)BOARD_DEBUG_UART_BASEADDR, (const uint8_t *)&c, 1);

    //...
}
```

#### 2.3 CoreMark参数配置
　　2.2里只是将基本CoreMark功能移植完成，但此时coremark工程还是无法编译，因为还有一些CoreMark配置没有确定，都在coremark_portme.h里，需要重定义下面三个宏，并且IAR的优化选项也要设置如下图所示：  

```C
#define ITERATIONS (40000)
#define COMPILER_VERSION "IAR EWARM v8.20.2"
#define COMPILER_FLAGS "High - Speed - No size constraints"
```

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/CoreMark_IAR_Optimizations_min.png" style="zoom:100%" />  

#### 2.4 输出CoreMark结果
　　到这里CoreMark的移植工作就完全结束了，此时coremark工程也应该能正常编译了。为得到最高的CoreMark得分，最后需要再确定两件事：一、Core Clock是否确定配置为600MHz；二、工程代码段/数据段是否放在了ITCM/DTCM RAM。  
　　打开clock_config.c文件，查看BOARD_BootClockRUN()函数，ARM Core/AHB时钟确实都为600MHz。  
```C
void BOARD_BootClockRUN(void)
{
    /* Boot ROM did initialize the XTAL, here we only sets external XTAL OSC freq */
    CLOCK_SetXtalFreq(24000000U);
    CLOCK_SetRtcXtalFreq(32768U);

    CLOCK_SetMux(kCLOCK_PeriphClk2Mux, 0x1); /* Set PERIPH_CLK2 MUX to OSC */
    CLOCK_SetMux(kCLOCK_PeriphMux, 0x1);     /* Set PERIPH_CLK MUX to PERIPH_CLK2 */

    /* Setting the VDD_SOC to 1.5V. It is necessary to config AHB to 600Mhz */
    DCDC->REG3 = (DCDC->REG3 & (~DCDC_REG3_TRG_MASK)) | DCDC_REG3_TRG(0x12);

    CLOCK_InitArmPll(&armPllConfig); /* Configure ARM PLL to 1200M */
    CLOCK_SetDiv(kCLOCK_ArmDiv, 0x1); /* Set ARM PODF to 1, divide by 2 */
    CLOCK_SetDiv(kCLOCK_AhbDiv, 0x0); /* Set AHB PODF to 0, divide by 1 */
    CLOCK_SetDiv(kCLOCK_IpgDiv, 0x3); /* Set IPG PODF to 3, divede by 4 */

	// ...
}
```
　　再打开MIMXRT1052xxxxx_ram.icf文件，查看text段确实放在了ITCM里(0x0 - 0x7FFFF)，data段确实放在了DTCM里（0x20000000 - 0x2007FFFF）。  
```C
define symbol m_interrupts_start       = 0x00000000;
define symbol m_interrupts_end         = 0x000003FF;

define symbol m_text_start             = 0x00000400;
define symbol m_text_end               = 0x0001FFFF;

define symbol m_data_start             = 0x20000000;
define symbol m_data_end               = 0x2001FFFF;
```

　　还等什么，将coremark工程赶紧下载进芯片并打开串口调试助手看CoreMark得分啊。痞子衡实测的CoreMark得分为2952，这与官方标称的3020有一点点差距，但这性能已经足够优秀了！！！  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/CoreMark_myScore.PNG" style="zoom:100%" />  

　　想偷懒的朋友直接移步痞子衡的github [https://github.com/JayHeng/cortex-m_app](https://github.com/JayHeng/cortex-m_app) 去下载移植好的工程，工程在\cortex-m_app\apps\coremark_imxrt1052\bsp\build\coremark.eww  

　　至此，飞思卡尔i.MX RT系列MCU的CoreMark性能与实测痞子衡便介绍完毕了，掌声在哪里~~~
