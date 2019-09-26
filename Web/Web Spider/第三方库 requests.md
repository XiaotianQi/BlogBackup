## 常用方法

### 发送请求

Requests 提供了几乎所有HTTP动词的功能：GET、OPTIONS、HEAD、POST、PUT、PATCH、DELETE。

```python
r = requests.get('http://httpbin.org/get')
r = requests.put('http://httpbin.org/put', data = {'key':'value'})
r = requests.delete('http://httpbin.org/delete')
r = requests.head('http://httpbin.org/get')
r = requests.options('http://httpbin.org/get')
```

#### 传递 URL 参数

以键/值对的形式

```bash
In [10]: payload = {'key1': 'value1', 'key2': 'value2'}

In [11]: r = requests.get("http://httpbin.org/get", params=payload)

In [12]: r.url
Out[12]: 'http://httpbin.org/get?key1=value1&key2=value2'
```

dict 的 value 是列表时：

```text
In [13]: payload = {'key1': 'value1', 'key2': ['value2', 'value3']}

In [14]: r = requests.get('http://httpbin.org/get', params=payload)

In [15]: r.url
Out[15]: 'http://httpbin.org/get?key1=value1&key2=value2&key2=value3'
```

### 响应内容

#### 文本响应内容

读取服务器响应的内容，Requests 会自动解码来自服务器的内容。

```bash
In [19]:r = requests.get('https://www.baidu.com')

In [20]: r.text
Out[20]: '<!DOCTYPE html>......</html>\r\n'
```

#### 二进制响应内容

以字节的方式访问请求响应体

```text
In [21]:r = requests.get('https://api.github.com/events')

In [22]: r.json() 
Out[22]:b'<!DOCTYPE html>......</html>\r\n'
```

#### JSON 响应内容

需要网站支持，比如百度就无法使用。

```bash
In [23]:r = requests.get('https://api.github.com/events')

In [24]:r.json()
Out[22]: .......
```

### 定制请求头

为请求添加 HTTP 头部，只要简单地传递一个 `dict` 给 `headers` 参数就可以了。

```bash
In [26]: url = 'https://api.github.com/some/endpoint'

In [27]: headers = {'user-agent': 'my-app/0.0.1'}

In [28]: r = requests.get(url, headers=headers)
```

NOTE:所有的 header 值必须是 `string`、`bytestring` 或者 `unicode`。尽管传递 unicode header 也是允许的，但不建议这样做。

Custom headers are given less precedence than more specific sources of information. 

* Authorization headers set with headers= will be overridden if credentials are specified in `.netrc`, which in turn will be overridden by the  `auth=` parameter.
* Authorization headers will be removed if you get redirected off-host.
* Proxy-Authorization headers will be overridden by proxy credentials provided in the URL.
* Content-Length headers will be overridden when we can determine the length of the content.

#### 更加复杂的 POST 请求

传递一个字典给 data 参数

```bash
In [29]: payload = {'key1': 'value1', 'key2': 'value2'}

In [30]: r = requests.post("http://httpbin.org/post", data=payload)

In [31]: r.json()
Out[31]:
{
  ...
  'form': {'key1': 'value1', 'key2': 'value2'},
  ...
}
```

可以为 `data` 参数传入一个元组列表。在表单中多个元素使用同一 key 的时候，这种方式尤其有效：

```bash
In [34]: payload = (('key1', 'value1'), ('key1', 'value2'))

In [35]: r = requests.post('http://httpbin.org/post', data=payload)

In [36]: r.json()
Out[36]:
{
  ...
 'form': {'key1': ['value1', 'value2']},
  ...
}
```

#### 查看发送到服务器的请求的头部

响应头部信息

```bash
In [59]: r = requests.get('http://httpbin.org/get')

In [60]: r.headers
Out[60]: {'Access-Control-Allow-Credentials': 'true', 'Access-Control-Allow-Origin': '*', 'Content-Encoding': 'gzip', 'Content-Type': 'application/json', 'Date': 'Mon, 08 Apr 2019 06:56:22 GMT', 'Server': 'nginx', 'Content-Length': '183', 'Connection': 'keep-alive'}
```

请求的头部信息

```bash
In [61]: r.request.headers
Out[61]: {'User-Agent': 'python-requests/2.21.0', 'Accept-Encoding': 'gzip, deflate', 'Accept': '*/*', 'Connection': 'keep-alive'}
```

### 状态码查询

检测响应状态码，以及内置的状态码查询对象

```bash
In [38]: r = requests.get('http://httpbin.org/get')

In [39]: r.status_code
Out[39]: 200

In [40]: r.status_code == requests.codes.ok
Out[40]: True
```

### 代理

为任意请求方法提供 `proxies` 参数来配置单个请求:

```python
proxies = {
  "http": "http://10.10.1.10:3128",
  "https": "http://10.10.1.10:1080",
}

requests.get("http://example.org", proxies=proxies)

# HTTP Basic Auth，可以使用 http://user:password@host/ 语法
proxies = {
    "http": "http://user:pass@10.10.1.10:3128/",
}

# 为某个特定的连接方式或者主机设置代理
# 使用 scheme://hostname 作为 key
# 它会针对指定的主机和连接方式进行匹配
proxies = {'http://10.20.1.128': 'http://10.10.1.10:5323'}
```

Note:代理 URL 必须包含连接方式。

### 响应头

查看以一个 Python 字典形式展示的服务器响应头

```bash
In [47]: r.headers
Out[47]: {'Access-Control-Allow-Credentials': 'true', 'Access-Control-Allow-Origin': '*', 'Content-Encoding': 'gzip', 'Content-Type': 'application/json', 'Date': 'Mon, 08 Apr 2019 06:44:29 GMT', 'Server': 'nginx', 'Content-Length': '183', 'Connection': 'keep-alive'}

In [48]: r.headers['Content-Type']
Out[48]: 'application/json'
```

### 重定向与请求历史

默认情况下，除了 HEAD, Requests 会自动处理所有重定向。

可以使用响应对象的 `history` 方法来追踪重定向。

Response.history 是一个 Response 对象的列表，为了完成请求而创建了这些对象。这个对象列表按照从最老到最近的请求进行排序。

```bash
In [49]: r = requests.get('http://github.com')

In [50]: r.url
Out[50]: 'https://github.com/'

In [51]: r.status_code
Out[51]: 200

In [52]: r.history
Out[52]: [<Response [301]>]
```

通过 `allow_redirects`参数禁用重定向处理：

```bash
In [53]: r = requests.get('http://github.com', allow_redirects=False)

In [54]: r.url
Out[54]: 'http://github.com/'

In [55]: r.history
Out[55]: []
```

### 超时（timeout）

为防止服务器不能及时响应，大部分发至外部服务器的请求都应该带着 timeout 参数。在默认情况下，除非显式指定了 timeout 值，requests 是不会自动进行超时处理的。如果没有 timeout，代码可能会挂起若干分钟甚至更长时间。

**连接超时**指的是在你的客户端实现到远端机器端口的连接时（对应的是`connect()`），Request 会等待的秒数。

一旦你的客户端连接到了服务器并且发送了 HTTP 请求，**读取超时**指的就是客户端等待服务器发送请求的时间。

```text
# 单一的值,默认是连接超时
r = requests.get('https://github.com', timeout=5)

# connect 和 read 超时
r = requests.get('https://github.com', timeout=(3.05, 27))
```

***

## API

### Main Interface

Requests 所有的功能都可以通过以下 7 个方法访问。它们全部都会返回一个 `Response` 对象的实例。

```text
requests.request(method, url, **kwargs)

Constructs and sends a Request.
Returns:Response object
Parameters:

method – method for the new Request object.

url – URL for the new Request object.

params – (optional) Dictionary, list of tuples or bytes to send in the query string for the Request.

data – (optional) Dictionary, list of tuples, bytes, or file-like object to send in the body of the Request.

json – (optional) A JSON serializable Python object to send in the body of the Request.

headers – (optional) Dictionary of HTTP Headers to send with the Request.

cookies – (optional) Dict or CookieJar object to send with the Request.

files – (optional) Dictionary of 'name': file-like-objects (or {'name': file-tuple}) for multipart encoding upload. file-tuple can be a 2-tuple ('filename', fileobj), 3-tuple ('filename', fileobj, 'content_type') or a 4-tuple ('filename', fileobj, 'content_type', custom_headers), where 'content-type' is a string defining the content type of the given file and custom_headers a dict-like object containing additional headers to add for the file.

auth – (optional) Auth tuple to enable Basic/Digest/Custom HTTP Auth.

timeout (float or tuple) – (optional) How many seconds to wait for the server to send data before giving up, as a float, or a (connect timeout, read timeout) tuple.

allow_redirects (bool) – (optional) Boolean. Enable/disable GET/OPTIONS/POST/PUT/PATCH/DELETE/HEAD redirection. Defaults to True.

proxies – (optional) Dictionary mapping protocol to the URL of the proxy.

verify – (optional) Either a boolean, in which case it controls whether we verify the server’s TLS certificate, or a string, in which case it must be a path to a CA bundle to use. Defaults to True.

stream – (optional) if False, the response content will be immediately downloaded.

cert – (optional) if String, path to ssl client cert file (.pem). If Tuple, (‘cert’, ‘key’) pair.
```

```text
requests.head(url, **kwargs)[source]
Sends a HEAD request.

requests.get(url, params=None, **kwargs)[source]
Sends a GET request.
params – (optional) Dictionary, list of tuples or bytes to send in the query string for the Request.

requests.post(url, data=None, json=None, **kwargs)[source]
Sends a POST request.
data – (optional) Dictionary, list of tuples, bytes, or file-like object to send in the body of the Request.
json – (optional) json data to send in the body of the Request.

requests.put(url, data=None, **kwargs)[source]
Sends a PUT request.
同 .post
```

任何时候进行了类似 requests.get() 的调用，都在做两件主要的事情。

- 其一，你在构建一个 Request 对象，
  该对象将被发送到某个服务器请求或查询一些资源。
- 其二，一旦 `requests` 得到一个从服务器返回的响应就会产生一个 `Response` 对象。该响应对象包含服务器返回的所有信息，也包含你原来创建的 `Request` 对象。

### Lower-Level Classes

#### Requset

```text
class requests.Request(method=None, url=None, headers=None, files=None, data=None, params=None, auth=None, cookies=None, hooks=None, json=None)[source]

A user-created Request object.

Used to prepare a PreparedRequest, which is sent to the server.
Parameters:	

method – HTTP method to use.
url – URL to send.
headers – dictionary of headers to send.
files – dictionary of {filename: fileobject} files to multipart upload.
data – the body to attach to the request. If a dictionary or list of tuples [(key, value)] is provided, form-encoding will take place.
json – json for the body to attach to the request (if files or data is not specified).
params – URL parameters to append to the URL. If a dictionary or list of tuples [(key, value)] is provided, form-encoding will take place.
auth – Auth handler or (user, pass) tuple.
cookies – dictionary or CookieJar of cookies to attach to this request.
hooks – dictionary of callback hooks, for internal usage.
```

#### Response

```text
class requests.Response[source]

The Response object, which contains a server’s response to an HTTP request.
apparent_encoding:The apparent encoding, provided by the chardet library.

.close()
Releases the connection back to the pool. Once this method has been called the underlying raw object must not be accessed again.
Note: Should not normally need to be called explicitly.

.text
Content of the response, in unicode.
If Response.encoding is None, encoding will be guessed using chardet.
The encoding of the response content is determined based solely on HTTP headers, following RFC 2616 to the letter. If you can take advantage of non-HTTP knowledge to make a better guess at the encoding, you should set r.encoding appropriately before accessing this property.

.content
Content of the response, in bytes.

.json(**kwargs)
Returns the json-encoded content of a response, if any.
Parameters:	**kwargs – Optional arguments that json.loads takes.
Raises:	ValueError – If the response body does not contain valid json.

.url = None
Final URL location of Response.

.headers = None
Case-insensitive Dictionary of Response Headers. For example, headers['content-encoding'] will return the value of a 'Content-Encoding' response header.

.history = None
 A list of Response objects from the history of the Request. Any redirect responses will end up here. The list is sorted from the oldest to the most recent request.

.status_code = None
Integer Code of responded HTTP Status, e.g. 404 or 200.

.ok
Returns True if status_code is less than 400, False if not.
This attribute checks if the status code of the response is between 400 and 600 to see if there was a client error or a server error. If the status code is between 200 and 400, this will return True. This is not a check to see if the response code is 200 OK.

.raise_for_status()[source]
Raises stored HTTPError, if one occurred.

.links
Returns the parsed header links of the response, if any.

.cookies = None
A CookieJar of Cookies the server sent back.

.encoding = None
 Encoding to decode with when accessing r.text.

.is_permanent_redirect
True if this Response one of the permanent versions of redirect.

.is_redirect
True if this Response is a well-formed HTTP redirect that could have been processed automatically (by Session.resolve_redirects).

.iter_content(chunk_size=1, decode_unicode=False)
Iterates over the response data. When stream=True is set on the request, this avoids reading the content at once into memory for large responses. The chunk size is the number of bytes it should read into memory. This is not necessarily the length of each item returned as decoding can take place.
chunk_size must be of type int or None. A value of None will function differently depending on the value of stream. stream=True will read data as it arrives in whatever size the chunks are received. If stream=False, data is returned as a single chunk.
If decode_unicode is True, content will be decoded using the best available encoding based on the response.
```

***

参考：

http://docs.python-requests.org/zh_CN/latest/