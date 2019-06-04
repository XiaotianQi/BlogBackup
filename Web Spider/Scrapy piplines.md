`SItem`对象是自定义的python字典,可以使用标准的字典语法来获取到其每个字段的值。Spider将会将爬取到的数据以 Item 对象返回。

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

## 下载、处理文件和图像

Scrapy提供可重用的item pipelines，用于下载附加在特定项目的文件。这些管道共享一些功能和结构(将它们称为media pipelines)，但是通常要么使用Files Pipeline，要么使用Images Pipeline。

这两个管道都实现了这些特性：

* 避免重新下载已下载的媒体
* 指定媒体存储的位置

Images Pipeline有一些处理图像的额外功能：

* 将下载图像转换为通用格式(JPG)和模式(RGB)
* 生成缩略图
* 检测图像的宽/高，确保它们满足最小限制

Pillow 是用来生成缩略图，并将图片归一化为JPEG/RGB格式，因此为了使用图片管道，需要安装这个库。

管道还保存了需要下载的媒体url的内部队列，并将包含相同媒体的响应连接到该队列。这就避免了同一种媒体在被多个项目共享时被多次下载。

### Files Pipeline

使用Files Pipeline时，典型的工作流是这样的：

1. 在spider中，抓取一个项，并将所需下载的的url放入`file_urls`字段中；
2. item从spider返回，并进入 item pipeline；
3. 当item到达`FilesPipeline`时，`file_urls`中的url被Scrapy scheduler和downloader进行调度下载 (这意味着scheduler and downloader middlewares将被重用)，但是具有更高的优先级，会在其他页面被抓取前处理。item 会在这个特定的管道阶段保持“locker”的状态，直到完成文件的下载（或者由于某些原因未完成下载）;
4. 当文件下载完后，另一个字段(`files`)将被更新到结构中。这个组将包含一个字典列表，其中包括下载文件的信息，比如下载路径、源抓取地址（从 `file_urls` 获得）和图片的校验码(checksum)。
   `files` 列表中的文件顺序将和源 `file_urls` 组保持一致。如果某个图片下载失败，将会记录下错误信息，图片也不会出现在 `files` 组中。

从上面的过程可以看出，使用`FilesPipeline`涉及的项：

1.  `Item`要包含`file_urls`和`files`两个字段；
2. 打开`FilesPipeline`配置；
3. 配置文件下载目录`FILE_STORE`。

具体实现如下：

```python
# test.py
import scrapy
from NewsSpider.items import TestItem

class TestSpider(scrapy.Spider):
    name = 'test'
    allowed_domains = ['twistedmatrix.com']
    start_urls = ['https://twistedmatrix.com/documents/current/core/examples/']

    custom_settings = {
        'ITEM_PIPELINES' :{
            'scrapy.pipelines.files.FilesPipeline': 1,  
        },
        'FILES_STORE':r'C:\test\files',
        }

    def parse(self, response):
        urls  = response.css('a.reference.download.internal::attr(href)').extract()
        for url in urls:
            yield TestItem(file_urls = [response.urljoin(url)])
```

```python
# items.py
import scrapy

class TestItem(scrapy.Item):
    file_urls = scrapy.Field()  # 指定文件下载的连接
    files = scrapy.Field()      # 文件下载完成后会往里面写相关的信息
```

### Images Pipeline

IMagesPipeline的过程与FilePipeline差不多，参数名称和配置不一样，如下：

|          | FilesPipeline                               | ImagesPipeline                              |
| -------- | ------------------------------------------- | ------------------------------------------- |
| Package  | scrapy.pipelines.files.FilesPipeline        | scrapy.pipelines.images.ImagesPipeline      |
| Item     | file_urls<br/>files                         | image_urls<br/>images                       |
| Settings | ITEM_PIPELINES<br/>FILES_STROE              | ITEM_PIPELINES<br/>IMAGES_STORE             |
| 文件失效 | FILES_EXPIRES = 90                          | IMAGES_EXPIRES = 90                         |
| 重新定向 | MEDIA_ALLOW_REDIRECTS = True<br/>默认 False | MEDIA_ALLOW_REDIRECTS = True<br/>默认 False |

除此之外，ImagesPipeline还支持以下特别功能：

1. 生成缩略图，通过配置`IMAGES_THUMBS = {'size_name': (width_size,heigh_size),}` 

   当使用这个特性时，图片管道将使用下面的格式来创建各个特定尺寸的缩略图：`<IMAGES_STORE>/thumbs/<size_name>/<image_id>.jpg`

2. 过滤过小图片，通过配置`IMAGES_MIN_HEIGHT`和`IMAGES_MIN_WIDTH`来过滤过小的图片。

```python
# test.py
import scrapy
import json
from NewsSpider.items import TestItem

class TestSpider(scrapy.Spider):
    name = 'test'
    custom_settings = {
        'ITEM_PIPELINES' :{
            'scrapy.pipelines.images.ImagesPipeline': 1,  
        },
        'IMAGES_STORE':r'C:\Users\bnwse\Desktop\test\files',
        }
    
    def start_requests(self):
        step = 30
        for page in range(0,3):
            url = self.url_pattern.format(offset = page*step)
            yield scrapy.Request(url, callback = self.parse)

    def parse(self, response):
        ret = json.loads(response.body)
        for row in ret['list']:
            yield TestItem(image_urls=[row['qhimg_url']], name=row['group_title'])
```

```python
# items.py

class TestItem(scrapy.Item):
    name = scrapy.Field()
    image_urls = scrapy.Field()
    images = scrapy.Field()
```

### 扩展 Media Pipelines

#### FilesPipeline

`class scrapy.pipelines.files.FilesPipeline`

* `get_media_requests(item, info)`

  在工作流程中可以看到，pipeline会得到文件的URL,并从item中下载。为了这么做，需要重写 get_media_requests() 方法，并对文件的URL返回一个Request:

  ```python
  def get_media_requests(self, item, info):
      for file_url in item['file_urls']:
          yield scrapy.Request(file_url)
  ```

  这些requests将被管道处理，当它们完成下载后，结果将以2元素的元组列表形式传送到 `item_completed()` 方法:每个元组包含 `(success, file_info_or_error)`:

  * `success` 是一个布尔值，当图片成功下载时为 `True` ，因为某个原因下载失败为`False`

  * `file_info_or_error` 是一个包含下列关键字的字典（如果成功为`True`）或者出问题时为 

    Twisted Failure

    - `url` - 文件下载的url。这是从 `get_media_requests()`) 方法返回请求的url。
    - `path` - 图片存储的路径（类似 `FILES_STORE`）
    - `checksum` - 图片内容的 MD5 hash

* ` item_completed(results, items, info)`

  当一个单独项目中的所有图片请求完成时（要么完成下载，要么因为某种原因下载失败）， FilesPipeline.item_completed() 方法将被调用。

  接收的元组列表需要保证与 get_media_requests() 方法返回请求的顺序相一致。

  将下载的图片路径（传入到results中）存储到 `file_paths` 项目组中，如果其中没有图片，我们将丢弃项目:

  ```python
  from scrapy.exceptions import DropItem
  
  def item_completed(self, results, item, info):
      file_paths = [x['path'] for ok, x in results if ok]
      if not file_paths:
          raise DropItem("Item contains no files")
      item['file_paths'] = file_paths
      return item
  ```

  默认情况下， `item_completed()` 方法返回项目。

#### ImagesPipeline

与 FilesPipeline 相同：

```text
class scrapy.pipeline.images.ImagesPipeline

    get_media_requests(item, info)

    item_completed(results, items, info)
```

```python
import scrapy
from scrapy.pipeline.images import ImagesPipeline
from scrapy.exceptions import DropItem

class MyImagesPipeline(ImagesPipeline):

    def get_media_requests(self, item, info):
        for image_url in item['image_urls']:
            yield scrapy.Request(image_url)

    def item_completed(self, results, item, info):
        image_paths = [x['path'] for ok, x in results if ok]
        if not image_paths:
            raise DropItem("Item contains no images")
        item['image_paths'] = image_paths
        return item
```

或者如下：

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

https://www.jianshu.com/p/a412c0277f8a