----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式开发里的linker文件**。  

　　在前一节课[源文件(.c/.h/.s)](http://www.cnblogs.com/henjay724/p/8183257.html)里，痞子衡给大家系统地介绍了source文件，source文件是嵌入式工程里典型的input文件，那么还有没有其他类型的input文件？既然痞子衡这么提问了，那答案肯定是有啦。今天痞子衡要讲的linker文件就属于另一种input文件。  
　　linker文件顾名思义就是嵌入式工程在链接阶段所要用到的文件，source文件在编译过程完成之后（此时已经是机器可识别的二进制机器码数据），需要再经过链接器从而将二进制数据有序组织起来形成最终的二进制可执行文件，该二进制文件最终会被下载进芯片内部非易失性存储器里。linker文件就是用来指示链接器如何组织编译生成的二进制数据。  
　　linker文件是跟IDE息息相关的，本文以IAR EWARM为例介绍linker文件，其他IDE下的linker文件可触类旁通。  

### 一、 嵌入式系统中的section
　　在讲linker文件之前，痞子衡必须先跟大家理清一个嵌入式系统中很重要的概念-section。那么什么是section？我们写的C或者汇编source文件里都是各种应用代码，这些代码按功能可以分为很多种类，比如常量、变量、函数、堆栈等，而相同类型的代码的集合便是一个section，链接器在链接时组织数据的基本单元便是section。那么一个典型的嵌入式系统中到底有多少种section呢？下面列出了IAR里默认的所有section，那些常见section在后续介绍linker文件里会被提到。  
```text
//常见Section
.bss                 // Holds zero-initialized static and global variables.
CSTACK               // Holds the stack used by C or C++ programs.
.data                // Holds static and global initialized variables.
.data_init           // Holds initial values for .data sections when the linker directive initialize is used.
HEAP                 // Holds the heap used for dynamically allocated data.
.intvec              // Holds the reset vector table
.noinit              // Holds __no_init static and global variables.
.rodata              // Holds constant data.
.text                // Holds the program code.
.textrw              // Holds __ramfunc declared program code.
.textrw_init         // Holds initializers for the .textrw declared section.

//较冷僻Section
.exc.text            // Holds exception-related code.
__iar_tls.$$DATA     // Holds initial values for TLS variables.
.iar.dynexit         // Holds the atexit table.
.init_array          // Holds a table of dynamic initialization functions.
IRQ_STACK            // Holds the stack for interrupt requests, IRQ, and exceptions.
.preinit_array       // Holds a table of dynamic initialization functions.
.prepreinit_array    // Holds a table of dynamic initialization functions.
Veneer$$CMSE         // Holds secure gateway veneers.

//更冷僻Section
.debug               // Contains debug information in the DWARF format
.iar.debug           // Contains supplemental debug information in an IAR format
.comment             // Contains the tools and command lines used for building the file
.rel or .rela        // Contains ELF relocation information
.symtab              // Contains the symbol table for a file
.strtab              // Contains the names of the symbol in the symbol table
.shstrtab            // Contains the names of the sections.
```
> Note：上述section的详细解释请查阅IAR软件安装目录下\IAR Systems\Embedded Workbench xxx\arm\doc\EWARM_DevelopmentGuide.ENU.pdf文档里的Section reference一节。  

### 二、解析linker文件
　　知道了section概念，那便可开始深入了解linker文件，什么是linker文件？linker文件是按IDE规定的语法写成的用于指示链接器分配各section在嵌入式系统存储器中存放位置的文件。大家都知道嵌入式系统存储器主要分为两类：ROM（非易失性），RAM（易失性），所以相应的这些section根据存放的存储器位置不同也分为两类属性：readonly, readwrite。实际上linker文件的工作就是将readonly section放进ROM，readwrite section放进RAM。  
　　那么到底该如何编写工程的linker文件呢？正如前面所言，linker文件也是有语法的，而且这语法是由IDE指定的，所以必须要先掌握IDE制定的语法规则，linker文件语法规则相对简单，最常用的关键字就是如下8个：  
```text
// 动词类关键字
define                // 定义各种空间范围、长度
initialize            // 设置section初始化方法
place in              // 放置section于某region中（具体地址由链接器分配）
place at              // 放置section于某绝对地址处

// 名词类关键字
symbol                // 各种空间范围、长度的标识
memory                // 整个ARM内存空间的标识
region                // 在整个ARM内存空间中划分某region空间的标识
block                 // 多个section的集合块的标识
```
> Note：上述linker语法的详细解释请查阅IAR软件安装目录下\IAR Systems\Embedded Workbench xxx\arm\doc\EWARM_DevelopmentGuide.ENU.pdf文档里的The linker configuration file一节。  

　　到这里我们已经可以开始愉快地写linker文件了，是不是有点按捺不住了？来吧，只需要三步走，Let's do it。  
　　此处假设MCU物理空间为：ROM（0x0 - 0x1ffff）、RAM（0x10000000 - 0x1000ffff），痞子衡要写的linker要求如下：  
> * 中断向量表必须放置于ROM起始地址0x0，且必须256字节对齐
> * STACK大小为8KB，HEAP大小为1KB，且必须8字节对齐
> * SATCK必须放置在RAM起始地址0x10000000
> * 其余section放置在正确的region里，具体空间由链接器自动分配

#### 2.1 定义物理空间
　　第一步我们先定义3块互不重叠的空间ROM\_region、RAM\_region、STACK\_region，其中ROM\_region对应的是真实的ROM空间，RAM\_region和STACK\_region组合成真实的RAM空间。  
```text
// 定义物理空间边界
define symbol __ICFEDIT_region_ROM_start__ = 0x00000000;
define symbol __ICFEDIT_region_ROM_end__   = __ICFEDIT_region_ROM_start__ + (128*1024 - 1);
define symbol __ICFEDIT_region_RAM_start__ = 0x10000000;
define symbol __ICFEDIT_region_RAM_end__   = __ICFEDIT_region_RAM_start__ + (64*1024 - 1);
define symbol __ICFEDIT_intvec_start__     = __ICFEDIT_region_ROM_start__;

// 定义堆栈长度
define symbol __ICFEDIT_size_cstack__      = (8*1024);
define symbol __ICFEDIT_size_heap__        = (1*1024);

// 定义各region具体空间范围
define memory mem with size = 4G;
define region ROM_region    = mem:[from __ICFEDIT_region_ROM_start__ to __ICFEDIT_region_ROM_end__];
define region STACK_region  = mem:[from __ICFEDIT_region_RAM_start__ to  __ICFEDIT_region_RAM_start__ + __ICFEDIT_size_cstack__ - 1];
define region RAM_region    = mem:[from __ICFEDIT_region_RAM_start__ + __ICFEDIT_size_cstack__  to __ICFEDIT_region_RAM_end__];
```

#### 2.2 定义section集合
　　第二步是自定义section集合块，细心的朋友可以看到右边花括号里包含的都是上一节介绍的系统默认section，我们会把具有相同属性的section集合成到一个block里，方便下一步的放置工作。  
```text
// 定义堆栈块及其属性
define block CSTACK    with alignment = 8, size = __ICFEDIT_size_cstack__   { };
define block HEAP      with alignment = 8, size = __ICFEDIT_size_heap__     { };

// 定义section集合块
define block Vectors with alignment=256 { readonly section .intvec };
define block CodeRelocate               { section .textrw_init };
define block CodeRelocateRam            { section .textrw };
define block ApplicationFlash           { readonly, block CodeRelocate };
define block ApplicationRam             { readwrite, block CodeRelocateRam, block HEAP };
```
　　有朋友可能会疑问，为何要定义CodeRelocate、CodeRelocateRam这两个block？按道理说这两个block对应的section可以分别放进ApplicationFlash和ApplicationRam，那为何多此一举？仔细上过痞子衡前一节课[source文件]()的朋友肯定就知道答案了，在那节课里介绍的startup.c文件里有一个叫init\_data\_bss()的函数，这个函数会完成初始化CodeRelocateRam块的功能，它找寻的就是CodeRelocate段名字，这个名字比系统默认的textrw名字看起来更清晰易懂。  
#### 2.3 安置section集合
　　第三步便是处理放置那些section集合块了，在放置集合块之前还有initialize manually语句，为什么会有这些语句？还是得结合前面提及的startup.c文件里的init\_data\_bss()函数来说，这个函数是开发者自己实现的data,bss段的初始化，所以此处需要通知IDE，你不需要再帮我做初始化工作了。  
```text
// 设置初始化方法
initialize manually { readwrite };
initialize manually { section .data};
initialize manually { section .textrw };
do not initialize   { section .noinit };

// 放置section集合块
place at start of ROM_region { block Vectors };
//place at address mem:__ICFEDIT_intvec_start__ { block Vectors };
place in ROM_region          { block ApplicationFlash };
place in RAM_region          { block ApplicationRam };
place in STACK_region        { block CSTACK };
```
　　当然如果你希望IDE帮你自动初始化data,bss,textrw段，那么可以用下面语句替换initialize manually语句。  
```text
initialize by copy { readwrite, section .textrw };
```
　　设置好初始化方法后，便是放置section集合块了，放置方法主要有两种，place in和place at，前者用于指定空间块放置（不指定具体地址），后者是指定具体地址放置。  

　　至此一个基本的linker文件便大功告成了，是不是so easy？  

### 番外一、自定义section
　　有耐心看到这里的朋友，痞子衡必须得放个大招奖励一下，前面讲的都是怎么处理系统默认段，那么有没有可能在代码里自定义段呢？想象一下你有这样的需求，你需要在你的应用里开辟一块1KB的可更新的数据区，你想把这个数据区指定到地址0x18000 - 0x183ff的范围内，你需要在应用里定义4 Byte的只读config block常量指向这个可更新数据区首地址（这段config block只会被外部debugger或者bootloader更新），如何做到？  
```C
// C文件中
/////////////////////////////////////////////////////
// 用@操作符指定变量myConfigBlock[4]放进自定义.myBuffer section
const uint8_t myConfigBlock[4] @ ".myBuffer" = {0x00, 0x01, 0x02, 0x03};

// Linker文件中
/////////////////////////////////////////////////////
// 自定义指定的mySection_region，并把.myBuffer放到这个region
define region mySection_region = mem:[from  0x0x18000 to 0x183ff];
place at start of mySection_region { readonly section .myBuffer };
```
　　上面做到了将代码中的常量放入自定义段？，那么怎么将代码中的函数也放进自定义段呢？继续看下去  
```C
// C文件中
/////////////////////////////////////////////////////
// 用#pragma location指定函数myFunction()放进自定义.myTask section
#pragma location = ".myTask"
void myFunction(void)
{
    __NOP();
}

// Linker文件中
/////////////////////////////////////////////////////
// 把.myTask放到mySection_region
place in mySection_region { readonly section .myTask };
```
　　看起来大功告成了，最后还有一个注意事项，如果myConfigBlock在代码中并未被引用，IDE在链接的时候可能会忽略这个变量（IDE认为它没用，所以优化了），那么怎么让IDE强制链接myConfigBlock呢？IAR留了个后门，在options->Linker->Input选项卡中的Keep symbols输入框里填入你想强制链接的对象名（注意是代码中的对象名，而非linker文件中的自定义段名）即可。  
> Note：关于番外内容的更多细节请查阅IAR软件安装目录下\IAR Systems\Embedded Workbench xxx\arm\doc\EWARM_DevelopmentGuide.ENU.pdf文档里的Pragma directives一节。  

　　至此，嵌入式开发里的linker文件痞子衡便介绍完毕了，掌声在哪里~~~ 