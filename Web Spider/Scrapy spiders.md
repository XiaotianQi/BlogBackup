spider是我们定义的类，scrapy用它从网站(或一组网站)中抓取信息。同时，spider继承自`scrapy.Spider`，定义要初始requests，跟踪页面中的链接，以及解析下载的页面内容、提取数据。

内置使用CSS选择器和XPath表达式，从HTML/XML选择和提取数据，以及正则表达式作为辅助方法进行提取。

***

## Spider 类

```python
class ChinanewsSpider(scrapy.Spider):
    name = 'chinanews'
    allowed_domains = ['www.chinanews.com']
    start_urls = ['http://www.chinanews.com/best-news/news1.html']

    custom_settings = {...}
    
    def start_requests(self):
        ...
        yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        ...
        yield Request(url=url, callback=self.parse_detail)

    def parse_detail(self, response):
        yield item
```

```text
name：标识Spider。在项目中必须是唯一的，不能为不同的Spider设置相同的名称。

start_urls：默认被的start_requests()用来创建spider的初始请求：

custom_settings:自定义不同的spider的settings

start_requests()：必须返回Spider一个开始抓取的迭代requests（可以返回一个Spider列表,或者一个Spider生成器）。 随后的requests将从这些初始requests中接连生成。按需重载。

parse()：用来处理requests下载的response。 response参数是TextResponse的一个实例，它保存页面内容，和进一步处理的方法。
该方法通常解析response，将抓取的数据提取为dict，并找到要跟踪的新url，并从中创建新的Request。
```

Scrapy schedules Spider的`start_requests`返回的`scrapy.Request`对象。 在收到每个response后，它会实例化`Response`对象，并调用与request相关的回调方法，将response作为参数传递。

***

## 示例

### 获取需要爬取的 URL

```python
class ChinanewsSpider(scrapy.Spider):
    name = 'chinanews'
    allowed_domains = ['www.chinanews.com']
    start_urls = ['http://www.chinanews.com/best-news/news1.html']

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

由此可见，Scrapy的跟随链接机制：当yield a Request in a callback method，Scrapy将调度要发送的request，并声明一个当request结束时执行的回调方法。

基于这个方法，可以自定义复杂的抓取程序，并根据所访问的页面提取不同类型的数据。

#### 处理相对 URL

```bash
>>> response.css('li.next a::attr(href)').extract_first()
'/page/2/'
```

得到的是一个相对的URL，需要进行`URLjoin()`：

```python
def parse(self, response):
	next_page = response.css('li.next a::attr(href)').extract_first()
    if next_page is not None:
        yield scrapy.Request(response.urljoin(next_page), callback=self.parse)
```

或者使用`urlparse.parse.urljoin()`：

```python
import urlparse as parse

def parse(self, response):
	next_page = response.css('li.next a::attr(href)').extract_first()
    if next_page is not None:
        yield scrapy.Request( parse.urljoin(response.url, next_page), callback=self.parse)
```

可以使用`response.follow`作为创建Request对象的快捷方式：

返回一个`Request`实例。
其参数与`Request`相同，但`url`不仅可以是绝对URL，还可以是：

* 相对URL
* 属性选择器（不是选择器列表，`response.css('a::attr(href)')[0]`）
* `<a>`或`<link>`元素的选择器,例如`response.css('a.my_link')[0]`.
* scrapy.link.Link对象

传入字符串：

```python
def parse(self, response):
	next_page = response.css('li.next a::attr(href)').extract_first()
	if next_page is not None:
		yield response.follow(next_page, callback=self.parse)
```

或者，传入单个选择器：

```python
for href in response.css('li.next a::attr(href)'):
    yield response.follow(href, callback=self.parse)
```

自动使用标签中的`href`属性，进一步缩短：

```python
for a in response.css('li.next a'):
    yield response.follow(a, callback=self.parse)
```

### 回调和跟踪链接

当requests的响应已下载后，调用该requests的回调函数。下载的response对象作为回调函数的第一个参数。

例如，爬取一个问答网站，其主页面是问题列表、右侧有热点问题列表、底端是页码

```python
class ExampleSpider(scrapy.Spider):
	# 伪代码：爬取问答网站
    # 获取：问题信息、答案信息、热点问题、下一页

    def parse(self, response):
        # 问题
        questions_url = response.css('xxx').extract()
        for url in questions_url:
            yield Request(url=url, callback=self.parse_question)
		
        # 热点
        hot_url = response.css('xxx').extract()
        for url in hot_url:
            yield Request(url=url, callback=self.hot)
          
        # 下一页
        next_url = response.css('xxx').extract()
        for url in next_url:
            yield Request(url=url, callback=self.parse)

    def parse_question(self, response):
		# 解析问题
        answers_url = response.css('xxx').extract()
        yield Request(url=answers_url, callback=self.parse_answers)
		yield question_item
        
    def parse_answers(self, response):
		# 解析问题
        yield answers_item
        
    def parse_hot(self, response):
		# 解析问题
        answers_url = response.css('xxx').extract()
        yield Request(url=answers_url, callback=self.parse_answers)
        yield hot_item
```

这个spider将从主页面开始，它将follow主页面中所有问题的url、热点问题的url、下一页的url，然后通过回调，在对应函数中进行解析，决定生成Items，还是继续发出request。

#### 将数据传递给回调函数

在某些情况下，可能会将参数传递给这些回调函数，以便稍后在第二个回调函数中接收参数。 可以使用`Request.meta`属性。将来自多个页面的数据保存在一个Item中。

```
def parse_page1(self, response):
    item = MyItem()
    item['main_url'] = response.url
    request = scrapy.Request("http://www.example.com/some_page.html",
                             callback=self.parse_page2)
    request.meta['item'] = item
    yield request

def parse_page2(self, response):
    item = response.meta['item']
    item['other_url'] = response.url
    yield item
```

***

### 解析具体网页内容

#### 简单使用

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

#### 使用 ItemLoader

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

