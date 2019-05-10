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

## Request & Response

```python
class Request(object_ref):

    def __init__(self, url, callback=None, method='GET', headers=None, body=None,
                 cookies=None, meta=None, encoding='utf-8', priority=0,
                 dont_filter=False, errback=None, flags=None):
```

```python
class Response(object_ref):

    def __init__(self, url, status=200, headers=None, body=b'', flags=None, request=None):
```

***

## 数据收集器

Stats Collectors

```python
# chinanews.py

class ChinanewsSpider(scrapy.Spider):
    ...
    #收集伯乐在线所有404的url以及404页面数
    handle_httpstatus_list = [404,]
    
    def __init__(self, **kwargs):
        self.fail_urls = []

    
    def parse(self, response):
        if response.status == 404:
            self.fail_urls.append(response.url)
            self.crawler.stats.inc_value("failed_url")
    ...
```

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

## 扩展

Extensions，基于信号机制。

***

参考：

https://docs.scrapy.org/en/latest/topics/architecture.html