高阶函数英文叫Higher-order function。在数学和计算机科学中，高阶函数是至少满足下列一个条件的函数：

* 接受一个或多个函数作为输入
* 输出一个函数

在数学中它们也叫做算子（运算符）或泛函。微积分中的导数就是常见的例子，因为它映射一个函数到另一个函数。 

一个最简单的高阶函数：

```python
def add(x, y, f):
    return f(x) + f(y)
```

***

## map

```text
map(function, iterable, ...)

返回迭代器。
会根据提供的函数对指定序列做映射。第一个参数 function 以参数序列中的每一个元素调用 function 函数，返回包含每次 function 函数返回值的新列表。
当传入多个可迭代对象时，函数的参数必须提供足够多的参数，保证每个可迭代对象同一索引的值均能正确传入函数。
```

```bash
In [102]: list(map(lambda x, y: x + y, range(1,10,2), range(2,11,2)))
Out[102]: [3, 7, 11, 15, 19]
```

将元组中的字符串，转化为数字：

```python
T1 = (('13', '17', '18', '21', '32'),
      ('07', '11', '13', '14', '28'),
      ('01', '05', '06', '08', '15', '16')
      )
print([list(map(int, x)) for x in T1])
```

同理，可迭代对象也可以是一个函数列表：

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

***

## reduce

在Python 3里，`reduce()` 函数已经被从全局名字空间里移除了，它现在被放置在`fucntools` 模块里

```text
functools.reduce(function, iterable[, initializer])
函数会对参数序列中元素进行累积。
```

```bash
In [30]: reduce(lambda x,y:x+y, range(5))
Out[30]: 10
```

与 `map` 函数结合使用：

```python
from functools import reduce


def char2num(s):
    digits = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}
    return digits[s]

r = reduce(lambda x, y:x*10 + y, map(char2num, '123'))
print(r)	# 123
```

整理为一个函数：

```python
from functools import reduce


def str2nums(s):
    def char2num(s):
        digits = {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}
        return digits[s]
    def fn(x, y):
        return x*10 +y
    return reduce(fn, map(char2num, s))

r = str2nums('123')
print(r)	# 123
```

***

## filter

```text
filter(function, iterable)

返回一个迭代器对象。
用于过滤序列，过滤掉不符合条件的元素，返回一个迭代器对象。如果要转换为列表，可以使用 list()来转换。
该接收两个参数，第一个为函数，第二个为序列，序列的每个元素作为参数传递给函数进行判，根据返回值是True还是False决定保留还是丢弃该元素，最后将返回 True 的元素放到新列表中。
```

1~100中平方根是整数的数

```python
import math
def is_sqr(x):
    return math.sqrt(x) % 1 == 0
 
tmplist = filter(is_sqr, range(1, 101))
newlist = list(tmplist)
```

```bash
In [78]: l = ['', False, 'I', {}]

In [79]: list(filter(None, l))		# 传入 None
Out[79]: ['I']
```

把一个序列中的空字符串删掉：

```pyton
def not_empty(s):
    return s and s.strip()

list(filter(not_empty, ['A', '', 'B', None, 'C', '  ']))
```

获得质数（素数）：

```python
def _not_divisible(n):
    return lambda x: x % n > 0

def primes():
    yield 2
    it = iter(range(3, 50, 2))
    while True:
        n = next(it) # 返回序列的第一个数
        yield n
        it = filter(_not_divisible(n), it) # 构造新序列

for n in primes():
    if n < 1000:
        print(n)
    else:
        break
```

***

## sorted

`sorted(iterable, key=None, reverse=False)`

返回重新排序的列表。

```bash
In [116]: sorted([5, 2, 3, 1, 4])
Out[116]: [1, 2, 3, 4, 5]

In [117]: sorted('dfgAabB')
Out[117]: ['A', 'B', 'a', 'b', 'd', 'f', 'g']

In [126]: sorted('dfgAabB', key=lambda x: x.lower())
Out[126]: ['A', 'a', 'b', 'B', 'd', 'f', 'g']

In [72]: sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)
Out[72]: ['about', 'bob', 'Credit', 'Zoo']
```

**sort 与 sorted 区别：**

sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。

list 的 sort 方法返回的是对已经存在的列表进行操作，而内建函数 sorted 方法返回的是一个新的 list，而不是在原来的基础上进行的操作。

***

参考：

[高阶函数](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014317849054170d563b13f0fa4ce6ba1cd86e18103f28000)，廖雪峰

[Python的高阶函数小结](https://www.cnblogs.com/ArsenalfanInECNU/p/9620931.html)，青山牧云人