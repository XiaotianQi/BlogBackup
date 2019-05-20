`Item`对象是自定义的python字典,可以使用标准的字典语法来获取到其每个字段的值。Spider将会将爬取到的数据以 Item 对象返回。

当Item在Spider中被收集之后，它将会被传递到Item Pipeline，一些组件会按照一定的顺序执行对Item的处理。

每个项目管道组件(有时仅称为Item Pipeline)都是一个Python类，实现了一个简单的方法。接收一个Item，并对其执行操作，同时还决定该Item是应该继续通过管道传输，还是应该删除并不再处理。

Item管道的典型用途是：

- 清理HTML数据
- 验证抓取的数据（检查Item是否包含某些字段）
- 检查重复项（并丢弃它们）
- 将抓取的Item存储在数据库中

## 自定义 pipline

每个Item Pipeline组件都是一个Python类，必须实现以下方法：

* `process_item(self, item, spider)`

  每个item pipeline组件**必须实现**该方法。
  item：由 parse 方法返回的 Item 对象(Item对象)
  spider：抓取到这个 Item 对象对应的Spider对象
  process_item()必须满足其中一条：

  * 返回一个带数据的字典
  * 返回一个Item（或任何后代类）对象
  * 返回一个Twisted Deferred或抛出DropItem异常。
  * 被丢弃的Item不会被进一步的Item组件处理。

* ` open_spider(self, spider)`

  当spider被开启时，这个方法被调用。

* ` close_spider(self, spider)`

  当spider被关闭时，这个方法被调用，可以再爬虫关闭后进行相应的数据处理。

* ` from_crawler(cls, crawler)`

  如果存在，则调用此类方法从`Crawler`创建管道实例。必须返回管道的一个新实例。Crawler对象提供Scrapy 的核心组件，如设置和信号。这是管道访问它们，并将其功能挂钩到Scrapy的一种方法。
  
* `from_settings(cls, settings) `

  获取 settings.py 文件中的配置信息。

## 激活 Item pipelin

要激活Item Pipeline组件，必须将其类添加到`ITEM_PIPELINES`设置中，如下例所示：

```python
ITEM_PIPELINES = {
    'myproject.pipelines.MySQLPipeline': 300,
    'myproject.pipelines.JsonWriterPipeline': 800,
}
```

设置中，分配给类的整数值决定了它们运行的顺序：由小到大。通常在0-1000范围内定义这些数字。

***

## 丢弃无用/重复 Item

丢弃无用的Item：

```python
# item.py
from scrapy.exceptions import DropItem

class PricePipeline(object):

    vat_factor = 1.15

    def process_item(self, item, spider):
        if item.get('price'):
            if item.get('price_excludes_vat'):
                item['price'] = item['price'] * self.vat_factor
            return item
        else:
            raise DropItem("Missing price in %s" % item)
```

丢弃重复的Item：

```python
from scrapy.exceptions import DropItem

class DuplicatesPipeline(object):

    def __init__(self):
        self.ids_seen = set()

    def process_item(self, item, spider):
        if item['id'] in self.ids_seen:
            raise DropItem("Duplicate item found: %s" % item)
        else:
            self.ids_seen.add(item['id'])
            return item
```

***

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
    #def __init__(self):
        # self.file = codecs.open('chinanews.json', 'w', encoding='utf8')
    
    def open_spider(self, spider):
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
    def __init__(self, dbpool, dbparms):
        self.dbparms = dbparms
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
        return cls(dbparms)
    
    '''
    from_settings() 也可以用 from_crawler()
	@classmethod
	def from_crawler(cls, crawler):
		dbparms = dict(
            host = crawler.settings.get['MYSQL_HOST'],
            port = crawler.settings.get['MYSQL_PORT'],
            db = crawler.settings.get['MYSQL_DBNAME'],
            user = crawler.settings.get['MYSQL_USER'],
            passwd = crawler.settings.get['MYSQL_PASSWORD'],
            charset = 'utf8',
            cursorclass = pymysql.cursors.DictCursor,
            use_unicode = True,)
        dbpool = adbapi.ConnectionPool('pymysql', **dbparms)
        return cls(dbpool)
    '''
    
    def open_spider(self, spider):
        self.dbpool = adbapi.ConnectionPool('pymysql', **self.dbparms)   
    
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

## Feed exports

### 导出文件

JSON

    FEED_FORMAT: json
    Exporter used: JsonItemExporter
    不适用大文件

JSON lines

    FEED_FORMAT: jsonlines
    Exporter used: JsonLinesItemExporter

CSV

    FEED_FORMAT: csv
    Exporter used: CsvItemExporter
    指定列名和顺序使用 FEED_EXPORT_FIELDS.

XML

    FEED_FORMAT: xml
    Exporter used: XmlItemExporter

Pickle

    FEED_FORMAT: pickle
    Exporter used: PickleItemExporter

Marshal

    FEED_FORMAT: marshal
    Exporter used: MarshalItemExporter
### 存储

使用导出文件时，可以使用URI（通过` FEED_URI`设置）定义存储文件的位置。 文件导出支持由URI方案定义的存储后端类型。

* Local filesystem
* FTP
* S3 (requires botocore or boto)
* Standard output

***

参考：

https://docs.scrapy.org/en/latest/topics/item-pipeline.html

https://docs.scrapy.org/en/latest/topics/feed-exports.html