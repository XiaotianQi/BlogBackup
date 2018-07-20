装饰器(Decorators)是 Python 的一个重要部分。简单地说：他们是修改其他函数的功能的函数。他们有助于让我们的代码更简短，也更Pythonic。

## 装饰器

装饰器是可调用对象，其参数是被装饰的函数/类。装饰器处理并返回被装饰的函数，或者替换成另一个可调用对象，以某种方式增强函数。

装饰器本质上是一个 Python 函数或类，它可以让其他函数或类在不需要做任何代码修改的前提下增加额外功能，装饰器的返回值也是一个函数/类对象。可以抽离出大量与函数功能本身无关的雷同代码到装饰器中并继续重用。概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。典型的行为：把被装饰的函数替换成新函数，二者接受相同的参数，而且返回被装饰函数本该返回的值，同时还会做些而外的操作。

函数装饰器再导入模块时立即执行，而被装饰的函数只有在调用时运行。

```python
registry = []

def register(func):
    print('running register(%s)' % func)
    registry.append(func)
    return func

@register
def f1():
    print('running f1')

@register
def f2():
    print('running f2')

def f3():
    print('running f3')
```

```python
if __name__ == '__main__':
    print('running main')
    print('registry --> ', registry)
    f1()
    f2()
    f3()
```

```bash
running register(<function f1 at 0x00000207F5C66C80>)
running register(<function f2 at 0x00000207F5C97D90>)
running main
registry -->  [<function f1 at 0x00000207F5C66C80>, <function f2 at 0x00000207F5C97D90>]
running f1
running f2
running f3
```

显而易见，当使用装饰器 `register` 时，`register` 已经执行。`f1`、`f2` 与 `f3`相同，直到被调用才执行。

```python
def decorator(func):
        def wrapper(*args, **kwargs):
            logging.warn("%s is running" % func.__name__)
            return func(*args, **kwargs)
        return wrapper
```

***

```python
import time
import functools

def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        t0 = time.time()
        result = func(*args, **kwargs)
        t1 = time.time() - t0
        name = func.__name__
        arg_lst = []
        if args:
            arg_lst.append(', '.join(repr(arg) for arg in args))
        if kwargs:
            pairs = ['%s=%r' % (k, v) for k, v in sorted(kwargs.items())]
            arg_lst.append(', '.join(pairs))
        arg_str = ', '.join(arg_lst)
        print('[%0.8fs] %s(%s) --> %r' % (t1, name, arg_str, result))
        return result
    return clocked
```

* 使用迭代计算斐波那契数列

```python
from clock import clock

@clock
def fib(n):
    if n < 2:
        return n
    return fib(n-2) + fib(n-1)

if __name__ == '__main__':
    print(fib(6))
```

```bash
[0.00000000s] fib(1) --> 1
[0.00000000s] fib(0) --> 0
[0.00000000s] fib(1) --> 1
[0.00099778s] fib(2) --> 1
[0.00000000s] fib(1) --> 1
[0.00000000s] fib(0) --> 0
[0.00000000s] fib(1) --> 1
[0.00199771s] fib(2) --> 1
[0.00499868s] fib(3) --> 2
[0.00000000s] fib(0) --> 0
[0.00000000s] fib(1) --> 1
[0.00098991s] fib(2) --> 1
[0.00000000s] fib(1) --> 1
[0.00000000s] fib(0) --> 0
[0.00000000s] fib(1) --> 1
[0.00299239s] fib(2) --> 1
[0.00799537s] fib(3) --> 2
[0.01098394s] fib(4) --> 3
[0.00000000s] fib(1) --> 1
[0.00000000s] fib(0) --> 0
[0.00000000s] fib(1) --> 1
[0.00299811s] fib(2) --> 1
[0.00598621s] fib(3) --> 2
[0.00000000s] fib(0) --> 0
[0.00000000s] fib(1) --> 1
[0.00199795s] fib(2) --> 1
[0.00000000s] fib(1) --> 1
[0.00000000s] fib(0) --> 0
[0.00000000s] fib(1) --> 1
[0.00399756s] fib(2) --> 1
[0.00599670s] fib(3) --> 2
[0.00899482s] fib(4) --> 3
[0.01697969s] fib(5) --> 5
[1, 1, 2, 3, 5]
```

* 使用生成器计算斐波那契数列：

```python
from clock import clock

@clock
def fib(n):
    a = b = 1
    for i in range(1, n):
        yield a
        a, b = b, a+b

if __name__ == '__main__':
    for i in fib(6):
        print(i, end=' ')
```

```bash
[0.00000000s] fib(6) --> <generator object fib at 0x0000025A44C82EB8>
1 1 2 3 5
```

***

## 常用装饰器

### `@property`

`@property` 使方法像属性一样调用，并且提供了可读、可写、可删除的操作。

#### 只读

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

#### 可写

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

### @total_ordering