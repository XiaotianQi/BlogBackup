
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

### readline()

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


输出结果：
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

### seek()

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


输出结果：
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