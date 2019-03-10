---
title: Git命令总结
date: 2017-09-21 18:13:00
comments: true
categories: [Git]
tags: [Git, GitHub]
---

最近看了廖雪峰的Git教程讲解的非常棒，这里做一个Git命令的学习总结，供日后查看

<!-- more -->

> 图片来源为廖雪峰的官方网站，在此表示感谢

- [廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
- [Git官网](https://git-scm.com/)

## Git常用命令

|Git command|Comments|
|-|
|git init|将当前目录初始化为Git仓库|
|git status|查看当前仓库状态|
|git add filename|将工作区的文件放到暂存区<br>![git-repo](http://res.cloudinary.com/code-clannad-site/image/upload/v1505987692/0_1_khjtwf.jpg)|
|git commit -m "comment"|将暂存区的文件**提交**到版本库（不在暂存区中的修改是不会被提交的），commet为提交的说明<br>![git-stage-after-commit](http://res.cloudinary.com/code-clannad-site/image/upload/v1505987799/0_1_o1jiyd.jpg)|
|git commit -a -m "comment"|直接将工作区中的文件提交到版本库|
|git log|查看提交记录，内容包括commit id，author，date，commit comment|
|git log --pretty=oneline|查看提交记录（以单行显示commit id和commit comment）|
|git log -1|显示最后一次提交信息|
|git reset --hard HEAD^|回退到上一个提交的版本（`HEAD`指向当前版本，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个^比较容易数不过来，所以写成`HEAD~100`）|
|git reset --hard 3628164|根据commit id回退到指定的提交版本|
|git reflog|查看命令历史|
|git diff filename|比较的是指定文件在**工作区**和**暂存区**之间的差别|
|git diff --cached|比较的是暂存区和版本库的差别|
|git diff HEAD -- filename|查看指定文件在**工作区**和**版本库**中最新版本的区别|
|git checkout -- filename|撤销指定文件在**工作区**中的修改回到和**版本库**中一模一样的状态（前提是文件还没有add到暂存区）|
|git reset HEAD filename|将已经add到暂存区的指定文件回退到工作区（之后如果在调用`git checkout -- filename`即可丢弃指定文件在工作区的修改）|
|git rm filename|从版本库中删除指定文件|
|git rm filename --cached|删除暂存区中的指定文件|
|git checkout -- filename|用版本库中的指定文件替换工作区中的指定文件（通过该命令无论工作区中是修改还是删除都可以“一键还原”）|
|git remote add origin git@github.com:1976841230/LearnGit.git|让本地仓库和远程仓库产生关联（远程仓库的名字默认就叫做`origin`，可以更改）|
|git remote add reponsitoryname|添加远程仓库|
|git fetch reponsitoryname branchname|从repositoryname仓库拉取master分支的代码到本地**repositoryname/branchname**分支，不会自动和本地其他分支合并,如要合并到本地master分支，需执行git merge repositoryname/branchname|
|git pull reponsitoryname master|从远程获取最新版本并merge到本地|
|git push -u origin master|把当前分支master推送到远程仓库（由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以省略-u参数）|
|git push origin master|将本地master分支的最新提交推送至远程仓库|
|git push origin branchname|将指定分支的所有本地提交推送到远程仓库对应的远程分支上|
|git clone git@github.com:michaelliao/gitskills.git|克隆远程仓库到本地（地址可以使用`https`协议也可以使用`ssh`协议，实际上https速度慢，而且每次都必须输入口令）|
|git branch|列出所有分支|
|git branch branchname|创建分支|
|git checkout -b dev|新建并切换到dev分支<br>![git-br-create](http://res.cloudinary.com/code-clannad-site/image/upload/v1505987812/0_1_d9btvd.png)|
|git checkout -b branch-name origin/branch-name|在本地创建和远程分支对应的分支|
|git checkout master|切换回master分支<br>![git-br-on-master](http://res.cloudinary.com/code-clannad-site/image/upload/v1505987822/0_1_bzuixm.png)|
|git merge dev|合并指定分支到当前分支（此处假设当前分支为master分支）<br>![git-br-ff-merge](http://res.cloudinary.com/code-clannad-site/image/upload/v1505987836/0_1_lgsqqv.png)|
|git branch -d dev|删除指定分支<br>![git-br-rm](http://res.cloudinary.com/code-clannad-site/image/upload/v1505987852/0_1_mtjqes.png)|
|git branch -D feature|强制删除指定分支，对于没有合并的分支使用`git branch -d branchname`无法删除，可以使用此命令强制删除|
|git log --graph|查看分支合并图|
|git log --graph --pretty=oneline --abbrev-commit|查看分支的合并情况|
|git merge --no-ff -m "merge with no-ff" dev|禁用`Fast forward`模式的分支合并<br>![git-no-ff-mode](http://res.cloudinary.com/code-clannad-site/image/upload/v1505987865/0_1_ntgxaw.png)|
|git stash|将工作区储存起来，此时再用`git status`查看工作区则是干净的|
|git stash list|查看储存的工作区列表|
|git stash apply|恢复工作区，此时stash中储存的工作区并不会删除|
|git stash drop|删除stash中存储的工作区|
|git stash pop|恢复工作区后将stash中的工作区删除<=>git stash apply+git stash drop|
|git stash apply stash@{0}|恢复指定编号的工作区（通过git stash list查看编号）|
|git remote|列出所有的远程仓库的名字（自己本地仓库对应的远程仓库默认名字为origin，所以至少会有这一个名字）|
|git remote -v(或--verbose)|列出所有远程仓库的名字及URL|
|git checkout -b dev origin/dev|创建远程的dev分支到本地（远程分支的名字和本地分支名称可以不同，但最好保持一致）|
|git branch --set-upstream dev origin/dev|设置本地dev分支与远程origin/dev分支的链接|
|git tag|查看所有标签|
|git tag v1.0|为当前分支的最新提交打上v1.0的标签（默认标签是打在当前分支的最新commit上）|
|git tag v0.9 6224832|为指定的commit打标签（commit id通过`git log --pretty=oneline --abbrev-commit`查看）|
|git show v0.9|查看指定标签的信息|
|git tag -a v0.1 -m "version 0.1 released" 6224832|-a指定标签名，-m指定标签说明，通过`git show v0.1`可以看到标签的说明|
|git tag -d v0.1|删除本地指定标签|
|git push origin :refs/tags/v0.1|删除远程指定标签|
|git push origin tagname|推送指定标签到远程仓库|
|git push origin --tags|推送所有未推送到远程的标签到远程仓库|
|git config --global user.name "Your Name"|`--global`参数表示这台机器上的所有的Git仓库都会使用这个配置|
|git config --global user.email "email@example.com"||
|git config --global color.ui true|让git命令行显示颜色|

## Git命令别名设置

|Git command|Comment|
|-|
|git config --global alias.st status|为status指定别名为st，输入git st将和git status效果相同。其中`--global`参数针对当前用户的所有仓库都起作用，省略该参数则只针对当前仓库起作用，当前仓库的Git配置文件在.git/config中可以查看到已经配置的别名；当前用户的Git配置文件在/user/.gitconfig文件中|
|git config --global alias.co checkout|Git默认已经设置了该别名|
|git config --global alias.ci commit|Git默认已经设置了该别名|
|git config --global alias.br branch|Git默认已经设置了该别名|
|git config --global alias.unstage 'reset HEAD'|注意对多个单词的命令必须用单引号括起来,对此例来说如果reset HEAD没有加引号，则别名unstage对应的是reset而不是reset HEAD|
|git config --global alias.last 'log -1'|查看最新的提交|

## 分支管理策略

![git-br-policy](http://res.cloudinary.com/code-clannad-site/image/upload/v1505987880/0_1_kc5log.png)
