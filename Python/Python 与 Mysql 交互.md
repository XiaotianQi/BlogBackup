基于 Python PyMySQL包，围绕 MySQL 的“增删改查”4个基本的数据库操作，展示 Python 与 MySQL 的基本交互方式。

Python 与 MySQL 交互的基本流程如下图：

<div align="center"><img src="https://wx4.sinaimg.cn/mw690/af9e9c30ly1fpydcbbku2j20qr08474s.jpg" width = "550" height = "170" align=center />
</div>

***

#### **准备事项**

* 安装 Python PyMySQL 包
* 创建测试数据库 TESTDB

***

#### **相关函数说明**

* pymysql.connect()

创建与数据库的连接。

```python
connect = pymysql.connect(
    host="localhost",
    port=3310,
    user="root",
    passwd="xxx",
    db="TESTDB",
    charset="utf8"
)
```

或者写成:

```python
connect = pymysql.connect("localhost", 3310, "root"，"xxx", "TESTDB", "utf8"
)
```

* connection 对象支持的方法

| 函数 | 作用 |
| ------ | :------- |
| cursor() | 使用该连接，创建并返回游标 |
| commit() | 提交当前事务 |
| rollback() | 回滚当前事务 |
| close() | 关闭连接 |

* cursor对象支持的方法

| 函数 | 作用 |
| ------ | :------- |
| execute() | 执行数据库命令 |
| fetchone() | 获取一个查询结果集 |
| fetchmany(n) | 获取n个结果集 |
| fetchall() | 获取全部查询结果集 |
| rowcount() | 只读属性，并返回执行execute()方法后影响的行数 |
| close() | 关闭游标对象 |

***

#### **创建数据库表**

```python
# _*_coding:utf-8_*_
import pymysql

db = pymysql.connect("localhost", "root", "xxx", "TESTDB")
cursor = db.cursor()
cursor.execute("DROP TABLE IF EXISTS EMPLOYEE")

sql = """CREATE TABLE EMPLOYEE(
    FIRST_NAME CHAR(20) NOT NULL,
    LAST_NAME CHAR(20),
    AGE INT,
    SEX CHAR(1),
    INCOME FLOAT)
"""

cursor.execute(sql)
db.close()

```

数据库输出结果：

<div align="center"><img src="https://wx3.sinaimg.cn/mw690/af9e9c30ly1fqe9fbas1uj20k4078t8z.jpg" width = "550" height = "170" align=center />
</div>

***

#### **添加数据**

```python
# _*_coding:utf-8_*_
import pymysql

db = pymysql.connect("localhost", "root", "xxx", "TESTDB")
cursor = db.cursor()

sql = """INSERT INTO EMPLOYEE(
    FIRT_NAME, LAST_NAME, AGE, SEX, INCOME)
    VALUES('MAC', 'MOHAN', 20, 'M', 2000)
)
"""

try:
    cursor.execute(sql)
    db.commit()
except:
    db.rollback()

db.close()

```

数据库输出结果：

<div align="center"><img src="https://wx2.sinaimg.cn/mw690/af9e9c30ly1fqea19accsj20hc04mjrg.jpg" width = "550" height = "130" align=center />
</div>

***

#### **查询数据**

```python
# _*_coding:utf-8_*_
import pymysql

db = pymysql.connect("localhost", "root", "xxx", "TESTDB")
cursor = db.cursor()

sql = "SELECT * FROM EMPLOYEE WHERE INCOME > '%d'" % (1000)

try:
    cursor.execute(sql)
    results = cursor.fetchall()
    for row in results:
        fname = row[0]
        lname = row[1]
        age = row[2]
        sex = row[3]
        income = row[4]
        print("fname=%s,lname=%s,age=%d,sex=%s,income=%d" % (fname, lname, age, sex, income))
except:
    print("Error: unable to fetch data")

db.close()

```

输出结果：

```python
fname=Mac,lname=Mohan,age=20,sex=M,income=2000
```

***

#### **更新数据**

```python
# _*_coding:utf-8_*_
import pymysql

db = pymysql.connect("localhost", "root", "xxx", "TESTDB")
cursor = db.cursor()

sql = "UPDATE EMPLOYEE SET AGE = AGE + 5 WHERE SEX = '%c'" % ('M')

try:
    cursor.execute(sql)
    db.commit()
except:
    db.rollback()

db.close()

```

更新前后数据对比：

<div align="center"><img src="https://wx4.sinaimg.cn/mw690/af9e9c30ly1fqeiquwiqrj20hc09ojrt.jpg" width = "550" height = "250" align=center />
</div>

***

#### **删除数据**

```python
# _*_coding:utf-8_*_
import pymysql

db = pymysql.connect("localhost", "root", "xxx", "TESTDB")
cursor = db.cursor()

sql = "DELETE FROM EMPLOYEE WHERE AGE > '%d'" % (20)

try:
    cursor.execute(sql)
    db.commit()
except:
    db.rollback()

db.close()

```