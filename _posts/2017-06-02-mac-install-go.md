---
layout: post
keywords: Mac Golang
title: Mac下安装Go和配置环境
date: 2017-06-02
categories: Golang
tags:
    - Golang
    - MacOS
---

如何在 Mac OS 下安装Go并且配置环境。
Mac系统下有两种方法安装：
### 方法一：homebrew
`homebrew`是Mac系统下面目前使用最多的管理软件的工具，目前已支持Go，可以通过命令直接安装Go。

```
brew update
brew install go
```
这样安装之后通过命令行输入go就可以看到相关的信息。输入go env查看环境信息：
<!-- more -->
```
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH=""
GORACE=""
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/rg/hv457ycs15s0mxj1d42x7bxc0000gp/T/go-build684276136=/tmp/go-build -gno-record-gcc-switches -fno-common"
CXX="clang++"
CGO_ENABLED="1"
PKG_CONFIG="pkg-config"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
```
其中GOBIN，GOPATH还没有开始配置，所以显示为空。

### 方法二：pkg包安装
直接去官方下载安装包，然后双击安装, 之后同样地输入go env、go version等查看是否安装。

因为homebrew有时连接不上，我用的是方法二安装的。

### 环境配置
1）查看是否存在.bash_profile, 如果不存在则新建.bash_profile文件
```
vim ~/.bash_profile
```

2）添加环境变量
```
# GOROOT
export GOROOT=/usr/local/go

# GOPATH
export GOPATH=$HOME/Documents/go_workspace

# GOPATH bin
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

需要立即生效，在终端执行如下命令：
```
source ~/.bash_profile
```

至此，Go语言已经安装好了。


