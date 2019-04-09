涉及数据的存储。

## Images

```python
# pipelines.py
from scrapy.pipelines.images import ImagesPipeline


class ChinanewsImagesPipeline(ImagesPipeline):
    # 获取 image 保存路径
    def item_completed(self, results, item, info):
        image_file_path = ''	# 设置默认值.有的文章，没有包含图片
        for ok, value in results:
            image_file_path = value['path']
        item['image_file_path'] = image_file_path
        return ite
```

```python
# settings.py

'ITEM_PIPELINES' = {            
            'NewsSpider.pipelines.ChinanewsImagesPipeline':1,
        }
'IMAGES_URLS_FIELD':'image_url'		# 需要下载的images的url
'IMAGES_STORE':'NewsSpider/images'	# 存储路径

# 图片大小过滤
# IMAGES_MIN_HEIGHT = 100
# IMAGES_MIN_WIDTH = 100
```

***

## JSON

### 自定义 JSON

```python
# pipelines.py
import json
improt codecs


class JsonPipeline(object):
    # 自定义 JSON 文件导出
    def __init__(self):
        self.file = codecs.open('chinanews.json', 'w', encoding='utf8')
    
    def spider_closed(self, spider):
        self.file.close()
    
    def process_item(self, item, spider):
        lines = json.dumps(dict(item), ensure_ascii=False) + "\n"
        self.file.write(lines)
        return item
```

```python
# settings.py
'ITEM_PIPELINES'={            
            'NewsSpider.pipelines.ChinanewsImagesPipeline':1,
            'NewsSpider.pipelines.JsonPipeline': 2,
        }
```

### 内置 JSON

```python
# pipelines.py
import json
from scrapy.exporters import JsonItemExporter


class JsonExporterPipleline(object):
    # 调用scrapy提供的json export导出json文件
    def __init__(self):
        self.file = open('chinanewsExport.json', 'wb')
        self.exporter = JsonItemExporter(self.file, encoding="utf-8", ensure_ascii=False)
        self.exporter.start_exporting()
    
    def close_spider(self, spider):
        self.exporter.finish_exporting()
        self.file.close()
        
    def process_item(self, item, spider):
        self.exporter.export_item(item)
        return item
```

```python
# settings.py
'ITEM_PIPELINES'={            
            'NewsSpider.pipelines.ChinanewsImagesPipeline':1,
            'NewsSpider.pipelines.JsonExporterPipleline': 2,
        }
```

***

## MySQL

### 同步

```python
# pipelines.py
import pymysql


class MySQLPipeline(object):
    def __init__(self):
        self.conn = pymysql.connect(
            host = 'localhost',
            port = 3306,
            user = 'root',
            passwd = 'qixt',
            db = 'news_spider',
            charset="utf8",
            use_unicode=True
        )
        self.cur = self.conn.cursor()
    
    def process_item(self, item, spider):
        insert_sql = '''
            INSERT INTO chinanews(url, url_id, title, category, time_created,
                source, content, image_file_path, editor) 
            VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
        '''
        params = (
            item['url'], item['url_id'], item['title'], item['category'],
            item['time_created'], item['source'], item['content'],
            item['image_file_path'], item['editor']
            )
        self.cur.execute(insert_sql, params)
        self.conn.commit()
```

```python
# settings.py
'ITEM_PIPELINES' = {            
            'NewsSpider.pipelines.ChinanewsImagesPipeline':1,
            #'NewsSpider.pipelines.JsonExporterPipleline': 2,
            'NewsSpider.pipelines.MySQLPipeline':2
}
```

### 异步

支持异步，并分离一些功能。

```python
# piplines.py

import pymysql
from twisted.enterprise import adbapi

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
    
    def process_item(self, item, spider):
        # 使用twisted将mysql插入变成异步执行
        query = self.dbpool.runInteraction(self.do_insert, item)
        # 处理异常
        query.addErrback(self.handle_error, item, spider)

    def do_insert(self, cursor, item):
        # 执行具体的插入
        insert_sql, params = item.get_insert_sql()
        cursor.execute(insert_sql, params)
    
    def handle_error(self, failure, item, spider):
        #处理异步插入的异常
        print(failure)
```

```python
# settings.py

'ITEM_PIPELINES' = {            
            'NewsSpider.pipelines.ChinanewsImagesPipeline':1,
            #'NewsSpider.pipelines.JsonExporterPipleline': 2,
            'NewsSpider.pipelines.MysqlTwistedPipline':2,
        }

MYSQL_HOST = 'localhost'
MYSQL_PORT = 3306
MYSQL_DBNAME = 'news_spider'
MYSQL_USER = 'root'
MYSQL_PASSWORD = 'qixt'
```

```python
# items.py

class ChinanewsItem(scrapy.Item):
    # 省略...
    def get_insert_sql(self):
        insert_sql = '''
            INSERT INTO chinanews(url, url_id, title, category, time_created,
                source, content, image_file_path, editor) 
            VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
        '''
        params = (
            self['url'], self['url_id'], self['title'], self['category'],
            self['time_created'], self['source'], self['content'],
            self['image_file_path'], self['editor']
            )
        return insert_sql, params
```



