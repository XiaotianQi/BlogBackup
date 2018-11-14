> 1.人名币大写转换。

```python
No2CN = {0: u'零', 1: u'壹', 2: u'贰', 3: u'叁', 4: u'肆', 5: u'伍',
            6: u'陆', 7: u'柒', 8: u'捌', 9: u'玖'}
units = [u'亿', u'仟', u'佰', u'拾', u'万', u'仟', u'佰', u'拾', u'圆']
replace_zero = [
    (u'零仟', u'零佰', u'零拾', u'零零零', u'零零', u'零万', u'零圆'),
    (u'零', u'零', u'零', u'零', u'零', u'万', u'圆')
    ]

b = abs(a)
new = []
for i in str(b):
    if int(i) in No2CN.keys():
        new.append(No2CN[int(i)])
units_new = units[-len(new):]
result = ''
for i, j in zip(new, units_new):
    result += i+j
for i, j in zip(replace_zero[0], replace_zero[1]):
    result = result.replace(i, j)
if a < 0:
    print(u'负'+result)
else:
    print(result)
```

* 整理思路

```python
M = [u'零', u'壹', u'贰', u'叁', u'肆', u'伍', u'陆', u'柒', u'捌', u'玖']
N = ['', u'圆', u'拾', u'佰', u'仟', u'万', u'拾', u'佰', u'仟', u'亿']
O = {
    u'零仟': u'零', u'零佰': u'零', u'零拾': u'零', u'零零零': u'零', u'零零': u'零',
    u'零万': u'万', u'零圆': u'圆', u'亿万': u'亿'
    }

s = str(abs(a))
r = ('' if a >= 0 else '负')
for i in range(0, len(s)):
    r = r + M[int(s[i])] + N[len(s)-i]
for i in range(len(s)):
    for j in O:
        r = r.replace(j, O[j])
print(r if a != 0 else u'零圆')
```

***

> 2.简单加密。给你个小写英文字符串a和一个非负数b(0<=b<26), 将a中的每个小写字符替换成字母表中比它大b的字母。这里将字母表的z和a相连，如果超过了z就回到了a。例如a="cagy", b=3,则输出 ：fdjb。

```python
c = ''
for i in a:
    if ord(i)+b <= ord('z'):
        c += chr(ord(i)+b)
    else:
        c += chr(ord(i)+b-26)
print(c)
```

* 整理思路

```python
c = ''
for i in a:
    c += chr((ord(i)+b-ord('a'))%26 + ord('a'))
print(c)
```

***
> 3.回文子串。给你一个字符串a和一个正整数n,判断a中是否存在长度为n的回文子串。如果存在，则输出YES，否则输出NO。

```python
for i in range(len(a)-n+1):
    if a[i:i+n] == a[i:i+n][::-1]:
        print('YES')
        break
    if i == len(a)-n:
        print('NO')
```

***

> 3.给你两个时间st和et(00:00:00<=st <= et<=23:59:59), 请你给出这两个时间间隔的秒数。如：st="00:00:00", et="00:00:10", 则输出10。

```python
from datetime import datetime
print((datetime.strptime(et, '%H:%M:%S')-datetime.strptime(st, '%H:%M:%S')).seconds)
```

***

> 4.闰年判定，365 Or 366？现在给你一个年份year(year为四位数字的字符串，如"2008","0012"),你输出这一年的天数。如year="2013", 则输出365。

```python
year = int(year)
if year%400 == 0:
    print(366)
elif (year%4 == 0) and (year%100 != 0):
    print(366)
else:
    print(365)
```

***