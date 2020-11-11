## settings 相关

### 禁用 cookies

```text
COOKIES_ENABLED = False
如果目标网站不需要登录，那么设置False，防止被跟踪，减少被发现概率
有些网站可能使用cookie来识别机器人行为
```

### 限速

```python
# 设置下载延迟
DOWNLOAD_DELAY = 5

# 开始自动限速
AUTOTHROTTLE_ENABLED = True
# The initial download delay
#AUTOTHROTTLE_START_DELAY = 5
# The maximum download delay to be set in case of high latencies
#AUTOTHROTTLE_MAX_DELAY = 60
# The average number of requests Scrapy should be sending in parallel to
# each remote server
#AUTOTHROTTLE_TARGET_CONCURRENCY = 1.0
# Enable showing throttling stats for every response received:
#AUTOTHROTTLE_DEBUG = False
```

***

## USE_AGENT

修改 settings 文件中：

```python
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:66.0) Gecko/20100101 Firefox/66.0'
```

### 随机更换 USE_AGENT

依赖第三方包 fake-useragent

在 middleware 中：

```python
from fake_useragent import UserAgent

class RandomUserAgentMiddleware(object):
    # 随机更换 user-agent
    def __init__(self, crawler):
        super().__init__()
        self.ua = UserAgent()
        self.ua_type = crawler.settings.get('UA_TYPE', 'random')

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler)

    def process_request(self, request, spider):
        def get_ua():
            return getattr(self.ua, self.ua_type)
        request.headers.setdefault('User-Agent', get_ua())
```

相应的 settings 文件：

```python
#随机USER_AGENT浏览器类型
UA_TYPE = 'random'

DOWNLOADER_MIDDLEWARES = {
    #'NewsSpider.middlewares.NewsspiderDownloaderMiddleware': 543,
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None,
    'NewsSpider.middlewares.RandomUserAgentMiddleware': 1,
}
```

增加 `UA_TYPE` 项，便于控制生成的浏览器类型。

***

## IP 池

### 从西刺代理获取IP

```python
# -*- coding: utf-8 -*-
import requests
import json
import asyncio
from scrapy.selector import Selector
from random import choice

async def get_ip():
    # 获取ip,并存入json
    headers = {
        'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.86 Safari/537.36',
    }
    res = requests.get('https://www.xicidaili.com/nn/', headers=headers)
    selector = Selector(res)
    all_trs = selector.css('#ip_list tr')
    ip_list = []
    for tr in all_trs[1:]:
        all_text = tr.css('td::text').extract()
        ip = all_text[0]
        port = all_text[1]
        proxy_type = all_text[5]
        speed = tr.css('.bar::attr(title)').extract()[0]
        if speed:
            speed = float(speed.split("秒")[0])
        if speed < 2:
            verify = await verify_ip(ip, port, proxy_type)
            if verify:
                ip_list.append((ip, port, proxy_type, speed))
    with open(r'tools/ips.json', 'w') as f:
        json.dump(ip_list, f)

async def verify_ip(ip, port, proxy_type):
    # 验证ip是否可用
    http_url = 'https://www.baidu.com/'
    proxy_url = '{}://{}:{}'.format(proxy_type, ip, port)
    try:
        proxies = {
            proxy_type: proxy_url,
        }
        print('START:', proxies)
        response = requests.get(http_url, proxies=proxies)
    except Exception as e:
        print("invalid ip and port")
        return False
    else:
        code = response.status_code
        if code >= 200 and code < 300:
            print("effective ip")
            return True
        else:
            print("invalid ip and port")
            return False

def get_random_ip(afresh=False):
    if afresh:
        loop = asyncio.get_event_loop()
        loop.run_until_complete(get_ip())
        loop.close
    with open(r'tools/ips.json', 'r') as f:
        ips = json.load(f)
    return choice(ips)

if __name__ == "__main__":
    print(get_random_ip()
```

middlewares 中设置：

```python
from tools.xici_daili import get_random_ip


class RandomProxyMiddleware(object):
    #动态设置ip代理
    def process_request(self, request, spider):
        request.meta["proxy"] = get_random_ip()
```

### crawlera代理设置

crawlera [官网](https://scrapinghub.com/crawlera)

[Using Crawlera with Python Requests](https://support.scrapinghub.com/support/solutions/articles/22000203567-using-crawlera-with-python)

```text
import requests

url = "http://twitter.com"
proxy = "paygo.crawlera.com:8010"
proxy_auth = "<API KEY>"

proxies = {
    "http": "http://{0}@{1}/".format(proxy_auth, proxy)
}

r = requests.get(url, proxies=proxies, verify=False)
```

[Using Crawlera with Scrapy](https://support.scrapinghub.com/support/solutions/articles/22000188399-using-crawlera-with-scrapy)

```text
安装
pip install scrapy-crawlera

修改settings.py
DOWNLOADER_MIDDLEWARES = {
	'scrapy_crawlera.CrawleraMiddleware': 300
	}
CRAWLERA_ENABLED = True
CRAWLERA_APIKEY = '<API key>'
```

***

## 解析动态内容

有些网站其内容或部分内容是通过JavaScript动态生成的，这就意味着在浏览器窗口中“查看网页源代码”时无法在HTML代码中找到这些内容，也就是说我们之前用的抓取数据的方式无法正常运转了。解决这样的问题基本上有两种方案：

* JavaScript逆向工程
* 渲染JavaScript，获得渲染后的内容

***

## Selenium

尽管很多网站对自己的网络API接口进行了保护，增加了获取数据的难度。

Selenium 框架底层使用JavaScript模拟真实用户对浏览器进行操作。以Chrome浏览器为例。

### 通过 Chromedriver 自动打开Chrome

爬虫文件：

```python
# -*- coding: utf-8 -*-
import scrapy
from selenium import webdriver

class SinaSpider(scrapy.Spider):
    # 省略...

    def start_requests(self):
        browser = webdriver.Chrome(executable_path=r'C:\GitHub\spiders\NewsSpider\tools\chromedriver.exe')
        browser.get('https://xxx')
        return [scrapy.Request(url=self.start_urls[0], dont_filter=True)]
```

### 手动启动 Chrome

有些网站会阻止，通过Chromedriver自动打开chrome浏览器的访问。那么，现在通过手动启动chrome浏览器。

通过cmd打开chrome

```powershell
# 在chrome 所在文件夹执行
chrome.exe --remote-debugging-port=9222
```

**NOTE**:启动之前，必须确保所有chrome实例已经关闭

此时，通过`127.0.0.1:9222/json`，能过正确访问，就表示启动成功。

爬虫文件：

```python
# -*- coding: utf-8 -*-
import scrapy
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

class SinaSpider(scrapy.Spider):
    # 省略...

    def start_requests(self):
        chrome_option = Options()
        chrome_option.add_argument('--disable-extensions')
        chrome_option.add_experimental_option('debuggerAddress', '127.0.0.1:9222')
        browser = webdriver.Chrome(
            executable_path=r'C:\GitHub\spiders\NewsSpider\tools\chromedriver.exe',
            chrome_options=chrome_option)
        browser.get('https://xxx')
        
        return [scrapy.Request(url=self.start_urls[0], dont_filter=True)]
```

### headless 模式

HEADLESS BROWSER 指的是不需要用户界面的浏览器，这种浏览器在自动化测试和爬虫领域有着广泛的应用。

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_argument('--headless')
options.add_argument('--disable-gpu') # 允许在无GPU的环境下运行，可选
options.add_argument('--window-size=1920x1080') # 建议设置
driver = webdriver.Chrome(chrome_options=chrome_options)
```

### 不加载图片

```python
chrome_options = webdriver.ChromeOptions()
prefs = {"profile.managed_default_content_settings.images": 2}
chrome_options.add_experimental_option("prefs", prefs)
browser = webdriver.Chrome(chrome_options=chrome_options)
```

### 模拟登录

模拟登录。

爬虫文件：

```python
# -*- coding: utf-8 -*-
import scrapy
import time
from selenium import webdriver

class SinaSpider(scrapy.Spider):
    # 省略...

    def start_requests(self):
        browser = webdriver.Chrome(executable_path=r'C:\GitHub\spiders\NewsSpider\tools\chromedriver.exe')
        browser.get('https://xxx')	# 登录页面URL
        browser.find_element_by_css_selector("xxx").send_keys(Keys.CONTROL + "a")
        browser.find_element_by_css_selector("xxx").send_keys("xxx")
        browser.find_element_by_css_selector("xxx").send_keys(Keys.CONTROL + "a")
        browser.find_element_by_css_selector("xxx").send_keys("xxx")
        browser.find_element_by_css_selector("xxx").click()
        time.sleep(10)

        return [scrapy.Request(url=self.start_urls[0], dont_filter=True)]
```

### 存储 cookies

登录成功后，存储登录后的cookies，从而实现之后访问时，自动登录

爬虫文件：

```python
# -*- coding: utf-8 -*-
import scrapy
import pickle
import time
from selenium import webdriver

class SinaSpider(scrapy.Spider):
    # 省略...

    def start_requests(self):
        ...
		cookies = browser.get_cookies()
        cookie_dict = {}
        for cookie in cookies:
			f = open(r'xxx' + cookie['name'] + '.sina', 'wb')
            pickle.dump(cookie, f)
        f.close()
        cookie_dict[cookie['name']] = cookie['value']
        browser.close()
```

同时，需要将 settings 文件中，设置：

```python
COOKIES_ENABLED = True
```

### 将 selenium 集成在 scrapy

首先，在 middleware 中：

```python
from selenium import webdriver
from scrapy.http import HtmlResponse

class JSPageMiddleware(object):
    # 通过selenium 请求动态网页
    def process_request(self, request, spider):
        if spider.name == "sina":
            spider.browser.get(request.url)
            import time
            time.sleep(5)
            return HtmlResponse(
                url=spider.browser.current_url,
                body=spider.browser.page_source,
                encoding="utf-8",
                request=request)
```

然后，sina.py 脚本爬虫中：

```python
from selenium import webdriver
from scrapy import signals
from scrapy.xlib.pydispatch import dispatcher

class SinaSpider(scrapy.Spider):
    # 省略...

    custom_settings = {
        'DOWNLOADER_MIDDLEWARES':{
            'NewsSpider.middlewares.JSPageMiddleware':1,
        }
    }
    
    def __init__(self):
        self.browser = webdriver.Chrome(executable_path=r'C:\GitHub\spiders\NewsSpider\tools\chromedriver.exe')
        super().__init__()
        dispatcher.connect(self.spider_closed, signals.spider_closed)
    
    def spider_closed(self, spider):
        print('Spider closed')
        self.browser.quit()
        
    # 省略...
```

***

参考：

https://coding.imooc.com/learn/list/92.html