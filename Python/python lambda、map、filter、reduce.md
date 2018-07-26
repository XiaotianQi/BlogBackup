## `lambda`

```python
lambda [parameter_list]: expression
```

返回函数对象。

改为定义函数模式如下：

```python
def <lambda>(parameter_list):
    return expression
```

* 简单示例：

```python
In [6]: f = lambda x, y, z=0: x+y+z

In [7]: f(1, 2)
Out[7]: 3

In [8]: f(1, 2, 3)
Out[8]: 6
```

* 按照指定元素排序：

```python
pairs = [('b', 1), ('a', 0), ('d', 3), ('c', 2)]
pairs.sort(key=lambda pairs:pairs[1])
print(pairs)
```

```bash
[('a', 0), ('b', 1), ('c', 2), ('d', 3)]
```

* 跳转表(jump table)：

```python
lst = [
    lambda x: x.__name__,
    lambda x: x.__class__.__name__,
    lambda x: type(x),
    lambda x: repr(x)
]
```

```python
for i in lst:
    print(i(iter))
for i in range(4):
    print(lst[i](iter))
```

两种 `for` 循环的结果是相同的。

```bash
iter
builtin_function_or_method
<class 'builtin_function_or_method'>
<built-in function iter>
```

* 嵌套：

```python
def func(num):
    return lambda x: x+num
```

```python
foo = func(1)
print(foo)
print(foo(1))
```

```bash
<function func.<locals>.<lambda> at 0x0000024C06612EA0>
2
```

* 闭包：

```python
In [33]: lst = [lambda x:x*i for i in range(10)]

In [34]: lst
Out[34]:
[<function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>]

In [35]: [i(1) for i in lst]
Out[35]: [9, 9, 9, 9, 9, 9, 9, 9, 9, 9]
```

`lst` 中储存的是函数，并不是函数的调用结果。且此函数引用了其作用域外的变量 `i`。当调用其中的函数时，`i` 的值已经循环至 `9`，之前的值都已经被覆盖。

当将 `i` 变量换为普通对象时，不在产生此种现象：

```python
In [36]: lst = [lambda x:x*2 for i in range(10)]

In [37]: [i(1) for i in lst]
Out[37]: [2, 2, 2, 2, 2, 2, 2, 2, 2, 2]
```

或者，添加 `lambda` 本地变量作为参数的方法来重定向 LEGB 的查找结果：

```python
In [38]: lst = [lambda x, i=i: x*i for i in range(10)]

In [39]: [i(1) for i in lst]
Out[39]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

或者，使用生成器表达式：

```python
In [40]: lst = (lambda x:x*i for i in range(10))

In [41]: [i(1) for i in lst]
Out[41]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

* 省去 `parameter_list`：

```python
In [51]: lst = [lambda :x*2 for x in range(10)]

In [52]: lst
Out[52]:
[<function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>]

In [53]: [i() for i in lst]
Out[53]: [18, 18, 18, 18, 18, 18, 18, 18, 18, 18]
```

使用生成器表达式：

```python
In [54]: lst = (lambda :x*2 for x in range(10))

In [55]: lst
Out[55]: <generator object <genexpr> at 0x000002AEC61E2840>

In [56]: [i() for i in lst]
Out[56]: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

不使用 `lambda` 的参数，仅在上述情况中可以使用，因为参数 `x` 可以在 `for` 循环中被赋值。如果使用类似 `lst = [lambda :x*2 for i in range(10)]` 虽然可以声明，但是在调用的时候回返回 `NameError` 或者 `TypeError`。

***

## `map`

```python
map(function, iterable)
```

`Map` 会将一个函数映射到一个输入列表的所有元素上。在 python3 中返回迭代器。

函数接收两个参数，一个是函数，一个是 `Iterable`，`map()`将传入的函数依次作用到序列的每个元素，并把结果作为新的 `Iterator` 返回。

大多数时候，我们使用匿名函数 `lambdas` 来配合 `map`。

操作列表中的元素：

```python
items = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, items))
```

```python
def multiply(x):
        return (x*x)

def add(x):
        return (x+x)

funcs = [multiply, add]
for i in range(5):
    value = map(lambda x: x(i), funcs)
    print(list(value))
```

```bash
[0, 0]
[1, 2]
[4, 4]
[9, 6]
[16, 8]
```

***

## `filter`

```python
filter(function, iterable)
```

和 `map()` 类似，`filter()` 也接收一个函数和一个序列。和 `map()` 不同的是，`filter()` 把传入的函数依次作用于每个元素，然后根据返回值是 `True` 还是 `False` 决定保留还是丢弃该元素。

```python
number_list = range(-5, 5)
less_than_zero = filter(lambda x: x < 0, number_list)
print(list(less_than_zero))
```

```bash
[-5, -4, -3, -2, -1]
```

***

## `reduce`

```python
reduce(function, iterable[, initializer])
```

函数将一个数据集合（链表，元组等）中的所有数据进行下列操作：用传入函数 function 先对集合中的第 1、2 个元素进行操作，得到的结果再与第 3 个元素参与 function 函数运算，以此类推，最后得到一个结果。

```python
from functools import reduce
product = reduce( (lambda x, y: x * y), [1, 2, 3, 4] )
```

```bash
24
```