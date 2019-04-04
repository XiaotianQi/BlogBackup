## 准备

在目标路径中，创建虚拟环境：

```powershell
mkvirtualenv --python=C:\Python\Python37\python.exe news
```

安装依赖：

* 安装 `scrapy`

可以使用豆瓣源安装：

```powershell
(news) λ pip install -i https://pypi.douban.com/simple/ scrapy
```

如果提示 `error: Microsoft Visual C++ 14.0 is required.`，那么独立安装 Twisted 包，即可解决。

可以在 `https://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted` 中找到 python 包。

PS：虚拟环境包：

* `virtualenv`
* `virtualenvwrapper`（或者 `virtualenvwrapper-win`）

***

## 创建一个项目

在目标路径，虚拟环境中：

```cmd
(news) λ scrapy startproject NewsSpider
```

便会出现创建成功信息：

```powershell
New Scrapy project 'NewsSpider', using template directory 'c:\python\envs\news\lib\site-packages\scrapy\templates\project', created in:
    C:\GitHub\spiders\NewsSpider

You can start your first spider with:
    cd NewsSpider
    scrapy genspider example example.com
```

### 文件结构

![简单的文件结构](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Web%20Spider/scrapy%20files-1.png)

```text
NewsSpider/
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

## 创建一个spider

 根据项目创建成功信息，在`NewsSpider`文件夹中，输入提示命令：

```powershell
(news) λ cd NewsSpider\
(news) λ scrapy genspider sina https://news.sina.com.cn/roll
```

`NewsSpider\NewsSpider\spiders\`文件夹中，目录新增 sina.py 文件。

### 创建 main.py 文件

在`NewsSpider\`文件夹中，创建 main.py 文件，便于调试。

```python
# _*_coding:utf-8_*_

import os
import sys
from scrapy.cmdline import execute

sys.path.append(os.path.dirname(os.path.abspath(__file__)))
execute(['scrapy', 'crawl', 'sina'])
```

### 文件结构

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Web%20Spider/scrapy%20files-2.png)

## scrapy shell

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

对指定 url 进行调试。

```text
response.xpath(...)
response.css(...)
```

