爬取的主要目标是从非结构化源(通常是web页面)中提取结构化数据。Scrapy 将提取的数据以dict形式返回。

虽然很方便和熟悉，但dict缺乏结构:很容易出现字段名拼写错误或返回不一致的数据，特别是在有许多spider的大型项目中。为了定义公共输出数据格式，Scrapy提供了Item类。Item对象是简单的容器，用于收集爬取到的数据。提供了一个类似于dict的API，使用简便语法声明可用字段。

各种Scrapy组件使用Items提供的额外信息：exporter查看已声明的字段以确定要导出的列，serialization 可以使用Item字段元数据定制，trackref跟踪Item实例以帮助查找内存泄漏，等等。

`Item`对象是自定义的python字典，可以使用标准的字典语法来获取到其每个字段的值。Spider将会将爬取到的数据以 Item 对象返回。

***

## Item

使用简单的类定义语法和`Field`对象声明Item。

```python
import scrapy

class Product(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    stock = scrapy.Field()
    last_updated = scrapy.Field(serializer=str)
```

`Field`对象用于定义每个字段元数据。

Note：用于声明的 `Field` objects不会作为类属性。相反，可以通过`Item.fields`访问。

***

## Item Loader

Item Loaders为抓取的Item提供了一种方便的存储机制。

Item提供了抓取数据的容器，而Item Loaders提供了存储该容器的机制。

Item Loaders的设计目的是提供一种灵活、高效和简单的扩展和覆盖不同字段解析规的机制，可以通过spider或源格式(HTML、XML等)进行扩展和覆盖，而不会成为维护的噩梦。

使用Item Loaders时，必须首先实例化。下面是一个典型的Item Loader在spider中的使用，使用 Item 文件中声明的类：

```python
# spider.py
from scrapy.loader import ItemLoader
from myproject.items import Product

def parse(self, response):
    l = ItemLoader(item=Product(), response=response)
    l.add_xpath('name', '//div[@class="product_name"]')
    l.add_xpath('name', '//div[@class="product_title"]')
    l.add_xpath('price', '//p[@id="price"]')
    l.add_css('stock', 'p#stock]')
    l.add_value('last_updated', 'today') # you can also use literal values
    return l.load_item()
```

当收集所有数据后，调用`load item()`方法，该方法实际返回之前提取和收集数据的item 。

### Input and Output processors

Item Loader 中的每个(item) field 都含有一个input processor 和 一个 output processor。

* input processor 在接收到已提取的数据时，即通过add xpath()、add css()或add value()方法提取的数据，收集input processor的结果，并保存在ItemLoader中。

* 在收集所有数据之后，Item Loader调用load item()方法，储存并返回已填充的`Item` object。

* 当使用之前收集的数据调用output processor(并使用input processor处理)时。output processor的结果是该item的最终值。

```python
l = ItemLoader(Product(), some_selector)
l.add_xpath('name', xpath1) # (1)
l.add_xpath('name', xpath2) # (2)
l.add_css('name', css) # (3)
l.add_value('name', 'test') # (4)
return l.load_item() # (5)
```

1. 从xpath1中提取数据，通过name字段的传入input processor。input processor的结果被收集并保存在Item Loader(但尚未分配给Item)；
2. 从xpath2中提取数据，并通过（1）中相同的input processor。 input processor的结果附加到（1）中收集的数据（如果有的话）；
3. 使用CSS选择器的与（2）相同；
4. 通过值提取数据，与上相同；
5. 在步骤（1）,（2）,（3）和（4）中收集的数据通过`name`字段的input processor传递。output processor的结果被分配给Item中的`name`字段的值。

处理器只是可调用的对象，调用这些对象用来解析数据，并返回一个解析后的值。因此，可以使用任何函数作为input processor或output processor。唯一的要求是，它们必须接收一个迭代器作为它们的第一个参数。这些函数的输出可以是任何值。

* input processor的结果将被附加到包含(该字段的)收集值的内部列表(在Loader中)；
* output processor的结果是最终分配给item的值。

input processor返回的值在内部收集（以列表形式），然后传递给output processor以填充字段。

#### 内置处理器

Scrapy提供了一些常用的处理器。

* ` class scrapy.loader.processors.Identity`

  最简单的处理器，它什么都不做。它返回原始值不变。它不接收任何构造函数参数，也不接受加载器上下文。

  ```bash
  >>> from scrapy.loader.processors import Identity
  >>> proc = Identity()
  >>> proc(['one', 'two', 'three'])
  ['one', 'two', 'three']
  ```

* ` class scrapy.loader.processors.TakeFirst`

  从接收到的值中返回第一个非空值，因此它通常用作单值字段的输出处理器。
  它不接收任何构造函数参数，也不接受Loader上下文。

  ```bash
  >>> from scrapy.loader.processors import TakeFirst
  >>> proc = TakeFirst()
  >>> proc(['', 'one', 'two', 'three'])
  'one'
  ```

* ` class scrapy.loader.processors.Join(separator=u' ')`

  返回使用构造函数中给出的分隔符连接后的值，默认值为`u' '`。 它不接受Loader上下文。

  当使用默认的分隔符时，这个处理器相当于下面的函数：`u' '.join`

  ```bash
  >>> from scrapy.loader.processors import Join
  >>> proc = Join()
  >>> proc(['one', 'two', 'three'])
  u'one two three'
  >>> proc = Join('<br>')
  >>> proc(['one', 'two', 'three'])
  u'one<br>two<br>three'
  ```

* ` class scrapy.loader.processors.Compose(*functions, **default_loader_context)`

  由给定函数组成的处理器。这意味着这个处理器的每个输入值都传递给第一个函数，这个函数的结果传递给第二个函数，以此类推，直到最后一个函数返回这个处理器的输出值。

  ```bash
  >>> from scrapy.loader.processors import Compose
  >>> proc = Compose(lambda v: v[0], str.upper)
  >>> proc(['hello', 'world'])
  'HELLO'
  ```

* `class scrapy.loader.processors.MapCompose(*functions, **default_loader_context)`

  类似 map() 函数

  ```bash
  >>> def filter_world(x):
  ...     return None if x == 'world' else x
  ...
  >>> from scrapy.loader.processors import MapCompose
  >>> proc = MapCompose(filter_world, unicode.upper)
  >>> proc([u'hello', u'world', u'this', u'is', u'scrapy'])
  [u'HELLO, u'THIS', u'IS', u'SCRAPY']
  ```

* ` class scrapy.loader.processors.SelectJmes(json_path)`

  使用提供给构造函数的json路径查询该值并返回输出。

#### 声明输入和输出处理器

在 Item Field metadata 中，可以自定义输入和输出处理器。

```python
import scrapy
from scrapy.loader.processors import Join, MapCompose, TakeFirst
from w3lib.html import remove_tags

def filter_price(value):
    if value.isdigit():
        return value

class Product(scrapy.Item):
    name = scrapy.Field(
        input_processor=MapCompose(remove_tags),
        output_processor=Join(),
    )
    price = scrapy.Field(
        input_processor=MapCompose(remove_tags, filter_price),
        output_processor=TakeFirst(),
    )
```

输入和输出处理器的优先顺序如下:

1. Item Loader中field-specific attributes: `field_in` and `field_out` (最高优先级)；
2. Field metadata (`input_processor` and `output_processor` key)
3. Item Loader defaults: `ItemLoader.default_input_processor()` and
   `ItemLoader.default_output_processor()` (最低优先级)

### 声明 Item Loader

```python
# item.py
from scrapy.loader import ItemLoader
from scrapy.loader.processors import TakeFirst, MapCompose, Join

class ProductLoader(ItemLoader):

    default_output_processor = TakeFirst()

    name_in = MapCompose(unicode.title)
    name_out = Join()

    price_in = MapCompose(unicode.strip)

    # ...
```

输入处理器是使用`_in`后缀声明的，而输出处理器是使用`_out`后缀声明的。 您还可以使用`ItemLoader.default_input_processor`和`ItemLoader.default_output_processor`属性声明默认的输入/输出处理器。

###  Item Loader Context

Item Loader Context是Item Loader中所有输入和输出处理器共享的任意键/值的字典。在声明，实例化或使用Item Loader时传递，用于修改输入/输出处理器的行为。

例如：函数`parse_length`接收文本值，并从中提取长度：

```python
def parse_length(text, loader_context):
    unit = loader_context.get('unit', 'm')
    # ... length parsing code goes here ...
    return parsed_length
```

通过`loader_context`参数，函数显式地告诉Item Loader，接收Item Loader Context。

如下方法可以修改Item Loader Context中的值：

1. 通过修改当前活动Item Loader Context(`context`)属性)

   ```python
   loader = ItemLoader(product)
   loader.context['unit'] = 'cm'
   ```

2. 在Item Loader实例化时（Item Loader构造函数的关键字参数存储在Item Loader上下文中）：

   ```python
   loader = ItemLoader(product, unit='cm')
   ```

3. 在Item Loader声明中，对于那些支持用Item Loader上下文实例化的输入/输出处理器。

   ```python
   class ProductLoader(ItemLoader):
       length_out = MapCompose(parse_length, unit='cm')
   ```

### 嵌套Loader

当从文档的一个小节解析相关值时，创建嵌套Loader可能很有用。

```html
<footer>
    <a class="social" href="https://facebook.com/whatever">Like Us</a>
    <a class="social" href="https://twitter.com/whatever">Follow Us</a>
    <a class="email" href="mailto:whatever@example.com">Email Us</a>
</footer>
```

```python
loader = ItemLoader(item=Item())
# load stuff not in the footer
footer_loader = loader.nested_xpath('//footer')
footer_loader.add_xpath('social', 'a[@class = "social"]/@href')
footer_loader.add_xpath('email', 'a[@class = "email"]/@href')
# no need to call footer_loader.load_item()
loader.load_item()
```

一般来说，使用嵌套加载器可以使代码变得更简单，但不要过度嵌套，否则解析器会变得难以阅读。

### 扩展 Item Loader

随着项目越来越大，使用越来越多的Spider，维护成为一个基本问题，尤其是当必须处理每个Spider的不同解析规则和大量的异常处理情况，但同时也想重复使用通用处理器。

例如，假设一些特定的网站用三个破折号（例如`---Plasma TV---`）封装其产品名称，而不想在最终产品名称中取得这些破折号。

可以通过重用和扩展默认Product Item Loader（`ProductLoader`）来删除这些破折号：

```
# item.py
from scrapy.loader.processors import MapCompose
from myproject.ItemLoaders import ProductLoader

def strip_dashes(x):
    return x.strip('-')

class SiteSpecificLoader(ProductLoader):
    name_in = MapCompose(strip_dashes, ProductLoader.name_in)
```

扩展、继承和覆盖Item Loader的方法还有很多，不同的Item Loader层次结构可能更适合不同的项目。Scrapy只提供了机制;它不会强制Item Loader集合的任何特定组织——这取决于项目的需要。

### ItemLoader objects

```text
 class scrapy.loader.ItemLoader([item, selector, response, ]**kwargs)
 返回用于填充Item的ItemLoader。如果没有给定Item，则使用default_item_class类自动实例化一个Item
 
item, selector, response和其余关键字参数被分配给Item Loader Context(通过context属性访问)。
```

* `.get_value(value, *processors, **kwargs)`

  通过给定的`processors`和关键字参数处理给定的`value`。

  可用关键字参数：re (str or compiled regex) 

  ```bash
  >>> from scrapy.loader.processors import TakeFirst
  >>> loader.get_value(u'name: foo', TakeFirst(), unicode.upper, re='name: (.+)')
  'FOO`
  ```

*  `.add_value(field_name, value, *processors, **kwargs)`

  处理，然后为给定字段添加给定的`value`。

  value首先通过`.get_value()`传递给`processors`和`kwargs`，然后通过field input processor传递，最后结果附加到字段收集的数据中。 如果该字段已包含收集的数据，则添加新数据。

* `.replace_value(field_name, value, *processors, **kwargs)`

  与`.add_value()`类似，但用新值替换收集的数据，而不是添加它。

* `.get_xpath(xpath, *processors, **kwargs)`

  与`ItemLoader.get_value()`)类似，但接收XPath而不是value，该Xpath用于从与`ItemLoader`关联的选择器中提取unicode字符串列表。

  - xpath（str） - 提取数据的XPath
  - re (str 或 已编译的正则表达式） - 从选定的XPath区域提取数据的正则表达式

  ```python
  # HTML snippet: <p class="product-name">Color TV</p>
  loader.get_xpath('//p[@class="product-name"]')
  # HTML snippet: <p id="price">the price is $1200</p>
  loader.get_xpath('//p[@id="price"]', TakeFirst(), re='the price is (.*)')
  ```

* `.add_xpath(field_name, xpath, *processors, **kwargs)`

  与`ItemLoader.add_value()`类似，但接收XPath而不是value。

  ```python
  # HTML snippet: <p class="product-name">Color TV</p>
  loader.add_xpath('name', '//p[@class="product-name"]')
  # HTML snippet: <p id="price">the price is $1200</p>
  loader.add_xpath('price', '//p[@id="price"]', re='the price is (.*)')
  ```

* `.replace_xpath(field_name, xpath, *processors, **kwargs)`

  类似 `.add_xpath()`。

* `.get_css(css, *processors, **kwargs)`

* `.add_css(field_name, css, *processors, **kwargs)`

* `.replace_css(field_name, css, *processors, **kwargs)`

* `.load_item()`

  用迄今收集的数据填充item，并返回它。 收集到的数据首先通过output processors传递给每个item字段以获取最终值。

* `.nested_xpath(xpath)`

  使用xpath选择器创建嵌套loader 。选择器与`ItemLoader`关联的选择器应用是相同的。 嵌套的Loader与父`ItemLoader`共享`Item`，所以调用`add_xpath()`，`add_value()` `replace_value()`等将正常工作。

* `.nested_css(css)`

* `.get_collected_values(field_name)`

  返回给定字段的收集值。

* `.get_output_value(field_name)`

  返回使用输出处理器为给定字段解析的收集值。此方法根本不填充或修改项。

* `.get_input_processor(field_name)`

* `.get_output_processor(field_name)`

* `.item`

  Item Loader解析的`Item`

* `.context`

  Item Loader中当前活动的Context

* `.default_item_class`

* `.default_input_processor`

* `.default_output_processor`

* `.default_selector_class`

* `.selector`


***

参考：

https://docs.scrapy.org/en/latest/topics/loaders.html#itemloader-objects