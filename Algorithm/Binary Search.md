二分查找要求对象**必须有序**。其基本原理如下：

- 从数组的中间元素开始，如果中间元素正好是要查找的元素，则搜素过程结束；
- 如果某一特定元素大于或者小于中间元素，则在数组大于或小于中间元素的那一半中查找，而且跟开始一样，从剩余中间元素开始比较。
- 如此循环
- 如果在某一步骤数组为空，则代表找不到。

这种搜索算法每一次比较都使搜索范围缩小一半。时间复杂度：
$$
O(log_2n)
$$
三种实现二分查找的方法，当元素相同时，取最左侧元素的下标：

```python
import bisect

def binary_search_recursion(lst, value, lo=0, hi=None):
    # 递归
    if lo < 0:
        raise ValueError('lo must be non-negative')
    if hi is None:
        hi = len(lst)
    mid = (lo + hi) // 2
    if lo >= hi:
        if lst[lo] == x:
            return lo
        else:
            return None
    elif lst[mid] < value:
        return binary_search_recursion(lst, value, mid+1, hi)
    else:
        return binary_search_recursion(lst, value, lo, mid)


def binary_search_loop(lst, value):
    # 循环
    lo, hi = 0, len(lst)
    while lo < hi:
        mid = (lo + hi) // 2
        if lst[mid] < value:
            lo = mid + 1	# 首次匹配到value时，lo值就是首次匹配的下标
        else:
            hi = mid
    if lst[lo] == x:
        return lo
    return None

def binary_search_bisect(lst, x):
    # 标准库 bisect
    i = bisect.bisect_left(lst, x)
    if lst[i] == x:
        return i
    return None


if __name__ == "__main__":
    import random
    lst = [2, 5, 5, 5, 7]
    x = 3
    print('x: ', x)
    print('re:', binary_search_recursion(lst, x))
    print('loop: ', binary_search_loop(lst, x))
    print('besect: ', binary_search_bisect(lst, x))
```

最右侧下标：

```python
def binary_search_right(lst, value):
    lo = 0
    hi = len(lst)
    while lo < hi:
        mid =(lo+hi) // 2
        if value < lst[mid]:
            hi = mid
        else:
            lo = mid + 1	# 匹配到value后，lo值仍不断增加继续匹配，lo值最后是value下标+1
    if lst[lo-1] == value:
        return lo-1
    return None

def binary_search_bisect(lst, x):
    # 标准库 bisect
    import bisect
    i = bisect.bisect_right(lst, x)
    if lst[i-1] == x:
        return i-1
    return None
```

