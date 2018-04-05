----
　　今天是Git系列课程第一课，痞子衡给大家要讲的是创建仓库的准备工作。  

### 1.建仓库git init
　　第一步是创建一个空仓库，这是一切操作的前提。  
> // 打开git bash命令行，切换到指定目录下
> jay@pc MINGW64 /
> <font style="font-weight:bold;">$ cd /d/my_project/</font>
>
> // 在指定目录下创建存放repo的文件夹，示例为gittest
> jay@pc MINGW64 /d/my_project
> <font style="font-weight:bold;">$ mkdir gittest</font>
>
> // 切换到gittest目录下
> jay@pc MINGW64 /d/my_project
> <font style="font-weight:bold;">$ cd gittest/</font>
>
> // 使用git init命令创建一个空仓库
> jay@pc MINGW64 /d/my_project/gittest
> <font style="font-weight:bold;" color="Blue">$ git init</font>
> ```nohighlight
> Initialized empty Git repository in D:/my_project/gittest/.git/
> ```

　　空仓库创建完成后gittest文件夹下会生成一个.git隐藏文件夹。仓库默认包含一个主支，即master，默认操作都是在主分支master上进行的。  

### 2.配置仓库信息git config
　　有了空仓库，我们便可以进行后续提交操作，但在提交之后需要做一些必要配置，Git的配置从上到下分三层system/global/local，此处我们仅用local选项对当前仓库操作做配置（即配置只对当前仓库有效）。  
> // 设置提交代码时的local用户信息（用户名，email地址）
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git config --local user.name "Jay Heng"</font>
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git config --local user.email "hengjie1989@foxmail.com"</font>
>
> // 查看local层次的config参数配置是否生效
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git config --local --list</font>
> ```nohighlight
> core.repositoryformatversion=0
> core.filemode=false
> core.bare=false
> core.logallrefupdates=true
> core.symlinks=false
> core.ignorecase=true
> user.name=Jay Heng
> user.email=hengjie1989@foxmail.com
> ```

　　设置好user.name, user.email两个必要用户信息后，后续任何提交都会默认包含此用户信息。  

### 3.设置过滤文件.gitignore
　　有了仓库，我们便可以在gittest文件夹下的工作区做文件增删修改工作了，但很多时候，我们只在意开发过程中的源文件，并不需要管理自动产生的其他临时文件。这时候我们便需要一个过滤文件，在这个文件中设置过滤规则，让Git能够自动过滤掉那些临时文件，这个文件便是.gitignore文件。  
> // 创建空的gitignore文件
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ touch .gitignore</font>
>
> // 文本编辑器打开gitignore文件，写入过滤规则
> ```nohighlight
> /project/demo.o       #过滤具体文件demo.o
> /project/settings/    #过滤整个settings文件夹
> *.o                   #过滤所有.o文件
> ```

　　上面仅列举了3种常用的过滤规则，可根据下面的过滤配置语法组合出任意你想要的过滤规则。  
> * 以斜杠“/”开头表示目录
> * 以星号“*”通配多个字符
> * 以问号“?”通配单个字符
> * 以方括号“[]”包含单个字符的匹配列表
> * 以叹号“!”表示不忽略(跟踪)匹配到的文件或目录

　　如果希望设置的过滤规则不仅仅对本地仓库的操作有效，也希望对其他机器上该仓库的操作有效，可以.gitignore提交到仓库中并且推送到远程，提交及推送操作后续会介绍。  