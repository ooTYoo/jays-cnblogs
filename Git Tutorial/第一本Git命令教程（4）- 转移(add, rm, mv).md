----
　　今天是Git系列课程第四课，上一课我们在Git空间里做了一些文件改动并且知道了如何利用Git查看这些变动，今天痞子衡要讲的是将这些变动提交到Git本地仓库前的准备工作。  

　　Git仓库目录下的文件改动操作默认都发生在Git工作区内，Git并不会主动管理。如果希望Git能够管理这些变动，你需要主动通知Git。共有3种通知Git的命令(git add/rm/mv)，痞子衡为大家一一讲解。  

### 1.将工作区文件改动添加到暂存区git add
　　git add是第一种通知Git命令，这个命令用于告诉Git我们新增了文件改动，被git add命令操作过的文件(改动)便会处于Git暂存区。  

#### 1.1添加单文件改动git add [file path]
　　上一节课我们已经在工作区创建了3个文件，让我们开始用git add将它们一一添加到暂存区：  
> // 将main.c，test.c, dummy.c分别添加到暂存区
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git add main.c</font>
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git add app/test.c</font>
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git add app/dummy.c</font>
>
> // 查看此时的文件状态，3个文件都已在暂存区中了
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```nohighlight
> On branch master
> Your branch is up to date with 'origin/master'.
>
> Changes to be committed:
>   (use "git reset HEAD <file>..." to unstage)
>
>         new file:   app/dummy.c
>         new file:   app/test.c
>         new file:   main.c
> ```

#### 1.2添加文件夹内全部改动git add -A [folder path]
　　有没有觉得git add [filepath]一次只能添加一个文件不够高效？别急，你还可以按文件夹来提交。这时我们再做一些改动，将dummy.c文件删除，将test.c文件里的内容全部删除，再新增一个名叫trash.c的文件。  
> // 查看dummy.c，test.c, track.c状态
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```nohighlight
> On branch master
> Your branch is up to date with 'origin/master'.
>
> Changes to be committed:
>   (use "git reset HEAD <file>..." to unstage)
>
>         new file:   app/dummy.c
>         new file:   app/test.c
>         new file:   main.c
>
> Changes not staged for commit:
>   (use "git add/rm <file>..." to update what will be committed)
>   (use "git checkout -- <file>..." to discard changes in working directory)
>
>         deleted:    app/dummy.c
>         modified:   app/test.c
>
> Untracked files:
>   (use "git add <file>..." to include in what will be committed)
>
>         app/trash.c
> ```

　　让我们试试git add -A [folderpath]，执行完这个命令后，我们可以看到app/文件夹下的所有类型的文件改动（新增、修改、删除）被一次性地存储到了暂存区，这下是不是效率高了很多。有兴趣的朋友还可以继续研究git add .（不包括删除操作）和git add -u（不包括新增操作）两个命令，实际上git add -A是这两个命令的并集。  
> // 将app/文件夹下的所有类型的文件改动全部添加到暂存区
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git add -A app/</font>
>
> // 查看此时Git状态，尤其是app/文件夹下的文件状态
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```nohighlight
> On branch master
> Your branch is up to date with 'origin/master'.
>
> Changes to be committed:
>   (use "git reset HEAD <file>..." to unstage)
>
>         new file:   app/trash.c
>         new file:   app/test.c
>         new file:   main.c
> ```

### 2.将暂存区文件改动退回git rm
　　git rm是第二种通知Git命令，这个命令用于告诉Git我们想把之前用git add添加的文件改动从Git暂存区里拿出去，不需要Git记录了。拿出去有两种拿法，一种是从暂存区退回到工作区，另一种是直接丢弃文件改动。让我们试着将test.c退回到工作区，trash.c直接丢弃。  

#### 2.1退回文件改动到工作区git rm --cache [file path]
> // 将test.c的改动从暂存区移回工作区
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git rm --cache app/test.c</font>
> ```nohighlight
> rm 'app/test.c'
> ```
>
> // 查看test.c是否已经移回到工作区
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```nohighlight
> On branch master
> Your branch is up to date with 'origin/master'.
>
> Changes to be committed:
>   (use "git reset HEAD <file>..." to unstage)
>
>         new file:   app/trash.c
>         new file:   main.c
>
> Untracked files:
>   (use "git add <file>..." to include in what will be committed)
>
>         app/test.c
> ```

#### 2.2直接删除文件git rm -f [file path]
> // 将track.c文件直接从Git空间里删除，不留痕迹
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git rm -f app/trash.c</font>
> ```nohighlight
> rm 'app/trash.c'
> ```
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```nohighlight
> On branch master
> Your branch is up to date with 'origin/master'.
>
> Changes to be committed:
>   (use "git reset HEAD <file>..." to unstage)
>
>         new file:   main.c
>
> Untracked files:
>   (use "git add <file>..." to include in what will be committed)
>
>         app/
```

### 3.将暂存区文件移动位置/重命名git mv
　　git mv是第三种通知Git命令，这个命令用于告诉Git我们想把之前用git add添加的文件直接在暂存区里重新命名或移动到新位置。让我们试着将main.c重命名为app.c，并移动到app/文件夹下。  

#### 3.1文件重命名git mv [src file] [dest file]
> // 将main.c在暂存区里直接改名为app.c（app.c也记录在暂存区）
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git mv main.c app.c</font>
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```nohighlight
> On branch master
> Your branch is up to date with 'origin/master'.
>
> Changes to be committed:
>   (use "git reset HEAD <file>..." to unstage)
>
>         new file:   app.c
>
> Untracked files:
>   (use "git add <file>..." to include in what will be committed)
>
>         app/
> ```

#### 3.2移动文件位置git mv [src file] [dest dir]
> // 将app.c从gittest主目录移动到app/目录下（移动操作记录在暂存区）
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git mv app.c app/</font>
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git status</font>
> ```nohighlight
> On branch master
> Your branch is up to date with 'origin/master'.
>
> Changes to be committed:
>   (use "git reset HEAD <file>..." to unstage)
>
>         new file:   app/app.c
>
> Untracked files:
>   (use "git add <file>..." to include in what will be committed)
>
>         app/test.c
> ```