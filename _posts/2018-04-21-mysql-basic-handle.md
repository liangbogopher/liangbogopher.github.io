---
layout: post
keywords: MySQL SQL
title: MySQL 数据库基本 SQL 操作
date: 2018-04-21
categories: 数据库
tags:
    - MySQL
    - SQL
---

### 启动
如何安装 `MySQL` 可以查看我之前的文章，这里不再赘述。

```
service mysqld start
或
systemctl start mysqld
```

检查MySQL服务器是否启动：

```
 ps -ef | grep mysqld
```

### 数据库

```
//创建数据库
create database if not exists `test` default character set utf8mb4;
//查看数据库
show databases;  
//查看数据库信息    
show create database test;
//修改数据库的编码，可使用上一条语句查看是否修改成功
alter database test default character set utf8mb4 collate utf8mb4_general_ci;      
//删除数据库
drop database test;
```
<!-- more -->

### 数据表

```
// 选择要操作的数据库
use test;

//创建表student
create table student(
  id int(11) AUTO_INCREMENT,
  name varchar(20),
  age int(11),
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;;

// 列出所选数据库的所有表
show tables;

// 如果要查看数据表的属性，属性类型，主键信息 ，是否为 NULL，默认值等其他信息
show columns from student;

// 或者使用如下命令查看更加详细的表属性信息(包括字段权限，表注释等)：
show full columns from student;

// 查看数据表的详细索引信息，包括 PRIMARY KEY（主键）
show index from student;

// 输出 Mysql 数据库管理系统的性能及统计信息
show table status like 'student' \G;
```

**查看表结构:**

```
desc 表名;

show columns from 表名;

describe 表名;
```

**清空表:**

```
delete from 表名;

truncate table 表名;
```

**删除表:**

```
drop table 表名;
```

### 表字段

**修改字段属性：**

```
alter table 表名 modify 字段名称 字段类型 [是否允许非空] comment '字段注释';
```

**改变字段名称：**

```
alter table 表名 change 字段原名称 字段新名称 字段类型 [是否允许非空] comment '字段注释';
```

**添加字段：**

```
alter table 表名 add 字段名称 字段类型 [是否允许非空] comment '字段注释';
```

**删除字段：**

```
alter table 表名 drop column 字段名称;
```

### 索引

**创建索引:**

ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。

```
ALTER TABLE table_name ADD INDEX index_name (column_list)

ALTER TABLE table_name ADD UNIQUE (column_list)

ALTER TABLE table_name ADD PRIMARY KEY (column_list)
```

**删除索引:**

可利用ALTER TABLE或DROP INDEX语句来删除索引。

```
DROP INDEX index_name ON table_name

ALTER TABLE table_name DROP INDEX index_name

ALTER TABLE table_name DROP UNIQUE index_name

ALTER TABLE table_name DROP PRIMARY KEY
```

### 数据导出

**导出Excel**

方案一：

```
mysql -uroot -ppassword -hxxx -e "query statement" db > ./xxx.xls
```

方案二：

```
select * from table into outfile 'xxx.xls'; 
```

**Dump数据**

导出所有数据库：

```
mysqldump -uroot -ppassword --all-databases >/tmp/all.sql
```

导出db1，db2所有的数据：

```
mysqldump -uroot -ppassword --databases db1 db2 >/tmp/all.sql
```

导出db1中的a1、a2表：

```
mysqldump -uroot -ppassword --databases db1 --tables a1 a2  >/tmp/all.sql
```

a) 加入`--no-data`，只导出表结构，不导出数据

b) 加入`--master-data`, 记录bin log的文件名和位置，其中有两个配置`--master-data=1`和`--master-data=2`。  
=1时，直接导入备份文件，start slave不需要指定file和position    
=2时，file和position是被注释掉的，需要手动指定file和position

### 一个实用的 shell 脚本：

```
__mysql() {
    mysql -uroot -ppassword
}

if [ "$*" != '' ] ; then
    echo "$*" | __mysql
else
    __mysql
fi
```
用该命令可以直接在命令行执行 mysql 命令和 sql 语句，而不用进入 mysql 的交互式解释器。例如：
```
chmod 777 _mysql.sh
./_mysql.sh "use test;show tables"
./_mysql.sh "show processlist;"
```


