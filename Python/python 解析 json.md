Json是一种轻量级的数据交换格式。数据交换格式是不同平台、语言中进行数据传递的通用格式。

Json是跨语言，跨平台的，但只能对Python的基本数据类型做操作，对Python的类就无能为力。JSON格式和Python中的字典非常像。但是，json的数据要求用双引号将字符串引起来，并且不能有多余的逗号。这是因为在别的语言中，双引号引起来的才是字符串，单引号引起来的是字符；Python程序员习惯性的在列表、元组或字典的最后一个元素的后面加个逗号，这在json中是不允许的，需要特别注意。

## 类型转换

将数据从Python转换到json格式，在数据类型上会有变化，如下表所示：

| Python                                 | JSON   |
| -------------------------------------- | ------ |
| dict                                   | object |
| list, tuple                            | array  |
| str                                    | string |
| int, float, int- & float-derived Enums | number |
| True                                   | true   |
| False                                  | false  |
| None                                   | null   |

反过来，从json格式转化为Python内置类型，见下表：

| JSON          | Python |
| ------------- | ------ |
| object        | dict   |
| array         | list   |
| string        | str    |
| number (int)  | int    |
| number (real) | float  |
| true          | True   |
| false         | False  |
| null          | None   |

NOTE：json模块不支持bytes类型，要先将bytes转换为str格式。

------

## 使用方法

json模块的使用其实很简单，对于绝大多数场合下，我们只需要使用下面四个方法就可以了：

| 方法                 | 功能                                             |
| -------------------- | ------------------------------------------------ |
| `json.dump(obj, fp)` | 将python数据类型转换并保存到json格式的文件内。   |
| `json.dumps(obj)`    | 将python数据类型转换为json格式的字符串。         |
| `json.load(fp)`      | 从json格式的文件中读取数据并转换为python的类型。 |
| `json.loads(s)`      | 将json格式的字符串转换为python的类型。           |

## JSON 格式化

```python
import json

dic = {'a': 1, 'b': 2, 'c': 3}
js = json.dumps(dic, sort_keys=False, indent=4, separators=(',', ':'))
print(js)
```

sort_keys：是否按照字典排序（a-z）输出，True代表是，False代表否。
indent=4：设置缩进格数，一般由于Linux的习惯，这里会设置为4。
separators：设置分隔符，在dic = {'a': 1, 'b': 2, 'c': 3}这行代码里可以看到冒号和逗号后面都带了个空格，这也是因为Python的默认格式也是如此，如果不想后面带有空格输出，那就可以设置成separators=(',', ':')，如果想保持原样，可以写成separators=(', ', ': ')。
