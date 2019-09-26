`urllib` 库包含一些处理 URLs 的模块：

>* `urllib.request` for opening and reading URLs;
>* `urllib.error` containing the exceptions raised by urllib.request;
>* `urllib.parse` for parsing URLs;
>* `urllib.robotparser` for parsing `robots.txt` files.

## 发送  HTTP 请求

`urllib.request` 模块提供了最基本的构造 HTTP 请求的方法，可以模拟浏览器请求发起过程，同时还可以处理授权验证（authenticaton）、重定向（redirection)、浏览器Cookies以及其他内容。

### urllib.request.urlopen

```python
urllib.request.urlopen(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None)
```

除了第一个 `url` 参数是必选参数，其他都是可选。如果传递了 `data` 参数，那么请求方式就不再是 `GET`，而是 `POST`。`timeout` 参数设置请求超时时间，单位为秒。如果请求超出了设置时间，还没有得到响应，就会抛出异常。如果不指定该参数，就使用全局默认时间。

返回 `HTTPResponse` 对象，主要包含 `.read()`、`.readinto()`、`.getheader(name)`、`.getheaders()`、`.fileno()` 等方法，以及`.msg`、`.version`、`.status`、`.reason`、`.debuglevel`、`.closed` 等属性。

```python
In [33]: response = urllib.request.urlopen('https://www.python.org')

In [34]: type(response)
Out[34]: http.client.HTTPResponse

In [35]: response.status
Out[35]: 200

In [36]: response.getheaders()
Out[36]:
[('Server', 'nginx'),
 ('Content-Type', 'text/html; charset=utf-8'),
 ('X-Frame-Options', 'SAMEORIGIN'),
 ('x-xss-protection', '1; mode=block'),
 ('X-Clacks-Overhead', 'GNU Terry Pratchett'),
 ('Via', '1.1 varnish'),
 ('Content-Length', '48825'),
 ('Accept-Ranges', 'bytes'),
 ('Date', 'Mon, 06 Aug 2018 06:40:10 GMT'),
 ('Via', '1.1 varnish'),
 ('Age', '3134'),
 ('Connection', 'close'),
 ('X-Served-By', 'cache-iad2149-IAD, cache-itm18824-ITM'),
 ('X-Cache', 'HIT, HIT'),
 ('X-Cache-Hits', '2, 1325'),
 ('X-Timer', 'S1533537610.189584,VS0,VE0'),
 ('Vary', 'Cookie'),
 ('Strict-Transport-Security', 'max-age=63072000; includeSubDomains')]
```

### urllib.request.Request

```python
In [37]: request = urllib.request.Request('https://python.org')

In [38]: type(request)
Out[38]: urllib.request.Request

In [39]: response = urllib.request.urlopen(request)

In [41]: response.getheaders()
Out[41]:
[('Server', 'nginx'),
 ('Content-Type', 'text/html; charset=utf-8'),
 ('X-Frame-Options', 'SAMEORIGIN'),
 ('x-xss-protection', '1; mode=block'),
 ('X-Clacks-Overhead', 'GNU Terry Pratchett'),
 ('Via', '1.1 varnish'),
 ('Content-Length', '48825'),
 ('Accept-Ranges', 'bytes'),
 ('Date', 'Mon, 06 Aug 2018 06:53:14 GMT'),
 ('Via', '1.1 varnish'),
 ('Age', '317'),
 ('Connection', 'close'),
 ('X-Served-By', 'cache-iad2122-IAD, cache-itm18828-ITM'),
 ('X-Cache', 'HIT, HIT'),
 ('X-Cache-Hits', '1, 136'),
 ('X-Timer', 'S1533538394.077397,VS0,VE0'),
 ('Vary', 'Cookie'),
 ('Strict-Transport-Security', 'max-age=63072000; includeSubDomains')]
```

虽然仍然使用 `urlopen()` 方法来发送请求，但是这次参数不再是 `URL`，而是一个 `Request` 对象。

如果请求需要加入 `Headers` 等信息，就可以使用的 `urllib.request.Request` 。通过构造`Request` 对象，可以将请求独立成一个对象，并自定义配置参数。

```python
class urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None
```

* `url`：用于请求 `URL`，必选参数，其他都是可选参数；
* `data`：同 `urlopen` 参数 `data`；
* `headers`：请求头，是个字典。常用的用法就是通过修改 `User-Agent` 来伪装浏览器，`User-Agent` 默认是 `Python-urllib`；
* `origin_req_host`：请求方的 host 名称或者 IP 地址；
* `unverifiable`：请求是否是无法验证的，默认是 `False`。用户没有足够权限来选择接收这个请求的结果；
* `method`：字符串，用来指示请求使用的方法，比如 `GET`、`POST`、`PUT` 等。

```python
url = 'http://httpbin.org/post'
headers = {
    'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)'
}
dict = {
    'name': 'Germey'
}
data = bytes(parse.urlencode(dict), encoding='utf8')
req = request.Request(url=url, data=data, headers=headers, method='POST')
response = request.urlopen(req)
print(str(response.read().decode('utf-8')))
```

```json
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "name": "Germey"
  }, 
  "headers": {
    "Accept-Encoding": "identity", 
    "Connection": "close", 
    "Content-Length": "11", 
    "Content-Type": "application/x-www-form-urlencoded", 
    "Host": "httpbin.org", 
    "User-Agent": "Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)"
  }, 
  "json": null, 
  "origin": "123.113.218.182", 
  "url": "http://httpbin.org/post"
}
```

也可以通过如下方式进行修改 `headers`：

```python
req = request.Request(url=url, data=data, method='POST')
req.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)')
```

***

## URL 处理

### URL 拆分

#### urllib.parse.urlparse

```python
urllib.parse.urlparse(urlstring, scheme='', allow_fragments=True)
```

实现 `URL` 的识别和拆分。返回值是一个 `ParseResult` 对象，包含6部分，分别是 `scheme`、`netloc`、`path`、`params`、`query` 和 `fragment`。

![属性表](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/urllib-1.png)

```python
In [60]: url = 'http://netloc/path;param?query=arg#frag'

In [61]: parsed = parse.urlparse(url)

In [62]: type(parsed)
Out[62]: urllib.parse.ParseResult

In [63]: parsed
Out[63]: ParseResult(scheme='http', netloc='netloc', path='/path', params='param', query='query=arg', fragment='frag')
```

`ParseResult` 是基于 `tuple` 的一个子类 `namedtupl`e 实现。因此可以名称或索引来获取 `URL` 中各部分属性的值。

```python
In [64]: parsed.netloc
Out[64]: 'netloc'

In [65]: parsed[1]
Out[65]: 'netloc'
```

#### urllib.parse.urlsplit

和 `urlparse()` 方法非常相似，只不过它不再单独解析params这一部分，只返回 5 个结果，不再有 `params` 属性。

![属性表](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/urllib-2.png)

```python
urllib.parse.urlsplit(urlstring, scheme='', allow_fragments=True)
```

```python
In [74]: parsed = parse.urlsplit(url)

In [75]: parsed
Out[75]: SplitResult(scheme='http', netloc='netloc', path='/path;param', query='query=arg', fragment='frag')
```

### URL 拼接

#### urllib.parse.urlunparse

```python
urllib.parse.urlunparse(parts)
```

`parts` 参数是一个可迭代对象，并且其长度必须是 6。

```python
In [63]: parsed
Out[63]: ParseResult(scheme='http', netloc='netloc', path='/path', params='param', query='query=arg', fragment='frag')

In [67]: parse.urlunparse(parsed)
Out[67]: 'http://netloc/path;param?query=arg#frag'
```

#### urllib.parse.urlunsplit

与 `urlsplit` 对应，传入参数长度必须是 5。

#### urllib.parse.urljoin

```python
urllib.parse.urljoin(base, url, allow_fragments=True)
```

`base` 提供了三项内容 `scheme`、`netloc` 和 `path`。如果这3项在新的链接里不存在，就予以补充。如果新的链接存在，就使用新的链接的部分，`base` 中对应项将被替换。`base` 中的 `params`、`query`和`fragment`是不起作用的。

```python
In [69]: parse.urljoin('http://www.baidu.com', 'FAQ.html')
Out[69]: 'http://www.baidu.com/FAQ.html'

In [70]: parse.urljoin('http://www.baidu.com', 'https://cuiqingcai.com/FAQ.html')
Out[70]: 'https://cuiqingcai.com/FAQ.html'

In [71]: parse.urljoin('http://www.baidu.com/about.html', 'https://cuiqingcai.com/FAQ.html?question=2')
Out[71]: 'https://cuiqingcai.com/FAQ.html?question=2'

In [72]: parse.urljoin('http://www.baidu.com?wd=abc', 'https://cuiqingcai.com/index.php')
Out[72]: 'https://cuiqingcai.com/index.php'

In [73]: parse.urljoin('www.baidu.com#comment', '?category=2')
Out[73]: 'www.baidu.com?category=2'
```

#### geturl

只对 `urlparse` 或 `urlsplit` 返回的对象有效。

```python
In [76]: parsed.geturl()
Out[76]: 'http://netloc/path;param?query=arg#frag'
```

***

参考：

[urllib — URL handling modules](https://docs.python.org/3/library/urllib.html)，官方文档

[Python3网络爬虫开发实战 3.1.1-发送请求](https://cuiqingcai.com/5500.html)，崔庆才