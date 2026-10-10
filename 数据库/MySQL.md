
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

主建约束是表的一个特殊字段，字段是唯一标识，不能重复；

1. **不允许为空；**
2. **不允许有重复值出现；**
3. **保证数据唯一性；**


## 外键约束 Foreign KEY   （FK）

1. **允许有空值；**
2. **允许有重复；**
3. **值必须是参照表的参照列中所包含的值；**（参照某个表的某个列，且这个列还必须是主键列）
4. **保证数据的参照完整性；**

eg：两个表，一个商品表一个订单表，商品表中的每个商品都有对应的订单号，订单号就是外键约束，对应订单表中id主键；

## 唯一性约束Unique

1. **相同值只能出现一次；**
2. **允许为多个列添加唯一性约束；**
3. **保证数据的唯一性**


## 非空约束 Not Null

1. 列中不能有
