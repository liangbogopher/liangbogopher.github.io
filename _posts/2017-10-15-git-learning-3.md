---
layout: post
keywords: Git Flow
title: Git 简单指南（三）- Git Flow
date: 2017-10-15 22:55:18
categories: Git
tags: Git
---

了解Git高级特性 - Git Flow。
### git-flow 的工作流程
当在团队开发中使用版本控制系统时，商定一个统一的工作流程是至关重要的。如果在你的团队中还没有能形成一个特定有效的工作流程，那么混乱就将是不可避免的。

大家工作在同一个仓库上，那么彼此的代码协作必然带来很多问题和挑战，如下：

1. 如何开始一个Feature的开发，而不影响别的Feature？
2. 由于很容易创建新分支，分支多了如何管理，时间久了，如何知道每个分支是干什么的？
哪些分支已经合并回了主干？
3. 如何进行Release的管理？开始一个Release的时候如何冻结Feature, 如何在Prepare Release的时候，开发人员可以继续开发新的功能？
4. 线上代码出Bug了，如何快速修复？而且修复的代码要包含到开发人员的分支以及下一个Release?

在这我们将一起学习一个当前非常流行的工作流程 git-flow。

<!-- more -->
### 什么是 git-flow？
git-flow 并不是要替代 Git，它仅仅是非常聪明有效地把标准的 Git 命令用脚本组合了起来。

严格来讲，你并不需要安装什么特别的东西就可以使用 git-flow 工作流程。你只需要了解，哪些工作流程是由哪些单独的任务所组成的，并且附带上正确的参数，以及在一个正确的顺序下简单执行那些对应的 Git 命令就可以了。当然，如果你使用 git-flow 脚本就会更加方便了，你就不需要把这些命令和顺序都记在脑子里。

Git Flow流程，借用这张在网上广为流传的图了
<img src="https://pic4.zhimg.com/v2-4bf7e68c49e29c353a01bd6b782a1be3_b.png" />

这个5个分支大体可以分为两类:
 
 - 主要分支
```
//  主分支，永远保持稳定的状态，切下来就能跑
master   
//  最新的开发状态       
develop
```

 - 辅助分支
```
//  功能分支，基于develop，开发新功能时从develop新建分支开发，完成后合并回develop
feature  
//  准备发布的状态，用来预发、修复bugs，基于 develop, 完成后 merge 回 develop 和 master        
release  
// 修复 master 上的问题, 等不及 release 版本就必须马上上线. 基于 master, 完成后 merge 回        
hotfix
```

详细说明：

初始分支：
1）master 只能用来包括产品代码。你不能直接工作在这个 master 分支上，而是在其他指定的，独立的特性分支中（这方面我们会马上谈到）。不直接提交改动到 master 分支上也是很多工作流程的一个共同的规则。

2）develop 是你进行任何新的开发的基础分支。当你开始一个新的功能分支时，它将是_开发_的基础。另外，该分支也汇集所有已经完成的功能，并等待被整合到 master 分支中。

这两个分支会存活在项目的整个生命周期中。而其他的分支，例如针对功能的分支，针对发行的分支，仅仅只是临时存在的。它们是根据需要来创建的，当它们完成了自己的任务之后就会被删除掉。

3）Feature 分支

分支名 feature/*
Feature分支做完后，必须合并回Develop分支, 合并完分支后一般会删点这个Feature分支，但是我们也可以保留。

4）Release分支

分支名 release/*
Release分支基于Develop分支创建，打完Release分之后，我们可以在这个Release分支上测试，修改Bug等。同时，其它开发人员可以基于开发新的Feature (记住：一旦打了Release分支之后不要从Develop分支上合并新的改动到Release分支)

发布Release分支时，合并Release到Master和Develop， 同时在Master分支上打个Tag记住Release版本号，然后可以删除Release分支了。

5）维护分支 Hotfix

分支名 hotfix/*
hotfix分支基于Master分支创建，开发完后需要合并回Master和Develop分支，同时在Master上打一个tag。


### 工具安装
OSX：
```
$ brew install git-flow
```
Debian/Ubuntu Linux：
```
$ apt-get install git-flow
```
Windows(cygwin)：
```
$ wget -q -O - --no-check-certificate https://github.com/nvie/gitflow/raw/develop/contrib/gitflow-installer.sh | bash
```


### Git flow 命令
[文档详情](https://github.com/petervanderdoes/gitflow-avh/wiki#installing-git-flow)

```
// 初始化
$ git flow init

// 新建feature
$ git flow feature start rss-feed
// 完成feature
$ git flow feature finish rss-feed

// 创建 release
$ git flow release start 1.1.5
// 完成 release
$ git flow release finish 1.1.5

// 创建 Hotfixes
$ git flow hotfix start oom-error
// 完成 Hotfixes
$ git flow hotfix finish oom-error

```
其中
1） 完成feature：
这个 “feature finish” 命令会把我们的工作整合到主 “develop” 分支中去，之后，git-flow 也会进行清理操作。它会删除这个当下已经完成的功能分支，并且换到 “develop” 分支。

2）完成Release:
这个命令会完成如下一系列的操作：

1. 首先，git-flow 会拉取远程仓库，以确保目前是最新的版本。
2. 然后，release 的内容会被合并到 “master” 和 “develop” 两个分支中去，这样不仅产品代码为最新的版本，而且新的功能分支也将基于最新代码。
3. 为便于识别和做历史参考，release 提交会被标记上这个 release 的名字（在我们的例子里是 “1.1.5”）。
4. 清理操作，版本分支会被删除，并且回到 “develop”。

3) 完成hotfix：

1. 完成的改动会被合并到 “master” 中，同样也会合并到 “develop” 分支中，这样就可以确保这个错误不会再次出现在下一个 release 中。
2. 这个 hotfix 程序将被标记起来以便于参考。
3. 这个 hotfix 分支将被删除，然后切换到 “master” 分支上去。

<img src="http://images.cnblogs.com/cnblogs_com/cnblogsfans/771108/o_git-flow-commands.png" />

### Git 代码示例
1）创建develop分支
```
git branch develop
git push -u origin develop 
```
2）开始新Feature开发
```
git checkout -b some-feature develop
# Optionally, push branch to origin:
git push -u origin some-feature    

# 做一些改动    
git status
git add some-file
git commit    
```
3）完成Feature
```
git pull origin develop
git checkout develop
git merge --no-ff some-feature
git push origin develop

git branch -d some-feature

# If you pushed branch to origin:
git push origin --delete some-feature  
```
4）开始Relase
```
git checkout -b release-0.1.0 develop

# Optional: Bump version number, commit
# Prepare release, commit
```
5） 完成Release
```
git checkout master
git merge --no-ff release-0.1.0
git push

git checkout develop
git merge --no-ff release-0.1.0
git push

git branch -d release-0.1.0

# If you pushed branch to origin:
git push origin --delete release-0.1.0   


git tag -a v0.1.0 master
git push --tags
```
6）开始Hotfix
```
git checkout -b hotfix-0.1.1 master  
```
7）完成Hotfix
```
git checkout master
git merge --no-ff hotfix-0.1.1
git push


git checkout develop
git merge --no-ff hotfix-0.1.1
git push

git branch -d hotfix-0.1.1

git tag -a v0.1.1 master
git push --tags
```

### Git flow GUI 工具

#### SourceTree

