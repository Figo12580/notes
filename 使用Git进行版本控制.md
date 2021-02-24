### 使用Git进行版本控制

#### Git介绍

Git是由Linus（同时也是Linux的开发者）开发的一套分布式版本管理系统。

三个工作区域：

- 工作区：指电脑可以直接访问到的文件区域
- 暂存区(stage)：修改暂存的地方
- 分支区域：版本保存的地方

> Git管理的是修改，而不是文件内容本身。每次commit以后，暂存区里的所有修改都会被提交到分支。

#### 基本命令

```cmd
git init <fold_name> #初始化文件夹，使文件夹成为仓库，所有git操作都要在repo内进行

#提交版本
git add <file> #在git暂存区中添加一个文件
git commit -m "<text>" #提交暂存区，并添加说明

#检查版本
git status #检查工作区有无变化（是否与最新版本一致）
git diff #查看工作区的变化
git log #查看版本变化日志和版本号(commit id)
git reflog #查看历史操作，同时也可能可以查看版本号

#版本跳转，结合log可以回到过去版本以及”未来版本“
git reset --hard <版本号>  #可以输入版本号的前几位，git会自动检索
git reset head^  #head是一个指针，代表当前版本，head^是当前的上一个版本，以此类推

#撤销修改
git checkout -- <filename> #撤销文件在工作区的所有修改，使文件回到暂存区的状态（如果已经commit，暂存区为空，就回到当前版本的状态）；也就是回到上一次add或者commit后的状态

git reset HEAD <filename> #丢弃暂存区修改。例如，在工作区做出错误修改以后，不小心把它暂存了，用这个命令可以使暂存区的错误消失；然后再使用上一个命令就可以清除错误


#删除文件
git rm <filename> #与git add相对，可以在repo的这个版本中删除这个文件，要配合git commit使用；注意这个删除也是一次修改，在历史版本中可以看到未被删除的版本；想对地，对于版本误删的文件，可以使用撤销修改的命令返回这个文件未被删除的状态
```

#### 远程仓库与GitHub

Git作为分布式系统，其一大特征就是远程控制，可以由多台设备平行地共享仓库。一般我们会用GitHub的服务器资源作为远程仓库网络的节点（当然也可以自己搭建咯）。使用GitHub远程仓库并同步本地内容的方法：

- 在自己的GitHub账号里添加设备的ssh key (方法见[这里](https://www.liaoxuefeng.com/wiki/896043488029600/896954117292416))。

- 在GitHub上创建空的远程仓库

  ```cmd
  git remote -v #查看远程仓库信息
  ```

- 在本地terminal上，远程访问github并添加一个远程库origin：

  ```cmd
  git remote add origin git@github.com:<username>/learngit.git
  ```

- 把本地库内容推到远程库上：

  ```cmd
  git push -u origin <localBranch>:<remoteBranch>  #把当前分支添加到origin的master分支；-u参数创建两个分支的关联关系
  git push origin master #由关联以后，可以使用这个命令将本地的master分支提交给远程master
  ```

- 从远程库创建本地克隆：

  ```cmd
  #先跳转到想要创建克隆的目录下
  git clone git@github.com:<username>/<reponame>.git
  ```

#### 分支管理

Git能够实现版本控制和分支，是因为它是对文件修改的操作。repo可以把每次commit的修改保存下来，而在Git提交修改，实际上是将`HEAD`指针指向的修改推到Repo中，这样就使得**HEAD推出的版本成为”当前版本“**。

Git的分支就是创建一些指针，标记不同的修改。只要HEAD的指向与该指针的指向一致，就说明”当前位于这条分支上“。

```cmd
git switch -c dev #新建dev分支并切换到这条分支上,等价于：
#git branch dev
#git switch dev

#删除分支
git branch -d dev

#查看当前分支
git branch

#合并分支
git merge dev  #将dev分支快速合并到当前分支（master）上
#合并分支很欧可能出现冲突（例如对于同一个文件作出了不同的修改），这时候必须手动调整
git --no-ff dev #在当前分支上创建一个新节点并复制指定分支
git log -graph #查看分支合并图

#推送分支
git push origin master #将本地当前分支的内容提交到远程库的指定分支中 ；开发过程中，dev分支是需要经常同步的

#抓取分支：使用git clone抓取分支，只可以抓到master分支。如果是多人开发，每个人需要在自己本地建立dev分支；当然了，个人还是可以把自己的dev分支提交给仓库（所有者）的远程dev分支
git pull #从远程dev分支中抓取版本到本地的dev分支上
#这需要先建立本地和远程dev分支的联系：git branch --set-upstream-to=origin/dev dev

```

#### [分支策略](http://www.ruanyifeng.com/blog/2012/07/git.html)

#### 多人协作问题

![image-20210219193927459](/Users/zhiwei/Library/Application Support/typora-user-images/image-20210219193927459.png)

