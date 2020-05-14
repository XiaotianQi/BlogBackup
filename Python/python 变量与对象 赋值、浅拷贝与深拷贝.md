## 赋值、浅拷贝、深拷贝

**赋值**操作 `=` 就是把一个变量绑定到一个对象上，建立一个对象的引用。 

浅拷贝和深拷贝的不同仅仅是对复杂对象来说，如嵌套列表。而对于非容器类型，如数字、字符串以及其它原子类型，即不可变类型，没有拷贝一说，产生的都是原对象的引用。

**浅拷贝**：创建新对象，其内部是原对象内部对象的引用。它仅仅只拷贝最外围一层，其内部的对象只拷贝了引用。 

**深拷贝**：创建一个新的对象，然后递归的拷贝原对象的内部对象。由深拷贝创建的对象与原对象没有任何关联。

对于内部对象是不可变对象：

* 浅拷贝和深拷贝的作用是一致的，原对象内部的不可变对象的改变，不会影响到复制对象。

对于内部对象是可变对象：

* 浅拷贝是拷贝了这些对象的引用。这些子对象改变时，会在原对象的对应元素上，有所体现。

* 深拷贝在遇到可变对象时，在内部新建了一个副本。所以，不管它内部的元素如何变化，都不会影响到原来副本的可变对象。

浅拷贝有三种形式： 切片，工厂函数，copy模块中的copy函数。

赋值不会新建对象，浅拷贝和深拷贝均会新建一个对象。对象包含其他对象的时候，拷贝操作可能会出现问题。

* 赋值是将变量指向某个对象
* 浅拷贝是拷贝第一层结构
* 深拷贝就是完全在新位置重造一个对象。

***

### 赋值

不可变对象，Python 用引用计数的方式管理它们，所以 Python 不会对值相同的不可变对象，申请单独的内存空间，只会记录它的引用次数。

```python
a = "hi abc 123"
b = "hi abc 123"
print(id(a), id(b))		# 2859878846992 2859878847152

a = 5
b = 5
print(id(a), id(b))		# 140724402348816 140724402348816
```

不可变对象发变化时，其标识也会随机变化。因此，不用担心，在进行赋值操作后，一个变量发生变化，另一个变量也随之变化。

```python
a = 5
b = a
print(id(a), id(b))		# 140731712955152 140731712955152

a = 7
print(a, b)				# 7 5
print(id(a), id(b))		# 140731712955216 140731712955152
```

可变对象，当对自身进行修改时，其标识不会发生改变。 

```python
a = [0, 1, 2]
b = [0, 1, 2]
print(id(a), id(b))		# 2200077174408 2200064644680

a[0] = -1
print(id(a), id(b))		# 2200077174408 2200064644680

a += [4, 5]
print(id(a), id(b))		# 2200077174408 2200064644680

a.append(6)
print(id(a), id(b))		# 2200077174408 2200064644680
```

对于可变对象，赋值操作仍是将变量绑定到一个对象上。 但是，当对其进行修改时，不会创建新的对象，因此所有引用该可变对象的变量，均受到影响。 

所以，当`b = a`时，变量`a` `b`均指向了同一个对象，无论二者哪个发生改变，另外一方都会受到影响。

```python
a = [0, 1, 2]
b = a
print(id(a), id(b))		# 2200077199752 2200077199752

a[0] = -1	
print(a, b)				# [-1, 1, 2] [-1, 1, 2]
print(id(a), id(b))		# 2200077199752 2200077199752
```

***

### 浅拷贝、深拷贝

对于不可变对象，当改变其值时，标识也会随机更改，其他变量也不受影响。

```python
import copy

a = "abc"
b = a
c = copy.copy(a)
d = copy.deepcopy(a)

print(a, b, c, d)	# abc abc abc abc
print(id(a), id(b), id(c), id(d))
# 2200004499976 2200004499976 2200004499976 2200004499976

a = "ABC"
print(a, b, c, d)	# ABC abc abc abc
print(id(a), id(b), id(c), id(d))
# 2200004526856 2200004499976 2200004499976 2200004499976
```

不过，在下面代码中`a`指向仍是不可变对象，但其内嵌套了可变对象列表，所以，当对该列表进行修改，只有深拷贝不受到影响。

```python
import copy

a = ([0, 1], 'a', 5)
b = a
c = copy.copy(a)
d = copy.deepcopy(a)

print(a, b, c, d)	# ([0, 1], 'a3') ([0, 1], 'a3') ([0, 1], 'a3') ([0, 1], 'a3')
print(id(a), id(b), id(c), id(d))
# 1451185543240 1451185543240 1451185543240 1451185018376
a[0][0] = "A"
print(a, b, c, d)	# (['A', 1], 'a3') (['A', 1], 'a3') (['A', 1], 'a3') ([0, 1], 'a3')
print(id(a), id(b), id(c), id(d))
# 1451185543240 1451185543240 1451185543240 1451185018376
```

浅拷贝是新建一种对象副本，拷贝了最外层的容器，但副本中的元素是源容器中元素的引用。

* 对于简单对象，对其操作不会影响其他对象。
* 对于复杂对象，对其操作同样会影响其他对象。

```python
l1 = [3, [55, 44], (7, 8, 9)]
l2 = list(l1)
l1.append(100)
l1[1].remove(55)
print('l1:', l1)
print('l2:', l2)
l2[1] += [33, 22]
l2[2] += (10, 11)
print('l1:', l1)
print('l2:', l2)
```

* `l2` 是对 `l1` 的浅拷贝；
* `l1` 加入 `100` 对 `l2` 没有造成影响；
* 对于可变对象 `list` 来说，`+=` 运算符相当于就地修改列表，所以 `l1`、`l2` 同时受到影响；
* 对于不可变对象 `tuple` 来说，`+=` 运算符创建了一个新的元组，然后重新绑定给变量 `l2`，所以对 `l1` 没有影响。

![示例](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/mutable-1.jpg)

![示例](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/mutable-2.jpg)

***

```python
In [76]: l1 = [1, 2, 3]

In [77]: l2 = [1, 2, 3]

In [78]: l1[1] = l1

In [79]: l1
Out[79]: [1, [...], 3]

In [80]: l2[1] = l2[:]

In [81]: l2
Out[81]: [1, [1, 2, 3], 3]
```

* `l1[1] = l1` 这个赋值操作将 `l1` 引用的列表对象的第二个元素的引用修改，将第二个元素引用修改为 `l1` 列表对象本身。然而，`l1` 仍然引用原来的对象，只不过其中元素的引用发生了变化。
* `l2[1] = l2[:]`中，`l1[:]` 是一个浅拷贝，创建了一个新的对象，所以 `l2[1]` 指向是一个新的对象，于是 `l2` 便指向了 `[1, [1, 2, 3], 3]`。

### 三者区别

```python
import copy

a = [1, 2, [3, 4, [5, 6]]]
b = a
c = a.copy()
d = copy.deepcopy(a)
```

其关系如下图：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/mutable-3.png)

修改其中的值后：

```python
b[0] = 'a'
c[2][0] = 'a'
d[2][2][0] = 'a'
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/mutable-4.png)

***

## 连续赋值

```cmd
In [86]: x = [1, 2, 3, 4, 5]
    ...: i = 0
    ...: i = x[i] = 3
    ...:
    ...:

In [87]: x
Out[87]: [1, 2, 3, 3, 5]

In [88]: i
Out[88]: 3
```

python 的赋值顺序是 自左往右 依次进行。对象语义指使用系统标准的拷贝方式将一个源对象拷贝成为目标对象后，源对象与目标对象之间依然共享底层资源，对任意一方的改变都将影响到另一方。

***

## 多元赋值

python 的多元赋值原理是元组封装（tuple packing）和序列拆封（sequence unpacking）。

### 元组封装

将多个值放进元组里。

```cmd
In [92]: t = 1, 2, 3

In [93]: t
Out[93]: (1, 2, 3)

In [94]: type(t)
Out[94]: tuple
```

### 元组拆包(序列拆封)

元组封装的逆操作就是序列拆封。

> *Sequence unpacking* and works for any sequence on the right-hand side.  Sequence unpacking requires that there are as many variables on the left side of the equals sign as there are elements in the sequence.  Note that multiple assignment is really just a combination of tuple packing and sequence unpacking.

元组拆包可以运用在任何可以迭代的对象上，唯一的硬性要求是可迭代对象中元素数量必须要和接受这些元素的变量个数保持一致。如果一个可迭代对象的元素数量超过变量数量时，会抛出一个 `ValueError`。除非采用 `*args`获取不确定数量的元素，拆封出的 `agrs` 变量永远都是列表类型，无论拆封出的数量是多少。`*args` 语法是专门为拆封不确定个数或任意个数元素的可迭代对象而设计。

最好辨认的元组拆包形式就是平行赋值，也就是说把一个可迭代对象里的元素，一并赋值到对应的变量中。`*agrs`可以让我们把注意力更集中在元组的部分元素上。在平行赋值中，* 前缀只能用在一个变量名前，不过，这个变量可以出现在赋值表达式的任意位置。

```cmd
# 平行赋值
user_tuple = 'Bob', 25, 180, 70
name, age, height, weight = user_tuple

# *args处理多余的元素
name, *others = user_tuple			# ('Bob', [25, 180, 70])
name, *others, weight = user_tuple	# ('Bob', [25, 180], 70)

# _ 作为占位符
user_tuple = 'Bob', 25
_, age = user_tuple
age

# 两个变量值交换
a, b = b, a
c, d = d, c+d
```

除此之外，* 运算符可以把一个可迭代对象拆开作为函数的参数。

```python
t = (20, 8)
divmod(*t)		# (2, 4)
```

### 嵌套元组拆包

接受的表达式元组可以是嵌套式的，例如`(a, b, (c, d))`，只要接受元组的嵌套结构符合表达式本身的嵌套结构即可。

```python
metro_areas = [
    ('Tokyo','JP',36.933,(35.689722,139.691667)),
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
    ('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
]

print('{:15} | {:^9} | {:^9}'.format('', 'lat.', 'long.'))
fmt = '{:15} | {:9.4f} | {:9.4f}'
for name, cc, pop, (latitude, longitude) in metro_areas:
    if longitude <= 0:
        print(fmt.format(name, latitude, longitude))
```

## 补充

### 小整数缓存机制

python 将 `[-5, 257)` 范围内的整数进行了缓存。当将范围内的整数进行赋值的时候，并不会重新创建对象，而是使用已经创建好的对象。

```python
a = 256
b = 256
c = 257
d = 257
print(a, b, c, d)					# 256 256 257 257
print(id(a), id(b), id(c), id(d))
# 140724638614128 140724638614128 2200075024336 2200075022448
```

仅有在赋值时，二者标识才相同。

```python
c = 257
d = c
print(c, d)			# 257 257
print(id(c), id(d))	# 2859885760240 2859885760240
```

不过，需要注意以下情况：

```python
a = 257
b = 257

def main():
	c = 257
	d = 257
	print(b is c)  # True
	print(a is b)  # False
	print(a is c)  # False
    print(a is e)  # False
```

代码块是程序的一个最小的基本执行单位，一个模块文件、一个函数体、一个类、交互式命令中的单行代码都叫做一个代码块。上面的代码由两个代码块构成，`a = 257`是一个代码块，`main`函数是另外一个代码块。Python内部为了进一步提高性能，凡是在一个代码块中创建的整数对象，如果值不在`small_ints`缓存范围之内，但在同一个代码块中已经存在一个值与其相同的整数对象了，那么就直接引用该对象，否则创建一个新的对象出来，这条规则对不在`small_ints`范围的负数并不适用，对负数值浮点数也不适用，但对非负浮点数和字符串都是适用的，这一点读者可以自行证明。所以 `b is c`返回了`True`，而`a`和`b`不在同一个代码块中，虽然值都是257，但却是两个不同的对象，`is`运算的结果自然是`False`了。
为了验证刚刚的结论，我们可以借用`dis`模块（听名字就知道是进行反汇编的模块）从字节码的角度来看看这段代码。

```python
import dis

dis.dis(main)
```

```bash
  4           0 LOAD_CONST               1 (257)
              2 STORE_FAST               0 (c)

  5           4 LOAD_CONST               1 (257)
              6 STORE_FAST               1 (d)

  6           8 LOAD_GLOBAL              0 (print)
             10 LOAD_FAST                0 (c)
             12 LOAD_FAST                1 (d)
             14 COMPARE_OP               8 (is)
             16 CALL_FUNCTION            1
             18 POP_TOP

  7          20 LOAD_GLOBAL              0 (print)
             22 LOAD_GLOBAL              1 (a)
             24 LOAD_GLOBAL              2 (b)
             26 COMPARE_OP               8 (is)
             28 CALL_FUNCTION            1
             30 POP_TOP

  8          32 LOAD_GLOBAL              0 (print)
             34 LOAD_GLOBAL              1 (a)
             36 LOAD_FAST                0 (c)
             38 COMPARE_OP               8 (is)
             40 CALL_FUNCTION            1
             42 POP_TOP

  9          44 LOAD_GLOBAL              0 (print)
             46 LOAD_GLOBAL              1 (a)
             48 LOAD_GLOBAL              3 (e)
             50 COMPARE_OP               8 (is)
             52 CALL_FUNCTION            1
             54 POP_TOP
             56 LOAD_CONST               0 (None)
             58 RETURN_VALUE                     
```

