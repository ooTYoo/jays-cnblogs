----
　　今天是Git系列课程第五课，上一课我们做了Git本地提交前的准备工作，今天痞子衡要讲的是Git本地提交操作。  

　　当我们在仓库工作区下完成了文件增删改操作之后，并且使用git add将文件改动记录在暂存区之后，便可以开始将其提交到Git本地仓库。  

### 1.本地文件改动提交git commit
　　Git空间本地的改动完成之后可以直接提交，有如下三种提交命令选项：  
#### 1.1将暂存区内容提交git commit -m ["description"]
　　暂存区里目前只有app/app.c文件，我们先将其提交至仓库。  
> // 将暂存区里所有改动提交到本地仓库，提交标题为"Initial application"
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git commit -m "Initial application"</font>
> ```text
> [master 0a0c0fc] Initial application
>  1 file changed, 7 insertions(+)
>  create mode 100644 app/app.c
> ```
>
> // 查看本地提交日志，确认提交是否已记录在仓库中
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git log</font>
> ```text
> commit 0a0c0fcec8a1ef56bfc6a24e68bbf1436b2ef2cf (HEAD -> master)
> Author: Jay Heng <hengjie1989@foxmail.com>
> Date:   Sat Mar 10 21:58:36 2018 +0800
>
>     Initial application
>
> commit 867df08b4e13649e30926b483279dddce32750c2 (origin/master, origin/HEAD)
> Author: Jay Heng <hengjie1989@foxmail.com>
> Date:   Sat Mar 10 20:11:04 2018 +0800
>
>     second commit
> ```

#### 1.2追加提交git commit --amend -m ["description"]
　　工作区里面还有app/test.c文件处于Untracked状态，我们想将其也加到1.1的提交里合并成一个提交。  
> // 将Untracked状态的test.c文件添加到暂存区，用于新提交
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git add app/test.c</font>
>
> // 将暂存区里新改动追加提交到本地仓库，与上一次提交进行合并，并修改提交标题为"Initial application and test"
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git commit --amend -m "Initial application and test"</font>
> ```text
> [master 589f65b] Initial application and test
>  Date: Sat Mar 10 21:58:36 2018 +0800
>  2 files changed, 7 insertions(+)
>  create mode 100644 app/app.c
>  create mode 100644 app/test.c
> ```
>
> // 查看本地提交日志，确实两次提交被合并成了一个
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git log</font>
> ```text
> commit 589f65b386dd4475bb884c40ea1441d8449fdcd1 (HEAD -> master)
> Author: Jay Heng <hengjie1989@foxmail.com>
> Date:   Sat Mar 10 21:58:36 2018 +0800
>
>     Initial application and test
>
> commit 867df08b4e13649e30926b483279dddce32750c2 (origin/master, origin/HEAD)
> Author: Jay Heng <hengjie1989@foxmail.com>
> Date:   Sat Mar 10 20:11:04 2018 +0800
>
>     second commit
> ```

#### 1.3非Untracked文件的改动全部提交git commit -a -m ["description"]
　　我们新增一个名叫platform.c空白文件，并将其git add到暂存区；且对已提交的空白test.c文件修改恢复其一开始的内容。此时我们工作区(test.c)和暂存区(platform.c)均存在文件改动。有没有可能一次性将test.c和platform.c改动提交上去，答案当然是有。  
> // 将platform.c改动添加到暂存区
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git add app/platform.c</font>
>
> // 查看Git空间当前状态，test.c改动在工作区，platform.c改动在暂存区
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```text
> On branch master
> Your branch is ahead of 'origin/master' by 1 commit.
>   (use "git push" to publish your local commits)
>
> Changes to be committed:
>   (use "git reset HEAD <file>..." to unstage)
>
>         new file:   app/platform.c
>
> Changes not staged for commit:
>   (use "git add <file>..." to update what will be committed)
>   (use "git checkout -- <file>..." to discard changes in working directory)
>
>         modified:   app/test.c
> ```
>
> // 将Git空间内非Untracked改动之外的所有改动（无论是工作区/暂存区）一次性提交到本地仓库，此处为test.c和platform.c
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git commit -a -m "Add initial platform and update test"</font>
> ```text
> [master 610feaf] Add initial platform and update test
>  2 files changed, 6 insertions(+)
>  create mode 100644 app/platform.c
> ```
>
> // 查看Git空间当前状态，没有任何改动记录，说明test.c和platform.c改动已被提交到仓库
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```text
> On branch master
> Your branch is ahead of 'origin/master' by 2 commits.
>   (use "git push" to publish your local commits)
>
> nothing to commit, working tree clean
> ```
>
> // 查看本地提交日志，又多了一次名为"Add initial platform and update test"的新提交
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git log</font>
> ```text
> commit 610feafbf5264b66dd0515f90cce79169aebd995 (HEAD -> master)
> Author: Jay Heng <hengjie1989@foxmail.com>
> Date:   Sun Mar 11 07:46:16 2018 +0800
>
>     Add initial platform and update test
>
> commit 589f65b386dd4475bb884c40ea1441d8449fdcd1
> Author: Jay Heng <hengjie1989@foxmail.com>
> Date:   Sat Mar 10 21:58:36 2018 +0800
>
>     Initial application and test
> ```

　　需要注意的是git commit -a虽然是-all的简写，但其并不是将Git空间内所有改动全部提交，其不会提交工作区里新建的文件（Untracked状态）。  

### 2.补丁包方式提交
　　Git是分布式版本控制系统，一个项目常常由多个人一起开发，有时候我们想把本地的改动交给别人去完成提交，传统的做法是把你改动的所有文件全部提取出来打包发送给对方，但显然这样做比较繁琐，而且你的提交描述也容易丢失。Git提供了一种很好方式解决这个需求，即生成一个patch，这个patch就是你本地的一次提交的所有信息，你只需要把这个patch发给别人就可以了。  

#### 2.1生成补丁包git format-patch
　　git format-patch命令是以本地提交为基础的，让我们先看看目前本地有多少提交：  
> // 图形化方式查看本地仓库里的历史提交，共记录4个提交
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ gitk</font>
> ![](http://henjay724.com/image/cnblogs/gittest_gitk_4commits.PNG)

　　由上图可知，仓库里一共有4次提交，其中两次已推送到远程，还有两次在本地未推送：  

#### 2.1.1指定任意单个提交git format-patch -1 [commit]
　　让我们试着将"Initial application and test"这个提交生成patch:  
> // 将SHA-1号前7位为589f65b的提交生成一个patch
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git format-patch -1 589f65b</font>
> 0001-Initial-application-and-test.patch

　　此时在gittest仓库目录下便可以看到生成的.patch文件，让我们用文本编辑器打开它看一看，确实包含了这个提交的所有改动。  
```diff
From 589f65b386dd4475bb884c40ea1441d8449fdcd1 Mon Sep 17 00:00:00 2001
From: Jay Heng <hengjie1989@foxmail.com>
Date: Sat, 10 Mar 2018 21:58:36 +0800
Subject: [PATCH] Initial application and test

---
 app/app.c  | 7 +++++++
 app/test.c | 0
 2 files changed, 7 insertions(+)
 create mode 100644 app/app.c
 create mode 100644 app/test.c

diff --git a/app/app.c b/app/app.c
new file mode 100644
index 0000000..20fe868
--- /dev/null
+++ b/app/app.c
@@ -0,0 +1,7 @@
+#include <stdio.h>
+#include <stdlib.h>
+int main(void)
+{
+    printf("hello world\n");
+    return 0;
+}
\ No newline at end of file
diff --git a/app/test.c b/app/test.c
new file mode 100644
index 0000000..e69de29
-- 
2.16.2.windows.1
```

#### 2.1.2当前提交之前n个提交git format-patch -n
　　如果一次想把本地未推送的提交全部生成patch，也可以试试-n参数，-1代表生成最近的一次提交的patch，-2代表生成最近两次提交的patch，以此类推。这个参数也可以用HEAD替换，HEAD^ 等效于-1，HEAD^^等效于-2，HEAD~n等效于-n。  
> // 将最近3次提交全部生成patch
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git format-patch HEAD~3</font>
> ```text
> 0001-second-commit.patch
> 0002-Initial-application-and-test.patch
> 0003-Add-initial-platform-and-update-test.patch
> ```
>
> // 依然是将最近3次提交全部生成patch
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git format-patch -3</font>
> ```text
> 0001-second-commit.patch
> 0002-Initial-application-and-test.patch
> 0003-Add-initial-platform-and-update-test.patch
> ```

#### 2.1.3某个提交之后的所有提交git format-patch [commit]
　　如果你觉得用-n参数显得有点繁琐，比如本地有很多个提交，你需要先往回数一共有多少个。还有一个方法是直接指定某个提交，以某个提交为基准往后的所有提交全部生成patch，比如我们以"second commit"为基准：  
> // 将SHA-1号前7位为867df08的提交之后的所有提交全部生成patch
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git format-patch 867df08</font>
> ```text
> 0001-Initial-application-and-test.patch
> 0002-Add-initial-platform-and-update-test.patch
> ```

#### 2.2应用补丁包git am
　　2.1.3节生成了2个补丁包，让我们试着用一下这两个补丁包，在用之前我们需要先把本地仓库中这两次提交撤销并且也不保留在工作区（即完全删除）。  
> // 将最近两次提交在仓库里直接删除，不留痕迹
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git reset --hard HEAD~2</font>
> ```text
> HEAD is now at 867df08 second commit
> ```
>
> // 查看本地提交日志，确实最近两次提交已被删除
> jay@pc MINGW64 /d/my_project/gittest (master)
> $ <font style="font-weight:bold;">gitk</font>
> ![](http://henjay724.com/image/cnblogs/gittest_gitk_2commits.PNG)
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```text
> On branch master
> Your branch is up to date with 'origin/master'.
>
> Untracked files:
>   (use "git add <file>..." to include in what will be committed)
>
>         0001-Initial-application-and-test.patch
>         0002-Add-initial-platform-and-update-test.patch
>
> nothing added to commit but untracked files present (use "git add" to track)
> ```

#### 2.2.1指定单个补丁包git am [patch path]
　　让我们试着将0001-Initial-application-and-test.patch打入我们的仓库。  
> // 将名为0001-Initial-application-and-test的patch文件打入仓库
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git am 0001-Initial-application-and-test.patch</font>
> ```text
> Applying: Initial application and test
> ```
>
> // 查看本地提交日志，确实patch已经进入仓库
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ gitk</font>
> ![](http://henjay724.com/image/cnblogs/gittest_gitk_3commits.PNG)

　　细心的朋友可能会注意到，同一个patch在生成和打入的时候SHA-1号是不一样的，是的，Git会保证任何时候任何人在任何本地仓库生成的任何提交都是唯一的。  

#### 2.2.2指定目录下所有补丁包git am [patch dir/*.patch]
　　让我们试着将gittest目录下的所有patch全部打入我们的仓库，需要注意的是由于2.2.1中已经将编号为0001 patch打入了仓库，所有我们需要先将这个patch文件删除，否则在打入的时候会报错。  
> // 将gittest目录下的所有patch文件全部打入仓库
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git am *.patch</font>
> ```text
> Applying: Add initial platform and update test
> ```
>
> // 查看本地提交日志，确实patch已经都进入仓库
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ gitk</font>
> ![](http://henjay724.com/image/cnblogs/gittest_gitk_4commits_with_sha.PNG)
