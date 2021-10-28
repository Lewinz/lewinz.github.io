---
layout: wiki
title: git 命令记录
categories: [git]
description: git 命令记录
keywords: git
---

# 基础命令

[参考博客](https://www.cnblogs.com/suzhen-2012/p/7881825.html)

初始化仓库  
git init

查看本地修改状态  
git status

查看本地修改内容  
git diff readme.txt

添加暂存  
git add  

提交本地仓库  
git commit -m "注释"

查看本地修改记录（简洁模式/已回退版本看不到）  
git log –pretty=oneline

查看分支所有操作记录（已回退版本也可看到）  
git reflog

恢复到某分支（分支号使用 git reflog 查看）  
git reset  –hard 分支号

回退本地仓库版本（已 commit 未 push） 
```
# 回退上一次提交
git reset  –hard HEAD^

# 回退两次提交
git reset  –hard HEAD^^

# 回退 100 次提交
git reset  –hard HEAD~100
```

放弃工作区修改（如果当前文件有修改在暂存区，则退回到暂存区版本）  
git checkout  — file

本地 git 仓库关联远程仓库  
git remote add origin https://github.com/tugenhua0707/testgit.git

提交远程仓库 -u 可省略  
git push (-u origin master)

clone 代码  
git clone -b master https://github.com/tugenhua0707/testgit.git

本地创建并切换分支  
git checkout -b dev  
等同于  
git branch dev  
git checkout dev  

查看当前分支（列出所有分支，当前分支前*号标识）  
git branch

合并分支（在当前分支合并 dev 分支，快进模式）  
git merge dev

删除分支  
git branch -d dev

隐藏工作现场  
git stash

查看工作现场列表  
git stash list

恢复工作现场，同时删除隐藏的工作现场  
git stash pop


```
   mkdir：         XX (创建一个空目录 XX 指目录名)

   pwd：          显示当前目录的路径。

   git init          把当前的目录变成可以管理的 git 仓库，生成隐藏.git 文件。

   git add XX       把 xx 文件添加到暂存区去。

   git commit –m “XX”  提交文件 –m 后面的是注释。

   git status        查看仓库状态

   git diff  XX      查看 XX 文件修改了那些内容

   git log          查看历史记录

   git reset  –hard HEAD^ 或者 git reset  –hard HEAD~ 回退到上一个版本

                        (如果想回退到 100 个版本，使用 git reset –hard HEAD~100 )

   cat XX         查看 XX 文件内容

   git reflog       查看历史记录的版本号 id

   git checkout — XX  把 XX 文件在工作区的修改全部撤销。

   git rm XX          删除 XX 文件

   git remote add origin https://github.com/tugenhua0707/testgit 关联一个远程库

   git push –u(第一次要用-u 以后不需要) origin master 把当前 master 分支推送到远程库

   git clone https://github.com/tugenhua0707/testgit  从远程库中克隆

   git checkout –b dev  创建 dev 分支 并切换到 dev 分支上

   git branch  查看当前所有的分支

   git checkout master 切换回 master 分支

   git merge dev    在当前的分支上合并 dev 分支

   git branch –d dev 删除 dev 分支

   git branch name  创建分支

   git stash 把当前的工作隐藏起来 等以后恢复现场后继续工作

   git stash list 查看所有被隐藏的文件列表

   git stash apply 恢复被隐藏的文件，但是内容不删除

   git stash drop 删除文件

   git stash pop 恢复文件的同时 也删除文件

   git remote 查看远程库的信息

   git remote –v 查看远程库的详细信息

   git push origin master  Git 会把 master 分支推送到远程库对应的远程分支上
```

# Tag

创建 tag（tag 名称：V1.2 附注信息：release 1.2）  
git tag -a V1.2 -m 'release 1.2'

查看 tag  
git tag

查看 tag 附注信息  
git show V1.2

提交本地 tag 至远程仓库
git push origin --tags

删除本地 tag  
git tag -d V1.2

删除远程 tag （推送本地空的 tag 至远程仓库，达到删除效果）  
git push origin :refs/tags/V1.2

获取远程 tag（多用于运维部署）  
git fetch origin tag V1.2

# GitHub 分支操作
* 查看本地分支 `git branch`
* 查看远程分支 `git branch -r`
* 查看所有分支 `git branch -a`
* 本地创建新的分支 `git branch branch_name`
* 切换到新的分支 `git checkout branch_name`
* 创建+切换分支 `git checkout -b branch_name`
* 将新分支推送到 github `git push origin branch_name`
* 删除本地分支 `git branch -d branch_name`
* 删除 github 远程分支 `git push origin :branch_name` PS:删除远程分支，必须先调整仓库 default 分支，否则无法删除

# GitHub 项目 Fork 后保持更新
step 1、执行命令 `git remote -v` 查看你的远程仓库的路径：

origin  https://github.com/Lewinz/lewinz.github.io (fetch)  
origin  https://github.com/Lewinz/lewinz.github.io (push)  
如果只有上面 2 行，说明你未设置 upstream （中文叫：上游代码库）。一般情况下，设置好一次 upstream 后就无需重复设置。

step 2、执行命令 `git remote add upstream https://github.com/selfteaching/the-craft-of-selfteaching.git` 把 xiaolai 的仓库设置为你的 upstream 。这个命令执行后，没有任何返回信息；所以再次执行命令 `git remote -v` 检查是否成功。

step 3、执行命令 `git status` 检查本地是否有未提交的修改。如果有，则把你本地的有效修改，先从本地仓库推送到你的 github 仓库。最后再执行一次 `git status` 检查本地已无未提交的修改。

step 4、执行命令 `git fetch upstream develop` 抓取 xiaolai 原仓库的更新

step 5、执行命令 `git merge upstream/develop` 合并远程的 master 分支  
这步相当于在当前所在分支合并 upstream 上游仓库的 develop 分支

step 6、执行命令 `git push` 把本地仓库向 github 仓库（你 fork 到自己名下的仓库）推送修改

# 撤销 git push 代码
``` shell
# 将本地仓库版本号指向上一 commit
git reset --soft HEAD~1

# 修改代码，后正常提交
git add xxxx
git commit -m "xxxx"

# 使用 -f 或者 --force 强制 push
git push origin master -f
```

# 本地新建分支推送至远程仓库
``` shell
# 本地新建分支 test 并切换到 test 分支
git checkout -b test

# 与远程分支关联，origin 为本地分支别名
git remote add origin https://github.com/yangxiaoyan20/BowlingScore.git

# 正常提交流程
git add .
git commit -m "xxxx"

# 将代码推送至远程仓库
git push origin test
```