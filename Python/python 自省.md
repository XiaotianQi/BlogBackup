自省（introspection）也称为反射（reflection）。简而言之，自省使用一些机制实现自我检查，通过这些机制可以查看对象的类型、class、属性、方法等信息。在计算机编程中，检查某些事物以确定它是什么、它知道什么以及它能做什么。自省向程序员提供了极大的灵活性和控制力。

***

## `dir`

```python
dir([object])
```

如果没有实参，则返回当前本地作用域中的名称列表。如果有实参，它会尝试返回该对象的有效属性列表。

`object`的默认值是当前的模块对象。如果参数包含方法 `__dir__`，该方法将被调用，并且必须返回一个属性列表。如果参数不包含 `__dir__`，该方法将最大限度地收集参数信息。

默认的 `dir()` 机制对不同类型的对象行为不同，它会试图返回最相关而不是最全的信息：

- 无参数时，返回当前作用域内的变量、方法和定义的类型列表。
- 模块对象：返回模块的属性列表。
- 类型或类对象：返回类及其子类的属性列表，并且递归查找所有基类的属性。
- 当对象定义了`__dir__`方法，则返回`__dir__`方法的结果

主要是为了便于在交互式时使用。

```python
class Cat(object): # Cat类

    def __init__(self, name='kitty'): # 属性
        self.name = name

    def sayHi(self): # 实例方法
        print(self.name, 'says Hi!')

cat = Cat('kitty')
```

```python
print(dir(Cat))
print(dir(cat))
```

```bash
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'sayHi']

['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'name', 'sayHi']
```

***

## `type`

返回对象的类型。

基本类型都可以用 `type` 判断。比如：`int`、`str`、`FunctionType`、`BuiltinFunctionType`、`LambdaType`、`GeneratorType`。

```python
import types

def fn():
    pass

print(type(123)==int)
print(type('abc')==str)
print(type(fn)==types.FunctionType)
print(type(abs)==types.BuiltinFunctionType)
print(type(lambda x: x)==types.LambdaType)
print(type((x for x in range(10)))==types.GeneratorType)
```

以上输出结果均为 `True`。

但在判断 class 的继承关系就不方便。

```python
class Animals(object):
    def __init__(self, name):
        self.name = name

class Dogs(Animals):
    pass

class Cats(Animals):
    pass
```

```python
print(type(husky))
print(type(siamese))
```

```bash
<class '__main__.Dogs'>
<class '__main__.Cats'>
```

均返回了各自所属的类，而没有返回父类 `Animals`。

***

## `hasattr`

```python
hasattr(obj, attr)
```

判断 `obj` 是否有一个属性名为 `attr`，返回一个布尔值。一般用于访问某属性前先做一个检查。

```python
class Cat(object):

    def __init__(self, name):
        self.name = name

    def hi(self):
        print('Hi,', self.name)

cat = Cat('siamese')
```

```python
if hasattr(cat, 'name'):
    setattr(cat, 'name', 'siamese')

print(getattr(cat, 'name'))
getattr(cat, 'hi')()
```

```bash
siamese
Hi, siamese
```

***

## `isinstance`

```python
isinstance(object, class-or-tuple)
```

* `object` -- 实例对象。
* `class-or-tuple` -- 可以是直接或间接类名、基本类型或者由它们组成的元组。
  

判断 `object` 对象是否是 `class` 的实例，或者是否属于 `type` 类型，相同则返回 `True`，否则返回 `False`。

```python
print(isinstance (1, int))
print(isinstance (1, str))
print(isinstance (1, (str,int,list)))
```

```bash
True
False
True
```

判断实例的 `class` 的类型，尤其存在继承关系的 `class`，相对 `type` 而言，好用一些。

继承关系如下：

```bash
object --> Animals --> Dogs
```

```python
test = Animals('Test')
husky = Dogs('Husky')
print(isinstance(test, Animals))
print(isinstance(husky, Animals))
print(isinstance(husky, Dogs))
```

```bash
True
True
True
```

`isinstance()` 与 `type()` 区别：

* `type()` 不会认为子类是一种父类类型，不考虑继承关系。
* `isinstance()` 会认为子类是一种父类类型，考虑继承关系。
***

## `id`

```python
id([object])
```

返回对象的内存地址。

------

## ` hash(object)`

返回该对象的哈希值（如果它有的话）。哈希值是整数。它们在字典查找元素时用来快速比较字典的键。相同大小的数字变量有相同的哈希值（即使它们类型不同，如 1 和 1.0）。

```bash
In [134]: hash(12345)
Out[134]: 12345

In [135]: hash(12345.0)
Out[135]: 12345

In [136]: hash('abc')
Out[136]: 7491864301760075081
```

------

## `callable`

```python
callable(object)
```

可调用返回 `True`，否则返回 `False`。

```python
def foo():
    pass

class Cat(object):
    pass

class Dog(object):
    def __call__(self):
        pass
```

```python
cat = Cat()
dog = Dog()

print('foo', callable(foo))
print('Cat', callable(Cat))
print('cat', callable(cat))
print('Dog', callable(Dog))
print('dog', callable(dog))
```

```bash
foo True
Cat True
cat False
Dog True
dog True
```

***

## `issubclass`

```python
isinstance(object, class-or-tuple)
```

判断 `object` 是否是 `class-or-tuple` 的子类，是则返回 True ，反之则返回 False。

```python
print(issubclass(Dogs, Animals))
print(issubclass(Cats, Animals))
print(issubclass(Dogs, Dogs))
```

```bash
True
True
True
```

***

## Special Attributes

> - `object.__dict__`
>
>   A dictionary or other mapping object used to store an object’s (writable) attributes. 
>
> -  `instance.__class__`
>
>   The class to which a class instance belongs. 
>
> -  `class.__bases__`
>
>   The tuple of base classes of a class object. 
>
> -  `definition.__name__`
>
>   The name of the class, function, method, descriptor, or generator instance. 
>
> -  `definition.__qualname__`
>
>   The [qualified name](https://docs.python.org/3/glossary.html#term-qualified-name) of the class, function, method, descriptor, or generator instance.  New in version 3.3.  
>
> -  `class.__mro__`
>
>   This attribute is a tuple of classes that are considered when looking for base classes during method resolution. 
>
> -  `class.mro`()
>
>   This method can be overridden by a metaclass to customize the method resolution order for its instances.  It is called at class instantiation, and its result is stored in [`__mro__`](https://docs.python.org/3/library/stdtypes.html#class.__mro__). 
>
> -  `class.__subclasses__`()
>
>   Each class keeps a list of weak references to its immediate subclasses.  This method returns a list of all those references still alive. Example:

在 python 中，对象的属性和方法都统称为属性。无论类还是实例，它们的属性都是以字典的形式存储在`__dict__`中。

```python
class A:
    attr_a = 1
class B(A):
    attr_b = 2
    def __init__(self, c):
        self.attr_c = c
```

类的属性只会存在本类的`__dict__`中，实例的属性只会存在实例的`__dict__`中。

```python
In [25]: a = B(3)

In [26]: a.attr_a, a.attr_b, a.attr_c
Out[26]: (1, 2, 3)

In [27]: a.__dict__
Out[27]: {'attr_c': 3}

In [28]: B.__dict__
Out[28]:
mappingproxy({'__module__': '__main__',
              'attr_b': 2,
              '__init__': <function __main__.B.__init__(self, c)>,
              '__doc__': None})

In [29]: A.__dict__
Out[29]:
mappingproxy({'__module__': '__main__',
              'attr_a': 1,
              '__dict__': <attribute '__dict__' of 'A' objects>,
              '__weakref__': <attribute '__weakref__' of 'A' objects>,
              '__doc__': None})

In [30]: A.__dict__['attr_a']
Out[30]: 1
```

而且，`A.attr_a` 也可以通过 `A.__dict__['attr_a']` 访问。

***

## ` vars([object])`

返回对象object的属性和属性值的字典对象。

如果没有参数，就打印当前调用位置的属性和属性值，类似 `locals()`。

 当函数接收一个参数时，参数可以是模块、类、类实例，或者定义了`__dict__`属性的对象，等价于`obj.__dict__`。

------

## `locals()` & ` globals()`

- `locals()`

  更新并返回表示当前本地符号表的字典。

  在函数块而不是类块中调用时，会返回自由变量。

  不要更改此字典的内容，因为更改不会影响解释器使用的局部变量或自由变量的值。

  ```bash
  In [140]: def func(n):
       ...:     str = 'a'
       ...:     print(locals())
       ...:     test = locals()
       ...:     test['str'] = 'b'
       ...:     print(locals())
       ...:
  
  In [141]: func(2)
  {'n': 2, 'str': 'a'}
  {'n': 2, 'str': 'a', 'test': {...}}
  ```

- ` globals()`

  返回全局变量的字典。

***

## `help`

```python
help([object])
```

返回对象帮助信息。

`help` 在 python 交互式环境中使用，比如 `Ipython`。

输入无参数 `help()`时，会进入 help 交互环境：

```bash
In [2]: help()

Welcome to Python 3.7s help utility!
...

help>
```

* 指定的modules，例如 collections 模块：`help("collections")`；
* keywords：`help('keywords')`；
* 内置的类型：`help("list")`；
* 类型的成员方法：`help("list.pop")`；
* 内置函数：`help("iter")`。

`help` 函数是查看函数或模块用途的详细说明，而 `dir` 函数是查看函数或模块内的属性、方法，输出列表。

***

## 检查工具

```python
def interrogate(item):
    if hasattr(item, '__name__'):
        print('NAME:     ', item.__name__)
    if hasattr(item, '__class__'):
        print('CLASS:    ', item.__class__.__name__)
    print('TYPE:     ', type(item))
    print('VALUE:    ', repr(item))
    if callable(item):
        print('CALLABLE:  YES')
    else:
        print('CALLABLE:  NO')
    if hasattr(item, '__doc__'):
        doc = getattr(item, '__doc__')
    doc = doc.strip()
    firstline = doc.split('\n')[0]
    print('DOC:      ', firstline)
```

```python
interrogate('a string')
interrogate(123)
interrogate(iter)
```

```bash
CLASS:     str
TYPE:      <class 'str'>
VALUE:     'a string'
CALLABLE:  NO
DOC:       str(object='') -> str
CLASS:     int

TYPE:      <class 'int'>
VALUE:     123
CALLABLE:  NO
DOC:       int([x]) -> integer

NAME:      iter
CLASS:     builtin_function_or_method
TYPE:      <class 'builtin_function_or_method'>
VALUE:     <built-in function iter>
CALLABLE:  YES
DOC:       iter(iterable) -> iterator
```

***

## 基于字符串的反射机制

利用字符串的形式去对象中操作成员属性和方法。Python的四个重要内置函数：getattr()、hasattr()、delattr()和setattr()较为全面的实现了基于字符串的反射机制。

场景：需要根据用户输入url的不同，调用不同的函数，实现不同的操作，也就是一个WEB框架的url路由功能。

* 具体执行操作的`commons.py`文件
* 入口文件`index.py`，接收用户输入，并根据输入展示相应的页面

```python
# commons.py

def login():
    print('登录页面!')

def logout():
    print('退出页面!')

def index():
    print('主页面')
```

```python
# visit.py
import commons

def run():
    inp = input("请输入您想访问页面的url：  ").strip()
    if hasattr(commons, inp):
        func = getattr(commons, inp)
        func()
    else:
        print("404")

if __name__ == '__main__':
    run()
```

```bash
请输入您想访问页面的url：  index
主页面
```

## 动态导入模块

使用Python内置的`__import__(字符串参数)`函数解决这个问题。通过它，可以实现类似getattr()的反射功能。`__import__()`方法会根据字符串参数，动态地导入同名的模块。

```python
# visit.py

def run():
    inp = input("请输入您想访问页面的url：  ").strip()
    modules, func = inp.split("/")
    obj = __import__(modules)
    if hasattr(obj, func):
        func = getattr(obj, func)
        func()
    else:
        print("404")

if __name__ == '__main__':
    run()
```

```python
请输入您想访问页面的url：  commons/index
主页面
```

如果我们的目录结构是这样的，visit.py和commons.py不在一个目录下，存在跨包的问题：

```python
def run():
    inp = input("请输入您想访问页面的url：  ").strip()
    modules, func = inp.split("/")
    obj = __import__("lib." + modules, fromlist=True)  # 注意fromlist参数
    if hasattr(obj, func):
        func = getattr(obj, func)
        func()
    else:
        print("404") 
if __name__ == '__main__':
    run()
```

因为对于`lib.xxx.xxx.xxx`这一类的模块导入路径，`__import__()`默认只会导入最开头的圆点左边的目录，也就是`lib`。

***

参考：

[Python 自省指南](https://www.ibm.com/developerworks/cn/linux/l-pyint/#ibm-pcon)

http://www.liujiangblog.com/course/python/48