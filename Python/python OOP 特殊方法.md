形如 `__xxx__` 的变量或者函数名，在 Python 中是有特殊用途，统称为特殊方法。比如接触最多的`__init__`。重载这些特殊方法一般会改变对象的内部行为。

特殊方法也称为魔法方法（Magic method）

> Python methods (including `staticmethod()` and `classmethod()`) are implemented as non-data descriptors.  Accordingly, instances can redefine and override methods.  This allows individual instances to acquire behaviors that differ from other instances of the same class.

特殊方法的存在是为了被python解释器调用，我们并不用直接调用它们。也就是说，没有`my_obj.__len__()`这种写法，而应该使用`len(my_obj)`。在执行`len(my_obj)`的时候，如果`my_obj`是个自定义的类对象，那么python会自己调用其中由我们实现的`__len__`方法。当`my_obj`是内置类型的时候，比如list、str等，那么Cpython就会走捷径，`__len__`实际上会直接返回 PyVarObject 里的 ob_size 属性。其中，PyVarObject 是表示内存中长度可变的内置对象的C语言结构体。直接读取这个值比调用一个方法要快的多。

对于`__repr__`、`__abs__`、`__add__`、`__mul__`、`__init__`等这些方法，除了`__init__`方法，其他特殊方法通常不会在这个类的自身代码中使用。及时其他程序想要使用该类的这些方法，也不会直接调用它们。一般只有python解释器会频繁的调用这些特殊方法。

很多时候，特殊方法的调用时隐式的。比如`for i in x:`这个语句，其实背后使用了`iter(x)`，而这个函数的背后是`x.__iter__()`方法。不过，前提是该方法在 x 中被实现了。

通过内置函数（例如：len、iter、str等）来使用特殊方法是最好的选择。这些内置函数不仅会调用特殊方法，还会有提供额外的功能。而且，对于内置的类来说，它们的速度更快。

使用 Python 特殊方法的方便之处就是可以构建一个拥有 Python 内置类型行为的对象。

不要想当然的添加特殊方法，比如`__foo__`之类，因为虽然现在这个名字还没有被python内部使用，但是以后就说不定了。

通过特殊方法，自定义数据类型可以表现得跟内置类型一样，从而写出更具表达力的代码。

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

## 构造方法（实例的创建与销毁）

当我们调用 `x = SomeClass()` 的时候，`__init__` 并不是第一个被调用的方法。事实上，第一个被调用的是 `__new__`，这个方法才真正地创建了实例。当这个对象的生命周期结束的时候， `__del__` 会被调用。

* `__slots__(self,[...])`

  限制该 `class` 实例能添加的属性。

  通常，动态语言允许我们在程序运行时给对象绑定新的属性或方法，当然也可以对已经绑定的属性和方法进行解绑定。但是如果需要限定自定义类型的对象只能绑定某些属性，可以通过在类中定义`__slots__`变量来进行限定。需要注意的是`__slots__`的限定只对当前类的对象生效，对子类并不起任何作用。

  ```python
  class Person(object):
  
      # 限定Person对象只能绑定_name, _age和_gender属性
      __slots__ = ('_name', '_age', '_gender')
  
      def __init__(self, name, age):
          self._name = name
          self._age = age
  
  
  if __name__ == '__main__':
      person = Person('王大锤', 12)
      person._city = 'BJ'	# AttributeError: 'Person' object has no attribute '_city'
  ```

  

* `__new__(cls,[...)`

  `__new__` 是对象实例化时第一个调用的方法，它只取 `cls` 参数，并把其他参数传给 `__init__`。`__new__`  的返回值是一个新的实例对象。

  该方法会涉及到元类编程，可由此进一步了解相关内容。

* `__init__(self,[...])`

  类的初始化方法。它获取任何传给构造器的参数。`__init__` 在 Python 的类定义中用的最多。

  在子类的`__init__`方法中调用超类的构造器。

  **Note**:

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

`__new__` 和 `__init__` 的区别：

`__new__`方法创建实例，`__init__`负责初始化一个实例。

***

## 对象表示形式

* `__str__(self)`

  返回一个对应对象的简洁的字符串表达形式，返回的字符串应该是面向用户的，可读的。python 内置的 `str()` 函数，`print(x)` 时调用。

* `__repr__(self)`

  返回的字符串主要是面向解释器。python 内置的 `repr()` 函数，`repr(x)` 时调用。

* `__bin__(self)`

  返回对象的字节序列表。python 内置的 `bin()` 函数。

* `__format__(self)`

  定义当类的实例用于新式字符串格式化时的行为。python 内置的 `format()` 函数。

`__str__`与`__repr__`区别：

```python
In [1]: x = 1

In [2]: y = 'abc'

In [3]: repr(x), repr(y)
Out[3]: ('1', "'abc'")

In [4]: str(x), str(y)
Out[4]: ('1', 'abc')
```

由`repr(x)`, `str(y)`返回值很明显看出二者的不同之处。

```python
In [5]: y == eval(repr(y))
Out[5]: True
```

> It is important to realize the default implementation of `__repr__` for a `str` object can be called as an argument to `eval` and the return value would be a valid `str` object.`repr` should return A string that can be evaluated to re-create the object.While the return value of `__str__` is not even a valid statement that can be executed by eval.

因此：

> Implement `__repr__` for any class you implement. This should be second nature. Implement `__str__` if you think it would be useful to have a string version which errs on the side of more readability in favor of more ambiguity.

除此之外，如果重载了`__repr__`，那么对`str()`同样有效。但是，反过来，只重载了`__str__`，对`repr()`无效。使用时，应该如此：

* `__repr__` goal is to be unambiguous.
* `__str__` goal is to be human-readable.

***

## 数值转换

| 方法                | 作用                                                         |
| :------------------ | :----------------------------------------------------------- |
| `__int__(self)`     | 类型转化为int                                                |
| `__long__(self)`    | 类型转化为long                                               |
| `__float__(self)`   | 类型转化为float                                              |
| `__complex__(self)` | 类型转化为complex                                            |
| `__bool__(self)`    | 返回 `True` 或 `False`，python 内置的 `bool()` 函数。        |
| `__hash__(self)`    | python 内置的  `hash()` 函数。                               |
| `__oct__(self)`     | 类型转化为八进制数                                           |
| `__hex__(self)`     | 类型转化为十六进制数                                         |
| `__index__(self)`   | 类型转化索引位置，切片操作使用                               |
| `__trunc__(self)`   | 当调用 `math.trunc(self)` 时调用该方法，返回 `self` 截取到一个整数类型（通常是long类型）的值。 |

### 自定布尔值

实际上，任何对象都可以用于需要布尔值的上下文中，比如 `if`、`while`语句，或者`and`、`or`、`not`运算符。

默认情况下，自定义的类的实例总被认为是真，除非对该类的`__bool__`和`__len__`方法重载。`bool(x)`的背后是调用 `x.__bool__()`的结果。如果该方法不存在，那么`bool(x)`会尝试调用`x.__len__()`。若返回0，则`bool(x)`返回 False，否则返回 True。

***

## 属性管理

* `__getattribute__(self, name)`

  `__getattribute__` 是属性查找的入口。实例对象访问属性时都会调用 `__getattribute__`，之后才会根据一定的规则在对应 `__dict__` 中查找相应的属性。若没有找到属性或遇到异常，则会调用 `__getattr__`。

  重载过程中，可能会出现递归调用，如下：

  ```python
  class MyTest:
      def __init__(self, name):
          self.name = name
  
      def __getattribute__(self, name): 
          return self.name
  ```

  使用 `super()`，调用父类的 `__getattribute__` 方法，进行属性查找。

  ```python
  class MyTest:
      def __init__(self, name):
          self.name = name
  
      def __getattribute__(self, item):
          return super().__getattribute__(item)
  ```

* `__getattr__(self, name)`

  当常规的属性访问无法找到目标属性时，会调用 `__getattr__()` 方法。通过这个方法来定义反馈信息。可以用于捕捉错误的拼写并且给出指引，使用废弃属性时给出警告，以及灵活地处理 `AttributeError`。只有当试图访问不存在的属性时它才会被调用。

* `__setattr__(self, name, value)`

  绑定实例的某个属性（赋值）时，会自动调用 `__setattr__()`方法。

  需要注意的是，重载时，如果其中包含 `self.x = x`或者 `setattr(self, name, value)`，会造成递归调用。**重载`__setattr__()`必须谨慎**，为避免这种情况，可以采用以下两种方法：

  * 调用父类方法：

    ```python
    class MyTest:
        def __init__(self, name):
            self.name = name
    
        def __setattr__(self, name, value):
    		super().__setattr__(name, value)
    ```

  * 使用`__dict__`：

    ```python
    class MyTest:
        def __init__(self, name):
            self.name = name
    
        def __setattr__(self, name, value):
            print(name, value)
            self.__dict__[name] = value
    ```


`__setattr__()`负责在`__dict__`中对属性进行注册，所以在重载时必须进行属性注册过程。

```python
class Fun:
    def __init__(self):
        self.name = "Liu"
        self.age = 12
        self.male = True
    
    def __setattr__(self, key, value):
        pass

not_fun = Fun()
print(not_fun.__dict__)		# {}
```

可以通过以下两种方法进行属性注册：

```python
# 方法1
self.__dict__[name] = value
# 方法2
super().__setattr__(name, value)
```

`__init__`中三个属性赋值时，每次都会调用一次`__setattr__`函数。

```python
class Fun:
    def __init__(self):
        self.name = "Liu"
        self.age = 12
        self.male = True
        
    def __setattr__(self, name, value):
        print("*"*50)
        print("setting:{},  with:{}".format(name, value))
        print("current __dict__ : {}".format(self.__dict__))
        super().__setattr__(name, value)

fun = Fun()
```

```shell
**************************************************
setting:name,  with:Liu
current __dict__ : {}
**************************************************
setting:age,  with:12
current __dict__ : {'name': 'Liu'}
**************************************************
setting:male,  with:True
current __dict__ : {'name': 'Liu', 'age': 12}
```

由于每次类实例进行属性赋值时都会调用`__setattr__()`，所以可以重载`__setattr__()`方法，来动态的观察每次实例属性赋值时`__dict__`的变化。

`__setattr__()`方法在类的属性赋值时被调用，并通常需要把属性名和属性值存储到self的`__dict__`字典中。

* `__delattr__(self, name)`

  当解绑定一个对象的某个属性时，会调用 `__delattr__()` 方法。无视返回值。

* `__dir__(self)`

  返回一个属性列表，python 内置的 `dir()` 函数。

在使用 `__getattr__`、`__setattr__`、`__getattribute__` 这三个特殊方式时，经常会遇到递归调用。解决办法就是：通过 `super()` 来触发父类的特殊访，问以避免当前类的属性的无限调用。

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

## 自定义序列

想实现一个**不可变**容器，需要定义 `__len__` 和 `__getitem__`。

**可变**容器的协议除了上面的两个方法之外，还需要定义 `__setitem__` 和 `__delitem__`。最后，如果想让对象可以迭代，需要定义 `__iter__`，这个方法返回一个迭代器。迭代器必须遵守迭代器协议，需要定义 `__iter__` （返回它自己）和 `next` 方法。

* `__len__(self)`

  返回容器的长度，可变和不可变类型都需要实现。

* `__getitem__(self, key)`

  定义通过键来获取值 `self[key]`。这也是可变和不可变容器类型都需要实现的一个方法。它应该在键的类型错误式产生 `TypeError` 异常，同时在没有与键值相匹配的内容时产生 `KeyError` 异常。

  通过重载此方法，便可以实现索引、迭代、切片操作。

* `__setitem__(self, key)`

  定义通过键来设置值 `self[key] = value`。它是可变容器类型必须实现的一个方法，同样应该在合适的时候产生 `KeyError` 和 `TypeError` 异常。

* `__delitem__(self, key)`

  删除一个键值对。

* `__contains__(self, item)`

  某序列是否包含特定的值。定义了使用 `in` 和 `not in` 进行成员测试时类的行为。这个方法不是序列协议的一部分，原因是，如果 `__contains__` 没有定义，`Python` 就会迭代整个序列，如果找到了需要的一项就返回 `True`。

* `__missing__(self ,key)`

  为缺失键提供默认值。


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

## 迭代枚举

- `__iter__(self, key)`

  遍历某个序列。返回当前容器的一个迭代器。无论何时创建迭代器都将调用 `__iter__` 方法。

- `__next__(self)`

  从迭代器中获取下一个值。无论何时从迭代器中获取下一个值都将调用 `__next__` 方法。

- `__reversed__(self)`

  按逆序创建一个迭代器。返回一个反转之后的序列。它以一个现有序列为参数，并将该序列中所有元素从尾到头以逆序排列生成一个新的迭代器。

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

## 运算符

如下例子中，通过`__add__`、`__mul__`，为向量类添加了 + 和 * 两个运算符。值得注意的是，这两个方法的返回值都是新创建的向量对象，被操作的两个向量`self`、`other`还是原封不动，代码里只是读取了他们的值而已。中缀运算符的基本原则就是不改变操作对象，而产出一个新的值。

```python
from math import hypot

class Vector:
    
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y
    
    def __repr__(self):
        return 'Vector({}, {})'.format(self.x, self.y)
    
    def __abs__(self):
        return hypot(self.y, self.y)
    
    def __bool__(self):
        return bool(self.y or self.y)
    
    def __add__(self, other):
        x = self.x + other.x
        y = self.y + other.y
        return Vector(x, y)
    
    def __mul__(self, scalar):
        return Vector(self.x * scalar, self.y * scalar)
```

### 比较运算

方法的返回值不是布尔，返回的是数值。

|方法|作用|
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

|方法|作用|
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

|方法|作用|
|:-|:-|
|`__add__(self, other)`|加法，`+`|
|`__sub__(self, other)`|减法，`-`|
|`__mul__(self, other)`|乘法，`*`|
|`__truediv__(self, other)`|除法，`/`|
|`__floordiv__(self, other)`|整除，`//`|
|`__mod__(self, other)`|取余，`%`|
|`__divmod__(self, other)`|实现 `divmod()` 内建函数，`/` 和 `%`|
|`__pow__(self, other)`|幂，`**`|

不能重载 `and`和`or` 的逻辑运算。

### 反向算数运算符

反向算数运算符是一种后备机制。假设针对 `some_object` 这个对象，`some_object` 的 `__add__` 方法：

```python
some_object + other
```

如果 `other` 没有定义 `__add__`方法，而是 `other` 定义了 `__radd__` , 那么当 `some_object` 没有实现 `__add__` 方法或者返回 `NotImplemented`时：

```python
other + some_object
```

|方法|作用|
|:-|:-|
|`__radd__(self, other)`||
|`__rsub__(self, other)`||
|`__rmul__(self, other)`||
|`__rtruediv__(self, other)`||
|`__rfloordiv__(self, other)`||
|`__rmod__(self, other)`||
|`__rdivmod__(self, other)`||
|`__rpow__(self, other)`||

***

### 增量赋值运算符

`+=` 就是增量赋值。

|方法|作用|
|:-|:-|
|`__iadd__(self, other)`||
|`__isub__(self, other)`||
|`__imul__(self, other)`||
|`__ifloordiv__(self, other)`||
|`__itruediv__(self, other)`||
|`__imod__(self, other)`||
|`__ipow__(self, other)`||

***

### 位运算符

|方法|作用|
|:--|:-|
|`__lshift__(self, other)`|位运算，`<<`|
|`__rshift__(self, other)`|位运算，`>>`|
|`__and__(self, other)`|按位与，`&`|
|`__or__(self, other)`|按位或，`|`|
|`__xor__(self, other)`|按位异或，`^`|

***

### 反向位运算符

|方法|作用|
|:--|:-|
|`__rlshift__(self, other)`||
|`__rrshift__(self, other)`||
|`__rand__(self, other)`||
|`__ror__(self, other)`||
|`__rxor__(self, other)`||

***

### 增量赋值位运算符

|方法|作用|
|:--|:-|
|`__ilshift__(self, other)`||
|`__irshift__(self, other)`||
|`__iand__(self, other)`||
|`__ior__(self, other)`||
|`__ixor__(self, other)`||

***

## 区别

### `__get__`, `__getattr__`, `__getattribute__`, `__getitem__` 

`__get__`, `__getattr__`, `__getattribute__`, `__getitem__` 均是与属性访问有关的特殊方法。

**属性的访问机制**

`obj.x` 属性被访问时：

```text
obj.__dict__['x'] --> type(obj).__dict__['x'] --> type(type(obj)).__dict__['x']
```

一般情况下，属性访问的默认行为是从对象的字典中获取，并当获取不到时会沿着一定的查找链进行查找，直至 `obj` 的基类，顺序符合 C3 算法。若查找链都获取不到属性，则抛出 `AttributeError` 异常。

**区别**

* `__getattr__` 方法是当对象的属性不存在是调用。如果通过正常的机制能找到对象属性的话，不会调用 。

* `__getattribute__ `方法会被**无条件调用**。不管属性存不存在。如果类中还定义了 `__getattr__` ，则只有在 `__getattribute__` 方法中显示调用`__getattr__()` 或者抛出了 `AttributeError` 时，才会调用`__getattr__` 。

* `__getitem__` 方法也是**无条件调用**，这点与 `__getattribute__` 一致。区别在于 `__getitem__` 让类实例允许下标`[]` 运算，可以这样理解：
  * `__getattribute__` 适用于所有 `.` 运算符；
  * `__getitem__` 适用于所有 `[]` 运算符。

* `__get__()`是描述符相关方法，与其他方法的关系不大。如果一个类中定义了 `__get__()`, `__set__()` 或 `__delete__()` 中的任何方法，则这个类的对象称为描述符。

**示例**

```python
class Test1(object):
    a = 'abc'

    def __getattribute__(self, *args, **kwargs):
        print("__getattribute__() is called")
        return object.__getattribute__(self, *args, **kwargs)

    def __getattr__(self, name):
        print("__getattr__() is called ")
        return name + " from getattr"

    def __get__(self, instance, owner):
        print("__get__() is called", instance, owner)
        return self

    def __getitem__(self, item):
        print('__getitem__ call')
        return object.__getattribute__(self, item)

    def foo(self, x):
        print(x)

class Test2(object):
    b = Test1()

if __name__ == '__main__':
    c1 = Test1()
    c2 = Test2()
    print(c1.a)
    print(c1.zzzzzzzz)
    c2.b
    print(c2.b.a)
    print(c1['a'])
```

***

### attributes 与 items 的区别

创建容器类型时，区分属性和项非常重要。

容器都有许多属性(通常是方法，但并不总是方法)，这些属性使它能够以优雅的方式管理其内容。例如，dict有`items`, `values`, `keys`, `iterkeys`等。这些属性都可以使用 `.` 符号进行访问。另一方面，使用`[]`符号访问项。所以，二者没有冲突。

***

参考：

[Python 魔法方法指南](http://pyzh.readthedocs.io/en/latest/python-magic-methods-guide.html#id2)，PyZh

[Difference between `__str__ `and `__repr__`?](https://stackoverflow.com/questions/1436703/difference-between-str-and-repr)，stackoverflow

[How to use __setattr__ correctly, avoiding infinite recursion](https://stackoverflow.com/questions/17020115/how-to-use-setattr-correctly-avoiding-infinite-recursion)，stackoverflow

[Why doesn't Python have a hybrid getattr + __getitem__ built in?](https://stackoverflow.com/questions/6738087/why-doesnt-python-have-a-hybrid-getattr-getitem-built-in)，stackoverflow

[python魔法方法之`__setattr__()`](https://zhuanlan.zhihu.com/p/101004827?from_voters_page=true)，机器学习入坑者