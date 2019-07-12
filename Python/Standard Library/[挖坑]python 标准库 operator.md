## `operator.itemgetter`

`itemgetter` 用于获取对象的维度的数据，参数即为代表维度的序号值。

```python
a = [1,2,3]
b = [
	[1,2,3],
	[4,5,6],
	[7,8,9]
] 
```

列表 `a` 可以看做 1 维数据，3个点：a[0] = 1, a[1] = 2, a[2] = 3。
列表 `b` 则是 2 维数据，9个点：b[0,0] = 1, b[0,1] = 2, b[0,2] = 3, b[1,0] = 4, …, b[2,2] = 9。

`itemgette`r 获取的不是值，而是定义了一个函数，通过该函数作用到目标对象，取出目标对象对应维度的值 ，且获得的对象是 1 维对象。

```bash
In [5]: get_1 = operator.itemgetter(1)

In [6]: get_1(a)
Out[6]: 2

In [7]: get_1(b)
Out[7]: [4, 5, 6]

In [8]: get_21 = operator.itemgetter(2, 1)

In [9]: get_21(a)
Out[9]: (3, 2)

In [10]: get_21(b)
Out[10]: ([7, 8, 9], [4, 5, 6])

In [11]: [get_21(item) for item in b]
Out[11]: [(3, 2), (6, 5), (9, 8)]
```