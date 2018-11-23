Python3 支持 `int`、`float`、`bool`、`complex`。数字类型是不可变类型，不可迭代。

`bool` 是 `int` 的一个子类型。

复数有实部和虚部，使用一对浮点数表示。可以通过 `z.real` 和 `z.imag` 访问。

```python
In [1]: z = 1.0 + 2.3j

In [2]: z = complex(1.0, 2.3)

In [3]: z.real, z.imag
Out[3]: (1.0, 2.3)
```

Python 中的混合运算、比较规则：

> Python fully supports mixed arithmetic: when a binary arithmetic operator has operands of different numeric types, the operand with the “narrower” type is widened to that of the other, where integer is narrower than floating point, which is narrower than complex.  Comparisons between numbers of mixed type use the same rule.The constructors `int()`, `float()`, and `complex()` can be used to produce numbers of a specific type.

## 数值运算

除了布尔类型外，`int`、`float`、`bool`、`complex`都可以使用的运算为：加、减、乘、除、整除、幂运算和取余。

```python
In [1]: 2 + 3   # 加法
Out[1]: 5

In [2]: 2 - 1   # 减法
Out[2]: 1

In [3]: 2 * 3   # 乘法
Out[3]: 6

In [4]: 4 / 2   # 除法，返回一个浮点数
Out[4]: 2.0

In [5]: 4 // 2  # 除法，返回一个整数
Out[5]: 2

In [6]: 3 / 2
Out[6]: 1.5

In [7]: 3 // 2
Out[7]: 1

In [8]: 5 % 2   # 取余
Out[8]: 1

In [9]: 3 ** 2  # 幂运算
Out[9]: 9
```

数值的除法包含两个运算符：`/` 返回一个浮点数，`//` 返回一个整数。

***

## 常用函数

* `abs(x)`

  返回对象的绝对值

* `divmod(x, y)`

  除法和取余运算结合起来，返回一个包含商和余数的元组

* `pow(x, y)`

  两个参数，幂运算。三个参数，乘方后取余

* `round(x)`

  四舍五入

* `math.floor(x)`

  向下取整

* `math.ceil(x)`

  向上取整

* `math.trunc(x)`

  取整

* `math.sqrt()`

  开方

```python
In [4]: divmod(5, 2)      # 除法、取余
Out[4]: (2, 1)

In [5]: pow(2, 2)         # 幂运算
Out[5]: 4

In [6]: pow(2, 2, 3)      # 幂运算后，取余
Out[6]: 1

In [7]: round(1.5), round(1.3)  # 四舍五入
Out[7]: (2, 1)

In [8]: import math

In [9]: math.floor(1.5)         # 向下取整
Out[9]: 1

In [10]: math.ceil(1.3)         # 向上取整
Out[10]: 2

In [11]: math.trunc(1.23)       # 取整
Out[11]: 1

In [12]: math.sqrt(4)           # 开方
Out[12]: 2.0
```

除了 标准库 `math`，还有 `franctions` 和 `decimal` 相关的数字类型标准库。

***

参考：

[Built-in Types](https://docs.python.org/3/library/stdtypes.html#set)