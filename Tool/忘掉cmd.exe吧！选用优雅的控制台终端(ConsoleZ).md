----
　　大家好，我是痞子衡，是正经搞技术的痞子。今天痞子衡给大家介绍的是**一款优雅的替换cmd的命令行终端ConsoleZ**。  

### 1.使用cmd的烦恼
　　嵌入式开发经常会用到命令行工具，Windows系统自带的command line工具（cmd.exe）的简陋程度不用说大家都深有体会。使用cmd.exe有如下几个主要的烦恼：  
> * 没有多标签支持：打开多个命令行窗口，任务栏下面就会显示多个终端窗口，且这些窗口都没有命名，切换极不方便。
> * 复制粘贴太麻烦：复制粘贴的操作之繁琐简直让人无法接受。
> * 窗口大小不可调：窗口宽度不可调整，对于比较长的命令输入或log显示，看起来极不雅观。

### 2.Console项目
　　在讲本文主角ConsleZ之前有必要提一下这个软件的前身，ConsoleZ实际上是Console项目的一个分支，Console是由Marko Bozikovic维护在SourceForge上的开源项目，第一个正式版本1.0于2002年1月发布。目前最新的版本是2.00b148-Beta（截止到2017年9月），发布于2011年10月。  

> Console项目官方主页(SourceForge) https://sourceforge.net/projects/console/  
> Console项目Github：https://github.com/bozho/console

　　Console(Console2)可以理解为cmd.exe的前端（Windows系统下），和Konsole，Gnome Ternimal之于bash的角色（Linux系统下）是一样的。

### 3.ConsoleZ项目
　　由于Console已经很久没有更新，Christophe Bucher在Console项目基础上开发出了ConsoleZ。相比Console，ConsoleZ主要是在更新的系统Windows Vista/7/8/10下的体验以及视觉效果上有进一步改进。目前最新的版本是1.18.2（截止到2017年9月），发布于2017年9月。  

> ConsoleZ项目官方主页(Github) https://github.com/cbucher/console  

　　跟Console一样，ConsoleZ也只是个shell工具（cmd.exe）的前端，它本身并没有实现shell工具的功能，它只是基于shell工具做了一个包装。无论是Console还是ConsoleZ，都可以解决我们在使用cmd.exe时的烦恼。如下是ConsoleZ（Console）基本特性：  
> * multiple tabs（多标签）
> * text editor-like text selection（像文本编辑器一样编辑）
> * different background types (solid color, image, fake transparency)（可设背景样式）
> * configurable font（可设字体类型）
> * different window styles（不同窗口式样）

　　除了上述基本特性外，ConsoleZ还支持更多有用的特性：  
> * Splitting Tabs into views (horizontally and vertically)（同窗分屏显示多标签）
> * Grouping views (so input sent to one goes to all of them)
> * and more...

　　尤其是看到同窗分屏显示多标签，小伙伴们是不是有点激动？在Edit->Settings->Hotkeys里找到或定义Spilt Horizontally/Vertically的热键，然后使用热键将打开的Console窗口按需分屏，下面贴一张效果图：  

<img src="http://henjay724.com/image/cnblogs/ConsoleZ-Splitting%20Tabs.PNG" style="zoom:70%" /></td>

### 4.如何包装更多的shell工具？
　　cmd.exe是ConsoleZ默认包装的shell工具，除了cmd.exe外，我们还会用到其他的shell工具，比如Git bash（痞子衡安装的版本是v2.12.0 x64）。那么ConsoleZ如何包装Git bash呢？在Edit->Settings->Tabs里使用Add新建一个Tab（痞子衡新建的叫ConsoleZ - git bash），然后将Main框里的一些选项配置上，其中最重要的是Shell一栏，需填入如下语句（cmd.exe和sh.exe路径需要根据自己PC路径而定）：  
```C
C:\Windows\SysWOW64\cmd.exe /c "C:\mcu_tools\Git\bin\sh.exe --login -i"
```

<img src="http://henjay724.com/image/cnblogs/ConsoleZ-Work_with_Git_bash.PNG" style="zoom:70%" /></td>

　　配置好之后新建Tab时选择ConsoleZ - git bash便可以看到Console打开的是Git bash。  

　　至此，命令行终端ConsoleZ痞子衡便介绍完毕了，掌声在哪里~~~ 

