类当中的方法分为三类：

* instance method
* class method
* static method

## 实例方法

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

text.get_data('test')			# 绑定
I_method.get_data(text, 'test')	# 未绑定，需要显性传入实例
```

```text
<bound method I_method.get_data of <__main__.I_method object at 0x0000026CA3E0A358>>
<function I_method.get_data at 0x0000026CA3E041E0>
test
test
```

`text.get_data` **绑定**了实例的方法，python 隐式的把对象传递给了 `self`参数，所以不再手动传递参数，这是调用实例方法的过程。

`I_method.get_data` 是**没有绑定**实例的方法。对于未绑定方法，调用时，需要**显性的传入实例对象**。即：`I_method.get_data(text, 'test')` 等价于 `text.get_data('test')`。

***

## 类方法

`@classmethod`:

返回函数的类方法。

Python还可以在类中定义类方法，类方法的第一个参数约定名为`cls`，它代表的是当前类相关的信息的对象（类本身也是一个对象，有的地方也称之为类的元数据对象）。通过这个参数，可以获取和类相关的信息并且可以创建出类的对象。

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

`text.get_data`、`C_method.get_data` 都**自动绑定**了类对象`C_method`的`get_data`方法。

对于`text.get_data`，因为 python 通过实例对象`text`找到它所属的类是`C_method`，找到`C_method`之后自动绑定到 `cls`。关于这个属性查找逻辑，可以了解 python 描述符。

如果一个类方法在子类上调用，子类会作为第一个实参传入。

适用于：

只在类中运行而不在实例中运行的方法。使用`@classmethod`后，不管这个方式是从实例调用还是从类调用，它都把类作为第一个参数传递进来。

```python
from time import time, localtime, sleep


class Clock(object):
    """数字时钟"""

    def __init__(self, hour=0, minute=0, second=0):
        self._hour = hour
        self._minute = minute
        self._second = second

    @classmethod
    def now(cls):
        ctime = localtime(time())
        return cls(ctime.tm_hour, ctime.tm_min, ctime.tm_sec)

    def run(self):
        """走字"""
        self._second += 1
        if self._second == 60:
            self._second = 0
            self._minute += 1
            if self._minute == 60:
                self._minute = 0
                self._hour += 1
                if self._hour == 24:
                    self._hour = 0

    def show(self):
        """显示时间"""
        return '%02d:%02d:%02d' % \
               (self._hour, self._minute, self._second)


def main():
    # 通过类方法创建对象并获取系统时间
    clock = Clock.now()
    while True:
        print(clock.show())
        sleep(1)
        clock.run()


if __name__ == '__main__':
    main()
```

***

## 静态方法

`@staticmethod`:

返回函数的静态方法。

该方法不强制要求传递参数。

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

`S_method` 中的`get_data`方法和普通函数没什么区别，与类和实例都**没有绑定关系**。

与`@classmethod`相同，`@staticmethod`也是描述符。

适用于：

Often there is some functionality that relates to the class, but does  not need the class or any instance(s) to do some work. Perhaps something like setting environmental variables, changing an attribute in another class, etc. In these situation we can also use a function, however doing so also spreads the interrelated code which can cause maintenance  issues later.

如果在方法中不需要访问任何实例方法和属性，纯粹地通过传入参数并返回数据的功能性方法，那么它就适合用静态方法来定义，它节省了实例化对象的开销成本，往往这种方法放在类外面的模块层作为一个函数存在也是没问题的，而放在类中，仅为这个类服务。

## 类方法 vs 静态方法

如果希望在方法里面调用静态方法，那么把方法定义成类方法是合适的。因为要是定义成静态方法，那么你就要显示地引用类，这对继承来说可不是一件好事情。

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
print(a.__class__)	# <class '__main__.A'>
print(b.__class__)  # <class '__main__.A'>
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

我们定义一个“三角形”类，通过传入三条边长来构造三角形，并提供计算周长和面积的方法，但是传入的三条边长未必能构造出三角形对象，因此我们可以先写一个方法来验证三条边长是否可以构成三角形，这个方法很显然就不是对象方法，因为在调用这个方法时三角形对象尚未创建出来（因为都不知道三条边能不能构成三角形），所以这个方法是属于三角形类而并不属于三角形对象的。我们可以使用静态方法来解决这类问题，代码如下所示:

```python
from math import sqrt


class Triangle(object):

    def __init__(self, a, b, c):
        self._a = a
        self._b = b
        self._c = c

    @staticmethod
    def is_valid(a, b, c):
        return a + b > c and b + c > a and a + c > b

    def perimeter(self):
        return self._a + self._b + self._c

    def area(self):
        half = self.perimeter() / 2
        return sqrt(half * (half - self._a) *
                    (half - self._b) * (half - self._c))


def main():
    a, b, c = 3, 4, 5
    if Triangle.is_valid(a, b, c):
        t = Triangle(a, b, c)
        print(t.perimeter())
        print(t.area())
    else:
        print('无法构成三角形.')


if __name__ == '__main__':
    main()
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

[Class vs static methods in Python](https://www.pythoncentral.io/difference-between-staticmethod-and-classmethod-in-python/)，Arun Tigeraniya 

[Meaning of @classmethod and @staticmethod for beginner?](https://stackoverflow.com/questions/12179271/meaning-of-classmethod-and-staticmethod-for-beginner)，StackOverflow

[正确理解Python中的 @staticmethod@classmethod方法](https://zhuanlan.zhihu.com/p/28010894)，刘志军

[静态方法和类方法](https://github.com/jackfrued/Python-100-Days/blob/master/Day01-15/09.%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%BF%9B%E9%98%B6.md)，骆昊