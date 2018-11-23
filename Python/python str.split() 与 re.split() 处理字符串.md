
区别在于：

* str.split() 不支持正则及多个切割符号，适合简单的字符分割。
* re.split() 支持正则表达式和多个字符切割。

***

## str.split()

```python
str.split(str="", maxsplit=string.count(str))
```

* `str`：分隔符，默认为所有的空字符，包括空格、换行(`\n`)、制表符(`\t`)等。
* `maxsplit`：分割次数。

返回分割后的字符串列表。

通过指定分隔符对字符串进行切片，如果参数 `maxsplit` 有指定值，则分裂器会分裂出 `maxsplit + 1` 个元素。如果未给值，或者给了 -1 作为值，那就表示没有分裂数量限制。

```python
test_str = "a:b::c:d"
print(test_str.split(':', maxsplit=2))
print(test_str.split(':', maxsplit=-1))
print(test_str.split(':'))


输出结果：
['a', 'b', ':c:d']
['a', 'b', '', 'c', 'd']
['a', 'b', '', 'c', 'd']
```

* 配合 `*args` 分割字符串：

```python
In [10]: line = 'nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false'

In [11]: uname, *fields, homedir, sh =line.split(':')

In [12]: uname
Out[12]: 'nobody'

In [13]: homedir
Out[13]: '/var/empty'

In [14]: sh
Out[14]: '/usr/bin/false'
```

* 提取字典中的数值

```python
# 提取'author'的值
test_dict = "{'title':'XXX', 'author':'A, B, C', 'journal':'abc, CHINA', 'year':'2000'}"
for i in test_dict.split('\','):
    if 'author' in i:
        print(i)
        for j in i.split(':\'')[1].split(','):
            print(j.replace(' ', ''))


输出结果：
 A
 B
 C
```

或者如下：

```python
test_str = "{'title':'XXX', 'author':'A, B, C', 'journal':'abc, CHINA', 'year':'2000'}"
s = test_str.replace('{', '').replace('}', '').replace(' ', '').replace('\',\'', ';').replace('\'', '')
# print(s)
test_dict = dict(a.split(':') for a in s.split(';'))
# print(test_dict)
print(test_dict['author'])


输出结果：
A,B,C
```

* 解析 json

```python
test_json = '{"a":1,"b":2,"c":3,"d":4,"e":5}'
s = test_json.replace('{', '').replace('}', '').replace('\"', '')
# print(s)
test_dict = dict(a.split(':') for a in s.split(','))
print(test_dict)


输出结果：
{'a': '1', 'b': '2', 'c': '3', 'd': '4', 'e': '5'}
```

* 替代正则表达式

```python
test_str = "<p>a</p>"
print(test_str.split('>')[1].split('<')[0])


输出结果：
a
```

***

## str.splitlines()

```python
str.splitlines([keepends])
```

按照行('\r', '\r\n', \n')分隔，返回一个包含各行作为元素的列表，如果参数 keepends 为 False，不包含换行符，如果为 True，则保留换行符。

***

## re.split()

re.split() 按照能够匹配的子串将字符串分割后返回列表。

```python
re.split(pattern, string[, maxsplit=0, flags=0])
```

* pattern：匹配的正则表达式；
* string：需要匹配的目标字符串；
* maxsplit：分隔次数，默认为 0，不限制次数；
* flags：标志位，用于控制正则表达式的匹配方式，如：是否区分大小写，多行匹配等等。参见：正则表达式修饰符 - 可选标志。

```python
import re

test_str = "Words, words, words."
print(re.split(r'\W+', test_str))
print(re.split(r'(\W+)', test_str))


输出结果：
['Words', 'words', 'words', '']
['Words', ', ', 'words', ', ', 'words', '.', '']
```

**tips:**

* 如果没有加小括号，则返回结果就是用正则表达式匹配的 groups；
* 如果加了小括号，会返回所有的 groups，包括不匹配的 groups 也会包括在返回结果中。

当正则表达式比较复杂时，可以单独写正则表达式，方便阅读理解。以如下代码示例：

```python
import re

test_str = "Words, words, words."
regEx = re.compile(r'\W+')
print(regEx.split(test_str))
```

***

## str.rsplit()

```python
str.rsplit(sep=None, maxsplit=-1)
```

Return a list of the words in the string, using *sep* as the delimiter string. If maxsplit is given, at most *maxsplit* splits are done, the *rightmost* ones.  If *sep* is not specified or `None`, any whitespace string is a separator.  Except for splitting from the right, `rsplit()` behaves like `split()`.