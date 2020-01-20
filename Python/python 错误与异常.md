在 python 中，有两种错误：

* 语法错误（syntax errors）
* 异常（exceptions）

良好的异常处理可以让你的程序更加健壮，清晰的错误信息更能帮助你快速修复问题。

经常使用 Try and Except，进行异常处理。检测 try 语句块中的错误，从而让 except 语句捕获异常信息并处理。

## 异常处理语句 try...excpet...finally

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
    总会执行，论try执行情况和except异常触发情况
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
except (RuntimeError, TypeError, NameError) as e:
	pass
except:
    print("Unexpected error:", sys.exc_info()[0])
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

- 出现异常，python 就执行第一个与该异常匹配的 `except` 代码块，异常处理完毕，最后执行 `finally` 中的代码块。
- 出现异常，但没有与匹配的 `except`，异常将被递交到上层的 `try`（如果存在），或者到程序的最上层，抛出默认的异常信息。`finally` 不会执行。如果程序发生的异常不在捕获列表中，那么依然会抛出别的异常。
- `except`语句不是必须的，`finally`语句也不是必须的，但是二者必须要有一个，否则就没有`try`的意义了。
- `except`语句可以有多个，分别来处理不同的特定的异常。Python会按`except`语句的顺序依次匹配指定的异常，如果异常已经处理就不会再进入后面的`except`语句，即最多只有一个`except`语句会被执行。
- `except`语句可以以元组形式同时指定多个异常
- 最后一个`except`子句可以忽略异常的名称，它将被当作通配符使用，也就是说匹配所有异常。`except`语句后面如果不指定异常类型，则默认捕获所有异常，可以通过logging或者sys模块获取当前异常。
- 不建议在不清楚逻辑的情况下捕获所有异常，有可能隐藏了很严重的问题。
- 没有异常，执行 `else` 语句，最后执行 `finally` 语句。
- 尽量使用内置的异常处理语句来替换`try/except`语句，比如`with`语句，`getattr()`方法。

在Python的异常中，有一个通用异常：`Exception`，它可以捕获任意异常。

```python
s1 = 'hello'
try:
    int(s1)
except Exception as e:
    print('错误')
```

很多时候程序只会弹出那么几个异常，没有必要针对所有的异常进行捕获，那样的效率会很低。另外，根据不同的异常种类，制定不同的处理措施，用于准确判断错误类型，存储错误日志，都是非常有必要甚至强制的。

### 加入 return 语句

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

如果有 `finally` 语句，且当中存在 `return` 时，那么无论是否发生异常，都将会得到 `finally` 语句中 `return` 的信息。同时，`else` 语句不再执行。

* 当 `finally` 语句不存在 `return`时：

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
finally block
try
```

此时，输出结果与预想中相同。

* `else`语句的 `return`仅在以下情况执行了：`try`语句运行成功，且`try`、`finally`语句均无 `return`语句。

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

嵌入其中`return`的语句，必须灵活运用：

```python
def read_filedata(filename):

    try:
        f = open(filename, 'r')
        f_data = f.read()
    except IOError as e:
        f = "File does not exist." + str(e)
    else:
        return f_data
    finally:
        if isinstance(f, str):
            return f
        elif hasattr(f, 'read'):
            f.close()
        else:
            return 'Unknown Error'
```

### 使用内置的语法范式代替try/except

Python 本身提供了很多的语法范式简化了异常的处理，比如`for`语句就处理了的`StopIteration`异常，让你很流畅地写出一个循环。

#### `with`

`with`语句在打开文件后会自动调用`finally`并关闭文件。我们在写 Python 代码时应该尽量避免在遇到这种情况时还使用try/except/finally的思维来处理。

使用 Try and Except 实现文件的读取与关闭。

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

#### `gethattr()`

再比如，当需要访问一个不确定的属性时，有可能会写出这样的代码：

```
try:
    test = Test()
    name = test.name  # not sure if we can get its name
except AttributeError:
    name = 'default'
```

其实可以使用更简单的`getattr()`来达到你的目的。               

```
name = getattr(test, 'name', 'default')
```

***

## 主动抛出异常：raise

很多时候，我们需要主动抛出一个异常。Python内置了一个关键字`raise`，可以主动触发异常。`raise`关键字后面可以指定你要抛出的异常实例，一般来说抛出的异常越详细越好。

```python
raise NameError("bad name!")
```

`raise`唯一的一个参数指定了要被抛出的异常的实例，如果什么参数都不给，那么会默认抛出当前异常。

### 传递异常 

捕捉到了异常，但是又想重新抛出它（传递异常），使用不带参数的`raise`语句即可：

```python
def f1():
    print(1/0)

def f2():
    try:
        f1()
    except Exception as e:
        raise  # don't raise e !!!

f2()
```

还有一些技巧可以考虑，比如抛出异常前你希望对异常的信息进行更新。

```python
def f2():
    try:
        f1()
    except Exception as e:
        e.args += ('more info',)
        raise
```

### raise "Exception string"

把字符串当成异常抛出看上去是一个非常简洁的办法，但其实是一个非常不好的习惯。

```python
if is_work_done():
    pass
else:
    raise "Work is not done!" # not cool
```

上面的语句如果抛出异常，那么会是这样的：

```
Traceback (most recent call last):
  File "/demo/exception_hanlding.py", line 48, in <module>
    raise "Work is not done!"
TypeError: exceptions must be old-style classes or derived from BaseException, not str
```

这在 Python2.4 以前是可以接受的做法，但是没有指定异常类型有可能会让下游没办法正确捕获并处理这个异常，从而导致你的程序难以维护。简单说，这种写法是是封建时代的陋习，应该扔了。

***

## 自定义异常

Python内置了很多的异常类，并且这些类都是从BaseException类派生的。

| 异常名            | 解释                                           |
| ----------------- | ---------------------------------------------- |
| AttributeError    | 试图访问一个对象没有的属性                     |
| IOError           | 输入/输出异常                                  |
| ImportError       | 无法引入模块或包；多是路径问题或名称错误       |
| IndentationError  | 缩进错误                                       |
| IndexError        | 下标索引错误                                   |
| KeyError          | 试图访问不存在的键                             |
| KeyboardInterrupt | Ctrl+C被按下，键盘终止输入                     |
| NameError         | 使用未定义的变量                               |
| SyntaxError       | 语法错误                                       |
| TypeError         | 传入对象的类型与要求的不符合                   |
| UnboundLocalError | 试图访问一个还未被设置的局部变量               |
| ValueError        | 传入一个调用者不期望的值，即使值的类型是正确的 |
| OSError           | 操作系统执行错误                               |

自定义异常应该继承`Exception`类，直接继承或者间接继承都可以。异常的名字都以`Error`结尾，我们在为自定义异常命名的时候也需要遵守这一规范，就跟标准的异常命名一样。

一般你在自定义异常类型时，需要考虑的问题应该是这个异常所应用的场景。如果内置异常已经包括了需要的异常，建议考虑使用内置的异常类型。比如希望在函数参数错误时抛出一个异常，可能并不需要定义一个`InvalidArgumentError`，使用内置的`ValueError`即可。

Exception VS BaseException

## 异常层级关系

```text
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration
      +-- ArithmeticError
      |    +-- FloatingPointError
      |    +-- OverflowError
      |    +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- MemoryError
      +-- NameError
      |    +-- UnboundLocalError
      +-- OSError
      |    +-- BlockingIOError
      |    +-- ChildProcessError
      |    +-- ConnectionError
      |    |    +-- BrokenPipeError
      |    |    +-- ConnectionAbortedError
      |    |    +-- ConnectionRefusedError
      |    |    +-- ConnectionResetError
      |    +-- FileExistsError
      |    +-- FileNotFoundError
      |    +-- InterruptedError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- PermissionError
      |    +-- ProcessLookupError
      |    +-- TimeoutError
      +-- ReferenceError
      +-- RuntimeError
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- SyntaxError
      |    +-- IndentationError
      |         +-- TabError
      +-- SystemError
      +-- TypeError
      +-- ValueError
      |    +-- UnicodeError
      |         +-- UnicodeDecodeError
      |         +-- UnicodeEncodeError
      |         +-- UnicodeTranslateError
      +-- Warning
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- ResourceWarning
```

当要捕获一个通用异常时，应该用`Exception`还是`BaseException`？

从`Exception`的层级结构来看，`BaseException`是最基础的异常类，`Exception`继承了它。`BaseException`除了包含所有的`Exception`外还包含了`SystemExit`，`KeyboardInterrupt`和`GeneratorExit`三个异常。

由此看来，程序在捕获所有异常时更应该使用`Exception`而不是`BaseException`，因为被排除的三个异常属于更高级别的异常，合理的做法应该是交给Python的解释器处理。

***

## 综上

1. 只处理预知的异常，避免捕获所有异常然后吞掉它们。
2. 抛出的异常应该说明原因，有时候通过异常类型也猜不出所以然。
3. 避免在`catch`语句块中干一些没意义的事情，捕获异常也是需要成本。
4. 不要使用异常来控制流程，那样程序会无比难懂和难维护。
5. 如果有需要，切记使用`finally`来释放资源。
6. 如果有需要，请不要忘记在处理异常后做清理工作或者回滚操作。

***

参考：

[Errors and Exceptions](https://docs.python.org/3.7/tutorial/errors.html)，官方文档

[异常处理](http://www.liujiangblog.com/course/python/49)，刘江

[总结：Python中的异常处理](https://segmentfault.com/a/1190000007736783)，betacat