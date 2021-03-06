----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家讲的是**嵌入式开发里的image文件(.bin, .hex, .s19)**。  

　　今天这节课是痞子衡《ARM Cortex-M文件那些事》主题系列的最后一节课（突然有点不舍，要告别的感觉，咳咳，让痞子衡整理下情绪先）。今天痞子衡主要讲的是工程开发最终的output文件，即image文件。image文件也叫镜像文件，这个文件主要包含的是只有芯片能够解释执行的二进制机器码数据，这些数据其实在前面介绍的relocatable、list、executable文件中出现过，在那些文件里我们还可以根据其他辅助信息来分析机器码数据的实际意义，但在image文件里，我们已经完全无法看懂这些机器码了。所以image文件主要是用来做大规模量产的。既要做大规模量产，由于各芯片厂家制定的标准不一，所以实际上image文件有很多种格式，今天我们主要讲的是其中最具有代表性也应用最广泛的3种image文件格式。  

### 一、通用镜像文件bin
　　第一种格式叫binary，以.bin为文件后缀，这种格式是一种通用image格式，其完全是机器码裸数据的集合，没有其他任何多余信息，这个数据可以直接被编程器/下载器下载到芯片内部非易失性存储器里，不需要任何额外的数据转换，所见即所得。由于是纯二进制编码的文件，所以普通text编辑器无法正确查看这个文件，需要用专用的十六进制编辑器（比如Hex Editor HxD）才能正常打开。以本系列创建的demo工程的demo.bin文件为例，用HxD打开可见如下数据（仅截取前后部分显示，demo.bin共6780 bytes）。  
```text
offset(h)
00000000: 00 20 00 10 41 00 00 00 DB 18 00 00 CB 19 00 00
00000010: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00000020: 00 00 00 00 00 00 00 00 00 00 00 00 07 1A 00 00
00000030: 00 00 00 00 00 00 00 00 1D 1A 00 00 1F 1A 00 00
00000040: 72 B6 0E 48 0E 49 88 60 00 22 00 23 00 24 00 25
...
00001A30: 01 23 00 24 13 E0 0A 68 09 1D 1A 42 02 D0 4D 46
00001A40: 6D 1E 52 19 14 60 12 1D 00 1F 04 28 FA D2 15 00
00001A50: 86 07 01 D5 14 80 AD 1C 18 40 00 D0 2C 70 08 68
00001A60: 09 1D 00 28 E7 D1 08 00 70 BC 70 47 C1 FF FF FF
00001A70: 08 02 00 00 14 20 00 10 00 00 00 00 -- -- -- --
```
　　有细心的朋友可能会疑问，打开bin文件看数据都是按连续地址排列的，如果在应用设计中，我们在linker文件里给各个段分配的地址不是连续的，这在bin文件里是怎么处理的？为了解决这个问题，bin文件会在非有效地址区域插入无效字节以保证有效地址处都是正确的数据，这在IDE里或相关转换工具里都会有相应option，可以让用户设置填充字节pattern（比如0x00，0xFF等）。  
　　好，现在让我们尝试分析一下这个bin文件，我们都知道ARM Cortex-M架构里，image bin文件前8个字节应该是初始SP和PC的值，从前面map文件里我们知道SP=0x10002000，PC=0x00000041，来检查一下bin文件里是不是这样，前4个字节分别是 00 20 00 10、看起来好像跟0x10002000数据是吻合的，但是这个数据排列方式看起来好像有点别扭，怎么回事？嵌入式老司机这时应该要莞尔一笑，是的，ARM Coretx-M默认采用的Little-Endian（小端）模式，即低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。数据查看地址显示都是从低到高的（从左到右），所以SP的最低字节应该显示在最左边（最低地址），而不是像查看0x10002000那样最低字节在最右边，这跟人的阅读习惯是有点不吻合。  
　　PS: 既有小端模式，那么与其对应的也有大端模式（Big-Endian）-高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。注意大端小端仅是针对以32bit为单元的数据排列方式差异，对于n个32bit数据，其都是统一的排列。  
```text
offset(h)
// 小端
00000000: 00 20 00 10 41 00 00 00 
// 大端
00000000: 10 00 20 00 00 00 00 41 
```
　　从上面对bin文件的分析，我们知道bin文件是不含地址信息的，也就是说bin文件数据应该被放在什么地址处，我们仅从bin文件本身是无法得知的。所以在使用编程器/下载器下载bin文件时，用户必须指定起始下载地址。由于bin文件的这种局限性，下面两种带地址信息的image格式应运而生。  

### 二、Intel镜像文件标准hex
　　第二种格式叫Intel hex，以.hex为文件后缀，这种格式是Intel公司推行的一种image格式标准，其不仅含有机器码裸数据还含有地址信息等额外信息，与bin文件不同的是，hex文件可以直接通用普通text编辑器打开查看，hex文件采用的ASCII编码，hex文件内的机器码数据不可以直接被下载进芯片内部，需要在帧数据解析的过程中进行转换。  
　　由于hex文件并不是纯机器码文件，还含有其他额外信息，那么hex文件就需要按某种约定格式进行数据组织，数据组织方式叫帧格式，hex文件是由n帧数据组成的。  
#### 2.1 hex帧格式
　　要想解析hex文件，必须要先了解其帧格式，hex每帧都由下表列出的6部分组成：  

<table>
    <tr>
        <th>帧段名</th>
        <td>Start code</th>
        <td>Byte count</th>
        <td>Address</th>
        <td>Record type</th>
        <td>Data</th>
        <td>Checksum</th>
    </tr>
    <tr>
        <th>帧段内容</td>
        <td>固定前导码':'，十六进制0x3A</th>
        <td>机器码数据长度</th>
        <td>机器码数据存储地址</th>
        <td>帧类型"00"-"05"，有6种帧</th>
        <td>机器码数据</th>
        <td>校验和</th>
    </tr>
    <tr>
        <th>帧段长度(bytes)</td>
        <td>1</th>
        <td>1*2</th>
        <td>2*2</th>
        <td>1*2</th>
        <td>(0-255)*2</th>
        <td>1*2</th>
    </tr>
</table>

　　编程器/下载器在解析hex文件时，先找到帧前导码，然后找到帧类型，如果该帧为数据帧，再根据帧机器码长度，将该帧机器码数据全部读出放到缓存里，在做完帧和校验后，如果没有错误，最后根据帧机器码存储地址将帧机器码数据下载到芯片指定存储器地址处，至此一帧处理结束，进入下一帧，直到所有帧全部处理完。需要注意的是由于hex文件是ASCII编码，所以相比bin文件长度至少大2倍以上，demo.hex文件大小有19,106 bytes，后面我们会截取部分hex文件进行分析。  
　　关于checksum的计算方法，其是将Byte count、Address、Record type、Data四个段内所有byte全部相加得到sum，截取sum的LSB（最低字节），再对该LSB取其补码（默认该LSB为负数的低8bit数据位(注意8bit中没有符号位)，其反码为原码各bit取反，其补码为反码+1）得到checksum。比如LSB是0xA5，其反码为0x5A，补码为0x5B，则checksum为0x5B。  

#### 2.2 hex帧类型
　　前面说到一共有6种类型的帧，其中最重要也是数量最多的帧是数据帧，除了数据帧之外还有其他5种帧，下面来统一介绍：  

<table>
    <tr>
        <th>帧类型码</th>
        <th>帧类型</th>
        <th>帧描述</th>
        <th>帧举例</th>
    </tr>
    <tr>
        <td>"00"</th>
        <td>数据帧</th>
        <td>以16bit地址描述开始的最大255个字节有效机器码的数据帧</th>
        <td>含11bytes机器码从0x0010地址处开始的数据帧:0B0010006164647265737320676170A7</th>
    </tr>
    <tr>
        <td>"01"</th>
        <td>文件结尾帧</th>
        <td>用于表明hex文件的结尾</th>
        <td>统一的文件结尾帧:00000001FF</th>
    </tr>
    <tr>
        <td>"02"</th>
        <td>拓展段地址帧</th>
        <td>多用于80x86架构芯片，ARM Cortex-M架构不用</th>
        <td>:020000021200EA</th>
    </tr>
    <tr>
        <td>"03"</th>
        <td>起始段地址帧</th>
        <td>多用于80x86架构芯片，ARM Cortex-M架构不用</th>
        <td>:0400000300003800C1</th>
    </tr>
    <tr>
        <td>"04"</th>
        <td>拓展线性地址帧</th>
        <td>用于32bit地址存储空间芯片，与数据帧配合使用，指引编程器将数据下载到正确地址</th>
        <td>标明拓展地址为0xFFFF的帧:02000004FFFFFC</th>
    </tr>
    <tr>
        <td>"05"</th>
        <td>起始程序地址（PC）帧</th>
        <td>指示调试器，程序初始PC地址，方便在线调试</th>
        <td>标明起始PC为0x000000CD的帧:04000005000000CD2A</th>
    </tr>
</table>

#### 2.3 解析hex文件
　　在了解hex文件帧数据格式之后，让我们开始尝试解析demo.hex文件(仅截取前后部分，与前面截取的bin文件内容对应着一起分析)  
```text
:100000000020001041000000DB180000CB190000A8
:1000100000000000000000000000000000000000E0
:10002000000000000000000000000000071A0000AF
:1000300000000000000000001D1A00001F1A000050
:1000400072B60E480E498860002200230024002565
....
:101A30000123002413E00A68091D1A4202D04D4612
:101A40006D1E52191460121D001F0428FAD21500D1
:101A5000860701D51480AD1C184000D02C70086892
:101A6000091D0028E7D1080070BC7047C1FFFFFFC7
:0C1A70000802000014200010000000001C
:0400000500000041B6
:00000001FF
```
　　hex文件前5帧均为数据帧，每帧包含16bytes机器码数据，帧数据地址分别为0x0000, 0x0010, 0x0020, 0x0030, 0x0040，可见帧数据是连续的，并且5帧机器码数据共80bytes与bin文件前80bytes是一致的。  
　　再来看最后7帧数据里的前5个数据帧，除最后一帧数据只包含12bytes数据外，其余数据帧均含有16bytes数据，5帧数据一共76bytes，帧数据地址从0x1A30 - 0x1A70。显然这与bin文件最后76bytes也是吻合的。  
　　倒数第二帧是起始程序地址帧，其标明的程序起始PC是0x00000041，这与bin文件里第二个32bit数据（起始PC）是一致的。  
　　倒数第一帧显然是标准文件结尾帧。  

### 三、Motorola镜像文件标准S-Record
　　第三种格式叫Motorola S-Record，以.s19或.srec为文件后缀，这种格式是Motorola公司推行的一种image格式标准，其与Intel hex文件比较类似，都是ASCII编码的文件，可以通过普通text编辑器打开查看，其也由帧数据组成，只是帧格式与Intel hex有差别，还是按照介绍Intel hex文件那样先来看S-Record文件的帧格式。  
#### 3.1 S-Record帧格式
　　S-Record每帧由下表列出的6部分组成：  

<table>
    <tr>
        <th>帧段名</th>
        <td>Start code</th>
        <td>Record type</th>
        <td>Byte count</th>
        <td>Address</th>
        <td>Data</th>
        <td>Checksum</th>
    </tr>
    <tr>
        <th>帧段内容</td>
        <td>固定前导码'S'，十六进制0x53</th>
        <td>帧类型'0'-'9'，有10种帧</th>
        <td>帧数据长度（包含后续段地址、数据、校验和）</th>
        <td>机器码数据存储地址</th>
        <td>机器码数据</th>
        <td>校验和</th>
    </tr>
    <tr>
        <th>帧段长度(bytes)</td>
        <td>1</th>
        <td>1</th>
        <td>1*2</th>
        <td>(2-4)*2</th>
        <td>(0-255)*2</th>
        <td>1*2</th>
    </tr>
</table>

　　编程器/下载器在解析S-Record文件时，先找到帧前导码，然后找到帧类型，如果该帧为数据帧，再根据帧长度，将该帧机器码数据全部读出放到缓存里，在做完帧和校验后，如果没有错误，最后根据帧机器码存储地址将帧机器码数据下载到芯片指定存储器地址处，至此一帧处理结束，进入下一帧，直到所有帧全部处理完。  
　　关于checksum的计算方法，其是将Byte count、Address、Data三个段内所有byte全部相加得到sum，截取sum的LSB（最低字节），再对该LSB取其反码（默认该LSB为负数的低8bit数据位(注意8bit中没有符号位)，其反码为原码各bit取反）得到checksum。比如LSB是0xA5，其反码为0x5A，则checksum为0x5A。  

#### 3.2 S-Record帧类型
　　前面说到一共有10种类型的帧，其中最重要也是数量最多的帧是数据帧，数据帧按地址长度可分为16bit、24bit、32bit地址长度数据帧，除了数据帧，还有其他种类帧，下面来统一介绍：

<table>
    <tr>
        <th>帧类型码</th>
        <th>帧类型</th>
        <th>帧描述</th>
        <th>帧举例</th>
    </tr>
    <tr>
        <td>'0'</th>
        <td>文件起始帧</th>
        <td>用于表明S-Record文件的开始</th>
        <td>标明文件名为HDR的文件起始帧S00600004844521B</th>
    </tr>
    <tr>
        <td>'1'</th>
        <td>数据帧x16地址</th>
        <td>以16bit地址描述开始的最大255个字节有效机器码的数据帧</th>
        <td>含14bytes机器码从0x0038地址处开始的数据帧S111003848656C6C6F20776F726C642E0A0042</th>
    </tr>
    <tr>
        <td>'2'</th>
        <td>数据帧x24地址</th>
        <td>以24bit地址描述开始的最大255个字节有效机器码的数据帧</th>
        <td>含4bytes机器码从0x100000地址处开始的数据帧S2081000000400FA05E5</th>
    </tr>
    <tr>
        <td>'3'</th>
        <td>数据帧x32地址</th>
        <td>以32bit地址描述开始的最大255个字节有效机器码的数据帧</th>
        <td>含4bytes机器码从0x13000160地址处开始的数据帧S309130001600400FA057F</th>
    </tr>
    <tr>
        <td>'4'</th>
        <td>N/A</th>
        <td>未定义</th>
        <td>N/A</th>
    </tr>
    <tr>
        <td>'5'</th>
        <td>数据帧总数帧x16</th>
        <td>用16bit count记录数据帧总帧数的总数帧</th>
        <td>标明总数据帧为4帧的总数帧S5030004F8</th>
    </tr>
    <tr>
        <td>'6'</th>
        <td>数据帧总数帧x24</th>
        <td>用24bit count记录数据帧总帧数的总数帧</th>
        <td>标明总数据帧为80000帧的总数帧S604080000F3</th>
    </tr>
    <tr>
        <td>'7'</th>
        <td>起始程序地址（PC）帧x32</th>
        <td>含32bit起始PC的帧</th>
        <td>标明起始PC为0x10000000的帧S70510000000EA</th>
    </tr>
    <tr>
        <td>'8'</th>
        <td>起始程序地址（PC）帧x24</th>
        <td>含24bit起始PC的帧</th>
        <td>标明起始PC为0x100000的帧S804100000EB</th>
    </tr>
    <tr>
        <td>'9'</th>
        <td>起始程序地址（PC）帧x16</th>
        <td>含16bit起始PC的帧</th>
        <td>标明起始PC为0x0000的帧S9030000FC</th>
    </tr>
</table>

#### 3.3 解析S-Record文件
　　在了解S-Record文件帧数据格式之后，让我们开始尝试解析demo.s19文件(仅截取前后部分，与前面截取的bin文件内容对应着一起分析)  
```text
S00B000064656D6F2E73313944
S11300000020001041000000DB180000CB190000A4
S113001000000000000000000000000000000000DC
S1130020000000000000000000000000071A0000AB
S113003000000000000000001D1A00001F1A00004C
S113004072B60E480E498860002200230024002561
....
S1131A300123002413E00A68091D1A4202D04D460E
S1131A406D1E52191460121D001F0428FAD21500CD
S1131A50860701D51480AD1C184000D02C7008688E
S1131A60091D0028E7D1080070BC7047C1FFFFFFC3
S10F1A7008020000142000100000000018
S9030041BB
```
　　S-Record文件第一帧是文件起始帧，帧数据64656D6F2E733139对应ASCII为demo.s19，即该文件名。  
　　第2-6帧均为16bit地址的数据帧，每帧包含16bytes机器码数据，帧数据地址分别为0x0000, 0x0010, 0x0020, 0x0030, 0x0040，可见帧数据是连续的，并且5帧机器码数据共80bytes与bin文件前80bytes是一致的。  
　　再来看最后6帧数据里的前5个数据帧，除最后一帧数据只包含12bytes数据外，其余数据帧均含有16bytes数据，5帧数据一共76bytes，帧数据地址从0x1A30 - 0x1A70。显然这与bin文件最后76bytes也是吻合的。  
　　倒数第一帧是起始程序地址帧，其标明的程序起始PC是0x0041，这与bin文件里第二个32bit数据（起始PC）是一致的。  

### 番外一、一些image文件辅助小工具

> SRecordizer：专用S19文件编辑器，可根据修改自动更新checksum，详见网页 https://srecordizer.codeplex.com/  
> SRecord项目：各种image文件格式互转的小工具合集，详见网页 http://srecord.sourceforge.net/  

　　至此，嵌入式开发里的image文件(.bin, .hex, .s19)文件痞子衡便介绍完毕了，掌声在哪里~~~ 