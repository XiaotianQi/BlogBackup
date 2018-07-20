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

## MixIn

一般编程语言都不允许多重继承，主要是为了避免 diamond problem，即两个父类如果有共同的祖父类，但对祖父类相同部分做了不同的修改，则这个类再继承两个父类就会产生冲突。类似于 git 版本控制中，如果两个人对同一段代码做了不同的修改，则合并时 就需要手动解决冲突。

Mixin 本意是混入，程序中用来将不同功能(functionality)组合起来，从而为类提供多种特性。虽然继承(inheritance)也可以实现多种功能，但继承一般有从属关系，即子类通常是父类更加具体的类。不过，mixin 则更多的是功能上的组合，因而相当于是接口（带实现的接口）。

由于 mixin 是组合，因而是做加法，为已有的类添加新功能，而不像继承一样一级会覆盖上一级相同的属性或方法，但在某些方面仍然表现得与继承一样，例如类的实例也是每个 mixin 的实例。

python 对于 mixin 命名方式一般以 MixIn, able, ible 为后缀。

***

## 多重继承排序

mro 即 method resolution order (方法解释顺序)，主要用于在多继承时判断属性的路径(来自于哪个类)，决定基类中的函数到底应该以什么样的顺序调用父类中的函数。Python 采用的是 C3算法，简单理解就是取最左原则。

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

![继承关系](https://wx1.sinaimg.cn/mw690/af9e9c30ly1fsr7fizg3nj20960e9jre.jpg)

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

![继承关系](https://wx3.sinaimg.cn/mw690/af9e9c30ly1fsr7fjg2l0j209a0e9gln.jpg)

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

![继承关系](https://wx2.sinaimg.cn/mw690/af9e9c30ly1fsr8ilqgfuj208z0dtmx7.jpg)

E --> D1 --> D2 --> A --> B --> C --> Object

***

参考：

[廖雪峰](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014318680104044a55f4a9dbf8452caf71e8dc68b75a18000)

[python 多重继承之拓扑排序](https://kevinguo.me/2018/01/19/python-topological-sorting/)