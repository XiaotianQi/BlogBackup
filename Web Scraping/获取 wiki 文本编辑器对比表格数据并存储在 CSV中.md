获取 wiki [文本编辑器对比](https://en.wikipedia.org/wiki/Comparison_of_text_editors)表格数据并存储在 CSV 文件中。以存储第一个展示表格为例。

  示例表格

![文本编辑器对比](https://wx3.sinaimg.cn/mw690/af9e9c30ly1frwks1r9zdj21ao0j9dhd.jpg)

   

大致路线：

  首先，由 ```bsObj.findAll("table", {"class":"wikitable"})[0]``` 获取页面中第一个表格，即上图所示表格；
  然后，```table.findAll("tr")``` 构建一个列表，列表以上表中的一行为一个元素进行存储；
  最后，经由 ```for``` 循环，将上述列表的每一个元素进行解析，并写入 CSV 文件。

   

  Python 代码：

```python
# _ _coding:utf-8_ _

import csv
from urllib.request import urlopen
from bs4 import BeautifulSoup

html = urlopen("http://en.wikipedia.org/wiki/Comparison_of_text_editors")
bsObj = BeautifulSoup(html, "html.parser")

table = bsObj.findAll("table", {"class":"wikitable"})[0]  # 获得页面中第一个表格
rows = table.findAll("tr")  # 以一行为一个列表元素，存储整个表格

# windows 路径要输入完整，若不适用 r""，这路径中使用 \\ 转义字符
csv_file = open(r"C:\GitHub\crawler_tests\files\editors.csv", 'wt', newline='', encoding='utf-8')
writer = csv.writer(csv_file)

try:
    for row in rows:
        csv_row = []
        # 传递列表，那么 find_all() 会逐个找到在列表中元素
        for cell in row.findAll(['td', 'th']):
            csv_row.append(cell.get_text())
        writer.writerow(csv_row)
finally:
    csv_file.close()

```

***

open() 函数

  创建 CSV 时，如果在 Windows 平台中，一下两个方式都是可以成功创建：

```python
csv_file = open(r"C:\GitHub\crawler_tests\files\editors.csv", 'wt', newline='', encoding='utf-8')
csv_file = open("C:\\GitHub\\crawler_tests\\files\\editors.csv", 'wt', newline='', encoding='utf-8')
```

  如果采用相对路径的方式，如下所示代码，无论是否有前缀 r，或者转义字符，都无法成功创建，并返回 IOError。

```python
csv_file = open("..\files\editors.csv", 'wt', newline='', encoding='utf-8')
```

  open() 函数

```python
open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
```

在使用该函数的时候，除了file参数必填外，其他参数可以选用。

file：文件名称。

mode：制定了文件打开的方式，函数提供了如下方式，其中，'rt'为默认方式。

| 模式 | 说明 |
|:--| :--|
|'r'|open for reading (default)——只读，默认方式|
|'w'|open for writing, truncating the file first——写入，会覆盖源文件内容|
|'x'|create a new file and open it for writing——创建新文件，并写入内容，如果文件已存在，将会报错：FileExistsError|
|'a'|open for writing, appending to the end of the file if it exists——写入，如果文件有内容，则在末尾追加写入|
|'b'|binary mode——二进制模式|
|'t'|text mode (default)——文本模式|
|'+'|open a disk file for updating (reading and writing)——更新磁盘文件，读写|
|'U'|universal newline mode (deprecated)——在paython3中已经弃用|

buffering：用于设置缓存策略。在二进制模式下，使用0来切换缓冲；在文本模式下，通过1表示行缓冲（固定大小的缓冲区）。
在不给参数的时候，二进制文件的缓冲区大小由底层设备决定，可以通过io.DEFAULT_BUFFER_SIZE获取，通常为4096或8192字节
文本文件则采用行缓冲。

encoding：编码或者解码方式。默认编码方式依赖平台。

errors：可选，并且不能用于二进制模式，指定了编码错误的处理方式，可以通过codecs.Codec获得编码错误字符串

newline：换行控制。参数有：None，'\n'，'\r'，'\r\n'。
输入时，如果参数为None，那么行结束的标志可以是：'\n'，'\r'，'\r\n'任意一个，并且三个控制符都首先会被转化为：'\n'，然后才会被调用；如果参数为''，所有的通用的换行结束标志都可以用，但是行结束标识符返回调用不会被编码。输出时，如果参数为None，那么行结束的标志可以是：'\n'被转换为系统默认的分隔符；如果是''，'\n'则不会被编码。

closefd：false：文件关闭时，底层文件描述符仍然为打开状态，这是不被允许的，所以，需要设置为ture。

opener：可以通过调用 opener 方式，使用自定义的开启器。底层文件描述符是通过调用 opener 或者 file ,  flags 获得的。
 opener 必须返回一个打开的文件描述。将os.open作为 opener 的结果，在功能上，类似于通过None。

***

findAll() 函数

```python
find_all(name,attrs,recursive,text,limit,**kwargs)
```

attrs, text 可以传入列表，进行逐个的查找过滤，例如：

```python
row.findAll(['td', 'th'])
row.find_all(text=["Free","Yes"])
```
