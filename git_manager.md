# git 项目管理

[参考资料](https://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)

## 创建 git repository

```shell
    1.> echo "# test_git_manager" >> README.md
    2.> git init
    3.> git add README.md
    4.> git commit -m "first commit"
    5.> git remote add origin https://github.com/ztxgithub/test_git_manager.git
    6.> git push -u origin master

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
           
            

            
            

```