面向对象编程（Object Oriented Programming），简称 OOP，是一种程序设计思想。OOP 把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数。

将计算机程序视为一组对象的集合，对象之间可以互动，接受、处理和反馈彼此之间传递的信息，计算机程序的执行就是一系列消息在各个对象之间传递。

------

## 概述

面向对象最重要的概念就是类（**Class**）和实例（**Instance**）。类是一种数据类型。类与实例之间的关系，类似生物中种属关系。

### 一个简单的面向对象的程序

以存储动物名称和数量信息为例，说明类的创建，以及类与实例的关系。

采用面向对象的程序设计思想，我们首选思考的不是程序的执行流程，而是 Animals 这个数据类型应该被视为对象。而后，`Animals` 这个对象需要含有 `name`(名称) 和 `qty`(数量)两个属性（**Property**）。最后，给对象发一个 `print_qty` 消息，把自己的数据打印出来。

```python
# 定义类
class Animals(object):
    # 类的方法
    def __init__(self, name, qty):
        # 类的属性
        self.name = name
        self.qty = qty
    # 方法
    def print_qty(self):
        print("{}:{}".format(self.name, self.qty))

# 实例化
cat = Animals('cat', 2)
dog = Animals('dog', 3)
# 引用
cat.print_qty()
dog.print_qty()
```

Class 是一种抽象概念，也具有模版的作用。实例（Instance）则是根据类创建出来的一个具体的“对象”，每个对象都拥有相同的方法，但各自的数据可能不同。

比如：`Animals` 就是个抽象的概念，`cat` 和 `dog` 是两个具体的 `Animals`。给对象发消息实际上就是调用对象对应的关联函数，我们称之为对象的方法（**Method**）。

类支持两种操作：属性引用和实例化。属性引用使用和 Python 中所有的属性引用一样的标准语法：`obj.name`。各个实例拥有的数据都互相独立，互不影响。

### 相关概念

- 类(Class): 用来描述具有相同的属性和方法的对象的集合。它定义了该集合中实例共有的属性和方法。
- 类变量：类变量在整个实例化的对象中是**公用**。类变量定义在类中且在函数体之外。类变量通常不作为实例变量使用。
- 类属性：类中变量和方法都可被称作类属性，也可单指类变量。
- 方法重写：如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写。
- 实例变量：定义在方法中的变量，只作用于当前实例。
- 继承：即一个派生类（derived class）继承基类（base class），从而获得继承基类的属性和方法。继承也允许把一个派生类的对象作为一个基类对象对待。
- 实例化：创建一个类的实例，类的具体对象。
- `self`：实例，其下属性是实例私有。

## Duck Typing

> In duck typing, an object's suitability is determined by **the *presence of certain* *methods and properties*** (with appropriate meaning), rather than the actual type of the object. 
>
> The actual Apparel of an entity doesn't matter if the entity does all the intended things.
>
> In other words, don't check whether it IS-A duck,check whether it QUACKS-like-a duck, WALKS-like-a duck, etc. depending on exactly what subset of duck-like behavior you  need for your program.

```python
class Duck:
    def quack(self):
        print("Quack Quack")
    def fly(self):
        print ("Flap,Flap!")

class Hawk:
    def fly(self):
        print ("I am Fast.")

class Person:
    quackcount = 0
    def quack(self):
        Person.quackcount+=1
        print("QQQQUUUUAACCCCK!!!!!")
    def fly(self):
        print(Person.quackcount)

def is_a_duck(check):
    try:
        check.quack()
        check.fly()
    except(AttributeError,TypeError):
        print("If it can’t quack and fly then it’s not a Duck")
```

```python
is_a_duck(Duck())
is_a_duck(Person())
is_a_duck(Hawk())
```

```text
Quack Quack
Flap,Flap!
QQQQUUUUAACCCCK!!!!!
1
If it can’t quack and fly then it’s not a Duck
```

显而易见，`Hawk` 中没有实现 `quack`，所以`Hawk` 并不是 `Duck`。

而，`Person`实现了`quack`、`fly`，虽然 `Person`不是`Duck`，但是 `Person`也是 `Duck`。

## 定义类

```python
class Animals(object):
    pass
```

`class` 后面紧接着是类名，通常是首字符大写。紧接着是 `(object)`，表示该类是从哪个类继承下来的，若没有继承，则使用 `object` 类，这是所有类最终都会继承的类。

------

## 类的属性

### 直接在类中定义属性

```python
class Test(object):
    name = 'test'
```

### 在构造函数中定义属性

```python
class Test(object):
    def __init__(self, name):
        self.name = 'test'
```

### 属性的访问控制

`_attrs` 单下划线开头的变量，标明是一个受保护的变量，原则上不允许直接访问，但外部类还是可以访问到这个。只是一个约定，用于警告说明这是一个私有变量，外部类不要去访问它。

一般情况下，使用 `__private_attrs` 两个下划线开头，声明该属性为私有，不能在类地外部被使用或直接访问。在类内部的方法中使用时 `self.__private_attrs`。

无法通过 `实例变量.__private_attrs` 这个访问定义属性。实际上， Python 中是没有提供私有属性等功能的。但是 Python 对属性的访问控制是靠程序员自觉的。

可以说，python 中的属性都是公开的。

```python
class Animals(object):
    def __init__(self, name, qty, id):
        self.name = name
        self._qty = qty
        self.__id = id

    def get_id(self):
        return self.__id

if __name__ == '__main__':
    cats = Animals('cat', 2, 123456)
    print(dir(cats))
    print(cats._qty)
    # 打印构造函数中的属性
    print(cats.__dict__)
    print(cats.get_id())
    # 用于验证双下划线是否是真正的私有属性
    print(cats._Animals__id)
```

```python
['_Animals__id', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_qty', 'get_id', 'name']
2
{'name': 'cat', '_qty': 2, '_Animals__id': 123456}
123456
123456
```

仅定义了 `__id`，但打印结果中，它的名字改为了 `_Animals__id`。不能直接访问 `__id` 是因为 Python 解释器对外把 `__id` 变量改成了 `_Animals__id`，所以，仍然可以通过 `_Animals__id` 来访问 `__id` 变量。

### 类属性与实例属性

- Python是动态语言，根据类创建的实例可以任意绑定属性给实例，即使绑定的属性没有在类属性中定义。

```python
cats = Animals('cat', 2, 123456)
cats.test='test'
```

- 若类本身需要绑定一个属性，可以直接在类中定义属性，即类属性。类属性虽然归类所有，但类的所有实例都可以访问。
- 不要把实例属性和类属性使用相同的名字，相同名称的实例属性将屏蔽掉类属性，实例属性优先级比类属性高。

------

## 类的方法

### `self` 参数

在类的内部，`self` 代表的是类的实例。使用 `def` 关键字来定义一个方法，与一般函数定义不同，参数 `self`, 作为第一个参数。

类的方法与普通的函数只有一个特别的区别：它们必须有一个额外的第一个参数(`self`、`cls`)。且调用时，不用传递该参数。除此之外，类的方法和普通函数没有什么区别，所以，你仍然可以用默认参数、可变参数、关键字参数和命名关键字参数。

`self` 代表的是类的实例。

```python
class Test:
    def prt(self):
        print(self)
        print(self.__class__)
```

```python
t = Test()
t.prt()
```

```bash
<__main__.Test object at 0x0000022F3152CE10>
<class '__main__.Test'>
```

### 访问实例方法

```python
test = Test()
print(test.prt)
print(Test.prt)
```

使用实例访问实例方法时，方法是绑定的(bound)。

使用类名访问实例方法时，方法是未绑定的(unbound)。

```bash
<bound method Test.prt of <__main__.Test object at 0x000001CF50712080>>
<function Test.prt at 0x000001CF506F2EA0>
```

### `__init__`

构造函数，生成对象时自动调用。

类定义了 `__init__` 方法的话，实例化操作会自动调用 `__init__` 方法。

`__init__` 方法的第一个参数永远是 `self`，表示创建的实例本身，因此，在 `__init__` 方法内部，就可以把各种属性绑定到`self`，因为`self`就指向创建的实例本身。其他参数通过 `__init__` 传递到类的实例化操作上。

若有了 `__init__` 方法，则在创建实例的时候，就不能传入空的参数，必须传入与 `__init__`方 法匹配的参数。但 `self` 不需要传，Python 解释器会把实例变量传进去。

### 其他类的专有办法

|               | 作用                       |
| :------------ | :------------------------- |
| `__init__`    | 构造函数，在生成对象时调用 |
| `__del__`     | 析构函数，释放对象时使用   |
| `__repr__`    | 打印，转换                 |
| `__setitem__` | 按照索引赋值               |
| `__getitem__` | 按照索引获取值             |
| `__len__`     | 获得长度                   |
| `__cmp__`     | 比较运算                   |
| `__call__`    | 函数调用                   |
| `__add__`     | 加运算                     |
| `__sub__`     | 减运算                     |
| `__mul__`     | 乘运算                     |
| `__div__`     | 除运算                     |
| `__mod__`     | 求余运算                   |
| `__pow__`     | 乘方                       |

### 类的私有方法

`self.__private_methods`：两个下划线开头，声明该方法为私有方法，只能在类的内部调用 ，不能在类地外部调用。

------

## 数据封装

直接在类的内部定义访问数据的函数，相当于就把“数据”给封装起来。这些封装数据的函数是和类本身是关联起来的，我们称之为类的方法(Method)。

通过在实例上调用方法，就直接操作了对象内部的数据，而无需知道方法内部的实现细节。

------

## 继承与多态

在定义类的时候，可以在括号里写继承的类。

子类的好处：

- 会继承父类的属性和方法
- 可以自己定义，覆盖父类的属性和方法

### 调用父类的方法

一个类继承了父类后，可以直接调用父类的方法的。

```python
class Animals(object):
    def __init__(self, name, qty):
        self.name = name
        self.qty = qty

    def print_qty(self):
        print("{}:{}".format(self.name, self.qty))

class Dogs(Animals):
    pass
```

### 父类方法的重写

```python
class Animals(object):
    def __init__(self, name, qty):
        self.name = name
        self.qty = qty

    def print_qty(self):
        print("{}:{}".format(self.name, self.qty))

class Dogs(Animals):
    def print_qty(self):
        print("DOGS:{}-{}".format(self.name, self.qty))
```

```bash
DOGS:Husky-3
```

### 继承关系

继承关系，类似生物中种属关系。判断 `class` 类型，可以观察出。

```python
cat = Animals('cats', 2)
dog = Animals('dogs', 3)
husky = Dogs('Husky', 2)

print(isinstance(cat, Animals))
print(isinstance(dog, Animals))
print(isinstance(husky, Dogs))
print(isinstance(husky, Animals))
```

```python
True
True
True
True
```

当定义一个 `class` 的时候，实际上定义了一种数据类型。定义的数据类型和 Python 自带的数据类型，比如str、list、dict没什么两样。

------

### 多态

有了继承，才有了多态。继承的另一个好处：多态。

```python
class Animals(object):
    def __init__(self, name, qty):
        self.name = name
        self.qty = qty

    def print_qty(self):
        print("{}:{}".format(self.name, self.qty))

class Dogs(Animals):
    def print_qty(self):
        print("DOGS:{}-{}".format(self.name, self.qty))

class Cats(Animals):
    def print_qty(self):
        print("CATS:{}-{}".format(self.name, self.qty))

def print_twice(Animals):
    Animals.print_qty()
    Animals.print_qty()

husky = Dogs('Husky', 2)
siamese  = Cats('Siamese', 2)
print_twice(husky)
print_twice(siamese)
```

对于 `print_twice()`，任何依赖 `Animals` 作为参数的函数或者方法都可以不加修改地正常运行，原因就在于多态。

对于一个变量，我们只需要知道它是 `Animals` 类型，无需确切地知道它的子类型，就可以放心地调用 `print_qty()` 方法。而具体调用的 `print_qty()` 方法是作用在 `Animals`、`Dogs`、`Cats` 对象上，由运行时该对象的确切类型决定，这就是多态真正的威力：调用方只管调用，不管细节，而当我们新增一种 `Animals` 的子类时，只要确保 `print_qty()` 方法编写正确，不用管原来的代码是如何调用的。这就是著名的“开闭”原则：

- 对扩展开放：允许新增子类；
- 对修改封闭：不需要修改依赖 class 类型的方法等函数。

## 多态和鸭子类型

> **Is it same like Polymorphism???** 
>
> No.
>
>  **Polymorphism** means a method of a class can do different things in subclasses.  For example: a class Animal can have a method talk () and the subclasses.  Dog and Cat of Animal can let the method talk () make different sounds.  
>
>  **Duck typing** means code will simply accept any object that has a particular method.  For example: animal.quack(). If the given object animal has the method we  want to call then we're good (no additional type requirements needed).  It does not matter whether animal is actually a Duck.   
>
> Duck  Typing and Polymorphism are simply separate features that a programming  language may have. There are programming languages which have  polymorphism but that do not have duck typing (such as Java). There are  also languages that have polymorphism and duck typing (such as Python).

------

OOP 的特点就是 数据封装和归一化，由此产生 继承和多态。目的是对 数据和逻辑 封装，提供用户简单的接口。

------

PS:

`getattr()`、`setattr()` 以及 `hasattr()`

`getattr()` 函数用于返回一个对象属性值。

```python
getattr(object, name[, default])
```

- `object` -- 对象。
- `name` -- 字符串，对象属性。
- `default` -- 默认返回值，如果不提供该参数，在没有对应属性时，将触发 `AttributeError`。

`setattr()` 函数对应函数 `getatt()`，用于设置属性值，该属性必须存在。

```python
setattr(object, name, value)
```

- `object` -- 对象。
- `name` -- 字符串，对象属性。
- `value` -- 属性值。

`hasattr()` 函数用于判断对象是否包含对应的属性。

```python
hasattr(object, name)
```

- `object` -- 对象。
- `name` -- 字符串，属性名。

如果对象有该属性返回 True，否则返回 False。

具体应用如下：

```python
class Animals(object):
    def __init__(self):
        self.name = 'Animals'
        self.qty = 100

    def print_qty(self):
        print("{}:{}".format(self.name, self.qty))


animals = Animals()
if hasattr(animals, 'name'):
    print(getattr(animals, 'name'))
print(getattr(animals, 'id', 404))
if hasattr(animals, 'id') == False:
    setattr(animals, 'id', '001')
    print(getattr(animals, 'id'))
if hasattr(animals, 'print_qty'):
    print(getattr(animals, 'print_qty'))
    fn = getattr(animals, 'print_qty')
    fn()
```

```bash
Animals
404
001
<bound method Animals.print_qty of <__main__.Animals object at 0x000001CC095FCE10>>
Animals:100
```

------

参考:

[廖雪峰](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431865288798deef438d865e4c2985acff7e9fad15e3000)

[What is duck typing?](https://www.quora.com/What-is-duck-typing)