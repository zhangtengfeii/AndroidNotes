---
title: sql语句
date: 2016-06-08 23:04:31
categories: Android
tags: sql
---
  

关键词最好大写！

<!--more-->

# 创建/删除数据库

```java
private String CREATE_DATABASE = "create database DB_NAME";
private String DELETE_DATABASE = "drop database DB_NAME";
```
# 创建删除表

```java
private String CREATE_TABLE = "create table TABLE_NAME (ID integer primary key autoincrement,TITLE text)";
private String DELETE_TABLE = "drop table if exists TABLE_NAME";
//db.execSQL();
```

# 增加/插入

insert into table_name (列名1,列名2) values (value1,value2)  
如果每一列都有值插入，那么列名可以省略不写，如下：  
INSERT INTO 表名称 VALUES (值1, 值2,....)直接按顺序对应列名


# 删除
delete from <表名> [where <删除条件>]  
例：delete from a where name='小明'（删除表a中name列值为小明的那一行数据）


# 更新
update <表名> set <列名=更新值> [where <更新条件>]   
例：update tongxunlu set 年龄=18 where 姓名='小明'（将tongxulu这个表中姓名那一列为小明的那条数据的年龄改成18）

# 查询

## 精确（条件）查询：
select <列名> from <表名> [where <查询条件表达试>] [order by <排序的列名>[asc或desc]]  
例：SELECT name FROM sqlite_master WHERE type='table' AND name='mytable'

## 查询所有数据行和列
例：select * from a  说明：查询a表中所有行和列

## 查询部分行列--条件查询
例：select i,j,k from a where f=5 说明：查询表a中f=5的所有行，并显示i,j,k３列


## 查询返回限制行数(关键字：top percent)
例1：select top 6 name from a 说明：查询表a，显示列name的前６行，top为关键字
例2：select top 60 percent name from a 说明：查询表a，显示列name的60%，percent为关键字
例3：select top 3 * from t1  order by a desc

## 查询排序（关键字：order by , asc , desc）
例：select name from a where chengji>=60 order by desc
说明：查询a表中chengji大于等于60的所有行，并按降序显示name列；默认为ＡＳＣ升序

# 参考链接
<http://www.w3school.com.cn/sql/index.asp>
<http://www.cnblogs.com/yubinfeng/archive/2010/11/02/1867386.html>