---
layout: post
keywords: Git SSH
title: Git 生成多个 SSH Key
date: 2017-01-07
categories: Git
tags:
      - Git
      - SSH
---

当有多个git账号的时候，比如一个github，用于自己进行一些开发活动，再来一个gitlab，一般是公司内部的git。这两者你的邮箱如果不同的话，就会涉及到一个问题，生成第二个git的key的时候会覆盖第一个的key，导致必然有一个用不了。

### 如何解决
在~/.ssh目录下，为每个git账号生成key，并新建一个config文件配置一下，就可以解决问题

### 具体操作

#### 生成第一个ssh key（个人github账号）
```
ssh-keygen -t rsa -C "email"
```
这里的email是注册github的邮箱地址
 
这里不要一路enter，让你选择在哪里选择存放key的时候写个名字，比如 id_rsa_github，之后的两个可以回车。
完成之后我们可以看到~/.ssh目录下多了两个文件。如下：
```
id_rsa_github
id_rsa_github.pub 
```
<!-- more --> 
#### 生成第二个ssh key（公司gitlab账号）
```
ssh-keygen -t rsa -C "email"
```
这里的email是公司gitlab注册的邮箱地址
 
还是一样不要一路回车，在第一个对话的时候继续写个名字，比如 id_rsa_gitlab, 之后的两个可以回车。如下：
```
id_rsa_gitlab
id_rsa_gitlab.pub
```

```
vim ~/.ssh/id_rsa.pub
```
然后使用鼠标拷贝里面的内容（退出vim编辑器命令:wq）

打开你的github,点击Setting, 打开SSH and GPG Keys,点击右边New SSH key.

 
#### 创建并修改config文件
 
到.ssh目录之行如下操作
```
touch config
chmod 600 config
```
添加如下内容：
```
# gitlab
  Host git.elenet.me
       HostName xxxx                           // 这里是你公司的git地址
       PreferredAuthentications publickey
       IdentityFile ~/.ssh/id_rsa_gitlab
       User xxxx                               // 这里是你的用户名

 # github
   Host github.com
        HostName github.com
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/id_rsa_github
        User xxxx                              // 这里是你的用户名
```
 
#### 在github和gitlab上添加公钥即可，这里不再多说。
 
#### 测试
```
ssh -T git@github.com
```
出现：
```
Hi yourname! You've successfully authenticated, but GitHub does not provide shell access.
```
表示github配置成功！如此这般改成公司gitlab地址再执行一遍。

 
### 补充一下
如果之前有设置全局用户名和邮箱的话，需要unset一下
 
```
git config --global --unset user.name
git config --global --unset user.email
```
然后在不同的仓库下设置局部的用户名和邮箱
比如在公司的repository下，执行：
```
git config user.name "yourname"  
git config user.email "youremail" 
```

在自己的github的仓库在执行刚刚的命令一遍即可。
这样就可以在不同的仓库，已不同的账号登录。

