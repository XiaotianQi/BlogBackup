## 运算符

从低到高，同一行的运算符具有相同优先级，具有相同优先级的运算符按照从左向右的顺序计算。

| 运算符            | 描述             |
| ----------------- | ---------------- |
| `lambda`          | Lambda表达式     |
| `or`              | 逻辑运算符，或   |
| `and`             | 逻辑运算符，与   |
| `not`             | 逻辑运算符，非   |
| `in` `not in`     | 成员运算符       |
| `is`  `is not`    | 身份运算符       |
| `==` `!=`         | 等于，不等于     |
| `<=` `<` `>` `>=` | 比较             |
| `|`               | 按位或           |
| `^`               | 按位异或         |
| `&`               | 按位与           |
| `>>` `<<`         | 移位             |
| `+` `-`           | 加、减           |
| `*` `/` `%` `//`  | 乘，除，模，整除 |
| `+x` `-x`         | 正负号           |
| `~`               | 按位取反         |
| `**`              | 指数             |
| `[]` `[:]`        | 下标，切片       |

想要改变计算顺序，使用圆括号。所有对象均支持比较操作符。它们拥有相同的优先级（高于布尔操作符）。

***

### x 与 +x

```python
>>> from collections import Counter
>>> ct = Counter('abcdbcaa')
>>> ct
Counter({'a': 3, 'b': 2, 'c': 2, 'd': 1})
>>> ct['c'] = 0
>>> ct['d'] = -2
>>> +ct
Counter({'a': 3, 'b': 2})
```

***

## 布尔操作符---and、or、not

* 没有括号的情况下，**and 的优先级大于 or**；
* x and y，x 为真则是 y，x 为假则是 x；
* x or y，x 为真则是x，x 为假则是 y。

```python
True or True and False	# True
```

```python
print(1 or 2 and 3)			# 1
print((1 or 2) and 3)		# 3
print(0 or 1 and 2 or 0)	# 2
```

闰年判断：

```python
year = int(input('请输入年份: '))
is_leap = (year % 4 == 0 and year % 100 != 0 or
           year % 400 == 0)
print(is_leap)
```

