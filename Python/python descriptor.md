Descriptor 的定义：
> In general, a descriptor is an object attribute with “binding behavior”, one whose attribute access has been overridden by methods in the descriptor protocol.  Those methods are `__get__`, `__set__`, and `__delete__`.  If any of those methods are defined for an object, it is said to be a descriptor.
>
> They are the mechanism behind properties, methods, static methods, class methods, and `super()`.

使用 Descriptor 的便捷之处：

> For the users of a class, properties are syntactically identical to ordinary attributes.You can start with the simplest implementation imaginable, and  you are free to later migrate to a property version without having to change the interface! So properties are not just a replacement for getters and setter! 
>
> The Pythonic way to introduce attributes is to make them public.

## Descriptor

> The default behavior for attribute access is to get, set, or delete the attribute from an object’s dictionary.

> ```
> descr.__get__(self, obj, type=None) -> value
> descr.__set__(self, obj, value) -> None
> descr.__delete__(self, obj) -> None
> ```

定义以上任何一个方法或者属性，都将被当做描述符，并覆盖默认的属性读写行为。

`obj.x` 属性被访问时：

```text
`obj.__dict__['x']` --> `type(obj).__dict['x']` --> `type(type(obj)).__dict__['x']`
```

直至 `obj` 的基类，顺序符合 C3 算法。

具体步骤：

* 查看当前是否是 descriptor；
* 若是，则调用 `__set__` 或 `__get__`，进行属性读写；
* 若不是，则调用 `__setattribute__` 或  `__getattribute__`，进行属性读写 。

即，descriptor 会改变默认的属性读写方式。

***

创建一个只读数据描述符时：

> To make a read-only data descriptor, define both `__get__` and `__set__` with the `__set__` raising an `AttributeError` when called.

```python
def __set__(self, instance, value):
	raise AttributeError(
	"'%s' is not modifiable" % self.value
	)
```

***

```python
class DataDes:
    def __init__(self):
        pass
    def __get__(self, instance, owner):
        print('DataDes.__get__')
    def __set__(self, instance, value):
        print('DataDes.__set__', value)
 

class DataDes_noget:
    def __init__(self):
        pass
    def __set__(self, instance, value):
        print('DataDes_noget.__set__', value)
       
    
class DataDes_noset:
    def __init__(self):
        pass
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
Out: {}                 # 没有调用__dict__
```
如果一个对象同时定义了`__get__`和`__set__`，则这个描述符被称为 `data descriptor`。

如果一个对象只定义了`__get__`方法，则这个描述符被称为`non-data descriptor`。

无`__set__`：

```python
In : a.datades = 1
DataDes.__set__ 1       # 调用__set__

In : a.datades
DataDes.__get__

In : a.data_noset = 2   # 调用__setattribute__

In : a.data_noset
Out: 2

In : a.__dict__
Out: {'data_noset': 2}
```
无 `__get__`：

```python
In :a.__dict__['datades'] = 1

In :a.__dict__['data_noget'] = 2
```

```python
In : a.datades
DataDes.__get__          # 调用 __get__

In : a.data_noget
Out: 2                   # 调用 __getattribute__

In : a.__dict__
Out: {'datades': 1, 'data_noget': 2}
```

PS:

当使用 `obj.name` 或 `obj.name = value`时

```python
object.__setattribute__(self,name,value)
object.__getattribute__(self,name)
```

***

## 优先级

> Data and non-data descriptors differ in how overrides are calculated with  respect to entries in an instance’s dictionary.  If an instance’s dictionary has an entry with the same name as a data descriptor, the data descriptor takes precedence.  If an instance’s dictionary has an entry with the same name as a non-data descriptor, the dictionary entry takes precedence.

> 对属性进行访问的时候需要几行打交道的基本上包含这几个对象：
>
> 1. data descriptor
> 2. non-data descriptor
> 3. 实例的字典
> 4. 内置的`__getattr__`函数
>
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

***

## `@property`、`property()`

在 python 中，所有的属性都是公共的。即使用双下划线声明的属性，也无法阻止将其获取并使用。

使用 `property` 不仅可以便捷的设置属性可读、可读写，从而隐藏并保护其不受其他代码影响，还可以在不需要更改客户代码的情况下，对属性进行修改，获得更好的兼容性。

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

`@staticmethod`、`@classmethod`也是描述符。

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

***

## `super()`

> The object returned by `super()` also has a custom `__getattribute__` method for invoking descriptors.  The call `super(B, obj).m()` searches `obj.__class__.__mro__` for the base class `A` immediately following `B` and then returns `A.__dict__['m'].__get__(obj, B)`.  If not a descriptor, `m` is returned unchanged.  If not in the dictionary, `m` reverts to a search using `object.__getattribute__`.

***

## 注意

对每个学生类创建对应的身高描述符，而且把身高数据放在描述符中：

```python
class Height(object):
    def __init__(self, height):
        self.height = height
    def __get__(self, instance, cls):
        return self.height
    def __set__(self, instance, value):
        self.height = value
 
class Student(object):
    height = Height(1.78)
```

```python
In [4]: a = Student()

In [5]: a.height
Out[5]: 1.78

In [6]: a.height = 1.8

In [7]: a.height
Out[7]: 1.8

In [8]: b = Student()

In [9]: b.height
Out[9]: 1.8

In [10]: b.height = 1.85

In [11]: b.height
Out[11]: 1.85

In [12]: a.height
Out[12]: 1.85
```

创建 `Student`类的两个实例，但身高使用的是同一个**类属性**。

***

参考：

[Properties vs. Getters and Setters](https://www.python-course.eu/python3_properties.php)

[有效的python属性管理: 描述符的使用](https://zhuanlan.zhihu.com/p/24305162)

[Descriptor HowTo Guide](https://docs.python.org/3/howto/descriptor.html#id1)