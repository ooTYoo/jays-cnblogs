----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**语音处理工具tinyPySPEECH诞生之音频显示实现**。  

　　音频显示是JaysPySPEECH的主要功能，JaysPySPEECH借助的是Matplotlib以及NumPy来实现的音频显示功能，今天痞子衡为大家介绍音频显示在JaysPySPEECH中是如何实现的。  

### 一、SciPy工具集
　　SciPy是一套Python科学计算相关的工具集，其本身也是一个Python库，这个工具集主要包含以下6大Python库，JaysPySPEECH所用到的Matplotlib以及NumPy均属于SciPy工具集。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/JaysPySPEECH_Waveform_scipy_list.PNG" style="zoom:100%" />

#### 1.1 NumPy
　　NumPy是一套最基础的Python科学计算包，它主要用于数组与矩阵运算，它是一个开源项目，被收录进 [NumFOCUS](https://numfocus.org/) 组织维护的 [Sponsored Project](https://numfocus.org/sponsored-projects) 里。JaysPySPEECH使用的是NumPy 1.15.0。  
　　NumPy库的官方主页如下：  

> * NumPy官方主页: http://www.numpy.org/  
> * NumPy安装方法: https://pypi.org/project/numpy/  

　　NumPy的快速上手可参考这个网页 https://docs.scipy.org/doc/numpy/user/quickstart.html  

#### 1.2 Matplotlib
　　Matplotlib是一套Python高质量2D绘图库，它的初始设计者为John Hunter，它也是一个开源项目，被同样收录进 [NumFOCUS](https://numfocus.org/) 组织维护的 [Sponsored Project](https://numfocus.org/sponsored-projects) 里。JaysPySPEECH使用的是Matplotlib 2.2.3。  
　　Matplotlib库的官方主页如下：  

> * Matplotlib官方主页: https://matplotlib.org/  
> * Matplotlib安装方法: https://pypi.org/project/matplotlib/  

　　Matplotlib绘图功能非常强大，但是作为一般使用，我们没有必要去通读其官方文档，其提供了非常多的example代码，这些example都在 https://matplotlib.org/gallery/index.html， 我们只要找到能满足我们需求的example，在其基础上简单修改即可。 下面就是一个最简单的正弦波示例：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/JaysPySPEECH_Waveform_matplotlib_simpleplot.PNG" style="zoom:100%" />

```Python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np

# Data for plotting
t = np.arange(0.0, 2.0, 0.01)
s = 1 + np.sin(2 * np.pi * t)

fig, ax = plt.subplots()
ax.plot(t, s)

ax.set(xlabel='time (s)', ylabel='voltage (mV)',
       title='About as simple as it gets, folks')
ax.grid()

fig.savefig("test.png")
plt.show()
```

### 二、JaysPySPEECH音频显示实现
　　JaysPySPEECH关于音频显示功能实现主要有四点：选择.wav文件、读取.wav文件、绘制.wav波形、添加光标功能，最终JaysPySPEECH效果如下图所示，痞子衡为逐一为大家介绍实现细节。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/JaysPySPEECH_Waveform_final_view.PNG" style="zoom:100%" />

#### 2.1 选择.wav文件功能
　　选择wav文件主要借助的是wxPython里的genericDirCtrl控件提供的功能实现的，我们使用genericDirCtrl控件创建了一个名为m_genericDirCtrl_audioDir的对象，借助其SetFilter()方法实现了仅显示.wav文件格式的过滤，并且我们为m_genericDirCtrl_audioDir还创建了一个event，即viewAudio()，这个event的触发条件是选中m_genericDirCtrl_audioDir里列出的.wav文件，当viewAudio（）被触发时，我们通过GetFilePath()方法即可获得选中的.wav文件路径。  

```Python
class mainWin(jayspyspeech_win.speech_win):

    def __init__(self, parent):
        jayspyspeech_win.speech_win.__init__(self, parent)
		# ...
        self.m_genericDirCtrl_audioDir.SetFilter("Audio files (*.wav)|*.wav")

    def viewAudio( self, event ):
        self.wavPath =  self.m_genericDirCtrl_audioDir.GetFilePath()
```

#### 2.2 读取.wav文件功能
　　读取.wav文件主要借助的是python自带的标准库wave，以及第三方的NumPy库。痞子衡创建了一个名为wavCanvasPanel的类，在这个类中定义了readWave(self, wavPath, wavInfo)方法，其中参数wavPath即是要读取的.wav文件路径，参数wavInfo是GUI状态栏对象，用于直观显示读取到的.wav文件信息。  
　　在wavCanvasPanel.readWave()方法中，痞子衡首先使用了wave库里的功能获取到.wav文件的所有信息以及所有PCM数据，然后借助NumPy库将PCM数据按channel重新组织，便于后续图形显示。关于数据重新组织，有一个地方需要特别说明，即int24类型（3-byte）是不被NumPy中的fromstring()原生支持，因此痞子衡自己实现了一个非标准类型数据的fromstring()。  

```Python
import numpy
import wave

class wavCanvasPanel(wx.Panel):

    def fromstring(self, wavData, alignedByte):
        if alignedByte <= 8:
            src = numpy.ndarray(len(wavData), numpy.dtype('>i1'), wavData)
            dest = numpy.zeros(len(wavData) / alignedByte, numpy.dtype('>i8'))
            for i in range(alignedByte):
                dest.view(dtype='>i1')[alignedByte-1-i::8] = src.view(dtype='>i1')[i::alignedByte]
            [hex(x) for x in dest]
            return True, dest
        else:
            return False, wavData

    def readWave(self, wavPath, wavInfo):
        if os.path.isfile(wavPath):
            # Open the wav file to get wave data and parameters
            wavFile =  wave.open(wavPath, "rb")
            wavParams = wavFile.getparams()
            wavChannels = wavParams[0]
            wavSampwidth = wavParams[1]
            wavFramerate = wavParams[2]
            wavFrames = wavParams[3]
            wavInfo.SetStatusText('Opened Audio Info = ' +
                                  'Channels:' + str(wavChannels) +
                                  ', SampWidth:' + str(wavSampwidth) + 'Byte' +
                                  ', SampRate:' + str(wavFramerate) + 'kHz' +
                                  ', FormatTag:' + wavParams[4])
            wavData = wavFile.readframes(wavFrames)
            wavFile.close()
            # Transpose the wav data if wave has multiple channels
            if wavSampwidth == 1:
                dtype = numpy.int8
            elif wavSampwidth == 2:
                dtype = numpy.int16
            elif wavSampwidth == 3:
                dtype = None
            elif wavSampwidth == 4:
                dtype = numpy.float32
            else:
                return 0, 0, 0
            if dtype != None:
                retData = numpy.fromstring(wavData, dtype = dtype)
            else:
                # Implement int24 manually
                status, retData = self.fromstring(wavData, 3)
                if not status:
                    return 0, 0, 0
            if wavChannels != 1:
                retData.shape = -1, wavChannels
                retData = retData.T
            # Calculate and arange wave time
            retTime = numpy.arange(0, wavFrames) * (1.0 / wavFramerate)
            retChannels = wavChannels
            return retChannels, retData, retTime
        else:
            return 0, 0, 0
```

#### 2.3 绘制.wav波形功能
　　绘制.wav波形是最主要的功能。痞子衡在wavCanvasPanel类中实现了showWave(self, wavPath, wavInfo)方法，这个方法会在GUI控件m_genericDirCtrl_audioDir的事件函数viewAudio()中被调用。  
　　在wavCanvasPanel.showWave()方法中，痞子衡首先使用了readWave()获取.wav文件中经过重新组织的PCM数据，然后借助Matplotlib中的figure类中的add_axes()方法逐一将各channel的PCM数据绘制出来，并辅以各种信息（x、y轴精度、标签等）一同显示出来。由于GUI控件里专门用于显示波形的Panel对象尺寸为720*360 inch，痞子衡限制了最多显示.wav的前8通道。  

```Python
import matplotlib
from matplotlib.backends.backend_wxagg import FigureCanvasWxAgg as FigureCanvas
from matplotlib.figure import Figure

MAX_AUDIO_CHANNEL = 8
#unit: inch
PLOT_PANEL_WIDTH = 720
PLOT_PANEL_HEIGHT = 360
#unit: percent
PLOT_AXES_WIDTH_TITLE = 0.05
PLOT_AXES_HEIGHT_LABEL = 0.075

class wavCanvasPanel(wx.Panel):

    def __init__(self, parent):
        wx.Panel.__init__(self, parent)
        dpi = 60
        width = PLOT_PANEL_WIDTH / dpi
        height = PLOT_PANEL_HEIGHT / dpi
        self.wavFigure = Figure(figsize=[width,height], dpi=dpi, facecolor='#404040')
        self.wavCanvas = FigureCanvas(self, -1, self.wavFigure)
        self.wavSizer = wx.BoxSizer(wx.VERTICAL)
        self.wavSizer.Add(self.wavCanvas, 1, wx.EXPAND|wx.ALL)
        self.SetSizerAndFit(self.wavSizer)
        self.wavAxes = [None] * MAX_AUDIO_CHANNEL

    def readWave(self, wavPath, wavInfo):
        # ...

    def showWave(self, wavPath, wavInfo):
        self.wavFigure.clear()
        waveChannels, waveData, waveTime = self.readWave(wavPath, wavInfo)
        if waveChannels != 0:
            # Note: only show max supported channel if actual channel > max supported channel
            if waveChannels > MAX_AUDIO_CHANNEL:
                waveChannels = MAX_AUDIO_CHANNEL
            # Polt the waveform of each channel in sequence
            for i in range(waveChannels):
                left = PLOT_AXES_HEIGHT_LABEL
                bottom = (1.0 / waveChannels) * (waveChannels - 1 - i) + PLOT_AXES_HEIGHT_LABEL
                height = 1.0 / waveChannels - (PLOT_AXES_WIDTH_TITLE + PLOT_AXES_HEIGHT_LABEL)
                width = 1 - left - 0.05
                self.wavAxes[i] = self.wavFigure.add_axes([left, bottom, width, height], facecolor='k')
                self.wavAxes[i].set_prop_cycle(color='#00F279', lw=[1])
                self.wavAxes[i].set_xlabel('time (s)', color='w')
                self.wavAxes[i].set_ylabel('value', color='w')
                if waveChannels == 1:
                    data = waveData
                else:
                    data = waveData[i]
                self.wavAxes[i].plot(waveTime, data)
                self.wavAxes[i].grid()
                self.wavAxes[i].tick_params(labelcolor='w')
                self.wavAxes[i].set_title('Audio Channel ' + str(i), color='w')
        # Note!!!: draw() must be called if figure has been cleared once
        self.wavCanvas.draw()

class mainWin(jayspyspeech_win.speech_win):

    def __init__(self, parent):
        jayspyspeech_win.speech_win.__init__(self, parent)
        self.wavPanel = wavCanvasPanel(self.m_panel_plot)
        # ...

    def viewAudio( self, event ):
        self.wavPath =  self.m_genericDirCtrl_audioDir.GetFilePath()
        self.wavPanel.showWave(self.wavPath, self.statusBar)
```

#### 2.4 添加光标功能
　　光标定位功能不是必要功能，但其可以让软件看起来高大上，痞子衡创建了一个名为wavCursor类来实现它，主要在这个类中实现了moveMouse方法，这个方法将会被FigureCanvasWxAgg类中的mpl_connect()方法添加到各通道axes中。  

```Python
MAX_AUDIO_CHANNEL = 8

class wavCursor(object):
    def __init__(self, ax, x, y):
        self.ax = ax
        self.vline = ax.axvline(color='r', alpha=1)
        self.hline = ax.axhline(color='r', alpha=1)
        self.marker, = ax.plot([0],[0], marker="o", color="crimson", zorder=3)
        self.x = x
        self.y = y
        self.xlim = self.x[len(self.x)-1]
        self.text = ax.text(0.7, 0.9, '', bbox=dict(facecolor='red', alpha=0.5))

    def moveMouse(self, event):
        if not event.inaxes:
            return
        x, y = event.xdata, event.ydata
        if x > self.xlim:
            x = self.xlim
        index = numpy.searchsorted(self.x, [x])[0]
        x = self.x[index]
        y = self.y[index]
        self.vline.set_xdata(x)
        self.hline.set_ydata(y)
        self.marker.set_data([x],[y])
        self.text.set_text('x=%1.2f, y=%1.2f' % (x, y))
        self.text.set_position((x,y))
        self.ax.figure.canvas.draw_idle()

class wavCanvasPanel(wx.Panel):
    def __init__(self, parent):
        # ...
        self.wavAxes = [None] * MAX_AUDIO_CHANNEL
		# 定义光标对象
        self.wavCursor = [None] * MAX_AUDIO_CHANNEL

    def showWave(self, wavPath, wavInfo):
        # ...
        if waveChannels != 0:
            # ...
            for i in range(waveChannels):
                # ...
                self.wavAxes[i].set_title('Audio Channel ' + str(i), color='w')
				# 实例化光标对象，并使用mpl_connect()将moveMouse()动作加入光标对象
                self.wavCursor[i] = wavCursor(self.wavAxes[i], waveTime, data)
                self.wavCanvas.mpl_connect('motion_notify_event', self.wavCursor[i].moveMouse)
        # ...
```

　　至此，语音处理工具tinyPySPEECH诞生之音频显示实现痞子衡便介绍完毕了，掌声在哪里~~~  

### 参考文档
> 1. [Embedding a matplotlib figure inside a WxPython panel](https://stackoverflow.com/questions/10737459/embedding-a-matplotlib-figure-inside-a-wxpython-panel)
> 2. [软妹子带你玩转Python数据可视化Axes绘图布局方法介绍](https://baijiahao.baidu.com/s?id=1575393755013387&wfr=spider&for=pc)

