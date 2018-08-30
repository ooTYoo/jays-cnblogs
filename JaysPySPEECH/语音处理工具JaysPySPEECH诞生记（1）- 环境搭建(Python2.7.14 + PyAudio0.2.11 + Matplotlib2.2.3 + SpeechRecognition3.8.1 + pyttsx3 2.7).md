----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**语音处理工具JaysPySPEECH诞生之环境搭建**。  

　　在写JaysPySPEECH时需要先搭好开发环境，下表列出了开发过程中会用到的所有软件/工具包：  

### 一、涉及工具列表

<table><tbody>
    <tr>
        <th style="width: 200px;">工具</th>
        <th style="width: 400px;">功能</th>
        <th style="width: 350px;">下载地址</th>
    </tr>
    <tr>
        <td>Python 2.7.14</td>
        <td>Python官方包（解释器）</td>
        <td><a href="https://www.python.org/">https://www.python.org/</a></td>
    </tr>
    <tr>
        <td>PyAudio 0.2.11</td>
        <td>跨平台开源Audio I/O库 <a href="http://www.portaudio.com/">PortAudio</a> 的Python封装</td>
        <td><a href="http://people.csail.mit.edu/hubert/pyaudio/">http://people.csail.mit.edu/hubert/pyaudio/</a></td>
    </tr>
    <tr>
        <td>Matplotlib 2.2.3</td>
        <td>一款非常强大的Python 2D绘图库</td>
        <td><a href="https://matplotlib.org/">https://matplotlib.org/</a><br>
            <a href="https://github.com/matplotlib/matplotlib">https://github.com/matplotlib/matplotlib</a><br>
        </td>
    </tr>
    <tr>
        <td>NumPy 1.15.0</td>
        <td>基础Python科学计算包</td>
        <td><a href="http://www.numpy.org/">http://www.numpy.org/</a><br>
            <a href="https://www.scipy.org/">https://www.scipy.org/</a><br>
        </td>
    </tr>
    <tr>
        <td>SpeechRecognition 3.8.1</td>
        <td>一款支持多引擎的Python语音识别（ASR）库</td>
        <td><a href="https://github.com/Uberi/speech_recognition">https://github.com/Uberi/speech_recognition</a></td>
    </tr>
    <tr>
        <td>PocketSphinx 0.1.15</td>
        <td>卡内基-梅隆大学开源语音识别引擎 <a href="https://cmusphinx.github.io/">CMU Sphinx</a> 的Python封装</td>
        <td><a href="https://github.com/bambocher/pocketsphinx-python">https://github.com/bambocher/pocketsphinx-python</a><br>
		    <a href="https://pypi.org/project/pocketsphinx/">https://pypi.org/project/pocketsphinx/</a>
		</td>
    </tr>
    <tr>
        <td>pyttsx3 2.7</td>
        <td>pyTTS, pyttsx项目的延续之作，一款轻量级的Python文语合成引擎</td>
        <td><a href="https://github.com/nateshmbhat/pyttsx3">https://github.com/nateshmbhat/pyttsx3</a><br>
            <a href="https://pypi.org/project/pyttsx3/">https://pypi.org/project/pyttsx3/</a>
		</td>
    </tr>
    <tr>
        <td>wxPython 4.0.3</td>
        <td>跨平台开源GUI库 <a href="https://www.wxwidgets.org/">wxWidgets</a> 的Python封装库</td>
        <td><a href="https://www.wxpython.org/">https://www.wxpython.org/</a><br>
            <a href="https://pypi.org/project/wxPython/">https://pypi.org/project/wxPython/</a>
        </td>
    </tr>
    <tr>
        <td>wxFormBuilder 3.8.0</td>
        <td>wxPython GUI界面构建工具</td>
        <td><a href="https://github.com/wxFormBuilder/wxFormBuilder">https://github.com/wxFormBuilder/wxFormBuilder</a></td>
    </tr>
    <tr>
        <td>PyCharm Community 2018.02</td>
        <td>一款流行的Python集成开发环境</td>
        <td><a href="http://www.jetbrains.com/pycharm/">http://www.jetbrains.com/pycharm/</a></td>
    </tr>
</table>

### 二、基础环境搭建（Python + PyAudio + Matplotlib + NumPy）
　　JaysPySPEECH工具是一个完全基于Python语言开发的应用软件，首先安装好Python 2.7.14，痞子衡的安装目录为C:\tools_mcu\Python27，安装完成后确保系统环境变量里包括该路径（C:\tools_mcu\Python27），因为该路径下包含python.exe，后续python命令需调用这个python.exe完成的。此外pip是Python的包管理工具，我们可以借助pip来安装PyAudio和Matplotlib包（NumPy含在Matplotlib里）：  

> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue"> .\pip.exe install pyaudio</font>
> ```text
> Collecting pyaudio
>  Downloading https://files.pythonhosted.org/packages/94/3e/430d4e4e24e89b19c1df052644f69e03d64c1ae2e83f5a14bd365e0236de/PyAudio-0.2.11-cp27-cp27m-win_amd64.whl (52kB)
> Installing collected packages: pyaudio
> Successfully installed pyaudio-0.2.11
> ```
>
> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue"> .\pip.exe install matplotlib</font>
> ```text
> Collecting matplotlib
>   Downloading https://files.pythonhosted.org/packages/f7/5b/4bc804df462961a3f0d138243611ce24b7899db04e6043e46df0ff1080e9/matplotlib-2.2.3-cp27-cp27m-win_amd64.whl (8.4MB)
> Collecting backports.functools-lru-cache (from matplotlib)
>   Downloading https://files.pythonhosted.org/packages/03/8e/2424c0e65c4a066e28f539364deee49b6451f8fcd4f718fefa50cc3dcf48/backports.functools_lru_cache-1.5-py2.py3-none-any.whl
> Requirement already satisfied, skipping upgrade: six>=1.10 in c:\tools_mcu\python27\lib\site-packages (from matplotlib) (1.11.0)
> Collecting pytz (from matplotlib)
>   Downloading https://files.pythonhosted.org/packages/30/4e/27c34b62430286c6d59177a0842ed90dc789ce5d1ed740887653b898779a/pytz-2018.5-py2.py3-none-any.whl (510kB)
> Collecting cycler>=0.10 (from matplotlib)
>   Downloading https://files.pythonhosted.org/packages/f7/d2/e07d3ebb2bd7af696440ce7e754c59dd546ffe1bbe732c8ab68b9c834e61/cycler-0.10.0-py2.py3-none-any.whl
> Collecting kiwisolver>=1.0.1 (from matplotlib)
>   Downloading https://files.pythonhosted.org/packages/e0/3a/2fda27dacdfafcf8f40cce2be09890b1443af3e65c3ab8f7294216a2946b/kiwisolver-1.0.1-cp27-none-win_amd64.whl (64kB)
> Collecting python-dateutil>=2.1 (from matplotlib)
>   Downloading https://files.pythonhosted.org/packages/cf/f5/af2b09c957ace60dcfac112b669c45c8c97e32f94aa8b56da4c6d1682825/python_dateutil-2.7.3-py2.py3-none-any.whl (211kB)
> Collecting pyparsing!=2.0.4,!=2.1.2,!=2.1.6,>=2.0.1 (from matplotlib)
>   Downloading https://files.pythonhosted.org/packages/6a/8a/718fd7d3458f9fab8e67186b00abdd345b639976bc7fb3ae722e1b026a50/pyparsing-2.2.0-py2.py3-none-any.whl (56kB)
> Collecting numpy>=1.7.1 (from matplotlib)
>   Downloading https://files.pythonhosted.org/packages/3d/d6/f04730ad69240be04584b3979dcd2f0b25f9e58463547df6fcafa139c567/numpy-1.15.0-cp27-none-win_amd64.whl (13.5MB)
> Requirement already satisfied, skipping upgrade: setuptools in c:\tools_mcu\python27\lib\site-packages (from kiwisolver>=1.0.1->matplotlib) (28.8.0)
> Installing collected packages: backports.functools-lru-cache, pytz, cycler, kiwisolver, python-dateutil, pyparsing, numpy, matplotlib
> Successfully installed backports.functools-lru-cache-1.5 cycler-0.10.0 kiwisolver-1.0.1 matplotlib-2.2.3 numpy-1.15.0 pyparsing-2.2.0 python-dateutil-2.7.3 pytz-2018.5
> ```

　　有了PyAudio便可以读写Audio，有了Matplotlib便可以将Audio以波形方式图形化显示出来。这两个工具安装完成，JaysPySPEECH工具开发的Python基础环境便搭好了。  

> Note: 关于GUI及调试等相关工具（wxPython、wxFormBuilder、PyCharm）的安装详见痞子衡另一个作品 [tinyPyCOM的环境搭建](http://www.cnblogs.com/henjay724/p/9416049.html)。

### 二、高级环境搭建（SpeechRecognition + PocketSphinx + pyttsx3）
　　上一步主要安装了JaysPySPEECH的基础开发环境，用于Audio的录播与显示，但是JaysPySPEECH设计之初便考虑支持语音识别、文语转换功能，因为我们还需要进一步安装相关Python库。  
　　首先安装语音识别库，SpeechRecognition是一款非常流行的支持多引擎的语音识别Python库，痞子衡为JaysPySPEECH选用的就是SpeechRecognition，其中语音识别引擎选用的是可以离线工作的PocketSphinx，具体安装如下：  

> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue">  .\pip.exe install SpeechRecognition</font>
> ```text
> Collecting SpeechRecognition
>   Downloading https://files.pythonhosted.org/packages/26/e1/7f5678cd94ec1234269d23756dbdaa4c8cfaed973412f88ae8adf7893a50/SpeechRecognition-3.8.1-py2.py3-none-any.whl (32.8MB)
> Installing collected packages: SpeechRecognition
> Successfully installed SpeechRecognition-3.8.1
> ```
>
> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue"> python -m pip install --upgrade pip setuptools wheel</font>
> ```text
> Requirement already up-to-date: pip in c:\tools_mcu\python27\lib\site-packages (18.0)
> Collecting setuptools
>   Downloading https://files.pythonhosted.org/packages/66/e8/570bb5ca88a8bcd2a1db9c6246bb66615750663ffaaeada95b04ffe74e12/setuptools-40.2.0-py2.py3-none-any.whl (568kB)
> Collecting wheel
>   Downloading https://files.pythonhosted.org/packages/81/30/e935244ca6165187ae8be876b6316ae201b71485538ffac1d718843025a9/wheel-0.31.1-py2.py3-none-any.whl (41kB)
> Installing collected packages: setuptools, wheel
>   Found existing installation: setuptools 28.8.0
>     Uninstalling setuptools-28.8.0:
>       Successfully uninstalled setuptools-28.8.0
> Successfully installed setuptools-40.2.0 wheel-0.31.1
> ```
>
> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue"> .\pip.exe install --upgrade pocketsphinx</font>
> ```text
> Collecting pocketsphinx
>   Downloading https://files.pythonhosted.org/packages/38/d3/192476022e989377ab00cb84fb0b18790e400bbd58e464155c58cb4622f8/pocketsphinx-0.1.15-cp27-cp27m-win_amd64.whl (29.1MB)
> Installing collected packages: pocketsphinx
> Successfully installed pocketsphinx-0.1.15
> ```

　　最后安装文语合成库，pyttsx3是一款超轻量级的文语合成Python库，其是经典的pyTTS、pyttsx项目的延续，其内核为Microsoft Speech API (SAPI5)，可离线工作，具体安装如下：  

> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue"> .\pip.exe install pyttsx3</font>
> ```text
> Collecting pyttsx3
>   Downloading https://files.pythonhosted.org/packages/24/4e/580726c73272344d3e74b7aaffae55ff6b6450061fbecb8cc6e112531c02/pyttsx3-2.7.tar.gz
> Requirement already satisfied: pypiwin32 in c:\tools_mcu\python27\lib\site-packages (from pyttsx3) (223)
> Requirement already satisfied: pywin32>=223 in c:\tools_mcu\python27\lib\site-packages (from pypiwin32->pyttsx3) (223)
> Building wheels for collected packages: pyttsx3
>   Running setup.py bdist_wheel for pyttsx3 ... done
>   Stored in directory: C:\Users\nxa07314\AppData\Local\pip\Cache\wheels\a2\8a\fe\11112aca9c89142c3a404bc67ef3393a7ad530da26639a05d4
> Successfully built pyttsx3
> Installing collected packages: pyttsx3
> Successfully installed pyttsx3-2.7
> ```

　　到了这里，JaysPySPEECH工具开发的Python环境便全部搭好了。

　　至此，语音处理工具JaysPySPEECH诞生之环境搭建痞子衡便介绍完毕了，掌声在哪里~~~  

