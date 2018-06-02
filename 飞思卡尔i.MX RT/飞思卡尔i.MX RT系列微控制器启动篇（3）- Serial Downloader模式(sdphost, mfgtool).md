----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的Serial Downloader模式**。  

　　在上一篇文章 [飞思卡尔i.MX RT系列微控制器启动篇（2）- Boot配置(BOOT Pin, eFUSE)](http://www.cnblogs.com/henjay724/p/9034563.html) 里痞子衡为大家介绍了i.MXRT Boot的行为配置，其中第一节里讲了Boot有三种行为模式：Serial Downloader、Boot From Fuses、Internal Boot，后两种是核心的加载启动行为模式，而Serial Downloder看起来是个次要的模式，那么Serial Downloader模式到底有什么用？今天痞子衡就来详细聊一聊Serial Downloader模式。  

　　痞子衡在前面已经讲过Serial Downloader模式是一种串行下载模式，在这种模式下，BootROM通过指定的USB或者UART口来接收来自Host（恩智浦提供了上位机工具sdphost.exe或者mfgtool）的Flashloader数据，并将数据存储在SRAM中执行，Flashloader程序可以用来将你的Application下载进i.MXRT支持的所有外部非易失性存储器中，为后续从外部存储器启动做准备。  

### 一、进入Serial Downloader模式
　　i.MXRT上电永远是从ROM启动去执行BootROM程序，最顶层的Boot行为模式由BOOT_MODE[1:0] pins的状态决定，想进入Serial Downloader模式最直接的方式便是将BOOT_MODE[1:0]输入状态拨成2'b01，在设计i.MXRT的硬件板时BOOT_MODE[1:0] pins应设计成可通过拨码开关选择输入电平，下图是RT1052硬件板的参考设计：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_BOOT_MODEx_sch.PNG" style="zoom:100%" />

　　拨码开关SW5应拨向SW_DIP-8的8和10，即设置BOOT_MODE[1:0]=2'b01，此时便直接进入了Serial Downloader模式。  
　　当然如果SW5不按上面这么设置，也有可能进入Serial Downloader模式，但是需要其他前提条件，即Boot From Fuses/Internal Boot模式下从Boot Device加载启动失败。  

### 二、sdphost/mfgtool的使用
　　进入了Serial Downloader模式，此时便可以用恩智浦提供的host工具与BootROM进行命令交互，host工具在 [Flashloader包](https://www.nxp.com/products/processors-and-microcontrollers/applications-processors/i.mx-applications-processors/i.mx-rt-series/i.mx-rt1050-crossover-processor-with-arm-cortex-m7-core:i.MX-RT1050?tab=Design_Tools_Tab) 里。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_FlashloaderDownloadPage.PNG" style="zoom:100%" />

　　下载好Flashloader包之后，在\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools下可以找到所有host工具，其中用于与BootROM通信的有两个：sdphost.exe与MfgTool2.exe。  

#### 2.1 支持的通信外设pinout
　　BootROM支持两种通信外设，分别是USB-HID和UART，pinout如下（Pinout适用RT105x和RT102x）：  

<table><tbody>
    <tr>
        <th style="width: 100px;">Peripheral</th>
        <th style="width: 100px;">Instance</th>
        <th style="width: 150px;">PAD</th>
        <th style="width: 110px;">Port</th>
        <th style="width: 110px;">Mode</th>
    </tr>
    <tr>
        <td rowspan="3">USB</td>
        <td rowspan="3">OTG1</td>
        <td>USB_OTG1_DN</td>
        <td rowspan="3">/</td>
		<td rowspan="3">/</td>
    </tr>
    <tr>
        <td>USB_OTG1_DP</td>
    </tr>
    <tr>
        <td>USB_OTG1_VBUS</td>
    </tr>
    <tr>
        <td rowspan="2">LPUART</td>
        <td rowspan="2">1</td>
        <td>GPIO_AD_B0_12</td>
        <td>LPUART1_TX</td>
        <td>ALT2</td>
    </tr>
    <tr>
        <td>GPIO_AD_B0_13</td>
        <td>LPUART1_RX</td>
        <td>ALT2</td>
    </tr>
</table>

#### 2.2 sdphost用法
　　sdphost.exe是命令行工具，使用sdphost既可以通过UART口也可以通过USB口与BootROM进行通信与命令交互。  
　　在命令行下打开sdphost.exe，输入-?命令可以看到sdphost使用帮助，一共7条命令：  

```text
PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win> .\sdphost.exe -?
usage: C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win\sdphost.exe
                       [-?|--help]
                       [-p|--port <name>[,<speed>]]
                       [-u|--usb [[[<vid>,]<pid>]]]
                       [-t|--timeout <ms>]
                       -- command <args...>

Options:
  -?/--help                    Show this help
  -p/--port <name>[,<speed>]   Connect to target over UART. Specify COM port
                               and optionally baud rate
                                 (default=COM1,115200)
                                 If -b, then port is BusPal port
  -u/--usb [[[<vid>,]<pid>] | [<path>]]
                               Connect to target over USB HID device denoted by
                               vid/pid (default=0x15a2,0x0083) or device path
  -t/--timeout <ms>            Set packet timeout in milliseconds
                                 (default=5000)
Commands:
  // 读指定AIPS外设寄存器值
  read-register <addr> [<format> [<count> [<file>]]]
                               Read one or more registers at address.
                               Format must be 8, 16, or 32;
                                 default format is 32.
                               Count is number of bytes to read;
                                 default count is sizeof format
                                 (i.e. one register).
                               Output file is binary;
                                 default is hex display on stdout.
  // 写值进指定AIPS外设寄存器
  write-register <addr> <format> <data>
                               Write one register at address.
                               Format must be 8, 16, or 32.
                               Data is data value to write.
  // 写image文件数据进指定SRAM地址
  write-file <addr> <file> [<count>]
                               Write file at address.
                               Count is size of data to write in bytes;
                                 size of file will be used by default.
  // 返回上条命令的执行状态
  error-status                 Read error status of last command.
  // 写DCD table进指定SRAM地址
  dcd-write <addr> <file>      Send DCD table from file.
                                 <addr> must point to a valid
                                   temporary storage area.
  // 忽略image文件中的DCD table
  skip-dcd-header              Ignore DCD table in image.
  // 跳转执行含IVT头的image
  jump-address <addr>          Jump to entry point of image
                                   with IVT at specified address.
```

　　当使用串口转USB模块连接i.MXRT的LPUART1或者使用USB Cable连接上USB_OTG1口后可以看到PC设备管理器会识别出相关设备：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_sdp_device_manager.PNG" style="zoom:100%" />

　　让我们尝试一下使用sdphost与BootROM通信，先试一下USB通信:  
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win> <font style="font-weight:bold;" color="Blue">.\sdphost.exe -u 0x1fc9,0x0130 -- error-status</font>
> ```text
> Status (HAB mode) = 1450735702 (0x56787856) HAB disabled.
> Reponse Status = 858993459 (0x33333333) HAB failure.
> ```

　　再接着试一下UART通信，似乎通信失败了。需要注意的是，当使用USB通信过一次之后，BootROM已经激活USB外设，不会再去检测UART外设，如果想使用UART通信，需要将板子reset一次，使BootROM重回外设检测状态。  
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win> <font style="font-weight:bold;" color="Blue">.\sdphost.exe -p COM19 -- error-status</font>
> ```text
> getStatusResponse.readPacket error 5.
> Status (HAB mode) = 10004 (0x2714) No response from device.
> ```

　　关于sdphost其他命令具体如何组合使用，痞子衡会在后续的文章里再具体介绍。  

#### 2.3 mfgtool用法
　　MfgTool2.exe是GUI工具，其实际上是在sdphost工具基础上做了一层图形化封装，但功能上有一些削减，并且使用MfgTool2.exe仅能通过USB口与BootROM进行通信。  
　　如果板子不连USB Cable，直接打开MfgTool2.exe，可看到如下界面，显示"No Device Connected"。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_mfgtool_no_connect.PNG" style="zoom:100%" />

　　当板子连上USB Cable后可以看到状态变为"HID-compliant vendor-defined device"，这表明软件已可以正常使用。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_mfgtool_hid_connected.PNG" style="zoom:100%" />

　　从软件界面来看，似乎能控制的只有2个按钮：Start和Exit，那到底如何使用这个软件，其实秘密藏在如下两个配置文件里：  
　　\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\mfgtools-rel\cfg.ini  
　　\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\mfgtools-rel\Profiles\MXRT105X\OS Firmware\ucl2.xml  
　　cfg.ini文件用于指定GUI工具工作平台与模式（默认为chip = MXRT105X和name = MXRT105X-DevBoot），而ucl2.xml文件包含了不同模式的具体实现。以MXRT105x-DevBoot模式来说，点击Start按钮后，GUI会使用sdphost.exe与BootROM建立通信，首先用write-file命令将ivt_flashloader.bin文件下载进SRAM（地址是固定的0x20000000），然后使用jump-address命令跳转到Flashloader程序中去执行（Stage 1），GUI继续使用blhost.exe与Flashloader建立通信，先用get-property 1命令测试连接，然后使用receive-sb-file命令接收boot_image.sb文件。  

```text
<LIST name="MXRT105x-DevBoot" desc="Manufacturing with Flashloader">
<!-- Stage 1, load and execute Flashloader -->    
    <CMD state="BootStrap" type="boot" body="BootStrap" file="ivt_flashloader.bin" > Loading Flashloader. </CMD>
    <CMD state="BootStrap" type="jump"  onError = "ignore"> Jumping to Flashloader. </CMD>

<!-- Stage 2, Program boot image into external memory using Flashloader -->   
    <CMD state="Blhost" type="blhost" body="get-property 1" > Get Property 1. </CMD> <!--Used to test if flashloader runs successfully-->
    <CMD state="Blhost" type="blhost" timeout="15000" body="receive-sb-file \"Profiles\\MXRT105X\\OS Firmware\\boot_image.sb\"" > Program Boot image </CMD> 
    <CMD state="Blhost" type="blhost" body="Update Completed!">Done</CMD>
</LIST>
```

　　关于blhost与sb文件的用法，痞子衡会在后续的文章里再具体介绍。  

### 三、启动Flashloader程序
　　启动Flashloader是Serial Downloader模式的核心任务，其实在上一节mfgtool工具介绍里，大家已经看见了Flashloader的身影，但是GUI工具将Flashloader的启动操作封装得不太直观，这里痞子衡用sdphost为大家直观地再现一遍启动Flashloader的过程。  
　　在\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Flashloader目录下，有Flashloader的可执行文件（.elf格式和.srec格式），但是sdphost仅能处理纯image数据文件（.bin格式），可使用IAR EWARM自带的ielftool.exe工具将elf文件转换成bin文件（命令为: ielftool --bin flashloader.elf flashloader.bin）。  
　　得到flashloader.bin文件后，便可使用sdphost的write-file命令将flashloader.bin下载进SRAM，并且使用jump-address命令跳转到Flashloader程序中，但还需要解决两个问题：  
　　第一个问题，该把flashloader.bin文件下载进SRAM的什么地址？其实这里的SRAM地址即是Flashloader程序的中断向量表所在地址，这个地址包含在elf文件里，可用专门的elf文件分析工具得到。也可以通过查看.srec或者.hex文件得到，由于包里自带了.srec文件，我们直接用文本编辑器打开.srec文件，可以得知这个地址是0x20002000。  

```text
S0130000666C6173686C6F616465722E737265638C
S31520002000705A2120914B01207B240020952E0120FF
```

　　让我们开始将flashloader.bin下载进SRAM的0x20002000地址处：  
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win> <font style="font-weight:bold;" color="Blue">.\sdphost.exe -u 0x1fc9,0x0130 -- write-file 0x20002000 ..\\..\\..\Flashloader\flashloader.bin</font>
> ```text
> Preparing to send 81847 (0x13fb7) bytes to the target.
> (1/1)1%Status (HAB mode) = 1450735702 (0x56787856) HAB disabled.
> Reponse Status = 2290649224 (0x88888888) Write File complete.
> ```

　　第二个问题，如何使用jump-address命令跳转到Flashloader程序中？痞子衡在前面sdphost命令列表里介绍过，jump-address只能跳转到含IVT头的image，现在image本身已经有了，但是IVT头是什么？用二进制编辑器打开\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\mfgtools-rel\Profiles\MXRT105X\OS Firmware\ivt_flashloader.bin文件，可以发现除了前2KB之外的其他数据跟我们生成的flashloader.bin是一样的，那么IVT就藏在前2KB数据里，仔细看一下，你会发现除了偏移0x400的位置处有一些数据外，其余都是0，是的IVT就在那里。  

```text
offset(h)
00000000: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00000010: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
...
000003F0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00000400: D1 00 20 40 91 4B 01 20 00 00 00 00 00 00 00 00
00000410: 20 04 00 20 00 04 00 20 00 00 00 00 00 00 00 00
00000420: 00 00 00 20 B7 5F 01 00 00 00 00 00 00 00 00 00
00000430: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
...
00001FF0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00002000: 70 5A 21 20 91 4B 01 20 7B 24 00 20 95 2E 01 20
00002010: 7B 24 00 20 7B 24 00 20 7B 24 00 20 00 00 00 00
...
00015FB0: 03 80 FF 71 00 70 08 -- -- -- -- -- -- -- -- --
```

　　你可能会疑问，真正有用的IVT数据只有几十个字节，为什么ivt_flashloader.bin文件会留出2KB的空间来放IVT？这得分析IVT本身才能知道答案，让我们来尝试解析IVT，IVT的原型是如下的hab_ivt_v0结构体，一共32个byte，对应的是偏移0x400 - 0x41F处的数据，hab_ivt_v0.self = 0x20000400，由于此处self成员指定的IVT地址是0x20000400，而Flashloader本身地址是0x20002000，bin文件本身并不含地址信息数据，所以只能用0来填充占位以保证IVT数据与Flashloader数据的相对位置关系，这就是0x430 - 0x1FFF全是0的原因。hab_ivt_v0.boot_data =0x20000420，指明了boot data数据在0x420 - 0x42b处，boot_data结构体原型如下，一共12个byte，boot_data.start = 0x20000000，指明boot data应从0x20000000处开始存放，所谓boot data即IVT和image的统称，看到这，你应该明白了0x0 - 0x3FF处为何全是0的原因了吧。  

```C
#define HAB_TAG_IVT0 0xd1     /**< Image Vector Table V0 */

/** @ref hab_header structure */
typedef struct hab_hdr {
    uint8_t tag;              /**< Tag field */
    uint8_t len[2];           /**< Length field in bytes (big-endian) */
    uint8_t par;              /**< Parameters field */
} hab_hdr_t;

/** @ref ivt structure */
struct hab_ivt_v0 {
    /** @ref hdr with tag #HAB_TAG_IVT0, length and HAB version fields */
    hab_hdr_t hdr;
    /** Absolute address of the first instruction to execute from the image */
    uint32_t entry;
    /** Reserved in this version of HAB: should be NULL. */
    uint32_t reserved1;
    /** Absolute address of the image DCD: may be NULL. */
    uint32_t dcd;
    /** Absolute address of the Boot Data: may be NULL, but not interpreted any further by HAB */
    uint32_t boot_data;
    /** Absolute address of the IVT.*/
    uint32_t self;
    /** Absolute address of the image CSF.*/
    uint32_t csf;
    /** Reserved in this version of HAB: should be zero. */
    uint32_t reserved2;
};

/** @ref boot_data structure */
typedef struct boot_data{
    uint32_t start;           /* Start address of the image */
    uint32_t size;            /* Size of the image */
    uint32_t plugin;          /* Plugin flag */
} BOOT_DATA_T;
```

　　既然ivt_flashloader.bin文件里的前2KB有很多冗余数据，那不妨我们只把有效数据（IVT和boot data，0x400 - 0x42d处共44个byte）提取出来放到ivt_bootdata.bin文件里，让我们将ivt_bootdata.bin下载进SRAM的0x20000400地址处：  
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win> <font style="font-weight:bold;" color="Blue">.\sdphost.exe -u 0x1fc9,0x0130 -- write-file 0x20000400 ..\\..\\..\Flashloader\ivt_bootdata.bin</font>
> ```text
> Preparing to send 44 (0x2c) bytes to the target.
> (1/1)100% Completed!
> Status (HAB mode) = 1450735702 (0x56787856) HAB disabled.
> Reponse Status = 2290649224 (0x88888888) Write File complete.
> ```

　　到这里IVT和image均已经下载进SRAM了，可以跳转去执行Flashloader程序了，使用jump-address命令：  
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\sdphost\win> <font style="font-weight:bold;" color="Blue">.\sdphost.exe -u 0x1fc9,0x0130 -- jump-address 0x20000400</font>
> ```text
> Status (HAB mode) = 1450735702 (0x56787856) HAB disabled.
> ```

　　至此，Flashloader就算启动完成了，jump-address命令执行完成之后，你会发现USB设备被重新枚举了，此时新枚举的USB-HID设备是Flashloader里的通信外设。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/i.MXRT_Boot_sdp_flashloader_usb.PNG" style="zoom:100%" />

　　如果你试着用blhost与Flashloader通信得到如下结果，恭喜你，Flashloader已被成功启动了。  
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win> <font style="font-weight:bold;" color="Blue">.\blhost.exe -u 0x15a2,0x0073 -- get-property 1</font>
> ```text
> Inject command 'get-property'
> Response status = 0 (0x0) Success.
> Response word 1 = 1258422528 (0x4b020100)
> Current Version = K2.1.0
> ```

　　至此，飞思卡尔i.MX RT系列MCU的Serial Downloader模式痞子衡便介绍完毕了，掌声在哪里~~~ 

