较新版本的 Python 内置一个轻量级的数据库 SQLite，暂时不需要配置数据库，因为 Python 支持和 SQLite 进行通信。如果使用 PostgreSQL 、MySQL 、Oracle 等数据库时，需要配置。配置过程分为两步：

* 安装和配置数据库服务器本身。
* 为服务器后端安装必要的 Python 库。这是一些允许 Python 连接数据库的第三方代码。

***

### **MySQL 安装**

为使用 MySQL，需要安装三个软件：mysql-server、mysql-client、libmysqlclient-dev。

检查是否安装成功：

```shell
sudo netstat -tap | grep mysql
```

如果看到 mysql 的 socket 处于 listen 状态则表示安装成功。

除此之外，创建新数据库时，注意使用 utf8 编码。

```shell
CREATE DATABASE xxx CHARACTER SET utf8
```

MySQL 其他相关命令：

```shell
service mysql start
service mysql stop
service mysql restart
mysql -u root -p
```

***

### **Django 相关配置**

mysqlclient 可以实现 Django 和 MySQL 之间的通信。

```shell
pip install mysqlclient
```

Django 数据库相关配置：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'xxx',
        'USER': 'xxx',
        'PASSWORD': 'xxx',
        'OPTIONS':{
            'init_command':"SET sql_mode='STRICT_TRANS_TABLES'",
        },
    }
}
```

重要配置信息：数据库引擎 ENGINE，数据库名称 NAME，数据库用户名 USER，密码 PASSWORD。主机地址 HOST和端口 PORT 使用默认设置。

***

### **ORM**

ORM 全称是：Object Relational Mapping，可称为对象关系映射，其主要作用是在编程中，在对象和关系之间提供一条桥梁，前台的对象型数据和数据库中的关系型的数据通过这个桥梁来相互转化。

Django 采用 ORM，使我们不用写原汁原味的 SQL 语句，而只需关心 model 。正是基于 ORM，在 Django 中，数据模型(ModelForm)、数据迁移(Migration)、后台管理(admin)等使用 Django 语句就可以实现。