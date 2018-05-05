---
layout: post
keywords: Linux MySQL
title: CentOS 7下 MySQL 5.7的安装与配置
date: 2018-04-05
categories: 数据库
tags:
    - MySQL
    - Linux
---

安装环境：`CentOS 7.4` 64位，安装 `MySQL 5.7`。

### 配置YUM源

在MySQL官网中下载YUM源rpm安装包：[http://dev.mysql.com/downloads/repo/yum/](http://dev.mysql.com/downloads/repo/yum/)

找到最新的yum源

```
 #下载mysql源安装包
shell> wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
 #安装mysql源
shell> yum localinstall mysql57-community-release-el7-8.noarch.rpm
```
检查mysql源是否安装成功

```
shell> yum repolist enabled | grep "mysql.*-community.*"

mysql-connectors-community/x86_64    MySQL Connectors Community               45
mysql-tools-community/x86_64         MySQL Tools Community                    59
mysql57-community/x86_64             MySQL 5.7 Community Server              247
```
<!-- more -->

看到上面所示表示安装成功。 
可以修改`vim /etc/yum.repos.d/mysql-community.repo`源，改变默认安装的mysql版本。比如要安装5.6版本，将5.7源`[mysql57-community]`的enabled=1改成enabled=0。然后再将5.6源的enabled=0改成enabled=1即可。

```
[mysql-connectors-community]
name=MySQL Connectors Community
baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql-tools-community]
name=MySQL Tools Community
baseurl=http://repo.mysql.com/yum/mysql-tools-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.5
[mysql55-community]
name=MySQL 5.5 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.5-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.6
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/7/$basearch/
enabled=0
```

### 安装MySQL
```
yum install mysql-community-server
```

### 启动MySQL服务
注意`service`和`systemctl`的区别，可以自行百度。

```
systemctl start mysqld
```
查看MySQL的启动状态：

```
shell> systemctl status mysqld

● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2018-04-08 17:33:26 CST; 24h ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 3336 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 3318 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 3340 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─3340 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

Apr 08 17:33:25 VM_0_14_centos systemd[1]: Starting MySQL Server...
Apr 08 17:33:26 VM_0_14_centos systemd[1]: Started MySQL Server.
```

### 开机启动
```
shell> systemctl enable mysqld
shell> systemctl daemon-reload
```

### 修改root登陆密码
mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个默认密码。通过下面的方式找到root默认密码，然后登录mysql进行修改：

```
shell> grep 'temporary password' /var/log/mysqld.log

2018-04-08T09:15:08.936150Z 1 [Note] A 'temporary password' is generated for root@localhost: 4XCd>0sA)nFe
```
拷贝下来，登陆：

```
shell> mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```
或者

```
mysql> set password for 'root'@'localhost'=password('MyNewPass4!'); 
```

注意：mysql5.7默认安装了密码安全检查插件（validate_password），默认密码检查策略要求密码必须包含：大小写字母、数字和特殊符号，并且长度不能少于8位。否则会提示：

```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```
共有以下几种密码策略：

	策略   | 检查规则
	----------   | ---------- 
	0 or LOW     | Length       
	1 or MEDIUM  | Length; numeric, lowercase/uppercase, and special characters       
	2 or STRONG  | Length; numeric, lowercase/uppercase, and special characters, dictionary file       

#### 修改密码策略

在`/etc/my.cnf`文件添加`validate_password_policy`配置，指定密码策略

选择0（LOW），1（MEDIUM），2（STRONG）其中一种，选择2需要提供密码字典文件

`validate_password_policy = 0`

如果不需要密码策略，添加my.cnf文件中添加如下配置禁用即可：

`validate_password = off`

重新启动mysql服务使配置生效：

```
systemctl restart mysqld
```

### 添加远程登录用户
默认只允许root帐户在本地登录，如果要在其它机器上连接mysql，必须修改root允许远程连接，或者添加一个允许远程连接的帐户，为了安全起见，我添加一个新的帐户：

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'yangxin'@'%' IDENTIFIED BY 'Yangxin0917!' WITH GRANT OPTION;

```

### 配置默认编码为utf8
修改/etc/my.cnf配置文件，在[mysqld]下添加编码配置，如下所示：
```
[mysqld]
character_set_server = utf8
init_connect='SET NAMES utf8'
```

重新启动mysql服务

### 默认配置文件路径： 
配置文件：`/etc/my.cnf `  
日志文件：`/var/log//var/log/mysqld.log`  
服务启动脚本：`/usr/lib/systemd/system/mysqld.service`   
socket文件：`/var/run/mysqld/mysqld.pid`  

<hr/>参考：  
[CentOS7 64位下MySQL5.7安装与配置（YUM）](https://www.linuxidc.com/Linux/2016-09/135288.htm)


