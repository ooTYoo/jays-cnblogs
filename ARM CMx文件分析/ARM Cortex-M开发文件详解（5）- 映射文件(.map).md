----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式开发里的map文件**。  

　　第四节课里，痞子衡给大家介绍了第一种output文件-relocatable文件，本文继续给大家讲project生成的第二种output文件-map文件，map文件记录了很多重要的信息，这对于后续调试有很大帮助。  

　　文件关系：[linker文件]() + [project文件]() + [relocatable文件]() -> **map文件**  

　　痞子衡在第四节课[relocatable文件]()里分析object文件里的symbol list时讲到由于object文件并没有经过链接，所以所有symbol地址信息是无效的（待分配的），而map文件就是所有relocatable文件经过链接器统一链接后生成的记录链接信息的文件，map文件里可以查到所有symbol在存储器中具体分配地址。话不多说，让我们直接开启map文件分析之旅，以第三节课[project文件]()里demo工程为例。  

### 一、解析map文件
　　在IAR软件选项设置options->Linker->List里选中Generate linker map file，编译链接demo工程可在D:\myProject\bsp\builds\demo\Release\List路径下得到demo.map文件。让我们从头到尾逐一分析里面内容：  

#### 1.1 工程文件信息
　　map文件里第一部分信息记录的是工程文件相关信息，包括工程使用的软件版本，工程编译时间，工程文件目录，工程文件生成文件信息。  
```text
###############################################################################
#
# IAR ELF Linker V8.11.2.13589/W32 for ARM                12/Jan/2018  17:37:39
# Copyright 2007-2017 IAR Systems AB.
#
#    Output file  =  D:\myProject\bsp\builds\demo\Release\Exe\demo.elf
#    Map file     =  D:\myProject\bsp\builds\demo\Release\List\demo.map
#    Command line =  
#        -f C:\Users\Baoge\AppData\Local\Temp\EW5D86.tmp
#        (D:\myProject\bsp\builds\demo\Release\Obj\main.o
#        D:\myProject\bsp\builds\demo\Release\Obj\reset.o
#        D:\myProject\bsp\builds\demo\Release\Obj\startup.o
#        D:\myProject\bsp\builds\demo\Release\Obj\startup_MKL25Z4.o
#        D:\myProject\bsp\builds\demo\Release\Obj\system_MKL25Z4.o
#        D:\myProject\bsp\builds\demo\Release\Obj\task.o -o
#        D:\myProject\bsp\builds\demo\Release\Exe\demo.elf --map
#        D:\myProject\bsp\builds\demo\Release\List\demo.map --config
#        D:\myProject\bsp\builds\demo/../../linker/iar/KL25Z128xxx4_flash.icf
#        --entry Reset_Handler --inline --vfe --text_out locale)
#
###############################################################################
```

#### 1.2 系统库使用信息
　　map文件里第二部分信息记录的是工程系统库使用情况，由于task.c里调用了malloc()、free()等HEAP相关操作的API，所以自然我们在编译链接工程时会使用到HEAP相关系统库，这里告诉我们用的是DLib里的DLMalloc，而DLMalloc有很多种不同的HEAP实现策略，我们可在options->General Options->Library Option 2->Heap selection指定具体策略，由于demo工程选的是Automatic，也就是让IDE自动选择，这里告诉我们最终用的策略是advanced heap。  
```text
*******************************************************************************
*** RUNTIME MODEL ATTRIBUTES
***

CppFlavor       = *
__Heap_Handler  = DLMalloc
__SystemLibrary = DLib
__dlib_version  = 6


*******************************************************************************
*** HEAP SELECTION
***

The advanced heap was selected because the application calls memory
allocation functions outside of system library functions, and there
are calls to deallocation functions in the application.
```

#### 1.3 各object中Section放置信息
　　从map文件第三部分开始，就进入非常有用的信息环节了。第一个重要信息就是section放置信息。我们在第四节课[relocatable文件]()里分析过单个relocatable文件task.o，task.o里各个基本section都有，但是都并没有分配有效地址，而这里列出了所有relocatable文件统一存储和地址分配信息，从这里我们可以看到，链接器在整合各section的时候，都是以object文件为单位的，这意味着同一个object文件里的同一个section里的对象（变量/函数）在存储空间里的位置也是靠在一起的。  
　　另外一个有意思的信息是在第二节课[linker文件]()里，我们一共有四句block放置语句，在这里section也被分成了四个block：A0,P1,P2,P3。IDE给每个block重命名了，这些重命名的信息将会在第六节课[executable文件]()里被提到。  
```text
*******************************************************************************
*** PLACEMENT SUMMARY
***

define block Vectors with alignment = 256 { ro section .intvec };
"A0":  place at start of [0x00000000-0x0001ffff] { block Vectors };
define block CodeRelocate { section .textrw_init };
define block ApplicationFlash { ro, block CodeRelocate };
"P1":  place in [from 0x00000000 to 0x0001ffff] { block ApplicationFlash };
define block CodeRelocateRam { section .textrw };
define block HEAP with size = 1K, alignment = 8 { };
define block ApplicationRam { rw, block CodeRelocateRam, block HEAP };
"P2":  place in [from 0x10002000 to 0x1000ffff] { block ApplicationRam };
define block CSTACK with size = 8K, alignment = 8 { };
"P3":  place in [from 0x10000000 to 0x10001fff] { block CSTACK };
initialize manually with packing = copy, complex ranges { section .data };
initialize manually with packing = copy, complex ranges { section .textrw };

  Section                Kind        Address    Size  Object
  -------                ----        -------    ----  ------
"A0":                                           0x40
  Vectors                         0x00000000    0x40  <Block>
    .intvec              ro code  0x00000000    0x40  startup_MKL25Z4.o [1]
                                - 0x00000040    0x40

"P1":                                         0x1a3c
  ApplicationFlash                0x00000040  0x1a3c  <Block>
    .noinit              ro code  0x00000040    0x58  reset.o [1]
    .rodata              const    0x00000098     0x4  main.o [1]
    Veneer               ro code  0x0000009c    0x10  - Linker created -
    .text                ro code  0x000000ac    0x20  main.o [1]
    .text                ro code  0x000000cc    0x58  task.o [1]
    .text                ro code  0x00000124  0x16f8  dlmalloc.o [3]
    .text                ro code  0x0000181c    0x50  ABImemset.o [4]
    .text                ro code  0x0000186c    0x5c  ABImemcpy.o [4]
    .text                ro code  0x000018c8     0x8  heaptramp0.o [3]
    .text                ro code  0x000018d0     0xa  abort.o [3]
    .text                ro code  0x000018da     0x2  startup_MKL25Z4.o [1]
    .text                ro code  0x000018dc    0x2c  xgetmemchunk.o [3]
    .text                ro code  0x00001908     0xc  XXexit.o [4]
    .text                ro code  0x00001914    0x90  startup.o [1]
    .text                ro code  0x000019a4     0xc  system_MKL25Z4.o [1]
    .text                ro code  0x000019b0    0x1a  cmain.o [4]
    .text                ro code  0x000019ca     0x2  startup_MKL25Z4.o [1]
    .text                ro code  0x000019cc    0x28  data_init.o [4]
    .text                ro code  0x000019f4     0x8  exit.o [3]
    .text                ro code  0x000019fc     0xa  cexit.o [4]
    .text                ro code  0x00001a06     0x2  startup_MKL25Z4.o [1]
    CodeRelocate                  0x00001a08    0x10  <Block>
      Initializer bytes  const    0x00001a08    0x10  <for CodeRelocateRam-1>
    .data_init                    0x00001a18     0x4  <Block>
      Initializer bytes  const    0x00001a18     0x4  <for .data-1>
    .text                ro code  0x00001a1c     0x2  startup_MKL25Z4.o [1]
    .text                ro code  0x00001a1e     0x2  startup_MKL25Z4.o [1]
    .text                ro code  0x00001a20     0xc  cstartup_M.o [4]
    .text                ro code  0x00001a2c    0x40  zero_init3.o [4]
    .iar.init_table      const    0x00001a6c    0x10  - Linker created -
    .rodata              const    0x00001a7c     0x0  zero_init3.o [4]
                                - 0x00001a7c  0x1a3c

"P3":                                         0x2000
  CSTACK                          0x10000000  0x2000  <Block>
    CSTACK               uninit   0x10000000  0x2000  <Block tail>
                                - 0x10002000  0x2000

"P2":                                          0x620
  ApplicationRam                  0x10002000   0x620  <Block>
    CodeRelocateRam               0x10002000    0x10  <Block>
      CodeRelocateRam-1           0x10002000    0x10  <Init block>
        .textrw          inited   0x10002000    0x10  task.o [1]
    .data                         0x10002010     0x4  <Block>
      .data-1                     0x10002010     0x4  <Init block>
        .data            inited   0x10002010     0x4  task.o [1]
    .bss                          0x10002014   0x208  <Block>
      .bss               zero     0x10002014     0x4  task.o [1]
      .bss               zero     0x10002018    0x10  task.o [1]
      .bss               zero     0x10002028    0x18  dlmalloc.o [3]
      .bss               zero     0x10002040   0x1d8  dlmalloc.o [3]
      .bss               zero     0x10002218     0x4  xgetmemchunk.o [3]
    .noinit              uninit   0x1000221c     0x4  task.o [1]
    HEAP                          0x10002220   0x400  <Block>
      HEAP               uninit   0x10002220   0x400  <Block tail>
                                - 0x10002620   0x620
```

#### 1.4 系统初始化表信息
　　map文件第四部分列出了经由系统初始化的表，这里只有bss段（即代码中所有仅定义但没有赋初值的全局变量），由于SRAM中数据存有一定不确定性，所以系统必须要在启动时将bss段内所有数据全部清零，以保证程序能正常运行。  
```text
*******************************************************************************
*** INIT TABLE
***

          Address     Size
          -------     ----
Zero (__iar_zero_init3)
    1 destination range, total size 0x208:
          0x10002014  0x208
```

#### 1.5 各object文件所占存储资源信息
　　map文件第五部分会列出各object文件所占存储资源具体信息，有了这部分信息，我们便知道工程具体是哪个object文件（功能模块）占用资源最多，如果有代码size方面优化的需求，可以选择占用资源较多的object文件里的代码进行针对性地优化。  
```text
*******************************************************************************
*** MODULE SUMMARY
***

    Module             ro code  rw code  ro data  rw data
    ------             -------  -------  -------  -------
D:\myProject\bsp\builds\demo\Release\Obj: [1]
    main.o                  32                 4
    reset.o                 88
    startup.o              144
    startup_MKL25Z4.o       74
    system_MKL25Z4.o        12
    task.o                  88       16       20       28
    -----------------------------------------------------
    Total:                 438       16       24       28

command line: [2]
    -----------------------------------------------------
    Total:

dl6M_tln.a: [3]
    abort.o                 10
    dlmalloc.o           5 880                        496
    exit.o                   8
    heaptramp0.o             8
    xgetmemchunk.o          44                          4
    -----------------------------------------------------
    Total:               5 950                        500

rt6M_tl.a: [4]
    ABImemcpy.o             92
    ABImemset.o             80
    XXexit.o                12
    cexit.o                 10
    cmain.o                 26
    cstartup_M.o            12
    data_init.o             40
    zero_init3.o            64
    -----------------------------------------------------
    Total:                 336

    Linker created          16                16    9 216
---------------------------------------------------------
    Grand Total:         6 740       16       40    9 744
```

#### 1.6 各object具体分配信息
　　map文件第六部分记录的是各object文件里的具体对象（变量，函数等）在存储空间里的具体分配，这里的信息对于调试来说非常重要。平时调试时我们除了单步执行、打断点之外，还会配合看内存的实时情况，有时候因为编译器优化的原因，从代码角度看不出逻辑问题（比如我们给变量s_variable0赋值为1），但是内存里（0x10002014）却并没有被更新为1，这时候工程肯定是有问题的，定位到了具体问题，然后我们再考虑解决问题的方法。  
```text
*******************************************************************************
*** ENTRY LIST
***

Entry                      Address   Size  Type      Object
-----                      -------   ----  ----      ------
.bss$$Base              0x10002014          --   Gb  - Linker created -
.bss$$Limit             0x1000221c          --   Gb  - Linker created -
.data$$Base             0x10002010          --   Gb  - Linker created -
.data$$Limit            0x10002014          --   Gb  - Linker created -
.data_init$$Base        0x00001a18          --   Gb  - Linker created -
.data_init$$Limit       0x00001a1c          --   Gb  - Linker created -
.iar.init_table$$Base   0x00001a6c          --   Gb  - Linker created -
.iar.init_table$$Limit  0x00001a7c          --   Gb  - Linker created -
?main                   0x000019b1         Code  Gb  cmain.o [4]
ApplicationFlash$$Base  0x00000040          --   Gb  - Linker created -
ApplicationFlash$$Limit
                        0x00001a7c          --   Gb  - Linker created -
ApplicationRam$$Base    0x10002000          --   Gb  - Linker created -
ApplicationRam$$Limit   0x10002620          --   Gb  - Linker created -
CSTACK$$Base            0x10000000          --   Gb  - Linker created -
CSTACK$$Limit           0x10002000          --   Gb  - Linker created -
CodeRelocate$$Base      0x00001a08          --   Gb  - Linker created -
CodeRelocate$$Limit     0x00001a18          --   Gb  - Linker created -
CodeRelocateRam$$Base   0x10002000          --   Gb  - Linker created -
CodeRelocateRam$$Limit  0x10002010          --   Gb  - Linker created -
HEAP$$Base              0x10002220          --   Gb  - Linker created -
HEAP$$Limit             0x10002620          --   Gb  - Linker created -
Region$$Table$$Base     0x00001a6c          --   Gb  - Linker created -
Region$$Table$$Limit    0x00001a7c          --   Gb  - Linker created -
Reset_Handler           0x00000041         Code  Gb  reset.o [1]
SystemInit              0x000019a5    0xc  Code  Gb  system_MKL25Z4.o [1]
Vectors$$Base           0x00000000          --   Gb  - Linker created -
Vectors$$Limit          0x00000040          --   Gb  - Linker created -
__Vectors_End           0x00000040         Data  Gb  startup_MKL25Z4.o [1]
__aeabi_memcpy          0x0000186d         Code  Gb  ABImemcpy.o [4]
__aeabi_memcpy4         0x00001895         Code  Wk  ABImemcpy.o [4]
__aeabi_memset          0x0000181d         Code  Gb  ABImemset.o [4]
__cmain                 0x000019b1         Code  Gb  cmain.o [4]
__data_GetMemChunk      0x000018dd   0x2c  Code  Gb  xgetmemchunk.o [3]
__data_GetMemChunk::start
                        0x10002218    0x4  Data  Lc  xgetmemchunk.o [3]
__exit                  0x00001909         Code  Gb  XXexit.o [4]
__iar_Memset4_word      0x0000183d         Code  Gb  ABImemset.o [4]
__iar_Memset_word       0x00001829         Code  Gb  ABImemset.o [4]
__iar_data_init3        0x000019cd   0x28  Code  Gb  data_init.o [4]
__iar_dlfree            0x00001271  0x5a4  Code  Gb  dlmalloc.o [3]
__iar_dlmalloc          0x00000f77  0x2f6  Code  Gb  dlmalloc.o [3]
__iar_program_start     0x00001a21         Code  Gb  cstartup_M.o [4]
__iar_systems$$module {Abs}
                        0x00000001         Data  Gb  command line/config [2]
__iar_zero_init3        0x00001a2d   0x40  Code  Gb  zero_init3.o [4]
__vector_table          0x00000000         Data  Gb  startup_MKL25Z4.o [1]
_call_main              0x000019bd         Code  Gb  cmain.o [4]
_exit                   0x000019fd         Code  Gb  cexit.o [4]
_gm_                    0x10002040  0x1d8  Data  Lc  dlmalloc.o [3]
_main                   0x000019c7         Code  Gb  cmain.o [4]
abort                   0x000018d1    0xa  Code  Gb  abort.o [3]
add_segment             0x00000539  0x208  Code  Lc  dlmalloc.o [3]
exit                    0x000019f5    0x8  Code  Gb  exit.o [3]
free                    0x000018c9    0x8  Code  Gb  heaptramp0.o [3]
heap_task               0x000000db   0x3c  Code  Gb  task.o [1]
init_data_bss           0x00001915   0x54  Code  Gb  startup.o [1]
init_interrupts         0x00001969   0x12  Code  Gb  startup.o [1]
init_mparams            0x00000145   0x2a  Code  Lc  dlmalloc.o [3]
init_top                0x0000016f   0x38  Code  Lc  dlmalloc.o [3]
main                    0x000000ad   0x20  Code  Gb  main.o [1]
mparams                 0x10002028   0x18  Data  Lc  dlmalloc.o [3]
n_variable1             0x1000221c    0x4  Data  Gb  task.o [1]
normal_task             0x000000cd    0xe  Code  Gb  task.o [1]
prepend_alloc           0x000001b5  0x384  Code  Lc  dlmalloc.o [3]
ram_task                0x10002001   0x10  Code  Gb  task.o [1]
s_array                 0x10002018   0x10  Data  Lc  task.o [1]
s_constant              0x00000098    0x4  Data  Gb  main.o [1]
s_variable0             0x10002014    0x4  Data  Lc  task.o [1]
s_variable2             0x10002010    0x4  Data  Lc  task.o [1]
segment_holding         0x00000125   0x20  Code  Lc  dlmalloc.o [3]
sys_alloc               0x00000749  0x16c  Code  Lc  dlmalloc.o [3]
sys_trim                0x000008b5   0x6a  Code  Lc  dlmalloc.o [3]
tmalloc_large           0x00000931  0x3fe  Code  Lc  dlmalloc.o [3]
tmalloc_small           0x00000d35  0x242  Code  Lc  dlmalloc.o [3]

[1] = D:\myProject\bsp\builds\demo\Release\Obj
[2] = command line
[3] = dl6M_tln.a
[4] = rt6M_tl.a
```

#### 1.7 image占用存储资源信息
　　map文件第七部分会给出整个工程占用存储资源情况的总结，这里我们可以看到工程占用ROM资源6780bytes，RAM资源9760bytes，所以我们在选择芯片时必须保证ROM(FLASH),RAM要大于工程所需。  
```text
  6 740 bytes of readonly  code memory
     16 bytes of readwrite code memory
     40 bytes of readonly  data memory
  9 744 bytes of readwrite data memory
```

### 二、代码对象与section的关系
　　痞子衡在第二节课[linker文件]()里的讲过section的概念，并且列出了IAR系统里默认的各section的含义。经过上面对map文件的分析，现在让我们直接用demo工程里的main.c和task.c源文件来实例分析section：  

<table>
    <tr>
        <th>Section</th>
        <th>Description</th>
        <th>Region</th>
        <th>Object</th>
    </tr>
    <tr>
        <td rowspan="2">.bss</td>
        <td rowspan="2">未赋初值的全局/静态变量</td>
        <td rowspan="2">RAM(0x10002014 - 0x1000221b)</td>
        <td>s_variable0 (0x10002014 - 0x10002017)</td>
    </tr>
    <tr>
        <td>s_array[16] (0x10002018 - 0x10002027)</td>
    </tr>
    <tr>
        <td>CSTACK</td>
        <td>栈：函数调用返回地址、函数传递实参、局部变量</td>
        <td>RAM(0x10000000 - 0x10001fff)</td>
        <td>normal_task/ram_task/heap_task地址、l_variable、*heap</td>
    </tr>
    <tr>
        <td>.data</td>
        <td>赋初值的全局/静态变量</td>
        <td>RAM(0x10002010 - 0x10002013)</td>
        <td>s_variable2</td>
    </tr>
    <tr>
        <td>.data_init</td>
        <td>赋初值的全局/静态变量的初值</td>
        <td>ROM(0x00001a18 - 0x00001a1b)</td>
        <td>0x5a(s_variable2)</td>
    </tr>
    <tr>
        <td>HEAP</td>
        <td>堆：动态内存分配</td>
        <td>RAM(0x10002220 - 0x1000261f)</td>
        <td>*heap = (uint8_t *)malloc(16 * sizeof(uint8_t))</td>
    </tr>
    <tr>
        <td>.intvec</td>
        <td>中断向量表</td>
        <td>ROM(0x00000000 - 0x0000003f)</td>
        <td>startup_MKL25Z4.s里DCD指定的ISR表</td>
    </tr>
    <tr>
        <td>.noinit</td>
        <td>指明不初始化的全局/静态变量</td>
        <td>RAM(0x1000221c - 0x1000221f)</td>
        <td>n_variable1</td>
    </tr>
    <tr>
        <td>.rodata</td>
        <td>常量</td>
        <td>ROM(0x00000098 - 0x0000009b)</td>
        <td>s_constant</td>
    </tr>
    <tr>
        <td rowspan="3">.text</td>
        <td rowspan="3">ROM中执行的函数代码</td>
        <td rowspan="3">ROM(0x000000ac - 0x00001a07)</td>
        <td>main函数体 (0x000000ad - 0x000000cc)</td>
    </tr>
    <tr>
        <td>normal_task函数体 (0x000000cd - 0x000000da)</td>
    </tr>
    <tr>
        <td>heap_task函数体 (0x000000db - 0x00000116)</td>
    </tr>
    <tr>
        <td>.textrw</td>
        <td>RAM中执行的函数代码</td>
        <td>RAM(0x10002000 - 0x1000200f)</td>
        <td>ram_task函数体</td>
    </tr>
    <tr>
        <td>.textrw_init</td>
        <td>RAM中执行的函数代码的数据</td>
        <td>ROM(0x00001a08 - 0x00001a17)</td>
        <td>ram_task函数体</td>
    </tr>
</table>

　　至此，嵌入式开发里的map文件痞子衡便介绍完毕了，掌声在哪里~~~ 


