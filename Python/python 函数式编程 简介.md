函数（function）是用于完成特定任务的程序代码的自包含单元。在面向对象编程的类中，函数通常被称作方法。不同的函数在程序中扮演着不同的角色，起着不同的作用，执行不同的动作。

## 函数

### 为什么要使用函数

1. 封装：封装内部实现，减少复杂性。很多时候，我们把函数看做“黑盒子”，即对应一定的输入会产生特定的结果或返回某个对象。往往函数的使用者并不是函数的编写者，函数的使用者对黑盒子的内部行为并不需要考虑，可以把精力投入到自身业务逻辑的设计而不是函数的实现细节。只有函数的设计者或者说编写者，才需要考虑函数内部实现的细节，如何暴露对外的接口，返回什么样的数据，也就是API的设计。
2. 复用：重用代码，省去重复性代码的编写，提高代码的重复利用率。如果程序中需要多次使用某种特定的功能，那么只需要编写一个合适的函数就可以了。程序可以在任何需要的地方调用该函数，并且同一个函数可以在不同的程序中调用。
3. 命名空间的隔离：保护数据。

### 如何使用函数

在密码学中，恺撒密码（英语：Caesar cipher），或称恺撒加密、恺撒变换、变换加密，是一种最简单且最广为人知的加密技术。它是一种替换加密的技术，明文中的所有字母都在字母表上向后（或向前）按照一个固定数目进行偏移后被替换成密文。例如，当偏移量是3的时候，所有的字母A将被替换成D，B变成E，以此类推。

```python
alphabet_src = 'abcdefghijklmnopqrstuvwxyz'

def encryptIt(src_str, num=0):
    result = ''
    for item in src_str:
        if item in alphabet_src:
            index_old = alphabet_src.index(item)
            index_new = (index_old + num) % 26
            result += alphabet_src[index_new]
        else:
            result += item
    return result

def decryptIt(src_str, num=0):
    result = ''
    for item in src_str:
        if item in alphabet_src:
            index_old = alphabet_src.index(item)
            index_new = (index_old - num) % 26
            result += alphabet_src[index_new]
        else:
            result += item
    return result

assert('abc xyz!') == decryptIt(encryptIt('abc xyz!', 3), 3)
```

再次基础上，进行扩展，可以从合并和分离两个方向：

* `encryptIt`和`decryptIt`合并成一个函数
* 抽取`encryptIt`和`decryptIt`其中解密加密部分，写成新的函数，这样以后如果修改加密方法，那么不会影响前两个函数。

### 常见函数形式

1. 普通函数

```python
def function():
    return 1
```

2. 生成器函数

```python
def generator():
    yield 1
```

在3.5过后，我们可以使用async修饰将普通函数和生成器函数包装成异步函数和异步生成器。

3. 异步函数（协程）

```python
async def async_function():
    return 1
```

4. 异步生成器

```python
async def async_generator():
    yield 1
```

通过类型判断可以验证函数的类型

```python
import types
print(type(function) is types.FunctionType)
print(type(generator()) is types.GeneratorType)
print(type(async_function()) is types.CoroutineType)
print(type(async_generator()) is types.AsyncGeneratorType)
```

***

## 函数式编程

### 定义

简单说，"函数式编程"（functional programming）是一种"**编程范式**"（programming paradigm），也就是如何编写程序的方法论。

它属于"**结构化编程**"（Structured programming）的一种，主要思想是把运算过程尽量写成一系列嵌套的函数调用。举例来说，现在有这样一个数学表达式：

```java
(1 + 2) * 3 - 4
```

传统的过程式编程，可能这样写：

```java
var a = 1 + 2;

var b = a * 3;

var c = b - 4;
```

函数式编程要求使用函数，我们可以把运算过程**定义**为不同的函数，然后写成下面这样：

```java
var result = subtract(multiply(add(1,2), 3), 4);
```

PS：

**编程范式**：是一类典型的编程风格，是指从事软件工程的一类典型的风格（可以对照方法学）。如：函数式编程、程序编程、面向对象编程、指令式编程等等为不同的编程范型。编程范型提供了（同时决定了）程序员对程序执行的看法。例如，在面向对象编程中，程序员认为程序是一系列相互作用的对象，而在函数式编程中一个程序会被看作是一个无状态的函数计算的序列。 

编程范型和编程语言之间的关系可能十分复杂，由于一个编程语言可以支持多种范型。例如，C++设计时，支持过程化编程、面向对象编程以及泛型编程。然而，设计师和程序员们要考虑如何使用这些范型元素来构建一个程序。一个人可以用C++写出一个完全过程化的程序，另一个人也可以用C++写出一个纯粹的面向对象程序，甚至还有人可以写出杂揉了两种范型的程序。

**结构化编程**:一种编程范式。它采用子程序、代码区块、for循环以及while循环等结构，来取代传统的 goto。希望借此来改善计算机程序的明晰性、质量以及开发时间，并且避免写出面条式代码。 

***

### 特点

函数式编程具有五个鲜明的特点。

* 函数是"第一等公民"

所谓"第一等公民"（first class），指的是函数与其他数据类型一样，处于平等地位，可以赋值给其他变量，也可以作为参数，传入另一个函数，或者作为别的函数的返回值。

举例来说，下面代码中的print变量就是一个函数，可以作为另一个函数的参数。

```java
var print = function(i){ console.log(i);};

[1,2,3].forEach(print);
```

* 只用"表达式"，不用"语句"

"表达式"（expression）是一个单纯的运算过程，总是有返回值；"语句"（statement）是执行某种操作，没有返回值。函数式编程要求，只使用表达式，不使用语句。也就是说，每一步都是单纯的运算，而且都有返回值。

原因是函数式编程的开发动机，一开始就是为了处理运算（computation），不考虑系统的读写（I/O）。"语句"属于对系统的读写操作，所以就被排斥在外。

当然，实际应用中，不做I/O是不可能的。因此，编程过程中，函数式编程只要求把I/O限制到最小，不要有不必要的读写行为，保持计算过程的单纯性

* 没有"副作用"

所谓"副作用"（side effect），指的是函数内部与外部互动（最典型的情况，就是修改全局变量的值），产生运算以外的其他结果。

函数式编程强调没有"副作用"，意味着函数要保持独立，所有功能就是返回一个新的值，没有其他行为，尤其是不得修改外部变量的值。

PS:

**副作用**：在计算机科学中，函数副作用指当调用函数时，除了返回函数值之外，还对主调用函数产生附加的影响。例如修改全局变量（函数外的变量）或修改参数。

函数副作用会给程序设计带来不必要的麻烦，给程序带来十分难以查找的错误，并降低程序的可读性。严格的函数式语言要求函数必须无副作用。 

下面是函数的副作用相关的几个概念，纯函数（Pure Function）、非纯函数（Impure Function）、引用透明（Referential Transparent）。 

**纯函数**：输入输出数据流全是**显式（Explicit）**的。 

显式（Explicit）的意思是，函数与外界交换数据只有一个唯一渠道——参数和返回值；函数从函数外部接受的所有输入信息都通过参数传递到该函数内部；函数输出到函数外部的所有信息都通过返回值传递到该函数外部。

在程序设计中，若一个函数符合以下要求，则它可能被认为是纯函数：

* 此函数在相同的输入值时，需产生相同的输出。函数的输出和输入值以外的其他隐藏信息或状态无关，也和由I/O设备产生的外部输出无关。
* 该函数不能有语义上可观察的函数副作用，诸如“触发事件”，使输出设备输出，或更改输出值以外物件的内容等。

纯函数的输出可以不用和所有的输入值有关，甚至可以和所有的输入值都无关。但纯函数的输出不能和输入值以外的任何资讯有关。纯函数可以传回多个输出值，但上述的原则需针对所有输出值都要成立。若引数是传引用调用，若有对参数物件的更改，就会影响函数以外物件的内容，因此就不是纯函数。 

**非纯函数**：如果一个函数通过**隐式（Implicit）**方式，从外界获取数据，或者向外部输出数据，那么，该函数就不是纯函数，叫作非纯函数（Impure Function）。

隐式（Implicit）的意思是，函数通过参数和返回值以外的渠道，和外界进行数据交换。比如，读取全局变量，修改全局变量，都叫作以隐式的方式和外界进行数据交换；比如，利用 I/O API（输入输出系统函数库）读取配置文件，或者输出到文件，打印到屏幕，都叫做隐式的方式和外界进行数据交换。 

**引用透明**：引用透明（Referential Transparent）的概念与函数的副作用相关，且受其影响。如果程序中任意两处具有相同输入值的函数调用能够互相置换，而不影响程序的动作，那么该程序就具有引用透明性。它的优点是比非引用透明的语言的语义更容易理解，不那么晦涩。纯函数式语言没有变量，所以它们都具有引用透明性。

* 不修改状态

上一点已经提到，函数式编程只是返回新的值，不修改系统变量。因此，不修改变量，也是它的一个重要特点。

在其他类型的语言中，变量往往用来保存"状态"（state）。不修改变量，意味着状态不能保存在变量中。函数式编程使用参数保存状态，最好的例子就是递归。下面的代码是一个将字符串逆序排列的函数，它演示了不同的参数如何决定了运算所处的"状态"。

```java
function reverse(string) {
	if(string.length == 0) {
		return string;
	} else {
		return reverse(string.substring(1, string.length)) + string.substring(0, 1);
	}
}
```

* 引用透明

引用透明（Referential transparency），指的是函数的运行不依赖于外部变量或"状态"，只依赖于输入的参数，任何时候只要参数相同，引用函数所得到的返回值总是相同的。

有了前面的第三点和第四点，这点是很显然的。其他类型的语言，函数的返回值往往与系统状态有关，不同的状态之下，返回值是不一样的。这就叫"引用不透明"，很不利于观察和理解程序的行为。

### 意义

* 代码简洁，开发快速

函数式编程大量使用函数，减少了代码的重复，因此程序比较短，开发速度较快。

* 接近自然语言，易于理解

函数式编程的自由度很高，可以写出很接近自然语言的代码。

前文曾经将表达式(1 + 2) * 3 - 4，写成函数式语言：

```java
subtract(multiply(add(1,2), 3), 4)
```

对它进行变形，不难得到另一种写法：

```java
add(1,2).multiply(3).subtract(4)
```

这基本就是自然语言的表达了。再看下面的代码，大家应该一眼就能明白它的意思吧：

```java
merge([1,2],[3,4]).sort().search("2")
```

因此，函数式编程的代码更容易理解。

* 更方便的代码管理

函数式编程不依赖、也不会改变外界的状态，只要给定输入参数，返回的结果必定相同。因此，每一个函数都可以被看做独立单元，很有利于进行单元测试（unit testing）和除错（debugging），以及模块化组合。

* 易于"并发编程"

函数式编程不需要考虑"死锁"（deadlock），因为它不修改变量，所以根本不存在"锁"线程的问题。不必担心一个线程的数据，被另一个线程修改，所以可以很放心地把工作分摊到多个线程，部署"并发编程"（concurrency）。

请看下面的代码：

```java
var s1 = Op1();

var s2 = Op2();

var s3 = concat(s1, s2);
```

由于s1和s2互不干扰，不会修改变量，谁先执行是无所谓的，所以可以放心地增加线程，把它们分配在两个线程上完成。其他类型的语言就做不到这一点，因为s1可能会修改系统状态，而s2可能会用到这些状态，所以必须保证s2在s1之后运行，自然也就不能部署到其他线程上了。

* 代码的热升级

函数式编程没有副作用，只要保证接口不变，内部实现是外部无关的。所以，可以在运行状态下直接升级代码，不需要重启，也不需要停机。

***

## 参数

### 默认参数

默认参数必须在位置参数后面！

默认参数尽量指向不变的对象！

### 可变参数 `*agrs`

`*args` 是将来一个**非键值对**的可变数量的参数列表传递给函数。接收的是一个 `tuple`。既可以直接传入：`func(1, 2, 3)`，又可以先组装 `list` 或 `tuple`，再通过 `*args` 传入：`func(*(1, 2, 3))`。

```python
def add(*args):
    total = 0
    for val in args:
        total += val
    return total

print(add())
print(add(1))
print(add(1, 2))
print(add(1, 2, 3))
print(add(1, 3, 5, 7, 9))
```

### 关键字参数 `**kwargs`

`**kwargs` 允许将不定长度的**键值对**作为参数传递给函数。接收的是一个 `dict`。既可以直接传入：`func(a=1, b=2)`，又可以先组装 `dict`，再通过 `**kwargs` 传入：`func(**{'a': 1, 'b': 2})`。

可变参数和关键字参数只有变量前面的 `*` 才是必须的。可以写成 `*var` 和 `**vars`。`*args` 和 `**kwargs` 只是一个通俗的命名约定。必须放在所有的位置参数和默认参数后面！当`*args`和`**kwargs`组合起来使用，理论上能接受任何形式和任意数量的参数。

`*args` 和 `**kwargs` 主要用于函数定义。可以将不定数量的参数传递给函数。预先不知道函数使用者会传递多少个参数的场景下使用这两个关键字。

### 命名关键字参数

如果要限制关键字参数的名字，就可以用命名关键字参数。

和关键字参数不同，命名关键字参数需要一个特殊分隔符，后面的参数被视为命名关键字参数，`*, a, b`。

```python
def person(name, age, *, city, job):
```

如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符 `*` 了

```python
def person(name, age, *args, city, job):
```

### 参数顺序

```python
def person(name, age, city=beijing, *args, **kwargs):
```

## 参数传递

**值传递**是指在调用函数时将实际参数复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。

**引用传递**是指在调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

**Python函数是通过引用传参的。**

如果是从C++转来的，那么就可以理解成Python的传值方式是按照C++中传指针的方式传值的，既不是引用也不是值。

* 如果数据类型是可变的，对其进行运算操作，则是在其本身上面进行。如果进行赋值，则这个符号指向一个新的对象，旧的对象根据情况进行垃圾回收。 
* 如果数据类型是不可变的，那么运算操作或者赋值，均相当于创建新的对象，然后将原先的变量名绑定到新的对象。

### 不可变对象

在函数发生的变化，不会波及函数外。

```python
def num_add(num):
    num += 10

d = 2
num_add(d)
print(d)	# 2
```

想要得到12，必须进行赋值操作。

```python
def num_add(num):
    return num + 10

d=2
d = num_add(d)
print(d)	# 12		
```

或者使用`global`

```python
def num_add():
    global d
    d = d + 10

d = 2
num_add()
print(d)	# 12
```

***

### 可变对象

#### 函数操作可变对象，参与赋值、`=...+`、`+=`、`.append()`操作

* 赋值（`=`）

如下进行赋值操作，并不会影响函数之外的对象。因为，可变对象的赋值操作是创建一个新的对象。

```python
def foo(ls):
    ls = [0, 0, 0]
    print(ls)
    
l1 = [0, 1, 2]
foo(l1)			# [0, 0, 0]
print(l1)		# [0, 1, 2]
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/Functional%20Programming%202.png)

* `=...+..`运算

```python
def foo(ls):
    ls = ls + [0, 0, 0]
    print(ls)
    
l1 = [0, 1, 2]
foo(l1)			# [0, 1, 2, 0, 0, 0]
print(l1)		# [0, 1, 2]
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/Functional%20Programming%201.png)

* `+=`运算

函数内部可以访问全局变量中的值，但是不能修改全局变量的值。但是，当参数使用可变对象时，会影响函数之外。

```python
def foo(ls):
    ls += [0, 0, 0]
    print(ls)
    
l1 = [0, 1, 2]
foo(l1)			# [0, 1, 2, 0, 0, 0]
print(l1)		# [0, 1, 2, 0, 0, 0]
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/Functional%20Programming%203.png)

* `.extend()`等运算。

过程和结果同 `+=` 运算。

```python
def foo(ls):
    ls.extend([0, 0, 0])
    print(ls)
    
l1 = [0, 1, 2]
foo(l1)         # [0, 1, 2, 0, 0, 0]
print(l1)		# [0, 1, 2, 0, 0, 0]
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/Functional%20Programming%204.png)

`+=` `.extend()`不同于赋值`=`和`=...+`语句，不会创建新的变量。而列表作为可变类型，这只是对列表的一种调用而已。

再如下面的例子：

```python
def func(a, b):
    a += b
    return a

if __name__ == "__main__":
    a = 1
    b = 1
    print(func(a, b))		# 2
    print(a, b)				# 1 1

    a = [1,]
    b = [2,]
    print(func(a, b))		# [1, 2]
    print(a, b)				# [1, 2] [2]
```

```python
def func(a, b):
    a = a + b
    return a

if __name__ == "__main__":
    a = 1
    b = 1
    print(func(a, b))		# 2
    print(a, b)				# 1 1

    a = [1,]
    b = [2,]
    print(func(a, b))		# [1, 2]
    print(a, b)				# [1] [2]
```

造成 `a` 变化的原因是:

* `..+=..` 不会改变变量引用，不会创建新的对象。`.append()`等列表相关用法，也不会改变引用。
*  `..=..+..` 会改变其引用

***

#### 可变对象作为默认参数

默认参数不推荐使用可变参数，否则后果如下：

```python
def foo(ls=[]):
    ls.append(0)
    return ls
```

一下两种调用方式，产生的结果不同：

```python
print(foo())		# [0]
print(foo())		# [0, 0]
```

```python
print(foo(), foo())	# [0, 0] [0, 0]
```

即使在函数外部重新定义一个可变对象变量，仍然产生相同的现象。这与类中不同的原因是，类可以把变量封装在实例中，虽然也使用了可变对象作为参数，但是会产生与现在不同的结果。

```python
def foo(ls=[]):
    ls.append(0)
    return ls

lst = [1, 2]
print(foo(lst))	# [1, 2, 0]
print(lst)		# [1, 2, 0]
print(foo())	# [0]
```

因为，如果函数修改了该对象，那么默认值也会被修改。

再例如，声明 `Company` 类：

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

***

## 函数的流程控制

### `pass`

占位，什么也不做。其后语句仍会执行。

### `yield`

生成器函数。其后语句仍会执行。

当调用这个函数的时候，函数内部的代码并不立马执行，这个函数只是返回一个生成器对象。进行迭代的时候，才会执行，例如 `for` 循环。

第一次迭代，执行函数，`yield` 返回值作为第一次迭代的返回值。下次迭代衔接上一次迭代，直至没有值可以返回。

### `return`

结束函数，选择性地返回一个值给调用方。函数可以return几乎任意Python对象。

不带表达式的 `return` 和 没有`return`语句，相当于返回 `None`。

- return 主要作用便是控制流程：结束函数调用并返回值。
- 当需要在程序指定位置退出时，可以在该处放置。
- 如果函数执行了 return 语句，函数会立刻结束调用，return 之后的语句亦不会被执行。

函数有返回值与无返回值：

- 函数可以有返回值，也可以没有返回值。对于没有返回值的函数，功能只是完成一个操作，函数体内可以没有 return 语句。如果一个函数没有 reutrn 语句，其实它有一个隐含的 return 语句，返回值是 None，类型也是 'NoneType'。
- 二者都会执行，只是申明函数的目的稍有不同；
- 有返回值：函数执行结束，将执行的某个结果数据返回给调用者，可以给变量赋值；
- 没有返回值：函数执行结束，没有返回任何结果给调用者，直接调用即可；
- 所以，有返回值的函数可以赋值给变量，可以使用 print() 函数打印。无返回值的函数只能直接调用。
- 有返回值的函数调用时，应注意返回值和函数执行是产生的结果可能稍有不同，尤其还在 print() 函数时。

```python
def plus(x):
    print(x)
    return x + 1

plus(1)  # 仅执行
print("plus(1):", plus(1))  # 执行并打印了返回值
a = plus(1)  # # 执行并赋值了返回值
print("a:", a)
print("------")
b = plus(5)
c = b + 2
print("c:", c)


输出结果：
1
1
plus(1): 2
1
a: 2
------
5
c: 8
```

所以，当直接调用函数时，应注意其返回值与打印在屏幕上的值的差别。

一个函数可以存在多条 return 语句，但只有一条可以被执行。如果任意一条 reutrn 语句都没执行，同样会隐式调用 return None 作为返回值。

```python
def fn(x):
    if x >= 18:
        return "adult"
    elif x >=12:
        return "teenager"
    else:
        print("kid")

print(fn(20))
print(fn(15))
print(fn(3))


输出结果：
adult
teenager
kid
None
```

无论定义的是返回什么类型，return 只能返回单值。但，即是单值可以存在多个元素。上面的例子中，返回值有的是 int 型，有的是 str 型，也是是 list、tuple 类型等。

***

参考：

[函数式编程初探](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)，阮一峰

[Python 的函数是怎么传递参数的？](https://www.zhihu.com/question/20591688)，resolvewang、Kies