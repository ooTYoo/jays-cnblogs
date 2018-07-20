----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式开发里的project文件**。  

　　前面两节课里，痞子衡分别给大家介绍了嵌入式开发中的两种典型input文件：[源文件(.c/.h/.s)](http://www.cnblogs.com/henjay724/p/8183257.html)、[链接文件(.icf)](http://www.cnblogs.com/henjay724/p/8191908.html)。痞子衡要再次提问了，还有没有input文件呢?答案确实是有，但这次真的是有且仅有了，本文要介绍的主角project文件也属于半个input文件。为什么说是半个?因为project文件不仅包含开发者指定的input信息，还包含很多其他辅助调试的input/output信息，算是嵌入式开发中承前启后的文件。而本文侧重点在于project文件中与开发者应用相关的input信息，仅当得到了这些input信息，再加上前面介绍的source和linker文件，那么你就已经得到了application所有的信息，你可以用它们来可以生成无歧义的可执行image binary。  
　　随着嵌入式软件工程的发展，为了应对日益复杂的需求，现代IDE的功能也越来越强大了，IDE版本更迭让人应接不暇，Keil MDK已然踏入5.0时代，IAR EWARM更是进入了8.0时代，IDE各有千秋，但本文要讲的内容却是每个IDE必须具有的基本功能，还是继续以IAR EWARM为例开始今天的内容：  

### 一、标准IDE功能
　　在开始今天的主题之前，痞子衡觉得有必要先简要给大家科普一下标准IDE应该具有的功能。现代IDE基本都是由组件构成，嵌入式开发中的每个阶段都对应着相应的组件，由这些组件去实现各阶段的需求。  
#### 1.1 IDE组件
　　标准嵌入式开发应该至少包括以下6个阶段，而IAR里对于每个阶段都有1个或多个组件：  
> * 输入（IAR Editor）：编辑源文件代码。
> * 编译（ICCARM、IASMARM）：编译源文件代码生成可执行二进制机器码。
> * 分析（C-STAT、MISRA-C）：编译过程中检查代码中潜在的问题。
> * 链接（ILINK）：链接可执行二进制机器码到指定ARM存储空间地址。
> * 下载（I-jet、flashloader）：将链接好的可执行二进制机器码下载进芯片内部非易失性存储器。
> * 调试（C-SPY、C-RUN）：在线调试代码在芯片中执行情况。

　　project文件主要用来记录整合上述6个阶段的所有开发需求。  

#### 1.2 IDE文件类型
　　既然IDE有很多组件，那么同时也会存在不同类型的文件以存储这些组件的所需要的信息。IAR里支持的文件扩展类型非常多，痞子衡在这里仅列举你所创建的工程根目录下的与工程同名的扩展文件，相信你一定会觉得眼熟。  
```text
.eww           // Workspace file
.ewp           // IAR Embedded Workbench project
.ewd           // Project settings for C-SPY
.ewt           // Project settings for C-STAT and C-RUN</td>
.dep           // Dependency information
```
　　本文要讲的内容都包含在.ewp文件里，ewp文件记录了开发者为应用指定的不可缺少的input信息，没有这些信息，application工程是不完整的。换句话说，如果你得到了application的所有source文件和linker文件，但没有ewp文件的话，可能导致最终生成的image binary文件是不同的。  
> Note：更多IAR支持的扩展文件类型请查阅IAR软件安装目录下\IAR Systems\Embedded Workbench xxx\arm\doc\EWARM_IDEGuide.ENU.pdf文档里的File types一节。  

### 二、解析project（ewp）文件
　　前面痞子衡铺垫了很多IDE/project基础概念，该是直奔主题的时候了，本文主角ewp工程文件到底包含哪些开发者指定的input信息？痞子衡从下面3个方面为大家揭秘：  
#### 2.1 源文件组织
　　一个稍微复杂一点的嵌入式工程，应用代码行数应该是以百行/千行为单位计算的（此处仅指的是由开发者自己创建的文件与代码），我们在组织代码的时候肯定不会只创建一个.c文件，单文件会导致代码功能模块结构不清晰，不方便工程的管理与维护。  
　　当我们为工程创建多个文件时，就会涉及到一个必然问题：引用路径问题（所以路径信息就是本文要说的第一个input信息）。当源文件数目较多时，通常我们会创建不同文件夹把相同功能的源文件都放在一起，当编译器开始编译.c源文件时会搜索include语句所包含的头文件。熟悉C语言的朋友肯定清楚下面两种不同include语句的用法：  
```C
#include <file.h>           // 引用编译器类库下的头文件（IDE安装路径）
#include "file.h"           // 引用当前工程下的头文件（project路径）
```
　　所以在ewp文件里会包含路径信息，所有路径都应该列在Options->C/C++ Compiler->Preprocessor下有Additional include directories里，这个路径既可以是当前PC的绝对路径，也可以是以ewp文件为基准的相对路径，为了保证工程可以在任意PC任意位置下正常编译，推荐使用如下相对路径方式列出所有路径：  
```text
ewp当前路径：$PROJ_DIR$/
ewp下级路径：$PROJ_DIR$/xxFolder/
ewp上级路径：$PROJ_DIR$/../
```
　　说到路径问题，痞子衡在这里顺便给大家介绍一种经典的嵌入式工程文件目录组织方式：  
```text
\projectDir
           \doc                            --放置工程文档

           \bsp                            --放置bsp（板级）相关的source file
                  \linker                    --工程linker文件
                  \src                       --板级相关的源文件（比如pinout，clock等）
                  \builds\xxBuild\.ewp       --工程ewp文件
                  .eww                       --工程workspace文件

           \src                            --放置bsp无关的source file
                  \platform                  --芯片头文件及CMSIS文件
                  \drivers                   --芯片片内外设driver
                  \include                   --要被所有source引用的头文件
                  \startup                   --标准的startup code
                  \utilities                 --标准的通用函数
                  \middleware                --独立的中间件
                  \components                --板级外设组件driver
                  \application               --当前应用主逻辑代码
```

#### 2.2 全局宏定义
　　经常使用条件编译的朋友肯定知道workspace文件与project文件的关系，一个项目通常只会有一个eww文件，但却可能会有多个ewp文件，这是因为源代码里常常会有条件编译，我们有时候会给项目不同的配置从而编译出不同的结果（速度优先/面积优先，特性控制...），这些配置就是由全局宏定义来实现的，打开Options->C/C++ Compiler->Preprocessor下的Defined symbols，在框内写入你需要定义的全局宏：  
```text
MACRO1            // 等价于源文件里的#define MACRO1 (1)
MACRO2=2          // 等价于源文件里的#define MACRO2 (2)
```
　　全局宏信息就是本文要说的第二个input信息，如果全局宏信息丢失，有时候工程编译并不会报错，因为编译器在处理如下普遍用法里的条件编译语句时会默认未定义的宏为0，而在处理普遍用法里的条件编译语句则会报错，所以推荐大家使用第二种条件编译用法来规避全局宏问题。  
```C
// 普遍用法
#if MACRO
    // your code block 1
#else
    // your code block 2
#endif

// 推荐用法
#if !defined(MACRO)
    #error "No valid MACRO defined!"
#elif (MACRO == 1)
    // your code block 1
#else
    // your code block 2
#endif
```

#### 2.3 编译选项
　　编译选项包含了编译器所需要的所有信息，代码需经过编译器编译才能生成二进制机器码，不同的编译器选项配置会生成不同的机器码，那么需要指定哪些选项呢？打开project的Options选项卡，分别设置下表item：  

<table>
    <tr>
        <th>Position</th>
        <th>Item</th>
        <th>Description</th>
    </tr>
    <tr>
        <td rowspan="3">General Options->Target-></td>
        <td>Processor variant->Core</td>
        <td><font color="Blue">指定ARM内核版本</td>
    </tr>
    <tr>
        <td>Endian mode</td>
        <td>指定内核大小端模式</td>
    </tr>
    <tr>
        <td>Floating point settings->FPU</td>
        <td>指定内核支持的FPU版本</td>
    </tr>
    <tr>
        <td>General Options->Library Configuration-></td>
        <td>Library</td>
        <td>选择C/C++动态链接库版本</td>
    </tr>
    <tr>
        <td>General Options->Library Option 2-></td>
        <td>Heap selection</td>
        <td>选择HEAP实现版本</td>
    </tr>
    <tr>
        <td rowspan="8">C/C++ Compiler-></td>
        <td>Language 1->Language</td>
        <td>指定编程语言类型</td>
    </tr>
    <tr>
        <td>Language 1->C dialect</td>
        <td>指定C语言标准</td>
    </tr>
    <tr>
        <td>Language 1->Language conformance</td>
        <td>选择对标准C/C++的遵循程度</td>
    </tr>
    <tr>
        <td>Language 2->Plain 'char' is</td>
        <td>选择对char的符号性默认处理方法</td>
    </tr>
    <tr>
        <td>Language 2->Floating-point semantics</td>
        <td>选择对浮点数的处理遵循C标准的程度</td>
    </tr>
    <tr>
        <td>Code->Process mode</td>
        <td>指定内核指令集模式</td>
    </tr>
    <tr>
        <td>Code->Position-independence</td>
        <td><font color="Blue">选择要生成位置无关代码的对象</td>
    </tr>
    <tr>
        <td>Optimizations->Level</td>
        <td>选择优化等级</td>
    </tr>
</table>

> Note：更多ewp文件中option解释请查阅IAR软件安装目录下\IAR Systems\Embedded Workbench xxx\arm\doc\EWARM_IDEGuide.ENU.pdf文档里的General Options和Compiler Options俩小节。  

　　编译设置信息就是本文要说的第三个input信息，当在project中组织好源文件并设置好正确的全局宏定义和编译选项，那么恭喜你，你的application设计工作已经基本完成了。  

### 三、创建demo工程
　　为方便后续课程的进行，本节课在最后顺便创建一个demo工程，以下是demo工程的信息：  

```text
IDE:        IAR EWARM v8.11.2
Device:     NXP MKL25Z128VLH4
project layout:   
    \D\myProject\bsp\builds\demo\demo.ewp
    \D\myProject\bsp\linker\iar\KL25Z128xxx4_flash.icf
    \D\myProject\bsp\src\startup_MKL25Z4.s   (仅保留前16个系统中断)
    \D\myProject\bsp\src\system_MKL25Z4.c   （仅做关闭WDOG操作）
    \D\myProject\bsp\src\system_MKL25Z4.h
    \D\myProject\bsp\helloArm.eww
    \D\myProject\src\platfrom\CMSIS
    \D\myProject\src\platfrom\devices\MKL25Z4
    \D\myProject\src\startup\reset.s
    \D\myProject\src\startup\startup.c
    \D\myProject\src\startup\startup.h
    \D\myProject\src\application\main.c
    \D\myProject\src\application\task.c
    \D\myProject\src\application\task.h
```

```c
// main.c
//////////////////////////////////////////////////////////
#include "task.h"
const uint32_t s_constant = 0x7f;
int main(void)
{
    uint32_t l_variable = 0x7f;
    if (s_constant == l_variable)
    {
        normal_task();
        ram_task();
        heap_task();
    }
    while (1);
}

// task.c
//////////////////////////////////////////////////////////
#include "task.h"
static    uint32_t s_variable0;
__no_init uint32_t n_variable1;
static    uint32_t s_variable2 = 0x5a;
static uint8_t s_array[16];
void normal_task(void)
{
    s_variable0 *= 2;
}
__ramfunc void ram_task(void)
{
    n_variable1++;
}
void heap_task(void)
{
    uint8_t *heap = (uint8_t *)malloc(16 * sizeof(uint8_t));
    if (heap != NULL)
    {
        memset(heap, 0xa5+s_variable2, 16);
        memcpy(s_array, heap, 16);
        s_variable0 = (uint32_t)heap;
        free(heap);
    }
}
```

### 番外一、几个小技巧  
　　又来到痞子衡番外时间了，细心的朋友看到上表有两处标蓝，是的没错，今天的番外内容就是标蓝的项目有关。  
#### 技巧1：运行于异构双核
　　目前嵌入式产品越来越复杂，对MCU的性能要求也越来越高，各大ARM厂商也在不断推出性能越来越强劲的ARM MCU产品，超高主频，双核，四核MCU已经不鲜见了。对于其中的一些异构双核MCU产品，有时在开发中会有这样的需求：你有一份的middleware会被异构双核同时调用，而两个不同内核的指令集有可能是不一致的，怎么解决这个问题？有朋友会想到分别在每个核下面都编译一份binary放置于存储器不同位置，运行时各自指向对应的binary，这是一个办法，但比较浪费存储空间，且有可能会搞混淆导致误调用。有没有更好的方法？  
　　为了能做到Cortex-M软件重用，ARM公司在设计Cortex-M处理器时为其赋予了**处理器向下兼容**、**软件二进制向上兼容**特性。通俗的话来说就是在较低版本处理器上编译的代码可以在较高版本处理器上执行。所以解决方法就是选用异构双核里较低版本的内核在编译middleware，这样这份middleware可以同时被两个核调用。  

#### 技巧2：生成PIC代码
　　经常和bootloader打交道的朋友肯定知道，代码在经过链接阶段生成binary文件后，这个binary并不是可以放在任意位置的，必须放到linker文件指定的位置，如果位置没有放正确，可能会导致执行出错。究其原因，是因为编译器在汇编源代码时因为一些策略并不总是将所有function都汇编成位置无关代码。如果我们借助于IDE编译选项将middleware汇编成PIC代码，那么我们可以在工程中直接加入middleware的binary，然后借助linker的自定义section功能将其放置于任意某个位置，最后只要为这个middleware binary建立一个以binary首地址为基准的函数指针地址列表即可无障碍调用这个middleware。  

#### 技巧3：引用.c文件 
　　在项目开发中，我们在一个workspace下会创建多个project，常常是因为不同project需要包含不同的.c文件以完成不同的功能。那么能不能只创建一个project呢能实现不同功能呢？当然可以！通常情况下我们在.c文件中只会用#include "xx.h"语句来引用.h头文件，其实我们也同样可以引用.c文件，比如这样#include "xx.c"，只是需要注意尽量不要在.h文件中引用.c文件（除非该.h只会被一个.c文件include）。看到这里的朋友如果脑洞再大一点，你甚至可以做到工程里只需要添加一个.c文件，而其他.c文件全部由添加进工程的那个.c文件逐级（仅能单级）引用进工程。  

　　至此，嵌入式开发里的project文件痞子衡便介绍完毕了，掌声在哪里~~~ 

