每个变量都有标识、类型和值。标识可以理解为对象在内存中的地址。

对引用式变量来说，把变量分配给对象更合适。但是，如果说把对象赋值给变量，那么就有点问题。理解 Python 中的赋值语句时，应始终先读右边。对象在右边创建或者获取，然后，左边的变量才会绑定在对象上。一个对象可以绑定多个变量。

***

## 可变与不可变

python 的数据类型其实由可变和不可变开始区分比较好些。而且，python 将所有类型都当做对象来处理。区别在于保存的是真正的对象，还是保存的共享对象的引用。

可变对象：改变变量的值时，变量标识不会改变。对于可变对象的修改相当于对内存地址存储的数据直接进行修改，所以标识不会改变。可变对象包含：`list`、`dict`、`set`。

```cmd
In [1]: l1 = [1, 2, 3]

In [2]: id(l1)
Out[2]: 2210988753992

In [3]: l1.append(4)

In [4]: l1
Out[4]: [1, 2, 3, 4]

In [5]: id(l1)
Out[5]: 2210988753992
```

由此可见， `l1` 值发生改变时，其标识没有变化。也就是说，我们只对 `l1` 的值进行操作。

不可变对象：改变变量的值时，会生成新的对象，变量亦绑定至新的对象，标识发生改变。对于不可变对象的修改相当于更换标识。不可变对象包含：`int`、`float`、`string`、`bool`、`tuple`、`User-defined class`。

```cmd
In [6]: a = 'abc'

In [7]: id(a)
Out[7]: 2210954493320

In [8]: a += 'd'

In [9]: a
Out[9]: 'abcd'

In [10]: id(a)
Out[10]: 2210988897088
```

可以看出 `a` 值由 `abc` 变化为 `abcd`。同时，其标识也发生了变化。也就是说，我们没有对 `a` 的值进行操作，而是对 `a` 进行了重新赋值！

```python
def add_to(num, target=[]):
    target.append(num)
    return target
```

```python
print(add_to(1))
print(add_to(2))
print(add_to(3))
```

```bash
[1]
[1, 2]
[1, 2, 3]
```

在设置函数默认参数时，使用可变变量会产生错误，可以使用 `target=None` 代替。

```python
def add_to(element, target=None):
    if target is None:
        target = []
    target.append(element)
    return target
```

官方文档对二者的解释：

* Immutable

> An object with a fixed value.  Immutable objects include numbers, strings and tuples.  Such an object cannot be altered.  A new object has to be created if a different value has to be stored.  They play an important role in places where a constant hash value is needed, for example as a key in a dictionary.

* Mutable

> Mutable objects can change their value but keep their `id()`.

***

## 元组的相对不可变性

元组和多数 python 集合（列表、字典、集等等）一样，保存的是对象的引用。

元组的不可变性是指 tuple 数据结构的中的引用不可变，与引用的对象无关。元组的值会随着引用的可变对象的变化而变化。元组中的元素的标识是不可变的。元组本身不可变，但是如果里边保存可变对象，那么元组的值可能会发生改变。

```cmd
In [53]: t1 = (1, 2, [3, 4],)

In [54]: id(t1)
Out[54]: 2371468157792

In [55]: id(t1[-1])
Out[55]: 2371468204296

In [56]: t1[-1].append(5)

In [57]: t1
Out[57]: (1, 2, [3, 4, 5])

In [58]: id(t1)
Out[58]: 2371468157792

In [59]: id(t1[-1])
Out[59]: 2371468204296

In [60]: t1 += (6, 7,)

In [61]: id(t1)
Out[61]: 2371467105032

In [62]: t1
Out[62]: (1, 2, [3, 4, 5], 6, 7)

In [63]: id(t1[-1])
Out[63]: 1885692320
```

* 可以对元组 `t1` 中的 list 类型的可变对象进行操作；
* 但是，对元组中的可变对象操作后，无论可变对象的标识还是元组的标识，都未发生变化；
* 只有，当对 `t1` 进行修改的时候，二者才同时发生了变化。

元组不可以修改，只能增加新的部分。可使用切片操作，但是无法使用 `append()`、`pop()` 等方法。

***

## 赋值、浅复制、深复制

赋值是将变量指向某个对象，浅复制是复制出第一层结构，深复制就是完全在新位置重造一个对象。赋值不会新建对象，浅复制和深复制均会新建一个对象。对象包含其他对象的时候，复制操作就会出现问题。

### 赋值与浅复制

```cmd
In [11]: l1 = [1, 2, 3]

In [12]: l2 = list(l1)

In [13]: l3 = l1

In [14]: l4 = l1[:]

In [15]: id(l1)
Out[15]: 2210987691208

In [16]: l2
Out[16]: [1, 2, 3]

In [17]: id(l2)
Out[17]: 2210987723080

In [18]: l3
Out[18]: [1, 2, 3]

In [19]: id(l3)
Out[19]: 2210987691208

In [20]: l4
Out[20]: [1, 2, 3]

In [21]: id(l4)
Out[21]: 2210988842888

In [22]: l1.append(4)

In [23]: l1
Out[23]: [1, 2, 3, 4]

In [24]: id(l1)
Out[24]: 2210987691208

In [25]: l2
Out[25]: [1, 2, 3]

In [26]: id(l2)
Out[26]: 2210987723080

In [27]: l3
Out[27]: [1, 2, 3, 4]

In [28]: id(l3)
Out[28]: 2210987691208

In [29]: l4
Out[29]: [1, 2, 3]

In [30]: id(l4)
Out[30]: 2210988842888
```

* `l2` 与 `l4` 是对 `l1` 的浅复制，各自具有不同的标识；
* `l3` 是对 `l1` 赋值，具有相同的标识。赋值语句是建立对象的引用，而不是复制对象；
* 首先，`l1` 对自身进行操作，值发生改变，但标识没有改变；

复制一个 list 最好并且最简单的方法就是用内建对象 list，当然你也可以用 copy 模块，不过没有直接用 list 来得痛快。

构造方法和 `[:]` 做的均是浅复制。

***

### 浅复制

浅复制是新建一种对象副本，复制了最外层的容器，但副本中的元素是源容器中元素的引用。

```python
l1 = [3, [55, 44], (7, 8, 9)]
l2 = list(l1)
l1.append(100)
l1[1].remove(55)
print('l1:', l1)
print('l2:', l2)
l2[1] += [33, 22]
l2[2] += (10, 11)
print('l1:', l1)
print('l2:', l2)
```

* `l2` 是对 `l1` 的浅复制；
* `l1` 加入 `100` 对 `l2` 没有造成影响；
* 对于可变对象 `list` 来说，`+=` 运算符相当于就地修改列表，所以 `l1`、`l2` 同时受到影响；
* 对于不可变对象 `tuple` 来说，`+=` 运算符创建了一个新的元组，然后重新绑定给变量 `l2`，所以对 `l1` 没有影响。

![示例](https://wx2.sinaimg.cn/mw690/af9e9c30ly1fswonbsrkwj20ds0gs74k.jpg)

![示例](https://wx4.sinaimg.cn/mw690/af9e9c30ly1fswondw5d5j20fu0jlmxj.jpg)

***

```python
In [76]: l1 = [1, 2, 3]

In [77]: l2 = [1, 2, 3]

In [78]: l1[1] = l1

In [79]: l1
Out[79]: [1, [...], 3]

In [80]: l2[1] = l2[:]

In [81]: l2
Out[81]: [1, [1, 2, 3], 3]
```

* `l1[1] = l1` 这个赋值操作将 `l1` 引用的列表对象的第二个元素的引用修改，将第二个元素引用修改为 `l1` 列表对象本身。然而，`l1` 仍然引用原来的对象，只不过其中元素的引用发生了变化。
* `l2[1] = l2[:]`中，`l1[:]` 是一个浅复制，创建了一个新的对象，所以 `l2[1]` 指向是一个新的对象，于是 `l2` 便指向了 `[1, [1, 2, 3], 3]`。

***

### 深复制

深复制：复制对象时，将对象的所有属性一起复制。

以客车乘客中途上下车为例：

```python
class Bus(object):

    def __init__(self, passengers=None):
        if passengers is None:
            self.passengers = []
        else:
            self.passengers = list(passengers)

    def pick(self, name):
        self.passengers.append(name)

    def drop(self, name):
        self.passengers.remove(name)
```

```cmd
In [103]: import copy

In [104]: bus1 = Bus(['a', 'b', 'c', 'd'])

In [105]: bus2 = copy.copy(bus1)

In [106]: bus3 = copy.deepcopy(bus1)

In [107]: id(bus1), id(bus2), id(bus3)
Out[107]: (2371469375416, 2371469375528, 2371469377152)

In [108]: bus1.drop('b')

In [109]: bus2.passengers
Out[109]: ['a', 'c', 'd']

In [110]: id(bus1.passengers), id(bus2.passengers), id(bus3.passengers)
Out[110]: (2371468182280, 2371468182280, 2371468866120)

In [111]: bus3.passengers
Out[111]: ['a', 'b', 'c', 'd']
```

* 创建 `Bus` 的实例 `bus1`，以及浅复制副本 `bus2`，深复制副本 `bus3`
* 审查 `passengers` 的属性后发现，`bus1` 与 `bus2` 共享同一个列表对象，因为 `bus2` 是 `bus1` 的浅复制副本；
* `bus3` 是 `bus1` 的深复制副本，所以 `bus3` 的 `passengers` 属性指向的是另一个列表。

***

## 补充

### 小整数缓存机制

python 将 `[-5, 257)` 范围内的整数进行了缓存。当将范围内的整数进行赋值的时候，并不会重新创建对象，而是使用已经创建好的对象。

```python
In [6]: a = -5

In [7]: b = -5

In [8]: a is b
Out[8]: True

In [9]: c = 257

In [10]: d = 257

In [11]: c is d
Out[11]: False

In [12]: e = 2

In [13]: b += 7

In [14]: b is e
Out[14]: True

In [15]: b is a
Out[15]: False
```

当 `a`、`b` 赋值相同时，其标识亦相同。当 `b` 进行运算，其值与 `e` 相同时，二者的标识也相同。

***

### 可哈希(hashable)和不可变(immutable)

一个对象的哈希值在对象的生命周期里不变，那么这个对象就是可哈希的。这个对象可以与其它对象进行比较, 相等的对象哈希值一定相等。这些数据结构内置了哈希值，每个可哈希的对象都内置了`__hash__`、`__eq__`、`__ne__` 方法，可哈希的对象可以通过哈希值进行对比，也可以作为字典的键值和作为 set 函数的参数。

在 Python 中，对于内置不可变对象，都是可哈希的，而字典和列表等可变对象不是可哈希的。自定义的类的实例默认是可哈希的，但是哈希值不等于 `id()` 值。

> An object is *hashable* if it has a hash value which never changes during its lifetime (it needs a `__hash__()` method), and can be compared to other objects (it needs an `__eq__()` method).  Hashable objects which compare equal must have the same hash value.
>
> Hashability makes an object usable as a dictionary key and a set member, because these data structures use the hash value internally.
>
> All of Python’s immutable built-in objects are hashable; mutable containers (such as lists or dictionaries) are not.  Objects which are instances of user-defined classes are hashable by default.  They all compare unequal (except with themselves), and their hash value is derived from their `id()`.

***

参考：

[glossary](https://docs.python.org/3/glossary.html#term-hashable)