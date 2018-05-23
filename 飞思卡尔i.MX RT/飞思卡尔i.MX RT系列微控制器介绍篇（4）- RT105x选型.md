----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的RT105x选型**。  

　　大家都知道i.MX RT105x是i.MX RT系列第一款产品，在提这款产品特性的时候，我们往往说的是i.MXRT1052DVL6B的特性，这也是RT105x系列主推的一款核心芯片，目前一些第三方硬件公司（比如ZLG、野火、正点原子、安富莱）做的RT105x开发板基本上都是基于这款芯片，但其实i.MXRT105x共包含8款芯片，8款芯片之间其实是有差异的，这也涉及到具体芯片选型，今天痞子衡就给大家介绍这些差异。  

　　打开恩智浦官网，进入 [i.MXRT105x购买页面](https://www.nxp.com/cn/products/processors-and-microcontrollers/applications-processors/i.mx-applications-processors/i.mx-rt-series/i.mx-rt1050-crossover-processor-with-arm-cortex-m7-core:i.MX-RT1050?tab=Buy_Parametric_Tab) 可以看到有以下全8款芯片在售，从Part Number命名来看主要有3类差异：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Intr_105x_parts.PNG" style="zoom:100%" />  

### 三类差异
#### 第一类差异：i.MXRT105<font color="Blue">1</font>xVLxx与i.MXRT105<font color="Blue">2</font>xVLxx
　　第一类差异可以通过查看Data Sheet找到，这类差异主要是区分芯片内部IP模块支持情况，进入Data Sheet下载页面：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Intr_105x_datasheets.PNG" style="zoom:100%" />  

　　打开其中任意一个文档，搜索Table 1. Ordering information可看到如下表，从表中可以看到1052子系列比1051子系列多了显示模块IP的支持（LCD/CSI/PXP）。  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Intr_105x_ordering_info.PNG" style="zoom:100%" />  

#### 第二类差异：i.MXRT105x<font color="Blue">CVL5</font>x与i.MXRT105x<font color="Blue">DVL6</font>x
　　第二类差异其实在芯片购买页面直接就可以看到，这类差异是用于区分芯片级别，含CVL5芯片是工业级别，而含DVL6芯片是消费级别，消费级别与工业级别差异主要在以下2点：  

<table><tbody>
    <tr>
        <th style="width: 120px;">芯片</th>
        <th style="width: 120px;">级别</th>
        <th style="width: 120px;">运行频率(MHz)</th>
        <th style="width: 120px;">温度范围(<sup>o</sup>C)</th>
    </tr>
    <tr>
        <td>i.MXRT105x<font color="Blue">CVL5</font>x</td>
        <td>工业级</td>
        <td>528</td>
        <td>-40 ~ +105</td>
    </tr>
    <tr>
        <td>i.MXRT105x<font color="Blue">DVL6</font>x</td>
        <td>消费级</td>
        <td>600</td>
        <td>0 ~ +95</td>
    </tr>
</table>

　　RT105x系列标准最高频率是528MHz，由于芯片生产工艺优良，可以超频到600MHz。工业级芯片要求工作在更宽广的温度范围下，并且稳定工作是第一诉求，所以超频对工业级芯片不适用。但是高性能是消费级芯片第一诉求，偶尔的不稳定对于消费级芯片也可以接受，因为超频适用于消费级芯片。  

#### 第三类差异：i.MXRT105xxVLx<font color="Blue">A</font>与i.MXRT105xxVLx<font color="Blue">B</font>
　　这三类差异必须查看Errata才能找到，这类差异主要是标示芯片不同Tapeout版本Bug情况，进入Errata下载页面：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Intr_105x_errata.PNG" style="zoom:100%" />  

　　打开Errata个文档，搜索Table 2. Summary of Silicon Errata可看到如下表，下面仅截取部分示例：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Intr_105x_errata_table2_index.PNG" style="zoom:100%" />  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Intr_105x_errata_table2_dcdc.PNG" style="zoom:100%" />  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Intr_105x_errata_table2_semc.PNG" style="zoom:100%" />  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Intr_105x_errata_table2_boot.PNG" style="zoom:100%" />  

　　其中DCDC电源问题是比较严重的问题，会导致芯片上电异常，这也是恩智浦在推出i.MXRT105xxVLx<font color="Blue">A</font>（也称A0版本）之后很快又推出了i.MXRT105xxVLx<font color="Blue">B</font>（也称A1版本）的原因，并且已将A0版本芯片全部下架。除了电源问题外，BootROM里也存在一些Bug，痞子衡后续介绍的Boot系列文章均针对的是A1版本。  

　　了解了以上三类差异，你在选型RT105x时便可找到想要的合适芯片。  

　　至此，飞思卡尔i.MX RT系列MCU的RT105x选型痞子衡便介绍完毕了，掌声在哪里~~~
