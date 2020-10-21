计算机的文件系统是一种存储和组织计算机数据的方法，它使得对其访问和查找变得容易，文件系统使用文件和树形目录的抽象逻辑概念代替了硬盘和光盘等物理设备使用数据块的概念，用户使用文件系统来保存数据不必关心数据实际保存在硬盘（或者光盘）的地址为多少的数据块上，只需要记住这个文件的所属目录和文件名。在写入新数据之前，用户不必关心硬盘上的那个块地址没有被使用，硬盘上的存储空间管理（分配和释放）功能由文件系统自动完成，用户只需要记住数据被写入到了哪个文件中。 

严格地说，文件系统是一套实现了数据的存储、分级组织、访问和获取等操作的抽象数据类型（Abstract data type）。 

## 文件对象

Python中，所有具有`read`和`write`方法的对象，都可以归类为file类型。而所有的file类型对象都可以使用`open`方法打开，`close`方法结束以及被with上下文管理器管理。

### `open()`

```python
open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
```

返回一个文件对象。在使用该函数的时候，除了 `file` 参数必填外，其他参数可以选用。

* `file`：文件名称。

* `mode`：制定了文件打开的方式，函数提供了如下方式，其中，`r`为默认方式，同 `'rt'` ，意味着它以文本模式打开并读取。在文本模式，如果 *encoding* 没有指定，则根据平台来决定使用的编码：使用 `locale.getpreferredencoding(False)` 来获取本地编码。（要读取和写入原始字节，请使用二进制模式并不要指定 *encoding*。）

  | `mode`              | 说明                                                         |
  | ------------------- | ------------------------------------------------------------ |
  | `r`                 | 只读（默认）                                                 |
  | `w`                 | 只写，会覆盖源文件内容                                       |
  | `a`                 | 只写，如果文件有内容，则在末尾追加写入                       |
  | `x`                 | 创建新文件，并写入内容，如果文件已存在，将会报错：FileExistsError |
  | `b`                 | 二进制模式                                                   |
  | `t`                 | 文本模式（默认）                                             |
  | `+`                 | 更新磁盘文件，读写                                           |
  | `U`                 | paython3中已经弃用                                           |
  | `r+`、`w+`、`a+`    | 读写，附带相应功能                                           |
  | `rb`、`wb`、`ab`    | 二进制，附带相应功能                                         |
  | `r+b`、`w+b`、`a+b` | 读写，二进制，附带相应功能                                   |

  ![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/file%20open%20mode.png)

  对于`w+`模式，在读写之前都会清空文件的内容，建议不要使用！

  对于`a+`模式，永远只能在文件的末尾写入，有局限性，建议不要使用！

  对于`r+`模式，也就是读写模式，配合seek()和tell()方法，可以实现更多操作。

  **Python区分二进制和文本I/O**。

  * 以二进制模式打开的文件（包括 mode 参数中的 `'b'` ）返回的内容为 `bytes`对象，不进行任何解码。
  * 在文本模式下（默认情况下，或者在 mode 参数中包含 `’t’` ）时，文件内容返回为 `str` ，首先使用指定的 encoding （如果给定）或者使用平台默认的的字节编码解码。

* `buffering`：用于设置缓存策略。

* `encoding`：编码或者解码方式。只在文本模式下使用。如果不指定，默认值是None，那么在读取文件时使用的是操作系统默认的编码（`locale.getpreferredencoding()`）。如果不能保证保存文件时使用的编码方式与encoding参数指定的编码方式是一致的，那么就可能因无法解码字符而导致读取失败。

* `errors`：可选的字符串参数，，用于指定如何处理编码和解码错误，并且不能用于二进制模式，可以通过 `codecs.Codec` 获得编码错误字符串。

* `newline`：换行控制。参数有：`None`，`\n`，`\r`，`\r\n` , `' '`。
  输入时，如果参数为 `None`，那么行结束的标志可以是：`\n`，`\r`，`\r\n`任意一个，并且三个控制符都首先会被转化为：`\n`，然后才会被调用；如果参数为''，所有的通用的换行结束标志都可以用，但是行结束标识符返回调用不会被编码。输出时，如果参数为 `None`，那么行结束的标志可以是：`\n`被转换为系统默认的分隔符；如果是''，`\n`则不会被编码。

* `closefd`：文件关闭时，底层文件描述符仍然为打开状态，这是不被允许的，所以，需要设置为 `Ture`。

* `opener`：可以通过调用 `opener` 方式，使用自定义的开启器。底层文件描述符是通过调用 `opener` 或者 `file`,  `flags` 获得的。`opener` 必须返回一个打开的文件描述。将 `os.open` 作为 `opener` 的结果，在功能上，类似于通过 `None`。

The type of file object returned by the `open()` function depends on the mode. 

* When `open()` is used to open a file in a text mode (`'w'`, `'r'`, `'wt'`, `'rt'`, etc.), it returns a subclass of `io.TextIOBase` (specifically `io.TextIOWrapper`). 
* When used to open a file in a binary mode with buffering, the returned class is a subclass of `io.BufferedIOBase`. The exact class varies: in read binary mode, it returns an `io.BufferedReader`; in write binary and append binary modes, it returns an `io.BufferedWriter`, and in read/write mode, it returns an `io.BufferedRandom`. When buffering is disabled, the raw stream, a subclass of `io.RawIOBase`,`io.FileIO`, is returned.

如果`open`函数指定的文件并不存在或者无法打开，那么将引发异常状况导致程序崩溃。为了让代码有一定的健壮性和容错性，我们可以使用Python的异常机制对可能在运行时发生状况的代码进行适当的处理，如下所示。

```python]
def main():
    f = None
    try:
        f = open('致橡树.txt', 'r', encoding='utf-8')
        print(f.read())
    except FileNotFoundError:
        print('无法打开指定的文件!')
    except LookupError:
        print('指定了未知的编码!')
    except UnicodeDecodeError:
        print('读取文件时解码错误!')
    finally:
        if f:
            f.close()

def main():
    try:
        with open('致橡树.txt', 'r', encoding='utf-8') as f:
            print(f.read())
    except FileNotFoundError:
        print('无法打开指定的文件!')
    except LookupError:
        print('指定了未知的编码!')
    except UnicodeDecodeError:
        print('读取文件时解码错误!')

```

## 文件对象方法

每当使用`open`方法打开一个文件时，将返回一个文件对象。这个对象内置了很多操作方法。

测试文件：

```txt
title
1:a
2:b
3:c
4:e
5:d
end
```

### 读取内容

几种不同的读取和遍历文件的方法比较：

- 如果文件很小，`read()`一次性读取最方便；
- 如果不能确定文件大小，反复调用`read(size)`比较保险；
- 如果是配置文件，调用`readlines()`最方便。
- 普通情况，使用`for`循环更好，速度更快。

```python
import time


def main():
    # 一次性读取整个文件内容
    with open('致橡树.txt', 'r', encoding='utf-8') as f:
        print(f.read())

    # 通过for-in循环逐行读取
    with open('致橡树.txt', mode='r') as f:
        for line in f:
            print(line, end='')
            time.sleep(0.5)
    print()

    # 读取文件按行读取到列表中
    with open('致橡树.txt') as f:
        lines = f.readlines()
    print(lines)
    

if __name__ == '__main__':
    main()
```

`f.read(size)`读取一定大小的数据, 然后作为字符串或字节对象返回。size是一个可选的数字类型的参数，用于指定读取的数据量。当`size`被忽略了或者为负值，那么该文件的所有内容都将被读取并且返回。

`readline(size)` 方法用于从文件读取整行，包括 `\n` 字符。如果返回一个空字符串，说明已经已经读取到最后一行。这种方法，通常是读一行，处理一行，并且不能回头，只能前进，读过的行不能再读了。如果指定了一个非负数的参数，则返回指定大小的字节数，包括 `\n` 字符。

* 返回值：返回从字符串中读取的字节
* size：从文件中读取的字节数，默认为一整行
* 读取最多为一整行，即至第一个`\n`为止，且包含此`\n`。

```python
# -*- coding: UTF-8 -*-


f = open("test.txt", "r+")
print ("文件名为: ", f.name)

line = f.readline()
print("读取的数据1为: %s" % (line))
line = f.readline(10)
print("读取的数据2为: %s" % (line))
line = f.readline(3)
print("读取的数据3为: %s" % (line))
print("全部内容：", f.read())

f.close()
```

```bash
文件名为:  test.txt
读取的数据1为: title

读取的数据2为: 1:a

读取的数据3为: 2:b
全部内容：
3:c
4:e
5:d
end
```

`f.readlines()`将文件中所有的行，一行一行全部读入一个列表内，按顺序一个一个作为列表的元素，并返回这个列表。`readlines`方法会一次性将文件全部读入内存，所以也存在一定的风险。但是它有个好处，每行都保存在列表里，可以随意存取。

```python
with open('test.txt') as f:
    text = f.readlines()
    print(text)
```

```text
['title\n', '1:a\n', '2:b\n', '3:c\n', '4:e\n', '5:d\n', 'end']
```

实际上，更多的时候，我们将文件对象作为一个迭代器来使用。

```python
with open('test.txt') as f:
    text = f.read()
    for line in text:
        print(line, end='')
```

```text
title
1:a
2:b
3:c
4:e
5:d
end
```

这个方法很简单, 不需要将文件一次性读出，但是同样没有提供一个很好的控制，与`readline`方法一样只能前进，不能回退。

### 写入内容

要将文本信息写入文件文件也非常简单，在使用`open`函数时指定好文件名并将文件模式设置为`'w'`即可。注意如果需要对文件内容进行追加式写入，应该将模式设置为`'a'`。

`write()`将字符串或bytes类型的数据写入文件内。`write()`动作可以多次重复进行，其实都是在内存中的操作，并不会立刻写回硬盘，直到执行`close()`方法后，才会将所有的写入操作反映到硬盘上。

```python
from math import sqrt


def is_prime(n):
    """判断素数的函数"""
    assert n > 0
    for factor in range(2, int(sqrt(n)) + 1):
        if n % factor == 0:
            return False
    return True if n != 1 else False


def main():
    filenames = ('a.txt', 'b.txt', 'c.txt')
    fs_list = []
    try:
        for filename in filenames:
            fs_list.append(open(filename, 'w', encoding='utf-8'))
        for number in range(1, 10000):
            if is_prime(number):
                if number < 100:
                    fs_list[0].write(str(number) + '\n')
                elif number < 1000:
                    fs_list[1].write(str(number) + '\n')
                else:
                    fs_list[2].write(str(number) + '\n')
    except IOError as ex:
        print(ex)
        print('写文件时发生错误!')
    finally:
        for fs in fs_list:
            fs.close()
    print('操作完成!')


if __name__ == '__main__':
    main()
```

### 读写二进制文件

二进制模式，通常用来读取图片、视频等二进制文件。注意，它在读写的时候是以bytes类型读写的，因此获得的是一个bytes对象而不是字符串。在这个读写过程中，需要自己指定编码格式。在使用带b的模式时一定要注意传入的数据类型，确保为bytes类型。

```python
s = 'this is a test'
b = bytes(s,encoding='utf-8')

f = open('test.txt','w')
f.write(s)

# 这样没问题，正常写入了文件。

# -------------------------------------------------
s = 'this is a test'
b = bytes(s,encoding='utf-8')

f = open('test.txt','wb')   	 	# 注意多了个b
f.write(s)

# 报错
TypeError: a bytes-like object is required, not 'str'
# 意思是它需要一个bytes类型数据，你却给了个字符串

# ---------------------------------------------------
s = 'this is a test'
b = bytes(s,encoding='utf-8')

f = open('test.txt','wb')   		# 注意多了个b
f.write(b)							# 将变量b传给它，b是个bytes类型
```

NOTE:

如果读取文件不是unicode编码，就不能直接使用`r`模式，直接读会出现乱码。**只能先以'rb'参数读取二进制文件的方式读取进来，`f.read()`之后再解码**。

```python
with open(filename. 'rb') as f:
    f = f.decode('该文件的编码方式')
```

下面的代码实现了复制图片文件的功能。

```python
def main():
    try:
        with open('guido.jpg', 'rb') as fs1:
            data = fs1.read()
            print(type(data))  # <class 'bytes'>
        with open('吉多.jpg', 'wb') as fs2:
            fs2.write(data)
    except FileNotFoundError as e:
        print('指定的文件无法打开.')
    except IOError as e:
        print('读写文件时出现错误.')
    print('程序执行结束.')


if __name__ == '__main__':
    main()
```

------

### `seek()`

seek() 方法用于移动文件读取指针到指定位置。

```python
fileObject.seek(offset[, whence])
```

* offset -- 开始的偏移量，也就是代表需要移动偏移的字节数；
* whence：可选，默认值为 0。给offset参数一个定义，表示要从哪个位置开始偏移；0代表从文件开头开始算起，1代表从当前位置开始算起，2代表从文件末尾算起。
* 该函数没有返回值。

```python
f = open("test.txt", "r+")
print ("文件名为: ", f.name)

line = f.readline()
print("读取的数据1为: %s" % (line))
line = f.readline(5)
print("读取的数据2为: %s" % (line))

print("全部内容：", f.read())
# 重新设置文件读取指针到开头
f.seek(0, 0)
print("全部内容：", f.read())

f.close()
```

```bash
文件名为:  test.txt
读取的数据1为: title

读取的数据2为: 1:a

全部内容： 2:b
3:c
4:e
5:d
end
全部内容： title
1:a
2:b
3:c
4:e
5:d
end
```

***

### `tell()`

返回文件读写指针当前所处的位置,它是从文件开头开始算起的字节数。一定要注意了，是字节数，不是字符数。

***

## fileinput 标准库

fileinput模块用于对标准输入或多个文件进行逐行遍历。这个模块的使用非常简单，相比`open()`方法批量处理文件，fileinput模块可以对文件、行号进行一定的控制。

```python
fileinput.input(files=None, inplace=False, backup='', bufsize=0, mode='r', openhook=None)
创建并返回一个FileInput类的实例

files:指定要处理的文件，可以是一个多元元组，表示按顺序批量处理元组内文件
inplace:否对源文件进行修改
backup:源文件进行备份的后缀名
mode:文件读写方式，和open()方法的定义一样， 默认为只读‘r’。
```

`input()`函数有点类似文件`readlines()`方法，区别在于:

* 前者是一个迭代对象，即每次只生成一行，需要用`for`循环迭代。

* 后者是一次性读取所有行。在碰到大文件的读取时，前者无疑效率更高效。

`fileinput.input()`方法也可以作为一个上下文管理器使用

```python
with fileinput.input(files=('spam.txt', 'eggs.txt')) as f:
    for line in f:
        process(line)
```

主要属性：

```python
fileinput.filename()
返回当前正在处理的文件名（也就是包含了当前正在处理的文本行的文件）

fileinput.fileno()
返回当前文件的总行数。

fileinput.lineno()
返回当前的行数，这个行数是累计的。多个文件的行数会累加起来。

fileinput.filelineno()
返回当前正在处理的文件的当前行数。每次处理完一个文件并开始处理下一个文件时，该值会重置为1，重新开始计数。

fileinput.isfirstline()
当前行是当前文件的第一行时返回True，否则False

fileinput.isstdin()
当前操作对象为sys.stdin时返回True否则False。

fileinput.nextfile()
关闭当前的文件，跳到下一个文件，跳过的行不计数。

fileinput.close()
关闭整个文件链，结束迭代。
```

统计文件夹中，文件行数：

```python
import os, fileinput

files_g = ()

for root, dirs, files in os.walk(r"C:\test"):
    print("directory {}".format(root))
    if '.vscode' in dirs:
        dirs.remove('.vscode')
    if '__pycache__' in dirs:
        dirs.remove('__pycache__')
    for i in range(len(files)):
        files[i] = '\\'.join([r'C:\test', files[i]])
    files_g = tuple(files)

   
with fileinput.input(files=files_g) as f:
    for file in f:
        pass
    print(f.lineno())
```

***

## chardet 第三方包

检测编码的功能。

```text
chardet.detect()
返回字典，其中confidence是检测精确度，encoding是编码形式
```

```python
import chardet

text_b = b'text strings'
r = chardet.detect(text_b)
print(r)	# {'encoding': 'ascii', 'confidence': 1.0, 'language': ''}

text_gbk = '离离原上草，一岁一枯荣'.encode('gbk')
r = chardet.detect(text_gbk)
print(r)	# {'encoding': 'GB2312', 'confidence': 0.7407407407407407, 'language': 'Chinese'}
```

***

## 读取大文件

python 读文件有一套标准流程：

```python
with open(fname) as file:
	for line in file:
        ...
```

为什么这种文件读取方式会成为标准？这是因为它有两个好处：

* with 上下文管理器采用的文件描述符
* 在迭代文件对象时，内容是一行一行返回的，不会占用太多内存

但这套标准做法并非没有缺点。如果被读取的文件里，根本就没有任何换行符，那么上面的第二个好处就不成立了。当代码执行到 for line in file 时，line 将会变成一个非常巨大的字符串对象，消耗掉非常可观的内存。

如果有一个 5GB 大的文件 big_file.txt，它里面装满了随机字符串。只不过它存储内容的方式稍有不同，所有的文本都被放在了同一行里。

为了解决这个问题，我们需要暂时把这个“标准做法”放到一边，使用更底层的 `file.read()` 方法。与直接循环迭代文件对象不同，每次调用 `file.read(chunk_size)` 会直接返回从当前位置往后读取 `chunk_size` 大小的文件内容，不必等待任何换行符出现。

所以，如果使用 `file.read()` 方法，我们的函数可以改写成这样:

```python
with open(fname) as fp:
	while True:
		chunk = fp.read(1024 * 8)
		# 当文件没有更多内容时，read 调用将会返回空字符串 ''
		if not chunk:
			break
	...
```

使用了一个 `while` 循环来读取文件内容，每次最多读取 8kb 大小，这样可以避免之前需要拼接一个巨大字符串的过程，把内存占用降低非常多。

然而，在循环体内部，存在着两个独立的逻辑：数据生成与数据消费。而这两个独立逻辑被耦合在了一起。为了提升复用能力，我们可以定义一个新的 `chunked_file_reader` 生成器函数，由它来负责所有与“数据生成”相关的逻辑。

```python
def chunked_file_reader(fp, block_size=1024 * 8):
    """生成器函数：分块读取文件内容
    """
    while True:
        chunk = fp.read(block_size)
        # 当文件没有更多内容时，read 调用将会返回空字符串 ''
        if not chunk:
            break
        yield chunk

with open(fname) as fp:
	for chunk in chunked_file_reader(fp):
        ...
```

进行到这一步，代码似乎已经没有优化的空间了，但其实不然。`iter(iterable)` 是一个用来构造迭代器的内建函数，但它还有一个更少人知道的用法。当我们使用 `iter(callable, sentinel)` 的方式调用它时，会返回一个特殊的对象，迭代它将不断产生可调用对象 callable 的调用结果，直到结果为 `setinel` 时，迭代终止。

```python
def chunked_file_reader(file, block_size=1024 * 8):
    """生成器函数：分块读取文件内容，使用 iter 函数
    """
    # 首先使用 partial(fp.read, block_size) 构造一个新的无需参数的函数
    # 循环将不断返回 fp.read(block_size) 调用结果，直到其为 '' 时终止
    for chunk in iter(partial(file.read, block_size), ''):
        yield chunk
```

最后只需要两行代码，就构造出了一个可复用的分块读取方法。

***

参考：

[文件系统](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)，wiki

[文件读写](http://www.liujiangblog.com/course/python/41)，刘江

[7. Input and Output](https://docs.python.org/3/tutorial/inputoutput.html)，官方文档

[文件和异常](https://github.com/jackfrued/Python-100-Days/blob/master/Day01-15/11.%E6%96%87%E4%BB%B6%E5%92%8C%E5%BC%82%E5%B8%B8.md)，骆昊

[Python花式读取大文件(10g/50g/1t)遇到的性能问题](https://juejin.im/post/6844904154037485576)，刘悦