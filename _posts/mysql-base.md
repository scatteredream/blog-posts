---
name: mysql-base
title: MySQL 基础
date: 2024-09-30
tags: 
- mysql
- 事务
categories: mysql
---

## 基本概念

**DB**: organized data

**DBMS**: manage system

**RDBMS**: SQLite PostgreSQL MySQL Oracle Microsoft SQL Server (relational)

**SQL**: programming language

**数据模型**：管理系统：数据库：表：数据

**RDBMS**: 表结构格式统一

![image-20240920142153024](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240920142153024.png)

## 约束

![image-20240923202742353](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923202742353.png)

auto_increment 从零开始自动增长 (列是数字的类型，并且是UNIQUE约束)

 这一列不指定id，或者id赋值null 不影响自增

### 非空，唯一，主键，默认，检查约束

![image-20240923220639875](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923220639875.png)

![image-20240923220734860](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923220734860.png)

![image-20240923220750052](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923220750052.png)

![image-20240923220711866](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923220711866.png)

![image-20241005154832392](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005154832392.png)

### 外键约束

数据一致性 完整性

外键：连接两个表的数据

- （**constraint** foreKeyCons）**foreign key**(dept_id) **references** dept(id) 建表时
- 给当前表中的dept_id列定义了一个 名为 foreKeyCons 的外键约束，外键引用dept表的id列，dept_id就是外键
- dept就是主表，必须存在，并且id是主键
- 因为有外键约束，所以不能直接删除主表的内容，

建表以后对外键的操作：添加和删除。外键属于表的属性

- **alter table** emp **drop foreign key** foreKeyCons
- **alter table** emp **add constraint** foreKeyCons **foreign key**(dept_id) **references** dept(id)

#### 外键的删除、更新行为

on update 更新   on delete 删除

![image-20241005155438576](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005155438576.png)

![image-20241005155543394](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005155543394.png)

NOACTION RESTRICT 默认

CASCADE 主表变了，子表跟着变，主表删了，子表跟着没

**ON DELETE** SET NULL 主表删了，子表把对应的值设为null，只支持删除操作，并且要求外键可以为null





## 数据库设计

![image-20240923224354382](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923224354382.png)

![image-20240923224409160](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923224409160.png)

![image-20240923224759437](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923224759437.png)

1-M 多的一方建立外键，少的一方作为主表，连接起来

![image-20240923225452261](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923225452261.png)

M-N 一起连到一张中间表，中间表做从表，建立外键，连到两张表的主键上

![image-20240923225514320](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240923225514320.png)

一个商品可能属于不同订单，一个订单也可能有不同商品，所以用类似坐标的方式

## 事务

管理DML语句，InnoDB引擎才支持事务

MySQL默认设置事务自动提交，也就是执行完自动COMMIT.

如果要显式开启事务要SET AUTOCOMMIT = 0;（以后的事务都需要手动提交）

或者 START Transaction /  BEGIN （临时开启一条事务）

COMMIT 提交事务 如果没有问题就提交

ROLLBACK 回滚事务 出现异常就要回滚事务到BEGIN处，即更改前

### 四大特征 acid

一致性依赖于应用层，开发者。

**A**tomic **C**onsistency **I**solation **D**urability

最小操作单位，不可分割；

完成时必须让所有数据都前后一致，由开发者指定，比如转账，金钱总额不能变化；

多个事务是互相隔离的，排除其他事务对本事务的影响（解决并发问题）；

事务对数据库的修改是持久的；

一什么是隔离性？

隔离性是数据库事务的四个基本属性之一，即 **ACID** 特性中的 “I”（Isolation）。隔离性保证了一个事务在未完成之前，它的操作对其他事务是不可见的，或者说部分可见（取决于隔离级别）。这样可以防止因并发执行而导致的数据问题。

### 高并发下可能遇到的问题：

1. **脏读**（Dirty Read）：一个事务读取了另一个事务**开始了但未提交**的数据。如果另一个事务回滚或者提交，这些数据将无效，导致第一个事务读取了错误数据。
   1. ![image-20241005213034469](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005213034469.png)

2. **不可重复读**（Non-repeatable Read）：在同一个事务中，前后两次读取相同的数据时，数据值发生了变化，因为另一个事务在两次读取之间**修改并提交了该数据**。
   1. ![image-20241005213018573](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005213018573.png)

3. **幻读**（Phantom Read）：一个事务内，连续两次执行相同的查询（select）时，再次进行查询的时候真实的数据集已经发生了变化，但是A却查询不出来这种变化，因此产生了幻读。
   1. 如果已经解决了12，将会发现：第一次查询没有结果，随后另一个事务插入数据并进行了提交，试图插入数据，会报错，提示不能有重复的主键，但是在第二次查询仍然没有结果，读不到别人已经提交的数据（repeatable read）但是别人提交的数据还在影响。
   2. ![image-20241005214729161](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005214729161.png)


### 通过隔离性解决高并发问题：

数据库系统通常提供多种**事务隔离级别**，每个级别可以解决一部分或全部并发问题。这些隔离级别定义了事务之间可见性规则，数据库可以根据应用场景选择适当的隔离级别来权衡性能与数据一致性。

`select @@transaction_isolation` 查看当前的隔离级别

`set session transaction isolation level read uncommitted`设置当前会话的隔离级别，

### SQL 标准定义的四种隔离级别

1. **读未提交（Read Uncommitted）**：
   - **脏读**、**不可重复读** 和 **幻读** 都可能发生。
   - 最低的隔离级别，事务可以读取未提交的数据。
   - 使用场景：极高并发要求且对数据一致性要求不高的场景。
2. **读已提交（Read Committed）**：Oracle 默认
   - 解决脏读问题，但仍然可能出现不可重复读和幻读。
   - 一个事务只能读取已提交的数据，保证不会读取到未提交的修改。
   - 使用场景：大多数数据库系统的默认隔离级别，较好的性能和数据一致性的平衡。
3. **可重复读（Repeatable Read）**：MySQL 默认
   - 解决脏读和不可重复读问题，但幻读仍然可能发生。
   - 在同一事务中，事务两次读取相同的数据，保证两次读取的结果一定相同。不会读取其他事务已经提交的数据，但是仍然无法避免其他事务的影响（比如重复插入相同主键失败但是查不到这条主键的数据）
   - 使用场景：需要保证数据一致性、避免更新数据不一致的场景。
   - 普通的select快照读不会受到其他事务update、insert的影响，但是自己执行update时会进行当前读，会把其他事务update、insert的数据更新成自己的版本号，下一次读取就会读到了。
   - 幻读：尽管B事务在A事务还未结束的时候，增加了表中的数据，但是为了维护可重复读，A事务中不管怎么查询，是查询不到新增的数据的。但是对于真实的表而言，表中的数据是的的确确增加了，所以插入重复的主键会报错。
4. **串行化（Serializable）**：
   - 解决脏读、不可重复读和幻读问题。
   - 最高的隔离级别，所有事务串行执行，完全避免并发导致的数据问题。要等先开始的事务执行完提交或者回滚，后开始的事务才能开始执行，完全放弃并发性。
   - 使用场景：极端数据一致性要求的场景，但代价是性能较低，容易出现锁等待甚至死锁。

### 隔离性和并发控制的关系

- **锁机制**：隔离性通常通过锁机制（例如行锁、表锁）实现。在高隔离级别下（如可串行化），数据库会使用更严格的锁定策略，确保其他事务在未提交前不能读取或修改被锁定的数据。
- **多版本并发控制 (MVCC)**：有些数据库（如 PostgreSQL、MySQL 的 InnoDB 存储引擎）采用了多版本并发控制，允许在较高隔离级别下提高性能。MVCC 通过保存数据的多个版本，允许读取操作无需阻塞写入操作，从而在高并发下仍然能够提供一致的数据读取。

## SQL

### 通用语法

- 可以多行书写，分号结尾
- 可用空格和缩进增强可读性
- MySQL的SQL语句不区分大小写，关键字建议大写

### 分类

DDL definition 定义数据库对象(表、db，字段)

DML manipulation 操作数据，增删改

DQL query 查询

DCL control 创建用户，控制访问权限

### DCL

control 控制数据库访问权限和管理数据库用户

#### DCL-用户管理

use mysql;

select * from user;

- 创建用户 **create user** 'itcast'**@**'localhost' **identified by** '123456' 密码123456用户名itcast 主机localhost
- 创建用户 **create user** 'heima'**@**'%' **identified by** '123456' 密码123456用户名heima  任意主机均可访问
- 改密码 alter user 'heima'**@**'%' **identified with** mysql_native_password by '1234' 改成1234

- drop user 'heima'**@**'%'  删除用户

#### DCL - 控制权限

- 查询有什么权限：SHOW GRANTS FOR 用户@主机
- 授予用户权限：grant all on 数据库名.表名 to 用户@主机
- 撤销用户权限：revoke all on 数据库名.表名 from 用户@主机

### DDL 

#### 数据库操作-database

- `SHOW DATABASES;` 查询所有数据库（展示）
- `SELECT DATABASE();` 查询当前数据库（展示）
- `CREATE DATABASE if not exists name;`如果不存在则创建一个名为 name 的数据库，后面可以加`default charset + 字符集` `COLLATE 排序规则`
- `DROP DATABASE IF EXISTS;` 如果存在则删除
- `USE name;` 使用名为name的数据库

####  表操作-查询 table

- `SHOW TABLES; ` 
- `DESC 表名;` 查询表结构
- `SHOW CREATE TABLE 表名;` 查询建表时候的信息

#### 表操作-创建 table

```sql
  use itcast;
  create table tb_user(
      id int comment '编号',
      name varchar(50) comment '姓名',
      age int comment '年龄',
      gender varchar(1) comment '性别'
      ) comment '用户表';
  desc tb_user;
```

#### 表操作- 数据类型

##### 数值类型：

TINYINT-byte   SMALLINT-short MEDIUMINT-3 bytes INT/INTEGER-int 

BIGINT-long  FLOAT DOUBLE DECIMAL 

TINYINT UNSIGNED(0-255) 无符号的tinyint

DOUBLE(4,1)4代表总位数 1代表小数部分的位数 

##### 字符串类型：

![image-20240920183842122](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240920183842122.png)

|          | char(10)     | varchar(10) |
| -------- | ------------ | ----------- |
| 最小长度 | 10           | 0           |
| 最大长度 | 10           | 10          |
| eg       | 性别、手机号 | 用户名      |

##### 日期类型

![image-20240920184425055](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240920184425055.png)

![image-20240926153912988](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926153912988.png)

![image-20240926153925471](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240926153925471.png)





#### 表操作-修改表属性

**alter table** employee **add** nickname varchar(20) [comment] [约束]; **添加属性**

**alter table** employee **modify** nickname char(20); **修改数据类型**

**alter table** employee **change** nickname idCard char(18) [comment] [约束]; **改名字+数据类型** 

**alter table** employee **drop** nickname; **删除** 

**alter table** employee **rename to** emp; **改表名**  

#### 表操作-删除

**drop table** [if exists] 表名;   **删除整个表**

**truncate table** 表名; **删除数据不删表结构**

#### MySQL GUI

SQLyog Navicat DataGrip

### DML

INSERT UPDATE DELETE 

#### 添加数据

**insert into** 表名(属性1，属性2....) **values**（v1,v2...）指定属性

**insert into** 表名 **values**（v1,v2...）所有

字段和值一一对应，字符串和日期在单引号中

**多条数据**: **values**

​		（v1,v2...）,

​		(v1,v2....) 不同条数据用逗号隔开

#### 修改数据

**update** employee **set** name = 'itheima' **where** id=1;

#### 删除数据

**delete from** 表名 [where 条件] 无条件会删除整张表格的数据

### DQL

SELECT 

#### 基本查询

##### 查询多个字段 

**select** 字段1，字段2........ **from** 表名

**select * from** 表名       通配符

##### 设置别名

**select** 字段1[**as** 别名1]，字段2[**as** 别名2].... **from** 表名 as可省略

##### 去重

 **select distinct** ........

#### 条件查询 WHERE

![image-20240920211218098](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240920211218098.png)

**select * from** employ where age **in**(12,45,43);

**select * from** employ where name **like '__'**; 名字是两个字符  模糊查询 模糊匹配

##### sql 模糊查询

通配符：

- %: %网% 查询含有网字的数据

  ​     %网    查询以网字结尾的数据

  ​      %网%车    查询含有 网 和 车的数据 有先后顺序

- _ :   网_ 网开头 长度为2个字

  ​	_ _ 网 长度为3个字 最后一个字是网



between and 

in 

is null 

#### 聚合函数 

![image-20240920212134476](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20240920212134476.png)

count(*)所有行 count(age)age非空的行数

作用在列数据，**null不参与运算~**

select count(age) from emp where workAddress = '西安'; 

位于select关键字之后 可以加where条件

#### 分组查询 GROUP BY & HAVING

group by xxx 将具有相同xxx值的行归为一组，每一组执行聚合count  



**select** gender, count(*) **from** emp **<u>group by</u>** gender;  计算行的数量，并按照gender分组

- where 分组之前过滤 
- **having** 对分组之后的结果进行过滤
- where 不能对聚合函数判断， having可以，要判断聚合函数就要分完组
- 执行顺序：where>聚合函数>having
- 先用where筛选个体，然后分组，再对组内执行聚合函数，执行的结果可以再用having筛选



#### 排序查询 ORDER BY

ASC:ascending 默认

DESC:descending

- 语句的最后，添加 **order by** age **asc**;
- **order by** age **desc** , entryDate; 先按照age降序，如果age相同在按照entryDate升序

#### 分页查询 LIMIT

查询xx页码

LIMIT 起始索引，查询记录数

每页是20条，第一页的起始数据是0，第二页的20

limit 0,5 第一页，一页5条记录

**limit** 5 **offset** 0 第一页 limit 5 offset 20 第五页

#### 执行顺序

先执行from 再用where过滤，然后用group by和having指定分组以及过滤，然后执行字段的select，接着是排序，最后分页

在字段名，表名后加别名，看是否报错，验证上述顺序

### 多表查询

#### 表关系

多对多：同一个学生可以选择多门课程，同一个课程也可以被多名学生选择，如果要描述他们之间的关系，添加一张中间表，里面的外键分别对应两张表的主键。

一对多：多的一方建立外键，关联到少的主键。

一对一：任意一方建立外键（UNIQUE），关联到另外的主键。

![image-20241005161911780](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005161911780.png)

#### 链接查询 JOIN

##### 消除笛卡尔积

![image-20241005162327168](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005162327168.png)

##### 内连接

只查询A和B相交的部分

**select** *e*.name,*e*.gender,*d*.dname **from** *emp e*,*dept d* ... **where**    条件 (隐式) 

**select** * **from** *emp* **join** *dept* on     条件（显式）

##### 外连接

![image-20241005163112728](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005163112728.png)

e.* emp的全部数据 

from *A* left join B 查询*A* ∪ (A∩B)

from *A* right join B 查询*B* ∪ (A∩B)

##### 自链接

一定要起别名

![image-20241005163553452](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005163553452.png)

没有必要专门搞一张领导表出来，如果要实现需求就要把一张表分成两张看，a的经理id等于b的id

![image-20241005163805094](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005163805094.png)

##### 外键？连接查询？

在多表查询中，**连接查询**（JOIN）并不强制要求外键的存在。外键的作用是维护表之间的**参照完整性**，确保一张表中的某些值对应另一张表中的有效值。但在进行连接查询时，只要有可以用于关联两个表的字段（如主键和某个相应的列），就可以进行查询，而不需要一定设置外键。

#### 联合查询

把两条单表查询结果联合起来，字段数量必须一致

UNION：自动根据主键去重

UNION ALL: 不去重

#### 子查询 按返回值分类

##### **单行单列**：只返回一个**值** 

- ![image-20241005165024271](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005165024271.png)

##### **单行多列**：返回的是一行数据

- ![image-20241005S171121759](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005171121759.png)![image-20241005171132602](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005171132602.png)

- 第一步返回的是**一行两列**的数据（salary，managerid），第二步的条件可以用where (salary, managerid) = 第一行的结果

- <a href="#extension">多行多列扩展</a>

##### **多行单列**：子查询返回的是多行单列的数据

也就是同一个字段的值的集合，用括号括起来，最后查询可以如下操作符（操作对象是同一字段的值的集合）

- in **等于**集合内部的某一个值

- any/some **大于** any 只需大于最小值 小于any只需小于最大值**相当于存在量词∃** 
- all **相当于全称量词∀**

- ![image-20241005165513314](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005165513314.png)
- ![image-20241005165204726](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005165204726.png)

##### **多行多列**： 虚拟表

子查询返回的是多行多列数据，也就是虚拟表，从这张虚拟表中再和其他表进行连接查询。

- **虚拟表**: 查询入职日期 在2011-11-11之后的员工信息和部门信息：先查入职在2011-11-11之后的员工信息，根据这些信息（虚拟表,也叫临时表）和部门表 连接查询


```sql
SELECT * FROM (select * from emp where join_date>'2011-11-11')e JOIN dept on e.dep_id = dept.id
```

- <span id="extension">**也可以作为单行多列的扩展**</span> in 关键字

  - 查询第一步返回的是**多行多列**的数据（salary，managerid），第二步的条件可以用where (salary, managerid) in 第一行的结果



## JDBC  

## 函数

可以直接被另一段程序调用的程序或代码-------常见MySQL内置函数

### 字符串函数

concat substring TRIM upper 

LPAD(STR, 6, '0') 左对齐，6位补0

![image-20241005154345722](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005154345722.png)

### 数值函数

ceil floor round rand（0-1随机数）mod

![image-20241005154156390](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005154156390.png)

### 日期函数

date_add now curdate curtime datediff

![image-20241005154146380](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005154146380.png)

### 流程控制

![image-20241005153559225](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005153559225.png)

`if(expr, caseTrue, caseFalse) expr = true` 返回caseTrue

`ifnull(v1,v2)` v1=null 返回v2 空字符串不是null，null必须是什么都没有，v1不为null，则返回v1

case when

![image-20241005153230584](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005153230584.png)

![image-20241005153515232](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241005153515232.png)



















