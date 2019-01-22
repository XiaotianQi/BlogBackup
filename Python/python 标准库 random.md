random模块用于生成伪随机数。

真正意义上的随机数（或者随机事件）是按照实验过程中表现的分布概率随机产生的，其结果是不可预测的。而计算机中的随机数是所谓的随机函数按照一定算法模拟产生的，其结果是确定的，是可预测的。所以用计算机随机函数所产生的“随机数”并不随机，是伪随机数，绝对不可以用来生成密码。

1.计算机的伪随机数是由随机种子根据一定的计算方法计算出来的数值。所以，只要计算方法一定，随机种子一定，那么产生的随机数就是固定的。

2.如果用户不设置随机种子，那么随机种子默认来自系统时钟。

##  基本方法

`random.seed(a=None, version=2)`

初始化随机数生成器。如果未提供a或者`a=None`，则使用系统时间为种子。如果a是一个整数，则作为新的种子。

`random.getstate()`

返回当前生成器的内部状态

`random.setstate(state)`

传入一个先前利用getstate方法获得的状态对象，使得生成器恢复到这个状态。

`random.getrandbits(k)`

返回一个不大于K位的Python整数（十进制），比如k=10，则结果是0~2^10之间的整数。

## 针对整数的方法

注意：在使用后面的方法时，一定要小心区间的开闭！

`random.randrange(stop)`

`random.randrange(start, stop[, step])`

等同于后面的`choice(range(start,stop,step))`，从range的范围内随机选择一个整数。这个方法并不实际创建range对象。

`random.randint(a, b)`

返回一个`a <= N <= b`的随机整数N。等同于`randrange(a, b+1)`。

## 针对序列类型的方法

`random.choice(seq)`

从非空序列seq中随机选取一个元素。如果seq为空则弹出`IndexError`异常。

`random.choices(population, weights=None, \*, cum_weights=None, k=1)`

3.6版本新增。从`population`集群中随机抽取K个元素。`weights`是相对权重列表，`cum_weights`是累计权重，两个参数不能同时存在。

`random.shuffle(x[, random])`

随机打乱序列x内元素的排列顺序，俗称“洗牌”。只能用于可变的序列，对于不可变序列，请使用下面的`sample()`方法。

`random.sample(population, k)`

从population样本或集合中随机抽取K个不重复的元素形成新的序列。常用于不重复的随机抽样。返回的是一个新的序列，不会破坏原有序列。比如从一个整数区间随机抽取一定数量的整数`random.sample(range(10000000), k=60)`，这非常有效和节省空间。 如果k大于population的长度，则弹出ValueError异常。

## 分布

random模块最高端的功能其实在这里。

`random.random()`

返回一个介于左闭右开`[0.0, 1.0)`区间的浮点数。

`random.uniform(a, b)`

返回一个介于a和b之间的浮点数。如果a>b，则是b到a之间的浮点数。这里的a和b都有可能出现在结果中。

`random.triangular(low, high, mode)`

返回一个`low <= N <=high`的三角形分布的随机数。参数mode指明众数出现位置。

`random.betavariate(alpha, beta)`

β分布。返回的结果在0~1之间。

`random.expovariate(lambd)`

指数分布

`random.gammavariate(alpha, beta)`

伽马分布

`random.gauss(mu, sigma)`

高斯分布

`random.lognormvariate(mu, sigma)`

对数正态分布

`random.normalvariate(mu, sigma)`

正态分布

`random.vonmisesvariate(mu, kappa)`

卡帕分布

`random.paretovariate(alpha)`

帕累托分布

`random.weibullvariate(alpha, beta)`

这个...请数学高手解答一下，囧。

## 可选择的生成器

`class random.SystemRandom([seed])`

使用`os.urandom()`方法生成随机数的类，由操作系统提供源码，不一定所有系统都支持。

## 示例

### 生成一个包含大写字母A-Z和数字0-9的随机4位验证码的程序

```python
import random

checkcode = ''
for i in range(4):
    current = random.randrange(0,4)
    if current != i:
        temp = chr(random.randint(65,90))
    else:
        temp = random.randint(0,9)
    checkcode += str(temp)
print(checkcode)
```

### 生成指定长度字母数字随机序列的代码

```python
import random, string

def gen_random_string(length):
    # 数字的个数随机产生
    num_of_numeric = random.randint(1,length-1)
    # 剩下的都是字母
    num_of_letter = length - num_of_numeric
    # 随机生成数字
    numerics = [random.choice(string.digits) for i in range(num_of_numeric)]
    # 随机生成字母
    letters = [random.choice(string.ascii_letters) for i in range(num_of_letter)]
    # 结合两者
    all_chars = numerics + letters
    # 洗牌
    random.shuffle(all_chars)
    # 生成最终字符串
    result = ''.join([i for i in all_chars])
    return result

if __name__ == '__main__':
    print(gen_random_string(64))
```

***

参考：

http://www.liujiangblog.com/course/python/56