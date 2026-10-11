
# 连接MySQL

![[Pasted image 20261010115555.png]]

**操作数据库就是使用客户端链接服务端，通过标准的SQL语言完成交互**

命令行中mysql -uroot -p进入数据库，进入后说明客户端已经链接到服务端

命令行只是客户端的最原始的一种，MySQL有很多可视化客户端工具方便使用。如Navicat

![[Pasted image 20261010122934.png]]

# SQL语言规范

结构化查询语言（Structured Query Language）简称SQL，是一种数据库查询和程序设计语言，用于存取数据以及查询、更新和管理关系数据库系统。

## SQL能做什么？
1. SQL面向数据库执行查询；
2. 在数据库中插入新的记录；
3. 更新数据库中的数据；
4. 从数据库删除记录；
5. 创建新数据库；
6. 在数据库中创建表；
7. 在数据库中创建存储过程；
8. 在数据库中创建视图；
9. 设置表、存储过程和视图的权限

## SQL语言分类

### 数据查询语言DQL
1. SELECT
2. FROM
3. WHERE
4. ORDER BY
5. HAVING

### 数据操作语言DML  针对表中的数据
1. INSERT 添加数据
2. UPDATE 更新数据
3. DELETE 删除数据

### 数据定义语言DDL   针对数据库对象：库Database、表table、索引、index、视图view、存储过程procedure、触发器trigger
1. CREATE 创建数据库对象
2. ALTER 修改数据库对象
3. DROP 删除数据库对象
4. MODIFY

### 数据控制语言DCL
1. GRANT 授权用户某种权限
2. REVOKE 回收授予的某种权限

### 事务控制语言TCL
1. COMMIT 提交事务
2. ROLLBACK 回滚事务
3. SAVEPOINT 设置回滚点


## 创建与删除数据库

### 创建数据库

CREATE DATABASE ——创建数据库关键字
test—————————数据库名称
DEFAULT CHARACTER SET———指定默认编码类型关键字
utf8—————————指定编码

1. 使用DDL语句创建数据库：
**CREATE DATABASE 数据库名 DEFAULT CHARACTER SET 字符编码**；

2. 查看数据库：
**show databases；**

3. 查看数据库编码：
**select schema_name,default_character_set_name from information_schema.schemata**
 **where schema_name = 'test1';**

### 删除数据库

DROP DATABASE——删除数据库关键字
test————————数据库名称

1. 使用DDL语言删除数据库：drop database 数据库名称；

### 选择数据库

use 数据库名；

## 创建与删除表

### 创建表

CREATE TABLE——创建表关键字

**使用DDL语句创建表：CREATE TABLE 表名（列名 类型，列名 类型，。。。）**

**show  tables**——查看已创建的表


### 删除表

**使用DDL语句删除表：DROP TABLE 表名**

### 修改表

navicat直接右键设计表，修改后保存；

1. **修改表名**： **ALTER TABLE 旧表名 RENAME 新表名**；
2. **修改列名： ALTER TABLE 表名 CHANGE COLUMN 旧列名 新列名 类型**；
3. **修改列类型：ALTER TABLE 表名 MODIFY 列名 新类型**；
4. **添加新列：ALTER TABLE 表名 ADD COLUMN 新列名 类型**；
5. **删除指定列：ALTER TABLE 表名 DROP COLUMN 列名**；


# MySQL中的约束

**数据库约束是对表中的数据进行进一步的限制，保证数据的正确性、有效性和完整性。**
## 主键约束 Primary KEY  （PK）

主建约束是表的一个特殊字段，一般情况表中都会有一个主键；

1. **不允许为空；**
2. **不允许有重复值出现；**
3. **保证数据唯一性；**

**存在两种主键类型：**
1. **单一主键**：使用一个列作为主键列，当该列的值有重复时，则违反唯一约束。
2. **联合主键**：使用多个列作为主键列，当多个列的值都相同时，则违反唯一约束。
比如A、B两个列作为主键列：
![[Pasted image 20261010174828.png]]
第一列和第四列数据都重复时才违反唯一约束。


**添加主建约束：**
使用DDL语句：  **ALTER TABLE 表名 ADD PRIMARY KEY(列名）**

**主键具备自动增长能力**（oricle没有此能力）
MySQL中的自动增长类型要求：
1. 一个表中只能有一个列为自动增长（因此联合主键不行）；
2. 自动增长的列的类型必须是整数类型；
3. 自动增长只能添加到具备主建约束与唯一性约束的列上；
4. 删除主键约束或唯一性约束，如果该列拥有自动增长能力，则需要先去掉自动增长然后再删除约束；
**主键设置自增**： **ALTER TABLE 表名 MODIFY 主键列名 类型 auto_increment**；

**删除主键约束**：ALTER TABLE 表名 DROP PRIMARY KEY；
==🔴注：删除主键时，如果主键具备自动增长能力，需要先去掉自动增长，然后再删除主键==
先去掉自动增长：**alter table 表名 modify 主键列名 主键类型**；


## 外键约束 Foreign KEY   （FK）

当前表中有列必须
1. **允许有空值；**
2. **允许有重复；**
3. **值必须是参照表的参照列中所包含的值；**（参照某个表的某个列，且这个列还必须是主键列）
4. **保证数据的参照完整性；**

eg：两个表，一个商品表一个订单表，商品表中的每个商品都有对应的订单号，订单号就是外键约束，对应订单表中id主键；

![[Pasted image 20261011093544.png]]

**添加外键约束**：
DDL语句： **ALTER TABLE 表名 ADD CONSTRAINT 约束名 FOREIGN KEY(列名) REFERENCES 参照的表名(参照的列名);**

**删除外键约束**：
**ALTER TABLE 表名 DROP FOREIGN KEY 约束名**；

## 唯一性约束Unique

1. **相同值只能出现一次；**
2. **允许为多个列添加唯一性约束；**
3. **保证数据的唯一性**

**添加唯一性约束**：
**ALTER TABLE 表名 ADD CONSTRAINT 约束名 UNIQUE(列名)**

navicat中直接**添加索引**，类型选择unique即可

**删除唯一性约束**：
**ALTER TABLE 表名 DROP KEY 约束名**

## 非空约束 Not Null

1. **列中不能有空值；**
2. **允许重复值；**
3. **允许为多个类添加非空约束；**
4. **保证数据没有空值；**

**添加非空约束**：
**ALTER TABLE 表名 MODIFY 列名 类型 NOT NULL**；

删除非空约束：
**ALTER TABLE 表名 MODIFY 列名 类型 NULL**；

## 检查约束 Check

1. **用户自定义约束条件；**
2. **保证数据满足自定义条件约束；**
3. **MySQL目前不支持检查约束**

## 创建表的时添加约束

**查询表中的约束信息**：**SHOW KEYS FROM 表名**

一般会在创建表时添加主键、自动增长、唯一性约束、非空约束等。如果要创建外键约束必须保证外键的另外一个表存在。

创建depts表包含department_id该列为**主键且自动增长**，department_name**不允许重复**，location_id列**不允许空值**；
**create table depts(department_id int primary key auto_increment,** 
**department_name varchar(20) unique,** 
**location_id int not null);**

# MySQL数据操作DML

DML语言是**操作数据**的语言：insert添加数据；update更新数据；delete删除数据

## 添加数据

### 选择插入

根据自己的需要向**指定列**中插入数据；

**选择插入的DML语言**：
**INSERT INTO 表名(列名1，列名2，列名3...) VALUES(值1，值2，值3...);**
==🔴注意：如果某列设置了非空约束，则在前面列名中必须包含，且values中也必须有值；==


### 完全插入

向表中的**所有列**都插入数据

DML语言中在表名后不需要指定列名：**INSERT INTO 表名 VALUES(值1， 值2， 值3...);**
==🔴注意：如果主键是自动增长，需要使用default或者null或者0占位。==

### 默认值处理DEFAULT

在MySQL中可以使用DEFAULT为列设定一个默认值。如果在插入数据时并未指定该列的值，那么MySQL会将默认值添加到该列中。

**创建表时就可以指定列的默认值**：create table 表名(列名 类型 default 默认值,.....);
**新增列时也可以指定默认值**：alter table 表名 ADD column varchar(20) default "unknown";

==🔴注意：完全插入时对于设置了默认值的列数据必须使用default占位。==

### 更新数据

**UPDATE 表名 SET 列名=值，列名=值 WHERE 条件**；
==🔴注意：更新语句中一定要给定更新条件，否则(不写where)表中的所有数据都会被更新==；