----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**ARM内核架构历史**。  

　　众所周知，ARM公司是一家微处理器行业的知名企业，ARM公司本身并不靠自有的设计来制造或出售CPU，而是将处理器架构授权给有兴趣的厂家。这些厂家基本涵盖了全球领先的知名半导体企业、软件和OEM厂商：TI, NXP, ST, Infineon, ADI, Cypress；Atollic，IAR system，FreeRTOS，SEGGER等。  

### 1.ARM内核体系架构历史

　　ARM是Advanced RISC Machines的缩写。ARM架构是一个32位精简指令集（RISC）处理器架构，其广泛地使用在许多嵌入式系统设计。

　　迄今为止（2016年）ARM架构已经发展到了第八代ARMv8，在了解最新架构之前有必要重温一下ARM架构发展史：

　　1985年，ARMv1架构诞生，该版架构只在原型机ARM1出现过，只有26位的寻址空间（64MB），没有用于商业产品。

　　1986年，ARMv2架构诞生，首颗量产的ARM处理器ARM2就是基于该架构，包含了对32位乘法指令和协处理器指令的支持，但同样仍为26位寻址空间。其后还出现了变种ARMv2a，ARM3即采用了ARMv2a，是第一片采用片上Cache的ARM处理器。

　　1990年，ARMv3架构诞生，第一个采用ARMv3架构的微处理器是ARM6(610)以及ARM7，其具有片上高速缓存、MMU和写缓冲，寻址空间增大到32位（4GB）。

　　1993年，ARMv4架构诞生，这个架构被广泛使用，ARM7(7TDMI)、ARM8、ARM9(9TDMI)和StrongARM采用了该架构。ARM在这个系列中引入了T变种指令集，即处理器可工作在Thumb状态，增加了16位Thumb指令集。

![arm-v5-v8](http://henjay724.com/image/cnblogs/ARM_V5_to_V8_Architecture.jpg)

　　　　　　　　　　　　图1. ARM V5 to V8 Architecture

　　1998年，ARMv5架构诞生，ARM7(EJ)、ARM9(E)、ARM10(E)和Xscale采用了该架构，这版架构改进了ARM/Thumb状态之间的切换效率。此外还引入了DSP指令和支持JAVA。

　　2001年，ARMv6架构诞生，ARM11采用的是该架构，这版架构强化了图形处理性能。通过追加有效进行多媒体处理的SIMD将语音及图像的处理功能大大提高。此外ARM在这个系列中引入了混合16位/32位的Thumb-2指令集。

![arm7-alternatives](http://henjay724.com/image/cnblogs/PD_ARM7_CPU_Alternatives.jpg)

　　　　　　　　　　　　图2. PD ARM7 CPU Alternativess

　　2004年，ARMv7架构诞生，从这个时候开始ARM以Cortex来重新命名处理器，Cortex-M3/4/7，Cortex-R4/5/6/7，Cortex-A8/9/5/7/15/17都是基于该架构。该架构包括NEON™技术扩展，可将DSP和媒体处理吞吐量提升高达400%，并提供改进的浮点支持以满足下一代3D图形和游戏以及传统嵌入式控制应用的需要。

　　2007年，在ARMv6基础上衍生了ARMv6-M架构，该架构是专门为低成本、高性能设备而设计，向以前由8位设备占主导地位的市场提供32位功能强大的解决方案。Cortex-M0/1/0+即采用的该架构。

　　2011年，ARMv8架构诞生，Cortex-A32/35/53/57/72/73采用的是该架构，这是ARM公司的首款支持64位指令集的处理器架构。

　　2015年，在ARMv6-M基础上衍生了ARMv8-M baseline，在ARMv7-M基础上衍生了ARMv8-M mainline，Cortex-M23采用的是ARMv8-M baseline架构，Cortex-M33采用的是ARMv8-M mainline。这两款处理器加入了TrustZone支持，面向IoT物联网市场。

![arm7-alternatives](http://henjay724.com/image/cnblogs/ARM_processor_overview_2017.PNG)

　　　　　　　　　　　　图3. Performance and scalability for a diverse range of applications

**参考资料**：

[1]. [ARM架构和ARM芯片](http://blog.csdn.net/flowingflying/article/details/7209226) 

[2]. [ARM体系版本](http://www.cnblogs.com/samuelwnb/p/4287711.html)
  
[3]. [ARM版本及系列](http://blog.csdn.net/ggbondg/article/details/3901858)  

[4]. [ARM内核全解析](http://www.myir-tech.com/resource/448.asp)  