`obj` 属性被访问时：

    `obj.__dict__` --> `type(obj).__dict` --> `type(type(obj)).__dict__`

直至 `obj` 的基类，顺序符合 C3 算法。

具体步骤：

* 查看当前是否是 descriptor；
* 若是，则调用 `__set__` 或 `__getattribute`，进行属性读写；
* 若不是，则调用 `__setattr__` 或  `__getattribute__`，进行属性读写 。

即，descriptor 会改变默认的属性读写方式。

```python
class DataDes:
    def __init__(self):
        pass
    def __get__(self, instance, owner):
        print('DataDes.__get__')
    def __set__(self, instance, value):
        print('DataDes.__set__', value)
 

class DataDes_noget:
    def __init__(self):
        pass
    def __set__(self, instance, value):
        print('DataDes_noget.__set__', value)
       
    
class DataDes_noset:
    def __init__(self):
        pass
    def __get__(self, instance, owner):
        print('non_DataDes.__get__')
        
        
class Manage:
    datades = DataDes()
    data_noget = DataDes_noget()
    data_noset = DataDes_noset()
```

```python
In :a = Manage()

In : a.__dict__
Out: {}
```
无`__set__`：

```python
In : a.datades = 1
DataDes.__set__ 1       # 调用__set__

In : a.datades
DataDes.__get__

In : a.data_noset = 2   # 调用__setattribute__

In : a.data_noset
Out: 2

In : a.__dict__
Out: {'data_noset': 2}
```
无 `__get__`：

```python
In :a.__dict__['datades'] = 1

In :a.__dict__['data_noget'] = 2
```

```python
In : a.datades
DataDes.__get__          # 调用 __get__

In : a.data_noget
Out: 2                   # 调用 __getattribute__

In : a.__dict__
Out: {'datades': 1, 'data_noget': 2}
```

***

PS:

当使用 `obj.name` 或 `obj.name = value`时

```python
object.__setattribute__(self,name,value)
object.__getattribute__(self,name)
```

