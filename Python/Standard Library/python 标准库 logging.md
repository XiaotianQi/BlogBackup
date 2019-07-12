不是所有的场景都需要使用logging模块，下面是Python官方推荐的使用方法：

| 任务场景                         | 最佳工具                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| 普通情况下，在控制台显示输出     | `print()`                                                    |
| 报告正常程序操作过程中发生的事件 | `logging.info()`(或者更详细的`logging.debug()`)              |
| 发出有关特定事件的警告           | `warnings.warn()`或者`logging.warning`()                     |
| 报告错误                         | 弹出异常                                                     |
| 在不引发异常的情况下报告错误     | `logging.error()`, `logging.exception()`或者`logging.critical()` |

python的logging模块（**logging是线程安全的**）给应用程序提供了标准的日志信息输出接口。logging不仅支持把日志输出到文件，还支持把日志输出到TCP/UDP服务器，EMAIL服务器，HTTP服务器，UNIX的syslog系统等。在logging中主要有四个概念：logger、handler、filter和formatter。

对logging进行更加灵活的控制：需要了解Logger,Handler,Formatter,Filter。

logging模块采用了模块化设计，主要包含四种组件：

1. **Loggers**：记录器，提供应用程序代码能直接使用的接口；
2. **Handlers**：处理器，将记录器产生的日志发送至目的地；
3. **Filters**：过滤器，提供更好的粒度控制，决定哪些日志会被输出；
4. **Formatters**：格式化器，设置日志内容的组成结构和消息字段。

***

logging模块定义了下表所示的日志级别，按事件严重程度由低到高排列（注意是全部大写！因为它们是常量。）：

| 级别     | 数值 | 使用时机                                           |
| -------- | ---- | -------------------------------------------------- |
| DEBUG    | 10   | 详细信息，常用于调试。                             |
| INFO     | 20   | 程序正常运行过程中产生的一些信息。                 |
| WARNING  | 30   | 警告用户，虽然程序还在正常工作，但有可能发生错误。 |
| ERROR    | 40   | 由于更严重的问题，程序已不能执行一些功能了。       |
| CRITICAL | 50   | 严重错误，程序已不能继续运行。                     |

默认级别是WARNING，表示只有WARING和比WARNING更严重的事件才会被记录到日志内，低级别的信息会被忽略。因此，默认情况下，DEBUG和INFO会被忽略，WARING、ERROR和CRITICAL会被记录。

***

## 日志流程

日志事件信息在loggers和handlers中的逻辑流程如下图所示：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/logging-2.png)

同时向屏幕和文件进行日志输出的流程：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/logging-1.png)

```text
1. 创建logger实例并配置
2. 创建formatter对象
3. 创建你需要的handler对象并配置
4. 将handler加载到logger实例中
5. 写日志
```

***

## 五种对象

### Loggers

logging模块的日志功能是基于Logger类实现的。logger对象有三重功能：

- 提供应用程序调用的接口
- 决定日志记录的级别或根据`Filter`对象，来决定记录哪些日志。  
- 将日志内容传递到相关联的handlers中

通过下面的方法获取一个Logger类的实例（建议以模块名命名logger实例）。

```text
logger = logging.getLogger(name=None)
```

Logger是一个树形层级结构，在使用debug()，info()，warn()，error()，critical()等方法之前**必须先**创建一个Logger的实例，即创建一个记录器。

* 如果提供了name参数，那么它就是这个`Logger`实例的名称。可以通过`Logger`实例的name属性，来查看`Logger`实例的名称。
* 如果没提供name参数，那么这个`Logger`实例的名称是root，即`root logger`并应用默认的日志级别(WARN)，默认的处理器Handler(StreamHandler，即将日志信息打印在标准输出上)，和默认的格式化器Formatter。

Logger实例的名称是使用句号（.）分隔的多级结构。在这种命名方式中，后面的logger是前面的logger的子（父子logger只是简单的通过命名来识别），比如：有一个名称为foo的logger，那么诸如foo.bar、foo.bar.baz和foo.bam这样的logger都是foo这个logger的子logger。子logger会自动继承父logger的定义和配置。

**NOTE**：使用同样的name参数，多次调用`getLogger()`，将返回同一个logger对象。这个规则不仅仅在同一个module有效,而且对在同一个Python解释器进程的多个module也有效。因此应用程序可以在一个module中定义一个父logger，然后在其他module中继承这个logger，而不必把所有的logger都配置一遍。

logger对象的用法，可以分成两类：configuration和message sending。

1. 常用configuration方法：
   * `Logger.setLevel(level)`：设置日志记录级别
   * `Logger.addHandler(hdlr)`和`Logger.removeHandler(hdlr)`：为logger对象添加或删除handler处理器对象。
   * `Logger.addFilter(filter)`和`Logger.removeFilter(filter)`：为为logger对象添加或删除filter过滤器对象。

2. message sending方法：

   * `Logger.debug(msg, *args, **kwargs)`, `Logger.info(msg, *args, **kwargs)`, `Logger.warning(msg, *args, **kwargs)`, `Logger.error(msg, *args, **kwargs)`, `and Logger.critical(msg, *args, **kwargs)`：创建对应级别的日志
   * `Logger.exception(msg, *args, **kwargs)`：创建一个类似`Logger.error()`的日志消息。不同的是`Logger.exception()`保存有一个追踪栈。该方法只能在异常handler中调用。
   * `Logger.log(lvl, msg, *args, **kwargs)`：创建整数级别的日志。

***

###  Handlers

Handlers对象是日志信息的处理器、分发器，负责把日志事件分发到具体的目的地。

logger对象可以使用`addHandler()`方法，添加零个或多个handler对象到它自身。比如：	有时候我们希望将所有的日志都记录在本地文件内，将error及其以上级别的日志发送到标准输出stdout，将critical级别的日志以邮件的方法发送给管理员。这就需要同时有三个独立的handler，分别负责一个方向的日志处理。

handler提供了下面的配置方法：   

-  `setLevel(level)`
  - handler对象的`setLevel()`方法，与logger对象的`setLevel()`方法一样，也是用于设置一个日志级别，如果日志的级别低于setLevel()方法设置的值，那么handler不会处理它。 
  - 那为什么要设置两层日志级别呢？logger对象的日志级别是全局性的，对所有handler都有效，相当于默认等级。而handlers的日志级别只对自己接收到的logger传来的日志有效，进行了更深一层的过滤。
- `setFormatter(fmt)`
  * 设置当前handler对象使用的消息格式
- `addFilter(filter)`、`removeFilter(filter)`
  * 配置或删除一个filter过滤对象

下面是logging模块内置的handler，由名称就能能看出它们的大概用途：

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

logging模块使用较多的handlers有两个，`StreamHandler`和`FileHandler`。

**StreamHandler**

标准输出stdout（如显示器）分发器。

创建方法: `sh = logging.StreamHandler(stream=None)`

**FileHandler**

将日志保存到磁盘文件的处理器。

创建方法: `fh = logging.FileHandler(filename, mode='a', encoding=None, delay=False)`

***

### Formatters

Formatter对象用来最终设置日志信息的顺序、结构和内容。其构造方法为：

```text
ft = logging.Formatter(fmt=None, datefmt=None, style=’%’)

fmt：用于日志信息的格式化字符串；
datefmt：用于日期的格式化字符串。可选，默认值是%Y-%m-%d %H:%M:%S。
style：默认为百分符%，表示前面的fmt参数应该是一个%(<dictionary key>)s格式的字符串，而可以使用的logging内置的keys
```

替换字符串和它们所代表的含义：

| 属性        | 格式            | 描述                                                         |
| ----------- | --------------- | ------------------------------------------------------------ |
| name        | %(name)s        | logger的名称                                                 |
| message     | %(message)s     | 具体的日志信息，`record.getMessage()`的返回结果              |
| levelname   | %(levelname)s   | 日志级别 ('DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL')   |
| levelno     | %(levelno)s     | 日志级别对应的数值                                           |
| pathname    | %(pathname)s    | 调用logging的源文件的全路径名                                |
| filename    | %(filename)s    | 调用logging的源文件名称                                      |
| module      | %(module)s      | 调用logging的模块名                                          |
| funcName    | %(funcName)s    | 调用logging的函数名                                          |
| created     | %(created)f     | LogRecord的创建时间，time.time()返回的时间戳                 |
| asctime     | %(asctime)s     | LogRecord的创建时间的文本表现形式，默认格式为`2003-07-08 16:49:45,896` |
| msecs       | %(msecs)d       | 创建时间的毫秒部分                                           |
| lineno      | %(lineno)d      | 日志所针对的代码行号（如果可用的话）                         |
| process     | %(process)d     | 生成日志的进程ID（如果可用）                                 |
| processName | %(processName)s | 进程名（如果可用）                                           |
| thread      | %(thread)d      | 生成日志的线程ID（如果可用）                                 |
| threadName  | %(threadName)s  | 线程名（如果可用）                                           |

***

### Filters

Handlers和Loggers可以使用Filters来完成比日志级别更复杂的过滤。比如我们定义了`filter = logging.Filter('a.b.c')`，并将这个Filter添加到了一个Handler上，则使用该Handler的Logger中只有名字带`a.b.c`前缀的Logger才能输出其日志。

创建方法: `filter = logging.Filter(name='')`

例如：

```
filter = logging.Filter('mylogger.child1.child2')  
fh.addFilter(filter)
```

则只会输出下面格式的日志，注意其用户名：

```bash
2017-09-27 16:27:46,227 - mylogger.child1.child2 - DEBUG - logger1 debug message
2017-09-27 16:27:46,227 - mylogger.child1.child2 - DEBUG - logger1 debug message
2017-09-27 16:27:46,227 - mylogger.child1.child2 - DEBUG - logger1 debug message
2017-09-27 16:27:46,227 - mylogger.child1.child2 - DEBUG - logger1 debug message
```

### LogRecord Objects

**LogRecord** instances are created **automatically** by the Logger every time something is logged, and can be created manually via makeLogRecord() (for example, from a pickled event received over the wire).

| Attribute name  | Format                                      | Description                                                  |
| --------------- | ------------------------------------------- | ------------------------------------------------------------ |
| args            | You shouldn’t need to format this yourself. | The tuple of arguments merged into `msg` to produce `message`, or a dict whose values are used for the merge (when there is only one argument, and it is a dictionary). |
| asctime         | `%(asctime)s`                               | Human-readable time when the `LogRecord` was created.  By default this is of the form ‘2003-07-08 16:49:45,896’ (the numbers after the comma are millisecond portion of the time). |
| created         | `%(created)f`                               | Time when the `LogRecord` was created (as returned by `time.time()`. |
| exc_info        | You shouldn’t need to format this yourself. | Exception tuple (à la `sys.exc_info`) or, if no exception has occurred, `None`. |
| filename        | `%(filename)s`                              | Filename portion of `pathname`.                              |
| funcName        | `%(funcName)s`                              | Name of function containing the logging call.                |
| levelname       | `%(levelname)s`                             | Text logging level for the message (`'DEBUG'`, `'INFO'`, `'WARNING'`, `'ERROR'`, `'CRITICAL'`). |
| levelno         | `%(levelno)s`                               | Numeric logging level for the message (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`). |
| lineno          | `%(lineno)d`                                | Source line number where the logging call was issued (if available). |
| message         | `%(message)s`                               | The logged message, computed as `msg % args`. This is set when `Formatter.format()` is invoked. |
| module          | `%(module)s`                                | Module (name portion of `filename`).                         |
| msecs           | `%(msecs)d`                                 | Millisecond portion of the time when the `LogRecord` was created. |
| msg             | You shouldn’t need to format this yourself. | The format string passed in the original logging call. Merged with `args` to produce `message`, or an arbitrary object (see Using arbitrary objects as messages). |
| name            | `%(name)s`                                  | Name of the logger used to log the call.                     |
| pathname        | `%(pathname)s`                              | Full pathname of the source file where the logging call was issued (if available). |
| process         | `%(process)d`                               | Process ID (if available).                                   |
| processName     | `%(processName)s`                           | Process name (if available).                                 |
| relativeCreated | `%(relativeCreated)d`                       | Time in milliseconds when the LogRecord was created, relative to the time the logging module was loaded. |
| stack_info      | You shouldn’t need to format this yourself. | Stack frame information (where available) from the bottom of the stack in the current thread, up to and including the stack frame of the logging call which resulted in the creation of this record. |
| thread          | `%(thread)d`                                | Thread ID (if available).                                    |
| threadName      | `%(threadName)s`                            | Thread name (if available).                                  |

***

## Module-Level Functions

`logging.getLogger(name=None)`：

返回logger对象。

`logging.basicConfig(**kwargs)`：

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

`Logger.debug(msg, *args, **kwargs)`, `Logger.info(msg, *args, **kwargs)`, `Logger.warning(msg, *args, **kwargs)`, `Logger.error(msg, *args, **kwargs)`, `and Logger.critical(msg, *args, **kwargs)`：

创建对应级别的日志

`Logger.exception(msg, *args, **kwargs)`：

创建一个类似`Logger.error()`的日志消息。不同的是`Logger.exception()`保存有一个追踪栈。该方法只能在异常handler中调用。

`Logger.log(lvl, msg, *args, **kwargs)`：

创建整数级别的日志。

***

## 示例

### 屏幕输出日志

```python
import logging
import sys 

logger = logging.getLogger(__name__)
filter = logging.Filter(__name__)
formatter = logging.Formatter("%(asctime)s|%(name)-12s|%(message)s", "%F %T")

stream_handler = logging.StreamHandler(sys.stdout)
stream_handler.addFilter(filter)
stream_handler.setLevel(logging.DEBUG)
stream_handler.setFormatter(formatter)

logger.setLevel(logging.DEBUG)
logger.addFilter(filter)
logger.addHandler(stream_handler)

if __name__ == "__main__":
    logger.info("info")
```

### 保存日志文件

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

### 保存日志文件的同时，在屏幕打印日志信息。

```python
import logging

# 创建一个logger
logger = logging.getLogger('mylogger')
logger.setLevel(logging.DEBUG)

# 创建一个handler，用于写入日志文件
fh = logging.FileHandler('test.log')
fh.setLevel(logging.DEBUG)

# 再创建一个handler，用于输出到控制台
sh = logging.StreamHandler()
sh.setLevel(logging.DEBUG)

# 定义handler的输出格式
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
fh.setFormatter(formatter)
sh.setFormatter(formatter)

# 给logger添加handler
logger.addHandler(fh)
logger.addHandler(sh)

# 记录一条日志
logger.info('foorbar')  
 
```

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

### 使用配置文件，配置logging

```python
import logging
import logging.config

logging.config.fileConfig("logging.conf")

if __name__ == "__main__":
    logger = logging.getLogger("test_logging.sublogger")
    logger.info("info")
```

`logging.conf`如下： 

```conf
[loggers]
keys = root,logger

[handlers]
keys = stream_handler

[formatters]
keys = formatter

[logger_root]
handlers = stream_handler

[logger_logger]
handlers = stream_handler
level = DEBUG
propagate = 1 
qualname = test_logging

[handler_stream_handler]
class = StreamHandler
args = (sys.stdout,)
formatter = formatter
level = DEBUG

[formatter_formatter]
format = %(asctime)s|%(name)-12s|%(message)s
datefmt = %F %T
```

***

参考：

http://www.liujiangblog.com/course/python/71

http://blog.timd.cn/python-logging/