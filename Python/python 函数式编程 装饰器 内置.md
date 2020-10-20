内置装饰器

***

## `@functools.wraps(wrapped[, assigned][, updated])`

将装饰过的函数的特殊属性保留。函数是有几个特殊属性比如函数名，在被装饰后，函数名func会变成包装函数的名字wrapper，如果你希望使用反射，可能会导致意外的结果。这个装饰器可以解决这个问题。

```python
import time
import functools
 
def timeit(func):
    @functools.wraps(func)
    def wrapper():
        start = time.clock()
        func()
        end =time.clock()
        print 'used:', end - start
    return wrapper
```



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

## `@classmethod`

类方法

***

## `@staticmethod`

静态方法

***

## `@contextmanager`

上下文管理器

***

## `@list`

`call`这个装饰器会把传入的参数送给目标函数然后直接执行。

```python
def call(*args, **kwargs):
    def call_fn(fn):
        return fn(*args, **kwargs)
    return call_fn
```

有一个这样一个生成器函数:

```python
def table(n):
    for i in range(n):
        yield i
```

直接将其转换成列表。

```python
@list
@call(5)
def table(n):
    for i in range(n):
        yield i

print(table)	# [0, 1, 2, 3, 4]
```

***

## `@functools.lru_cache`

```python
@functools.lru_cache(maxsize=None, typed=False)
```

```python
from functools import lru_cache

@lru_cache(None)
def add(x, y):
    print("calculating: %s + %s" % (x, y))
    return x + y

print(add(1, 2))
print(add(1, 2))
print(add(2, 3))
```

```python
calculating: 1 + 2
3
3
calculating: 2 + 3
5
```

可以看到第二次调用并没有真正的执行函数体，而是直接返回缓存里的结果。