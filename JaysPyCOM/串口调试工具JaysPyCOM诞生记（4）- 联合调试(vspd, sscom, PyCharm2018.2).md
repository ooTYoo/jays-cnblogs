----

　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**串口调试工具Jays-PyCOM诞生之联合调试**。  

　　软件开发离不开调试，调试手段分两种：一是黑盒调试，即直接从输入/输出角度测试软件功能是否正常，这种方式仅能发现问题，但无法直接定位问题原因所在；二是白盒调试，即直接拿源代码在线debug，python虽是一种脚本语言，但借助一些Python IDE也可以实现单步调试，通过单步调试可以找到问题根本原因。  
　　前面我们已经初步实现了Jays-PyCOM，下面痞子衡会从黑盒和白盒的角度分别测试Jays-PyCOM功能：  

### 一、黑盒调试：vspd + sscom
　　要测试Jays-PyCOM功能，首先得要有串口设备，当然我们可以使用真实的物理串口设备，比如使用如下这个经典的CH34x串口转USB模块，CH34x芯片官方主页为 [http://www.wch.cn/products/category/1.html](http://www.wch.cn/products/category/1.html)。  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_test_ch340.PNG" style="zoom:100%" />

　　安装好 [CH34x模块驱动](http://www.wch.cn/products/CH341.html) 并将该模块USB接口端插上PC后便可在设备管理器的Ports里看到串口设备，一个物理模块就可以完成测试，只需要将模块的RXD和TXD线直接对接，这样便可实现回环测试。  
　　很多时候手头并没有物理串口设备，那么这时候我们就需要借助虚拟串口软件，vspd就是一款虚拟串口驱动，其官方主页为 [https://www.eltima.com/products/vspdxp/](https://www.eltima.com/products/vspdxp/)，使用vspd可以在PC上虚拟出串口设备并实现虚拟连接，由于vspd不支持单设备回环连接，那么我们需要虚拟出两个串口设备并实现连接，痞子衡使用vspd虚拟出了COM10和COM11，并将其进行了连接:  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_test_vspd_device.PNG" style="zoom:100%" />

　　痞子衡选用的是vspd虚拟串口来测试，最终搭建的黑盒测试环境示意图如下：  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_test_connection.PNG" style="zoom:100%" />

　　联合sscom测试串口数据收发，可知Jays-PyCOM基本串口数据收发功能是正常的，最基本的黑盒测试便通过了。  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_test_txrx_with_sscom.PNG" style="zoom:100%" />

### 二、白盒调试：PyCharm
　　如果在黑盒测试过程中发现Jays-PyCOM功能有问题，从代码逻辑角度也不能立刻推断出问题，此时便需要使用Python IDE进行在线debug，痞子衡选用的PyCharm软件，创建Jays-PyCOM工程后将其放于Jays-PyCOM主目录，工程会自动添加目录下所有源文件，选中main.py文件后选择Debug（Shift+F9）便可以进行单步调试。  

<img src="http://henjay724.com/image/cnblogs/JaysPyCOM_test_pycharm_debug.PNG" style="zoom:100%" />

　　关于PyCharm调试技巧可参考下面两篇文章：  

> [https://confluence.jetbrains.com/display/PYH/Debugger#Debugger-Preparinganexample](https://confluence.jetbrains.com/display/PYH/Debugger#Debugger-Preparinganexample)
> [https://www.jetbrains.com/help/pycharm/debug-tool-window.html](https://www.jetbrains.com/help/pycharm/debug-tool-window.html)

　　至此，串口调试工具Jays-PyCOM诞生之联合调试痞子衡便介绍完毕了，掌声在哪里~~~  



