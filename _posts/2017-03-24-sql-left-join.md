---
layout: post
keywords: SQL Left Join
title: Left Join 加上where条件的困惑
date: 2017-03-24
categories: 数据库
tags: SQL
---

left join 的困惑：一旦加上where条件，则显示的结果等于 inner join。

### 使用 where 或 and 的区别

1、用 where 是先连接然后再筛选
2、用 and 是先筛选再连接

数据库在通过连接两张或多张表来返回记录时，都会生成一张中间的临时表，然后再将这张临时表返回给用户。  

在使用 left join 时，on 和 where 条件的区别如下：

1、on 条件是在生成临时表时使用的条件，它不管on中的条件是否为真，都会返回左边表中的记录。  
2、where 条件是在临时表生成好后，再对临时表进行过滤的条件。这时已经没有left join的含义（必须返回左边表的记录）了，条件不为真的就全部过滤掉。  
<!-- more -->
##### 假设有两张表： 

表1 tab1：
<table><tr><th>id</th><th>size</th></tr><tr><td>1</td><td>10</td></tr><tr><td>2</td><td>20</td></tr><tr><td>3</td><td>30</td></tr></table>

表2 tab2：
<table><tr><th>size</th><th>name</th></tr><tr><td>10</td><td>AAA</td></tr><tr><td>20</td><td>BBB</td></tr><tr><td>20</td><td>CCC</td></tr></table>

### 两条SQL： 

1、`select * form tab1 left join tab2 on (tab1.size = tab2.size) where tab2.name='AAA'`  
2、`select * form tab1 left join tab2 on (tab1.size = tab2.size and tab2.name='AAA')`  

#### 第一条SQL的过程：

1、中间表  
on 条件: tab1.size = tab2.size  

<table><tr><th>tab1.id</th><th>tab1.size</th><th>tab2.size</th><th>tab2.name</th></tr><tr><td>1</td><td>10</td><td>10</td><td>AAA</td></tr>	<tr><td>2</td><td>20</td><td>20</td><td>BBB</td></tr><tr><td>2</td><td>20</td><td>20</td><td>CCC</td></tr><tr><td>3</td><td>30</td><td>(null)</td><td>(null)</td></tr></table>

2、再对中间表过滤  
where 条件：tab2.name = 'AAA'  

<table><tr><th>tab1.id</th><th>tab1.size</th><th>tab2.size</th><th>tab2.name</th></tr><tr><td>1</td><td>10</td><td>10</td><td>AAA</td></tr></table>

#### 第二条SQL的过程：

1、中间表  
on 条件: tab1.size = tab2.size and tab2.name = 'AAA' (条件不为真也会返回左表中的记录) 

<table><tr><th>tab1.id</th><th>tab1.size</th><th>tab2.size</th><th>tab2.name</th></tr><tr><td>1</td><td>10</td><td>10</td><td>AAA</td></tr><tr><td>2</td><td>20</td><td>(null)</td><td>(null)</td></tr><tr><td>3</td><td>30</td><td>(null)</td><td>(null)</td></tr></table>

其实以上结果的关键原因就是 left join, right join, full join 的特殊性，不管on上的条件是否为真都会返回 left 或 right 表中的记录，full 则具有 left 和 right 的特性的并集。 而 inner jion 没这个特殊性，则条件放在 on 中和 where 中，返回的结果集是相同的。

