----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式开发里的executable文件(elf)**。  

　　第四、五节课里，痞子衡已经给大家介绍了2种output文件，本文继续给大家讲project生成的另一种output文件-executable文件，也是特别重要的output文件。  

　　文件关系：[链接文件(.icf)](http://www.cnblogs.com/henjay724/p/8191908.html) + [工程文件(.ewp)](http://www.cnblogs.com/henjay724/p/8232585.html) + [可重定向文件(.o/.a)](http://www.cnblogs.com/henjay724/p/8276595.html) -> **可执行文件(.out/.elf)**  

　　仔细看过痞子衡之前课程的朋友肯定知道，痞子衡在第四节课[可重定向文件(.o/.a)](http://www.cnblogs.com/henjay724/p/8276595.html)里介绍的object文件在格式上其实跟本文要讲的elf文件是类似的，它们都属于ELF文件分支。只不是relocatable文件只是中间过渡文件，而本文要讲的elf却是标准的output文件，这个文件几乎包含了工程的所有信息，有了这个文件我们既可以在线调试工程，也可以将elf文件转换成image文件，直接下载image文件数据进芯片中脱机运行。今天痞子衡就为大家仔细分析elf文件。  

### 一、elf文件基础
　　ELF全称Executable and Linkable Format，可执行连接格式，ELF格式的文件最早用于存储Linux程序，后演变到ARM系统上存储ARM程序。ELF文件（目标文件）格式主要三种：  
> * **可重定向文件**：用来和其他的目标文件一起来创建一个可执行文件或者共享目标文件（也称object文件或者静态库文件，通常后缀为.o和.a的文件）。这个文件是用于编译和链接阶段。
> * **可执行文件**：用于生成应用image，载入存储器执行（后缀通常为.out或者.elf）。这个文件是用于加载执行阶段。
> * **共享目标文件**：用于和其他共享目标文件或者object文件一起生成可执行文件，或者和可执行文件一起创建应用image。（也称共享库文件，后缀为.so的文件）。这个文件既可用于编译和链接阶段，也可用于加载执行阶段。

　　我们在ARM开发中更多接触的是前两种格式，第一种格式前面系列文章[可重定向文件(.o/.a)](http://www.cnblogs.com/henjay724/p/8276595.html)已经介绍过，本文的主角是第二种格式-可执行文件。不管是哪种格式的ELF文件，其都可能包含如下三种基本索引表：  
> * **file header**：一般在文件的开始，描述了ELF文件的整体组织情况。
> * **program header**：告诉系统如何创建image，可执行文件必须具有program header，而可重定向文件则不需要。
> * **section header**：包含了描述文件section的信息，每个section都有一个header，每一个header给出诸如section名称、section大小等信息。可重定向文件必须包含section header。

　　既然知道了存在三种索引表，那么表的结构定义在哪里呢？github上的linux仓库里有具体定义，在elf.h头文件里。  

> Linux仓库：https://github.com/torvalds/linux.git  
> elf.h路径：\linux\include\uapi\linux\elf.h  

　　打开elf.h文件便可找到三个表的原型定义，鉴于目前的ARM Cortex-M都是32bit，所以此处仅列出32bit下的表的原型：Elf32\_Ehdr、Elf32\_Phdr、Elf32\_Shdr。  
```c
// file header
#define EI_NIDENT    16
typedef struct elf32_hdr{
  unsigned char	e_ident[EI_NIDENT];     /* Magic number and other info */
  Elf32_Half    e_type;                 /* Object file type */  
  Elf32_Half    e_machine;              /* Architecture */  
  Elf32_Word    e_version;              /* Object file version */  
  Elf32_Addr    e_entry;                /* Entry point virtual address */  
  Elf32_Off     e_phoff;                /* Program header table file offset */  
  Elf32_Off     e_shoff;                /* Section header table file offset */  
  Elf32_Word    e_flags;                /* Processor-specific flags */  
  Elf32_Half    e_ehsize;               /* ELF header size in bytes */  
  Elf32_Half    e_phentsize;            /* Program header table entry size */  
  Elf32_Half    e_phnum;                /* Program header table entry count */  
  Elf32_Half    e_shentsize;            /* Section header table entry size */  
  Elf32_Half    e_shnum;                /* Section header table entry count */  
  Elf32_Half    e_shstrndx;             /* Section header string table index */ 
} Elf32_Ehdr;

// program header
typedef struct elf32_phdr{
  Elf32_Word    p_type;           /* Segment type */
  Elf32_Off     p_offset;         /* Segment file offset */
  Elf32_Addr    p_vaddr;          /* Segment virtual address */
  Elf32_Addr    p_paddr;          /* Segment physical address */
  Elf32_Word    p_filesz;         /* Segment size in file */
  Elf32_Word    p_memsz;          /* Segment size in memory */
  Elf32_Word    p_flags;          /* Segment flags */
  Elf32_Word    p_align;          /* Segment alignment, file & memory */
} Elf32_Phdr;

// section header
typedef struct elf32_shdr {
  Elf32_Word    sh_name;          /* Section name, index in string tbl */
  Elf32_Word    sh_type;          /* Type of section */
  Elf32_Word    sh_flags;         /* Miscellaneous section attributes */
  Elf32_Addr    sh_addr;          /* Section virtual addr at execution */
  Elf32_Off     sh_offset;        /* Section file offset */
  Elf32_Word    sh_size;          /* Size of section in bytes */
  Elf32_Word    sh_link;          /* Index of another section */
  Elf32_Word    sh_info;          /* Additional section information */
  Elf32_Word    sh_addralign;     /* Section alignment */
  Elf32_Word    sh_entsize;       /* Entry size if section holds table */
} Elf32_Shdr;
```

### 二、解析elf文件
　　所谓工欲善其事，必先利其器，在开始解析elf文件之前，我们必须先找到一款合适的解析工具，readelf就是GNU/Linux官方推出的专用解析工具。有了这个解析工具，我们便可以逐步分析elf文件。  

#### 2.1 解析工具readelf
　　既然elf文件是Linux系统下常用的可执行文件格式，那么Linux社区一定会有配套的工具去解析它，是的，这个工具就叫readelf，在GNU工具集binutils里。  

##### 2.1.1 GNU工具集（binutils）
　　GNU是“GNU's Not Unix”的递归缩写，又称为GNU计划，很多著名的开源软件及工具都是GNU开发的（比如大名鼎鼎的C语言编译器GCC）。binutils是GNU一系列binary小工具的集合，大家从下面的链接里找到官方binutils包。  

> 主页：http://www.gnu.org/software/binutils/  
> 仓库：git://sourceware.org/git/binutils-gdb.git  
> 下载：http://ftp.gnu.org/gnu/binutils/  
> 文档：https://sourceware.org/binutils/docs-2.29/binutils/index.html  

　　但是使用上述包里的readelf会有一个问题，上述工具是在Linux系统下使用的，而大家平常做ARM Cortex-M开发很多都是在windows平台下，那么怎么在windows下使用readelf工具呢？别急，cygwin给了我们帮助。  

##### 2.1.2 cygwin（windows下使用GNU）
　　Cygwin是一个在windows平台上运行的类UNIX模拟环境，是cygnus solutions公司（已被Redhat收购）开发的自由软件。它对于学习UNIX/Linux操作环境，或者从UNIX到Windows的应用程序移植，尤其是使用GNU工具集在Windows上进行嵌入式系统开发，非常有用。  

> Installer：http://cygwin.com/install.html  
> Package：https://cygwin.com/packages/package_list.html  

```text
// 相关工具包
binutils                - GNU assembler, linker, and similar utilities
cygwin32-binutils       - Binutils for Cygwin 32bit toolchain
mingw64-x86_64-binutils - Binutils for MinGW-w64 Win64 toolchain
mingw64-i686-binutils   - Binutils for MinGW-w64 Win32 toolchain
```

　　下载安装好cygwin包后，便可在安装目录下\cygwin64\bin\找到x86\_64-w64-mingw32-readelf.exe工具（痞子衡选择的是mingw64-x86_64-binutils包）。  

##### 2.1.3 readelf.exe用法
　　readelf.exe遵循标准的windows命令行用法，使用--help可以列出所有命令option及其简介，下面仅列出比较常用的option。  
```text
C:\cygwin64\bin>x86_64-w64-mingw32-readelf.exe --help
Usage: readelf <option(s)> elf-file(s)
 Display information about the contents of ELF format files
 Options are:
  -a --all               Equivalent to: -h -l -S -s -r -d -V -A -I
  -h --file-header       Display the ELF file header
  -l --program-headers   Display the program headers
     --segments          An alias for --program-headers
  -S --section-headers   Display the sections' header
     --sections          An alias for --section-headers
  -t --section-details   Display the section details
  -e --headers           Equivalent to: -h -l -S
  -s --syms              Display the symbol table
     --symbols           An alias for --syms
  --dyn-syms             Display the dynamic symbol table
  -r --relocs            Display the relocations (if present)
  -d --dynamic           Display the dynamic section (if present)
  -V --version-info      Display the version sections (if present)
  -A --arch-specific     Display architecture specific information (if any)
  -I --histogram         Display histogram of bucket list lengths
  @<file>                Read options from <file>
```

#### 2.2 逐步分析elf文件
　　万事俱备了，开始分析elf文件，以第三节课[工程文件(.ewp)](http://www.cnblogs.com/henjay724/p/8232585.html)里demo工程为例。编译链接该工程可在D:\myProject\bsp\builds\demo\Release\Exe路径下得到demo.elf文件。该文件大小32612 bytes，显然这么精简的一个小工程image size不可能这么大，说明elf文件里的记录信息数据占比非常大。  

##### 2.2.1 获得file header
```text
C:\cygwin64\bin>x86_64-w64-mingw32-readelf.exe -h demo.elf
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           ARM
  Version:                           0x1
  Entry point address:               0x41
  Start of program headers:          31740 (bytes into file)
  Start of section headers:          31772 (bytes into file)
  Flags:                             0x5000000, Version5 EABI
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         1
  Size of section headers:           40 (bytes)
  Number of section headers:         21
  Section header string table index: 1
```
　　第一步首先分析file header，前面介绍里说过file header是放在文件最前面的。通过readelf -h命令可以获得file header解析后的信息。让我们来对照一下，使用HexEditor直接打开demo.elf可得到如下数据，仅取前52bytes（0x34）数据，因为Elf32_Ehdr大小就是52bytes：  
```text
offset(h)
00000000: 7F 45 4C 46 01 01 01 00 00 00 00 00 00 00 00 00
00000010: 02 00 28 00 01 00 00 00 41 00 00 00 FC 7B 00 00
00000020: 1C 7C 00 00 00 00 00 05 34 00 20 00 01 00 28 00
00000030: 15 00 01 00 -- -- -- -- -- -- -- -- -- -- -- --
```
　　可以看到前16byte是e\_ident[16]，与解析后的Magic是一致的；再来验证prgram header偏移e_phoff=0x00007BFC，数量e\_phnum=0x0001，大小e\_phentsize=0x0020，也是与解析后的信息匹配的；余下可自行对照。  

##### 2.2.2 获得program header
```text
C:\cygwin64\bin>x86_64-w64-mingw32-readelf.exe -l demo.elf

Elf file type is EXEC (Executable file)
Entry point 0x41
There are 1 program headers, starting at offset 31740

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  LOAD           0x000034 0x00000000 0x00000000 0x004c4 0x004c4 R E 0x100

 Section to Segment mapping:
  Segment Sections...
   00     A0 rw P1 ro
```
　　再来分析program header，通过readelf -l命令可以获得program header解析后的信息。从上面可以得知header起始位置在demo.elf的31740 byte处（与file header里的e\_phoff信息是对应的），header信息提示program data从offset 0x34开始，大小共0x4c4 bytes，Reset\_Handler入口是0x41。继续在HexEditor查看31740处开始的32byte数据，因为Elf32\_Phdr大小就是32bytes：  
```text
offset(h)
00007BF0: -- -- -- -- -- -- -- -- -- -- -- -- 01 00 00 00
00007C00: 34 00 00 00 00 00 00 00 00 00 00 00 C4 04 00 00
00007C10: C4 04 00 00 05 00 00 00 00 01 00 00 -- -- -- --
```
　　可以看到p\_offset=0x00000034，p\_memsz=0x000004c4, 与上面解析后的信息是一致的；余下可自行对照。 这里的信息就比较重要了，因为这指示了整个image binary数据所在（知道了这个信息，我们便可以直接写脚本根据elf文件生成image binary），继续在HexEditor里看下去（仅截取部分显示）：  
```text
offset(h)
00000030: -- -- -- -- 00 20 00 10 41 00 00 00 03 04 00 00
00000040: 3F 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00000050: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00000060: 61 04 00 00 00 00 00 00 00 00 00 00 63 04 00 00
00000070: 65 04 00 00 72 B6 0E 48 0E 49 88 60 00 22 00 23
00000080: 00 24 00 25 00 26 00 27 B8 46 B9 46 BA 46 BB 46
```
　　ARM系统的image前16个指针都是系统中断向量，我们可以看到SP=0x10002000, PC=0x00000041，这与上面解析的Reset_Handler入口是0x41是匹配的。  

##### 2.2.3 获得section header
```text
c:\cygwin64\bin>x86_64-w64-mingw32-readelf.exe -S demo.elf
There are 21 section headers, starting at offset 0x7c1c:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 006338 000000 00      0   0  4
  [ 1] .shstrtab         STRTAB          00000000 006338 0000e6 00      0   0  4
  [ 2] .strtab           STRTAB          00000000 006420 000b7c 00      0   0  4
  [ 3] .symtab           SYMTAB          00000000 006f9c 000c60 10      2 135  4
  [ 4] A0 rw             PROGBITS        00000000 000034 000040 01  AX  0   0 256
  [ 5] P1 ro             PROGBITS        00000040 000074 000484 01  AX  0   0  4
  [ 6] P3 ui             NOBITS          10000000 0004f8 002000 01  WA  0   0  8
  [ 7] P2 rw             NOBITS          10002000 0004f8 000438 01  WA  0   0  8
  [ 8] .debug_abbrev     PROGBITS        00000000 0004f8 0002c6 01      0   0  0
  [ 9] .debug_aranges    PROGBITS        00000000 0007c0 00016c 01      0   0  0
  [10] .debug_frame      PROGBITS        00000000 00092c 00057c 01      0   0  0
  [11] .debug_info       PROGBITS        00000000 000ea8 000e2e 01      0   0  0
  [12] .debug_line       PROGBITS        00000000 001cd8 000dcb 01      0   0  0
  [13] .debug_loc        PROGBITS        00000000 002aa4 00024c 01      0   0  0
  [14] .debug_macinfo    PROGBITS        00000000 002cf0 00011e 01      0   0  0
  [15] .debug_pubnames   PROGBITS        00000000 002e10 00012a 01      0   0  0
  [16] .iar.debug_frame  PROGBITS        00000000 002f3c 00007e 01      0   0  0
  [17] .iar.debug_line   PROGBITS        00000000 002fbc 000367 01      0   0  0
  [18] .comment          PROGBITS        00000000 003324 002fa2 01      0   0  0
  [19] .iar.rtmodel      PROGBITS        00000000 0062c8 000047 01      0   0  0
  [20] .ARM.attributes   ARM_ATTRIBUTES  00000000 006310 000026 01      0   0  0
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  y (purecode), p (processor specific)
```
　　再来分析section header，通过readelf -S命令可以获得section header解析后的信息。可以看到有很多个section，其中最重要的4个section是A0(readonly vector)， P1(readonly code,data)， P2(readwrite data, heap)， P3(STACK)。具体分析，各位朋友自己试试看。  

##### 2.2.4 获得symbol list
```text
c:cygwin64\bin>x86_64-w64-mingw32-readelf.exe -s demo.elf

Symbol table '.symtab' contains 198 entries:
   Num:    Value  Size Type    Bind   Vis      Ndx Name
    74: 10002018    16 OBJECT  LOCAL  DEFAULT    7 s_array
    75: 10002014     4 OBJECT  LOCAL  DEFAULT    7 s_variable0
    76: 10002010     4 OBJECT  LOCAL  DEFAULT    7 s_variable2
   135: 00000000     0 OBJECT  GLOBAL DEFAULT    4 __vector_table
   140: 00000041     0 FUNC    GLOBAL DEFAULT    5 Reset_Handler
   141: 00000098     4 OBJECT  GLOBAL DEFAULT    5 s_constant
   142: 000000ad    32 FUNC    GLOBAL DEFAULT    5 main
   143: 000000cd    14 FUNC    GLOBAL DEFAULT    5 normal_task
   144: 000000db    60 FUNC    GLOBAL DEFAULT    5 heap_task
   155: 0000034d    84 FUNC    GLOBAL DEFAULT    5 init_data_bss
   156: 000003a1    18 FUNC    GLOBAL DEFAULT    5 init_interrupts
   157: 000003dd    12 FUNC    GLOBAL DEFAULT    5 SystemInit
   186: 10002001    16 FUNC    GLOBAL DEFAULT    7 ram_task
   191: 10002034     4 OBJECT  GLOBAL DEFAULT    7 n_variable1
```
　　通过readelf -s命令可以获得symbol list解析后的信息。可以看到有很多个symbol，痞子衡在这里仅列出应用工程里自定义的函数和变量，从symbol表里我们可以得知函数/变量在存储器中具体分配地址和长度，这对于我们进一步分析和调试应用是有帮助的。  

#### 2.3 elf文件layout
　　经过上一节对demo.elf里各个header的分析，此时我们便可以粗略地画出elf文件layout。  

<table>
    <tr>
        <th>File offset</th>
        <th>Data content</th>
        <th>Data size in bytes</th>
    </tr>
    <tr>
        <td>0x00000000</td>
        <td>ELF file header</td>
        <td>52</td>
    </tr>
    <tr>
        <td>0x00000034</td>
        <td>Image binary (Section4-A0 rw, .intvec中断向量表)</td>
        <td>0x40</td>
    </tr>
    <tr>
        <td>0x00000074</td>
        <td>Image binary (Section5-P1 ro, readonly section(.text, .rodata...))</td>
        <td>0x484</td>
    </tr>
    <tr>
        <td>0x000004F8</td>
        <td>Section8-20 (包含各种辅助调试和系统段.debug_xx, .iar.xx)</td>
        <td>0x5E3E</td>
    </tr>
    <tr>
        <td>0x00006336</td>
        <td>NULL</td>
        <td>0x2</td>
    </tr>
    <tr>
        <td>0x00006338</td>
        <td>Section1-.shstrtab字符串表</td>
        <td>0xE6</td>
    </tr>
    <tr>
        <td>0x00006420</td>
        <td>Section2-.strtab字符串信息</td>
        <td>0xB7C</td>
    </tr>
    <tr>
        <td>0x00006F9C</td>
        <td>Section3-.symtab符号信息</td>
        <td>0xC60</td>
    </tr>
    <tr>
        <td>0x00007BFC</td>
        <td>ELF Program header</td>
        <td>0x20</td>
    </tr>
    <tr>
        <td>0x00007C1C</td>
        <td>ELF Section headers (0 - 20)</td>
        <td>21 * 40</td>
    </tr>
</table>

### 番外一、几个elf转换image工具
　　在今天的番外篇里，痞子衡给大家顺便介绍几款标准的elf文件转换成image文件的工具。  
#### 工具1：GNU工具objcopy
```text
位置：C:\cygwin64\bin>x86_64-w64-mingw32-objcopy.exe
用法：
      objcopy.exe -O binary -S demo.elf demo.bin
      objcopy.exe -O srec   -S demo.elf demo.s19

备注：一说需用arm-linux-objcopy，待验证
```

#### 工具2：IAR工具ielftool.exe
```text
位置：\IAR Systems\Embedded Workbench xxx\arm\bin\ielftool.exe
用法：
      ielftool.exe --bin  demo.elf demo.bin
      ielftool.exe --ihex demo.elf demo.hex
      ielftool.exe --srec demo.elf demo.s19
```

　　至此，嵌入式开发里的executable文件(elf)文件痞子衡便介绍完毕了，掌声在哪里~~~ 


