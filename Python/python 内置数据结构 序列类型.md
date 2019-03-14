## 序列的分类

------

序列表示索引为非负数的有序对象集合，支持迭代。

Python 中的序列主要有下几种类型：

- 基本序列类型(Basic Sequence Types)：list、tuple、range objects
- 专门处理文本的附加序列类型(Text Sequence Types)：str
- 专门处理二进制数据的附加序列类型(Binary Sequence Types): bytes、bytearray、memoryview

按照序列是否可被改变分类：

- 可变序列: list
- 不可变序列：tuple、str

> The only operation that immutable sequence types generally implement that is not also implemented by mutable sequence types is support for the `hash()` built-in.
>
> This support allows immutable sequences, such as `tuple` instances, to be used as `dict` keys and stored in `set` and `frozenset` instances.
>
> Attempting to hash an immutable sequence that contains unhashable values will result in `TypeError`.

相关 ABC：

* `collections.abc.Sequence`
* `collections.abc.MutableSequence`
* `collections.abc.ByteString`

通过这些抽象基类，可以便捷的自定义序列。

***

## `list`

列表具有有序、可索引、可修改、可迭代的特点。

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

print(l.count('a'))


输出结果：
2
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
print(l)


输出结果：
['a', 'b', 'c', 'd']
```

* `list.extend(iterable)`

在列表中添加新的列表内容。等价于 `a[len(a):] = iterable`。

```python
l = ['a', 'b', 'c']
l1 = [i for i in range(0,3)]
l.extend(l1)
print(l)


输出结果：
['a', 'b', 'c', 0, 1, 2]
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

### 4.Using Lists as Stacks

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

### 5.Using Lists as Queues

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

但是用Python的列表做队列的效率并不高。因为，虽然在列表的最后添加或者弹出元素速度很快，但在列头部弹出第一个元素的速度却不快（因为所有其他的元素都得跟着一个一个地往左移动一位）。通常我们使用queue.Queue作为单向队列，使用collections.deque作为双向队列。

***

### 6.嵌套列表

嵌套列表就是列表推导式相关内容。

```python
list_variable = [out_exp for out_exp in input_list if exp]
```

***

### 7.扩展

#### 遍历

```python
lst = [i for i in range(1, 5)]

for i in lst:
    i = i + 1	# 迭代过程中对元素的修改无效
print(lst)	# [1, 2, 3, 4]

for i in list(enumerate(lst, 0)):
    lst[i[0]] += 1
print(lst) 	# [2, 3, 4, 5]
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

### 1.创建方式

> - Using a pair of parentheses to denote the empty tuple: `()`
> - Using a trailing comma for a singleton tuple: `a,` or `(a,)`
> - Separating items with commas: `a, b, c` or `(a, b, c)`
> - Using the `tuple()` built-in: `tuple()` or `tuple(iterable)`

> **Note that** it is actually the comma which makes a tuple, not the parentheses.
> The parentheses are optional, except in the empty tuple case, or when they are needed to avoid syntactic ambiguity. 

> For heterogeneous collections of data where **access by name** is clearer than access by index, [`collections.namedtuple()` may be a more appropriate choice than a simple tuple object.

### 2. `tuple` 和 `list` 区别

> Tuples are immutable, and usually contain a heterogeneous sequence of elements that are accessed via unpacking (see later in this section) or indexing (or even by attribute in the case of `namedtuples`.
> Lists are mutable, and their elements are usually homogeneous and are accessed by iterating over the list.

元组与列表相同的操作：

    使用方括号加下标访问元素
    切片（形成新元组对象）
    count()/index()
    len()/max()/min()/tuple()

元组中不允许的操作，确切的说是元组没有的功能：

    修改、新增元素
    删除某个元素（但可以删除整个元组）
    所有会对元组内部元素发生修改动作的方法。例如，元组没有remove，append，pop等方法。
Note：元组只保证它的一级子元素不可变，对于嵌套的元素内部，不保证不可变！

***

## `range`

`range`类型表示一个不可变的数字序列，通常用于在 `for`循环。 

```python
range(start, stop[, step])
```

> The **advantage** of the `range` type over a regular `list` or `tuple` is that a `range` object will always take the same (small) amount of memory, no matter the size of the range it represents (as it only stores the `start`, `stop` and `step` values, calculating individual items and subranges as needed).

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

#### 2.连接&分隔

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

  此方法在文章《python str.split() 与 re.split() 处理字符串》有详细说明。

* `str.rsplit(sep=None, maxsplit=-1)`

  Splitting from the right.

  此方法在文章《python str.split() 与 re.split() 处理字符串》有详细说明。

* `str.splitlines([keepends])`

  Return a list of the lines in the string, breaking at line boundaries.  Line breaks are not included in the resulting list unless keepends is given and true.

  此方法在文章《python str.split() 与 re.split() 处理字符串》有详细说明。

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


#### 6. 替换

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

参考：

[Data Structures](https://docs.python.org/3/tutorial/datastructures.html)

[Sequence Types —list, tuple, range](https://docs.python.org/3/library/stdtypes.html#typesseq)

http://www.liujiangblog.com/course/python/1