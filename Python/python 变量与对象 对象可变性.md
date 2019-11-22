每个变量都有标识、类型和值。标识可以理解为对象在内存中的地址。对象也是通过它所属的类型构造出来。

对引用式变量来说，把变量分配给对象更合适。但是，如果说把对象赋值给变量，那么就有点问题。理解 Python 中的赋值语句时，应始终先读右边。对象在右边创建或者获取，然后，左边的变量才会绑定在对象上。一个对象可以绑定多个变量。先生成对象，然后变量获得对象的引用。Python中的一切都是对象，变量是对象的引用。

在Python中，变量本身没有数据类型的概念，通常所说的“变量类型”是变量所引用的对象的类型，或者说是变量的值的类型。

***

## 变量

变量一般有三种风格：

* 引用式：变量是对象的名字，在运行时可以绑定到任意对象上，比如python。
* 值式：变量是存储位置的名字，在编译时绑定已经完成，运行时不可以改变，比如c。
* 混合式：某些类型是值式的，某些类型是引用式的，比如c#的struct和class。

c++比较特殊，基本上可以说是值式的，但是由于有别名存在，所以也有些引用式的特点。

引用式和值式的区别就是在赋值操作上，引用式的赋值是变量名的重新绑定，而值式的赋值是对象的拷贝。

Python 是一种动态类型化语言，所以无需声明变量类型。事实上在单个程序中，变量的类型是可以改变（多次）的。python的特性 ，类型属于对象，变量无类型，对象有类型。

一种直观演示动态类型化工作方式的简单方法是：创建一个名为 PyObject 的基类，并且让 Python 中的所有其他对象类型都继承它。在该模型中，我们创建的所有变量都将引用该类层次结构中创建的对象。PyObject 类创建并分配给变量的子类就是对象类型。

***

## 对象：可变与不可变

python 的数据类型其实由可变和不可变开始区分比较好些。而且，python 将所有类型都当做对象来处理。区别在于保存的是真正的对象，还是保存的对象的引用。

* 不可变对象：如果值不变，其标识不变。因此，如果其值改变，标识也会随之改变。

* 可变对象：如果值发生变化，其标识不变。

在Python中，对于不可变对象，调用自身的任意方法，并不会改变对象自身的内容，这些方法会创建新的对象并返回，保证了不可变对象本身是永远不可变。对不可变对象进行运算操作其实就是创建新的对象，然后将原先的变量名绑定到新的对象上。

可变对象：改变变量的值时，变量标识不会改变。对于可变对象的修改相当于对内存地址存储的数据直接进行修改，所以标识不会改变。可变对象包含：`list`、`dict`、`set`。

```cmd
a = [0, 1, 2]
print(id(a), a)	# 2200075713544 [0, 1, 2]

a[0] = -1
print(id(a), a)	# 2200075713544 [-1, 1, 2]

a += [4, 5]
print(id(a), a)	# 2200075713544 [-1, 1, 2, 4, 5]

a.append(6)
print(id(a), a)	# 2200075713544 [-1, 1, 2, 4, 5, 6]
```

由此可见， `l1` 值发生改变时，其标识没有变化。也就是说，我们只对 `l1` 的值进行操作。

不可变对象：改变对象的值时，会生成新的对象，变量亦绑定至新的对象，标识发生改变。对于不可变对象的修改相当于更换标识。不可变对象包含：`int`、`float`、`string`、`bool`、`tuple`、`User-defined class`。

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

## 嵌套列表

```python
lst = [[0] * 3] * 5
lst[0] = [1, 1, 1]
lst[0][0] = 'a'
lst[1][1] = 2
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/list.png)

对列表进行`[[0] * 3] * 5`操作时，仅仅是将`[0, 0, 0]`这个列表的地址进行了复制，并没有创建新的列表对象，所以容器中虽然有5个元素，但是这5个元素引用了同一个列表对象，这一点可以通过`id`函数检查`scores[0]`和`scores[1]`的地址得到证实。

```python
lst = [[0] * 3 for _ in range(5)]
lst[0] = [1, 1, 1]
lst[0][0] = 'a'
lst[1][1] = 2
```



![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/list1.png)

```python
scores = [[]] * 5
lst = [[0] * 3 for _ in range(5)]
lst[0] = [1, 1, 1]
lst[0][0] = 'a'
# lst[1][1] = 2	# 报错
```



![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/list2.png)

***

### 1.代码块

模块、函数体、类定义、脚本都可以看做一个代码块，代码块作为一个执行单元。

在交互命令行中，每行代码都单独作为一个代码块。例如：cmd、powershell、bash等。

而在同一个代码块中，执行时，增加新的变量时，会在代码池中查找其值是否存在。如果存在，那么就会重用，相同赋值的变量会指向同一个对象，而不是新建对象。

```python
a = 10000
b = 10000
print(a is b)
```

上边的例子会打印 `True`。

### 2.可哈希(hashable)和不可变(immutable)

一个对象的哈希值在对象的生命周期里不变，那么这个对象就是可哈希的。这个对象可以与其它对象进行比较, 相等的对象哈希值一定相等。这些数据结构内置了哈希值，每个可哈希的对象都内置了`__hash__`、`__eq__`、`__ne__` 方法，可哈希的对象可以通过哈希值进行对比，也可以作为字典的键值和作为 set 函数的参数。

在 Python 中，对于内置不可变对象，都是可哈希的，而字典和列表等可变对象不是可哈希的。自定义的类的实例默认是可哈希的，但是哈希值不等于 `id()` 值。

> An object is *hashable* if it has a hash value which never changes during its lifetime (it needs a `__hash__()` method), and can be compared to other objects (it needs an `__eq__()` method).  Hashable objects which compare equal must have the same hash value.
>
> Hashability makes an object usable as a dictionary key and a set member, because these data structures use the hash value internally.
>
> All of Python’s immutable built-in objects are hashable; mutable containers (such as lists or dictionaries) are not.  Objects which are instances of user-defined classes are hashable by default.  They all compare unequal (except with themselves), and their hash value is derived from their `id()`.

***

### 3.`is` 和 `==` 的区别

`is` will return `True` if two variables point to the same object, `==` if the objects referred to by the variables are equal.

```python
In [3]: a == b
Out[3]: True

In [4]: a is b
Out[4]: False

In [5]: id(a), id(b)
Out[5]: (1889956815560, 1889956951240)
```

***

### 4.`..=..+..` 与 `..+=..` 区别

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

`+` 需要两侧必须是相同类型的对象，而 `+=` 则没有要求。

除此之外，`+=` **不改变**对象标识。

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

### 5.None

空值是Python里一个特殊的值，用None表示。None不能理解为0，因为0是整数类型，而None是一个特殊的值。None也不是布尔类型，而是NoneType。

***

参考：

[glossary](https://docs.python.org/3/glossary.html#term-hashable) ，官方文档

[那些年我们踩过的那些坑](https://github.com/jackfrued/Python-100-Days/blob/master/那些年我们踩过的那些坑.md)，骆昊

[关于python中“赋值就是建立一个对象的引用”，大家怎么看？Python一切皆为对象又是什么意思？](https://www.zhihu.com/question/27026782)，蓝色、王楠