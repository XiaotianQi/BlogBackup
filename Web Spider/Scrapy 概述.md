Scrapy是一种用于抓取网站和提取结构化数据的应用程序框架，例如用于数据挖掘，信息处理或历史存档。而且，scrapy 也是异步框架。

Scrapy是用纯Python编写的，并且依赖于几个关键的Python包：

- [lxml](http://lxml.de/)，一种高效的XML和HTML解析器
- [parsel](https://pypi.python.org/pypi/parsel)，一个基于lxml的HTML/XML数据提取库
- [w3lib](https://pypi.python.org/pypi/w3lib)，一款用于处理网址和网页编码的多用途帮手
- [twisted](https://twistedmatrix.com/)，一个异步网络框架
- [cryptography](https://cryptography.io/)和[pyOpenSSL](https://pypi.python.org/pypi/pyOpenSSL)，用来处理各种网络级安全需求

## 文件结构

![简单的文件结构](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Web%20Spider/scrapy%20files-1.png)

```text
NewsSpider/						# 项目根目录
	scrapy.cfg            		# 部署配置文件
    NewsSpider/
        spiders/   	  		 	# 项目的Python模块,将在这里输入代码
            __init__.py
            items.py          	# items定义文件，数据保存的格式
            middlewares.py   	# 中间件，灵活控制scrapy
            pipelines.py     	# 管道，数据的存储
            settings.py    	  	# 设置
            spiders/    	    # 存储各类spider的文件夹
                __init__.py
```

为了便于调试，在`NewsSpider\`文件夹中，创建 main.py 文件。

```python
# _*_coding:utf-8_*_

import os
import sys
from scrapy.cmdline import execute

sys.path.append(os.path.dirname(os.path.abspath(__file__)))
execute(['scrapy', 'crawl', 'sina'])
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Web%20Spider/scrapy%20files-2.png)

***

## 数据流

Scrapy的体系结构，以及它的组件之间的交互。数据流如下图：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Web%20Spider/Scrapy%20Data%20flow.png)

Scrapy中的数据流由执行引擎控制，如下所示：

1. Engine 从Spider中获取初始Requests，并进行爬取。
2. Engine通过Scheduler调度请求，并 asks for the next Requests to crawl。
3. Scheduler将下一个Requests返回给Engine。
4. Engine通过Downloader Middlewares将Requests发送到Downloader。
5. 一旦页面下载完成，Downloader就会生成一个该页面Response，并通过Downloader Middlewares将其发送到Engine（参见`process_response() `）。
6. Engine接收由Downloader发送来的Response，并将该Response通过Spider Middleware发送至Spider，进行处理（参见`process_spider_input()`）。
7. Spider处理Response后，通过Spider Middleware，将已抓取的Item和一个新Requests返回Engine (参阅`process_spider_output()`）。
8. Engine将已处理的Item发送到Item Pipelines，然后将处理后的新Requests发送到Scheduler，并asks for possible next Requests to crawl。
9. 重复这个过程（由步骤1开始），直到 there are no more requests from the Scheduler.

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Web%20Spider/Scrapy%20Data%20flow-1.png)

**Scrapy Engine**：

负责控制系统所有组件之间的数据流，并在某些操作发生时触发事件。

**Scheduler**：

接收来自引擎的请求，并对它们排入队列，以便稍后Engine请求它们时返回。

**Downloader**：

负责抓取网页内容，并将其提供给Engine，而Engine又将其提供给Spiders。

**Spiders**：

Spider是由用户编写的自定义类，用于解析responses ，并从中提取Item，或追加额外的requests 。

**Item Pipeline**：

负责处理由spiders提取得到的item。主要的功能是清理、持久化、验证。

**Downloader middlewares**：

位于Engine和Downloader之间的钩子，负责处理由从Engine传递到Downloader的requests，以及从Downloader传递到Engine的responses。

需要执行下列操作时，可以使用Downloader middlewares：

- process a request just before it is sent to the Downloader (i.e. right before Scrapy sends the request to the website);
- change received response before passing it to a spider;
- send a new Request instead of passing received response to a spider;
- pass response to a spider without fetching a web page;
- silently drop some requests.

**Spider middlewares**：

位于Engine和Spider之间的钩子，能够处理Spider输入(responses)和输出(items and
requests)。

需要使用Spider middlewares的几种情况：

- post-process output of spider callbacks - change/add/remove requests or items;
- post-process start_requests;
- handle spider exceptions;
- call errback instead of callback for some of the requests based on response content.

**Event-driven networking**：

Scrapy是使用Twisted编写的，Twisted是一种流行的Python事件驱动的网络框架。因此，Scrapy使用非阻塞(即异步)代码实现并发。

***

## 信号

Signals。middlewares 和 扩展的桥梁。

将数据收集器示例中的代码改写：

```python
# chinanews.py

class ChinanewsSpider(scrapy.Spider):
    ...
    #收集伯乐在线所有404的url以及404页面数
    handle_httpstatus_list = [404,]
    
    def __init__(self, **kwargs):
        self.fail_urls = []
		dispatcher.connect(self.handle_spider_closed, signals.spider_closed)
         
    def handle_spider_closed(self, spider, reason):
        self.crawler.stats.set_value("failed_urls", ",".join(self.fail_urls))
    
    def parse(self, response):
        if response.status == 404:
            self.fail_urls.append(response.url)
            self.crawler.stats.inc_value("failed_url")
    ...
```

***

## 可扩展组件

Spider Middleware：处于 `Engine` 和 `Spider` 之间的组件，可以用于处理 `Spider` 的输入（response）和输出（item和 request）。

Downloader Middleware：处于 `Engine` 和 `Downloader` 之间的组件，可以用于处理从 `Engine` 传递给 `Downloader` 的 request和从 `Downloader` 传递给 `Engine` 的 response。

Item Pipeline：用于处理 Spider 生成的 item，对其进行清理、验证、持久化等处理。

Extensions，基于信号机制。

***

## 内置异常处理

* DropItem

```python
 scrapy.exceptions.DropItem
```

必须由 item pipeline 阶段引发的异常，以停止处理项。

* CloseSpider

```text
scrapy.exceptions.CloseSpider(reason='cancelled')

reason (str)：关闭原因
```

这个异常可以从Spider回调中抛出以请求关闭/停止Spider。 

* DontCloseSpider

```python
scrapy.exceptions.DontCloseSpider
```

可以在`spider_idle`信号处理程序中引发，以防止Spider被关闭。

* IgnoreRequest

```python
scrapy.exceptions.IgnoreRequest
```

Scheduler 或任何 downloader middleware 都可以引发此异常，以表明请求应被忽略。

* NotConfigured

```python
scrapy.exceptions.NotConfigured
```

某些组件可能会引发此异常，以表明它们将保持禁用状态。 这些组件包括：

* 扩展
* Item pipelines
* Downloader middlewares
* Spider middlewares

必须在组件的`__init__`方法中引发异常。

***

## 内置服务

### 日志

Scrapy使用 python logging标准库记录日志。日志级别、使用均与标准库相同。

Scrapy为每个Spider实例提供一个logger，可以像这样访问和使用它：

```python
import scrapy

class MySpider(scrapy.Spider):

    name = 'myspider'
    start_urls = ['https://scrapinghub.com']

    def parse(self, response):
        self.logger.info('Parse function called on %s', response.url)
```

该日志记录器使用spider的名称创建，也可以自定义Python日志记录器。例如：

```python
import logging
import scrapy

logger = logging.getLogger('mycustomlogger')

class MySpider(scrapy.Spider):

    name = 'myspider'
    start_urls = ['https://scrapinghub.com']

    def parse(self, response):
        logger.info('Parse function called on %s', response.url)
```

日志可以开箱即用，并且可以在 settings 中进行配置。

```text
LOG_FILE		# 日志文件保存路径
LOG_ENABLED		# 是否启用日志
LOG_ENCODING
LOG_LEVEL		# 过滤级别
LOG_FORMAT		# 格式
LOG_DATEFORMAT	# 时间格式
LOG_STDOUT
LOG_SHORT_NAMES
```

可用的命令行参数，使用它们来覆盖与日志记录相关的scrapy设置：

```text
--logfile FILE
	重载 LOG_FILE

--loglevel/-L LEVEL
	重载 LOG_LEVEL

--nolog
	设置LOG_ENABLED 为 False，不启用日志
```

自定义日志：

```text
scrapy.utils.log.configure_logging(settings=None, install_root_handler=True)
为Scrapy初始化日志默认值。

settings(dict, Settings object or None)：为根记录器创建和配置处理程序的设置(默认值:None)。

install_root_handler(bool)：是否使用根日志处理程序(默认值:True) 
```

```python
import logging
from scrapy.utils.log import configure_logging

configure_logging(install_root_handler=False)
logging.basicConfig(
    filename='log.txt',
    format='%(levelname)s: %(message)s',
    level=logging.INFO
)
```

### 信息统计

Scrapy提供了一个方便的功能，以键/值的形式收集统计数据，其中值通常是计数器。可以通过Crawler API的`stats`属性进行访问。

统计收集器为每个打开的Spider存放一个统计表，当Spider打开时它会自动打开，当Spider关闭时会自动关闭。

通过crawler的stats属性访问stats收集器。

```python
class ExtensionThatAccessStats(object):

    def __init__(self, stats):
        self.stats = stats

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.stats)
    
    def parse(self, response):
        stats.set_value('hostname', socket.gethostname()) 	# 设置值
        stats.inc_value('custom_count')	# 增加值
        stats.max_value('max_items_scraped', value) # 最大值
        stats.min_value('min_free_memory_percent', value) # 最小值
```

可以在 shell 中获取值：

```shell
>>> stats.get_value('custom_count')	# 获取单个

>>> stats.get_stats()	# 获取全部
```

收集所有404的url以及404页面数：

```python
# chinanews.py

class ChinanewsSpider(scrapy.Spider):
    ...
    # 收集所有404的url以及404页面数
    handle_httpstatus_list = [404,]
    
    def __init__(self, **kwargs):
        self.fail_urls = []

    
    def parse(self, response):
        if response.status == 404:
            self.fail_urls.append(response.url)
            self.crawler.stats.inc_value("failed_url")
    ...
```

### 发送邮件

虽然Python通过smtplib标准库发送电子邮件很容易，但是Scrapy提供了自己的发送电子邮件的工具，使用起来非常方便，并且使用Twisted非阻塞IO来实现，以避免干扰爬虫的非阻塞IO。它还提供了一个用于发送附件的简单API，通过一些设置，它非常容易配置。

实例化 mail sender 的两种方法：

```python
from scrapy.mail import MailSender
mailer = MailSender()

# 通过Scrapy settings对象实例化它，该对象将采用这些设置
mailer = MailSender.from_settings(settings)
```

使用：

```python
mailer.send(to=["someone@example.com"], subject="Some subject", body="Some body", cc=["another@example.com"])
```

MailSender：

```text
：class scrapy.mail.MailSender(smtphost=None, mailfrom=None, smtpuser=None, smtppass=None, smtpport=None)
MailSender是用于从Scrapy发送电子邮件的首选类，因为它使用了Twisted非阻塞IO，就像框架的其他部分一样。

smtphost(str or bytes)：发送电子邮件的SMTP主机。如果省略，将使用MAIL_HOST设置。

mailfrom(tr)：用于发送电子邮件的地址(in the From: header). 如果省略，将使用MAIL_FROM设置。

smtpuser：SMTP用户。如果省略，将使用MAIL_USER设置。如果没有给出，则不执行SMTP身份验证。

smtppass(str or bytes)：SMTP通过身份验证。

smtpport(int)：连接到的SMTP端口

smtptls(boolean)：强制使用SMTP STARTTLS

smtpssl(boolean)：强制使用安全SSL连接
```

* ` classmethod from_settings(settings)`

  使用Scrapy settings 实例化对象

  settings (scrapy.settings.Settings object) ：电子邮件收件人

* ` send(to, subject, body, cc=None, attachs=(), mimetype='text/plain', charset=None)`

  向指定的收件人发送电子邮件。

```text
to(str or list of str)：收件人

subject(str)：主题

cc(str or list of str)：抄送

body(str)：正文

attachs(iterable)：元组(attach name, mimetype, file object)。其中attach name是一个字符串，其名称将出现在电子邮件附件中，mimetype是附件的mimetype, file对象是一个可读的文件对象，其中包含附件的内容

mimetype(str)：MIME类型

charset(str)：字符编码
```

Mail settings：

```text
MAIL_FROM
Default: 'scrapy@localhost'
发件人

MAIL_HOST
Default: 'localhost'
发送电子邮件的SMTP主机

MAIL_PORT
Default: 25
发送电子邮件的SMTP端口。

MAIL_USER
Default: None
用于SMTP身份验证的用户。如果禁用SMTP，则不会执行任何SMTP身份验证。

MAIL_PASS
Default: None
SMTP身份验证的密码

MAIL_TLS
Default: False
使用STARTTLS

MAIL_SSL
Default: False
强制使用SSL加密连接
```

***

## 调试

除了常用调试方法之外，scrapy 提供了其他的方法：

### Scrapy Shell

```python
from scrapy.shell import inspect_response

def parse(self, response):
    item = response.meta.get('item', None)
    if item:
        # populate more `item` fields
        return item
    else:
        inspect_response(response, self)
```

需要再命令行中运行爬虫，才会进入 scrapy shell 模式。

### 在浏览器中打开需要调试URL

```python
from scrapy.utils.response import open_in_browser

def parse(self, response):
    if "item name" not in response.body:
        open_in_browser(response)
```

### Scrapy 内置日志

也就是上面提到的内置服务-日志。

***

参考：

https://docs.scrapy.org/en/latest/topics/architecture.html

https://docs.scrapy.org/en/latest/topics/exceptions.html