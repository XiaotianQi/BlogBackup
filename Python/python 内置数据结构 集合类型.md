集合有两种类型：

* `set`：可变，不存在哈希值。
* `frozenset`：冻结的集合，不可变的，存在哈希值。好处是它可以作为字典的key，也可以作为其它集合的元素。缺点是一旦创建便不能更改。

## `set`

`set` 是一个无序不重复元素的序列。

可以使用大括号 `{}` 或者 `set()` 函数创建集合，注意：创建一个空集合必须用 `set()` 而不是 `{ }`，因为 `{ }` 是用来创建一个空字典。

> A set is an unordered collection with no duplicate elements.  Basic uses include membership testing and eliminating duplicate entries.  Set objects also support mathematical operations like union, intersection, difference, and symmetric difference.

### 1.基本函数与操作

#### 添加元素

* `s.add(x)`

将元素 `x` 添加到集合 `s` 中，若重复则不进行任何操作。

#### 删除元素

* `s.discard(x)`

将 `x` 从集合 `s` 中移除，若 `x` 不存在，不会引发错误。

* `s.remove(x)`

将 `x` 从集合 `s` 中移除，若 `x` 不存在，会引发错误。

* `s.pop()`

随机删除并返回集合 `s` 中某个值。因为set是无序的，不支持下标操作，`pop()` 移除随机一个元素。

* `s.clear()`

清空。

#### 其他

还有 `len(s)` 和 `in` 操作。

***

### 2.数学运算

集合对象还支持 union(并集)，intersection(交集)，difference(补集) 和 sysmmetric difference(对称差)数学运算。

```python
In [14]: a = {1, 1, 2, 2, 3}

In [15]: b = set([1, 1, 2, 4])

In [22]: a.union(b)
Out[22]: {1, 2, 3, 4}

In [23]: a.intersection(b)
Out[23]: {1, 2}

In [24]: a.difference(b)
Out[24]: {3}

In [25]: b.difference(a)
Out[25]: {4}

In [26]: a.symmetric_difference(b)
Out[26]: {3, 4}

In [27]: b.symmetric_difference(a)
Out[27]: {3, 4}

In [28]: a
Out[28]: {1, 2, 3}

In [29]: b
Out[29]: {1, 2, 4}
```

数学运算并未改变 `a` 和 `b`。

上述代码也可写作如下模式：

```python
In [30]: a|b
Out[30]: {1, 2, 3, 4}

In [31]: a&b
Out[31]: {1, 2}

In [32]: a-b
Out[32]: {3}

In [33]: b-a
Out[33]: {4}

In [34]: a^b
Out[34]: {3, 4}
```

***

### 3. 更新

* `s.update(x)`

  将集合 `x` 并入原集合 `s` 中，`x` 还可以是列表，元组，字典等，无法添加 int。`x` 可以有多个，用逗号分开。

  以下代码示例中，均默认 `s1 = {1, 2 ,3}`，`s2 = {2, 3, 4}`.

  ```python
  In [33]: s1.update(s2)
  
  In [34]: s1
  Out[34]: {1, 2, 3, 4}
  ```

* `intersection_update(others)`

  Update the set, keeping only elements found in it and all others.

  ```python
  In [27]: s1.intersection_update(s2)
  
  In [28]: s1
  Out[28]: {2, 3}
  ```

* `difference_update(others)`

  Update the set, removing elements found in others.

  ```python
  In [29]: s1 = set([1, 2, 3])
  
  In [30]: s1.difference_update(s2)
  
  In [31]: s1
  Out[31]: {1}
  ```

* `symmetric_difference_update(other)`

  Update the set, keeping only elements found in either set, but not in both.

  ```python
  In [36]: s1.symmetric_difference_update(s2)
  
  In [37]: s1
  Out[37]: {1, 4}
  ```


***

### 4.判断

* `s.issubset(x)`

判断集合 `s` 是否是集合 `x` 子集。

* `s.issuperset(x)`

判断集合 `x` 是否是集合 `s` 的子集。

```python
In [37]: a = {1, 2, 3}

In [38]: b = {2,}

In [39]: b.issubset(a)
Out[39]: True

In [40]: a.issuperset(b)
Out[40]: True
```

***

### 5.可哈希

> Set elements, like dictionary keys, must be hashable.

set 中每一个元素都是唯一而不重复的，正是通过每个元素对象的 hash 值唯一来保证的，因此:

* set 是不可哈希的，因为集合本身是可变对象。

* set 中的元素是可哈希的，因为集合中无法存入可变对象。

```python
In [16]: s = set([1, 2, 3])

In [17]: s
Out[17]: {1, 2, 3}

In [18]: hash(s)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-18-9333020f3184> in <module>
----> 1 hash(s)

TypeError: unhashable type: 'set'

In [19]: for i in s:
    ...:     print(hash(i))
    ...:
1
2
3
```

***

### 6.示例

* 获取列表中，多次出现的元素

```python
In [47]: l = ['a', 'b', 'c', 'b', 'd', 'm', 'n', 'n']

In [48]: s = set([i for i in l if l.count(i)>1])

In [49]: s
Out[49]: {'b', 'n'}
```

* 删除序列相同元素并保持顺序

```python
def dedupe(items, key=None):
    seen = set()
    for item in items:
        val = item if key is None else key(item)
        if val not in seen:
            yield item
            seen.add(val)
```

```python
a = [ {'x':1, 'y':2}, {'x':1, 'y':3}, {'x':1, 'y':2}, {'x':2, 'y':4}]
print(list(dedupe(a, key=lambda d: (d['x'], d['y']))))
print(list(dedupe(a, key=lambda d: (d['x']))))
```

```bash
[{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 2, 'y': 4}]
[{'x': 1, 'y': 2}, {'x': 2, 'y': 4}]
```

***

## `fozenset`

冻结的集合，不可变的，存在哈希值。因此不能进行修改，其他同上。