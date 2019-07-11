## URL 获取

网站的 URL 是树形结构，可以采取以下两种方法获取。

`BFS` 核心在于 队列，取出队列中第一个元素。

`DFS` 核心在于 堆栈，取出栈中最后被压入的元素。

以如下树结构为例：

```python
gragh = {
    'A':['B', 'C'],
    'B':['A', 'C', 'D'],
    'C':['A', 'B', 'D', 'E'],
    'D':['B', 'C', 'E', 'F'],
    'E':['C', 'D'],
    'F':['D']
}
```

### 广度优先

```python
def bfs(gragh, node='A'):
    queue = []
    queue.append(node)
    seen = set()
    seen.add(node)
    while len(queue) > 0:
        vertex = queue.pop(0)
        nodes = gragh[vertex]
        for node in nodes:
            if node not in seen:
                queue.append(node)
                seen.add(node)
        yield vertex	# 'A', 'B', 'C', 'D', 'E', 'F'
```

### 深度优先

```python
def dfs(gragh, node='A'):
    stack = []
    stack.append(node)
    seen = set()
    seen.add(node)
    while len(stack)>0:
        vertex = stack.pop()
        nodes = gragh[vertex]
        for node in nodes:
            if node not in seen:
                stack.append(node)
                seen.add(node)
        yield vertex	# 'A', 'C', 'E', 'D', 'F', 'B'
```

`BFS` 和 `DFS` 二者是队列和堆栈的体现形式。在代码实现中，唯一的区别就是：

```python
vertex = queue.pop(0)  # BFS
vertex = queue.pop()   # DFS
```

对于知乎这样的问答网站，使用的深度优先：

```python
# zhihu.py
    def parse(self, response):
        """
        提取出html页面中的所有url 并跟踪这些url进行一步爬取
        如果提取的url中格式为 /question/xxx 就下载之后直接进入解析函数
        """
        all_urls = response.css("a::attr(href)").extract()
        all_urls = [parse.urljoin(response.url, url) for url in all_urls]
        all_urls = filter(lambda x:True if x.startswith("https") else False, all_urls)
        for url in all_urls:
            match_obj = re.match("(.*zhihu.com/question/(\d+))(/|$).*", url)
            if match_obj:
                #如果提取到question相关的页面则下载后交由提取函数进行提取
                request_url = match_obj.group(1)
                yield scrapy.Request(request_url, headers=self.headers, callback=self.parse_question)
            else:
                #如果不是question页面则直接进一步跟踪
                yield scrapy.Request(url, headers=self.headers, callback=self.parse)
```

默认情况下，Scrapy使用LIFO queue 存储挂起的请求，这基本上意味着它按照DFO顺序爬行。如果确实想按真正的BFO顺序爬行，可以通过设置以下设置来实现：

```python
DEPTH_PRIORITY = 1
SCHEDULER_DISK_QUEUE = 'scrapy.squeues.PickleFifoDiskQueue'
SCHEDULER_MEMORY_QUEUE = 'scrapy.squeues.FifoMemoryQueue'
```

***

## URL 去重

1. 将已访问的URL存入数据库
   * 便于实现
   * 每次通过数据库查询 URL，效率低
2. 将已访问的URL存入 set
   * 查询的复杂度仅是$O(1)$
   * 随着URL数量变多，内存占用会越来越大
3. URL经md5等方法哈希后，存入 set
   * 查询的复杂度仅是$O(1)$
   * 哈希后，很大程度节省内存
4. URL通过bitmap方法哈希
   * 进一步节省内存
   * 哈希冲突较多
5. URL通过bloomfilter对bitmap方法进行改进
   * 节省内存的同时，降低多重哈希函数冲突

***

## Broad Crawls

Broad Crawls 搜索引擎使用的典型爬虫。其典型特点：

* 抓取许多域(通常是无限制的)，而不是一组特定的站点
* 不需要对域名完全抓取，而是按时间或抓取页数来限制抓取
* 逻辑上更简单（与具有许多提取规则的非常复杂的Spider相反），因为数据通常在单独的阶段中进行后处理
* 同时抓取多个域

Scrapy默认设置是为focused crawls优化的，而不是Broad Crawls。然而，由于它的异步架构，Scrapy非常适合执行快速的广泛抓取。通过以下几点进行优化：

### 提高并发性

增加多少将取决于有多少CPU可用，选择CPU在使用率80-90%时的并发数

```python
CONCURRENT_REQUESTS = 100
```

### 增加 Twisted IO 线程池

目前，Scrapy使用线程池以阻塞的方式进行DNS解析。增加处理DNS查询的线程数量。

```python
REACTOR_THREADPOOL_MAXSIZE = 20
```

### 设置 DNS

如果有多个爬行进程和一个中心DNS，它可以像DoS攻击DNS服务器一样导致整个网络变慢，甚至阻塞计算机。设置专属的DNS服务器。

### 降低日志级别

在进行Broad Crawls时，通常只对获取的抓取速率和发现的任何错误感兴趣。当使用信息日志级别时，这些统计数据由Scrapy报告。为了节省CPU(和日志存储需求)，降低日志级别。

```python
LOG_LEVEL = 'INFO'
```

### 禁用 cookies

原因同上。

```python
COOKIES_ENABLED = False
```

### 禁用重试

重试失败的HTTP请求会显著降低爬行速度。

```python
RETRY_ENABLED = False
```

### 减少下载超时

减少下载超时，以便快速丢弃卡住的请求，并释放处理下一个请求的资源。

```python
DOWNLOAD_TIMEOUT = 15
```

### 禁用重定向

当进行Broad Crawls时，通常保存重定向并在以后的抓取中重新访问站点时解析它们。这还有助于保持每个爬行批处理的请求数量不变，否则重定向循环可能会导致爬行器在任何特定域中占用太多资源。

```python
REDIRECT_ENABLED = False
```

### 开启Ajax Crawlable Pages

```python
AJAXCRAWL_ENABLED = True
```

***

参考：

https://docs.scrapy.org/en/latest/topics/broad-crawls.html