----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**串口调试工具JaysPyCOM诞生之软件优化**。  

　　前面痞子衡已经初步实现了JaysPyCOM的串口功能，并且通过了最基本的测试，但目前的JaysPyCOM相比市面上流行的串口调试工具还差得很远，有很大的优化空间。优化可以从两方面进行：一、是功能上的优化，可以添加更多实用的功能；二、是界面效果上的优化，可以增加一些界面动画效果或者重新配色美化界面。下面痞子衡从这两方面分别为JaysPyCOM做一些简单的优化：  

### 一、功能优化
#### 1.1 增强鲁棒性
　　最开始要做的功能优化应该是增强软件鲁棒性，即在任何异常用户输入的情况下，软件都不能挂掉，痞子衡在实测中发现当用户在"Com Port"里输入的是无效串口设备号时，软件会挂掉，因此做了以下改进，在打开设备时使用try except语句，如有异常，直接退出，不会继续后面代码的执行。此类改进还有很多，不一一例举。  

```Python
class mainWin(jayspycom_win.com_win):

    def openClosePort( self, event ):
        if s_serialPort.isOpen():
            s_serialPort.close()
            self.m_button_openClose.SetLabel('Open')
        else:
            # ...
            self.setParitybits()
			# 添加代码开始
            try:
                s_serialPort.open()
            except Exception, e:
                # Show warning message
                return
			# 添加代码结束
            self.m_button_openClose.SetLabel('Close')
            # ...
```

#### 1.2 自动检测可用Port
　　最初版本实现Port口选择是用户按标准格式“COMx”手动输入，但这样有一个问题，即用户输入的格式有可能不合法，并且即使是一个合法的格式输入，但也可能不是一个可用的有效Port。参照市面上流行的串口调试助手，有的是下拉菜单选择所有COM口（比如AccessPort，这样可以解决不合法格式输入的问题），有的是下拉菜单选择可用的COM口（比如sscom，这样可以解决Port是否有效的问题），痞子衡参照sscom的做法对JaysPyCOM进行了如下优化：  

```Python
class mainWin(jayspycom_win.com_win):

    def __init__(self, parent):
        self.refreshComPort(None)
        self.m_choice_comPort.SetSelection( 0 )

    def refreshComPort( self, event ):
        comports = list(serial.tools.list_ports.comports())
        ports = [None] * len(comports)
        for i in range(len(comports)):
            comport = list(comports[i])
            # example comport = [u'COM3', u'Intel(R) Active Management Technology - SOL (COM3)', u'PCI\\VEN_8086&DEV_9D3D&SUBSYS_06DC1028&REV_21\\3&11583659&0&B3']
            ports[i] = comport[0] + ' - ' + comport[1]
        self.m_choice_comPort.Clear()
        self.m_choice_comPort.SetItems(ports)

    def setPort ( self ):
        index = self.m_choice_comPort.GetSelection()
        comPort = self.m_choice_comPort.GetString(index)
        comPort = comPort.split(' - ')
        s_serialPort.port = comPort[0]
```

#### 1.3 实现格式切换功能
　　Char/Hex格式转换属于比较实用的功能，一般的串口调试助手都会有这个功能，JaysPyCOM之前默认总是按照Char格式来输入和显示，"Format"选项框的功能实际上并没有实现，因此痞子衡在这里加上了格式切换功能。  

```Python
import jayspycom_formatter

s_formatter = jayspycom_formatter.formatter()
s_lastRecvFormat = None
s_lastSendFormat = None

class mainWin(jayspycom_win.com_win):

    # 函数功能实现
    def setSendFormat( self, event ):
        lines = self.m_textCtrl_send.GetNumberOfLines()
        if lines != 0:
            m_sendFormat = self.m_choice_sendFormat.GetString(self.m_choice_sendFormat.GetSelection())
            global s_lastSendFormat
            if s_lastSendFormat == m_sendFormat:
                return
            else:
                s_lastSendFormat = m_sendFormat
            # Get existing data from textCtrl_send
            data = ''
            for i in range(0, lines):
                data += str(self.m_textCtrl_send.GetLineText(i))
            # Convert data format according to choice_sendFormat
            if m_sendFormat == 'Char':
                status, data = s_formatter.hexToChar(data)
                if not status:
                    self.m_textCtrl_send.Clear()
                    self.m_textCtrl_send.write('Invalid format! Correct example: 12 34 56 ab cd ef')
                    return
            elif m_sendFormat == 'Hex':
                data = s_formatter.charToHex(data)
            # Re-show converted data in textCtrl_send
            self.m_textCtrl_send.Clear()
            self.m_textCtrl_send.write(data)

    def sendData( self, event ):
        if s_serialPort.isOpen():
            lines = self.m_textCtrl_send.GetNumberOfLines()
            if lines != 0:
                data = ''
                for i in range(0, lines):
                    data += str(self.m_textCtrl_send.GetLineText(i))
				# 添加代码开始
                # Make sure data is always in 'Char' format
                m_sendFormat = self.m_choice_sendFormat.GetString(self.m_choice_sendFormat.GetSelection())
                if m_sendFormat == 'Hex':
                    status, data = s_formatter.hexToChar(data)
                    if not status:
                        self.m_textCtrl_send.Clear()
                        self.m_textCtrl_send.write('Invalid format! Correct example: 12 34 56 ab cd ef')
                        return
				# 添加代码结束
                s_serialPort.write(data)

    # 函数功能实现
    def setRecvFormat( self, event ):
        lines = self.m_textCtrl_recv.GetNumberOfLines()
        if lines != 0:
            m_recvFormat = self.m_choice_recvFormat.GetString(self.m_choice_recvFormat.GetSelection())
            global s_lastRecvFormat
            if s_lastRecvFormat == m_recvFormat:
                return
            else:
                s_lastRecvFormat = m_recvFormat
            # Get existing data from textCtrl_recv
            data = ''
            for i in range(0, lines):
                data += str(self.m_textCtrl_recv.GetLineText(i))
            # Convert data format according to choice_recvFormat
            if m_recvFormat == 'Char':
                status, data = s_formatter.hexToChar(data)
            elif m_recvFormat == 'Hex':
                data = s_formatter.charToHex(data)
            # Re-show converted data in textCtrl_recv
            self.m_textCtrl_recv.Clear()
            self.m_textCtrl_recv.write(data)

    def recvData( self ):
        if s_serialPort.isOpen():
            num = s_serialPort.inWaiting()
            if num != 0:
                data = s_serialPort.read(num)
				# 添加代码开始
                # Note: Assume that data is always in 'Char' format
                # Convert data format if dispaly format is 'Hex'
                m_recvFormat = self.m_choice_recvFormat.GetString(self.m_choice_recvFormat.GetSelection())
                if m_recvFormat == 'Hex':
                    data = s_formatter.charToHex(data)
			    # 添加代码结束
                self.m_textCtrl_recv.write(data)

```

　　发送输入框格式切换功能实测如下，尤其是在Hex模式下，如果有异常输入，JaysPyCOM会直接清屏，并在输入框里提示正确的示例。接收显示框格式切换功能雷同，但并不包含异常输入提示，因为这是个结果显示输出框。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/JaysPyCOM_optimization_format_send.PNG" style="zoom:100%" />

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/JaysPyCOM_optimization_format_send_error.PNG" style="zoom:100%" />

#### 1.4 启用菜单栏
　　菜单栏是一个功能齐全的软件的标配，用于实现各种特性功能，此处痞子衡仅添加了一个“Help”菜单，用于显示JaysPyCOM的主页以及作者信息。首先需要在wxFormBuilder添加menu控件，然后设置回调函数名，下面是回调函数的实现：  

```Python
    def showHomepageMessage( self, event ):
        messageText = (('Code: \n    https://github.com/JayHeng/JaysPyCOM.git \n') +
                       ('Doc: \n    https://www.cnblogs.com/henjay724/p/9416096.html \n'))
        wx.MessageBox(messageText, "Homepage", wx.OK | wx.ICON_INFORMATION)

    def showAboutMessage( self, event ):
        messageText = (('Author: Jay Heng \n') +
                       ('Email: hengjie1989@foxmail.com \n'))
        wx.MessageBox(messageText, "About", wx.OK | wx.ICON_INFORMATION)
```

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/JaysPyCOM_optimization_menu.PNG" style="zoom:100%" />

#### 1.5 启用状态栏
　　状态栏也是一般串口调试助手的标配，用于显示发送/接收数据统计信息以及串口开关状态，因此痞子衡为JaysPyCOM也加上了状态栏功能，状态栏主要分为三栏：接收数据统计、发送数据统计、串口状态。  

```Python
s_recvStatusFieldIndex = 0
s_sendStatusFieldIndex = 1
s_infoStatusFieldIndex = 2

s_recvStatusStr = 'Recv: '
s_recvTotalBytes = 0
s_sendStatusStr = 'Send: '
s_sendTotalBytes = 0

class mainWin(jayspycom_win.com_win):

    def openClosePort( self, event ):
        if s_serialPort.isOpen():
            s_serialPort.close()
            self.m_button_openClose.SetLabel('Open')
			# 添加代码开始
            self.statusBar.SetStatusText(s_serialPort.name + ' is closed', s_infoStatusFieldIndex)
			# 添加代码结束
        else:
		    # 添加代码开始
            self.statusBar.SetFieldsCount(3)
            self.statusBar.SetStatusWidths([150, 150, 400])
		    # 添加代码结束
            self.setPort()
			# ...
            self.m_button_openClose.SetLabel('Close')
            # 添加代码开始
            self.statusBar.SetStatusText(s_recvStatusStr + str(s_recvTotalBytes), s_recvStatusFieldIndex)
            self.statusBar.SetStatusText(s_sendStatusStr + str(s_sendTotalBytes), s_sendStatusFieldIndex)
            self.statusBar.SetStatusText(s_serialPort.name + ' is open, ' +
                                               str(s_serialPort.baudrate) + ', ' +
                                               str(s_serialPort.bytesizes) + ', ' +
                                               s_serialPort.parity + ', ' +
                                               str(s_serialPort.stopbits), s_infoStatusFieldIndex)
		    # 添加代码结束
            s_serialPort.reset_input_buffer()
            # ...

    def sendData( self, event ):
        if s_serialPort.isOpen():
            lines = self.m_textCtrl_send.GetNumberOfLines()
            if lines != 0:
                # ...
                s_serialPort.write(data)
                # 添加代码开始
                global s_sendTotalBytes
                s_sendTotalBytes += len(data)
                self.statusBar.SetStatusText(s_sendStatusStr + str(s_sendTotalBytes), s_sendStatusFieldIndex)
				# 添加代码结束
        else:
            self.statusBar.SetStatusText(s_serialPort.name + ' is not open !!!', s_infoStatusFieldIndex)

    def recvData( self ):
        if s_serialPort.isOpen():
            num = s_serialPort.inWaiting()
            if num != 0:
                # ...
                self.m_textCtrl_recv.write(data)
                # 添加代码开始
                global s_recvTotalBytes
                s_recvTotalBytes += len(data)
                self.statusBar.SetStatusText(s_recvStatusStr + str(s_recvTotalBytes), s_recvStatusFieldIndex)
				# 添加代码结束
```

　　状态栏实测功能如下：  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/JaysPyCOM_optimization_statusbar_info.PNG" style="zoom:100%" />

### 二、界面优化
#### 2.1 添加串口开关亮灯效果
　　界面优化的地方有很多，痞子衡简单做了一个与串口开关按钮同步的小灯显示效果，当串口打开时，小灯显示绿色；当串口关闭时，小灯显示黑色；代码里的实现其实就是两张图片之间的切换。  

```Python
class mainWin(jayspycom_win.com_win):

    def openClosePort( self, event ):
        if s_serialPort.isOpen():
            s_serialPort.close()
            self.m_button_openClose.SetLabel('Open')
			# 添加代码开始
            self.m_bitmap_led.SetBitmap(wx.Bitmap( u"../img/led_black.png", wx.BITMAP_TYPE_ANY ))
			# 添加代码结束
        else:
            # ...
            self.m_button_openClose.SetLabel('Close')
			# 添加代码开始
            self.m_bitmap_led.SetBitmap(wx.Bitmap( u"../img/led_green.png", wx.BITMAP_TYPE_ANY ))
			# 添加代码结束
            # ...
```

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/JaysPyCOM_optimization_led_switch.PNG" style="zoom:100%" />

　　至此，串口调试工具JaysPyCOM诞生之软件优化痞子衡便介绍完毕了，掌声在哪里~~~  



