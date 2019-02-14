不是所有的场景都需要使用logging模块，下面是Python官方推荐的使用方法：

| 任务场景                         | 最佳工具                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| 普通情况下，在控制台显示输出     | `print()`                                                    |
| 报告正常程序操作过程中发生的事件 | `logging.info()`(或者更详细的`logging.debug()`)              |
| 发出有关特定事件的警告           | `warnings.warn()`或者`logging.warning`()                     |
| 报告错误                         | 弹出异常                                                     |
| 在不引发异常的情况下报告错误     | `logging.error()`, `logging.exception()`或者`logging.critical()` |

logging模块定义了下表所示的日志级别，按事件严重程度由低到高排列（注意是全部大写！因为它们是常量。）：

| 级别     | 级别数值 | 使用时机                                           |
| -------- | -------- | -------------------------------------------------- |
| DEBUG    | 10       | 详细信息，常用于调试。                             |
| INFO     | 20       | 程序正常运行过程中产生的一些信息。                 |
| WARNING  | 30       | 警告用户，虽然程序还在正常工作，但有可能发生错误。 |
| ERROR    | 40       | 由于更严重的问题，程序已不能执行一些功能了。       |
| CRITICAL | 50       | 严重错误，程序已不能继续运行。                     |

默认级别是WARNING，表示只有WARING和比WARNING更严重的事件才会被记录到日志内，低级别的信息会被忽略。因此，默认情况下，DEBUG和INFO会被忽略，WARING、ERROR和CRITICAL会被记录。

有多种方法用来处理被跟踪的事件。最简单的方法就是把它们打印到终端控制台上。或者将它们写入一个磁盘文件内。

```python
import logging
logging.warning('Watch out!')  # 消息会被打印到控制台上
logging.info('I told you so')  # 这行不会被打印，因为级别低于默认级别
```

## 简单用法

保存日志文件

```python
import logging

# FileHandler
logging.basicConfig(filename='example.log',
                    format='%(asctime)s - %(name)s - %(levelname)-8s -%(module)s: %(message)s',
                    datefmt='%Y-%m-%d %H:%M:%S %p', level=10)

logging.debug('This is debug message')
logging.info('This is info message')
logging.warning('This is warning message')
```

`logging.basicConfig(**kwargs)`

| Format     | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| *filename* | Specifies that a FileHandler be created, using the specified filename, rather than a StreamHandler. |
| *filemode* | If *filename* is specified, open the file in this mode. Defaults to `'a'`. |
| *format*   | Use the specified format string for the handler.             |
| *datefmt*  | Use the specified date/time format, as accepted by `time.strftime()`. |
| *style*    | If *format* is specified, use this style for the format string. One of `'%'`, `'{'` or `'$'` for `printf-style`, `str.format()` or `string.Template` respectively. Defaults to `'%'`. |
| *level*    | Set the root logger level to the specified level.            |
| *stream*   | Use the specified stream to initialize the StreamHandler. Note that this argument is incompatible with *filename* - if both are present, a `ValueError` is raised. |
| *handlers* | If specified, this should be an iterable of already created handlers to add to the root logger. Any handlers which don’t already have a formatter set will be assigned the default formatter created in this function. Note that this argument is incompatible with *filename* or *stream* - if both are present, a `ValueError` is raised. |

```text
format: 指定输出的格式和内容，format可以输出很多有用信息，如上例所示:
    %(levelno)s: 打印日志级别的数值
    %(levelname)s: 打印日志级别名称
    %(pathname)s: 打印当前执行程序的路径，其实就是sys.argv[0]
    %(filename)s: 打印当前执行程序名
    %(funcName)s: 打印日志的当前函数
    %(lineno)d: 打印日志的当前行号
    %(asctime)s: 打印日志的时间
    %(thread)d: 打印线程ID
    %(threadName)s: 打印线程名称
    %(process)d: 打印进程ID
    %(message)s: 打印日志信息
```

***

## 高级用法

对logging进行更加灵活的控制：需要了解Logger,Handler,Formatter,Filter。

logging模块采用了模块化设计，主要包含四种组件：

**Loggers**：记录器，提供应用程序代码能直接使用的接口；

**Handlers**：处理器，将记录器产生的日志发送至目的地；

**Filters**：过滤器，提供更好的粒度控制，决定哪些日志会被输出；

**Formatters**：格式化器，设置日志内容的组成结构和消息字段。

### 日志流程图

日志事件信息在loggers和handlers中的逻辑流程如下图所示：

![](http://static.zybuluo.com/feixuelove1009/ko55xbrzpa5nk3bwgq25w4gu/image.png)

同时向屏幕和文件进行日志输出的流程：

![](http://static.zybuluo.com/feixuelove1009/g09y63fuzu9omngnn6k2rego/image.png)

```text
1. 创建logger实例并配置
2. 创建formatter对象
3. 创建你需要的handler对象并配置
4. 将handler加载到logger实例中
5. 写日志
```

### Loggers

logging模块的日志功能是基于Logger类实现的。logger对象有三重功能：

- 提供应用程序调用的接口
- 决定日志记录的级别
- 将日志内容传递到相关联的handlers中

通过下面的方法获取一个Logger类的实例（建议以模块名命名logger实例）。

```
logger = logging.getLogger(__name__)
```

Logger是一个树形层级结构，在使用debug()，info()，warn()，error()，critical()等方法之前必须先创建一个Logger的实例，即创建一个记录器。如果没有显式的进行创建，则默认创建一个`root logger`，并应用默认的日志级别(WARN)，默认的处理器Handler(StreamHandler，即将日志信息打印在标准输出上)，和默认的格式化器Formatter，就像我们在前面举的那些例子一样。

总结logger对象的用法，可以分成两类：配置和消息发送。

最常用的配置方法：

`Logger.setLevel()`：设置日志记录级别

`Logger.addHandler()`和`Logger.removeHandler()`：为logger对象添加或删除handler处理器对象。

`Logger.addFilter()`和`Logger.removeFilter()`：为为logger对象添加或删除filter过滤器对象。

配置好logger对象后，就可以使用下面的方法创建日志消息了：

`Logger.debug()`, `Logger.info()`, `Logger.warning()`, `Logger.error()`, `and Logger.critical()`：创建对应级别的日志，但不一定会被记录。

`Logger.exception()`：创建一个类似`Logger.error()`的日志消息。不同的是`Logger.exception()`保存有一个追踪栈。该方法只能在异常handler中调用。

`Logger.log()`：显式的创建一条日志，是前面几种方法的通用方法。

注意，`getLogger()`方法返回一个logger对象的引用，并以你提供的name参数命名，如果未提供名字，那么默认为‘root’。使用同样的name参数，多次调用`getLogger()`，将返回同样的logger对象。

###  Handlers

Handlers对象是日志信息的处理器、分发器。它们将日志分发到不同的目的地。比如有时候我们希望将所有的日志都记录在本地文件内，将error及其以上级别的日志发送到标准输出stdout，将critical级别的日志以邮件的方法发送给管理员。这就需要同时有三个独立的handler，分别负责一个方向的日志处理。

logging模块使用较多的handlers有两个，`StreamHandler`和`FileHandler`。

**StreamHandler**

标准输出stdout（如显示器）分发器。

创建方法: `sh = logging.StreamHandler(stream=None)`

**FileHandler**

将日志保存到磁盘文件的处理器。

创建方法: `fh = logging.FileHandler(filename, mode='a', encoding=None, delay=False)`

handlers对象有下面的方法：

`setLevel()`：和logger对象的一样，设置日志记录级别。那为什么要设置两层日志级别呢？logger对象的日志级别是全局性的，对所有handler都有效，相当于默认等级。而handlers的日志级别只对自己接收到的logger传来的日志有效，进行了更深一层的过滤。

`setFormatter()`：设置当前handler对象使用的消息格式。

`addFilter()` 和 `removeFilter()`：配置或删除一个filter过滤对象

logging模块内置了下面的handler处理器，从字面上你就能看出它们的大概用途：

- StreamHandler
- FileHandler
- BaseRotatingHandler
- RotatingFileHandler
- TimedRotatingFileHandler
- SocketHandler
- DatagramHandler
- SMTPHandler
- SysLogHandler
- NTEventLogHandler
- HTTPHandler
- WatchedFileHandler
- QueueHandler
- NullHandler 

### Formatters

Formatter对象用来最终设置日志信息的顺序、结构和内容。其构造方法为：

```
ft = logging.Formatter.__init__(fmt=None, datefmt=None, style=’%’)
```

如果不指定datefmt，那么它默认是`%Y-%m-%d %H:%M:%S`样式的。

style参数默认为百分符%，这表示前面的fmt参数应该是一个`%(<dictionary key>)s`格式的字符串，而可以使用的logging内置的keys，如下表所示：

| 属性        | 格式            | 描述                                                       |
| ----------- | --------------- | ---------------------------------------------------------- |
| asctime     | %(asctime)s     | 日志产生的时间，默认格式为`2003-07-08 16:49:45,896`        |
| created     | %(created)f     | time.time()生成的日志创建时间戳                            |
| filename    | %(filename)s    | 生成日志的程序名                                           |
| funcName    | %(funcName)s    | 调用日志的函数名                                           |
| levelname   | %(levelname)s   | 日志级别 ('DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL') |
| levelno     | %(levelno)s     | 日志级别对应的数值                                         |
| lineno      | %(lineno)d      | 日志所针对的代码行号（如果可用的话）                       |
| module      | %(module)s      | 生成日志的模块名                                           |
| msecs       | %(msecs)d       | 日志生成时间的毫秒部分                                     |
| message     | %(message)s     | 具体的日志信息                                             |
| name        | %(name)s        | 日志调用者                                                 |
| pathname    | %(pathname)s    | 生成日志的文件的完整路径                                   |
| process     | %(process)d     | 生成日志的进程ID（如果可用）                               |
| processName | %(processName)s | 进程名（如果可用）                                         |
| thread      | %(thread)d      | 生成日志的线程ID（如果可用）                               |
| threadName  | %(threadName)s  | 线程名（如果可用）                                         |

### Filter

Handlers和Loggers可以使用Filters来完成比日志级别更复杂的过滤。比如我们定义了`filter = logging.Filter('a.b.c')`，并将这个Filter添加到了一个Handler上，则使用该Handler的Logger中只有名字带`a.b.c`前缀的Logger才能输出其日志。

创建方法: `filter = logging.Filter(name='')`

例如：

```
filter = logging.Filter('mylogger.child1.child2')  
fh.addFilter(filter)
```

则只会输出下面格式的日志，注意其用户名：

```
2017-09-27 16:27:46,227 - mylogger.child1.child2 - DEBUG - logger1 debug message
2017-09-27 16:27:46,227 - mylogger.child1.child2 - DEBUG - logger1 debug message
2017-09-27 16:27:46,227 - mylogger.child1.child2 - DEBUG - logger1 debug message
2017-09-27 16:27:46,227 - mylogger.child1.child2 - DEBUG - logger1 debug message
```

### 示例

保存日志文件的同时，在屏幕打印日志信息。

```python
import logging

# FileHandler
logging.basicConfig(filename='example.log',
                    format='%(asctime)s - %(name)s - %(levelname)-8s -%(module)s: %(message)s',
                    datefmt='%Y-%m-%d %H:%M:%S %p', level=10)
# StreamHandler
console = logging.StreamHandler()
console.setLevel(logging.INFO)
formatter = logging.Formatter('%(name)s: %(levelname)-8s %(message)s')
console.setFormatter(formatter)
logging.getLogger().addHandler(console)

logging.debug('This is debug message')
logging.info('This is info message')
logging.warning('This is warning message')
```

***

参考：

http://www.liujiangblog.com/course/python/71