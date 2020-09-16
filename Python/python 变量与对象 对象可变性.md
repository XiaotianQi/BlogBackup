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

> An object with a fixed value.  Immutable objects include numbers, strings and tuples.  Such an object cannot be altered.  A new object has to be created if a different value has to be stored.  They play an important role in places where a constant hash value is needed, for example as a key in a dictionary.

> Mutable objects can change their value but keep their `id()`.

>  Objects have individuality, and multiple names (in multiple scopes) can be bound to the same object. This is known as aliasing in other languages. This is usually not appreciated on a first glance at Python, and can be safely ignored when dealing with immutable basic types (numbers, strings, tuples). However, aliasing has a possibly surprising effect on the semantics of Python code involving mutable objects such as lists, dictionaries, and most other types. This is usually used to the benefit of the program, since aliases behave like pointers in some respects. For example, passing an object is cheap since only a pointer is passed by the implementation; and if a function modifies an object passed as an argument, the caller will see the change — this eliminates the need for two different argument passing mechanisms as in Pascal.

**可变对象旨在共享数据，发挥类似于指针的作用。**

***

## 嵌套列表

```python
lst = [[1, 'c'], [3, 'b'], [2, 'a']]
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/variables%201.png)

```python
lst.sort()	# [[1, 'c'], [2, 'a'], [3, 'b']]
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/variables%202.png)

```python
a = sorted(lst, key=lambda x:x[1])	# [[2, 'a'], [3, 'b'], [1, 'c']]
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/variables%203.png)

使用嵌套列表，尤其子元素仍然是个可变对象的时候，谨慎操作可变对象子元素。

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
Out[58]: 2371468157792		# id 未变化

In [59]: id(t1[-1])
Out[59]: 2371468204296		# id 未变化

In [60]: t1 += (6, 7,)		# 对元组 t1 进行操作

In [61]: id(t1)
Out[61]: 2371467105032		# id 发生变化

In [62]: t1
Out[62]: (1, 2, [3, 4, 5], 6, 7)

In [63]: id(t1[-1])
Out[63]: 1885692320			# id 发生变化
```

* 可以对元组 `t1` 中的 list 类型的可变对象进行操作；
* 但是，对元组中的可变对象操作后，无论可变对象的标识还是元组的标识，都未发生变化；
* 只有，当对 `t1` 进行修改的时候，二者才同时发生了变化。

```python
In [1]: t= (1, 2, [3, 4])

In [2]: t
Out[2]: (1, 2, [3, 4])

In [3]: t[2] += [5, 6]
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-3-1dc9eac03230> in <module>
----> 1 t[2] += [5, 6]

TypeError: 'tuple' object does not support item assignment

In [4]: t
Out[4]: (1, 2, [3, 4, 5, 6])
    
In [5]: t[2].extend([50, 60])

In [6]: t
Out[6]: (1, 2, [3, 4, 5, 6, 50, 60])
    
In [7]: t[2].pop()
Out[7]: 60

In [8]: t
Out[8]: (1, 2, [3, 4, 5, 6, 50])
```

不要把可变对象放在元组里面。

增量赋值不是一个原子操作。它虽然抛出了异常，但还是完成了操作。

***

## + 和 *

通常 + 号两侧的序列由相同类型的数据所构成，在拼接的过程中，两个被操作的序列都不会被修改，Python 会新建一个包含同样类型数据的序列来作为拼接的结果。

如果想要把一个序列复制几份然后再拼接起来，更快捷的做法是把这个序列乘以一个整数。同样，这个操作会产生一个新序列：

```python
l = list(range(3))
print(l * 5)		# [0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2, 0, 1, 2]
print(l)			# [0, 1, 2]

a = 'abc'
print(a + 'abc')	# abcabc
print(a)			# abc
```

`+`和 `*` 都遵循这个规律，不修改原有的操作对象，而是构建一个全新的序列。

### [[] * m] * n

```python
# [[]*m] * n等价于：
row = ['_'] * 3
board = []
for i in range(3):
    board.append(row)	# 每次都追加同一个对象
board[1][1] = 'abc'		# [['_', 'abc', '_'], ['_', 'abc', '_'], ['_', 'abc', '_']]

# [[...]] * n等价于：
board = []
for i in range(3):
    row = ['_'] * 3		# 每次迭代都新建了一个列表
    board.append(row)
board[1][1] = 'abc'		# [['_', '_', '_'], ['_', 'abc', '_'], ['_', '_', '_']]
```

如下例：

```python
lst = [['_'] * 3] * 3	# [['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
lst[0] = [1, 1, 1]		# [[1, 1, 1], ['_', '_', '_'], ['_', '_', '_']]
lst[0][0] = 'a'			# [['a', 1, 1], ['_', '_', '_'], ['_', '_', '_']]
lst[1][1] = 2			# [['a', 1, 1], ['_', 2, '_'], ['_', 2, '_']]
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/list.jpg)

对列表进行`[[]] * 3`操作时，仅仅是将`['_', '_', '_']`这个列表的地址进行了复制，并没有创建新的列表对象，所以容器中虽然有3个列表，但是这3个列表引用了同一个列表对象，这一点可以通过`id`函数检查地址得到证实。

```python
lst = [[0] * 3 for _ in range(5)]
lst[0] = [1, 1, 1]
lst[0][0] = 'a'
lst[1][1] = 2
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/list1.png)

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

## 补充

### 1.编译单元

模块、函数体、类定义、脚本都可以看做一个代码块，代码块作为一个执行单元。

```powershell
>>> (10.1) is (10.1)
True
>>> a = 10.1
>>> b = 10.1
>>> a is b
False
>>> def foo():
...     a = 10.1
...     b = 10.1
...     return a is b
...
>>> foo()
True
>>> def bar():
...   return 10.1
...
>>> def quux():
...   return 10.1
...
>>> bar() is quux()
False
```

在所有题主用的CPython环境里执行下面的代码：

```python
def foo():
  a = 10.1
  b = 10.1
  return a is b

print(foo())
```

而结果总是True。

再试试：

```python
def bar():
  return 10.1

def quux():
  return 10.1

print(bar() is quux())
```

而结果总是False。

这是跟CPython的编译单元以及常量池处理有关。

***

**背景知识**

CPython的代码的“编译单元”是函数——每个函数单独编译，得到的结果是一个PyFunctionObject对象，其中带有字节码、常量池等各种信息。Python的顶层代码也被看作一个函数。
函数之间有嵌套时，外层函数的代码并不包含内层函数的代码，而只是包含创建出内层函数的函数对象（PyFunctionObject）的逻辑。

让我们看看上面的`foo()`函数的字节码（通过`dis.dis(foo)`获取）：

```powershell
>>> import dis
>>> dis.dis(foo)
  2           0 LOAD_CONST               1 (10.1)
              2 STORE_FAST               0 (a)

  3           4 LOAD_CONST               1 (10.1)
              6 STORE_FAST               1 (b)

  4           8 LOAD_FAST                0 (a)
             10 LOAD_FAST                1 (b)
             12 COMPARE_OP               8 (is)
             14 RETURN_VALUE
```

可见a和b的赋值都是由同一个常量池项获得的：LOAD_CONST 1
这条字节码指令的意思是：从当前PyFunctionObject的co_const字段所指向的常量池里，取出下标为1的项，压到操作数栈的栈顶。

每个PyFunctionObject有一个独立的常量池；换句话说，每个PyFunctionObject的co_const字段都指向自己专有的一个常量池对象，而里面的常量池项也是自己专有。

在同一个编译单元（PyFunctionObject）里出现的值相同的常量，只会在常量池里出现一份，一定会对应到运行时的同一个对象。所以在`foo()`的例子里，a和b都从同一个常量池项获取值；
在不同的编译单元里，值相同的常量**不一定**会对应到运行时的同一个对象，要看具体的类型是否自带了某种interning / caching机制（例如Python 2.x系的PyInt / Python 3.x系的PyLong的小整数缓存机制）。

PyFloatObject没有“小数字缓存”机制，所以每次“创建”一个对象（例如PyFloat_FromString() / PyFloat_FromDouble()）都一定会得到一个新的对象——id不同、is运算符比较为False。

***

在CPython的交互式解释器（例如python命令不指定参数时）里，每输入一行可以立即执行的代码，Python就会把它当作一个编译单元来编译到字节码并解释执行；如果输入的代码尚未构成一个完整的单元，例如函数声明或者类声明，则等到获得了完整单元的输入后再当作一个编译单元来处理。

所以当我们在CPython的交互式解释器中分别输入`a = 10.1`、`b = 10.1`这两行时，它们分别被当作一个编译单元处理，其中的常量池没有共享，常量池项也都是各自新创建的，所以会得到`a is b`为False的结果。
而在同一环境里输入`(10.1) is (10.1)`时，这一行被看作一个编译单元，其中两次对10.1这个常量的使用都变成了对同一对象的引用，因而is的结果为True。

当使用python命令去整体执行一个Python源码文件时，其中位于顶层的

```python
a = 10.1
b = 10.1
```

两行代码就会处于同一个编译单元中，因而共享常量池，因而`a is b`就会是True。

而在同一个代码块中，执行时，增加新的变量时，会在代码池中查找其值是否存在。如果存在，那么就会重用，相同赋值的变量会指向同一个对象，而不是新建对象。

***

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

### 5.None

空值是Python里一个特殊的值，用None表示。None不能理解为0，因为0是整数类型，而None是一个特殊的值。None也不是布尔类型，而是NoneType。

在python中是没有NULL，取而代之的是None。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/Data%20Structures%20-2.png)

***

### 6. 替换str中某个元素

因为str是不可变对象，所以使用类似`str[2] = 'abc'` 这样的语句，会报错。不过，str可以使用切片操作：

```python
str = 'abc'
str = str[:1] + 'x' + str[2:]	# 'axc'
```

再或者，转换为list进行处理。

***

参考：

[glossary](https://docs.python.org/3/glossary.html#term-hashable) ，官方文档

[那些年我们踩过的那些坑](https://github.com/jackfrued/Python-100-Days/blob/master/那些年我们踩过的那些坑.md)，骆昊

[关于python中“赋值就是建立一个对象的引用”，大家怎么看？Python一切皆为对象又是什么意思？](https://www.zhihu.com/question/27026782)，蓝色、王楠

[python idle 解释和直接 python script.py 解释有什么差别？](https://www.zhihu.com/question/29089863)，RednaxelaFX

《Fluent Python》