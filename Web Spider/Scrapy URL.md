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