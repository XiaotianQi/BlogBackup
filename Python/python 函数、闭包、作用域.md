## 参数

只有变量前面的 `*` 才是必须的。可以写成 `*var` 和 `**vars`。`*args` 和 `**kwargs` 只是一个通俗的命名约定。

`*args` 和 `**kwargs` 主要用于函数定义。可以将不定数量的参数传递给函数。预先不知道函数使用者会传递多少个参数的场景下使用这两个关键字。

### 可变参数 `*agrs`

`*args` 是将来一个**非键值对**的可变数量的参数列表传递给函数。接收的是一个 `tuple`。既可以直接传入：`func(1, 2, 3)`，又可以先组装 `list` 或 `tuple`，再通过 `*args` 传入：`func(*(1, 2, 3))`。

### 关键字参数 `**kwargs`

`**kwargs` 允许将不定长度的**键值对**作为参数传递给函数。接收的是一个 `dict`。既可以直接传入：`func(a=1, b=2)`，又可以先组装 `dict`，再通过 `**kwargs` 传入：`func(**{'a': 1, 'b': 2})`。

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

***

## `pass`、`break`、`continue`、`yield`、`return`

### `pass`

占位，什么也不做。其后语句仍会执行。

### `break`

结束循环。其后语句不再执行。

### `continue`

跳出本次循环，进行下一次循环。其后语句不再执行。

```python
i = 0
while i < 10 :
    i += 1
    if i == 4:
        pass
    if i == 6:
        continue
    if i == 8:
        break
    print(i)
else:
    print('It is over.')
```

```bash
1
2
3
4
5
7
```

循环至6时，符合条件，执行 `continue`，所以6并未输出。

循环至8时，符合条件，执行 `break`，跳出循环。

```python
for x in range(3):
    for y in range(3):
        for z in range(3):
            print(x,y,z)
            if x*y*z == 2:
                break
        else:
            continue
        break
    else:
        continue
    break
```

```bash
0 0 0
0 0 1
0 0 2
0 1 0
0 1 1
0 1 2
0 2 0
0 2 1
0 2 2
1 0 0
1 0 1
1 0 2
1 1 0
1 1 1
1 1 2
```

循环体的 `else` 分支触发条件是循环正常结束。如果循环内被 `break` 跳出，就不执行 `else`。

循环至 1 1 2时，符合条件，执行 `break`，跳出 z 的循环。

因 z 循环跳出，`else` 不会执行，`break` 执行，跳出 y 的循环。

同上，x 循环亦被 `break` 跳出。

### `yield`

生成器函数。其后语句仍会执行。

当调用这个函数的时候，函数内部的代码并不立马执行，这个函数只是返回一个生成器对象。进行迭代的时候，才会执行，例如 `for` 循环。

第一次迭代，执行函数，`yield` 返回值作为第一次迭代的返回值。下次迭代衔接上一次迭代，直至没有值可以返回。

### `return`

结束函数，选择性地返回一个值给调用方。不带表达式的 `return` 和 没有`return`语句，相当于返回 `None`。

更多关于函数返回值的说明，参考以前一篇博文：

[python 函数有返回值与无返回值](http://starrynight.tech/post/19/)

***

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

## 闭包

### 概念与演示

内部函数对外部函数作用域里变量的引用（非全局变量），则称内部函数为闭包。简而言之，闭包指延伸了作用域的函数，其中包含函数定义体中引用，但是不在定义体中定义的非全局变量。

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

## 作用域

### 概念与使用

Python 的作用域遵循一个叫做LEGB的法则。Python 不要求申明变量，但是假定在函数定义体中赋值的变量是局部变量。

* L （Local） 局部作用域
* E （Enclosing） 闭包函数外的函数中
* G （Global） 全局作用域
* B （Built-in） 内建作用域

```python
globalVar = 100           #G
def test_scope():
    enclosingVar = 200    #E
    def func():
        localVar = 300    #L
print(__name__)           #B
```

以 `L --> E --> G -->B` 的规则查找。函数内部可以访问全局变量中的值，但是不能修改全局变量的值。如果需要修改，要加上 `global`、`nonlocal` 关键字。在L中修改G中的变量，使用 `global` 关键字。在L中修改E中的变量，使用 `nonlocal` 关键字。

使用 `module`、`class`、`def`、`lambda` 会引入一个本地作用域(L)，而且本地作用域是可以嵌套的。内层的变量会屏蔽外层，内层找不到的变量会去外层找。局部作用域会覆盖全局作用域，但不会影响全局作用域。

除了`class`、`def`、`lambda` 外，其他如: `if`、`elif`、`else`、`try`、`except`、`for`、`while`并不能改变其作用域。定义在他们之内的变量，外部还是可以访问。

全局作用域(G)就是模块内的作用域，他的作用范围是单一文件内。

***

### tips

变量作用域在函数执行之前就已经确定了。在函数执行之前，这个函数中的变量作用域就已经被记录到内存中了，不会因为后期的调用或嵌套而更改。

```python
name = "123"

def func1():
    print(name)

def func2():
    name = "456"
    return func1

foo = func2()
foo()
```

```bash
123
```

`func2` 对变量的修改，无法影响 `func1` 中的变量值。

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

这与在闭包中提到的 `for` 循环带来的问题相同。

可以修改为：

```python
def outer():
    l = []
    def inner():
        for i in range(4):
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
```