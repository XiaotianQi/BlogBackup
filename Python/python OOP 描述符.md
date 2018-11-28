## Descriptor

> In general, a descriptor is an object attribute with “binding behavior”, one whose attribute access has been overridden by methods in the descriptor protocol.  Those methods are `__get__`, `__set__`, and `__delete__`.  If any of those methods are defined for an object, it is said to be a descriptor.
>
> They are the mechanism behind properties, methods, static methods, class methods, and `super()`.

使用 Descriptor 的便捷之处：

把函数的调用伪装成对属性的访问。

> For the users of a class, properties are syntactically identical to ordinary attributes.You can start with the simplest implementation imaginable, and  you are free to later migrate to a property version without having to change the interface! So properties are not just a replacement for getters and setter! 
>
> The Pythonic way to introduce attributes is to make them public.

> The default behavior for attribute access is to get, set, or delete the attribute from an object’s dictionary.

> ```
> descr.__get__(self, obj, type=None) -> value
> descr.__set__(self, obj, value) -> None
> descr.__delete__(self, obj) -> None
> ```

如果一个对象同时定义了`__get__`和`__set__`，则这个描述符被称为 `data descriptor`。

如果一个对象只定义了`__get__`方法，则这个描述符被称为`non-data descriptor`。

***

## 优先级

`obj.x` 属性被访问时：

```text
obj.__dict__['x'] --> type(obj).__dict__['x'] --> type(type(obj)).__dict__['x']
```

直至 `obj` 的基类，顺序符合 C3 算法。Python中对象的属性具有 **层次性**，属性在哪个对象上定义，便会出现在哪个对象的`__dict__`中。

具体步骤：

* 查看当前是否是描述符；
* 若是，则调用 `__set__` 或 `__get__`，进行属性读写；
* 若不是，则采用默认调用，进行属性读写 。

即，descriptor 会改变默认的属性读写方式。

***

```python
class DataDes:
    def __get__(self, instance, owner):
        print('DataDes.__get__')
    def __set__(self, instance, value):
        print('DataDes.__set__', value)
 

class DataDes_noget:
    def __set__(self, instance, value):
        print('DataDes_noget.__set__', value)
       
    
class DataDes_noset:
    def __get__(self, instance, owner):
        print('non_DataDes.__get__')
        
        
class Manage:
    datades = DataDes()
    data_noget = DataDes_noget()
    data_noset = DataDes_noset()
```

```python
In :a = Manage()

In : a.__dict__
Out: {}
    
In : a.datades = 1
DataDes.__set__ 1

In : a.__dict__
Out: {}                 # 没有调用实例__dict__
```
* 赋值时，三者的不同：

```python
In [15]: a.datades = 1			# 调用 __set__
DataDes.__set__ 1

In [16]: a.data_noget = 2		# 调用 __set__
DataDes_noget.__set__ 2

In [17]: a.data_noset = 3		# 默认调用

In [18]: a.__dict__				
Out[18]: {'data_noset': 3}
```

* 取值是，三者的不同：

```python
In [19]: a.datades				# 调用 __get__
DataDes.__get__

In [20]: a.data_noget			# 重载默认调用
Out[20]: <__main__.DataDes_noget at 0x1984c586470>

In [21]: a.data_noset			# 默认调用
Out[21]: 3
```

不过，使用实例的字典直接赋值和访问，会绕过描述符。

```python
In [26]: a.__dict__['datades'] = 1

In [27]: a.__dict__['data_noget'] = 2

In [28]: a.__dict__['data_noset'] = 3

In [29]: a.__dict__
Out[29]: {'data_noset': 3, 'datades': 1, 'data_noget': 2}
    
In [33]: a.__dict__['datades']
Out[33]: 1

In [34]: a.__dict__['data_noget']
Out[34]: 2

In [35]: a.__dict__['data_noset']
Out[35]: 3
```

> Data and non-data descriptors differ in how overrides are calculated with  respect to entries in an instance’s dictionary.  If an instance’s dictionary has an entry with the same name as a data descriptor, the data descriptor takes precedence.  If an instance’s dictionary has an entry with the same name as a non-data descriptor, the dictionary entry takes precedence.

对属性进行访问的时候需要几行打交道的基本上包含这几个对象：

1. data descriptor
2. non-data descriptor
3. 实例的字典
4. 内置的`__getattr__`函数

> 优先级顺序是：
>
> ```text
> data descriptor  >>  instance's dict  >>  non-data descriptor  >>   __getattr__()
> ```

>The details of invocation depend on whether obj is an object or a class.
> For objects, the machinery is in `object.__getattribute__()` which transforms `b.x` into `type(b).__dict__['x'].__get__(b, type(b))`. The implementation works through a precedence chain that gives data descriptors priority over instance variables, instance variables priority over non-data descriptors, and assigns lowest priority to `__getattr__()` if provided. The full C implementation can be found in `PyObject_GenericGetAttr()` in `Objects/object.c`.
> For classes, the machinery is in `type.__getattribute__()` which transforms `B.x` into `B.__dict__['x'].__get__(None, B)`. 
>
>The important points to remember are:
>
>- descriptors are invoked by the`__getattribute__()` method
>- overriding`__getattribute__()` prevents automatic descriptor calls
>- `__getattribute__()` and `type.__getattribute__()` make different calls to `__get__()`.
>- data descriptors always override instance dictionaries.
>- non-data descriptors may be overridden by instance dictionaries.

> Classes can turn-off descriptor invocation by overriding `__getattribute__()`.

PS:

当使用 `obj.name` 或 `obj.name = value`时

```python
object.__setattribute__(self,name,value)
object.__getattribute__(self,name)
```

创建一个只读数据描述符：

> To make a read-only data descriptor, define both `__get__` and `__set__` with the `__set__` raising an `AttributeError` when called.

```python
def __set__(self, instance, value):
	raise AttributeError(
	"'%s' is not modifiable" % self.value
	)
```

***

## 示例

检查输入参数的正负。

```python
from weakref import WeakKeyDictionary
 
class NonNegative(object):
    """A descriptor that forbids negative values"""
    def __init__(self, default):
        self.default = default
        self.data = WeakKeyDictionary()
 
    def __get__(self, instance, owner):
        # we get here when someone calls x.d, and d is a NonNegative instance
        # instance = x
        # owner = type(x)
        print('Get:instance--{};owner--{}'.format(instance, owner))
        return self.data.get(instance, self.default)
 
    def __set__(self, instance, value):
        # we get here when someone calls x.d = val, and d is a NonNegative instance
        # instance = x
        # value = val
        print('Set:instance--{};value--{}'.format(instance, value))
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self.data[instance] = value
 
class Movie(object):
 
    # always put descriptors at the class-level
    rating = NonNegative(0)
    runtime = NonNegative(0)
    budget = NonNegative(0)
    gross = NonNegative(0)
 
    def __init__(self, title, rating, runtime, budget, gross):
        self.title = title
        self.rating = rating
        self.runtime = runtime
        self.budget = budget
        self.gross = gross
 
    def profit(self):
        return self.gross - self.budget
```

```python
if __name__ == "__main__":
    m = Movie('Casablanca', 97, 102, 964000, 1300000)
    print(m.budget)  # calls Movie.budget.__get__(m, Movie)
    m.rating = 100   # calls Movie.rating.__set__(m, 100)
    try:
        m.rating = -1   # calls Movie.rating.__set__(m, -100)
    except ValueError:
        print("Woops, negative value")
```

```text
Set:instance--<__main__.Movie object at 0x0000026325AE8160>;value--97
Set:instance--<__main__.Movie object at 0x0000026325AE8160>;value--102
Set:instance--<__main__.Movie object at 0x0000026325AE8160>;value--964000
Set:instance--<__main__.Movie object at 0x0000026325AE8160>;value--1300000
Get:instance--<__main__.Movie object at 0x0000026325AE8160>;owner--<class '__main__.Movie'>
964000
Set:instance--<__main__.Movie object at 0x0000026325AE8160>;value--100
Set:instance--<__main__.Movie object at 0x0000026325AE8160>;value---1
Woops, negative value
```

### 1. 在类的层次上（class level)使用描述符

为了让描述符能够正常工作，它们必须定义在类的层次上。否则，Python 无法**自动**调用`__get__`和`__set__`方法。

```python
class Broken(object):
    y = NonNegative(5)
    def __init__(self):
        self.x = NonNegative(0)  # NOT a good descriptor
 
b = Broken()
print("X is %s, Y is %s" % (b.x, b.y))

Get:instance--<__main__.Broken object at 0x0000017A5C89BA20>;owner--<class '__main__.Broken'>
X is <__main__.NonNegative object at 0x0000017A5C89BA58>, Y is 5
```

### 2. 确保实例属性只属于实例（keep instance-level data instance-specific）

```python
class BrokenNonNegative(object):
    def __init__(self, default):
        self.value = default
        
    def __get__(self, instance, owner):
        return self.value
    
    def __set__(self, instance, value):
        if value < 0:
            raise ValueError("Negative value not allowed: %s" % value)
        self.value = value
	
class Foo(object):
    bar = BrokenNonNegative(5) 

f = Foo()
g = Foo()

f.bar = 10
print(g.bar)	# 10
```

`bar`是类变量，因此所有的 `Foo`实例都共享同一个`bar`。

### 3. 注意：使用描述符的不可哈希类（unhashable descriptor owners）

```python
class MoProblems(list):  # you can't use lists as dictionary keys
    x = NonNegative(5)
 
m = MoProblems()
print(m.x)				# TypeError: unhashable type: 'MoProblems'
```

 `NonNegative` 类使用字典保存实例的数据，字典的 `key` 要求必须是可哈希。然而，`MoProblems` 类的实例是不可哈希的，不能作为字典的 `key`，所以抛出异常。

规避这个问题最好的方法是：给描述符添加标签。

```python
class Descriptor(object):
 
    def __init__(self, label):
        self.label = label
 
    def __get__(self, instance, owner):
        print('__get__', instance, owner)
        return instance.__dict__.get(self.label)
 
    def __set__(self, instance, value):
        print('__set__')
        instance.__dict__[self.label] = value
 
class Foo(list):
    x = Descriptor('x')
    y = Descriptor('y')

f = Foo()
f.x = 5
f.y = 10
print(f.x)
print(f.y)
print(f.__dict__)
```

### 4. 在元类中使用带标签的描述符

```python
class Descriptor(object):
    def __init__(self):
        #notice we aren't setting the label here
        self.label = None
 
    def __get__(self, instance, owner):
        print('__get__. Label = %s' % self.label)
        return instance.__dict__.get(self.label, None)
 
    def __set__(self, instance, value):
        print('__set__')
        instance.__dict__[self.label] = value
 
class DescriptorOwnerMetaClass(type):
    def __new__(cls, name, bases, attrs):
        # find all descriptors, auto-set their labels
        for n, v in attrs.items():
            if isinstance(v, Descriptor):
                v.label = n
        return super().__new__(cls, name, bases, attrs)
 
class Foo(metaclass=DescriptorOwnerMetaClass):
    x = Descriptor()
```

### 5. 访问描述符

描述符是一个用来回调属性的很好的方法。比如一个类的某个属性的状态发生变化时，立刻通知我们。

```python
from weakref import WeakKeyDictionary

class CallbackProperty(object):
    """A property that will alert observers when upon updates"""
    def __init__(self, default=None):
        self.data = WeakKeyDictionary()
        self.default = default
        self.callbacks = WeakKeyDictionary()
 
    def __get__(self, instance, owner):
        # 通过类访问描述器的时候，instance为None
        if instance is None:
            return self        
        return self.data.get(instance, self.default)
 
    def __set__(self, instance, value):
        # alert callback function of new value
        #for callback in self.callbacks.get(instance, []):
            #callback(value)
        self.callbacks[instance][0](value)
        self.data[instance] = value
 
    def add_callback(self, instance, callback):
        """Add a new function to call everytime the descriptor within instance updates"""
        if instance not in self.callbacks:
            self.callbacks[instance] = []
        self.callbacks[instance].append(callback)
 
class BankAccount(object):
    balance = CallbackProperty(0)			# 仅访问 __init__
 
def low_balance_warning(value):
    if value < 100:
        print("You are now poor")
 
ba = BankAccount()
BankAccount.balance.add_callback(ba, low_balance_warning) # 仅访问 __get__、add_callback
ba.balance = 5000		# 第一次访问__set__
print("Balance is %s" % ba.balance)
ba.balance = 99
print("Balance is %s" % ba.balance)
```

声明一个回调函数，用来响应类中某个属性的状态变化，而且无需修改这个类的代码。现在，要做的就是调用 `ba.balance.add_callback(ba, low_balance_warning)`，以使得每次 `balance` 变化时，`low_balance_warning` 都会被调用。

代码运行时，描述符总是会先调用 `__get__` 方法。那么，当从**类层级**访问时，`__get__`方法的参数`instance`是`None`。此时，即可通过 `add_callback` 方法，给 `__set__` 方法添加属性判断。

当一个类(BankAccount)中存在一个描述符属性(balance)，当这个属性被访问时，会自动调用描述符的`__get__`和`__set__`方法：

* 当使用类名访问描述符时(BankAccount.balance，类层级) , `__get__`方法返回描述器本身
* 当使用对象访问描述符时(ba.name，实例层级), `__get__`方法会返回自定义的值（data[instance]）

***

## 描述符相关用法

### 1.`@property`、`property()`

`property` 让我们将自定义的代码与变量的访问/修改联系在了一起，同时保持一个简单的访问属性的接口。

除此之外，在 python 中，所有的属性都是公共的。即使用双下划线声明的属性，也无法阻止将其获取并使用。

使用 `property` 不仅可以便捷的设置属性可读、可读写，从而隐藏并保护其不受其他代码影响，还可以在不需要更改客户代码的情况下，对属性进行修改，获得更好的兼容性。

不过，缺点是，不能重复使用，每次只能创建一个属性。可以看做是修饰符的简化版。

```python
class Celsius:
    def __init__(self, temperature = 0):
        self.temperature = temperature

    def to_fahrenheit(self):
        return self.temperature*1.8 + 32

    @property
    def temperature(self):
        """I'm the 'temperature' property."""
        print("Getting value")
        return self._temperature
    
    @temperature.setter
    def temperature(self, value):
        if value < -273:
            raise ValueError("Temperature below -273 is not possible")
        print("Setting value")
        self._temperature = value
```

或者如下：

```python
class Celsius:
    def __init__(self, temperature = 0):
        self.temperature = temperature

    def to_fahrenheit(self):
        return self.temperature*1.8 + 32

    def get_temperature(self):   # 相对于 @property，需要实现 getter 方法
        print("Getting value")
        return self._temperature

    def set_temperature(self, value):  # 仍然可以 obj.set_temperatur() 进行赋值
        if value < -273:
            raise ValueError("Temperature below -273 is not possible")
        print("Setting value")
        self._temperature = value

    temperature = property(get_temperature,set_temperature, "I'm the 'temperature' property.")
```

相交而言，使用 `@property` 代码更加简洁。

```python
In [107]: t = Celsius()
Setting value

In [108]: t.temperature = 20
Setting value

In [109]: t.temperature
Getting value
Out[109]: 20

In [110]: t.to_fahrenheit()
Getting value
Out[110]: 68.0
```

其中：

```python
def to_fahrenheit(self):
	return self.temperature*1.8 + 32

def to_fahrenheit(self):
	return self._temperature*1.8 + 32
```

这两种写法结果不同。

```python
Getting value     # self.temperature*1.8 + 32 
Out[115]: 68.0

Out[116]: 68.0    # self._temperature*1.8 + 32
```

`self.temperature` 会调用 `temperature()`函数。

PS:

在声明函数 `to_fahrenheit` 时：

注意 `self.temperature*1.8 + 32` 属性的命名需要区别与 `getter` 或者 `setter` 中属性的名字，以免发生递归调用。 

```python
class property([fget[, fset[, fdel[, doc]]]])
```

* `fget`  --  获取属性值的函数 
* `fset`  -- 设置属性值的函数 
* `fdel `  -- 删除属性值函数 
* `doc`    -- 属性描述信息 

> The `property()` function is implemented as a data descriptor. Accordingly,instances cannot override the behavior of a property.

***

### 2.`@staticmethod`、`@classmethod`

`@staticmethod`、`@classmethod`也是描述符。

 [`staticmethod()`](https://docs.python.org/3/library/functions.html#staticmethod) and [`classmethod()`](https://docs.python.org/3/library/functions.html#classmethod)) are implemented as non-data descriptors.  

***

###  3.`super()`

> The object returned by `super()` also has a custom `__getattribute__` method for invoking descriptors.  The call `super(B, obj).m()` searches `obj.__class__.__mro__` for the base class `A` immediately following `B` and then returns `A.__dict__['m'].__get__(obj, B)`.  If not a descriptor, `m` is returned unchanged.  If not in the dictionary, `m` reverts to a search using `object.__getattribute__`.

### 4. function

函数也是描述符，从中可以看到 function 中存在 `__get__`方法。

```python
In [1]: def func():				# 声明 func 函数
   ...:     pass
   ...:

In [2]: func					# func 函数亦是对象
Out[2]: <function __main__.func()>

In [3]: type(func)				# func 函数的类型是 function
Out[3]: function

In [5]: type(func).__dict__		# function 类包含 __get__、__call__ 方法
Out[5]:
mappingproxy({'__repr__': <slot wrapper '__repr__' of 'function' objects>,
              '__call__': <slot wrapper '__call__' of 'function' objects>,
              '__get__': <slot wrapper '__get__' of 'function' objects>,
              '__new__': <function function.__new__(*args, **kwargs)>,
              '__closure__': <member '__closure__' of 'function' objects>,
              '__doc__': <member '__doc__' of 'function' objects>,
              '__globals__': <member '__globals__' of 'function' objects>,
              '__module__': <member '__module__' of 'function' objects>,
              '__code__': <attribute '__code__' of 'function' objects>,
              '__defaults__': <attribute '__defaults__' of 'function' objects>,
              '__kwdefaults__': <attribute '__kwdefaults__' of 'function' objects>,
              '__annotations__': <attribute '__annotations__' of 'function' objects>,
              '__dict__': <attribute '__dict__' of 'function' objects>,
              '__name__': <attribute '__name__' of 'function' objects>,
              '__qualname__': <attribute '__qualname__' of 'function' objects>})
```

不仅如此，类中定义的函数亦是描述符。

```python
In [1]: class A:
   ...:     def func(self):
   ...:         print('---do something ---')
   ...:

In [7]: A.func
Out[7]: <function __main__.A.func(self)>

In [8]: type(A.func)
Out[6]: function
```

在类中定义函数，相当于定义一个描述器。那么以下二者某种意义上等价：

```python
class A:
    def func(self):
        pass
```

```python
class A:
    func = function()
```

当通过两种不同方式访问 `func`时：

```python
In [7]: a = A()

In [8]: A.func		# A.test.__get__(None, A)
Out[8]: <function __main__.A.func(self)>

In [9]: a.func		# A.test.__get__(a, A)
Out[9]: <bound method A.func of <__main__.A object at 0x000001AAF4143F60>>
```

说明了绑定(bound)和非绑定(unbound)的不同之处。

```python
In [14]: a.func()
---do something ---

In [15]: A.func(a)
---do something ---

In [16]: A.func()
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-16-21f556f4a136> in <module>
----> 1 A.func()

TypeError: func() missing 1 required positional argument: 'self'
```

由此可以发现，在定义A的时候，`func`方法是有一个参数`self`。

* `A.func`返回一个function对象，是一个未绑定函数，所以调用的时候要传对象(`A.func(a)`)
* `a.func`返回一个bound method对象，是一个绑定函数，所以调用的时候不需要再传入对象(`a.func()`)

可以看出，所谓绑定，就是把调用函数的对象，绑定到函数的第一个参数上。

做一个总结，function是一个可以被调用(实现了`__call__`方法)的描述器(实现了`__get__`方法)对象。

* 通过类获取函数对象的时候，`__get__`方法会返回function本身，
* 通过实例获取函数对象的时候，`__get__`方法会返回一个bound method，也就是将实例绑定到这个function上。

***

## 补充

### `__getattr__`

> **Called when** the default attribute access fails with an `AttributeError` (either `__getattribute__()` raises an `AttributeError` because *name* is not an instance attribute or an attribute in the class tree for `self`; or `__get__()` of a *name* property raises `AttributeError`).  This method should either return the (computed) attribute value or raise an `AttributeError` exception.
>
> **Note that** if the attribute is found through the normal mechanism,`__getattr__()` is not called.  (This is an intentional asymmetry between `__getattr__()` and `__setattr__()`.) This is done both for efficiency reasons and because otherwise `__getattr__()` would have no way to access other attributes of the instance.  Note that at least for instance variables,you can fake total control by not inserting any values in the instance attribute dictionary (but instead inserting them in another object).

```python
class Company:
    def __init__(self, name, info={}):
        self.name = name
        self.info = info

    def __getattr__(self, item):
        print('getattr')
        return self.info[item]

if __name__ == "__main__":
    user = Company(name='abc', info={'company':'ABC',})
    print(user.name)
    print(user.company)
```

```python
abc
getattr
ABC
```

* 正常调用  `name` 属性时，潜在调用的是 `__getattribute__`方法
* 当调用的属性无法访问时，会调用 `__getattr__`

重载 `__getattr__` 方法可以方便处理属性调用失败之后处理操作。

谨慎使用 `__getattribute__`，此方法直接涉及属性访问，容易产生各种问题，比如递归调用等。

***

参考：

[Properties vs. Getters and Setters](https://www.python-course.eu/python3_properties.php)

[有效的python属性管理: 描述符的使用](https://zhuanlan.zhihu.com/p/24305162)

[Descriptor HowTo Guide](https://docs.python.org/3/howto/descriptor.html#id1)

[Difference between `__getattr__` vs `__getattribute__`](https://stackoverflow.com/questions/3278077/difference-between-getattr-vs-getattribute)

[Python Descriptors Demystified](http://nbviewer.jupyter.org/urls/gist.github.com/ChrisBeaumont/5758381/raw/descriptor_writeup.ipynb)

[深度解析并实现python中的super](https://blog.csdn.net/zhangjg_blog/article/details/83033210)