----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RTyyyy系列MCU的基本特性**。  

　　ARM Cortex-M微控制器芯片厂商向来竞争激烈，具体可参看我的另一篇文章[《第一款Cortex-M微控制器》](http://www.cnblogs.com/henjay724/p/8408825.html)，其中意法半导体凭借高性价比的STM32系列牢牢占据市场主要份额，但随着物联网IoT、边缘计算等市场对MCU性能要求越来越高。各芯片厂商也在不断推出性能越来越强的MCU，其中飞思卡尔半导体（现恩智浦半导体）于2017年开始推出的i.MX RT系列更是进一步拉高了MCU的上限，其第一款芯片i.MX RT1052，搭载Cortex-M7内核，主频高达600MHz，单片价格低到3$，直接引爆众多MCU开发者的神经，今天痞子衡就来介绍一下这个i.MX RT系列MCU究竟是何方神圣。  

### 一、源自i.MX 6ULL
　　了解飞思卡尔半导体的朋友应该听说过i.MX系列处理器，该系列处理器在车载娱乐、电子书等市场应用比较广泛，i.MX的发展经历了早期的i.MX28系列，到大获成功的i.MX 6系列，再到更高性能的i.MX 7/8系列。其中i.MX 6ULL是i.MX 6系列中一款比较低阶的处理器，让我们先来了解下这款MPU：  

<table><tbody>
    <tr>
        <th>Feature</th>
        <th>Detail</th>
    </tr>
    <tr>
        <th>Block<br>Diagram</th>
        <td><img src="http://henjay724.com/image/cnblogs/i.MX6ULL_PD.PNG" style="zoom:70%" /></td>
    </tr>
    <tr>
        <th>CPU</th>
        <td>ARM Cortex-A7, 900 MHz, 双32KB L1 cache, 128 KB L2 cache</td>
    </tr>
    <tr>
        <th>External<br>Memory</th>
        <td>16-bit LP-DDR2, DDR3/DDR3L<br>
            8/16-bit Parallel NOR FLASH / PSRAM<br>
            Dual-channel Quad-SPI NOR FLASH<br>
            8-bit Raw NAND FLASH with 40-bit ECC<br>
			eMMC4.5 / SD3.0</td>
    </tr>
    <tr>
        <th>Package</th>
        <td>BGA289, BGA272</td>
    </tr>
    <tr>
        <th>Price</th>
        <td>e络盟MCIMX6Y2CVM08AA - CNY60.88(1000+)</td>
    </tr>
</table>

　　i.MX 6ULL基本是一款为低功耗而设计的比较低阶的MPU。

### 二、跨界王i.MX RTyyyy
　　i.MX RTyyyy与i.MX 6ULL是同一个平台产物，因此在设计上沿用了6ULL大部分的模块，性能可以做到接近MPU的性能，研发成本也可以做到很低。  
　　RT1050是i.MX RTyyyy系列第一款产品，标准而又均衡的特性。  
　　RT1020是i.MX RTyyyy系列第二款产品，主要是为低成本PCB而设计，采用LQFP封装，与RT1050相比在主频、内存上稍有降低。  
　　RT1060是i.MX RTyyyy系列第三款产品，与RT1050相比主要是增大了SRAM容量，为一些复杂应用提供更多代码执行空间。

<table><tbody>
    <tr>
        <th>Feature</th>
        <th>i.MX RT1050</th>
        <th>i.MX RT1020</th>
        <th>i.MX RT1060</th>
    </tr>
    <tr>
        <th>Block<br>Diagram</th>
        <td><img src="http://henjay724.com/image/cnblogs/i.MXRT1050_PD.PNG" style="zoom:80%" /></td>
        <td><img src="http://henjay724.com/image/cnblogs/i.MXRT1020_PD.PNG" style="zoom:80%" /></td>
        <td><img src="http://henjay724.com/image/cnblogs/i.MXRT1060_PD.PNG" style="zoom:85%" /></td>
    </tr>
    <tr>
        <th>CPU</th>
        <td>ARM Cortex-M7, 600 MHz<br>双32KB L1 cache<br>3020 CoreMark/1284 DMIPS</td>
        <td>ARM Cortex-M7, 500 MHz<br>双16KB L1 cache<br>2517 CoreMark/1070 DMIPS</td>
        <td>Same as RT1050</td>
    </tr>
    <tr>
        <th>Internal<br>Memory</th>
        <td>512KB TCM SRAM</td>
        <td>256KB TCM SRAM</td>
        <td>1MB SRAM（最大512KB TCM）</td>
    </tr>
    <tr>
        <th>External<br>Memory</th>
        <td>Dual-channel Quad-SPI NOR/NAND FLASH<br>
		    8/16 bit SDRAM<br>
			8/16 bit Parallel NOR Flash / PSARM（异步）<br>
			8/16 bit Parallel NAND Flash（异步）<br>
			eMMC4.5 / SD3.0</td>
        <td>Same as RT1050</td>
        <td>Dual-channel Quad-SPI NOR/NAND FLASH<br>
		    8/16 bit SDRAM<br>
		    8/16 bit Parallel NOR Flash / PSARM（同步&异步）<br>
			8/16 bit Parallel NAND Flash（同步&异步）<br>
			eMMC4.5 / SD3.0</td>
    </tr>
    <tr>
        <th>IP<br>Change</th>
        <td>基准IP</td>
        <td>无多媒体相关IP(PXP, CSI, LCD)<br>
		    Timer IP数量均减半</td>
        <td>增加一个网口<br>
		    增加一个CANFD</td>
    </tr>
    <tr>
        <th>Package</th>
        <td>BGA196</td>
        <td>LQFP100, LQFP144</td>
        <td>Same as RT1050，Pin2Pin兼容</td>
    </tr>
    <tr>
        <th>Price</th>
        <td>官方定价USD2.98(10K+)<br>
		    立创商城MIMXRT1052DVL6A - CNY43.8（2018.03.13）</td>
        <td>官方定价USD2.18(10K+)</td>
        <td>官方定价USD3.48(10K+)</td>
    </tr>
    <tr>
        <th>Production</th>
        <td>Rev A0(MIMXRT1052xxxA) - 2017.10（有电源问题）<br>
		    Rev A1(MIMXRT1052xxxB) - 2018.04</td>
        <td>2018.04</td>
        <td>2018.10</td>
    </tr>
</table>

　　至此，飞思卡尔i.MX RTyyyy系列MCU的基本特性痞子衡便介绍完毕了，掌声在哪里~~~ 

