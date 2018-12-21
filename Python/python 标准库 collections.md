Python `collections` 模块包含许多容器数据类型。

* `namedtuple`：命名元组，是一个工厂函数；
* `defaultdict`：使用工厂函数创建字典，是一个工厂函数；
* `deque`：双向队列；
* `Counter`：为 `hashable` 对象计数，是 `dict` 的子类；
* `OrderedDict`：有序字典，是 `dict` 的子类；
* `ChainMap`：合并多个 `map(dict)`，但保持原数据结构。



| 数据类型       | 含义                                                         |
| -------------- | ------------------------------------------------------------ |
| `namedtuple()` | factory function for creating tuple subclasses with named fields |
| `deque`        | list-like container with fast appends and pops on either end |
| `ChainMap`     | dict-like class for creating a single view of multiple mappings |
| `Counter`      | dict subclass for counting hashable objects                  |
| `OrderedDict`  | dict subclass that remembers the order entries were added    |
| `defaultdict`  | dict subclass that calls a factory function to supply missing values |
| `UserDict`     | wrapper around dictionary objects for easier dict subclassing |
| `UserList`     | wrapper around list objects for easier list subclassing      |
| `UserString`   | wrapper around string objects for easier string subclassing  |

***

## `namedtuple`

```python
collections.namedtuple(typename, field_names, *, verbose=False, rename=False, module=None)
```

一个命名元组(namedtuple)有两个必需的参数。它们是元组名称(typename)和字段名称(field_names)。`field_names` 是可迭代对象，用于指定数据对象的元素。例如 `'x y'`、`'x, y'`、`['x', 'y']`。

`namedtuple` 主要用来产生可以使用名称来访问元素的数据对象，通常用来增强代码的可读性，在访问一些 tuple 类型的数据时尤其好用。`namedtuple` 是 `tuple` 的子类。

`namedtuple` 可以使用如下3中方法进行声明：

```python
User = namedtuple('User', ['name', 'age', 'height', 'weight'])
User = namedtuple('User', 'name age height weight')
User = namedtuple('User', 'name, age, height, weight')
```

实例可以采用以下5种方法，`field_names` 是可迭代对象即可：

```python
user1 = User(name='Bob', age=25, height=180, weight=75)
user2 = User('Alice', 22, 170, 55)

test_tuple = ('Clur', 24, 175)
user3 = User(*test_tuple, 65)

test_dict = {
    'name':'David',
    'age':27,
    'height':177,
    'weight':74
}
user4 = User(**test_dict)

test_list = ['Ema', 23, 160, 45]
user5 = User(*test_list)
```

输出相关属性的方式类似实例：

```python
print('user1: ', user1.name, user1.age, user1.height, user1.weight)
```

将命名元组转化为字典：

```python
user_dict = user5._asdict()
```

序列拆分：

```python
name, *others = user5
```

命名元组的一个主要用途是将代码从下标操作中解脱出来。因此，如果从数据库调用中返回了一个很大的元组列表，通过下标去操作其中的元素，当在表中添加了新的列的时候代码可能就会出错了。下标操作通常会让代码表意不清晰，并且非常依赖记录的结构。

```python
from collections import namedtuple

Stock = namedtuple('Stock', ['name', 'Chinese', 'Math'])
def compute_cost(records):
    total = 0.0
    for rec in records:
        s = Stock(*rec)
        total += s.Chinese + s.Math
    return total
```

```python
user =(
    ('Alice', 90, 94),
    ('Bob', 80, 80),
    ('Clur', 76, 86),
)
print(compute_cost(user))
```

命名元组另一个用途就是替代字典，因为字典存储需要更多的内存空间。如果你需要构建一个非常大的包含字典的数据结构，那么使用命名元组会更加高效。但是需要注意的是，不像字典那样，一个命名元组是不可更改的。如果定义一个需要更新很多实例属性的高效数据结构，那么命名元组并不是最佳选择。这时候应该考虑定义一个包含 `__slots__` 方法的类。

***

## `defaultdict`

```python
collections.defaultdict([default_factory[, …]])
```

在 `dict` 基础上，添加了 `default_factory`，使得 `key` 不存在时会自动生成相应类型的 `value`。`default_factory` 参数可以指定成 `list`、`set`、`int` 等可调用对象，也可以是个函数。`defaultdict` 是 `dict` 子类，重载了 `__missing__`。

```python
test = ['a', 'b', 'c', 'a', 'b', 'a']
```

对 `test` 中元素进行数量统计时，常用的方法：

```python
c_dict = {}
for i in test:
    if i not in c_dict:
        c_dict[i] = 1
    else:
        c_dict[i] += 1
```

改进：

```python
c_dict = {}
for i in test:
    c_dict.setdefault(i, 0)
    c_dict[i] += 1
print(c_dict)
```

使用 `defaultdict`：

```python
c_dict = defaultdict(int)
for i in test:
    c_dict[i] += 1
print(c_dict)
```

使用函数，生成嵌套 `dict`：

```python
from collections import defaultdict

def nested_dict():
    return {
        'name':'',
        'city':'bj'
    }

c_dict = defaultdict(nested_dict)

print(c_dict['group1'])
```

```python
import json, collections

tree = lambda: collections.defaultdict(tree)
some_dict = tree()
some_dict['colours']['favourite'] = "yellow"

print(json.dumps(some_dict))
```

补充实例：

```python
from collections import defaultdict

colours = (
    ('Yasoob', 'Yellow'),
    ('Ali', 'Blue'),
    ('Arham', 'Green'),
    ('Ali', 'Black'),
    ('Yasoob', 'Red'),
    ('Ahmed', 'Silver'),
)

favourite_colours = defaultdict(list)

for name, colour in colours:
    favourite_colours[name].append(colour)

print(favourite_colours)
```

```python
defaultdict(<class 'list'>, {'Yasoob': ['Yellow', 'Red'], 'Ali': ['Blue', 'Black'], 'Arham': ['Green'], 'Ahmed': ['Silver']})
```

***

## `Counter`

```python
collections.Counter([iterable-or-mapping])
```

`Counter`是一个计数器，可以针对某项数据进行计数。 `Counter` 对象可以接受任意的由可哈希(hashable)元素构成的序列对象。在底层实现上，一个 `Counter` 对象就是一个字典，将元素映射到它出现的次数上。`Counter` 是 `dict` 的子类。

```python
test = ['a', 'b', 'c', 'a', 'b', 'a']
```

对上文中， `test` 中元素进行数量统计，使用 `Counter`：

```python
from collections import Counter

c = Counter(test)
print(c)
```

在操作同 `dict` 基础上，又添加了`most_common()`、`elements()`。

```python
from collections import Counter

test = '''A Counter is a dict subclass for counting hashable objects.'''.lower()

c = Counter(test)
print(c)
print(c['a'])
print(c.most_common(5))
print(c.most_common()[:-6:-1])
```

```python
Counter({' ': 9, 's': 6, 'a': 5, 'c': 5, 'o': 4, 't': 4, 'u': 3, 'n': 3, 'e': 3, 'i': 3, 'b': 3, 'r': 2, 'l': 2, 'h': 2, 'd': 1, 'f': 1, 'g': 1, 'j': 1, '.': 1})
5
[(' ', 9), ('s', 6), ('a', 5), ('c', 5), ('o', 4)]
[('.', 1), ('j', 1), ('g', 1), ('f', 1), ('d', 1)]
```

`most_common([n])` 返回一个 TopN 列表。如果 `n` 没有被指定，则返回所有元素。当多个元素计数值相同时，排列是无确定顺序的。

```python
from collections import Counter

c1 = Counter(a=3, b=1, c=0, d=-1)
c2 = Counter(a=2, b=2, c=2)

for i in c1.elements():
    print(i)

print(c1+c2)
print(c1-c2)
print(c2-c1)
print(c1&c2)
print(c1|c2)

c1 += Counter()
print(c1)
```

```bash
a
a
a
b

Counter({'a': 5, 'b': 3, 'c': 2})
Counter({'a': 1})
Counter({'c': 2, 'b': 1, 'd': 1})
Counter({'a': 2, 'b': 1})
Counter({'a': 3, 'b': 2, 'c': 2})

Counter({'a': 3, 'b': 1})
```

`elements()` 返回一个迭代器。元素被重复了多少次，在该迭代器中就包含多少个该元素。元素排列无确定顺序，个数小于1的元素不被包含。

`Counter` 也可以进行数学运算。`c1 += Counter()` 可以移除0和负值。

```python
from collections import Counter

colours = (
    ('Yasoob', 'Yellow'),
    ('Ali', 'Blue'),
    ('Arham', 'Green'),
    ('Ali', 'Black'),
    ('Yasoob', 'Red'),
    ('Ahmed', 'Silver'),
)

favs = Counter(name for name, colour in colours)
print(favs)
```

```bash
Counter({'Yasoob': 2, 'Ali': 2, 'Arham': 1, 'Ahmed': 1})
```

***

## `deque`

```python
collections.deque([iterable[, maxlen]])
```

`deque` 是 double-ended queue 的缩写，即双端队列。

Note:Deques support **thread-safe**, memory efficient appends and pops from either side of the deque with approximately the same O(1) performance in either direction.

除了包含 `list` 的方法外，还新增了`appendleft`、`popleft`、`extendleft` 等方法允许我们高效的在元素的开头来插入/删除元素。

```python
d1 = deque(range(5))
d2 = deque(range(5, 10))

d1.extendleft(d2)
print(d1)
```

值得注意的是，使用 `extendleft` 时，添加在 deque 左侧元素的顺序。

```bash
deque([9, 8, 7, 6, 5, 0, 1, 2, 3, 4])
```

以及，限制列表的大小，当超出设定的限制时，数据会从对队列另一端被挤出去(pop)。如果没有指定 maxlen 参数，那么 deque 可以是任意宽度。

```python
d3.extend(d1)
print(d3)
```

```bash
deque([6, 5, 0, 1, 2, 3, 4], maxlen=7)
```

***

## `OrderedDict`

在 python3 中，`dict` 默认是有序的。python2 中 `dict` 是无序的。无论 `dict` 还是 `OrderedDict`，都只能通过 `key` 访问，无法通过下标访问。

`OrderedDict` 是 `dict` 子类。并在其基础上，增加了 `popitem(last=True)` 和 `move_to_end(key, last=True)` 功能。

```python
In [24]: d = OrderedDict({'a':1,'b':2,'c':3})

In [25]: d.popitem()
Out[25]: ('c', 3)

In [26]: d
Out[26]: OrderedDict([('a', 1), ('b', 2)])

In [27]: d.popitem(last=False)
Out[27]: ('a', 1)

In [28]: d
Out[28]: OrderedDict([('b', 2)])
```

```python
In [29]: d = OrderedDict({'a':1,'b':2,'c':3})

In [30]: d.move_to_end('b')

In [31]: d
Out[31]: OrderedDict([('a', 1), ('c', 3), ('b', 2)])

In [32]: d.move_to_end('b', last=False)

In [33]: d
Out[33]: OrderedDict([('b', 2), ('a', 1), ('c', 3)])
```

***

## `ChainMap`

```python
collections.ChainMap(*maps)
```

`ChainMap` 用来成合并多个字典，但不会改变原对象。但如果 `key` 相同，会造成数据丢失。

```python
In [35]: dict1 = { 'a' : 1, 'b' : 2 }

In [36]: dict2 = { 'b' : 3, 'c' : 4 }

In [37]: chain = ChainMap(dict1, dict2)

In [38]: chain
Out[38]: ChainMap({'a': 1, 'b': 2}, {'b': 3, 'c': 4})

In [39]: chain.maps
Out[39]: [{'a': 1, 'b': 2}, {'b': 3, 'c': 4}]

In [40]: list(chain.keys())
Out[40]: ['b', 'c', 'a']
```

***

参考：

[Huoty's Blog](http://kuanghy.github.io/2017/03/03/python-collections)