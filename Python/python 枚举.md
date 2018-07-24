包含 `enum` 模块和 `enumerate` 内置函数。

## `enum` 模块

***

### 构造枚举类

`enum` 模块定义了一个具备可迭代性和可比较性的枚举类型。

`MonthEnum` 类是一个枚举。属性 `MonthEnum.Jan`、`MonthEnum.Feb` 等是枚举成员，并且是函数常量。每个成员的数据类型就是它所属的枚举。枚举成员具有名称和值，`MonthEnum.Jan` 的名称为 `Jan`，`MonthEnum.Jan` 的值为 1。

枚举类得成员值可以是任何值，比如 `int`、`str` 等。

两个枚举成员拥有相同的名字是不合法的。但是，两个枚举成员允许拥有相同的值。

构造枚举类有多种种方式。

```python
from enum import Enum

class MonthEnum(Enum):
    Jan = 1
    Feb = 2
    Mar = 3
    Apr = 4
    May = 5
    Jun = 6
    Jul = 7
    Aug = 8
    Sep = 9
    Oct = 10
    Nov = 11
    Dec = 12

    jan = 1
```

使用装饰器 `@unique` 确保每个值在枚举中仅使用一次，重复的成员会触发一个 `ValueError` 异常。

```python
from enum import Enum, unique

@unique
class MonthEnum(Enum):
    Jan = 1
    Feb = 2
    Mar = 3
    Apr = 4
    May = 5
    Jun = 6
    Jul = 7
    Aug = 8
    Sep = 9
    Oct = 10
    Nov = 11
    Dec = 12
```

动态创建枚举。`value` 参数是枚举的名称，用于构建枚举成员的表示。`names` 参数列出了枚举所包含的成员。当只传递一个字符串的时候，它会被自动以空格和逗号进行分割，并且将结果序列作为枚举成员的名称，这些字段会被自动赋值为从 `1` 开始自增的数字。

```python
MonthEnum = Enum(value='Month',
    names=('Jan Feb Mar Apr May Jun '
           'Jul Aug Sep Oct Nov Dec'))
```

为了更加方便的控制枚举成员的值，`names` 字符串可以被替换成元组组成的序列或者是由名称和值组成的字典。

```python
MonthEnum = Enum(
    value='Month',
    names={
        'Jan':1,
        'Feb':2,
        'Mar':3,
        'Apr':4,
        'May':5,
        'Jun':6,
        'Jul':7,
        'Aug':8,
        'Sep':9,
        'Oct':10,
        'Nov':11,
        'Dec':12})
```

```python
MonthEnum = Enum(
    value='Month',
    names=(
        ('Jan', 1),
        ('Feb', 2),
        ('Mar', 3),
        ('Apr', 4),
        ('May', 5),
        ('Jun', 6),
        ('Jul', 7),
        ('Aug', 8),
        ('Sep', 9),
        ('Oct', 10),
        ('Nov', 11),
        ('Dec', 12)
    )
)
```

可简化为：

```python
MonthEnum = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))
```

以上只列举了当枚举成员值是 `int` 类型，枚举成员值可以是任何类型的对象，例如嵌套的对象。

***

### 枚举取值

* 通过成员的名称来获取成员，`MonthEnum['Jan']`；
* 通过成员值来获取成员，`MonthEnum(1)`；
* 通过成员，来获取它的名称和值，`.name` 和 `.value`。

```python
In [1]: from enum import Enum

In [2]: MonthEnum = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))

In [3]: type(MonthEnum.Jan)
Out[3]: <enum 'Month'>

In [4]: MonthEnum.Jan.name
Out[4]: 'Jan'

In [5]: MonthEnum.Jan.value
Out[5]: 1

In [6]: MonthEnum['Jan']
Out[6]: <Month.Jan: 1>

In [7]: MonthEnum(1)
Out[7]: <Month.Jan: 1>

In [8]: MonthEnum['Jan'].name
Out[8]: 'Jan'

In [9]: MonthEnum['Jan'].value
Out[9]: 1

In [10]: MonthEnum(1).name
Out[10]: 'Jan'

In [11]: MonthEnum(1).value
Out[11]: 1
```

如果两个成员 `Jan` 和 `jan` 有相同的值，并且 `Jan` 先定义，那么 `jan` 是 `Jan` 的一个别名。枚举成员中第一个关联到成员值的名称是规范名称。根据值 `1` 查询会返回 `Jan`。根据名字查询 `jan` 也会返回 `Jan`。

```python
In [2]: MonthEnum(1)
Out[2]: <MonthEnum.Jan: 1>

In [3]: MonthEnum['Jan']
Out[3]: <MonthEnum.Jan: 1>

In [4]: MonthEnum['jan']
Out[4]: <MonthEnum.Jan: 1>
```

***

### 遍历枚举成员

枚举支持迭代器，可以遍历枚举成员。如果枚举有值重复的成员，循环遍历枚举时只获取值重复成员的第一个成员。

```python
for i in MonthEnum:
    print('{} = {}'.format(i.name, i.value))
```

```bash
Jan = 1
Feb = 2
Mar = 3
Apr = 4
May = 5
Jun = 6
Jul = 7
Aug = 8
Sep = 9
Oct = 10
Nov = 11
Dec = 12
```

如果想把值重复的成员也遍历出来，要用枚举的一个特殊属性`__members__`。`__members__` 返回一个 dict。

```python
for name, member in MonthEnum.__members__.items():
    print('{} = {} - {}'.format(name, member.value, member))
```

```bash
Jan = 1 - MonthEnum.Jan
Feb = 2 - MonthEnum.Feb
Mar = 3 - MonthEnum.Mar
Apr = 4 - MonthEnum.Apr
May = 5 - MonthEnum.May
Jun = 6 - MonthEnum.Jun
Jul = 7 - MonthEnum.Jul
Aug = 8 - MonthEnum.Aug
Sep = 9 - MonthEnum.Sep
Oct = 10 - MonthEnum.Oct
Nov = 11 - MonthEnum.Nov
Dec = 12 - MonthEnum.Dec
jan = 1 - MonthEnum.Jan
```

***

### 枚举比较

枚举成员可以进行 `is`、`==` 比较，但是无法比较大小。

```python
In [13]: MonthEnum.Jan is MonthEnum.Feb
Out[13]: False

In [14]: MonthEnum.Jan == MonthEnum.Feb
Out[14]: False

In [15]: MonthEnum.Jan < MonthEnum.Feb
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-15-21d8f1f92df6> in <module>()
----> 1 MonthEnum.Jan < MonthEnum.Feb

TypeError: '<' not supported between instances of 'Month' and 'Month'

```

***

## `enumerate` 内置函数

```python
enumerate(iterable, [start=0])
```

`enumerate()` 函数用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。

返回 `enumerate` 对象。

```python
In [24]: lst = ['a', 'b', 'c', 'd', 'e']

In [25]: counter_lst = list(enumerate(lst, 1))

In [26]: counter_lst
Out[26]: [(1, 'a'), (2, 'b'), (3, 'c'), (4, 'd'), (5, 'e')]

In [27]: for i, j in counter_lst:
    ...:     print(i, j)
    ...:
1 a
2 b
3 c
4 d
5 e
```

***

参考：

[Catharina博客](https://blog.csdn.net/xionghuixionghui/article/details/66476601)