---
layout: post
keywords: MySQL MacOS
title: Mac 下 MySQL 5.7 的安装和配置
date: 2018-03-02
categories: 数据库
tags: 
    - MySQL
    - MacOS
---

Mac系统下有两种方法安装，本人主要是使用第二种homebrew来安装。

### 方法一：pkg包安装
在 [MySQL官网](http://www.mysql.com/downloads/mysql) 上根据自己系统版本下载所需要的dmg文件。

打开dmg根据步骤安装好MySQL，安装好后系统通知栏会通知你默认的root账号的密码。待会可以将初始密码修改成自己所需要的密码。

<!-- more -->

可以在系统设置偏好->MySQL->启动MySQL进行启动

安装的地址为：  
```
/usr/local/mysql
```

初始化密码可以自行百度

<!-- more -->

### 方法二：homebrew
`homebrew`是Mac系统下面目前使用最多的管理软件的工具，可以通过命令直接安装MySQL。  

```
brew update
brew install mysql
```

启动方法：  

```
sudo mysql.server start  # 启动
sudo mysql.server stop   # 关闭
```

如果启动不了，可以初始化数据目录： 
 
```
rm -rf /usr/local/var/mysql   # 清除默认数据目录
mysqld --initialize-insecure  # 初始化完成后，默认空密码
sudo chown -R mysql:mysql /usr/local/var/mysql   #修改默认数据目录的权限

```

如果使用了`--initialize-insecur`生成了空密码，需要修改密码，方法如下：

```
mysql -u root --skip-password  # 登录MySQL
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```
修改密码在MySQL 5.7.5及之前的写法为：

```
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123456');
```

brew提供了配置向导脚本`/usr/local/opt/mysql/bin/mysql_secure_installation`，可根据这个向导进行初始化设置：

```
$ /usr/local/opt/mysql/bin/mysql_secure_installation   //mysql 提供的配置向导
Securing the MySQL server deployment.
Connecting to MySQL using a blank password.
VALIDATE PASSWORD PLUGIN can be used to test passwordsand improve security. It     checks the strength of password and allows the users to set only those passwords which are secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: k //是否采用mysql密码安全检测插件（这里作为演示选择否，密码检查插件要求密码复杂程度高，大小写字母+数字+字符等）
Please set the password for root here. // 首次使用自带配置脚本，设置root密码

New password:

Re-enter new password:

By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.    
Remove anonymous users? [Y/n] Y //是否删除匿名用户
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y //是否禁止远程登录
 ... Success!

By default, MySQL comes with a database named 'test' that anyone can
access.This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y //删除测试数据库，并登录
 Dropping test database...
 ... Success!
 Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y//重新载入权限表
 ... Success!

All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!

Cleaning up...
```

### 配置文件

MySQL启动时会读取配置文件`my.cnf`，读取次序依次为 `/etc/my.cnf`、`/etc/mysql/my.cnf`、`/usr/local/etc/my.cnf`、`~/.my.cnf`。安装完MySQL后可能上述位置上都没有`my.cnf`文件，要想指定配置文件，可以将MySQL安装目录下的示例配置文件拷贝到对应位置。

```
cp $(brew --prefix mysql)/support-files/my-default.cnf /etc/my.cnf
```

但是我的mac电脑上没有，配置文件在`/usr/local/etc/my.cnf`中，

```
cp /usr/local/etc/my.cnf /etc/my.cnf
```


上文提到默认的数据目录为/usr/local/var/mysql，试验将my.cnf里的datadir修改为：


```
datadir = /Users/liangbo/mysql-data​
```

重新初始化数据目录：

```
$ mysqld --initialize-insecure --basedir="$(brew --prefix mysql)" --datadir=/Users/liangbo/mysql-data
$ sudo chown -R mysql:mysql /Users/liangbo/mysql-data
```

设置完之后就可以正常启动MySQL。

