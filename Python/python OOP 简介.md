面向对象编程（Object Oriented Programming），简称 OOP，是一种程序设计思想。OOP 把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数。把一组数据结构和处理它们的方法组成对象（object），把相同行为的对象归纳为类（class），通过类的封装（encapsulation）隐藏内部细节，通过继承（inheritance）实现类的特化（specialization）和泛化（generalization），通过多态（polymorphism）实现基于对象类型的动态分派。

将计算机程序视为一组对象的集合，对象之间可以互动，接受、处理和反馈彼此之间传递的信息，计算机程序的执行就是一系列消息在各个对象之间传递。

------

## 概述

### 类和对象

类是抽象的概念，而对象是具体的东西。对象都有属性和行为，每个对象都是独一无二的，而且对象一定属于某个类（型）。当把一大堆拥有共同特征的对象的静态特征（属性）和动态特征（行为）都抽取出来后，就可以定义出一个叫做“类”的东西。

### 定义类

以存储动物名称和数量信息为例，说明类的创建，以及类与实例的关系。

采用面向对象的程序设计思想，我们首选思考的不是程序的执行流程，而是 Animals 这个数据类型应该被视为对象。而后，`Animals` 这个对象需要含有 `name`(名称) 和 `qty`(数量)两个属性（Property）。最后，给对象发一个 `print_qty` 消息，把自己的数据打印出来。

```python
class Animals(object):
    # __init__是一个特殊方法用于在创建对象时进行初始化操作
    # 通过这个方法我们可以为动物对象绑定name和qty两个属性
    def __init__(self, name, qty):
        self.name = name
        self.qty = qty
    # 方法
    def print_qty(self):
        print("{}:{}".format(self.name, self.qty))
```

`class` 后面紧接着是类名，通常是首字符大写。紧接着是 `(object)`，表示该类是从哪个类继承下来的，若没有继承，则使用 `object` 类，这是所有类最终都会继承的类。

### 创建和使用实例

```python
# 实例化
cat = Animals('cat', 2)
dog = Animals('dog', 3)
# 引用
cat.print_qty()
dog.print_qty()
```

Class 是一种抽象概念，也具有模版的作用。实例（Instance）则是根据类创建出来的一个具体的“对象”，每个对象都拥有相同的方法，但各自的数据可能不同。

比如：`Animals` 就是个抽象的概念，`cat` 和 `dog` 是两个具体的 `Animals`。给对象发消息实际上就是调用对象对应的关联函数，我们称之为对象的方法（Method）。

类支持两种操作：属性引用和实例化。属性引用使用和 Python 中所有的属性引用一样的标准语法：`obj.name`。各个实例拥有的数据都互相独立，互不影响。

### 类和实例

面向对象最重要的概念就是类（Class）和实例（Instance）。类是一种数据类型。类与实例之间的关系，类似生物中种属关系。类是创建实例的模板，而实例则是一个具体的对象，各个实例拥有的数据都互相独立，互不影响。方法就是与实例绑定的函数，和普通函数不同，方法可以直接访问实例的数据。通过在实例上调用方法，我们就直接操作了对象内部的数据，但无需知道方法内部的实现细节。

### 相关概念

- 类(Class): 用来描述具有相同的属性和方法的对象的集合。它定义了该集合中实例共有的属性和方法
- 数据成员：类变量、实例变量、方法、类方法和静态方法等的统称
- 类变量：类变量是所有实例公有的变量。类变量定义在类中，但在方法体之外。类变量通常不作为实例变量使用
- 类属性：类中变量和方法都可被称作类属性，也可单指类变量
- 方法：类中定义的函数
- 静态方法：不需要实例化就可以由类执行的方法
- 类方法：类方法是将类本身作为对象进行操作的方法
- 方法重写：如果从父类继承的方法不能满足子类的需求，可以对其进行改写，这个过程叫方法的覆盖（override），也称为方法的重写
- 实例：也称对象。
- 实例化：创建一个类的实例，类的具体对象
- 实例变量：定义在方法中的变量，只作用于当前实例
- `self`：实例，其下属性是实例私有
- 封装：将内部实现包裹起来，对外透明，提供api接口进行调用的机制
- 继承：即一个派生类（derived class）继承基类（base class），从而获得继承基类的属性和方法。继承也允许把一个派生类的对象作为一个基类对象对待
- 多态：根据对象类型的不同以不同的方式进行处理。

------

## 属性

在类中，无论变量或者方法，都可以被称作属性。

类属性是类固有属性，不会因为实例不同而改变。因此，多实例时，能够减少所创造出来的内存空间，加快运行速度。类、类的所有方法以及类变量在内存中只有一份，所有的实例共享它们。而每一个实例都在内存中独立的保存自己和自己的实例变量。

### 变量

#### 类变量

定义在类中，方法之外的变量，称作类变量。类变量是所有实例公有的变量，每一个实例都可以访问、修改类变量。

```python
class Test(object):
    name = 'test'
```

如果类变量使用了可变对象，那么需要谨慎操作：

```python
class Dog:
    kind = 'canine'
    tricks = []

    def __init__(self, name):
        self.name = name

    def add_trick(self, trick):
        self.tricks.append(trick)
        
d = Dog('Fido')
e = Dog('Buddy')
d.add_trick('roll over')
e.add_trick('play dead')
d.kind = 'abc'
d.tricks 	# ['roll over', 'play dead']
d.kind		# 'abc'
e.kind		# 'canine'

f = Dog('AL')
f.tricks	# ['roll over', 'play dead']

Dog.kind	# 'canine'
Dog.tricks	# ['roll over', 'play dead']
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/OOP%20Variables.png)

将可变变量放入`__init__`后：

```python
class Dog:
    kind = 'canine'

    def __init__(self, name):
        self.name = name
        self.tricks = []

    def add_trick(self, trick):
        self.tricks.append(trick)
        
d = Dog('Fido')
e = Dog('Buddy')
d.add_trick('roll over')
e.add_trick('play dead')
d.kind = 'abc'
d.tricks 	# ['roll over']
d.kind		# 'abc'

e.kind		# 'canine'
e.tricks	# ['play dead']

f = Dog('AL')
f.tricks	# []

Dog.kind	# 'canine'
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/OOP%20Variables%202.png)

但是，当在类中也使用可变参数作为默认参数时：

```python
class Company:
    def __init__(self, name, staffs=[]):
        self.name = name
        self.staffs = staffs
    
    def add(self, staff_name):
        self.staffs.append(staff_name)


if __name__ == "__main__":
    com1 = Company('com1', ['a', 'b'])
    com1.add('c')
    print(com1.staffs)            # ['a', 'b', 'c']

    com2 = Company('com2')
    com2.add('a')
    print(com2.staffs)            # ['a']
    
    
    com3 = Company('com3')
    com3.add('b')
    print(com3.staffs)            # ['a', 'b']
    print(com2.staffs)            # ['a', 'b']
```

如下图：

* 实例化时，使用默认值，该实例与类与其他实例互不影响
* 实例化时，不使用默认值，该实例与类与相同方法的实例互相影响

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/OOP%20Variables%203.png)

#### 实例变量

```python
class Test(object):
    def __init__(self, name):
        self.name = 'test'
```

#### 变量的访问控制

`_attrs` 单下划线开头的变量，标明是一个受保护的变量，原则上不允许直接访问，但外部类还是可以访问到这个。只是一个约定，用于警告说明这是一个私有变量，外部类不要去访问它。

一般情况下，使用 `__private_attrs` 两个下划线开头，声明该属性为私有，不能在类地外部被使用或直接访问。无法通过 `实例变量.__private_attrs` 这个访问定义属性。在实际开发中，并不建议将属性设置为私有的，因为这会导致子类无法访问。通常遵循一种命名惯例就是让属性名以单下划线开头来表示属性是受保护的，本类之外的代码在访问这样的属性时应该要保持慎重。

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
    # 构造函数中的属性
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

实际上， Python 中是没有提供私有属性等功能的。但是 Python 对属性的访问控制是靠程序员自觉的。

可以说，python 中的属性都是公开的。

#### 类变量 VS 实例变量

- 一般来说，实例变量用于每个实例的唯一数据，而类变量用于类的所有实例共享的属性和方法。
- 共享数据可能在涉及 mutable 对象例如列表和字典的时候导致令人惊讶的结果。
- Python是动态语言，根据类创建的实例可以任意绑定属性给实例，即使绑定的属性没有在类属性中定义。

```python
cats = Animals('cat', 2, 123456)
cats.test='test'
```

- 不要把实例属性和类属性使用相同的名字，相同名称的实例属性将屏蔽掉类属性，实例属性优先级比类属性高。但是当删除实例属性后，再使用相同的名称，访问到的将是类属性。

------

### 方法

Python的类中包含实例方法、静态方法和类方法三种方法。这些方法无论是在代码编排中还是内存中都归属于类，区别在于传入的参数和调用方式不同。

self.__private_methods：两个下划线开头，声明该方法为私有方法，只能在类的内部调用 ，不能在类地外部调用。

#### 实例方法

##### `self` 参数

在类的内部，`self` 代表的是类的实例。使用 `def` 关键字来定义一个方法，与一般函数定义不同，参数 `self`, 作为第一个参数。

类的实例方法由实例调用，至少包含一个self参数，且为第一个参数。执行实例方法时，会自动将调用该方法的实例赋值给self。self代表的是类的实例，而非类本身。self不是关键字，而是Python约定成俗的命名，你完全可以取别的名字，但不建议这么做。

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

##### 访问实例方法

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

##### `__init__`

构造函数。类定义了 `__init__` 方法的话，实例化操作会自动调用 `__init__` 方法。生成实例时自动调用一次。生成其他实例时，再次自动调用。在实例化的时候体现实例的不同。

`__init__` 方法的第一个参数永远是 `self`，表示创建的实例本身，因此，在 `__init__` 方法内部，就可以把各种属性绑定到`self`，因为`self`就指向创建的实例本身。其他参数通过 `__init__` 传递到类的实例化操作上。

若有了 `__init__` 方法，则在创建实例的时候，就不能传入空的参数，必须传入与 `__init__`方 法匹配的参数。但 `self` 不需要传，Python 解释器会把实例变量传进去。

#### 静态方法

静态方法由类调用，无默认参数。将实例方法参数中的self去掉，然后在方法定义上方加上`@staticmethod`，就成为静态方法。它属于类，和实例无关。建议只使用`类名.静态方法`的调用方式。（虽然也可以使用`实例名.静态方法`的方式调用）

```python
class Foo:

    @staticmethod
    def static_method():
        pass

#调用方法
Foo.static_method()
```

#### 类方法

类方法由类调用，采用@classmethod装饰，至少传入一个cls（代指类本身，类似self）参数。执行类方法时，自动将调用该方法的类赋值给cls。建议只使用类名.类方法的调用方式。（虽然也可以使用实例名.类方法的方式调用）

```python
class Foo:

    @classmethod
    def class_method(cls):
        pass

Foo.class_method()
```

#### 成员保护和访问限制

往往我们也不希望所有的变量和方法能被外部访问，需要针对性地保护某些成员，限制对这些成员的访问。这样的程序才是健壮、可靠的，也符合业务的逻辑。

Python没有这个机制，Python利用变量和方法名字的变化，实现这一功能。在Python中，如果要让内部成员不被外部访问，可以在成员的名字前加上两个下划线__，这个成员就变成了一个私有成员（private）。私有成员只能在类的内部访问，外部无法访问。

- `_name`、`_name_`、`_name__`:建议性的私有成员，不要在外部访问。
- `__name`、 `__name_` :强制的私有成员，但是你依然可以蛮横地在外部危险访问。
- `__name__`:特殊成员，与私有性质无关，例如`__doc__`。
- `name_`、`name__`:没有任何特殊性，普通的标识符，但最好不要这么起名。

------

## 面向对象的支柱

面向对象有三大支柱：封装、继承和多态。OOP 的特点就是 数据封装和归一化，由此产生 继承和多态。目的是对 数据和逻辑 封装，提供用户简单的接口。

### 数据封装

封装是指将数据与具体操作的实现代码放在某个对象内部，使这些代码的实现细节不被外界发现，外界只能通过接口使用该对象，而不能通过任何形式修改对象内部实现。隐藏一切可以隐藏的实现细节，只向外界暴露（提供）简单的编程接口。

在类中定义的方法其实就是把数据和对数据的操作封装起来，在创建对象之后，只需要调用对象方法就可以执行方法中的代码，也就是说只需要知道方法的名字和传入的参数，而不需要知道方法内部的实现细节。

正是由于封装机制，程序在使用某一对象时不需要关心该对象的数据结构细节及实现操作的方法。使用封装能隐藏对象实现细节，使代码更易维护，同时因为不能直接调用、修改对象内部的私有信息，在一定程度上保证了系统安全性。类通过将函数和变量封装在内部，实现了比函数更高一级的封装。通过在实例上调用方法，就直接操作了对象内部的数据，而无需知道方法内部的实现细节。

示例：

```python
from time import sleep


class Clock(object):
    """数字时钟"""

    def __init__(self, hour=0, minute=0, second=0):
        """初始化方法

        :param hour: 时
        :param minute: 分
        :param second: 秒
        """
        self._hour = hour
        self._minute = minute
        self._second = second

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
    clock = Clock(23, 59, 58)
    while True:
        print(clock.show())
        sleep(1)
        clock.run()


if __name__ == '__main__':
    main()
```

```python
from math import sqrt


class Point(object):

    def __init__(self, x=0, y=0):
        """初始化方法
        
        :param x: 横坐标
        :param y: 纵坐标
        """
        self.x = x
        self.y = y

    def move_to(self, x, y):
        """移动到指定位置
        
        :param x: 新的横坐标
        "param y: 新的纵坐标
        """
        self.x = x
        self.y = y

    def move_by(self, dx, dy):
        """移动指定的增量
        
        :param dx: 横坐标的增量
        "param dy: 纵坐标的增量
        """
        self.x += dx
        self.y += dy

    def distance_to(self, other):
        """计算与另一个点的距离
        
        :param other: 另一个点
        """
        dx = self.x - other.x
        dy = self.y - other.y
        return sqrt(dx ** 2 + dy ** 2)

    def __str__(self):
        return '(%s, %s)' % (str(self.x), str(self.y))


def main():
    p1 = Point(3, 5)
    p2 = Point()
    print(p1)
    print(p2)
    p2.move_by(-1, 2)
    print(p2)
    print(p1.distance_to(p2))


if __name__ == '__main__':
    main()
```

------

### 继承

在已有类的基础上创建新类，这其中的一种做法就是让一个类从另一个类那里将属性和方法直接继承下来，从而减少重复代码的编写，而被继承的类称为基类、父类或超类（Base class、Super class）。得到继承信息的我们称之为子类，也叫派生类或衍生类。继承最大的好处是子类获得了父类的全部变量和方法的同时，又可以根据需要进行修改、拓展。

#### 调用父类的方法

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

#### 父类方法的重写

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

#### 继承关系

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

子类在继承了父类的方法后，可以对父类已有的方法给出新的实现版本，这个动作称之为方法重写（override）。通过方法重写可以让父类的同一个行为在子类中拥有不同的实现版本，当调用这个经过子类重写的方法时，不同的子类对象会表现出不同的行为，这个就是多态（poly-morphism）。

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

## 抽象类

```python
from abc import ABCMeta, abstractmethod


class Pet(object, metaclass=ABCMeta):
    """宠物"""

    def __init__(self, nickname):
        self._nickname = nickname

    @abstractmethod
    def make_voice(self):
        """发出声音"""
        pass


class Dog(Pet):
    """狗"""

    def make_voice(self):
        print('%s: 汪汪汪...' % self._nickname)


class Cat(Pet):
    """猫"""

    def make_voice(self):
        print('%s: 喵...喵...' % self._nickname)


def main():
    pets = [Dog('旺财'), Cat('凯蒂'), Dog('大黄')]
    for pet in pets:
        pet.make_voice()


if __name__ == '__main__':
    main()
```

在上面的代码中，将`Pet`类处理成了一个抽象类，所谓抽象类就是不能够创建对象的类，这种类的存在就是专门为了让其他类去继承它。Python从语法层面并没有像Java或C#那样提供对抽象类的支持，但是可以通过`abc`模块的`ABCMeta`元类和`abstractmethod`包装器来达到抽象类的效果，如果一个类中存在抽象方法那么这个类就不能够实例化（创建对象）。上面的代码中，`Dog`和`Cat`两个子类分别对`Pet`类中的`make_voice`抽象方法进行了重写并给出了不同的实现版本，当我们在`main`函数中调用该方法时，这个方法就表现出了多态行为（同样的方法做了不同的事情）。

抽象类是不完整的，它只能用作基类。在面向对象方法中，抽象类主要用来进行类型隐藏和充当全局变量的角色。目的就是让别的类继承它并实现特定的抽象方法。

***

## Duck Typing

In duck typing, an object's suitability is determined by **the *presence of certain* *methods and properties*** (with appropriate meaning), rather than the actual type of the object. 

The actual Apparel of an entity doesn't matter if the entity does all the intended things.

In other words, don't check whether it IS-A duck,check whether it QUACKS-like-a duck, WALKS-like-a duck, etc. depending on exactly what subset of duck-like behavior you  need for your program.

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

***

## 多态和鸭子类型

**Is it same like Polymorphism???** 

No.

**Polymorphism** means a method of a class can do different things in subclasses.  For example: a class Animal can have a method talk () and the subclasses.  Dog and Cat of Animal can let the method talk () make different sounds.  

**Duck typing** means code will simply accept any object that has a particular method.  For example: animal.quack(). If the given object animal has the method we  want to call then we're good (no additional type requirements needed).  It does not matter whether animal is actually a Duck.   

Duck  Typing and Polymorphism are simply separate features that a programming  language may have. There are programming languages which have  polymorphism but that do not have duck typing (such as Java). There are  also languages that have polymorphism and duck typing (such as Python).

考虑用于一个使用鸭子类型的语言的以下伪代码： 

```text
function calculate(a, b, c) => return (a+b)*c

example1 = calculate (1, 2, 3)
example2 = calculate ([1, 2, 3], [4, 5, 6], 2)
example3 = calculate ('apples ', 'and oranges, ', 3)

print to_string example1
print to_string example2
print to_string example3
```

在样例中，每次对`calculate`的调用都使用的对象（数字、列表和字符串）在继承关系中没有联系。只要对象支持“+”和“*”方法，操作就能成功。例如，翻译成python语言，运行结果应该是： 

```pyton
9
[1, 2, 3, 4, 5, 6, 1, 2, 3, 4, 5, 6]
apples and oranges, apples and oranges, apples and oranges, 
```

鸭子类型在**不使用继承**的情况下使用了多态，它并不要求严格的继承体系。

就`in_the_forest`函数而言，对象是一个鸭子：

```python
class Duck:

    def quack(self):
        print("這鴨子在呱呱叫")

    def feathers(self):
        print("這鴨子擁有白色與灰色羽毛")


class Person:
    
    def quack(self):
        print("這人正在模仿鴨子")

    def feathers(self):
        print("這人在地上拿起1根羽毛然後給其他人看")
        

def in_the_forest(duck):
    duck.quack()
    duck.feathers()

def game():
    donald = Duck()
    john = Person()
    in_the_forest(donald)
    in_the_forest(john)

game()
```

------

参考:

[继承和多态](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431865288798deef438d865e4c2985acff7e9fad15e3000)，廖雪峰

[What is duck typing?](https://www.quora.com/What-is-duck-typing)，Sujit Ingale

[鸭子类型](https://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B)，wiki

[面向对象编程基础](https://github.com/jackfrued/Python-100-Days/blob/master/Day01-15/08.%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80.md)，骆昊