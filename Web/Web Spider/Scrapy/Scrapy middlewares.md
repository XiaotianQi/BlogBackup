 downloader middleware 和 spider middleware 在scrapy框架中的位置。二者都可以被视为 scrapy 的扩展。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Web%20Spider/Scrapy%20Data%20flow.png)

## downloader middleware

它是处于 `Engine` 和 `Downloader` 之间的组件，可以用于处理从 `Engine` 传递给 `Downloader` 的 request和从 `Downloader` 传递给 `Engine` 的 response。它一般可用于：

* 处理即将发到网络上的请求；
* 修改传递即将给 Spider 的响应数据；
* 丢掉响应数据，然后生成一个新的请求；
* 根据请求凭空构造一个响应（并不发出实际的请求）；
* 丢弃某些请求等等。

其中最主要的方法便是：process_request 和 process_response

* process_request：当每个request通过下载中间件时，该方法被调用
* process_response：当下载器完成http请求，传递响应给引擎的时候调用

settings.py 中的相关设置：

```python
DOWNLOADER_MIDDLEWARES = {
    'NewsSpider.middlewares.NewsspiderDownloaderMiddleware': 543,
}
```

middlewares.py 中的相关设置：

```python
class NewsspiderDownloaderMiddleware(object):
    # Not all methods need to be defined. If a method is not defined,
    # scrapy acts as if the downloader middleware does not modify the
    # passed objects.

    @classmethod
    def from_crawler(cls, crawler):
        # This method is used by Scrapy to create your spiders.
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened)
        return s

    def process_request(self, request, spider):
        # Called for each request that goes through the downloader
        # middleware.

        # Must either:
        # - return None: continue processing this request
        # - or return a Response object
        # - or return a Request object
        # - or raise IgnoreRequest: process_exception() methods of
        #   installed downloader middleware will be called
        return None

    def process_response(self, request, response, spider):
        # Called with the response returned from the downloader.

        # Must either;
        # - return a Response object
        # - return a Request object
        # - or raise IgnoreRequest
        return response

    def process_exception(self, request, exception, spider):
        # Called when a download handler or a process_request()
        # (from other downloader middleware) raises an exception.

        # Must either:
        # - return None: continue processing this exception
        # - return a Response object: stops process_exception() chain
        # - return a Request object: stops process_exception() chain
        pass

    def spider_opened(self, spider):
        spider.logger.info('Spider opened: %s' % spider.name)

```

***

## spider middleware

处于 `Engine` 和 `Spider` 之间的组件，可以用于处理 `Spider` 的输入（response）和输出（item和 request）。它一般可以用于：

* 处理 Spider 回调函数的输出，可以用于修改、增加和删除 request  或者 item；
* 处理 `Spider.start_requests()` 函数生成的 request；
* 捕捉 Spider 回调函数抛出的异常等等。

settings.py 中的相关设置：

```python
SPIDER_MIDDLEWARES = {
    'NewsSpider.middlewares.NewsspiderSpiderMiddleware': 543,
}
```

middlewares.py 中的相关设置：

```python
class NewsspiderSpiderMiddleware(object):
    # Not all methods need to be defined. If a method is not defined,
    # scrapy acts as if the spider middleware does not modify the
    # passed objects.

    @classmethod
    def from_crawler(cls, crawler):
        # This method is used by Scrapy to create your spiders.
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened)
        return s

    def process_spider_input(self, response, spider):
        # Called for each response that goes through the spider
        # middleware and into the spider.

        # Should return None or raise an exception.
        return None

    def process_spider_output(self, response, result, spider):
        # Called with the results returned from the Spider, after
        # it has processed the response.

        # Must return an iterable of Request, dict or Item objects.
        for i in result:
            yield i

    def process_spider_exception(self, response, exception, spider):
        # Called when a spider or process_spider_input() method
        # (from other spider middleware) raises an exception.

        # Should return either None or an iterable of Response, dict
        # or Item objects.
        pass

    def process_start_requests(self, start_requests, spider):
        # Called with the start requests of the spider, and works
        # similarly to the process_spider_output() method, except
        # that it doesn’t have a response associated.

        # Must return only requests (not items).
        for r in start_requests:
            yield r

    def spider_opened(self, spider):
        spider.logger.info('Spider opened: %s' % spider.name)

```

