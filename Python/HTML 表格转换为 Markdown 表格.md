在写博文的时候，需要网上的一个表格。于是，查看网页源码，借此契机，机遇 python 编写了一个 HTML 表格转化为 Markdown 表格的工具。

* 待转换表格：

![待转换表格](https://wx4.sinaimg.cn/mw690/af9e9c30ly1fs1srv9e92j20ia0opwel.jpg)

* 相关页面 HTML 源码：

![相关源码](https://wx3.sinaimg.cn/mw690/af9e9c30ly1fs1srxios5j20di0ml0sv.jpg)

* Markdown 显示样式：

![Markdown 显示样式](https://wx1.sinaimg.cn/mw690/af9e9c30ly1fs1ss079fvj20kk0hvq3b.jpg)

***

* python 代码如下：

```python
# _*_coding:utf-8_*_

from bs4 import BeautifulSoup
import re


f = open('HTML tables.html', encoding='UTF-8')
contents = f.read()
# print(contents)
f_out = open('Markdown tables.md','w+', encoding='UTF-8')

bsObj = BeautifulSoup(contents, 'html.parser')

for idx, tr in enumerate(bsObj.findAll('tr')):
    if idx != 0:
        # 写入表格内容（第二行起），表格的每一行为一个 list 元素
        tds = tr.findAll('td')
        row_str = "|"
        for td in tds:
            td_content_list = ""
            for content in td.findAll('p'):
                # 处理 p 标签下可能存在的子标签
                print("content:", content)
                p_regex = re.compile(r">(.+?)[\s\n]*?<")
                td_content_list = p_regex.findall(str(content))
                td_content_list = ''.join(td_content_list)
                # 使用 markdown 转义字符，避免 | 竖杠引起数据丢失
                td_content_list = td_content_list.replace(r"|", r"\|")
                print("td_content_list:", td_content_list)
            #print td_content_str
            row_str = row_str + " " + td_content_list + " |"
        f_out.write(row_str + "\n")
    else:
        # 表头
        headers = tr.findAll('td')
        row_str = "|"
        row_str2 = "|"
        # 写入表头内容，以及 列表右对齐
        for hearder in headers:
            row_str = row_str + " " + hearder.find('p').get_text() + " |"
            row_str2 = row_str2 + " :- |"
        f_out.write(row_str + "\n")  
        f_out.write(row_str2 + "\n") 

f.close()
f_out.close()

```

编写时，本以为很简单就能写完，所以没有深入的了解难易层次，以至于没申明函数。如果声明函数将源码中的一部分抽离出来，将 for() 循环重写成函数，那么更易读和理解，层次更分明。

* 重新代码：

```python
# _*_coding:utf-8_*_

from bs4 import BeautifulSoup
import re

# 制作表格
def make_forms(bsObj, f_out):
    for idx, tr in enumerate(bsObj.findAll('tr')):
        if idx != 0:
            # 写入表格内容（第二行起），表格的每一行为一个 list 元素
            tds = tr.findAll('td')
            row_str = "|"
            for td in tds:
                td_content_list = get_content(td)
                row_str = row_str + " " + td_content_list + " |"
            f_out.write(row_str + "\n")
        else:
            # 表头
            headers = tr.findAll('td')
            row_str = "|"
            row_str2 = "|"
            # 写入表头内容，以及 列表右对齐
            for hearder in headers:
                row_str = row_str + " " + hearder.find('p').get_text() + " |"
                row_str2 = row_str2 + " :- |"
            f_out.write(row_str + "\n")  
            f_out.write(row_str2 + "\n") 
    return

# 处理 p 标签下可能存在的子标签
def get_content(tag):
    td_content_list = ""
    for content in tag.findAll('p'):
        print("content:", content)
        p_regex = re.compile(r">(.+?)[\s\n]*?<")
        td_content_list = p_regex.findall(str(content))
        td_content_list = ''.join(td_content_list)
        # 使用 markdown 转义字符，避免 | 竖杠引起数据丢失
        td_content_list = td_content_list.replace(r"|", r"\|")
        print("td_content_list:", td_content_list)
    return td_content_list

with open('HTML tables.html', encoding='UTF-8') as f:
    with open('Markdown tables.md','w+', encoding='UTF-8') as f_out:
        contents = f.read()
        bsObj = BeautifulSoup(contents, 'html.parser')
        make_forms(bsObj, f_out)

```

***

* 转化的 Markdown 表格代码:

```markdown
| 运算符 | 描述 |
| :- | :- |
| （） | 最高优先级，先计算括号内的 |
| ** | 指数 |
| ~ + - | 按位翻转,一元加号和减号(最后两个的方法名为+@和-@) |
| * / % // | 乘，除，取模和取整除 |
| + - | 加法减法 |
| &gt;&gt; &lt;&lt; | 右移，左移运算符 |
| &amp; | 位'AND' |
| ^ \| | 位运算符 |
| &lt;= &lt; &gt; &gt;= | 比较运算符 |
| &lt;&gt; == != | 等于运算符 |
| = %= /= //= -= += *= **= | 赋值运算符 |
| is is not | 身份运算符 |
| in not in | 成员运算符 |
| not or and | 逻辑运算符 |

```