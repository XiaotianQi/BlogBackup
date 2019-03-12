## 类也是对象

在大多数编程语言中，类就是一组用来描述如何生成一个对象的代码段。

```bash
In [1]: class ObjectCreator:
   ...:     pass
   ...:

In [2]: my_obj = ObjectCreator()

In [3]: my_obj
Out[3]: <__main__.ObjectCreator at 0x27d44c480f0>
```

但是，在Python中的类还远不止如此。类同样也是一种对象。只要使用关键字class，Python解释器在执行的时候就会创建一个对象。

将在内存中创建一个对象，名字就是ObjectCreator。**这个对象（类）自身拥有创建对象（类实例）的能力，而这就是为什么它是一个类的原因。**但是，它的本质仍然是一个对象，于是乎你可以对它做如下的操作：

1. 可以将它赋值给一个变量
2. 可以拷贝它
3. 可以为它增加属性
4. 可以将它作为函数参数进行传递

***

## 动态地创建类

因为类也是对象，那可以在运行时动态的创建它们，就像其他任何对象一样。首先，可以在函数中创建类，使用class关键字即可。

```bash
In [4]: def choose_class(name):
   ...:     if name=='foo':
   ...:         class Foo:
   ...:             pass
   ...:         return Foo	# 返回的是类，不是类的实例
   ...:     else:
   ...:         class Bar:
   ...:             pass
   ...:         return Bar
   ...:

In [5]: MyClass = choose_class('foo')

In [6]: MyClass		# 函数返回的是类，不是类的实例
Out[6]: __main__.choose_class.<locals>.Foo

In [7]: MyClass()	# 通过这个类创建类实例，也就是对象
Out[7]: <__main__.choose_class.<locals>.Foo at 0x27d44b17668>
```

type可以接受一个类的描述作为参数，然后返回一个类。

使用type()动态创建类。

`type(name, bases, dict)`

- With **one argument**, return the type of an *object*.  The return value is a type object and generally the same object as returned by `object.__class__`.
- With **three arguments**, return a new type object.  This is essentially a dynamic form of the [`class`](https://docs.python.org/3/reference/compound_stmts.html#class) statement. 
  - The *name* string is the class name and becomes the `__name__`  attribute; 
  - the *bases* tuple itemizes the base classes and becomes the `__bases__` attribute; 
  - the *dict* dictionary is the namespace containing definitions for class body and is copied to a standard dictionary to become the `__dict__` attribute.  

```bash
In [11]: ObjCreator = type('ObjCreator', (), {})	# 返回一个类对象

In [12]: ObjCreator
Out[12]: __main__.ObjCreator

In [13]: ObjCreator()	#  创建一个该类的实例
Out[13]: <__main__.ObjCreator at 0x27d44cda710>
```

```python
class X:
	a = 1
# 可以写为：
X = type('X', (object,), dict(a=1))
```

最终希望为类增加方法。只需要定义一个有着恰当签名的函数并将其作为属性赋值就可以了。

```bash
In [14]: def foo(self):
    ...:     print(self.bar)
    ...:

In [15]: Obj_Creator = type('Obj_Creator', (),{'foo':foo})

In [16]: hasattr(Obj_Creator, 'foo')
Out[16]: True
```

在Python中，类也是对象，你可以动态的创建类。这就是当使用关键字class时Python在幕后做的事情，而这就是通过元类来实现的。

***

## 元类

元类就是用来创建类的“东西”。创建类就是为了创建类的实例对象，不是吗？但是已经学习到了Python中的类也是对象。好吧，元类就是用来创建这些类（对象）的，元类就是类的类，可以理解为：

```python
MyClass = MetaClass()
MyObject = MyClass()
```

type可以像这样做：

```python
MyClass = type('MyClass', (), {})
```

这是因为函数type实际上是一个元类。type就是Python在背后用来创建所有类的元类。type就是创建类对象的类。Python中所有的东西，注意，所有的东西——都是对象。这包括整数、字符串、函数以及类。它们全部都是对象，而且它们都是从一个类创建而来。元类创建了Python中所有的对象。

```bash
In [19]: x = 5

In [20]: type(x)
Out[20]: int

In [21]: type(type(x))
Out[21]: type
```

元类就是创建类这种对象的东西。type就是Python的内建元类，当然了，也可以创建自己的元类。

### 设置元类

#### python2

在python 2中，在写一个类的时候为其添加`__metaclass__`属性。

```python
class Foo(object):
	__metaclass__ = something…
[…]
```

如果这么做了，Python就会用元类来创建类Foo。小心点，这里面有些技巧。你首先写下`class  
Foo(object)`，但是类对象Foo还没有在内存中创建。Python会在类的定义中寻找`__metaclass__`属性，如果找到了，Python就会用它来创建类Foo，如果没有找到，就会用内建的type来创建这个类。

当代码如下时 :

```python
class Foo(Bar):
    pass
```

Python做了如下的操作：

Foo中有`__metaclass__`这个属性吗？

* 如果是，Python会在内存中通过`__metaclass__`创建一个名字为Foo的**类对象**（我说的是类对象，请紧跟我的思路）。
* 如果Python没有找到`__metaclass__`，它会继续在Bar（父类）中寻找`__metaclass__`属性，并尝试做和前面同样的操作。
  * 如果Python在任何父类中都找不到`__metaclass__`，它就会在模块层次中去寻找`__metaclass__`，并尝试做同样的操作。
    * 如果还是找不到`__metaclass__`,Python就会用**内置的type来创建这个类对象**。

由此可发现，python创建class的机制：

当创建class的时候，python会先检查当前类中有没有`__metaclass__`，如果有，就用此方法创建对象；如果没有，则会一级一级的检查父类中有没有`__metaclass__`，用来创建对象。创建的这个“对象”，就是当前的这个类。如果当前类和父类都没有，则会在当前package中寻找`__metaclass__`方法，如果还没有，则会调用自己隐藏的的type函数来创建对象。

#### python3

Python 3中更改了设置元类的语法：

```python
class Foo(object, metaclass=something):
    ...
```

不再使用元类`__metaclass__` 属性，而是使用基类列表中的关键字参数。

然而元类的行为在很大程度上保持不变。

PS:

`__new__` 和 `__init__` 区别：

```text
__new__(cls, name, bases, attrs)
__init__(self, attrs)
```

Use `__new__` when you need to control   the creation of a new instance. Use   `__init__` when you need to control initialization of a new instance.

`__new__` is static class method, while `__init__` is instance method. `__new__` has to create the instance first, so `__init__` can initialize it. Note that `__init__` takes `self` as parameter. Until you create instance there is no `self`.

`__new__` is the first step of instance creation.  It's called first, and is responsible for **returning** a new   instance of your class.  In contrast,   `__init__` **doesn't return** anything; it's only responsible for initializing the instance after it's been created.

In general, you shouldn't need to override `__new__` unless you're subclassing an immutable type like   str, int, unicode or tuple.

***

### 自定义元类

元类的主要目的就是为了当创建类时能够**自动地改变类**。通常，会为API做这样的事情，你希望可以创建符合当前上下文的类。

假想一个很傻的例子，决定将模块里所有的类的属性都应该是大写形式。有好几种方法可以办到，但其中一种就是通过在模块级别设定`__metaclass__`。采用这种方法，这个模块中的所有类都会通过这个元类来创建，只需要告诉元类把所有的属性都改成大写形式就万事大吉了。

幸运的是，`__metaclass__`实际上可以被任意调用，它并不需要是一个正式的类（我知道，某些名字里带有‘class’的东西并不需要是一个class，画画图理解下，这很有帮助）。所以，我们这里就先以一个简单的函数作为例子开始。

```python
# remember that `type` is actually a class like `str` and `int`
# so you can inherit from it
class UpperAttrMetaclass(type):
    # __new__ is the method called before __init__
    # it's the method that creates the object and returns it
    # while __init__ just initializes the object passed as parameter
    # you rarely use __new__, except when you want to control how the object
    # is created.
    # here the created object is the class, and we want to customize it
    # so we override __new__
    # you can do some stuff in __init__ too if you wish
    # some advanced use involves overriding __call__ as well, but we won't
    # see this
    def __new__(upperattr_metaclass, future_class_name,
                future_class_parents, future_class_attr):

        uppercase_attr = {}
        for name, val in future_class_attr.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val

        return type(future_class_name, future_class_parents, uppercase_attr)
```

但是，这种方式其实不是OOP。直接调用了type，而且没有改写父类的`__new__`方法。现在这样处理:

```python
class UpperAttrMetaclass(type):

    def __new__(upperattr_metaclass, future_class_name,
                future_class_parents, future_class_attr):

        uppercase_attr = {}
        for name, val in future_class_attr.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val

        # reuse the type.__new__ method
        # this is basic OOP, nothing magic in there
        return type.__new__(upperattr_metaclass, future_class_name,
                            future_class_parents, uppercase_attr)
```

可能已经注意到了有个额外的参数upperattr_metaclass，这并没有什么特别的。类方法的第一个参数总是表示当前的实例，就像在普通的类方法中的self参数一样。当然了，为了清晰起见，这里的名字我起的比较长。但是就像self一样，所有的参数都有它们的传统名称。因此，在真实的产品代码中一个元类应该是像这样的：

```python
class UpperAttrMetaclass(type):

    def __new__(cls, clsname, bases, dct):

        uppercase_attr = {}
        for name, val in dct.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val

        return type.__new__(cls, clsname, bases, uppercase_attr)
```

如果使用super方法的话，还可以使它变得更清晰一些，这会简化继承（是的，可以拥有元类，从元类继承，从type继承）

```python
class UpperAttrMetaclass(type):

    def __new__(cls, clsname, bases, dct):

        uppercase_attr = {}
        for name, val in dct.items():
            if not name.startswith('__'):
                uppercase_attr[name.upper()] = val
            else:
                uppercase_attr[name] = val

        return super(UpperAttrMetaclass, cls).__new__(cls, clsname, bases, uppercase_attr)
```

在python3中，如果可以使用关键字参数来做这个调用，就像这样

```python
class Foo(object, metaclass=Thing, kwarg1=value1):
    ...
```

其元类：

```python
class Thing(type):
    def __new__(cls, clsname, bases, dct, kwargs1=default):
        ...
```

就是这样，除此之外，关于元类真的没有别的可说的了。使用到元类的代码比较复杂，这背后的原因倒并不是因为元类本身，而是因为你通常会使用元类去做一些晦涩的事情，依赖于自省，控制继承等等。确实，用元类来搞些“黑暗魔法”是特别有用的，因而会搞出些复杂的东西来。但就元类本身而言，它们其实是很简单的：

1. 拦截类的创建
2. 修改类
3. 返回修改之后的类

给我们自定义的MyList增加一个`add`方法：

```python
class ListMetaclass(type):
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return super().__new__(cls, name, bases, attrs)

class MyList(list, metaclass=ListMetaclass):
    pass

l = MyList()
print(hasattr(l, 'add'))	# True
l.add(1)
print(l)	# [1]
```

值得注意的是，如果我们在做类的定义时，在class声明处传入关键字metaclass=ListMetaclass，那么如果传入的这个metaclass有`__call__`函数，这个`__call__`函数将会覆盖掉MyList class的`__new__`函数。这是为什么呢？请大家回想一下，当我们实例化MyList的时候，用的语句是L1=MyList()，而我们知道，`__call__`函数的作用是能让类实例化后的对象能够像函数一样被调用。也就是说MyList是ListMetaclass实例化后的对象，而MyList()调用的就是ListMetaclass的`__call__`函数。另外，值得一提的是，如果class声明处，我们是让MyList继承ListMetaclass，那么ListMetaclass的`__call__`函数将不会覆盖掉MyList的`__new__`函数。

```python
class ListMetaclass(type):
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return super().__new__(cls, name, bases, attrs)
    
    def __call__(self):
        return 'call'

class MyList(list, metaclass=ListMetaclass):
    pass

l = MyList()
print(hasattr(l, 'add'))	# False
print(l)					# call
```

***

**为什么要用metaclass类而不是函数?**

由于`__metaclass__`可以接受任何可调用的对象，那为何还要使用类呢，因为很显然使用类会更加复杂啊？这里有好几个原因：

1. 意图会更加清晰。当读到UpperAttrMetaclass(type)时，会知道接下来要发生什么。
2. 可以使用OOP编程。元类可以从元类中继承而来，改写父类的方法。元类甚至还可以使用元类。
3. 可以把代码组织的更好。当使用元类的时候肯定不会是像上面举的这种简单场景，通常都是针对比较复杂的问题。将多个方法归总到一个类中会很有帮助，也会使得代码更容易阅读。
4. 可以使用`__new__`, `__init__`以及`__call__`这样的特殊方法。它们能处理不同的任务。就算通常把所有的东西都在`__new__`里处理掉，有些人还是觉得用`__init__`更舒服些。
5. 哇哦，这东西的名字是metaclass，肯定非善类，我要小心！

**究竟为什么要使用元类？**

现在回到我们的大主题上来，究竟是为什么你会去使用这样一种容易出错且晦涩的特性？好吧，一般来说，你根本就用不上它：

> “元类就是深度的魔法，99%的用户应该根本不必为此操心。如果你想搞清楚究竟是否需要用到元类，那么你就不需要它。那些实际用到元类的人都非常清楚地知道他们需要做什么，而且根本不需要解释为什么要用元类。”  —— Python界的领袖 Tim Peters

元类的主要用途是创建API。一个典型的例子是Django ORM。它允许像这样定义：

```python
class Person(models.Model):
    name = models.CharField(max_length=30)
    age = models.IntegerField()
```

但是如果像这样做的话：

```python
guy  = Person(name='bob', age='35')
print guy.age
```

这并不会返回一个IntegerField对象，而是会返回一个int，甚至可以直接从数据库中取出数据。这是有可能的，因为models.Model定义了`__metaclass__`，并且使用了一些魔法能够将你刚刚定义的简单的Person类转变成对数据库的一个复杂hook。Django框架将这些看起来很复杂的东西通过暴露出一个简单的使用元类的API将其化简，通过这个API重新创建代码，在背后完成真正的工作。

分离功能，提供便捷的接口，便于用户使用。

涉及的4个角色：

- 描述符：重载`__set__`、`__get__`，设置参数输入检查
- 元类：重载`__new__`，分离实例创建方法
- 中间类：`class A(metaclass=ModelMetaClass):`完成描述符和元类的任务，并分离子类部分方法，从而减少子类的代码
- 子类:`class B(A):`专注于业务需求实现

**结语**

首先，你知道了类其实是能够创建出类实例的对象。好吧，事实上，类本身也是实例，当然，它们是元类的实例。

```bash
In [1]: class Foo:
   ...:     pass
   ...:

In [2]: id(Foo)
Out[2]: 2120528661480
```

Python中的一切都是对象，它们要么是类的实例，要么是元类的实例，除了type。type实际上是它自己的元类，在纯Python环境中可不能够做到的，这是通过在实现层面耍一些小手段做到的。其次，元类是很复杂的。对于非常简单的类，可能不希望通过使用元类来对类做修改。可以通过其他两种技术来修改类：

1. Monkey patching
2. class decorators

当需要动态修改类时，99%的时间里最好使用上面这两种技术。当然了，其实在99%的时间里根本就不需要动态修改类 :D

***

参考：

[Why is __init__() always called after __new__()?](https://stackoverflow.com/questions/674304/why-is-init-always-called-after-new)

https://stackoverflow.com/questions/100003/what-are-metaclasses-in-python

https://www.cnblogs.com/ArsenalfanInECNU/p/9036407.html