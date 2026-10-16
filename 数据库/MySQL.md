
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

## 添加数据INSERT INTO

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

## 更新数据UPDATE

**UPDATE 表名 SET 列名=值，列名=值 WHERE 条件**；
==🔴注意：更新语句中一定要给定更新条件，否则(不写where)表中的所有数据都会被更新==；


## 删除数据DELETE

**DELETE FROM 表名 WHERE 条件**；
==🔴注意：如果没有给定条件则会删除表中所有数据==；

除过使用delete from不添加条件来清空表之外，也可以使用DML语句：**TRUNCATE TABLE 表名**

### 清空表时delete和truncate区别
1. truncate是整体删除（速度较快），delete是逐条删除（速度较慢）；
2. truncate不写服务器日志log，delete写服务器日志log，也就是truncate效率比delete高的原因；
3. truncate是会重置自增值的，delete不会重置，自增值依然累加。




# MySQL查询数据DQL

## SELECT基本查询

**select语句从数据库中返回信息**。使用一个SELECT语句，可以完成下面操作：
1. **列选择**：能够使用select语句的列选择功能选择表中的列，返回列中的数据；
2. **行选择**：使用不同的标准限制看见的行；
3. **连接**：使用select语句的连接功能来集合被存储到不同表中的数据；

**基本语法**：
	**SELECT 确定列**
	**FROM 确定表**
	
**基本SELECT语句**：
	在最简单的形式中，SELECT语句必须包含下面的内容：
	1. 一个SELECT子句，指定被显示的列
	2. 一个FROM子句，指定表，该表包含SELECT子句中的字段。

| 语句                   | 含义          |
| -------------------- | ----------- |
| SELECT               | 一个或多个字段的列表  |
| *                    | 选择所有列       |
| DISTINCT             | 禁止重复        |
| column \| expression | 选择指定的字段或表达式 |
| alias                | 给所选择的列不同的标题 |
| FROM table           | 指定包含列的表     |

![[Pasted image 20261014113818.png]]

![[Pasted image 20261014113755.png]]


## 查询中的算式表达式

![[Pasted image 20261014114337.png]]

需要修改数据显示方式，如执行计算，或者作假定推测，这些都可能用到算术表达式。一个算术表达式可以包含列名、固定的数字值和算术运算符。

### 使用算术运算符

![[Pasted image 20261014114626.png]]


### MySQL的定义空值

![[Pasted image 20261014115838.png]]

如果一行中的某个列缺少数据值，该值被置为null，或者表达为包含一个空；
空null不是0或空格，0是一个数字，空格是一个字符。

### 算术表达式中的空值

![[Pasted image 20261014120711.png]]如果列里的值为空，则计算结果也为空

### MySQL中的别名

查询时可以为**列**起别名 关键字：**AS**（也可以省略，直接在列名后空格再写别名）

![[Pasted image 20261014120942.png]]


**表**别名：
**SELECT 表别名.列名 FROM 表名 AS(可省略) 表别名；**


### MySQL中去除重复

![[Pasted image 20261014123424.png]]

**除去相同行**关键字：**DISTINCT**，放在列名前面select后面
==🔴**注意：如果同时在查询别的列，此时distinct只会剔除所有列都相同的行**==


### 查询中的行选择

![[Pasted image 20261014125739.png]]


用**WHERE子句**限制从查询返回的行。一个WHERE子句包含一个必须满足的条件；
**WHERE子句紧跟FROM子句**。如果条件是true，返回满足条件的行。
条件可以是：**列名、表达式、常数、比较操作**；

**SELECT 列(或者*) FROM 表名 WHERE 选择条件**；


### MySQL中的比较条件

![[Pasted image 20261014151756.png]]


!=也表示不等于；

![[Pasted image 20261014152315.png]]



**查找范围区间**用between and。

![[Pasted image 20261014152415.png]]包含上下限。



**查找符合指定条件列表（集合）中的值**：
![[Pasted image 20261014152650.png]]



**查找字符串（模糊查询）**：

![[Pasted image 20261014153051.png]]



**查找空**：
![[Pasted image 20261014153402.png]]
NULL条件，包含IS NULL条件和IS NOT NULL条件。

IS NULL条件用于空值测试。空值的意思是难以获得的、未指定的、未知的或不适用的。



### MySQL中的逻辑条件

![[Pasted image 20261014153739.png]]

and是并且，or表示或，not表示取反
**多条件之间的逻辑关系**


### 条件中的优先规则

![[Pasted image 20261014154330.png]]


## ORDER BY 排序

![[Pasted image 20261014155611.png]]

对不明确的查询结果排序返回行。ORDER BY子句必须位于SQL语句最后。

**SELECT语句的执行顺序如下**：
1. FROM子句；先找表，定位表
2. WHERE子句；表里面哪些数据，定位行
3. SELECT子句；投影哪些列，定位列
4. ORDER BY子句；执行排序

可以进行**多列排序**，比如值相同的列则用另一列辅助排序，比如下面例子，用department升序排，如果department相同则用salary降序排；
![[Pasted image 20261014160330.png]]


==🔴注意：按某列来排序也可以不用select显示该列。==


## 多表查询

![[Pasted image 20261015145957.png]]

从多个表中查询数据结果集

![[Pasted image 20261015150034.png]]当一个连接条件无效或被遗漏时，其结果是一个笛卡尔乘积，其中所有行的组合都被显示。第一个表中的所有行连接到第二个表中的所有行。一个笛卡尔乘积会产生大量的行，其结果没有什么用。

多表查询分类：
1. sql92标准：内连接（等值连接，非等值连接，自连接）；
2. sql99标准：内连接、外连接（左外、右外）、交叉连接；


### SQL92标准的查询

#### 1.等值连接

![[Pasted image 20261015151350.png]]
两个表中存储的**数据列名**以及**值**相同，以列中值的**相等性**来连接。不相等的检索不出来，相等的检索出。常用场景为（**主键和另外一个表的外键**）

等值连接特点：
1. 多表等值连接的结果为多表的交集部分；
2. n表连接，至少需要n-1个连接条件；
3. 多表不分主次，没有顺序要求；
4. 一般为表起别名，提高阅读性和性能；
5. 可以搭配排序、分组、筛选。。等子句使用；
==🔴注意：等值连接也叫简单连接或内连接==

示例：
![[Pasted image 20261015152007.png]]

**SELECT**子句指定要返回的列名（需要根据表名来指定列名）；
**FROM**子句指定数据库必须访问的表名；
**WHERE**子句指定表怎样被连接；

**增加搜索条件**：
![[Pasted image 20261015152547.png]]
**使用表别名**：
![[Pasted image 20261015152811.png]]

**多表连接**：
![[Pasted image 20261015152943.png]]


#### 2.非等值连接
![[Pasted image 20261015154308.png]]例子表示为，salary列数据在第二个表中哪一个区间，代表哪一个等级
![[Pasted image 20261015154618.png]]

#### 3.自连接
![[Pasted image 20261015160435.png]]

employees表中既包含员工数据也包含经理数据，可以通过manager_id来区分谁是谁的经理；
**用同一个表的不同的列作为查询连接的条件**；
![[Pasted image 20261015160800.png]]


### SQL99标准中的查询

#### 1.交叉连接
![[Pasted image 20261015165315.png]]

等价于92中的不写任何条件的连接；
**两个表的交叉连接**：select * from employees cross join departments;
==🔴很少使用==。

#### 2.自然连接
![[Pasted image 20261015165642.png]]**自然连接**：连接只能发生在两个表中有**相同名字**和**数据类型**的列上。如果列有相同名字，但是数据类型不同，NATURAL JOIN语法会引起错误。
比如AB两个表，会将两个表中列名和数据类型相同的列**等值连接**，多个列的话会增加连接条件and连接。

![[Pasted image 20261015170023.png]]例如：location_id列在两个表中**名称和数据类型**相同。如果存在其他的同名同类型的列，自然连接会使用**等值连接**的方式连接，连接条件的关系为and。


#### 3.内连接INNER JOIN

![[Pasted image 20261015171455.png]]

语法：
**1.select 查询列表；**
**2.from 表 别名；**
**3.inner join 链接表（放在from中的两个表之间）；**
**4.on 连接条件**

**用ON子句指定连接条件**：


![[Pasted image 20261015194541.png]]


**用ON指定更多连接条件**：
![[Pasted image 20261015194902.png]]



#### 4.外连接查询

![[Pasted image 20261015195507.png]]

**孤儿数据**：指被连接的列的值为空的数据（外连接中不匹配的数据）；



**左外连接**：左边的表中即使没有与目标表中的匹配行，该查询也会取出左表中的行。
**左表名 LEFT OUTER JOIN 右表名**
![[Pasted image 20261016160223.png]]

employees去连接departments，并且左外连接：
![[Pasted image 20261016160251.png]]

对比：
正常内连接：![[Pasted image 20261016160853.png]]
其中以department_id列为连接条件，下面使用左外连接时会将employees表中的department_id列为空的行也显示出来。

![[Pasted image 20261016161014.png]]



**右外连接**：
RIGHT OUTER JOIN![[Pasted image 20261016161446.png]]右外连接以department_id为连接条件查询，因为employees表中的department_id只有1-6，而departments表中department_id是1-20，因此多了14条孤儿数据。![[Pasted image 20261016161740.png]]



**全外连接**：FULL OUTER JOIN连接（MySQL不支持FULL OUTER JOIN可以使用union实现）
除了返回满足连接条件的数据外，还会将两个表的孤儿数据都显示出来
![[Pasted image 20261016162000.png]]



**UNION**：可以将两个查询结果集合并，返回的行都是唯一的，如同对整个结果集合使用distinct
**UNION ALL**：相较于上面UNION是包含了重复数据的。


# 多表查询练习：
![[Pasted image 20261016162721.png]]
**分析**：
来源于两张表，等值连接即可，employees表中的所有雇员都有department_id，==🔴如果存在员工没有department_id，需要考虑用左外连接查询==。
![[Pasted image 20261016163059.png]]


![[Pasted image 20261016163107.png]]

