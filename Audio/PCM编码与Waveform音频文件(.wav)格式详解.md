----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**PCM编码及Waveform音频文件格式**。  

　　嵌入式里有时候也会和音频打交道，比如最近特别火的智能音箱产品，离不开前端的音频信号采集、降噪，中间的语音识别（ASR）、自然语言处理（NLP），以及后端的文语合成（TTS）、音频播放。音频信号采集是处理声音的第一步，要采集音频就离不开PCM编码，音频采集完成自然需要保存，waveform格式（.wav）是一种最经典的音频文件格式。今天痞子衡就给大家详细介绍PCM编码以及waveform文件格式。  

### 一、声音基础
　　众所周知，声音是由物体振动产生的声波，声音通过介质（空气或固体、液体）传播并能被人听觉器官所感知。发音物体情况（材料，距离，振动强度等）不同，产生的声音也不同。为了区分不同的声音，我们主要用如下三个参数来描述声音的特征：  

> * 音量：人主观上感觉到的声音大小（也叫响度），由“振幅”（amplitude）和人离声源的距离决定。
> * 音调：声音的高低（高音、低音），由“频率”（frequency）决定，频率越高音调越高。
> * 音色：音色是一种抽象的东西，波形决定了声音的音色。声音因不同发声物体材料而具有不同特性，波形是把这个抽象特性直观的表现出来。典型的音色波形有方波，锯齿波，正弦波，脉冲波等。

　　三大参数里除了音色没有度量单位外（可以认为音色是声音的UID，每种音色都是独一无二的），音量和音调均有度量单位，这意味着音量和音调是可调整的，也是声音之间可对比的特征参数。  

#### 1.1 音量单位-分贝(dB)
　　声波是一种机械波（压力波）。声波（空气质点）的连续振动，使空气分子不断交替的压缩和松弛，使大气压迅速产生起伏，这种气压的起伏部分，就称为声压。声压的振幅表示质点离开平衡位置的距离，反映从波形波峰到波谷的压力变化，以及波所携带的能量的多少。  
　　声压值虽然可以反映音量大小，但<font color="Blue">人们日常生活中遇到的声音，若以声压值表示，变化范围非常大（达到六个数量级以上），并且人体听觉对声信号强弱刺激反应不是线形的，而是成对数比例关系。因此音量并不是声压值来计量，而是用分贝来计量</font>，首先来看分贝计算的标准公式：  

> N<sub>dB</sub> = 10 \* log<sub>10</sub> (P<sub>i</sub> / P<sub>o</sub>)

　　上述公式中P<sub>o</sub>为基准声压值，N<sub>dB</sub>即是声压信号P<sub>i</sub>对基准声压P<sub>o</sub>的分贝值。从公式可以看出分贝是指两个相同类型物理量（P<sub>i</sub>、P<sub>o</sub>）的比较结果，它是无量纲的。分贝又称为被量度量P<sub>i</sub>的"级"，它代表被量度量比基准量高出多少"级"。下面列举常见分贝值：  

<table><tbody>
    <tr>
        <th style="width: 100px;">分贝值</th>
        <th style="width: 400px;">人耳感觉</th>
    </tr>
    <tr>
        <td>1dB</td>
        <td>刚能听到的声音</td>
    </tr>
    <tr>
        <td>1 - 15dB</td>
        <td>感觉非常安静</td>
    </tr>
    <tr>
        <td>20 - 40dB</td>
        <td>耳语音，冰箱的嗡嗡声</td>
    </tr>
    <tr>
        <td>40 - 60dB</td>
        <td>室内正常交谈的声音</td>
    </tr>
    <tr>
        <td>60 - 70dB</td>
        <td>走在闹市区的感觉，有点吵</td>
    </tr>
    <tr>
        <td>70 - 85dB</td>
        <td>汽车穿梭在马路上，85dB是保护听力的一般要求</td>
    </tr>
    <tr>
        <td>85 - 100dB</td>
        <td>摩托车启动，装修电钻</td>
    </tr>
    <tr>
        <td>100 - 150dB</td>
        <td>飞机起飞、燃放烟花爆竹</td>
    </tr>
</table>

#### 1.2 音调单位-频率(Hz)
　　频率是每秒经过一给定点的声波周期数量，其单位是Hz，1KHz表示每秒经过一给定点的声波有1000个周期。根据频率范围，我们将声波分为如下三种：

<table><tbody>
    <tr>
        <th style="width: 100px;">声波类别</th>
        <th style="width: 150px;">频率范围</th>
        <th style="width: 500px;">特性与应用</th>
    </tr>
    <tr>
        <td>次声波</td>
        <td>低于20Hz</td>
        <td>部分动物（狗、大象）能发出/感知，常用于自然灾害监测</td>
    </tr>
    <tr>
        <td>可闻声</td>
        <td>20Hz ~ 20KHz</td>
        <td>人的听觉感知范围</td>
    </tr>
    <tr>
        <td>超声波</td>
        <td>高于20KHz</td>
        <td>部分动物（狗、蝙蝠）能发出/感知，常用于深海探测(声呐)、医学检查(B超)</td>
    </tr>
</table>

　　关于声波频率特别要提的是，<font color="Blue">声波可以被分解为不同频率不同强度正弦波的叠加，这种变换（或分解）的过程，称为傅立叶变换(Fourier Transform)</font>。  

### 二、PCM编码原理
　　声波是一种在时间上和振幅上均连续的模拟量，在嵌入式里要想研究声波，首先需要将声波转换成一连串电压变化的模拟电信号，麦克风器件就是一种采集声波信号并将其转换成模拟电压信号输出的装置。  
　　有了声波的模拟电压信号，下一步需要将模拟信号数字化，即将模拟信号经过模数转换器（A/D）后变成数字信号，说白了就是将声音数字化。最常见的声音数字化方式就是脉冲编码调制PCM（Pulse Code Modulation），PCM是70年代末发展起来的技术，最早应用于由飞利浦和索尼公司共同推出的CD上，下图给出了PCM编码全过程：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Waveform_PCM-from_analog_to_digital.PNG" style="zoom:100%" />

　　从上图中我们可以看到PCM编码主要有三个过程：采样、量化、编码，在这过程中主要有4个参数用于评价PCM：<font color="Blue">声道数、采样率、量化位数、编码方式</font>。痞子衡会在下面逐一介绍PCM编码过程时穿插介绍这4个参数：  

#### 2.1 采样
　　所谓采样，即按一定的采样频率将模拟信号变成时间轴上离散的抽样信号的过程。原则上采样频率越高，声音的质量也就越好，声音的还原也就越真实。  
　　采样率即每秒从模拟信号中提取并组成离散信号的采样个数，用赫兹（Hz）来表示。说到采样率有一个不得不提的著名定律，即<font color="Blue">香农（Shannon）/奈奎斯特（Nyquist）采样定律，该定律表明采样频率必须大于或等于所传输的模拟信号的最高频率的2倍，才能不失真地恢复模拟信号</font>。  

> 最常说的“无损音频”，一般都是指传统CD格式中的44.1kHz/16bit的文件格式，而之所以称为无损压缩，是因为其包含了20Hz-22.05kHz这个完全覆盖人耳可闻范围的声音频率而得名。

　　关于声道数，其实非常好理解，就是采集声音的通道数，我们知道有单声道（mono），立体声（双声道stereo）、杜比7.1环绕声，其实就是声音采集的通道数有差别，声道越多，越能体现声音的空间立体效果。  

#### 2.2 量化
　　前面采样得到的抽样信号虽然是时间轴上离散的信号，但仍然是模拟信号，其采样值在一定的取值范围内，可有无限多个值，必须采用“四舍五入”的方法把样值分级“取整”，使一定取值范围内的样值由无限多个值变为有限个值，这一过程称为量化。  
　　量化位数指的是描述数字信号所使用的位数。如麦克风采集的电压范围为0-3.3V，8bit的量化精度为3.3V/256，16bit的量化精度为3.3V/65536。  

#### 2.3 编码
　　量化后的抽样信号就转化为按抽样时序排列的一串十进制数字码流，即十进制数字信号。简单高效的数据系统是二进制码系统，因此，应将十进制数字码变换成二进制编码，这种把量化的抽样信号变换成给定字长(量化位数)的二进制码流的过程称为编码。  
　　编码方式种类非常多，其对比可见 [Comparison of audio coding formats](https://en.wikipedia.org/wiki/Comparison_of_audio_coding_formats)，PCM音频格式编码常见有四种：[PCM](https://en.wikipedia.org/wiki/Pulse-code_modulation)（Linear PCM）、ADPCM（Adaptive differential PCM）、 [A-law](https://en.wikipedia.org/wiki/A-law_algorithm)（A律13折线码）、[μ-law](https://en.wikipedia.org/wiki/%CE%9C-law_algorithm)（μ律15折线码），最简单的当然是下图所示的[LPCM](https://commons.wikimedia.org/wiki/File:Pcm-ru.svg)（示例为4bit），这是一种均匀量化编码，广泛用于	Audio CD, AES3, WAV, AIFF, AU, M2TS, VOB中。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Waveform_PCM-lpcm_4bit.PNG" style="zoom:100%" />

　　除LPCM外，A-law和μ-law是两种不得不提的非均匀量化编码，这两种非均匀量化编码是为了提高小信号的信噪比，其基本思想是在量化之前先让信号经过一次处理，对大信号进行压缩而对小信号进行较大的放大，这一处理过程通常也称为“压缩量化”。压缩量化的实质是“压大补小”，使小信号在整个动态范围内的信噪比基本一致。下面是这两种编码与LPCM的 [对比图](https://en.wikipedia.org/wiki/File:Ulaw_alaw_db.svg)。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Waveform_PCM-Alaw_vs_Mlaw.PNG" style="zoom:100%" />

### 三、Waveform文件格式解析
　　前面讲的PCM编码后的声音数据是需要保存的，WAVE文件常常用来保存PCM编码数据。WAVE文件是微软公司（Microsoft）开发的一种声音文件格式，用于保存Windows平台的音频信息资源，被Windows平台及其应用程序所广泛支持，WAVE文件默认打开工具是WINDOWS的媒体播放器。  
#### 3.1 RIFF文件格式标准
　　WAVE文件是以微软RIFF格式为标准的，RIFF全称为资源互换文件格式（Resources Interchange File Format），是Windows下大部分多媒体文件遵循的一种文件结构。RIFF文件所包含的数据类型由该文件的扩展名来标识，能以RIFF格式存储的数据有很多：音频视频交错格式数据（.AVI）、波形格式数据（.WAV）、位图数据格式（.RDI）、MIDI格式数据（.RMI）、调色板格式（.PAL）、多媒体电影（.RMN）、动画光标（.ANI）等。  
　　如下代码所示的CK结构体是RIFF文件的基本单元，该基本单元也称 [Chunk](http://www.johnloomis.org/cpe102/asgn/asgn1/riff.html)。其中ckID用于标识块中所包含的数据类型，其取值可有'RIFF'、'LIST'、'fmt '、'data'等；ckSize表示存储在ckData域中的数据长度（不包含ckID和ckSize的大小）；ckData存储数据，数据以字节为单位存放，如果数据长度为奇数，则最后添加一个空字节。  

> 由于RIFF文件结构最初是由Microsoft和IBM为PC机所定义，RIFF文件是按照小端little-endian字节顺序写入的。  

```C
typedef unsigned long DWORD;
typedef unsigned char BYTE;
typedef DWORD         FOURCC;    // Four-character code

typedef struct { 
     FOURCC ckID;          // The unique chunk identifier 
     DWORD ckSize;         // The size of field <ckData> 
     BYTE ckData[ckSize];  // The actual data of the chunk 
} CK; 
```

　　Chunk是可以嵌套的，但是只有ckID为'RIFF'或者'LIST'的Chunk才能包含其他的Chunk。标志为'RIFF'的Chunk是比较特殊的，每一个RIFF文件首先存放的必须是一个'RIFF' Chunk，并且只能有这一个标志为'RIFF'的Chunk。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Waveform_PCM-riff_chunk.PNG" style="zoom:100%" />

　　更多RIFF的知识详见这个网站链接 [RIFF (Resource Interchange File Format)](https://www.loc.gov/preservation/digital/formats/fdd/fdd000025.shtml)，链接里收集了很多介绍RIFF的资源。  

#### 3.2 WAVE文件结构
　　WAVE是Microsoft开发的一种音频文件格式，它符合上面提到的RIFF文件格式标准，可以看作是RIFF文件的一个具体实例。既然WAVE符合RIFF规范，其基本的组成单元也是Chunk。一个 [WAVE文件](http://soundfile.sapp.org/doc/WaveFormat/) 通常有三个Chunk以及一个可选Chunk，其在文件中的排列方式依次是：RIFF Chunk，Format Chunk，Fact Chunk（附加块，可选），Data Chunk，如下图所示：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Waveform_PCM-wave_file_format.PNG" style="zoom:100%" />

　　根据上面的WAVE文件结构图，可以定义如下44bytes的wave_head_t用来描述WAVE文件的头。如果你曾经接触过Windows的音频接口API，你会发现wave_fmt_t中的部分结构与标准MSDN里的 [WAVEFORMAT](https://msdn.microsoft.com/en-us/library/ms713498.aspx) 是一致的。  

```C
typedef char    int8_t;    //有符号8位整数
typedef short   int16_t;   //有符号16位整数
typedef int     int32_t;   //有符号32位整数

struct _wave_tag
{
	int8_t     riff[4];            //"RIFF",资源交换文件标志
	int32_t    filesize;           //文件大小(从下个地址开始到文件尾的总字节数)
	int8_t     wave[4];            //"WAVE",文件标志
} wave_tag_t;
struct _wave_format
{
	int8_t     fmt[4];             //"fmt ",波形格式标志 
	int32_t    chunksize;          //文件内部Chunk信息大小
	int16_t    wFormatTag;         //音频数据编码方式
	int16_t    wChanles;           //声道数
	int32_t    nSamplesPerSec;     //采样率
	int32_t    nAvgBytesPerSec;    //波形数据传输速率（每秒平均字节数）
	int16_t    nBlockAlign;        //数据的调整数（按字节计算）
	int16_t    wBitsPerSample;     //样本数据位数
} wave_fmt_t;
struct _wave_data
{
	int8_t     data[4];            //"data",数据标志符
	int32_t    datasize;           //采样数据总长度
} wave_dat_t;

struct _wave_head
{
	wave_tag_t   waveTag;
	wave_fmt_t   waveFmt;
	wave_dat_t   waveDat;
} wave_head_t;
```

　　wave_head_t结构体内除了wFormatTag成员之外，其他都可以根据字面上的意思来理解。关于wFormatTag的具体定义，可见Windows SDK里的 [mmreg.h文件](http://www-mmsp.ece.mcgill.ca/Documents/AudioFormats/WAVE/Docs/Pages%20from%20mmreg.h.pdf)，下面列举了几个最常见Format的Tag值定义：

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Waveform_PCM-wave_format_code.PNG" style="zoom:100%" />

　　当WAVE文件的头被解析成功后，下一步便是获取WAVE文件里的声音源数据，我们知道声音文件有单声道和多声道之分，对于单声道文件很好理解，声音数据就是按序排放，而如果是立体声（2声道）文件，那么左右声道的声音数据到底是怎么排放的呢？下面以一个示例立体声文件数据（仅分析前72bytes）进行解释：  

```text
offset(h)
00000000: 52 49 46 46 24 08 00 00 57 41 56 45 66 6d 74 20
00000010: 10 00 00 00 01 00 02 00 22 56 00 00 88 58 01 00
00000020: 04 00 10 00 64 61 74 61 00 08 00 00 00 00 00 00
00000030: 24 17 1e f3 3c 13 3c 14 16 f9 18 f9 34 e7 23 a6
00000040: 3c f2 24 f2 11 ce 1a 0d
```

　　下图是这个72bytes数据解析图，从图中可以看到，左右声道的声音数据是按块（nBlockAlign指定）交替排放的。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Waveform_PCM-wave_file_stereo_example.PNG" style="zoom:100%" />

　　更多WAVE的知识详见这两个网站链接 [WAVE Audio File Format](https://www.loc.gov/preservation/digital/formats/fdd/fdd000001.shtml) 和 [Audio File Format Specifications](http://www-mmsp.ece.mcgill.ca/Documents/AudioFormats/WAVE/WAVE.html)，链接里收集了很多介绍WAVE的资源。  

#### 3.3 WAVE文件实例分析
　　WAVE文件格式我们都了解透彻了，下面我们尝试分析一个经典的WAVE文件："Windows XP 启动.wav"，这个文件可以说是最知名的WAVE文件了，痞子衡特别喜欢这段music，让我们直接用二进制编辑器HxD打开它：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Waveform_PCM-winxp_startup.PNG" style="zoom:100%" />

　　按wave_head_t解析WAVE头可知，这段wave是44.1kHz/16bit双声道线性PCM码音频，实际音频数据总长度为1361076bytes（1361076（datasize）/176400（nAvgBytesPerSec）=7.7158秒），最后再用[Adobe Audition（原Cool Edit）](https://www.adobe.com/products/audition/free-trial-download.html)打开查看其波形图：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/Waveform_PCM-winxp_startup_audition.PNG" style="zoom:100%" />

　　至此，PCM编码及Waveform音频文件格式痞子衡便介绍完毕了，掌声在哪里~~~ 

