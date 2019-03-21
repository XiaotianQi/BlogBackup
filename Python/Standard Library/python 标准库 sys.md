sys模块主要是针对与Python解释器相关的变量和方法，不是主机操作系统。

| 属性及方法                       | 使用说明                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| sys.argv                         | 获取命令行参数列表，第一个元素是程序本身                     |
| sys.exit(n)                      | 退出Python程序，exit(0)表示正常退出。当参数非0时，会引发一个`SystemExit`异常，可以在程序中捕获该异常 |
| sys.version                      | 获取Python解释程器的版本信息                                 |
| sys.maxsize                      | 最大的Int值，64位平台是`2**63 - 1`                           |
| sys.path                         | 返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值       |
| sys.platform                     | 返回操作系统平台名称                                         |
| sys.stdin                        | 输入相关                                                     |
| sys.stdout                       | 输出相关                                                     |
| sys.stderr                       | 错误相关                                                     |
| sys.exc_info()                   | 返回异常信息三元元组                                         |
| sys.getdefaultencoding()         | 获取系统当前编码，默认为utf-8                                |
| sys.setdefaultencoding()         | 设置系统的默认编码                                           |
| sys.getfilesystemencoding()      | 获取文件系统使用编码方式，默认是utf-8                        |
| sys.modules                      | 以字典的形式返回所有当前Python环境中已经导入的模块           |
| sys.builtin_module_names         | 返回一个列表，包含所有已经编译到Python解释器里的模块的名字   |
| sys.copyright                    | 当前Python的版权信息                                         |
| sys.flags                        | 命令行标识状态信息列表。只读。                               |
| sys.getrefcount(object)          | 返回对象的引用数量                                           |
| sys.getrecursionlimit()          | 返回Python最大递归深度，默认1000                             |
| sys.getsizeof(object[, default]) | 返回对象的大小                                               |
| sys.getswitchinterval()          | 返回线程切换时间间隔，默认0.005秒                            |
| sys.setswitchinterval(interval)  | 设置线程切换的时间间隔，单位秒                               |
| sys.getwindowsversion()          | 返回当前windwos系统的版本信息                                |
| sys.hash_info                    | 返回Python默认的哈希方法的参数                               |
| sys.implementation               | 当前正在运行的Python解释器的具体实现，比如CPython            |
| sys.thread_info                  | 当前线程信息                                                 |

实例：带百分比的进度条

```python
import sys
import time


def bar(num, total=100):
    rate = num / total
    rate_num = int(rate * 100)
    r = '\r[{}{}]{}%%'.format("="*num, " "*(100-num), rate_num, )
    sys.stdout.write(r)
    sys.stdout.flush()


if __name__ == '__main__':
    for i in range(0, 101):
        time.sleep(0.1)
        bar(i)
```

***

参考：

http://www.liujiangblog.com/course/python/54