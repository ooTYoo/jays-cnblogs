----
　　今天是Git系列课程第三课，前两课我们都是在做Git仓库准备工作，今天痞子衡要讲的是如何查看Git空间内发生的改动。  

　　本地有了仓库，我们便可以在仓库所在目录下做文件增删改操作，为了确定改动操作的正确性，我们需要实时查看这些改动状态，有两种查看方式git status和git diff，痞子衡为大家逐一介绍：  

### 1.查看Git空间文件改动状态git status
　　前面讲过Git空间内文件改动有4种状态，除了Unmodified状态的文件因为并未改动默认没有状态不做显示之外，其他文件改动状态都可以通过git status来查看。让我们开始在工作区创建3个文件:main.c、test.c、dummy.c  
　　dummy.c（路径：/gittest/app/dummy.c）为空白文件。  
　　test.c（路径：/gittest/app/test.c）文件内容如下：  
```C
#include <stdio.h>
#include <stdlib.h>
void test(void)
{
    printf("this is test\n");
}
```
　　main.c（路径：/gittest/main.c）文件内容如下：  
```C
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
    printf("hello world\n");
    return 0;
}
```
　　为了使改动类型更加丰富一点，我们在已存在Git本地&远程仓库的README.md文件中增加一行内容"# first update"。我们来看看Git记录的状态，从下面结果可知，新增的3个文件在Git空间里都属于Untracked文件，存放在工作区内。READMED.md文件的改动处于Modified状态，也存放在工作区。  

> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git status</font>
> ```nohighlight
> On branch master
> Your branch is up to date with 'origin/master'.
>
> Changes not staged for commit:
>   (use "git add <file>..." to update what will be committed)
>   (use "git checkout -- <file>..." to discard changes in working directory)
>
>         modified:   README.md
>
> Untracked files:
>   (use "git add <file>..." to include in what will be committed)
>
>         app/
>         main.c
>
> no changes added to commit (use "git add" and/or "git commit -a")
> ```

### 2.查看Git空间文件具体改动git diff
　　git status只能让我们知道文件在Git空间内的改动状态，但如果我们想查看某个文件内具体改了什么（也可以理解为在不同Git空间中的差异），此时需要用git diff命令。  
　　对于main.c文件，由于是新增的文件，其只存在于工作区，且处于Untracked状态，Git认为无论是哪两个Git空间之间的比对都没有意义，得到的结果是没有区别。  
　　而对于README.md文件，由于已经被提交到仓库了，处于Git管理中，所以这个文件同时存在于三个Git空间（工作区，暂存区，仓库），我们在工作区内对其进行了文件改动，无论是比对工作区vs暂存区或者工作区vs仓库，得到的结果应该都是README.md文件里的具体变化内容。而如果比对暂存区vs仓库，得到的结果也应该是没有区别。  

#### 2.1查看文件当前变动（工作区vs暂存区）git diff [file path]
> // 查看main.c得不到任何结果
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git diff main.c</font>
>
> // 查看README.md可看到文件具体变化
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git diff README.md</font>
> ```text
> diff --git a/README.md b/README.md
> index 92eca93..229dc5f 100644
> --- a/README.md
> +++ b/README.md
> @@ -1 +1,2 @@
>  # gittest
> +# first update
> ```

#### 2.2查看文件跨越变动（工作区vs仓库）git diff [commit] [file path]
　　由于gittest仓库目前只有一次提交，所以此处commit只能是HEAD，只能与上一次提交对比，得到的结果与2.1是一致的。为了充分展示这个功能，我们将此次的README.md的改动先提交到仓库。  
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git add README.md</font>
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git commit -m "second commit"</font>
> ```text
> [master aa9db9d] second commit
>  1 file changed, 1 insertion(+)
> ```
>
> jay@pc  MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git fetch</font>
>
> jay@pc  MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git rebase</font>
> ```text
> First, rewinding head to replay your work on top of it...
> Applying: second commit
> ```
>
> jay@pc  MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git push</font>
> ```text
> Counting objects: 3, done.
> Writing objects: 100% (3/3), 276 bytes | 276.00 KiB/s, done.
> Total 3 (delta 0), reused 0 (delta 0)
> To github.com:JayHeng/gittest.git
>    5fe04f8..867df08  master -> master
> ```

　　然后对README.md再修改一次增加新一行内容"# second update"。现在我们再来查看README.md跨级变动（HEAD表示是最近一次commit，HEAD^表示上一次commit，HEAD~100表示上100次commit）：  
> // 查看README.md与最近一次commit的变化（等同于当前变化）
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git diff HEAD README.md</font>
> ```text
> diff --git a/README.md b/README.md
> index 229dc5f..db5442d 100644
> --- a/README.md
> +++ b/README.md
> @@ -1,2 +1,3 @@
>  # gittest
>  # first update
> +# second update
> ```
>
> // 查看README.md与上一次commit的变化（等同于2次变化的汇总）
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git diff HEAD^ README.md</font>
> ```text
> diff --git a/README.md b/README.md
> index 92eca93..db5442d 100644
> --- a/README.md
> +++ b/README.md
> @@ -1 +1,3 @@
>  # gittest
> +# first update
> +# second update
> ```

　　对于README.md文件的第二次改动仅是用于演示跨越变动对比的功能，为不影响后续讲解，我们现在将这个变动恢复（文件编辑器打开文件，直接删除"# second update"）。  

#### 2.3查看文件历史变更（仓库vs仓库）git diff [commit] [commit]
　　gittest仓库目前已有2次提交，让我们直接比对这两次提交。Note：Git每次commit都会产生一个唯一ID（SHA-1号）用于记录这个commit，可在git commit/git push命令的返回结果里看到。  
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git diff 5fe04f8 867df08</font>
> ```text
> diff --git a/README.md b/README.md
> index 92eca93..229dc5f 100644
> --- a/README.md
> +++ b/README.md
> @@ -1 +1,2 @@
>  # gittest
> +# first update
> ```
