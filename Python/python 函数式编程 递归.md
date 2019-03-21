递归的核心只有两个：

* 递归结束的语句

* 递归调用的逻辑

***

## 斐波那契数列

```python
def fib(n):
    if n == 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fib(n-1) + fib(n-2)
```

***

## 汉诺塔

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

***

## 格雷码

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

