----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**语音处理工具Jays-PySPEECH诞生之语音识别实现**。  

　　语音识别是Jays-PySPEECH的核心功能，Jays-PySPEECH借助的是SpeechRecognition系统以及CMU Sphinx引擎来实现的语音识别功能，今天痞子衡为大家介绍语音识别在Jays-PySPEECH中是如何实现的。  

### 一、SpeechRecognition系统简介
　　SpeechRecognition是一套基于python实现语音识别的系统，该系统的设计者为 [Anthony Zhang (Uberi)](https://anthony-zhang.me/)，该库从2014年开始推出，一直持续更新至今，Jays-PySPEECH使用的是SpeechRecognition 3.8.1。  
　　SpeechRecognition系统的官方主页如下：  

> * SpeechRecognition官方主页: https://github.com/Uberi/speech_recognition  
> * SpeechRecognition安装方法: https://pypi.org/project/SpeechRecognition/  

　　SpeechRecognition系统自身并没有语音识别功能，其主要是调用第三方语音识别引擎来实现语音识别，SpeechRecognition支持的语音识别引擎非常多，有如下8种：  

> * CMU Sphinx (works offline)
> * Google Speech Recognition
> * Google Cloud Speech API
> * Wit.ai
> * Microsoft Bing Voice Recognition
> * Houndify API
> * IBM Speech to Text
> * Snowboy Hotword Detection (works offline)

　　不管是选用哪一种语音识别引擎，在SpeechRecognition里调用接口都是一致的，我们以实现音频文件转文字的示例代码 [audio_transcribe.py](https://github.com/Uberi/speech_recognition/blob/master/examples/audio_transcribe.py) 为例了解SpeechRecognition的用法，截取audio_transcribe.py部分内容如下：   

```Python
import speech_recognition as sr

# 指定要转换的音频源文件（english.wav）
from os import path
AUDIO_FILE = path.join(path.dirname(path.realpath(__file__)), "english.wav")

# 定义SpeechRecognition对象并获取音频源文件（english.wav）中的数据
r = sr.Recognizer()
with sr.AudioFile(AUDIO_FILE) as source:
    audio = r.record(source)  # read the entire audio file

# 使用CMU Sphinx引擎去识别音频
try:
    print("Sphinx thinks you said " + r.recognize_sphinx(audio))
except sr.UnknownValueError:
    print("Sphinx could not understand audio")
except sr.RequestError as e:
    print("Sphinx error; {0}".format(e))

# 使用Microsoft Bing Voice Recognition引擎去识别音频
BING_KEY = "INSERT BING API KEY HERE"  # Microsoft Bing Voice Recognition API keys 32-character lowercase hexadecimal strings
try:
    print("Microsoft Bing Voice Recognition thinks you said " + r.recognize_bing(audio, key=BING_KEY))
except sr.UnknownValueError:
    print("Microsoft Bing Voice Recognition could not understand audio")
except sr.RequestError as e:
print("Could not request results from Microsoft Bing Voice Recognition service; {0}".format(e))

# 使用其他引擎去识别音频
# ... ...
```

　　有木有觉得SpeechRecognition使用起来特别简单？是的，这正是SpeechRecognition系统强大之处，更多示例可见 [https://github.com/Uberi/speech_recognition/tree/master/examples](https://github.com/Uberi/speech_recognition/tree/master/examples)。  

#### 1.1 选用CMU Sphinx引擎
　　前面痞子衡讲了SpeechRecognition系统自身并没有语音识别功能，因此我们需要为SpeechRecognition安装一款语音识别引擎，痞子衡为JaysPySPEECH选用的是可离线工作的CMU Sphinx。  
　　CMU Sphinx是卡内基梅隆大学开发的一款开源语音识别引擎，该引擎可以离线工作，并且支持多语种（英语、中文、法语等）。CMU Sphinx引擎的官方主页如下：  

> * CMU Sphinx官方主页: https://cmusphinx.github.io/  
> * CMU Sphinx官方下载: https://sourceforge.net/projects/cmusphinx/  

　　由于JaysPySPEECH是基于Python环境开发的，因此我们不能直接用CMU Sphinx，那该怎么办？别着急，Dmitry Prazdnichnov大牛为CMU Sphinx写了Python封装接口，即PocketSphinx，其官方主页如下：  

> * PocketSphinx官方主页: https://github.com/bambocher/pocketsphinx-python  
> * PocketSphinx安装方法: https://pypi.org/project/pocketsphinx/  

　　我们在JaysPySPEECH诞生系列文章第一篇 [环境搭建](https://www.cnblogs.com/henjay724/p/9542690.html) 里已经安装了SpeechRecognition和PocketSphinx，痞子衡的安装路径为C:\tools_mcu\Python27\Lib\site-packages下的\speech_recognition与\pocketsphinx，安装好这两个包，引擎便选好了。  

#### 1.2 为PocketSphinx引擎增加中文语言包
　　默认情况下，PocketSphinx仅支持US English语言的识别，在C:\tools_mcu\Python27\Lib\site-packages\speech_recognition\pocketsphinx-data目录下仅能看到en-US文件夹，先来看一下这个文件夹里有什么:  

```text
\pocketsphinx-data\en-US
                        \acoustic-model                     --声学模型
                                       \feat.params           --HMM模型的特征参数
                                       \mdef                  --模型定义文件
                                       \means                 --混合高斯模型的均值
                                       \mixture_weights       --混合权重
                                       \noisedict             --噪声也就是非语音字典
                                       \sendump               --从声学模型中获取混合权重
                                       \transition_matrices   --HMM模型的状态转移矩阵
                                       \variances             --混合高斯模型的方差
						\language-model.lm.bin              --语言模型
						\pronounciation-dictionary.dict     --拼音字典
```

　　看到这一堆文件是不是觉得有点难懂？这其实跟CMU Sphinx引擎的语音识别原理有关，此处我们暂且不深入了解，对我们调用API的应用来说只需要关于如何为CMU Sphinx增加其他语言包（比如中文包）。  
　　要想增加其他语言，首先得要有语言包数据，CMU Sphinx主页提供了12种主流语言包的下载 [https://sourceforge.net/projects/cmusphinx/files/Acoustic_and_Language_Models/](https://sourceforge.net/projects/cmusphinx/files/Acoustic%20and%20Language%20Models/)，因为JaysPySPEECH需要支持中文识别，因此我们需要下载\Mandarin下面的三个文件：  

```text
\Mandarin
		 \zh_broadcastnews_16k_ptm256_8000.tar.bz2  --声学模型
         \zh_broadcastnews_64000_utf8.DMP           --语言模型
		 \zh_broadcastnews_utf8.dic                 --拼音字典
```

　　有了中文语言包数据，然后我们需要根据 [Notes on using PocketSphinx](https://github.com/Uberi/speech_recognition/blob/master/reference/pocketsphinx.rst) 里指示的步骤操作，痞子衡整理如下：  

> 1. \speech_recognition\pocketsphinx-data目录下创建zh-CN文件夹
> 2. 将zh_broadcastnews_16k_ptm256_8000.tar.bz2解压缩并里面所有文件放入\zh-CN\acoustic-model文件夹下
> 3. 将zh_broadcastnews_utf8.dic重命名为pronounciation-dictionary.dict并放入\zh-CN文件夹下
> 4. 借助SphinxBase工具将zh_broadcastnews_64000_utf8.DMP转换成language-model.lm.bin并放入\zh-CN文件夹下

　　关于第4步里提到的SphinxBase工具，我们需要从 [https://github.com/cmusphinx/sphinxbase](https://github.com/cmusphinx/sphinxbase) 里下载源码，然后使用Visual Studio 2010（或以上）打开\sphinxbase\sphinxbase.sln工程Rebuild All后会在\sphinxbase\bin\Release\x64下看到生成了如下6个工具：  

```text
\\sphinxbase\bin\Release\x64
		                    \sphinx_cepview.exe
                            \sphinx_fe.exe
		                    \sphinx_jsgf2fsg.exe
		                    \sphinx_lm_convert.exe
		                    \sphinx_pitch.exe
		                    \sphinx_seg.exe
```

　　我们主要使用sphinx_lm_convert.exe工具完成转换工作生成language-model.lm.bin，具体命令如下：  


> PS C:\tools_mcu\sphinxbase\bin\Release\x64><font style="font-weight:bold;" color="Blue"> .\sphinx_lm_convert.exe -i .\zh_broadcastnews_64000_utf8.DMP -o language-model.lm - ofmt arpa</font>
> ```text
> Current configuration:
> [NAME]          [DEFLT] [VALUE]
> -case
> -help           no      no
> -i                      .\zh_broadcastnews_64000_utf8.DMP
> -ifmt
> -logbase        1.0001  1.000100e+00
> -mmap           no      no
> -o                      language-model.lm
> -ofmt                   arpa
>
> INFO: ngram_model_trie.c(354): Trying to read LM in trie binary format
> INFO: ngram_model_trie.c(365): Header doesn't match
> INFO: ngram_model_trie.c(177): Trying to read LM in arpa format
> INFO: ngram_model_trie.c(70): No \data\ mark in LM file
> INFO: ngram_model_trie.c(445): Trying to read LM in dmp format
> INFO: ngram_model_trie.c(527): ngrams 1=63944, 2=16600781, 3=20708460
> INFO: lm_trie.c(474): Training quantizer
> INFO: lm_trie.c(482): Building LM trie
> ```
>
> PS C:\tools_mcu\sphinxbase\bin\Release\x64><font style="font-weight:bold;" color="Blue"> .\sphinx_lm_convert.exe -i .\language-model.lm -o language-model.lm.bin</font>
> ```text
> Current configuration:
> [NAME]          [DEFLT] [VALUE]
> -case
> -help           no      no
> -i                      .\language-model.lm
> -ifmt
> -logbase        1.0001  1.000100e+00
> -mmap           no      no
> -o                      language-model.lm.bin
> -ofmt
>
> INFO: ngram_model_trie.c(354): Trying to read LM in trie binary format
> INFO: ngram_model_trie.c(365): Header doesn't match
> INFO: ngram_model_trie.c(177): Trying to read LM in arpa format
> INFO: ngram_model_trie.c(193): LM of order 3
> INFO: ngram_model_trie.c(195): #1-grams: 63944
> INFO: ngram_model_trie.c(195): #2-grams: 16600781
> INFO: ngram_model_trie.c(195): #3-grams: 20708460
> INFO: lm_trie.c(474): Training quantizer
> INFO: lm_trie.c(482): Building LM trie
> ```

### 二、Jays-PySPEECH语音识别实现
　　语音识别代码实现其实很简单，直接调用speech_recognition里的API即可，目前仅实现了CMU Sphinx引擎，并且仅支持中英双语识别。具体到Jays-PySPEECH上主要是实现GUI界面上"ASR"按钮的回调函数，即audioSpeechRecognition()，如果用户选定了配置参数（语言类型、ASR引擎类型），并点击了"ASR"按钮，此时便会触发audioSpeechRecognition()的执行。代码如下：  

```Python
import speech_recognition

class mainWin(win.speech_win):

    def getLanguageSelection(self):
        languageType = self.m_choice_lang.GetString(self.m_choice_lang.GetSelection())
        if languageType == 'Mandarin Chinese':
            languageType = 'zh-CN'
            languageName = 'Chinese'
        else: # languageType == 'US English':
            languageType = 'en-US'
            languageName = 'English'
        return languageType, languageName

    def audioSpeechRecognition( self, event ):
        if os.path.isfile(self.wavPath):
		    # 创建speech_recognition语音识别对象asrObj
            asrObj = speech_recognition.Recognizer()
			# 获取wav文件里的语音内容
            with speech_recognition.AudioFile(self.wavPath) as source:
                speechAudio = asrObj.record(source)
            self.m_textCtrl_asrttsText.Clear()
            # 获取语音语言类型（English/Chinese）
            languageType, languageName = self.getLanguageSelection()
            engineType = self.m_choice_asrEngine.GetString(self.m_choice_asrEngine.GetSelection())
            if engineType == 'CMU Sphinx':
                try:
				    # 调用recognize_sphinx完成语音识别
                    speechText = asrObj.recognize_sphinx(speechAudio, language=languageType)
					# 语音识别结果显示在asrttsText文本框内
                    self.m_textCtrl_asrttsText.write(speechText)
                    self.statusBar.SetStatusText("ASR Conversation Info: Successfully")
					# 语音识别结果写入指定文件
                    fileName = self.m_textCtrl_asrFileName.GetLineText(0)
                    if fileName == '':
                        fileName = 'asr_untitled1.txt'
                    asrFilePath = os.path.join(os.path.dirname(os.path.abspath(os.path.dirname(__file__))), 'conv', 'asr', fileName)
                    asrFileObj = open(asrFilePath, 'wb')
                    asrFileObj.write(speechText)
                    asrFileObj.close()
                except speech_recognition.UnknownValueError:
                    self.statusBar.SetStatusText("ASR Conversation Info: Sphinx could not understand audio")
                except speech_recognition.RequestError as e:
                    self.statusBar.SetStatusText("ASR Conversation Info: Sphinx error; {0}".format(e))
            else:
                self.statusBar.SetStatusText("ASR Conversation Info: Unavailable ASR Engine")

```

　　至此，语音处理工具Jays-PySPEECH诞生之语音识别实现痞子衡便介绍完毕了，掌声在哪里~~~  
