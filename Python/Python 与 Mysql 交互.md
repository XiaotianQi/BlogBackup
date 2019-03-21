基于 Python PyMySQL包，围绕 MySQL 的“增删改查”4个基本的数据库操作，展示 Python 与 MySQL 的基本交互方式。

Python 与 MySQL 交互的基本流程如下图：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/mysql%20flow.png)

***

## 相关函数说明

### `pymysql.connect()`

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

### `connection` 对象支持的方法

| 方法 | 作用 |
| ------ | :------- |
| `.cursor()` | 使用该连接，创建并返回游标 |
| `.commit()` | 提交当前事务 |
| `.rollback()` | 回滚当前事务 |
| `.close()` | 关闭连接 |

### `cursor` 对象支持的方法和属性

| 方法 | 作用 |
| ------ | :------- |
| `.execute(operation, params=None, multi=False)` | 执行单条数据库命令 |
|`.executemany(operation, seq_of_params)`|执行单条数据库命令，循环执行参数列表中的数据，直至结束|
| `.fetchone()` | 获取 1 个查询结果集 |
| `.fetchmany(size=1)` | 获取 n 个结果集 |
| `.fetchall()` | 获取全部查询结果集，返回一个由元组构成的列表 |
| `.close()` | 关闭游标对象 |

| 属性 | 作用 |
| ------ | :------- |
|`.lastrowid`| 只读属性，获取所最后数据的主键 `id`，且 `id` 必须是 `AUTO_INCREMENT` |
| `.rowcount` | 只读属性，并返回执行 `execute()` 方法后影响的行数 |

#### `.execute()`

```python
cursor.execute(operation, params=None, multi=False)
iterator = cursor.execute(operation, params=None, multi=True)
```

由其可以执行 MySQL 的命令。

* `operation`：SQL 语句；
* `params`：数据，tuple 或者 dict 类型；
* `multi`：是否可以迭代。

```python
insert = (
    'INSERT INTO student (name, birthday, age)'
    'VALUES (%s %s %s)'
)
data = ('Alice', datetime.date(1995, 1, 5), 23)

cursor.execute(insert, data)

select = 'SELECT * FROM student WHERE age = %(age)s'
cursor.execute(select, {'age': 23})
```

#### `.executemany()`

```python
cursor.executemany(operation, seq_of_params)
```

作用同 `.execute()`，不同之处在于，传入数据可以是列表，但类表中的元素仍是 `tuple`。

```python
data = [
  ('Alice', datetime.date(1995, 1, 5), 23),
  ('Bob', datetime.date(1995, 2, 15), 23),
  ('Cur', datetime.date(1995, 3, 25), 23),
]
insert = 'INSERT INTO student (name, birthday, age) VALUES (%s %s %s)'
cursor.executemany(insert, data)
```

写为 SQL 语句，则是：

```sql
INSERT INTO student (name, birthday, age)
VALUES ('Alice', '1995-01-05', 23), ('Bob', '1995-02-15', 23), ('Cur', '1995-03-25', 23)
```

#### `.fetchone()`

返回一个 `tuple`，一行查询结果。如果没有，则返回 `None`。

还有以下拓展用法：

```python
# Using a while loop
cursor.execute("SELECT * FROM student")
row = cursor.fetchone()
while row is not None:
    print(row)
    row = cursor.fetchone()

# Using the cursor as iterator
cursor.execute("SELECT * FROM student")
for row in cursor:
    print(row)
```

***

## 创建数据库表

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

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/mysql-1.png)

***

## 添加数据

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

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/mysql-2.png)

***

## 查询数据

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

## 更新数据

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

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/mysql-3.png)

***

## 删除数据

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

***

参考：

[cursor.MySQLCursor Class](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor.html)