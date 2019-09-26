在使用 XML 时，最想了解的三个问题是：

* 有一个XML文件，如何解析
* 解析后，如果查找、定位某个标签
* 定位后如何操作标签，比如访问属性、文本内容等

使用正则来处理 HTML、XML 文档来说比较麻烦，一般来说处理 HTML、XML 文档都是使用 Xpath。Xpath 可以用来查找HTML、XML节点或元素，是一门在 HTML、XML 文档中查找信息的语言。

Element 是 XML 处理的核心类，Element 对象可以直观的理解为 XML 的节点，大部分 XML 节点的处理都是围绕该类进行的。

文件解析，解析为 Element 对象或者 ElementTree 对象：

```text
lxml.etree.parse(source, parser=None, base_url=None)
将source对象解析为ElementTree对象。
source 参数可以是以下几种：
    a file name/path
    a file object
    a file-like object
    a URL using the HTTP or FTP protocol
解析string时，采用lxml.etree.fromstring()

lxml.etree.fromstring(text, parser=None, base_url=None)
解析xml的字符串为 Element 或者 ElementTree。Parses an XML document or fragment from a string. Returns the root node (or the result returned by a parser target).

lxml.etree.HTML(text, parser=None, base_url=None)
Parses an HTML document from a string constant. Returns the root node (or the result returned by a parser target). 

lxml.etree.XML(text, parser=None, base_url=None)
Parses an XML document or fragment from a string constant. Returns the root node (or the result returned by a parser target). 
```

Element 的属性：

```text
element.tag
元素的名称

element.attrib
一个包含元素属性的字典，key 是属性名，value 是对应的值。

element.text
Element 的文本均为直接子文本，不包含子元素中的文本，这其中又包含两部分 .text 和 .tail 。.text 是第一个子元素标签之前的，如果没有则为 None 。

element.tail
.tail 为 Element 的关闭标签之后的文本，并且是在下一个兄弟子标签之前的部分。没有则为 None 。
```

Element 的方法:

```text
element.items()
返回由元素属性的键值所构成的( name, value)元组的列表。

element.keys()
返回一个没有特定顺序的元素属性名的列表。

element.set(A, V)
创建或者改变属性 A 的值为 V。

element.xpath()
通过xpath查找信息

element.append( child )
添加一个新的子节点(可以是 Element 、Comment)到当前 Element 中。

element.insert( index, elt)
将子元素 elt 插入到指定位置，这个 index 的随意性比较大，如果是正数但是超过了最大值，那么 lxml 会直接将这个元素插到末尾，如果是负数，但是这个负数所指位置不存在，那么就插到末尾，这个负数的位置计算规则和列表的那个不太一样，不知道正确的规律是什么，但是经过测试，-n所插的位置，后面有 n (以变化前计算)个元素。如果对 Element 中的子元素执行 insert() 操作，那么子元素位置会按 index 指定的变换。

element.clear()
调用该函数，将移除所有内容 .attrib 将被移除 .text 和 .tail 将被设置为 None 所有的子节点将被删除。

element.remove( child )
将子节点 child 从Element 中移除 ，如果child 不是 Element 的子节点，将会引发 ValueError 异常。

element.find( path )
从 Element 的子元素及后代元素中查找第一个符合 path 的 subelement 。如果没有返回 None 。
ElementPath 是 ElementTree 自带的一个 XPath-like 的路径语言，和 XPath 差不太多，主要区别是 ElementPath 能用 {namespace}tag，但是 ElementPath 不能使用值比较和函数。

element.findall( path )
返回一个匹配的 Element 的列表。

element.findtext( path, default=None )
返回第一个匹配元素的 .text 内容，如果存在匹配，但是没有 .text 内容，那么将返回一个空字符串，如果没有一个匹配的元素，那么将会返回一个 None ，但是有 default 参数，返回 default 所指定的。

element.get( key, default=None )
返回字符串形式的 属性 key 的值，没有返回 None 或者 default 指定的。

element.getchildren()
返回一个包含 Element 子元素的列表。

element.getiterator( tag=None, *tags )
返回元素的一个生成器，返回元素类别取决于参数 tag ，生成顺序是in document order (depth first pre-order) 深度优先的先根遍历。如果没有参数的话，则第一个就是元素本身。如果想使用一个高效的生成器，可以使用 .iter() 。

element.getroottree()
返回该元素的 ElementTree 。

element.iter( tag=None, *tags )
过滤特定标签，生成迭代器。默认情况下，iter() 迭代所有的节点，包括PI(处理指令) 、Comment(注释) 等，如果只想迭代标签元素，可以使用 Element factory 做参数e.iter(tag = etree.Element)。

element.iterfind( path )
迭代所有匹配 ElementPath 的 Element 。

element.itertext( tag=None, *tags, with_tail=True )
迭代 Element 元素的文本内容，with_tail 参数决定是否迭代子元素的 tail 。Element 的tail 不会进行迭代。

element.iterancestors( tag=None )
如果忽略参数，那么将会迭代所有先辈，加标签名可以只迭代该标签先辈。

element.iterchildren( reversed=False, tag=None )
通过设置 reversed=True 可以以反向的顺序迭代子元素。

element.itersiblings( preceding=False )
迭代 Element 之后的兄弟元素，可以通过设置 preceding=True 仅迭代 Element  之前的兄弟元素。

element.iterdescendants( tag=None )
同 iterancestors()
```
