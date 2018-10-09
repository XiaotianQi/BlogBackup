
## `open()`

```python
open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
```

在使用该函数的时候，除了 `file` 参数必填外，其他参数可以选用。

`file`：文件名称。

`mode`：制定了文件打开的方式，函数提供了如下方式，其中，`rt`为默认方式。

| `mode` | 说明 |
|:--| :--|
|`r`|open for reading (default)——只读，默认方式|
|`w`|open for writing, truncating the file first——写入，会覆盖源文件内容|
|`x`|create a new file and open it for writing——创建新文件，并写入内容，如果文件已存在，将会报错：FileExistsError|
|`a`|open for writing, appending to the end of the file if it exists——写入，如果文件有内容，则在末尾追加写入|
|`b`|binary mode——二进制模式|
|`t`|text mode (default)——文本模式|
|`+`|open a disk file for updating (reading and writing)——更新磁盘文件，读写|
|`U`|universal newline mode (deprecated)——在paython3中已经弃用|

`buffering`：用于设置缓存策略。在二进制模式下，使用0来切换缓冲；在文本模式下，通过1表示行缓冲（固定大小的缓冲区）。
在不给参数的时候，二进制文件的缓冲区大小由底层设备决定，可以通过 `io.DEFAULT_BUFFER_SIZE` 获取，通常为 4096 或 8192 字节。文本文件则采用行缓冲。

`encoding`：编码或者解码方式。默认编码方式依赖平台。

`errors`：可选，并且不能用于二进制模式，指定了编码错误的处理方式，可以通过 `codecs.Codec` 获得编码错误字符串

`newline`：换行控制。参数有：`None`，`\n`，`\r`，`\r\n`。
输入时，如果参数为 `None`，那么行结束的标志可以是：`\n`，`\r`，`\r\n`任意一个，并且三个控制符都首先会被转化为：`\n`，然后才会被调用；如果参数为''，所有的通用的换行结束标志都可以用，但是行结束标识符返回调用不会被编码。输出时，如果参数为 `None`，那么行结束的标志可以是：`\n`被转换为系统默认的分隔符；如果是''，`\n`则不会被编码。

`closefd`：`False`：文件关闭时，底层文件描述符仍然为打开状态，这是不被允许的，所以，需要设置为 `Ture`。

`opener`：可以通过调用 `opener` 方式，使用自定义的开启器。底层文件描述符是通过调用 `opener` 或者 `file`,  `flags` 获得的。`opener` 必须返回一个打开的文件描述。将 `os.open` 作为 `opener` 的结果，在功能上，类似于通过 `None`。

***

### 测试文件：

```txt
title
1:a
2:b
3:c
4:e
5:d
end
```

### `readline()`

readline() 方法用于从文件读取整行，包括 "\n" 字符。如果指定了一个非负数的参数，则返回指定大小的字节数，包括 "\n" 字符。

```python
fileObject.readline()
```

* 返回值：返回从字符串中读取的字节；
* size -- 从文件中读取的字节数，默认为一整行；
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