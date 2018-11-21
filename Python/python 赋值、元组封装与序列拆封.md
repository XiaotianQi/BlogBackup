
## 连续赋值

```cmd
In [86]: x = [1, 2, 3, 4, 5]
    ...: i = 0
    ...: i = x[i] = 3
    ...:
    ...:

In [87]: x
Out[87]: [1, 2, 3, 3, 5]

In [88]: i
Out[88]: 3
```

python 的赋值顺序是 自左往右 依次进行。对象语义指使用系统标准的拷贝方式将一个源对象拷贝成为目标对象后，源对象与目标对象之间依然共享底层资源，对任意一方的改变都将影响到另一方。

**

## 多元赋值

python 的多元赋值原理是元组封装（tuple packing）和序列拆封（sequence unpacking）。

### 元组封装

将多个值放进元组里。

```cmd
In [92]: t = 1, 2, 3

In [93]: t
Out[93]: (1, 2, 3)

In [94]: type(t)
Out[94]: tuple
```

### 序列拆封

元组封装的逆操作就是序列拆封。

> *Sequence unpacking* and works for any sequence on the right-hand side.  Sequence unpacking requires that there are as many variables on the left side of the equals sign as there are elements in the sequence.  Note that multiple assignment is really just a combination of tuple packing and sequence unpacking.

任何的序列(或者是可迭代对象)可以通过一个简单的赋值语句拆封并赋值给多个变量。唯一的前提就是变量的数量必须跟序列元素的数量是一样的。如果一个可迭代对象的元素个数超过变量个数时，会抛出一个 `ValueError`。下面采用 `*args`，拆封出的 `others` 变量永远都是列表类型，无论拆封出的数量是多少。`*args` 语法是专门为拆封不确定个数或任意个数元素的可迭代对象而设计。

```cmd
In [1]: user_tuple = ('Bob', 25, 180, 70)

In [2]: name, age, height, weight = user_tuple

In [3]: print(name, age, height, weight)
Bob 25 180 70

In [4]: name, *others = user_tuple

In [5]: print(name, others)
Bob [25, 180, 70]
```

星号表达式也能用在列表的开始部分。

```python
In [1]: *others, current = [i for i in range(5)]

In [2]: others
Out[2]: [0, 1, 2, 3]

In [3]: current
Out[3]: 4
```

拆封一些元素后丢弃它们，可以使用一个普通的废弃名称，例如：`*_`。

```python
In [4]: record = ('ACME', 50, 123.45, (12, 18, 2012))

In [5]: name, *_, (*_, year) = record

In [6]: name
Out[6]: 'ACME'

In [7]: year
Out[7]: 2012
```

多元赋值变量交换：

```python
In [89]: a = 1
    ...: b = 2
    ...: a, b = b, a+b
    ...:
    ...:

In [90]: a
Out[90]: 2

In [91]: b
Out[91]: 3
```

将 `(b, a+b)` 打包成元组，再序列的分给 `(a, b)` 这个序列。先计算右边表示式的值，再分别赋值给左边的变量。区别于上面的连续赋值。

### 代码块

模块、函数体、类定义、脚本都可以看做一个代码块，代码块作为一个执行单元。

在交互命令行中，每行代码都单独作为一个代码块。例如：cmd、powershell、bash等。

而在同一个代码块中，执行时，增加新的变量时，会在代码池中查找其值是否存在。如果存在，那么就会重用，相同赋值的变量会指向同一个对象，而不是新建对象。

```python
a = 10000
b = 10000
print(a is b)
```

上边的例子会打印 `True`。
