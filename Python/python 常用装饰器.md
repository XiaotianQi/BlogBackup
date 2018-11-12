常用装饰器

***

## `@property`

`@property` 使方法像属性一样调用，并且提供了可读、可写、可删除的操作。

### 只读

```python
class Exam(object):
    def __init__(self, score):
        self._score = score

    @property
    def score(self):
        return self._score
```

仅使用 `@property` 时，`score` 就变成一个只读属性。此时只能用 `test = Exam(60)` 进行赋值。若使用 `test.score = 75` 则会返回 `AttributeError: can't set attribute` 错误。

```python
test = Exam(60)
print(test.score)


输出结果：
60
```

### 可写

`@property` 本身又创建了另一个装饰器 `@score.setter`，负责把一个 `setter` 方法变成属性赋值。此时，`score` 不仅可读，还可以写入。

```python
class Exam(object):
    def __init__(self, score):
        self._score = score

    @property
    def score(self):
        return self._score

    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value
```

```python
test = Exam(60)
print(test.score)
test.score = 75
print(test.score)
test.score = 175
print(test.score)


输出结果：
60
75
ValueError: score must between 0 ~ 100!
```

***

## `@total_ordering`

让类支持比较操作。只需要在类中实现 `__eq__` 方法和以下方法中的任意一个 `__lt__`、`__le__`、`__gt__`、`__ge__`，那么 `@total_ordering` 就能自动实现余下的几种比较运算。

```python
from functools import total_ordering

@total_ordering
class Rectangle(object):
    __slots__ = ('width', 'height')
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def __eq__(self,rect):
        return self.area() == rect.area()

    def __lt__(self,rect):
        return self.area() < rect.area()
```

```python
rect1 = Rectangle(1.0,2.0)
rect2 = Rectangle(2.0,1.0)
print(rect1 <= rect2)
```

```bash
True
```

***

## `@enum.unique`

专门用于枚举的类装饰器。搜索 `__members__`，收集发现的任何别名。如果有的话，反馈 `ValueError`。

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

***

## `@lru_cache`

***

## `@classmethod`

***

## `@staticmethod`