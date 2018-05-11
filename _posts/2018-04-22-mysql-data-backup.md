---
layout: post
keywords: MySQL SQL
title: MySQL 数据备份和恢复 -- mysqldump
date: 2018-04-22 18:00
categories: 数据库
tags:
    - MySQL
    - Backup
---

MySQL数据备用有很多种方式，这里只重点讲下mysqldump的用法。
### 1. 语法说明

- `-u[ name], --user=`

连接服务器时使用的MySQL用户名。

- `-p[password], --password[=password]`

连接服务器时使用的密码。如果你使用短选项形式(-p)，不能在选项和密码之间有一个空格。如果在命令行中，忽略了--password或-p选项后面的 密码值，将提示你输入一个。

- `-h, --host=name`

主机名

- `-P[ port_num], --port=port_num`

用于连接MySQL服务器的的TCP/IP端口号

- `--master-data`

这个选项可以把`binlog`的位置和文件名添加到输出中，如果等于1，将会打印成一个`CHANGE MASTER`命令；如果等于2，会加上注释前缀。并且这个选项会自动打开`--lock-all-tables`，除非同时设置了`--single-transaction`（这种情况下，全局读锁只会在开始dump的时候加上一小段时间，不要忘了阅读`--single-transaction`的部分）。在任何情况下，所有日志中的操作都会发生在导出的准确时刻。这个选项会自动关闭`--lock-tables`。

- `-x, --lock-all-tables`

锁定所有库中所有的表。这是通过在整个dump的过程中持有全局读锁来实现的。会自动关闭`--single-transaction`和`--lock-tables`。

- `--single-transaction`

通过将导出操作封装在一个事务内来使得导出的数据是一个一致性快照。只有当表使用支持`MVCC`的存储引擎（目前只有`InnoDB`）时才可以工作；其他引擎不能保证导出是一致的。当导出开启了`--single-transaction`选项时，要确保导出文件有效（正确的表数据和二进制日志位置），就要保证没有其他连接会执行如下语句：`ALTER TABLE`, `DROP TABLE`, `RENAME TABLE`, `TRUNCATE TABLE`，这会导致一致性快照失效。这个选项开启后会自动关闭`--lock-tables`。

- `-l, --lock-tables`

对所有表加读锁。（默认是打开的，用`--skip-lock-tables`来关闭，上面的选项会关闭-l选项）

- `-F, --flush-logs`

在开始导出前刷新服务器的日志文件。注意，如果你一次性导出很多数据库（使用 `-databases=`或`--all-databases`选项），导出每个库时都会触发日志刷新。例外是当使用了`--lock-all-tables`或`--master-data`时：日志只会被刷新一次，那个时候所有表都会被锁住。所以如果你希望你的导出和日志刷新发生在同一个确定的时刻，你需要使用`--lock-all-tables`，或者`--master-data`配合`--flush-logs`。

- `--delete-master-logs`

备份完成后删除主库上的日志。这个选项会自动打开`–master-data`。

- `--opt`

同`-add-drop-table`, `--add-locks`, `--create-options`, `--quick`, `--extended-insert`, `--lock-tables`, `--set-charset`, `--disable-keys`。（默认已开启，`--skip-opt`关闭表示这些选项保持它的默认值）应该给你为读入一个MySQL服务器的尽可能最快的导出，`--compact`差不多是禁用上面的选项。

- `-q, --quick`

不缓冲查询，直接导出至`stdout`。（默认打开，用`--skip-quick`来关闭）该选项用于转储大的表。

- `--set-charset`

将`SET NAMES default_character_set`加到输出中。该选项默认启用。要想禁用`SET NAMES`语句，使用`--skip-set-charset`。

- `--add-drop-tables`

在每个`CREATE TABLE`语句前添加`DROP TABLE`语句。默认开启。

- `--add-locks`

在每个表导出之前增加`LOCK TABLES`并且之后`UNLOCK TABLE`。(为了使得更快地插入到MySQL)。默认开启。

- `--create-option`
在`CREATE TABLE`语句中包括所有MySQL表选项。默认开启，使用`--skip-create-options`来关闭。

- `-e, --extended-insert`

使用全新多行INSERT语法，默认开启（给出更紧缩并且更快的插入语句）

- `-d, --no-data`

不写入表的任何行信息。如果你只想得到一个表的结构的导出，这是很有用的。

- `--add-drop-database`

在`create`数据库之前先`DROP DATABASE`，默认关闭，所以一般在导入时需要保证数据库已存在。

- `--default-character-set=`

使用的默认字符集。如果没有指定，mysqldump使用utf8。

- `--all-databases`

导出所有的库和表

- `-B, --databases`

转储几个数据库。通常情况，mysqldump将命令行中的第1个名字参量看作数据库名，后面的名看作表名。使用该选项，它将所有名字参量看作数据库名。`CREATE DATABASE IF NOT EXISTS db_name`和`USE db_name`语句包含在每个新数据库前的输出中。

- `--tables`

覆盖`--database`选项。选项后面的所有参量被看作表名。


### 2. 栗子

假设用户名为`root`, 密码为`password`

```
$ mysqldump -hlocalhost -uroot -ppassword \
--all-databases --master-data=1 --single-transaction --flush-logs > /tmp/backup-file.sql
```
它只是在一开始的瞬间请求锁表，然后就刷新binlog了，而后在导出的文件中加入CHANGE MASTER 语句来指定当前备份的binlog位置，如果要把这个文件恢复到slave里去，就可以采用这种方法来做。

### 3. 还原

- 直接使用mysql客户端

```
/usr/local/mysql/bin/mysql -uroot -ppassword < /tmp/backup-file.sql
```

- 使用SOURCE语法

```
SOURCE /tmp/backup-file.sql;
```

<hr/>参考：  
[4.5.4 mysqldump — A Database Backup Program](https://dev.mysql.com/doc/refman/5.5/en/mysqldump.html)

