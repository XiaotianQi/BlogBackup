在Python标准库中居然有两个模块处理时间，一个叫time，另外一个叫datetime，里面提供了类似的方法但是两个完全不是一回事。到这还没完，标准库里还有一个叫calendar的模块，也是用来处理时间的。

arrow是一个专门处理时间和日期的轻量级Python库，它提供了一种**合理、智能**的方式来创建、操作、格式化、转换时间和日期。

```pyhon
In [1]: import arrow

# 获取当前时间
In [2]: utc = arrow.utcnow()

In [3]: utc
Out[3]: <Arrow [2019-08-16T06:44:05.994965+00:00]>

# 调整时间
In [4]: utc = utc.shift(days=+1, hours=-1)

In [5]: utc
Out[5]: <Arrow [2019-08-17T05:44:05.994965+00:00]>

# 修改时间
In [6]: utc.replace(hour=4, minute=40)
Out[6]: <Arrow [2019-08-17T04:40:05.994965+00:00]>

# 转换时区
In [7]: local = utc.to('US/Pacific')

In [8]: local
Out[8]: <Arrow [2019-08-16T22:44:05.994965-07:00]>

# 从文本转为时间对象
In [9]: arrow.get('2019-08-16T22:44:05.994965-07:00')
Out[9]: <Arrow [2019-08-16T22:44:05.994965-07:00]>

In [10]: arrow.get('June was born in May 1980', 'MMMM YYYY')
Out[10]: <Arrow [1980-05-01T00:00:00+00:00]>

# 获取时间戳
In [11]: local.timestamp
Out[11]: 1566020645

# 从时间戳转为时间对象
In [12]: arrow.get('1566020645')
Out[12]: <Arrow [2019-08-17T05:44:05+00:00]>

# 格式化输出
In [13]: local.format()
Out[13]: '2019-08-16 22:44:05-07:00'

In [14]: local.format('YYYY-MM-DD HH:mm:ss')
Out[14]: '2019-08-16 22:44:05'

In [15]: local.humanize()
Out[15]: 'in a day'

# 转为标准库对象
In [16]: local.date()
Out[16]: datetime.date(2019, 8, 16)

In [17]: local.time()
Out[17]: datetime.time(22, 44, 5, 994965)
```

更多内容可以通过官方文档获得：https://arrow.readthedocs.io/en/latest/#api-guide

***

参考：

[使用Python优雅地处理时间数据](https://segmentfault.com/a/1190000011478102)，betacat