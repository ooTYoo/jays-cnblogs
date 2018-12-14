----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**ARM Cortex-M功能模块**。  

　　ARM Cortex-M处理器家族发展至今(2016)，已有5代产品，分别是CM0/CM0+、CM1、CM3、CM4、CM7。

### 1.Cortex-M兼容特性

　　为了能做到Cortex-M软件重用，ARM公司在设计Cortex-M处理器时为其赋予了**处理器向下兼容**、**软件二进制向上兼容**特性。

　　首先看什么是**二进制兼容**，这个特性主要是针对软件而言，这里指的是当某软件(程序)依赖的头文件或库文件分别升级时，软件功能不受影响。要做到二进制兼容，被软件所依赖的头文件或库文件升级时必须是二进制兼容的。

　　那么什么又是**向上兼容**，向上兼容又叫向前兼容，指的是在较低版本处理器上编译的**软件**可以在较高版本处理器上执行。

　　跟向上兼容相对的另一个概念叫**向下兼容**，向下兼容又叫向后兼容，指的是较高版本**处理器**可以正确运行在较低版本处理器上编译的软件。

　　所以其实既可以用向上兼容，也可以用向下兼容来形容Cortex-M特性，只不过描述的主语不一样，我们可以说Cortex-M程序是向上兼容的，也可以说Cortex-M处理器是向下兼容的。

　　具体到Cortex-M处理器时，这个兼容特性表现为：

> * 从处理器角度看：CM0指令集和功能模块是最精简的，CM7指令集和功能模块是最丰富的。不存在低版本处理器上存在的特性是高版本处理器所没有的。
> * 从软件角度来看：CMSIS提供的头文件和功能函数是二进制向上兼容的，比如某CM0软件App使用的是core\_cm0.h头文件，而这个App要在CM7上运行时，不需要使用core\_cm7.h再重新编译一次（当然使用新头文件编译后的App也是正常的。）


### 2.Cortex-M功能模块差异

　　由于CM1主要是用在FPGA产品中，故下面对比忽略CM1。我们知道CM处理器是向下兼容的，故CM功能模块是随着版本的升级而逐步增加的，我们逐步从最低版本开始对比。

#### 2.1 CM0 vs CM0+

![cortex-m0](http://henjay724.com/image/cnblogs/Cortex-M0-chip-diagram-16.png)

　　先来聊聊CM0与CM0+，从最基准的CM0模块看起：

> * **ARMv6-M CPU内核**：ARM公司于2007年推出的内核。冯·诺依曼体系结构，3级流水线，支持大部分Thumb和小部分Thumb-2指令集，所有指令一共57条。此外还内嵌32-bit返回结果的硬件乘法器。
> * **NVIC嵌套向量中断控制器**：用于CPU在正常Run模式下中断管理。最大支持32个外部中断，外部中断可设4级抢占优先级（2bit）。
> * **WIC唤醒中断控制器**：用于CPU在低功耗Sleep模式下中断管理。
> * **AHB-Lite总线**：一条32bit AMBA-3标准的高性能system总线负责所有Flash、SRAM指令和数据存取。
> * **调试模块**：0-4个硬件断点Breakpoint，0-2个数据监测点Watchpoint。
> * **DAP调试接口**：通过DAP模块支持JTAG和SWD接口。

![cortex-m0+](http://henjay724.com/image/cnblogs/Cortex-M0-plus-chip-diagram-16.png)

　　那么CM0+到底改进了什么？

> * **ARMv6-M CPU内核**：流水线改为2级（很多8bit MCU都是2级流水线，主要用于降低功耗）
> * **NVIC嵌套向量中断控制器**：增加了VTOR即中断重定向功能。

　　那么CM0+到底增加了什么？

> * **MPU存储器保护单元**：提供硬件方式管理和保护内存，控制访问权限，最大可将内存分为8*8个region。内存越权访问，将返回MemManage Fault。
> * **MTB片上跟踪单元**：用户体验更好的的跟踪调试，优化的异常捕获机制，可以更快地定位bug。
> * **Fast I/O**：可单周期访问的快速I/O口，更易于Bit-banging（比如GPIO模拟SPI、IIC协议）。

#### 2.2 CM0+ vs CM3

![cortex-m3](http://henjay724.com/image/cnblogs/Cortex-M3-chip-diagram-16.png)

　　前面比较完了CM0与CM0+，再来看看CM3比CM0+增强在了哪里：

　　那么CM3到底改进了什么？

> * **ARMv7-M CPU内核**：ARM公司于2004年推出的内核。哈佛体系结构，3级流水线+分支预测，支持全部的Thumb和Thumb-2指令集。内嵌32-bit硬件乘法器可返回64-bit运算结果，且新增32-bit硬件除法器。
> * **NVIC嵌套向量中断控制器**：最大支持240个外部中断，中断优先级可分组（抢占优先级、响应优先级），8bit优先级设置（最大128级抢占优先级(对应最小2级响应优先级)，最大256级响应优先级(对应无抢占优先级)）。
> * **3x AHB-Lite总线**：除了原system总线负责SRAM存取外，还新增两条ICode、DCode总线分别完成Flash上指令和数据存取。
> * **调试模块**：0-8个硬件断点Breakpoint，0-4个数据监测点Watchpoint。
> * **ITM/ETM跟踪单元**：ITM更好地支持printf风格debug，ETM提供实时指令和数据跟踪。

　　那么CM3到底增加了什么？

　　额，CM3相比CM0+并没有增加什么独有模块，反倒是少了Fast I/O Port。

#### 2.3 CM3 vs CM4

![cortex-m4](http://henjay724.com/image/cnblogs/Cortex-M4-chip-diagram-16.png)

　　前面比较完了CM0+与CM3，再来看看CM4比CM3增强在了哪里：

　　那么CM4到底改进了什么？

> * **ARMv7E-M CPU内核**：增加了DSP相关指令支持。

　　那么CM4到底增加了什么？

> * **DSP数字信号处理单元**：新增支持单周期16/32-bit MAC、dual 16-bit MAC, 8/16-bit SIMD算法的数字信号处理单元。
> * **FPU浮点运算单元**：新增单精度（float型）兼容IEEE-754标准的浮点运算单元（VFPv4-SP）。

#### 2.4 CM4 vs CM7

![cortex-m7](http://henjay724.com/image/cnblogs/Cortex-M7-chip-diagram-16.png)

　　前面比较完了CM3与CM4，再来看看CM7比CM4增强在了哪里：

　　那么CM7到底改进了什么？

> * **ARMv7E-M CPU内核**：6级流水线+分支预测。
> * **2x AHB-Lite总线**：精简为2条AHB总线，其中AHB-P外设接口完成原来system总线功能, AHB-S从属接口负责外部总线控制器（如DMA）功能以及与TCM接口功能。
> * **MPU存储器保护单元**：最大可将内存分为16*8个region。
> * **FPU浮点运算单元**：新增双精度（double型）兼容IEEE-754标准的浮点运算单元（VFPv5）。

　　那么CM7到底增加了什么？

> * **I/D-Cache缓存区**：即是我们通常理解的L1 Cache，每个Cache大小为4-64KB。
> * **I/D-TCM紧密耦合存储器**：紧密的与处理器内核相耦合的RAM，提供与Cache相当的性能，但比Cache更具确定性，memory最大均为16MB。
> * **ECC特性**：对L1 Cache提供错误校正和恢复功能，提高系统的可靠性。
> * **AXI-M总线**：基于AMBA 4的64bit AXI总线，用于支持挂在系统上的L2 memory。

**参考资料**

[1]. [维基百科ARM Cortex-M](https://en.wikipedia.org/wiki/ARM_Cortex-M)

[2]. [Cortex系列M0-4简单对比](http://blog.sina.com.cn/s/blog_7dbd9c0e01018e4l.html)

[3]. [使用MTB模块快速跟踪定位Cortex-M0+指令执行状态](http://comm.chinaaet.com/adi/blogdetail/33468.html)

[4]. [Cortex-M0+单周期GPIO的使用方法](http://blog.chinaaet.com/jihceng0622/p/39731)

[5]. [ARM Cortex-M4和Cortex-M0+中断优先级及嵌套抢占问题](http://blog.chinaaet.com/jihceng0622/p/5100001238)

[6]. [STM32中断优先级概念](http://blog.csdn.net/kevinhg/article/details/40424403)

[7]. [基于ARM Cortex-M3核的SoC架构设计及性能分析](http://www.chinaaet.com/article/199546)

[8]. [ARM调试CoreSight、ETM、PTM、ITM、HTM、ETB等常用术语解析](http://www.myir-tech.com/resource/510.asp)


