列举 python 内置的函数，说明其用途，避免重复造轮子。

在调用这些函数时，如果对象定义了相关的 Magic Method，那么就会调用对应方法。

***

## 内置函数

* `abs(x)`

  返回一个数的绝对值。实参可以是整数或浮点数。如果实参是一个复数，返回它的模。

* `max(iterable, [, key, default])`

   `max(arg1, arg2, args[, key])`

   返回可迭代对象中最大的元素，或者返回两个及以上实参中最大的。

   ```bash
     >>> max(1, 2, 3)			# 传入3个位置参数
     3
     >>> max('1234') 			# 传入1个可迭代对象
     '4'
     >>> max(-1, 0, key = abs) # 传入 key 值
     -1
   ```

* ` min(iterable, [, key, default])`
   `min(arg1, arg2, args[, key])`

   返回可迭代对象中最小的元素，或者返回两个及以上实参中最小的。

   ```bash
   >>> min(1,2,3) 				# 传入3个参数
   1
   >>> min('1234') 			# 传入1个可迭代对象
   '1'
   >>> min(-1,-2,key = abs)  	# 传入 key 值
   -1
   ```

* `round(number[, ndigits])`

   对浮点数进行四舍五入求值。

   ```bash
   In [2]: round(12345.12345)			# 默认取整
   Out[2]: 12345
   
   In [3]: round(12345.12345, 2)		# 保留2位小数
   Out[3]: 12345.12
   
   In [4]: round(12345.12345, -2)		# 对整数部分求值
   Out[4]: 12300.0
   
   In [5]: round(12345.6, 0)			# 返回浮点型
   Out[5]: 12346.0
   
   In [6]: round(0.5)					# 0.5 -> 0
   Out[6]: 0
   
   In [7]: round(-0.5)					# -0.5-> 0
   Out[7]: 0
   
   In [8]: round(1.5)					# 1.5 -> 2
   Out[8]: 2
   
   In [9]: round(12345, 2)
   Out[9]: 12345
   
   In [10]: round(12345, -2)
   Out[10]: 12300
   ```

* `divmod(a, b)`

   返回两个数（非负数）的商和余数组成的元组。

   ```bash
   In [16]: divmod(5, 2)		# 相当于 (a//b,a%b)
   Out[16]: (2, 1)
   
   In [17]: divmod(5.0, 2)
   Out[17]: (2.0, 1.0)
   ```

* `sum(iterable[, start])`

   求和。参数必须是可迭代，切均为数值。

   ```python
   In [22]: sum([1, 2, 3])
   Out[22]: 6
   
   In [23]: sum([1, 2, 3], 10)		# 传入 start 参数
   Out[23]: 16
   ```

* `pow(x, y[, z])`

   2个参数则幂运算，3个参数则幂运算后取余（比直接 `pow(x, y) % z` 计算更高效）。

   如果传入参数 `z`，`x`,`y`必须为整数，且`y`不能为负数。

   ```bash
   In [28]: pow(10, 2)		# 等价于 10**2
   Out[28]: 100
   
   In [29]: pow(10, -2)	# 等价于 10**-2
   Out[29]: 0.01
   
   In [30]: pow(2, 2, 3)	# 等价于 2**2%3
   Out[30]: 1
   ```

***

* `class bool([x])`

  返回一个布尔值，`True` 或者 `False`。 `x` 使用标准的**真值测试过程**来转换。如果 *x* 是假的或者被省略，返回 `False`。`bool` 类是 `int` 的子类。其他类不能继承自它。它只有 `False` 和 `True` 两个实例。

  * 定义为 false 的常数: `None` and `False`.
  * 各种类型的 0 值: `0`, `0.0`, `0j`, `Decimal(0)`, `Fraction(0, 1)`
  * `len()`为0，例如空序列和空集合: `''`, `()`, `[]`, `{}`, `set()`, `range(0)`

* ` class int([x])`
  `class int(x, base=10)`

  返回一个使用数字或字符串 *x* 生成的整数对象，或者没有实参的时候返回 `0` 。如果 *x* 定义了 `__int__()`，`int(x)` 返回 `x.__int__()` 。如果 *x* 定义了 `__trunc__()`，它返回 `x.__trunc__()` 。对于浮点数，它向零舍入。

  ```bash
  In [34]: int()			# 无参数
  Out[34]: 0
  
  In [35]: int(2.7)		# 向下取整
  Out[35]: 2
  
  # 字符串中允许包含"+"、"-"号，但是与数值间不能有空格，数值后、符号前可出现空格。
  In [36]: int(   +2   )
  Out[36]: 2
  
  In [37]: int(   -2   )
  Out[37]: -2
  
  In [38]: int('10', 2)	# 2进制转换10进制
  Out[38]: 2
  
  In [39]: int('0f', 16)	# 16进制转换10进制
  Out[39]: 15
  
  In [59]: int('1_2_3')	# 可以使用下划线分组
  Out[59]: 123
  ```

* `class float([x])`

  返回从数字或字符串 *x* 生成的浮点数。

  ```bash
  In [42]: float()			# 无参数
  Out[42]: 0.0
  
  In [43]: float(3)			# 类型转换
  Out[43]: 3.0
  
  In [44]: float('+inf')		# 无穷大，不区分大小写
  Out[44]: inf
  
  In [45]: float('-inf')		# 无穷小，不区分大小写
  Out[45]: -inf
  
  In [46]: float('nan')		# 非数(Not a Number)，表示未定义或不可表示的值，不区分大小写
  Out[46]: nan
  
  In [51]: float('1_2_3')		# 可以使用下划线分组
  Out[51]: 123.0
  ```

* `class complex([real[, imag]])`

  返回值为 `real + imag*j` 的复数，或将字符串或数字转换为复数。

  如果第一个形参是字符串，则它被解释为一个复数，并且函数调用时必须没有第二个形参。

  如果省略了 `imag`，则默认值为零。

  ```bash
  In [60]: complex()			# 无参数
  Out[60]: 0j
  
  In [61]: complex('2+1j')	# '2 + 1j'会报错
  Out[61]: (2+1j)
  
  In [62]: complex('2')		# 只传入实部
  Out[62]: (2+0j)
  ```

  complex 类也可以被当做二维向量。

* ` class str(object='')`
  `class str(object=b'', encoding='utf-8', errors='strict')`

  返回一个 `str` 版本的 *object* 。

* ` class list([iterable])`

  传入可迭代对象，返回可迭代对象中元素组成的列表。

  ```bash
  In [22]: list()				# 空列表
  Out[22]: []
  
  In [23]: list(123)			# 必须可迭代
  TypeError: 'int' object is not iterable
  
  In [25]: list('abc123')
  Out[25]: ['a', 'b', 'c', '1', '2', '3']
  
  In [26]: list((1, 2 , 3))
  Out[26]: [1, 2, 3]
  
  In [27]: list(range(1, 4))
  Out[27]: [1, 2, 3]
  ```

* ` tuple([iterable])`

  传入可迭代对象，返回可迭代对象中元素组成的元组。

* ` class dict(**kwarg)`
  `class dict(mapping, **kwarg)`
  `class dict(iterable, **kwarg)`

  返回字典。

  ```bash
  In [34]: dict()							# 空字典
  Out[34]: {}
  
  In [35]: dict(a='a', b='b', t='t')		# 传入：关键字参数
  Out[35]: {'a': 'a', 'b': 'b', 't': 't'}
  
  In [36]: dict([('one', 1), ('two', 2), ('three', 3)])	# 传入：可迭代对象
  Out[36]: {'one': 1, 'two': 2, 'three': 3}
  
  In [37]: dict(zip(['one', 'two', 'three'], [1, 2, 3]))	# 传入：映射函数
  Out[37]: {'one': 1, 'two': 2, 'three': 3}
  ```

* ` class set([iterable])`

  传入可迭代对象，返回可迭代对象中元素组成的集合对象(可变)。 

* ` class frozenset([iterable])`

  返回一个冻结的集合(不可变集合)，冻结后集合不能再添加或删除任何元素。

* ` class bytes([source[, encoding[, errors]]])`

  返回一个新的 bytes 对象， 是一个不可变序列，包含范围为 `0 <= x < 256` 的整数。

  与 `bytearray` 的区别是，`bytes` 返回的值不可被修改。

  可选形参 *source* 可以用不同的方式来初始化数组：

  - 如果 source 为整数，则返回一个长度为 source 的初始化数组；
  - 如果 source 为字符串，则按照指定的 encoding 将字符串转换为字节序列；
  - 如果 source 为可迭代类型，则元素必须为[0 ,255] 中的整数；
  - 如果 source 为与 buffer 接口一致的对象，则此对象也可以被用于初始化 bytearray。
  - 如果没有输入任何参数，默认就是初始化数组为0个元素。

* ` class bytearray([source[, encoding[, errors]]])`

  返回一个新的 bytes 数组。 bytearray 类是一个可变序列，包含范围为 0 <= x < 256 的整数。

  可选形参 *source* 可以用不同的方式来初始化数组：

  * 如果 source 为整数，则返回一个长度为 source 的初始化数组；
  * 如果 source 为字符串，则按照指定的 encoding 将字符串转换为字节序列；
  * 如果 source 为可迭代类型，则元素必须为[0 ,255] 中的整数；
  * 如果 source 为与 buffer 接口一致的对象，则此对象也可以被用于初始化 bytearray。
  * 如果没有输入任何参数，默认就是初始化数组为0个元素。

  ***

* `enumerate(iterable, start=0)`

  返回一个枚举对象。 `enumerate()` 返回一个元组（迭代器的 `__next__()` 方法），将一个可迭代对象（*iterable*）和计数值（从 *start* 开始，默认为 0）组合为一个索引序列，同时列出数据和数据下标。

  ```bash
  In [38]: str = 'abc'
  
  In [39]: list(enumerate(str, 1))
  Out[39]: [(1, 'a'), (2, 'b'), (3, 'c')]
  
  In [40]: for x, y in enumerate(str, 1):
      ...:     print(x, y)
      ...:
  1 a
  2 b
  3 c
  ```

* ` range(stop)`
  `range(start, stop[, step])`

  返回的是一个可迭代对象。一个不可变的序列类型，可以对它进行取元素、切片等序列操作，但是不能对其中元素修改值。

  start, stop 表示整数范围，遵循的是左臂右开原则。接收的参数都必须是整数，不能是浮点数等其它数据类型。

* ` iter(object[, sentinel])`

  * 返回一个迭代器对象。
  * `object` 参数必须是可调用的对象，用于不断调用（没有参数），产出各个值
  * `sentinel` 是哨符，当可调用的对象返回这个值时，触发迭代器抛出 `StopIteration` 异常，但不输出哨符。

  官方文档提供的实例，逐行读取文件，直至遇到空行或者文件末尾为止：

  ```python
  with open('mydata.txt') as fp:
      for line in iter(fp.readline, '\n'):
          process_line(line)
  ```

* ` class slice(stop)`
  `class slice(start, stop[, step])`

  返回一个切片对象。

  实现切片对象，主要用在切片操作函数里的参数传递。

  ```bash
  In [61]: s = slice(5)
  
  In [62]: s
  Out[62]: slice(None, 5, None)
  
  In [63]: l = [x for x in range(10)]
  
  In [64]: l[s], l[:5]
  Out[64]: ([0, 1, 2, 3, 4], [0, 1, 2, 3, 4])
  
  In [65]: s = slice(0, 5, 2)
  
  In [66]: s
  Out[66]: slice(0, 5, 2)
  
  In [67]: l[s], l[:5:2]
  Out[67]: ([0, 2, 4], [0, 2, 4])
  ```

* `memoryview(obj)`

  返回给定参数的内存查看对象(Momory view)。

  Create a memoryview that references obj. obj must support the buffer protocol. Built-in objects that support the buffer protocol include bytes and bytearray.

  相关用途可查看：

  [What exactly is the point of memoryview in Python](https://stackoverflow.com/questions/18655648/what-exactly-is-the-point-of-memoryview-in-python)

* ` super([type[, object-or-type]])`

  调用父类(超类)的方法。

* `class object`

  返回一个没有特征的新对象。`object` 是所有类的基类。它具有所有 Python 类实例的通用方法。这个函数不接受任何实参。

  `object` 类没有定义 `__dict__`，所以不能对object类实例对象尝试设置属性值。

***

* ` ord(c)`

  返回对应的十进制整数。

  `ord()` 函数是 `chr()` 函数（对于8位的ASCII字符串）或 `unichr()` 函数（对于Unicode对象）的配对函数，它以一个字符（长度为1的字符串）作为参数，返回对应的 ASCII 数值，或者 Unicode 数值，如果所给的 Unicode 字符超出了你的 Python 定义范围，则会引发一个 `TypeError` 的异常。

* `chr(i)`

  返回当前整数对应的ascii字符。

  `i` -- 范围在[0, 255]内的整数，可以是10进制也可以是16进制的形式的数字。

  ```bash
  In [3]: ord('a')
  Out[3]: 97
  
  In [4]: chr(97)
  Out[4]: 'a'
  ```

  ***

* `bin(x)`

  将一个整数转变为一个前缀为“0b”的二进制字符串。

  如果 *x* 不是 Python 的 `int` 对象，那它需要定义 `__index__()`  方法返回一个整数。

  ```bash
  In [5]: bin(97)
  Out[5]: '0b1100001'
  
  In [7]: format(97, 'b'), format(97, '#b') # 隐藏\显示前缀“0b”
  Out[7]: ('1100001', '0b1100001')
  ```

* `oct(x)`

  将一个整数转变为一个前缀为“0o”的八进制字符串。

  如果 *x* 不是 Python 的 `int` 对象，那它需要定义 `__index__()`  方法返回一个整数。

  可设置隐藏\显示前缀“0b”。

  官方示例：

  ```bash
  In [18]: '%#o' % 10, '%o' % 10
  Out[18]: ('0o12', '12')
  
  In [19]: format(10, '#o'), format(10, 'o')
  Out[19]: ('0o12', '12')
  
  In [20]: f'{10:#o}', f'{10:o}'
  Out[20]: ('0o12', '12')
  ```

* `hex(x)`

  将整数转换为以“0x”为前缀的小写十六进制字符串。

  如果 *x* 不是 Python 的 `int` 对象，那它需要定义 `__index__()`  方法返回一个整数。

  不仅可以隐藏\显示“0x”前缀，还可设置十六进制字符串的大写或小写。

  官方示例：

  ```bash
  In [14]: '%#x' % 255, '%x' % 255, '%X' % 255
  Out[14]: ('0xff', 'ff', 'FF')
  
  In [15]: format(255, '#x'), format(255, 'x'), format(255, 'X')
  Out[15]: ('0xff', 'ff', 'FF')
  
  In [16]: f'{255:#x}', f'{255:x}', f'{255:X}'
  Out[16]: ('0xff', 'ff', 'FF')
  ```

***

* ` all(iterable)`

  如果 *iterable* 的所有元素为真（或迭代器为空），返回 `True` 。

  元素除了是 0、空、FALSE 外都算 TRUE。

  ```bash
  In [72]: all([1, 2, 3])
  Out[72]: True
  
  In [73]: all([1, 2, 0])
  Out[73]: False
  
  In [74]: all([])
  Out[74]: True
  ```

* `any(iterable)`

  如果 *iterable* 的任一元素为真则返回``True``。如果迭代器为空，返回``False``。

  ```bash
  In [75]: any([1, 2, 3])
  Out[75]: True
  
  In [76]: any([1, 2, 0])
  Out[76]: True
  
  In [77]: any([])
  Out[77]: False
  ```

* ` isinstance(object, classinfo)`

  判断 `object` 对象是否是 `class` 的实例，或者是否属于 `type` 类型，相同则返回 `True`，否则返回 `False`。

  ```bash
  In [142]: isinstance(1, (int, float))	# 多个类型时，必须是元组
  Out[142]: True
  ```

  判断实例的 `class` 的类型，尤其存在继承关系的 `class`，相对 `type` 而言，好用一些。

  继承关系如下：

  ```bash
  object --> Animals --> Dogs
  ```

  ```python
  test = Animals('Test')
  husky = Dogs('Husky')
  print(isinstance(test, Animals))
  print(isinstance(husky, Animals))
  print(isinstance(husky, Dogs))
  ```

  ```bash
  True
  True
  True
  ```

  `isinstance()` 与 `type()` 区别：

  - `type()` 不会认为子类是一种父类类型，不考虑继承关系。
  - `isinstance()` 会认为子类是一种父类类型，考虑继承关系。

* ` issubclass(class, classinfo)`

  如果 *class* 是 *classinfo* 的子类（直接、间接或虚拟的），则返回 true。*classinfo* 可以是类对象的元组，此时 *classinfo* 中的每个元素都会被检查。其他情况，会触发 `TypeError` 异常。

* ` hasattr(object, name)`

  该实参是一个对象和一个字符串。如果字符串是对象的属性之一的名称，则返回 `True`，否则返回 `False`。

* ` callable(object)`

  可调用返回 True，否则返回 False。

  对于函数, 方法, lambda 函式, 类, 以及实现了 `__call__` 方法的类实例, 它都返回 True。 

  类对象都是可被调用对象，类的实例对象是否可调用对象，取决于类是否定义了`__call__`方法。

***

* `sorted(iterable, key=None, reverse=False)`

  返回重新排序的列表。

  ```bash
  In [116]: sorted([5, 2, 3, 1, 4])
  Out[116]: [1, 2, 3, 4, 5]
  
  In [117]: sorted('dfgAabB')
  Out[117]: ['A', 'B', 'a', 'b', 'd', 'f', 'g']
  
  In [126]: sorted('dfgAabB', key=lambda x: x.lower())
  Out[126]: ['A', 'a', 'b', 'B', 'd', 'f', 'g']
  ```

  > **sort 与 sorted 区别：**
  >
  > sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。
  >
  > list 的 sort 方法返回的是对已经存在的列表进行操作，而内建函数 sorted 方法返回的是一个新的 list，而不是在原来的基础上进行的操作。

* ` reversed(seq)`

  返回一个反转的迭代器。

  如果参数不是一个序列对象，则其必须定义一个`__reversed__方法。`

  ```bash
  In [132]: list(reversed(range(10)))
  Out[132]: [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
  ```

* `filter(function, iterable)`

  返回一个迭代器对象。

  用于过滤序列，过滤掉不符合条件的元素，返回一个迭代器对象。如果要转换为列表，可以使用 `list()`来转换。

  该接收两个参数，第一个为函数，第二个为序列，序列的每个元素作为参数传递给函数进行判，然后返回 True 或 False，最后将返回 True 的元素放到新列表中。

  1~100中平方根是整数的数

  ```python
  import math
  def is_sqr(x):
      return math.sqrt(x) % 1 == 0
   
  tmplist = filter(is_sqr, range(1, 101))
  newlist = list(tmplist)
  ```

  ```bash
  In [78]: l = ['', False, 'I', {}]
  
  In [79]: list(filter(None, l))		# 传入 None
  Out[79]: ['I']
  ```

* `map(function, iterable, ...)`

  返回迭代器。

  会根据提供的函数对指定序列做映射。第一个参数 function 以参数序列中的每一个元素调用 function 函数，返回包含每次 function 函数返回值的新列表。

  当传入多个可迭代对象时，函数的参数必须提供足够多的参数，保证每个可迭代对象同一索引的值均能正确传入函数。

  ```bash
  In [102]: list(map(lambda x, y: x + y, range(1,10,2), range(2,11,2)))
  Out[102]: [3, 7, 11, 15, 19]
  ```

* ` zip(*iterables)`

  函数用于将可迭代的对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的对象。

  如果各个迭代器的元素个数不一致，则返回列表长度与最短的对象相同。

  利用 * 号操作符，可以将元组解压为列表。

  ```bash
  In [103]: a = [1, 2, 3]
  
  In [104]: b = [3, 2, 1]
  
  In [106]: z = list(zip(a, b))
  
  In [107]: z
  Out[107]: [(1, 3), (2, 2), (3, 1)]
  
  In [108]: x, y = zip(*z)		# 解压
  
  In [110]: x, y
  Out[110]: ((1, 2, 3), (3, 2, 1))
  
  In [112]: list(zip(x))
  Out[112]: [(1,), (2,), (3,)]
  ```

* `next(iterator[, default])`

  * 必须接收一个迭代器，每次调用的时候，返回迭代器的下一个元素。如果所有元素均已返回，则抛出`StopIteration` 异常。
  * 可以接收一个可选的default参数，传入default参数后，如果迭代器还有元素没有返回，则依次返回其元素值，如果所有元素已经返回，则返回default指定的默认值而不抛出`StopIteration` 异常。

  ```bash
  In [85]: it = iter(range(10))
  
  In [86]: next(it)
  Out[86]: 0
  
  In [87]: next(it)
  Out[87]: 1
  
  In [95]: next(it)
  Out[95]: 9
  
  In [96]: next(it, 'NONE')
  Out[96]: 'NONE'
  
  In [97]: next(it, 'NONE')
  Out[97]: 'NONE'
  ```

***

* `help([object])`

  返回对象帮助信息。

  `help` 在 python 交互式环境中使用，比如 `Ipython`。

  输入无参数 `help()`时，会进入 help 交互环境：

  ```bash
  In [2]: help()
  
  Welcome to Python 3.7s help utility!
  ...
  
  help>
  ```

  - 指定的modules，例如 collections 模块：`help("collections")`；
  - keywords：`help('keywords')`；
  - 内置的类型：`help("list")`；
  - 类型的成员方法：`help("list.pop")`；
  - 内置函数：`help("iter")`。

  `help` 函数是查看函数或模块用途的详细说明，而 `dir` 函数是查看函数或模块内的属性、方法，输出列表。

* ` dir([object])`

  如果没有实参，则返回当前本地作用域中的名称列表。如果有实参，它会尝试返回该对象的有效属性列表。

  `object`的默认值是当前的模块对象。如果参数包含方法 `__dir__`，该方法将被调用，并且必须返回一个属性列表。如果参数不包含 `__dir__`，该方法将最大限度地收集参数信息。

  默认的 `dir()` 机制对不同类型的对象行为不同，它会试图返回最相关而不是最全的信息：

  * 无参数时，返回当前作用域内的变量、方法和定义的类型列表。
  * 模块对象：返回模块的属性列表。
  * 类型或类对象：返回类及其子类的属性列表，并且递归查找所有基类的属性。
  * 当对象定义了`__dir__`方法，则返回`__dir__`方法的结果

  主要是为了便于在交互式时使用。

* ` id(object)`

  返回对象的“标识值”。该值是一个整数，在此对象的生命周期中保证是唯一且恒定的。

* ` hash(object)`

  返回该对象的哈希值（如果它有的话）。哈希值是整数。它们在字典查找元素时用来快速比较字典的键。相同大小的数字变量有相同的哈希值（即使它们类型不同，如 1 和 1.0）。

  ```bash
  In [134]: hash(12345)
  Out[134]: 12345
  
  In [135]: hash(12345.0)
  Out[135]: 12345
  
  In [136]: hash('abc')
  Out[136]: 7491864301760075081
  ```

* `class type(object)`
  `class type(name, bases, dict)`

  返回对象的类型。

  基本类型都可以用 `type` 判断。比如：`int`、`str`、`FunctionType`、`BuiltinFunctionType`、`LambdaType`、`GeneratorType`。

  但在判断 class 的继承关系就不方便。返回了各自所属的类，而没有返回父类。

  传入3个参数，函数将使用3个参数来创建一个新的类型。这是动态创建类的方法。

  - The *name* string is the class name and becomes the `__name__`  attribute; 
  - the *bases* tuple itemizes the base classes and becomes the `__bases__` attribute; 
  - the *dict* dictionary is the namespace containing definitions for class body and is copied to a standard dictionary to become the `__dict__` attribute.  

  ```python
  class X:
  	a = 1
  
  X = type('X', (object,), dict(a=1))
  ```

* ` vars([object])`

  返回对象object的属性和属性值的字典对象。

  如果没有参数，就打印当前调用位置的属性和属性值，类似 `locals()`。

   当函数接收一个参数时，参数可以是模块、类、类实例，或者定义了`__dict__`属性的对象，等价于`obj.__dict__`。

* `locals()`

  更新并返回表示当前本地符号表的字典。

  在函数块而不是类块中调用时，会返回自由变量。

  不要更改此字典的内容，因为更改不会影响解释器使用的局部变量或自由变量的值。

  ```bash
  In [140]: def func(n):
       ...:     str = 'a'
       ...:     print(locals())
       ...:     test = locals()
       ...:     test['str'] = 'b'
       ...:     print(locals())
       ...:
  
  In [141]: func(2)
  {'n': 2, 'str': 'a'}
  {'n': 2, 'str': 'a', 'test': {...}}
  ```

* ` globals()`

  返回全局变量的字典。

* ` len(s)`

  返回对象的长度（元素个数）。

  如果参数为其它类型，则其必须实现`__len__`方法，并返回整数，否则报错。

* `ascii(object)`

  类似 `repr()` 函数, 返回一个表示对象的字符串, 但是对于字符串中的非 ASCII 字符则返回通过 `repr()`  函数使用 \x, \u 或 \U 编码的字符。

* ` repr(object)`

  将对象转化为供解释器读取的形式。

  返回的结果一般能通过eval()求值的方法获取到原对象。

***

* ` format(value[, format_spec])`

  将 *value* 转换为 *format_spec* 控制的“格式化”表示。

  改函数用法很多，详见《python 字符串格式化》。

***

* ` print(*objects, sep=' ', end='\n', file=sys.stdout, flush=False)`

  *  objects -- 复数，表示可以一次输出多个对象。输出多个对象时，需要用 , 分隔。 
  *  sep -- 用来间隔多个对象，默认值是一个空格。 
  *  end -- 用来设定以什么结尾。默认值是换行符 \n，我们可以换成其他字符串。 
  *  file -- 要写入的文件对象。 

  ```bash
  In [147]: print(1,2,3,sep = '+',end = '=?')
  1+2+3=?
  ```

* ` input([prompt])`

  如果存在 *prompt* 参数，则将其写入标准输出，末尾不带换行符。接下来，该函数从输入中读取一行，将其转换为字符串（除了末尾的换行符）并返回。

  ```bash
  In [144]: s = input('-->')
  -->abc
  
  In [145]: s
  Out[145]: 'abc'
  ```

***

* ` getattr(object, name[, default])`

  返回一个对象属性值。

  *name* 必须是字符串。

  * 如果该字符串是对象的属性之一，则返回该属性的值。例如， `getattr(x, 'foobar')` 等同于 `x.foobar`。
  * 如果指定的属性不存在，且提供了 *default* 值，则返回它，否则触发 `AttributeError`。

* ` setattr(object, name, value)`

  设置属性值，无返回值。与 `getatr()` 函数对应。

  * 如果 `name` 属性是 `object` 对象已经存在的属性，那么更新其属性值；
  * 如果 `name` 属性不存在，则对象将创建 `name` 属性。 `setattr(x, 'foobar', 123)` 等价于 `x.foobar = 123`。

* ` delattr(object, name)`

  `name` 必须是对象的某个属性，且为字符串。如果对象允许，该函数将删除指定的属性。例如 `delattr(x, 'foobar')`  等价于 `del x.foobar` 。

  不能删除对象的方法。

***

* ` open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)`

  打开 *file* 并返回对应的 file object]。如果该文件不能打开，则触发 `OSError`。

  该函数详细内容，《python 文件操作》。

***

* ` compile(source, filename, mode, flags=0, dont_inherit=False, optimize=-1)`

  将一个字符串编译为字节代码，返回表达式执行结果。

  将字符串编译为代码或者AST对象，使之能够通过`exec()`或者`eval()`执行。

  * source -- 字符串或者AST（Abstract Syntax Trees）对象。。
  * filename -- 代码文件名称，如果不是从文件读取代码则传递一些可辨认的值。
  * mode -- 指定编译代码的种类。可以指定为 exec, eval, single。
  * flags -- 变量作用域，局部命名空间，如果被提供，可以是任何映射对象。。
  * flags和dont_inherit是用来控制编译源码时的标志

* ` eval(expression, globals=None, locals=None)`

  返回表达式计算结果。动态执行 Python 代码。

  - expression -- 表达式。
  - global -- 变量作用域，全局命名空间，如果被提供，则必须是一个字典对象。
  - locals -- 变量作用域，局部命名空间，如果被提供，可以是任何映射对象。

  ```bash
  In [1]: x = 1
  
  In [2]: eval('x*3')					# 无参数
  Out[2]: 3
  
  In [3]: eval('x*3', {'x':2})		# 使用 globals
  Out[3]: 6
  
  In [5]: eval('x*3', {}, {'x':2})	# 使用 locals
  Out[5]: 6
  
  In [6]: eval('pow(2,2)')			# 执行语句
  Out[6]: 4
  ```

* ` exec(object[, globals[, locals]])`

  返回值永远为 `None`。动态执行储存在字符串或文件中的 Python 语句，相比于 `eval`，`exec`可以执行更复杂的 Python 代码。

  `object`：必选参数，表示需要被指定的Python代码。它必须是字符串或code对象。如果object是一个字符串，该字符串会先被解析为一组Python语句，然后在执行（除非发生语法错误）。如果object是一个code对象，那么它只是被简单的执行。

  ```bash
  In [13]: exec('for i in range(5):print(i)')
  0
  1
  2
  3
  4
  ```

  默认情况下，*locals* 的行为如下面 `locals()` 函数描述的一样：不要试图改变默认的 *locals* 字典。如果您想在 `exec()` 函数返回时知道代码对 *locals* 的变动，请明确地传递 *locals* 字典。参考：[How does exec work with locals?](https://stackoverflow.com/questions/1463306/how-does-exec-work-with-locals)

  ```bash
  In [7]: def f():
     ...:     a = 1
     ...:     exec("a = 3")
     ...:     print(a)
     ...:
  
  In [8]: f()
  1
  
  In [10]: def foo():
      ...:     ldict = {}
      ...:     exec("a=3",globals(),ldict)
      ...:     a = ldict['a']
      ...:     print(a)
      ...:
  
  In [11]: foo()
  3
  ```

***

* ` property(fget=None, fset=None, fdel=None, doc=None)`

  返回 property 属性。

  描述符相关内容，详见：《python OOP 描述符》

  ```python
  class C(object):
      def __init__(self):
          self._x = None
   
      def getx(self):
          return self._x
   
      def setx(self, value):
          self._x = value
   
      def delx(self):
          del self._x
   
      x = property(getx, setx, delx, "I'm the 'x' property.")
  ```

  或者使用 `@property`：

  ```python
  class C(object):
      def __init__(self):
          self._x = None
   
      @property
      def x(self):
          """I'm the 'x' property."""
          return self._x
   
      @x.setter
      def x(self, value):
          self._x = value
   
      @x.deleter
      def x(self):
          del self._x
  ```

* `@classmethod`

  返回函数的类方法。

  对应的函数不需要实例化，不需要 `self` 参数，但第一个参数需要是表示自身类的 `cls` 参数，可以来调用类的属性，类的方法，实例化对象等。

  ```python
  class C_method:
      data = 'DATA'
  
      @classmethod
      def get_data(cls, test):
          print(cls.data, test)
  ```

  OOP、描述符相关内容，详见：《python OOP 描述符》《python OOP 实例方法、类方法、静态方法》

* ` @staticmethod`

  返回函数的静态方法。

  该方法不强制要求传递参数。

  ```python
  class S_method:
      data = 'DATA'
  
      @staticmethod
      def get_data(test):
          print(test)
  ```

  OOP、描述符相关内容，详见：《python OOP 描述符》《python OOP 实例方法、类方法、静态方法》

***

* ` __import__(name, globals=None, locals=None, fromlist=(), level=0)`

  动态加载类和函数 。

***

## 区别

### hash & id

```python
hash(500) == hash(1.0 * 500)	# True

id(500) == id(500)				# False
```

***

参考：

[Built-in Functions](https://docs.python.org/3/library/functions.html)

[Python内置函数详解——总结篇](https://www.cnblogs.com/sesshoumaru/p/6140987.html)