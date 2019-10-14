**Unicode 概述**

 一个**字符**是文本的最小部件。  Unicode标准描述了如何使用**码点(code points)**表示字符。 一个 Unicode 字符串是一组码点组成的序列，它们是从 0 到 0x10FFFF (十进制:1,114,111)的数字。

***

**编码**

在内存中，码点序列表示为一组**码元(code units)**，然后将码元映射到8位字节。 将 Unicode 字符串转换为字节的规则称为**字符编码(character encoding)**，或者简称为**编码(encoding)**。 

***

**UTF**

Unicode 解决了字符和二进制的对应关系，但是使用unicode表示一个字符，太浪费空间。例如：利用unicode表示“Python”需要12个字节才能表示，比原来ASCII表示增加了1倍。

为了解决存储和网络传输的问题，出现了Unicode Transformation Format，学术名UTF，即：对unicode中的进行转换，以便于在存储和网络传输时可以节省空间!

* UTF-8： 使用1、2、3、4个字节表示所有字符；优先使用1个字符、无法满足则使增加一个字节，最多4个字节。英文占1个字节、欧洲语系占2个、东亚占3个，其它及特殊字符占4个

* UTF-16： 使用2、4个字节表示所有字符；优先使用2个字节，否则使用4个字节表示。

* UTF-32： 使用4个字节表示所有字符；

UTF 是为 unicode 编码设计的一种在存储和传输时节省空间的编码方案。

 无论以什么编码在内存里显示字符，存到硬盘上都是2进制。存到硬盘上时是以何种编码存的，再从硬盘上读出来时，就必须以何种编码读，要不然就乱了。

***

**Python’s Unicode 支持**

在新版本的python3中，取消了 unicode 类型，代替它的是使用 unicode 字符的字符串类型`str`，字符串类型`str`成为基础类型，Unicode 字符串以code points序列的形式存储在内部。而编码后的变为了字节类型`bytes`，但是`bytes.decode()` 与 `str.encode()`函数的使用方法不变：

```text
      decode              encode
bytes ------> str(unicode)------>bytes
```

```python
u = '中文'					# 类型:str,  指定字符串类型对象u
string = u.encode('utf-8')	 # 类型:bytes,以utf-8编码对u进行编码
u1 = string.decode('utf-8')  # 类型:str,	以utf-8编码对字符串str进行解码
```

在py3里，str 使用 Unicode 标准来表示字符，而其他编码格式的统统都叫bytes，如：gbk，utf-8，gb2312…………

当输入 `str` 时，它使用几种标准方案之一进行编码(encode)，以便字节序列可以被解码(decode)为相同的文本字符串。编码定义了两组值之间转换的方式，因此编码值的字节不一定与码点相同。读取文本数据还需要知道编码方案，以便传入的字节可以转换(即:解码)为 Unicode 类型的 `str`。 

***

**python2与python3之间编码区别**

python中，`decode()`和`encode()`来进行解码和编码

在python2中，使用unicode类型作为编码的基础类型。即

```text
     decode              encode
str ---------> unicode --------->str
```

```python
u = u'中文' 				# 显示指定 unicode 类型对象u
str = u.encode('utf-8')  # 以utf-8编码对unicode对像进行编码
u = str.decode('utf-8')  # 如果以utf-8的编码对str进行解码
```

Python2并不会自动的把文件编码转为unicode存在内存。

python3 把字符串变成了unicode，文件默认编码变成了utf-8。这意味着，只要用python3，无论你的程序是以哪种编码开发的，都可以在全球各国电脑上正常显示。

python3 除了把字符串的编码改成了unicode，还把str 和bytes 做了明确区分，str 就是unicode格式的字符， bytes就是单纯二进制。

***

**编码问题**

出现各种编码问题，无非是哪里的编码设置出错了。常见编码错误的原因有：

* Python解释器的默认编码
* Python源文件文件编码
* Terminal使用的编码
* 操作系统的语言设置 

***

参考：

[Python 编码](https://www.zhihu.com/question/31833164)，冯若航、xlzd、Coldwings的回答