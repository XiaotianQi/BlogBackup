`dict` 属于 `mapping` 类型。

> Unlike sequences, which are indexed by a range of numbers, dictionaries are indexed by *keys*, which can be any immutable type; strings and numbers can always be keys.  Tuples can be used as keys if they contain only strings, numbers, or tuples; if a tuple contains any mutable object either directly or indirectly, it cannot be used as a key. You can’t use lists as keys, since lists can be modified in place using index assignments, slice assignments, or methods like `append()` and `extend()`.

关于 `key`的两点：

* `key` 必须是不可变对象。
* 避免同时使用相同的 `key`。当存在多个相同的`key`时，那么存储最后一个`key`对应的值。

> A dictionary’s keys are *almost* arbitrary values.  Values that are not hashable, that is, values containing lists, dictionaries or other mutable types (that are compared by value rather than by object identity) may not be used as keys.  Numeric types used for keys obey the normal rules for numeric comparison: if two numbers compare equal (such as `1` and `1.0`) then they can be used interchangeably to index the same dictionary entry.  (Note however, that since computers store floating-point numbers as approximations it is usually unwise to use them as dictionary keys.)

字典可精确描述为不定长、可变、无序、散列的集合类型。遍历字典获得的键值对是随机无序的！

Python的字典数据类型是基于hash散列算法实现的，采用键值对(key:value)的形式，根据key的值计算value的地址，具有非常快的查取和插入速度。但它是无序的，包含的元素个数不限，值的类型也可以是其它任何数据类型！

***

## 创建方式

创建方式如下：

```python
a = dict(a=1, b=2, c=3)
b = {'a': 1, 'b': 2, 'c': 3}
c = dict(zip(['a', 'b', 'c'], [1, 2, 3]))
d = dict([('a', 1), ('b', 2), ('c', 3)])
e = {x:y for x, y in b.items()}
```

***

##  方法和函数

* `dict.clear()`

  Removes all Items

* `dict.copy()`

  Returns a shallow copy of dictionary

* `dict.items()`

  Returns a list of dict's (key, value) tuple pairs

* `dict.keys()`

  Returns a list of dictionary dict's keys

* `dict.values()`

  Returns a list of dictionary dict's values

* `iter(d)`

   Return an iterator over the keys of the dictionary.  This is a shortcut for `iter(d.keys())`.

   ```python
   In [48]: for i in a.keys():
       ...:     print(i)
       ...:
   a
   b
   c
   
   In [49]: for i in iter(a):
       ...:     print(i)
       ...:
   a
   b
   c
   
   In [50]: for i in a.items():
       ...:     print(i)
       ...:
   ('a', 1)
   ('b', 2)
   ('c', 3)
   ```

* `dict.update([other])`

  Update the dictionary with the key/value pairs from *other*, overwriting existing keys.  Return `None`.

  ```python
  In [76]: d1 = {'a':1, 'b':1}
  
  In [77]: d2 = {'b':2, 'c':2}
  
  In [78]: d1.update(d2)
  
  In [79]: d1
  Out[79]: {'a': 1, 'b': 2, 'c': 2}
  ```

* `dict.pop(key[, default])`

  Removes and returns element having given `key`.If `key` is in the dictionary, remove it and return its value, else return default.  If `default` is not given and `key` is not in the dictionary,
  a `KeyError` is raised.

* `dict.popitem()`

  Remove and return a `(key, value)` pair from the dictionary. Pairs are returned in LIFO order.

  `popitem()` is useful to destructively iterate over a dictionary, as often used in set algorithms.  If the dictionary is empty, calling `popitem()` raises a `KeyError`.

  ```python
  In [39]: a
  Out[39]: {'a': 1, 'b': 2, 'c': 3}
  
  In [40]: a.popitem()
  Out[40]: ('c', 3)
  
  In [41]: a.popitem()
  Out[41]: ('b', 2)
  
  In [42]: a.popitem()
  Out[42]: ('a', 1)
  
  In [43]: a
  Out[43]: {}
  
  In [44]: a.popitem()
  ---------------------------------------------------------------------------
  KeyError                                  Traceback (most recent call last)
  <ipython-input-44-e562075264b8> in <module>
  ----> 1 a.popitem()
  
  KeyError: 'popitem(): dictionary is empty'
  ```

* `classmethod dict.fromkeys(seq[, value])`

   Create a new dictionary with keys from *seq* and values set to *value*.

   `fromkeys()` is a class method that returns a new dictionary. *value* defaults to `None`.

   传入的序列必须是可哈希的，即不可变对象。

   ```python
   In [51]: seq = 'abc'
   
   In [52]: d = dict.fromkeys(seq)
   
   In [53]: d
   Out[53]: {'a': None, 'b': None, 'c': None}
   
   In [54]: d = dict.fromkeys(seq, 0)
   
   In [55]: d
   Out[55]: {'a': 0, 'b': 0, 'c': 0}
   ```

* `dict.get(key[, default])`

   Return the value for *key* if *key* is in the dictionary, else *default*.If *default* is not given, it defaults to `None`, so that this method never raises a `KeyError`.

* `dict.setdefault(key, default = None)`

   Similar to `dict.get()`, but will set `dict[key] = default` if `key` is not already in dict.

   If *key* is in the dictionary, return its value.  If not, insert *key* with a value of *default* and return *default*.  *default* defaults to `None`.

   调用时，不会出现因为`key`不存在而抛异常的情况。

   ```python
   In [71]: d
   Out[71]: {'a': 0, 'b': 0, 'c': 0}
   
   In [74]: d.get('d')
       
   In [72]: d.setdefault('d', 0)
   Out[72]: 0
   
   In [73]: d
   Out[73]: {'a': 0, 'b': 0, 'c': 0, 'd': 0}
   ```

## 相关

Dictionary order is guaranteed to be insertion order.

所以，使用`dict.popitem()`时，会遵循 LIFO.

此外，使用 `list()`创建列表式，`list(d)`等价于`list(d.keys())`.

```python
In [81]: d = {"one": 1, "two": 2, "three": 3, "four": 4}

In [82]: list(d)
Out[82]: ['one', 'two', 'three', 'four']

In [83]: list(d.values())
Out[83]: [1, 2, 3, 4]
```

遍历字典的多种方法：

```python
d = {'a': 1, 'b': 2, 'c': 3}

for k, v in d.items():
    print(k, v)

for i in d.items():
    print(i[0], i[1])
```

当然，使用`d.keys()`或者`d.values()`选择性遍历也是可以的。

***

参考：

[Python 3 - Dictionary](https://www.tutorialspoint.com/python3/python_dictionary.htm)