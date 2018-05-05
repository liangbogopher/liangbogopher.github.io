---
layout: post
keywords: Git
title: Git 简单指南（一）- Git基础
date: 2017-10-14
categories: Git
tags: Git
---

简单介绍一下Git的历史以及基础。
### 关于版本控制
什么是“版本控制”？版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。 你可以对任何类型的文件进行版本控制。

使用版本控制系统通常还意味着，就算你乱来一气把整个项目中的文件改的改删的删，你也照样可以轻松恢复到原先的样子。 但额外增加的工作量却微乎其微。
<!-- more -->
#### 本地版本控制系统
许多种本地版本控制系统，大多都是采用某种简单的数据库来记录文件的历次更新差异。如下：
<img src="https://git-scm.com/book/en/v2/images/local.png" />

#### 集中化的版本控制系统
接下来人们又遇到一个问题，如何让在不同系统上的开发者协同工作？ 于是，集中化的版本控制系统（Centralized Version Control Systems，简称 CVCS）应运而生。 这类系统，诸如 CVS、Subversion 以及 Perforce 等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新。多年以来，这已成为版本控制系统的标准做法。

<img src="https://git-scm.com/book/en/v2/images/centralized.png" />

这种做法带来了许多好处，特别是相较于老式的本地 VCS 来说。 现在，每个人都可以在一定程度上看到项目中的其他人正在做些什么。 而管理员也可以轻松掌控每个开发者的权限，并且管理一个 CVCS 要远比在各个客户端上维护本地数据库来得轻松容易。

事分两面，有好有坏。 这么做最显而易见的缺点是中央服务器的单点故障。 如果宕机一小时，那么在这一小时内，谁都无法提交更新，也就无法协同工作。 如果中心数据库所在的磁盘发生损坏，又没有做恰当备份，毫无疑问你将丢失所有数据——包括项目的整个变更历史，只剩下人们在各自机器上保留的单独快照。 本地版本控制系统也存在类似问题，只要整个项目的历史记录被保存在单一位置，就有丢失所有历史更新记录的风险。

#### 分布式版本控制系统
于是分布式版本控制系统（Distributed Version Control System，简称 DVCS）面世了。 在这类系统中，像 Git、Mercurial、Bazaar 以及 Darcs 等，客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。 这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。 因为每一次的克隆操作，实际上都是一次对代码仓库的完整备份。

<img src="https://git-scm.com/book/en/v2/images/distributed.png" />


### Git的诞生
Linux 内核开源项目，在2002 年，整个项目组开始启用一个专有的分布式版本控制系统 BitKeeper 来管理和维护代码。到了 2005 年，开发 BitKeeper 的商业公司同 Linux 内核开源社区的合作关系结束，他们收回了 Linux 内核社区免费使用 BitKeeper 的权力。 这就迫使 Linux 开源社区（特别是 Linux 的缔造者 Linus Torvalds）基于使用 BitKeeper 时的经验教训，开发出自己的版本系统。 他们对新的系统制订了若干目标：
 
 - 速度

 - 简单的设计

 - 对非线性开发模式的强力支持（允许成千上万个并行开发的分支）

 - 完全分布式

 - 有能力高效管理类似 Linux 内核一样的超大规模项目（速度和数据量）

自诞生于 2005 年以来，Git 日臻成熟完善，在高度易用的同时，仍然保留着初期设定的目标。 它的速度飞快，极其适合管理大项目，有着令人难以置信的非线性分支管理系统

### Git安装

Linux 安装：
```
sudo yum install git
```
在 Mac 上安装:
建议使用brew安装。百度搜索brew官网
```
brew install git
```
在 Windows 上安装:
直接去 Git 官方网站下载。

如果安装好了，可以使用如下命令查看：
```
macbook:hexo liangbo$ git --version
git version 2.11.0 (Apple Git-81)
```

### Git基础
Git 究竟是怎样的一个系统呢？Git 在保存和对待各种信息的时候与其它版本控制系统有很大差异，尽管操作起来的命令形式非常相近，理解这些差异将有助于防止你使用中的困惑。

#### 直接记录快照，而非差异比较
这类系统（CVS、Subversion、Perforce、Bazaar 等等）将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异：
<img src="https://git-scm.com/book/en/v2/images/deltas.png" />

Git 不按照以上方式对待或保存数据。 反之，Git 更像是把数据看作是对小型文件系统的一组快照。 每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 快照流。
<img src="https://git-scm.com/book/en/v2/images/snapshots.png" />

#### 近乎所有操作都是本地执行
因为你在本地磁盘上就有项目的完整历史，所以大部分操作看起来瞬间完成。这也意味着你离线或者没有 VPN 时，几乎可以进行任何操作。直到有网络连接时再上传.

#### Git 保证完整性
Git 中所有数据在存储前都计算校验和，然后以校验和来引用。用以计算校验和的机制叫做 SHA-1 散列（hash，哈希）。 这是一个由 40 个十六进制字符（0-9 和 a-f）组成字符串，基于 Git 中文件的内容或目录结构计算出来。 SHA-1 哈希看起来是这样：
```
24b9da6552252987aa493b52f8696cd6d3b00373
```

#### Git 一般只添加数据
你执行的 Git 操作，几乎只往 Git 数据库中增加数据。 很难让 Git 执行任何不可逆操作，或者让它以任何方式清除数据。一旦你提交快照到 Git 中，就难以再丢失数据，

#### 三种状态
Git 有三种状态，你的文件可能处于其中之一：已提交（committed）、已修改（modified）和已暂存（staged）。 
已提交表示数据已经安全的保存在本地数据库中。 
已修改表示修改了文件，但还没保存到数据库中。 
已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

 Git 项目的三个工作区域的概念：Git 仓库、工作目录以及暂存区域。
 <img src="https://git-scm.com/book/en/v2/images/areas.png" />

Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据。

工作目录是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。

暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。 有时候也被称作`‘索引’'，不过一般说法还是叫暂存区域。

基本的 Git 工作流程如下：

1. 在工作目录中修改文件。

2. 暂存文件，将文件的快照放入暂存区域。

3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。

### 运行Git前的一些配置
#### 用户信息
```
$ git config --global user.name "liangbo"
$ git config --global user.email "liangbo@codoon.com"
```
再次强调，如果使用了 --global 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情， Git 都会使用那些信息。

#### 检查配置信息
```
macbook:hexo liangbo$ git config --list
user.name=liangbo
user.email=liangbo@codoon.com
http.postbuffer=524288000
core.autocrlf=input
mergetool.sourcetree.trustexitcode=true
commit.template=/Users/liangbo/.stCommitMsg
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
remote.origin.url=git@github.com:amuguelove/hexo.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.master.remote=origin
branch.master.merge=refs/heads/master
color.ui=true
...
```

你可以通过输入 git config <key>： 来检查 Git 的某一项配置
```
macbook:hexo liangbo$ git config user.name
liangbo
```

### 获取Git仓库
#### 在现有目录中初始化仓库
```
$ git init
```
该命令将创建一个名为 .git 的子目录，但是你的项目里的文件还没有被跟踪。你可通过 git add 命令来实现对指定文件的跟踪，然后执行 git commit 提交：
```
$ git add .			// 添加当前目录下所有文件夹和文件
$ git add README		// 添加指定文件
$ git commit -m 'initial project version'
```

#### 克隆现有的仓库
可以用下面的命令：
```
$ git clone https://github.com/libgit2/libgit2
```

如果你想在克隆远程仓库的时候，自定义本地仓库的名字，你可以使用如下命令：
```
$ git clone https://github.com/libgit2/libgit2 mylibgit
```
Git 支持多种数据传输协议。 上面的例子使用的是 https:// 协议，不过你也可以使用 git:// 协议或者使用 SSH 传输协议，比如 user@server:path/to/repo.git 。

### Git仓库如何纪录更新

请记住，你工作目录下的每一个文件都不外乎这两种状态：已跟踪或未跟踪。 
初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态。工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区。 

使用 Git 时文件的生命周期如下：
<img src="https://git-scm.com/book/en/v2/images/lifecycle.png" />

#### 检查当前文件状态

```
$ git status
On branch master
nothing to commit, working directory clean
```

在项目下创建一个新的 README 文件。
```
$ echo 'READ ME' > README
$ git status
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

nothing added to commit but untracked files present (use "git add" to track)
```

#### 跟踪新文件
```
$ git add README

$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
```

#### 暂存已修改文件
现在我们来修改一个已被跟踪的文件。 如果你修改了一个名为 CONTRIBUTING.md 的已被跟踪的文件，然后运行 git status 命令，会看到下面内容：
```
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

文件 CONTRIBUTING.md 出现在 Changes not staged for commit 这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区。 要暂存这次更新，需要运行 git add 命令。
```
$ git add CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
```
现在两个文件都已暂存，下次提交时就会一并记录到仓库。 假设此时，你想要在 CONTRIBUTING.md 里再加条注释， 重新编辑存盘后，准备好提交。 不过这时，再运行 git status 看看：
```
$ vim CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

怎么回事？ 现在 CONTRIBUTING.md 文件同时出现在暂存区和非暂存区。实际上 Git 只不过暂存了你运行 git add 命令时的版本， 如果你现在提交，CONTRIBUTING.md 的版本是你最后一次运行 git add 命令时的那个版本，而不是你运行 git commit 时，在工作目录中的当前版本。 所以，运行了 git add 之后又作了修订的文件，需要重新运行 git add 把最新版本重新暂存起来：

```
$ git add CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
```

#### 状态简览
```
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```
??: 新添加的未跟踪文件
A: 新添加到暂存区中的文件
M: 修改过的文件
MM: 出现在右边的 M 表示该文件被修改了但是还没放入暂存区，出现在靠左边的 M 表示该文件被修改了并放入了暂存区

例如，上面的状态报告显示： README 文件在工作区被修改了但是还没有将修改后的文件放入暂存区, lib/simplegit.rb 文件被修改了并将修改后的文件放入了暂存区。 而 Rakefile 在工作区被修改并提交到暂存区后又在工作区中被修改了，所以在暂存区和工作区都有该文件被修改了的记录。

#### 忽略文件
一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 .gitignore 的文件。

文件 .gitignore 的格式规范如下：

 - 所有空行或者以 ＃ 开头的行都会被 Git 忽略。

 - 可以使用标准的 glob 模式匹配。

 - 匹配模式可以以（/）开头防止递归。

 - 匹配模式可以以（/）结尾指定目录。

 - 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号 * 匹配零个或多个任意字符；[abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。 使用两个星号（*) 表示匹配任意中间目录，比如`a/**/z` 可以匹配 a/z, a/b/z 或 `a/b/c/z`等。

#### 提交更新

给 `git commit` 加上 -a 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 git add 步骤：

```
$ git commit -a -m 'added new benchmarks'
[master 83e38c7] added new benchmarks
 1 file changed, 5 insertions(+), 0 deletions(-)
```

### 提交纪录

#### 查看提交历史
```
$ git log

commit de6fa0e2fbd4927ba44d9ab15c8ad1fe508c2b95
Author: liangbo <liangbo@codoon.com>
Date:   Sun Oct 15 11:10:01 2017 +0800

    sync files

commit 9965b2ad570557b4e0d3c147a0b40a318d536e0b
Author: liangbo <liangbo@codoon.com>
Date:   Sun Sep 24 20:24:11 2017 +0800

    modify blog name

commit 2f0d15818f6bbf30bd8144137f839a494e6cb70a
Author: liangbo <liangbo@codoon.com>
Date:   Fri Sep 22 15:52:31 2017 +0800

    update about html page

commit fca9af8e3f7be2ee13e51208084f5f32542bacb8
Author: liangbo <liangbo@codoon.com>
Date:   Fri Sep 22 15:30:36 2017 +0800

    add mari blog link 
...
    
```

默认不用任何参数的话，git log 会按提交时间列出所有的更新，最近的更新排在最上面。 正如你所看到的，这个命令会列出每个提交的 SHA-1 校验和、作者的名字和电子邮件地址、提交时间以及提交说明。

git log 有许多选项可以帮助你搜寻你所要找的提交， 接下来我们介绍些最常用的。

一个常用的选项是 -p，用来显示每次提交的内容差异。 你也可以加上 -2 来仅显示最近两次提交：
```
$ git log -p -2
```
该选项除了显示基本信息之外，还附带了每次 commit 的变化。 当进行代码审查，或者快速浏览某个搭档提交的 commit 所带来的变化的时候，这个参数就非常有用了。

如果你想看到每次提交的简略的统计信息，你可以使用 --stat 选项：
```
$ git log --stat
```
另外一个常用的选项是 --pretty。 这个选项可以指定使用不同于默认格式的方式展示提交历史。 这个选项有一些内建的子选项供你使用。 比如用 oneline 将每个提交放在一行显示，查看的提交数很大时非常有用。 另外还有 short，full 和 fuller 可以用，展示的信息或多或少有些不同，请自己动手实践一下看看效果如何。
```
git log --pretty=oneline

```

### 撤消操作
可以使用 git reset HEAD <file>... 来取消暂存。 
```
$ git reset HEAD CONTRIBUTING.md
Unstaged changes after reset:
M	CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

在调用时加上 --hard 选项可以令 git reset 成为一个危险的命令（可能导致工作目录中所有当前进度丢失！），但工作目录内的文件并不会被修改。 不加选项地调用 git reset 并不危险 — 它只会修改暂存区域。


### 远程仓库的使用

#### 查看远程仓库
指定选项 -v，会显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL。
```
$ git remote -v
origin	git@github.com:amuguelove/hexo.git (fetch)
origin	git@github.com:amuguelove/hexo.git (push)
```

####  添加或关联远程仓库

```
git remote add <shortname> <url>

git remote add origin git@github.com:amuguelove/demo.git
```

#### 从远程仓库中抓取与拉取
```
$ git fetch [remote-name]
```
这个命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。

如果你使用 clone 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写。 所以，git fetch origin 会抓取克隆（或上一次抓取）后新推送的所有工作。 必须注意 git fetch 命令会将数据拉取到你的本地仓库 - 它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。

如果你有一个分支设置为跟踪一个远程分支（阅读下一节与 Git 分支 了解更多信息），可以使用 git pull 命令来自动的抓取然后合并远程分支到当前分支。 这对你来说可能是一个更简单或更舒服的工作流程；默认情况下，git clone 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 master 分支（或不管是什么名字的默认分支）。 运行 git pull 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。


#### 推送到远程仓库
```
git push [remote-name] [branch-name]

$ git push origin master
```

#### 查看远程仓库
```
git remote show [remote-name] 

$ git remote show origin
* remote origin
  Fetch URL: git@github.com:amuguelove/hexo.git
  Push  URL: git@github.com:amuguelove/hexo.git
  HEAD branch: master
  Remote branch:
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)

```
它同样会列出远程仓库的 URL 与跟踪分支的信息。 这些信息非常有用，它告诉你正处于 master 分支，并且如果运行 git pull，就会抓取所有的远程引用，然后将远程 master 分支合并到本地 master 分支。 它也会列出拉取到的所有远程引用。

如果你是 Git 的重度使用者，那么还可以通过 git remote show 看到更多的信息。
```
$ git remote show origin
* remote origin
  URL: https://github.com/my-org/complex-project
  Fetch URL: https://github.com/my-org/complex-project
  Push  URL: https://github.com/my-org/complex-project
  HEAD branch: master
  Remote branches:
    master                           tracked
    dev-branch                       tracked
    markdown-strip                   tracked
    issue-43                         new (next fetch will store in remotes/origin)
    issue-45                         new (next fetch will store in remotes/origin)
    refs/remotes/origin/issue-11     stale (use 'git remote prune' to remove)
  Local branches configured for 'git pull':
    dev-branch merges with remote dev-branch
    master     merges with remote master
  Local refs configured for 'git push':
    dev-branch                     pushes to dev-branch                     (up to date)
    markdown-strip                 pushes to markdown-strip                 (up to date)
    master                         pushes to master                         (up to date)
```
这个命令列出了当你在特定的分支上执行 git push 会自动地推送到哪一个远程分支。 它也同样地列出了哪些远程分支不在你的本地，哪些远程分支已经从服务器上移除了，还有当你执行 git pull 时哪些分支会自动合并。

### 标签
#### 列出标签
```
$ git tag
v1.0
```

#### 创建标签
Git 使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）。

一个轻量标签很像一个不会改变的分支 - 它只是一个特定提交的引用。

然而，附注标签是存储在 Git 数据库中的一个完整对象。 它们是可以被校验的；其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息；并且可以使用 GNU Privacy Guard （GPG）签名与验证。 通常建议创建附注标签，这样你可以拥有以上所有信息；但是如果你只是想用一个临时的标签，或者因为某些原因不想要保存那些信息，轻量标签也是可用的。
```
$ git tag -a v1.1 -m 'my version 1.1'
```
通过使用 `git show` 命令可以看到标签信息与对应的提交信息：
```
$ git show v1.0
```
输出显示了打标签者的信息、打标签的日期时间、附注信息，然后显示具体的提交信息。

#### 检出标签
可以使用 `git checkout -b [branchname] [tagname]` 在特定的标签上创建一个新分支：
```
$ git checkout -b version2 v2.0.0
Switched to a new branch 'version2'
```


