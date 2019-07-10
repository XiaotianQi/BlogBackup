## 常见字段类型

* 数字类型：int（4 bytes）、float（4 bytes）、double（8 bytes）、decimal（可指定小数长度）
* 字符串类型：char（定长字符串 0-255）、varchar（变长字符串 0-255）、text（长文本数据）、longtext（极长文本数据）
* 日期：date、time、datetime、timestamp

## 字段约束

* 主键约束：primary key
* 非空约束：not null
* 唯一约束：unique
* 外键约束：foreign key
* CHECK 约束：check
* 默认值约束：default

## 操作命令

### 数据库相关操作命令

``` text
CREATE DATABASE 数据库名;	# 创建数据库

USE 数据库名;				# 切换数据库

CREATE DATABASES;		   # 显示所有数据库

SELECT DATABASE();		   # 查看当前数据库

DROP DATABASE 数据库名;		# 删除数据库
```

### 数据表相关操作命令

```text
# 创建表
CREATE TABLE 表名(
    id INT AUTO_INCREMENT PRIMARY KEY,
    column_name column_type 字段约束 默认值,
    column_name column_type 字段约束 默认值,
    ...
    column_name column_type FOREIGN KEY REFERENCES 表名1(关联表column_name),
	)ENGINE=InnoDB DEFAULT CHARSET=utf8;

# 查看表结构
DESC 表名;
SHOW COLUMNS FROM 表名;

# 查看表的创建语句
SHOW CREATE TABLE 表名;

# 修改表结构
ALTER TABEL 表名 ADD column_name column_type 字段约束 默认值;
ALTER TABLE 表名 CHANGE column_name_i column_name_j BIGINT;
ALTER TABEL 表名 CHANGE column_name_i  BIGINT;
ALTER TABLE 表名 MODIFY column_name_i CHAR(10);
ALTER TABEL 表名 DROP column_name;

# 删除表
DROP TABLE 表名;

# 修改表名
ALTER TABLE testalter_tbl RENAME TO alter_tbl;
```

### 数据表中数据操作

#### 添加数据

```sql
# 全列插入(其中的一些具有默认值的字段可以使用占位符来代替)
INSERT INTO 表名 VALUES(对应表中字段的值);

# 缺省插入，可以指定插入的字段
INSERT INTO table_name (field1, field2,...fieldN)
                       VALUES
                       (value1, value2,...valueN);

# 一次插入多条数据（mysql特有）
INSERT INTO table_name ( field1, field2,...fieldN)
                       VALUES
                       (value1, value2,...valueN),
                       (value1, value2,...valueN),
                       ...
                       ;
```

#### 删除数据

```text
DELETE FROM 表名 [WHERE Clause];
```

#### 修改数据

```text
UPDATE 表名 SET field1=new-value1, field2=new-value2
	[WHERE Clause];
```

#### 查询数据

```text
SELECT column_name,column_name
	FROM 表名
	[WHERE Clause]
	[GROUP BY]
    [ORDER BY]
	[LIMIT N][ OFFSET M];
	
select distinct *
    from 表名
    where ...
    group by ... having ...
    order by ...
    limit start count;
```

***

## 表间关系

根据 E-R模型 来设计表的时候还应考虑表表之间的关系。常见的表间关系包括：

* 一对一
* 一对多
* 多对多

## 连接查询

JOIN 按照功能大致分为如下三类：

* INNER JOIN（内连接,或等值连接）：获取两个表中字段匹配关系的记录
* LEFT JOIN（左连接）：获取左表所有记录，即使右表没有对应匹配的记录
* RIGHT JOIN（右连接）： 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

## 事务

一系列数据库操作的整体，要么全部执行，要么都不会执行。保证一次数据操作业务逻辑的完整性。

 一般来说，事务是必须满足4个条件（ACID）：：原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）。

* 原子性（atomicity）：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样
* 一致性（consistency）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作
* 隔离性（isolation）：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）
* 持久性（durability）：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

MYSQL 事务处理主要有两种方法：

1、用 BEGIN, ROLLBACK, COMMIT来实现

* BEGIN 开始一个事务

- ROLLBACK 事务回滚
- COMMIT  事务确认

2、直接用 SET 来改变 MySQL 的自动提交模式: 

- SET AUTOCOMMIT=0   禁止自动提交
- SET AUTOCOMMIT=1 开启自动提交

## 索引

为了加速数据库的检索速度而在数据库的指定字段上添加对应的索引来提高对这些字段的查找速度。但是建立过多的索引会降低更新表的速度。

### 普通索引

```sql
CREATE INDEX indexName ON mytable(username(length)); 

ALTER table tableName ADD INDEX indexName(columnName);

CREATE TABLE mytable(  
	ID INT NOT NULL,   
	username VARCHAR(16) NOT NULL,  
	INDEX [indexName] (username(length))  
	);  
	
DROP INDEX [indexName] ON mytable; 

SHOW INDEX FROM table_name; \G
```

### 唯一索引

```sql
CREATE UNIQUE INDEX indexName ON mytable(username(length)) 

ALTER table mytable ADD UNIQUE [indexName] (username(length))

CREATE TABLE mytable(  
	ID INT NOT NULL,   
	username VARCHAR(16) NOT NULL,  
	UNIQUE [indexName] (username(length))  
	);  
	
SHOW INDEX FROM table_name; \G
```

***

参考：

* [MySQL 教程 ](https://www.runoob.com/mysql/mysql-tutorial.html)，菜鸟教程