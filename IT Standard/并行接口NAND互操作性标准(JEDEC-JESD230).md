----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**JESD230标准**。  

　　众所周知，最早也最流行的Raw NAND接口标准是ONFI标准，痞子衡在 [并行接口NAND标准(ONFI)及SLC Raw NAND简介](https://www.cnblogs.com/henjay724/p/9152535.html) 里详细介绍过ONFI 1.0，很多知名NAND厂商（Hynix、Intel、Micron、Phison、Sony、ST）都是ONFI标准的制定者，但发展到高速DDR NAND的时候出现了一个与ONFI分庭抗礼的标准Toggle DDR，这个Toggle DDR标准是由Samsung和Toshiba两家共同推出的，这两家NAND的产能占据全球70%，所以Toggle DDR标准不容忽视。眼看着这两大NAND标准要打起来，微电子行业的标准权威JEDEC组织出来打圆场了，JEDEC尝试制定了一套Raw NAND互操作标准，即本文的主角JESD230标准，该标准的目的主要是为了融合ONFI和Toggle两大标准。痞子衡今天就为大家介绍一下JESD230标准。  

### 一、JESD230标准由来
　　在讲JESD230标准之前有必要先简要介绍一下ONFI以及Toggle DDR标准，ONFI组织成立于2006年，该组织于2006年12月发布了ONFI 1.0标准（仅针对50MB/s低速异步模式NAND），ONFI标准从此便成为了Raw NAND事实上的标准，ONFI标准一直指导着Raw NAND接口技术的发展，到了2012年9月，ONFI标准已经发展到了3.1（可支持400MB/s（NV-DDR2）高速同步模式NAND）。再说到Samsung和Toshiba联盟，这两家在2007年签署了NAND技术互换授权协议，并于2010年6月正式推出Toggle DDR 1.0接口标准的NAND产品，在这样的背景下JEDEC组织站出来与ONFI组织合作并于2012年10月推出JESD230标准第一个版本，从此JESD230标准便与ONFI、Toggle DDR标准同步发展，其对应关系可见下表：  

<table><tbody>
    <tr>
        <th style="width: 120px;">时间</th>
        <th style="width: 200px;">JEDEC标准</th>
        <th style="width: 200px;">对应ONFI标准</th>
        <th style="width: 200px;">对应Toggle DDR标准</th>
    </tr>
    <tr>
        <td>2012.10</td>
        <td>JESD230</td>
        <td>ONFI 3.1</td>
        <td>Toggle DDR 2.0</td>
    </tr>
    <tr>
        <td>2013.08</td>
        <td>JESD230A</td>
        <td rowspan="2">ONFI 3.2</td>
        <td></td>
    </tr>
    <tr>
        <td>2014.07</td>
        <td>JESD230B</td>
        <td></td>
    </tr>
    <tr>
        <td>2016.11</td>
        <td>JESD230C</td>
        <td>ONFI 4.0</td>
        <td></td>
    </tr>
</table>

　　现在说回JESD230标准，其全名是NAND Flash Interface Interoperability，从全称可见JESD230是NAND接口的一种补充协议，我们再来看一下JESD230官方Scope：  

```text
This document defines a standard NAND flash device interface interoperability standard that provides means for a system to be designed that can support Asynchronous SDR, Synchronous DDR and Toggle DDR NAND flash devices that are interoperable between JEDEC and ONFI member implementations. This standard was jointly developed by JEDEC and the Open NAND Flash Interface Workgroup (ONFI).
```

　　从Scope可以看出，JESD230主要是对Asynchronous SDR, Synchronous DDR and Toggle DDR NAND设备互操作性方面进行了规范。  

### 二、JESD230标准概要
　　让我们从分析JESD230与ONFI区别的角度来概要了解JESD230，就以最新的JESD230C与ONFI 4.0对比着来分析吧，ONFI 4.0手册共315页，而JESD230C仅有60页，可见JESD230标准补充的内容相比ONFI原规范内容是少很多的。下面我们仅从软件驱动设计的角度（命令集、参数表）来看两者区别：  
　　首先从命令集角度来看两者区别，下图是JESD230C与ONFI 4.0命令集对比，根据对比我们可以发现，两者命令基本是兼容的，只是JESD230多了Toggle Mode下的一些Multi-plane相关命令的第二种实现。  

<img src="http://henjay724.com/image/cnblogs/jesd230_cmd_vs_onfi.PNG" style="zoom:100%" />

　　再从参数表角度来看两者区别，下图是JESD230C与ONFI 4.0参数表对比（仅截取部分），ONFI参数表是256bytes，而JESD230参数表是512bytes，关于具体byte定义两者有很多相似之处，其中对于AC特性尤其是速度等级定义，两者是一致，这是互操作性的保证。  

<img src="http://henjay724.com/image/cnblogs/jesd230_para_table_vs_onfi.PNG" style="zoom:100%" />

　　至此，JESD230标准痞子衡便介绍完毕了，掌声在哪里~~~ 

**参考资料**：

[1]. [关于SSD的二三事，NAND闪存的一些常识](http://www.expreview.com/17963.html) 
