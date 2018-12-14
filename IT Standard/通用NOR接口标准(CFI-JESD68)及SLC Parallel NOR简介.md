----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**CFI标准及SLC Parallel NOR**。  

　　NOR Flash是嵌入式世界里最常见的存储器，常常内嵌在微控制器里(Parallel型)或外置作为内存扩展(Serial型)，作为代码存储器。对于嵌入式开发而言，NOR主要分为两大类：Serial NOR、Parallel NOR，最早出现的NOR是Parallel NOR，后来为了简化引脚数，逐渐发展出了Serial NOR，目前的格局是Serial NOR主要占据低容量NOR市场（128Mb以下），而Parallel NOR主要占据高容量NOR市场（128Mb以上），虽然这两类NOR的差异比较大（软件驱动开发角度而言），但是它们有一个共性，那就是均兼容CFI接口标准，这给软件驱动开发带来了便利。  

### 一、CFI-JESD68标准由来
　　关于NOR Flash发展史，我们知道早在1988年Intel便发表了NOR Flash结构，从此改变了由EPROM/EEPROM一统天下的局面。早期的NOR产品主要是Parallel NOR，作为半导体行业领导大厂，为了使NOR产品发展标准化，Intel于1996年制定了CFI 1.0标准（1.0版本属于draft版本，正式版本是1997年发布的1.1），CFI标准的存在是为了让Host能够从NOR Flash device中直接获取制造商ID、设备ID、Flash大小以及各种物理特性，从而使得NOR Flash产品在前后兼容性上表现更好，软件驱动设计也更加标准化。  
　　鉴于CFI已慢慢发展成为NOR Flash事实上的接口标准，JEDEC组织于1999年9月正式将CFI标准命名为JESD68，下表是CFI-JESD68标准制定的时间关系：  

<table><tbody>
    <tr>
        <th style="width: 120px;">时间</th>
        <th style="width: 200px;">NOR标准</th>
        <th style="width: 200px;">制定者</th>
    </tr>
    <tr>
        <td>1996.07</td>
        <td>CFI 1.0</td>
        <td>Intel</td>
    </tr>
    <tr>
        <td>1997.05</td>
        <td>CFI 1.1</td>
        <td>Intel</td>
    </tr>
    <tr>
        <td>1999.09</td>
        <td>JESD68</td>
        <td>JEDEC</td>
    </tr>
    <tr>
        <td>2001.12</td>
        <td>CFI 2.0</td>
        <td>AMD</td>
    </tr>
    <tr>
        <td>2003.09</td>
        <td>JESD68.01</td>
        <td>JEDEC</td>
    </tr>
</table>

> Note: CFI-JESD68标准原则上是为Parallel NOR制定的，但是部分Serial NOR也支持这一标准。

### 二、SLC Parallel NOR原理
#### 2.1 Parallel NOR分类
　　从软件驱动开发角度而言，Parallel NOR可以从以下几个方面进一步细分：  

> 单元层数(bit/cell)：SLC（1bit/cell） / MLC（2bit/cell）
> 地址模式： ADP / ADM / AADM
> 数据线宽度：x8 / x16
> 信号线模式：Asynchronous / Synchronous
> 数据采集模式：SDR / DDR
> 接口标准：CFI

　　本文的主要研究对象是兼容CFI 2.0 (JESD68.01)标准的Asynchronous SDR SLC Parallel NOR Flash。  

#### 2.2 Parallel NOR内存模型
　　NOR内存单元从大到小一般分为如下5层：Device、Block、Sector、Page、Byte，其中Byte、Page和Sector是必有的，因为<font color="Blue">Byte是读取的最小单元（即可以任意地址随机访问），Page是编程的最小单元，Sector是擦除的最小单元</font>，而Block则不是必有的（如没有，可认为Block=1）。  

> Note: 关于Block有一点需要特别说明，即NOR的RWW（Read While Write）特性，我们知道NOR是可以存储代码直接原地执行XIP的，如果在某个Block里执行代码（即CPU从NOR中读取指令数据）的同时去擦除或编程这块Block会发生什么情况呢？有些NOR是支持这种RWW操作的，但也有的NOR不支持RWW（此时会产生hardfault/lockup），Block的存在是为了规避RWW问题，RWW问题的作用范围是Block，如果某Block中执行代码擦除/编程的是另一块Block，则完全不用担忧RWW问题。  

#### 2.3 Parallel NOR信号与封装
　　CFI手册里并没有明确规定Parallel NOR信号线与封装，但业界有默认的标准，从信号线角度来说NOR和SRAM基本是一样的，如下是典型的Parallel NOR内部结构图，除了内存单元外，还有三大组成，分别是控制单元、地址译码单元和输出缓冲单元，信号线主要挂在这三大组成上，关于各信号线具体作用，请查阅相关文档。  

<img src="http://henjay724.com/image/cnblogs/cfi_block_diagram.PNG" style="zoom:100%" />

　　NOR芯片根据Flash size大小不同封装也不尽相同，以经典的128Mb容量的ADP NOR芯片为例，其封装一般有三种TSOP-56, TFBGA-56, LFBGA-64，下图是TSOP-56封装信号分布：  

<img src="http://henjay724.com/image/cnblogs/cfi_tsop56.PNG" style="zoom:100%" />

#### 2.4 Parallel NOR接口命令
　　CFI手册里也没有明确规定Parallel NOR接口命令，同样地业界还是有默认的标准，如下是从Micron MT28EW系列手册里截取的部分基本命令，包括Reset、Read、Read CFI、Program(Word program)、Buffer Program、Block(Sector) Erase，涵盖读写擦最基本的三种操作，这些基本命令在主流厂商的NOR产品里都被支持。此外，NOR还支持更多高级命令（Blank Check、security/protect相关，lower power相关等），那些命令因Device而异，不是本文讨论重点。  

<img src="http://henjay724.com/image/cnblogs/cfi_cmd_set.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_cmd_set2.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_cmd_set3.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_cmd_set4.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_cmd_set5.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_cmd_set6.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_cmd_set7.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_cmd_set8.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_cmd_set9.PNG" style="zoom:100%" />

　　除了读写擦这三个最基本命令外，还有一个Data Polling机制也非常常用，这个机制用于获取命令（主要是Program/Erase）执行状态与结果，当Program/Erase命令发给NOR device之后，NOR device是通过DQ[7:0]引脚来返回命令执行状态的，各bit意义如下图所示，简单来说DQ3（Erase timer bit）表明操作的开始，DQ5（Error bit）表明操作过程中是否发生硬件错误，DQ6（Toggle bit）表明操作是否结束。  

<img src="http://henjay724.com/image/cnblogs/cfi_data_polling_register_bit.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_data_polling_register_bit2.PNG" style="zoom:100%" />

　　下图为Toggle bit检测的流程图，这个检测流程是最常用的，主要用于确认Program/Erase操作是否成功地执行了。  

<img src="http://henjay724.com/image/cnblogs/cfi_toggle_bit_flowchart.PNG" style="zoom:100%" />

　　此外，还有一个必有命令不得不提，这个命令是Read CFI，用于获取芯片内部存储的出厂信息（包括内存结构、特性、其他行为参数等），这个query table最多由五部分组成，其结构已由CFI规定如下表，痞子衡已经圈出了一些重要信息，在设计NOR软件驱动时，可以通过获取这个query table来做到代码通用。  

<img src="http://henjay724.com/image/cnblogs/cfi_query_table.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_query_table2.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_query_table3.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_query_table4.PNG" style="zoom:100%" />

#### 2.5 Parallel NOR数据速率
　　数据存取速率是个重要的技术指标，对于这个指标，CFI手册里并没有定义，所以需要根据NOR Flash手册里的AC characteristics表来确定。以异步模式Read命令为例讲解（以Micron生产的型号为MT28EW128ABA为例），下面是Read的两种模式Random Read和Page Read完整时序简图：  

<img src="http://henjay724.com/image/cnblogs/cfi_random_read_ac_timing_x16.PNG" style="zoom:100%" />
<img src="http://henjay724.com/image/cnblogs/cfi_page_read_ac_timing_x16.PNG" style="zoom:100%" />

　　从上述时序图，我们可以知道Page Read平均数据率肯定是比Random Read要高的，那么这两种数据率分别能达到多少呢？可继续参看如下时序参数表，t<sub>RC</sub>最小为70ns，那么Random Read数据率最大为2Bytes/70ns = 228.571Mbps，而t<sub>PAGE</sub>最大为20ns，那么Page Read的数据率最小为2Bytes / 20ns = 800Mbps。（注：均以x16 Device为例）    
<img src="http://henjay724.com/image/cnblogs/cfi_read_ac_characteristics.PNG" style="zoom:100%" />

　　如果想快捷地了解NOR Flash的性能，最简单的就是打开NOR Flash手册，看首页的feature介绍，如下是MT28EW128ABA的简要feature：  

```text
• Single-level cell (SLC) process technology
• Density: 128Mb
• Supply voltage
  – VCC = 2.7–3.6V (program, erase, read)
  – VCCQ = 1.65 - VCC (I/O buffers)
• Asynchronous random/page read
  – Page size: 16 words or 32 bytes
  – Page access: 20ns
  – Random access: 70ns (VCC = VCCQ = 2.7-3.6V)
  – Random access: 75ns (VCCQ = 1.65-VCC)
• Buffer program (512-word program buffer)
  – 2.0 MB/s (TYP) when using full buffer program
  – 2.5 MB/s (TYP) when using accelerated buffer program (VHH)
• Word/Byte program: 25us per word (TYP)
• Block erase (128KB): 0.2s (TYP)
• Memory organization
  – Uniform blocks: 128KB or 64KW each
  – x8/x16 data bus
• CFI (Common Flash Interface) support
```

### 三、SLC Parallel NOR产品
　　最后痞子衡收集了可以售卖SLC Parallel NOR芯片的厂商及产品系列：  

<table><tbody>
    <tr>
        <th>厂商</th>
        <th>芯片系列</th>
        <th>官方网址</th>
    </tr>
    <tr>
        <td>Micron</td>
        <td>MT28EW, MT28FW</td>
		<td><a href="https://www.micron.com">https://www.micron.com</a><br>
		    <a href="https://www.micron.com/products/nor-flash/parallel-nor-flash/parallel-nor-flash-part-catalog#/">parallel-nor-part-catalog</a></td>
    </tr>
    <tr>
        <td>Macronix</td>
        <td>MX68GL<br>
		    MX29LA, MX29GL, MX29GA, MX29LV<br>
		    MX29VS, MX29NS, MX29SL, MX29F</td>
		<td><a href="http://www.macronix.com">http://www.macronix.com</a><br>
		    <a href="http://www.macronix.com/en-us/products/NOR-Flash/Parallel-NOR-Flash/Pages/default.aspx#!tabs=1-8V32Mb">parallel-nor-part-catalog</a></td>
    </tr>
    <tr>
        <td>Winbond</td>
        <td>W29GL</td>
		<td><a href="http://www.winbond.com.tw">http://www.winbond.com.tw</a><br>
		    <a href="http://www.winbond.com.tw/hq/product/code-storage-flash-memory/parallel-nor-flash/?__locale=zh&selected=256Mbit#categoryArea">parallel-nor-part-catalog</a></td>
    </tr>
    <tr>
        <td>Spansion</td>
        <td>S29GL, S29AL, S29AS, S29PL</td>
		<td><a href="http://www.cypress.com/">http://www.cypress.com/</a><br>
		    <a href="http://www.cypress.com/products/parallel-nor-flash-memory">parallel-nor-part-catalog</a></td>
    </tr>
    <tr>
        <td>ISSI</td>
        <td>IS29GL</td>
		<td><a href="www.issi.com">www.issi.com</a><br>
		    <a href="http://www.issi.com/US/product-flash.shtml#jump8">parallel-nor-part-catalog</a></td>
    </tr>
    <tr>
        <td>Microchip</td>
        <td>SST38VF, SST39VF</td>
		<td><a href="https://www.microchip.com">https://www.microchip.com</a><br>
		    <a href="http://www.microchip.com/ParamChartSearch/chart.aspx?branchID=7130107">parallel-nor-part-catalog</a></td>
    </tr>
</table>

　　至此，CFI标准及SLC Parallel NOR痞子衡便介绍完毕了，掌声在哪里~~~ 



