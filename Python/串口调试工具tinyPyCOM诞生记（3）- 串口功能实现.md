----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**串口调试工具tinyPyCOM诞生之串口功能实现**。  

　　串口调试助手是最核心的当然是串口数据收发与显示的功能，tinyPyCOM借助的是pySerial库实现串口收发功能，今天痞子衡为大家介绍pySerial是如何在tinyPyCOM发挥功能的。  

### 一、pySerial简介
　　pySerial是一套基于python实现serial port访问的库，该库的设计者为Chris Liechti，该库从2001年开始推出，一直持续更新至今，tinyPyCOM使用的是pySerial 3.4。  
　　pySerial的使用非常简单，可在其官网浏览一遍其提供的API： [https://pythonhosted.org/pyserial/pyserial_api.html](https://pythonhosted.org/pyserial/pyserial_api.html)，下面痞子衡整理了比较常用的API如下：  

```Python
class Serial(SerialBase):
    # 初始化串口参数
    def __init__(self, *args, **kwargs):
    # 打开串口
    def open(self):
    # 关闭串口
    def close(self):
    # 获取串口打开状态
    def isOpen(self):
    # 设置input_buffer/output_buffer大小
    def set_buffer_size(self, rx_size=4096, tx_size=None):

    # 获取input_buffer（接收缓冲区）里的byte数据个数
    def inWaiting(self):
    # 从串口读取size个byte数据
    def read(self, size=1):
    # 清空input_buffer
    def reset_input_buffer(self):

    # 向串口写入data里所有数据
    def write(self, data):
    # 等待直到output_buffer里的数据全部发送出去
    def flush(self):
    # 清空output_buffer
    def reset_output_buffer(self):
```

　　pySerial常用参数整理如下，需要特别强调的是任何运行时刻对如下参数进行修改，均是直接应用生效的，不需要重新调用open()和close()去激活，因为参数的修改在pySerial内部是通过与参数同名的方法实现的，而这些方法均调用了Serial里的一个叫_reconfigure_port()的方法实现的。  

<table><tbody>
    <tr>
        <th style="width: 150px;">参数名</th>
        <th style="width: 100px;">功能解释</th>
        <th style="width: 500px;">备注/可设值</th>
    </tr>
    <tr>
        <td>port</td>
        <td>设备名</td>
        <td>/dev/ttyUSB0 on GNU/Linux or COM3 on Windows</td>
    </tr>
    <tr>
        <td>baudrate (int)</td>
        <td>波特率</td>
        <td>/</td>
    </tr>
    <tr>
        <td>bytesize</td>
        <td>数据位bit个数</td>
        <td>FIVEBITS, SIXBITS, SEVENBITS, EIGHTBITS</td>
    </tr>
    <tr>
        <td>stopbits</td>
        <td>停止位</td>
        <td>STOPBITS_ONE, STOPBITS_ONE_POINT_FIVE, STOPBITS_TWO</td>
    </tr>
    <tr>
        <td>parity</td>
        <td>奇偶校验位</td>
        <td>PARITY_NONE, PARITY_EVEN, PARITY_ODD PARITY_MARK, PARITY_SPACE</td>
    </tr>
    <tr>
        <td>timeout (float)</td>
        <td>接收超时</td>
        <td>None：blocking mode<br>
		    0: non-blocking mode<br>
			x: 超时时间x秒</td>
    </tr>
    <tr>
        <td>write_timeout (float)</td>
        <td>发送超时</td>
        <td>同timeout</td>
    </tr>
</table>

### 二、tinyPyCOM串口功能实现
　　串口功能代码实现主要分为三大部分：配置功能实现、接收功能实现、发送功能实现。在实现这些功能之前首先需要import两个module，分别是serial、threading，serial就是pySerial库；threading是python自带线程库，其具体作用下面代码里会介绍。  
　　除此以外还定义两个全局变量，s_serialPort和s_recvInterval，s_serialPort是串口设备object实例，s_recvInterval是线程间隔时间。  
```Python
import serial
import threading

s_serialPort = serial.Serial()
s_recvInterval = 0.5
```

#### 2.1串口配置功能
　　串口配置里主要就是实现GUI界面上"Open/Close"按钮的回调函数，即openClosePort()，软件刚打开时所有可用Port默认是Close状态，如果用户选定了配置参数（串口号、波特率...），并点击了"Open/Close"按钮，此时便会触发openClosePort()的执行，在openClosePort()里我们需要配置s_serialPort的参数并打开指定的串口设备。  

```Python
class mainWin(tinypycom_win.com_win):

    def setPort ( self ):
        s_serialPort.port = self.m_textCtrl_comPort.GetLineText(0)
    def setBaudrate ( self ):
        index = self.m_choice_baudrate.GetSelection()
        s_serialPort.baudrate = int(self.m_choice_baudrate.GetString(index))
    def setDatabits ( self ):
        # ...
    def setStopbits ( self ):
        # ...
    def setParitybits ( self ):
        # ...

    def openClosePort( self, event ):
        if s_serialPort.isOpen():
            s_serialPort.close()
            self.m_button_openClose.SetLabel('isClosed->Open')
        else:
		    # 获取GUI配置面板里的输入值赋给s_serialPort
            self.setPort()
            self.setBaudrate()
            self.setDatabits()
            self.setStopbits()
            self.setParitybits()
			# 打开s_serialPort指定的串口设备
            s_serialPort.open()
            self.m_button_openClose.SetLabel('isOpen->Close')
            s_serialPort.reset_input_buffer()
            s_serialPort.reset_output_buffer()
			# 开启串口接收线程（每0.5秒定时执行一次）
			threading.Timer(s_recvInterval, self.recvData).start()
```

　　上述代码里需要特别讲一下的是串口接收线程，我们知道串口设备s_serialPort一旦打开之后，只要该串口设备的RXD信号线上有数据传输，pySerial底层会自动将其存入s_serialPort对应的input_buffer里，但并不会主动通知我们。那我们怎么知道input_buffer里有没有数据？此时就需要我们开启一个定时执行的线程，线程里会去查看input_buffer里是否有数据，如果有数据便显示出来，因此在串口设备打开的同时我们需要创建一个串口接收线程recvData()。  

#### 2.2串口接收功能
　　串口接收功能其实在串口配置里已经提到了，主要就是串口接收线程recvData()的实现，recvData()实现很简单，只有一个注意点，那就是threading.Timer()的用法，这是个软件定时器，它只能超时触发一次任务的执行，如果想让任务循环触发，那么需要在任务本身里添加threading.Timer()的调用。  

```Python
    def clearRecvDisplay( self, event ):
        self.m_textCtrl_recv.Clear()

    def setRecvFormat( self, event ):
        event.Skip()

    def recvData( self ):
        if s_serialPort.isOpen():
		    # 获取input_buffer里的数据个数
            num = s_serialPort.inWaiting()
            if num != 0:
			    # 获取input_buffer里的数据并显示在GUI界面的接收显示框里
                data = s_serialPort.read(num)
                self.m_textCtrl_recv.write(data)
			# 这一句是线程能够定时执行的关键
            threading.Timer(s_recvInterval, self.recvData).start()
```

#### 2.3串口发送功能
　　串口发送功能相比串口接收功能就简单多了，串口发送主要就是实现GUI界面上"Send"按钮的回调函数，即sendData()，代码实现比较简单，不予赘述。  

```Python
    def clearSendDisplay( self, event ):
        self.m_textCtrl_send.Clear()

    def setSendFormat( self, event ):
        event.Skip()

    def sendData( self, event ):
        if s_serialPort.isOpen():
		    # 获取发送输入框里的数据并通过串口发送出去
            lines = self.m_textCtrl_send.GetNumberOfLines()
            for i in range(0, lines):
                data = self.m_textCtrl_send.GetLineText(i)
                s_serialPort.write(str(data))
        else:
            self.m_textCtrl_send.Clear()
            self.m_textCtrl_send.write('Port is not open')
```

　　目前串口收发与显示实现均是基于字符方式，即发送输入框、接收显示框里仅支持ASCII码字符串，关于Char/Hex显示转换的功能（setRecvFormat()/setSendFormat()）并未加上，后续优化里会进一步做。  

　　至此，串口调试工具tinyPyCOM诞生之串口功能实现痞子衡便介绍完毕了，掌声在哪里~~~  



