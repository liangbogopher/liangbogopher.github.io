---
layout: post
keywords: Linux Golang Yum
title: Linux 使用Yum安装Go和配置环境
date: 2018-04-05
categories: Golang
tags: 
    - Golang
    - Linux
---

安装环境：CentOS7.4 64位，安装Golang

### 安装Golang
查看是否安装了golang：
```
$ yum info golang
Loading mirror speeds from cached hostfile
Installed Packages
Name        : golang
Arch        : x86_64
Version     : 1.8.3
Release     : 1.el7
Size        : 11 M
Repo        : installed
From repo   : os
Summary     : The Go Programming Language
URL         : http://golang.org/
License     : BSD and Public Domain
Description : The Go Programming Language.
```
如果没有安装，执行安装命令：
```
yum install golang
```
<!-- more -->
这样安装之后通过命令行输入go就可以看到相关的信息。输入`go env`查看环境信息：

```
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH="/root/go"
GORACE=""
GOROOT="/usr/lib/golang"
GOTOOLDIR="/usr/lib/golang/pkg/tool/linux_amd64"
GCCGO="gccgo"
CC="gcc"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build681960794=/tmp/go-build -gno-record-gcc-switches"
CXX="g++"
CGO_ENABLED="1"
PKG_CONFIG="pkg-config"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
```

### 环境配置
1）查看是否存在.bash_profile, 如果不存在则新建.bash_profile文件
```
vi /etc/profile
```

2）添加环境变量
在文件后面追加如下文本：
```
# GOROOT
export GOROOT=/usr/lib/golang
# GOPATH
export GOPATH=/root/go
# GOPATH bin
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

需要立即生效，在终端执行如下命令：
```
source /etc/profile
```

至此，Go语言已经安装好了。


