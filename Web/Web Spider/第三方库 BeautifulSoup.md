Beautiful Soup 是一个可以从 HTML 或 XML 文件中提取数据的 Python 库。

## 对象的种类

Beautiful Soup 将复杂 HTML 文档转换成一个复杂的树形结构。每个节点都是 Python 对象，所有对象可以归纳为4种: `Tag`、`NavigableString`、`BeautifulSoup`、`Comment`。

### `tag`

`Tag` 对象与 `XML` 或 `HTML` 原生文档中的 `tag` 相同。其最重要的属性: `name` 和 `attributes`，二者操作方法与字典相同。每个 tag 都有自己的名字,通过 `.name` 来获取。一个 `tag` 可能有很多个属性，可以直接 `.attrs` 取属性。`tag` 的属性可以被添加，删除或修改。

通过 `.name` 获取属性的方式只能获得当前名字的第一个 `tag`。

```python
In [10]: soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')

In [11]: tag = soup.b

In [12]: type(tag)
Out[12]: bs4.element.Tag

In [13]: tag.name
Out[13]: 'b'

In [14]: tag.attrs
Out[14]: {'class': ['boldest']}

In [15]: tag['class']
Out[15]: ['boldest']
```

### `NavigableString`

可以遍历的字符串。字符串常被包含在 tag 内。Beautiful Soup 用 NavigableString 类来包装 tag 中的字符串。

```python
In [16]: tag.string
Out[16]: 'Extremely bold'

In [17]: type(tag.string)
Out[17]: bs4.element.NavigableString
```

一个 NavigableString 字符串与 Python 中的普通字符串相同，并且还支持包含在 遍历文档树 和 搜索文档树 中的一些特性。通过 `str()` 方法可以直接将 `NavigableString` 对象转换成 `Unicode` 字符串：

```python
In [18]: unicode_string = str(tag.string)

In [19]: type(unicode_string)
Out[19]: str
```

tag 中包含的字符串不能编辑，但是可以被替换成其它的字符串，用 `replace_with()` 方法：

```python
In [20]: tag.string.replace_with("No longer bold")
Out[20]: 'Extremely bold'

In [21]: tag
Out[21]: <b class="boldest">No longer bold</b>
```

NavigableString 对象支持 遍历文档树 和 搜索文档树 中定义的大部分属性, 并非全部。尤其是，一个字符串不能包含其它内容(tag能够包含字符串或是其它tag)，字符串不支持 `.contents` 或 `.string` 属性或 `find()` 方法。

### `BeautifulSoup`

`BeautifulSoup` 对象表示的是一个文档的全部内容。大部分时候，可以把它当作 `Tag` 对象，它支持 遍历文档树 和 搜索文档树 中描述的大部分的方法。

因为 `BeautifulSoup` 对象并不是真正的 `HTML` 或 `XML` 的 `tag`，所以它没有 `name` 和 `attribute` 属性，但有时查看它的 `.name` 属性是很方便的。

```python
In [22]: soup.name
Out[22]: '[document]'

In [23]: type(soup)
Out[23]: bs4.BeautifulSoup
```

### `Comment`

Comment 对象是一个特殊类型的 NavigableString 对象，包含文档的注释部分。

```python
In [24]: markup = "<b><!--Hey, buddy. Want to buy a used parser?--></b>"

In [25]: soup = BeautifulSoup(markup)

In [26]: comment = soup.b.string

In [27]: comment
Out[27]: 'Hey, buddy. Want to buy a used parser?'

In [28]: type(comment)
Out[28]: bs4.element.Comment

In [29]: soup.b.prettify()
Out[29]: '<b>\n <!--Hey, buddy. Want to buy a used parser?-->\n</b>'
```

***

## 遍历文档树

### 子节点

一个 Tag 可能包含多个字符串或其它的 Tag，这些都是这个 Tag 的子节点。Beautiful Soup 中字符串节点不支持这些属性,因为字符串没有子节点。

```html
from bs4 import BeautifulSoup

html_doc = """<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>"""

soup = BeautifulSoup(html_doc, 'lxml')
```

#### `contents`

`tag` 的 `.contents` 属性可以将 `tag` 的子节点以列表的方式输出，字符串没有 `.contents` 属性，因为字符串没有子节点。

通过 `.name` 获取属性的方式只能获得当前名字的第一个 `tag`。所以，`head_tag` 和 `title_tag` 均长度为 1 的列表。

```html
In [52]: head_tag = soup.head

In [53]: head_tag
Out[53]: <head><title>The Dormouse's story</title></head>

In [54]: head_tag.contents
Out[54]: [<title>The Dormouse's story</title>]

In [55]: len(head_tag.contents)
Out[55]: 1

In [56]: title_tag = head_tag.contents[0]

In [57]: title_tag
Out[57]: <title>The Dormouse's story</title>

In [58]: type(title_tag)
Out[58]: bs4.element.Tag

In [59]: title_tag.contents
Out[59]: ["The Dormouse's story"]

In [60]: len(title_tag.contents)
Out[60]: 1

In [61]: text = title_tag.contents[0]

In [62]: text
Out[62]: "The Dormouse's story"

In [63]: text = title_tag.contents[0]
In [63]: text.contents
# AttributeError: 'NavigableString' object has no attribute 'contents'
```

BeautifulSoup 对象本身一定会包含子节点,也就是说 `<html>` 标签也是 BeautifulSoup 对象的子节点:

```python
In [8]: len(soup.contents)
Out[8]: 1

In [9]: soup.contents[0].name
Out[9]: 'html'
```

#### `.children`、`.descendants`

`.children` 生成器，可以对 `tag` 的子节点进行循环。

`.descendants` 属性可以对所有 `tag` 的子孙节点进行递归循环。

例如，`<head>` 标签只有一个直接子节点 `<title>`，`<title>` 标签也包含一个子节点：字符串 `“The Dormouse’s story”`，这种情况下字符串 `“The Dormouse’s story”` 也属于 `<head>` 标签的子孙节点。`<head>` 标签只有 1 个子节点,但是有 2 个子孙节点:`<head>`节点和 `<head>` 的子节点。

```html
In [22]: head_tag
Out[22]: <head><title>The Dormouse's story</title></head>

In [23]: for child in head_tag.children:
    ...:     print(child)
    ...:
<title>The Dormouse's story</title>

In [24]: for child in head_tag.descendants:
    ...:     print(child)
    ...:
<title>The Dormouse's story</title>
The Dormouse's story
```

BeautifulSoup 有 1 个直接子节点(`<html>`节点)，却有很多子孙节点：

```python
In [26]: len(list(soup.children))
Out[26]: 1

In [27]: len(list(soup.descendants))
Out[27]: 25
```

#### `.string`、`.strings` 和 `stripped_strings`

如果 `tag` 只有 1 个 `NavigableString` 类型子节点，那么这个 `tag` 可以使用 `.string` 得到子节点。如果tag包含了多个子节点，`tag` 就无法确定 `.string` 方法应该调用哪个子节点的内容，输出结果是 `None`。

```python
In [32]: title_tag.string
Out[32]: "The Dormouse's story"

In [33]: head_tag.string
Out[33]: "The Dormouse's story"

In [34]: soup.body.string

In [35]: print(soup.body.string)
None
```

如果 `tag` 中包含多个字符串，可以使用 `.strings` 来循环获取：

```python
In [36]: for string in soup.strings:
    ...:     print(repr(string))
    ...:
"The Dormouse's story"
'\n'
'\n'
"The Dormouse's story"
'\n'
'Once upon a time there were three little sisters; and their names were\n'
'Elsie'
',\n'
'Lacie'
' and\n'
'Tillie'
';\nand they lived at the bottom of a well.'
'\n'
'...'
```

输出的字符串中可能包含了很多空格或空行，使用 `.stripped_strings` 可以去除多余空白内容：

```python
In [37]: for string in soup.stripped_strings:
    ...:     print(repr(string))
    ...:
"The Dormouse's story"
"The Dormouse's story"
'Once upon a time there were three little sisters; and their names were'
'Elsie'
','
'Lacie'
'and'
'Tillie'
';\nand they lived at the bottom of a well.'
'...'
```

***

### 父节点

#### `.parent` 和 `.parents`

`.parent` 属性来获取某个元素的父节点。

```html
In [39]: title_tag.string.parent
Out[39]: <title>The Dormouse's story</title>

In [40]: title_tag.parent
Out[40]: <head><title>The Dormouse's story</title></head>
```

`.parents` 属性可以递归得到元素的所有父辈节点。

```python
In [42]: for parent in title_tag.string.parents:
    ...:     if parent is None:
    ...:         print(parent)
    ...:     else:
    ...:         print(parent.name)
    ...:
title
head
html
[document]
```

***

### 兄弟节点

因为 `<b>` 标签和 `<c>` 标签是同一层，他们是同一个元素的子节点，所以 `<b>` 和 `<c>` 可以被称为兄弟节点。

```html
In [44]: sibling_soup = BeautifulSoup("<a><b>text1</b><c>text2</c></b></a>", 'lxml')

In [45]: sibling_soup.prettify()
Out[45]: '<html>\n <body>\n  <a>\n   <b>\n    text1\n   </b>\n   <c>\n    text2\n   </c>\n  </a>\n </body>\n</html>'

In [46]: print(sibling_soup.prettify())
<html>
 <body>
  <a>
   <b>
    text1
   </b>
   <c>
    text2
   </c>
  </a>
 </body>
</html>
```

#### `.next_sibling` 和 `.previous_sibling`

```python
In [47]: sibling_soup.b.next_sibling
Out[47]: <c>text2</c>

In [48]: sibling_soup.c.previous_sibling
Out[48]: <b>text1</b>
```

`<b>` 标签有 `.next_sibling` 属性，但是没有 `.previous_sibling` 属性，因为 `<b>` 标签在同级节点中是第一个。同理，`<c>` 标签有 `.previous_sibling` 属性，却没有 `.next_sibling` 属性。

实际文档中的 `tag` 的 `.next_sibling` 和 `.previous_sibling` 属性通常是字符串或空白。

```html
In [49]: link = soup.a

In [50]: link
Out[50]: <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

In [51]: link.next_sibling
Out[51]: ',\n'

In [52]: link.next_sibling.next_sibling
Out[52]: <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
```

#### `.next_siblings` 和 `.previous_siblings`

通过 `.next_siblings` 和 `.previous_siblings` 属性可以对当前节点的兄弟节点迭代输出。

```html
In [59]: for sibling in link.next_siblings:
    ...:     print(repr(sibling))
    ...:
',\n'
<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
' and\n'
<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
';\nand they lived at the bottom of a well.'
```

### 回退和前进

#### `.next_element` 和 `.previous_element`

HTML 解析器把这段字符串转换成一连串的事件，文档被解析成树形结构，所以下一步解析过程应该是当前节点的子节点。

```html
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
```

`.next_element` 属性指向解析过程中下一个被解析的对象(字符串或tag)，结果可能与 `.next_sibling` 相同，但通常是不一样的。`.next_element` 是按照解析的顺序，`.next_sibling` 按照兄弟节点的等级，不会读取类似 `Lacie` 这样的子节点。

```html
In [64]: mid_a_tag = soup.find('a', id='link2')

In [65]: mid_a_tag
Out[65]: <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>

In [66]: mid_a_tag.next_sibling
Out[66]: ' and\n'

In [67]: mid_a_tag.next_element
Out[67]: 'Lacie'

In [68]: mid_a_tag.next_element.next_element
Out[68]: ' and\n'
```

`.previous_element` 属性刚好与 `.next_element` 相反，它指向当前被解析的对象的前一个解析对象。

#### `.next_elements` 和 `.previous_elements`

通过 `.next_elements` 和 `.previous_elements` 的迭代器就可以向前或向后访问文档的解析内容，就好像文档正在被解析一样。

```html
In [73]: for element in mid_a_tag.next_elements:
    ...:     print(repr(element))
    ...:
'Lacie'
' and\n'
<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
'Tillie'
';\nand they lived at the bottom of a well.'
'\n'
<p class="story">...</p>
'...'
```

***

## 搜索文档树

### `find_all`

```python
find_all(name=None, attrs={}, recursive=True, text=None, limit=None, **kwargs)
```

通过标签的名称和属性查找标签，搜索当前节点的所有子节点,孙子节点等。大多数时间，只会用到 `name`、`attrs`、`**kwargs`。

#### `name`

查找所有名字为 `name` 的 `tag`，字符串对象会被自动忽略掉。可以是字符串、正则表达式、列表、函数或者 `True`。`True` 可以匹配任何值。

```html
In [3]: soup.find_all(['a', 'p'])
Out[3]:
[<p class="title"><b>The Dormouse's story</b></p>,
 <p class="story">Once upon a time there were three little sisters; and their names were
 <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
 and they lived at the bottom of a well.</p>,
 <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>,
 <p class="story">...</p>]

In [4]: for tag in soup.find_all(True):
   ...:     print(tag.name)
   ...:
html
head
title
body
p
b
p
a
a
a
p

In [6]: soup.find_all(re.compile(r'b'))
Out[6]:
[<body>
 <p class="title"><b>The Dormouse's story</b></p>
 <p class="story">Once upon a time there were three little sisters; and their names were
 <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
 and they lived at the bottom of a well.</p>
 <p class="story">...</p></body>, <b>The Dormouse's story</b>]
```

获取只有 1 个属性的标签。函数作为 `find_all` 参数的唯一条件就是，传入函数必须把一个标签作为参数，且返回布尔型结果。

```html
In [3]: soup.find_all(lambda tag: len(tag.attrs)==1)
Out[3]:
[<p class="title"><b>The Dormouse's story</b></p>,
 <p class="story">Once upon a time there were three little sisters; and their names were
 <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
 and they lived at the bottom of a well.</p>,
 <p class="story">...</p>]
```

#### `attrs`

查找一个标签的若干属性或属性值。可以是字符串、正则表达式、列表、字典、函数或者 `True`。`attrs` 本身也是由字典封装。

```html
In [17]: soup.find_all('p', {'class':{'title', 'story'}})
Out[17]:
[<p class="title"><b>The Dormouse's story</b></p>,
 <p class="story">Once upon a time there were three little sisters; and their names were
 <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
 and they lived at the bottom of a well.</p>,
 <p class="story">...</p>]

In [20]: soup.find_all('p', re.compile(r'story'))
Out[20]:
[<p class="story">Once upon a time there were three little sisters; and their names were
 <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
 and they lived at the bottom of a well.</p>, <p class="story">...</p>]
```

#### `**kwargs`

可看做 `keywords`。获取指定属性的标签。

下面代码中，若使用 `class='sister'` 则需修改为：`class_='sister'`。`class` 在 Python 中是关键字，使用 `class` 做参数会导致语法错误。

```html
In [6]: soup.find_all(id='link2')
Out[6]: [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

In [9]: soup.find_all(id=re.compile(r'link'))
Out[9]:
[<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
 <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

In [10]: soup.find_all(id=['link1', 'link2'])
Out[10]:
[<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
 <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

In [19]: soup.find_all(class_="sister", id='link2')
Out[19]: [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

#### `text`

通过 `text` 参数可以搜搜文档中的字符串内容。可以是字符串、正则表达式、列表、`True`。

```html
In [23]: soup.find_all('a', text='Lacie')
Out[23]: [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

#### 'recursive`

递归参数 `recursive` 是个布尔变量，默认查找标签参数的所有子标签。

#### `limit`

`limit=k` 表示获取网页中前k个结果。

`limit=1` 等价于 `find`。

***

### `find`

`find` 等价于 `find_all` 参数中 `limit=1`。

```html
In [26]: soup.find('p', class_='story')
Out[26]:
<p class="story">Once upon a time there were three little sisters; and their names were
<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
```

***

### 父节点与兄弟节点

`find_next_siblings()` 和 `find_next_sibling()`、`find_previous_siblings()` 和 `find_previous_sibling()`

对象不能把自己作为兄弟标签，任何时候获取一个标签的兄弟标签，都不会包含这个标签本身。

***

## 输出

### `prettify`

`prettify()` 方法将 Beautiful Soup 的文档树格式化后以Unicode编码输出，每个 XML/HTML 标签都独占一行。

```html
In [8]: print(soup.p.prettify())
<p class="title">
 <b>
  The Dormouse's story
 </b>
</p>
```

### `get_text()`

`get_text()` 获取到 `tag` 中包含的所有文本内容，包括 `tag` 子孙中的内容。可以设置输出格式。

```html
In [9]: print(soup.p.get_text())
The Dormouse's story

In [10]: print(soup.body.get_text())

The Dormouse's story
Once upon a time there were three little sisters; and their names were
Elsie,
Lacie and
Tillie;
and they lived at the bottom of a well.
...

In [11]: print(soup.body.get_text('|', strip=True))
The Dormouse's story|Once upon a time there were three little sisters; and their names were|Elsie|,|Lacie|and|Tillie|;
and they lived at the bottom of a well.|...
```

***

参考：

[Beautiful Soup 4.2.0 文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/index.html)，官方文档

