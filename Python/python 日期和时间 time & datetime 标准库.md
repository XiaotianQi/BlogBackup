在Python中，与时间处理有关的模块包括`time`，`datetime`以及`calendar`

## time

在Python中，用三种方式来表示时间，分别是时间戳、格式化时间字符串和结构化时间

- 时间戳（`timestamp`）：也就是1970年1月1日之后的秒，例如1506388236.216345，可以通过`time.time()`获得。时间戳是一个浮点数，可以进行加减运算，但请注意不要让结果超出取值范围。
- 格式化的时间字符串（`string_time`）：也就是年月日时分秒这样的我们常见的时间字符串，例如`2017-09-26  09:12:48`，可以通过`time.localtime()`获得;
- 结构化时间（`struct_time`）：一个包含了年月日时分秒的多元元组，例如`time.struct_time(tm_year=2017, tm_mon=9, tm_mday=26, tm_hour=9, tm_min=14, tm_sec=50, tm_wday=1, tm_yday=269, tm_isdst=0)`，可以通过`time.strftime('%Y-%m-%d')`获得。

### 结构化时间

使用`time.localtime()`等方法可以获得一个结构化时间元组。

```python
>>> time.localtime()
time.struct_time(tm_year=2019, tm_mon=1, tm_mday=31, tm_hour=14, tm_min=8, tm_sec=58, tm_wday=3, tm_yday=31, tm_isdst=0)
```

结构化时间元组共有9个元素，按顺序排列如下表：

| 索引 | 属性                      | 取值范围           |
| ---- | ------------------------- | ------------------ |
| 0    | tm_year（年）             | 比如2017           |
| 1    | tm_mon（月）              | 1 - 12             |
| 2    | tm_mday（日）             | 1 - 31             |
| 3    | tm_hour（时）             | 0 - 23             |
| 4    | tm_min（分）              | 0 - 59             |
| 5    | tm_sec（秒）              | 0 - 61             |
| 6    | tm_wday（weekday）        | 0 - 6（0表示周一） |
| 7    | tm_yday（一年中的第几天） | 1 - 366            |
| 8    | tm_isdst（是否是夏令时）  | 默认为-1           |

既然结构化时间是一个元组，那么就可以通过索引进行取值，也可以进行分片，或者通过属性名获取对应的值。但是，Python的time类型是不可变类型，所有的时间值都只读，不能改。

### 格式化时间字符串

利用`time.strftime('%Y-%m-%d %H:%M:%S')`等方法可以获得一个格式化时间字符串。

```python
>>> time.strftime('%Y-%m-%d %H:%M:%S')
'2019-01-31 14:11:07'
```

注意其中的空格、短横线和冒号都是美观修饰符号，真正起控制作用的是百分符。对于格式化控制字符串`"%Y-%m-%d %H:%M:%S`，其中每一个字母所代表的意思如下表所示，注意大小写的区别：

| 格式 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| %a   | 本地星期名称的简写（如星期四为Thu）                          |
| %A   | 本地星期名称的全称（如星期四为Thursday）                     |
| %b   | 本地月份名称的简写（如八月份为agu）                          |
| %B   | 本地月份名称的全称（如八月份为august）                       |
| %c   | 本地相应的日期和时间的字符串表示（如：15/08/27 10:20:06）    |
| %d   | 一个月中的第几天（01 - 31）                                  |
| %f   | 微秒（范围0.999999）                                         |
| %H   | 一天中的第几个小时（24小时制，00 - 23）                      |
| %I   | 第几个小时（12小时制，0 - 11）                               |
| %j   | 一年中的第几天（001 - 366）                                  |
| %m   | 月份（01 - 12）                                              |
| %M   | 分钟数（00 - 59）                                            |
| %p   | 本地am或者pm的标识符                                         |
| %S   | 秒（00 - 61）                                                |
| %U   | 一年中的星期数。（00 - 53星期天是一个星期的开始。）第一个星期天之    前的所有天数都放在第0周。 |
| %w   | 一个星期中的第几天（0 - 6，0是星期天）                       |
| %W   | 和%U基本相同，不同的是%W以星期一为一个星期的开始。           |
| %x   | 本地相应日期字符串（如15/08/01）                             |
| %X   | 本地相应时间字符串（如08:08:10）                             |
| %y   | 去掉世纪的年份（00 - 99）两个数字表示的年份                  |
| %Y   | 完整的年份（4个数字表示年份）                                |
| %z   | 与UTC时间的间隔（如果是本地时间，返回空字符串）              |
| %Z   | 时区的名字（如果是本地时间，返回空字符串）                   |
| %%   | ‘%’字符                                                      |

### time模块主要方法

**1. time.sleep(t)**

time模块最常用的方法之一，用来睡眠或者暂停程序t秒，t可以是浮点数或整数。

**2. time.time()**

返回当前系统时间戳。时间戳可以做算术运算。

**3. time.gmtime([secs])** 

将一个时间戳转换为UTC时区的结构化时间。可选参数secs的默认值为`time.time()`。

```bash
In [9]: time.time()
Out[9]: 1548915214.4344785

In [10]: time.gmtime()
Out[10]: time.struct_time(tm_year=2019, tm_mon=1, tm_mday=31, tm_hour=6, tm_min=13, tm_sec=43, tm_wday=3, tm_yday=31, tm_isdst=0)
```

**4. time.localtime([secs])** 

将一个时间戳转换为当前时区的结构化时间。如果secs参数未提供，则以当前时间为准，即`time.time()`。 

**5. time.ctime([secs])** 

把一个时间戳转化为本地时间的格式化字符串。默认使用`time.time()`作为参数。

```bash
In [11]: time.ctime()
Out[11]: 'Thu Jan 31 14:15:04 2019'
```

**6. time.asctime([t])** 

把一个结构化时间转换为`Sun Aug 23 14:31:59 2017`这种形式的格式化时间字符串。默认将`time.localtime()`作为参数。 

```bash
In [12]: time.asctime()
Out[12]: 'Thu Jan 31 14:15:55 2019'
```

**7. time.mktime(t)**

将一个结构化时间转化为时间戳。`time.mktime()`执行与`gmtime()`,`localtime()`相反的操作，它接收`struct_time`对象作为参数,返回用秒数表示时间的浮点数。如果输入的值不是一个合法的时间，将触发`OverflowError`或`ValueError`。 

```bash
In [14]: time.mktime(time.localtime())
Out[14]: 1548915459.0
```

**8. time.strftime(format [, t])** 

返回格式化字符串表示的当地时间。把一个`struct_time`（如`time.localtime()`和`time.gmtime()`的返回值）转化为格式化的时间字符串，显示的格式由参数`format`决定。如果未指定t，默认传入`time.localtime()`。如果元组中任何一个元素越界，就会抛出`ValueError`的异常。

```bash
In [15]: time.strftime("%Y-%m-%d %H:%M:%S")
Out[15]: '2019-01-31 14:19:15'
```

**9. time.strptime(string[,format])** 

将格式化时间字符串转化成结构化时间。该方法是`time.strftime()`方法的逆操作。`time.strptime()`方法根据指定的格式把一个时间字符串解析为时间元组。要注意的是，提供的字符串要和format参数的格式一一对应，如果string中日期间使用“-”分隔，format中也必须使用“-”分隔，时间中使用冒号“:”分隔，后面也必须使用冒号分隔，否则会报格式不匹配的错误。

```bash
In [16]: stime = time.strftime("%Y-%m-%d %H:%M:%S")

In [17]: stime
Out[17]: '2019-01-31 14:21:01'

In [18]: time.strptime(stime,"%Y-%m-%d %H:%M:%S")
Out[18]: time.struct_time(tm_year=2019, tm_mon=1, tm_mday=31, tm_hour=14, tm_min=21, tm_sec=1, tm_wday=3, tm_yday=31, tm_isdst=-1)
```

**10. time.clock()** 

返回执行当前程序的CPU时间。用来衡量不同程序的耗时。该方法在不同的系统上含义不同。在Unix系统上，它返回的是“进程时间”，用秒表示的浮点数（时间戳）。在Windows中，第一次调用，返回的是进程运行的实际时间，而第二次之后的调用是自第一次调用以后到现在的运行时间。  

### 时间格式之间的转换

![](http://static.zybuluo.com/feixuelove1009/puafhqjgxe3uc2e4jvf0f60h/image.png)

| FROM           | TO             | 方法              |
| -------------- | -------------- | ----------------- |
| 时间戳         | UTC结构化时间  | gmtime()          |
| 时间戳         | 本地结构化时间 | localtime()       |
| UTC结构化时间  | 时间戳         | calendar.timegm() |
| 本地结构化时间 | 时间戳         | mktime()          |
| 结构化时间     | 格式化字符串   | strftime()        |
| 格式化字符串   | 结构化时间     | strptime()        |

------

## datetime

datetime模块定义了以下几个类（注意：这些类的对象都是不可变的！）。

| 类名                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| `datetime.date`      | 日期类                                                       |
| `datetime.time`      | 时间类                                                       |
| `datetime.datetime`  | 日期与时间类                                                 |
| `datetime.timedelta` | 表示两个date、time、datetime实例之间的时间差                 |
| `datetime.tzinfo`    | 时区相关信息对象的抽象基类。                                 |
| `datetime.timezone`  | Python3.2中新增的功能，实现tzinfo抽象基类的类，表示与UTC的固定偏移量 |

使用datetime模块主要就是对其前四个类的操作。另外，datetime模块中还定义了两个常量：

**datetime.MINYEAR：**

datetime.date或datetime.datetime对象所允许的年份的最小值，该值为1。

**datetime.MAXYEAR：**

datetime.date或datetime.datetime对象所允许的年份的最大值，该值为9999。

### datetime.date类

定义：`class datetime.date(year, month, day)`

datetime模块下的日期类，只能处理年月日这种日期时间，不能处理时分秒。

在构造datetime.date对象的时候需要传递下面的参数：

| 参数名称 | 取值范围                    |
| -------- | --------------------------- |
| year     | [MINYEAR, MAXYEAR]          |
| month    | [1, 12]                     |
| day      | [1, 指定年份的月份中的天数] |

主要属性和方法：

| 类方法/属性名称                 | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| date.max                        | date对象所能表示的最大日期：9999-12-31                       |
| date.min                        | date对象所能表示的最小日期：00001-01-01                      |
| date.resoluation                | date对象表示的日期的最小单位：天                             |
| date.today()                    | 返回一个表示当前本地日期的date对象                           |
| date.fromtimestamp(timestamp)   | 根据给定的时间戳，返回一个date对象                           |
| d.year                          | 年                                                           |
| d.month                         | 月                                                           |
| d.day                           | 日                                                           |
| d.replace(year[, month[, day]]) | 生成并返回一个新的日期对象，原日期对象不变                   |
| d.timetuple()                   | 返回日期对应的time.struct_time对象                           |
| d.toordinal()                   | 返回日期是是自 0001-01-01 开始的第多少天                     |
| d.weekday()                     | 返回日期是星期几，[0, 6]，0表示星期一                        |
| d.isoweekday()                  | 返回日期是星期几，[1, 7], 1表示星期一                        |
| d.isocalendar()                 | 返回一个元组，格式为：(year, weekday, isoweekday)            |
| d.isoformat()                   | 返回‘YYYY-MM-DD’格式的日期字符串                             |
| d.strftime(format)              | 返回指定格式的日期字符串，与time模块的strftime(format, struct_time)功能相同 |

### datetime.time类

定义：`class datetime.time(hour, [minute[, second, [microsecond[, tzinfo]]]])`

datetime模块下的时间类，只能处理时分秒。

在构造datetime.time对象的时候需要传递下面的参数：

| 参数名称    | 取值范围                             |
| ----------- | ------------------------------------ |
| hour        | [0, 23]                              |
| minute      | [0, 59]                              |
| second      | [0, 59]                              |
| microsecond | [0, 1000000]                         |
| tzinfo      | tzinfo的子类对象，如timezone类的实例 |

主要属性和方法：

| 类方法/属性名称                                              | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| time.max                                                     | time类所能表示的最大时间：time(23, 59, 59, 999999)           |
| time.min                                                     | time类所能表示的最小时间：time(0, 0, 0, 0)                   |
| time.resolution                                              | 时间的最小单位，即两个不同时间的最小差值：1微秒              |
| t.hour                                                       | 时                                                           |
| t.minute                                                     | 分                                                           |
| t.second                                                     | 秒                                                           |
| t.microsecond                                                | 微秒                                                         |
| t.tzinfo                                                     | 返回传递给time构造方法的tzinfo对象，如果该参数未给出，则返回None |
| t.replace(hour[, minute[, second[, microsecond[, tzinfo]]]]) | 生成并返回一个新的时间对象，原时间对象不变                   |
| t.isoformat()                                                | 返回一个‘HH:MM:SS.%f’格式的时间字符串                        |
| t.strftime()                                                 | 返回指定格式的时间字符串，与time模块的strftime(format, struct_time)功能相同 |

### datetime.datetime类

定义：`class datetime.datetime(year, month, day, hour=0, minute=0, second=0, microsecond=0, tzinfo=None)`

datetime模块下的日期时间类，你可以理解为datetime.time和datetime.date的组合类。

在构造datetime.datetime对象的时候需要传递下面的参数：

| 参数名称    | 取值范围                             |
| ----------- | ------------------------------------ |
| year        | [MINYEAR, MAXYEAR]                   |
| month       | [1, 12]                              |
| day         | [1, 指定年份的月份中的天数]          |
| hour        | [0, 23]                              |
| minute      | [0, 59]                              |
| second      | [0, 59]                              |
| microsecond | [0, 1000000]                         |
| tzinfo      | tzinfo的子类对象，如timezone类的实例 |

主要属性和方法：

| 类方法/属性名称                         | 描述                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| datetime.today()                        | 返回一个表示当前本期日期时间的datetime对象                   |
| datetime.now([tz])                      | 返回指定时区日期时间的datetime对象，如果不指定tz参数则结果同上 |
| datetime.utcnow()                       | 返回当前utc日期时间的datetime对象                            |
| datetime.fromtimestamp(timestamp[, tz]) | 根据指定的时间戳创建一个datetime对象                         |
| datetime.utcfromtimestamp(timestamp)    | 根据指定的时间戳创建一个datetime对象                         |
| datetime.combine(date, time)            | 把指定的date和time对象整合成一个datetime对象                 |
| datetime.strptime(date_str, format)     | 将时间字符串转换为datetime对象                               |
| dt.year, dt.month, dt.day               | 年、月、日                                                   |
| dt.hour, dt.minute, dt.second           | 时、分、秒                                                   |
| dt.microsecond, dt.tzinfo               | 微秒、时区信息                                               |
| dt.date()                               | 获取datetime对象对应的date对象                               |
| dt.time()                               | 获取datetime对象对应的time对象， tzinfo 为None               |
| dt.timetz()                             | 获取datetime对象对应的time对象，tzinfo与datetime对象的tzinfo相同 |
| dt.replace()                            | 生成并返回一个新的datetime对象，如果所有参数都没有指定，则返回一个与原datetime对象相同的对象 |
| dt.timetuple()                          | 返回datetime对象对应的tuple（不包括tzinfo）                  |
| dt.utctimetuple()                       | 返回datetime对象对应的utc时间的tuple（不包括tzinfo）         |
| dt.timestamp()                          | 返回datetime对象对应的时间戳，Python 3.3才新增的             |
| dt.toordinal()                          | 同date对象                                                   |
| dt.weekday()                            | 同date对象                                                   |
| dt.isocalendar()                        | 同date对象                                                   |
| dt.isoformat([sep])                     | 返回一个‘%Y-%m-%d’字符串                                     |
| dt.ctime()                              | 等价于time模块的time.ctime(time.mktime(d.timetuple()))       |
| dt.strftime(format)                     | 返回指定格式的时间字符串                                     |

### datetime.timedelta类

定义：`class datetime.timedelta(days=0, seconds=0, microseconds=0, milliseconds=0, hours=0, weeks=0)`

`timedelta`对象表示两个不同时间之间的差值。可以对`datetime.date`, `datetime.time`和`datetime.datetime`对象做算术运算。

主要属性和方法：

| 类方法/属性名称      | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| timedelta.min        | timedelta(-999999999)                                        |
| timedelta.max        | timedelta(days=999999999, hours=23, minutes=59, seconds=59, microseconds=999999) |
| timedelta.resolution | timedelta(microseconds=1)                                    |
| td.days              | 天 [-999999999, 999999999]                                   |
| td.seconds           | 秒 [0, 86399]                                                |
| td.microseconds      | 微秒 [0, 999999]                                             |
| td.total_seconds()   | 时间差中包含的总秒数，等价于: td / timedelta(seconds=1)      |

```bash
In [35]: dt = datetime.datetime.now()

In [36]: dt + datetime.timedelta(3)
Out[36]: datetime.datetime(2019, 2, 3, 14, 58, 38, 425562)

In [37]: dt + datetime.timedelta(-3)
Out[37]: datetime.datetime(2019, 1, 28, 14, 58, 38, 425562)

In [38]: dt - datetime.timedelta(3)
Out[38]: datetime.datetime(2019, 1, 28, 14, 58, 38, 425562)
```

------

参考：

http://www.liujiangblog.com/course/python/69