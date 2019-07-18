Python3的继承机制不同于Python2。其核心原则是下面两条，请谨记！

- 子类在调用某个方法或变量的时候，首先在自己内部查找，如果没有找到，则开始根据继承机制在父类里查找。
- 根据父类定义中的顺序，以深度优先的方式逐一查找父类！
- 继承参数的书写有先后顺序，写在前面的被优先继承。

## 多重继承

通过多重继承，一个子类就可以同时获得多个父类的所有功能。

```python
class Runnable(object):
    def run(self):
        print("I can run...")

class Swimable(object):
    def swim(self):
        print("I can swim...")

class Frog(Runnable, Swimable):
    pass
```

```python
print(Frog.__bases__)
f = Frog()
f.run()
f.swim()
```

```python
(<class '__main__.Runnable'>, <class '__main__.Swimable'>)
I can run...
I can swim...
```

***

## MRO

mro 即 method resolution order (方法解释顺序)，主要用于在多继承时判断属性的路径(来自于哪个类)，决定基类中的函数到底应该以什么样的顺序调用父类中的函数。Python 采用的是 C3 算法，简单理解就是取最左原则。

与拓扑排序(Topological Sorting)相似。拓扑排序是一个有向无环图(DAG,Directed Acyclic Graph)的所有顶点的线性序列。

### 例1

```python
class A(object):
    def foo(self):
        print('A foo')
    def bar(self):
        print('A bar')

class B(object):
    def foo(self):
        print('B foo')
    def bar(self):
        print('B bar')

class C1(A):
    pass

class C2(B):
    def bar(self):
        print('C2-bar')

class D(C1,C2):
    pass
```

继承关系如下图：

![继承关系](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/OOP%20MRO-1.png)

D --> C1 --> A --> C2 --> B --> Object

验证：

```python
if __name__ == '__main__':
    print(D.__mro__)
    d=D()
    d.foo()
    d.bar()
```

```python
(<class '__main__.D'>, <class '__main__.C1'>, <class '__main__.A'>, <class '__main__.C2'>, <class '__main__.B'>, <class 'object'>)
A foo
A bar
```

### 例2

```python
class A(object):
    def foo(self):
        print('A foo')
    def bar(self):
        print('A bar')

class B(object):
    def foo(self):
        print('B foo')
    def bar(self):
        print('B bar')

class C1(A,B):
    pass

class C2(A,B):
    def bar(self):
        print('C2-bar')

class D(C1,C2):
    pass
```

继承关系如下图：

![继承关系](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/OOP%20MRO-2.png)

D --> C1 --> C2 --> A --> B --> Object

验证：

```python
if __name__ == '__main__':
    print(D.__mro__)
    d=D()
    d.foo()
    d.bar()
```

```python
(<class '__main__.D'>, <class '__main__.C1'>, <class '__main__.C2'>, <class '__main__.A'>, <class '__main__.B'>, <class 'object'>)
A foo
C2-bar
```

### 例3

继承关系如下图：

![继承关系](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/OOP%20MRO-3.png)

E --> D1 --> D2 --> A --> B --> C --> Object

***

## super() 函数

在子类中如果有与父类同名的成员，那就会覆盖掉父类里的成员。那如果你想强制调用父类的成员呢？使用super()函数！这是一个非常重要的函数，最常见的就是通过super调用父类的实例化方法`__init__`！

`super()` 查找循序亦是通过 C3 算法，遵循 MRO顺序。调用的方法准确来说不是父类的方法，而是继承顺序中下一个类的方法。

```python
class A:
    def __init__(self):
        print('A')

class B(A):
    def __init__(self):
        print('B')
        super().__init__()

class C(A):
    def __init__(self):
        print('C')
        super().__init__()

class D(B, C):
    def __init__(self):
        print('D')
        super().__init__()
```

```python
print(D.__mro__)
d = D()
```

```text
(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
D
B
C
A
```

在如下例子中：

```python
class A:
    def test(self):
        print('A.test')

class TestMixin:
    def test(self):
        print('TestMixin.test')
        super().test()

class B(TestMixin, A):
    def test(self):
        print('B.test')
        super().test()


b = B()
b.test()

```

TestMixin中的test函数中通过super调到了A中的test函数，但是A不是TestMixin的父类。在这个继承体系中，A和TestMixin都是B的父类，但是A和TestMixin没有任何继承关系。为什么TestMixin中的super会调到A中的test函数呢？

其实super不是针对调用父类而设计的，它的本质是在一个由多个类组成的有序集合中搜寻一个特定的类，并找到这个类中的特定函数，将一个实例绑定到这个函数上，生成一个绑定方法(bound method)，并返回这个bound method。

其mro为：

```python
(<class '__main__.B'>, <class '__main__.TestMixin'>, <class '__main__.A'>, <class 'object'>)
```

可见B的mro为`(B, TestMixin, A, object)`。这个列表的意义是B的实例b在调用一个函数时，首先在B类中找这个函数，如果B中调用了`super`，则需要从B的下一个类(即TestMixin)中找函数，如果在TestMixin中又调用了`super`，则从TestMixin的下一个类(即A)中找函数。

除此之外，`super()`也经常用于防止递归调用：

```python
class A:
    def __setattr__(self, value):
        do something
        super().__setattr__(name, value)
```

```python
class TestMetaClass(type):
    def __new__(cls, name, bases, attrs):
		do somthing
        return super().__new__(cls, name, bases, attrs)
```

***

## MixIn

一般编程语言都不允许多重继承，主要是为了避免 diamond problem，即两个父类如果有共同的祖父类，但对祖父类相同部分做了不同的修改，则这个类再继承两个父类就会产生冲突。类似于 git 版本控制中，如果两个人对同一段代码做了不同的修改，则合并时 就需要手动解决冲突。

Mixin 本意是混入，程序中用来将不同功能(functionality)组合起来，从而为类提供多种特性。虽然继承(inheritance)也可以实现多种功能，但继承一般有从属关系，即子类通常是父类更加具体的类。不过，mixin 则更多的是功能上的组合，因而相当于是接口（带实现的接口）。

由于 mixin 是组合，因而是做加法，为已有的类添加新功能，而不像继承一样一级会覆盖上一级相同的属性或方法，但在某些方面仍然表现得与继承一样，例如类的实例也是每个 mixin 的实例。

python 对于 mixin 命名方式一般以 MixIn, able, ible 为后缀。

一般在一下两种情况下使用 MixIn 类：

> 1. You want to provide a lot of optional features for a class.
> 2. You want to use one particular feature in a lot of different classes.

### 多重继承和MixIn的区别

MixIn 是多重继承的一种特殊方式。

从某种程度上来说，继承强调 I am，Mixin 强调 I can，符合 `Duck Typing`。

优先考虑通过多重继承来组合多个Mixin的功能，而不是设计多层次的复杂的继承关系，从而降低多重继承的复杂关系。

### MixIn 类

mixin 类的特点：

* 功能单一，代码简洁，易于理解；
* 不能继承其他类(只能继承 `object`)，可以和任何类组合；
* 没有实例属性(可以有类常量)，只有 method。即：不存在实例属性(没有`__init`)，不应该被实例化，从而在被宿主类中调用 `super` 也就不涉及到调用混乱的情况。

创建 MixIn 类时，应遵循：

*  不具有实例属性
*  不能继承普通类
*  仅提供 接口和功能
*  命名方式一般以 MixIn, able, ible 为后缀

按照以上的原则，类在层次上具有单一继承一样的树结构，同时又可以实现功能的共享（方法是：把共享的功能放在 Mix-in 类中，再把 Mix-in 类插入到树结构里）。

```python
class TagMixin(object):
 
    def add_tag(self, tag_id):
        sql = ('insert into target_tagged'
               ' (target_id, target_kind, tag_id, creation_time) '
               'values (?, ?, ?, CURRENT_TIMESTAMP)')
        params = (self.ident, self.kind, tag_id)
        storage.execute(sql, params)
        storage.commit()
 
    def get_tags(self):
        sql = ('select tag_id, creation_time from target_tagged '
               'where target_id = ? and target_kind = ?')
        params = (self.ident, self.kind)
        cursor = storage.execute(sql, params)
        return cursor.fetchall()
 
 
class Post(Model, TagMixin):

    kind = 1001
 
    def __init__(self, ident, title):
        self.ident = ident
        self.title = title
 
    def __repr__(self):
        return 'Post(%r, %r)' % (self.ident, self.title)
```

以上代码来自[松鼠奥利奥](https://www.zhihu.com/question/20778853/answer/42943272)。

***

参考：

[廖雪峰](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014318680104044a55f4a9dbf8452caf71e8dc68b75a18000)

[python 多重继承之拓扑排序](https://kevinguo.me/2018/01/19/python-topological-sorting/)

[Mixin是什么概念?](https://www.zhihu.com/question/20778853)

[What is a mixin, and why are they useful?](https://stackoverflow.com/questions/533631/what-is-a-mixin-and-why-are-they-useful)

[深度解析并实现python中的super](https://blog.csdn.net/zhangjg_blog/article/details/83033210)，张纪刚