Scrapy settings 允许自定义所有Scrapy组件的行为，包括core, extensions, pipelines and spiders themselves.

设置的基础结构提供了一个键值映射的全局名称空间，代码可以使用该名称空间来提取配置值。可以通过不同的机制填充设置，如下所述。

可以使用不同的机制来填充设置，每种机制都有不同的优先级。 以下是按优先级降序排列的列表：

1. Command line options（最优先）
2. Settings per-spider
3. 项目settings模块
4. 每个命令的默认设置
5.  默认的全局设置（优先级最低）

设置scrapy路径：

```python
# settings.py

BASE_DIR = os.path.dirname(os.path.abspath(os.path.dirname(__file__)))
sys.path.insert(0, os.path.join(BASE_DIR, 'NewsSpider'))
```

获取 settings.py 中的值：

* `from_crawler(cls, settings)`

```python
# spider.py

class MyExtension(object):
    def __init__(self, log_is_enabled=False):
        if log_is_enabled:
            print("log is enabled!")

    @classmethod
    def from_crawler(cls, crawler):
        settings = crawler.settings
        return cls(settings.getbool('LOG_ENABLED'))
```

`from_crawler()` 在 `__init__()` 之前运行。

* `from_settings(cls, settings)`

```python
# pipelines.py

class MysqlTwistedPipline(object):
    # 异步
    def __init__(self, dbpool):
        self.dbpool = dbpool
        
	@classmethod
    def from_settings(cls, settings):
        dbparms = dict(
            host = settings['MYSQL_HOST'],
            port = settings['MYSQL_PORT'],
            db = settings['MYSQL_DBNAME'],
            user = settings['MYSQL_USER'],
            passwd = settings['MYSQL_PASSWORD'],
            charset = 'utf8',
            cursorclass = pymysql.cursors.DictCursor,
            use_unicode = True,)
        dbpool = adbapi.ConnectionPool('pymysql', **dbparms)
        return cls(dbpool)
```

`from_settings()` 现在仅能在pipelines中使用。

* `from xx.settings import xxx`

直接从 settings 文件中获取目标变量。

内置设置参考：

内置的变量很多，按需设置。

***

参考：

https://docs.scrapy.org/en/latest/topics/settings.html