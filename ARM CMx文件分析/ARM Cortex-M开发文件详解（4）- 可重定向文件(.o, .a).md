----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式开发里的relocatable文件(object, library)**。  

　　前三节课里，痞子衡都是在给大家介绍嵌入式开发中的input文件。从今天这节课开始，痞子衡就陆续为大家讲output文件。上一节课[project文件]()里讲说到project文件是一个承前启后的文件，今天痞子衡就为大家讲project生成的第一类output文件：relocatable文件。  

　　文件关系：[source文件]() + [project文件]() -> **relocatable文件**  

　　relocatable文件，即可重定向文件，这个文件是由编译器汇编源文件（.c/.s）而成的。直接生成的重定向文件叫object file，经过封装的重定向文件称为library file。可重定向文件属于ELF文件的分支，关于ELF文件的详细解释可见第六节课[executable文件]()。  
　　本文主角object file和library file，仅是一个中间的过渡文件，其本身也不能被ARM直接执行，需经过第二步转换，即链接，所以这两个文件都是链接器的输入文件。让我们来简单分析一下这两个文件。在开始分析之前我们先回到上一节课[project文件]()的最后创建的demo工程上，编译这个demo工程可以得到如下.o文件，这些文件全是object文件，每一个源文件都对应一个object文件，本文以task.o为例讲解relocatable文件。  
```C
D:\myProject\bsp\builds\demo\Release\Obj\main.o
D:\myProject\bsp\builds\demo\Release\Obj\reset.o
D:\myProject\bsp\builds\demo\Release\Obj\startup.o
D:\myProject\bsp\builds\demo\Release\Obj\startup_MKL25Z4.o
D:\myProject\bsp\builds\demo\Release\Obj\system_MKL25Z4.o
D:\myProject\bsp\builds\demo\Release\Obj\task.o -o
```

### 一、解析object文件
　　task.o文件大小有11683bytes，而从源文件里看其仅包含4个变量和3个函数，可见更多的数据是记录性数据。  

#### 1.1 获得file header
```C
c:\cygwin64\bin>x86_64-w64-mingw32-readelf.exe -h task.o
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           ARM
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          8283 (bytes into file)
  Flags:                             0x5000000, Version5 EABI
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         0
  Size of section headers:           40 (bytes)
  Number of section headers:         85
  Section header string table index: 1
```
　　分析file header可知task.o是REL类型ELF文件，其一共含有85个section header，没有program header。  

#### 1.2 获得section header
```C
c:\cygwin64\bin>x86_64-w64-mingw32-readelf.exe -S task.o
There are 85 section headers, starting at offset 0x205b:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000034 000000 00      0   0  0
  [ 1] .shstrtab         STRTAB          00000000 000034 0001eb 00      0   0  0
  [ 2] .symtab           SYMTAB          00000000 00021f 000350 10      3  45  0
  [ 3] .strtab           STRTAB          00000000 00056f 000248 00      0   0  0
  [ 8] .bss              NOBITS          00000000 000e1c 000004 00  WA  0   0  4
  [ 9] .noinit           NOBITS          00000000 000e1c 000004 00  WA  0   0  4
  [10] .data             PROGBITS        00000000 000e1c 000004 00  WA  0   0  4
  [11] .bss              NOBITS          00000000 000e20 000010 00  WA  0   0  4
  [12] .text             PROGBITS        00000000 000e20 000058 00  AX  0   0  4
  [13] .textrw           PROGBITS        00000000 000e78 000010 00 WAX  0   0  4
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  y (purecode), p (processor specific)
```
　　分析section header可知该task.o里的各个常见section（.bss, .noinit, .data, .text, .textrw）的大小，各个段的含义详见第二节课[linker文件]()。  

#### 1.3 获得symbol list
```C
c:\cygwin64\bin>x86_64-w64-mingw32-readelf.exe -s task.o

Symbol table '.symtab' contains 53 entries:
   Num:    Value  Size Type    Bind   Vis      Ndx Name
     3: 00000000    16 OBJECT  LOCAL  DEFAULT   11 s_array
     4: 00000000     4 OBJECT  LOCAL  DEFAULT    8 s_variable0
     5: 00000000     4 OBJECT  LOCAL  DEFAULT   10 s_variable2
    45: 00000000     0 FUNC    GLOBAL DEFAULT  UND __aeabi_memcpy
    46: 00000000     0 FUNC    GLOBAL DEFAULT  UND __aeabi_memset
    47: 00000000     0 FUNC    GLOBAL DEFAULT  UND free
    48: 00000000     0 FUNC    GLOBAL DEFAULT  UND malloc
    49: 0000000f    60 FUNC    GLOBAL DEFAULT   12 heap_task
    50: 00000000     4 OBJECT  GLOBAL DEFAULT    9 n_variable1
    51: 00000001    14 FUNC    GLOBAL DEFAULT   12 normal_task
    52: 00000001    16 FUNC    GLOBAL DEFAULT   13 ram_task
```
　　分析symbol list可知我们在task.c里定义的函数和全局变量的信息，其中Value表明的是各symbol对象（函数/全局变量）在存储器中的分配地址，由于object文件并没有经过链接，所以此处地址信息是无效的（待分配的）。翻看到第六节课[executable文件]()里2.2.4一节，便可看到这些symbol对象Value的值开始变得真实有效了。这就解释了为什么object文件是relocatable的。  

### 二、关于library文件
　　本质上library文件跟object文件是一样的，都是未经链接器链接的文件。library文件的应用场景是，在一些特殊场合，你不想把你的C源代码开放给别人阅读和自由修改，但是你又需要分享你的代码给别人使用，怎么解决这个问题？library文件就是解决这个问题的，可以借助编译器的选项（IAR下是Options->General Options->Output->Output file里选择Library（默认是executable）），那么添加进整个工程的所有源文件会被汇编封装成一个.a文件（即library文件），这时候你只需要将该.a文件以及配套API头文件分享给别人即可。别人只需要添加你的.a文件以及配套.h文件进他自己的工程，便可直接调用你的API。  

　　至此，嵌入式开发里的relocatable文件(object, library)文件痞子衡便介绍完毕了，掌声在哪里~~~ 

