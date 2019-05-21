```bash
scrapy <command> [options] [args]
```

全局命令：

    startproject
    genspider
    settings
    runspider
    shell
    fetch
    view
    version

```text
startproject--->scrapy startproject <project_name> [project_dir]
在project_dir目录下创建一个名为project_name的新Scrapy项目。
如果未指定project_dir，project_dir将与project_name相同。

genspider--->scrapy genspider [-t template] <name> <domain>
如果在项目中调用，则在当前文件夹或当前项目的spiders文件夹中创建一个新Spider。
<name> 参数用来设置Spider的 name, <domain>设置allowed_domains 和 start_urls

settings--->scrapy settings [options]
获取Scrapy设置值。
如果在项目中使用，它将显示项目设置值，否则将显示该设置的默认剪贴值。
例如：scrapy settings --get BOT_NAME

runspider--->scrapy runspider <spider_file.py>
运行一个Python文件中的Spider，而且不必创建一个项目。

shell--->scrapy shell [url]
调试指定的url，还支持UNIX风格的本地文件路径，./或../前缀的相对路径或绝对路径。
--spider=SPIDER：绕过Spider自动检测并强制使用特定的Spider
-c code：获取shell中的代码，打印结果并退出
--no-redirect：不follow HTTP 3xx重定向（默认是follow它们）；这只会影响您在命令行上作为参数传递的URL；在shell运行时，fetch(url)默认仍然会follow HTTP重定向。

fetch--->scrapy fetch <url>
采用spider的设置，使用Scrapy下载器下载给定的URL，并将内容写入标准输出。
--spider=SPIDER：绕过spider自动检测，并强制使用特定的spider
--headers ：打印响应的HTTP headers，而不是响应的正文
--no-redirect：不follow HTTP 3xx重定向（默认是follow它们）
--nolog：不使用日志
例如：scrapy fetch --nolog --headers http://www.example.com/

view--->scrapy view <url>
查看页面，就像Scrapy Spider获取的那样。有时候，Spider获取的网页与通过浏览器获得的不同，因此可以用来检查Spider“看到”了什么，并确认它是否是期望的。
例如：爬取动态网页信息
--spider=SPIDER：绕过Spider自动检测并强制使用特定的Spider
--no-redirect：不follow HTTP 3xx重定向（默认是follow它们）

version--->scrapy version [-v]
打印Scrapy版本。 如果与-v一起使用，它还会打印Python，Twisted和Platform信息，这对于错误报告很有用。
```

仅限项目的命令：

    crawl
    check
    list
    edit
    parse
    bench
```text
crawl--->scrapy crawl <spider>
开始使用Spider爬取

check--->scrapy check [-l] <spider>
运行检查

list--->scrapy list
列出当前项目中所有可用的Spider。

edit--->scrapy edit <spider>
使用EDITOR环境变量中指定的编辑器，编辑爬虫

parse--->scrapy parse <url> [options]
爬取给定的URL，并使用spider解析，使用—callback选项传递的方法，如果没有给出，则解析。
--spider=SPIDER：绕过spider自动检测，并强制使用特定的spider
--a NAME=VALUE：设置spider参数（可以重复）
--callback或-c：解析spider响应的回调方法
--meta或-m：通过回调请求传回附加请求元标签。 这必须是有效的json字符串。 例如：-meta ='{“foo”：“bar”}'
--pipelines：通过pipeline处理item
--rules或-r：使用CrawlSpider规则来发现用于解析响应的回调（即spider方法）
--noitems：不显示被抓到的item
--nolinks：不显示提取的链接
--nocolour：避免使用pygments对输出着色
--depth或-d：递归请求后的深度级别（默认值：1）
--verbose或-v：显示每个深度级别的信息

bench--->scrapy bench
运行快速基准测试。
```

日志相关：

```text
--logfile FILE
	重载 LOG_FILE

--loglevel/-L LEVEL
	重载 LOG_LEVEL

--nolog
	设置LOG_ENABLED 为 False，不启用日志
```

## 创建

创建项目：

```bash
scrapy startproject NewsSpider
```

创建爬虫：

在项目目录下创建

```bash
scrapy genspider sina news.sina.com.cn/roll
```

指定模版创建：

```bash
scrapy genspider -t crawl xxx xxx
```

## 启动爬虫

```bash
scrapy crawl chinanews
```

## 调试

```text
# 命令行模式
scrapy shell http://www.chinanews.com/scroll-news/news1.html

# 输出如下提示
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x000002A83CC619E8>
[s]   item       {}
[s]   request    <GET http://www.chinanews.com/scroll-news/news1.html>
[s]   response   <200 http://www.chinanews.com/scroll-news/news1.html>
[s]   settings   <scrapy.settings.Settings object at 0x000002A83CC61748>
[s]   spider     <ChinanewsSpider 'chinanews' at 0x2a83d6e3908>
[s] Useful shortcuts:
[s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
[s]   fetch(req)                  Fetch a scrapy.Request and update local objects
[s]   shelp()           Shell help (print this help)
[s]   view(response)    View response in a browser
```

对指定 url 进行调试，用于调试CSS和XPath表达式来抓取数据，在编写或调试spider时非常有用。

```text
response.xpath(...)
response.css(...)
```

提取信息：

`.extract()` 和 `.extract_first()`

```cmd
# 返回 SelectorList 对象的列表
In [1]: response.css('title')	
Out[1]: [<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]

# 返回所有字符串列表
In [2]: response.css('title::text').extract()
Out[2]: ['Quotes to Scrape']

# 返回列表中第一个对象	
In [3]: response.css('title::text').extract_first()
Out[3]: 'Quotes to Scrape'

# 同上，但会发生IndexError并返回None。
In [4]: response.css('title::text')[0].extract()
Out[4]: 'Quotes to Scrape'
```

`.getall()` 和 `.get()`

```bash
In [5]: response.css('title::text').getall()
Out[5]: ['Quotes to Scrape']

In [6]: response.css('title::text').get()
Out[6]: 'Quotes to Scrape'
```

`.re()`

```bash
In [7]: response.css('title::text').re(r'(\w+) to (\w+)')
Out[7]: ['Quotes', 'Scrape']
```

`view(response)`

下载该页面源码，并查看。

```bash
In [8]: view(response)
Out[8]: True
```

除此之外，如果通过浏览器自带的 Network 找到正确的request，获得例如json或者xml数据格式，不仅获取信息方便，还避免了爬取动态网页。

## 存储抓取的数据

```bash
scrapy crawl chinanews -o chinanews.json
```

将生成一个`chinanews.json`文件，其中包含所有序列化为JSON的已抓取项目。

由于历史原因，Scrapy附加到给定文件而不是覆盖其内容。 如果在第二次执行该命令前未移除该文件，则会生成一个损坏的JSON文件。

可以使用其他格式，如JSON Lines：

```bash
scrapy crawl chinanews -o chinanews.jsonl
```

JSON行格式非常有用，因为它类似于流，可以轻松地向其添加新记录。 当运行两次时，它不会发生和JSON相同的问题。 另外，由于每条记录都是一条独立的行，因此可以处理大文件而不必将所有内容都放在内存中。

To dump into a JSON file:

```bash
scrapy crawl myspider -o items.json
```

To dump into a CSV file:

```bash
scrapy crawl myspider -o items.csv
```

To dump into a XML file:

```bash
scrapy crawl myspider -o items.xml
```

## 暂停与重启

```bash
scrapy crawl chinanews -s JOBDIR=job_info/chinanews/001
```

## scrapy shell

如果安装了IPython，Scrapy shell将使用它（而不是标准的Python控制台）。

或者，在scrapy.cfg中定义它：

```text
[settings]
shell = ipython
```

启动 scrapy shell

```bash
scrapy shell <url>
```

想要抓取的链接，也适用于本地文件。

```text
shelp()：打印可用对象和方法列表帮助

fetch(url[, redirect=True])：从URL获取新的响应，并相应地更新相关的对象。通过redirect=False使HTTP 3xx重定向不传递

fetch(request)：从请求中获取新的响应，并相应地更新相关对象。

view(response)：在本地网络浏览器中打开响应，以便进行检查。 这将添加一个<base>标签便于外部链接(例如图片或样式表)正常显示. 但请注意，这将在您的计算机中创建一个临时文件，该文件不会被自动删除。
```
```text
crawler：当前的Crawler对象。

spider：已知处理URL的Spider,如果没有为当前URL指定Spider,则为Spider对象.

request：最后获取页面的Request对象。

response：最后获取页面的Response对象

settings：当前Scrapy设置
```

***

参考：

https://docs.scrapy.org/en/latest/topics/commands.html