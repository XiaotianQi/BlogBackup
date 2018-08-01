### 占位符

```python
%[(name)][flag][width][.][precision]type
```

* name：命名(传递参数名，不能以数字开头)，以字典格式映射格式化，其为键名；
* flag：标记格式限定符号，包含 '+'、'-'、'0'、' '、'#'。'+' 表示右对齐；'-' 左对齐；'0' 表示填充 0；' ' 表示填充空格；'#' 表示八进制时前面补充 0，16 进制数填充 0x，二进制填充 0b；
* width：宽度。当指定的 width **大于**字符串长度时，会根据具体传入字符串以及 flag 的设置，选择如何填入 0 和 空格；
* precision：小数保留位数；
* type：输出格式类型，如下表。

|格式|描述|
|:-|:-|
|%%|百分号标记|
|%c|字符及其ASCII码|
|%s|字符串|
|%d|有符号整数(十进制)|
|%u|无符号整数(十进制)|
|%o|无符号整数(八进制)|
|%x|无符号整数(十六进制)|
|%X|无符号整数(十六进制大写字符)|
|%e|浮点数字(科学计数法)|
|%E|浮点数字(科学计数法，用E代替e)|
|%f|浮点数字(用小数点符号)|
|%g|浮点数字(根据值的大小采用%e或%f)|
|%G|浮点数字(类似于%g)|
|%p|指针(用十六进制打印值的内存地址)|
|%n|存储输出字符的数量放进参数列表的下一个变量中|

```python
# name
a = 1
b = 2
print("test: %(test1)s, %(test2)d" % {"test1":a, "test2":b})
```

```bash
test: 1, 2
```

```python
# 进制
print("%d" % 5)
print("%#x" % 5)
# 左、右对齐
print("%+3d" % 5)
print("%+3d" % -5)
```

```bash
5
0x5
 +5
 -5
```

```python
# 宽度、精度
print("%3d" % 5)
print("% 3d" % 5)
print("%07d" % 5)
# width 大于传入参数宽度
print("%7.3f" % 5.5)
print("%7f" % 5.5)
print("%-7.3f" % 5.5)
```

```bash
  5
  5
0000005
  5.500
5.500000
5.500
```

**tips**:当指定 float 型参数时，最好采用类似 %2.3f 这种形式的占位符，以免使用 %2f 这种引起的长度无法对齐。

```python
# '*' 的使用
print("%7.3f" % 5.5)
print("%*.3f" % (7, 5.5))
print("%7.*f" % (3, 5.5))
```

```bash
  5.500
  5.500
  5.500
```

**tips**: '*' 可以用来替代 width, precision 中的数值。

***

### `format`

基本语法是通过 {} 和 : 来代替以前的 % 。

先由各个参数功能展示各项功能，

再由 format 的3中映射方式说明如何使用。

```python
{[name][:][[fill]align][sign][#][0][width][,][.precision][type]}
```

* name：命名；
* fill：填充的字符，只能是一个字符，不指定则默认是用空格填充；
* align：对齐方式。< 是左对齐，> 是右对齐，^ 是居中对齐；
* sign：符号。+表示正号，-表示负号；
* width：宽度；
* precision：小数保留位数；
* type：输出格式类型，类似占位符格式表中内容，如下表。

|格式|描述|
|:-|:-|
|%|百分号标记|
|c|字符及其ASCII码|
|s|字符串|
|d|有符号整数(十进制)|
|u|无符号整数(十进制)|
|o|无符号整数(八进制)|
|x|无符号整数(十六进制)|
|X|无符号整数(十六进制大写字符)|
|e|浮点数字(科学计数法)|
|E|浮点数字(科学计数法，用E代替e)|
|f|浮点数字(用小数点符号)|
|g|浮点数字(根据值的大小采用%e或%f)|
|G|浮点数字(类似于%g)|
|p|指针(用十六进制打印值的内存地址)|
|n|存储输出字符的数量放进参数列表的下一个变量中|

```python
# name
print("{a}{b}".format(a="1",b="2"))
l = ["b", "a"]
print("{0[1]}{0[0]}".format(l))
print("{l[1]}{l[0]}".format(l=["b", "a"]))
```

```bash
12
ab
ab
```

**tip**：'0' 是必须的。

```python
# name
dict = {
    "host":"www","domain":"xxx.cn","dot":"."
    }
print("{host}{dot}{domain}".format(**dict))
print("{dict[host]}{dict[dot]}{dict[domain]}".format
        (dict={"host":"www","domain":"xxx.cn","dot":"."}))
```

```bash
www.xxx.cn
www.xxx.cn
```

```python
# fill、sign、width
print("{a:w^3}".format(a="8"))
```

```bash
w8w
```

```python
# 以逗号分隔的数字格式
print("{:,}".format(123456789))
```

```bash
123,456,789
```

```python
# 百分比格式
print("{:.2%}".format(0.25))
```

```bash
25.00%
```

```python
# 使用大括号 {} 来转义大括号
print ("{l[0]} 对应的位置是 {{0}}".format(l=["b", "a"]))
```

```bash
b 对应的位置是 {0}
```

* 位置映射

```python
print("{}:{}".format('127.0.0.1', 1080))
```

```bash
127.0.0.1:1080
```

* 关键字映射

```python
print("{server}{}:{}".format('127.0.0.1', 1080, server='Web Server Info :'))
```

```bash
Web Server Info :127.0.0.1:1080
```

* 索引映射

```python
print("{0[0]}.{1[0]}.{1[1]}".format(('a', 'b'), ('c', 'd')))
```

```bash
a.c.d
```

* 定义对象属性

```python
class Person:  
    def __init__(self,name,age):  
        self.name,self.age = name,age  
    def __str__(self):  
        return 'This guy is {self.name},is {self.age} old.'.format(self=self)

a = Person
print(a('Bob', 18))
```

```bash
is guy is Bob,is 18 old.
```
