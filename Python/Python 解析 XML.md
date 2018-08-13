#### **XML**

XML 指可扩展标记语言(eXtensible Markup Language)，被设计用来结构化、存储以及传输数据，而不是显示数据。XML 语言没有预定义的标签，允许定义标签和文档结构。区别于，HTML 中使用的标签都是预定义的。XML 不会替代 HTML，而是对 HTML 的补充。

* XML 树结构

XML 文档必须包含根元素。该元素是所有其他元素的父元素。所有的元素都可以有子元素。父、子以及同胞等术语用于描述元素之间的关系。相同层级上的子元素成为同胞（兄弟或姐妹）。

```XML
<root>
<child>
<subchild>.....</subchild>
</child>
</root>
```

每个元素都可以包含：其他元素、文本、属性、以上混合。
文件格式如下：

```xml
<tag attrib = > text </tag> tail
```

* XML 示例

```XML
<collection shelf="New Arrivals">
<movie title="Enemy Behind">
   <type>War, Thriller</type>
   <format>DVD</format>
   <year>2003</year>
   <rating>PG</rating>
   <stars>10</stars>
   <description>Talk about a US-Japan war</description>
</movie>
<movie title="Transformers">
   <type>Anime, Science Fiction</type>
   <format>DVD</format>
   <year>1989</year>
   <rating>R</rating>
   <stars>8</stars>
   <description>A schientific fiction</description>
</movie>
</collection>
```

***

#### **解析 XML 的方法**

* DOM

DOM 会把整个 XML 读入内存，解析为树，通过对树的操作来操作 XML。因此占用内存大，解析慢，优点是可以任意遍历树的节点。

* SAX

SAX 解析是通过流模式在解析 XML 的过程中触发对应的事件，边读边解析，占用内存小，解析快，缺点是我们需要自己处理事件。

* ElementTree

一个轻量级、Pythonic的API，同时还有一个高效的C语言实现，即xml.etree.cElementTree。与DOM相比，ET的速度更快，API使用更直接、方便。与SAX相比，ET.iterparse函数同样提供了按需解析的功能，不会一次性在内存中读入整个文档。

***

#### **DOM(Document Object Model)**

DOM（Document Object Model）定义了一个访问 XML 的标准方式。

DOM 的解析器在解析 XML 文档时，一次性读取整个文档，把文档中所有元素保存在内存中的一个树结构里，之后可以利用 DOM 提供的不同的函数来读取或修改文档的内容和结构，也可以把修改过的内容写入 xml 文件。

getElementsByTagName() 返回拥有指定标签名的所有元素。

```python
node.getElementsByTagName("tagname")
```

* python 代码：

```python
# _*_coding:utf-8_*_

import os
import xml.dom.minidom

# minidom解析器打开 XML 文档
xml_path = os.path.abspath("movies.xml")
print ("xml file path:", xml_path)

dom_tree = xml.dom.minidom.parse(xml_path)
collection = dom_tree.documentElement
if collection.hasAttribute("shelf"):
    print("Root element : %s" % collection.getAttribute("shelf"))

# 获取所有电影及其详细信息

movies = collection.getElementsByTagName("movie")

for i in movies:
    print("*** Movie ***")
    if i.hasAttribute("title"):
        print("Title: %s" % i.getAttribute("title"))

    type = i.getElementsByTagName("type")[0]
    print("Type: %s" % type.childNodes[0].data)
    format = i.getElementsByTagName("format")[0]
    print("format: %s" % format.childNodes[0].data)
    rating = i.getElementsByTagName("rating")[0]
    print("rating: %s" % rating.childNodes[0].data)
    description = i.getElementsByTagName("description")[0]
    print ("Description: %s" % description.childNodes[0].data)

```

* 主要方法：

| | |
| :- | -: |
|xml.dom.minidom.parse(filename)|加载XML文件|
|doc.documentElement|获取XML文档对象|
|node.getAttribute(AttributeName)|获取XML节点属性值|
|node.getElementsByTagName(TagName)|获取XML节点对象集合|
|node.childNodes|返回子节点列表|
|node.childNodes[index].nodeValue|获取XML节点值|

* 输出结果：

```shell
xml file path: c:\GitHub\Python_tests\XML\movies.xml
Root element : New Arrivals
*** Movie ***
Title: Enemy Behind
Type: War, Thriller
format: DVD
rating: PG
Description: Talk about a US-Japan war
*** Movie ***
Title: Transformers
Type: Anime, Science Fiction
format: DVD
rating: R
Description: A schientific fiction
```

***

#### **SAX(simple API for XML)**

python 标准库包含 SAX 解析器，SAX 用事件驱动模型，通过在解析XML的过程中触发一个个的事件并调用用户定义的回调函数来处理XML文件。SAX 是一种基于事件驱动的 API。

利用 SAX 解析 XML 文档牵涉到两个部分:解析器(Readers)和事件处理器(Handlers)。

* 解析器负责读取 XML 文档，并向事件处理器发送事件。如元素开始跟元素结束事件；

* 事件处理器则负责对事件作出相应，对传递的 XML 数据进行处理。

在 Python 中使用 SAX 解析 XML 非常简洁，通常我们关心的事件是 ```start_element```，```end_element``` 和 ```char_data```。准备好这3个函数，然后就可以解析 XML 了。

* python 代码：

```python
# _*_coding:utf-8_*_

import xml.sax

class MovieHandler(xml.sax.ContentHandler):
    def __init__(self):
        self.CurrentData = ""
        self.type = ""
        self.format = ""
        self.year = ""
        self.rating = ""
        self.stars = ""
        self.description = ""

    # 遇到 XML开始标签时调用
    def startElement(self, tag, attributes):
        self.CurrentData = tag
        if tag == "movie":
            print("*** movie ***")
            title = attributes["title"]
            print("Title:", title)

    # 遇到 XML 结束标签时调用
    def endElement(self, tag):
        if self.CurrentData == "type":
            print ("Type:", self.type)
        elif self.CurrentData == "format":
            print ("Format:", self.format)
        elif self.CurrentData == "year":
            print ("Year:", self.year)
        elif self.CurrentData == "rating":
            print ("Rating:", self.rating)
        elif self.CurrentData == "stars":
            print ("Stars:", self.stars)
        elif self.CurrentData == "description":
            print ("Description:", self.description)
        self.CurrentData = ""

    # 遇到 XML 元素内容时调用
    def characters(self, content):
        if self.CurrentData == "type":
            self.type = content
        elif self.CurrentData == "format":
            self.format = content
        elif self.CurrentData == "year":
            self.year = content
        elif self.CurrentData == "rating":
            self.rating = content
        elif self.CurrentData == "stars":
            self.stars = content
        elif self.CurrentData == "description":
            self.description = content

if (__name__ == "__main__"):
    # 创建 SAX XMLReader 对象
    parser = xml.sax.make_parser()
    # 关闭命名空间
    parser.setFeature(xml.sax.handler.feature_namespaces, 0)

    # 实例 ContentHandler
    Handler = MovieHandler()
    # 如果不进行设置，content 事件会被忽略
    parser.setContentHandler(Handler)

    parser.parse("movies.xml")
```

* characters(content)方法调用时机：

从行开始，遇到标签之前，存在字符，content的值为这些字符串。

从一个标签，遇到下一个标签之前， 存在字符，content的值为这些字符串。

从一个标签，遇到行结束符之前，存在字符，content的值为这些字符串。

标签可以是开始标签，也可以是结束标签。

* 输出结果：

```shell
*** movie ***
Title: Enemy Behind
Type: War, Thriller
Format: DVD
Year: 2003
Rating: PG
Stars: 10
Description: Talk about a US-Japan war
*** movie ***
Title: Transformers
Type: Anime, Science Fiction
Format: DVD
Year: 1989
Rating: R
Stars: 8
Description: A schientific fiction
```

***

#### **ElementTree**

ET 模块可以归纳为三个部分：```ElementTree``` 类，```Element``` 类以及一些操作 XML 的函数。ElementTree 表示整个 XML 树，Element 表示树上的单个节点。操作整个 XML 文档时使用 ElementTree 类。操作 XML 元素时使用 Element 类。Element是一个灵活的容器对象，在内存中存储层次化的数据结构。

* python代码：

```python
# _*_coding:utf-8_*_

import xml.etree.ElementTree as ET

tree = ET.parse("movies.xml")
root = tree.getroot()

for child in root:
    if child.tag == "movie":
        print("*** movie ***")
        print("Title:", child.attrib['title'])
    for sub in child:
         print(sub.tag.capitalize(),":",sub.text)

```

调用 ET.parse(filename) 将 XML 文档解析为 ElementTree 对象，调用 ElementTree.getroot() 获取 Element 类型的树根，然后分别访问树根元素的 tag、attrib、text、tail 等属性。

| | |
|:-|:-|
|tag|获取标签|
|attrib|获取以 Python **字典** 形式存储的元素属性|
|text|获取开始标签和第一个子元素之间的字符串或开始标签和结束标签之间的字符串或None|
|tail|获取结束标签和下一个标签之间的字符串或None|

Element 对象的子元素，可以直接遍历，也可以使用下标访问。

* 输出结果：

```shell
*** movie ***
Title: Enemy Behind
Type : War, Thriller
Format : DVD
Year : 2003
Rating : PG
Stars : 10
Description : Talk about a US-Japan war
*** movie ***
Title: Transformers
Type : Anime, Science Fiction
Format : DVD
Year : 1989
Rating : R
Stars : 8
Description : A schientific fiction
```

* 修改 XML

| | |
|:- | :-|
|find(*'nodeName'*)|在该节点下，查找其中第一个 tag 为nodeName的节点|
|findall(*'nodeName'*)|在该节点下，查找其中所有 tag 为nodeName的节点。|

以及：

| | |
|:- | :-|
|Element.text=*value*|修改text属性|
|Element.tail=*value*|修改tail属性|
|Element.set(*key, vlaue*)|添加attrib|
|Element.append(*subelement*)|添加子元素|
|Element.extend(*subelements*)|添加子元素的列表（参数类型是序列）|
|Element.remove(*subelement*)|删除子元素|

删除 format，同时新增 studio，并赋值 studio = XX。

* python 代码：

```python
# _*_coding:utf-8_*_

import xml.etree.ElementTree as ET

tree = ET.parse("movies_1.xml")
root = tree.getroot()

for child in root.findall('movie'):
    child.remove(child.find('format'))

    studio = ET.Element('studio')
    studio.text = 'XX'
    child.append(studio)

tree.write('movies_new.xml')
```

* 输出结果：

```XML
<collection shelf="New Arrivals">
<movie title="Enemy Behind">
   <type>War, Thriller</type>
   <year>2003</year>
   <rating>PG</rating>
   <stars>10</stars>
   <description>Talk about a US-Japan war</description>
<studio>XX</studio></movie>
<movie title="Transformers">
   <type>Anime, Science Fiction</type>
   <year>1989</year>
   <rating>R</rating>
   <stars>8</stars>
   <description>A schientific fiction</description>
<studio>XX</studio></movie>
</collection>
```
