spider类定义如何抓取某个站点(或一组站点)，包括：

* 如何执行抓取(即跟踪链接)
* 如何从页面中提取结构化数据(即抓取项)

同时，spider必须继承自`scrapy.Spider`。

对于蜘蛛来说，抓取周期如下：

1. 生成初始Requests，下载这些requests的response，并调用指定回调函数。
   * 通过调用`start_requests()`方法，获得第一个需要爬取的requests 。该方法默认给`start_urls`中的URL生成`Request`，`parse`方法作为回调函数。
2. 在回调函数中，解析response(web页面)，并返回一个包含提取数据的dict、Item对象、Request对象或这些对象的迭代。这些Request包含一个回调函数，然后由scrapy下载，通过回调函数处理下载的response
3. 在回调函数中，通常使用Selectors解析页面内容，并使用解析的数据生成Item
4. 最后，从spider返回的Item通常会被持久化到数据库（通过Item Pipeline），或使用Feed exports写入文件。

***

## scrapy.Spider 类

`class scrapy.spiders.Spider`

其他所有spider都必须继承它

* `name`

  一个字符串，定义Spider的名字。

  Scrapy通过Spider名称定位（并实例化）Spider，因此必须是唯一的，不能为不同的Spider设置相同的名称。

* `allowed_domains`

  一个可选的字符串列表，其中包含允许此spider爬取的域名。

* `start_urls`

  URL 列表。当没有指定特定url时，默认被spider创建初始requests，由start_requests()用来创建。

* `custom_settings`

  设置字典。必须定义为一个类属性，自定义不同的spider的settings，运行Spider时将从项目范围覆盖配置。

* `start_requests(self)`

  必须返回Spider一个开始抓取的迭代Requests （可以返回一个Spider列表,或者一个Spider生成器）。当Spider开始抓取时它会被Scrapy调用，Scrapy只调用它一次。 随后的requests将从这些初始requests中接连生成。按需重载。

  默认实现：为`start_urls`中的每个url生成`Request(url, dont_filter=True)`。

  如果想更改用于开始抓取url的Request，需要覆盖此方法。例如，实现网站登录。

  Scrapy schedules Spider的`start_requests`返回的`scrapy.Request`对象。 在收到每个response后，它会实例化`Response`对象，并调用与request相关的回调方法，并将response作为参数传递。

* `parse(self, response)`

  用来处理requests下载的response。当用户的requests没有指定回调时，这是Scrapy用来处理下载response的默认回调。

  response参数是TextResponse的一个实例，它保存页面内容，和进一步处理的方法。
  该方法通常解析response，将抓取的数据提取为dict，并找到要跟踪的新url，并从中创建新的Request。
  内置使用CSS选择器和XPath表达式，从HTML/XML选择和提取数据，以及正则表达式作为辅助方法进行提取。

* `log(message[, level, component])`

  通过Spider的`logger`发送日志消息。

* `closed(reason)`

  当Spider关闭时调用。

  该方法为 spider 的 `spider_closed `信号提供了`signals.connect()`的快捷方式。

***

## CrawlSpider 类

`class scrapy.spiders.CrawlSpider`

这是爬取常规网站最常用的spider，因为它通过定义一组规则，为跟踪链接提供了一种方便的机制。

除了从Spider继承的属性（必须指定）之外，该类还支持的新属性：

* `rules`

  一个（或多个）`Rule`对象的列表。  每个`Rule`定义用于规定抓取站点的特定行为。

  如果多个`Rule`匹配相同的 url，则会使用第一个`Rule`。

* `parse_start_url(response)`

  这个方法被`start_urls`的responses调用。

  它解析初始响应，并且必须返回一个Item对象、一个Request对象或一个包含其中任何一个的可迭代对象。

Note：不能重载`parse(self, response)`方法，因为CrawlSpider已将其重载。

#### Rule 类

```text
class scrapy.spiders.Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)

link_extractor：LinkExtractor对象，定义从当前网页获取新的链接的规则。

callback：回调函数。处理按照link_extractor提取URL的response。该回调函数接收response作为其第一个参数，并返回Item或Request。同时，不要使用parse()作为回调函数。

cb_kwargs：字典参数，并传递给回调函数。

follow：bool类型，用于指定是否要继续从当前规则解析出的Response中，继续提取url。如果callback是None，则follow默认为True，否则默认为False。

process_links：回调函数，用于进一步处理那些用规则提取的url。

process_request：回调函数，用于处理使用当前规则的request。
```

#### Link Extractors

```text
class scrapy.linkextractors.lxmlhtml.LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), restrict_css=(), tags=('a', 'area'), attrs=('href', ), canonicalize=False, unique=True, process_value=None, strip=True)

allow(a regular expression (or list of)):匹配成功，则提取的该url。默认则匹配所有url。

deny(a regular expression (or list of)):区别只在于正则表达式匹配到url都不提取。其优先级高于allow。默认无效。

allow_domains(str or list):url字符串或列表，用于提取。

deny_domains(str or list):参数同allow_domains，功能相反。

deny_extensions(list):在提取url时，需要忽略的单个值或字符串列表。

restrict_xpaths(str or list):Xpath字符串或字符串列表，定义了网页中提取链接的特定区域。只有满足Xpath的网页特定区域内可以提取到链接。

restrict_css(str or list):功能类似restrict_xpaths。

tags(str or list):提取定义标签下的url。默认为('a', 'area')。

attrs(list):必须是满足tags参数规定的标签的属性,定义了url的属性来源。默认为('href',)。

unique(boolean):是否对url去重

process_value(callable)：回调函数，可以对提取的内容，做进一步处理。默认为：lambda x: x

strip(boolean):是否去除空白，默认为True
```

```html
<a href="javascript:goToPage('../other/page.html'); return false">Link text</a>
```

```python
def process_value(value):
    m = re.search("javascript:goToPage\('(.*?)'", value)
    if m:
        return m.group(1)
```

除此之外，还有`XMLFeedSpider`、`CSVFeedSpider`、`SitemapSpider`

***

## 示例

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

### `start_requests(self)`

#### 代替`start_urls`

```python
class MySpider(scrapy.Spider):
    name = 'example.com'
    allowed_domains = ['example.com']

    def start_requests(self):
        yield scrapy.Request('http://www.example.com/1.html', self.parse)
        yield scrapy.Request('http://www.example.com/2.html', self.parse)
        yield scrapy.Request('http://www.example.com/3.html', self.parse)

    def parse(self, response):
        for title in response.xpath(xxx).extract():
            yield MyItem(title=title)

        for url in response.xpath(xxx).extract():
            yield scrapy.Request(url, callback=self.parse)
```

#### 网站登录

```python
class MySpider(scrapy.Spider):
    name = 'myspider'

    def start_requests(self):
        return [scrapy.FormRequest("http://www.example.com/login",
                                   formdata={'user': 'john', 'pass': 'secret'},
                                   callback=self.logged_in)]

    def logged_in(self, response):
        # 在这里你用另外一个回调函数提取follow的链接
        # 并返回每个请求
        pass
```

可以将 logged_in() 中的代码，写在 start_requests() 中，不过分开写更明确。

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

例如，爬取一个问答网站，其主页面是问题列表、右侧有热点问题列表、底端是页码。

#### 从单个回调中返回多个Requests和Item：

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

