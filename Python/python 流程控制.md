流程控制指的是代码运行逻辑、分支走向、循环控制，是真正体现我们程序执行顺序的操作。流程控制一般分为顺序执行、条件判断和循环控制。流程控制一般分为顺序执行、条件判断和循环控制。这里面体现了一种传统编程中的“因果关系”，也就是有什么样的因就产生什么样的果，有什么输入就会有相应的输出，同一个输入不管执行多少次必然得到同样的输出，所有的都是确定的、可控的。与之相对的则是最近火得一塌糊涂的人工智能，比如机器学习、神经网络和深度学习，这些不再是“因果关系”编程，而是“统计关系”编程，同一个输入每次的执行结果有可能不一样。

## 伪代码

伪代码(Pseudocode)是一种算法描述语言。使用伪代码的目的是使被描述的算法可以容易地以任何一种编程语言实现。因此，伪代码必须结构清晰、代码简单、可读性好，并且类似自然语言。 通俗简单地讲，就是用中文把你的程序先写出来，在检查过伪代码没有问题后，再用Python等语言将其真正实现。

```text
输入一个变量age 
将输入字符串转换为数字
（条件判断：）
如果age大于等于18：
    打印“成人”
如果age小于18，又分两种情况：
    如果age大于等于12:
        打印“少年”
    如果age小于12：
        打印“儿童”
```

***

## 流程图

程序流程图和伪代码在本质上其实是一样的，只不过一个用文字表达一个用图片表达，图片画着费点事，但看着直观，文字书写方便，但不够直观。

***

## 顺序执行

虽然我们有各种流程判断、循环、跳转、控制、中断等等，但从根本上程序还是逐行顺序执行的。

Python代码在执行过程中，遵循下面的基本原则：

- 普通语句，直接执行；
- 碰到函数，将函数体载入内存，并不直接执行
- 碰到类，执行类内部的普通语句，但是类的方法只载入，不执行
- 碰到if、for等控制语句，按相应控制流程执行
- 碰到@，break，continue等，按规定语法执行
- 碰到函数、方法调用等，转而执行函数内部代码，执行完毕继续执行原有顺序代码

## 条件判断

一条一条语句顺序执行的Python代码，这种代码结构通常称之为顺序结构。

条件判断是通过一条或多条判断语句的执行结果（True或者False）来决定执行的代码块。将这种结构称之为“分支结构”或“选择结构”。

在Python语法中，使用`if`、`elif`和`else`三个关键字来进行条件判断。

顺序判断每一个分支，任何一个分支首先被命中并执行，则其后面的所有分支被忽略，直接跳过！

输入三条边长如果能构成三角形就计算周长和面积：

```python
import math

a = float(input('a = '))
b = float(input('b = '))
c = float(input('c = '))
if a + b > c and a + c > b and b + c > a:
    print('周长: %f' % (a + b + c))
    p = (a + b + c) / 2
    area = math.sqrt(p * (p - a) * (p - b) * (p - c))
    print('面积: %f' % (area))
else:
    print('不能构成三角形')
```

***

### 条件语句的其他写法

```python
>>> age1 = 17
>>> age2 = 20
```

**1**

```python
if age > 18:
    return "已成年"
else:
    return "未成年"
```

**2**

```python
<on_true> if <condition> else <on_false>
```

```python
>>> '已成年' if age1 > 18 else '未成年'
'未成年'
>>> '已成年' if age2 > 18 else '未成年'
'已成年'
```

**3**

```python
<condition> and <on_true> or <on_false>
```

```python
>>> age1 >18 and '已成年' or '未成年'
'未成年'
>>> age2 >18 and '已成年' or '未成年'
'已成年'
```

```python
((<condition>) and (<on_true>,) or (<on_false>,))[0]
```

```python
>>> ((age1 > 18) and ('已成年',) or ('未成年',))[0]
'未成年'
>>> ((age1 > 18) and ('已成年',) or ('未成年',))[0]
'已成年'
```

**4**

```python
(<on_false>, <on_true>)[condition]
```

```python
>>> ('未成年', '已成年')[age1 > 18]
'未成年'
>>> ('未成年', '已成年')[age2 > 18]
'已成年'
```

**5**

```python
(lambda: <on_false>, lambda:<on_true>)[<condition>]()
```

```python
>>> (lambda:'未成年', lambda:'已成年')[age1 > 18]()
'未成年'
>>> (lambda:'未成年', lambda:'已成年')[age2 > 18]()
'已成年'
```

**6**

```python
{True: <on_true>, False: <on_false>}[<condition>]
```

```python
>>> {True: '未成年', False: '已成年'}[age1 > 18]
'未成年'
>>> {True: '未成年', False: '已成年'}[age2 > 18]
'已成年'
```

***

### 成员判断

`''`值得注意。

```python
>>> '' in ''
True
>>> '' in 'abc'
True
```

```python
>>> 'abc'.count('a')
1
>>> 'abc'.count('')
4
```

**in 和 not in**

```python
>>> 'a' in 'abc'
True
```

**使用 str.find() 方法**

```python
>>> 'abc'.find('b') != -1
True
```

***

## 循环控制

循环是计算机科学运算领域的用语，也是一种常见的控制流程。循环是一段在程序中只出现一次，但可能会连续运行多次的代码。循环中的代码会运行特定的次数，或者是运行到特定条件成立时结束循环，或者是针对某一集合中的所有项目都运行一次。

* 指定运行次数的循环(`for loop`)
  * python不支持。不过，指定次数的循环可以用重复incrementing list或generator的方式来达到其效果，`range()`。
* 指定条件的循环(`while loop`/`doWhile loop`)
  * Python 不支持 `do〜while` 语法、可以使用 `while`和 `if...break` 组合起来实现 `do ~ while` 语法
* 指定集合的循环（遍历）
* 死循环

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/flow.png)

在Python中构造循环结构有两种做法，一种是`for-in`循环，一种是`while`循环。虽然上表中有些功能python并没有直接支持，但是可以通过其他方法实现。

### for 循环

如果明确的知道循环执行的次数或者要对一个容器进行迭代（后面会讲到），那么推荐使用`for-in`循环。

for循环通常用来遍历可迭代的对象，`for ... in ....`属于固定套路。

与while一样，for循环也可以有else子句。同样是正常结束循环时，else子句执行。被中途break时，则不执行。

除此之外，在推导式和生成器表达式中，for循环也被使用。

for循环可以胜任很多任务：遍历一个序列以求的总数或者挑出来特定的元素、用来计算总和或者平均数等等。

```python
for i in range(10):
    if i == 5:
        break
print(i)	# 5
```

除此外，还有`for...else...`，与 `try...else...` 相同，只要代码不被 break，不抛出异常，就可以执行`else`。

### while 循环

while循环用伪代码描述就是：当条件满足的时候，就一直运行while所管理的代码块，当条件不满足的时候，就结束while循环。

如果要构造不知道具体循环次数的循环结构，推荐使用`while`循环。`while`循环通过一个能够产生或转换出`bool`值的表达式来控制循环，表达式的值为`True`循环继续，表达式的值为`False`循环结束。

while循环还可以增加一个else从句。当while循环正常执行完毕，会执行else语句。但如果是被break等机制强制提前终止的循环，不会执行else语句。

### 循环控制

#### break

break只能用于循环体内。其效果是直接结束并退出**当前**循环，剩下的未循环的工作全部被忽略和取消。注意当前两个字，Python的break只能退出一层循环，对于多层嵌套循环，不能全部退出。

#### continue

与break不同，continue语句用于跳过当前循环的剩余部分代码，直接开始下一轮循环。它不会退出和终止循环，只是提前结束当前轮次的循环。同样的，continue语句只能用在循环内。

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

- `for/else`

`else` 会在循环正常结束时执行。这意味着，循环没有遇到任何 `break`。根据这个特点，可以清楚掌握循环结束的原因，或者获取与 `break` 相左的信息。

找出2到10之间的数字的因子和质数：

```python
for n in range(2, 10):
    for x in range(2, n):
        if n % x == 0:
            print(n, 'equals', x, '*', n / x)
            break
    else:
        print(n, 'is a prime number')
```

## 构造程序逻辑

分支和循环结构会帮助我们将程序中逻辑建立起来，将来我们的程序无论简单复杂，都是由顺序结构、分支结构、循环结构构成的。

## 练习

素数判断：

```python
def is_prime(num):
    end = int(sqrt(num))
    is_prime = True
    for x in range(2, end + 1):
        if num % x == 0:
            is_prime = False
            break
    if is_prime and num != 1:
        print('%d是素数' % num)
    else:
        print('%d不是素数' % num)
```

输入两个正整数，计算最大公约数和最小公倍数:

```python
x = int(input('x = '))
y = int(input('y = '))
if x > y:
    x, y = y, x
for factor in range(x, 0, -1):
    if x % factor == 0 and y % factor == 0:
        print('%d和%d的最大公约数是%d' % (x, y, factor))
        print('%d和%d的最小公倍数是%d' % (x, y, x * y // factor))
        break
```

输出三角形：

```python
for i in range(row):
    for _ in range(row - i - 1):
        print(' ', end='')
    for _ in range(2 * i + 1):
        print('*', end='')
    print()
```

水仙花数：

```python
for num in range(100, 1000):
    x, y, z = str(num)
    nar_num = int(x)**3 + int(y)**3 + int(z)**3
    if num == nar_num:
        print(num)
```

完全数：

```python
import math

for num in range(1, 10000):
    sum = 0
    for factor in range(1, int(math.sqrt(num)) + 1):
        if num % factor == 0:
            sum += factor
            if factor > 1 and num / factor != factor:
                sum += num / factor
    if sum == num:
        print(num)
```

百鸡百钱：

```python
for x in range(0, 20):
    for y in range(0, 33):
        z = 100 - x - y
        if 5 * x + 3 * y + z / 3 == 100:
            print('公鸡{0}只，母鸡{1}只，雏鸡{2}只'.format(x, y, z))
```

斐波那契数列：

```python
def fib(num):
    a, b = 0, 1
    for i in range(num):
        yield a
        a, b = b, a+b
```

Craps赌博游戏：

```python
from random import randint

money = 1000
while money > 0:
    print('你的总资产为:', money)
    needs_go_on = False
    while True:
        debt = int(input('请下注: '))
        if debt > 0 and debt <= money:
            break
    first = randint(1, 6) + randint(1, 6)
    print('玩家摇出了%d点' % first)
    if first == 7 or first == 11:
        print('玩家胜!')
        money += debt
    elif first == 2 or first == 3 or first == 12:
        print('庄家胜!')
        money -= debt
    else:
        needs_go_on = True

    while needs_go_on:
        current = randint(1, 6) + randint(1, 6)
        print('玩家摇出了%d点' % current)
        if current == 7:
            print('庄家胜')
            money -= debt
            needs_go_on = False
        elif current == first:
            print('玩家胜')
            money += debt
            needs_go_on = False

print('你破产了, 游戏结束!')
```

***

参考：

[流程控制](http://www.liujiangblog.com/course/python/25)，刘江

[分支结构](https://github.com/jackfrued/Python-100-Days/blob/master/Day01-15/03.%E5%88%86%E6%94%AF%E7%BB%93%E6%9E%84.md)，骆昊

[条件语句的七种写法](http://magic.iswbm.com/zh/latest/c03/c03_04.html)，王炳明