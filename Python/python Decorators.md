装饰器(Decorators)是 Python 的一个重要部分。简单地说：他们是修改其他函数的功能的函数。他们有助于让我们的代码更简短，也更Pythonic。

装饰器是可调用对象，其参数是被装饰的函数/类。装饰器处理并返回被装饰的函数，或者替换成另一个可调用对象，以某种方式增强函数。

## 函数装饰器

### 简单的日志

每次函数调用，都会在屏幕中打印日志信息。

```python
import functools

def logit(func):
    @functools.wraps(func)
    def wrapped_function(*args, **kwargs):
        logging.warn("%s was called" % func.__name__)
        return func(*args, **kwargs)
    return wrapped_function
```

```python
@decorator
def foo(x):
    print('its foo')
    return x+x

print(foo(2))
```

```bash
WARNING:root:foo was called
its foo
4
```

装饰器本质上是一个 Python 函数或类，它可以让其他函数或类在不需要做任何代码修改的前提下增加额外功能，装饰器的返回值也是一个函数/类对象。可以抽离出大量与函数功能本身无关的雷同代码到装饰器中并继续重用。概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。典型的行为：把被装饰的函数替换成新函数，二者接受相同的参数，而且返回被装饰函数本该返回的值，同时还会做些而外的操作。

***

### 带参数的日志装饰器

插入参数 `logfile` 是日志信息输出的文件。被装饰函数调用成功和失败，都会打印不同的日志信息。

```python
import functools, logging

def logit(logfile='out.log'):
    def logging_decorator(func):
        logging.basicConfig(level=logging.DEBUG,
                    format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
                    datefmt='%a, %d %b %Y %H:%M:%S',
                    filename=logfile,
                    filemode='w')
        console = logging.StreamHandler()
        console.setLevel(logging.INFO)
        formatter = logging.Formatter('%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s')
        console.setFormatter(formatter)
        @functools.wraps(func)
        def wrapped_function(*args, **kwargs):
            try:
                result = func(*args, **kwargs)
            except:
                logging.getLogger('').addHandler(console)
                logging.warn("%s was crashed" % func.__name__)
            else:
                logging.getLogger('').addHandler(console)
                logging.warn("%s was called" % func.__name__)
            return result
        return wrapped_function
    return logging_decorator
```

关键是 `logit` 是装饰器工厂函数，要返回 `logging_decorator`。

```python
@logit(logfile='out.log')
def foo(x):
    print('its foo')
    return x+x

print(foo(2))
```

若不使用语法糖 `@logit(logfile='out.log')`，则：`logit()(foo)` 或者 `logit(logfile='out_1.log')(foo)`。

```bash
its foo
2018-07-21 10:55:50,861 1.py[line:24] WARNING foo was called
4
```

***

### 装饰器调用顺序

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

***

### clock 装饰器

计算函数调用使用的时间的装饰器。

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

## 类装饰器

相比函数装饰器，类装饰器具有灵活度大、高内聚、封装性等优点。使用类装饰器主要依靠类的__call__方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。

```python
class logit(object):
    def __init__(self, func):
        self._func = func

    def __call__(self, *args, **kwargs):
        logging.warn("%s started" % self._func.__name__)
        self._func()
        logging.warn("%s stopped" % self._func.__name__)
```

```python
@logit
def bar():
    print ('bar')

bar()
```

```bash
WARNING:root:bar started
bar
WARNING:root:bar stopped
```

***

参考：

[刘志军](https://foofish.net/python-decorator.html)