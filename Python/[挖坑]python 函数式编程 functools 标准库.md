

* `functools.reduce()`

在Python 3里，`reduce()` 函数已经被从全局名字空间里移除了，它现在被放置在`fucntools` 模块里

```text
functools.reduce(function, iterable[, initializer])
函数会对参数序列中元素进行累积。
```

```bash
In [30]: reduce(lambda x,y:x+y, range(5))
Out[30]: 10
```

* `functools.partial()`

```text
 functools.partial(func[,*args][, **keywords])
 把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。
```

`int()`函数可以把字符串转换为整数，当仅传入字符串时，`int()`函数默认按十进制转换：

```bash
In [73]: int('10')
Out[73]: 10

In [74]: int('10', base='2')	# 传入base参数，就可以做N进制的转换：
```

假设要转换大量的二进制字符串，每次都传入`int(x, base=2)`非常麻烦，于是，我们想到，可以定义一个`int2()`的函数，默认把`base=2`传进去：

```python
def int2(x, base=2):
    return int(x, base)
```

`functools.partial`就是帮助我们创建一个偏函数的，不需要我们自己定义`int2()`，可以直接使用下面的代码创建一个新的函数`int2`：

```bash
In [81]: import functools

In [82]: int2 = functools.partial(int, base=2)

In [83]: int2('10')
Out[83]: 2
```

当函数的参数个数太多，需要简化时，使用`functools.partial`可以创建一个新的函数，这个新函数可以固定住原函数的部分参数，从而在调用时更简单。