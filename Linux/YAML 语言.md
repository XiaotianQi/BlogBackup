YAML 语言实质上是一种通用的数据串行化格式。

它的基本语法规则如下：

- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用Tab键，只允许使用空格。
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

`#` 表示注释，从这个字符一直到行尾，都会被解析器忽略。

YAML 支持的数据结构有三种：

- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
- 纯量（scalars）：单个的、不可再分的值

***

## 对象

使用冒号代表，格式为`key: value`。冒号后面要加一个空格：

```yaml
key: value

对应：
{'key': 'value'}

hash: { name: Steve, foo: bar } 

对应：
{'hash': {'name': 'Steve', 'foo': 'bar'}}
```

## 数组

一组连词线开头的行，构成一个数组。

数据结构的子成员是一个数组，则可以在该项下面缩进一个空格。也可以采用行内表示法。

```yaml
- Cat
- Dog
- Goldfish

对应：
['Cat', 'Dog', 'Goldfish']

-
 - Cat
 - Dog
 - Goldfish
 
对应：
[['Cat', 'Dog', 'Goldfish']]

animal: [Cat, Dog]

对应：
{'animal': ['Cat', 'Dog']}
```

## 复合结构

对象和数组可以结合使用，形成复合结构。

```yaml
languages:
 - Ruby
 - Perl
 - Python 
websites:
 YAML: yaml.org 
 Ruby: ruby-lang.org 
 Python: python.org 
 Perl: use.perl.org 

对应：
{'languages': ['Ruby', 'Perl', 'Python'],
 'websites': {'YAML': 'yaml.org',
  'Ruby': 'ruby-lang.org',
  'Python': 'python.org',
  'Perl': 'use.perl.org'}}
```

***

## 常量

YAML中提供了多种常量结构，包括：整数，浮点数，字符串，NULL，日期，布尔，时间。

```yaml
boolean: 
    - TRUE   # true,True都可以
    - FALSE  # false，False都可以
float:
    - 3.14
    - 6.8523015e+5  # 科学计数法
int:
    - 123
    - 0b1010_0111_0100_1010_1110    #二进制表示
null:
    nodeName: 'node'
    parent: ~  # 使用~表示 null
string:
    - 哈哈
    - 'Hello world'  # 可以使用双引号或者单引号包裹特殊字符
    - newline
      newline2    # 字符串可以拆成多行，每一行会被转化成一个空格
date:
    - 2018-07-17    # 日期必须使用ISO 8601格式，即yyyy-MM-dd
datetime: 
    -  2018-07-17T19:02:31+08:00    # 时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区

对应:
{'boolean': [True, False],
 'float': [3.14, 685230.15],
 'int': [123, 685230],
 None: {'nodeName': 'node', 'parent': None},
 'string': ['哈哈', 'Hello world', 'newline newline2'],
 'date': [datetime.date(2018, 7, 17)],
 'datetime': [datetime.datetime(2018, 7, 17, 11, 2, 31)]}
```

***

参考：

[https://www.yiibai.com/yaml](https://www.yiibai.com/yaml)，Maxsu

