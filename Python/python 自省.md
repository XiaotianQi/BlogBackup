自省（introspection）也称为反射（reflection）。简而言之，自省使用一些机制实现自我检查，通过这些机制可以查看对象的类型、class、属性、方法等信息。

* `dir`
* `type`
* `isinstance`、`issubclass`
* `id`
* `hasattr`
* `callable`
* `__name__`、`__doc__`、`__dir__`
* `help`

***

### `dir`

```python
dir([object])
```

返回包含 `object` 大多数属性名的列表（会有一些特殊的属性不包含在内）。`object`的默认值是当前的模块对象。如果参数包含方法 `__dir__`，该方法将被调用。如果参数不包含 `__dir__`，该方法将最大限度地收集参数信息。

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

### `type`

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

### `isinstance`

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

### `issubclass`

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

### `id`

```python
id([object])
```

返回对象的内存地址。

***

### `hasattr`

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

### `callable`

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

### `__name__`、`__doc__`、`__dir__`

`__name__`、`__doc__`、`__dir__` 分别返回对象名称、描述该对象的作用、对象的所有属性。

类似这种方法很多，例如 `__class__`。

***

### `help`

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

参考：

[Python 自省指南](https://www.ibm.com/developerworks/cn/linux/l-pyint/#ibm-pcon)