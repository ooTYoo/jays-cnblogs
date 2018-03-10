----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式开发里的source文件**。  

　　众所周知，嵌入式开发属于偏底层的开发，主要编程语言是C和汇编。所以本文要讲的source文件主要指的就是c文件和汇编文件。  
　　尽管在平常开发中，我们都只会关注自己创建的.c/.h/.s源文件，但实际上我们不知不觉中也跟很多不是我们创建的源文件在打交道，那么问题来了，一个完整的嵌入式工程（以基于ARM Cortex-M控制器的工程为例）到底会包含哪些source文件呢？  
　　现在就到了痞子衡的show time了，痞子衡将这些文件按来源分为五类十种，下面痞子衡按类别逐一分析这些文件：  

### 第一类：Provided by Committee
　　第一类文件由C标准委员会提供，该类文件伴随着标准的发布而逐渐壮大。该类文件主要就是一种，即C标准库。  
**1. C standard Library**  
　　大家都知道C语言是有标准的，常见的C标准有ANSI C（C89）、C99、C11，而C标准函数库（C Standard library）就是所有符合C标准的头文件的集合，以及常用的函数库实现程序。C标准库由Committee制订发布，通常会被包含在IDE里。列举一些常见文件和函数如下，是不是觉得似曾相识？  
```text
/* 常用文件 */ assert.h，stdio.h，stddef.h，stdint.h，string.h ...
/* 常用定义 */ bool，NULL，uint8_t，uint16_t，uint32_t...
/* 常用函数 */ assert()，printf()，memset()，memcpy()...
```

### 第二类：Provided by IDE(Compiler)
　　第二类文件由IDE提供，C语言是编译型语言，需要编译器将C程序汇编成机器码，所有便有了一些跟编译器特性相关的函数库。  
**2. Compiler Library**  
　　我们在开发嵌入式应用时需要借助集成开发环境（IDE），常见的IDE有GCC(GNUC)，Keil MDK(ARMCC)，IAR EWARM(ICCARM)，这些IDE都有配套的C编译器，这些编译器是各有特色的，为了充分展示各编译器特色，配套的函数库便应运而生。  
　　编译器函数库是因IDE而异的，此处仅讲一个例子以供参考，需要了解更多需查看各IDE手册。  
　　以IAR EWARM里的DLib\_Product\_string.h文件为例，该文件中重定义了memcpy的实现:  
```C
#define _DLIB_STRING_SKIP_INLINE_MEMCPY
#pragma inline=forced_no_body
__EFF_NENR1NW2R1 __ATTRIBUTES void * memcpy(void * _D, const void * _S, size_t _N)
{
  __aeabi_memcpy(_D, _S, _N);
  return _D;
}
```

### 第三类：Provided by ARM
　　第三类文件由ARM提供，嵌入式程序的执行靠的是控制器内核（此处指的内核便是ARM内核），ARM公司在设计内核时，提供了一些内核模块的接口，开发者可以通过这些接口访问内核资源，CMSIS header里就是这些内核模块资源的接口。  
**3. CMSIS header**  
　　完整的CMSIS header目录应该是下面这个样子，而必须要关注的只有\CMSIS\Include下面的core_cmx.h文件  
```text
\CMSIS
     \Core      
     \DAP            /* ARM debugger实现 */
     \Driver         /* ARM统一的常用外设driver API */
     \DSP_Lib        /* ARM优化实现的DSP Lib */
     \Include        /* ARM内核资源接口 */
            \arm_xx.h
            \cmsis_xx.h
            \core_cmx.h
     \Lib            /* ARM优化实现的标准Lib */
     \Pack
     \RTOS           /* ARM推出的RTOS- RTX */
     \RTOS2
     \SVD
     \Utilities
```
　　core_cmx.h文件里定义了内核资源接口，里面最常用的三大模块是SCB，SysTick，NVIC，一个嵌入式开发的老手看到这些模块应该要向痞子衡挥手示意，来，让痞子衡看见你们的双手~~~  

### 第四类：Provided by Chip Producer
　　第四类文件是由ARM芯片生产商提供，我们在选型一个ARM芯片时，除了看ARM内核类型外，还得看芯片内部外设资源，是这些外设导致了ARM芯片差异，于是便有了各大ARM厂商争奇斗艳，比如NXP(Freescale), ST, Microchip(Atmel)，ARM厂商赋予了ARM芯片各种外设资源，同时也会提供这些外设资源的接口。该类别下文件有四种：  
**4. device.h**：芯片头文件，主要包含中断号定义（xx\_IRQn）、外设模块类型定义(xx\_Type) 、外设基地址定义（xx\_BASE）。  
```C
/////////////////////////////////////////////////////
// 中断号定义
typedef enum IRQn {
  NotAvail_IRQn                = -128,
  /* Core interrupts */
  NonMaskableInt_IRQn          = -14,
  HardFault_IRQn               = -13,
  ...
  SysTick_IRQn                 = -1,
  /* Device specific interrupts */
  WDT0_IRQn                = 0,
  ...
} IRQn_Type;
////////////////////////////////////////////////////
// 外设寄存器定义
typedef struct {
  __IO uint32_t MOD;
  ...
  __IO uint32_t WINDOW;
} WWDT_Type;
#define WWDT_WINDOW_WINDOW_MASK       (0xFFFFFFU)
#define WWDT_WINDOW_WINDOW_SHIFT      (0U)
#define WWDT_WINDOW_WINDOW(x)         (((uint32_t)(((uint32_t)(x)) << WWDT_WINDOW_WINDOW_SHIFT)) & WWDT_WINDOW_WINDOW_MASK)
////////////////////////////////////////////////////
// 外设基地址定义
#define WWDT0_BASE                    (0x5000E000u)
```
**5. startup\_device.s**：芯片中断向量表文件，主要包含中断向量表定义（DCD xx_Handler） ，以及各中断服务程序的弱定义（PUBWEAK）。 Note：该文件因编译器而异。  
```text
;;基于IAR的startup_device.s文件
        MODULE  ?cstartup
        ;; Forward declaration of sections.
        SECTION CSTACK:DATA:NOROOT(3)
        SECTION .intvec:CODE:NOROOT(2)
        PUBLIC  __vector_table
        PUBLIC  __Vectors_End
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; 中断向量表定义
        DATA
__vector_table
        DCD     sfe(CSTACK)
        DCD     Reset_Handler
        DCD     NMI_Handler
        DCD     HardFault_Handler
        ...
        DCD     SysTick_Handler
        ; External Interrupts
        DCD     WDT0_IRQHandler
        ...
__Vectors_End
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; 中断服务程序弱定义
        THUMB
        PUBWEAK WDT0_IRQHandler
        PUBWEAK WDT0_DriverIRQHandler
        SECTION .text:CODE:REORDER:NOROOT(2)
WDT0_IRQHandler
        LDR     R0, =WDT0_DriverIRQHandler
        BX      R0
WDT0_DriverIRQHandler
        B .
        END
```
**6. system\_device.c/h**：芯片系统初始化文件，主要包含全局变量SystemCoreClock定义（提供芯片内核默认工作频率）、SystemInit()函数定义（完成最基本的系统初始化，比如WDOG初始化，RAM使能等，这部分因芯片设计而异）。  
**7. device SDK Library**：官方提供的芯片外设SDK driver包文件，有了这个SDK包可以直接使用片内外设设计自己的应用，而不需要查看芯片手册里的外设模块寄存器去重写外设驱动。当然并不是每个厂商都有完善的SDK包，这取决于各厂商对软件服务的重视程度。  
```C
// 来自于NXP SDK的WWDT driver API
void WWDT_GetDefaultConfig(wwdt_config_t *config);
void WWDT_Init(WWDT_Type *base, const wwdt_config_t *config);
void WWDT_Deinit(WWDT_Type *base);
void WWDT_ClearStatusFlags(WWDT_Type *base, uint32_t mask);
void WWDT_Refresh(WWDT_Type *base);
```

### 第五类：Created by Developer
　　第五类文件是开发者自己创建，用于实现开发者自己的嵌入式应用，分为应用系统启动文件，应用系统初始化文件，应用文件。其中应用系统启动和初始化文件属于main函数之前的文件，一般可以通用，大部分开发者并不关心其具体内容，但是了解其过程可以加深对嵌入式系统结构的理解。  
**8. reset.s**： 应用系统复位启动文件，了解ARM原理的都知道，image前8个字节数据分别是芯片上电的初始SP, PC，其中PC指向的便是本文件里的Reset_Handler，这是芯片执行的第一个函数入口，该函数主要用于完成应用系统初始化工作，包含应用中断向量表重定向、调用芯片系统初始化、ARM系统寄存器rx清零、初始化应用程序各数据段、初始化ARM系统中断、跳转main函数。  
```text
// 一段经典的startup code
        SECTION .noinit : CODE
        THUMB
        import SystemInit
        import init_data_bss
        import main
        import CSTACK$$Limit
        import init_interrupts
        EXTERN __vector_table
        REQUIRE __vector_table
#define SCB_BASE            (0xE000ED00)
#define SCB_VTOR_OFFSET     (0x00000008)
        PUBLIC  Reset_Handler
        EXPORT  Reset_Handler
Reset_Handler
        // Mask interrupts
        cpsid   i
        // Set VTOR register in SCB first thing we do.
        ldr     r0,=__vector_table
        ldr     r1,=SCB_BASE
        str     r0,[r1, #SCB_VTOR_OFFSET]
        // Init the rest of the registers
        ldr     r2,=0
        ldr     r3,=0
        ldr     r4,=0
        ldr     r5,=0
        ldr     r6,=0
        ldr     r7,=0
        mov     r8,r7
        mov     r9,r7
        mov     r10,r7
        mov     r11,r7
        mov     r12,r7
        // Initialize the stack pointer
        ldr     r0,=CSTACK$$Limit
        mov     r13,r0
        // Call the CMSIS system init routine
        ldr     r0,=SystemInit
        blx     r0
        // Init .data and .bss sections
        ldr     r0,=init_data_bss
        blx     r0
        // Init interrupts
        ldr     r0,=init_interrupts
        blx     r0
        // Unmask interrupts
        cpsie   i
        // Set argc and argv to NULL before calling main().
        ldr     r0,=0
        ldr     r1,=0
        ldr     r2,=main
        blx     r2
__done
        B       __done
        END
```
**9. startup.c**：应用系统初始化文件，该文件里主要包含两个初始化函数，init\_data\_bss()、  init\_interrupts()，data, bss段数据的初始化是为了保证嵌入式系统中所有全局变量能有一个开发者指定的初值。由于data，bss段的位置是在链接阶段确定的，所以此处需要配合linker文件才能找到正确的data,bss位置，linker文件是因IDE而异的，所有本文件要想做到通用，必须增加各IDE条件编译，此处仅以IAR下的实现为例：  
```C
//基于IAR的startup.c文件
#if (defined(__ICCARM__))
#pragma section = ".intvec"
#pragma section = ".data"
#pragma section = ".data_init"
#pragma section = ".bss"
#pragma section = "CodeRelocate"
#pragma section = "CodeRelocateRam"
#endif
void init_data_bss(void)
{
#if defined(__ICCARM__)
    uint8_t *data_ram, *data_rom, *data_rom_end;
    uint8_t *bss_start, *bss_end;
    uint8_t *code_relocate_ram, *code_relocate, *code_relocate_end;
    uint32_t n;
// 初始化data段 .data section (initialized data section)
    data_ram = __section_begin(".data");
    data_rom = __section_begin(".data_init");
    data_rom_end = __section_end(".data_init");
    n = data_rom_end - data_rom;
    if (data_ram != data_rom)
    {
        while (n--)
        {
            *data_ram++ = *data_rom++;
        }
    }
// 初始化bss段 .bss section (zero-initialized data)
    bss_start = __section_begin(".bss");
    bss_end = __section_end(".bss");
    n = bss_end - bss_start;
    while (n--)
    {
        *bss_start++ = 0;
    }
// 初始化CodeRelocate段 （执行在RAM中的函数（由IAR指定的__ramfunc修饰的函数））.
    code_relocate_ram = __section_begin("CodeRelocateRam");
    code_relocate = __section_begin("CodeRelocate");
    code_relocate_end = __section_end("CodeRelocate");
    n = code_relocate_end - code_relocate;
    while (n--)
    {
        *code_relocate_ram++ = *code_relocate++;
    }
#endif
}
void init_interrupts(void)
{
    NVIC_ClearEnabledIRQs();
    NVIC_ClearAllPendingIRQs();
}
```
**10. application.c/h**： 应用文件，此处便是主函数以及各功能函数的集合了，嵌入式老司机们，请开始你的表演~~~  
```C
void taskn(void)
{
    // Your task code
}
int main(void)
{
    printf("hello world\r\n");
    taskn();
    ...
    return 0;
}
```

　　至此，嵌入式开发里的各种来源的source文件痞子衡便介绍完毕了，掌声在哪里~~~ 