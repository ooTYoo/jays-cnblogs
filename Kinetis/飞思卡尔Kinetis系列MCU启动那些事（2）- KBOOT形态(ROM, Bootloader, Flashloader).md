----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**飞思卡尔Kinetis系列MCU的KBOOT形态**。  

　　痞子衡在前一篇文章里简介了 [KBOOT架构](https://www.cnblogs.com/henjay724/p/9316150.html)，我们知道KBOOT是一个完善的Bootloader解决方案，这个解决方案主要设计用于Kinetis芯片上，目前Kinetis芯片起码有上百种型号，KBOOT在这上百种Kinetis芯片里存在的形式并不是完全一样的，KBOOT主要有三种存在形式（ROM Bootloader、Flashloader、Flash-Resident Bootloader），下面痞子衡为大家细说这三种形态：  

### 一、KBOOT形态区别
　　KBOOT有三种形态，分别是如下图所示的ROM Bootloader、Flashloader、Flash-Resident Bootloader，三种形态共享大部分KBOOT源码，仅在一些细节上有差别，这些细节在KBOOT源码里是用条件编译加以区分的，对应的条件编译宏分别是BL_TARGET_ROM, BL_TARGET_RAM, BL_TARGET_FLASH。三种形态最大的区别其实是在链接文件上，经过汇编器后的read only section分别链接在了Kinetis芯片System memory空间里的ROM（起始地址0x1c000000）、RAM（区间地址0x20000000）、Flash（起始地址0x00000000）区域。  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_block.PNG" style="zoom:100%" />

　　下表是KBOOT三种形态的对比，分别从use case、delivery mechanism、supported device、clock configuration、feature五大角度进行了对比：  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_comparison.PNG" style="zoom:100%" />

　　总结来说，可以这么看KBOOT这三种存在的由来：  
> * 对于2014年初及以后问世的Kinetis芯片（比如MKL03、MKL27、MKL43、MKL80、MKE18F等），芯片内基本都是含ROM空间的，因此KBOOT是以ROM Bootloader的形式存在的；  
> * 对于2014年初及以后主推的Kinetis芯片（比如MK22、MK65、MKV31、MKS22等），芯片内虽然没有ROM空间，但飞思卡尔希望能给客户提供至少一次免编程器烧录Application（用于量产）的机会，因此KBOOT是以Flashloader的形式存在的；  
> * 对于在市场上主流又畅销的Kinetis芯片（比如MKL25、MK22、MK66、MKL28等），不管芯片内是否有ROM空间，飞思卡尔都希望能够给出Bootloader源码，以便让客户自由修改来满足其个性化需求，因此KBOOT是以Flash-Resident Bootloader的形式存在的；  

### 二、KBOOT各形态实现
#### 2.1 ROM Bootloader
　　KBOOT的ROM Bootloader形态是放在ROM空间里的，随着芯片一起Tape-out出厂，固化在芯片里面，所以该形态可以被当做硬件模块，可以被无限次使用。  
　　因为有了ROM的存在，所以芯片上电启动便有了两种选择：从ROM启动、从内部Flash启动，这种启动选择是由芯片系统决定的。  
　　如果是从ROM启动，那么我们可以借助ROM将Application烧写进Flash（内部/外部）的起始空间并跳转过去执行。跳转至Flash执行分为：从内部Flash执行、从外部QSPI NOR Flash执行，这种执行选择是由ROM代码决定的。  
　　如果已经使用ROM将Application下载进内部Flash起始地址，并在系统设置里设置芯片从内部Flash启动，那么下次芯片复位启动完全可以绕开ROM直接从内部Flash起始地址执行Application。  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_mem_layout_ROM.PNG" style="zoom:100%" />

#### 2.2 Flash-Resident Bootloader
　　KBOOT的Flash-Resident Bootloader形态是放在内部Flash起始空间的，以源代码的形式提供给客户，客户需要自己编译KBOOT工程并使用编程器/调试器将编译生成的KBOOT binary下载进芯片内部Flash起始地址，除非使用调试器将其擦除，否则其也可以被无限次使用。  
　　对于没有ROM的芯片，芯片上电只能从内部Flash起始地址处开始启动，因为Flash-Resident Bootloader已经占据了内部Flash的起始空间，所以芯片永远是先执行Flash-Resident Bootloader。借助Flash-Resident Bootloader只能将Application烧写进内部Flash一定偏移处（这个偏移地址由Flash-Resident Bootloader指定）并跳转过去执行。  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_mem_layout_Bootloader.PNG" style="zoom:100%" />

#### 2.3 Flashloader
　　KBOOT的Flashloader形态其实也是放在内部Flash起始空间的，不过与Flash-Resident Bootloader形态在Flash里执行不同之处在于Flashloader形态是在SRAM里执行的，众所周知，SRAM断电是不保存数据的，因此Flashloader需要一个放在内部Flash里的配套loader程序，在芯片上电时先运行Flash里的loader程序，由loader程序将Flashloader从Flash中搬运到SRAM中并跳转到SRAM中运行。  
　　Flashloader是在芯片出厂之后由飞思卡尔产品工程师将其binary预先下载进内部Flash再售卖给客户，所以客户拿到芯片之后至少可以使用一次Flashloader，客户借助Flashloader可以将Application烧写进内部Flash起始空间(同时也覆盖了原Flashloader-loader)，这就是Flashloader只能被使用一次的原因。  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_mem_layout_Flashloader.PNG" style="zoom:100%" />

##### 2.3.1 loader机制
　　关于loader机制与实现，有必要详细讲解一下，让我们结合代码分析，首先从官网下载NXP_Kinetis_Bootloader_2_0_0.zip包，就以KS22芯片为例（\targets\MKS22F25612\bootloader.eww）：  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_codebase_package.PNG" style="zoom:100%" />

　　使用IAR EWARM 7.80.x开发环境打开KS22的Bootloader工程，可以看到有如下三个工程，其中flashloader.ewp便是主角，其源码文件与ROM和Flash-Resident Bootloader是一样，只是工程链接文件有区别，其代码段链接在于SRAM里；maps_bootloader.ewp便是Flash-Resident Bootloader，不是此处讨论的重点；flashloader_loader.ewp就是Flashloader能在SRAM里执行的关键所在。  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_ks22_projects.PNG" style="zoom:100%" />

　　flashloader_loader.ewp工程里除了必要的芯片startup文件外，只有三个源文件：flashloader_image.c/h，bl_flashloader.c，其中bl_flashloader.c里包含了工程main函数，让我们试着分析这个文件以及main函数，下面是bl_flashloader.c的文件内容：  

```C
#include <string.h>
#include "bootloader_common.h"
#include "fsl_device_registers.h"
#include "bootloader/flashloader_image.h"

#if DEBUG
#include "debug/flashloader_image.c"
#else
#include "release/flashloader_image.c"
#endif

////////////////////////////////////////////////////////////////////////////////
// Code
////////////////////////////////////////////////////////////////////////////////

// @brief Run the bootloader.
void bootloader_run(void)
{
    // Copy flashloader image to RAM.
	// 关键拷贝，实现了flashloader binary从Flash到RAM的转移
    memcpy((void *)g_flashloaderBase, g_flashloaderImage, g_flashloaderSize);

    // Turn off interrupts.
    __disable_irq();

    // Set the VTOR to default.
    SCB->VTOR = 0x0;

    // Memory barriers for good measure.
    __ISB();
    __DSB();

    // Set main stack pointer and process stack pointer.
    __set_MSP(g_flashloaderStack);
    __set_PSP(g_flashloaderStack);

    // Jump to flashloader entry point, does not return.
	// 关键跳转，执行位置从Flash切换到了RAM
    void (*entry)(void) = (void (*)(void))g_flashloaderEntry;
    entry();
}

// @brief Main bootloader entry point.
int main(void)
{
    bootloader_run();

    // Should never end up here.
    while (1);
}
```

　　从上述bl_flashloader.c的文件里我们可以看到，其实loader工程的main函数特别简单，它就是将内部Flash里的g_flashloaderImage[]数据（即flashloader.ewp编译生成的binary）拷贝到g_flashloaderBase地址（即flashloader binary起始地址）处，并将SP和PC分别指向g_flashloaderStack（即flashloader初始SP）和g_flashloaderEntry（即flashloader的Reset Handler入口）。  
　　那么g_flashloaderXX常量都放在哪里的呢？打开\targets\MKS22F25612\iar\flashloader\output\Release\flashloader_image.c可以找到答案:  

```C
const uint8_t g_flashloaderImage[] = {
    0x70, 0x62, 0x00, 0x20, 0x11, 0xc4, 0xff, 0x1f, 0xeb, 0xc4, 0xff, 0x1f, 0x75, 0xef, 0xff, 0x1f, 
    // 此处省略41552 bytes
    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
};
const uint32_t g_flashloaderSize = 41584U;
const uint32_t g_flashloaderBase = 0x1fffc000;
const uint32_t g_flashloaderEntry = 0x1fffc411;
const uint32_t g_flashloaderStack = 0x20006270;
```

　　loader机制越来越清晰了，现在只剩最后一个问题了，flashloader_image.c文件是哪里来的？这个文件当然可以手动创建，文件里的信息都可以从flashloader.ewp工程生成的elf/map文件里中找到，但本着高效的原则，但凡能脚本自动生成的决不手动创建，是的这个flashloader_image.c文件就是脚本自动生成的，在flashloader.ewp的Option选项的Build Actions里可以看到调用脚本的命令，这个脚本名叫create_flashloader_image.bat。  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_ks22_flashloader_project_option.PNG" style="zoom:100%" />

　　在\bin目录下存放了所有脚本文件，当然也包括create_flashloader_image.bat，先打开这个脚本看一下：  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_codebase_package_bin_folder.PNG" style="zoom:100%" />

```Shell
cd /d %1
ielftool --bin output\%2\flashloader.elf flashloader.bin
python ..\..\..\..\bin\create_fl_image.py output\%2\flashloader.elf flashloader.bin output\%2\flashloader_image.c
```

　　ielftool.exe是IAR软件目录下的工具，可以将elf文件转换成bin文件。最核心的脚本其实是create_fl_image.py，这个python脚本根据elf文件和bin文件生成了flashloader_image.c文件。打开create_fl_image.py文件如下（作了一些异常判断的删减，为了突出脚本主逻辑）：  

```Python
import sys
import os
import elf

# usage: create_fl_image.py <elffile> <binfile> <cfile> 

def main(argv):
    # Collect arguments
    elfFilename = argv[0]
    binFilename = argv[1]
    cFilename = argv[2]

    # Open files
    binFile = open(binFilename, 'rb')
    cFile = open(cFilename, 'w')
	# 创建了elfData对象，用于后续处理.elf格式文件
    elfData = elf.ELFObject()
	with open(elfFilename, 'rb') as elfFile:
	    # 开始处理输入的.elf文件
		elfData.fromFile(elfFile)
		if elfData.e_type != elf.ELFObject.ET_EXEC:
			raise Exception("No executable")
		# 开始从.elf里获取关键信息
		resetHandler = elfData.getSymbol("Reset_Handler")
		vectors = elfData.getSymbol("__Vectors")
		stack = elfData.getSymbol("CSTACK$$Limit")

    # Print header
    print >> cFile, 'const uint8_t g_flashloaderImage[] = {'
    # Print byte data
    totalBytes = 0
    while True:
        data = binFile.read(16)
        dataLen = len(data)
        if dataLen == 0: break
        totalBytes += dataLen;
        cFile.write('    ')
        for i in range(dataLen):
            cFile.write('0x%02x, ' % ord(data[i]))
        print >> cFile
    print >> cFile, '};\n'

    # Print size and other info
    cFile.write('const uint32_t g_flashloaderSize = %dU;\n' % totalBytes)
    cFile.write('const uint32_t g_flashloaderBase = 0x%x;\n' % vectors.st_value)
    cFile.write('const uint32_t g_flashloaderEntry = 0x%x;\n' % resetHandler.st_value)
    cFile.write('const uint32_t g_flashloaderStack = 0x%x;\n' % stack.st_value)

if __name__ == "__main__":
   main(sys.argv[1:])
```

　　create_fl_image.py脚本里除了普通文件操作外，最关键的是这句elfData = elf.ELFObject()，调用了elf.py文件提供的elf格式文件操作接口，通过这些接口得到了flashloader里的关键信息（vectors、resetHandler、stack），感兴趣的可以自己去分析elf.py文件。  

### 三、KBOOT各形态芯片支持
　　截止目前（2017年），KBOOT支持的Kinetis芯片全部列出在下表：  

<img src="http://henjay724.com/image/cnblogs/Kinetis_Boot_form_chip_list.PNG" style="zoom:100%" />

　　至此，飞思卡尔Kinetis系列MCU的KBOOT形态痞子衡便介绍完毕了，掌声在哪里~~~ 

