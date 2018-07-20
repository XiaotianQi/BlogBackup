## Map

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

## Filter

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

## Reduce

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