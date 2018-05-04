---
layout: post
keywords: Git Command
title: Git 常用命令
date: 2017-03-10
categories: Git
tags: Git
---

记录下一些常用的Git命令，使你能快速地将本地项目上传到git远程仓库（如：github）。

### 本地安装Git
本人是macOS，所以建议使用brew安装。百度搜索brew官网，一条简单的命令就可以安装了，这里就不赘述了。

```
brew install git
```

如何免密登陆Git，可以参考本人的一篇文章：[Git生成多个SSH Key](/2017/01/07/git-ssh-key/)

### 初始化一个Git仓库

首先通过命令行进入你需要上传的目录：

```
cd <your folder>
git init
```

如果该目录已经是你从远程clone下来的项目，那么你可以删除掉.git文件夹。
<!-- more -->
### 添加文件到Git仓库

```
git add readme.txt
git commit readme.txt -m "commit file"
```
这样你就把readme.txt文件提交本地Git仓库了，提交的注释为“commit file”
如果你想提交当前目录下所有的文件和子目录：

```
git add . 
git commit -m "commit message"
```

### 关联一个远程仓库

```
git remote add origin git@github.com:<username>/<projectname>.git
```
其中username是你github的用户名，projectname是你项目或根目录的名称，注意去掉<>

### 第一次推送到master分支的所有内容：

```
git push -u origin master
```

### 每次本地提交后，推送到远程

```
git push origin master
```

### 克隆一个远程仓库

```
git clone <repoUrl>
git clone -b <branchName> <repoUrl>
```

### 拉取远程代码

```
git pull
```

### 使用远程覆盖本地

```
git fetch -all
git reset -hard origin/master
```


