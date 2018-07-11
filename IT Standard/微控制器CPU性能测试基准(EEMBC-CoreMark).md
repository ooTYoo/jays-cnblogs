----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**微控制器CPU性能测试基准CoreMark**。  

　　在嵌入式系统行业用于评价CPU性能指标的标准主要有三种：Dhrystone、MIPS、CoreMark，其中CoreMark是一种新兴流行的嵌入式系统处理器测试基准，被认为是比Dhrystone和MIPS更具有实际价值的测试基准。今天痞子衡就和大家详细聊一聊CoreMark。  

### 一、EEMBC协会
　　在讲CoreMark之前，必须要先提EEMBC（Embedded Microprocessor Benchmark Consortium）即嵌入式微处理器基准评测协会，它是一个非盈利性组织，该组织目前为止（2018.03）共发布了46个性能测试基准，有了这些性能基准参考，嵌入式设计人员可以快速有效地选择处理器。  
　　EEMBC测试是基于每秒钟算法执行的次数和编译代码大小的综合统计结果。众所周知，编译器选项会对代码大小和执行效率会产生巨大的影响，所以每种测试必须包括足够多的编译器信息并设置不同的优化项。  
　　EEMBC发展势头很好，其很有可能发展成为嵌入式系统开发人员进行处理器和编译器性能比较的工业标准。关于EEMBC的更多介绍可移步它的官方网站 [http://www.eembc.org/](http://www.eembc.org/)  

### 二、CoreMark标准
　　CoreMark是由EEMBC的Shay Gla-On于2009年提出的一项基准测试程序，其主要目标是测试处理器核心性能。  
　　CoreMark标准的测试方法很简单，就是在某配置参数组合下单位时间内跑了多少次CoreMark程序，其指标单位为**CoreMark/MHz**。CoreMark数字越高，意味着性能更高。  

#### 2.1 获取程序
　　CoreMark程序的目前（2018.03）最新版本是1.01。  
> 核心程序下载 http://www.eembc.org/coremark/download.php  
> 平台移植示例 http://www.eembc.org/coremark/ports.php  

　　核心程序包下载后，在\coremark_v1.0\readme.txt里可见基本介绍，在\coremark_v1.0\docs\Coremark-requirements.doc里可见设计需求。详细文件目录如下：  
```text
\coremark_v1.0
              \barebones            --移植到裸机下需要修改的文件
			            \core_portme.h    -- 移植平台工程具体配置信息
						\core_portme.c    -- 计时以及板级初始化实现  
						\cvt.c
						\ee_printf.c      -- 打印函数串口发送实现
              \cygwin               --移植到cygwin下需要修改的文件
              \linux                --移植到linux下需要修改的文件
              \linux64              --移植到linux64下需要修改的文件
              \simple               --基本移植需要修改的文件

              core_main.c           --主程序入口
              core_state.c          --状态机控制子程序
              core_list_join.c      --列表操作子程序
              core_matrix.c         --矩阵运算子程序
              core_util.c           --CRC计算子程序
              coremark.h            --工程配置与数据结构定义

              \docs                 --设计文档
              coremark.md5
              LICENSE.txt
              Makefile
              readme.txt            --基本介绍
              release_notes.txt     --版本说明
```

　　如果是移植到ARM Cortex-M平台下裸系统运行，一般只需要修改\barebones目录下的文件即可（仅需改动三个函数portable_init()、barebones_clock()、uart_send_char()以及core_portme.h中若干宏定义），其余代码文件不需要修改。关于\barebones下的文件修改，EEMBC上有如下4个示例平台可参考：  

![](http://odox9r8vg.bkt.clouddn.com/image/cnblogs/EEMBC-CoreMark-Ports.PNG)  

#### 2.2 配置参数
　　前面讲到做平台移植时除了必须要改动3个函数外，还需要设置core_portme.h中若干宏定义，这些宏定义即为配置参数，需要根据要移植到的具体平台的属性而定。一共如下14个宏：  

<table><tbody>
    <tr>
        <th style="width: 200px;">宏</th>
        <th style="width: 600px;">解释</th>
        <th style="width: 150px;">示例</th>
    </tr>
    <tr>
        <td>HAS_FLOAT</td>
        <td>Define to 1 if the platform supports floating point</td>
        <td>1</td>
    </tr>
    <tr>
        <td>HAS_TIME_H</td>
        <td>Define to 1 if platform has the time.h header file and implementation of functions thereof</td>
        <td>0</td>
    </tr>
    <tr>
        <td>USE_CLOCK</td>
        <td>Define to 1 if the platform has stdio.h.</td>
        <td>0</td>
    </tr>
    <tr>
        <td>HAS_STDIO</td>
        <td>Define to 1 if the platform has stdio.h and implements the printf function.</td>
        <td>0</td>
    </tr>
    <tr>
        <td>HAS_PRINTF</td>
        <td>Define to 1 if the platform has stdio.h and implements the printf function.</td>
        <td>0</td>
    </tr>
    <tr>
        <td>COMPILER_VERSION</td>
        <td>Please put compiler version here (e.g. gcc 4.1)</td>
        <td></td>
    </tr>
    <tr>
        <td>COMPILER_FLAGS</td>
        <td>Please put compiler flags here (e.g. -o3)</td>
        <td></td>
    </tr>
    <tr>
        <td>MEM_LOCATION</td>
        <td>Please put the mem location of code execution here (e.g STACK)</td>
        <td></td>
    </tr>
    <tr>
        <td>CORETIMETYPE</td>
        <td>Define type of return from the timing functions.</td>
        <td>ee_u32</td>
    </tr>
    <tr>
        <td>SEED_METHOD</td>
        <td>Defines method to get seed values that cannot be computed at compile time.</td>
        <td>SEED_VOLATILE</td>
    </tr>
    <tr>
        <td>MEM_METHOD</td>
        <td>Defines method to get a block of memry.</td>
        <td>MEM_STATIC</td>
    </tr>
    <tr>
        <td>MULTITHREAD</td>
        <td>Define for parallel execution</td>
        <td>1</td>
    </tr>
    <tr>
        <td>MAIN_HAS_NOARGC</td>
        <td>Needed if platform does not support getting arguments to main.</td>
        <td>1</td>
    </tr>
    <tr>
        <td>MAIN_HAS_NORETURN</td>
        <td>Needed if platform does not support returning a value from main.</td>
        <td>0</td>
    </tr>
</table>

　　细心的朋友应该能注意到core_portme.h文件的最后有如下条件编译，实际上CoreMark主程序的运行有3种模式可选，即PROFILE_RUN（原型模式）/PERFORMANCE_RUN（性能模式）/VALIDATION_RUN（验证模式）  

```c
#if !defined(PROFILE_RUN) && !defined(PERFORMANCE_RUN) && !defined(VALIDATION_RUN)
#if (TOTAL_DATA_SIZE==1200)
#define PROFILE_RUN 1
#elif (TOTAL_DATA_SIZE==2000)
#define PERFORMANCE_RUN 1
#else
#define VALIDATION_RUN 1
#endif
#endif
```

　　而在coremark.h文件的最开始就定义了缺省的TOTAL_DATA_SIZE的值为2000，即CoreMark程序默认跑在PERFORMANCE_RUN（性能模式）下。如果你想修改运行模式，需要在编译器预编译选项里自定义TOTAL_DATA_SIZE。  

```c
/* Configuration: TOTAL_DATA_SIZE
	Define total size for data algorithms will operate on
*/
#ifndef TOTAL_DATA_SIZE 
#define TOTAL_DATA_SIZE 2*1000
#endif
```

#### 2.3 程序解析
　　CoreMark程序使用C语言写成，包含如下四类运算法则：数学矩阵操作（普通矩阵运算）、列举（寻找并排序）、状态机（用来确定输入流中是否包含有效数字）、CRC（循环冗余校验），都是在真实的嵌入式应用中很常见的操作，这也是CoreMark比其他测试标准更有实际价值的原因所在。  
> a. Matrix multiply (allow for use of MAC operations, common math use)  
> b. Linked list search/sort/read (common pointer use)  
> c. State machine (common use of data dependent branches)  
> d. CRC (common in embedded)  

　　让我们尝试分析CoreMark主函数入口main（以2.2节中配置示例值为例）：  
```c
/* Function: main
	Main entry routine for the benchmark.
	This function is responsible for the following steps:

	1 - Initialize input seeds from a source that cannot be determined at compile time.
	2 - Initialize memory block for use.
	3 - Run and time the benchmark.
	4 - Report results, testing the validity of the output if the seeds are known.

	Arguments:
	1 - first seed  : Any value
	2 - second seed : Must be identical to first for iterations to be identical
	3 - third seed  : Any value, should be at least an order of magnitude less then the input size, but bigger then 32.
	4 - Iterations  : Special, if set to 0, iterations will be automatically determined such that the benchmark will run between 10 to 100 secs

*/
MAIN_RETURN_TYPE main(void) {
	int argc=0;
	char *argv[1];
	ee_u16 i,j=0,num_algorithms=0;
	ee_s16 known_id=-1,total_errors=0;
	ee_u16 seedcrc=0;
	CORE_TICKS total_time;
	core_results results[MULTITHREAD];

	// 系统板级初始化
	portable_init(&(results[0].port), &argc, argv);

	// ...
	// 设置PERFORMANCE_RUN的初始参数
	results[0].seed1=get_seed(1);  //0x0
	results[0].seed2=get_seed(2);  //0x0
	results[0].seed3=get_seed(3);  //0x66
	results[0].iterations=get_seed_32(4);  //ITERATIONS
	// execs参数为需要跑的算法使能位
	results[0].execs=get_seed_32(5);       //0x0
	if (results[0].execs==0) { /* if not supplied, execute all algorithms */
		results[0].execs=ALL_ALGORITHMS_MASK;
	}
	// ...

	results[0].memblock[0]=(void *)static_memblk;
	results[0].size=TOTAL_DATA_SIZE;
	results[0].err=0;

	/* Data init */ 
	/* Find out how space much we have based on number of algorithms */
	// ...

	// 各算法子程序初始化（共LIST, MATRIX, STATE三种）
	for (i=0 ; i<MULTITHREAD; i++) {
		if (results[i].execs & ID_LIST) {
			results[i].list=core_list_init(results[0].size,results[i].memblock[1],results[i].seed1);
		}
		if (results[i].execs & ID_MATRIX) {
			core_init_matrix(results[0].size, results[i].memblock[2], (ee_s32)results[i].seed1 | (((ee_s32)results[i].seed2) << 16), &(results[i].mat) );
		}
		if (results[i].execs & ID_STATE) {
			core_init_state(results[0].size,results[i].seed1,results[i].memblock[3]);
		}
	}

	/* automatically determine number of iterations if not set */
	// ...

	// 开始跑CoreMark程序且记录累计消耗时间
	start_time();
	iterate(&results[0]);
	stop_time();
	total_time=get_time();

    // ...
	// 最终信息的打印
    // ...
	if (total_errors==0) {
		ee_printf("Correct operation validated. See readme.txt for run and reporting rules.\n");
		if (known_id==3) {
			ee_printf("CoreMark 1.0 : %f / %s %s",default_num_contexts*results[0].iterations/time_in_secs(total_time),COMPILER_VERSION,COMPILER_FLAGS);
			ee_printf("\n");
		}
	}
    // ...

	/* And last call any target specific code for finalizing */
	portable_fini(&(results[0].port));

	return MAIN_RETURN_VAL;	
}
```

#### 2.4 结果格式
　　当移植好CoreMark程序后，便可以开始跑起来了，在跑程序的时候，EEMBC同时制定了必须要遵守规则（不遵守的话，跑分结果不被EEMBC所认可），详见 [https://www.eembc.org/coremark/CoreMarkRunRules.pdf](https://www.eembc.org/coremark/CoreMarkRunRules.pdf)。  
　　当得到跑分结果后可将结果提交到EEMBC网站上，跑分结果需按如下标准格式进行提交：  

```text
	CoreMark 1.0 : N / C [/ P] [/ M]

	N - Number of iterations per second with seeds 0,0,0x66,size=2000)
	C - Compiler version and flags
	P - Parameters such as data and code allocation specifics
		- This parameter *may* be omitted if all data was allocated on the heap in RAM.
		- This parameter *may not* be omitted when reporting CoreMark/MHz
	M - Type of parallel execution (if used) and number of contexts
		This parameter may be omitted if parallel execution was not used.

	e.g.
	> CoreMark 1.0 : 128 / GCC 4.1.2 -O2 -fprofile-use / Heap in TCRAM / FORK:2
	or
	> CoreMark 1.0 : 1400 / GCC 3.4 -O4

	If reporting scaling results, the results must be reported as follows:

	CoreMark/MHz 1.0 : N / C / P [/ M]

	P - When reporting scaling results, memory parameter must also indicate memory frequency:core frequency ratio.
		- If the core has cache and cache frequency to core frequency ratio is configurable, that must also be included.

	e.g.
	> CoreMark/MHz 1.0 : 1.47 / GCC 4.1.2 -O2 / DDR3(Heap) 30:1 Memory 1:1 Cache
```

　　如果移植的CoreMark能够正确运行，你应该可以看到串口会打印出类似如下格式的信息，上述要求的CoreMark标准结果就在打印信息的最后。  

```text
2K performance run parameters for coremark. (Run type)
CoreMark Size       : 666                   (Buffer size)
Total ticks         : 25875                 (platform dependent value)
Total time (secs)   : 25.875000             (actual time in seconds)
Iterations/Sec      : 3864.734300           (Performance value to report)
Iterations          : 100000                (number of iterations used)
Compiler version    : GCC3.4.4              (Compiler and version)
Compiler flags      : -O2                   (Compiler and linker flags)
Memory location     : Code in flash, data in on chip RAM
seedcrc             : 0xe9f5                (identifier for the input seeds)
[0]crclist          : 0xe714                (validation for list part)
[0]crcmatrix        : 0x1fd7                (validation for matrix part)
[0]crcstate         : 0x8e3a                (validation for state part)
[0]crcfinal         : 0x33ff                (iteration dependent output)
Correct operation validated. See readme.txt for run and reporting rules.  (*Only when run is successful*)
CoreMark 1.0 : 6508.490622 / GCC3.4.4 -O2 / Heap                          (*Only on a successful performance run*)
```

#### 2.5 跑分榜
　　截止到目前（2018.03），EEMBC网站共记录535款微控制器的CoreMark跑分结果（注意并不是所有跑分结果都经过EEMBC核实），所有跑分结果可在这里查询 [http://www.eembc.org/coremark/index.php](http://www.eembc.org/coremark/index.php)，下图是跑分榜部分结果（按提交日期排序）。如果是设计人员根据性能选型的话，可以选按得分高低排序。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/EEMBC-CoreMark-Scores.PNG" style="zoom:80%" />  

#### 2.6 结果示例
　　在上节介绍的跑分榜里可点击微控制器型号查看具体结果，也可选择多个微控制器进行结果对比。最近两家ARM Cortex-M微控制器知名厂商恩智浦半导体和意法半导体在高性能微控制器上正一决雌雄，恩智浦推出的i.MX RT1050和意法半导体推出的STM32H743均是基于Cortex-M7内核，且都在2017.10实现初版量产，我们且来比比看这两款微控制器：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/EEMBC-CoreMark-Scores-RT1050vsSTM32H743.PNG" style="zoom:80%" />  

　　从对比结果来看，i.MX RT1050在性能上完爆STM32H743，其3036的总得分在Cortex-M微控制器里独孤求败，这个跑分结果虽未经过EEMBC认证，但与恩智浦官方给的数据3020基本吻合。  
　　关于i.MX RT系列微控制器简介可详见我的另一篇文章 [飞思卡尔i.MX RT系列微控制器介绍篇（1）- 概览](http://www.cnblogs.com/henjay724/p/8556171.html)，对于i.MX RT1050跑分结果的验证与复现可详见我的文章 [飞思卡尔i.MX RT系列微控制器介绍篇（2）- 性能CoreMark](http://www.cnblogs.com/henjay724/p/8727199.html)。  

　　至此，微控制器CPU性能测试基准CoreMark痞子衡便介绍完毕了，掌声在哪里~~~
