## 文本自动换行与填充

textwrap 模块提供了一些快捷函数，以及可以完成所有工作的类 TextWrapper。如果你只是要对一两个文本字符串进行自动换行或填充，快捷函数应该就够用了；否则的话，你应该使用 TextWrapper 的实例来提高效率。

样本：

```bash
In [36]: text1
Out[36]: 'The textwrap module can be used to format text for output in situations where pretty-printing is desired.  It offers programmatic functionality similar to the paragraph wrapping or filling features found in many text editors.'

In [37]: text2
Out[37]: '\n    The textwrap module can be used to format text for output in\n    situations where pretty-printing is desired.  It offers\n    programmatic functionality similar to the paragraph wrapping\n    or filling features found in many text editors.'
```

***

`textwrap.wrap(text, width=70, **kwargs)`

对 text (字符串) 中的单独段落自动换行以使每行长度最多为 width 个字符。返回由输出行组成的列表，行尾不带换行符。

```bash
In [39]: textwrap.wrap(text1)
Out[39]:
['The textwrap module can be used to format text for output in',
 'situations where pretty-printing is desired.  It offers programmatic',
 'functionality similar to the paragraph wrapping or filling features',
 'found in many text editors.']

In [40]: textwrap.wrap(text2)
Out[40]:
['     The textwrap module can be used to format text for output in',
 'situations where pretty-printing is desired.  It offers',
 'programmatic functionality similar to the paragraph wrapping     or',
 'filling features found in many text editors.']
```

***

`textwrap.fill(text, width=70, **kwargs)`

对 *text* 中的单独段落自动换行，并返回一个包含被自动换行段落的单独字符串。fill()` 是以下语句的快捷方式

```
"\n".join(wrap(text, ...))
```

特别要说明的是，`fill()` 接受与 `wrap()` 完全相同的关键字参数。

```bash
In [41]: textwrap.fill(text1)
Out[41]: 'The textwrap module can be used to format text for output in\nsituations where pretty-printing is desired.  It offers programmatic\nfunctionality similar to the paragraph wrapping or filling features\nfound in many text editors.'

In [42]: textwrap.fill(text2)
Out[42]: '     The textwrap module can be used to format text for output in\nsituations where pretty-printing is desired.  It offers\nprogrammatic functionality similar to the paragraph wrapping     or\nfilling features found in many text editors.'
```

***

`textwrap.shorten(text, width, **kwargs)`

折叠并截短给定的 text 以符合给定的 width。

首先将折叠 text 中的空格（所有连续空格替换为单个空格）。如果结果能适合 width 则将其返回。否则将丢弃足够数量的末尾单词以使得剩余单词加 placeholder 能适合 width:

```bash
In [45]: textwrap.shorten(text1, 20)
Out[45]: 'The [...]'

In [46]: textwrap.shorten(text1, width=10, placeholder="...")
Out[46]: 'The...'
```

***

` textwrap.dedent(text)`

移除 text 中每一行的任何相同前缀空白符。

这可以用来清除三重引号字符串行左侧空格，而仍然在源码中显示为缩进格式。

请注意制表符和空格符都被视为是空白符，但它们并不相等：以下两行 `" hello"` 和 `"\thello"` 不会被视为具有相同的前缀空白符。

只包含空白符的行会在输入时被忽略并在输出时被标准化为单个换行符。

```bash
In [48]: textwrap.dedent(text2)
Out[48]: '\nThe textwrap module can be used to format text for output in\nsituations where pretty-printing is desired.  It offers\nprogrammatic functionality similar to the paragraph wrapping\nor filling features found in many text editors.'
```

***

`textwrap.indent(text, prefix, predicate=None)`

将 prefix 添加到 text 中选定行的开头。

通过调用 `text.splitlines(True)` 来对行进行拆分。

默认情况下，prefix 会被添加到所有不是只由空白符（包括任何行结束符）组成的行。

```bash
In [54]: s
Out[54]: 'hello\n\n \nworld'

In [55]: textwrap.indent(s, '  ')
Out[55]: '  hello\n\n \n  world'

In [56]: textwrap.indent(s, '+ ', lambda line: True)
Out[56]: '+ hello\n+ \n+  \n+ world'
```

***

参考：

[textwrap — Text wrapping and filling](https://docs.python.org/3.8/library/textwrap.html)