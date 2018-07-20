`set` 是一个无序不重复元素的序列。

可以使用大括号 `{ }` 或者 `set()` 函数创建集合，注意：创建一个空集合必须用 `set()` 而不是 `{ }`，因为 `{ }` 是用来创建一个空字典。

## 基本函数与操作

### 添加元素

* `s.add(x)`

将元素 `x` 添加到集合 `s` 中，若重复则不进行任何操作。

* `s.update(x)`

将集合 `x` 并入原集合 `s` 中，`x` 还可以是列表，元组，字典等，无法添加 int。`x` 可以有多个，用逗号分开。

### 删除元素

* `s.discard(x)`

将 `x` 从集合 `s` 中移除，若 `x` 不存在，不会引发错误。

* `s.remove(x)`

将 `x` 从集合 `s` 中移除，若 `x` 不存在，会引发错误。

* `s.pop()`

随机删除并返回集合 `s` 中某个值。因为set是无序的，不支持下标操作，`pop()` 移除随机一个元素。

* `s.clear()`

清空。

### 其他

还有 `len(s)` 和 `in` 操作。

***

## 数学运算

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

## 判断

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

## 示例

获取列表中，多次出现的元素

```python
In [47]: l = ['a', 'b', 'c', 'b', 'd', 'm', 'n', 'n']

In [48]: s = set([i for i in l if l.count(i)>1])

In [49]: s
Out[49]: {'b', 'n'}
```