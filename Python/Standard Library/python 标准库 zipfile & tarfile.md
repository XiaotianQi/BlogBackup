## ZipFile 模块

ZIP是通用的归档和压缩格式。zipfile模块提供了通用的创建、读取、写入、附加和显示压缩文件的方法，可以简单地把它理解为Python中的zip解压缩软件。该模块可以解密带有密码的压缩文件，但不提供附加密码的压缩功能。此模块使用原声python实现，所以效率较差。

zipfile里有两个非常常用的类, 分别是ZipFile和ZipInfo, 在绝大多数的情况下，只需要使用这两个class就可以了。ZipFile是主要的类，用来创建和读取zip文件而ZipInfo是存储的zip文件的每个文件的信息的。

zipfile模块其实很简单，记住下面几个重要的方法就可以了。

| 方法                  | 用途                        |
| --------------------- | --------------------------- |
| z = zipfile.ZipFile() | 打开或者新建一个zip文件对象 |
| z.write()             | 添加文件到压缩包内          |
| z.infolist()          | 查看压缩包内的文件信息      |
| z.extract()           | 解压单个文件                |
| z.extractall()        | 解压所有文件                |
| z.close()             | 关闭压缩文件                |

zipfile里有两个非常重要的class, 分别是ZipFile和ZipInfo, 在绝大多数的情况下，只需要使用这两个class就可以。

* ZipFile是主要的类，用来创建和读取zip文件；

* ZipInfo是存储的zip文件的每个文件的信息的。

### ZipFile 对象

`class zipfile.ZipFile(file, mode='r', compression=ZIP_STORED, allowZip64=True, compresslevel=None)`

创建一个ZipFile对象，返回的也是一个类似文件的ZipFile对象，可以读写。

file可以是一个文件地址字符串、文件类对象或地址类对象。

mode参数为r时，表示读取一个已经存在的文件；为w的时候表示覆盖或写入一个新文件；为a时表示在已有文件后追加；为x时表示新建文件并写入。x模式下，如果文件名已经存在，则抛出`FileExistsError`异常。这些特点和open()方法打开文件一样样的。

compression 指明压缩格式，支持`ZIP_STORED`, `ZIP_DEFLATED`, `ZIP_BZIP2`和`ZIP_LZMA`。使用不支持的格式会抛出`NotImplementedError`异常。默认是`ZIP_STORED`格式。如果指定为`ZIP_DEFLATED`, `ZIP_BZIP2`或者`ZIP_LZMA`格式，但对应的支持模块`zlib`, `bz2`或者`lzma`不可用，则抛出`RuntimeError`异常。当文件大小超过4GB时，将使用`ZIP64`扩展（默认启用）。

在`w/x/a`模式下，如果没有写入任何数据就`close`了，则会生成空的ZIP文件。

ZipFile也是一种上下文管理器（3.2版本新特性），同样支持with语句，如下例子所示：

```
with ZipFile('spam.zip', 'w') as myzip:
    myzip.write('eggs.txt')
```

#### 属性和方法

`ZipFile.write(filename, arcname=None, compress_type=None)`

往ZIP文件里添加新文件，单个文件可以重复添加，但是会弹出警告。如果指定`arcname`参数，则在ZIP文件内部将原来的filename改成arcname。

`ZipFile.open(name, mode='r', pwd=None, \*, force_zip64=False)`

访问档案中的指定文件。`pwd`是解压密码。该方法也是个上下文管理器，支持with语法。

```python
with ZipFile('spam.zip') as myzip:
    with myzip.open('eggs.txt') as myfile:
        print(myfile.read())
```

`ZipFile.extract(member, path=None, pwd=None)`

解压单个文件。核心方法之一。将ZIP文件中的某个成员解压到当前目录。member必须是完整名，path是指定的解压目录。解压的过程不会破坏原压缩文件。

`ZipFile.extractall(path=None, members=None, pwd=None)`

批量解压文件。默认是全部解压。

path：解压缩目录
member：需要解压缩的文件名儿列表 
password：当zip文件有密码时需要该选项 

`ZipFile.close()`

确保压缩文件被正确关闭。

`ZipFile.setpassword(pwd)`

设置通用的解压密码，用于解压加密压缩文件。

`ZipFile.read(name, pwd=None)`

从已打开的ZIP文件成员中读取数据。

`ZipFile.testzip()`

校验ZIP文件完整性。

`ZipFile.writestr(zinfo_or_arcname, data[, compress_type])`

往ZIP文件里添加字符串类型数据

`zipfile.is_zipfile(filename)`

如果文件是个ZIP文件则返回True，否则False。

`ZipFile.infolist()`

返回一个包含ZIP文件内所有成员信息的ZipInfo对象。

`ZipFile.namelist()`

返回ZIP文件内所有成员名字列表。

`ZipFile.printdir()`

在stdout上打印ZIP文件的目录表。

`ZipFile.filename`

ZIP文件的名字

`ZipFile.debug`

调试输出的级别。从0到3。

`ZipFile.comment`

ZIP文件的注释内容。

#### 压缩多个文件

以压缩txt文件为例

```python
import os, zipfile

startdir = r'C:\Desktop\test\testdir'
file_news = startdir +'.zip'
z = zipfile.ZipFile(file_news, 'w', zipfile.ZIP_DEFLATED)
for dirpath, dirnames, filenames in os.walk(startdir):
    for filename in filenames:
        if filename.endswith('.txt'):
            print('start:', filename)
            z.write(os.path.join(dirpath, filename), filename)
            print ('well done')
z.close()
```

#### 压缩文件夹

```python
import zipfile, os

startdir = r'C:\Desktop\test\testdir'
file_news = startdir +'.zip'
z = zipfile.ZipFile(file_news, 'w', zipfile.ZIP_DEFLATED)
for dirpath, dirnames, filenames in os.walk(startdir):
    for filename in filenames:
        print('start:', filename)
        z.write(os.path.join(dirpath, filename), filename)
        print ('well done')
z.close()
```

#### 提取所有文件

```python
z = zipfile.ZipFile(r'C:\Desktop\test\testdir.zip')
z.extractall(r'C:\Desktop\test\test123')
z.close()
```

### ZipInfo 对象

zipfile模块允许我们查询存档的内容，而不必提取它。

`ZipFile.getinfo(name)` 方法返回的是一个ZipInfo对象，表示zip文档中相应文件的信息。

#### 属性和方法

```text
ZipInfo.filename： 获取文件名称。

ZipInfo.date_time： 获取文件最后修改时间。返回一个包含6个元素的元组：(年, 月, 日, 时, 分, 秒)

ZipInfo.compress_type： 压缩类型。

ZipInfo.comment： 文档说明。

ZipInfo.extr： 扩展项数据。

ZipInfo.create_system： 获取创建该zip文档的系统。

ZipInfo.create_version： 获取 创建zip文档的PKZIP版本。

ZipInfo.extract_version： 获取 解压zip文档所需的PKZIP版本。

ZipInfo.reserved： 预留字段，当前实现总是返回0。

ZipInfo.flag_bits： zip标志位。

ZipInfo.volume： 文件头的卷标。

ZipInfo.internal_attr： 内部属性。

ZipInfo.external_attr： 外部属性。

ZipInfo.header_offset： 文件头偏移位。

ZipInfo.CRC： 未压缩文件的CRC-32。

ZipInfo.compress_size： 获取压缩后的大小。

ZipInfo.file_size： 获取未压缩的文件大小。
```

#### 提取信息

使用ZipFile对象的namelist（）方法将按名称返回归档的所有成员的列表。要获取档案中特定文件的信息，可以使用ZipFile对象的getinfo（）方法。这将允许您访问特定文件的信息，例如文件的压缩前后的大小或其上次修改时间。

当有很多文件需要处理时，对所有文件逐个调用getinfo（）方法可能是一个令人讨厌的过程。在这种情况下，可以使用infolist（）方法返回包含归档中每个成员的ZipInfo对象的列表。列表中这些对象的顺序与实际zip文件的顺序相同。

您还可以使用read（file）方法从归档中直接读取特定文件的内容，其中file是要读取的文件的名称。为此，必须以读取或追加模式打开归档。

要从归档中获取单个文件的压缩大小，可以使用compress_size属性。同样，要知道未压缩的大小，可以使用file_size属性。

```python
z = zipfile.ZipFile(r'C:\Users\bnwse\Desktop\test\testdir.zip')
for file in z.namelist():
    print(file, z.getinfo(file).date_time)
z.close()
```

```python
with zipfile.ZipFile(r'C:\Users\bnwse\Desktop\test\testdir.zip') as z:
    for file in z.namelist():
        zipInfo = z.getinfo(file)
        print('-------------------------------')
        print ('filename\t:', zipInfo.filename) #获取文件名称
        print ('date_time\t:', zipInfo.date_time) #获取文件最后修改时间。返回一个包含6个元素的元组：(年, 月, 日, 时, 分, 秒)
        print ('compress_type\t:', zipInfo.compress_type) #压缩类型
        print ('comment\t\t:', zipInfo.comment) #文档说明
        print ('extra\t\t:', zipInfo.extra) #扩展项数据
        print ('create_system\t:', zipInfo.create_system) #获取创建该zip文档的系统。
        print ('create_version\t:', zipInfo.create_version) #获取 创建zip文档的PKZIP版本。
        print ('extract_version\t:', zipInfo.extract_version) #获取 解压zip文档所需的PKZIP版本。
        print ('extract_version\t:', zipInfo.reserved) # 预留字段，当前实现总是返回0。
        print ('flag_bits\t:', zipInfo.flag_bits) #zip标志位。
        print ('volume\t\t:', zipInfo.volume) # 文件头的卷标。
        print ('internal_attr\t:', zipInfo.internal_attr) #内部属性。
        print ('external_attr\t:', zipInfo.external_attr) #外部属性。
        print ('header_offset\t:', zipInfo.header_offset) # 文件头偏移位。
        print ('CRC\t\t:', zipInfo.CRC) # 未压缩文件的CRC-32。
        print ('compress_size\t:', zipInfo.compress_size) #获取压缩后的大小。
        print ('file_size\t:', zipInfo.file_size) #获取未压缩的文件大小。
```

***

## TarFile 模块

既然有压缩模块zipfile，那有一个归档模块tarfile也是很自然的。tarfile模块用于解包和打包文件，包括被`gzip`，`bz2`或`lzma`压缩后的打包文件。如果是`.zip`类型的文件，建议使用zipfile模块，更高级的功能请使用shutil模块。

tarfile模块看似复杂，其实也很简单，只需要掌握下面几个重点方法就可以了：

| 方法               | 说明                                               |
| ------------------ | -------------------------------------------------- |
| t = tarfile.open() | 打开或新建一个归档文件，返回一个TarFile类型的对象t |
| t.getmembers()     | 获取包内所有成员的信息                             |
| t.add()            | 将指定文件加入包内                                 |
| t.extract()        | 解包指定文件                                       |
| t.extractall()     | 解包所有文件                                       |
| TarFile.close()    | 关闭TarFile文件                                    |

感觉和zipfile模块的使用是不是很像？需要注意的是两者在打开文件、获取文件信息和添加文件的命令名称不一样。

`tarfile.open(name=None, mode='r', fileobj=None, bufsize=10240, **kwargs)`

返回一个TarFile类型的对象。本质上就是打开一个文件对象。

`name`是文件名或路径。

`bufsize`用于指定数据块的大小，默认为20*512字节。

`mode`是打开模式，一个类似`filemode[:compression]`格式的字符串，可以有下表所示的组合，默认为“r”。

| 模式       | 说明                                                       |
| ---------- | ---------------------------------------------------------- |
| 'r'or'r:*' | 自动解压并打开文件（推荐模式）                             |
| 'r:'       | 只打开文件不解压                                           |
| 'r:gz'     | 采用gzip格式解压并打开文件                                 |
| 'r:bz2'    | 采用bz2格式解压并打开文件                                  |
| 'r:xz'     | 采用lzma格式解压并打开文件                                 |
| 'x'or'x:'  | 仅创建打包文件，不压缩                                     |
| 'x:gz'     | 采用gzip方式压缩并打包文件                                 |
| 'x:bz2'    | 采用bzip2方式压缩并打包文件                                |
| 'x:xz'     | 采用lzma方式压缩并打包文件                                 |
| 'a'or'a:'  | 打开文件，并以不压缩的方式追加内容。如果文件不存在，则新建 |
| 'w'or'w:'  | 以不压缩的方式写入                                         |
| 'w:gz'     | 以gzip的方式压缩并写入                                     |
| 'w:bz2'    | 以bzip2的方式压缩并写入                                    |
| 'w:xz'     | 以lzma的方式压缩并写入                                     |
| `注意`     | 不支持'a:gz', 'a:bz2'和'a:xz'的模式                        |

### TarFile对象

TarFile 对象提供到tar存档的接口。打包文件本质上是数据块的序列。包中的每个文件成员都是由头部块和数据块组成的。在包中，文件有可能重复。包里的每个文件都是一个TarInfo对象。所以遍历一个TarFile对象，就是遍历一个TarInfo对象的集合。

TarFile对象同样可以使用with语句进行上下文管理。

```python
class tarfile.TarFile(name=None, mode='r', fileobj=None, format=DEFAULT_FORMAT, tarinfo=TarInfo, dereference=False, ignore_zeros=False, encoding=ENCODING, errors='surrogateescape', pax_headers=None, debug=0, errorlevel=0)

name：指定归档文件路径名称；如果fileobj参数被指定该参数可以被忽略，且如果fileobj的name属性存在则取该属性的值；

mode：指定文档打开模式；r：读取已存在的归档，a：向一个已存在的文件追加数据，w：创建一个新的文件覆盖已经存在的文件，x：如果文件不存在才创建一个新文件

fileobj：指定要读写的文件对象；如果指定了该参数，那么mode参数的值会被fileojb的mode属性值覆盖，且name参数可以被忽略；
```



#### 属性和方法

`TarFile.getmember(name)`

获取某个成员的信息，返回TarInfo对象

`TarFile.getmembers()`

获取包内所有成员的信息，返回TarInfo对象

`TarFile.getnames()`

获取包内所有成员的名字

`gettarinfo(name=None, arcname=None, fileobj=None) `

创建一个TarInfo对象

`TarFile.list(verbose=True, \*, members=None)`

列表显示包内成员信息

`TarFile.next()`

显示下一个文件的信息

`TarFile.extractall(path=".", members=None, \*, numeric_owner=False)`

解包所有文件到当前目录，或path指定的目录。警告：解包文件之前一定要确保安全，防止覆盖本地或上级目录等恶意行为。

`TarFile.extract(member, path="", set_attrs=True, \*, numeric_owner=False)`

解包指定文件

`TarFile.extractfile(member)`

同上

`TarFile.add(name, arcname=None, recursive=True, exclude=None, \*, filter=None)`

将指定文件加入包内。`arcname`参数用于变换路径和文件名。默认情况下，文件夹会被递归的加入包内，除非`recursive`参数设为`False`。`filter`参数指向一个方法，该方法用来过滤哪些文件不会被打入包内，不被打包的就返回个None，会的就返回`tarinfo`本身，该方法为3.2版本新增，同时废弃原有的exclude方法。

`TarFile.addfile(tarinfo, fileobj=None)`

将一个tarinfo对象关联的文件加入包内。

`tarfile.is_tarfile(name)`

判断一个文件是否打包文件类型。

`TarFile.close()`

关闭TarFile文件。在“w”模式下，会在文件末尾添加量个zero块。

#### 打包文件夹下的文件及目录

```python
import os  
import tarfile  

def package_file(tarfilename, dirname):
    if os.path.isfile(dirname):
        with tarfile.open(tarfilename, 'w') as tar:
            tar.add(dirname)
    else:
        with tarfile.open(tarfilename, 'w') as tar:
            for dirpath, dirs, files in os.walk(dirname):
                for single_file in files:
                    print('start:', single_file)
                    tar.add(os.path.join(dirpath, single_file), single_file)
                    print ('well done')
 
package_file('test.tar', r'C:\Desktop\test\testdir')
```

#### 解包

```python
import tarfile

with tarfile.open(r'C:\Desktop\test\test.tar', 'r') as tar:
    print(tar.getmembers())     # 查看压缩包内文件成员
    tar.extractall(r'C:\Desktop\test\test123')
```

#### 指定包内某一类型文件被解包

```python
import os
import tarfile

def py_files(members):
    for tarinfo in members:
        if os.path.splitext(tarinfo.name)[1] == ".py":
            yield tarinfo

with tarfile.open(r'C:\Desktop\test\test.tar') as tar:
    tar.extractall(r'C:\Desktop\test\test123', members=py_files(tar))
```

### TarInfo对象

一个TarInfo对象代表TarFile里的一个成员。除了保存所有文件必需的属性（例如文件类型、大小、时间、权限、拥有者等等），它还提供了很多有用的方法，用来判断它的类型。但是它不包含文件的具体数据内容。

TarFile对象的`getmember()`,`getmembers()`和`gettarinfo()`会返回TarInfo对象。

| 名称                           | 解释               |
| ------------------------------ | ------------------ |
| class tarfile.TarInfo(name="") | TarInfo类          |
| TarInfo.name                   | 名字               |
| TarInfo.size                   | 大小               |
| TarInfo.mtime                  | 最近的修改时间     |
| TarInfo.mode                   | 权限               |
| TarInfo.type                   | 文件类型           |
| TarInfo.linkname               | 连接目标的名字     |
| TarInfo.uid                    | 用户id             |
| TarInfo.gid                    | 组id               |
| TarInfo.uname                  | 用户名             |
| TarInfo.gname                  | 组名               |
| TarInfo.isfile()               | 判断是否文件       |
| TarInfo.isdir()                | 判断是否目录       |
| TarInfo.issym()                | 判断是否符号链接   |
| TarInfo.islnk()                | 判断是否硬链接     |
| TarInfo.ischr()                | 判断是否字符设备   |
| TarInfo.isblk()                | 判断是否块设备     |
| TarInfo.isfifo()               | 判断是否FIFO设备   |
| TarInfo.isdev()                | 判断是否是设备文件 |

#### 解包文件，并显示部分信息

```python
import tarfile

with tarfile.open(r'C:\Desktop\test\test.tar') as tar:
    for tarinfo in tar:
        print('{} is {} bytes an is '.format(tarinfo.name, tarinfo.size), end='')
        if tarinfo.isreg():
            print("a regular file.")
        elif tarinfo.isdir():
            print("a directory.")
        else:
            print("something else.")
```

#### 往包内添加文件，并使用filter参数修改文件信息

```python
import tarfile
def reset(tarinfo):
    tarinfo.uid = tarinfo.gid = 0
    tarinfo.uname = tarinfo.gname = "root"
    return tarinfo
with tarfile.open("sample.tar.gz", "a") as tar:
	tar.add("foo", filter=reset)
```

***

参考

http://www.liujiangblog.com/course/python/63