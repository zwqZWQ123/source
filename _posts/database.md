---
title: 数据库原理（一）
date: 2017-04-02 19:32:38
tags: 
- SQL语言
- 内连接 & 外连接 & 交叉连接
- 范式

description: 数据库（Data  Base，简称DB）———— 存储在计算机存储介质上的、有一定组织形式的、可共享的、相互关联的数据集合。简单的说即为存储数据的仓库。

categories: 数据库
---
# SQL语言的功能
&ensp;&emsp;&emsp;**SQL**是结构化查询语言（Structured Query Language）的缩写，其功能包括**数据查询**、**数据操纵**、**数据定义**和**数据控制**4个部分。

* **数据查询**是数据库中最常见的操作，通过select语句可以得到所需的信息。
* **数据操纵**语句（Data Manipulation Language,DML）主要包括*插入数据*、*修改数据*以及*删除数据*3种语句。
* **数据定义**语句(Data Definition Language,DDL)实现数据定义功能，可对数据库用户、基本表、视图、索引进行定义与撤销。
* **数据控制**语句（Data Control Language，DCL）用于对数据库进行统一的控制管理，保证数据在多用户共享的情况下能够安全。

<center>**表1 基本SQL语句的使用方式**</center>

|   |关键字|描述|语法格式|
|---|-----|----|------|
|数据查询|select|选择符合条件的记录|select * from table where 条件语句|
|数据操纵|insert|插入一条记录|insert into table (字段1，字段2...) values (值1，值2...)|
|数据操纵|update|更新语句|update table set 字段名=字段值 where 条件表达式|
|数据操纵|delete|删除记录|Delete from table where 条件表达式|
|数据定义|create|数据表的建立|create table tablename(字段1，字段2...)|
|数据定义|drop|数据表的删除|drop table tablename|
|数据控制|grant|为用户授予系统权限|grant<系统权限> &#124; <角色>[,<系统权限> &#124; <角色>]...to <用户名> &#124; <角色> &#124; public[,<用户名> &#124; <角色>]...[with admin option]|
|数据控制|revoke|收回系统权限|revoke <系统权限> &#124; <角色>[,<系统权限> &#124; <角色> ]... from <用户名> &#124; <角色> &#124; public [,<用户名> &#124; <角色>]...|

**引申：delete 与 truncate命令有哪些区别？**
**相同点：** 都可以用来删除一个表中的数据。
**不同点：**

1. **truncate**是一个数据定义语言（Data Definition Language,DLL）,它会被隐式地提交，一旦执行后将不能回滚。**delete**执行的过程是每次从表中删除一行数据，同时将删除的操作以日志的形式进行保存，以便将来进行回滚操作。
2. 用**delete**操作后，被删除的数据占用的存储空间还在，还可以恢复。而用**truncate**操作删除数据后，被删除的数据会立即释放占用的存储空间，被删除的数据是不能被恢复的。
3. **truncate**的执行速度比**delete**快。

# 内连接、外连接、交叉连接

## 连接查询
**概念：**根据两个表或多个表的列之间的关系，从这些表中查询数据。
**目的：**实现多个表查询操作。
## 连接类型
连接分为三种：内连接、外连接、交叉连接。

举个栗子，做两张表：学生表（T_student）和班级表（T_class）。
<center> **T_student** </center>

|id|studentName|classId|
|:--|:--|:--|
|1|张三|11|
|2|李四|12|
|3|王五|13|
|4|马六|15|

<center> **T_class** </center>

|classId|className|
|---|---|
|11|一班|
|12|二班|
|13|三班|
|14|四班|

## 连接标准语法格式
SQL-92标准所定义的FROM子句的连接语法格式为：

```
FROM  join_table join_type join_table[ON (join_condition)]
```
 其中**join_table**指出参与连接操作的表名，连接可以对同一个表操作，也可以对多表操作，对同一个表操作的连接又称做自连接。**join_type**指出连接类型。**join_condition**指连接条件。
## 内连接（INNER JOIN）
&ensp;&emsp;&emsp;使用比较运算符（包括=、>、<、<>、>=、<=、!>和!<）进行表间的比较操作，查询与连接条件相匹配的数据。根据比较运算符不同，内连接分为**等值连接**、**自然连接**和**不等连接**三种。
### 等值连接
概念：在连接条件中使用等于号（=）运算符，其查询结果中列出被连接表中的所有列，**包括其中的重复列**。
<span style="font-size:18px;"><span style="font-family:System;">       
    select * from T_student s,T_class c where s.classId = c.classId   
     等于  
     select * from T_student s inner join T_class c on s.classId = c.classId</span></span> 

结果是：

|id|stundentName|classId|classId|className|
|:--|:--|:--|:--|:--|
|1|张三|11|11|一班|
|2|李四|12|12|二班|
|3|王五|13|13|三班|

### 不等连接
 概念：在连接条件中使用除等于号之外运算符**（>、<、<>、>=、<=、!>和!<）**
 <span style="font-size:18px;"><span style="font-family:System;">       
    select * from T_student s inner join T_class c on s.classId **<>** c.classId</span></span>  
    
|id|studentName|classId|classId|className|
|:--|:--|:--|:--|:--|
|2|李四|12|11|一班|
|3|王五|13|11|一班|
|4|马六|15|11|一班|
|1|张三|11|12|二班|
|3|王五|13|12|二班|
|4|马六|15|12|二班|
|1|张三|11|13|三班|
|2|李四|12|13|三班|
|4|马六|15|13|三班|
|1|张三|11|14|四班|
|2|李四|12|14|四班|
|3|王五|13|14|四班|
|4|马六|15|14|四班|

### 自然连接
概念：连接条件和等值连接相同，但是会**删除连接表中的重复列**。
查询语句同等值连接基本相同:
<span style="font-size:18px;"><span style="font-family:System;">      
    select s.*,c.className from T_student s inner join T_class c on s.classId = c.classId</span></span> 

与等值连接对比：结果是**少一个一列classId**

|id|stundentName|classId|className|
|:--|:--|:--|:--|
|1|张三|11|一班|
|2|李四|12|二班|
|3|王五|13|三班|
**总结：内连接是只显示满足条件的!**
## 外连接（OUTER JOIN）
&ensp;&emsp;&emsp;外连接分为**左连接**（LEFT JOIN）或**左外连接**（LEFT OUTER JOIN）、**右连接**（RIGHT JOIN）或**右外连接**（RIGHT OUTER JOIN）、**全连接**（FULL JOIN）或**全外连接**（FULL OUTER JOIN）。我们就简单的叫：左连接、右连接和全连接。
### 左连接
 概念：返回左表中的所有行，**如果左表中行在右表中没有匹配行，则结果中右表中的列返回空值**。
 <span style="font-size:18px;"><span style="font-family:System;">      
    select * from  T_student s left join T_class c on s.classId = c.classId</span></span> 
结果是：

|id|stundentName|classId|classId|className|
|:--|:--|:--|:--|:--|
|1|张三|11|11|一班|
|2|李四|12|12|二班|
|3|王五|13|13|三班|
|4|马六|15|NULL|NULL|

**总结：左连接显示左表全部行，和右表与左表相同行。**
### 右连接
概念：恰与左连接相反，返回右表中的所有行，**如果右表中行在左表中没有匹配行，则结果中左表中的列返回空值**。
<span style="font-size:18px;"><span style="font-family:System;">  　  
   select * from  T_student s right join T_class c on s.classId = c.classId</span></span> 
结果是：   

|id|stundentName|classId|classId|className|
|:--|:--|:--|:--|:--|
|1|张三|11|11|一班|
|2|李四|12|12|二班|
|3|王五|13|13|三班|
|NULL|NULL|NULL|14|四班|

**总结：右连接恰与左连接相反，显示右表全部行，和左表与右表相同行。**
## 全连接
概念：返回左表和右表中的所有行。**当某行在另一表中没有匹配行，则另一表中的列返回空值**
<span style="font-size:18px;"><span style="font-family:System;">      
   select * from  T_student s full join T_class c on s.classId = c.classId</span></span> 
结果是：

|id|stundentName|classId|classId|className|
|:--|:--|:--|:--|:--|
|1|张三|11|11|一班|
|2|李四|12|12|二班|
|3|王五|13|13|三班|
|4|马六|15|NULL|NULL|
|NULL|NULL|NULL|14|四班|

**总结：返回左表和右表中的所有行。**
## 交叉连接（CROSS JOIN）：也称迪卡尔积
概念：不带where条件子句，它将会返回被连接的两个表的笛卡尔积，返回结果的行数等于两个表行数的乘积（例如：T_student和T_class，返回4*4=16条记录），如果带where，返回或显示的是匹配的行数。
### 不带where
<span style="font-size:18px;"><span style="font-family:System;">     
   select *from T_student **cross join** T_class  
  等于  
   select *from T_student, T_class</span></span> 
结果是：

|id|studentName|classId|classId|className|
|:--|:--|:--|:--|:--|
|1|张三|11|11|一班|
|2|李四|12|11|一班|
|3|王五|13|11|一班|
|4|马六|15|11|一班|
|1|张三|11|12|二班|
|2|李四|12|12|二班|
|3|王五|13|12|二班|
|4|马六|15|12|二班|
|1|张三|11|13|三班|
|2|李四|12|13|三班|
|3|王五|13|13|三班|
|4|马六|15|13|三班|
|1|张三|11|14|四班|
|2|李四|12|14|四班|
|3|王五|13|14|四班|
|4|马六|15|14|四班|

**总结：相当与笛卡尔积，左表和右表组合。**
### 有where子句
&ensp;&emsp;&emsp;往往会先生成两个表行数乘积的数据表，然后才根据where条件从中选择。
<span style="font-size:18px;"><span style="font-family:System;"> 
select * from T_student s cross join T_class c where s.classId = c.classId </span></span>   
　  (注:cross join后加条件只能用where,不能用on)  

**总结：查询结果跟等值连接的查询结果是一样**

# 范式
&ensp;&emsp;&emsp;在设计与操作维护数据库时，最关键的问题就是要确保数据能够正确地分布到数据库的表中。使用正确的数据结构，不仅有助于对数据库进行相应的存取操作，还可以极大地简化应用程序中的其他内容（查询、窗体、报表、代码等），按照“数据库规范化”对表进行设计，**其目的就是减少数据库中的冗余，以增加数据的一致性**。
&ensp;&emsp;&emsp;**范化**是在识别数据库中的数据元素、关系以及定义所需的表和各表中的项目这些初始工作之后的一个细化的过程。常见的范式有**1NF**、**2NF**、**3NF**、**BCNF**以及**4NF**。
## 1NF（第一范式）
第一范式：当关系模式R的所有属性都**不能在分解为更基本的数据单位**时，称R是满足第一范式的，简记为1NF。满足第一范式是关系模式规范化的最低要求，否则，将有很多基本操作在这样的关系模式中实现不了。

## 2NF（第二范式）
第二范式：如果关系模式R满足第一范式，并且**R的所有非主属性都完全依赖于R的每一个候选关键属性**，称R满足第二范式，简记为2NF。

## 3NF（第三范式）
第三范式：设R是一个满足第一范式条件的关系模式，**X是R的任意属性集，如果X非传递依赖于R的任意一个候选关键字**，称R满足第三范式，简记为3NF.

## BCNF（巴斯-科德范式）
巴斯-科德范式：构建在第三范式的基础上，如果关系模式R是第一范式，且每个属性都不传递依赖于R的候选键，那么称R为BCNF的模式。

## 4NF（第四范式）
第四范式：设R是一个关系模式，D是R上的多值依赖集合。如果D中存在凡多值依赖X->Y,X必是R的超键，那么称R是第四范式的模式。
　  




