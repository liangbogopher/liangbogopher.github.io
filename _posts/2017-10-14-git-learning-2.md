---
layout: post
keywords: Git 分支
title: Git 简单指南（二）- Git分支
date: 2017-10-14
categories: Git
tags: Git
---

下面要介绍 Git 的杀手级特性：分支模型。
### 分支简介
使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。 在很多版本控制系统中，这是一个略微低效的过程——常常需要完全创建一个源代码目录的副本。对于大项目来说，这样的过程会耗费很多时间。

为何 Git 的分支模型如此出众呢？ Git 处理分支的方式可谓是难以置信的轻量，创建新分支这一操作几乎能在瞬间完成，并且在不同分支之间的切换操作也是一样便捷。 与许多其它版本控制系统不同，Git 鼓励在工作流程中频繁地使用分支与合并，哪怕一天之内进行许多次。理解和精通这一特性，你便会意识到 Git 是如此的强大而又独特，并且从此真正改变你的开发方式。

为了真正理解 Git 处理分支的方式，我们需要回顾一下 Git 是如何保存数据的。
Git 保存的不是文件的变化或者差异，而是一系列不同时刻的文件快照。

为了更加形象地说明，我们假设现在有一个工作目录，里面包含了三个将要被暂存和提交的文件。
当使用 git commit 进行提交操作时，Git 会先计算每一个子目录的校验和，然后在 Git 仓库中这些校验和保存为树对象。 随后，Git 便会创建一个提交对象，它除了包含上面提到的那些信息外，还包含指向这个树对象（项目根目录）的指针。
<!-- more -->
现在，Git 仓库中有五个对象：三个 blob 对象（保存着文件快照）、一个树对象（记录着目录结构和 blob 对象索引）以及一个提交对象（包含着指向前述树对象的指针和所有提交信息）。如图：

<img src="https://git-scm.com/book/en/v2/images/commit-and-tree.png" />

做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针。
<img src="https://git-scm.com/book/en/v2/images/commits-and-parents.png" />

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。 Git 的默认分支名字是 master。 在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 master 分支。 它会在每次的提交操作中自动向前移动。
<img src="https://git-scm.com/book/en/v2/images/branch-and-history.png" />

#### 分支创建
```
$ git branch testing
```
这会在当前所在的提交对象上创建一个指针。
<img src="https://git-scm.com/book/en/v2/images/two-branches.png" />

那么，Git 又是怎么知道当前在哪一个分支上呢？ 也很简单，它有一个名为 HEAD 的特殊指针。在 Git 中，它是一个指针，指向当前所在的本地分支。

<img src="https://git-scm.com/book/en/v2/images/head-to-master.png" />

你可以简单地使用 `git log` 命令查看各个分支当前所指的对象。 提供这一功能的参数是 `--decorate`
```
$ git log --oneline --decorate
de6fa0e (HEAD -> master, origin/master) sync files
```

#### 分支切换
```
$ git checkout testing
```
这样 HEAD 就指向 testing 分支了。
<img src="https://git-scm.com/book/en/v2/images/head-to-testing.png" />


### 分支合并
如图：
<img src="https://git-scm.com/book/en/v2/images/basic-merging-1.png" />
Git 会使用两个分支的末端所指的快照（C4 和 C5）以及这两个分支的工作祖先（C2），做一个简单的三方合并。

和之前将分支指针向前推进所不同的是，Git 将此次三方合并的结果做了一个新的快照并且自动创建一个新的提交指向它。 这个被称作一次合并提交，它的特别之处在于他有不止一个父提交。
<img src="https://git-scm.com/book/en/v2/images/basic-merging-2.png" />

#### 遇到冲突时的分支合并
此时 Git 做了合并，但是没有自动地创建一个新的合并提交。 Git 会暂停下来，等待你去解决合并产生的冲突。 你可以在合并冲突后的任意时刻使用 git status 命令来查看那些因包含合并冲突而处于未合并（unmerged）状态的文件：
```
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

    both modified:      index.html

no changes added to commit (use "git add" and/or "git commit -a")
```
出现冲突的文件会包含一些特殊区段，看起来像下面这个样子：
```
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```
在你解决了所有文件里的冲突之后，对每个文件使用 git add 命令来将其标记为冲突已解决。 一旦暂存这些原本有冲突的文件，Git 就会将它们标记为冲突已解决。

这时你可以输入 git commit 来完成合并提交。

### 分支管理
`git branch`如果不加任何参数运行它，会得到当前所有分支的一个列表：
```
$ git branch
* master
  tesing
```

如果需要查看每一个分支的最后一次提交，可以运行 `git branch` -v 命令：
```
* master de6fa0e sync files
  tesing de6fa0e sync files
```
`--merged` 与 `--no-merged` 这两个有用的选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支。
```
$ git branch --merged

$ git branch --no-merged
```

删除分支：
```
$ git branch -d testing
```
它包含了还未合并的工作，尝试使用 `git branch -d` 命令删除它时会失败，可以使用 `-D` 选项强制删除它。

### 分支开发工作流

由于分支管理的便捷，才衍生出一些典型的工作模式，你可以根据项目实际情况选择一种用用看。

#### 长期分支
许多使用 Git 的开发者都喜欢使用这种方式来工作，比如只在 master 分支上保留完全稳定的代码——有可能仅仅是已经发布或即将发布的代码。 他们还有一些名为 develop 或者 next 的平行分支，被用来做后续开发或者测试稳定性——这些分支不必保持绝对稳定，但是一旦达到稳定状态，它们就可以被合并入 master 分支了。 这样，在确保这些已完成的特性分支（短期分支，比如之前的 iss53 分支）能够通过所有测试，并且不会引入更多 bug 之后，就可以合并入主干分支中，等待下一次的发布。

<img src="https://git-scm.com/book/en/v2/images/lr-branches-1.png" />

通常把他们想象成流水线（work silos）可能更好理解一点：
<img src="https://git-scm.com/book/en/v2/images/lr-branches-2.png" />
这么做的目的是使你的分支具有不同级别的稳定性；当它们具有一定程度的稳定性后，再把它们合并入具有更高级别稳定性的分支中。 再次强调一下，使用多个长期分支的方法并非必要，但是这么做通常很有帮助，尤其是当你在一个非常庞大或者复杂的项目中工作时。


#### 特性分支
特性分支对任何规模的项目都适用。 特性分支是一种短期分支，它被用来实现单一特性或其相关工作。在 Git 中一天之内多次创建、使用、合并、删除分支都很常见。

考虑这样一个例子：
<img src="https://git-scm.com/book/en/v2/images/topic-branches-1.png" />

这时，如果你觉得iss91v2和dumbidea不错，这时你可以抛弃 iss91 分支（即丢弃 C5 和 C6 提交），然后把另外两个分支合并入主干分支。 最终你的提交历史看起来像下面这个样子：
<img src="https://git-scm.com/book/en/v2/images/topic-branches-2.png" />

请牢记，当你做这么多操作的时候，这些分支全部都存于本地。 当你新建和合并分支的时候，所有这一切都只发生在你本地的 Git 版本库中 —— 没有与服务器发生交互。

### 远程分支
通过 `git remote show (remote)` 获得远程分支的更多信息

```
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

#### 推送
运行 `git push (remote) (branch)`:
```
$ git push origin serverfix

```

#### 拉取
当 git fetch 命令从服务器上抓取本地没有的数据时，它并不会修改工作目录中的内容。 它只会获取数据然后让你自己合并。 然而，git pull 在大多数情况下它的含义是一个 git fetch 紧接着一个 git merge 命令。 git pull 都会查找当前分支所跟踪的服务器与分支，从服务器上抓取数据然后尝试合并入那个远程分支。

由于 git pull 的魔法经常令人困惑所以通常单独显式地使用 fetch 与 merge 命令会更好一些。

#### 删除远程分支
```
$ git push origin --delete serverfix
```

### 变基
在 Git 中整合来自不同分支的修改主要有两种方法：merge 以及 rebase。


总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史，从不对已推送至别处的提交执行变基操作，这样，你才能享受到两种方式带来的便利。

Merge:

<img src="https://git-scm.com/book/en/v2/images/basic-rebase-2.png" />

Rebase:

<img src="https://git-scm.com/book/en/v2/images/basic-rebase-3.png" />

现在回到 master 分支，进行一次快进合并。
```
$ git checkout master
$ git merge experiment
```
<img src="https://git-scm.com/book/en/v2/images/basic-rebase-4.png" />

此时，C4' 指向的快照就和上面使用 merge 命令的例子中 C5 指向的快照一模一样了。 这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。 你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的，但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。

一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁——例如向某个其他人维护的项目贡献代码时。 在这种情况下，你首先在自己的分支里进行开发，当开发完成时你需要先将你的代码变基到 origin/master 上，然后再向主项目提交修改。 这样的话，该项目的维护者就不再需要进行整合工作，只需要快进合并便可。

请注意，无论是通过变基，还是通过三方合并，整合的最终结果所指向的快照始终是一样的，只不过提交历史不同罢了。 变基是将一系列提交按照原有次序依次应用到另一分支上，而合并是把最终结果合在一起。

