----
　　今天是Git系列课程第六课，上一课我们学会了Git本地提交，今天痞子衡要讲的是如何查看Git本地历史提交。  

　　当我们在仓库里做了很多次提交之后，免不了需要回看提交记录，看看自己之前的改动。有三种Git命令可以帮我们查看记录，痞子衡为大家一一讲解：  

### 1.查看本地历史提交git log
　　git log是最直接的查看历史提交的命令，git log可直接用也可带参数用，常用的有下面4种：  
#### 1.1标准查看git log
> // 显示所有历史提交标准信息，每个提交信息包括SHA号，作者，时间以及标题
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git log</font>
> ```text
> commit ea3925e786f7975265fd43eface72f48af4306dd (HEAD -> master)
> Author: Jay Heng <hengjie1989@foxmail.com>
> Date:   Sun Mar 11 07:46:16 2018 +0800
>
>     Add initial platform and update test
>
> // 此处略去其他commit信息
> ...
> ```

#### 1.2精简查看git log --pretty=oneline
> // 显示所有历史提交精简信息，每个提交信息仅占一行，信息包括SHA号以及标题。  
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git log --pretty=oneline</font>
> ```text
> ea3925e786f7975265fd43eface72f48af4306dd (HEAD -> master) Add initial platform and update test
> fdec58a389772a14f71c391214e90f5c5c00570a Initial application and test
> 867df08b4e13649e30926b483279dddce32750c2 (origin/master, origin/HEAD) second commit
> 5fe04f86701d1d0ccb710140d440fa86daab5ffb first commit
> ```

#### 1.3完整查看git log -p
> // 显示所有历史提交完整信息，比标准查看多了提交的具体文件改动信息。  
> jay@pc MINGW64 MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git log -p</font>
> ```text
> commit ea3925e786f7975265fd43eface72f48af4306dd (HEAD -> master)
> Author: Jay Heng <hengjie1989@foxmail.com>
> Date:   Sun Mar 11 07:46:16 2018 +0800
>
>     Add initial platform and update test
>
> diff --git a/app/platform.c b/app/platform.c
> new file mode 100644
> index 0000000..e69de29
> diff --git a/app/test.c b/app/test.c
> index e69de29..70dde01 100644
> --- a/app/test.c
> +++ b/app/test.c
> @@ -0,0 +1,6 @@
> +#include <stdio.h>
> +#include <stdlib.h>
> +void test(void)
> +{
> +    printf("this is test\n");
> +}
> \ No newline at end of file
>
> // 此处略去其他commit信息
> ...
> ```

#### 1.4定制查看git log --pretty=format:"%opt1 %opt2" --graph
> // 按指定格式显示所有提交历史信息。  
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git log --pretty=format:"%h %an %s" --graph</font>
> ```text
> * ea3925e Jay Heng Add initial platform and update test
> * fdec58a Jay Heng Initial application and test
> * 867df08 Jay Heng second commit
> * 5fe04f8 Jay first commit
> ```

　　其中opt选项列出如下：  
```text
%H  提交对象（commit）的完整哈希字串
%h  提交对象的简短哈希字串
%T  树对象（tree）的完整哈希字串
%t  树对象的简短哈希字串
%P  父对象（parent）的完整哈希字串
%p  父对象的简短哈希字串
%an 作者（author）的名字
%ae 作者的电子邮件地址
%ad 作者修订日期（可以用 -date= 选项定制格式）
%ar 作者修订日期，按多久以前的方式显示
%cn 提交者(committer)的名字
%ce 提交者的电子邮件地址
%cd 提交日期
%cr 提交日期，按多久以前的方式显示
%s  提交说明
```

### 2.图形化查看本地历史gitk
　　如果你觉得git log这种命令行方式查看与显示提交记录不够直观，Git也提供了图形化方式显示提交记录。  
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ gitk</font>
> ![](http://henjay724.com/image/cnblogs/gittest_gitk_4commits_full_screen.PNG)

### 3.查看本地历史操作git reflog
　　无论是gitk还是git log都仅能查看最终在仓库存在的提交信息，无法查看被删除的提交，以及在本地具体Git命令操作记录，这时候你需要使用git reflog。  
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git reflog</font>
> ```text
> ea3925e (HEAD -> master) HEAD@{0}: am: Add initial platform and update test
> fdec58a HEAD@{1}: am --abort
> fdec58a HEAD@{2}: am: Initial application and test
> 867df08 (origin/master, origin/HEAD) HEAD@{3}: reset: moving to HEAD~1
> b69153f HEAD@{4}: am: Initial application and test
> 867df08 (origin/master, origin/HEAD) HEAD@{5}: reset: moving to HEAD~2
> 610feaf HEAD@{6}: commit: Add initial platform and update test
> 589f65b HEAD@{7}: reset: moving to HEAD
> 589f65b HEAD@{8}: reset: moving to HEAD
> 589f65b HEAD@{9}: reset: moving to 589f65b
> 4378dee HEAD@{10}: commit: Initial platform and driver
> 589f65b HEAD@{11}: reset: moving to 589f65b
> 1eaa025 HEAD@{12}: reset: moving to HEAD
> 1eaa025 HEAD@{13}: commit: Initial platform and driver
> 589f65b HEAD@{14}: commit (amend): Initial application and test
> 0a0c0fc HEAD@{15}: commit: Initial application
> 867df08 (origin/master, origin/HEAD) HEAD@{16}: rebase finished: returning to refs/heads/master
> 867df08 (origin/master, origin/HEAD) HEAD@{17}: rebase: second commit
> 5fe04f8 HEAD@{18}: rebase: checkout refs/remotes/origin/master
> aa9db9d HEAD@{19}: commit: second commit
> 5fe04f8 HEAD@{20}: reset: moving to HEAD
> 5fe04f8 HEAD@{21}: clone: from git@github.com:JayHeng/gittest.git
> ```
