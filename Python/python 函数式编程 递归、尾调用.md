## 递归

### 什么是递归

递归就是一个函数直接或间接的调用自身。一般来说，递归需要有**边界条件**、**递归前进段**、**递归返回段**。当边界条件不满足时，递归前进；边界条件满足时，递归返回。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/recursion%201.png)

递归的核心只有两个：

* 必须有一个明确的递归结束条件，称为**递归出口**
* 递归调用的逻辑

递归算法的出发点不在初始条件（递归边界）上，而是在求解目标上，也就是从所求的未知项出发，逐次调用本身，直至递归边界（初始条件）的求解过程。

递归是单线程运行，单激活余挂起。

* 递
  * 调用，以一定参数传递的方式传递给下家
  * 调用的目的是为了返回
* 归
  * 返回值
  * 返回控制权，上家由挂起状态转回激活状态

无论递归有多深，最终回到原点，产生递归的入口出口函数。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/recursion%202.png)

整体看来，递归过程就是：

* 传递---值、控制权
* 函数---出栈、入栈

### 什么问题适合采用递归

具备以下3个条件：

* 问题可以分解成遵循**重复**模式的多个**过程**
* 必须有一个明确的**终止条件**
* 递归深度不能太深，防止**堆栈溢出**

***

### 如何使用递归

#### 阶乘

$f(n)=n!$

$f(n)= n \times (n-1) \times (n-2) \times ... \times 1=n \times f(n-1), n>1$

```python
def f(x):
    if x == 1:
        return 1
    return x * f(x-1)
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/recursion%203.png)

#### 斐波那契数列

$f(0)=0,f(1)=1,f(n)=f(n-1) + f(n-2)$

```python
def fib(n):
    if n == 0:
        return 0
    elif n == 1:
        return 1
    return fib(n-1) + fib(n-2)
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/recursion%204.png)

一定程度节省资源：

```python
fib_mem = {0:0, 1:1}

def fib(n):
    if n in fib_mem.keys():
        return fib_mem[n]
    fib_mem[n] = fib(n-1) + fib(n-2)
    return fib_mem[n]
```

#### 汉诺塔

```python
def move(n, a, b, c):
    if n == 1:
        print('move', a, '-->', c)
    else:
        move(n-1, a, c, b)
        move(1, a, b, c)
        move(n-1, b, a, c
```

下图来自优酷一个老师视频，因为以前截的图，现在找不到出处了。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/hanoi.png)

#### 格雷码

给定一个整数n，请返回n位的格雷码，顺序为从0开始。

```python
def func(n):
    gray = []
    if n == 1:
        gray.append('0')
        gray.append('1')
        return gray
    for i in range(pow(2, n-1)):
        gray.append('0' + func(n-1)[i])
    for i in range(pow(2, n-1)-1, -1, -1):
        gray.append('1' + func(n-1)[i])
    return gray
```

改进：

```python
def gray_code(n):
    gray = []
    if n == 1:
        gray.append('0')
        gray.append('1')
        return gray
    gray_pre = func(n-1)
    for i in range(len(gray_pre)):
        gray.append('0' + gray_pre[i])
    for i in range(len(gray_pre)-1, -1, -1):
        gray.append('1' + gray_pre[i])
    return gray
```

***

### 记录递归次数

一般用全局变量可以解决，以斐波那契数列为例：

```python
count = 0

def fib(n):
    global count
    count += 1
    
    if n == 0:
        return 0
    if n == 1:
        return 1
    return fib(n-1) + fib(n-2)

fib(5)
print(count)	# 15

# 副作用
fib(5)
print(count)	# 30
```

不过会产生副作用。如果其中使用`assert(5) == fib(5)`语句，`count`还会再计算一次。

以下方法均会产生同样的副作用。这些做法本质都是修改**作用域**外的值。及时如下这样使用装饰器，更多的也仅仅是将数据封装，也没有消除副作用。

```python
# 函数装饰器1
def callCounter(func):
    def counter(*args, **kwargs):
        count[0] += 1					# 使用可变对象
        print(count)
        return func(*args, **kwargs)
    count = [0]
    return counter

# 函数装饰器2
def callCounter(func):
    def counter(*args, **kwargs):
        nonlocal count					# 使用nonlocal
        count += 1
        print(count)
        return func(*args, **kwargs)
    count = 0
    return counter
```

也可以通过给函数**设置属性**来实现，不过也会带来副作用：

```python
def fib(n):
    fib.count += 1

    if n==0:
        return 0
    elif n==1:
        return 1
    else:
        return fib(n-1) + fib(n-2)

fib.count = 0
print(fib(5))
print(fib.count)	# 15
print(fib(5))
print(fib.count)	# 30
fib.count = 0
print(fib(5))
print(fib.count)	# 15
```

```python
# 如下这个装饰器，并没有规避副作用
def callCounter(func):
    def counter(*args, **kwargs):
        counter.calls += 1
        return func(*args, **kwargs)
    counter.calls = 0
    return counter
```

```python
# 类装饰器
class CallCounter(object) :
    def __init__(self, fun) :
        self._fun = fun
        self.counter=0
    def __call__(self,*args, **kwargs) :
        self.counter += 1
        return self._fun(*args, **kwargs)

@CallCounter
def fib(n):  
    if n == 0:
        return 0
    if n == 1:
        return 1
    return fib(n-1) + fib(n-2)

print(fib(5))
print(fib.counter) # 15
print(fib(5))
print(fib.counter) # 30
```

使用嵌套函数可以解决：

```python
def fib(n):
    count = 0
    def inner(n):
        nonlocal count
        count += 1
        if n==0:
            return 0
        elif n==1:
            return 1
        else:
            return inner(n-1) + inner(n-2)
    return inner(n), count
    
fib(5)	# (5, 15)
fib(5)	# (5, 15)
```

***

### 循环、迭代、递推、递归

**循环**：循环是计算机科学运算领域的用语，也是一种常见的控制流程。循环中的代码会运行特定的次数，或者是运行到特定条件成立时结束循环，或者是针对某一集合中的所有项目都运行一次。所有能用递归解决的问题，都可以用循环来解决。

**迭代**：迭代是重复反馈过程的活动，其目的通常是为了接近并到达所需的目标或结果。每一次对过程的重复被称为一次“迭代”，而每一次迭代得到的结果会被用来作为下一次迭代的初始值。当使用一个循环来遍历时，这个过程本身就叫迭代。

**递归**（英语：recursion）：递归在计算机科学中是指一种通过重复将问题分解为同类的子问题而解决问题的方法。递归式方法可以被用于解决很多的计算机科学问题，因此它是计算机科学中十分重要的一个概念。绝大多数编程语言支持函数的自调用，在这些语言中函数可以通过调用自身来进行递归。计算理论可以证明递归的作用可以完全取代循环，因此有很多在函数编程语言（如Scheme）中用递归来取代循环的例子。**从未知到已知**。

**递推**：数学概念，从初值出发反复进行某一运算得到所需结果。**从已知到未知**。

以斐波那契函数为例：

```python
# 递归实现
def fib(n):
    if n==0:
        return 0
    elif n==1:
        return 1
    else:
        return fib(n-1) + fib(n-2)

# 递推实现
a, b = 0 , 1
for i in range(7):
    if i == 0:
        print(0)
    if i == 1:
        print(1)
    else:
        a, b = b, a + b
        print(b)
```

递归是逆向的，从 n 到1， 递推是正向的，从1到n。

对于求斐波那契数列，递归是把问题从 n 开始分解成 n-1、n-2、... 2、1,是从大到小分解求解，是逆向的。

对于递推，则是先求f(2)、f(3)，再求 f(n),是从小到大，是正向的。



***

## 尾调用

尾调用（tail call）指的是一个函数的最后一条语句也是一个返回调用函数的语句。在函数体末尾被返回的可以是对另一个函数的调用，也可以是对自身调用（即自身递归调用）。若这个函数在尾位置调用本身（或是一个尾调用本身的其他函数等等），则称这种情况为**尾递归**。尾调用之所以与其他调用不同，就在于它的特殊的调用位置。

```python
def func(x):
    return g(x)		# 尾调用

def func(x):
    y = g(x)
    return y		# 不是尾调用

def func(x):
    return g(x)+1	# 不是尾调用

def func(x):
    if x > 0:
    	return m(x)	# 尾调用
   	return n(x)		# 尾调用
```

尾调用不一定出现在函数尾部，只要是最后一步操作即可。

### 尾调用优化

传统模式的编译器对于尾调用的处理方式就像处理其他普通函数调用一样，总会在调用时创建一个新的栈帧（stack frame）并将其推入调用栈顶部，用于表示该次函数调用。当一个函数调用发生时，电脑必须 “记住” 调用函数的位置 —— 返回位置，才可以在调用结束时带着返回值回到该位置，返回位置一般存在调用栈上。在尾调用这种特殊情形中，电脑理论上可以不需要记住尾调用的位置而从被调用的函数直接带着返回值返回调用函数的返回位置（相当于直接连续返回两次）。

尾调用原则上都可以通过简化函数调用栈的结构而获得性能优化（称为“尾调用消除”），但是优化尾调用是否方便可行取决于运行环境对此类优化的支持程度如何。 

在程序运行时，计算机会为应用程序分配一定的内存空间；应用程序则会自行分配所获得的内存空间，其中一部分被用于记录程序中正在调用的各个函数的运行情况，这就是函数的**调用栈**。

常规的函数调用总是会在调用栈最上层添加一个新的**堆栈帧**（stack frame，也翻译为“栈帧”或简称为“帧”），这个过程被称作“入栈”或“压栈”（意即把新的帧压在栈顶）。

当函数的调用层数非常多时，调用栈会消耗不少内存，甚至会撑爆内存空间（**栈溢出**），造成程序严重卡顿或意外崩溃。尾调用的调用栈则特别易于优化，从而可减少内存空间的使用，也能提高运行速度。其中，对尾递归情形的优化效果最为明显，尤其是递归算法非常复杂的情形。

**调用帧**（call frame），保存调用位置和内部变量等信息。如果在函数A的内部调用函数B，那么在A的调用记录上方，还会形成一个B的调用记录。等到B运行结束，将结果返回到A，B的调用记录才会消失。如果函数B内部还调用函数C，那就还有一个C的调用记录栈，以此类推。所有的调用记录，就形成一个"调用栈"（call stack）。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/recursion%20call%20frame.png)

尾调用由于是函数的最后一步操作，所以**不需要保留外层函数的调用记录**，因为调用位置、内部变量等信息都不会再用到了，只要**直接用内层函数的调用记录**，取代外层函数的调用记录就可以了。

```python
def func(x, y):
    x = 1
    y = 2
    return g(x + y)
func()

# 等同于
def func():
    return g(3)
func()

# 等同于
g(3)
```

上面代码中，如果函数g不是尾调用，函数f就需要保存内部变量m和n的值、g的调用位置等信息。但由于调用g之后，函数f就结束了，所以执行到最后一步，完全可以删除 f() 的调用记录，只保留 g(3) 的调用记录。

这就叫做"尾调用优化"（Tail call optimization），即只保留内层函数的调用记录。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用记录只有一项，这将大大节省内存。这就是"尾调用优化"的意义。

### 尾递归

若函数在尾位置调用自身（或是一个尾调用本身的其他函数等等），则称这种情况为**尾递归**。经过适当处理，尾递归形式的函数的运行效率可以被极大地优化。

递归非常耗费内存，因为需要同时保存成千上百个调用记录，很容易发生"栈溢出"错误（stack overflow）。但对于尾递归来说，由于只存在一个调用记录，所以永远不会发生"栈溢出"错误。

尾递归在普通尾调用的基础上，多出了2个特征： 

- 在尾部调用的是函数自身 (Self-called)；
- 可通过优化，使得计算仅占用常量栈空间 (Stack Space)。

```python
def factorial(n):
    if n == 1: return 1
    return n * factorial(n-1)

factorial(5)
```

```text
factorial(5)
5 * factorial(4)
5 * (4 * factorial(3))
5 * (4 * (3 * factorial(2)))
5 * (4 * (3 * (2 * factorial(1))))
5 * (4 * (3 * (2 * 1)))
5 * (4 * (3 * 2))
5 * (4 * 6)
5 * 24
120
```

上面代码是一个阶乘函数，计算n的阶乘，最多需要保存n个调用记录，复杂度 O(n) 。

如果改写成尾递归，只保留一个调用记录，复杂度 O(1) 。

```python
def factorial(n, result):
    if n == 1: return result
    return factorial(n - 1, n * result)

factorial(5, 1)
```

```text
factorial(5, 1)
factorial(4, 5)
factorial(3, 20)
factorial(2, 60)
factorial(1, 120)
120
```

对函数调用在尾位置的递归或互相递归的函数，由于函数自身调用次数很多，递归层级很深，尾递归优化则使原本 O(n) 的调用栈空间只需要 O(1)。

### 递归函数的改写

尾递归的实现，往往需要改写递归函数，确保最后一步只调用自身。做到这一点的方法，就是把所有用到的内部变量改写成函数的参数。比如上面的例子，阶乘函数  factorial 需要用到一个中间变量 total  ，那就把这个中间变量改写成函数的参数。这样做的缺点就是不太直观，第一眼很难看出来，为什么计算5的阶乘，需要传入两个参数5和1？

两个方法可以解决这个问题。方法一是在尾递归函数之外，再提供一个正常形式的函数。

```python
def tailFactorial(n, result):
    if n == 1: return result
    return tailFactorial(n - 1, n * result)

def factorial(n):
    return tailFactorial(n, 1)
```

或者采用柯里化。柯里化（英语：Currying），又译为卡瑞化或加里化，是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

python 中可以使用 `functools.partial(func, *args, **keywords)` 函数

```python
from functools import partial

def tailFactorial(n, result):
    if n == 1: return result
    return tailFactorial(n - 1, n * result)

factorial = partial(tailFactorial, result=1)
```

方法二就更简单了，使用默认参数

```python
def factorial(n, result=1):
    if n == 1: return result
    return factorial(n - 1, n * result)
```

递归本质上是一种循环操作。纯粹的函数式编程语言没有循环操作命令，所有的循环都用递归实现，这就是为什么尾递归对这些语言极其重要。

斐波那契函数改写：

```python
def fib(num, res, temp):
    if num == 0:
        return res 
    else:
        return fib(num-1, temp, res+temp)
```

***

参考：

[递归](http://blog.timd.cn/recursive/)，Tim

[你一定听得懂的递归教程！](https://www.bilibili.com/video/BV1954y1Q7u8)，爱可可-爱生活

[尾调用优化](http://www.ruanyifeng.com/blog/2015/04/tail-call.html)，阮一峰