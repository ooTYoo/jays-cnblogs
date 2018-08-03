----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**串口调试工具tinyPyCOM诞生之环境搭建**。  

　　在写tinyPyCOM时需要先搭好开发和调试环境，下表列出了开发过程中会用到的所有软件/工具包：  

### 一、涉及工具列表

<table><tbody>
    <tr>
        <th style="width: 200px;">工具</th>
        <th style="width: 400px;">功能</th>
        <th style="width: 300px;">下载地址</th>
    </tr>
    <tr>
        <td>Python 2.7.14</td>
        <td>Python官方包（解释器）</td>
        <td><a href="https://www.python.org/">https://www.python.org/</a></td>
    </tr>
    <tr>
        <td>pySerial 3.4</td>
        <td>Serial Port访问的Python封装库</td>
        <td><a href="https://pypi.org/project/pyserial/">https://pypi.org/project/pyserial/</a><br>
		    <a href="https://github.com/pyserial/pyserial">https://github.com/pyserial/pyserial</a><br>
		    <a href="https://pythonhosted.org/pyserial/">https://pythonhosted.org/pyserial/</a><br>
		</td>
    </tr>
    <tr>
        <td>wxPython 4.0.3</td>
        <td>跨平台开源GUI库wxWidgets的Python封装库</td>
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
    <tr>
        <td>vspd 9</td>
        <td>虚拟串口驱动，可以在PC上虚拟出Serial Port</td>
        <td><a href="https://www.eltima.com/products/vspdxp/">https://www.eltima.com/products/vspdxp/</a></td>
    </tr>
    <tr>
        <td>sscom 5.13.1</td>
        <td>大虾和丁丁联合推出的一款很流行的串口调试工具</td>
        <td><a href="http://www.daxia.com/sscom/">http://www.daxia.com/sscom/</a></td>
    </tr>
</table>

### 二、开发环境搭建（Python + pySerial + wxPython + wxFormBuilder）
　　tinyPyCOM工具是一个完全基于Python语言开发的应用软件，首先安装好Python 2.7.14，痞子衡的安装目录为C:\tools_mcu\Python27，安装完成后确保系统环境变量里包括该路径（C:\tools_mcu\Python27），因为该路径下包含python.exe，后续python命令需调用这个python.exe完成的。  
　　在C:\tools_mcu\Python27\Scripts目录下默认有easy_install.exe，这是PEAK(Python Enterprise Application Kit)开发的setuptools包里的工具，这个工具可以用来完成安装python第三方模块的工作。我们需要借助easy_install.exe来安装pip工具：  

> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue"> .\easy_install.exe pip</font>
> ```text
> Searching for pip
> Best match: pip 9.0.1
> Adding pip 9.0.1 to easy-install.pth file
> Installing pip-script.py script to c:\tools_mcu\python27\Scripts
> Installing pip.exe script to c:\tools_mcu\python27\Scripts
> Installing pip3.5-script.py script to c:\tools_mcu\python27\Scripts
> Installing pip3.5.exe script to c:\tools_mcu\python27\Scripts
> Installing pip3-script.py script to c:\tools_mcu\python27\Scripts
> Installing pip3.exe script to c:\tools_mcu\python27\Scripts
>
> Using c:\tools_mcu\python27\lib\site-packages
> Processing dependencies for pip
> Finished processing dependencies for pip
> ```
>
> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue"> python -m pip install --upgrade pip</font>
> ```text
> Collecting pip
>   Downloading > https://files.pythonhosted.org/packages/5f/25/e52d3f31441505a5f3af41213346e5b6c221c9e086a166f3703d2ddaf940/pip-18.0-py2.py3-none-any.whl (1.3MB)
> Installing collected packages: pip
>   Found existing installation: pip 9.0.1
>     Uninstalling pip-9.0.1:
>       Successfully uninstalled pip-9.0.1
> Successfully installed pip-18.0
> ```

　　pip是Python的包管理工具，提供了对Python包的查找、下载、安装、卸载的功能。安装好pip工具之后，可以看到C:\tools_mcu\Python27\Scripts目录下多了pip.exe，为方便后续使用pip来安装其他Python包，确保系统环境变量里包括pip路径（C:\tools_mcu\Python27\Scripts）。我们可以借助pip来安装pySerial和wxPython包：  


> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue"> .\pip.exe install pyserial</font>
> ```text
> Collecting pyserial
>   Downloading https://files.pythonhosted.org/packages/0d/e4/2a744dd9e3be04a0c0907414e2a01a7c88bb3915cbe3c8cc06e209f59c30/pyserial-3.4-py2.py3-none-any.whl (193kB)
> Installing collected packages: pyserial
> Successfully installed pyserial-3.4
> ```
>
> PS C:\tools_mcu\Python27\Scripts><font style="font-weight:bold;" color="Blue"> .\pip.exe install wxPython</font>
> ```text
> Collecting wxPython
>   Downloading https://files.pythonhosted.org/packages/88/88/a23b96662c5ab82dd8dbbb68c68dedea466229e8151fd2911713a1cd27b2/wxPython-4.0.3-cp27-cp27m-win_amd64.whl (22.8MB)
Collecting six (from wxPython)
>   Downloading https://files.pythonhosted.org/packages/67/4b/141a581104b1f6397bfa78ac9d43d8ad29a7ca43ea90a2d863fe3056e86a/six-1.11.0-py2.py3-none-any.whl
> Collecting PyPubSub (from wxPython)
>   Downloading https://files.pythonhosted.org/packages/14/80/8e1d34848fea10826763600ca7eeb7a76d914ccab7cb0d64c9c180c30a73/Pypubsub-4.0.0.zip (64kB)
> Collecting typing (from PyPubSub->wxPython)
>   Downloading https://files.pythonhosted.org/packages/0d/4d/4e5985d075d241d686a1663fa1f88b61d544658d08c1375c7c6aac32afc3/typing-3.6.4-py2-none-any.whl
> Installing collected packages: six, typing, PyPubSub, wxPython
>   Running setup.py install for PyPubSub ... done
>   The scripts helpviewer.exe, img2png.exe, img2py.exe, img2xpm.exe, pycrust.exe, pyshell.exe, pyslices.exe, pyslicesshell.exe, pywxrc.exe, wxdemo.exe, wxdocs.exe and wxget.exe are installed in 'c:\tools_mcu\python27\Scripts'
> Successfully installed PyPubSub-4.0.0 six-1.11.0 typing-3.6.4 wxPython-4.0.3
> ```

　　有了pySerial便可以访问Serial Port，有了wxPython便可以设计GUI。  
　　单纯使用wxPython设计tinyPyCOM GUI界面时仅能是手工写代码布局，手工布局的界面创建和修改起来都比较繁琐，我们需要一款可视化的界面设计工具，痞子衡选择的是wxFormBuilder，从其github官网下载安装包并安装到C:\tools_mcu\wxFormBuilder目录下。安装完成打开软件便可在Designer里尽情创作界面，创作完成后点击"Python"便可看到Python GUI源代码，这个GUI源代码后续直接复制到tinyPyCOM工程里使用。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/tinyPyCOM_preparation_wxFormBuilder_view.PNG" style="zoom:100%" />

　　至此tinyPyCOM工具开发的Python基础环境便搭好了。  

### 三、测试环境搭建（PyCharm + vspd + sscom）
　　在开发tinyPyCOM工具过程中免不了要调试Python代码，所以我们还需要一个Python IDE，痞子衡选择的是PyCharm，在jetbrains官网下载PyCharm community免费版并安装，安装完成后打开PyCharm并创建名为tinyPyCOM空工程，成功创建后会看到tinyPyCOM目录下自动生成一个.idea的文件夹，该文件夹是用于pycharm管理项目。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/tinyPyCOM_preparation_pycharm_project.PNG" style="zoom:100%" />

　　有了PyCharm环境，便可以开始写tinyPyCOM代码，代码在开发过程中，需要结合Serial Port进行联合调试，如果手里没有硬件串口设备，可以使用虚拟串口设备，vspd便是著名的虚拟串口驱动，从eltima官网下载vspd标准版并安装，安装完成后打开vspd可看到如下界面，COM10和COM11（COM号是自定义的）便是虚拟出来的串口设备号，并且已经完成了对接。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/tinyPyCOM_preparation_vspd_view4.PNG" style="zoom:100%" />

　　虚拟Serial Port设备已经有了并且对接了，最后还需要一个成熟的串口调试助手，作为串口通讯的另一方，痞子衡选取的是非常经典的sscom，从大虾官网下载sscom包，sscom是个免安装的工具，可以直接打开使用，设置sscom使用COM11，将来tinyPyCOM使用COM10，到这里环境搭建大功告成。  

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/tinyPyCOM_preparation_sscom_view.PNG" style="zoom:100%" />

　　至此，串口调试工具tinyPyCOM诞生之环境搭建痞子衡便介绍完毕了，掌声在哪里~~~  



