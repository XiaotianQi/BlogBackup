类当中的方法分为三类：

* instance method
* class method
* static method

## instance method

```python
class I_method:
    def get_data(self, data):
        print(data)
```

`self` 参数会传入实例。这也是最常见的用法。

```python
text = I_method()

print(text.get_data)
print(I_method.get_data)

text.get_data('test')
I_method.get_data(text, 'test')
```

```text
<bound method I_method.get_data of <__main__.I_method object at 0x0000026CA3E0A358>>
<function I_method.get_data at 0x0000026CA3E041E0>
test
test
```

`text.get_data` 绑定了实例的方法，python 隐式的把对象传递给了 `self`参数，所以不再手动传递参数，这是调用实例方法的过程。

`I_method.get_data` 是没有绑定实例的方法。对于未绑定方法，调用时，需要显性的传入实例对象。即：`I_method.get_data(text, 'test')` 等价于 `text.get_data('test')`。

***

## class method

```python
class C_method:
    data = 'DATA'

    @classmethod
    def get_data(cls, test):
        print(cls.data, test)
```

```python
text = C_method()

print(text.get_data)
print(C_method.get_data)

text.get_data('test')
C_method.get_data('test')
```

```text
<bound method C_method.get_data of <class '__main__.C_method'>>
<bound method C_method.get_data of <class '__main__.C_method'>>
DATA test
DATA test
```

`text.get_data`、`C_method.get_data` 都自动绑定了类对象`C_method`的`get_data`方法。

对于`text.get_data`，因为 python 通过实例对象`text`找到它所属的类是`C_method`，找到`C_method`之后自动绑定到 `cls`。关于这个查找逻辑，可以了解 python 描述符。

适用于：

只在类中运行而不在实例中运行的方法。使用`@classmethod`后，不管这个方式是从实例调用还是从类调用，它都把类作为第一个参数传递进来。

***

## static method

```python
class S_method:
    data = 'DATA'

    @staticmethod
    def get_data(test):
        print(test)
```

```python
text = S_method()

print(text.get_data)
print(S_method.get_data)

text.get_data('test')
S_method.get_data('test')
```

```text
<function S_method.get_data at 0x0000020331406378>
<function S_method.get_data at 0x0000020331406378>
test
test
```

`S_method` 中的`get_data`方法和普通函数没什么区别，与类和实例都没有绑定关系。

与`@classmethod`相同，`@staticmethod`也是描述符。

适用于：

> Often there is some functionality that relates to the class, but does  not need the class or any instance(s) to do some work. Perhaps something like setting environmental variables, changing an attribute in another class, etc. In these situation we can also use a function, however doing so also spreads the interrelated code which can cause maintenance  issues later.

> 如果在方法中不需要访问任何实例方法和属性，纯粹地通过传入参数并返回数据的功能性方法，那么它就适合用静态方法来定义，它节省了实例化对象的开销成本，往往这种方法放在类外面的模块层作为一个函数存在也是没问题的，而放在类中，仅为这个类服务。

值得注意是：

> 如果希望在方法里面调用静态方法，那么把方法定义成类方法是合适的。因为要是定义成静态方法，那么你就要显示地引用类，这对继承来说可不是一件好事情。

```python
class A:
    def __init__(self, data=0):
        self.data = data

    @staticmethod
    def get_data(data):
        return A(data)

class B(A):
    pass
```

```python
a = A.get_data(123)
b = B.get_data(123)
print(a.__class__)
print(b.__class__)
```

```text
<class '__main__.A'>
<class '__main__.A'>
```

纵然，`B` 继承 `A`，但使用 `@staticmethod`后，`B` 中的 `get_data` 所属类是 `A`类，产生硬编码。所以此时，应使用 `@classmethod`。

```python
class A:
    def __init__(self, data=0):
        self.data = data

    @classmethod
    def get_data(cls, data):
        return cls(data)

class B(A):
    pass
```

***

示例：

处理日期信息，来自 StackOverflow：

```python
class Date(object):

    def __init__(self, day=0, month=0, year=0):
        self.day = day
        self.month = month
        self.year = year

    @classmethod
    def from_string(cls, date_as_string):
        day, month, year = map(int, date_as_string.split('-'))
        return cls(day, month, year)

    @staticmethod
    def is_date_valid(date_as_string):
        day, month, year = map(int, date_as_string.split('-'))
        return day <= 31 and month <= 12 and year <= 3999
```

使用 `@classmethod`创建了另一个“构造函数”，同时也完成日期解析，将数据和方法封装。

```python
date = Date.from_string('11-09-2012')
is_date = Date.is_date_valid('11-09-2012')

print(date)
print(date.day, date.month, date.year)
print(is_date)
```

```text
<__main__.Date object at 0x000001D2E4F07080>
11 9 2012
True
```

***

参考：

[Class vs static methods in Python](https://www.pythoncentral.io/difference-between-staticmethod-and-classmethod-in-python/)

[Meaning of @classmethod and @staticmethod for beginner?](https://stackoverflow.com/questions/12179271/meaning-of-classmethod-and-staticmethod-for-beginner)

[正确理解Python中的 @staticmethod@classmethod方法](https://zhuanlan.zhihu.com/p/28010894)