----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔i.MX RT系列MCU的Flashloader**。  

　　在上一篇文章 [Serial Downloader模式(sdphost, mfgtool)](http://www.cnblogs.com/henjay724/p/9034563.html) 里痞子衡为大家介绍了i.MXRT Boot的Serial Downloader模式，这种模式主要是用来引导启动Flashloader，那么Flashloader到底具有哪些功能？这是本篇文章痞子衡要为大家解惑的主题。  

　　痞子衡在前面提过Flashloader程序主要是用来将你的Application下载进i.MXRT支持的所有外部非易失性存储器中，为后续从外部存储器启动做准备。BootROM只有启动Application功能，没有下载更新Application功能，而Flashloader最核心的就是下载更新Application功能，所以Flashloader是BootROM的完美补充。你可能会疑问，为什么不把Flashloader的下载更新Application的功能也放进BootROM里？痞子衡个人觉得应该是芯片成本问题，ROM的空间是96KB（RT102x/RT105x）/128KB（RT106x），如果把Flashloader功能也放进BootROM里，势必要扩大ROM空间，从而导致芯片成本上升，做成二级Flashloader既不占ROM空间，也方便Flashloader程序自身的维护升级（目前RT1050的Flashloader版本是1.1，你看，这不显然升级过嘛）。  

### 一、进入Flashloader程序
#### 1.1 官方程序包
　　恩智浦提供了 [Flashloader程序包](https://www.nxp.com/products/processors-and-microcontrollers/applications-processors/i.mx-applications-processors/i.mx-rt-series/i.mx-rt1050-crossover-processor-with-arm-cortex-m7-core:i.MX-RT1050?tab=Design_Tools_Tab)，你首先需要下载这个Flashloader包，Flashloader所有相关资源全在包里面。注：每个i.MXRT子系列均有一个以子系列名字命名的Flashloader包，即Flashloader程序并不是通用的（偷偷告诉你，其实RT105x与RT106x是通用的），此处以RT1050系列为例：  

<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_FlashloaderDownloadPage.PNG" style="zoom:100%" />

　　Flashloader程序是\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Flashloader\flashloader.elf  
　　Flashloader工具在\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools目录下  

#### 1.2 三种引导方式
　　其实引导启动Flashloader的方式并不唯一，Flashloader就是一段运行在SRAM中的应用程序而已，只要能有工具将Flashloader下载进SRAM，并将CPU的PC指针指向Flashloader的程序入口便可启动Flashloader。下面痞子衡分别介绍3种引导方式：  
##### 1.2.1 标准方式：通过sdphost
　　第一种引导方式是通过BootROM的Serial Downloader模式和sdphost.exe工具，这是恩智浦官方推荐的方式，这种方式在上一篇文章里已经详细介绍过了，这里不再赘述。
　　这种方式的优点是不需要外接调试器，缺点是涉及到BootROM启动以及IVT的知识，需要有BootROM相关知识储备。  

##### 1.2.2 简便方式：通过J-Link Commander
　　第二种引导方式是通过外接J-Link调试器和J-Link Commander工具（JLink.exe）。i.MXRT芯片JTAG口连接上J-Link调试器后，安装好Jlink驱动（痞子衡安装的是v6.30e版本），打开J-Link Commander（即\SEGGER\JLink_V630e\JLink.exe），连接上i.MXRT的Core，按顺序执行如下JLink命令：  

> J-Link><font style="font-weight:bold;" color="Blue">loadfile C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Flashloader\flashloader.srec</font>
> ```text
> Downloading file [C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Flashloader\flashloader.srec]...
O.K.
> ```
> J-Link><font style="font-weight:bold;" color="Blue">mem32 0x20002000 2</font>
> ```text
> 20002000 = 20215A70 20014B91
> ```
> J-Link><font style="font-weight:bold;" color="Blue">wreg MSP 20215A70</font>
> ```text
> MSP = 0x20215A70
> ```
> J-Link><font style="font-weight:bold;" color="Blue">wreg PSP 20215A70</font>
> ```text
> PSP = 0x20215A70
> ```
> J-Link><font style="font-weight:bold;" color="Blue">SetPC 20014B91</font>
> J-Link><font style="font-weight:bold;" color="Blue">g</font>

　　到这里Flashloader就已经被成功启动了，有朋友看不懂上面的一串JLink命令，痞子衡为大家解释一下：  
　　- loadfile命令用于将Flashloader程序数据（.srec格式，含地址信息）下载进SRAM（0x20002000）中；  
　　- mem32命令用于从起始下载地址(0x20002000)读回部分数据（8字节）确认上一步的下载操作是否成功，并且获取Flashloader的起始SP和PC值。  
　　- wreg命令用于设置R13(MSP/PSP)寄存器的值，使其等于Flashloader的初始SP值。  
　　- SetPC命令用于设置R15(PC)寄存器的值，使其指向Flashloader的初始PC。  
　　- g命令用于让Core开始执行代码（Flashloader程序）。  

　　这种方式的优点是不需要借助BootROM（你可以不用了解BootROM相关工具用法），缺点是需要额外准备一个J-Link调试器。

> Note：关于JLink命令的详细解释请查阅JLink驱动软件安装目录下\SEGGER\JLink_V630e\Doc\UM08001_JLink.pdf文档里的3.2 J-Link Commander (Command line tool)一节。  

##### 1.2.3 高级方式：通过Ozone
　　前两种引导方式都是把Flashloader程序当做黑盒子，只需要将其启动运行即可，接下来痞子衡介绍的引导方式可以让你看到Flashloader源代码并且可以让你调试Flashloader。这第三种引导方式还是通过外接J-Link调试器，但还需要一个特殊的软件，即SEGGER公司提供的Ozone软件，去SEGGER先下载Ozone软件（痞子衡下载的版本是v2.56c），下载安装后打开Ozone软件，第一步选择CM7，第二步选择SWD，第三步选择elf文件时要选择Flashloader下载包里的flashloader.elf  
<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_flashloader_ozone_elf.PNG" style="zoom:100%" />

　　确认elf文件后，点击“Download & Reset Program”  
<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_flashloader_ozone_load.PNG" style="zoom:100%" />

　　此时Flashloader程序已经被下载进i.MXRT中，并且在Disassembly窗口可以看到Flashloader汇编源程序，底下你就可以开始调试执行Flashloader程序。由于下载包里的flashloader.elf文件并没有包含所有Flashloader工程的信息（至少没有包含C代码信息，应该是恩智浦官方故意这么做的），所以我们使用Ozone软件调试看不到C代码，稍微有点遗憾。  
<img src="http://henjay724.com/image/cnblogs/i.MXRT_Boot_flashloader_ozone_asm.PNG" style="zoom:100%" />

#### 1.3 支持的通信外设pinout
　　Flashloader支持的通信外设与BootROM支持的通信外设是一模一样的，也是USB-HID和UART，并且pinout也一样（Pinout适用RT105x和RT102x）：  

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

> <font color="Red">Note: 如果硬件板上UART_RX引脚没有接上拉电阻，可能会导致USB-HID设备枚举成功率降低，因为UART_RX悬空输入会有干扰数据使得Flashloader误以为UART是active peripheral，所以安全起见，请保证UART_RX引脚连接上拉电阻。</font>

### 二、blhost/elftosb/mfgtool的使用
　　Flashloader配套上位机工具有3个，elftosb.exe用于生成sb格式文件（这个工具后续会详细介绍），mfgtool是GUI软件（上一篇文章已经介绍过，其主要配合sb文件使用），blhost.exe是痞子衡在这里要着重介绍的软件。  
　　blhost.exe是命令行工具，使用blhost既可以通过UART口也可以通过USB口与Flashloader进行通信与命令交互。  
　　在命令行下打开blhost.exe，输入-?命令可以看到blhost使用帮助，相比sdphost，blhost支持的命令更多：  

```text
PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win> .\blhost.exe -?
usage: C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win\blhost.exe
                       [-?|--help]
                       [-p|--port <name>[,<speed>]]
                       [-u|--usb [[[<vid>,]<pid>]]]
                       [-t|--timeout <ms>]
                       -- command <args...>

Options:
  -?/--help                    Show this help
  -p/--port <name>[,<speed>]   Connect to target over UART. Specify COM port
                               and optionally baud rate
                                 (default=COM1,57600)
                                 If -b, then port is BusPal port
  -u/--usb [[[<vid>,]<pid>] | [<path>]]
                               Connect to target over USB HID device denoted by
                               vid/pid (default=0x15a2,0x0073) or device path
  -t/--timeout <ms>            Set packet timeout in milliseconds
                                 (default=5000)

Memory ID:
  Internal Memory              Device internal memory space
    0                            Internal Memory
                                 (Default selected memory)
  Mapped External Memory       The memories that are remapped to internal space,
                               and must be accessed by internal addresses.
                               (IDs in this group are only used for flash-erase-all and
                               configure-memory, and ignored by write-memory, read-memory,
                               flash-erase-region and flash-image(use default 0))
    1                            QuadSPI Memory
    8                            SEMC NOR Memory
    9                            FlexSPI NOR Memory
  Unmapped External Memory     Memories which cannot be remapped to internal space,
                               and only can be accessed by memories' addresses.
                               (Must be specified for all commands with <memoryId> argument)
    256 (0x100)                  SEMC NAND Memory
    257 (0x101)                  SPI NAND Memory
    272 (0x110)                  SPI NOR/EEPROM Memory
    273 (0x111)                  I2C NOR/EEPROM Memory
    288 (0x120)                  uSDHC SD Memory
    289 (0x121)                  uSDHC MMC Memory

** Note that not all memories are supported on all
      Kinetis Bootloader platforms.

Command:
  reset                        Reset the chip
  get-property <tag> [<memoryId> | <index>]
    1                          Bootloader version
    2                          Available peripherals
    7                          Available commands
    10                         Verify Writes flag
    11                         Max supported packet size
    12                         Reserved regions
    14                         Start of RAM, <index> is required
    15                         Size of RAM, <index> is required
    23                         QuadSpi initialization status
    24                         Target version
    25                         External Memory Attrubutes, <memoryId> is required.
  set-property <tag> <value>
    10                         Verify Writes flag
  flash-erase-region <addr> <byte_count> [memory_id]
                               Erase a region of flash according to [memory_id].
  flash-erase-all [memory_id]  Erase all flash according to [memory_id],
                               excluding protected regions.
  read-memory <addr> <byte_count> [<file>] [memory_id]
                               Read memory according to [memory_id] and write to file
                               or stdout if no file specified
  write-memory <addr> [<file>[,byte_count]| {{<hex-data>}}] [memory_id]
                               Write memory according to [memory_id] from file
                               or string of hex values,
                               e.g. data.bin (writes entire file)
                               e.g. data.bin 8 (writes first 8 bytes from file)
                               e.g. "{{11 22 33 44}}" (w/quotes)
                               e.g. {{11223344}} (no spaces)
  fill-memory <addr> <byte_count> <pattern> [word | short | byte]
                               Fill memory with pattern; size is
                               word (default), short or byte
  receive-sb-file <file>       Receive SB file
  execute <addr> <arg> <stackpointer>
                               Execute at address with arg and stack pointer
  call <addr> <arg>            Call address with arg
  configure-memory <memory_id> <internal_addr>
                               Apply configuration block at internal memory address
                               <internal_addr> to memory with ID <memory_id>
  flash-image <file> [erase] [memory_id]
                               Write a formated image <file> to memory with ID
                               <memory_id>. Supported file types: SRecord
                               (.srec and .s19) and HEX (.hex). Flash is erased
                               before writing if [erase]=erase. The erase unit
                               size depends on the target and the minimum erase
                               unit size is 1K.
  list-memory                  List all on-chip Flash and RAM regions, and off-chip
                               memories, supported by current device.
                               Only the configured off-chip memory will be list.
  efuse-program-once <addr> <data>
                               Program one word of OCOTP Field
                               <addr> is ADDR of OTP word, not the shadowed memory address.
                               <data> is hex digits without prefix '0x'
  efuse-read-once <addr>
                               Read one word of OCOTP Field
                               <addr> is ADDR of OTP word, not the shadowed memory address.
  generate-key-blob <dek_file> <blob_file>
                               Generate the Blob for given Dek Key
                               <dek_file> - input, a binary Dek Key (128 Bits) generated by CST tool.
                               <blob_file> - output, a generated blob (72 Bytes) in binary format.

** Note that not all commands/properties are supported on all
      Kinetis Bootloader platforms.
```

　　让我们尝试一下使用blhost与Flashloader通信（通过USB-HID），如果能得到如下结果，说明Flashloader工作正常，可以进行后续操作。  
> PS C:\Flashloader_i.MXRT1050_GA\Flashloader_RT1050_1.1\Tools\blhost\win> <font style="font-weight:bold;" color="Blue">.\blhost.exe -u 0x15a2,0x0073 -- get-property 1</font>
> ```text
> Inject command 'get-property'
> Response status = 0 (0x0) Success.
> Response word 1 = 1258422528 (0x4b020100)
> Current Version = K2.1.0
> ```

### 三、下载更新Application示例
　　因为BootROM支持启动的外部存储器很多，所以Flashloader支持下载更新的外部存储器也与BootROM一一对应。在上一节blhost的命令帮助里，我们可以看到Memory ID里已经给各种外部储存器分配了ID号，在使用blhost命令时使用不同的ID号即可操作相应外部存储器。  
　　其实Flashloader已经把外部存储器的下载更新Application操作封装得很简单也很统一，我们其实只需要3步操作即可完成Application的下载。以Block Size为128KB的Raw NAND为例（即SEMC NAND Memory，Memory ID=0x100）：  

```text
// 在SRAM里临时存储Raw NAND配置数据
blhost -u -- fill-memory 0x2000 0x4 0xD0010101 // ONFI 1.0, non-EDO, Timing mode 0, 8bit IO, CSX0, HW ECC Check, inital HW ECC is enabled
blhost -u -- fill-memory 0x2004 0x4 0x00010101 // image copy = 1, search stride = 1, search count = 1
blhost -u -- fill-memory 0x2008 0x4 0x00020001 // block index = 2, block count = 1

// 使用Raw NAND配置数据去配置Raw NAND接口
blhost -u -- configure-memory 0x100 0x2000

// 擦除Raw NAND并将image下载进Raw NAND
blhost -u -- flash-erase-region 0x40000 0x20000 0x100    // Erase 1 block starting from block 2
blhost -u -- write-memory 0x40000 ivt_image.bin 0x100    // Program ivt_image.bin to block 2
```

　　其中image.bin是包含IVT的Application镜像数据，关于上述命令的具体意义痞子衡会在后续Raw NAND启动的文章里详尽解释，这里只是给大家一个初步体验。  

　　至此，飞思卡尔i.MX RT系列MCU的Flashloader痞子衡便介绍完毕了，掌声在哪里~~~ 

