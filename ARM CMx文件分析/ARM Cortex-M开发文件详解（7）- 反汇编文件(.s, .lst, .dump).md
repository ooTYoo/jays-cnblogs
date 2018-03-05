----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式开发里的反汇编文件(.s, .lst, .dump)**。  

　　痞子衡在第四、五、六节课分别介绍了编译器/链接器生成的3种output文件（relocatable、map、exectuable文件），这3种文件都是侧重的代码经过汇编/链链接后的二进制数据在存储中分布情况。如果想知道二进制数据对应的机器码具体是什么意思应该怎么办？痞子衡今天要介绍的反汇编文件会给你答案。  

### 一、标准汇编源文件
　　使用IAR进行编译的时候会在D:\myProject\bsp\builds\demo\Release\List目录下生成.s文件，每一个relocatable文件都对应着一个.s文件，这是编译器对C源文件进行汇编后得到的汇编文件。以task.c汇编生成的task.s为例：  
　　task.s文件就是用汇编语言对task.c文件的逐句汇编式翻译，下面仅列出normal_task()函数的汇编代码，如果你愿意的话，你可以直接用这个task.s文件替代task.c文件放进工程里，功能是一样的。  
```C
        SECTION `.text`:CODE:NOROOT(1)
          CFI Block cfiBlock0 Using cfiCommon0
          CFI Function normal_task
          CFI NoCalls
        THUMB
//   17 void normal_task(void)
//   18 {
//   19     s_variable0 *= 2;
normal_task:
        LDR      R0,??DataTable1
        LDR      R0,[R0, #+0]
        MOVS     R1,#+2
        MULS     R0,R1,R0
        LDR      R1,??DataTable1
        STR      R0,[R1, #+0]
//   20 }
        BX       LR               ;; return
          CFI EndBlock cfiBlock0
//   21 
```
　　task.s文件最后还会给出文件里的object在各section中的总size情况。  
```C
// 20 bytes in section .bss
//  4 bytes in section .data
//  4 bytes in section .noinit
// 86 bytes in section .text
// 16 bytes in section .textrw
// 
// 102 bytes of CODE memory
//  28 bytes of DATA memory
```

### 二、中间汇编list文件
　　使用IAR进行编译的时候会在D:\myProject\bsp\builds\demo\Release\List目录下生成.lst文件，每一个relocatable文件都对应着一个.lst文件，这是编译器对C源文件进行汇编后得到的汇编文件的补充信息文件。继续以task.c汇编生成的task.lst为例：  
　　task.lst文件在task.s的基础上还加入了对汇编指令的机器码翻译信息，其中有0x....表明该文件没有经过全局的链接，所以还无法确定机器码。  
```C
   \                                 In section .text, align 2, keep-with-next
     17          void normal_task(void)
     18          {
     19              s_variable0 *= 2;
   \                     normal_task: (+1)
   \   00000000   0x....             LDR      R0,??DataTable1
   \   00000002   0x6800             LDR      R0,[R0, #+0]
   \   00000004   0x2102             MOVS     R1,#+2
   \   00000006   0x4348             MULS     R0,R1,R0
   \   00000008   0x....             LDR      R1,??DataTable1
   \   0000000A   0x6008             STR      R0,[R1, #+0]
     20          }
   \   0000000C   0x4770             BX       LR               ;; return
     21   
```
　　task.lst文件最后还给出最大栈使用的分析以及各object具体size情况。  
```C
   Maximum stack usage in bytes:

   .cstack Function
   ------- --------
      24   heap_task
        24   -> __aeabi_memcpy
        24   -> __aeabi_memset
        24   -> free
        24   -> malloc
       0   normal_task
       0   ram_task


   Section sizes:

   Bytes  Function/Label
   -----  --------------
       4  ??DataTable1
       4  ??DataTable1_1
       4  ??DataTable1_2
      60  heap_task
       4  n_variable1
      14  normal_task
      16  ram_task
      16  s_array
       4  s_variable0
       4  s_variable2
```

### 三、完整汇编dump文件
　　dump文件是所有list文件的集合，也是整个image文件的机器码数据的逐句汇编式翻译，还是以task.c里的normal_task()为例，在list文件中我们会看到部分未知机器码0x....，而在dump文件里这部分位置机器码被填充成真实的机器码。有了dump文件，我们就可以从汇编角度对整个工程进行解读分析。  
```C
  //     s_variable0 *= 2;
            $t:
            `.text12`:
            normal_task:
    0xcc: 0x4812         LDR.N R0, `.text_8`          ; `.data$$Limit`
    0xce: 0x6800         LDR   R0, [R0]
    0xd0: 0x2102         MOVS  R1, #2
    0xd2: 0x4348         MULS  R0, R1, R0
    0xd4: 0x4910         LDR.N R1, `.text_8`          ; `.data$$Limit`
    0xd6: 0x6008         STR   R0, [R1]
  // }
    0xd8: 0x4770         BX    LR
```

### 四、使用ielfdumparm.exe生成dump文件
　　dump文件默认是不生成的，但是IAR里提供了工具可以帮我们生成dump文件，这个工具叫ielfdumparm.exe。
```C
位置：\IAR Systems\Embedded Workbench xxx\arm\bin\ielfdumparm.exe
用法：ielfdumparm.exe --source --code demo.elf -o demo.dump
```  

　　至此，嵌入式开发里的反汇编文件(.s, .lst, .dump)文件痞子衡便介绍完毕了，掌声在哪里~~~ 