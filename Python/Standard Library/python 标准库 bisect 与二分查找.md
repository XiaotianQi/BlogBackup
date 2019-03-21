Python 的列表（list）内部实现是一个数组，也就是一个线性表。在列表中查找元素可以使用 list.index() 方法，其时间复杂度为O(n)。对于大数据量，则可以用二分查找进行优化。二分查找要求对象必须有序，且必须事先排序。其基本原理如下：

- 从数组的中间元素开始，如果中间元素正好是要查找的元素，则搜素过程结束；
- 如果某一特定元素大于或者小于中间元素，则在数组大于或小于中间元素的那一半中查找，而且跟开始一样，从剩余中间元素开始比较。
- 如此循环
- 如果在某一步骤数组为空，则代表找不到。

这种搜索算法每一次比较都使搜索范围缩小一半。时间复杂度：O(logn)

Python 有一个 `bisect` 模块，用于维护有序列表。`bisect`
模块实现了一个算法用于插入元素到有序列表。在一些情况下，这比反复排序列表或构造一个大的列表再排序的效率更高。Bisect 
是二分法的意思，这里使用二分法来排序，它会将一个元素插入到一个有序列表的合适位置，这使得不需要每次调用 sort 的方式维护有序列表。

bisect模块的源码：

```python
"""Bisection algorithms."""

def insort_right(a, x, lo=0, hi=None):
    """Insert item x in list a, and keep it sorted assuming a is sorted.

    If x is already in a, insert it to the right of the rightmost x.

    Optional args lo (default 0) and hi (default len(a)) bound the
    slice of a to be searched.
    """

    if lo < 0:
        raise ValueError('lo must be non-negative')
    if hi is None:
        hi = len(a)
    while lo < hi:
        mid = (lo+hi)//2
        if x < a[mid]: hi = mid
        else: lo = mid+1
    a.insert(lo, x)

def bisect_right(a, x, lo=0, hi=None):
    """Return the index where to insert item x in list a, assuming a is sorted.

    The return value i is such that all e in a[:i] have e <= x, and all e in
    a[i:] have e > x.  So if x already appears in the list, a.insert(x) will
    insert just after the rightmost x already there.

    Optional args lo (default 0) and hi (default len(a)) bound the
    slice of a to be searched.
    """

    if lo < 0:
        raise ValueError('lo must be non-negative')
    if hi is None:
        hi = len(a)
    while lo < hi:
        mid = (lo+hi)//2
        if x < a[mid]: hi = mid
        else: lo = mid+1
    return lo

def insort_left(a, x, lo=0, hi=None):
    """Insert item x in list a, and keep it sorted assuming a is sorted.

    If x is already in a, insert it to the left of the leftmost x.

    Optional args lo (default 0) and hi (default len(a)) bound the
    slice of a to be searched.
    """

    if lo < 0:
        raise ValueError('lo must be non-negative')
    if hi is None:
        hi = len(a)
    while lo < hi:
        mid = (lo+hi)//2
        if a[mid] < x: lo = mid+1
        else: hi = mid
    a.insert(lo, x)


def bisect_left(a, x, lo=0, hi=None):
    """Return the index where to insert item x in list a, assuming a is sorted.

    The return value i is such that all e in a[:i] have e < x, and all e in
    a[i:] have e >= x.  So if x already appears in the list, a.insert(x) will
    insert just before the leftmost x already there.

    Optional args lo (default 0) and hi (default len(a)) bound the
    slice of a to be searched.
    """

    if lo < 0:
        raise ValueError('lo must be non-negative')
    if hi is None:
        hi = len(a)
    while lo < hi:
        mid = (lo+hi)//2
        if a[mid] < x: lo = mid+1
        else: hi = mid
    return lo

# Overwrite above definitions with a fast C implementation
try:
    from _bisect import *
except ImportError:
    pass

# Create aliases
bisect = bisect_right
insort = insort_right

```

模块提供下面几个方法：

`bisect.bisect_left(a, x, lo=0, hi=len(a))`

定位x在序列a中的插入点，并保持原来的有序状态不变。参数lo和hi用于指定查找区间。如果x已经存在于a中，那么插入点在已存在元素的左边。返回：列表的下标。

`bisect.bisect_right(a, x, lo=0, hi=len(a))`

`bisect.bisect(a, x, lo=0, hi=len(a))`

和上面的区别是插入点在右边。返回：列表的下标

注意，前面这三个方法都是获取插入位置，也就是列表的某个下标的，并不实际插入。但下面三个方法实际插入元素，没有返回值。

`bisect.insort_left(a, x, lo=0, hi=len(a))`

将x插入有序序列a内，并保持有序状态。相当于`a.insert(bisect.bisect_left(a, x, lo, hi), x)`。碰到相同的元素则插入到元素的左边。这个操作通常是 O(log n) 的时间复杂度。

`bisect.insort_right(a, x, lo=0, hi=len(a))`

`bisect.insort(a, x, lo=0, hi=len(a))`

同上，只不过碰到相同的元素则插入到元素的右边。

示例：

该模块比较典型的应用是计算分数等级：

```python
def grade(score,breakpoints=(60, 70, 80, 90), grades='FDCBA'):
    i = bisect.bisect(breakpoints, score)
    return grades[i]

print([grade(score) for score in [33, 99, 77, 70, 89, 90, 100]])
```

利用bisect模块实现通用的列表元素查询方法：

```python
import bisect

def index_left(a, x):
    # 当元素相同时，取左侧元素
    i = bisect.bisect_left(a, x)
    if a[i] == x:
        return i
    else:
        return None

def index_right(a, x):
    # 当元素相同时，取右侧元素
    i = bisect.bisect_right(a, x)
    if a[i-1] == x:
        return i-1
    else:
        return None


if __name__ == "__main__":
    lst = [2, 5, 5, 5, 7]
    x = 3
    print('x:  ', x)
    print('left:', index_left(lst, x))
    print('right: ', index_right(lst, x))
```

***

参考：

http://www.liujiangblog.com/course/python/57