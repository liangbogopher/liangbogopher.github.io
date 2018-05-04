---
layout: post
keywords: Git Aliyun
title: 阿里云搭建Git服务，实现自动部署
date: 2017-03-13
categories: Git
tags:
      - Git
      - Aliyun
---

在阿里云服务器上建个 Git 远程仓库，用 Git Hooks 实现自动部署。

## 搭建Git服务器
本人是阿里云服务器，系统是CentOS 7，下面，正式开始安装。

第一步，安装`git`：
```
yum install git
```

第二步，创建一个git用户，用来运行git服务：
```
sudo adduser git
```
用 git 用户运行貌似是约定俗成的事，像 github 一个仓库的 URL 为git@github.com:user/project.git，@ 符号前面的 git 就是运行 git 服务的用户名。

<!-- more -->

第三步，初始化远程仓库：
我的目录名为`srv`，仓库名为`blog`，以此为例。
```
cd /srv
sudo git init --bare blog.git
```
Git就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以`.git`结尾。然后，把owner改为`git`：

```
sudo chown -R git:git blog.git
```

出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：

```
git:x:1000:1000::/home/git:/bin/bash
```
改为：

```
git:x:1000:1000::/home/git:/bin/git-shell
```
这样，`git`用户可以正常通过ssh使用git，但无法登录shell，因为我们为`git`用户指定的`git-shell`每次一登录就自动退出。

第四步，创建证书登陆：

在自己的本地机器，比如mac OS，通过如下命令来生成证书。

```
ssh-keygen -t rsa
```
这样在本机的.ssh目录下生成id_rsa和id_rsa.pub两个文件，将id_rsa.pub里的内容拷贝到阿里云服务器的`/home/git/.ssh/authorized_keys`，没有则新建。

有时默认的id_rsa公钥一般都被占用了，你可以参考本人的一篇文章：[Git生成多个SSH Key](/2017/01/07/git-ssh-key/)

这时你就可以在本地电脑克隆仓库了

```
git clone git@server:/srv/blog.git
```

## Git Hooks
Git Hooks 就是一些触发特定事件的脚本。比如 commit、push、merge 等等，也区分本地 Hooks 和服务端 Hooks。

我这次使用的是`post-receive`。

> 当用户在本地仓库执行`git-push`命令时，服务器上远程仓库就会对应执行`git-receive-pack`命令，而`git-receive-pack`命令会调用`pre-receive`钩子。

总体的流程为：
1. 本地执行`git push`；
2. Git 服务器更新并 Hook；
3. 执行`pre-receive`脚本，命令为：定位到服务器本地仓库目录，执行`git pull`。

初始化服务器本地仓库：

```
cd /srv
git clone blog.git
```
因为是在同一台电脑上，所以克隆直接使用本地路径。这将会在srv生成一个名为blog本地仓库。

```
ls -la
```
可以看到当前路径下有一个`blog.git`目录和一个`blog`目录。

考虑到 git-hooks 运行使用的是 git 用户，也对服务器本地仓库授权：

```
chown -R git:git blog
```

设置 hook 执行脚本：

```
vi blog.git/hooks/post-receive
```

输入以下内容：

```
#!/bin/sh
unset GIT_DIR #还原环境变量
cd /srv/blog
git pull origin master
```
基本上实现本地和服务器的同步了。

## Nginx安装配置
安装nginx：

```
sudo yum install nginx
```

启动nginx:

```
sudo service nginx start
```

定位到配置文件，修改配置：

```
vi /etc/nginx/conf.d/default.conf
```

找到`server { ... }`区域：

```
server {
    listen      80;     #端口
    server_name localhost or yoursite.com;     #域名或IP
    root        /srv/blog;                     #站点根目录
    charset     utf-8;                         #文件编码
    index       index.html index.htm;          #首页
    error_page  404     /404.html;             #404页面
    error_page  500 502 503 504     /50x.html; #服务端错误页面
    #url访问匹配路径，可以添加多个
    location / {
        index       index.html index.htm;
        root        /srv/blog;                 #这里可以是绝对路径或者相对路径，基于站点根目录
    }
}
```

可以用一条命令测试配置是否正确：

```
nginx -t -c /etc/nginx/conf.d/default.conf
```

重启 Nginx 服务器使配置生效：

```
service nginx restart
```

## Hexo部署
原来的 Hexo 使用的是 hexo-deployer-git 插件，会在 Hexo 下生成一个 .deploy_git 目录，从这个目录上传到 pages 分支。
现在我觉得不需要这个插件了，可以直接在 public 目录下初始化 git 仓库然后上传。

```
cd hexo/public
git init
git add -A
git commit -m 'commit messages'
git remote add origin git@server:/srv/blog.git
git push origin master
```
其中server是你服务器的ip地址。

这时候通过服务器ip打开站点，就可以看到博客了。（国内服务器没有备案, 只能通过ip来访问）


