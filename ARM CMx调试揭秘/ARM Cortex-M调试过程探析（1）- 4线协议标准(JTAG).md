----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式调试里的接口标准JTAG**。  

　　在结束[《ARM Cortex-M开发文件详解》](http://www.cnblogs.com/henjay724/p/8166334.html)系列文章之后，痞子衡休整了一小段时间，但是讲课的心完全停不下来啊，所以忍不住新开了一个系列文章，叫《ARM Cortex-M调试过程探析》，本文是这个系列文章的第一篇，欢迎各位嵌入式朋友前来围观捧场~~~  
　　嵌入式开发中，大家免不了需要仿真调试代码，尤其是当应用工程功能逻辑复杂到一定程度时，免不了在写代码时会引入一些逻辑bug，仅靠代码审查有时候并不一定能排除所有bug，所以在线调试便成为排除bug最有效直接的方式，今天我们要聊的是调试里最基础的东西，即接口标准。ARM内核原生支持2种业界通用的接口标准，分别是JTAG和SWD。本节课痞子衡先给大家详细讲讲JTAG接口。  

### 一、JTAG接口标准
　　JTAG全称“Joint Test Action Group”，既是个标准也是个组织，这是个由几家主要的电子制造商（IBM、AT&T、TI、Philips等）成立于1985年的组织，这个组织成立的目的是发起制订一种PCB和芯片测试标准。  
　　JTAG标准于1990年被IEEE批准为IEEE1149.1测试访问端口和边界扫描结构标准。JTAG标准规定了进行边界扫描所需要的硬件和软件，主要应用于电路的边界扫描测试和可编程芯片的在线系统编程。  

#### 1.1 IEEE 1149.1标准

> IEEE 1149.1工作组 http://grouper.ieee.org/groups/1149/1/  
> 最初版手册1149.1-1990 http://standards.ieee.org/findstds/standard/1149.1-1990.html  
> 最新版手册1149.1-2013 http://standards.ieee.org/findstds/standard/1149.1-2013.html  

#### 1.2 JTAG接口信号
　　JTAG接口，总称测试访问接口TAP（Test Access Port），使用如下信号来实现边界扫描操作：  
> * TCK（测试时钟）：同步内部状态机操作的时钟信号。
> * TMS（测试模式选择）：控制内部状态机转换的模式信号（TCK上升沿采样）。
> * TDI（测试数据输入）：移入器件测试或编程逻辑的数据（TCK上升沿采样）。
> * TDO（测试数据输出）：移出器件测试或编程逻辑的数据（TCK下降沿采样）。

　　除了以上信号线外，还有1个可选的信号：  
> * TRST（测试重置）：重置TAP控制器的状态机的复位信号。

#### 1.3 JTAG系统内部构造
　　JTAG系统内部最基本的单元是边界扫描单元（其扫描获取的值存在边界扫描寄存器BSR（Boundary Scan Register）中），每个边界扫描单元都位于目标器件的边界上，所以很多时候JTAG测试也被称为边界扫描。  
　　所有目标器件核心逻辑与针脚之间的信号都会被串联的边界扫描单元所拦截。正常运行时，这些边界扫描单元是不可见的。但是，在测试模式下这些单元可以被用来设置/读取目标器件针脚或核心逻辑的值。  

<table><tbody>
    <tr>
        <td><img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/jtag-registers-cn.jpg" style="zoom:35%" /></td>
        <td><img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/schematic_diagram_jtag_enab-cn.gif" style="zoom:45%" /></td>
    </tr>
</table> 

　　除了上述BSR之外，JTAG系统还需要以下3个寄存器：
> * 指令寄存器：存储当前的指令，指令内容被TAP控制器用来决定如何处理接收到的信号。
> * 旁路寄存器（BYPASS）：把信息从TDI传到TDO的单位寄存器。
> * 识别码寄存器（IDCODES）：含有器件的识别码和版本序号，该信息可以使器件和它的边界扫描描述语言（BSDL）文件相关联。

　　JTAG系统最核心的是TAP控制器，TAP控制器被设计用来与JTAG系统内部寄存器相互动，TAP控制器是一个被TMS信号控制转换的同步状态机，控制着JTAG系统的行为。  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/jtag-tap_state_machine-cn.gif" style="zoom:50%" />
　　如上图所示，TAP控制器的内部状态机一共16个状态，关于各个状态具体含义可查阅IEEE1149.1手册。TAP控制器的基本功能是产生BSR和指令寄存器正常工作所需要的时钟和控制信号，其主要功能有以下几点：  
> * 提供信号将指令装入指令寄存器。
> * 提供信号将输入数据从TDI管脚移入内部寄存器、把输出数据从内部寄存器移出到TDO管脚。
> * 执行相应功能，如捕获、移位和更新数据等。

　　指令寄存器是用来存储需要解释执行的指令的，IEEE 1149.1标准规定了JTAG兼容器件必须要具备的指令：  
> * BYPASS：用单一单元旁路寄存器传送数据，缩短JTAG链上不必要的扫描链路。
> * EXTEST：将已知值（存在BSR）驱动到芯片针脚上。
> * SAMPLE/PRELOAD：将捕获到的芯片针脚值装入BSR。

　　除了必备的指令外，IEEE 1149.1标准还规定了如下可选的指令：  
> * IDCODE：将IDCODES寄存器中的数据移出。
> * INTEST：将已知值（存在BSR）驱动到芯片核心逻辑上。
> * RUNBIST：当TAP进入测试运行空闲状态时，芯片进行自检。

#### 1.4 JTAG调试工具pinout
　　通常支持JTAG接口的调试编程工具其实只是利用了JTAG技术的四线TAP通信协议，而除了标准TAP信号线外，有时还加入其他辅助信号线构成完整pinout，对于ARM JTAG调试工具来说，有两种比较通用的pinout标准，即ARM20 JTAG header和ARM14 JTAG header：  

<table><tbody>
    <tr>
        <td><img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/jtag-arm20-pinout.jpg" style="zoom:80%" /></td>
        <td><img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/jtag-arm14-pinout.jpg" style="zoom:100%" /></td>
    </tr>
</table> 

　　上述两种ARM JTAG header中除了标准TAP信号线外，其他辅助信号线含义如下：  

<table><tbody>
    <tr>
        <th>信号名</th>
        <th>ARM20-pin</th>
        <th>ARM14-pin</th>
        <th>信号含义</th>
    </tr>
    <tr>
        <td>Vref</td>
        <td>P1</td>
        <td>P1，P13</td>
        <td>JTAG接口电平参考电压。用于检查目标系统是否供电，该引脚通常与目标系统Vdd相连，中间不允许串接电阻。</td>
    </tr>
    <tr>
        <td>Vsupply</td>
        <td>P2</td>
        <td>N/A</td>
        <td>电源输入</td>
    </tr>
    <tr>
        <td>nSRST</td>
        <td>P15</td>
        <td>P12</td>
        <td>System Reset信号，与目标系统复位信号相连。可以直接对目标系统复位，同时可以检测目标系统的复位情况。为了防止误触发，应在目标端加上适当的上拉电阻。</td>
    </tr>
    <tr>
        <td>RTCK</td>
        <td>P11</td>
        <td>N/A</td>
        <td>Return Test Clock。由目标系统反馈给JTAG的时钟信号，用来动态控制TCK速率。不使用时可以直接接地。</td>
    </tr>
    <tr>
        <td>GND</td>
        <td>P4，P6，P8，P10，P12，P14，P16，P18，P20</td>
        <td>P2，P4，P6，P8，P10，P14</td>
        <td>接地</td>
    </tr>
    <tr>
        <td>DBGRQ</td>
        <td>P17</td>
        <td>N/A</td>
        <td>连接到目标系统的调试请求信号</td>
    </tr>
    <tr>
        <td>DBGACK</td>
        <td>P19</td>
        <td>N/A</td>
        <td>由目标系统反馈回来的调试应答信号</td>
    </tr>
</table> 

> Note：更多JTAG pinout详见JTAG test网站的整理 http://www.jtagtest.com/pinouts/

### 二、JTAG接口进阶
　　前面讲完了JTAG基础知识，下面痞子衡再给大家多介绍一些JTAG相关的“黑科技”。  

#### 2.1 BSDL文件
　　现如今支持JTAG接口的芯片越来越多，为了统一各芯片厂商的具体JTAG实现，促进整个电子行业的一致性，IEEE1149.1标准制订了BSDL语言规范。BSDL是JTAG设备的标准建模语言，它的语法是VHDL的子集，是对JTAG器件的边界扫描特性的描述，主要用来沟通芯片厂商、用户与测试工具之间的联系。  

> 开源的JTAG BSDL库网站（http://bsdl.info/），涵盖主流厂商的主流芯片的BSDL文件  

　　痞子衡随便找一款芯片的BSDL文件（Freescale K60_1M（K24_144QFP））简单分析下：  
```nohighlight
entity k60_1m is
     generic (PHYSICAL_PIN_MAP : string := "K24_144qfp");
-- 此处描述芯片所有引脚属性
    port (
                                PTA0:        in   bit;
                                  ...
                              XTAL32:   linkage   bit);
    use STD_1149_1_2001.all;
    use STD_1149_6_2003.all;
-- 描述芯片引脚pinout
    attribute COMPONENT_CONFORMANCE of k60_1m: entity is "STD_1149_1_2001";
    attribute PIN_MAP of k60_1m: entity is PHYSICAL_PIN_MAP;
    constant K24_144qfp :PIN_MAP_STRING :=
                                "PTA0:      50," &
                                   ...
                              "XTAL32:      40" ;
-- 描述JTAG信号在芯片引脚分配
    attribute TAP_SCAN_OUT of PTA2 : signal is true;
    attribute TAP_SCAN_CLOCK of PTA0 : signal is (2.00e+07,BOTH);
    attribute TAP_SCAN_MODE of PTA3 : signal is true;
    attribute TAP_SCAN_IN of PTA1 : signal is true;
-- 描述JTAG指令寄存器相关属性
    attribute INSTRUCTION_LENGTH of k60_1m: entity is 4;
    attribute INSTRUCTION_OPCODE of k60_1m: entity is
        "BYPASS  (1111)," &
        "CLAMP  (1100)," &
        "EXTEST  (0100)," &
        "HIGHZ  (1001)," &
        "IDCODE  (0000)," &
        "PRELOAD  (0010)," &
        "SAMPLE  (0011)," &
        "ENABLE_CENSOR_CTRL  (0111)," &
        "ENABLE_TEST_CTRL  (0110)," &
        "EZPORT  (1101)," &
        "JTAGDP_ABORT  (1000)," &
        "JTAGDP_APACC  (1011)," &
        "JTAGDP_DPACC  (1010)," &
        "JTAGDP_IDCODE  (1110)";
    attribute INSTRUCTION_CAPTURE of k60_1m: entity is  "xx01";
    attribute INSTRUCTION_PRIVATE of k60_1m: entity is
        "ENABLE_CENSOR_CTRL," &
        "ENABLE_TEST_CTRL," &
        "EZPORT," &
        "JTAGDP_ABORT," &
        "JTAGDP_APACC," &
        "JTAGDP_DPACC," &
        "JTAGDP_IDCODE";
-- 描述JTAG识别码寄存器相关属性
    attribute IDCODE_REGISTER of k60_1m: entity is
        "0000"  & -- Version
        "1011001100011010"  & -- Part Number
        "00000001110"  & -- Manufacturer Identity
        "1";  -- IEEE 1149.1 Requirement
 
    attribute REGISTER_ACCESS of k60_1m: entity is
        "BYPASS (BYPASS)," &
        "DEVICE_ID (IDCODE)";
-- 描述JTAG边界扫描寄存器相关属性
    attribute BOUNDARY_LENGTH of k60_1m: entity is 196;
    attribute BOUNDARY_REGISTER of k60_1m: entity is
-- num cell   port/*                            function  safe  [ccell  dis  rslt] 
"   0  (BC_2, *,                                control,  1)                       ," &
"   1  (BC_8, PTE0,                             bidir,    X,    0,      1,   Z)    ," &
...
" 194  (BC_2, *,                                control,  1)                       ," &
" 195  (BC_8, PTD15,                            bidir,    X,    194,    1,   Z)    ";
end k60_1m;
```

#### 2.2 JTAG菊花链
　　当你的系统中有多个JTAG设备时，为解决JTAG口过多占用PCB的问题，JTAG支持如下菊花链方式连接（在FPGA应用尤其广泛）：  

<table><tbody>
    <tr>
        <td><img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/jtag-device-symbol.PNG" style="zoom:60%" /></td>
        <td><img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/jtag-device-chain.PNG" style="zoom:95%" /></td>
    </tr>
</table> 

　　从上图可以看出TMS、TCK是一主多从并联的结构（设备过多时TMS,TCK电路需加缓冲器（如74LVC245）增加驱动能力）；TDI、TDO是一主一从串联的结构，这种菊花链方式使得PCB上只需要一个JTAG接口便可以访问所有JTAG设备。  

　　至此，嵌入式调试里的接口标准JTAG痞子衡便介绍完毕了，掌声在哪里~~~ 

