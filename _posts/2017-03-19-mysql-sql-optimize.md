---
layout: post
keywords: MySQL 优化
title: MySQL优化总结
date: 2017-03-19
categories: 数据库
tags: MySQL
---

网上有很多优化SQL的教程，也比较杂，自己整理一份出来，方便查看。

### 一、通过 show status 了解SQL执行频率
通过show status可以提供服务器状态信息，可以根据需要显示当前session级别的统计结果和 global级别的统计结果。 如：显示当前session，show status like "Com_%"，全局级别：show global status。

以下几个参数对 Myisam 和 Innodb 存储引擎都计数： 

```
Com_select 执行 select 操作的次数，一次查询只累加 1
Com_insert 执行 insert 操作的次数，对于批量插入的 insert 操作，只累加一次
Com_update 执行 update 操作的次数
Com_delete 执行 delete 操作的次数
```

以下几个参数是针对 Innodb 存储引擎计数的，累加的算法也略有不同：

```
Innodb_rows_read        执行select 查询返回的行数
Innodb_rows_inserted	执行 Insert 操作插入的行数
Innodb_rows_updated 	执行 update 操作更新的行数
Innodb_rows_deleted 	执行 delete 操作删除的行数
```

通过以上几个参数，可以很容易的了解当前数据库的应用是以插入更新为主还是以查询操作为主，以及各种类型的 SQL 大致的执行比例是多少。

此外，以下几个参数便于我们了解数据库的基本情况：

```
Connections 	试图连接 Mysql 服务器的次数
Uptime 		服务器工作时间
Slow_queries 	慢查询的次数
```
<!-- more -->
### 二、定位执行效率低的SQL
可以通过以下两种方式定位执行效率较低的 SQL 语句：

1）可以通过慢查询日志定位那些执行效率较低的 sql 语句
在my.ini文件下的mysqld下添加如下代码：
```
 log-slow-queries=c:/mysql/slowquery.log 
 long_query_time=2 
```
上面的配置打开了slow query日志,将会捕获了执行时间超过了2秒的查询，包括执行速度较慢的管理命令(比如OPTIMEZE TABLE)，并且记录了没有使用索引的查询。这些SQL，都会被记录到log-slow-queries指定的文件c:/mysql/slowquery.log 文件中。

2）使用 show processlist 查看当前MYSQL的线程，因慢查询日志在查询结束以后才纪录，所以在应用反映执行效率出现问题的时候查询慢查询日志并不能定位问题，可以使用 show processlist 命令查看当前 MySQL 在进行的线程，包括线程的状态，是否锁表等等，可以实时的查看 SQL 执行情况， 同时对一些锁表操作进行优化。 

### 三、通过explain或desc分析低效的SQL

通过以上步骤查询到效率低的 SQL 后，我们可以通过 explain 或者 desc 来获取 MySQL 如何执行 SELECT 语句的信息，包括 select 语句执行过程表如何连接和连接的次序。 下面再来介绍explain 或 desc。

### 四、MySQL索引

索引用于快速找出在某个列中有一特定值的行。对相关列使用索引是提高 SELECT 操作性能的最佳途径。 
查询要使用索引最主要的条件是查询条件中需要使用索引关键字，如果是多列索引，那么只有查询条件使用了多列关键字最左边的前缀时（ 前缀索引 ），才可以使用索引，否则将不能使用索引。

下列情况下， Mysql 不会使用已有的索引：

1）如果 mysql 估计使用索引比全表扫描更慢，则不使用索引。

2）如果使用 heap 表，并且 where 条件中不用 ＝ 索引列， 其他 > 、 < 、 >= 、 <= 均不使用索引（MyISAM和innodb表使用索引）。

3）使用or分割的条件，如果or前的条件中的列有索引，后面的列中没有索引，那么涉及到的索引都不会使用。

4）如果创建复合索引，如果条件中使用的列不是索引列的第一部分（不是前缀索引）。

5）如果 like 是以％开始。

6）对 where 后边条件为字符串的一定要加引号，字符串如果为数字 mysql 会自动转为字符串，但是不使用索引。 

### 五、索引的使用情况

如果索引正在工作， Handler_read_key 的值将很高，这个值代表了一个行被索引值读的次数，很低的值表明增加索引得到的性能改善不高，因为索引并不经常使用。 

Handler_read_rnd_next 的值高则意味着查询运行低效，并且应该建立索引补救。这个值的含义是在数据文件中读下一行的请求数。如果你正进行大量的表扫描，该值较高。通常说明表索引不正确或写入的查询没有利用索引。 

语法：`show status like 'Handler_read%'`。

### 六、优化查询语句

1）对查询进行优化，应避免全表查询，首先应该考虑在where及order by设计的列上建立索引

2）应尽量避免在 where 子句中对字段进行 null 值判断 ，否则将导致引擎放弃使用索引而进行全表扫描 

NULL对于大多数数据库都需要特殊处理，MySQL也不例外，它需要更多的代码，更多的检查和特殊的索引逻辑，有些开发人员完全没有意识到，创建表时NULL是默认值，但大多数时候应该使用NOT NULL，或者使用一个特殊的值，如 0，-1作为默认值。

不能用null作索引，任何包含null值的列都将不会被包含在索引中。即使索引有多列这样的情况下，只要这些列中有一列含有null，该列就会从索引中排除。也就是说如果某列存在空值，即使对该列建索引也不会提高性能。任何在where子句中使用is null或is not null的语句优化器是不允许使用索引的。

3）应尽量避免在 where 子句中使用!=或<>操作符， 否则将引擎放弃使用索引而进行全表扫描

MySQL只有对以下操作符才使用索引：<，<=，=，>，>=，BETWEEN，IN，以及某些时候的LIKE。 
可以在LIKE操作中使用索引的情形是指另一个操作数不是以通配符（%或者_）开头的情形。
例如:
`SELECT id FROM  t WHERE col LIKE 'Mich%'`       - 这个查询将使用索引，
`SELECT id FROM  t WHERE col  LIKE '%ike'`       - 这个查询不会使用索引。

4）应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描

在某些情况下，or条件可以避免全表扫描：      
a. where 语句里面如果带有or条件, myisam表能用到索引，innodb不行。
b. 必须所有的or条件都必须是独立索引

5）并不是所有索引对查询都有效

SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引，如一表中有字段sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。

6）索引并不是越多越好 

索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。 

7）应尽可能的避免更新 clustered 索引数据列

因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。

8）尽量使用数字型字段 

若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。

9）尽可能的使用 `varchar/nvarchar` 代替 `char/nchar` 

因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。 

10）最好不要使用`*`返回所有：`select * from t;`

用具体的字段列表代替`*`，不要返回用不到的任何字段。


### 七、优化Order by语句

MySQL的弱点之一是它的排序。虽然MySQL可以在1秒中查询大约15,000条记录，但由于MySQL在查询时最多只能使用一个索引。因此，如果WHERE条件已经占用了索引，那么在排序中就不使用索引了，这将大大降低查询的速度。我们可以看看如下的SQL语句:

`SELECT * FROM SALES WHERE NAME = ‘name’ ORDER BY SALE_DATE DESC` 

在以上的 SQL 的 WHERE 子句中已经使用了 NAME 字段上的索引，因此，在对 SALE_DATE 进行排序时将不再使用索引。为了解决这个问题，我们可以对SALES表建立复合索引:

`ALTER TABLE SALES DROP INDEX NAME, ADD INDEX (NAME, SALE_DATE);`
     
这样再使用上述的 SELECT 语句进行查询时速度就会大副提升。但要注意，在使用这个方法时，要确保 WHERE 子句中没有排序字段，在上例中就是不能用 SALE_DATE 进行查询，否则虽然排序快了，但是 SALE_DATE 字段上没有单独的索引，因此查询又会慢下来。

在某些情况中， MySQL可以使用一个索引来满足 ORDER BY 子句，而不需要额外的排序。where 条件和 order by 使用相同的索引，并且 order by 的顺序和索引顺序相同，并且 order by 的字段都是升序或者都是降序。例如：下列sql可以使用索引。

 `SELECT * FROM t1 ORDER BY key_part1,key_part2, ... ;`
 `SELECT * FROM t1 WHERE key_part1 = 1 ORDER BY key_part1 DESC, key_part2 DESC;`
 `SELECT * FROM t1 ORDER BY key_part1 DESC, key_part2 DESC;`

但是以下情况不使用索引：
 `SELECT * FROM t1 ORDER BY key_part1 DESC, key_part2 ASC;`    -- order by 的字段混合 ASC 和 DESC
 `SELECT * FROM t1 WHERE key2 = constant ORDER BY key1;`       -- 查询行的关键字与ORDER BY中所使用的不相同
 `SELECT * FROM t1 ORDER BY key1, key2;`                       -- 对不同的关键字使用 ORDER BY

### 八、Explain/Desc 解释说明

explain显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。使用方法，在select语句前加上explain就可以了。
结果分析：
`table |  type | possible_keys | key | key_len  | ref | rows | Extra`

##### 1）table
显示这一行的数据是关于哪张表的 

##### 2）type
这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为：system、const、eg_ref、ref、ref_or_null、range、index、ALL。

system: 
表仅有一行(=系统表)。这是const联接类型的一个特例

const: (PRIMARY KEY 或 UNIQUE)
表最多有一个匹配行，它将在查询开始时被读取。因为仅有一行，在这行的列值可被优化器剩余部分认为是常数。 const表很快，因为它们只读取一次！
eg: `SELECT * from tb_name WHERE primary_key = 1；`    

eq_ref: key
对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。
它用在一个索引的所有部分被联接使用并且索引是 UNIQUE 或 PRIMARY KEY。
eq_ref可以用于使用 = 操作符比较的带索引的列。比较值可以为常量或一个使用在该表前面所读取的表的列的表达式。

ref: key
对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是 UNIQUE 或 PRIMARY KEY（换句话说，如果联接不能基于关键字选择单个行的话），则使用ref。
如果使用的键仅仅匹配少量行，该联接类型是不错的。 ref可以用于使用=或<=>操作符的带索引的列。

ref_or_null: Or Is null
该联接类型如同ref，但是添加了 MySQL 可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化。
    
range: =、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN
只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。key_len包含所使用索引的最长关键元素。在该类型中ref列为NULL。
当使用=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN操作符，用常量比较关键字列时，可以使用range。
    
index:
该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。
当查询只使用作为单索引一部分的列时，MySQL可以使用该联接类型。
    
ALL：
对于每个来自于先前的表的行组合，进行完整的表扫描。如果表是第一个没标记const的表，这通常不好，并且通常在它情况下很差。通常可以增加更多的索引而不要使用ALL，使得行能基于前面的表中的常数值或列值被检索出。

##### 3）possible_keys
显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句 

##### 4）key
实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。
这种情况下，可以在SELECT语句中使用USEINDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引。

##### 5）ken_len
使用的索引的长度。在不损失精确性的情况下，长度越短越好 

##### 6）ref
显示索引的哪一列被使用了，如果可能的话，是一个常数 

##### 7）rows
MySQL认为必须检查的用来返回请求数据的行数 (扫描行的数量)

##### 8）Extra
该列包含MySQL解决查询的详细信息，这里可以看到的坏的例子是 Using temporary 和 Using filesort，意思MySQL根本不能使用索引，结果是检索会很慢。 

Extra列返回的描述的意义：

Distinct :
  一旦MYSQL找到了与行相联合匹配的行，就不再搜索了。 

Not exists :
MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了。
下面是一个可以这样优化的查询类型的例子：
`SELECT * FROM t1 LEFT JOIN t2 ON t1.id=t2.id WHERE t2.id IS NULL；`
假定t2.id定义为NOT NULL。在这种情况下，MySQL使用t1.id的值扫描t1并查找t2中的行。如果MySQL在t2中发现一个匹配的行，它知道t2.id绝不会为NULL，并且不再扫    描t2内有相同的id值的行。
换句话说，对于t1的每个行，MySQL只需要在t2中查找一次，无论t2内实际有多少匹配的行。

Range checked for each Record（index map:#) : 
没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。
这是使用索引的最慢的连接之一 ，MySQL没有发现好的可以使用的索引，但发现如果来自前面的表的列值已知，可能部分索引可以使用。
对前面的表的每个行组合，MySQL检查是否可以使用range或index_merge访问方法来索取行。
不同的是前面表的所有列值已知并且认为是常量。这并不很快，但比执行没有索引的联接要快得多。

Using filesort :
看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。
它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行。 

Using index : 
列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，
这发生在对表的全部的请求列都是同一个索引的部分的时候。

Using temporary : 
看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上。

Using where :
使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题。


