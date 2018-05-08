----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**ARM Cortex-M性能指标**。  

### 1.处理器的性能指标

　　用于评价CPU的性能指标非常多，不同的性能侧重点下的测试标准可能得出的指标值不同，下面介绍嵌入式行业广泛使用的两个经典的测试标准。

#### 1.1 Dhrystone标准

　　Dhrystone是由Reinhold P. Weicker在1984年提出来的一个基准测试程序，其主要目的是测试处理器的整数运算和逻辑运算的性能。

　　Dhrystone程序最初用Ada语言发布，后来Rick Richardson为Unix开发了用C语言编写的Version 1.1，这个版本也成功的推动了Dhrystone的广泛应用。Dhrystone程序的最新版本是1988年更新的Version 2.1。

　　Dhrystone标准的测试方法很简单，就是单位时间内跑了多少次Dhrystone程序，其指标单位为**DMIPS/MHz**。MIPS是Million Instructions Per Second的缩写，每秒处理的百万级的机器语言指令数。DMIPS中的D是Dhrystone的缩写，它表示了在Dhrystone标准的测试方法下的MIPS。


#### 1.2 CoreMark标准

　　CoreMark是由[嵌入式微处理器基准评测协会EEMBC](http://www.eembc.org/)的Shay Gla-On于2009年提出的一项基准测试程序，其主要目标是测试处理器核心性能，这个标准被认为比陈旧的Dhrystone标准更有实际价值。

　　CoreMark程序使用C语言写成，包含如下的运算法则：列举（寻找并排序），数学矩阵操作（普通矩阵运算）和状态机（用来确定输入流中是否包含有效数字），最后还包括CRC（循环冗余校验）。CoreMark程序的最新版本是Version 1.0。

　　CoreMark标准的测试方法也很简单，就是在某配置参数组合下单位时间内跑了多少次CoreMark程序，其指标单位为**CoreMark/MHz**。CoreMark数字越高，意味着性能更高。
　
### 2. Cortex-M处理器的性能对比

　　ARM公司提供了Cortex-M系列处理器的官方性能对比柱状图：  

![性能指标柱状图](http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Cortex-M-performance-graph3.PNG)

　　关于各处理器具体指标数值如下：  
![性能指标具体值](http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Cortex-M-performance-table.PNG)

**参考资料**：

[1]. [[ARM嵌入式系统开发]第一章之Dhrystone](http://blog.csdn.net/masxcy/article/details/5357793)

[2]. [测试cpu的简单工具-dhrystone](http://blog.csdn.net/feixiaoxing/article/details/9005587)

[3]. [处理器性能测试基准程序(CoreMark)简介](http://www.360doc.com/content/13/0822/14/11948835_309094390.shtml)

[4]. [Cortex-M7 Launches:Embedded, IoT and Wearables](http://www.anandtech.com/show/8542/cortexm7-launches-embedded-iot-and-wearables/2)

[5]. [CSDN-markdown 表格样式设置（跨行表格，背景色等)](http://blog.csdn.net/thither_shore/article/details/52328313)