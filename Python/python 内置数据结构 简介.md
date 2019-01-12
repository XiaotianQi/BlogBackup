## 概览

在Python中，变量本身没有数据类型的概念，通常所说的“变量类型”是变量所引用的对象的类型，或者说是变量的值的类型。

Python 中常见的内置数据结构可以统称为容器（container）。

* 数字类型：`int`、`float`、`complex`、`bool`
* 序列类型：`list`、`tuple`、`str`、`range`、`bytes`、`bytearray`
* 映射类型：`dict`
* 集合类型：`set`、`frozenset`

各类型特点：

* 序列类型：有序、可索引、可迭代、可切片
* 映射类型：无序、key唯一、可迭代
* 集合类型：无序、不重复、可迭代

按照是否可变分类如下：

* 可变对象：
  * 序列类型：`list`、`bytearray`
  * 映射类型：`dict`
  * 集合类型：`set`

* 不可变对象：

  * 数字类型：`int`、`float`

  * 序列类型：`tuple`、`bytes`

  * 集合类型：`frozenset`

不可变对象都是可哈希的。

`dict`的`key`或者`set`的值 都必须是可哈希的


此外：

`dict`的查找性能强于`list`，但` dict` 的内存花销大，但是查询速度快。


在`list`中随着`list`数据的增大 查找时间会增大

在`dict`中查找元素不会随着`dict`的增大而增大

PS：

各个数据结构的抽象基类可以在官方文档查看：

[`collections.abc` — Abstract Base Classes for Containers](https://docs.python.org/3/library/collections.abc.html)

***

## Special Attributes

The implementation adds a few special read-only attributes to several object types, where they are relevant.  Some of these are not reported by the `dir()` built-in function.

-  `object.__dict__`

  A dictionary or other mapping object used to store an object’s (writable) attributes. 

-  `instance.__class__`

  The class to which a class instance belongs. 

-  `class.__bases__`

  The tuple of base classes of a class object. 

-  `definition.__name__`

  The name of the class, function, method, descriptor, or generator instance. 

-  `definition.__qualname__`

  The qualified name of the class, function, method, descriptor, or generator instance.  

-  `class.__mro__`

  This attribute is a tuple of classes that are considered when looking for base classes during method resolution. 

-  `class.mro`()

  This method can be overridden by a metaclass to customize the method resolution order for its instances.  It is called at class instantiation, and its result is stored in `__mro__`. 

-  `class.``__subclasses__`()

  Each class keeps a list of weak references to its immediate subclasses.  This method returns a list of all those references still alive. 

***

## 其他类型

除了以上常用内置数据类型，还有上下文管理类型、模块、函数等。

***

参考：

(Built-in Types)[https://docs.python.org/3/library/stdtypes.html#special-attributes]

