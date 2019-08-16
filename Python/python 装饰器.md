装饰器(Decorators)是 Python 的一个重要部分。装饰器的理念是对原函数、对象的加强，相当于重新封装，所以一般装饰器函数都被命名为wrapper()，意义在于包装。在被装饰函数自身代码不变的情况下，增添一些**具有普适性的功能，为已经存在的函数或对象添加额外的功能。**最大程度的减少改动，尽可能做到以不变应万变。装饰器有很多种，有函数的装饰器，也有类的装饰器。装饰器在很多语言中的名字也不尽相同，它体现的是设计模式中的装饰模式，强调的是开放封闭原则。

装饰器是可调用对象，其参数是被装饰的函数/类。装饰器处理并返回被装饰的函数，或者替换成另一个可调用对象，以某种方式增强函数。

## 函数装饰器

一个简单的装饰器如下：

```python
def debug(func):
    print('OUTER')
    def wrapper():
        print('INNER')
        return func()
    return wrapper

@debug
def say_hello():
    print("hello!")
```

函数`say_hello()`使用装饰器魔法糖时，与以下代码功能相同：

```python
def debug(func):
    print('OUTER')
    def wrapper():
        print('INNER')
        return func()
    return wrapper

def say_hello():
    print("hello!")

say_hello = debug(say_hello)
```

此时，即使没有调用`say_hello()`函数，仍会输出 `OUTER`。

只有，调用`say_hello()`函数后，才会输出`INNER`。

***

### 装饰器调用顺序

函数装饰器再导入模块时立即执行，而被装饰的函数只有在调用时运行。

#### 装饰器函数未封装时

```python
registry = []

def register(func):
    print('running register(%s)' % func.__name__)
    registry.append(func.__name__)
    result = func()
    return result

@register
def f1():
    print('running f1')

@register
def f2():
    print('running f2')

def f3():
    print('running f3')


if __name__ == '__main__':
    print('running main')
    print('registry --> ', registry)
```

```python
if __name__ == '__main__':
    print('running main')
    print('registry --> ', registry)
```

```bash
running register(f1)
running register(f2)
running main
registry -->  ['f1', 'f2']
```

显而易见，当使用装饰器 `register` 时，`register`已经执行。

#### 装饰器函数封装时

```python
registry = []

def register(func):
    def inner():
        print('running register(%s)' % func.__name__)
        registry.append(func.__name__)
        result = func()
        return result
    return inner

@register
def f1():
    print('running f1')

@register
def f2():
    print('running f2')
```

```python
if __name__ == '__main__':
    print('running main')
    print('registry --> ', registry)
    f1()
    f2()
    print('registry --> ', registry)
```

```text
running main
registry -->  []
running register(f1)
running f1
running register(f2)
running f2
registry -->  ['f1', 'f2']
```

封装之后，只有调用`f1`、`f2`时，才会执行。

这是因为，装饰器函数`registry`返回的是`inner`函数名。`registry`函数仍然执行了，但是没执行`inner`函数。不过，这也有个缺点：

```python
if __name__ == '__main__':
    print(f1.__name__)		# 输出inner
```

这是装饰器本身规则导致的。

当 python 解释器读到`@registry`的时候，别到到这是一个装饰器，开始运行`registry`函数。开始执行装饰器的语法规则，规则是：被装饰的函数的名字会被当作参数传递给装饰函数。装饰函数执行它自己内部的代码后，会将它的返回值赋值给被装饰的函数。

`registry`函数return的是`inner`这个函数名，而不是`inner()`这样被调用后的返回值。这个函数名会被赋值给f1这个被装饰的函数，也就是`f1 = inner`。

此时`f1`函数被新的函数`inner`覆盖了（实际上是`f1`这个函数名更改成指向`inner`这个函数名指向的函数体内存地址，`f1`不再指向它原来的函数体的内存地址），再往后调用`f1`的时候将执行`inner`函数内的代码，而不是先前的函数体。那么先前的函数体去哪了？还记得我们将f1当做参数传递给`func`这个形参么？`func`这个变量保存了老的函数在内存中的地址，通过它就可以执行老的函数体，在`inner`函数里看到`result = func()`这句代码，它就是这么干的！

因此，原来的`f1`函数被当做参数传递给了`func`，而`f1`这个函数名之后会指向`inner`函数。当调用`f1()`时，执行的就不再是旧的`f1`函数的代码，而是`inner`函数的代码。这也体现了，解释器的机制。

使用`@functools.wraps(func)`，就可以保持被装饰函数的函数名不会发生改变：

```pytho
def register(func):
	@functools.wraps(func)
    def inner():
        print('running register(%s)' % func.__name__)
        registry.append(func.__name__)
        result = func()
        return result
    return inner
```

#### NOTE：位置错误的代码

```python
def html_tags(tag_name):
    print('begin outer function.')
    def inner(func):
        print("begin inner function.")
        def wrapper(*args, **kwargs):
            content = func(*args, **kwargs)
            print("<{tag}>{content}</{tag}>".format(tag=tag_name, content=content))
        print('end inner function.')
        return wrapper
    print('end outer function')
    return inner

@html_tags('b')
def hello(name='Toby'):
    return('Hello {}!'.format(name))
```

因为装饰器的规定，及时没有调用`hello()`函数，就会出现以下输出：

```bash
begin outer function.
end outer function
begin inner function.
end inner function.
```

在装饰器函数之外添加逻辑功能，容易让装饰器就不受控制。当对函数调用时：

```python
@html_tags('b')
def hello(name='Toby'):
    return('Hello {}!'.format(name))

print('-----')
hello()
hello('abc')
```

会出现以下结果：

```python
begin outer function.
end outer function
begin inner function.
end inner function.
-----
<b>Hello Toby!</b>
<b>Hello abc!</b>
```

***

### 给被装饰函数传入参数

当需要给被装饰参数传入参数时：

```python
def register(func):
    def inner(*args, **kwargs):
        print('running register(%s)' % func.__name__)
        registry.append(func.__name__)
        result = func(*args, **kwargs)
        return result
    return inner
```

***

### 使用多个装饰器

```python
def outer1(func):
    def inner(*args, **kwargs):
        print("认证成功！")
        result = func(*args, **kwargs)
        print("日志添加成功")
        return result
    return inner


def outer2(func):
    def inner(*args, **kwargs):
        print("一条欢迎信息。。。")
        result = func(*args, **kwargs)
        print("一条欢送信息。。。")
        return result
    return inner


@outer1
@outer2
def f1(name,age):
    print('f1')
    
f1()
```

```text
认证成功！
一条欢迎信息。。。
f1
一条欢送信息。。。
日志添加成功
```

### 给装饰器函数传入参数

```python
# 认证函数
def auth(request, kargs):
    print("{}-{}认证成功！".format(request, kargs))
# 日志函数
def log(request, kargs):
    print("{}-{}日志添加成功".format(request, kargs))
# 装饰器函数。接收两个参数，这两个参数应该是某个函数的名字。
def wrapper_f(auth_func, log_func):
    # 第一层封装，f1函数实际上被传递给了main_fuc这个参数
    def outer(main_func):
        # 第二层封装，auth和log函数的参数值被传递到了这里
        def wrapper(*args, **kwargs):
            # 下面代码的判断逻辑不重要，重要的是参数的引用和返回值
            before_result = auth(*args, **kwargs)
            if before_result != None:
                return before_result

            main_result = main_func(*args, **kwargs)
            if main_result != None :
                return main_result

            after_result = log(*args, **kwargs)
            if after_result != None:
                return after_result

        return wrapper
    return outer
# 注意了，这里的装饰器函数有参数哦，它的意思是先执行wrapper_f函数
# 然后将wrapper_f函数的返回值作为装饰器函数的名字返回到这里，所以，
# 其实这里，wrapper_f(auth,log) = outer , @wrapper_f(auth,log) =  @outer
@wrapper_f(auth, log)
def f1(name, age):
    print('f1:{}-{}'.format(name, age))

# 调用方法
f1('a', 1)
print(f1.__name__)
```

```text
a-1认证成功！
a-1
a-1日志添加成功
wrapper
```

***

### 示例

#### `call()`

这个装饰器会把传入的参数送给目标函数然后直接执行。

```python
def call(*args, **kwargs):
    def call_fn(fn):
        return fn(*args, **kwargs)
    return call_fn
```

```python
@call(5)
def table(n):
    value = []
    for i in range(n):
        value.append(i*i)
    return value

print(len(table), table[3])	# 5 9
```

`@call()`装饰器适用于任何函数，传入的参数会被直接使用然后结果赋值给同名函数。这样避免了重新定义一个变量来存储结果。

#### 简单的日志

每次函数调用，都会在屏幕中打印日志信息。

```python
def logging(level):
    def wrapper(func):
        def inner_wrapper(*args, **kwargs):
            print("[{level}]: enter function {func}()".format(
                level=level,
                func=func.__name__))
            return func(*args, **kwargs)
        return inner_wrapper
    return wrapper

@logging(level='INFO')
def say(something):
    print("say {}!".format(something))

@logging(level='DEBUG')
def do(something):
    print("do {}...".format(something))

if __name__ == '__main__':
    say('hello')
    do("my work")
```

```bash
[INFO]: enter function say()
say hello!
[DEBUG]: enter function do()
do my work...
```

装饰器本质上是一个 Python 函数或类，它可以让其他函数或类在不需要做任何代码修改的前提下增加额外功能，装饰器的返回值也是一个函数/类对象。可以抽离出大量与函数功能本身无关的雷同代码到装饰器中并继续重用。概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。典型的行为：把被装饰的函数替换成新函数，二者接受相同的参数，而且返回被装饰函数本该返回的值，同时还会做些而外的操作。

***

#### 带参数的日志装饰器

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

#### clock 装饰器

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

装饰器函数其实是这样一个接口约束，它必须接受一个callable对象作为参数，然后返回一个callable对象。在Python中一般callable对象都是函数，但也有例外。只要某个对象重载了`__call__()`方法，那么这个对象就是callable的。

相比函数装饰器，类装饰器具有灵活度大、高内聚、封装性等优点。使用类装饰器主要依靠类的`__call__`方法，当使用 @ 形式将装饰器附加到函数上时，就会调用此方法。

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

### 带参数的类装饰器

```python
class logging(object):
    def __init__(self, level='INFO'):
        self.level = level
        
    def __call__(self, func): # 接受函数
        def wrapper(*args, **kwargs):
            print("[{level}]: enter function {func}()".format(
                level=self.level,
                func=func.__name__))
            func(*args, **kwargs)
        return wrapper  #返回函数

@logging(level='INFO')
def say(something):
    print("say {}!".format(something))
```

### ptest

[ptest](https://pypi.python.org/pypi/ptest)中的`@TestClass()`用于声明一个测试类，其源代码大致如此。

```python
def TestClass(enabled=True, run_mode="singleline"):
    def tracer(cls):
        cls.__pd_type__ ='test'
        cls.__enabled__ = enabled
        cls.__run_mode__ = run_mode.lower()
        return cls
    return tracer

@TestClass()
class TestCases(object):
    pass

print(TestCases.__dict__)
# {'__module__': '__main__', '__dict__': <attribute '__dict__' of 'TestCases' objects>, '__weakref__': <attribute '__weakref__' of 'TestCases' objects>, '__doc__': None, '__pd_type__': 'test', '__enabled__': True, '__run_mode__': 'singleline'}
```

当装饰器在被使用时，`TestClass()`函数会马上被执行并返回一个装饰器函数，这个函数是一个闭包函数，保存了`enabled`和`run_mode`两个变量。另外它还接受一个类作为参数，并使用之前保存的变量为这个类添加属性，最后返回。所以经过`@TestClass()`装饰过的类都会带上`__enabled__`、`__pd_type__`以及`__run_mode__`的属性。

***

参考：

[理解 Python 装饰器](https://foofish.net/python-decorator.html)，刘志军

[装饰器](http://www.liujiangblog.com/course/python/39)，刘江

[Python装饰器的另类用法](https://segmentfault.com/a/1190000007408026)，betacat

