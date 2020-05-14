## OS 模块

os模块是Python标准库中的一个用于访问操作系统相关功能的模块，os模块提供了一种可移植的使用操作系统功能的方法。使用os模块中提供的接口，可以实现跨平台访问。但是，并不是所有的os模块中的接口在全平台都通用，有些接口的实现是一来特定平台的，比如linux相关的文件权限管理和进程管理。

os模块的主要功能：系统相关、目录及文件操作、执行命令和管理进程。

在使用os模块的时候，如果出现了问题，会抛出`OSError`异常，表明无效的路径名或文件名，或者路径名(文件名)无法访问，或者当前操作系统不支持该操作。

```
os.pardir  获取当前目录的父目录字符串名：('..')
os.makedirs('dirname1/dirname2')    可生成多层递归目录
os.removedirs('dirname1')    若目录为空，则删除，并递归到上一级目录，如若也为空，则删除，依此类推
os.mkdir('dirname')    生成单级目录；相当于shell中mkdir dirname
os.rmdir('dirname')    删除单级空目录，若目录不为空则无法删除，报错；相当于shell中rmdir dirname
os.listdir('dirname')    列出指定目录下的所有文件和子目录，包括隐藏文件，并以列表方式打印
os.remove()  删除一个文件
os.rename("oldname","newname")  重命名文件/目录
os.stat('path/filename')  获取文件/目录信息
os.sep    输出操作系统特定的路径分隔符，win下为"\\",Linux下为"/"
os.linesep    输出当前平台使用的行终止符，win下为"\t\n",Linux下为"\n"
os.pathsep    输出用于分割文件路径的字符串
os.name    输出字符串指示当前使用平台。win->'nt'; Linux->'posix'
os.system("bash command")  运行shell命令，直接显示
os.environ  获取系统环境变量
os.path.abspath(path)  返回path规范化的绝对路径
os.path.split(path)  将path分割成目录和文件名二元组返回
os.path.dirname(path)  返回path的目录。其实就是os.path.split(path)的第一个元素
os.path.basename(path)  返回path最后的文件名。如何path以／或\结尾，那么就会返回空值。即os.path.split(path)的第二个元素
os.path.exists(path)  如果path存在，返回True；如果path不存在，返回False
os.path.isabs(path)  如果path是绝对路径，返回True
os.path.isfile(path)  如果path是一个存在的文件，返回True。否则返回False
os.path.isdir(path)  如果path是一个存在的目录，则返回True。否则返回False
os.path.join(path1[, path2[, ...]])  将多个路径组合后返回，第一个绝对路径之前的参数将被忽略
os.path.getatime(path)  返回path所指向的文件或者目录的最后存取时间
os.path.getmtime(path)  返回path所指向的文件或者目录的最后修改时间
```

### 系统相关

os模块提供了一些操作系统相关的变量，可以在跨平台的时候提供支持，便于编写移植性高，可用性好的代码。所以在涉及操作系统相关的操作时，请尽量使用本模块提供的方法，而不要使用当前平台特定的用法或格式，否则一旦移植到其他平台，可能会造成难以解决的困扰。

| 方法和变量 | 用途                                                         |
| ---------- | ------------------------------------------------------------ |
| os.name    | 查看当前操作系统的名称。windows平台下返回‘nt’，Linux则返回‘posix’。 |
| os.environ | 获取系统环境变量                                             |
| os.sep     | 当前平台的路径分隔符。在windows下，为‘\’，在POSIX系统中，为‘/’。 |
| os.altsep  | 可替代的路径分隔符，在Windows中为‘/’。                       |
| os.extsep  | 文件名和文件扩展名之间分隔的符号，在Windows下为‘.’。         |
| os.pathsep | PATH环境变量中的分隔符，在POSIX系统中为‘:’，在Windows中为‘;’。 |
| os.linesep | 行结束符。在不同的系统中行尾的结束符是不同的，例如在Windows下为‘\r\n’。 |
| os.devnull | 在不同的系统上null设备的路径，在Windows下为‘nul’，在POSIX下为‘/dev/null’。 |
| os.defpath | 当使用exec函数族的时候，如果没有指定PATH环境变量，则默认会查找os.defpath中的值作为子进程PATH的值。 |

### 文件和目录操作

os模块中包含了一系列文件操作相关的函数，其中有一部分是Linux平台专用方法。Linux是用C写的，底层的`libc`库和系统调用的接口都是`C API`，Python的os模块中包括了对这些接口的Python实现，通过Python的os模块，可以调用Linux系统的一些底层功能，进行系统编程。

各平台通用的方法：

| 方法和变量                          | 用途                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| os.getcwd()                         | 获取当前工作目录，即当前python脚本工作的目录路径             |
| os.chdir("dirname")                 | 改变当前脚本工作目录；相当于shell下cd                        |
| os.curdir                           | 返回当前目录: ('.')                                          |
| os.pardir                           | 获取当前目录的父目录字符串名：('..')                         |
| os.makedirs('dir1/dir2')            | 可生成多层递归目录                                           |
| os.removedirs(‘dirname1’)           | 递归删除空目录（要小心）                                     |
| os.mkdir('dirname')                 | 生成单级目录                                                 |
| os.rmdir('dirname')                 | 删除单级空目录，若目录不为空则无法删除并报错                 |
| os.listdir('dirname')               | 列出指定目录下的所有文件和子目录，包括隐藏文件               |
| os.remove('filename')               | 删除一个文件                                                 |
| os.rename("oldname","new")          | 重命名文件/目录                                              |
| os.stat('path/filename')            | 获取文件/目录信息                                            |
| os.path.abspath(path)               | 返回path规范化的绝对路径                                     |
| os.path.split(path)                 | 将path分割成目录和文件名二元组返回                           |
| os.path.dirname(path)               | 返回path的目录。其实就是`os.path.split(path)`的第一个元素    |
| os.path.basename(path)              | 返回path最后的文件名。如果path以`／`或`\`结尾，那么就会返回空值。 |
| os.path.exists(path或者file)        | 如果path存在，返回True；如果path不存在，返回False            |
| os.path.isabs(path)                 | 如果path是绝对路径，返回True                                 |
| os.path.isfile(path)                | 如果path是一个存在的文件，返回True。否则返回False            |
| os.path.isdir(path)                 | 如果path是一个存在的目录，则返回True。否则返回False          |
| os.path.join(path1[, path2[, ...]]) | 将多个路径组合后返回，第一个绝对路径之前的参数将被忽略       |
| os.path.getatime(path)              | 返回path所指向的文件或者目录的最后存取时间                   |
| os.path.getmtime(path)              | 返回path所指向的文件或者目录的最后修改时间                   |
| os.path.getsize(filename)           | 返回文件包含的字符数量                                       |

在Python中，使用windows的文件路径时一定要小心，比如你要引用d盘下的1.txt文件，那么路径要以字符串的形式写成`'d:\\1.txt'`或者`r'd:\1.txt`。前面的方式是使用windwos的双斜杠作为路径分隔符，后者是使用`原生字符串`的形式，以r开始的字符串都被认为是原始字符串，表示字符串里所有的特殊符号都以本色出演，不进行转义，此时可以使用普通windows下的路径表示方式。这两种方法使用哪种都可以，但不可混用。

```text
os.walk(top, topdown=True, onerror=None, followlinks=False)
```

walk方法是os模块中非常重要和强大的一个方法。可以帮助我们非常便捷地以递归方式自顶向下或者自底向上的方式遍历目录树，对每一个目录都返回一个三元元组`(dirpath, dirnames, filenames)`。

三元元组`(dirpath，dirnames，filenames)`：

* `dirpath` - 遍历所在目录树的位置，是一个字符串对象

* `dirnames` - 目录树中的子目录组成的列表，不包括("."和"..")

* `filenames` - 目录树中的文件组成的列表

`topdown = True`或者没有指定，则采用自顶向下的方式进行目录遍历，也就是从父目录向子目录逐步深入遍历

`topdown = False`，则采用自底向上的方式遍历目录，也就是先打印子目录再打印父目录的方式。

如果可选参数`onerror`被指定，则`onerror`必须是一个函数，该函数有一个`OSError`实例的参数，这样可以允许在运行的时候即使出现错误的时候不会打断`os.walk()`的执行，或者抛出一个异常并终止`os.walk()`的运行。通俗的讲，就是定义这个参数用于指定当发生了错误时的处理方法。

默认情况下，os.walk()遍历的时候不会进入符号链接，如果设置了可选参数`followlinks = True`，则会进入符号链接。注意，这可能会出现遍历死循环，因为符号链接可能会出现自己链接自己的情况。

```python
import os

try:
    for root, dirs, files in os.walk(r"C:\GitHub\BlogBackup"):
        print("directory {}".format(root))
        if '.git' in dirs:
            dirs.remove('.git')
        for directory in dirs:
            print("<DIR>\t{}".format(directory))
        for file in files:
            print("\t{}".format(file))
except OSError as ex:
    print(ex)
```

***

**绝对路径 & 相对路径**

`\`：绝对路径

`/`：相对路径

```python
os.path.dirname(__file__)					# c:/test/path test
os.path.abspath(os.path.dirname(__file__))	# c:\test\path test
```

```text
''     ：当前目录
'./'   ：当前目录
'../'  ：上级目录
'/'    ：根目录
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/path.png)

```python
with open('f1.txt', 'w') as f:
    f.write('test text')

with open('./f2.txt', 'w') as f:
    f.write('test text')

with open('../f3.txt', 'w') as f:
    f.write('test text')

with open('../F2/f4.txt', 'w') as f:
    f.write('test text')
```

当前工作路径在F3文件夹中：

```text
F1\
	f3.txt
	F2\
		f4.txt
	F3\
		f1.txt
		f2.txt
		test.py
```

***

### 执行命令

在早期的Python版本中，通常使用os模块的system或者popen等方法执行操作系统的命令。但是，最近Python官方逐渐弃用了这些命令，而是改用内置的subprocess模块执行操作系统相关命令。

`os.system(command)`

运行操作系统命令，直接显示结果。但返回值是0或-1，不能获得显示在屏幕上的数据。 command是要执行的命令字符串。

***

## shutil 模块

shutil模块是对os模块的补充，主要针对文件的拷贝、删除、移动、压缩和解压操作。

` shutil.copyfileobj(fsrc, fdst[, length=16\*1024])`

copy文件内容到另一个文件，可以copy指定大小的内容。这个方法是shutil模块中其它拷贝方法的基础，其它方法在本质上都是调用这个方法。

源码：

```
def copyfileobj(fsrc, fdst, length=16*1024):

    while 1:
        buf = fsrc.read(length)
        if not buf:
            break
        fdst.write(buf)
```

注意，其中的fsrc，fdst都是使用open()方法打开后的文件对象。

`shutil.copyfile(src, dst)`

拷贝整个文件。

该方法的核心在最后几行，我们可以很清楚地看到`copyfile()`方法对`copyfileobj()`进行了调用。

```
def copyfile(src, dst, *, follow_symlinks=True):
    if _samefile(src, dst):
        raise SameFileError("{!r} and {!r} are the same file".format(src, dst))

    for fn in [src, dst]:
        try:
            st = os.stat(fn)
        except OSError:      
            pass
        else:
            if stat.S_ISFIFO(st.st_mode):
                raise SpecialFileError("`%s` is a named pipe" % fn)

    if not follow_symlinks and os.path.islink(src):
        os.symlink(os.readlink(src), dst)
    else:
        with open(src, 'rb') as fsrc:
            with open(dst, 'wb') as fdst:
                copyfileobj(fsrc, fdst)
    return dst
```

`shutil.copymode(src, dst)`

仅拷贝权限。内容、组、用户均不变。

`shutil.copystat(src, dst)`

仅复制所有的状态信息，包括权限，组，用户，时间等。

`shutil.copy(src,dst)` 

同时复制文件的内容以及权限，也就是先copyfile()然后copymode()。

`shutil.copy2(src, dst)`

同时复制文件的内容以及文件的所有状态信息。先copyfile()后copystat()。

`shutil.ignore_patterns(*patterns)`

忽略指定的文件。通常配合下面的copytree()方法使用。

`shutil.copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2,ignore_dangling_symlinks=False)`   

递归地复制目录及其子目录的文件和状态信息

- symlinks：指定是否复制软链接。小心陷入死循环。
- ignore：指定不参与复制的文件，其值应该是一个ignore_patterns()方法。
- copy_function：指定复制的模式

```
# 典型用法
from shutil import copytree, ignore_patterns

copytree(source, destination, ignore=ignore_patterns('*.pyc', 'tmp*'))

copytree('folder1', 'folder2', ignore=ignore_patterns('*.pyc', 'tmp*'))

copytree('f1', 'f2', symlinks=True, ignore=ignore_patterns('*.pyc', 'tmp*'))
```

`shutil.rmtree(path, ignore_errors=False, onerror=None)`

递归地删除目录及子目录内的文件。

注意！该方法不会询问yes或no，被删除的文件也不会出现在回收站里，请务必小心！

下面的例子在碰到只读文件时，尝试清除只读属性，然后再删除。

```
import os, stat
import shutil

def remove_readonly(func, path, _):
    "去除文件的只读属性，尝试再次删除"
    os.chmod(path, stat.S_IWRITE)
    func(path)

shutil.rmtree(directory, onerror=remove_readonly)
```

`shutil.move(src, dst, copy_function=copy2)`

递归地移动文件，类似mv命令。

`shutil.which(cmd)`

类似linux的`which`命令，返回执行该命令的程序路径。Python3.3新增

`12. shutil.make_archive(base_name, format[, root_dir[, base_dir[, verbose[, dry_run[, owner[, group[, logger]]]]]]])`

创建归档或压缩文件。

- `base_name`：压缩后的文件名。如果不指定绝对路径，则压缩文件保存在当前目录下。这个参数必须指定。
- `format`：压缩格式，可以是“zip”, “tar”, “bztar” ，“gztar”，“xztar”中的一种。这个参数也必须指定。
- `root_dir`：设置压缩包里的根目录，一般使用默认值，不特别指定。
- `base_dir`：要进行压缩的源文件或目录。
- `owner`：用户，默认当前用户。
- `group`：组，默认当前组。
- `logger`：用于记录日志，通常是`logging.Logger`对象。

范例：

```
import shutil
shutil.make_archive("d:\\3", "zip",  base_dir="d:\\1.txt")
```

`13. shutil.unpack_archive(filename[, extract_dir[, format]])`

解压缩或解包源文件。

- filename是压缩文档的完整路径
- extract_dir是解压缩路径，默认为当前目录。
- format是压缩格式。默认使用文件后缀名代码的压缩格式。

范例：

```
import shutil
shutil.unpack_archive("d:\\3.zip", "f:\\3", 'zip')
```

shutil模块的压缩和解压功能，在后台是通过调用zipfile和tarfile两个模块来进行的。下面我们会立刻介绍这两个模块。

***

参考：

[os](http://www.liujiangblog.com/course/python/53)，刘江

[Python绝对路径和相对路径详解](http://c.biancheng.net/view/5693.html)，C语言中文网