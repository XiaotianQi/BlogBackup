## 函数作为对象

```python
def hi(name='Bob'):
    return 'hi,'+name

print(hi())

hello = hi  # 没有在使用小括号，因为并不是在调用hi函数
print(hello())

del hi
print(hello())
```

```python
hi,Bob
hi,Bob
hi,Bob
```

删除 `hi()` 函数后，`hello()` 仍能执行，而运行 `hi()` 函数，则会提示 `NameError: name 'hi' is not defined`。

搞清楚几样东西：函数名、函数体、返回值，函数的内存地址、函数名加括号、函数名被当作参数、函数名加括号被当作参数、返回函数名、返回函数名加括号。

函数名： `foo`、`outer`、`inner`

函数体：函数的整个代码结构

返回值： return后面的表达式

函数的内存地址：`id(foo)`、`id(outer)`等等

函数名加括号：对函数进行调用，比如`foo()`、`outer(foo)`

函数名作为参数： `outer(foo)`中的foo本身是个函数，但作为参数被传递给了outer函数

函数名加括号被当做参数：其实就是先调用函数，再将它的返回值当做别的函数的参数，例如`outer(foo())`

返回函数名：`return inner`

返回函数名加括号：`return inner()`，其实就是先执行inner函数，再将其返回值作为别的函数的返回值。

如果你能理解函数也是一个对象，就能很容易地理解上面的概念。

***

## 嵌套函数

```python
def hi(name='Bob'):
    def greet():
        return "greet() function"

    def welcome():
        return "welcome() function"

    if name == 'Bob':
        return greet
    else:
        return welcome
```

在 `if/else` 语句中返回 `greet` 和 `welcome`，而不是 `greet()` 和 `welcome()`。不放括号，那么它可以作为参数传递，并且可以赋值给变量而不执行。放小括号，这个函数就会执行。

```python
f1 = hi()
print(f1)
print(f1())

f2 = hi
print(f2)
print(f2())
print(f2()())
```

```python
# f1
<function hi.<locals>.greet at 0x000001C2E4A24C80>
greet() function
```

`f1` 指向 `hi()` 函数中的 `greet()` 函数，对 `fi()` 的调用。因为在赋值时，`hi()` 已经被调用执行。且，根据 `if/else` 语句，函数 `greet` 被返回。

```python
# f2
<function hi at 0x000001C2E45F3D90>
<function hi.<locals>.greet at 0x000001C2E4A58D90>
greet() function
```

`f2` 仅指向 `hi()` 函数，对 `fi()` 的引用。

***

## 匿名函数

```python
lambda [parameter_list]: expression
```

返回函数对象。

改为定义函数模式如下：

```python
def <lambda>(parameter_list):
    return expression
```

字典排序：

```python
prices = {
    'A': 45.23,
    'B': 612.78,
    'C': 205.55,
    'D': 37.20,
    'E': 10.75
}

prices = sorted(prices.items(), key=lambda x: x[1])
print(prices)	# [('E', 10.75), ('D', 37.2), ('A', 45.23), ('C', 205.55), ('B', 612.78)]
```

跳转表(jump table)：

```python
lst = [
    lambda x: x.__name__,
    lambda x: x.__class__.__name__,
    lambda x: type(x),
    lambda x: repr(x)
]

for i in lst:
    print(i(iter))
```

```bash
In [33]: lst = [lambda x:x*i for i in range(10)]

In [34]: lst
Out[34]:
[<function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>,
 <function __main__.<listcomp>.<lambda>(x)>]

In [35]: [i(1) for i in lst]
Out[35]: [9, 9, 9, 9, 9, 9, 9, 9, 9, 9]
```

***

## 作用域

### 概念与使用

作用域指的是变量的有效范围。Python 的作用域遵循一个叫做LEGB的法则。Python 不要求申明变量，但是假定在函数定义体中赋值的变量是局部变量。

- L （Local） 局部作用域
- E （Enclosing） 闭包函数外的函数中
- G （Global） 全局作用域
- B （Built-in） 内建作用域

```python
globalVar = 100           #G
def test_scope():
    enclosingVar = 200    #E
    def func():
        localVar = 300    #L
print(__name__)           #B
```

以 `L --> E --> G -->B` 的规则查找。函数内部可以访问全局变量中的值，但是不能修改全局变量的值。如果需要修改，要加上 `global`、`nonlocal` 关键字。在L中修改G中的变量，使用 `global` 关键字。在L中修改E中的变量，使用 `nonlocal` 关键字。

使用 `module`、`class`、`def`、`lambda` 会引入一个局部作用域(L)，而且局部作用域是可以嵌套的。内层的变量会屏蔽外层，内层找不到的变量会去外层找。局部作用域会覆盖全局作用域，但不会影响全局作用域。

除了`class`、`def`、`lambda` 外，其他如: `if`、`elif`、`else`、`try`、`except`、`for`、`while`并不能改变其作用域。定义在他们之内的变量，外部还是可以访问。

全局作用域(G)就是模块内的作用域，他的作用范围是单一文件内。

Python 采用的是词法作用域（静态作用域），在书写代码或者说定义时确定作用域，而动态作用域是在运行时确定。词法作用域关注函数在何处声明，在函数定义时的环境中查询变量。而动态作用域关注函数从何处调用，在函数调用时的环境中查询变量。

实际上，在Python中，只有模块，类以及函数才会引入新的作用域，其它的代码块是不会引入新的作用域的。

```python
In [1]: if True:
   ...:     i = 1
   ...:

In [2]: print(i)
1
```

在Python中，名字绑定在所属作用域中引入新的变量，同时绑定到一个对象。名字绑定发生在以下几种情况之下：

- 参数声明：参数声明在函数的局部作用域中引入新的变量；
- 赋值操作：对一个变量进行初次赋值会在当前作用域中引入新的变量，后续赋值操作则会重新绑定该变量；
- 类和函数定义：类和函数定义将类名和函数名作为变量引入当前作用域，类体和函数体将形成另外一个作用域；
- import语句：import语句在当前作用域中引入新的变量，一般是在全局作用域；
- for语句：for语句在当前作用域中引入新的变量（循环变量）；
- except语句：except语句在当前作用域中引入新的变量（异常对象）。

### 示例1

```python
a = 10
def test():
    a += 1
    print(a)
test()
```

上述代码 ，`a += 1`报错。因为：

Python的规则是，如果在函数内部要修改一个变量，那么这个变量需要是内部变量，除非你用global声明了它是外部变量。很明显，我们没有在函数内部定义变量a，所以会弹出局部变量在未定义之前就引用的错误。

### 示例2

变量作用域在函数执行之前就已经确定了。在函数执行之前，这个函数中的变量作用域就已经被记录到内存中了，不会因为后期的调用或嵌套而更改。

Python函数的作用域取决于其函数代码块在整体代码中的位置，而不是调用时机的位置。

```python
name = 123

def func1():
    print(name)

def func2():
    name = 789
    return func1

name = 456
foo = func2()
foo()			# 456
```

`func2` 对变量的修改，无法影响 `func1` 中的变量值。调用f1的时候，会去f1函数的定义体查找，对于f1函数，它的外部是`name = 456`，而不是`name =789 `。

```python
name = 'jack'

def outer():
    name='tom'
    
    def inner():
        name ='marry'
        print(name)

    inner()

outer()				# marry
```

因为inner函数本身有name变量，所以打印结果是marry。

### 示例3

```python
l = [x + 1 for x in range(5)]
print("l:", l)
print("l[0]:", l[0])

ll = [lambda : x+1 for x in range(5)]
print("l1:", ll)
print("l[0]:", ll[0]())
```

```bash
l: [1, 2, 3, 4, 5]
l[0]: 1

l1: [<function <listcomp>.<lambda> at 0x0000025BB44A5BF8>, <function <listcomp>.<lambda> at 0x0000025BB44D8D08>, <function <listcomp>.<lambda> at 0x0000025BB44D8D90>, <function <listcomp>.<lambda> at
0x0000025BB44D8E18>, <function <listcomp>.<lambda> at 0x0000025BB44D8EA0>]
l[0]: 5
```

造成 `l[0]` 与 `l1[0]` 值不同的原因：

Python 程序执行至 `lambda` 的时候，并没有执行 `lambda` 中的代码，而是直接保存到 `l1` 列表中。随着外侧循环的结束，`x` 的值已经变成4，此时再调用保存在 `l1` 中的 `lambda` 函数，`x` 变量获取到的就是当前已经变成 4 的这个值。

这与在闭包中提到的 `for` 循环带来的问题相同。返回函数不要引用任何循环变量，或者后续会发生变化的变量。

可以修改为：

```python
def outer():
    l = []
    def inner():
        for i in range(4):
            l.append(i+1)
        return l
    return inner

# 区别于
def outer():
    l = []
    for i in range(4):
        def inner():
            l.append(i+1)
            return l
    return inner
```

或者：

```python
l = []
for i in range(4):
    def func(x = i):
        return x+1
    l.append(func)
    
# 区别于：
l = []
for i in range(4):
    def func():
        return i+1
    l.append(func)
```

***

## 闭包

### 概念与演示

在函数中可以（嵌套）定义另一个函数时，如果内部函数引用了外部函数作用域里变量（非全局变量），则可能产生闭包。大部分情况下外部作用域指的是外部函数。 简而言之，闭包指延伸了作用域的函数，其中包含函数定义体中引用，但是不在定义体中定义的非全局变量。运行时，一旦外部的函数被执行，一个闭包就形成了，闭包中包含了内部函数的代码，以及所需外部函数中的变量的引用。

闭包(closure)和类(class)有相通之处，带有面向对象的封装思维。而面向对象编程正是为了更佳的可读性和更关键的可移植性。

```python
def outer():
    fs = []
    for i in range(4):
        print("Outer:{}; i:{}".format(id(i), i))
        def inner():
            print("{} --> {}".format(id(i), i))
            return i
        fs.append(inner)
    return fs
```

调用外层函数时，执行至内层 `def inner()` 语句，仅仅是完成对内层函数 `inner` 的定义，而不会调用内层函数，除非在嵌套函数之后又显式的对其进行调用。仅仅是返回却不调用，因此通过调用函数 `outer`，可以得到内嵌函数 `inner` 的一个引用。

`fs.append(inner)` 使得 `fs` 列表中添加的是 `inner` 函数，而不是 `inner` 函数的调用结果。

```python
fun1 = outer()
print([i.__closure__ for i in fun1], end='\n')
print([i() for i in fun1])
```

```bash
Outer:1973510336; i:0
Outer:1973510368; i:1
Outer:1973510400; i:2
Outer:1973510432; i:3

[(<cell at 0x00000201618852E8: int object at 0x0000000075A16120>,),
 (<cell at 0x00000201618852E8: int object at 0x0000000075A16120>,),
 (<cell at 0x00000201618852E8: int object at 0x0000000075A16120>,),
 (<cell at 0x00000201618852E8: int object at 0x0000000075A16120>,)]

1973510432 --> 3
1973510432 --> 3
1973510432 --> 3
1973510432 --> 3
[3, 3, 3, 3]
```

`fs` 列表中的函数位于同一个 `closure` 中，所以最后使用的是同一个变量 `i`。且，变量的值在 `outer` 函数执行过程中被变成最后一个值 `i=3`。当循环结束以后，循环体中的临时变量 `i` 不会销毁，而是继续存在于执行环境中。python 的函数只有在执行时，才会去找函数体里的变量的值。当把函数 `inner` 加入 `fs` 列表时，python 还没有将 `i` 传递给函数 `inner`，只有当执行时，再去寻找 `i` 的值。当函数 `inner` 被调用时，`i` 已经是 3，所以结果呈现出 `3, 3, 3, 3`。

不过，这也凸显一种现象：本地作用域在函数结束后就立即失效，而嵌套作用域在嵌套的函数返回后却仍然有效。一个能记住嵌套作用域变量值的函数，尽管作用域已经不存在。这种现象就称为闭包。`fs` 列表中存储的都是嵌套函数 `inner`，但当再次调用时，扔会使用嵌入其中的参数 `i=3`。

当修改 `fs.append(inner)` 为 `fs.append(inner())` 时：

```python
fun1 = outer()
```

```bash
Outer:1973510336; i:0
1973510336 --> 0
Outer:1973510368; i:1
1973510368 --> 1
Outer:1973510400; i:2
1973510400 --> 2
Outer:1973510432; i:3
1973510432 --> 3
[0, 1, 2, 3]
```

`fs.append(inner())` 就是显性调用，使得 `fs` 存储的是 `f` 函数的调用结果。每次循环，`inner` 函数都会进行调用，`i` 也会跟随循环传递至函数 `inner`。所以结果呈现出 `0, 1, 2, 3`。

***

### 注意事项

* 闭包使用 `for` 循环时，临时变量 `i` 带了的问题，如上面使用的代码。`for` 循环内的变量可以在 `for` 循环结束后继续被访问。Python中的 `for` 循环并没有引入作用域(scope)的概念，但函数定义有引入作用域。

如果希望 `for` 循环变量不被其他程序误用／覆盖等，可以把 `for` 循环单独放在一个函数内:

```python
def handleForLoop(seq):
    for x in seq:
        # Do something
    return
```

* 闭包是不能修改外部作用域的局部变量。

```python
def outer():
    a = 1
    def inner():
        a = 2
        print(a)
    print(a)
    inner()
    print(a)

outer()
```

```bash
1
2
1
```

* nonloacal

```python
def outer():
    fs = 1
    def inner():
        fs = fs + 1
        return fs
    return inner
```

错误信息：`UnboundLocalError: local variable 'fs' referenced before assignment`。

使用 `nonlocal`，该语句显式的指定 `a` 不是闭包的局部变量：

```python
def outer():
    fs = 1
    def inner():
        nonlocal fs
        fs = fs + 1
        return fs
    return inner
```

使用可变对象 list：

```python
def outer():
    fs = [1]
    def inner():
        fs[0] = fs[0] + 1
        return fs
    return inner
```

### 线性方程

```python
def line_conf(a, b):
    def line(x):
        return a*x + b
    return line
```

```python
line1 = line_conf(1, 1)
line2 = line_conf(1, 2)
print(line1(5))
print(line2(5))
```

```bash
6
7
```

***

