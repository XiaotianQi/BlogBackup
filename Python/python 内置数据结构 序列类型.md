## 序列的分类

------

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

## 序列类支持的型操作



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

***

### 6.嵌套列表

嵌套列表就是列表推导式相关内容。

```python
list_variable = [out_exp for out_exp in input_list if exp]
```

***

### 7.扩展

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

#### `+` 与 `+=` 区别

```python
In [16]: l = [1 , 2, 3]

In [17]: l = l + (4, 5)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-17-c70fae2ee178> in <module>
----> 1 l = l + (4, 5)

TypeError: can only concatenate list (not "tuple") to list

In [18]: l += (4, 5)

In [19]: l
Out[19]: [1, 2, 3, 4, 5]
```

`+` 需要两侧必须是相同类型的对象，而 `+=` 则没有要求。

除此之外，`+` 会改变对象标识。

```python
In [20]: l = [1, 2]

In [21]: id(l)
Out[21]: 1612094081352

In [22]: l += [3, 4]

In [23]: id(l)
Out[23]: 1612094081352

In [24]: l = l + [5, 6]

In [25]: id(l)
Out[25]: 1612093971528
```

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

***

## `bytes`

bytes：具有有序、不可修改、可迭代的特点

bytearray：具有有序、可修改、可迭代的特点

***

参考：

[Data Structures](https://docs.python.org/3/tutorial/datastructures.html)

[Sequence Types —list, tuple, range](https://docs.python.org/3/library/stdtypes.html#typesseq)