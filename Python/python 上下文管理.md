## Try and Except

我们经常使用 Try and Except，进行异常处理。检测 try 语句块中的错误，从而让 except 语句捕获异常信息并处理。

其过程可以简化为：

```python
try:
    测试代码
   	...
except:
    发生异常，执行这块代码
    ...
else:
    如果没有异常执行这块代码
    ...
finally:
    总会执行
```

例如：

```python
try:
    print('try block')
    raise KeyError
except KeyError as e:
    print('except block 1')
except SyntaxError as e:
    print('except block 2')
else:
    print('else block')
finally:
    print('finally block')
```

```text
try block
except block 1
finally block
```

整个 Try and Except 的工作原理是，当开始一个`try`语句后，python 就在当前程序的上下文中作标记，当异常出现时就可以回到这里，首先执行 `try` 中的代码块，接下来发生的状况，依赖于执行时是否出现异常：

* 出现异常，python 就跳回到 `try` 并执行第一个与该异常匹配的 `except` 代码块，异常处理完毕，最后执行 `finally` 中的代码块。
* 出现异常，但没有与匹配的 `except`，异常将被递交到上层的 `try`（如果存在），或者到程序的最上层，抛出默认的异常信息。`finally` 不会执行。
* 没有异常，执行 `else` 语句，最后执行 `finally` 语句。

当加入 `return` 语句后：

```python
def func():
    try:
        print('try block')
        # raise KeyError 
        return 'try'
    except KeyError as e:
        print('except block 1')
        return 'except 1'
    except SyntaxError as e:
        print('except block 2')
        return 'except 2'
    else:
        print('else block')
        return 'else'
    finally:
        print('finally block')
        return 'finally'
```

```text
try block
finally block
finally
```

如果有 `finally` 语句，且当中存在 `return` 时，那么无论是否发生异常，都将会得到 `finally` 语句中 `return` 的信息。

当 `finally` 语句不存在 `return`时：

```python
def func():
    try:
        print('try block')
        # raise KeyError 
        return 'try'
    except KeyError as e:
        print('except block 1')
        return 'except 1'
    except SyntaxError as e:
        print('except block 2')
        return 'except 2'
    else:
        print('else block')
        return 'else'
    finally:
        print('finally block')
```

```text
try block
except block 1
finally block
except 1
```

此时，输出结果与预想中相同。

值得注意的是：当 `try` 语句存在 `return`时，那么无论是否发生异常，`else`语句均不会执行。因为，在函数中，`return` 标志着函数运行结束。

```python
def func():
    try:
        print('try block')
        # raise KeyError 
        # return 'try'
    except KeyError as e:
        print('except block 1')
        return 'except 1'
    except SyntaxError as e:
        print('except block 2')
        return 'except 2'
    else:
        print('else block')
        return 'else'
    finally:
        print('finally block')
        return 'finally'
```

```text
try block
else block
finally block
finally
```

但此时，`else`语句虽然会执行，但是返回的仍然是`finally`中的信息。

所以，如果想要获取 `else` 中的 `return`信息，那么 `try`、`finally` 中均不能使用 `return`。

除此之外，使用 Try and Except 实现文件的读取与关闭。

```python
try:
    f = open('xx.txt', w)
finally:
    f.close()
```

如果使用 with 语句操作文件对象，那么更为简单：

```python
with open('xx.txt', w) as f:
    do something
```

不过，有了上下文管理器，with 语句才能工作。

***

## 上下文管理器

> A context manager is an object that defines the runtime context to be established when executing a `with`statement. The context manager handles the entry into, and the exit from, the desired runtime context for the execution of the block of code.  Context managers are normally invoked using the `with` statement, but can also be used by directly invoking their methods.
>
> Typical uses of context managers include saving and restoring various kinds of global state, locking and unlocking resources, closing opened files, etc.
>
> Python’s `with`statement supports the concept of a runtime context defined by a context manager.  This is implemented using a pair of methods that allow user-defined classes to define a runtime context that is entered before the statement body is executed and exited when the statement ends:
>
> - `contextmanager.__enter__(self)`
>
>   Enter the runtime context related to this object. The `with` statement will bind this method’s return value to the target(s) specified in the `as`clause of the statement, if any. 
>
> - `contextmanager.__exit__(self, exc_type, exc_value, traceback)`
>
>   Exit the runtime context related to this object. The parameters describe the exception that caused the context to be exited. If the context was exited without an exception, all three arguments will be `None`. If an exception is supplied, and the method wishes to suppress the exception (i.e., prevent it from being propagated), it should return a true value. Otherwise, the exception will be processed normally upon exit from this method. 
>
>   Note that `__exit__()` methods should not reraise the passed-in exception; this is the caller’s responsibility. 
>

上下文管理器的目的很明确: to make working with resources and creating managed contexts easier.

相关的概念如下:

以下内容引用自：[浅谈 Python 的 with 语句](https://www.ibm.com/developerworks/cn/opensource/os-cn-pythonwith/index.html)，仅在原文基础上修饰文字表达。

**上下文管理协议（Context Management Protocol）**：支持该协议的对象需要实现 `__enter__()` 和 `__exit__()`两个方法。

**上下文管理器（Context Manager）**：支持上下文管理协议的对象。上下文管理器定义了执行 `with` 语句时的上下文（ `__enter__()` 和 `__exit__()`方法），负责 `with` 语句上下文的进入与退出。

**运行时上下文（runtime context）**：由上下文管理器创建，通过`__enter__()` 和 `__exit__()` 方法实现，`__enter__()` 方法是在语句体执行之前，`__exit__()` 在语句体执行完毕后退出。`with` 语句支持运行时上下文这一概念。

**上下文表达式（Context Expression）**：返回一个上下文管理器对象。

**语句体（with-body）**：with 语句包裹起来的代码块，在执行语句体之前会调用上下文管理器的 `__enter__()` 方法，执行完语句体之后会执行 `__exit__()` 方法。

***

## `with` 语句

> The `with` statement is used to wrap the execution of a block with methods defined by a context manager.This allows common `try`…`except`…`finally` usage patterns to be encapsulated for convenient reuse.
>
> PEP 343 adds a new statement "with" to the Python language to make it possible to factor out standard uses of `try/finally` statements.
>
> In this PEP, context managers provide `__enter__()` and `__exit__()` methods that are invoked on entry to and exit from the body of the with statement.

with 语句适用于对资源进行访问的场合，确保不管使用过程中是否发生异常都会执行必要的“清理”操作，释放资源，比如文件使用后自动关闭、线程中锁的自动获取和释放等。

with语句即“上下文管理器”，在程序中用来表示代码执行过程中所处的前后环境 上下文管理器：含有`__enter__`和`__exit__`方法的对象就是上下文管理器。

* `__enter__()`：在执行语句之前，首先执行该方法，通常返回一个实例对象，如果with语句有as目标，则将对象赋值给as目标。
* `__exit__()`：执行语句结束后，自动调用`__exit__()`方法，用户释放资源，若此方法返回布尔值True，程序会忽略异常。

###   1. `with` 语句的语法格式

> ```python
> with context_expression [as target(s)]:
>    with-body
> ```

`context_expression` 返回一个上下文管理器对象。如果指定了 `as`  子句的话，会将上下文管理器的 `__enter__()` 方法的返回值赋值给 `target(s)`。

Python 对一些内建对象进行改进，增加对上下文管理器的支持，可以用于 `with` 语句。比如自动关闭文件、线程锁的自动获取和释放等。

### 2.文件操作

####   使用 `with` 语句操作文件对象

假设要对一个文件进行操作，使用 `with` 语句可以有如下代码：

> ```python
> with open(r'somefileName') as somefile:
>     for line in somefile:
>         print(line)
>         # ...more code
> ```

使用 `with` 语句后，无论处理文件过程中是否发生异常，都能保证 `with` 语句执行完毕后，关闭打开的文件。

####   `try/finally` 方式操作文件对象

如果使用传统的 `try/finally` ，则要使用类似如下代码：

> ```python
> somefile = open(r'somefileName')
> try:
>     for line in somefile:
>         print(line)
>         # ...more code
> finally:
>     somefile.close()
> ```

对比而言，使用 `with` 语句可以减少编码量。已经加入对上下文管理协议支持的还有模块 threading、decimal 等。

###   3. `with` 语句执行过程

> ```python
> context_manager = context_expression
> exit = type(context_manager).__exit__  
> value = type(context_manager).__enter__(context_manager)
> exc = True   # True 表示正常执行，即便有异常也忽略；False 表示重新抛出异常，需要对异常进行处理
> try:
>     try:
>         target = value  # 如果使用了 as 子句
>         with-body     # 执行 with-body
>     except:
>         # 执行过程中有异常发生
>         exc = False
>         # 如果 __exit__ 返回 True，则异常被忽略；如果返回 False，则重新抛出异常
>         # 由外层代码对异常进行处理
>         if not exit(context_manager, *sys.exc_info()):
>             raise
> finally:
>     # 正常退出，或者通过 statement-body 中的 break/continue/return 语句退出
>     # 或者忽略异常退出
>     if exc:
>         exit(context_manager, None, None, None) 
>     # 缺省返回 None，None 在布尔上下文中看做是 False
> ```

1. 执行 context_expression，生成上下文管理器 context_manager
2. 调用上下文管理器的 `__enter__()` 方法；如果使用了 `as` 子句，则将 `__enter__()` 方法的返回值赋值给 `as` 子句中的 `target(s) `
3. 执行语句体 with-body 
4. 无论执行过程中是否发生了异常，都将会执行上下文管理器的 `__exit__()` 方法。`__exit__()`  方法负责执行“清理”工作，如释放资源等。如果执行过程中没有出现异常，或者语句体中执行了语句 `break`/`continue`/`return`，则以  `None` 作为参数调用 `__exit__(None, None, None)`；如果执行过程中出现异常，则使用 sys.exc_info  得到的异常信息为参数调用 `__exit__(exc_type, exc_value, exc_traceback)`
5. 出现异常时，如果 `__exit__(type, value, traceback)` 返回 `False`，则会抛出异常，让 `with`  之外的语句逻辑来处理异常，这也是通用做法；如果返回 `True`，则忽略异常，不再对异常进行处理                                                                                                                                                                                      

***

## 自定义上下文管理器

可以自定义支持上下文管理协议的类。自定义的上下文管理器需要实现上下文管理协议所需的 `__enter__()` 和 `__exit__()` 两个方法：

- `context_manager.__enter__(self)`：进入上下文管理器，在语句体执行前调用。如果 `with` 语句指定了 `as` 子句，那么该方法的返回值赋值给 `as` 子句中的 `target`
- `context_manager.__exit__(self, exc_type, exc_value, exc_traceback)` ：退出上下文管理器，返回一个布尔值，表示是否对发生的异常进行处理。
如果因异常而退出，则`type`, `value`, `traceback`参数传入 `__exit__()`。如果退出时没有发生异常，则 3 个参数都为 `None`。

> If the suite was exited due to an exception, and the return value from the `__exit__()`method was **false**, the exception is reraised.  If the return value was **true**, the exception is suppressed, and execution continues with the statement following the `with` statement.
>
> If the suite was exited for any reason other than an exception, the return value from `__exit__()` is ignored, and execution proceeds at the normal location for the kind of exit that was taken.     
>
> Note that `__exit__()` methods should not reraise the passed-in exception;this is the caller’s responsibility.

> 注意，上下文管理器必须同时提供  `__enter__()` 和`__exit__()` 方法的定义，缺少任何一个都会导致 AttributeError；`with`  语句会先检查是否提供了`__exit__()` 方法，然后检查是否定义了 `__enter__()` 方法。

> ```python
> class controlled_execution:
> 	def __enter__(self):
> 		set things up
> 		return thing
> 	def __exit__(self, type, value, traceback):
> 		tear things down
> 
> with controlled_execution() as thing:
> 	some code
> ```
>
> Now, when the “with” statement is executed, Python evaluates the expression, calls the `__enter__()` method on the resulting value (which is called a “context guard”), and assigns whatever `__enter__()` returns to the variable given by`as`.  Python will then execute the code body, and *no matter what happens in that code*, call the guard object’s `__exit__()` method.  
>
> As an extra bonus, the `__exit__()` method can look at the exception, if any, and suppress it or act on it as necessary.  To suppress the exception, just return a true value.  For example, the following `__exit__()` method swallows any `TypeError`, but lets all other exceptions through:  
>
> ```python
> def __exit__(self, type, value, traceback):
> 	return isinstance(value, TypeError)
> ```
>

###    1. 自定义支持 `with` 语句的对象

> ```python
> class DummyResource:
>     def __init__(self, tag):
>         self.tag = tag
>         print('Resource [%s]' % tag)
>         
>     def __enter__(self):
>         print('[Enter %s]: Allocate resource.' % self.tag)
>         return self   # 可以返回不同的对象
>     
>     def __exit__(self, exc_type, exc_value, exc_tb):
>         print('[Exit %s]: Free resource.' % self.tag)
>         if exc_tb is None:
>             print('[Exit %s]: Exited without exception.' % self.tag)
>         else:
>             print('[Exit %s]: Exited with exception raised.' % self.tag)
>             return False   # 可以省略，缺省的None也是被看做是False
> ```
>

DummyResource 中的 `__enter__()` 返回的是自身的引用，这个引用可以赋值给 `as` 子句中的 `target` 变量；返回值的类型可以根据实际需要设置为不同的类型，不必是上下文管理器对象本身。

`__exit__()`  方法中对变量 `exc_tb` 进行检测。如果不为 `None`，则表示发生了异常，返回 `False`  表示需要由外部代码逻辑对异常进行处理；注意到如果没有发生异常，缺省的返回值为 `None`，在布尔环境中也是被看做  `False`，但是由于没有异常发生，`__exit__()` 的三个参数都为 `None`，上下文管理代码可以检测这种情况，做正常处理。

###   2. 使用自定义的支持 with 语句的对象

> ```python
> with DummyResource('Normal'):
> 	print('[with-body] Run without exceptions.')
> ```
>

> ```python
> Resource [Normal]
> [Enter Normal]: Allocate resource.
> [with-body] Run without exceptions.
> [Exit Normal]: Free resource.
> [Exit Normal]: Exited without exception.
> ```
>

无异常时，先执行完语句体 with-body，然后执行 `__exit__()` 方法释放资源。

> ```python
> with DummyResource('With-Exception'):
>     print('[with-body] Run with exception.')
>     raise Exception
>     print('[with-body] Run with exception. Failed to finish statement-body!')
> ```

> ```python
> Resource [With-Exception]
> [Enter With-Exception]: Allocate resource.
> [with-body] Run with exception.
> [Exit With-Exception]: Free resource.
> [Exit With-Exception]: Exited with exception raised.
>  
> Traceback (most recent call last):
>   File "G:/demo", line 20, in <module>
>    raise Exception
> Exception
> ```
>

发生异常时，with-body 中发生异常，其中语句并没有执行完，但资源会保证被释放掉，同时产生的异常由 `with` 语句之外的代码逻辑捕获处理。

***

## 标准库 contextlib

### 1. 装饰器

#### 1. `contextmanager(func)`

> One nice shortcut to creating a context manager from a class is to use the `@contextmanager` decorator. To use it, decorate a generator function that calls `yield` exactly *once*. Everything *before* the call to `yield` is considered the code for `__enter__()`. Everything after is the code for `__exit__()`.
>
> The function being decorated must return **a generator-iterator** when called. This iterator must yield exactly one value, which will be bound to the targets in the `with` statement’s `as` clause, if any.

以文件操作为例：

```python
from contextlib import contextmanager

@contextmanager
def open_file(path, mode):
    the_file = open(path, mode)
    yield the_file
    the_file.close()

with open_file('xx.txt', w) as f:
    do something
```

>  We **open** the file,**yield** it, then **close** it. 

生成器函数中 `yield` 之前的语句在 `__enter__()` 方法中执行，`yield` 之后的语句在 `__exit__()`中执行，而 `yield` 产生的值赋给了 `as` 子句中的 `value` 变量。

#### 2. `ContextDecorator`

> It lets you define a context manager using the class-based approach, but inheriting from `contextlib.ContextDecorator`. By doing so, you can use your context manager with the `with` statement as normal *or as a function decorator*. We could do something similar to the HTML example above using this pattern (which is truly insane and shouldn't be done):

> ```python
> from contextlib import ContextDecorator
> 
> class makeparagraph(ContextDecorator):
>     def __enter__(self):
>         print('<p>')
>         return self
> 
>     def __exit__(self, *exc):
>         print('</p>')
>         return False
> 
> @makeparagraph()
> def emit_html():
>     print('Here is some non-HTML')
> ```

### 2. `nested(mgr1[, mgr2[, ...]])`

> Combine multiple context managers into a single nested context manager.
>
> ```python
> from contextlib import nested
> 
> with nested(A, B, C) as (X, Y, Z):
>     do_something()
> ```
>
> is equivalent to this:
>
> ```python
> with A as X:
>     with B as Y:
>         with C as Z:
>             do_something()
> ```
>
> Note that if the `__exit__()` method of one of the nested context managers indicates an exception should be suppressed, no exception information will be passed to any remaining outer context managers. Similarly, if the `__exit__()`method of one of the nested managers raises an exception, any previous exception state will be lost; the new exception will be passed to the `__exit__()`methods of any remaining outer context managers. In general, `__exit__()`methods should avoid raising exceptions, and in particular they should not re-raise a passed-in exception.

### 3. `closing(thing)`

> Return a context manager that closes thing upon completion of the block.
>
> This is basically equivalent to:
>
> ```python
> from contextlib import contextmanager
> 
> @contextmanager
> def closing(thing):
>     try:
>         yield thing
>     finally:
>         thing.close()
> ```

> ```python
> from contextlib import closing
> 
> class ClosingDemo(object):
>     def __init__(self):
>         self.acquire()
>     def acquire(self):
>         print('Acquire resources.')
>     def free(self):
>         print('Clean up any resources acquired.')
>     def close(self):
>         self.free()
> 
> with closing(ClosingDemo()):
>     print('Using resources')
> ```
>
> ```python
> Acquire resources.
> Using resources
> Clean up any resources acquired.
> ```

`closing` 适用于实现了 `close()` 的对象，比如网络连接、数据库连接等，也可以在自定义类时通过接口 `close()` 来执行所需要的资源“清理”工作。

> ```python
> from contextlib import closing
> from urllib.request import urlopen
> 
> with closing(urlopen('http://www.python.org')) as page:
>     for line in page:
>         print(line)
> ```

***

## PS

异常处理中，经常会使用上下文管理器。

***

参考：

[浅谈 Python 的 with 语句](https://www.ibm.com/developerworks/cn/opensource/os-cn-pythonwith/index.html)

[Compound statements](https://docs.python.org/3/reference/compound_stmts.html#with)

[PEP 343 -- The "with" Statement](https://www.python.org/dev/peps/pep-0343/)

[Understanding Python's "with" statement](http://effbot.org/zone/python-with-statement.htm)

[Python with Context Managers](https://jeffknupp.com/blog/2016/03/07/python-with-context-managers/)

[contextlib — Utilities for with-statement contexts.](https://docs.python.org/3.0/library/contextlib.html)