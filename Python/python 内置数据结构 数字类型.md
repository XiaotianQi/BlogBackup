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

## 小整数对象池

python 将 `[-5, 257)` 范围内的整数进行了缓存。当将范围内的整数进行赋值的时候，并不会重新创建对象，而是使用已经创建好的对象。

```python
a = 256
b = 256
c = 257
d = 257
print(a, b, c, d)					# 256 256 257 257
print(id(a), id(b), id(c), id(d))	# 140724638614128 140724638614128 2200075024336 2200075022448
```

仅有在赋值时，二者标识才相同。

```python
c = 257
d = c
print(c, d)			# 257 257
print(id(c), id(d))	# 2859885760240 2859885760240
```

不过，需要注意以下情况：

```python
a = 257
b = 257

def main():
	c = 257
	d = 257
	print(b is c)  # True
	print(a is b)  # False
	print(a is c)  # False
    print(a is e)  # False
```

代码块是程序的一个最小的基本执行单位，一个模块文件、一个函数体、一个类、交互式命令中的单行代码都叫做一个代码块。上面的代码由两个代码块构成，`a = 257`是一个代码块，`main`函数是另外一个代码块。Python内部为了进一步提高性能，凡是在一个代码块中创建的整数对象，如果值不在`small_ints`缓存范围之内，但在同一个代码块中已经存在一个值与其相同的整数对象了，那么就直接引用该对象，否则创建一个新的对象出来，这条规则对不在`small_ints`范围的负数并不适用，对负数值浮点数也不适用，但对非负浮点数和字符串都是适用的，这一点读者可以自行证明。所以 `b is c`返回了`True`，而`a`和`b`不在同一个代码块中，虽然值都是257，但却是两个不同的对象，`is`运算的结果自然是`False`了。
为了验证刚刚的结论，我们可以借用`dis`模块（听名字就知道是进行反汇编的模块）从字节码的角度来看看这段代码。

```python
import dis

dis.dis(main)
```

```bash
  4           0 LOAD_CONST               1 (257)
              2 STORE_FAST               0 (c)

  5           4 LOAD_CONST               1 (257)
              6 STORE_FAST               1 (d)

  6           8 LOAD_GLOBAL              0 (print)
             10 LOAD_FAST                0 (c)
             12 LOAD_FAST                1 (d)
             14 COMPARE_OP               8 (is)
             16 CALL_FUNCTION            1
             18 POP_TOP

  7          20 LOAD_GLOBAL              0 (print)
             22 LOAD_GLOBAL              1 (a)
             24 LOAD_GLOBAL              2 (b)
             26 COMPARE_OP               8 (is)
             28 CALL_FUNCTION            1
             30 POP_TOP

  8          32 LOAD_GLOBAL              0 (print)
             34 LOAD_GLOBAL              1 (a)
             36 LOAD_FAST                0 (c)
             38 COMPARE_OP               8 (is)
             40 CALL_FUNCTION            1
             42 POP_TOP

  9          44 LOAD_GLOBAL              0 (print)
             46 LOAD_GLOBAL              1 (a)
             48 LOAD_GLOBAL              3 (e)
             50 COMPARE_OP               8 (is)
             52 CALL_FUNCTION            1
             54 POP_TOP
             56 LOAD_CONST               0 (None)
             58 RETURN_VALUE                     
```

同理，在终端Python环境下测试，如果在IDE中测试，由于 IDE 的影响，效果会有所不同。这是跟CPython的编译单元以及常量池处理有关的。看如下示例：

```powershell
>>> a = 257; b = 257
>>> a is b
True
```

因为在同一行里，同时给两个变量赋同一值时，解释器知道这个对象已经生成，那么它就会引用到同一个对象。如果分成两行的话，解释器并不知道这个对象已经存在了，就会重新申请内存存放这个对象。

***

参考：

[Built-in Types](https://docs.python.org/3/library/stdtypes.html#set)