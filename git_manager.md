# git 项目管理

[参考资料](https://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)

## 简介

```shell
    1.origin
        origin指向的就是你本地的代码库托管在Github上的版本,origin就是一个名字,
        它是在你clone一个托管在Github上代码库时，git为你默认创建的指向这个 远程代码库 的标签
        
        (1) git为你默认创建了一个指向远端代码库的origin（因为你是从这个地址clone下来的）
        >  git remote -v
          origin  https://github.com/ztxgithub/git_time.git (fetch)
          origin  https://github.com/ztxgithub/git_time.git (push)
                  
        假设现在有一个用户user2 fork你的repository,那么他的代码库链接就是这个样子https://github.com/user2/repository
        如果他也照着这个clone，然后在他的终端上里输入git remote -v,他会看的的就是
        origin https://github.com/user2/repository.git (fetch)
        origin https://github.com/user2/repository.git (push)
        可以看到origin指向的位置是user2的的远程代码库,如果user2想加一个远程指向你的代码库，他可以在终端上
        输入git remote add upstream(可以定义其他名字) https://github.com/user1/repository.git
        然后再输入一遍 git remote -v,输出结果就会变为
        origin https://github.com/user2/repository.git (fetch)
        origin https://github.com/user2/repository.git (push)
        upstream https://github.com/user1/repository.git (push)
        upstream https://github.com/user1/repository.git (push)
        增加了指向user1代码库的upstream，也就是之前对指向位置的命名
        
        (2) 在clone完成之后,Git 会自动为你将此 远程仓库 命名为origin(origin只相当于一个别名，
            运行git remote –v或者查看.git/config可以看到origin的含义）,并下载其中所有的数据，建立一个指向它的master 分支的指针,
            我们用(远程仓库名)/(分支名) 这样的形式表示远程分支,所以origin/master指向的是一个remote branch
            (从那个branch我们clone数据到本地),但你无法在本地更改其数据,同时，Git 会建立一个属于你自己的本地master 分支,
            它指向的是你刚刚从remote server传到你本地的副本。随着你不断的改动文件，
            git add, git commit，master的指向会自动移动，你也可以通过merge（fast forward）来移动master的指向。
```

## 创建 git repository

```shell
    1.> echo "# test_git_manager" >> README.md
    2.> git init
    3.> git add README.md
    4.> git commit -m "first commit"
    5.> git remote add origin https://github.com/ztxgithub/test_git_manager.git
    
    6.> git push -u origin master
    
    注意:
        master其实是一个“refspec”，正常的“refspec”的形式为”+<src>:<dst>”，冒号前表示local branch的名字,
        冒号后表示remote repository下 branch的名字.如果你省略了<dst>,git就认为你想push到remote repository下和
        local branch相同名字的branch. 
        push是怎么个push法,就是把本地branch指向的 commit 和 push到remote repository下的branch，比如
        (1) > git push origin master:master 
                  在local repository中找到名字为master的branch,使用它去更新remote repository下名字为master的branch,
                  如果remote repository下不存在名字是master的branch，那么新建一个
                  
        (2) > git push origin master （省略了<dst>，等价于“git push origin master:master”）
        (3) > git push origin master:refs/for/mybranch 
                    在local repository中找到名字为master的branch，用他去更新remote repository下面名字为mybranch的branch)
        (4) > git push origin HEAD:refs/for/mybranch 
                HEAD指向当前工作的branch，master不一定指向当前工作的branch，所以我觉得用HEAD还比master好些
                
        (5) >　git push origin :mybranch 
                在origin repository里面查找mybranch，删除它。用一个空的去更新它，就相当于删除了
```

## 建立新的分支

```shell
    1.新建并切换到该分支(进行功能开发)
        > git checkout -b 分支名
        例如:
            git checkout -b iss53
    
        这时只要对iss53进行任何操作,都不会影响到生产环境的代码
    2.切换到Master分支
        现在你就接到了那个网站问题的紧急电话,需要马上修补.有了 Git ,我们就不需要同时发布这个补丁和 iss53 里作出的修改,
        唯一需要的仅仅是切换回 master 分支。
        不过在此之前,留心你的暂存区或者工作目录里,那些还没有提交的修改,它会和你即将检出的分支产生冲突从而阻止 Git 为你切换分支。
        切换分支的时候最好保持一个清洁的工作区域.稍后会介绍几个绕过这种问题的办法（分别叫做 stashing 和 commit amending）.
        目前已经提交了所有的修改，所以接下来可以正常转换到 master 分支：
        
        > git checkout master
        
    3. 在线上的master之后创建 hotfix分支进行问题修复
        > git checkout -b hotfix
        经过一些列的操作
        
    4.修复完后,再进行提交(留心你的暂存区或者工作目录里,那些还没有提交的修改,它会和你即将检出的分支产生冲突从而阻止 Git 为你切换分支。
                         切换分支的时候最好保持一个清洁的工作区域)
        > git commit -m "message"
    5.再切换到master分支并把它合并进来,用 git merge 命令来进行合并
        > git checkout master
        > git merge hotfix
        结果:
            Updating 11b78b2..3cf9d44
            Fast-forward
            README.md | 2 ++
            1 file changed, 2 insertions(+)
            
         由于当前 master 分支所在的提交对象是要并入的 hotfix 分支的直接上游,Git 只需把 master 分支指针直接右移。
         换句话说，如果顺着一个分支走下去可以到达另一个分支的话，那么 Git 在合并两者时，只会简单地把指针右移，
         因为这种单线的历史分支不存在任何需要解决的分歧，所以这种合并过程可以称为快进（Fast forward）
         
    6.这个时候 hotfix 分支就可以删除了
        >  git branch -d hotfix
    7.如果将iss53合并到master分支时,遇到了冲突
            (1) 通过 > git status 进行查看
                    任何包含未解决冲突的文件都会以未合并（unmerged）的状态列出,Git会在有冲突的文件里加入标准的冲突解决标记,
                    可以通过它们来手工定位并解决这些冲突.可以看到此文件包含类似下面这样的部分
                    
                        <<<<<<< HEAD
                        <div id="footer">contact : email.support@github.com</div>
                        =======
                        <div id="footer">
                          please contact us at support@github.com
                        </div>
                        >>>>>>> iss53
                        
                    可以看到 ======= 隔开的上半部分,是 HEAD（即 master 分支,在运行 merge 命令时所切换到的分支）中的内容,
                    下半部分是在 iss53 分支中的内容。解决冲突的办法无非是二者选其一或者由你亲自整合到一起。
                    比如你可以通过把这段内容替换为下面这样来解决：
                    
                    <div id="footer">
                    please contact us at email.support@github.com
                    </div>
                    
                    这个解决方案各采纳了两个分支中的一部分内容，而且我还删除了 <<<<<<<，======= 和 >>>>>>> 这些行。
                    在解决了所有文件里的所有冲突后,运行 git add 将把它们标记为已解决状态
                    （译注：实际上就是来一次快照保存到暂存区域。）。因为一旦暂存,就表示冲突已经解决。
                    如果你想用一个有图形界面的工具来解决这些问题，不妨运行 git mergetool，
                    它会调用一个可视化的合并工具并引导你解决所有冲突：
            (2) 解决完冲突后再运行一次 git status 来确认所有冲突都已解决
            (3) 用 git commit 来完成这次合并

```

## tag操作

```shell
    1.tag就像里程碑标志的一个点，branch是一个新的征程的一条线；
      tag 是静态的，而branch要往前走；稳定版本备份用tag，新功能开发多人用branch，开发完之后再merge到master上。
      tag是一个只读的branch。
      
    2.打tag git命令
        (1) > git tag  //列出git中现有的所有标签
        (2) > git tag -l v1.4.2.*  //按照字母表顺序给出tag
                v1.4.2.1
                v1.4.2.2

        (3) > git tag -a v1.4 -m ‘version 1.4′  //创建标签， -a 加标签，-m  加标签注释。
        (4) > git tag v1.4-lw                         //创建轻量级标签，不用-a，-m等参数
        (5) > git show v1.4   //git show 命令查看相应标签的版本信息，并连同显示打标签时的提交对象
        (6) > git tag -a v1.2 9fceb02        //为已提交的信息贴上标签，为校验码为9fceb02*的版本贴上标签。
        (7) > git push --tags  //如果上传到服务器时不能上传tag，可加上--tags命令

```

## git 相关命令

```shell
    1. 远程仓库相关命令
           (1) 检出仓库：> git clone git://github.com/jquery/jquery.git
           (2) 查看远程仓库：> git remote -v
           (3) 添加远程仓库：> git remote add [name] [url]
           (4) 删除远程仓库：> git remote rm [name]
           (5) 修改远程仓库：> git remote set-url --push[name][newUrl]
           (6) 拉取远程仓库：> git pull [remoteName] [localBranchName]
           (7)推送远程仓库：$ git push [remoteName] [localBranchName]
           
    2. 分支(branch)操作相关命令
            (1) 查看本地分支：> git branch
            (2) 查看远程分支：> git branch -r
            (3) 创建本地分支：> git branch [name] ----注意新分支创建后不会自动切换为当前分支
            (4) 切换分支：> git checkout [name]
            (5) 创建新分支并立即切换到新分支：> git checkout -b [name]
            (6) 删除分支：> git branch -d [name] ---- -d选项只能删除已经参与了合并的分支，
                                                        对于未有合并的分支是无法删除的。
                                                        如果想强制删除一个分支，可以使用-D选项
            (7) 合并分支：> git merge [name] ----将名称为[name]的分支与当前分支合并
            (8) 创建远程分支(本地分支push到远程)：> git push origin [name]
            (9) 删除远程分支：> git push origin :[name]
                    例如:
                        > git push origin :iss53_1  (但本地的iss53_1分支没有被删除)
            (10) 查看本地和远程的所有分支
                            > git branch -a
                            结果:
                                  develop_smonitor
                                * master                    (* 代表目前所处在的分支)
                                  remotes/origin/e1_merge
                                  remotes/origin/master
                                  
            (11) 提交本地test分支作为远程的master分支 
                        > git push origin test:master         // 
                        $ git push origin test:test              // 提交本地test分支作为远程的test分支
                                  
            我从master分支创建了一个issue5560分支,做了一些修改后,使用git push origin master提交,
            但是显示的结果却是'Everything up-to-date',发生问题的原因是git push origin master 
            在没有track远程分支的本地分支中默认提交的master分支,因为master分支默认指向了origin master 分支,
            这里要使用git push origin issue5560：master 就可以把issue5560推送到远程的master分支了。
           
     3.查看git分支的状态
            > git status;
```

## 注意

```shell
    1.在进行切换分支时，一定要先 commit
    2.在进行　pull 时，也一定要进行 commit
```