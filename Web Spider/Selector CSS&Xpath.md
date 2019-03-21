## XPath

XPath 是一门在 XML 文档中查找信息的语言。XPath 可用来在 XML 文档中对元素和属性进行遍历。

### 基础内容

#### 节点

在 XPath 中，有七种类型的节点：元素、属性、文本、命名空间、处理指令、注释以及文档节点（或称为根节点）。XML 文档是被作为节点树来对待的。树的根被称为文档节点或者根节点。基本值（或称原子值，Atomic value）基本值是无父或无子的节点。

节点关系：父（Parent）、子（Children）、同胞（Sibling）、先辈（Ancestor）、后代（Descendant）

#### 选取节点

XPath 使用路径表达式来选取 XML 文档中的节点或节点集。节点是通过沿着路径 (path) 或者步 (steps) 来选取的。

| 表达式   | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| nodename | 选取此节点的所有子节点。                                     |
| /        | 从根节点选取。<br />注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的**绝对路径**！ |
| //       | 从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。   |
| .        | 选取当前节点。                                               |
| ..       | 选取当前节点的父节点。                                       |
| @        | 选取属性。                                                   |

| 路径表达式      | 结果                                                         |
| --------------- | ------------------------------------------------------------ |
| bookstore       | 选取 bookstore 元素的所有子节点。                            |
| /bookstore      | 选取根元素 bookstore。 <br />注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的**绝对路径**！ |
| bookstore/book  | 选取属于 bookstore 的子元素的所有 book 元素。<br />**相对路径** |
| //book          | 选取所有 book 子元素，而不管它们在文档中的位置。             |
| bookstore//book | 选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。 |
| //@lang         | 选取名为 lang 的所有属性。                                   |

匹配未知元素：

| 通配符 | 描述                 |
| ------ | -------------------- |
| *      | 匹配任何元素节点。   |
| @*     | 匹配任何属性节点。   |
| node() | 匹配任何类型的节点。 |

| 路径表达式   | 结果                              |
| ------------ | --------------------------------- |
| /bookstore/* | 选取 bookstore 元素的所有子元素。 |
| //*          | 选取文档中的所有元素。            |
| //title[@*]  | 选取所有带有属性的 title 元素。   |

选取若干路径：

通过在路径表达式中使用“|”运算符，您可以选取若干个路径。

| 路径表达式                       | 结果                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| //book/title \| //book/price     | 选取 book 元素的所有 title 和 price 元素。                   |
| //title \| //price               | 选取文档中的所有 title 和 price 元素。                       |
| /bookstore/book/title \| //price | 选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。 |

#### 谓语（Predicates）

谓语用来查找某个特定的节点或者包含某个指定的值的节点。谓语被嵌在方括号中。

| 路径表达式                         | 结果                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| /bookstore/book[1]                 | 选取属于 bookstore 子元素的第一个 book 元素。                |
| /bookstore/book[last()]            | 选取属于 bookstore 子元素的最后一个 book 元素。              |
| /bookstore/book[last()-1]          | 选取属于 bookstore 子元素的倒数第二个 book 元素。            |
| /bookstore/book[position()<3]      | 选取最前面的两个属于 bookstore 元素的子元素的 book 元素。    |
| //title[@lang]                     | 选取所有拥有名为 lang 的属性的 title 元素。                  |
| //title[@lang='eng']               | 选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。   |
| /bookstore/book[price>35.00]       | 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。 |
| /bookstore/book[price>35.00]/title | 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。 |

#### XPath 轴

轴可定义**相对于当前节点**的节点集。

| 轴名称             | 结果                                                     |
| ------------------ | -------------------------------------------------------- |
| ancestor           | 选取当前节点的所有先辈（父、祖父等）。                   |
| ancestor-or-self   | 选取当前节点的所有先辈（父、祖父等）以及当前节点本身。   |
| attribute          | 选取当前节点的所有属性。                                 |
| child              | 选取当前节点的所有子元素。                               |
| descendant         | 选取当前节点的所有后代元素（子、孙等）。                 |
| descendant-or-self | 选取当前节点的所有后代元素（子、孙等）以及当前节点本身。 |
| following          | 选取文档中当前节点的结束标签之后的所有节点。             |
| namespace          | 选取当前节点的所有命名空间节点。                         |
| parent             | 选取当前节点的父节点。                                   |
| preceding          | 选取文档中当前节点的开始标签之前的所有节点。             |
| preceding-sibling  | 选取当前节点之前的所有同级节点。                         |
| self               | 选取当前节点。                                           |

#### 步（step）

- 轴（axis）

  定义所选节点与当前节点之间的树关系

- 节点测试（node-test）

  识别某个轴内部的节点

- 零个或者更多谓语（predicate）

  更深入地提炼所选的节点集

步的语法：

```text
轴名称::节点测试[谓语]
```

| 例子                   | 结果                                                         |
| ---------------------- | ------------------------------------------------------------ |
| child::book            | 选取所有属于当前节点的子元素的 book 节点。                   |
| attribute::lang        | 选取当前节点的 lang 属性。                                   |
| child::*               | 选取当前节点的所有子元素。                                   |
| attribute::*           | 选取当前节点的所有属性。                                     |
| child::text()          | 选取当前节点的所有文本                                       |
| child::node()          | 选取当前节点的所有子节点。                                   |
| descendant::book       | 选取当前节点的所有 book 后代。                               |
| ancestor::book         | 选择当前节点的所有 book 先辈。                               |
| ancestor-or-self::book | 选取当前节点的所有 book 先辈以及当前节点（如果此节点是 book 节点） |
| child::*/child::price  | 选取当前节点的所有 price 孙节点。                            |

#### 运算符

| 运算符 | 描述           | 实例                      | 返回值                                          |
| ------ | -------------- | ------------------------- | ----------------------------------------------- |
| \|     | 计算两个节点集 | //book \| //cd            | 返回所有拥有 book 和 cd 元素的节点集            |
| +      | 加法           | 6 + 4                     | 10                                              |
| -      | 减法           | 6 - 4                     | 2                                               |
| *      | 乘法           | 6 * 4                     | 24                                              |
| div    | 除法           | 8 div 4                   | 2                                               |
| mod    | 取余           | 5 mod 2                   | 1                                               |
| =      | 等于           | price=9.80                | 如果 price 是 9.80，则返回 true。               |
| !=     | 不等于         | price!=9.80               | 如果 price 不是 9.80，则返回 true。             |
| <      | 小于           | price<9.80                | 如果 price 小于 9.80，则返回 true。             |
| <=     | 小于或等于     | price<=9.80               | 如果 price 小于等于 9.80，则返回 true。         |
| >      | 大于           | price>9.80                | 如果 price 大于 9.80，则返回 true。             |
| >=     | 大于或等于     | price>=9.80               | 如果 price 大于等于 9.80，则返回 true。         |
| or     | 或             | price=9.80 or price=9.70  | 如果 price 是 9.80 或 9.70，则返回 true。       |
| and    | 与             | price>9.00 and price<9.90 | 如果 price 大于 9.00 且小于 9.90，则返回 true。 |

### Xpath 选择器

语法：

```text
//tag[@attribute='value']	# value必须完全匹配
```

```html
<div class="gb_Ja gb_pg gb_f gb_og gb_sg gb_Ef" data-ogsr-up="" style="min-width: 209px;" activated="1"></div>
```

```text
//div[@class='gb_Ja gb_pg gb_f gb_og gb_sg gb_Ef']	# NOTE:class值必须填写完整,完全匹配
```

绝对路径VS相对路径：

* 由 `/` 开始，表示是绝对路径
* 由 `//` 开始，表示是相对路径

`/` VS `//`：

* `/`：其两侧的节点必须相邻，不能跳级
* `//`：其左侧节点下任何一个符合右侧要求的子节点，可以不相邻，可以跳级，从而忽略不重要的节点

```text
//div/a/span
//div//span
```

使用 Text 文本定位：

```text
//tag[text()='value']	# value必须完全匹配
```

使用 Contains 关键字定位：

```text
//tag[contains(attribute,'value')]
```

```text
//tag[contains(@class,'gb_Ja')]
//tag[contains(@class,'gb_Ja') and contains(@id, 'test_id')]
//tag[contains(text(),'test_string')]	# atrribute可以是text()
```

使用 starts-with 关键字定位：

```text
//tag[starts-with(attribute,'value')]
```

定位节点：

```text
xpath-to-element//parent::tag	# 定位父节点

xpath-to-element//preceding-sibling::tag	# 定位前面同胞节点

xpath-to-element//following-sibling::tag	# 定位前面同胞节点
```

***

## CSS

语法：

```text
tag[attribute='value']

针对ID和Class的简写方式
# --> Id
. --> Class
```

### ID 选择器

```html
<img alt="Google" id="hplogo" src="/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png" srcset="/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png 1x, /images/branding/googlelogo/2x/googlelogo_color_272x92dp.png 2x" style="padding-top:109px" onload="window.lol&amp;&amp;lol()" data-atf="1" width="272" height="92">
```

可以写为：

```text
img[id='hplogo']
#hplogo
img#hplogo
```

### Class 选择器

```html
<div class="jhp" id="searchform"></div>
```

```text
div[class='jhp']
.jhp
div.jhp
```

多个class选择：

```html
.class1.class2.class3···  -->直到准确定位
```

```html
<div class="gb_Ja gb_pg gb_f gb_og gb_sg gb_Ef" data-ogsr-up="" style="min-width: 209px;" activated="1"></div>
```

```text
.gb_Ja.gb_pg.gb_f	# 顺序不分先后
```

### 通配符 选择器

```text
^ --> 以文本的开始
$ --> 以文本的结尾
* --> 任意文本
```

```text
img[id^='hplogo']	以hplogo开始的id

img[id$='hplogo']	以hplogo结尾的id

img[id*='hplogo']	任何包含hplogo的id
```

### 节点定位

```text
> --> 节点切换
```

```html
<div id="gb" class="gb_Ef">
    <div id="gbw">
    </div>
</div>
```

```
div>#gbw
```

***

参考：

http://www.w3school.com.cn/xpath/xpath_nodes.asp