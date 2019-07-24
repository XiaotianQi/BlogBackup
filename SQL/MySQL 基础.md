## 安装

ubuntu安装 mysql：

```text
sudo apt-get update

sudo apt-get install mysql-server

# 初始化配置
sudo mysql_secure_installation

# 检查服务状态
systemctl status mysql.service

sudo mysql -uroot -p
```

安装 mysql-workbench:

```text
sudo apt-get install mysql-workbench

# 如果无法连接数据库
# 本地root@host的账号的auth_socket，需改成mysql_native_password
SELECT host, user, plugin FROM mysql.user;

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

设置远程访问：

```text
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

# 将bind-address修改为 0.0.0.0，或者注释
bind-address = 0.0.0.0
# bind-address = 127.0.0.1

# 进入数据库，进行如下设置
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
flush privileges;

# 重新启动服务
systemctl restart mysql.service
```

其他：

```text
# 关闭密码安全策略
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

# 结尾添加
plugin-load=validate_password.so 
validate-password=OFF

# 重新启动服务
systemctl restart mysql.service
```

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

***

## DDL

``` text
CREATE DATABASE 数据库名;	# 创建数据库

SHOW DATABASES;			   # 查看所有数据库

USE 数据库名;				# 切换数据库

CREATE DATABASES;		   # 显示所有数据库

SELECT DATABASE();		   # 查看当前数据库

DROP DATABASE 数据库名;		# 删除数据库
```

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

# 查看数据库下所有表
SHOW TABLES;

# 删除表
DROP TABLE 表名;

# 修改表名
ALTER TABLE testalter_tbl RENAME TO alter_tbl;
```

示例：

```sql
-- 如果存在名为school的数据库就删除它
drop database if exists school;

-- 创建名为school的数据库并设置默认的字符集和排序方式
create database school default charset utf8 collate utf8_bin;

-- 切换到school数据库上下文环境
use school;

-- 创建学院表
create table tb_college
(
collid		int auto_increment comment '编号',
collname	varchar(50) not null comment '名称',
collmaster	varchar(20) not null comment '院长',
primary key (collid)
);

-- 创建学生表
create table tb_student
(
stuid		int not null comment '学号',
stuname		varchar(20) not null comment '姓名',
stusex		boolean default 1 comment '性别',
stubirth	date not null comment '出生日期',
stuaddr		varchar(255) default '' comment '籍贯',
collid		int not null comment '所属学院',
primary key (stuid),
foreign key (collid) references tb_college (collid)
);

-- 创建教师表
create table tb_teacher
(
teaid		int not null comment '工号',
teaname		varchar(20) not null comment '姓名',
teatitle	varchar(10) default '助教' comment '职称',
collid		int not null comment '所属学院',
primary key (teaid),
foreign key (collid) references tb_college (collid)
);

-- 创建课程表
create table tb_course
(
couid		int not null comment '编号',
couname		varchar(50) not null comment '名称',
coucredit	int not null comment '学分',
teaid		int not null comment '授课老师',
primary key (couid),
foreign key (teaid) references tb_teacher (teaid)
);

-- 创建选课记录表
create table tb_record
(
recid		int auto_increment comment '选课记录编号',
sid			int not null comment '选课学生',
cid			int not null comment '所选课程',
seldate		datetime default now() comment '选课时间日期',
score		decimal(4,1) comment '考试成绩',
primary key (recid),
foreign key (sid) references tb_student (stuid),
foreign key (cid) references tb_course (couid),
unique (sid, cid)
);
```

## DML

### 增、删、改

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

```text
DELETE FROM 表名 [WHERE Clause];
```

```text
UPDATE 表名 SET field1=new-value1, field2=new-value2
	[WHERE Clause];
```

示例：

```sql
-- 插入学院数据
insert into tb_college (collname, collmaster) values 
('计算机学院', '左冷禅'),
('外国语学院', '岳不群'),
('经济管理学院', '风清扬');

-- 插入学生数据
insert into tb_student (stuid, stuname, stusex, stubirth, stuaddr, collid) values
(1001, '杨逍', 1, '1990-3-4', '四川成都', 1),
(1002, '任我行', 1, '1992-2-2', '湖南长沙', 1),
(1033, '王语嫣', 0, '1989-12-3', '四川成都', 1),
(1572, '岳不群', 1, '1993-7-19', '陕西咸阳', 1),
(1378, '纪嫣然', 0, '1995-8-12', '四川绵阳', 1),
(1954, '林平之', 1, '1994-9-20', '福建莆田', 1),
(2035, '东方不败', 1, '1988-6-30', null, 2),
(3011, '林震南', 1, '1985-12-12', '福建莆田', 3),
(3755, '项少龙', 1, '1993-1-25', null, 3),
(3923, '杨不悔', 0, '1985-4-17', '四川成都', 3),
(4040, '隔壁老王', 1, '1989-1-1', '四川成都', 2);

-- 删除学生数据
delete from tb_student where stuid=4040;

-- 更新学生数据
update tb_student set stuname='杨过', stuaddr='湖南长沙' where stuid=1001;

-- 插入老师数据
insert into tb_teacher (teaid, teaname, teatitle, collid) values 
(1122, '张三丰', '教授', 1),
(1133, '宋远桥', '副教授', 1),
(1144, '杨逍', '副教授', 1),
(2255, '范遥', '副教授', 2),
(3366, '韦一笑', '讲师', 3);

-- 插入课程数据
insert into tb_course (couid, couname, coucredit, teaid) values 
(1111, 'Python程序设计', 3, 1122),
(2222, 'Web前端开发', 2, 1122),
(3333, '操作系统', 4, 1122),
(4444, '计算机网络', 2, 1133),
(5555, '编译原理', 4, 1144),
(6666, '算法和数据结构', 3, 1144),
(7777, '经贸法语', 3, 2255),
(8888, '成本会计', 2, 3366),
(9999, '审计学', 3, 3366);

-- 插入选课数据
insert into tb_record (sid, cid, seldate, score) values 
(1001, 1111, '2017-09-01', 95),
(1001, 2222, '2017-09-01', 87.5),
(1001, 3333, '2017-09-01', 100),
(1001, 4444, '2018-09-03', null),
(1001, 6666, '2017-09-02', 100),
(1002, 1111, '2017-09-03', 65),
(1002, 5555, '2017-09-01', 42),
(1033, 1111, '2017-09-03', 92.5),
(1033, 4444, '2017-09-01', 78),
(1033, 5555, '2017-09-01', 82.5),
(1572, 1111, '2017-09-02', 78),
(1378, 1111, '2017-09-05', 82),
(1378, 7777, '2017-09-02', 65.5),
(2035, 7777, '2018-09-03', 88),
(2035, 9999, default, null),
(3755, 1111, default, null),
(3755, 8888, default, null),
(3755, 9999, '2017-09-01', 92);
```

### 查询

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

示例：

```sql
-- 查询所有学生信息
select * from tb_student;

-- 查询所有课程名称及学分(投影和别名)
select couname, coucredit from tb_course;
select couname as 课程名称, coucredit as 学分 from tb_course;

-- 查询所有学生的姓名和性别(条件运算)
select stuname as 姓名, case stusex when 1 then '男' else '女' end as 性别 from tb_student;
select stuname as 姓名, if(stusex, '男', '女') as 性别 from tb_student;

-- 查询所有女学生的姓名和出生日期(筛选)
select stuname, stubirth from tb_student where stusex=0;

-- 查询所有80后学生的姓名、性别和出生日期(筛选)
select stuname, stusex, stubirth from tb_student where stubirth>='1980-1-1' and stubirth<='1989-12-31';
select stuname, stusex, stubirth from tb_student where stubirth between '1980-1-1' and '1989-12-31';

-- 查询姓"杨"的学生姓名和性别(模糊)
select stuname, stusex from tb_student where stuname like '杨%';

-- 查询姓"杨"名字两个字的学生姓名和性别(模糊)
select stuname, stusex from tb_student where stuname like '杨_';

-- 查询姓"杨"名字三个字的学生姓名和性别(模糊)
select stuname, stusex from tb_student where stuname like '杨__';

-- 查询名字中有"不"字或"嫣"字的学生的姓名(模糊)
select stuname, stusex from tb_student where stuname like '%不%' or stuname like '%嫣%';

-- 查询没有录入家庭住址的学生姓名(空值)
select stuname from tb_student where stuaddr is null;

-- 查询录入了家庭住址的学生姓名(空值)
select stuname from tb_student where stuaddr is not null;

-- 查询学生选课的所有日期(去重)
select distinct seldate from tb_record;

-- 查询学生的家庭住址(去重)
select distinct stuaddr from tb_student where stuaddr is not null;

-- 查询男学生的姓名和生日按年龄从大到小排列(排序)
select stuname as 姓名, datediff(curdate(), stubirth) div 365 as 年龄 from tb_student where stusex=1 order by 年龄 desc;

-- 查询年龄最大的学生的出生日期(聚合函数)
select min(stubirth) from tb_student;

-- 查询年龄最小的学生的出生日期(聚合函数)
select max(stubirth) from tb_student;

-- 查询男女学生的人数(分组和聚合函数)
select stusex, count(*) from tb_student group by stusex;

-- 查询课程编号为1111的课程的平均成绩(筛选和聚合函数)
select avg(score) from tb_record where cid=1111;

-- 查询学号为1001的学生所有课程的平均分(筛选和聚合函数)
select avg(score) from tb_record where sid=1001;

-- 查询每个学生的学号和平均成绩(分组和聚合函数)
select sid as 学号, avg(score) as 平均分 from tb_record group by sid;

-- 查询平均成绩大于等于90分的学生的学号和平均成绩
-- 分组以前的筛选使用where子句 / 分组以后的筛选使用having子句
select sid as 学号, avg(score) as 平均分 from tb_record group by sid having 平均分>=90;

-- 查询年龄最大的学生的姓名(子查询/嵌套的查询)
select stuname from tb_student where stubirth=( select min(stubirth) from tb_student );

-- 查询年龄最大的学生姓名和年龄(子查询+运算)
select stuname as 姓名, datediff(curdate(), stubirth) div 365 as 年龄 from tb_student where stubirth=( select min(stubirth) from tb_student );

-- 查询选了两门以上的课程的学生姓名(子查询/分组条件/集合运算)
select stuname from tb_student where stuid in ( select stuid from tb_record group by stuid having count(stuid)>2 );

-- 查询学生姓名、课程名称以及成绩(连接查询)
select stuname, couname, score from tb_student t1, tb_course t2, tb_record t3 where stuid=sid and couid=cid and score is not null;

-- 查询学生姓名、课程名称以及成绩按成绩从高到低查询第11-15条记录(内连接+分页)
select stuname, couname, score from tb_student inner join tb_record on stuid=sid inner join tb_course on couid=cid where score is not null order by score desc limit 5 offset 10;

select stuname, couname, score from tb_student inner join tb_record on stuid=sid inner join tb_course on couid=cid where score is not null order by score desc limit 10, 5;

-- 查询选课学生的姓名和平均成绩(子查询和连接查询)
select stuname, avgmark from tb_student, ( select sid, avg(score) as avgmark from tb_record group by sid ) temp where stuid=sid;

select stuname, avgmark from tb_student inner join ( select sid, avg(score) as avgmark from tb_record group by sid ) temp on stuid=sid;

-- 查询每个学生的姓名和选课数量(左外连接和子查询)
select stuname, ifnull(total, 0) from tb_student left outer join ( select sid, count(sid) as total from tb_record group by sid ) temp on stuid=sid;
```

***

## TCL

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

***

## 表间关系

根据 E-R模型 来设计表的时候还应考虑表表之间的关系。常见的表间关系包括：

- 一对一
- 一对多
- 多对多

## 连接查询

JOIN 按照功能大致分为如下三类：

- INNER JOIN（内连接,或等值连接）：获取两个表中字段匹配关系的记录
- LEFT JOIN（左连接）：获取左表所有记录，即使右表没有对应匹配的记录
- RIGHT JOIN（右连接）： 与 LEFT JOIN 相反，用于获取右表所有记录，即使左表没有对应匹配的记录。

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

[MySQL 教程 ](https://www.runoob.com/mysql/mysql-tutorial.html)，菜鸟教程

[关系数据库入门](https://github.com/jackfrued/Python-100-Days/blob/master/Day36-40/36-38.%E5%85%B3%E7%B3%BB%E5%9E%8B%E6%95%B0%E6%8D%AE%E5%BA%93MySQL.md)，骆昊