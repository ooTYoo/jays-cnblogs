----
　　大家好，我是痞子衡，是正经搞技术的痞子。本系列痞子衡给大家讲的是**Git命令汇编**，共12篇文章，循序渐进地介绍Git操作的完整过程。  

　　在开始Git课程之前，需要先跟大家普及2个重要概念（四度空间、四种状态），后续课程都是围绕这两个重要概念展开的。  

### 四度空间
　　第一个重要概念是Git的四度空间。在Git仓库目录下的文件改动（增删改操作）共有如下4个空间来记录/存储，Git命令就是用于将文件改动切换到不同的空间来记录。  

> * Workspace：工作区
> * Index / Stage / Cached：暂存区
> * Repository：本地仓库
> * Remote：远程仓库

　　如果你只是Git的轻度用户，原则上只需要记住如下图所示的7个Git命令就可以了。这7个命令可以帮你将文件改动记录到任意Git空间。  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/gittest%20-%20four%20spaces.PNG" style="zoom:65%" />  

### 四种状态
　　前面讲了Git有四度空间，而单就文件改动状态层面而言，Git空间内的文件也有4种状态（需要注意的是文件状态并不是与Git空间一一对应的），这是Git第二个重要概念。  

> * Untracked：新增的文件的状态，未受Git管理，记录在工作区
> * Modified：受Git管理过的文件的改动状态（包括改动内容、删除文件），记录在工作区
> * Staged：将记录在工作区的文件变动状态通知了Git，记录在暂存区
> * Unmodified：受Git管理中的文件状态（没有变动），记录在本地仓库/远程仓库

<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/gittest%20-%20file%20status.PNG" style="zoom:85%" />  

### 正文十二篇（持续更新中...）
　　知道了2个Git重要概念，我们便可以开始Git的命令学习，痞子衡课程使用的Git版本是2.16.2，共十二节课，Enjoy it！  

> [第一本Git命令教程（1）- 准备(init/config/gitignore)](http://www.cnblogs.com/henjay724/p/8525512.html)
> [第一本Git命令教程（2）- 连接(remote/clone)](http://www.cnblogs.com/henjay724/p/8525957.html)  
> [第一本Git命令教程（3）- 变动(status/diff)](http://www.cnblogs.com/henjay724/p/8537277.html)  
> [第一本Git命令教程（4）- 转移(add/rm/mv)](http://www.cnblogs.com/henjay724/p/8541981.html)  
> [第一本Git命令教程（5）- 提交(commit/format-patch/am)](http://www.cnblogs.com/henjay724/p/8543292.html)  
> [第一本Git命令教程（6）- 日志(log/reflog/gitk)](http://www.cnblogs.com/henjay724/p/8593034.html)  

> [第一本Git命令教程（7）- 清理(revert/reset/stash/clean)](http://www.cnblogs.com/henjay724/p/8525497.html)  
> [第一本Git命令教程（8）- 分支(branch/checkout)](http://www.cnblogs.com/henjay724/p/8525497.html)  
> [第一本Git命令教程（9）- 更新(pull/fetch)](http://www.cnblogs.com/henjay724/p/8525497.html)  
> [第一本Git命令教程（10）- 整合(cherry-pick/merge/rebase)](http://www.cnblogs.com/henjay724/p/8525497.html)  
> [第一本Git命令教程（11）- 推送(push)](http://www.cnblogs.com/henjay724/p/8525497.html)  
> [第一本Git命令教程（12）- 发布(tag/archive)](http://www.cnblogs.com/henjay724/p/8525497.html)  

### 参考资料
[Git命令官方手册](https://git-scm.com/docs)  
[猴子都能懂的GIT入门](https://backlog.com/git-tutorial/cn/)
[廖雪峰的Git完整教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)  
[Mary Rose Cook的深入浅出Git](https://blog.coding.net/blog/git-from-the-inside-out)
[深入浅出Git教程（转载）](https://www.cnblogs.com/syp172654682/p/7689328.html)
[阮一峰的Git命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)  
[景春雷的Git命令脑图](http://www.cnblogs.com/1-2-3/archive/2010/07/18/git-commands.html)  
[InMicro的Git文件状态](https://www.cnblogs.com/ptqueen/p/6723147.html)
[精进吧Aaron的常用Git命令](https://segmentfault.com/a/1190000011673663)
