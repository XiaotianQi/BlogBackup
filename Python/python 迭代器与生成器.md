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
    n = 0
    while n<num:
        yield lst
        lst = [sum(i) for i in zip([0] + lst, lst + [0])]
        n += 1

for i in triangles(10):
    print(i)
```

***

参考：

[迭代器](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143178254193589df9c612d2449618ea460e7a672a366000)