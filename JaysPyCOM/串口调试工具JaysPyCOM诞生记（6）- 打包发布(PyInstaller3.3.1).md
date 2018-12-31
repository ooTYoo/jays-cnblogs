----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**串口调试工具Jays-PyCOM诞生之打包发布**。  

　　经过上一篇软件优化之后，Jays-PyCOM已经初长成，该到了出去历练的时候了，只有经历广大群众考验过的软件才是合格的软件。痞子衡在开发Jays-PyCOM时本地安装了很多软件：Python、pySerial、wxPython等，这些软件是Jays-PyCOM运行的基础，但如果将来别人想用Jays-PyCOM，是不是也需要安装这些软件呢？答案当然不是，如果是的话，Jays-PyCOM基本就没人愿意使用了。为了让别人能够不需要特殊环境便能运行Jays-PyCOM，我们需要将Jays-PyCOM打包成独立可执行文件，此时我们需要借助专门的Python打包工具，本篇是这个系列的最后一篇，痞子衡为大家讲如何使用打包工具打包Jays-PyCOM去发布。  

### 一、PyInstaller简介
　　Python打包工具有很多，如py2exe、cx_Freeze、PyInstaller，其中痞子衡首推PyInstaller。PyInstaller诞生于2005年，经过这么多年的发展，其版本已经更新到v3.x，其主页如下：  

>  PyInstaller官方主页： [http://www.pyinstaller.org/](http://www.pyinstaller.org/)
>  PyInstaller的github主页： [https://github.com/pyinstaller/pyinstaller](https://github.com/pyinstaller/pyinstaller)

　　在使用PyInstaller进行打包工作之前，首先需要确定你的Python应用程序所调用的所有第三方库是不是在PyInstaller支持列表里，这个主页显示了PyInstaller支持的所有第三方库： [https://github.com/pyinstaller/pyinstaller/wiki/Supported-Packages](https://github.com/pyinstaller/pyinstaller/wiki/Supported-Packages)，基本上常用的第三方库都在列表里，比如Django、numpy、PyGame、PyOpenGL、PyQt、PyWin32等。  
　　PyInstaller的使用非常简单，可先阅读一遍官方文档。对于Jays-PyCOM的打包，我们只需要掌握-F、-w、-i三个命令选项以及.spec文件使用就可以了。  

>  PyInstaller官方文档主页： [https://readthedocs.org/projects/pyinstaller/](https://readthedocs.org/projects/pyinstaller/)
>  PyInstaller 3.3.1命令上手： [https://pyinstaller.readthedocs.io/en/v3.3.1/usage.html#options](https://pyinstaller.readthedocs.io/en/v3.3.1/usage.html#options)
>  PyInstaller 3.3.1 spec文件： [https://pyinstaller.readthedocs.io/en/v3.3.1/spec-files.html#using-spec-files](https://pyinstaller.readthedocs.io/en/v3.3.1/spec-files.html#using-spec-files)

### 二、将JaysPyCOM打包
　　安装好PyInstaller工具便可以开始打包Jays-PyCOM软件了，让我们开始吧，开始之前先介绍下Jays-PyCOM文件夹目录结构，结构目录是很简单的，只有三个.py源文件和三张图片，这构成了Jays-PyCOM软件的全部源文件。  

```text
\Jays-PyCOM
           \.idea                          --放置PyCharm工程文件
           \bin                            --放置工程发布的exe文件
           \gui                            --放置工程GUI设计文件
                  \Jays-PyCOM.fbp            --wxFormBuilder工程文件
           \img                            --放置工程引用图片文件
                  \led_black.png
                  \led_green.png
                  \logo_merge.jpg
           \src                            --放置工程源代码文件
                  \formatter.py    --工程linker文件
                  \main.py         --板级相关的源文件（比如pinout，clock等）
                  \win.py          --wxPython窗口源文件（wxFormBuilder生成）
```

#### 2.1 打包准备
　　由于Jays-PyCOM应用程序总共只有6个源文件，并且都已经准备就绪，Jays-PyCOM依赖的pySerial、wxPython库也都在PyInstaller支持的列表里，所以唯一剩下的准备工作便是制作Jays-PyCOM的图标文件。  
　　要制作图标文件，首先你得有一张图片文件，痞子衡将pySerial的logo截取了部分用作Jays-PyCOM的图标，有了图片，可以使用这个网站 [https://converticon.com/](https://converticon.com/) 将其转换成图标文件(.ico)，图标文件制作好之后将其放在 \Jays-PyCOM\img\目录下：  

```text
\Jays-PyCOM
           \img                            --放置工程引用图片文件
                  \Jays-PyCOM.png
                  \Jays-PyCOM.ico
```

#### 2.2 开始打包
　　准备工作就绪，可以开始打包了，<font color="Blue">在使用PyInstaller打包前必须明白一点的是，PyInstaller仅能将.py格式的源文件以及其所调用的相关Python第三方源文件库打包进最终的.exe文件，如果你的应用程序会用到图片等多媒体文件，这些多媒体文件并不能被打包，后续exe在使用时，这些多媒体文件必须一同在场，并且还要保证与打包/开发时的相对路径是一致的</font>。  
　　痞子衡使用的是如下命令格式打包Jays-PyCOM: pystaller -F -w [src1.py] [src2.py]... -i [pic.ico]，解释一下这个命令组合，-F的意思是将应用程序打包成单个可执行文件（与其对立的命令是-D，打包成多文件放在一个文件夹），-w表明要打包成窗口型（与其对立的命令是-c，控制台型），[src1.py][src2.py][...]为你自己创建的应用程序源文件(src1.py必须是含\_\_main\_\_的主函数文件)，-i指定图标文件。  

> PS D:\my_git_repo\JaysPyCOM\bin><font style="font-weight:bold;" color="Blue">  pyinstaller -F -w ..\src\main.py ..\src\formatter.py ..\src\win.py -i ..\img\Jays-PyCOM.ico </font>
> ```text
> 223 INFO: PyInstaller: 3.3.1
> 225 INFO: Python: 2.7.14
> 227 INFO: Platform: Windows-10-10.0.15063
> 230 INFO: wrote D:\my_git_repo\Jays-PyCOM\bin\main.spec
> 233 INFO: UPX is not available.
> 237 INFO: Extending PYTHONPATH with paths
> ['D:\\my_git_repo\\JaysPyCOM\\bin',
 'D:\\my_git_repo\\JaysPyCOM\\src',
 'D:\\my_git_repo\\JaysPyCOM\\src',
 'D:\\my_git_repo\\JaysPyCOM\\src']
> 238 INFO: checking Analysis
> 240 INFO: Building Analysis because out00-Analysis.toc is non existent
> 240 INFO: Initializing module dependency graph...
> 246 INFO: Initializing module graph hooks...
> 323 INFO: running Analysis out00-Analysis.toc
> 342 INFO: Adding Microsoft.VC90.CRT to dependent assemblies of final executable
  required by c:\tools_mcu\python27\python.exe
> 5611 INFO: Found C:\WINDOWS\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.1_none_3da38fdebd0e6822.manifest
> 5615 INFO: Found C:\WINDOWS\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.6161_none_acd388d7e1d8689f.manifest
> 5621 INFO: Found C:\WINDOWS\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.9279_none_acd3d86fe1d846c4.manifest
> 5825 INFO: Searching for assembly amd64_Microsoft.VC90.CRT_1fc8b3b9a1e18e3b_9.0.30729.9279_none ...
> 5826 INFO: Found manifest C:\WINDOWS\WinSxS\Manifests\amd64_microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.9279_none_08e667efa83ba076.manifest
> 5828 INFO: Searching for file msvcr90.dll
> 5829 INFO: Found file C:\WINDOWS\WinSxS\amd64_microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.9279_none_08e667efa83ba076\msvcr90.dll
> 5830 INFO: Searching for file msvcp90.dll
> 5832 INFO: Found file C:\WINDOWS\WinSxS\amd64_microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.9279_none_08e667efa83ba076\msvcp90.dll
> 5833 INFO: Searching for file msvcm90.dll
> 5835 INFO: Found file C:\WINDOWS\WinSxS\amd64_microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.9279_none_08e667efa83ba076\msvcm90.dll
> 6032 INFO: Found C:\WINDOWS\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.1_none_3da38fdebd0e6822.manifest
> 6033 INFO: Found C:\WINDOWS\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.6161_none_acd388d7e1d8689f.manifest
> 6034 INFO: Found C:\WINDOWS\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.9279_none_acd3d86fe1d846c4.manifest
> 6037 INFO: Adding redirect Microsoft.VC90.CRT version (9, 0, 21022, 8) -> (9, 0, 30729, 9279)
> 6175 INFO: Caching module hooks...
> 6180 INFO: Analyzing D:\my_git_repo\Jays-PyCOM\src\main.py
> 8021 INFO: Analyzing D:\my_git_repo\Jays-PyCOM\src\formatter.py
> 8024 INFO: Analyzing D:\my_git_repo\Jays-PyCOM\src\win.py
> 8040 INFO: Loading module hooks...
> 8040 INFO: Loading module hook "hook-wx.xrc.py"...
> 8046 INFO: Loading module hook "hook-encodings.py"...
> 8750 INFO: Looking for ctypes DLLs
> 8758 INFO: Analyzing run-time hooks ...
> 8766 INFO: Looking for dynamic libraries
> 9258 INFO: Looking for eggs
> 9258 INFO: Using Python library C:\WINDOWS\system32\python27.dll
> 9260 INFO: Found binding redirects:
[BindingRedirect(name=u'Microsoft.VC90.CRT', language=None, arch=u'amd64', oldVersion=(9, 0, 21022, 8), newVersion=(9, 0, 30729, 9279), publicKeyToken=u'1fc8b3b9a1e18e3b')]
9264 INFO: Warnings written to D:\my_git_repo\JaysPyCOM\bin\build\jayspycom_main\warnjayspycom_main.txt
> 9287 INFO: Graph cross-reference written to D:\my_git_repo\JaysPyCOM\bin\build\jayspycom_main\xref-jayspycom_main.html
> 9329 INFO: checking PYZ
> 9329 INFO: Building PYZ because out00-PYZ.toc is non existent
> 9330 INFO: Building PYZ (ZlibArchive) D:\my_git_repo\JaysPyCOM\bin\build\main\out00-PYZ.pyz
> 9608 INFO: Building PYZ (ZlibArchive) D:\my_git_repo\JaysPyCOM\bin\build\main\out00-PYZ.pyz completed successfully.
> 9648 INFO: checking PKG
> 9649 INFO: Building PKG because out00-PKG.toc is non existent
> 9651 INFO: Building PKG (CArchive) out00-PKG.pkg
> 9796 INFO: Redirecting Microsoft.VC90.CRT version (9, 0, 21022, 8) -> (9, 0, 30729, 9279)
> 14667 INFO: Building PKG (CArchive) out00-PKG.pkg completed successfully.
> 14674 INFO: Bootloader c:\tools_mcu\python27\lib\site-packages\PyInstaller\bootloader\Windows-64bit\runw.exe
> 14674 INFO: checking EXE
> 14677 INFO: Building EXE because out00-EXE.toc is non existent
> 14678 INFO: Building EXE from out00-EXE.toc
> 14695 INFO: SRCPATH [('..\\img\\jayspycom.ico', None)]
> 14697 INFO: Updating icons from ['..\\img\\Jays-PyCOM.ico'] to c:\users\nxa07314\appdata\local\temp\1\tmpcvu1zy
> 14698 INFO: Writing RT_GROUP_ICON 0 resource with 20 bytes
> 14698 INFO: Writing RT_ICON 1 resource with 4264 bytes
> 14707 INFO: Appending archive to EXE D:\my_git_repo\Jays-PyCOM\bin\dist\main.exe
> 14724 INFO: Building EXE from out00-EXE.toc completed successfully.
```

　　打包命令成功执行之后，便可以在\Jays-PyCOM\bin目录下看到如下生成的文件：   

```text
\JaysPyCOM
           \bin                            --放置工程源代码文件
                  \build\                    --
                  \dist\main.exe             --可执行exe文件
                  \main.spec                 --spec文件
```

　　其中build文件夹存放的是PyInstaller在打包过程中生成的调试信息文件，dist文件夹下面的main.exe便是我们要的最终的可执行文件，main.spec是PyInstaller自动生成的命令解释文件，其实你在命令行里输入的命令首先被翻译放到.spec文件里，然后PyInstaller主要是根据.spec文件来打包的，不信你可以试着用pyinstaller main.spec命令重新再打包一次，得到的结果是一样的。下面是.spec文件里的内容，如果你对.spec文件了解，当然也可以自己创建.spec文件来进行打包。  

```text
# -*- mode: python -*-

block_cipher = None


a = Analysis(['main.py', 'formatter.py', 'win.py'],
             binaries=[],
             datas=[],
             hiddenimports=[],
             hookspath=[],
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher)
pyz = PYZ(a.pure, a.zipped_data,
             cipher=block_cipher)
exe = EXE(pyz,
          a.scripts,
          a.binaries,
          a.zipfiles,
          a.datas,
          name='Jays-PyCOM',
          debug=False,
          strip=False,
          upx=True,
          runtime_tmpdir=None,
          console=False , icon='..\\img\\Jays-PyCOM.ico')
```

　　main.exe可执行文件已经生成好了，让我们试着打开使用一下，直接在\Jays-PyCOM\bin\dist\目录下打开这个文件发现报了如下错误，看起来是找不到图片路径，这是怎么回事？痞子衡其实在前面已经提到过，需要保证文件夹内图片相对路径与打包时相对路径一致，试着将main.exe放到\Jays-PyCOM\bin\目录下再打开看是不是正常了，因为这时候相对路径是一致的。大功告成了，最后将main.exe重命名为Jays-PyCOM.exe。  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_exe_error.PNG" style="zoom:100%" />

### 番外篇
　　正文中讲了，最终的Jays-PyCOM.exe必须配合Jays-PyCOM文件夹（主要是\img里的图片）一起使用，并且不能任意移动Jays-PyCOM.exe在Jays-PyCOM文件夹中位置，看起来这个Jays-PyCOM.exe还是有一些使用限制（当然你可以创建Jays-PyCOM.exe的快捷方式到桌面，你可以任意移动这个快捷方式，这算是一个workaround），能不能打破这个限制？只要一个Jays-PyCOM.exe文件即可，并且放到任意目录下都能运行？答案是有，可以参看这篇文章的思路 [pyinstaller打包——图片资源无法显示问题](https://blog.csdn.net/monster_li57/article/details/80601050)，思路大概原理是事先将图片编码存到.py源文件里，这样在打包时便可将这个图片数据.py源文件直接打包进Jays-PyCOM.exe，后续Jays-PyCOM.exe在运行时首先将图片数据解码出来并在本地保存为临时图片，这样Jays-PyCOM.exe启动便可完成图片加载，等Jays-PyCOM.exe图片加载完成之后可以删除临时图片文件。思路有了，小伙伴赶紧动手试一试，这算是痞子衡在这个系列最后一课留给大家的一个课后作业。  

　　至此，串口调试工具Jays-PyCOM诞生之打包发布痞子衡便介绍完毕了，掌声在哪里~~~  

