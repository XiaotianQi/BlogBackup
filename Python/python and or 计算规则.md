Python 里 and、or 的计算规则。

* 没有括号的情况下，and 的优先级大于 or；
* x and y，x 为真则是 y，x 为假则是 x；
* x or y，x 为真则是x，x 为假则是 y。

```python
True or True and False

输出： True
```

```python
print(1 or 2 and 3)
print((1 or 2) and 3)
print(0 or 1 and 2 or 0)


输出结果：
1
3
2
```