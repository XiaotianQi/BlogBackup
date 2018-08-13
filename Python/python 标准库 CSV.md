> 逗号分隔值（Comma-Separated Values，CSV，有时也称为字符分隔值，因为分隔字符也可以不是逗号），其文件以纯文本形式存储表格数据（数字和文本）。纯文本意味着该文件是一个字符序列，不含必须像二进制数字那样被解读的数据。CSV文件由任意数目的记录组成，记录间以某种换行符分隔；每条记录由字段组成，字段间的分隔符是其它字符或字符串，最常见的是逗号或制表符。

## `csv.reader()`

```python
csv.reader(csvfile, dialect='excel', **fmtparams)
```

Return **a reader object** which will **iterate** over lines in the given csvfile.

* `csvfile`：`csvfile` can be any object which **supports the iterator protocol** and **returns a string** each time its `__next__()` method is called — **file objects and list objects** are both suitable. If csvfile is a file object, it should be opened with `newline=''`.
* `dialect='excel'`：An optional dialect parameter can be given which is used to define a set of parameters specific to a particular CSV dialect.
* `**fmtparams`：The other optional fmtparams keyword arguments can be given to **override individual formatting parameters** in the current dialect.

***

## `csv.writer()`

```python
csv.writer(csvfile, dialect='excel', **fmtparams)
```

Return **a writer object** responsible for converting the user’s data into delimited strings on the given file-like object.

参数同 `csv.reader`。

### `csvwriter.writerow(row)`

Write the row parameter to the writer’s file object, formatted according to the current dialect.

### `csvwriter.writerows(rows)`

Write all elements in rows (an iterable of row objects as described above) to the writer’s file object, formatted according to the current dialect.

***

```python
import csv

def writeCsvFile(path, data, header=None, mode='w'):
    try:
        with open(path, mode, newline='') as csv_file:
            writer = csv.writer(csv_file, dialect='excel')

            if header is not None:
                writer.writerow(header)

            writer.writerows(data)

            print("Successful! Path: %s." % path)
    except Exception as e:
        print("Error! Path: %s, Case: %s" % (path, e))

def openCsvFile(path):
    with open(path) as f:
        reader = csv.reader(f)
        for row in reader:
            print(row)
```

```python
header = ['Symbol','Price','Date','Time','Change','Volume']
data = [
    ["AA", 39.48, "6/11/2007", "9:36am", -0.18,181800],
    ["AIG", 71.38, "6/11/2007", "9:36am", -0.15,195500],
    ["AXP", 62.58, "6/11/2007", "9:36am", -0.46,935000],
    ["BA", 98.31, "6/11/2007", "9:36am", +0.12,104800],
    ["C", 53.08, "6/11/2007", "9:36am", -0.25,360900],
    ["CAT", 78.29, "6/11/2007", "9:36am", -0.23,225400]
]
writeCsvFile('testCSV.csv', data, header)
openCsvFile('testCSV.csv')
```

```bash
Successful! Path: testCSV.csv.
['Symbol', 'Price', 'Date', 'Time', 'Change', 'Volume']
['AA', '39.48', '6/11/2007', '9:36am', '-0.18', '181800']
['AIG', '71.38', '6/11/2007', '9:36am', '-0.15', '195500']
['AXP', '62.58', '6/11/2007', '9:36am', '-0.46', '935000']
['BA', '98.31', '6/11/2007', '9:36am', '0.12', '104800']
['C', '53.08', '6/11/2007', '9:36am', '-0.25', '360900']
['CAT', '78.29', '6/11/2007', '9:36am', '-0.23', '225400']
```

![csv文件](https://wx3.sinaimg.cn/mw690/af9e9c30ly1fu60zqjnn8j20g006f74c.jpg)

***

## `dialect`

|属性|默认|含义|
|:--|:--|:--|
|`delimiter`|`,`|字段分隔符（单字符）|
|`doublequote`|`True`|控制 quotechar 实例是否翻倍|
|`escapechar`|`None`|用于表示转义序列的字符|
|`lineterminator`|`\r\n`|写入时用来换行的字符|
|`quotechar`|`"`|引用含特殊值字段的字符（一个字符）|
|`quoting`|`QUOTE_MINIMAL`|控制前面表述的引用行为|
|`skipinitialspace`|`False`|是否在字段分隔符后忽略空格|

|`quoting` 常量|含义|
|:--|:--|
|`QUOTE_ALL`|无论什么类型的字段都会被引用。|
|`QUOTE_MINIMAL`|这是默认的选项, 使用指定的字符引用各字段（如果解析器被配置为相同的 dialect 和选项时，可能会让解析器在解析时产生混淆）。|
|`QUOTE_NONNUMERIC`|引用那些不是整数或浮点数的字段。当使用读取对象时, 如果输入的字段是没有引号的, 那么它们会被转换成浮点数。|
|`QUOTE_NONE`|对所有的输出内容都不加引用，当使用读取对象时，引用字符看作是包含在每个字段的值里（但在正常情况下，它们被当成定界符而被去掉）。|

### `csv.register_dialect(name[, dialect[, **fmtparams]])`

Associate dialect with name. **name must be a string**. The dialect can be specified either by passing a sub-class of Dialect, or by fmtparams keyword arguments, or both, with keyword arguments overriding parameters of the dialect.

### `csv.get_dialect(name)`

获取已定义的 `dialect`。

### `csv.unregister_dialect(name)`

注销自定义的 `dialect`。

### `csv.list_dialects()`
  
列出当前 `csv` 模块里所有的 `dialect`。

```python
import csv
import sys

csv.register_dialect('escaped',
                     escapechar='\\',
                     doublequote=False,
                     quoting=csv.QUOTE_NONE,
                     )
csv.register_dialect('singlequote',
                     quotechar="'",
                     quoting=csv.QUOTE_ALL,
                     )

quoting_modes = {
    getattr(csv, n): n
    for n in dir(csv)
    if n.startswith('QUOTE_')
}

TEMPLATE = '''
Dialect: "{name}"

  delimiter   = {dl!r:<6}    skipinitialspace = {si!r}
  doublequote = {dq!r:<6}    quoting          = {qu}
  quotechar   = {qc!r:<6}    lineterminator   = {lt!r}
  escapechar  = {ec!r:<6}
'''

for name in sorted(csv.list_dialects()):
    dialect = csv.get_dialect(name)

    print(TEMPLATE.format(
        name=name,
        dl=dialect.delimiter,
        si=dialect.skipinitialspace,
        dq=dialect.doublequote,
        qu=quoting_modes[dialect.quoting],
        qc=dialect.quotechar,
        lt=dialect.lineterminator,
        ec=dialect.escapechar,
    ))

    writer = csv.writer(sys.stdout, dialect=dialect)
    writer.writerow(
        ('col1', 1, '10/01/2010', 'Special chars: {} to parse'.format(dialect.delimiter))
    )
    print()
```

```bash
Dialect: "escaped"

  delimiter   = ','       skipinitialspace = 0
  doublequote = 0         quoting          = QUOTE_NONE
  quotechar   = '"'       lineterminator   = '\r\n'
  escapechar  = '\\'

col1,1,10/01/2010,Special chars: \, to parse


Dialect: "excel"

  delimiter   = ','       skipinitialspace = 0
  doublequote = 1         quoting          = QUOTE_MINIMAL
  quotechar   = '"'       lineterminator   = '\r\n'
  escapechar  = None

col1,1,10/01/2010,"Special chars: , to parse"


Dialect: "excel-tab"

  delimiter   = '\t'      skipinitialspace = 0
  doublequote = 1         quoting          = QUOTE_MINIMAL
  quotechar   = '"'       lineterminator   = '\r\n'
  escapechar  = None

col1    1       10/01/2010      "Special chars:          to parse"


Dialect: "singlequote"

  delimiter   = ','       skipinitialspace = 0
  doublequote = 1         quoting          = QUOTE_ALL
  quotechar   = "'"       lineterminator   = '\r\n'
  escapechar  = None

'col1','1','10/01/2010','Special chars: , to parse'


Dialect: "unix"

  delimiter   = ','       skipinitialspace = 0
  doublequote = 1         quoting          = QUOTE_ALL
  quotechar   = '"'       lineterminator   = '\n'
  escapechar  = None

"col1","1","10/01/2010","Special chars: , to parse"
```

***

参考：

[CSV File Reading and Writing](https://docs.python.org/3.7/library/csv.html#id3)

[csv — 逗号分隔值文件格式](https://pythoncaff.com/docs/pymotw/csv-comma-separated-value-files/125)
