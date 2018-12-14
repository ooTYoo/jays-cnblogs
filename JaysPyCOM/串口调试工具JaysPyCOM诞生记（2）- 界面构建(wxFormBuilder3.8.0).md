----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**串口调试工具JaysPyCOM诞生之界面构建**。  

　　一个软件的UI界面是非常重要的，这是软件与用户交互的接口，软件功能即使再强大，但如果没有清晰的UI界面，那也发挥不出软件的功能，使得用户体会不到软件的优势。今天痞子衡给大家介绍JaysPyCOM的界面构建过程。  

### 一、界面设计简图
　　在真正进入代码设计JaysPyCOM界面前，首先应该在纸上画一个界面草图，确定JaysPyCOM界面应该有哪些元素构成，这些元素分别位于界面上什么位置。下面是痞子衡画的JaysPyCOM的界面简图，界面主要包括三大部分：接收区、配置区、发送区，接收区用于显示从串口接收到的数据；配置区用于配置串口参数；发送区用于编辑要从串口发送出去的数据。  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_GUI_protocol.PNG" style="zoom:100%" />

### 二、界面设计wxPython组件图
　　有了JaysPyCOM的界面设计简图指导，下一步需要将设计简图解析成如下的wxPython组件图，将简图里的元素转换成wxPython里的真实组件。这一步需要配合查阅wxPython相关手册，了解wxPython有哪些组件。  
　　有一个地方需要特别提醒的是，wxWrapSizer里的控件是从左到右自上而下排列的，有的时候为了排版，会故意插入一些无效的wxStaticText来占位，下图中便用了4个占位的wxStaticText（浅色框表示）。  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_GUI_element.PNG" style="zoom:100%" />

### 三、在wxFormBuilder里创作
　　有了JaysPyCOM的界面设计wxPython组件图，下面便可以在wxFormBuilder里照样子创作出JaysPyCOM的真正界面了。关于wxFormBuilder的使用可参考痞子衡另一篇文章 [极易上手的可视化wxPython GUI构建工具(wxFormBuilder)](http://www.cnblogs.com/henjay724/p/9426966.html)。  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_GUI_product.PNG" style="zoom:100%" />

### 四、使用生成的Python代码
　　将wxFormBuilder生成的python代码保存为jayspycom_win.py文件（其中Frame class名为com_win），并存放于\JaysPyCOM\gui目录下，此时需要另外新建一个名为jayspycom_main的主函数文件，并放在\JaysPyCOM\src目录下。其中jayspycom_main文件内容如下：  

```Python
import wx
import sys, os
sys.path.append(os.path.abspath("../gui"))
import jayspycom_win

class mainWin(jayspycom_win.com_win):

    def clearRecvDisplay( self, event ):
        event.Skip()

    def openClosePort( self, event ):
        event.Skip()

    def clearSendDisplay( self, event ):
        event.Skip()

    def sendData( self, event ):
        self.m_textCtrl_recv.Clear()
        self.m_textCtrl_recv.SetValue('hello world')

if __name__ == '__main__':
    app = wx.App()

    main_win = mainWin(None)
    main_win.SetTitle(u"JaysPyCOM v.0.1.0")
    main_win.Show()

    app.MainLoop()
```

　　jayspycom_main.py里并没有实现具体功能，只有一个hello world打印的效果，此处只是演示界面已经创建成功，界面运行效果如下：  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_GUI_demo.PNG" style="zoom:100%" />

　　至此，串口调试工具JaysPyCOM诞生之界面构建痞子衡便介绍完毕了，掌声在哪里~~~  



