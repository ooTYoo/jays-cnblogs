----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**微处理器CPU性能测试基准Dhrystone**。  

　　在嵌入式系统行业用于评价CPU性能指标的标准主要有三种：Dhrystone、MIPS、CoreMark，其中Dhrystone是一种古老的却历时30年而不衰的嵌入式系统处理器测试基准，至今仍为各大处理器生产厂商所采用。今天痞子衡就和大家详细聊一聊Dhrystone。  

### 一、经典性能测试标准集
　　在讲Dhrystone之前，痞子衡想先给大家简介一下19世纪70-80年代开始流行的几个性能测试标准，它们分别是Livermore、Whetstone、Linpack、Dhrystone，这四个性能测试标准也被合称为Classic Benchmark。这个网址简单介绍了四大经典性能测试标准历史 [http://www.roylongbottom.org.uk/classic.htm](http://www.roylongbottom.org.uk/classic.htm)。  

![](http://henjay724.com/image/cnblogs/Dhrystone_classic_benchmarks.PNG)  

　　细心的朋友应该会注意到Dhrystone与另一标准Whetsone名字有点类似，其实Dhrystone就是为了与算法Whetsone区分而设计的。Whetsone于1972年所开发，主要目的是模仿60个1970年后的程序算法。其最有名的版本为Fortran版，高度反映了60年代数字计算方向。Dhrystone与Whetsone不同之处在于其并不包括浮点运算。  

### 二、Dhrystone标准
　　Dhrystone是由Reinhold P. Weicker在1984年提出来的一个基准测试程序，其主要目的是测试处理器的整数运算和逻辑运算的性能。Dhrystone首先用Ada语言发布，后来Rick Richardson为Unix开发了用C语言编写的Version 1.1，这个版本也成功的推动了Dhrystone的广泛应用。  
　　Dhrystone标准的测试方法很简单，就是单位时间内跑了多少次Dhrystone程序，其指标单位为DMIPS/MHz。MIPS是Million Instructions Per Second的缩写，每秒处理的百万级的机器语言指令数。DMIPS中的D是Dhrystone的缩写，它表示了在Dhrystone标准的测试方法下的MIPS。  

#### 2.1 获取程序
　　Dhrystone程序的最新版本是2.1，其实际上于1988年便已停更。Dhrystone并没有官网，所以想下载其源程序可能会有很多来源，有各种语言版本的实现，以及各种平台下的移植程序。  
　　Roy Longbottom，是一个来自英国政府计算机采购部门Central Computer and Telecommunications Agency (CCTA)的职员，他制作了一个PC性能测试结果网站，搜集了很多性能测试程序以及结果，其中便有Dhrystone，我们可以从他的网站下载Dhrystone源码(C语言版)。  
> 核心程序下载 http://www.roylongbottom.org.uk/classic_benchmarks.tar.gz  

　　核心程序包下载后，在\classic_benchmarks\source_code\dhrystone2\下可找到源代码。详细文件目录如下：  
```text
\classic_benchmarks\source_code\dhrystone2
                                          \dhry.h          --关于兼容性的原型定义
                                          \dhry_1.c        --主程序入口
                                          \dhry_2.c        --算法子程序
```

　　如果是移植到ARM Cortex-M平台下裸系统运行，一般只需要简单修改dhry.h和dhry_1.c文件即可，Dhrystone本身并没有太多移植工作，其源码本是用作在PC上运行的，而在嵌入式系统里运行仅需要把一些文件I/O的相关代码删除即可，此外就是计时函数和打印函数的重实现。  

#### 2.2 配置参数
　　Dhrystone源码几乎没有提供配置选项，唯一一个能算得上的配置就是关于REG的宏定义，即你所选用的IDE和嵌入式平台是否支持regiser关键字。  

#### 2.3 程序解析
　　让我们尝试分析Dhrystone主函数入口main：  
```c
void main (int argc, char *argv[])
{
          One_Fifty   Int_1_Loc;
    REG   One_Fifty   Int_2_Loc;
          One_Fifty   Int_3_Loc;
    REG   char        Ch_Index;
          Enumeration Enum_Loc;
          Str_30      Str_1_Loc;
          Str_30      Str_2_Loc;
    REG   int         Run_Index;
    REG   int         Number_Of_Runs;
          int         endit, count = 10;
    // ...

    // 定义和初始化关键buffer
    Next_Ptr_Glob = (Rec_Pointer) malloc (sizeof (Rec_Type));
    Ptr_Glob = (Rec_Pointer) malloc (sizeof (Rec_Type));
    Ptr_Glob->Ptr_Comp                    = Next_Ptr_Glob;
    // ...

    // 设置循环跑Dhrystone核心算法程序次数
    Number_Of_Runs = 5000;

    do
    {
        Number_Of_Runs = Number_Of_Runs * 2;
        count = count - 1;

        // 开始循环跑Dhrystone核心算法程序且记录累计消耗时间
        start_time();
        for (Run_Index = 1; Run_Index <= Number_Of_Runs; ++Run_Index)
        {
            Proc_5();
            Proc_4();
            // ...
        }
        end_time();
        User_Time = secs;

        printf ("%12.0f runs %6.2f seconds \n",(double) Number_Of_Runs, User_Time);
        if (User_Time > 2)
        {
            count = 0;
        }
        else
        {
            if (User_Time < 0.05)
            {
                Number_Of_Runs = Number_Of_Runs * 5;
            }
        }
    }
    while (count >0);

    // ...
	// 最终信息的打印
    if (User_Time < Too_Small_Time)
    {
        printf ("Measured time too small to obtain meaningful results\n");
        printf ("Please increase number of runs\n");
        printf ("\n");
    }
    else
    {
        Microseconds = User_Time * Mic_secs_Per_Second / (double) Number_Of_Runs;
        Dhrystones_Per_Second = (double) Number_Of_Runs / User_Time;
        Vax_Mips = Dhrystones_Per_Second / 1757.0;

        printf ("Microseconds for one run through Dhrystone: ");
        printf ("%12.2lf \n", Microseconds);
        printf ("Dhrystones per Second:                      ");
        printf ("%10.0lf \n", Dhrystones_Per_Second);
        printf ("VAX  MIPS rating =                          ");
        printf ("%12.2lf \n",Vax_Mips);
        printf ("\n");
	}
    // ...
}
```

#### 2.4 结果格式
　　当移植好Dhrystone程序后，便可以开始跑起来了，下面是一个主频100MHz的Pentium处理器跑分结果：  

```text
 Dhrystone Benchmark  Version 2.1 (Language: C)

 Final values:

 Int_Glob:      O.K.  5
 Bool_Glob:     O.K.  1
 Ch_1_Glob:     O.K.  A
 Ch_2_Glob:     O.K.  B
 Arr_1_Glob[8]: O.K.  7
 Arr_2_Glob8/7: O.K.     1600010
 Ptr_Glob->
   Ptr_Comp:       *  98008
   Discr:       O.K.  0
   Enum_Comp:   O.K.  2
   Int_Comp:    O.K.  17
   Str_Comp:    O.K.  DHRYSTONE PROGRAM, SOME STRING
 Next_Ptr_Glob->
   Ptr_Comp:       *  98008 same as above
   Discr:       O.K.  0
   Enum_Comp:   O.K.  1
   Int_Comp:    O.K.  18
   Str_Comp:    O.K.  DHRYSTONE PROGRAM, SOME STRING
 Int_1_Loc:     O.K.  5
 Int_2_Loc:     O.K.  13
 Int_3_Loc:     O.K.  7
 Enum_Loc:      O.K.  1
 Str_1_Loc:     O.K.  DHRYSTONE PROGRAM, 1'ST STRING
 Str_2_Loc:     O.K.  DHRYSTONE PROGRAM, 2'ND STRING

 Register option      Selected.

 Microseconds 1 loop:          4.53
 Dhrystones / second:      220690
 VAX MIPS rating:            125.61
```

　　其中最核心的数据便是Dhrystones / second的数值。  

#### 2.5 跑分榜
　　Roy Longbottom的网站收集记录了很多款处理器的Dhrystone跑分结果，可移步他的网站链接查看 [http://www.roylongbottom.org.uk/dhrystone%20results.htm#anchorAndroid](http://www.roylongbottom.org.uk/dhrystone%20results.htm#anchorAndroid)  

　　至此，微处理器CPU性能测试基准Dhrystone痞子衡便介绍完毕了，掌声在哪里~~~
