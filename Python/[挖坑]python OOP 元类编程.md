分离功能，提供便捷的接口，便于用户使用。



涉及的4个角色：

* 描述符：重载`__set__`、`__get__`，设置参数输入检查
* 元类：重载`__new__`，分离实例创建方法
* 中间类：`class A(metaclass=ModelMetaClass):`完成描述符和元类的任务，并分离子类部分方法，从而减少子类的代码
* 子类:`class B(A):`专注于业务需求实现



`__new__` 和 `__init__` 区别：

> Use `__new__` when you need to control   the creation of a new instance. Use   `__init__` when you need to control initialization of a new instance.
>
> `__new__` is static class method, while `__init__` is instance method. `__new__` has to create the instance first, so `__init__` can initialize it. Note that `__init__` takes `self` as parameter. Until you create instance there is no `self`.
>
> `__new__` is the first step of instance creation.  It's called first, and is   responsible for returning a new   instance of your class.  In contrast,   `__init__` doesn't return anything; it's only responsible for initializing the   instance after it's been created.
>
> In general, you shouldn't need to   override `__new__` unless you're   subclassing an immutable type like   str, int, unicode or tuple.

***

PS：

`type(name, bases, dict)`

* With **one argument**, return the type of an *object*.  The return value is a type object and generally the same object as returned by `object.__class__`.
* With **three arguments**, return a new type object.  This is essentially a dynamic form of the [`class`](https://docs.python.org/3/reference/compound_stmts.html#class) statement. 
  * The *name* string is the class name and becomes the `__name__`  attribute; 
  * the *bases* tuple itemizes the base classes and becomes the `__bases__` attribute; 
  * the *dict* dictionary is the namespace containing definitions for class body and is copied to a standard dictionary to become the `__dict__` attribute.  

```python
class X:
	a = 1

X = type('X', (object,), dict(a=1))
```



***

参考：

[Why is __init__() always called after __new__()?](https://stackoverflow.com/questions/674304/why-is-init-always-called-after-new)