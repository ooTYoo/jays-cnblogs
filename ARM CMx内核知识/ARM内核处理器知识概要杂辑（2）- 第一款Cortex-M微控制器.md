### 1.天生荣耀：ARM Cortex-M处理器由来

　　ARM公司自2004年推出ARMv7内核架构时，摒弃了以往"ARM+数字"这种处理器命名方法（ARM11之前的处理器统称经典处理器系列），重新启用Cortex来命名，并将Cortex系列细分为三大类：

> * Cortex-A系列：面向性能密集型系统的应用处理器内核
> * Cortex-R系列：面向实时应用的高性能内核
> * Cortex-M系列：面向各类嵌入式应用的微控制器内核

　　Cortex-M系列主要是用来取代经典处理器ARM7系列（比如基于ARMv4架构的ARM7TDMI），Cortex-M比ARM7的架构高了3代，性能也有较大提升，所以新的设计推荐使用Cortex-M，关于从ARM7到Cortex-M的移植详见ARM官网指导 [ARM7处理器系列](http://www.arm.com/zh/products/processors/classic/arm7/index.php)，想要对ARM内核版本演变有更多了解，可以看看我的另一篇博客 [ARM内核体系架构编年史(精简版)](http://www.cnblogs.com/henjay724/p/8408775.html)。

### 2.羽翼渐丰：ARM Cortex-M处理器家族

　　从2004年ARM公司推出第一款基于ARMv7M架构的Cortex-M3处理器开始，至今(2018)为止Cortex-M处理器家族已经发展到了Cortex-M33，下面是具体各处理器的发布时间及特点：

> * Cortex-M3：2004年10月发布，基于ARMv7M架构，面向标准嵌入式市场的高性能低成本的ARM处理器
> * Cortex-M1：2007年03月发布，基于ARMv6M架构，专门面向FPGA中设计实现的ARM处理器
> * Cortex-M0：2009年02月发布，基于ARMv6M架构，面积最小以及能耗极低的ARM处理器
> * Cortex-M4：2010年02月发布，基于ARMv7M架构，在M3基础上增加浮点、DSP功能以满足数字信号控制市场的ARM处理器
> * Cortex-M0+：2012年03月发布，基于ARMv6M架构，在M0基础上进一步降低功耗的ARM处理器
> * Cortex-M7：2014年09月发布，基于ARMv7M架构，在M4基础上进一步提升计算性能和DSP处理能力的ARM处理器，主要面向高端嵌入式市场
> * Cortex-M23：2016年11月发布，基于ARMv8M baseline架构，在M0/M0+基础上加入TrustZone安全特性支持的ARM处理器，满足IoT物联网安全要求。
> * Cortex-M33：2016年11月发布，基于ARMv8M mainline架构，在M3/M4基础上加入TrustZone安全特性支持的ARM处理器，满足IoT物联网安全要求。

　　关于ARM Cortex-M具体特点 详见官网介绍 [ARM Cortex-M内核系列介绍](http://www.arm.com/products/processors/cortex-m)

　　备注：每个Cortex-Mx处理器并非只有一个版本，以Cortex-M3为例，至今已有4个版本：r0p0、 r1p0、 r1p1、 r2p0，版本间有微小差异，详见 [ARM Cortex-M系列内核文档](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0552a/index.html)

### 3.初露锋芒：第一款Cortex-Mx微控制器产品

　　ARM公司提供了强大的Cortex-M处理器，接下来就到了各大半导体OEM厂商施展身手的时候了，谁都知道，抢占市场先机很重要，接下来让我们看看到底是谁分别抢先发布了ARM Cortex-Mx第一款微控制器：

> * 2006年03月，流明诺瑞Luminary Micro（09年被TI收购）率先推出了第一款基于ARM Cortex-M3处理器的Stellaris LM3S系列MMCU，但当时反响寥寥，直到2007年6月ST同样推出基于该内核的STM32 F1系列MCU才使之大放光芒。
> * 2009年03月，恩智浦半导体NXP率先推出了第一款基于ARM Cortex-M0处理器的LPC1100系列MCU。
> * 2010年08月，飞思卡尔半导体Freescale（15年被NXP并购）率先推出了第一款基于ARM Cortex-M4处理器的Kinetis K系列MCU。
> * 2012年11月，恩智浦半导体NXP继续率先推出了第一款基于ARM Cortex-M0+处理器的LPC800系列MCU。
> * 2014年09月，意法半导体ST率先推出了第一款基于ARM Cortex-M7处理器的STM32 F7系列MCU。

### 4.逐鹿中原：Cortex-Mx微控制器产品市场份额

　　有的时候，抢占了先机，但不一定能笑到最后，打江山容易守江山难。Cortex-M微控制器市场发展至今，天下大势，分分合合，各半导体厂商为了争夺市场份额，各显神通：
> * 意法半导体：主打通用市场份额、产品价格优势第一，旗下产品线STM32囊括Cortex-M家族全系列，对于竞争对手的合并动作不以为意。
> * 恩智浦半导体：主打汽车半导体市场、产品线广度第一，并购飞思卡尔后，旗下Kinetis产品线（Cortex-M0+/4/7）以及LPC产品线（Cortex-M0/0+/3/4）整合出最广产品线。
> * 赛普拉斯半导体：主打存储器领域市场、产品总数量第一，收购飞索半导体Spansion以及博通IoT部门后，造就最多产品数。

　　此外一些知名半导体厂商在Cortex-M产品市场份额争夺战中渐渐掉了队，比如收购Luminary的德州仪器TI，因为其DSP产品超强计算能力以及MSP430产品超低功耗优势，导致其对于Cortex-M产品推广未尽全力；还有8/16位 MCU时代霸主爱特梅尔Atmel（16年被Microchip收购），其8051及AVR产品的优势导致其在Cortex-M之战中失了先机。

　　Cortex-M处理器还在继续发展，32bit微控制器市场风云变幻，谁都不知道到底下一秒鹿死谁手。

**参考资料**：

[1]. [你知道哪家半导体拥有最多种基于Cortex-M 内核的MCU？](http://www.eetrend.com/news/100065615)

[2]. [忘掉MCU吧 解析Cortex-M4的时代](http://www.51hei.com/bbs/dpj-43776-1.html) 

[3]. [【揭秘】9年时间，ST如何把STM32出货从0做到16亿的？](http://www.eetrend.com/interview/100063378)

[4]. [恩智浦+飞思卡尔后你需要知道的十件事](http://www.eepw.com.cn/article/271240.htm)

[5]. [Cypress与Spansion都联姻了，你还在等什么？](http://www.csdn.net/article/2015-04-17/2824501)

[6]. [非ARM架构MCU四面楚歌，抢来的Atmel适合Microchip吗？](http://www.eefocus.com/mcu-dsp/357138)