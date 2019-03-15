在解释生成器(Generators)之前，需要解释一些前置概念。

## 迭代(Iteration)

重复执行一系列运算步骤，从前面的量依次求出后面的量的过程。此过程的每一次结果，都是由对前一次所得结果施行相同的运算步骤得到的。

每一次迭代得到的结果会作为下一次迭代的初始值。

当使用一个循环来遍历时，这个过程本身就叫迭代。

### 可迭代对象(Iterable)

Python 中任意的对象，只要它定义了可以返回一个迭代器的 `__iter__` 方法，或者定义了可以支持下标索引的 `__getitem__`方法，那么它就是一个可迭代对象。

可迭代的数据类型大致一下两类：

* 内置数据类型：`list`、`tuple`、`dict`、`set`、`str`；
* `generator`：包括生成器和带 `yield` 的 `generator function`。

可以通过collections模块的Iterable类型来判断一个对象是否可迭代：

```python
In [2]: from collections.abc import Iterable

In [3]: isinstance('abc', Iterable)
Out[3]: True

In [4]: isinstance([1,2,3], Iterable)
Out[4]: True
```

### 迭代器(Iterator)

迭代器是访问集合内元素的一种方式。迭代器对象从集合的第一个元素开始访问，直到所有的元素都被访问一遍后结束。 迭代器是单向的，一旦用完，就不会再收到任何输出。

迭代器只能往后遍历不能回溯，不像列表，随时可以取后面的数据，也可以返回头取前面的数据。迭代器通常要实现两个基本的方法：`iter()` 和 `next()`。同时定义了 `__next__` 和 `__iter__` 两个方法，它就是一个迭代器。可以被 `next()` 函数调用并不断返回下一个值。

`Iterator` 对象表示的是一个数据流，不能提前预知其的长度，只能不断通过 `next()` 函数实现按需计算得到下一个数据，所以 `Iterator` 的计算是惰性的，只有在需要返回下一个数据时它才会计算。`Iterator` 可以表示一个无限大的数据流，例如全体自然数。

NOTE：**迭代器只能迭代一次**

### iter()函数

`iter(object[, sentinel])`

- 返回一个迭代器对象。
- `object` 参数必须是可调用的对象，用于不断调用（没有参数），产出各个值
- `sentinel` 是哨符，当可调用的对象返回这个值时，触发迭代器抛出 `StopIteration` 异常，但不输出哨符。

官方文档提供的实例，逐行读取文件，直至遇到空行或者文件末尾为止：

```python
with open('mydata.txt') as fp:
    for line in iter(fp.readline, '\n'):
        process_line(line)
```

解释器迭代对象 `x` 时，会自动调用 `iter(x)` , `iter函数`将做如下检查:

- 检查对象是否实现了 `__iter__` 方法，如果实现了就调用它，获取一个迭代器。
- 如果没有实现 `__iter__` 方法，但是实现了 `__getitem__` 方法，Python 会创建一个迭代器，尝试按顺序（从索引 `0` 开始）获取元素。
- 如果尝试失败，Python 抛出 `TypeError` 异常，通常会提示“`C object is not iterable`”其中 `C` 是 `x`所属的类。

for item in Iterable 循环的本质就是先通过iter()函数获取可迭代对象Iterable的迭代器，然后对获取到的迭代器不断调用next()方法来获取下一个值并将其赋值给item，当遇到StopIteration的异常后循环结束。

### 可迭代对象VS迭代器

*  Python 从可迭代的对象中获取迭代器
* 凡是可作用于for循环的对象都是可迭代类型
* 凡是可作用于next()函数的对象都是迭代器类型

list、dict、str等是可迭代的但不是迭代器，因为它们**无法调用next()函数**。可以通过iter()函数将它们转换成迭代器。Python的for循环本质上就是通过不断调用next()函数实现的

`list`、`dict`、`str` 虽然是 `Iterable`，却不是 `Iterator`。不过可以通过 `iter()` 函数获得一个 `Iterator` 对象。

```python
In [1]: from collections import Iterator

In [2]: isinstance([], Iterator)
Out[2]: False

In [3]: isinstance(iter([]), Iterator)
Out[3]: True
```

从类的角度:

* 有魔法函数`__iter__`则为可迭代
* 有魔法函数`__iter__`、`__next__`则为迭代器

迭代器也不是线程安全的，在多线程环境中对可变集合使用迭代器是一个危险的操作。但如果小心谨慎，或者干脆贯彻函数式思想坚持使用不可变的集合，那这也不是什么大问题。 

使用迭代器的循环可以避开索引，但有时候我们还是需要索引来进行一些操作的。这时候内建函数enumerate就派上用场咯，它能在iter函数的结果前加上索引，以元组返回，用起来就像这样：

```python
for idx, ele in enumerate(lst):
    print(idx, ele)
```

***

## 推导式(Comprehensions)

推导式（又称推导式）是Python的一种独有特性。推导式是可以从一个数据序列构建另一个新的数据序列的结构体。 共有三种推导：

* 列表(list)推导式
* 集合(set)推导式
* 字典(dict)推导式
* 元组(tuple)推导式

因为if子句里的条件需要计算，同时结果也需要进行同样的计算，不希望计算两遍，就像这样： 

```python
(x.doSomething() for x in lst if x.doSomething()>0)

# 通过组合
(x for x in (y.doSomething() for y in lst) if x>0)
```

列表解析可以替代绝大多数需要用到map和filter的场合。

### 列表推导式

```python
list_variable = [out_exp for out_exp in input_list if exp]
```

```python
In [1]: lst = [i for i in range(20) if i%2 == 0]

In [2]: lst
Out[2]: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

* 使用嵌套 `if`：

```python
In [7]: lst = [i**2 for i in range(50) if i%2 == 0 if i%5 == 0]

In [8]: lst
Out[8]: [0, 100, 400, 900, 1600]
```

* 多维数组：

```python
In [15]: lst = [[x, x**2] for x in range(5)]

In [16]: lst
Out[16]: [[0, 0], [1, 1], [2, 4], [3, 9], [4, 16]]
```

`out_exp` 也可以替换为 dict 和 tuple 等：

```python
In [17]: lst = [{x:x**2} for x in range(5)]

In [18]: lst
Out[18]: [{0: 0}, {1: 1}, {2: 4}, {3: 9}, {4: 16}]
```

5x3 矩阵：

```python
In [30]: lst = [[i for i in range(5)] for j in range(3)]

In [31]: lst
Out[31]: [[0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4]]
```

区别于不使用 `[]` 的情况：

```python
In [11]: lst = [i*j for i in [1, 3, 5] for j in [2, 4, 6]]

In [12]: lst
Out[12]: [2, 4, 6, 6, 12, 18, 10, 20, 30]
```

与下面代码效果相同：

```python
lst = []
for i in [1, 3, 5]:
    for j in [2, 4, 6]:
        lst.append(i*j)
```

生成 5x5 数组：

```python
In [5]: l = [[x/10+y for x in range(1,6)] for y in range(1,6)]

In [6]: l
Out[6]:
[[1.1, 1.2, 1.3, 1.4, 1.5],
 [2.1, 2.2, 2.3, 2.4, 2.5],
 [3.1, 3.2, 3.3, 3.4, 3.5],
 [4.1, 4.2, 4.3, 4.4, 4.5],
 [5.1, 5.2, 5.3, 5.4, 5.5]]
```

并将其转置：

```python
In [12]: list(map(list, zip(*l)))
Out[12]:
[[1.1, 2.1, 3.1, 4.1, 5.1],
 [1.2, 2.2, 3.2, 4.2, 5.2],
 [1.3, 2.3, 3.3, 4.3, 5.3],
 [1.4, 2.4, 3.4, 4.4, 5.4],
 [1.5, 2.5, 3.5, 4.5, 5.5]]

In [13]: [[y[x] for y in l] for x in range(len(l[0]))]
Out[13]:
[[1.1, 2.1, 3.1, 4.1, 5.1],
 [1.2, 2.2, 3.2, 4.2, 5.2],
 [1.3, 2.3, 3.3, 4.3, 5.3],
 [1.4, 2.4, 3.4, 4.4, 5.4],
 [1.5, 2.5, 3.5, 4.5, 5.5]]
```

* 列表展平

```python
In [31]: lst
Out[31]: [[0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4]]

In [32]: [i for j in lst for i in j]
Out[32]: [0, 1, 2, 3, 4, 0, 1, 2, 3, 4, 0, 1, 2, 3, 4]
```

与下面代码效果相同：

```python
l = []
for j in lst:
    for i in j:
        l.append(i)
print(l)
```

* 矩阵转置

```python
In [65]: lst = [[i for i in range(5)] for j in range(3)]

In [66]: lst
Out[66]: [[0, 1, 2, 3, 4], [0, 1, 2, 3, 4], [0, 1, 2, 3, 4]]

In [67]: l = [[row[i] for row in lst] for i in range(5)]

In [68]: l
Out[68]: [[0, 0, 0], [1, 1, 1], [2, 2, 2], [3, 3, 3], [4, 4, 4]]
```

或者写为：

```python
l = []
for i in range(5):
    l_row = []
    for row in lst:
        l_row.append(row[i])
    l.append(l_row)
```

或者：

```python
list(zip(*lst))
```

* `out_exp` 为函数：

以 `lambda` 为例，`lst` 中储存的是函数，并不是函数的调用结果。

```python
In [36]: lst = [lambda x:x*2 for i in range(10)]

In [37]: lst
Out[37]:
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

In [38]: [i(1) for i in lst]
Out[38]: [2, 2, 2, 2, 2, 2, 2, 2, 2, 2]
```

如果引起闭包：

```python
In [33]: lst = [lambda x:x*i for i in range(10)]

In [34]: [i(1) for i in lst]
Out[34]: [9, 9, 9, 9, 9, 9, 9, 9, 9, 9]
```

解决上面代码中 `i` 值为 9 的方法，可以使用 `lambda` 本地变量，或者使用生成器表达式。

```python
In [38]: lst = [lambda x, i=i: x*i for i in range(10)]

In [39]: [i(1) for i in lst]
Out[39]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 集合推导式

集合推导式跟列表推导式类似。唯一的区别在于集合推导式使用大括号 `{}`。

```python
In [3]: s = {i for i in range(20) if i%2 != 0}

In [4]: s
Out[4]: {1, 3, 5, 7, 9, 11, 13, 15, 17, 19}
```

### 字典推导式

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

### 元组推导式

```python
tuple(t for t in range(11))
```

***

## 生成器(Generators)

生成器也是一种迭代器对象。首先请确信，生成器就是一种迭代器。在python 3.x中 generator（有yield关键字的函数则会被识别为generator函数）中的next变为`__next__()`了,`next()`是python 3.x以前版本中的方法

生成器拥有`__next__()`方法并且行为与迭代器完全相同，这意味着生成器也可以用于Python的for循环中。只可以读取它一次，因为没有把所有的值存在内存中，而是在运行时生成值。通过遍历来使用它们，要么用一个 `for` 循环，要么将它们传递给任意可以进行迭代的函数和结构。

生成器都是 `Iterator` 。生成器可以用于 `for` 循环，也可以被 `__next__()` 不断调用并返回下一个值，直至无法继续返回下一个值抛出 `StopIteration` 错误。区别于迭代器是，生成器中的值的数量是有限的。可以通过以下方式使用生成器：

* 使用生成器函数定义生成器
* 生成器表达式

### 生成器函数

函数中使用关键字''yield"之后，函数就不再是普通的函数了，而变成了生成器函数。generator函数和函数的执行流程不一样。函数是顺序执行，遇到return语句或者最后一行函数语句就返回。而变成generator函数，在每次调用next()的时候执行，遇到yield语句返回，再次执行时从上次返回的yield语句处继续执行。

生成器函数的特性如下：

* 第一次调用生成器函数的`__next__()`方法时，生成器函数才开始执行生成器（而不是构建生成器时），直到遇到yield时暂停执行（挂起），并且yield的参数将作为此次`__next__()`方法的返回值； 
* 之后每次调用生成器函数的`__next__()`方法，生成器将从上次暂停执行的位置恢复执行生成器，直到再次遇到yield时暂停，并且同样的，yield的参数将作为`__next__()`方法的返回值； 
* 如果当调用`__next__()`方法时生成器函数结束（遇到空的return语句或是到达函数体末尾），则这次`__next__()`方法的调用将抛出StopIteration异常（即for循环的终止条件）； 
* 生成器在每次暂停执行时，函数体内的所有变量都将被封存(freeze)在生成器中，并将在恢复执行时还原，并且类似于闭包，即使是同一个生成器函数返回的生成器，封存的变量也是互相独立的。

生成器虽然也是函数，但是它不可以使用return输出返回值，遇到return就会中断。

比如实现 `range()`函数：

```python
def frange(start, stop, increment):	# 生成器函数
    x = start
    while x < stop:
        yield x
        x += increment
```

1. 斐波那契数列的生成器

```python
def fib(num):
    a, b = 0, 1
    for i in range(num):
        yield a
        a, b = b, a+b

def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield a
        a, b = b, a + b
        n = n + 1
    return 'Done'
```

生成器中可以没有return语句，并不影响生成器的功能。上述代码例子中带的return 'Done'，将会在抛出StopIteration的错误时打印出来。你可以用next()函数触发StopIteration错误试一试。使用 for 循环不会触发。

```text
StopIteration                             Traceback (most recent call last)
<ipython-input-9-dcf180275632> in <module>
----> 1 f.__next__()

StopIteration: Done
```

2. 杨辉三角生成器

```python
def triangles(num):
    lst = [1]
    for i in range(num):
        yield lst
        lst = [sum(i) for i in zip([0] + lst, lst + [0])]


for i in triangles(10):
    print(i)
```

3. 字母加密

```python
def key_value_gen(k):
    yield chr(k+65)
    yield chr((k+13)%26+65)

d = dict(map(key_value_gen, range(26)))
```

### 生成器表达式

将推导式外围括号改为圆括号 `(` `)`，便是生成器表达式。

生成器表达式的语法与推导式相似，但它返回的是生成器对象。推导式自创建之后便存在，会占用所有必要内存来存储其值。生成器表达式不会使用这么多的存储空间，而是处于暂挂状态，当对其进行迭代时才恢复，就像生成器函数的主体一样。

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

### 迭代器VS生成器

每个生成器都是一个迭代器，但是反过来不行。通常生成器是通过调用一个或多个yield表达式构成的函数生成，同时满足迭代器的定义。生成器其实是一种特殊的迭代器，但是不需要像迭代器一样实现`__iter__`和`__next__`方法，只需要使用关键字yield就可以。

```bash
In [63]: from collections.abc import Iterable, Iterator, Generator

In [64]: isinstance(range(5), Iterable)
Out[64]: True

In [65]: isinstance(range(5), Iterator)
Out[65]: False

In [66]: isinstance(iter(range(5)), Iterator)
Out[66]: True

In [67]: isinstance(iter(range(5)), Generator)
Out[67]: False
```

***

## 示例

### map()

使用 `map` 函数，可以处理迭代器中的每一个项，以便生成新的迭代器。接受一个迭代器并生成一个派生迭代器。

```python
In [1]: l = range(10)

In [2]: r = map(str, l)

In [3]: list(r)
Out[3]: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']
```

### 降维

不想将一个迭代器转换为另一个迭代器，而只是想通过迭代器来计算一个值。

```text
functools.reduce(function, iterable[, initializer])

Apply function of two arguments cumulatively to the items of sequence, from left to right, so as to reduce the sequence to a single value. For example, reduce(lambda x, y: x+y, [1, 2, 3, 4, 5]) calculates ((((1+2)+3)+4)+5). The left argument, x, is the accumulated value and the right argument, y, is the update value from the sequence. If the optional initializer is present, it is placed before the items of the sequence in the calculation, and serves as a default when the sequence is empty. If initializer is not given and sequence contains only one item, the first item is returned.
```

`map` 接受一个迭代器并生成一个派生迭代器。`reduce`接受一个迭代器并返回由所有项派生的单个值。这些概念共同构成一种处理大量数据的强大方法。

```text
str.join(sequence)

返回通过指定字符连接序列中元素后生成的新字符串。
```

```python
In [8]: strings = (chr(ord('a')+i) for i in range(10))

In [9]: '-'.join(strings)
Out[9]: 'a-b-c-d-e-f-g-h-i-j'
```

### 过滤

```bash
In [15]: not_by2or3 = ( i for i in range(20) if math.gcd(i, 2) == 1 and math.gcd(i, 3)=
    ...: = 1)

In [16]: list(not_by2or3)
Out[16]: [1, 5, 7, 11, 13, 17, 19]
```

或者使用filter()函数

```bash
In [17]: def not_by2or3(n):
    ...:     return math.gcd(n, 2)==1 and math.gcd(n, 3)==1
    ...:

In [18]: list(filter(not_by2or3, range(20)))
Out[18]: [1, 5, 7, 11, 13, 17, 19]

In [19]: list(filter(lambda x: math.gcd(x, 2)==1 and math.gcd(x, 3)==1, range(20)))
Out[19]: [1, 5, 7, 11, 13, 17, 19]
```

### 连接一系列迭代器

```text
itertools.chain(*iterables)

Make an iterator that returns elements from the first iterable until it is exhausted, then proceeds to the next iterable, until all of the iterables are exhausted. Used for treating consecutive sequences as a single sequence.

chain('ABC', 'DEF') --> A B C D E F
```

```bash
In [20]: import itertools

In [21]: it = itertools.chain(range(5), range(5, 0, -1), range(5))

In [22]: list(it)
Out[22]: [0, 1, 2, 3, 4, 5, 4, 3, 2, 1, 0, 1, 2, 3, 4]

In [23]: list_of_iters = [range(5), range(5, 0, -1), range(5)]

In [24]: it = itertools.chain(*list_of_iters)

In [25]: list(it)
Out[25]: [0, 1, 2, 3, 4, 5, 4, 3, 2, 1, 0, 1, 2, 3, 4]
```

### 无限迭代器

```python
def zigzag_iters(period): 
    while True: 
        yield range(period)
        yield range(period, 0, -1)
```

```bash
In [26]: def zigzag_iters(period):
    ...:     while True:
    ...:         yield range(period)
    ...:         yield range(period, 0, -1)
    ...:

In [27]: it = zigzag_iters(2)

In [28]: next(it)
Out[28]: range(0, 2)

In [29]: next(it)
Out[29]: range(2, 0, -1)

In [30]: next(it)
Out[30]: range(0, 2)

In [31]: next(it)
Out[31]: range(2, 0, -1)
```

如果继续输入 `next(it)`，将继续在两个输出范围之间反复循环，因为 `while True`会实现无限循环，其内部没有代码可用于中断循环的执行。此生成器持续处于暂挂和恢复状态，直到 Python进程（在此例中为交互式解释器）终止。或者，可以对任何生成器对象使用 `close()`方法来结束它，而无论它在何处执行。

无限迭代器实际上是一个有用的概念。但是，必须知道要尝试通过此类迭代器的内容构建任何集合的操作。

### 级联迭代器

```python
def zigzag(period): 
    while True: 
        for n in range(period): 
            yield n
        for n in range(period, 0, -1): 
            yield n
```

```bash
In [35]: def zigzag(period):
    ...:     while True:
    ...:         for n in range(period):
    ...:             yield n
    ...:         for n in range(period, 0, -1):
    ...:             yield n
    ...:

In [36]: it = zigzag(2)

In [37]: next(it)
Out[37]: 0

In [38]: next(it)
Out[38]: 1

In [39]: next(it)
Out[39]: 2

In [40]: next(it)
Out[40]: 1

In [41]: next(it)
Out[41]: 0

In [42]: next(it)
Out[42]: 1
```

这是另一种常见模式，其中外部迭代器是一个或多个内部迭代器的级联形式。Python 为它提供了标准语法，即
`yield from` 语句，您可以在下面重写的功能相同的 `zigzag` 中看到此语句。

```python
def zigzag(period): 
    while True: 
        yield from range(period)
        yield from range(period, 0, -1)
```

如果想要一个迭代器完整提供另一个迭代器，请考虑使用`yield from`。在此例中，将完整使用所提供的迭代器。如果它是一个无限迭代器，那么外部迭代器也将是无限迭代器，并且无限地继续执行`yield from` 语句。

### 等差数列生成器

```python
class ArithmeticProgression:

    def __init__(self, begin, step, end=None):  # end 可选,默认生成无穷数列
        self.begin = begin
        self.step = step
        self.end = end  # None -> 无穷数列

    def __iter__(self):
        result = type(self.begin + self.step)(self.begin)  # 强制转换成加法算式得到的类型
        forever = self.end is None
        index = 0
        while forever or result < self.end:
            yield result  # 生成值
            index += 1
            result = self.begin + self.step * index  # 终止时，该值不会产出.

# 生成器函数形式：
def aritprog_gen(begin, step, end=None):
    result = type(begin + step)(begin)
    forever = end is None
    index = 0
    while forever or result < end:
        yield result
        index += 1
        result = begin + step * index
```



***

参考：

[迭代器](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143178254193589df9c612d2449618ea460e7a672a366000)

https://www.cnblogs.com/huxi/archive/2011/07/01/2095931.html