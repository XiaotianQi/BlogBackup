## 序列的分类

------

Python 中的序列主要有下几种类型：

- 基本序列类型(Basic Sequence Types)：list、tuple、range
- 专门处理文本的附加序列类型(Text Sequence Types)：str
- 专门处理二进制数据的附加序列类型(Binary Sequence Types): bytes、bytearray、memoryview

按照序列是否可被改变分类：

- 可变序列: list
- 不可变序列：tuple、str

***

## 序列类支持的型操作



***

## `list`

### 创建方式

- 使用方括号，用逗号分隔各条目：`[]`,`['a']`, `[a, b, c]`
- 使用类型构造函数：`list(iterable)`
- 使用列表生成式：`[x for x in iterable]`

***

### 函数

* `len(list)`:返回列表元素个数；
* `max(list)`:返回列表元素最大值；
* `min(list)`:返回列表元素最小值；
* `list(seqq)`:将 tuple 转化为list，返回 list。

***

### 方法

#### 有返回值的方法

* `list.count(obj)`

统计某个元素在列表中出现的次数。

返回元素在列表中出现的次数。

```python
l = ['a', 'b', 'c', 'a']

print(l.count('a'))


输出结果：
2
```

***

* `list.index(obj)`

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

返回复制后的新列表。

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

* list.append(obj)

在列表末尾添加新的对象。

```python
l = ['a', 'b', 'c']
l.append('d')
print(l)


输出结果：
['a', 'b', 'c', 'd']
```

* list.extend(seq)

在列表中添加新的列表内容。

```python
l = ['a', 'b', 'c']
l1 = [i for i in range(0,3)]
l.extend(l1)
print(l)


输出结果：
['a', 'b', 'c', 0, 1, 2]
```

* list.insert(index, obj)

将指定对象插入列表的指定位置。

```python
l = ['a', 'b', 'c']
l.insert(1, 'A')
print(l)


输出结果：
['a', 'A', 'b', 'c']
```

* list.remove(obj)

移除列表中某个值的第一个匹配项。

```python
l = ['a', 'b', 'c' ,'a', 'b', 'c']
l.remove('a')
print(l)


输出结果：
['b', 'c', 'a', 'b', 'c']
```

* list.reverse()

反向列表中元素。

```python
l = ['a', 'b', 'c']
l.reverse()
print(l)


输出结果：
['c', 'b', 'a']
```

* **list.sort(cmp=None, key=None, reverse=False)**

对原列表进行排序，如果指定参数，则使用比较函数指定的比较函数。

cmp -- 可选参数, 如果指定了该参数会使用该参数的方法进行排序。

key -- 主要是用来进行比较的元素，只有一个参数，具体的函数的参数就是取自于可迭代对象中，指定可迭代对象中的一个元素来进行排序。

reverse -- 排序规则，reverse = True 降序， reverse = False 升序（默认）。

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

清空列表，类似于 del list[:]。

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

## 扩展

### `list.copy()` 和 `=` 的区别

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

### `.append()`  与 `.extend()` 区别



***

### `+` 与 `+=` 区别