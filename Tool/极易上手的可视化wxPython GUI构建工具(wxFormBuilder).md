----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**wxPython GUI构建工具wxFormBuilder**。  

### 一、手工代码布局GUI界面的烦恼
　　如果你曾经设计过上位机软件GUI界面，初始阶段一定是纯手工代码布局GUI界面上的各个控件，相信你肯定遇到过如下烦恼：  

> * 控件类型较难找：UI界面里有很多控件类型，纯手工写代码需要翻看文档一个个去查找这些控件的名字与用法。
> * 尺寸位置难调整：如果界面上已经布了多个控件，想要整体去调整这些控件的尺寸与位置是一件头疼的事。
> * 效果查看不实时：每新添加一个控件都需要运行才能查看整体效果，如果代码添加不完善，可能无法运行看不到效果。

### 二、wxFormBuilder工具背景
　　在讲本文主角wxFormBuilder之前有必要提一下这个软件的来历，首先要追述到大名鼎鼎的跨平台GUI库wxWidgets，这个库主要是用C++语言实现的；鉴于wxWidgets的流行，Robin Dunn用Python语言对wxWidgets做了一层封装，封装后便成了Python版GUI库wxPython；下面是这两个GUI库的官方主页：  

> * wxWidgets项目官方主页: https://www.wxwidgets.org/  
> * wxPython项目官方主页: https://www.wxpython.org/  

　　wxWidgets的各种UI控件功能均是通过class来实现的，这个链接 [http://docs.wxwidgets.org/3.0/page_class_cat.html](http://docs.wxwidgets.org/3.0/page_class_cat.html) 列出了wxWidgets里的所有class，wxPython并没有实现wxWidgets里全部class，但基本实现了大部分常用class，这个链接 [https://docs.wxpython.org/wx.1moduleindex.html](https://docs.wxpython.org/wx.1moduleindex.html) 列出了wxPython里所有的class。  

　　知道了wxPython的class便可以开始设计GUI界面，但手工写代码设计界面太繁琐，因此wxFormBuilder应运而生，这是一款能够可视化设计界面的工具（并不是唯一工具，还有wxGlade、Boa Constructor等），通过该工具设计GUI界面后可自动生成wxPython代码，下面是wxFormBuilder的官方主页：  

> * wxFormBuilder项目Github: https://github.com/wxFormBuilder/wxFormBuilder  

### 三、wxFormBuilder快速上手
　　使用wxFormBuilder去设计GUI界面可以不用掌握wxPython里的各个控件class的具体用法，你只需要在wxFormBuilder软件里添加这些控件即可，下面痞子衡将简介wxFormBuilder的用法：  

#### 3.1软件界面
　　安装好wxFormBuilder软件之后打开这个软件，可见到如下界面，界面主要分为四大区：项目区、控件区、编辑区、属性区。软件使用起来非常简单，就是在【控件区】里点击添加需要的控件，这些控件的效果会在【编辑区】里实时显示，并在【属性区】这些控件的属性，【项目区】用于显示控件间的层级关系。  

<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-mainWin.PNG" style="zoom:100%" />

#### 3.2基础布局
　　让我们开始创建一个GUI的基础框架，基础框架包括：Frame（外围轮廓）、Sizer（内部控件区）、menubar（顶部菜单栏）、statusBar（底部状态栏）。  
　　第一步是添加一个Frame，这是GUI的轮廓基础，其size（default为500； 300）决定了GUI整体界面的大小。  
<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-step1-form.PNG" style="zoom:100%" />

　　第二步是在Frame下添加一个Sizer，后续所有控件均是放在Sizer里的。关于Sizer部分需要特别说明一下，wxPython提供的Sizer类型有如下七种：wxBoxSizer、wxWrapSizer、wxStaticBoxSizer、wxGridSizer、wxFlexGridSizer、wxGridBagSizer、wxStdDialogButtonSizer，<font color="Blue">Sizer的样式决定了后续控件的整体相对位置关系，选定了Sizer即选定了GUI界面样式</font>。关于这七种Sizer的具体样式请见 [https://docs.wxpython.org/sizers_overview.html#sizers-overview](https://docs.wxpython.org/sizers_overview.html#sizers-overview)。<font color="Blue">如果你觉得单个Sizer里的控件布局太单调，你可以嵌套使用Sizer，这是实现GUI界面控件布局多样化的关键</font>。  
<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-step2-layout.PNG" style="zoom:100%" />

　　第三步是在Frame顶部添加一个menubar：  
<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-step3-menu.PNG" style="zoom:100%" />

　　第四步是在Frame底部添加一个statusBar：  
<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-step4-status.PNG" style="zoom:100%" />

#### 3.3多种控件
　　基础布局搞定之后，接下来便是在Sizer里添加控件，wxPython支持的控件非常丰富，其中比较常用的是如下几个：button（按钮）、staticText（静态显示文本框）、textCtrl（输入输出文本框）、Choice（复选框）、checkBox（选中框）、slider（滑动条）。<font color="Blue">前面痞子衡选择的Sizer是wxBoxSizer，即自上而下布局，因此这些控件在Sizer是自上而下排列的，各个控件的位置后续在属性里还可以微调，但改变不了自上而下的格局</font>。  
<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-step5-ctrl.PNG" style="zoom:100%" />

#### 3.4控件属性
　　添加了所有控件之后，下一步便是分别设置控件的属性，进一步调整控件。痞子衡以Button属性为例，痞子衡勾选了如下4项比较重要的属性设置，分别是name（button在后续python代码的对象名，一般需要按其功能修改，修改后使得代码阅读/修改起来更直观）、label（button在GUI里显示的标签名，此处是MyButton，也需要按其功能修改，方便用户使用软件）、size（设置button的尺寸，这个尺寸最大不应超过Sizer尺寸）、flag（调整对齐方式从而调整Button在Sizer里的位置）。<font color="Red">另外有一个属性不得不说，即控件位置pos，在wxFormBuilder里设置这个属性并不生效，痞子衡猜想可能跟Sizer样式有关，因为Sizer决定了控件间相对位置关系，因此控件的pos不能随意设置</font>。  
<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-step6-button_property.PNG" style="zoom:100%" />

#### 3.5触发事件
　　有些控件是需要有响应的，比如Button，在GUI软件实际使用中，用户如果按下了Button，应该需要触发某个任务，任务需要有响应函数，这个响应函数需要在【Events】里设置，Button的响应函数在OnButtonClick里设置，痞子衡在这里指定了响应函数名为showMessage。在wxFormBuilder里我们只需要指定控件响应函数名即可，响应函数的具体功能实现，不属于wxFormBuilder设计范畴。  
<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-step6-button_event.PNG" style="zoom:100%" />

#### 3.6生成代码
　　当GUI界面布局全部完成之后，需选择File->Generate Code或F8生成python代码，需要复制所有的python代码并保存在单独的文件里，痞子衡保存在了my\_win.py文件里。  
<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-step7-generate_code.PNG" style="zoom:100%" />

　　可以简单看一下这个my\_win.py里的内容，代码里首先import了wx库（即wxPython库），并定义了名为MyFrame1的class，这个class主要包含两个函数\_\_init\_\_()和showMessage()：\_\_init\_\_()里初始化了各个控件成员self.m_xx，这与我们在wxFormBuilder里添加控件是对应的；showMessage()是Button控件的响应函数，但这个响应函数并没有任何实质代码，当然我们可以在这个函数里面实现Button响应功能，但一般不建议直接在wxFormBuilder生成的代码里添加代码，因为你可能随时调整GUI页面布局，那么main\_win.py里的代码会重新生成，这样会覆盖我们自己添加的代码，导致维护起来比较麻烦。  

```Python
# -*- coding: utf-8 -*- 

###########################################################################
## Python code generated with wxFormBuilder (version Jul 11 2018)
## http://www.wxformbuilder.org/
##
## PLEASE DO *NOT* EDIT THIS FILE!
###########################################################################

import wx
import wx.xrc

###########################################################################
## Class MyFrame1
###########################################################################

class MyFrame1 ( wx.Frame ):
	
	def __init__( self, parent ):
		wx.Frame.__init__ ( self, parent, id = wx.ID_ANY, title = wx.EmptyString, pos = wx.DefaultPosition, size = wx.Size( 500,300 ), style = wx.DEFAULT_FRAME_STYLE|wx.TAB_TRAVERSAL )
		
		self.SetSizeHints( wx.DefaultSize, wx.DefaultSize )
		
		bSizer1 = wx.BoxSizer( wx.VERTICAL )
		
		self.m_button1 = wx.Button( self, wx.ID_ANY, u"MyButton", wx.Point( -1,-1 ), wx.DefaultSize, 0 )
		bSizer1.Add( self.m_button1, 0, wx.ALL, 5 )
		
		self.m_staticText1 = wx.StaticText( self, wx.ID_ANY, u"MyLabel", wx.DefaultPosition, wx.DefaultSize, 0 )
		self.m_staticText1.Wrap( -1 )
		
		bSizer1.Add( self.m_staticText1, 0, wx.ALL, 5 )
		
		self.m_textCtrl1 = wx.TextCtrl( self, wx.ID_ANY, wx.EmptyString, wx.Point( -1,-1 ), wx.DefaultSize, 0 )
		bSizer1.Add( self.m_textCtrl1, 0, wx.ALL, 5 )
		
		m_choice1Choices = []
		self.m_choice1 = wx.Choice( self, wx.ID_ANY, wx.DefaultPosition, wx.DefaultSize, m_choice1Choices, 0 )
		self.m_choice1.SetSelection( 0 )
		bSizer1.Add( self.m_choice1, 0, wx.ALL, 5 )
		
		self.m_checkBox1 = wx.CheckBox( self, wx.ID_ANY, u"Check Me!", wx.DefaultPosition, wx.DefaultSize, 0 )
		bSizer1.Add( self.m_checkBox1, 0, wx.ALL, 5 )
		
		self.m_slider1 = wx.Slider( self, wx.ID_ANY, 50, 0, 100, wx.DefaultPosition, wx.DefaultSize, wx.SL_HORIZONTAL )
		bSizer1.Add( self.m_slider1, 0, wx.ALL, 5 )
		
		
		self.SetSizer( bSizer1 )
		self.Layout()
		self.m_menubar1 = wx.MenuBar( 0 )
		self.m_menu1 = wx.Menu()
		self.m_menuItem1 = wx.MenuItem( self.m_menu1, wx.ID_ANY, u"MyMenuItem", wx.EmptyString, wx.ITEM_NORMAL )
		self.m_menu1.Append( self.m_menuItem1 )
		
		self.m_menubar1.Append( self.m_menu1, u"MyMenu" ) 
		
		self.SetMenuBar( self.m_menubar1 )
		
		self.m_statusBar1 = self.CreateStatusBar( 1, wx.STB_SIZEGRIP, wx.ID_ANY )
		
		self.Centre( wx.BOTH )
		
		# Connect Events
		self.m_button1.Bind( wx.EVT_BUTTON, self.showMessage )
	
	def __del__( self ):
		pass
	
	
	# Virtual event handlers, overide them in your derived class
	def showMessage( self, event ):
		event.Skip()
	
```

### 四、使用wxFormBuilder生成的代码
　　前面已经使用wxFormBuilder生成GUI界面类MyFrame1并保存在my\_win.py文件中，此时需要创建一个主函数文件去调用MyFrame1，下面是痞子衡创建的main_win.py中的代码：  

```Python
import wx
# 导入my_win.py中内容
import my_win

# 创建mainWin类并传入my_win.MyFrame1
class mainWin(my_win.MyFrame1):

   # 实现Button控件的响应函数showMessage
   def showMessage(self, event):
       self.m_textCtrl1.Clear()
       self.m_textCtrl1.SetValue('hello world')

if __name__ == '__main__':
    # 下面是使用wxPython的固定用法
    app = wx.App()

    main_win = mainWin(None)
    main_win.Show()

    app.MainLoop()
```

　　最后让我们测试一下这个GUI软件，在命令行下运行main_win.py  

```text
PS D:\my_git_repo\> python .\main_win.py
```

<img src="http://henjay724.com/image/cnblogs/wxFormBuilder-gui_run.PNG" style="zoom:100%" />

　　至此，wxPython GUI构建工具wxFormBuilder痞子衡便介绍完毕了，掌声在哪里~~~  

### 参考资料
> 1. [wxPython的界面设计wxformbuilde初学笔记](https://blog.csdn.net/baoyan2015/article/details/54613930)
> 2. [wxPython 基础使用教程](http://blog.topspeedsnail.com/archives/1190)
