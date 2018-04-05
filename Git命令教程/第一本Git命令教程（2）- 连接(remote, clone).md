----
　　今天是Git系列课程第二课，上一课我们已经学会在本地创建一个空仓库，痞子衡今天要讲的是如何将本地仓库与远程建立联系。  

#### 1.将本地仓库挂上远程git remote
　　本地建好了仓库，我们希望能够挂到远程服务器上，方便与其他人共享。目前最流行的远程Git服务器当然是github，此时你需要在github上注册账户并在线创建一个仓库，此处我们输入仓库名为gittest  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/gittest%20-%20create.PNG" style="zoom:70%" />  
　　点击"Create repository"之后便弹出如下画面，最重要的是我们可以得到一个远程仓库的地址：git@github.com:JayHeng/gittest.git。  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/gittest%20-%20url.PNG" style="zoom:70%" />  
　　有了远程仓库地址，我们便可以开始将本地仓库与远程仓库建立联系：  
> // 与远程建立连接之前需要保证本地仓库为非空，即至少需要一次本地提交
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ echo "# gittest" >> README.md</font>
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git add README.md</font>
> ```nohighlight
> warning: LF will be replaced by CRLF in README.md.
> The file will have its original line endings in your working directory.
> ```
>
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git commit -m "first commit"</font>
> ```nohighlight
> [master (root-commit) 5fe04f8] first commit
>  1 file changed, 1 insertion(+)
>  create mode 100644 README.md
> ```
>
> // 本地有了提交之后，开始与远程的地址建立联系
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ git remote add origin git@github.com:JayHeng/gittest.git</font>
>
> // 确认本地与远程的联系
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git push -u origin master</font>
> ```nohighlight
> The authenticity of host 'github.com (xxx.xx.xxx.xxx)' can't be established.
> RSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
> Are you sure you want to continue connecting (yes/no)? yes
> Warning: Permanently added 'github.com,xxx.xx.xxx.xxx' (RSA) to the list of known hosts.
> git@github.com: Permission denied (publickey).
> fatal: Could not read from remote repository.
>
> Please make sure you have the correct access rights
> and the repository exists.
> ```

　　很不幸，我们似乎没有与远程建立正确的联系，提示“Permission denied”，这是因为github账号没有设置ssh公钥信息所致，需要前往github网站的"account settings"，依次点击"Setting -> SSH and GPG Keys"->"New SSH key"，将本地的rsa key（id_rsa.pub里的字符串）填写进去，下面是生成本地rsa key的方法：  
> // 创建本地rsa key（如果没有的话，一直enter/yes；此处痞子衡已经生成过，故直接用之前的key）
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;" color="Blue">$ ssh-keygen -t rsa</font>
> ```nohighlight
> Generating public/private rsa key pair.
> Enter file in which to save the key (/c/Users/jay/.ssh/id_rsa):
> /c/Users/jay/.ssh/id_rsa already exists.
> Overwrite (y/n)? n
> ```

　　在github网站设置好正确rsa key之后便可以再次尝试与将本地与远程进行连接：  
<img src="http://odox9r8vg.bkt.clouddn.com/image/cnblogs/gittest%20-%20ssh-key.PNG" style="zoom:70%" />  

> // 再试一次确认本地与远程的联系
> jay@pc MINGW64 /d/my_project/gittest (master)
> <font style="font-weight:bold;">$ git push -u origin master</font>
> ```nohighlight
> Warning: Permanently added the RSA host key for IP address 'xxx.xx.xxx.xxx' to the list of known hosts.
> Counting objects: 3, done.
> Writing objects: 100% (3/3), 213 bytes | 213.00 KiB/s, done.
> Total 3 (delta 0), reused 0 (delta 0)
> To github.com:JayHeng/gittest.git
>  * [new branch]      master -> master
> Branch 'master' set up to track remote branch 'master' from 'origin'.
> ```

　　好了，大功告成，此时我们已经成功将本地与远程建立了联系，本地分支叫master，对应的远程分支是origin。  

#### 2.克隆远程仓库到本地git clone
　　Git是可以远程协作的，这意味着任何人建立的共享远程仓库都可以被复制到任何机器上，只需要知道远程仓库地址即可。  
> // 将远程repo克隆到本地
> jay@pc MINGW64 /d/my_project
> <font style="font-weight:bold;" color="Blue">$ git clone git@github.com:JayHeng/gittest.git</font>
> ```nohighlight
> Cloning into 'gittest'...
> remote: Counting objects: 3, done.
> remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0
> Receiving objects: 100% (3/3), done.
> ```

　　有了远程共享，再也不用担心本地仓库丢失了，想clone就clone，so easy！  


