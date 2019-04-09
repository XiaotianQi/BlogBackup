## 自定义 settings

```python
# chinanews.py


class ChinanewsSpider(scrapy.Spider):
    ...
    custom_settings = {
        'DOWNLOADER_MIDDLEWARES':{
            'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None,
            'NewsSpider.middlewares.RandomUserAgentMiddleware': 1,
        },
        'ITEM_PIPELINES':{            
            'NewsSpider.pipelines.ChinanewsImagesPipeline':1,
            #'NewsSpider.pipelines.JsonExporterPipleline': 2,
            'NewsSpider.pipelines.MysqlTwistedPipline':2,
        },
        'IMAGES_URLS_FIELD':'image_url',
        'IMAGES_STORE':'NewsSpider/images',
    }
    ...
```

***

## 获取需要爬取的 URL

```python
class ChinanewsSpider(scrapy.Spider):
    name = 'chinanews'
    allowed_domains = ['www.chinanews.com']
    start_urls = ['http://www.chinanews.com/best-news/news1.html']

    def parse(self, response):
        # 获取需要爬取的 URL
        news_urls = response.css('.dd_bt>a::attr(href)').extract()
        for news_url in news_urls:
            if 'shipin' not in news_url:
                yield Request(url=news_url, callback=self.parse_detail)

    def parse_detail(self, response):
        # 具体网页内容解析
        yield news_item
```

***

## 解析具体网页内容

### 简单使用

```python
import scrapy
import datetime
from scrapy.http import Request
from scrapy.loader import ItemLoader
from NewsSpider.items import ChinanewsItem
from NewsSpider.utils.common import get_md5


class ChinanewsSpider(scrapy.Spider):
	# 省略...
    
    def parse_detail(self, response):
        title = response.css('#cont_1_1_2>h1::text').extract()[0].strip()
        category = response.xpath('//div[@id="nav"]/a[2]/text()').extract_first()
        time_created = response.css('div.left-t::text').extract()[0].strip()[:17]
        try:
            time_created = datetime.datetime.strptime(time_created, "%Y年%m月%d日 %H:%M")
        except Exception as e:
            time_created = datetime.datetime.now()
        source = response.css('div.left-t>a:nth-child(1)::text').extract_first()
        if not source:
            source = response.css('div.left-t::text').extract_first().strip().split(u'来源：')[1]
        body_nodes = response.css('div.left_zw')
        for node in body_nodes:
            content = node.css('p::text').extract()
            content = '\n'.join([i.strip() for i in content[1:]])
            image_url = node.css('img::attr(src)').extract()
        editor = response.css('.left_name .left_name::text').extract()
        if editor:
            editor = editor[0].strip()[4:-1] 
        else:    
            editor = 'None'
        
        news_item=ChinanewsItem()
        news_item['url'] = response.url
        news_item['url_id'] = get_md5(response.url)
        news_item['title'] = title
        news_item['category'] = category
        news_item['time_created'] = time_created
        news_item['source'] = source
        news_item['content'] = content
        news_item['image_url']= image_url
        news_item['editor'] = editor

        yield news_item
```

### ItemLoader

```python
import scrapy
from scrapy.http import Request
from scrapy.loader import ItemLoader
from NewsSpider.items import ChinanewsItem
from NewsSpider.utils.common import get_md5


class ChinanewsSpider(scrapy.Spider):
	# 省略...
    
    def parse_detail(self, response):
		item_loader = ItemLoader(item=ChinanewsItem(), response=response)
        item_loader.add_value('url', response.url)
        item_loader.add_css('title', '#cont_1_1_2>h1::text')
        item_loader.add_xpath('category', '//div[@id="nav"]/a[2]/text()')
        item_loader.add_css('time_created', 'div.left-t::text')
        item_loader.add_css('source', 'div.left-t>a:nth-child(1)::text')
        item_loader.add_css('content', 'div.left_zw>p::text')
        item_loader.add_css('image_url', '.left_zw img::attr(src),.left_ph img::attr(src)')
        item_loader.add_css('editor', '.left_name .left_name::text')
        news_item = item_loader.load_item()

        yield news_item
```

对于各个字段的转换，位于 item.py 文件中：

```python
# item.py
import scrapy
import datetime
from NewsSpider.utils.common import get_md5


def date_convert(dt):
    try:
        time_created = datetime.datetime.strptime(dt, '%Y年%m月%d日 %H:%M')
    except Exception as e:
        time_created = datetime.datetime.now()
    return time_created


class ChinanewsItem(scrapy.Item):
    url = scrapy.Field()
    url_id = scrapy.Field()
    title = scrapy.Field()
    category = scrapy.Field()
    time_created = scrapy.Field()
    source = scrapy.Field()
    content = scrapy.Field()
    image_url = scrapy.Field()
    image_file_path = scrapy.Field()
    editor = scrapy.Field()       

    def get_insert_sql(self):
        url = self['url'][0]
        url_id = get_md5(url)
        title = self['title'][0].strip()
        category = self['category'][0]
        time_created = date_convert(self['time_created'][0].strip()[:17])
        source = self['source'][0] if 'source' in self else self['time_created'][0].strip().split(u'来源：')[1]
        content = ''.join(self['content'])
        image_file_path = self['image_file_path']
        editor = self['editor'][0].strip()[4:-1] if 'editor' in self else ''

        insert_sql = '''
            INSERT INTO chinanews(url, url_id, title, category, time_created,
                source, content, image_file_path, editor) 
            VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
        '''
        params = (
            url, url_id, title, category, time_created,
            source, content, image_file_path, editor
            )
        return insert_sql, params

```

