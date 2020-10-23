## 运算符

从低到高，同一行的运算符具有相同优先级，具有相同优先级的运算符按照从左向右的顺序计算。

| 运算符            | 描述             |
| ----------------- | ---------------- |
| `lambda`          | Lambda表达式     |
| `or`              | 逻辑运算符，或   |
| `and`             | 逻辑运算符，与   |
| `not`             | 逻辑运算符，非   |
| `in` `not in`     | 成员运算符       |
| `is`  `is not`    | 身份运算符       |
| `==` `!=`         | 等于，不等于     |
| `<=` `<` `>` `>=` | 比较             |
| `|`               | 按位或           |
| `^`               | 按位异或         |
| `&`               | 按位与           |
| `>>` `<<`         | 移位             |
| `+` `-`           | 加、减           |
| `*` `/` `%` `//`  | 乘，除，模，整除 |
| `+x` `-x`         | 正负号           |
| `~`               | 按位取反         |
| `**`              | 指数             |
| `[]` `[:]`        | 下标，切片       |

想要改变计算顺序，使用圆括号。所有对象均支持比较操作符。它们拥有相同的优先级（高于布尔操作符）。

***

### x 与 +x

```python
>>> from collections import Counter
>>> ct = Counter('abcdbcaa')
>>> ct
Counter({'a': 3, 'b': 2, 'c': 2, 'd': 1})
>>> ct['c'] = 0
>>> ct['d'] = -2
>>> +ct
Counter({'a': 3, 'b': 2})
```

***

### ..=..+.. 与 ..+=.. 区别

增量赋值运算符 `+=` 和 `*=` 的表现取决于它们的第一个操作对象。简单起见，我们把讨论集中在增量加法`+=`上，但是这些概念对 `*=` 和其他增量运算符来说都是一样的。

`+=` 背后的特殊方法是 `__iadd__` （用于“就地加法”）。但是如果一个类没有实现这个方法的话，Python 会退一步调用 `__add__` 。对于下面这个表达式：

```python
a += b
```

如果 a 实现了 `__iadd__ `方法，就会调用这个方法。同时对可变序列（例如 list、bytearray 和 array.array）来说，a 会就地改动，就像调用了 `a.extend(b)` 一样。但是如果 a 没有实现 `__iadd__` 的话，`a += b` 这个表达式的效果就变得跟 `a = a + b` 一样了：首先计算 `a + b`，得到一个新的对象，然后赋值给 a。也就是说，在这个表达式中，变量名会不会被关联到新的对象，完全取决于这个类型有没有实现`__iadd__` 这个方法。
总体来讲，可变序列一般都实现了 `__iadd__` 方法，因此 `+=` 是就地加法。而不可变序列根本就不支持这个操作，对这个方法的实现也就无从谈起。
上面所说的这些关于 `+=` 的概念也适用于 `*=`，不同的是，后者相对应的是 `__imul__`。

接下来有个小例子，展示的是 `*=` 在可变和不可变序列上的作用：

```python
In [15]: l = [1, 2, 3]

In [16]: id(l)
Out[16]: 1716577124872

In [17]: l *= 2

In [18]: l, id(l)
Out[18]: ([1, 2, 3, 1, 2, 3], 1716577124872)	# 列表 id 未发生变化

In [19]: t = (1, 2, 3)

In [20]: id(t)
Out[20]: 1716564040728

In [21]: t *= 2

In [22]: t, id(t)
Out[22]: ((1, 2, 3, 1, 2, 3), 1716537702056)	# 元组 id 发生变化
```

对不可变序列进行重复拼接操作的话，效率会很低，因为每次都有一个新对象，而解释器需要把原来对象中的元素先复制到新的对象里，然后再追加新的元素。

```python
In [16]: l = [1 , 2, 3]

In [17]: l = l + (4, 5)
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-17-c70fae2ee178> in <module>
----> 1 l = l + (4, 5)

TypeError: can only concatenate list (not "tuple") to list

In [18]: l += (4, 5)

In [19]: l
Out[19]: [1, 2, 3, 4, 5]
```

`+` 需要两侧必须是相同类型的对象，而 `+=` 则没有要求。增量赋值不是一个原子操作。

除此之外，`+=` **不改变**对象标识。对列表 进行`+=` 操作相当于 `extend`，而使用 `=+` 操作是新建了一个列表。

```python
In [20]: l = [1, 2]

In [21]: id(l)
Out[21]: 1612094081352

In [22]: l += [3, 4]

In [23]: id(l)
Out[23]: 1612094081352

In [24]: l = l + [5, 6]

In [25]: id(l)
Out[25]: 1612093971528
```

***

## 布尔操作符---and、or、not

* 没有括号的情况下，**and 的优先级大于 or**；
* x and y，x 为真则是 y，x 为假则是 x；
* x or y，x 为真则是x，x 为假则是 y。

```python
True or True and False	# True
```

```python
print(1 or 2 and 3)			# 1
print((1 or 2) and 3)		# 3
print(0 or 1 and 2 or 0)	# 2
```

闰年判断：

```python
year = int(input('请输入年份: '))
is_leap = (year % 4 == 0 and year % 100 != 0 or
           year % 400 == 0)
print(is_leap)
```

***

## 海象运算符

下例中，`len(a)`会被调用两次。

```python
if len(a) > 10:
    print("List is to long({len(a)} elements, expected <= 10)")
```

或者这样写，避免调用两次`len()`方法，却又多了一次赋值给中间变量的步骤。

```python
n = len(a)
if n > 10:
    print("List is to long({n} elements, expected <= 10)")
```

采用海象运算符。

```python
if (n := len(a)) > 10:
    print(f"List is too long ({n} elements, expected <= 10)")
```

不仅减少了一次调用，还省去了一个赋值中间变量的步骤。简化流程，提高运算速度。

***

**while循环控制**

省去赋值：

```python
n = 0
while n < 3:
   print(n) # 0,1,2
   n += 1

# when converting to walrus operator...
w = 0
while (w := w + 1) < 3:
   print(w) # 1,2
```

节省赋值和判断：

```python
while 1:
    block = f.read(256)
    if block != '':
        process(block)
        
while (block := f.read(256)) != '':
    process(block)
```

***

**简化列表解析**

```python
[f(x) for x in y if f(x)]

[z for x in y if (z := f(x))]
```

***

## 三元运算符

```python
>>> age1 = 17
>>> age2 = 20
```

**1**

```python
condition_is_true if condition else condition_is_false
```

```python
>>> '已成年' if age1 > 18 else '未成年'
'未成年'
>>> '已成年' if age2 > 18 else '未成年'
'已成年'
```

**2**

```python
<condition> and <on_true> or <on_false>
```

```python
>>> age1 >18 and '已成年' or '未成年'
'未成年'
>>> age2 >18 and '已成年' or '未成年'
'已成年'
```

```python
((<condition>) and (<on_true>,) or (<on_false>,))[0]
```

```python
>>> ((age1 > 18) and ('已成年',) or ('未成年',))[0]
'未成年'
>>> ((age1 > 18) and ('已成年',) or ('未成年',))[0]
'已成年'
```

**3**

```python
(<on_false>, <on_true>)[condition]
```

```python
>>> ('未成年', '已成年')[age1 > 18]
'未成年'
>>> ('未成年', '已成年')[age2 > 18]
'已成年'
```

**4**

```python
(lambda: <on_false>, lambda:<on_true>)[<condition>]()
```

```python
>>> (lambda:'未成年', lambda:'已成年')[age1 > 18]()
'未成年'
>>> (lambda:'未成年', lambda:'已成年')[age2 > 18]()
'已成年'
```

**5**

```python
{True: <on_true>, False: <on_false>}[<condition>]
```

```python
>>> {True: '未成年', False: '已成年'}[age1 > 18]
'未成年'
>>> {True: '未成年', False: '已成年'}[age2 > 18]
'已成年'
```

