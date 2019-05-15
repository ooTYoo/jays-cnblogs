----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**恩智浦i.MX RTxxx系列MCU的性能**。  

　　在前面的文章 [i.MXRTxxx微控制器概览]() 里，痞子衡给大家简介过恩智浦半导体在2018年推出的全新跨界微控制器i.MX RTxxx系列，该系列第一款芯片i.MXRT600搭载一颗Cortex-M33控制内核和一颗Tensilica HiFi4 DSP处理内核，该芯片可在超低功耗边缘处理应用中实现高效本地音频预处理、沉浸式3D音频播放和支持语音的体验。今天痞子衡先为大家实测一下其Cortex-M33控制内核的性能，性能测试程序采用经典的Dhrystone算法。  

　　关于Dhrystone标准的基本知识，痞子衡之前专门写过一篇文章 [微处理器CPU性能测试基准(Dhrystone)](https://www.cnblogs.com/henjay724/p/10856831.html) ，本篇就是基于了解Dhrystone基本知识之后的一次实践。来，让我们开始吧。  

### 一、准备工作
#### 1.1 硬件平台NXP i.MX RT600 EVK
　　要开始实测i.MXRT600的Dhrystone，首先你得有一块开发板，恩智浦官网上有i.MXRT600配套的评估板，如下图所示，痞子衡今天用的就是这块板子，板载主芯片型号为iMXRT685EVKA。  

<img src="http://henjay724.com/image/cnblogs/Dhrystone_i.MXRT685EVK_min.jpg" style="zoom:100%" />

#### 1.2 集成开发环境IAR EWARM
　　ARM Cortex-M微控制器的集成开发环境有很多，其中IAR EWARM凭借优良的特性备受广大工程师青睐，今天痞子衡就选用IAR作为软件环境，具体版本为IAR EWARM v8.32.2。  

#### 1.3 官方软件开发包NXP MCUXpresso SDK
　　在开始移植Dhrystone程序到i.MXRT685上之前，我们需要先有一个i.MXRT685的基本hello world的例程，当然我们可以对着数据手册自己从头写一个，但是这里痞子衡推荐使用官方软件开发包。  
　　注册并登录恩智浦官网，来到 [MCUXpresso SDK Builder](https://mcuxpresso.nxp.com/en/select) 页面，在"Select Development Board"里选择EVK-IMXRT685后点击Build MCUXpresso SDK后跳转到下一个页面，在"Developer Environment Settings"里选择IAR并点击Download SDK后便可得到SDK_2.5.0_EVK-MIMXRT685.zip，下面是痞子衡下载的开发包具体版本信息：  

<img src="http://henjay724.com/image/cnblogs/Dhrystone_i.MXRT600_SDK.PNG" style="zoom:100%" />

### 二、开始实测
#### 2.1 跑通hello world
　　使用USB线连接电脑与板子的J5 USB口，此时在设备管理器应该可以看到USB虚拟的串口（EVK板载LPC-LINK2调试器内含USB转串口功能，如果看不到串口，请自行安装LPC-LINK2驱动）。  
　　打开前一步下载的开发包里的\SDK_2.5.0_EVK-MIMXRT685\boards\evkimxrt685\demo_apps\hello_world\iar\hello_world.eww工程，确认工程option里linker文件选择的是MIMXRT685Sxxxx_ram.icf，然后使用板载调试器直接将工程下载进主芯片的RAM运行。  
　　如果工程运行正常，你在串口调试助手（115200，8N1）里应该能看到"hello world."打印输出。  

#### 2.2 移植Dhrystone程序
　　以hello_world工程为基础，将从Roy Longbottom的网站下载到的classic_benchmarks.tar.gz包解压，将\classic_benchmarks\source_code\dhrystone2\路径下的如下所有源文件(.c或.h)全部拷贝到hello world工程目录下  
```text
\classic_benchmarks\source_code\dhrystone2
                                          \dhry.h          --关于兼容性的原型定义
                                          \dhry_1.c        --主程序入口
                                          \dhry_2.c        --算法子程序
```

　　将上面所有Dhrystone源文件全部添加进hello_world工程并将工程更名为dhrystone，然后再将工程中原主函数入口文件hello_world.c更名为dhrystone.c，此时基本Dhrystone工程就完成了。但注意此时工程无法编译，因为Dhrystone源文件还需要进一步修改。  

##### 2.2.1 适配嵌入式平台
　　我们下载的Dhrystone源码本用作在PC上运行的，所以源码里面有一些仅适用于PC上运行的代码，比如计时部分、文件I/O部分，需要将这些代码全部删除以适合在嵌入式平台运行。  
　　关于计时部分，需要删除dhry_1.c文件里的#include < time.h> 语句，并且删除dhry.h文件里跟TIME宏相关的如下代码：  

```C
#ifndef TIME
#define TIMES
#endif
/* Use times(2) time function unless    */
/* explicitly defined otherwise         */

#ifdef TIMES
#include <sys/types.h>
#include <sys/times.h>
/* for "times" */
#endif
```

　　关于文件I/O部分，需要删除dhry_1.c文件里的#include < stdio.h> 语句，以及如下涉及文件I/O（主要是关于Dhry.txt的操作）的代码：  

```C
#include <stdio.h> // 需删除

void main (int argc, char *argv[]) // 需更改为void main(void)
{
    // ...
	// 以下代码需删除
    /* Initializations */
    if (argc > 1)
    {
        switch (argv[1][0])
        {
            case 'N':
                nopause = 0;
                break;
            case 'n':
                nopause = 0;
                break;
        }
    }

    if ((Ap = fopen("Dhry.txt","a+")) == NULL)
    {
        printf(" Can not open Dhry.txt\n\n");
        printf(" Press Enter\n\n");
        int g = getchar();
        exit(1);
    }

    // ...
    {
	    // ...
        /************************************************************************
         *                Add results to output file Dhry.txt                   *
         ************************************************************************/
        fprintf (Ap, " #####################################################\n\n");
        fprintf (Ap, " Dhrystone Benchmark 2.1 %s via C/C++ %s\n", options, timeday);
        // ...
        fclose(Ap);
    }
}
```

##### 2.2.2 板级初始化
　　上一节里已经将dhry_1.c里面的main函数形参int argc, char *argv[]改成了void，由于dhrystone.c（原hello_world.c）里已有main函数，且该main函数中含有板级初始化代码，所以我们需要将dhry_1.c里的main函数更名（比如可更名为dhrystone()），使dhrystone源码作为子程序来调用，这样便于往不同MCU平台移植，最后直接dhrystone.c里的main函数中增加dhrystone()的调用即可。  

```C
// dhry_1.c文件中
void main (void) // 需更改为void dhrystone(void)
{
    ...
}

// dhrystone.c文件中
int main(void)
{
    /* Init board hardware. */
    BOARD_InitPins();
    BOARD_BootClockRUN();
    BOARD_InitDebugConsole();

	// 增加的dhrystone函数调用
    dhrystone();

    while (1)
    {
    }
}
```

##### 2.2.3 计时功能
　　原dhry_1.c文件里的关于计时部分的代码应该做一些微调整，time.h头文件的包含应该去除，这是Windows系统里的头文件，start_time(), end_time()是基于time.h里clock_gettime()函数而封装的API，secs是用于记录时间差的变量，这些API和全局变量都可以保留，但需要用MCU里的计时器重新实现。  

```C
#include <time.h> // 需删除

void dhrystone (void)
{
    // ...
    do
    {
        start_time();
        // ...
        end_time();
        User_Time = secs;
    }
    while (count >0);
}
```

　　关于计时器，第一个想到的自然是Cortex-M内核里的SysTick，不过考虑到Dhrystone程序是要跑在300MHz的主频下，而SysTick计时器只有24bit，也就是说即使SysTick->LOAD设最大的reload值0xFFFFFF，也将每隔0.05592s(0x1000000/300MHz)产生一次SysTick中断，而Dhrystone程序至少要跑2s以上，在Dhrystone运行的2s内会产生35次（2/0.05592）SysTick中断，这无疑会降低Dhrystone得分，所以SysTick直接被pass。(备注：SysTick->CTRL[2]用于选择SysTick的时钟源，默认1'b0为Core Clock，1'b1为外部clock，暂未研究这里的外部clock在i.MXRT685上是否能用)。  
　　翻看i.MXRT685的参考手册，其支持的计时器种类很多，有CTIMER、SCT、MRT、UTICK、WWDT，就选择比较常用的32bit计时器CTIMER吧。  
　　之前下载的软件包里也有CTIMER的例程\SDK_2.5.0_EVK-MIMXRT685\boards\evkimxrt685\driver_examples\ctimer，打开simple_match_interrupt.c文件将其中CTIMER初始化相关代码放入新定义的timer_ctimer_init()函数中并在start_time()中调用timer_ctimer_init()一次以完成CTIMER初始化。  
　　此外还需要添加定义#define CLOCKS_PER_SEC (16000000)，因为CTIMER选择的时钟源是16MHz。  

```C
#define CLOCKS_PER_SEC (16000000)

volatile uint32_t s_timerHighCounter = 0;
void ctimer_match0_callback(uint32_t flags)
{
    s_timerHighCounter++;
}

void timer_ctimer_init(void)
{
    ctimer_config_t config;

    /* Use 16 MHz clock for the Ctimer2 */
    CLOCK_AttachClk(kSFRO_to_CTIMER2);

    CTIMER_GetDefaultConfig(&config);
    CTIMER_Init(CTIMER, &config);

    /* Configuration 0 */
    matchConfig0.enableCounterReset = true;
    matchConfig0.enableCounterStop = false;
    matchConfig0.matchValue = (uint32_t)~0;
    matchConfig0.outControl = kCTIMER_Output_Toggle;
    matchConfig0.outPinInitState = false;
    matchConfig0.enableInterrupt = true;

    CTIMER_RegisterCallBack(CTIMER, &ctimer_callback_table[0], kCTIMER_SingleCallback);
    CTIMER_SetupMatch(CTIMER, CTIMER_MAT0_OUT, &matchConfig0);
    CTIMER_StartTimer(CTIMER);
}

void start_time(void)
{
    timer_ctimer_init();
    s_timerHighCounter = 0;
}

void end_time(void)
{
    uint64_t retVal;
    uint32_t high;
    uint32_t low;
    do
    {
        high = s_timerHighCounter;
        low = CTIMER_GetTimerCountValue(CTIMER);
    } while (high != s_timerHighCounter);
    retVal = ((uint64_t)high << 32U) + low;
    
    CTIMER_StopTimer(CTIMER);

    secs = retVal / (CLOCKS_PER_SEC * 1.0);
}
```

##### 2.2.4 串口打印功能
　　串口打印功能的改动比较简单，直接把原dhry_1.c文件里的printf()全部替换成PRINTF()即可，PRINTF函数在原hello world工程里已经实现了。  

#### 2.3 Dhrystone参数配置
　　痞子衡在Dhrystone标准的基本知识介绍里说过，Dhrystone几乎没有参数配置，唯一需要注意的就是REG，Cortex-M33平台支持register关键字，所以我们在IAR工程option里宏定义框内加上 REG=register。  
　　此外我们还需要在宏定义框内设置额外两个宏 NUMBER_OF_RUNS、CORE_FREQ_MHz，前者代表跑dhrystone核心算法程序的总次数，后者是当前MCU实际运行主频。最后还需要设置一下IAR的优化选项，如下图所示：  

<img src="http://henjay724.com/image/cnblogs/Dhrystone_IAR_Optimizations_min.png" style="zoom:100%" />  

#### 2.4 输出Dhrystone结果
　　到这里Dhrystone的移植工作就完全结束了，此时Dhrystone工程也应该能正常编译了。为得到最高的Dhrystone得分，最后需要再确定两件事：一、Core Clock是否确定配置为300MHz；二、工程代码段/数据段是否放在了SRAM。  
　　打开clock_config.c文件，查看BOARD_BootClockRUN()函数，ARM Core/AHB时钟仅保守地配了250MHz，让我们修改Pfd0的分频系数，从19修改为16，直接超频到300MHz。  
```C
void BOARD_BootClockRUN(void)
{
    // ...

    CLOCK_InitSysPll(&g_configSysPll); /* Configure system PLL to 528Mhz. */
    /* Valid PFD values are decimal 12-35. */
	// 将分频系数修改16
    // CLOCK_InitSysPfd(kCLOCK_Pfd0, 19); /* Enable main PLL clock 500MHz. */
	CLOCK_InitSysPfd(kCLOCK_Pfd0, 16); /* Enable main PLL clock 594MHz. */

    // ...

    /* Let CPU run on SYS PLL PFD0 with divider 2. */
    CLOCK_SetClkDiv(kCLOCK_DivSysCpuAhbClk, 2);
    CLOCK_AttachClk(kMAIN_PLL_to_MAIN_CLK);

	// ...
}
```
　　再打开MIMXRT685Sxxxx_ram.icf文件，查看text/data段确实放在了SRAM里(0x0 - 0x47FFFF)：  

```C
define symbol m_interrupts_start               = 0x00080000;
define symbol m_interrupts_end                 = 0x00080143;

define symbol m_text_start                     = 0x00080144;
define symbol m_text_end                       = 0x001BFFFF;

define symbol m_data_start                     = 0x001C0000;
define symbol m_data_end                       = 0x002FFFFF;
```

　　还等什么？将Dhrystone工程赶紧下载进芯片并打开串口调试助手看Dhrystone得分啊。痞子衡实测的Dhrystone得分为400 DMIPS。  
<img src="http://henjay724.com/image/cnblogs/Dhrystone_myScore.PNG" style="zoom:100%" />

　　想偷懒的朋友直接移步痞子衡的github [https://github.com/JayHeng/Cortex-M-Apps](https://github.com/JayHeng/Cortex-M-Apps) 去下载移植好的工程，工程在\Cortex-M-Apps\apps\dhrystone_imxrt685\bsp\下面。  

　　至此，恩智浦i.MX RTxxx系列MCU的Dhrystone性能与实测痞子衡便介绍完毕了，掌声在哪里~~~  
