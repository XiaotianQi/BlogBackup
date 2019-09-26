增量爬取，一般两类情况：

* 网站出现了新的页面
* 页面内容变更

当出现新的数据时，重复数据不会抓取，直接抓取新的内容。

***

## 出现新页面

### scrapy 去重策略

scrapy通过`request_fingerprint`函数，对Request对象生成指纹，看注释：

```python
# 该函数在scrapy/utils/request.py文件中
def request_fingerprint(request, include_headers=None):
    if include_headers:
        include_headers = tuple(to_bytes(h.lower())
                                 for h in sorted(include_headers))
    cache = _fingerprint_cache.setdefault(request, {})
    if include_headers not in cache:
        fp = hashlib.sha1()
        """计算指纹时，请求方法(如GET、POST)被计算在内"""
        fp.update(to_bytes(request.method)) 

        """下面这句有意思，canonicalize_url()将url规范化，意味着
          http://www.example.com/query?id=111&cat=222
          http://www.example.com/query?cat=222&id=111
        这样参数位置变化，但参数值不变的网址，表示的仍是同一个网址，符合现实逻辑。
         """
        fp.update(to_bytes(canonicalize_url(request.url)))

        """request.body的属性是字符串：
        一般GET方法的body为空字符串，不考虑；
        而POST方法要上传一个字典data（类型是dict），
        要经过urllib.parse.urlencode()函数转换后才能变成request.body
       """
        fp.update(request.body or b'')
        if include_headers:
            for hdr in include_headers:
                if hdr in request.headers:
                    fp.update(hdr)
                    for v in request.headers.getlist(hdr):
                        fp.update(v)
        cache[include_headers] = fp.hexdigest()
    return cache[include_headers]
"""我们甚至可以根据需求将request.meta的内容作为指纹计算的一部分"""
```

scrapy生成的唯一指纹，存在内存的一个集合里，即set。如果下一次请求产生的指纹在这个set里面，请求被判定为重复，这次请求就被忽略，也就是所谓的**去重**了。

从上面可以可出，scrapy认为，如果url/POST、data/method都一致，这个请求就是重复的，这适合绝大多数情况。

需要提一下：上述的处理方式，意味着想要变更request的指纹就要改变request，即是在downloaderMiddleware 的process_request方法中变更。

### 举个栗子

一个网站，本来一共有10页，过段时间之后变成了100页。假设，已经爬取了前10页，为了增量爬取，我们现在只想爬取第11-100页。

因此，为了增量爬取，我们需要将前10页请求的**指纹**保存下来。

以下命令是将内存中的set里指纹保存到本地硬盘的一种方式。

```bash
scrapy crawl somespider -s JOBDIR=crawls/somespider-1
```

但还有更常用的，是将scrapy中的指纹存在一个redis数据库中，这个操作已经有造好轮子了，即scrapy-redis库。

scrapy-redis库将指纹保存在了redis数据库中，是可以持久保存的。（基于此，还可以实现分布式爬虫，那是另外一个用途了）

scrapy-redis库不仅存储了已请求的指纹，还存储了带爬取的请求，这样无论这个爬虫如何重启，每次scrapy从redis中读取要爬取的队列，将爬取后的指纹存在redis中。如果要爬取的页面的指纹在redis中就忽略，不在就爬取。

***

## 页面内容变更

### 爬取修改中的博客

刚开始爬时，博客写了一段就交了，后来某天，这个博客又续写了，这就是网页url没变，内容变了的情况，一般这种网页不常抓。

根据增量爬取的思路，需要保存老页面的信息，如果再次访问这个页面，内容变化，就更新内容即可。

有个小TIP，比如一个页面是文字，很大，可以将文字哈希，下一次爬取的页面哈希值与上一次相同就说明页面没有更改。

### ajax页面中，页面内容不断更新

比如豆瓣TOP100 的电影之类的，url/参数/请求方法 可能都一样，但内容是变更的。

ajax请求获得的json文件一般会有一个唯一id，如果没有可以自己造一个，将唯一id记录在redis数据库中，如果这个json返回值已经爬取过，丢弃即可。一般没有办法直接通过fingerprint过滤，总是要加入其它一些人为的考虑在里面。

因为是将东西爬取下来之后才去重，意味着此步骤在pipeline的process_item方法中进行。

***

参考：

[基于python的scrapy爬虫，关于增量爬取是怎么处理](https://www.zhihu.com/question/19793879/answer/312467126)，乌尔班