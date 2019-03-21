`time.time()`和`time.clock()`方法可以用来计算程序执行时间及cpu时间。但是，很多时候我们只想对某些代码片段或者算法进行执行时间的统计，这时候，使用timeit模块就比较方便。timeit模块是Python内置的用于统计小段代码执行时间的模块，它同时提供命令行调用接口。

## Python接口

**timeit.timeit(stmt='pass', setup='pass', timer=, number=1000000, globals=None)**

创建一个Timer实例，并运行代码进行计时，默认将代码执行一百万次。

`stmt`是要执行的代码，字符串形式，多行就多个字符串。`setup`是执行前的环境配置，比如import语句。`timer`是使用的计时器。`number`是执行的次数。`globals`是执行的命名空间。

**timeit.repeat(stmt='pass', setup='pass', timer=, repeat=3, number=1000000, globals=None)**

指定重复次数的执行timeit方法，返回一个结果列表。

**timeit.default_timer()**

默认的计时器，也就是`time.perf_counter()`

**class timeit.Timer(stmt='pass', setup='pass', timer=, globals=None)**

用于进行代码执行速度测试的计时类。该类有四个方法：

- timeit(number=1000000)
- autorange(callback=None)
- repeat(repeat=3, number=1000000)
- print_exc(file=None)

```python
def test():
    """Stupid test function"""
    L = [i for i in range(100)]

if __name__ == '__main__':
    import timeit
    print(timeit.timeit("test()", setup="from __main__ import test"))
```

## 命令行界面

命令格式： `python -m timeit [-n N] [-r N] [-u U] [-s S] [-t] [-c] [-h] [语句 ...]`

参数：

```
-n：执行次数
-r：计时器重复次数
-s：执行环境配置（通常该语句只被执行一次）
-p：处理器时间
-v：打印原始时间
-h：帮助
```

***

参考：

http://www.liujiangblog.com/course/python/70