形如 `__xxx__` 的变量或者函数名，在 Python 中是有特殊用途，统称为"魔术方法"。比如接触最多的`__init__`。

特殊方法也称为魔法方法（Magic method）

> Python methods (including `staticmethod()` and `classmethod()`) are implemented as non-data descriptors.  Accordingly, instances can redefine and override methods.  This allows individual instances to acquire behaviors that differ from other instances of the same class.

## 常见的特殊属性

* `__name__`

  返回系统内置或自定义类的名称字符串，实例没有属性 `__name__`。

* `__bases__`

  返回一个元组，包含的全部基类，实例没有属性 `__bases__`。

* `__class__`

  返回实例或类型的类型，类似等于 `type()`。

* `__dict__`

  返回一个字典，包含实例或类的所有属性。

Python中对象的属性具有 **层次性**，属性在哪个对象上定义，便会出现在哪个对象的`__dict__`中。

***

## 构造方法

当我们调用 `x = SomeClass()` 的时候，`__init__` 并不是第一个被调用的方法。事实上，第一个被调用的是 `__new__`，这个方法才真正地创建了实例。当这个对象的生命周期结束的时候， `__del__` 会被调用。

* `__slots__(self,[...])`

  限制该 `class` 实例能添加的属性。告诉 Python 不要使用字典，而且只给一个固定集合的属性分配空间，为内存减轻负担。

* `__new__(cls,[...)`

  `__new__` 是对象实例化时第一个调用的方法，它只取下 `cls` 参数，并把其他参数传给 `__init__`。`__new__` 很少使用，但是也有它适合的场景，尤其是当类继承自一个像元组或者字符串这样不经常改变的类型的时候。

* `__init__(self,[...])`

  类的初始化方法。它获取任何传给构造器的参数。比如调用 `x = SomeClass(10, ‘foo’)`，`__init__` 就会接到参数 `10` 和 `foo`。`__init__` 在 Python 的类定义中用的最多。

  Note:

  ```python
  In [1]: class Movie(object):
     ...:     def __init__(self, budget):
     ...:         if budget < 0:
     ...:             raise ValueError("Negative value not allowed: %s" % budget)
     ...:         self.budget = budget
     ...:
  
  In [2]: a = Movie(-1)
  ---------------------------------------------------------------------------
  ValueError                                Traceback (most recent call last)
  <ipython-input-2-70e67b61f884> in <module>
  ----> 1 a = Movie(-1)
  
  <ipython-input-1-84298c96c6b2> in __init__(self, budget)
        2     def __init__(self, budget):
        3         if budget < 0:
  ----> 4             raise ValueError("Negative value not allowed: %s" % budget)
        5         self.budget = budget
        6
  
  ValueError: Negative value not allowed: -1
  
  In [3]: Movie.budget = -1
  
  In [4]: Movie.budget
  Out[4]: -1
  ```

  如果直接通过 `Movie.budget` 来赋值，那么不会提示输入错误，因为在这个类只会在`__init__` 方法中捕获错误的数据。

* `__del__(self)`

  `__new__` 和 `__init__` 是对象的构造器， `__del__` 是对象的销毁器。它并非实现了语句 `del x` ，因此该语句不等同于 `x.__del__()`。而是定义了当对象被垃圾回收时的行为。当对象需要在销毁时做一些处理的时候这个方法很有用，比如 `socket` 对象、文件对象。但是需要注意的是，当 Python 解释器退出但对象仍然存活的时候，`__del__` 并不会执行。所以养成一个手工清理的好习惯是很重要的，比如及时关闭连接。

  ```python
  # 文件对象的装饰类，用来保证文件被删除时能够正确关闭
  class FileObject(object):
      __slots__ = ('filepath', 'filename')
      def __init__(self, filepath='~', filename='sample.txt'):
          # 使用读写模式打开 filepath 中的 filename 文件
          self.file = open(join(filepath, filename), 'r+')
  
      def __del__(self):
          self.file.close()
          del self.file
  ```

***

## 对象表示形式

* `__str__(self)`

返回一个对应对象的简洁的字符串表达形式，返回的字符串应该是面向用户的，可读的。python 内置的 `str()` 函数，`print(x)` 时调用。

* `__repr__(self)`

返回的字符串主要是面向解释器。python 内置的 `repr()` 函数，`x` 时调用。

* `__bin__(self)`

返回对象的字节序列表。python 内置的 `bin()` 函数。

* `__bool__(self)`

返回 `True` 或 `False`，python 内置的 `bool()` 函数。

* `__hash__(self)`

python 内置的  `hash()` 函数。

* `__dir__(self)`

返回一个属性列表，python 内置的 `dir()` 函数。

* `__format__(self)`

定义当类的实例用于新式字符串格式化时的行为。python 内置的 `format()` 函数。

### `__str__(self)`与`__repr__(self)`区别

```python
In [1]: x = 1

In [2]: y = 'abc'

In [3]: repr(x), repr(y)
Out[3]: ('1', "'abc'")

In [4]: str(x), str(y)
Out[4]: ('1', 'abc')
```

由`repr(x)`, `str(y)`返回值很明显看出二者的不同之处。

> * `__repr__` goal is to be unambiguous.
> * `__str__` goal is to be readable.
> * Container’s `__str__` uses contained objects’ `__repr__`

```python
In [5]: y == eval(repr(y))
Out[5]: True
```

> It is important to realize the default implementation of `__repr__` for a `str` object can be called as an argument to `eval` and the return value would be a valid `str` object.While the return value of `__str__` is not even a valid statement that can be executed by eval.

因此：

> Implement `__repr__` for any class you implement. This should be second nature. Implement `__str__` if you think it would be useful to have a string version which errs on the side of more readability in favor of more ambiguity.

	

***

## 访问控制

* `__getattribute__(self, name)`

  `__getattribute__` 是属性查找的入口。实例对象访问属性时都会调用 `__getattribute__`，之后才会根据一定的规则在对应 `__dict__` 中查找相应的属性。若没有找到或遇到异常，则会调用 `__getattr__`。

* `__getattr__(self, name)`

  当常规的属性访问无法找到目标属性时，会调用 `__getattr__()` 方法。通过这个方法来定义反馈信息。可以用于捕捉错误的拼写并且给出指引，使用废弃属性时给出警告，以及灵活地处理 `AttributeError`。只有当试图访问不存在的属性时它才会被调用。

* `__setattr__(self, name, value)`

  绑定实例的某个属性（赋值）时，会自动调用 `__setattr__()`方法。

  需要注意的是，重载时，如果其中包含 `self.x = x`或者 `setattr(self, name, value)`，会造成递归调用。为避免这种情况，可以采用以下两种方法：

  * 调用父类方法：

    ```python
    class MyTest:
        def __init__(self, x):
            self.x = x
    
        def __setattr__(self, name, value):
    		super().__setattr__(name, value)
    ```

  * 使用`__dict__`：

    ```python
    class MyTest:
        def __init__(self, x):
            self.x = x
    
        def __setattr__(self, name, value):
            print(name, value)
            self.__dict__[name] = value
    ```

* `__delattr__(self, name)`

  当解绑定一个对象的某个属性时，会调用 `__delattr__()` 方法。无视返回值。

***

## 属性描述符

>  In general, a descriptor is an object attribute with “binding behavior”, one whose attribute access has been overridden by methods in the descriptor protocol.  Those methods are `__get__`, `__set__`, and `__delete__`.  If any of those methods are defined for an object, it is said to be a descriptor.

* `__get__(self, instance, owner)`

  > Called to get the attribute of the owner class (class attribute access) or of an instance of that class (instance attribute access). *owner* is always the owner class, while *instance* is the instance that the attribute was accessed through, or `None` when the attribute is accessed through the *owner*.  This method should return the (computed) attribute value or raise an `AttributeError` exception.

* `__set__(self, instance, value)`

  > Called to set the attribute on an instance *instance* of the owner class to a new value, *value*.

* `__delete__(self, instance)`

  >  Called to delete the attribute on an instance *instance* of the owner class.

***

## 运算符重载

使用 Python 魔法方法的一个巨大优势就是可以构建一个拥有 Python 内置类型行为的对象。这意味着你可以避免使用非标准的、丑陋的方式来表达简单的操作。这样做让代码变得冗长而混乱。不同的类库可能对同一种比较操作采用不同的方法名称，这让使用者需要做很多没有必要的工作。

### 比较运算

方法的返回值不是布尔，返回的是数值。

|||
|:-|:-|
|`__eq__(self, other)`|==|
|`__ne__(self, other)`|!=|
|`__lt__(self, other)`|<|
|`__le__(self, other)`|<=|
|`__gt__(self, other)`|>|
|`__ge__(self, other)`|>=|

单词类，按照单词长度来定义比较行为，返回单词长度。

```python
class Word(str):

    def __new__(self, word):
        # 注意，我们只能使用 __new__ ，因为str是不可变类型
        # 所以我们必须提前初始化它（在实例创建时）
        if ' ' in word:
            print("Value contains spaces. Truncating to first space.")
            word = word[:word.index(' ')]
            # Word现在包含第一个空格前的所有字母
        return str.__new__(self, word)

    def __gt__(self, other):
        return len(self) > len(other)
    def __lt__(self, other):
        return len(self) < len(other)
    def __ge__(self, other):
        return len(self) >= len(other)
    def __le__(self, other):
        return len(self) <= len(other)
```

```python
print(Word('foo')>Word('s'))
```

```python
True
```

当然，也可以使用 `@total_ordering`。

### 一元运算符

|||
|:-|:-|
|`__pos__(self)`|取正|
|`__neg__(self)`|取负|
|`__abs__(self)`|实现内建绝对值函数 `abs()` |
|`__invert__(self)`|取反运算符 `~`|
|`__round__(self，n)`|实现内建函数 `round()`，`n` 是近似小数点的位数|
|`__floor__(self)`|实现 `math.floor()` 函数，即向下取整|
|`__ceil__(self)`|实现 `math.ceil()` 函数，即向上取整|
|`__trunc__(self)`|实现 `math.trunc()` 函数，即向零取整|

### 算数运算符

|||
|:-|:-|
|`__add__(self, other)`|加法，`+`|
|`__sub__(self, other)`|减法，`-`|
|`__mul__(self, other)`|乘法，`*`|
|`__truediv__(self, other)`|除法，`/`|
|`__floordiv__(self, other)`|整除，`//`|
|`__mod__(self, other)`|取余，`%`|
|`__divmod__(self, other)`|实现 `divmod()` 内建函数，`/` 和 `%`|
|`__pow__(self, other)`|幂，`**`|
|`__lshift__(self, other)`|位运算，`<<`|
|`__rshift__(self, other)`|位运算，`>>`|
|`__and__(self, other)`|按位与，`&`|
|`__or__(self, other)`|按位或，`|`|
|`__xor__(self, other)`|按位异或，`^`|

不能重载 `and`和`or` 的逻辑运算。

```python
class Arithmetic(object):

    def __init__(self, num):
        self.data = num
    # +
    def __add__(self, other):
        return self.data + other.data
    # -
    def __sub__(self, other):
        return self.data - other.data
    # *
    def __mul__(self, other):
        return self.data * other.data
    # /
    def __truediv__(self, other):
        return self.data / other.data
    # //
    def __floordiv__(self, other):
        return self.data // other.data
    # %
    def __mod__(self, other):
        return self.data % other.data
    # divmod()
    def __divmod__(self, other):
        return self.data / other.data, self.data % other.data
    # **
    def __pow__(self, other):
        return self.data ** other.data
    # <<
    def __lshift__(self, other):
        return self.data << other.data
    # >>
    def __rshift__(self, other):
        return self.data >> other.data
    # &
    def __and__(self, other):
        return self.data & other.data
    # ^
    def __xor__(self, other):
        return self.data ^ other.data
    # |
    def __or__(self, other):
        return self.data | other.data
```

### 反向算数运算符

反向算数运算符是一种后备机制。假设针对 `some_object` 这个对象，`some_object` 的 `__add__` 方法：

```python
some_object + other
```

如果 `other` 没有定义 `__add__`方法，而是 `other` 定义了 `__radd__` , 那么当 `some_object` 没有实现 `__add__` 方法或者返回 `NotImplemented`时：

```python
other + some_object
```

||
|:-|
|`__radd__(self, other)`|
|`__rsub__(self, other)`|
|`__rmul__(self, other)`|
|`__rtruediv__(self, other)`|
|`__rfloordiv__(self, other)`|
|`__rmod__(self, other)`|
|`__rdivmod__(self, other)`|
|`__rpow__(self, other)`|
|`__rlshift__(self, other)`|
|`__rrshift__(self, other)`|
|`__rand__(self, other)`|
|`__ror__(self, other)`|
|`__rxor__(self, other)`|

***

### 增量赋值运算符

`+=` 就是增量赋值。

||
|:-|
|`__iadd__(self, other)`|
|`__isub__(self, other)`|
|`__imul__(self, other)`|
|`__ifloordiv__(self, other)`|
|`__itruediv__(self, other)`|
|`__imod__(self, other)`|
|`__ipow__(self, other)`|
|`__ilshift__(self, other)`|
|`__irshift__(self, other)`|
|`__iand__(self, other)`|
|`__ior__(self, other)`|
|`__ixor__(self, other)`|

***

### 类型转换操作符

|||
|:-|:-|
|`__int__(self)`|类型转化为int|
|`__long__(self)`|类型转化为long|
|`__float__(self)`|类型转化为float|
|`__complex__(self)`|类型转化为complex|
|`__oct__(self)`|类型转化为八进制数|
|`__hex__(self)`|类型转化为十六进制数|
|`__index__(self)`|类型转化索引位置，切片操作使用|
|`__trunc__(self)`|当调用 `math.trunc(self)` 时调用该方法，返回 `self` 截取到一个整数类型（通常是long类型）的值。|

***

## 自定义序列

想实现一个不可变容器，需要定义 `__len__` 和 `__getitem__`。可变容器的协议除了上面的两个方法之外，还需要定义 `__setitem__` 和 `__delitem__`。最后，如果想让对象可以迭代，需要定义 `__iter__`，这个方法返回一个迭代器。迭代器必须遵守迭代器协议，需要定义 `__iter__` （返回它自己）和 `next` 方法。

* `__len__(self)`

  返回容器的长度，可变和不可变类型都需要实现。

* `__getitem__(self, key)`

  定义通过键来获取值 `self[key]`。这也是可变和不可变容器类型都需要实现的一个方法。它应该在键的类型错误式产生 `TypeError` 异常，同时在没有与键值相匹配的内容时产生 `KeyError` 异常。

* `__setitem__(self, key)`

  定义通过键来设置值 `self[key] = value`。它是可变容器类型必须实现的一个方法，同样应该在合适的时候产生 `KeyError` 和 `TypeError` 异常。

* `__delitem__(self, key)`

  删除一个键值对。

* `__contains__(self, item)`

  某序列是否包含特定的值。定义了使用 `in` 和 `not in` 进行成员测试时类的行为。这个方法不是序列协议的一部分，原因是，如果 `__contains__` 没有定义，`Python` 就会迭代整个序列，如果找到了需要的一项就返回 `True`。

* `__missing__(self ,key)`

  为缺失键提供默认值。

* `__iter__(self, key)`

  遍历某个序列。返回当前容器的一个迭代器。无论何时创建迭代器都将调用 `__iter__` 方法。

* `__next__(self)`

  从迭代器中获取下一个值。无论何时从迭代器中获取下一个值都将调用 `__next__` 方法。

* `__reversed__(self)`

  按逆序创建一个迭代器。返回一个反转之后的序列。它以一个现有序列为参数，并将该序列中所有元素从尾到头以逆序排列生成一个新的迭代器。

```python
class FunctionalList:
    '''一个列表的封装类，实现了一些额外的函数式
    方法，例如head, tail, init, last, drop和take。'''

    def __init__(self, values=None):
        if values is None:
            self.values = []
        else:
            self.values = values

    def __len__(self):
        return len(self.values)

    def __getitem__(self, key):
        # 如果键的类型或值不合法，列表会返回异常
        return self.values[key]

    def __setitem__(self, key, value):
        self.values[key] = value

    def __delitem__(self, key):
        del self.values[key]

    def __iter__(self):
        return iter(self.values)

    def __reversed__(self):
        return reversed(self.values)

    def append(self, value):
        self.values.append(value)

    def head(self):
        # 取得第一个元素
        return self.values[0]

    def tail(self):
        # 取得除第一个元素外的所有元素
        return self.valuse[1:]

    def init(self):
        # 取得除最后一个元素外的所有元素
        return self.values[:-1]

    def last(self):
        # 取得最后一个元素
        return self.values[-1]

    def drop(self, n):
        # 取得除前n个元素外的所有元素
        return self.values[n:]

    def take(self, n):
        # 取得前n个元素
        return self.values[:n]
```

***

## 类型判断

* `__instancecheck__(self, instance)`

  检查某个对象是否是该对象的实例（例如 isinstance(instance, class) ）。

* `__subclasscheck__(self, subclass)`

  检查某个类是否是该类的子类（例如 issubclass(subclass, class) ）。

***

## 可调用

`__call__(self, [args...])`

像调用函数一样“调用”一个实例。本质上这代表了 `x()` 和 `x.__call__()` 是相同的。

***

## 上下文管理

定义当 `with` 声明语句块执行完毕（或终止）时上下文管理器的行为。它可以用来处理异常，进行清理，或者做其他应该在语句块结束之后立刻执行的工作。如果语句块顺利执行， `exception_type`、`exception_value` 和 `traceback` 会是 `None`。否则，你可以选择处理这个异常或者让用户来处理。如果你想处理异常，确保 `__exit__` 在完成工作之后返回 `True`。

* `__enter__(self)`

  在进入 with 语块时进行一些特别操作。

* `__exit__(self, exception_type, exception_value, traceback)`

  在退出 with 语块时进行一些特别操作。`__exit__` 方法将总是被调用，哪怕是在 `with` 语块中引发了例外。实际上，如果引发了例外，该例外信息将会被传递给 `__exit__` 方法。

***

参考：

[Python 魔法方法指南](http://pyzh.readthedocs.io/en/latest/python-magic-methods-guide.html#id2)

[Difference between `__str__ `and `__repr__`?](https://stackoverflow.com/questions/1436703/difference-between-str-and-repr)

[How to use __setattr__ correctly, avoiding infinite recursion](https://stackoverflow.com/questions/17020115/how-to-use-setattr-correctly-avoiding-infinite-recursion)