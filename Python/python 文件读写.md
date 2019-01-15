Python中，所有具有read和write方法的对象，都可以归类为file类型。而所有的file类型对象都可以使用open方法打开，close方法结束和被with上下文管理器管理。

## 文件对象

### `open()`

```python
open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
```

返回一个文件对象。在使用该函数的时候，除了 `file` 参数必填外，其他参数可以选用。

* `file`：文件名称。

* `mode`：制定了文件打开的方式，函数提供了如下方式，其中，`r`为默认方式，同 `'rt'` ，意味着它以文本模式打开并读取。在文本模式，如果 *encoding* 没有指定，则根据平台来决定使用的编码：使用 `locale.getpreferredencoding(False)` 来获取本地编码。（要读取和写入原始字节，请使用二进制模式并不要指定 *encoding*。）

  | `mode` | 说明                                                         |
  | ------ | ------------------------------------------------------------ |
  | `r`    | open for reading (default)——只读，默认方式                   |
  | `w`    | open for writing, truncating the file first——写入，会覆盖源文件内容 |
  | `x`    | create a new file and open it for writing——创建新文件，并写入内容，如果文件已存在，将会报错：FileExistsError |
  | `a`    | open for writing, appending to the end of the file if it exists——写入，如果文件有内容，则在末尾追加写入 |
  | `b`    | binary mode——二进制模式                                      |
  | `t`    | text mode (default)——文本模式                                |
  | `+`    | open a disk file for updating (reading and writing)——更新磁盘文件，读写 |
  | `U`    | universal newline mode (deprecated)——在paython3中已经弃用    |

  对于w+模式，在读写之前都会清空文件的内容，建议不要使用！

  对于a+模式，永远只能在文件的末尾写入，有局限性，建议不要使用！

  对于r+模式，也就是读写模式，配合seek()和tell()方法，可以实现更多操作。

  Python区分二进制和文本I/O。

  * 以二进制模式打开的文件（包括 *mode* 参数中的 `'b'` ）返回的内容为 `bytes`对象，不进行任何解码。
  * 在文本模式下（默认情况下，或者在 *mode* 参数中包含 ``’t’` ）时，文件内容返回为 `str` ，首先使用指定的 *encoding* （如果给定）或者使用平台默认的的字节编码解码。

* `buffering`：用于设置缓存策略。

* `encoding`：编码或者解码方式。只在文本模式下使用，默认编码方式依赖平台（`locale.getpreferredencoding()`）。

* `errors`：可选的字符串参数，，用于指定如何处理编码和解码错误，并且不能用于二进制模式，可以通过 `codecs.Codec` 获得编码错误字符串。

* `newline`：换行控制。参数有：`None`，`\n`，`\r`，`\r\n` , `' '`。
  输入时，如果参数为 `None`，那么行结束的标志可以是：`\n`，`\r`，`\r\n`任意一个，并且三个控制符都首先会被转化为：`\n`，然后才会被调用；如果参数为''，所有的通用的换行结束标志都可以用，但是行结束标识符返回调用不会被编码。输出时，如果参数为 `None`，那么行结束的标志可以是：`\n`被转换为系统默认的分隔符；如果是''，`\n`则不会被编码。

* `closefd`：文件关闭时，底层文件描述符仍然为打开状态，这是不被允许的，所以，需要设置为 `Ture`。

* `opener`：可以通过调用 `opener` 方式，使用自定义的开启器。底层文件描述符是通过调用 `opener` 或者 `file`,  `flags` 获得的。`opener` 必须返回一个打开的文件描述。将 `os.open` 作为 `opener` 的结果，在功能上，类似于通过 `None`。

The type of file object returned by the `open()` function depends on the mode. 

* When `open()` is used to open a file in a text mode (`'w'`, `'r'`, `'wt'`, `'rt'`, etc.), it returns a subclass of `io.TextIOBase` (specifically `io.TextIOWrapper`). 
* When used to open a file in a binary mode with buffering, the returned class is a subclass of `io.BufferedIOBase`. The exact class varies: in read binary mode, it returns an `io.BufferedReader`; in write binary and append binary modes, it returns an `io.BufferedWriter`, and in read/write mode, it returns an `io.BufferedRandom`. When buffering is disabled, the raw stream, a subclass of `io.RawIOBase`,`io.FileIO`, is returned.

### B模式

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

***

## 文件对象方法

每当我们用open方法打开一个文件时，将返回一个文件对象。这个对象内置了很多操作方法。

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

### `f.read(size)`

读取一定大小的数据, 然后作为字符串或字节对象返回。size是一个可选的数字类型的参数，用于指定读取的数据量。当size被忽略了或者为负值，那么该文件的所有内容都将被读取并且返回。

***

### `f.readline()`

readline() 方法用于从文件读取整行，包括 "\n" 字符。从文件中读取一行n内容，换行符为'\n'。如果返回一个空字符串，说明已经已经读取到最后一行。这种方法，通常是读一行，处理一行，并且不能回头，只能前进，读过的行不能再读了。如果指定了一个非负数的参数，则返回指定大小的字节数，包括 "\n" 字符。

* 返回值：返回从字符串中读取的字节
* size -- 从文件中读取的字节数，默认为一整行
* 读取最多为一整行，即至第一个 \n 为止，且包含此 \n。

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

* readline()、readline(10)、readline(3) 之间均有换行符，说明默认或者全部读取均最多读取至 第一个 \n，且包含此换行符；
* readline(3) 和 read() 之间无换行符，说明 readline(3) 读取的那一行文本的 \n 由 read() 读取。所以这二者之间的无空白行；
* read() 首先读取到的是 \n，所以“全部内容”与“3:c”之间换行，否则应为“全部内容：3:c”。

***

### `f.readlines()`

将文件中所有的行，一行一行全部读入一个列表内，按顺序一个一个作为列表的元素，并返回这个列表。readlines方法会一次性将文件全部读入内存，所以也存在一定的风险。但是它有个好处，每行都保存在列表里，可以随意存取。

```python
with open('test.txt') as f:
    text = f.readlines()
    print(text)
```

```text
['title\n', '1:a\n', '2:b\n', '3:c\n', '4:e\n', '5:d\n', 'end']
```

------

### 文件遍历

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

这个方法很简单, 不需要将文件一次性读出，但是同样没有提供一个很好的控制，与readline方法一样只能前进，不能回退。

几种不同的读取和遍历文件的方法比较：如果文件很小，read()一次性读取最方便；如果不能确定文件大小，反复调用read(size)比较保险；如果是配置文件，调用readlines()最方便。普通情况，使用for循环更好，速度更快。

***

### `write()`

将字符串或bytes类型的数据写入文件内。write()动作可以多次重复进行，其实都是在内存中的操作，并不会立刻写回硬盘，直到执行close()方法后，才会将所有的写入操作反映到硬盘上。

***

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

参考：

http://www.liujiangblog.com/course/python/41

https://docs.python.org/3/tutorial/inputoutput.html