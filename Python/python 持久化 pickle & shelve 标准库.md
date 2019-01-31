## pickle

json作为一种通用的数据交换格式和Python的持久化方式之一，只能对基本的一些内置数据类型（并且不是所有的）进行持久化。

而pickle模块则是Python专用的持久化模块，可以持久化包括自定义类在内的各种数据，比较适合Python本身复杂数据的存贮。pickle与json的操作基本一样，但是不同的是,它持久化后的字串是不可认读的，不如json的来得直观，并且只能用于Python环境，不能用作与其它语言进行数据交换，不通用。pickle序列化后的数据，可读性差，人一般无法识别。

### 主要方法

与json模块一模一样的方法名。但是在pickle中dumps()和loads()操作的是bytes类型，而不是json中的str类型；在使用dump()和load()读写文件时，要使用rb或wb模式，也就是只接收bytes类型的数据。

| 方法                       | 功能                                             |
| -------------------------- | ------------------------------------------------ |
| pickle.dump(obj, file)     | 将Python数据转换并保存到pickle格式的文件内       |
| pickle.dumps(obj)          | 将Python数据转换为pickle格式的bytes字串          |
| pickle.load(file)          | 从pickle格式的文件中读取数据并转换为python的类型 |
| pickle.loads(bytes_object) | 将pickle格式的bytes字串转换为python的类型        |

```text
pickle.dump(obj, file[, protocol])

obj: 要持久化保存的对象；
file: 一个拥有 write() 方法的对象，并且这个 write() 方法能接收一个字符串作为参数。这个对象可以是一个以写模式打开的文件对象或者一个 StringIO 对象，或者其他自定义的满足条件的对象。
protocol: 这是一个可选的参数，默认为 0 ，如果设置为 1 或 True，则以高压缩的二进制格式保存持久化后的对象，否则以ASCII格式保存。

pickle.load(file)

只有一个参数 file ，对应于上面 dump 方法中的 file 参数。这个 file 必须是一个拥有一个能接收一个整数为参数的 read() 方法以及一个不接收任何参数的 readline() 方法，并且这两个方法的返回值都应该是字符串。这可以是一个打开为读的文件对象、StringIO 对象或其他任何满足条件的对象。
```

### 简单示例

```python
import pickle
 
data = {
    'a': [1, 2.0, 3, 4+6],
    'b': ("character string", b"byte string"),
    'c': {None, True, False}
}
 
# 将 data 持久化保存到文件 tmp.txt 中
with open("data.txt", "wb") as f:
    pickle.dump(data, f)
 
# do something else ...
 
# 从 data.txt 中读取并恢复 data 对象
with open("data.txt", "rb") as f:
    obj = pickle.load(f)
    print(obj)
```

可以尝试用文本编辑器打开上面保存的data文件，会发现其中全是不可认读的编码。

```python
import pickle

a1 = 'apple'  
b1 = {1: 'One', 2: 'Two', 3: 'Three'}  
c1 = ['fee', 'fie', 'foe', 'fum']  
with open('temp.pkl', 'wb') as f:
    pickle.dump(a1, f)  
    pickle.dump(b1, f) 
    pickle.dump(c1, f)  

with open('temp.pkl', 'rb') as f:
    a2 = pickle.load(f)  
    print(a2)				# apple
    b2 = pickle.load(f)  
    print(b2)				# {1: 'One', 2: 'Two', 3: 'Three'}
    c2 = pickle.load(f)  
    print(c2)				# ['fee', 'fie', 'foe', 'fum']
```

`dump()`方法能一个接着一个地将几个对象转储到同一个文件。随后调用`load()`可以同样的顺序一个一个检索出这些对象。

Pickle可以持久化Python的自定义数据类型，但是在反持久化的时候，必须能够读取到类的定义。在代码中删除了Person类的定义，那么后面的`load()`方法将会出现错误。这一点很好理解，因为如果连数据的内部结构都不知道，pickle怎么能将数据正确的解析出来呢？

***

## shelve

shelve模块以类似字典的方式将Python对象持久化，它依赖于pickle模块，但比pickle用起来简单。当我们写程序的时候如果不想用关系数据库那么重量级的程序去存储数据，可以简单地使用shelve。shelve使用起来和字典类似，也是用key来访问的，键为普通字符串，值则可以是任何Python数据类型，并且支持所有字典类型的操作。

`shelve.open(filename, flag='c', protocol=None, writeback=False)`

创建或打开一个shelve对象。

```python
# _*_coding:utf-8_*_
import shelve
import datetime
 
info = {'name': 'bigberg', 'age': 22}
name = ['Apoll', 'Zous', 'Luna']
t = datetime.datetime.now()
 
with shelve.open('shelve.txt') as f:
    f['info'] = info                     # 持久化字典
    f['name'] = name                     # 持久化列表
    f['time'] = t                        # 持久化时间类型

with shelve.open('shelve.txt') as f:
    klist = list(f.keys())               # 列出所有的键    
    f['test'] = 'testdata'               # 存入数据
    i = f.get('info')                   
    n = f['name']                        # 获取数据
    del f['time']                        # 删除某个键值对
    flag = 'time' in f                   # 判断某个键是否在字典内

print(klist)    	# ['info', 'name', 'test', 'time']
print(i)			# {'name': 'bigberg', 'age': 22}
print(n)			# ['Apoll', 'Zous', 'Luna']
print(flag)			# False
```

使用`d=shelve.open(filename,writeback=True)`的方式可以让你正常进行`d['list'].append(3)`的操作。 但是这会消耗大量的内存，同时让`d.close()`操作变得缓慢。

shelve在默认情况下是不会记录对持久化对象的任何后续修改的！

如果我们想让shelve去自动捕获对象的变化，应该在打开shelve文件的时候将`writeback`参数设置为True。此时，shelve会将所有数据放到缓存中，并接收后续对数据的修改操作。最后，当我们`close()`的时候，缓存中所有的对象一次性写回磁盘内。

`writeback=True`有优点也有缺点。优点是可以动态修改数据，并减少出错的概率，让对象的持久化对用户更加的透明。但也有很大的缺点，在`open()`的时候会增加额外的内存消耗，并且当`close()`对象的时候会将缓存中的每一个细节都写回到文件系统，这也会带来额外的等待时间和计算消耗，因为shelve没有办法知道缓存中哪些对象修改了，哪些对象没有修改，所有的对象都必须被写入。

***

参考

http://www.liujiangblog.com/course/python/67