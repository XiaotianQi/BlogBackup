在解释生成器(Generators)之前，需要解释一些前置概念。

## 可迭代对象(Iterable)

Python 中任意的对象，只要它定义了可以返回一个迭代器的 `__iter__` 方法，或者定义了可以支持下标索引的 `__getitem__`方法，那么它就是一个可迭代对象。

可直接用于 `for` 循环的数据类型大致一下两类：

* 集合数据类型：`list`、`tuple`、`dict`、`set`、`str`等；
* `generator`：包括生成器和带 `yield` 的 `generator function`。

这些对象统称为可迭代对象：`Iterable`。也可以理解为：凡是可以 `for` 循环的对象都是 `Iterable`。

***

## 迭代器(Iterator)

任意对象，只要定义了 `__next__` 方法，它就是一个迭代器。可以被 `next()` 函数调用并不断返回下一个值。

`Iterator` 对象表示的是一个数据流，不能提前预知其的长度，只能不断通过 `next()` 函数实现按需计算得到下一个数据，所以 `Iterator` 的计算是惰性的，只有在需要返回下一个数据时它才会计算。`Iterator` 可以表示一个无限大的数据流，例如全体自然数。而 `list` 是永远不可能存储全体自然数。

`list`、`dict`、`str` 虽然是 `Iterable`，却不是 `Iterator`。不过可以通过 `iter()` 函数获得一个 `Iterator` 对象。

```python
iter(iterable)
```

* `iterable` 参数是一个容器，则返回这个容器的迭代器对象；
* `iterable` 参数本身就是一个迭代器，则返回其自身。

```python
In [1]: from collections import Iterator

In [2]: isinstance([], Iterator)
Out[2]: False

In [3]: isinstance(iter([]), Iterator)
Out[3]: True
```

***

## 迭代(Iteration)

重复执行一系列运算步骤，从前面的量依次求出后面的量的过程。此过程的每一次结果，都是由对前一次所得结果施行相同的运算步骤得到的。

每一次迭代得到的结果会作为下一次迭代的初始值。

当使用一个循环来遍历时，这个过程本身就叫迭代。

***

## 生成器(Generators)

生成器也是一种迭代器对象。只可以读取它一次，因为没有把所有的值存在内存中，而是在运行时生成值。通过遍历来使用它们，要么用一个 `for` 循环，要么将它们传递给任意可以进行迭代的函数和结构。

生成器都是 `Iterator` 对象。生成器可以用于 `for` 循环，也可以被 `next()` 函数不断调用并返回下一个值，直至无法继续返回下一个值抛出 `StopIteration` 错误。区别于迭代器是，生成器中的值的数量是有限的。

斐波那契数列的生成器。

```python
def fib(num):
    a = b = 1
    for i in range(num):
        yield a
        a, b = b, a+b

for i in fib(100):
    print(i)
```

杨辉三角生成器。

```python
def triangles(num):
    lst = [1]
    for i in range(num):
        yield lst
        lst = [sum(i) for i in zip([0] + lst, lst + [0])]


for i in triangles(10):
    print(i)
```

字母加密：

```python
def key_value_gen(k):
    yield chr(k+65)
    yield chr((k+13)%26+65)

d = dict(map(key_value_gen, range(26)))
```

***

## 解析式(comprehensions)

解析式（又称推导式）是Python的一种独有特性。解析式是可以从一个数据序列构建另一个新的数据序列的结构体。 共有三种推导：

* 列表(list)解析式
* 集合(set)解析式
* 字典(dict)解析式

### 列表(list)解析式

```python
list_variable = [out_exp for out_exp in input_list if exp]
```

```python
In [1]: lst = [i for i in range(20) if i%2 == 0]

In [2]: lst
Out[2]: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

使用嵌套 `if`：

```python
In [7]: lst = [i**2 for i in range(50) if i%2 == 0 if i%5 == 0]

In [8]: lst
Out[8]: [0, 100, 400, 900, 1600]
```

嵌套循环：

```python
In [11]: lst = [i*j for i in [1, 3, 5] for j in [2, 4, 6]]

In [12]: lst
Out[12]: [2, 4, 6, 6, 12, 18, 10, 20, 30]
```

当传入函数时：

以 `lambda` 为例，`lst` 中储存的是函数，并不是函数的调用结果。

```python
In [10]: lst = [lambda x:x*2 for x in range(3)]

In [11]: lst
Out[11]:
[<function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>]

In [12]: lst[0](1)
Out[12]: 2

In [13]: lst[2](1)
Out[13]: 2

In [14]: for i in lst:
    ...:     print(i(1))
    ...:
2
2
2
```

区别于以下这种情况：

当调用其中的函数时，`x` 的值已经是 `2`。

```python
In [22]: lst = [lambda :x*2 for x in range(3)]

In [23]: lst
Out[23]:
[<function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>,
 <function __main__.<listcomp>.<lambda>()>]

In [24]: lst[0]()
Out[24]: 4

In [25]: lst[1]()
Out[25]: 4

In [26]: lst[2]()
Out[26]: 4
```

### 集合(set)解析式

集合解析式跟列表解析式类似。唯一的区别在于集合解析式使用大括号 `{}`。

```python
In [3]: s = {i for i in range(20) if i%2 != 0}

In [4]: s
Out[4]: {1, 3, 5, 7, 9, 11, 13, 15, 17, 19}
```

### 字典(dict)解析式

```python
d = {key: value for (key, value) in iterable}
```

```python
In [45]: d = {i:chr(i+65) for i in range(5)}

In [46]: d
Out[46]: {0: 'A', 1: 'B', 2: 'C', 3: 'D', 4: 'E'}
```

交换上面代码中字典 `d` 的 key 和 value 的位置。

```python
In [50]: d = {v: k for k, v in d.items()}

In [51]: d
Out[51]: {'A': 0, 'B': 1, 'C': 2, 'D': 3, 'E': 4}

```

把同一个字母但不同大小写的值合并。

```python
mcase = {'a': 10, 'b': 34, 'A': 7, 'Z': 3}

mcase_frequency = {
    k.lower(): mcase.get(k.lower(), 0) + mcase.get(k.upper(), 0)
    for k in mcase.keys()
}
```

```bash
mcase_frequency == {'a': 17, 'z': 3, 'b': 34}
```

***

## 生成器表达式

将解析式外围括号改为圆括号 `(` `)`，便是生成器表达式。

生成器表达式的语法与解析式相似，但它返回的是生成器对象。解析式自创建之后便存在，会占用所有必要内存来存储其值。生成器表达式不会使用这么多的存储空间，而是处于暂挂状态，当对其进行迭代时才恢复，就像生成器函数的主体一样。

```python
In [52]: lst = (i for i in 'abcdef')

In [53]: lst
Out[53]: <generator object <genexpr> at 0x0000022A5A55DA20>

In [54]: for i in lst:
    ...:     print(i)
    ...:
a
b
c
d
e
f
```
***

参考：

[迭代器](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143178254193589df9c612d2449618ea460e7a672a366000)