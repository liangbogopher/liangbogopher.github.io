---
layout: post
keywords: MacOS Ruby
title: MacOS 下安装 Ruby
date: 2018-04-15
categories: Tools
tags:
    - Ruby
    - MacOS
---

`MacOS`默认安装有`ruby`, 但是在使用一些ruby东西的使用, 经常会遇到`You don't have write permissions for... `等类似没有操作权限的问题, 一般简单但是危险的操作是在终端命令前面添加 `sudo` 赋予指定以系统权限即可. 一般为了系统安全与稳定性不允许用户执行这种操作。而且系统默认的版本在2.0。

在`MacOS`上Ruby安装方式有两种,一个是 rvm多环境安装, 一种是homebrew安装。

### 1. Homebrew 安装 Ruby

使用homebrew安装

```
$ brew install ruby
```
安装成功后，发现ruby的版本还是之前系统默认的。

```
$ ruby -v
ruby 2.0.0p648 (2015-12-16 revision 53162) [universal.x86_64-darwin16]
```
之后我就卸载了brew安装的ruby包，使用第二种方法

<!-- more -->

### 2. RVM

**MAC** 安装使用 **Ruby** 最安全方便的方式就是使用 **RVM**, 安装链接点击右侧: [rvm-install-link](https://rvm.io/rvm/install)

官方推荐使用`gpg`来安装RVM，但是我的MacOS没有`gpg`，所以使用离线安装的方式。

#### 2.1 安装 RVM

```
// 离线包
curl -sSL https://github.com/rvm/rvm/tarball/stable -o rvm-stable.tar.gz
// 创建文件夹
mkdir rvm && cd rvm
// 解包
tar --strip-components=1 -xzf ../rvm-stable.tar.gz
// 安装 
./install --auto-dotfiles
// 加载
source ~/.rvm/scripts/rvm
// if --path was specified when instaling rvm, use the specified path rather than '~/.rvm'
```
#### 2.2 rvm 安装 ruby

```
// 查询 ruby的版本
rvm list known
// 下载指定的版本
rvm install 2.4.1
// 将系统的ruby切换为下载的版本
rvm use 2.4.1  --default
```

查看版本

```
$ ruby -v
ruby 2.4.1p111 (2017-03-22 revision 58053) [x86_64-darwin16]
```

安装成功！

<hr/>**PS: 补充，(2018-04-23 19:13:00)**
以上两种方式都可以更新`ruby`，只不过是系统环境变量没有生效。
使用方法2 rvm安装方法，如果在另外的终端窗口无法打开的话，应该是系统变量加载出现了问题。

`MacOS`的环境变量，加载顺序为：

```
/etc/profile /etc/paths ~/.bash_profile ~/.bash_login ~/.profile ~/.bashrc
```
当然`/etc/profile`和`/etc/paths`是系统级别的，系统启动就会加载，后面几个是当前用户级的环境变量。后面3个按照从前往后的顺序读取，如果`~/.bash_profile`文件存在，则后面的几个文件就会被忽略不读了，如果`~/.bash_profile`文件不存在，才会以此类推读取后面的文件。`~/.bashrc`没有上述规则，它是`bash shell`打开的时候载入的。

因为之前我新建了`~/.bash_profile`文件，而使用RVM安装Ruby的环境变量配置在`~/.profile`文件中，导致打开的其他终端窗口Ruby升级失效，只要把配置代码拷贝到`~/.bash_profile`文件即可。
如果需要立即生效，执行：

```
> source ~/.bash_profile
```


