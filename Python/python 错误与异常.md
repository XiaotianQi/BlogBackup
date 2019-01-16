在 python 中，有两种错误：

* 语法错误（syntax errors）
* 异常（exceptions）

***

## Try and Except

我们经常使用 Try and Except，进行异常处理。检测 try 语句块中的错误，从而让 except 语句捕获异常信息并处理。

### 原理

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

- 出现异常，python 就跳回到 `try` 并执行第一个与该异常匹配的 `except` 代码块，异常处理完毕，最后执行 `finally` 中的代码块。
- 出现异常，但没有与匹配的 `except`，异常将被递交到上层的 `try`（如果存在），或者到程序的最上层，抛出默认的异常信息。`finally` 不会执行。如果程序发生的异常不在你的捕获列表中，那么依然会抛出别的异常。
- 可能包含多个except子句，分别来处理不同的特定的异常。但最多只有一个分支会被执行，所以except子句有排序先后问题
- 一个except子句可以同时处理多个异常，这些异常将被放在一个括号里成为一个元组
- 最后一个except子句可以忽略异常的名称，它将被当作通配符使用，也就是说匹配所有异常
- 没有异常，执行 `else` 语句，最后执行 `finally` 语句。

在Python的异常中，有一个通用异常：`Exception`，它可以捕获任意异常。

```python
s1 = 'hello'
try:
    int(s1)
except Exception as e:
    print('错误')
```

很多时候程序只会弹出那么几个异常，没有必要针对所有的异常进行捕获，那样的效率会很低。另外，根据不同的异常种类，制定不同的处理措施，用于准确判断错误类型，存储错误日志，都是非常有必要甚至强制的。

### 主动抛出异常：raise

很多时候，我们需要主动抛出一个异常。Python内置了一个关键字`raise`，可以主动触发异常。

raise唯一的一个参数指定了要被抛出的异常的实例，如果什么参数都不给，那么会默认抛出当前异常。

### 自定义异常

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

### 使用 return 语句

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
finally block
try
```

此时，输出结果与预想中相同。

`else`语句的 `return`仅在以下情况执行了：`try`语句运行成功，且`try`、`finally`语句均无 `return`语句。

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

### 文件读取

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

***

参考：

[Errors and Exceptions](https://docs.python.org/3.7/tutorial/errors.html)

http://www.liujiangblog.com/course/python/49