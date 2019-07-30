



* pytest
* unittest：Python自带的unittest单元测试框架



## 断言



```python
a = 1
b = 2
assert a == b, 'NOT equal:{} {}'.format(a, b)	# 断言抛出问题后，代码不会继续向下运行
print('1212')
```









assertpy

```python
from assertpy import assert_that

def test_something():
    assert_that(1 + 2).is_equal_to(3)
    assert_that('foobar')\
        .is_length(6)\
        .starts_with('foo')\
        .ends_with('bar')
    assert_that(['a', 'b', 'c'])\
        .contains('a')\
        .does_not_contain('x')
```



****

参考：

[Python中不尽如人意的断言Assertion](https://segmentfault.com/a/1190000007248161)，betacat

