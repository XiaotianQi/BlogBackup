Scrapy使用`Request`和`Response`对象来抓取网站。

通常，在Spider中生成`Request`对象，通过系统，直到它们到达Downloader。Downloader执行请求，并返回一个`Response`对象给发出请求的Spider。

`Request`和`Response`类都有子类，它们添加了基类中非必需的功能。

## Request objects

```text
class scrapy.http.Request(url[, callback, method='GET', headers, body, cookies, meta, encoding='utf-8', priority=0, dont_filter=False, errback, flags])
一个Request对象表示一个HTTP请求，通常在Spider中生成，并由Downloader执行，从而生成一个Response。

url(string)：请求的URL

callback(callable)：将此请求的响应，下载完成后，作为调用函数的第一个参数。如果请求没有指定回调函数，则将使用Spider的parse()方法。如果在处理期间引发异常，则会调用errback。

method(string)：请求的HTTP方法. 默认为'GET'.

meta(dict)：Request.meta属性的初始值. 如果给出，在此参数中传递的字典将被浅拷贝。

body(str or unicode)：请求正文。如果传递了一个unicode，那么它将被编码为相应的(默认为utf-8)str。如果未给出body，则会存储空字符串。无论此参数的类型如何，存储的最终值都将是str（不会是unicode或None）。

headers(dict)：请求的头文件。字典值可以是字符串（对于单值标题）或列表（对于多值标题）。如果将None作为值传递，则不会发送HTTP头文件。

cookies(dict or list)：请求的cookies。
	cookies={'currency': 'USD', 'country': 'UY'}
	cookies=[{'name': 'currency',
              'value': 'USD',
              'domain': 'example.com',
              'path': '/currency'}]
    后一种形式允许定制cookie的domain和path属性。这仅在cookie被保存用于以后的请求时才有用。

encoding(string)：请求的编码（默认为'utf-8'）。该编码将用于对URL进行百分比编码，并将主体转换为str（如果以unicode的形式给出）。

priority(int)：请求的优先级(默认为0). 调度程序使用优先级来处理请求的顺序。具有较高优先级值的请求将更早执行。允许用负值表示相对低的优先级。

dont_filter(boolean)：表示此请求不应被scheduler过滤。当想多次执行相同的请求时，使用此选项以忽略重复过滤器。小心使用它，否则将进入爬取循环。默认为False.

errback(callable)：如果在处理请求时引发异常，将会调用该函数。这包括404 HTTP错误等失败的页面。它接受一个Twisted Failure实例作为第一个参数。

flags(list)：发送到请求的标志,可用于日志记录或类似目的。
```

* `.url`

  请求的URL的字符串。

  只读。 更改时，使用`replace()`。

* `.method`

  请求HTTP方法的字符串。 确保它是大写的。 例: "GET", "POST", "PUT"等

* `.headers`

  请求头文件。

* `.body`

  请求主体的str。

  只读。 更改时，使用`replace()`。

* `.meta`

  请求的任意元数据的字典。

  对于新的请求这个字典是空的，通常由不同的Scrapy组件（extensions, middlewares等）填充。

  当使用`copy()`或`replace()`方法克隆请求时，该字典被浅拷贝，同时也可以在Spider中通过`response.meta`属性访问。

* `.copy()`

  返回一个新请求，它是此请求的副本.

* `.replace([url, method, headers, body, cookies, meta, encoding, dont_filter, callback, errback])`

  返回具有相同成员的Request 对象，但指定关键字参数的新值所赋予的成员除外。属性`Request.meta`默认复制（除非在`meta`参数中给出新值）。

### Request.meta特殊键

    dont_redirect
    dont_retry
    handle_httpstatus_list
    handle_httpstatus_all
    dont_merge_cookies (see cookies parameter of Request constructor)
    cookiejar
    dont_cache
    redirect_urls
    bindaddress
    dont_obey_robotstxt
    download_timeout：downloader等待的时间（以秒为单位）。另请参阅：DOWNLOAD_TIMEOUT。
    download_maxsize
    download_latency
    download_fail_on_dataloss：响应是否失败。请参阅：DOWNLOAD_FAIL_ON_DATALOSS
    proxy
    ftp_user（更多信息参见FTP_USER）
    ftp_password（有关更多信息，请参阅FTP_PASSWORD）
    referrer_policy
    max_retry_times：每个请求的重试次数。初始化时，max_retry_times元键优先于RETRY_TIMES设置。
如果出于某种原因想要避免与现有Cookie合并，可以通过在`Request.meta`中将`dont_merge_cookies`键设置为True来指示Scrapy执行此操作。不合并Cookie的请求示例：

```python
request_with_cookies = Request(url="http://www.example.com",
                               cookies={'currency': 'USD', 'country': 'UY'},
                               meta={'dont_merge_cookies': True})
```

### 将附加数据传递给回调函数

请求的回调函数将在该请求的响应下载完成时调用。 回调函数将下载的`Response`对象作为第一个参数进行调用。

将参数传递给这些回调函数，以便稍后在第二个回调函数中接收参数。 可以使用`Request.meta`属性。

```python
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

### 使用errbacks捕获请求处理中异常

当处理请求发生异常时，调用errback函数。

它收到一个Twisted Failure实例作为第一个参数，可用于跟踪连接建立超时，DNS错误等。

```python
import scrapy

from scrapy.spidermiddlewares.httperror import HttpError
from twisted.internet.error import DNSLookupError
from twisted.internet.error import TimeoutError, TCPTimedOutError

class ErrbackSpider(scrapy.Spider):
    name = "errback_example"
    start_urls = [
        "http://www.httpbin.org/",              # HTTP 200 expected
        "http://www.httpbin.org/status/404",    # Not found error
        "http://www.httpbin.org/status/500",    # server issue
        "http://www.httpbin.org:12345/",        # non-responding host, timeout expected
        "http://www.httphttpbinbin.org/",       # DNS error expected
    ]

    def start_requests(self):
        for u in self.start_urls:
            yield scrapy.Request(u, callback=self.parse_httpbin,
                                    errback=self.errback_httpbin,
                                    dont_filter=True)

    def parse_httpbin(self, response):
        self.logger.info('Got successful response from {}'.format(response.url))
        # do something useful here...

    def errback_httpbin(self, failure):
        # log all failures
        self.logger.error(repr(failure))

        # in case you want to do something special for some errors,
        # you may need the failure's type:

        if failure.check(HttpError):
            # these exceptions come from HttpError spider middleware
            # you can get the non-200 response
            response = failure.value.response
            self.logger.error('HttpError on %s', response.url)

        elif failure.check(DNSLookupError):
            # this is the original request
            request = failure.request
            self.logger.error('DNSLookupError on %s', request.url)

        elif failure.check(TimeoutError, TCPTimedOutError):
            request = failure.request
            self.logger.error('TimeoutError on %s', request.url)
```

***

## Response objects

```text
class scrapy.http.Response(url[, status=200, headers=None, body=b'', flags=None, request=None])
一个Response对象表示一个HTTP响应，它通常被Downloader下载,并且被反馈给Spider进行处理。

url(string)：此响应的URL

status(integer)：响应的HTTP状态。 默认为200。

headers(dict)：此响应的头文件。字典值可以是字符串（对于单值头文件）或列表（对于多值头文件）。

body(bytes)：响应正文。str（Python 2中的unicode）的形式解码后的文本，可以使用自适应编码的Response子类的response.text，例如TextResponse。

flags(list)：是包含Response.flags属性初始值的列表。如果给出，列表将被浅拷贝。

request(Request 对象)：Response.request属性的初始值。这表示生成此响应的Request。
```

* `.url`

  请求的URL的字符串。

  只读。 更改时，使用`replace()`。

* `.status`

  表示响应的HTTP状态的整数。 例如：`200`，`404`。 

* `.body`

  这个响应的主体。 

  Response.body始终是一个字节对象。 如果想要unicode版本可以使用`TextResponse.text`（仅在`TextResponse`和子类中可用）。 

  该属性是只读的。 要更改Response的主体，使用`replace()`。 

* `.request`

  生成此响应的`Request`对象。 

  当响应和请求已经通过所有Downloader Middlewares之后，在Scrapy引擎中分配此属性。 特别是，这意味着：  

  * HTTP重定向会将原始请求（重定向之前的URL）分配给重定向的响应（重定向后使用最终的URL）。
  * Response.request.url并不总是等于Response.url
  * 该属性仅在spider代码和Spider Middlewares中可用，但不能在Downloader Middleware中（尽管可以通过其他方式获得请求）和`response_downloaded`信号处理程序中使用.  

* `.meta`

  `Response.request`对象的`Request.meta`属性的快捷方式（即 `self.request.meta`).

  与`Response.request`属性不同，`Response.meta`属性在重定向和重试之间传递，因此将获得Spider发送的原始`Request.meta`数据。
  
* `.flags`

  标记包含此响应标志的列表。标志是用于标记响应的标签。例如:缓存、重定向等。

* `.copy`()

  返回一个新的Response，它是Response的副本。 

* `. replace([url, status, headers, body, request, flags, cls])`

  使用相同的成员返回一个Response对象，除了那些由指定的关键字参数赋予新值的成员。 属性`Response.meta`默认被复制。 

* `. urljoin(url)`

  通过将响应的`url`与可能的相对网址结合，构建绝对路径的url。

  这是urlparse.urljoin的一个包装，它仅仅是一个用于进行此调用的别名：`urlparse.urljoin(response.url, url)`

* `.follow(url, callback=None, method='GET', headers=None, body=None, cookies=None, meta=None, encoding='utf-8', priority=0, dont_filter=False, errback=None)`

  返回一个跟踪url的Request实例。

  `url`不仅仅是绝对URL,还可以是相对URL或`scrapy.link.Link`对象。

  TextResponse提供了一个follow()方法，除了绝对/相对URL和链接对象以外，还支持选择器。

## Response subclasses

### TextResponse objects

```text
class scrapy.http.TextResponse(url[, encoding[, ...]])
TextResponse对象为基本Response类添加了编码功能，这意味着该类仅用于二进制数据，如图像，声音或任何媒体文件。
TextResponse对象在基础Response对象之上还添加了新的构造函数参数。其余功能与Response类相同，这里不再赘述。

encoding(string):一个包含此响应编码的字符串。如果使用unicode主体创建一个TextResponse对象，它将使用此编码进行编码（记住body属性始终是一个字符串）。如果encoding为None（默认值），则将在响应头文件和正文中查找编码。
```

* `.text`

  响应主体。

  与`response.body.decode（response.encoding）`相同，但结果在第一次调用后被缓存，因此可以多次访问`response.text`而无需额外的开销。

* `.encoding`

  响应编码的字符串。 顺序地通过以下机制尝试解决编码问题：

  1. 构造函数的参数encoding；
  2. Content-Type HTTP头中声明的编码；
  3. 响应正文中声明的编码。 TextResponse类没有为此提供任何特殊功能。 但是，`HtmlResponse`和`XmlResponse`类可以；
  4. 通过查看响应主体来推断编码。 这是更脆弱的方法，但也是最后的尝试。

* `.selector`

  用响应作为目标的`Selector`实例。选择器在第一次访问时延迟实例化。

* `.xpath(query)`

  `TextResponse.selector.xpath(query)`的快捷方式： `response.xpath('//p') `  

* `.css(query)`

  `TextResponse.selector.css(query)`的快捷方式： `response.css('p') `  

* `.follow(url, callback=None, method='GET', headers=None, body=None, cookies=None, meta=None, encoding=None, priority=0, dont_filter=False, errback=None)`

* `. body_as_unicode()`

  与`text`相同，但可作为方法使用。 这个方法保持向后兼容；请优先使用`response.text`。

### HtmlResponse objects

```text
class scrapy.http.HtmlResponse(url[, ...])
HtmlResponse类是TextResponse的一个子类，添加了通过查看HTML meta http-equiv属性自动发现支持编码。 参见TextResponse.encoding。
```

## XmlResponse objects

```text
class scrapy.http.XmlResponse(url[, ...])
XmlResponse类是TextResponse的一个子类，添加了通过查看XML声明行来自动发现支持编码。 参见TextResponse.encoding。
```

***

参考：

https://docs.scrapy.org/en/latest/topics/request-response.html#request-subclasses