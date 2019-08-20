## unittest核心工作原理

unittest中最核心的四个概念是:

* test case
* test suite
* test runner
* test fixture

下面分别解释这四个概念的意思，先来看一张unittest的静态类之间的关系图：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/unittest.png)

- 一个**TestCase**的实例就是一个测试用例。测试用例就是一个完整的测试流程，包括测试前准备环境的搭建(setUp)，执行测试代码(run)，以及测试后环境的还原(tearDown)。元测试(unit  test)的本质也就在这里，一个测试用例是一个完整的测试单元，通过运行这个测试单元，可以对某一个问题进行验证。
- 而将多个TestCase集合在一起，就是**TestSuite**，而且TestSuite也可以嵌套TestSuite。
- **TestLoader**是用来加载TestCase到TestSuite中的，其中有几个`loadTestsFrom__()`方法，就是从各个地方寻找TestCase，创建它们的实例，然后add到TestSuite中，再返回一个TestSuite实例。
- **TextTestRunner**是来执行测试用例的，其中的`run(test)`会执行TestSuite/TestCase中的`run(result)`方法。
- 测试的结果会保存到**TextTestResult**实例中，包括运行了多少测试用例，成功了多少，失败了多少等信息。

由此，整个流程就清楚了：

1. 编写TestCase
2. 然后由TestLoader加载TestCase到TestSuite
3. 然后由TextTestRunner来运行TestSuite，运行的结果保存在TextTestResult中

- 通过命令行或者`unittest.main()`执行时，main会调用TextTestRunner中的run来执行
- 或者可以直接通过TextTestRunner来执行。这里加个说明，在Runner执行时，默认将执行结果输出到控制台，我们可以设置其输出到文件，在文件中查看结果（你可能听说过HTMLTestRunner，是的，通过它可以将结果输出到HTML中，生成漂亮的报告，它跟TextTestRunner是一样的，从名字就能看出来，这个我们后面再说）。

现在已经涉及了test case, test suite, test runner这三个概念，还有test fixture没有提到，那什么是test fixture呢？？在TestCase的docstring中有这样一段话：

> A test fixture represents the preparation needed to perform one or more tests, and any associate cleanup actions. This may involve, for example, creating temporary or proxy databases, directories, or starting a server process.

可见，对一个测试用例环境的搭建和销毁，是一个fixture，通过覆盖TestCase的`setUp()`和`tearDown()`方法来实现。这个有什么用呢？比如说在这个测试用例中需要访问数据库，那么可以在`setUp()`中建立数据库连接以及进行一些初始化，在`tearDown()`中清除在数据库中产生的数据，然后关闭连接。注意`tearDown`的过程很重要，要为以后的TestCase留下一个干净的环境。关于fixture，还有一个专门的库函数叫做fixtures，功能更加强大，以后会介绍到。

***

## 简单示例

待测试 funcs.py 文件：

```python
#_*_coding:utf8_*_

def add(a, b):
    return a+b

def minus(a, b):
    return a-b

def multi(a, b):
    return a*b

def divide(a, b):
    return a/b
```

测试脚本 test_funcs.py 文件：

```python
#_*_coding:utf8_*_

import unittest
from funcs import *


class TestMathFunc(unittest.TestCase):
    """Test mathfuc.py"""

    def test_add(self):
        """Test method add(a, b)"""
        self.assertEqual(3, add(1, 2))
        self.assertNotEqual(3, add(2, 2))

    def test_minus(self):
        """Test method minus(a, b)"""
        self.assertEqual(1, minus(3, 2))

    def test_multi(self):
        """Test method multi(a, b)"""
        self.assertEqual(6, multi(2, 3))

    def test_divide(self):
        """Test method divide(a, b)"""
        self.assertEqual(2, divide(6, 3))
        self.assertEqual(2, divide(5, 2))	# 故意写错，方便展示功能

if __name__ == '__main__':
    unittest.main()
```

一个class继承了`unittest.TestCase`，便是一个测试类，但如果其中有多个以 `test` 开头的方法，那么每有一个这样的方法，在load的时候便会生成一个TestCase用例，如：一个class中有四个test_xxx方法，最后在load到suite中时也有四个测试用例。

执行结果：

```bash
.F..
======================================================================
FAIL: test_divide (__main__.TestMathFunc)
Test method divide(a, b)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "c:/Users/bnwse/Desktop/test/test_funcs.py", line 26, in test_divide
    self.assertEqual(2, divide(5, 2))
AssertionError: 2 != 2.5

----------------------------------------------------------------------
Ran 4 tests in 0.001s

FAILED (failures=1)
```

能够看到一共运行了4个测试，失败了1个，并且给出了失败原因，2 != 2.5，也就是说divide方法是有问题的。

这就是一个简单的测试，有几点需要说明的：

* `.F..`给出了每一个用例执行的结果的标识，成功是 `.`，失败是 `F`，出错是 `E`，跳过是 `S`。不过，执行测试的顺序与方法的顺序没有关系，`test_divide`写在了第4个，但是却是第2个执行
* 每个测试方法均以 `test_` 开头，否则是不被unittest识别

在`unittest.main()`中设置 `verbosity` 参数可以控制输出的错误报告的详细程度：

* 默认是 `1`
* 如果设为 `0`，则不输出每一用例的执行结果，即没有上面的结果中的第1行
* 如果设为 `2`，则输出详细的执行结果，如下：

```bash
test_add (__main__.TestMathFunc)
Test method add(a, b) ... ok
test_divide (__main__.TestMathFunc)
Test method divide(a, b) ... FAIL
test_minus (__main__.TestMathFunc)
Test method minus(a, b) ... ok
test_multi (__main__.TestMathFunc)
Test method multi(a, b) ... ok

======================================================================
FAIL: test_divide (__main__.TestMathFunc)
Test method divide(a, b)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "c:/Users/bnwse/Desktop/test/test_funcs.py", line 26, in test_divide
    self.assertEqual(2, divide(5, 2))
AssertionError: 2 != 2.5

----------------------------------------------------------------------
Ran 4 tests in 0.002s

FAILED (failures=1)
```

而且，每一个测试方法的描述均被输出了出来，**加合适的注释能够使输出的测试报告更加便于阅读**）

### 跳过某个case

如果临时想要跳过某个case不执行怎么办？unittest也提供了几种方法：

1. skip装饰器

```python
...

class TestMathFunc(unittest.TestCase):
    """Test mathfuc.py"""

    ...

    @unittest.skip("I don't want to run this case.")
    def test_divide(self):
        """Test method divide(a, b)"""
        print "divide"
        self.assertEqual(2, divide(6, 3))
        self.assertEqual(3, divide(5, 2))
```

skip装饰器一共有三个 `unittest.skip(reason)`、`unittest.skipIf(condition, reason)`、`unittest.skipUnless(condition, reason)`。

* `skip(reason)`无条件跳过
* `skipIf(condition, reason)`当condition为True时跳过
* `skipUnless(condition, reason)`当condition为False时跳过

2. `TestCase.skipTest()`方法

```python
...

class TestMathFunc(unittest.TestCase):
    """Test mathfuc.py"""

    ...

    def test_divide(self):
        """Test method divide(a, b)"""
        self.skipTest('Do not run this.')
        print("DIVIDE")
        self.assertEqual(2, divide(6, 3))
        self.assertEqual(3, divide(5, 2))
```



***

## TestSuite

上面的代码展示了如何编写一个简单的测试，但仍存在两个问题：

* 怎么控制执行的顺序呢？之后测试方法可能会有先后关系，需要先执行方法A，再执行方法B。
* 现在只有一个测试文件，直接执行该文件即可，但如果有多个测试文件，怎么进行组织？

在上面的代码基础上，再新建一个文件，test_suite.py：

```python
#_*_coding:utf8_*

import unittest
from test_funcs import TestMathFunc

if __name__ == '__main__':
    suite = unittest.TestSuite()

    tests = [TestMathFunc("test_add"), TestMathFunc("test_minus"), TestMathFunc("test_divide")]
    suite.addTests(tests)

    runner = unittest.TextTestRunner(verbosity=2)
    runner.run(suite)
```

输出如下：

```bash
test_add (test_funcs.TestMathFunc)
Test method add(a, b) ... ok
test_minus (test_funcs.TestMathFunc)
Test method minus(a, b) ... ok
test_divide (test_funcs.TestMathFunc)
Test method divide(a, b) ... FAIL

======================================================================
FAIL: test_divide (test_funcs.TestMathFunc)
Test method divide(a, b)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "c:\Users\bnwse\Desktop\test\test_funcs.py", line 26, in test_divide
    self.assertEqual(2, divide(5, 2))
AssertionError: 2 != 2.5

----------------------------------------------------------------------
Ran 3 tests in 0.001s

FAILED (failures=1)
```

添加到TestSuite中的case是会按照添加的顺序执行的。

上面用了TestSuite的 addTests() 方法，并直接传入了TestCase列表，我们还可以：

```python
# 直接用addTest方法添加单个TestCase
suite.addTest(TestMathFunc("test_multi"))

# 用addTests + TestLoader
# loadTestsFromName()，传入'模块名.TestCase名'
suite.addTests(unittest.TestLoader().loadTestsFromName('test_mathfunc.TestMathFunc'))
suite.addTests(unittest.TestLoader().loadTestsFromNames(['test_mathfunc.TestMathFunc']))  # loadTestsFromNames()，类似，传入列表

# loadTestsFromTestCase()，传入TestCase
suite.addTests(unittest.TestLoader().loadTestsFromTestCase(TestMathFunc))
```

注意，用TestLoader的方法是无法对case进行排序的，同时，suite中也可以套suite。

***

## Test Fixture

通过TestSuite，但仍可能遇到点特殊的情况：如果测试需要在每次执行之前准备环境，或者在每次执行完之后需要进行一些清理怎么办？比如执行前需要连接数据库，执行完成之后需要还原数据、断开连接。总不能每个测试方法中都添加准备环境、清理环境的代码吧。对一个测试用例环境的搭建和销毁，是一个fixture，通过覆盖TestCase的`setUp()`和`tearDown()`方法来实现。

这就要涉及到我们之前说过的test fixture了，修改`test_func.py`：

```python
#_*_coding:utf8_*_

import unittest
from funcs import *


class TestMathFunc(unittest.TestCase):
    """Test mathfuc.py"""
    def setUp(self):
        print("do something before test.Prepare environment.")

    def tearDown(self):
        print("do something after test.Clean up.")

    def test_add(self):
        """Test method add(a, b)"""
        self.assertEqual(3, add(1, 2))
        self.assertNotEqual(3, add(2, 2))

    def test_minus(self):
        """Test method minus(a, b)"""
        self.assertEqual(1, minus(3, 2))

    def test_multi(self):
        """Test method multi(a, b)"""
        self.assertEqual(6, multi(2, 3))

    def test_divide(self):
        """Test method divide(a, b)"""
        self.assertEqual(2, divide(6, 3))
        self.assertEqual(2, divide(5, 2))

if __name__ == '__main__':
    unittest.main(verbosity=2)
```

重写了TestCase的 `setUp()` 和 `tearDown()` 两个方法，这两个方法在每个测试方法执行前以及执行后执行一次，`setUp`用来为测试准备环境，`tearDown`用来清理环境，已备之后的测试。

再执行一次 test_suite.py：

```bash
test_add (test_funcs.TestMathFunc)
Test method add(a, b) ... do something before test.Prepare environment.
ADD
do something after test.Clean up.
ok
test_minus (test_funcs.TestMathFunc)
Test method minus(a, b) ... do something before test.Prepare environment.
MINUS
do something after test.Clean up.
ok
test_divide (test_funcs.TestMathFunc)
Test method divide(a, b) ... do something before test.Prepare environment.
DIVIDE
do something after test.Clean up.
FAIL

======================================================================
FAIL: test_divide (test_funcs.TestMathFunc)
Test method divide(a, b)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "c:\Users\bnwse\Desktop\test\test_funcs.py", line 35, in test_divide
    self.assertEqual(2, divide(5, 2))
AssertionError: 2 != 2.5

----------------------------------------------------------------------
Ran 3 tests in 0.004s

FAILED (failures=1)
```

可以看到setUp和tearDown在每次执行case前后都执行了一次。

如果想要在所有case执行之前准备一次环境，并在所有case执行结束之后再清理环境，我们可以用 `setUpClass()` 与 `tearDownClass()`:

```python
...

class TestMathFunc(unittest.TestCase):
    """Test mathfuc.py"""

    @classmethod
    def setUpClass(cls):
        print("This setUpClass() method only called once.")

    @classmethod
    def tearDownClass(cls):
        print("This tearDownClass() method only called once too.")
        
...
```

***

## 将结果输出到文件中

修改test_suite.py：

```python
#_*_coding:utf8_*

import unittest
from test_funcs import TestMathFunc

if __name__ == '__main__':
    suite = unittest.TestSuite()
    suite.addTests(unittest.TestLoader().loadTestsFromTestCase(TestMathFunc))

    with open('UnittestTextReport.txt', 'a') as f:
        runner = unittest.TextTestRunner(stream=f, verbosity=2)
        runner.run(suite)
```

执行此文件，可以看到，在同目录下生成了`UnittestTextReport.txt`，所有的执行报告均输出到了此文件中，这下我们便有了txt格式的测试报告了。

***

参考：

[Python必会的单元测试框架 —— unittest](https://blog.csdn.net/huilan_same/article/details/52944782)，huilan_same