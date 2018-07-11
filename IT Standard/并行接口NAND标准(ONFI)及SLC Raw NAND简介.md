----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**ONFI标准及SLC Raw NAND**。  

　　NAND Flash是嵌入式世界里常见的存储器，对于嵌入式开发而言，NAND主要分为两大类：Serial NAND、Raw NAND，这两类NAND的差异是很大的（软件驱动开发角度而言），即使你掌握其中一种，也不代表你能了解另一种。  
　　Raw NAND是相对于Serial NAND而言的，Serial NAND即串行接口的NAND Flash，而Raw NAND是并行接口的NAND FLASH，早期并行接口通信数据率是明显高于串行通信数据率的，但随着串行通信速度越来越快，并行接口速度优势显得不那么重要了，反而因信号线太多导致设计成本较高（PCB走线复杂）显得有点不合潮流。但其实这么说对Raw NAND是不公平的，现在的Serial NOR/NAND信号线其实也不少，比如高速的串行HyperFlash信号线数量已经直逼x8 bit的Raw NAND FLASH，所以Raw NAND市场还是坚挺的，你会发现各大存储厂商都还在不断推出Raw NAND FLASH产品。  

### 一、ONFI标准由来
　　说到Raw NAND发展史，其实早期的Raw NAND没有统一标准，虽然早在1989年Toshiba便发表了NAND Flash结构，但具体到Raw NAND芯片，各厂商都是自由设计，因此尺寸不统一、存储结构差异大、接口命令不通用等问题导致客户使用起来很难受。为了改变这一现状，2006年几个主流的Raw NAND厂商（Hynix、Intel、Micron、Phison、Sony、ST）联合起来商量制订一个Raw NAND标准，这个标准叫Open NAND Flash Interface，简称ONFI，2006年12月ONFI 1.0标准正式推出，此标准一经推出大受欢迎（好像不欢迎也不行，那些大厂说了算啊），此后<font color="Blue">几乎所有的Raw NAND厂商都按照ONFI标准设计生产Raw NAND，从此Raw NAND世界清静了，不管哪家生产的Raw NAND对嵌入式设计者来说几乎都是一样的，至少在驱动代码层面是一样的，那么各厂商竞争优势在哪呢？主要在三个方面：数据存取速率、ECC能力、ONFI之外的个性化功能。</font>  
　　你可以从 [ONFI官网](http://www.onfi.org/) 下载ONFI标准手册，从2006年推出1.0标准至今，ONFI标准已经发展到4.1，这也说明了Raw NAND技术在不断更新升级。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_specs1.PNG" style="zoom:100%" />

### 二、SLC Raw NAND原理
#### 2.1 Raw NAND分类
　　从软件驱动开发角度而言，Raw NAND可以从以下几个方面进一步细分：  

> 单元层数(bit/cell)：SLC（1bit/cell） / MLC（2bit/cell） / TLC（3bit/cell） / QLC（4bit/cell）
> 数据线宽度：x8 / x16
> 信号线模式：Asynchronous / Synchronous
> 数据采集模式：SDR / DDR
> 接口命令标准：非标 / ONFI

　　本文的主要研究对象是兼容ONFI 1.0标准的Asynchronous SDR SLC NAND Flash。  

#### 2.2 Raw NAND内存模型
　　ONFI规定了Raw NAND内存单元从大到小最多分为如下5层：Device、LUN(Die)、Plane、Block、Page（如下图所示），其中Page和Block是必有的，因为<font color="Blue">Page是读写的最小单元，Block是擦除的最小单元</font>。而LUN和Plane则不是必有的（如没有，可认为LUN=1, Plane=1），一般在大容量Raw NAND（至少8Gb以上）上才会出现。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_mem_model.PNG" style="zoom:100%" />

　　根据以上5层分级的内存模型，Raw NAND地址也很自然地由如下图中多个部分组成:  

> <font color="Blue">Raw NAND Address = LUN Addr + Block Addr + Page Addr + Byte Addr (Column Addr)</font>

　　可能有朋友对Plane Address bit的位置有疑问，其实结合上面内存模型图就不难理解了，每个Plane里包含的Block并不是连续的，而是与其他Plane含有的Block是交错的。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_mem_addressing.PNG" style="zoom:100%" />

#### 2.3 Raw NAND信号与封装
　　ONFI规定了Raw NAND信号线与封装，如下是典型的x8 Raw NAND内部结构图，除了内存单元外，还有两大组成，分别是IO控制单元和逻辑控制单元，信号线主要挂在IO控制与逻辑单元，x8 Raw NAND主要有15根信号线（其中必须的是13根，WP#和R/B#可以不用），关于各信号线具体作用，请查阅相关文档。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_block_diagram1.PNG" style="zoom:100%" />

　　ONFI规定的封装标准有很多，比如TSOP48、LGA52、BGA63/100/132/152/272/316，其中对于嵌入式开发而言，最常用的是如下图扁平封装的TSOP-48，这种封装常用于容量较小的Raw NAND（1/2/4/8/16/32Gb），1-32Gb容量对于嵌入式设计而言差不多够用，且TSOP-48封装易于PCB设计，因此得以流行。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_tsop48.PNG" style="zoom:100%" />

#### 2.4 Raw NAND接口命令
　　ONFI 1.0规定了Raw NAND接口命令，如下表所示，其中一部分是必须要支持的（M），还有一部分是可选支持的（O）。必须支持的命令里最常用的是Read(Read Page)、Page Program、Block Erase这三条，涵盖读写擦最基本的三种操作。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_cmd_set.PNG" style="zoom:100%" />

　　除了读写擦这三个最基本命令外，还有一个必有命令也非常常用，这个命令是Read Status，用于获取命令执行状态与结果，ONFI规定Raw NAND内部必须包含一个8bit的状态寄存器，这个状态寄存器用来存储NAND命令执行状态与结果，其中有两个bit（RDY-SR[6]和FAIL-SR[0]）需要特别关注，RDY用于指示命令执行状态（这个bit与外部R/B#信号线功能是一致的），FAIL用于返回命令执行结果（主要是有无ECC错误）。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_status_register.PNG" style="zoom:100%" />

　　此外，还有一个必有命令不得不提，这个命令是Read Parameter Page，用于获取芯片内部存储的出厂信息（包括内存结构、特性、时序、其他行为参数等），这个Parameter Page大小为256Bytes，其结构已由ONFI规定如下表，痞子衡已经圈出了一些重要信息，在设计NAND软件驱动时，可以通过获取这个Parameter Page来做到代码通用。  


<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_parameter_page000.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_parameter_page001.PNG" style="zoom:100%" />

#### 2.5 Raw NAND数据速率
　　前面讲了，数据存取速率这个技术指标是各厂商竞争力的体现，对于这个指标，其实ONFI标准定义了一部分，我们知道Raw NAND数据存取操作是以Page为单位的，Page操作时间决定了数据存取速率，Page操作时间由3部分组成：  

> Page操作时间（t<sub>ReadPage</sub>） = Page命令操作时间（t<sub>Cmd</sub>） + Page命令执行等待时间（t<sub>Busy</sub>） + Page数据操作时间（t<sub>Data</sub>）

　　以上三部分时间里，<font color="Blue">ONFI定义了Page命令/数据操作时间标准，但Page命令执行等待时间无法强制，因此各厂商NAND速度差异主要是这个Page命令执行等待时间不同造成的</font>。  
　　以异步模式Read Page命令（0x00 - 0x30）为例讲解，下图是Read Page完整时序简图，0x00是主机发送的第一个字节，用于通知NAND Device主机想要读取Page，随后的5个字节发送的是地址数据，用于通知NAND Device主机想要从什么地址获取数据，0x30是主机发送的最后一个字节，用于通知NAND Device读取Page命令发送已经完成，至此命令操作周期已经结束，NAND Device此时开始进行内部处理流程：拉低外部引脚R/B#或将内部寄存器SR[6]置0表明我正在忙，然后从内存块里将主机指定地址所在的Page数据全部拷贝到临时缓存区（Page Buffer），对这一整个Page数据进行ECC校验，如Page数据校验通过，拉高外部引脚R/B#或将内部寄存器SR[6]置1表明我已经准备好了，至此命令执行等待周期已经结束，主机开始按Byte依次将Page数据读出来，所有Page数据全部都被读出来后，整个Read Page时序就结束了。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_read_timing.PNG" style="zoom:100%" />

　　下图是命令/地址操作具体时序，根据时序图我们可以粗略计算出t<sub>Cmd</sub>：  

> t<sub>Cmd</sub> = (cmdBytes + addrBytes) x (t<sub>WP</sub> + t<sub>WH</sub>) = 7 * (t<sub>WP</sub> + t<sub>WH</sub>)

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_command_latch_cycle1.PNG" style="zoom:100%" />

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_address_latch_cycle1.PNG" style="zoom:100%" />

　　下图是数据读取操作具体时序，分为两种：<font color="Blue">Non-EDO模式（RE#上沿采样数据）和EDO模式（RE#下沿采样数据），从图中我们知道t<sub>RC</sub>是RE#信号的一个周期，通俗地说，Non-EDO模式一般用于低速模式（即t<sub>RC</sub> > 30ns时），而EDO模式一般用于高速模式（即t<sub>RC</sub> < 30ns时）</font>。根据时序图我们可以粗略计算出t<sub>Data</sub>：  

> t<sub>Data</sub> = dataBytesInOnePage * t<sub>RC</sub>

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_data_output_cycle.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_data_output_cycle_edo.PNG" style="zoom:100%" />

　　让我们把t<sub>Cmd</sub>和t<sub>Data</sub>代入t<sub>ReadPage</sub>计算公式可得如下等式，我们知道其中t<sub>Busy</sub>是无法得知的，那么其他三个时间t<sub>WP</sub>、t<sub>WH</sub>、t<sub>RC</sub>到底是多少呢？  

> t<sub>ReadPage</sub> = 7 x (t<sub>WP</sub> + t<sub>WH</sub>) + t<sub>Busy</sub> + dataBytesInOnePage * t<sub>RC</sub>

　　继续查看ONFI手册可以找到答案，ONFI规定了六种timing mode（0-5），timing mode table里指明了所有时序相关的参数数值范围，当然也包括t<sub>WP</sub>、t<sub>WH</sub>、t<sub>RC</sub>，以最快的timing mode 5来计算：  

> t<sub>ReadPage</sub> = 7 x 20ns + t<sub>Busy</sub> + dataBytesInOnePage * 20ns = (dataBytesInOnePage + 7) x 20ns + t<sub>Busy</sub>  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_timing_mode_table.PNG" style="zoom:100%" />

　　我们似乎离答案更近一步了，但t<sub>Busy</sub>是多少这个问题始终困扰着我们，其实痞子衡带你绕了路，想要知道Read Page的时间没有这么复杂，我们可以从任何一款Raw NAND数据手册的扉页Features里直接找到答案，如下是Micron生产的型号为MT29F4GxxABBxA的部分feature：  

```text
• Open NAND Flash Interface (ONFI) 1.0-compliant
• Single-level cell (SLC) technology
• Organization
  – Page size x8: 2112 bytes (2048 + 64 bytes)
  – Page size x16: 1056 words (1024 + 32 words)
  – Block size: 64 pages (128K + 4K bytes)
  – Plane size: 2 planes x 2048 blocks per plane
  – Device size: 4Gb: 4096 blocks; 8Gb: 8192 blocks; 16Gb: 16,384 blocks
• Asynchronous I/O performance
  – tRC/tWC: 20ns (3.3V), 25ns (1.8V)
• Array performance
  – Read page: 25μs
  – Program page: 200μs (TYP: 1.8V, 3.3V)
  – Erase block: 700μs (TYP)
• Operating voltage range
  – VCC: 2.7–3.6V
  – VCC: 1.7–1.95V
```

　　从feature里我们可以知道t<sub>ReadPage</sub>最小为25us（此数值应是在x16 bits，timing mode 5下得出的最快速度），那么可以反算出t<sub>Busy</sub> = 25us - 20ns \* （1024 + 7） = 4.38us，知道了t<sub>Busy</sub>让我们计算一下x8 bits下的t<sub>ReadPage</sub> = 20ns \* （2048 + 7） + 4.38us = 45.48us，再计算x8 bits下的读取数据率 2048Bytes / 45.48us = 360.246Mbps，这个数据率对于普通嵌入式应用来说其实是够快的。  

#### 2.6 Raw NAND坏块与ECC
　　Raw NAND开发绕不开坏块（Bad Block）问题，这是NAND Flash区别于NOR Flash的一个重要特点。NAND技术上允许坏块的存在，这降低了NAND生产工艺要求，因此NAND单位容量价格比NOR低。  
　　既然物理上的坏块无法避免，那有什么方法可以改善/解决坏块问题呢？方法当然是有的，这个方法就是ECC（Error Correcting Code），ECC的具体实现原理详见痞子衡的另一篇文章 [汉明码校验(Hamming Code SEC-DED)](https://www.cnblogs.com/henjay724/p/8456821.html)，在这里你只需要知道ECC是一种错误检测与纠正算法，它通过对一定量的数据块（一般是256/512bytes）进行计算得到ECC码（一般8bytes），在Page Program时将原始Page数据与ECC码一同存入NAND Flash，在Read Page时同时获取Page数据与ECC码再进行一次计算，如果该Page数据没有ECC错误或者bit错误能够被ECC码纠正，那么Page读写操作就能够正常进行，如果bit错误个数太多不能够被纠正，那么该Page所在的块就应该被认定为一个坏块。  
　　ECC能力主要根据纠正单数据块中错误bit个数来区分的，最基本的ECC只能够纠正1bit错误，强一点的ECC可以纠正4或者8个甚至更多的错误bit。  
　　让我们用一款实际芯片来具体分析坏块与ECC，依旧以前面分析过的Micron生产的型号为MT29F4G08ABBxA为例，下图是其内存结构图，从图中我们可以知道这款NAND的Page大小为2KB，但如果你仔细看，你会发现每个Page还额外含有64Bytes数据，这个64Bytes区域即所谓的Spare Area，这个区域到底是干嘛用的呢？  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_mt29f4g08_arch.PNG" style="zoom:100%" />

　　下图是Spare Area的mapping图，由于每个Page是2KB，而ECC计算块是512Bytes，因此Page区域被均分为4块，分别是Main 0、1、2、3，每块大小为512Bytes，而相应的Spare Area也被均分为4块，分别是Spare 0、1、2、3，每块大小为16bytes，与Main区域一一对应。每个Spare x由2bytes坏块信息、8bytes ECC码、6bytes用户数据组成。要特别说一下的是ECC区域，当芯片硬件ECC功能开启时，8bytes ECC码区域会被自动用来存储ECC信息，而如果芯片没有硬件ECC功能，这个区域可以用来手动地存放软件ECC值。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_mt29f4g08_spare_area.PNG" style="zoom:100%" />

　　下图是芯片Error管理相关信息，也包含了ECC，从图中我们可以知道这款芯片ECC是4bits，坏块是用0x00来标识的，并且承诺该芯片出厂时每个Die里所含有的4096个block最多只会有80个坏块。这些信息除了在芯片手册里可以找到之外，前面介绍过的ONFI Parameter Page也同样记录了。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_mt29f4g08_ecc_detail0.PNG" style="zoom:100%" />
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_mt29f4g08_ecc_detail1.PNG" style="zoom:100%" />

#### 2.7 Raw NAND个性化功能
　　Raw NAND还有一些个性化的功能，这个是因厂商而异的，ONFI规定了两个可选的命令Get/Set Feature，个性化功能可以通过Get/Set Feature命令来扩展。下表是ONFI 1.0规定的Feature address范围，其中0x01是Timing Mode，0x80-0xFF用于各厂商实现自己的特色功能。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_feature_address.PNG" style="zoom:100%" />

　　关于Timing Mode地址的具体定义如下，<font color="Blue">ONFI规定芯片上电初始为Timing mode 0，即最低速的模式，如果我们想要更快的NAND访问速度，必须使用Set feature命令将Timing mode设置到想要的数值</font>。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_feature_address_01h_timing_mode.PNG" style="zoom:100%" />

　　继续以前面分析过的Micron生产的型号为MT29F4G08ABBxA为例，下图是该芯片的Feature定义，除了ONFI规定之外，还定义了自己的特色部分（0x80, 0x81, 0x90）。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_mt29f4g08_feature_address.PNG" style="zoom:100%" />

　　比如0x90定义的Array operation mode，我们可以通过其实现OTP控制以及硬件ECC的开关。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/onfi_mt29f4g08_feature_address_90h_array_mode.PNG" style="zoom:100%" />

### 三、SLC Raw NAND产品
　　Raw NAND厂商产品有两种，一种是裸Raw NAND芯片，另一种是含Raw NAND的存储方案（比如SSD硬盘），对于嵌入式开发而言，我们更关心的是裸Raw NAND芯片产品，下面痞子衡收集了可以售卖SLC Raw NAND芯片的厂商及产品系列：  

<table><tbody>
    <tr>
        <th>厂商</th>
        <th>芯片系列</th>
        <th>官方网址</th>
    </tr>
    <tr>
        <td>Micron</td>
        <td>MT29F</td>
		<td><a href="https://www.micron.com">https://www.micron.com</a><br>
		    <a href="https://www.micron.com/products/nand-flash/slc-nand/slc-nand-part-catalog#/">slc-nand-part-catalog</a></td>
    </tr>
    <tr>
        <td>Numonyx</td>
        <td>NAND256, NAND512</td>
		<td><a href="https://www.micron.com">https://www.micron.com</a><br>
		    <a href="https://www.micron.com/products/nand-flash/slc-nand/slc-nand-part-catalog#/">slc-nand-part-catalog</a></td>
    </tr>
    <tr>
        <td>Macronix</td>
        <td>MX30LF, MX60LF</td>
		<td><a href="http://www.macronix.com">http://www.macronix.com</a><br>
		    <a href="http://www.macronix.com/en-us/products/NAND-Flash/SLC-NAND-Flash/Pages/default.aspx">slc-nand-part-catalog</a></td>
    </tr>
    <tr>
        <td>Winbond</td>
        <td>W29N</td>
		<td><a href="http://www.winbond.com.tw">http://www.winbond.com.tw</a><br>
		    <a href="http://www.winbond.com.tw/hq/product/code-storage-flash-memory/slc-nand-flash/?__locale=zh">slc-nand-part-catalog</a></td>
    </tr>
    <tr>
        <td>Spansion</td>
        <td>S34ML, S34MS, S34SL</td>
		<td><a href="http://www.cypress.com/">http://www.cypress.com/</a><br>
		    <a href="http://www.cypress.com/search/psg/85771#/?_facetShow=fs_pdensity_gbit_,ss_pinterface,fs_poperating_voltage_v_,ss_ppackage,ss_ptemperature_range_min_c_to_max_c_,ss_ppart_family,fs_pmin_operating_voltage_v_,fs_pmax_operating_voltage_v_,fs_pmin_operating_temp_c_,fs_pmax_operating_temp_c_,ss_pautomotive_qualified,fs_part_price">slc-nand-part-catalog</a></td>
    </tr>
    <tr>
        <td>ISSI</td>
        <td>IS34ML, IS34MW</td>
		<td><a href="www.issi.com">www.issi.com</a><br>
		    <a href="http://www.issi.com/US/product-flash.shtml#jump8">slc-nand-part-catalog</a></td>
    </tr>
    <tr>
        <td>Toshiba</td>
        <td>TC58B, TC58N</td>
		<td><a href="http://toshiba.semicon-storage.com">http://toshiba.semicon-storage.com</a></td>
    </tr>
    <tr>
        <td>SK Hynix</td>
        <td>H27U</td>
		<td><a href="http://www.hynix.com">http://www.hynix.com</a><br>
		    <a href="http://www.hynix.com/eng/product/nandRaw.jsp#tg03">slc-nand-part-catalog</a></td>
    </tr>
    <tr>
        <td>Samsung</td>
        <td>K9F, K9K</td>
		<td><a href="http://www.samsung.com/semiconductor/">http://www.samsung.com/semiconductor/</a></td>
    </tr>
</table>

　　至此，ONFI标准及SLC Raw NAND痞子衡便介绍完毕了，掌声在哪里~~~ 

