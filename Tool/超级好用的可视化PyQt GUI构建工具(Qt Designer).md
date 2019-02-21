----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**PyQt GUI构建工具Qt Designer**。  

　　痞子衡开博客至今已有好几年，一直以嵌入式开发相关主题的文章为主线，偶尔穿插一些其他技术或工具的介绍，前段时间因为要做一个跟恩智浦MCU启动相关的上位机工具 [NXP-MCUBootUtility](https://github.com/JayHeng/NXP-MCUBootUtility)，网上搜索对比了几个Python下的GUI框架，最终选择了wxPython这个成熟稳定的GUI库，从而接触到wxFormBuilder这个配套wxPython使用的GUI构建工具。苦于网上关于该构建工具的中文资料不多，所以根据自己使用经验写了一篇 [极易上手的可视化wxPython GUI构建工具(wxFormBuilder)](https://www.cnblogs.com/henjay724/p/9426966.html)，没想到该篇博客很受欢迎，居然目前是痞子衡博客里阅读量最高的一篇博客，而且也是搜索 wxFormBuilder 关键字出来的中文结果排名第二位的链接，真是万万没想到。  

　　wxPython框架虽然成熟稳定，但是相对最近更火的PyQt框架来说，还是显得古老了一些，控件风格不符合现代审美观，因此痞子衡决定学习一下PyQt的用法，感受下PyQt做出来的界面效果到底如何。根据wxPython学习经验，当然首先要从PyQt的可视化GUI构建工具Qt Designer开始下手，因此便有了本篇博客。  

### 一、Qt Designer工具背景
　　Qt Designer从名字上来看显然就是久负盛名的跨平台GUI库Qt的配套设计工具。Qt库本身是C++语言实现的；Riverbank公司用Python语言对Qt做了一层封装，封装后便成了Python版GUI库PyQt（目前最新的版本是PyQt5）；下面是这两个GUI库的官方主页：  

> * Qt项目官方网站: https://www.qt.io/  
> * PyQt项目官方主页: https://www.riverbankcomputing.com/software/pyqt/intro  

　　Qt的各种UI控件功能均是通过class来实现的，这个链接 [https://doc.qt.io/qt-5/classes.html](https://doc.qt.io/qt-5/classes.html) 列出了Qt里的所有class。PyQt5其用法基本与Qt一致，这个链接 [https://www.riverbankcomputing.com/static/Docs/PyQt5/module_index.html#ref-module-index](https://www.riverbankcomputing.com/static/Docs/PyQt5/module_index.html#ref-module-index) 列出了PyQt5里所有的Modules，其中用于设计界面最常用的便是 [QtWidgets](https://www.riverbankcomputing.com/static/Docs/PyQt5/api/qtwidgets/qtwidgets-module.html) 模块。  

　　在Qt官网的Tools下面可以看到所有Qt相关的工具，在UI design tools下面可以找到Qt Designer，可见Qt Designer是用于设计GUI界面的工具之一。由于痞子衡介绍的PyQt5下的GUI构建工具，因此本文的Qt Designer并不是直接在Qt官网下载安装的，具体安装方法详见下一章节。  

<img src="http://henjay724.com/image/cnblogs/QtDesigner-Toolset.png" style="zoom:100%" />

### 二、Qt Designer快速上手
　　使用Qt Designer去设计GUI界面可以不用掌握PyQt5里的各个控件class的具体用法，你只需要在Qt Designer软件里添加这些控件即可，下面痞子衡将简介Qt Designer的用法：  

#### 2.1软件安装
　　简单了解PyQt5的module和class便可以开始设计GUI界面，首先得安装Qt Designer，在安装完Python3之后（痞子衡安装的是Python 3.6），借助\Python36\Scripts\下的pip.exe工具来分别安装PyQt5和Qt Designer，命令见如下主页：  

> * PyQt5安装: https://pypi.org/project/PyQt5/  
> * Qt Designer安装: https://pypi.org/project/pyqt5-tools/  

　　安装完成之后打开\Python36\Lib\site-packages\pyqt5_tools\designer.exe，这便是Qt Designer。  

#### 2.2软件界面
　　打开Qt Designer可见到如下界面，界面主要分为四大区：项目区、控件区、编辑区、属性区。软件使用起来非常简单，就是在【控件区】里点击添加需要的控件，这些控件的效果会在【编辑区】里实时显示，并在【属性区】这些控件的属性，【项目区】用于显示控件间的层级关系。  

<img src="http://henjay724.com/image/cnblogs/QtDesigner-mainWin.PNG" style="zoom:100%" />

#### 2.3基础布局
　　让我们开始创建一个GUI的基础框架，基础框架包括：Container（局部外围轮廓）、Layout（内部控件区）、menubar（顶部菜单栏）、statusbar（底部状态栏）。  
　　第一步是添加一个Container（此处选择常用的Frame），这是GUI的轮廓基础，有了Frame之后还需要在Frame里添加Layout（此处选择竖排样式），用于规范后续控件的排列样式。默认GUI即有menubar和statusbar。  

<img src="http://henjay724.com/image/cnblogs/QtDesigner-step1-containerLayout.PNG" style="zoom:100%" />

#### 2.4多种控件
　　基础布局搞定之后，接下来便是在Layout里添加控件，PyQt5支持的控件非常丰富，其中比较常用的是如下几个：各种Button（按钮）、Label（静态显示文本框）、Text Edit（输入输出文本框）、Check Box（选中框）、各种Slider（滑动条）等。由于前面痞子衡选择的是verticalLayout，因此你会看到控件们都是竖着排的。  

<img src="http://henjay724.com/image/cnblogs/QtDesigner-step2-ctrl.PNG" style="zoom:100%" />

#### 2.5控件属性
　　添加了所有控件之后，下一步便是分别设置控件的属性，进一步调整控件。痞子衡以Push Button属性为例，痞子衡勾选了如下3项比较重要的属性设置，分别是objectName（button在后续python代码的对象名，一般需要按其功能修改，修改后使得代码阅读/修改起来更直观）、geometry（设置button的尺寸与位置，如果是放在Layout里，则受限于Layout不可设置）、text（button在GUI里显示的标签名，此处是PushButton，也需要按其功能修改，方便用户使用软件）。  

<img src="http://henjay724.com/image/cnblogs/QtDesigner-step3-button_property.PNG" style="zoom:100%" />

#### 2.6保存为xml代码（工程文件）
　　当GUI界面布局全部完成之后，需选择File->Save As保存为.ui文件，该文件既是Qt Designer的工程文件也是最终生成的GUI xml代码文件，痞子衡保存在了my_win.ui文件里。  

<img src="http://henjay724.com/image/cnblogs/QtDesigner-step4-xml.PNG" style="zoom:100%" />

#### 2.7转换成python代码
　　虽然保存的my_win.ui文件里是可以直接在python代码里被加载使用的，但是更好的办法是直接将.ui文件转换成相应的.py文件。需要借助 \Python36\Scripts\pyuic5.exe工具，命令如下：  

> <font style="font-weight:bold;" color="Blue">pyuic5 - o my_win.py my_win.ui</font>

　　转换成功后，让我们打开my_win.py文件，可以简单看一下这个my_win.py里的内容，代码里首先import了PyQt5相关库，并定义了名为Ui_MainWindow的class，这个class主要包含两个函数setupUi()和retranslateUi()。setupUi()里初始化了各个控件成员self.xx，这与我们在Qt Designer里添加控件是对应的。  

```Python
# -*- coding: utf-8 -*-

# Form implementation generated from reading ui file '.\my_win.ui'
#
# Created by: PyQt5 UI code generator 5.11.3
#
# WARNING! All changes made in this file will be lost!

from PyQt5 import QtCore, QtGui, QtWidgets

class Ui_MainWindow(object):
    def setupUi(self, MainWindow):
        MainWindow.setObjectName("MainWindow")
        MainWindow.resize(603, 448)
        self.centralwidget = QtWidgets.QWidget(MainWindow)
        self.centralwidget.setObjectName("centralwidget")
        self.frame = QtWidgets.QFrame(self.centralwidget)
        self.frame.setGeometry(QtCore.QRect(100, 80, 361, 211))
        self.frame.setFrameShape(QtWidgets.QFrame.StyledPanel)
        self.frame.setFrameShadow(QtWidgets.QFrame.Raised)
        self.frame.setObjectName("frame")
        self.verticalLayoutWidget = QtWidgets.QWidget(self.frame)
        self.verticalLayoutWidget.setGeometry(QtCore.QRect(30, 20, 160, 172))
        self.verticalLayoutWidget.setObjectName("verticalLayoutWidget")
        self.verticalLayout = QtWidgets.QVBoxLayout(self.verticalLayoutWidget)
        self.verticalLayout.setContentsMargins(0, 0, 0, 0)
        self.verticalLayout.setObjectName("verticalLayout")
        self.pushButton = QtWidgets.QPushButton(self.verticalLayoutWidget)
        self.pushButton.setEnabled(True)
        self.pushButton.setObjectName("pushButton")
        self.verticalLayout.addWidget(self.pushButton)
        self.label = QtWidgets.QLabel(self.verticalLayoutWidget)
        self.label.setObjectName("label")
        self.verticalLayout.addWidget(self.label)
        self.textEdit = QtWidgets.QTextEdit(self.verticalLayoutWidget)
        self.textEdit.setObjectName("textEdit")
        self.verticalLayout.addWidget(self.textEdit)
        self.checkBox = QtWidgets.QCheckBox(self.verticalLayoutWidget)
        self.checkBox.setObjectName("checkBox")
        self.verticalLayout.addWidget(self.checkBox)
        self.horizontalSlider = QtWidgets.QSlider(self.verticalLayoutWidget)
        self.horizontalSlider.setOrientation(QtCore.Qt.Horizontal)
        self.horizontalSlider.setObjectName("horizontalSlider")
        self.verticalLayout.addWidget(self.horizontalSlider)
        MainWindow.setCentralWidget(self.centralwidget)
        self.menubar = QtWidgets.QMenuBar(MainWindow)
        self.menubar.setGeometry(QtCore.QRect(0, 0, 603, 21))
        self.menubar.setObjectName("menubar")
        MainWindow.setMenuBar(self.menubar)
        self.statusbar = QtWidgets.QStatusBar(MainWindow)
        self.statusbar.setObjectName("statusbar")
        MainWindow.setStatusBar(self.statusbar)

        self.retranslateUi(MainWindow)
        QtCore.QMetaObject.connectSlotsByName(MainWindow)

    def retranslateUi(self, MainWindow):
        _translate = QtCore.QCoreApplication.translate
        MainWindow.setWindowTitle(_translate("MainWindow", "MainWindow"))
        self.pushButton.setText(_translate("MainWindow", "PushButton"))
        self.label.setText(_translate("MainWindow", "TextLabel"))
        self.checkBox.setText(_translate("MainWindow", "CheckBox"))
```

### 三、使用Qt Designer生成的代码
　　前面已经使用Qt Designer生成GUI界面类Ui_MainWindow并保存在my_win.py文件中，此时需要创建一个主函数文件去调用Ui_MainWindow，下面是痞子衡创建的main_win.py中的代码：

```Python
import sys
from PyQt5.QtWidgets import QApplication, QMainWindow
# 导入my_win.py中内容
from my_win import *

# 创建mainWin类并传入Ui_MainWindow
class mainWin(QMainWindow, Ui_MainWindow):
    def __init__(self, parent=None):
        super(mainWin, self).__init__(parent)
        self.setupUi(self)

if __name__ == '__main__':
    # 下面是使用PyQt5的固定用法
    app = QApplication(sys.argv)
    main_win = mainWin()
    main_win.show()
    sys.exit(app.exec_())
```

#### 3.1触发事件与响应
　　有了Button，我们肯定希望其能与一个响应函数相联系起来，此处痞子衡定义了showMessage()函数，并且将showMessage()与PushButton绑定起来，点击Button便会执行一次这个showMessage()函数。代码如下：  

```Python
class mainWin(QMainWindow, Ui_MainWindow):
    def __init__(self, parent=None):
        super(mainWin, self).__init__(parent)
        self.setupUi(self)
		# 将响应函数绑定到指定Button
		self.pushButton.clicked.connect(self.showMessage)

    # Button响应函数
    def showMessage(self):
        self.textEdit.setText('hello world')
```

　　最后让我们测试一下这个GUI软件，在命令行下运行main_win.py  

```text
PS D:\my_git_repo\> python .\main_win.py
```

<img src="http://henjay724.com/image/cnblogs/QtDesigner-gui_run.PNG" style="zoom:100%" />

　　至此，PyQt5 GUI构建工具Qt Designer痞子衡便介绍完毕了，掌声在哪里~~~  

### 参考资料
> 1. [使用PyQt来编写第一个Python GUI程序](https://www.cnblogs.com/rrxc/p/4462890.html)
> 2. [PyQT5速成教程-2 Qt Designer介绍与入门](https://www.jianshu.com/p/5b063c5745d0)
