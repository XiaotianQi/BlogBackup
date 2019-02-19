标准库提供了concurrent.futures模块，它提供了 ThreadPoolExecutor 和 ProcessPoolExecutor 两个类，实现了对threading和multiprocessing的进一步抽象，对编写线程池/进程池提供了直接的支持。使用`concurrent.futures`编写的代码可以轻松地在线程版本与进程版本之间转换。

线程池或进程池是用于在程序中优化和简化线程/进程的使用。通过池，可以提交任务给executor。池由两部分组成，一部分是内部的队列，存放着待执行的任务；另一部分是一系列的进程或线程，用于执行这些任务。池的概念主要目的是为了重用：让线程或进程在生命周期内可以多次使用。它减少了创建创建线程和进程的开销，提高了程序性能。重用不是必须的规则，但它是程序员在应用中使用池的主要原因。

***

## ThreadPoolExecutor

在使用多线程处理任务时也不是线程越多越好。因为在切换线程的时候，需要切换上下文环境，线程很多的时候，依然会造成CPU的大量开销。为解决这个问题，线程池的概念被提出来了。

标准库提供了concurrent.futures模块，它提供了 ThreadPoolExecutor 类，实现了对threading的进一步抽象，对编写线程池提供了直接的支持。

线程池是用于在程序中优化和简化线程的使用。通过池，可以提交任务给executor。池由两部分组成，一部分是内部的队列，存放着待执行的任务；另一部分是一系列的线程，用于执行这些任务。池的概念主要目的是为了重用：让线程在生命周期内可以多次使用。它减少了创建创建线程的开销，提高了程序性能。重用不是必须的规则，但它是程序员在应用中使用池的主要原因。

线程池不是线程安全的。

ThreadPoolExecutor 是 Executor 的子类，它使用线程池执行异步调用。

### 语法

```text
class concurrent.futures.ThreadPoolExecutor(max_workers=None, thread_name_prefix='', initializer=None, initargs=())

max_workers:最多使用线程数量.默认是计算机处理器数量*5

thread_name_prefix:线程名称前缀

initializer:可选调用，在每个线程启动前调用

initargs:元组，传入initializer的参数
```

### `as_completed()`和`map()`区别

* `concurrent.futures.Executor.map()`

输出结果的顺序和传入参数的顺序相同。

```python
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

def worker(times):
    time.sleep(times)
    print('Get--', times)
    return times

l = [3, 1, 2]
with ThreadPoolExecutor(max_workers=2) as executor:
    for data in executor.map(worker, l):
        print('Done--', data)
```

输出结果：

```python
Get-- 1
Get-- 3
Done-- 3
Done-- 1
Get-- 2
Done-- 2
```

*  `concurrent.futures.as_complete()`

谁先完成就返回谁。

```python
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

def worker(times):
    time.sleep(times)
    print('Get--', times)
    return times

l = [3, 1, 2]
with ThreadPoolExecutor(max_workers=2) as executor:
    tasts = [executor.submit(worker, i) for i in l]
    for future in as_completed(tasts):
        data = future.result()
        print('Done--', data)
```

输出结果：

```python
Get-- 1
Done-- 1
Get-- 3
Get-- 2
Done-- 3
Done-- 2
```

***

## ProcessPoolExecutor

ProcessPoolExecutor 类是Executor子类，使用一个进程池来执行异步调用。ProcessPoolExecutor使用multiprocessing模块，这允许它绕过GIL，但也意味着只能执行和返回picklable objects。

The `__main__ `module must be importable by worker subprocesses. This means that ProcessPoolExecutor will not work in the interactive interpreter.

```python
class concurrent.futures.ProcessPoolExecutor(max_workers=None, mp_context=None, initializer=None, initargs=())
```

使用方法和 ThreadPoolExecutor 相同。max_workers 默认是计算机处理器数量。

官方示例：

```python
import concurrent.futures
import math

PRIMES = [
    112272535095293,
    112582705942171,
    112272535095293,
    115280095190773,
    115797848077099,
    1099726899285419]

def is_prime(n):
    if n % 2 == 0:
        return False

    sqrt_n = int(math.floor(math.sqrt(n)))
    for i in range(3, sqrt_n + 1, 2):
        if n % i == 0:
            return False
    return True

def main():
    with concurrent.futures.ProcessPoolExecutor() as executor:
        for number, prime in zip(PRIMES, executor.map(is_prime, PRIMES)):
            print('%d is prime: %s' % (number, prime))

if __name__ == '__main__':
    main()
```

***

## Executor Objects

### 语法、方法

```text
class concurrent.futures.Executor:执行异步调用的方法的抽象基类

Executor.submit(fn, *args, **kwargs):将 args、kwargs 传入可调用函数 fn，并返回一个 future 对象。
submit方法会对future进行排期
如果运行的线程数没达到最大线程数，则future会被立即运行，并将其状态置为running；
否则就等待，并将其状态置为pending。

Executor.map(func, *iterables, timeout=None, chunksize=1):和内置map(func, *iterables)函数类似。
区别在于:立即执行而不是惰性计算，func 是异步执行。输出结果的顺序和传入参数的顺序相同。

Executor.shutdown(wait=True):向executor发出信号，释放所有资源。
wait=True时，直到所有pending futures执行完毕，再返回。
wait=False时，立即返回。
```

### wait=True&swait=False

调用shutdown(wait=True)或使用上下文管理器将阻塞，直到所有挂起的期货完成执行。

```python
from concurrent.futures import ThreadPoolExecutor
import time

def task(name):
    print("name",name)
    time.sleep(1)

start = time.time()
executor = ThreadPoolExecutor(max_workers=2)
for i in range(5):
    executor.submit(task,"task-%d"%i)
executor.shutdown(wait=True)
end = time.time()
print('cost:', end-start)
```

输出结果：

```text
name task-0
name task-1
name task-2
name task-3
name task-4
cost: 3.013853073120117
```

当修改为`wait=False`时：

```text
name task-0
name task-1
cost: 0.002997159957885742
name task-2
name task-3
name task-4
```

***

## Future Objects

Future 类封装了可调用的异步执行。Future instances由Executor.submit()创建。

```text
future.cancel():尝试取消调用.如果调用当前正在执行且无法取消，则该方法将返回False，否则调用将被取消，返回True

future.cancelled():成功取消，则返回True

future.running():调用正在执行或者无法取消，返回True

future.done():运行完毕或者成功取消，则返回True

future.result(timeout=None):返回调用的值。阻塞调用所在的线程，直到有结果可返回
如果在timeout时间内future没有运行完毕，将抛出TimeoutError异常。
如果调用在执行前被取消，则抛出CancelledError
如果调用触发了异常，则抛出同样的异常

future.exception(timeout=None):返回调用引发的异常

future.add_done_callback(fn):将可调用函数fn附加到future。当future被取消或者执行完毕，将调用fn，并将future作为传入参数
```

***

## Module Functions

* `concurrent.futures.wait(fs, timeout=None, return_when=ALL_COMPLETED)`

```text
concurrent.futures.wait(fs, timeout=None, return_when=ALL_COMPLETED)

ALL_COMPLETED:阻塞,直到线程池里面的所有任务都完成
FIRST_COMPLETED:阻塞,直到任何一个future完成或者取消
FIRST_EXCEPTION:阻塞,任何一个future抛出异常
```

Wait for the Future instances (possibly created by different Executor instances) given by fs to complete. 

返回一个 namedtuple，包含两个set(集合)，如下：

done：在执行 wait() 之前，已经完成或者取消的 future

not_done：未完成的 future。

```python
def worker(times):
    time.sleep(times)
    print('Get--', times)
    return times

l = [3, 1, 2]
with ThreadPoolExecutor(max_workers=2) as executor:
    tasts = [executor.submit(worker, i) for i in l]
	tast_result = wait(tasts, return_when=FIRST_COMPLETED)
	print(tast_result)
    print('main')
```

输出结果：

```text
Get-- 1
DoneAndNotDoneFutures(done={<Future at 0x25fec176e48 state=finished returned int>}, not_done={<Future at 0x25fec09e780 state=running>, <Future at 0x25fec17a358 state=running>})
main
Get-- 3
Get-- 2
```

* ` concurrent.futures.as_completed(fs, timeout=None)`

返回一个生成器，包含所有已完成的 future 对象。谁先完成就返回谁。fs 是 future 对象。

***

## Exception classes



***

参考：

[concurrent.futures — Launching parallel tasks](https://docs.python.org/3/library/concurrent.futures.html#threadpoolexecutor)



[使用future处理并发](https://juejin.im/post/5b24e19d6fb9a00e325e6d07)