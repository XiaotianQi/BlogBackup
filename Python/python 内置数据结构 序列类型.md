## 序列的分类

------

序列表示索引为非负数的有序对象集合，支持迭代。

Python 中的序列主要有下几种类型：

* 容器序列：list、tuple、collections.deque，这些序列能存放不同类型的数据
* 扁平序列：str、bytes、bytearray、memoryview和array.array，这类序列只能容纳单一类型。

容器序列存放的是其所包含的任意类型的对象的引用，而扁平队列里存放的是值而不是引用，换句话说，扁平序列其实是一段连续的内存空间，因此，更加进错，但是，它们只能存储字符、字节和数值这种基础类型。

按照序列是否可被改变分类：

- 可变序列: list、bytearray、array.array、collections.deque、memoryview
- 不可变序列：tuple、str、bytes

> The only operation that immutable sequence types generally implement that is not also implemented by mutable sequence types is support for the `hash()` built-in.
>
> This support allows immutable sequences, such as `tuple` instances, to be used as `dict` keys and stored in `set` and `frozenset` instances.
>
> Attempting to hash an immutable sequence that contains unhashable values will result in `TypeError`.

下方的UML类图，列举了 collections.abc 中的几个类，显示了可变序列（MutableSequence）和不可变序列（Sequence）的差异，同时也可以看出前者从后者继承了一些方法。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/Data%20Structures%20-1.png)

虽然内置的序列类型并不是直接从 Sequence 和 MutableSequence 这两个抽象基类（Abstract Base Class，ABC）继承而来，但是了解这些基类可以帮助我们总结出那些完整的序列类型包含了哪些功能。

通过记住这些类的共有特性，把可变与不可变序列或是容器与扁平序列的概念融会贯通，在探索并学习新的序列类型时，更加得心应手。

***

## `list`

list 是一个可变序列，并且能同时存放不同类型的元素，具有有序、可索引、可修改、可迭代的特点。

> Lists are **mutable sequences**, typically used to store collections of **homogeneous items** (where the precise degree of similarity will vary by application).

### 1.创建方式

> - Using a pair of square brackets to denote the empty list: `[]`
> - Using square brackets, separating items with commas: `[a]`, `[a, b, c]`
> - Using a list comprehension: `[x for x in iterable]`
> - Using the type constructor: `list()` or `list(iterable)`

***

### 2.函数

* `len(list)`:返回列表元素个数；
* `max(list)`:返回列表元素最大值；
* `min(list)`:返回列表元素最小值；
* `list(seqq)`:将 tuple 转化为list，返回 list。

***

### 3.方法

#### 有返回值的方法

* `list.count(x)`

统计某个元素在列表中出现的次数。

返回元素在列表中出现的次数。

```python
l = ['a', 'b', 'c', 'a']

print(l.count('a'))		# 2
```

***

* `list.index(x[, start[, end]])`

从列表中找出某个值第一个匹配项的索引位置。

返回查找对象的索引位置，如果没有找到对象则抛出异常。

```python
l = ['a', 'b', 'c']
print(l.index('b'))


输出结果：
1
```

* `list.pop([index=-1]])`

移除列表中的一个元素（默认最后一个元素），并且返回该元素的值。

返回从列表中移除的元素对象。

index -- 可选参数，要移除列表元素的索引值，不能超过列表总长度，默认为 index=-1，删除最后一个列表值。

```python
l = ['a', 'b', 'c']
l.pop()
print(l)
a = l.pop(0)
print(a, l)


输出结果：
['a', 'b']
a ['b']
```

* `list.copy()`

返回复制后的新列表。等价于 `a[:]`。

```python
a = [0, 1, 2]
b = a.copy()
print(a, b)
a.pop()
print(a, b)
b.pop()
print(a, b)


输出结果：
[0, 1, 2] [0, 1, 2]
[0, 1] [0, 1, 2]
[0, 1] [0, 1]
```

***

#### 无返回值的方法

* `list.append(x)`

在列表末尾添加新的对象。等价于 `a[len(a):] = [x]`。

```python
l = ['a', 'b', 'c']
l.append('d')
print(l)		# ['a', 'b', 'c', 'd']
```

* `list.extend(iterable)`

在列表中添加新的列表内容。等价于 `a[len(a):] = iterable`。

```python
l = ['a', 'b', 'c']
l1 = [i for i in range(0,3)]
l.extend(l1)
print(l)		# ['a', 'b', 'c', 0, 1, 2]
```

* `list.insert(index, x)`

在指定位置插入对象。

> The first argument is the index of the element before which to insert, so `a.insert(0, x)` inserts at the front of the list, and `a.insert(len(a), x)` is equivalent to `a.append(x)`.

```python
l = ['a', 'b', 'c']
l.insert(1, 'A')
print(l)


输出结果：
['a', 'A', 'b', 'c']
```

* `list.remove(x)`

移除列表中某个值的**第一个**匹配项。

```python
l = ['a', 'b', 'c' ,'a', 'b', 'c']
l.remove('a')
print(l)


输出结果：
['b', 'c', 'a', 'b', 'c']
```

* `list.reverse()`

反向列表中元素。

```python
l = ['a', 'b', 'c']
l.reverse()
print(l)


输出结果：
['c', 'b', 'a']
```

* `list.sort(key=None, reverse=False)`

对原列表进行排序，如果指定参数，则使用比较函数指定的比较函数。

`key` -- 主要是用来进行比较的元素，只有一个参数，具体的函数的参数就是取自于可迭代对象中，指定可迭代对象中的一个元素来进行排序。

`reverse` -- 排序规则，`reverse = True` 降序， `reverse = False` 升序（默认）。

```python
l = ['bc', 'a', 'def']
l.sort()
print(l)
l.sort(reverse=True)
print(l)
l.sort(key=lambda x:len(x))
print(l)


输出结果：
['a', 'bc', 'def']
['def', 'bc', 'a']
['a', 'bc', 'def']
```

根据列表中元素的一个或者多个属性进行排序：

```python
l = [
    (1, 3, 'a'), (2, 2, 'a'), (3, 1, 'b')
]
l.sort(key=lambda x:x[2])
print(l)
l.sort(key=lambda x:x[2]+str(x[1]))
print(l)


输出结果：
[(1, 3, 'a'), (2, 2, 'a'), (3, 1, 'b')]
[(2, 2, 'a'), (1, 3, 'a'), (3, 1, 'b')]
```

当同时存在 int 和 str 时，二者无法直接比较。

* `list.clear()`

清空列表，等价于 `del list[:]`。

```python
l = ['a', 'b', 'c']
l.clear()
print(l)
l = ['a', 'b', 'c']
del l[:]
print(l)


输出结果：
[]
[]
```

***

#### list.sort方法和内置函数sorted

`list.sort` 方法会就地排序列表，也就是说不会把原列表复制一份。这也是这个方法的返回值是 `None` 的原因，提醒你本方法不会新建一个列表。在这种情况下返回 `None` 其实是 Python 的一个惯例：如果一个函数或者方法对对象进行的是就地改动，那它就应该返回 `None`，好让调用者知道传入的参数发生了变动，而且并未产生新的对象。

用返回 `None` 来表示就地改动这个惯例有个弊端，那就是调用者无法将其串联起来。而返回一个新对象的方法（比如说 str 里的所有方法）则正好相反，它们可以串联起来调用，从而形成连贯接口（fluent interface）。

与 `list.sort` 相反的是内置函数 `sorted`，它会新建一个列表作为返回值。这个方法可以接受任何形式的可迭代对象作为参数，甚至包括不可变序列或生成器。而不管 `sorted` 接受的是怎样的参数，它最后都会返回一个列表。

***

### 4.列表推导

列表推导是构建列表的快捷方式，具有更好的可读性和更高的效率。

具体内容已在《python 函数式编程 迭代、推导式、生成器》中写明。

***

### 5.扩展

#### Using Lists as Stacks

堆栈是一种特定的数据结构，遵循先进先出。将列表的表头作为栈底，表尾作为栈顶，就形成了一个堆栈。

* append()方法可以把一个元素添加到堆栈顶部（实际上就是在列表的尾部添加一个元素）。
* pop()方法可以把一个元素从堆栈顶释放出来（也就是从列表尾部弹出一个元素）。

> The list methods make it very easy to use a list as a stack, where the last element added is the first element retrieved (“**last-in, first-out**”).  To add an item to the top of the stack, use `append()`.  To retrieve an item from the top of the stack, use `pop()` without an explicit index.  For example:

```python
In [37]: stack = [0, 1, 2]

In [38]: stack.append(3)

In [39]: stack.append(4)

In [40]: stack
Out[40]: [0, 1, 2, 3, 4]

In [41]: stack.pop()
Out[41]: 4

In [42]: stack.pop()
Out[42]: 3

In [43]: stack.pop()
Out[43]: 2

In [44]: stack
Out[44]: [0, 1]
```

***

#### Using Lists as Queues

> It is also possible to use a list as a queue, where the first element added is the first element retrieved (“**first-in, first-out**”); however, lists are not efficient for this purpose.  While appends and pops from the end of list are fast, doing inserts or pops from the beginning of a list is slow (because all of the other elements have to be shifted by one).

```python
In [44]: stack
Out[44]: [0, 1]

In [45]: queue = [0, 1, 2]

In [46]: queue.append(3)

In [47]: queue.pop(0)
Out[47]: 0
```

> To implement a queue, use `collections.deque` which was designed to have fast appends and pops from both ends.  For example:

```python
In [48]: from collections import deque

In [49]: queue = deque([0, 1, 2])

In [50]: queue.append(3)

In [51]: queue.popleft()
Out[51]: 0

In [52]: queue
Out[52]: deque([1, 2, 3])
```

但是用Python的列表做队列的效率并不高。因为，虽然在列表的最后添加或者弹出元素速度很快，但在列头部弹出第一个元素的速度却不快（因为所有其他的元素都得跟着一个一个地往左移动一位）。通常我们使用queue. Queue作为单向队列，使用collections.deque作为双向队列。

***

#### 列表元素修改

```python
lst = [i for i in range(1, 5)]

# 迭代过程中对元素的修改无效
for i in lst:
    i = i + 1

# 可修改为：
for i, j in enumerate(lst, 0):
    lst[i] += 1
    # j += 1 便是和上边一样的错误

# 更简单的实现
lst = [i+1 for i in lst]
```

#### 列表求平均值 

```python
data = [[240, 240, 239],
        [250, 249, 237], 
        [242, 239, 237],
        [240, 234, 233]]
avg = [float(sum(col))/len(col) for col in zip(*data)]
# [243.0, 240.5, 236.5]
```

#### `list.copy()` 和 `=` 的区别

```python
a = [0, 1, 2]
b = a
c = a.copy()
d = a[:]
print(a, b, c, d)
a.pop()
print(a, b, c, d)
b.pop()
print(a, b, c, d)


输出结果：
[0, 1, 2] [0, 1, 2] [0, 1, 2] [0, 1, 2]
[0, 1] [0, 1] [0, 1, 2] [0, 1, 2]
[0] [0] [0, 1, 2] [0, 1, 2]
```

由此可见，使用 = 直接赋值，是引用赋值，更改一个，另一个同样跟着改变。

copy() 则复制一个副本给予新的变量，原变量和新变量互不影响。

***

#### `.append()`  与 `.extend()` 区别

```python
In [1]: l1 = [i for i in range(5)]

In [2]: l2 = [i for i in range(5)]

In [3]: l1, l2
Out[3]: ([0, 1, 2, 3, 4], [0, 1, 2, 3, 4])

In [4]: l1.append([5, 6, 7])

In [5]: l2.extend([5, 6, 7])

In [6]: l1, l2
Out[6]: ([0, 1, 2, 3, 4, [5, 6, 7]], [0, 1, 2, 3, 4, 5, 6, 7])
```

`list.extend(iterable)`中传入的可迭代对象

***

#### `x * n`

 对于 `x * n`操作，x 序列中的元素没有被复制，他们只是被引用了多次。

```python
In [34]: l = [[0]] * 3

In [35]: l[0] += [1]

In [36]: l
Out[36]: [[0, 1], [0, 1], [0, 1]]
```

***

## `tuple`

元组具有有序、不可修改、可索引、可迭代的特点。

> Tuples are **immutable sequences**, typically used to store collections of **heterogeneous data** (such as the 2-tuples produced by the `enumerate()` built-in). 
>
> Tuples are also used for cases where an immutable sequence of **homogeneous data is needed** (such as allowing storage in a `set` or `dict` instance).

元组其实是对数据的记录：元组中的每个元素都存放了记录中的一个字段的数据，外加这个字段的位置。正是这个位置信息给数据赋予的意义。

如果只把元组理解为不可变的列表，那其他信息——它所含有的元素的总数和它们的位置——似乎就变得可有可无。但是如果把元组当作一些字段的集合，那么数量和位置信息就变得非常重要了。因此，命名元组具有两个功能：

* 不可变的列表
* 可以当做记录使用的数据类型

除此之外，不要把可变对象放在元组里面。

不过，元组作为记录来用的话，还是缺少了一个功能：记录的字段名。因此，`collections.namedtuple()`函数解决这个问题。

### 1.创建方式

> - Using a pair of parentheses to denote the empty tuple: `()`
> - Using a trailing comma for a singleton tuple: `a,` or `(a,)`
> - Separating items with commas: `a, b, c` or `(a, b, c)`
> - Using the `tuple()` built-in: `tuple()` or `tuple(iterable)`

> **Note that** it is actually the comma which makes a tuple, not the parentheses.
> The parentheses are optional, except in the empty tuple case, or when they are needed to avoid syntactic ambiguity. 

> For heterogeneous collections of data where **access by name** is clearer than access by index, [`collections.namedtuple()` may be a more appropriate choice than a simple tuple object.

### 2. 元组拆分

该部分已在《python 变量与对象 赋值、浅拷贝与深拷贝》中写明。

### 3. `tuple` 和 `list` 区别

> Tuples are immutable, and usually contain a heterogeneous sequence of elements that are accessed via unpacking (see later in this section) or indexing (or even by attribute in the case of `namedtuples`.
> Lists are mutable, and their elements are usually homogeneous and are accessed by iterating over the list.

元组中不允许的操作，确切的说是元组没有的功能，除了跟增减元素相关的方法外，元组支持列表的其他所有方法。

不过，虽然元组没有`__reversed__`方法，但是`reversed(my_tuple)`仍然可以使用，只改变了输出结果，不会改变元组的元素的位置。

元组与列表相同的操作：

* 下标访问
* 切片（形成新元组对象）
* count()/index()
* len()/max()/min()/tuple()
* ...

Note：元组只保证它的一级子元素不可变，对于嵌套的元素内部，不保证不可变！

***

## `range`

`range`类型表示一个不可变的数字序列，通常用于在 `for`循环。 

```python
range(start, stop[, step])
```

The **advantage** of the `range` type over a regular `list` or `tuple` is that a `range` object will always take the same (small) amount of memory, no matter the size of the range it represents (as it only stores the `start`, `stop` and `step` values, calculating individual items and subranges as needed).

* step>0时，stop值必须大于start值
* step<0时，start值必须大于stop值

```bash
In [8]: list(range(-10))
Out[8]: []

In [9]: list(range(0, -10, -1))
Out[9]: [0, -1, -2, -3, -4, -5, -6, -7, -8, -9]

In [10]: list(range(-10, 0))
Out[10]: [-10, -9, -8, -7, -6, -5, -4, -3, -2, -1]

In [14]: list(range(10, 0, -1))
Out[14]: [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```

***

## `str`

字符串具有有序、可索引、不可修改、可迭代的特点。
### 常用方法

列出常用的方法，英文部分引用自官方文档：[Sequence Types —list, tuple, range](https://docs.python.org/3/library/stdtypes.html#typesseq)。

#### 1.大小写

* `str.lower()`

  Return a copy of the string with all the cased characters converted to lowercase.

* `str.upper()`

  Return a copy of the string with all the cased characters converted to uppercase.

* `str.capitalize()`

  Return a copy of the string with its first character capitalized and the rest lowercased.

* `str.swapcase()`

  Return a copy of the string with uppercase characters converted to lowercase and vice versa. Note that it is not necessarily true that `s.swapcase().swapcase() == s`.

  ```python
  In [1]: str = 'abcABC'
  
  In [2]: str.lower()
  Out[2]: 'abcabc'
  
  In [3]: str.upper()
  Out[3]: 'ABCABC'
  
  In [4]: str.capitalize()
  Out[4]: 'Abcabc'
      
  In [5]: str.swapcase()
  Out[5]: 'ABCabc'
  ```

* `str.title()`

  Return a titlecased version of the string where words start with an uppercase character and the remaining characters are lowercase.

  ```python
  In [42]: str = 'a string example: 3g 5g e.g.'
  
  In [43]: str.title()
  Out[43]: 'A String Example: 3G 5G E.G.'
  
  In [44]: str.capitalize()
  Out[44]: 'A string example: 3g 5g e.g.'
  ```

#### 2.连接&分割

* `str.join(iterable)`

  Return a string which is the concatenation of the strings in iterable.

  ```python
  In [33]: str = ['a', 'b', 'c']
  
  In [34]: ('-').join(str)
  Out[34]: 'a-b-c'
  
  In [35]: str = 'abc'
  
  In [36]: ('-').join(str)
  Out[36]: 'a-b-c'
  ```

* `str.split(sep=None, maxsplit=-1)`

  Return a list of the words in the string, using *sep* as the delimiter string.

  - `str`：分隔符，默认为所有的空字符，包括空格、换行(`\n`)、制表符(`\t`)等。
  - `maxsplit`：分割次数。

  返回分割后的字符串列表。

  通过指定分隔符对字符串进行切片，如果参数 `maxsplit` 有指定值，则分裂器会分裂出 `maxsplit + 1` 个元素。如果未给值，或者给了 -1 作为值，那就表示没有分裂数量限制。

  ```python
  test_str = "a:b::c:d"
  print(test_str.split(':', maxsplit=2))	# ['a', 'b', ':c:d']
  print(test_str.split(':', maxsplit=-1))	# ['a', 'b', '', 'c', 'd']
  print(test_str.split(':'))				# ['a', 'b', '', 'c', 'd']
  ```

  - 配合 `*args` 分割字符串：

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

  - 提取字典中的数值

  ```python
  # 提取'author'的值
  test_dict = "{'title':'XXX', 'author':'A, B, C', 'journal':'abc, CHINA', 'year':'2000'}"
  for i in test_dict.split('\','):
      if 'author' in i:
          print(i)
          for j in i.split(':\'')[1].split(','):
              print(j.replace(' ', ''))
  
  
  输出结果：
  'author':'A, B, C
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
  print(test_dict['author'])	# A,B,C
  ```

  - 解析 json

  ```python
  test_json = '{"a":1,"b":2,"c":3,"d":4,"e":5}'
  s = test_json.replace('{', '').replace('}', '').replace('\"', '')
  # print(s)
  test_dict = dict(a.split(':') for a in s.split(','))
  print(test_dict)	# {'a': '1', 'b': '2', 'c': '3', 'd': '4', 'e': '5'}
  ```

  - 替代正则表达式

  ```python
  test_str = "<p>a</p>"
  print(test_str.split('>')[1].split('<')[0])	# a
  ```

* `str.rsplit(sep=None, maxsplit=-1)`

  Splitting from the right.

* `str.splitlines([keepends])`

  Return a list of the lines in the string, breaking at line boundaries.  Line breaks are not included in the resulting list unless keepends is given and true.

  按照行`\r`, `\r\n`, `\n`)分隔，返回一个包含各行作为元素的列表，如果参数 keepends 为 False，不包含换行符，如果为 True，则保留换行符。

* `str.partition(sep)`

  Split the string at the first occurrence of sep, and return a 3-tuple containing the part before the separator, the separator itself, and the part after the separator.  If the separator is not found, return a 3-tuple containing the string itself, followed by two empty strings.

* `str.partition(sep)`

  Split the string at the last occurrence of *sep*, and return a 3-tuple containing the part before the separator, the separator itself, and the part after the separator.  If the separator is not found, return a 3-tuple containing two empty strings, followed by the string itself.

  ```python
  In [32]: str = 'www.python.com'
  
  In [33]: str.partition('.')
  Out[33]: ('www', '.', 'python.com')
  
  In [34]: str.rpartition('.')
  Out[34]: ('www.python', '.', 'com')
  ```

#### 3.格式化

* `str.format(args, kwargs)`

  字符串格式化，关于 `format` 的使用有很多技巧，可以查看已写的《python 字符串格式化》了解更多。

* `str.center(width[, fillchar])`

  Return centered in a string of length *width*. Padding is done using the specified fillchar (default is an ASCII space). The original string is returned if width is less than or equal to `len(s)`.

* `str.ljust(width[, fillchar])`

  Return the string left justified in a string of length *width*. Padding is done using the specified fillchar (default is an ASCII space). The original string is returned if width is less than or equal to `len(s)`.

* `str.rjust(width[, fillchar])`

  Return the string right justified in a string of length *width*. Padding is done using the specified *fillchar* (default is an ASCII space). The original string is returned if *width* is less than or equal to `len(s)`.

  ```python
  In [4]: str = 'abcABC'
  
  In [5]: str.center(10,'-')
  Out[5]: '--abcABC--'
      
  In [6]: str.ljust(10, '-')
  Out[6]: 'abcABC----'
  
  In [7]: str.rjust(10, '-')
  Out[7]: '----abcABC'
  ```

* `str.zfill(width)`

  Return a copy of the string left filled with ASCII `'0'` digits to make a string of length *width*. A leading sign prefix (`'+'`/`'-'`) is handled by inserting the padding *after* the sign character rather than before. The original string is returned if *width* is less than or equal to `len(s)`.

  ```python
  In [45]: str = 'abc'
  
  In [46]: str.zfill(5)
  Out[46]: '00abc'
  ```

#### 4.查找

- `str.find(sub[,start[,end]])`

  Return the lowest index in the string where substring *sub* is found within the slice `s[start:end]`.  Optional arguments *start* and *end* are interpreted as in slice notation.  Return `-1` if *sub* is not found.

- `str.index(sub[,start[,end]])`

  Like `find()`, but raise `ValueError` when the substring is not found.

  ```python
  In [16]: str
  Out[16]: 'a string example'
  
  In [17]: str.find('str')
  Out[17]: 2
  
  In [18]: str.find('exam')
  Out[18]: 9
  
  In [19]: str.find('abc')
  Out[19]: -1
  
  In [20]: str.index('abc')
  ---------------------------------------------------------------------------
  ValueError                                Traceback (most recent call last)
  <ipython-input-20-daba474a0c9f> in <module>
  ----> 1 str.index('abc')
  
  ValueError: substring not found
  ```

- `str.rfind(sub[, start[, end]])`

  Return the highest index in the string where substring *sub* is found, such that *sub* is contained within `s[start:end]`.  Optional arguments *start* and *end* are interpreted as in slice notation.  Return `-1` on failure.

- `str.index(sub[,start[,end]])`

  Like `rfind()` but raises `ValueError` when the substring *sub* is not found.

  ```python
  In [26]: str = 'abcabc'
  
  In [27]: str.rfind('a')
  Out[27]: 3
  
  In [28]: str.rfind('d')
  Out[28]: -1
  
  In [29]: str.rindex('d')
  ---------------------------------------------------------------------------
  ValueError                                Traceback (most recent call last)
  <ipython-input-29-2cb78aee13b2> in <module>
  ----> 1 str.rindex('d')
  
  ValueError: substring not found
  ```

#### 5.判断


* `str.startwith(prefix[, start[, end]])`

  Return `True` if string starts with the *prefix*, otherwise return `False`.prefix can also be a tuple of prefixes to look for.  With optional *start*,test string beginning at that position.  With optional *end*, stop comparing string at that position.

  ```python
  In [11]: str = 'a string example'
  
  In [12]: str.startswith('e')
  Out[12]: False
  
  In [13]: str.startswith(('e', 'a'))
  Out[13]: True
  ```

* `str.endswith(suffix[, start[, end]])`

  Return `True` if the string ends with the specified suffix, otherwise return`False`.  *suffix* can also be a tuple of suffixes to look for.  With optional  start, test beginning at that position.  With optional *end*, stop comparing at that position.

  ```python
  In [14]: str.endswith('e')
  Out[14]: True
  
  In [15]: str.endswith(('e', 'a'))
  Out[15]: True
  ```

* `str.count(sub[, start[, end]])`

  Return the number of non-overlapping occurrences of substring *sub* in the range [*start*, *end*].  Optional arguments *start* and *end* are interpreted as in slice notation.

  ```python
  In [6]: str = 'a string example'
  
  In [7]: str.count('a')
  Out[7]: 2
  ```

* `str.isalpha()`

  Return true if all characters in the string are alphabetic and there is at least one character, false otherwise.  

  ```python
  In [21]: str = 'abc'
  
  In [22]: str.isalpha()
  Out[22]: True
  
  In [23]: str = 'a b c'
  
  In [24]: str.isalpha()
  Out[24]: False
  ```

* `str.isdigit()`

  Return true if all characters in the string are digits and there is at least one character, false otherwise.  Digits include decimal characters and digits that need special handling, such as the compatibility uperscript digits.

* `str.isalnum()`

  Return true if all characters in the string are alphanumeric and there is at least one character, false otherwise.  A character `c` is alphanumeric if one of the following returns `True`: `c.isalpha()`, `c.isdecimal()`,`c.isdigit()`, or `c.isnumeric()`.

  ```python
  In [25]: str = 'abc123'
  
  In [26]: str.isalnum()
  Out[26]: True
  
  In [27]: str = 'abc 123'
  
  In [28]: str.isalnum()
  Out[28]: False
  ```

* `str.isspace()`

  Return true if there are only whitespace characters in the string and there is at least one character, false otherwise. 

* `str.islower()`

  Return true if all cased characters in the string are lowercase and there is at least one cased character, false otherwise.

* `str.isupper()`

  Return true if all cased characters in the string are uppercase and there is at least one cased character, false otherwise.

* `str.istitle()`

  Return true if the string is a titlecased string and there is at least one character, for example uppercase characters may only follow uncased characters and lowercase characters only cased ones.  Return false otherwise.

  ```python
  In [29]: str = 'A String Example'
  
  In [30]: str.istitle()
  Out[30]: True
  
  In [31]: str = 'A string example'
  
  In [32]: str.istitle()
  Out[32]: False
  ```


#### 6.替换

* `str.replace(old, new[, count])`

  Return a copy of the string with all occurrences of substring *old* replaced by new.  If the optional argument count is given, only the first count occurrences are replaced.

  ```python
  In [23]: str = 'abcabc'
  
  In [24]: str.replace('a', '1')
  Out[24]: '1bc1bc'
  
  In [25]: str.replace('a', '1', 1)
  Out[25]: '1bcabc'
  ```

* `str.strip([chars])`

  Return a copy of the string with the **leading and trailing** characters removed.The *chars* argument is a string specifying the set of characters to be removed.If omitted or `None`, the *chars* argument defaults to removing whitespace.The *chars* argument is not a prefix or suffix; rather, all combinations of its
  values are stripped.

  The outermost leading and trailing *chars* argument values are stripped from the string. Characters are removed from the leading end until reaching a string character that is not contained in the set of characters in *chars*. A similar action takes place on the trailing end.

  ```python
  In [35]: str = '#....... Section ## 3.2.1 ## Issue #32 .......'
  
  In [38]: str.strip('.#')
  Out[38]: ' Section ## 3.2.1 ## Issue #32 '
  
  In [39]: str.rstrip('.#')
  Out[39]: '#....... Section ## 3.2.1 ## Issue #32 '
  ```

* `str.strip([chars])`

  删除 string 字符串末尾的指定字符。

- `static str.maketrans(x[, y[, z]])`

  This static method returns a translation table usable for `str.translate()`.

  If there is only one argument, it must be a dictionary mapping Unicode ordinals (integers) or characters (strings of length 1) to Unicode ordinals, strings (of arbitrary lengths) or `None`.  Character keys will then be converted to ordinals.

  If there are two arguments, they must be strings of equal length, and in the resulting dictionary, each character in x will be mapped to the character at the same position in y.  If there is a third argument, it must be a string, whose characters will be mapped to `None` in the result.

* `str.translate(table)`

  Return a copy of the string in which each character has been mapped through the given translation table.  The table must be an object that implements indexing via `__getitem__()`, typically a mapping or sequence.  When indexed by a Unicode ordinal (an integer), the table object can do any of the following: return a Unicode ordinal or a string, to map the character to one or more other characters; return `None`, to delete the character from the return string; or raise a `LookupError` exception, to map the character to itself.

  ```python
  In [9]: intab = 'abcde'
  
  In [10]: outab = '12345'
  
  In [11]: trantab = str.maketrans(intab, outab)
  
  In [12]: text = 'a string example'
      
  In [13]: text.translate(trantab)
  Out[13]: '1 str3ng 2x1mpl2'
  
  In [14]: trantab
  Out[14]: {97: 49, 101: 50, 105: 51, 111: 52, 117: 53}
  ```

- `str.lstrip([chars])`

  Return a copy of the string with leading characters removed.  The chars argument is a string specifying the set of characters to be removed.  If omitted or `None`, the *chars* argument defaults to removing whitespace.  The chars argument is not a prefix; rather, all combinations of its values are stripped.

  ```python
  In [17]: '   spacious   '.lstrip()
  Out[17]: 'spacious   '
  
  In [18]:  'www.example.com'.lstrip('cmowz.')
  Out[18]: 'example.com'
  ```

#### 7.编码

- `str.encode(encoding="utf-8", errors="strict")`

  Return an encoded version of the string as a bytes object. 

  ```python
  In [10]: str.encode()
  Out[10]: b'a string example'
      
  In [11]: b'a string example'.decode()
  Out[11]: 'a string example'
  ```

***

## `bytes`

字符串是以字符为单位进行处理的，bytes类型是以字节为单位处理的。

bytes对象只负责以二进制字节序列的形式记录所需记录的对象，至于该对象到底表示什么（比如到底是什么字符）则由相应的编码格式解码所决定。

bytes：具有有序、不可修改、可迭代的特点

bytearray：具有有序、可修改、可迭代的特点

***

## 切片

在 Python 里，像列表、元组和字符串这类序列类型都支持切片操作，但是实际上切片操作比想象的要强大很多。

对`seq[start:stop:step]` 进行求值的时候，Python 会调用`seq.__getitem__(slice(start, stop, step))`，对seq在start和stop之间以step为间隔取值，step值为负，则是反向取值。

### 为什么切片和区间会忽略最后一个元素

在切片和区间操作里不包含区间范围的最后一个元素是 Python 的风格，这个习惯符合 Python、C 和其他语言里以 0 作为起始下标的传统。这样做带来的好处如下：

* 当只有最后一个位置信息时，我们也可以快速看出切片和区间里有几个元素：range(3) 和 my_list[:3] 都返回 3 个元素。
* 当起止位置信息都可见时，我们可以快速计算出切片和区间的长度，用后一个数减去第一个下标（stop - start）即可。
* 利用任意一个下标来把序列分割成不重叠的两部分，只要写成 my_list[:x] 和 my_list[x:] 就可以了

```python
>>> l = [10, 20, 30, 40, 50, 60]
>>> l[:2] # 在下标2的地方分割
[10, 20]
>>> l[2:]
[30, 40, 50, 60]
>>> l[:3] # 在下标3的地方分割
[10, 20, 30]
>>> l[3:]
[40, 50, 60]

>>> s = 'bicycle'
>>> s[::3]
'bye'
>>> s[::-1]
'elcycib'
>>> s[::-2]
'eccb'
```

### 给切片赋值

如果把切片放在赋值语句的左边，或把它作为 del 操作的对象，我们就可以对序列进行嫁接、切除或就地修改操作。

```python
>>> l = list(range(10))
>>> l
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> l[2:5] = [20, 30]
>>> l
[0, 1, 20, 30, 5, 6, 7, 8, 9]
>>> del l[5:7]
>>> l
[0, 1, 20, 30, 5, 8, 9]
>>> l[3::2] = [11, 22]
>>> l
[0, 1, 20, 11, 5, 22, 9]
>>> l[2:5] = 100
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: can only assign an iterable
```

如果赋值的对象是一个切片，那么赋值语句的右侧必须是个可迭代对象。即便只有单独一个值，也要把它转换成可迭代的序列。

### 切片对象

给切片命名，就像电子表格软件里给单元格区域取名字一样。比如，要解析如下所示的纯文本文件，这时使用有名字的切片比用硬编码的数字区间要方便得多，注意示例里的 for 循环的可读性有多强。

```python
invoice = """
0.....6................................40........52...55........
1909  Pimoroni PiBrella                    $17.50    3    $52.50
1489  6mm Tactile Switch x20                $4.95    2     $9.90
1510  Panavise Jr. - PV-201                $28.00    1    $28.00
1601  PiTFT Mini Kit 320x240               $34.95    1    $34.95
"""
SKU = slice(0, 6)
DESCRIPTION = slice(6, 40)
UNIT_PRICE = slice(40, 52)
QUANTITY = slice(52, 55)
ITEM_TOTAL = slice(55, None)
line_items = invoice.split('\n')[2:]
for item in line_items:
    print(item[UNIT_PRICE], item[DESCRIPTION])
```

***

## 字符串的 intern 机制

驻留的具体表现是在内存中**不会创建新的对象**，而是引用已用的对象。

```python
>>> a = 'hello'
>>> b = 'hello'
>>> a is b
True
>>> a = 'hello world'
>>> b = 'hello world'
>>> a is b
False
```

```python
>>> a = 'hello' * 5
>>> b = 'hello' * 5
>>> a is b
True
>>> a = 'hello_' * 5
>>> b = 'hello_' * 5
>>> a is b
True
>>> a = 'hello ' * 5
>>> b = 'hello ' * 5
>>> a is b
False
```

当两个或以上的字符串变量它们的值相同且仅由**数字字母下划线**构成，或者值仅含有一个字符时，内存空间中只创建一个对象来让这些变量都指向该内存地址。当字符串不满足该条件时，相同值的字符串变量在创建时都会申请一个新的内存地址来保存值。

***

参考：

[Data Structures](https://docs.python.org/3/tutorial/datastructures.html)

[Python教程](https://www.liujiangblog.com/course/python/1)，刘江

Fluent Python