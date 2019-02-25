通常在Python中我们进行并发编程一般都是使用多线程或者多进程来实现的，对于CPU计算密集型任务由于GIL的存在通常使用多进程来实现，而对于IO密集型任务可以通过线程调度来让线程在执行IO任务时让出GIL，从而实现表面上的并发。

其实对于IO密集型任务我们还有一种选择就是协程。协程，又称微线程，英文名Coroutine。Python中的异步IO模块asyncio就是基本的协程模块。可以多次进出、每次暂停并恢复执行的函数被称为协程。生成器只是一种简化的协程。

***

## 基于生成器的协程

基于生成器的协程（generator-based coroutine）特指使用 `yield from` 句法的协程。

1. 生成器用于生成供迭代的数据
2. 协程是数据的消费者

### yield/send

Python对协程的支持是通过generator实现的。

在generator中，我们不但可以通过`for`循环来迭代，还可以不断调用`next()`函数获取由`yield`语句返回的下一个值。但是Python的`yield`不但可以返回一个值，它还可以接收调用者发出的参数。当协程执行到yield关键字时，会暂停在那一行，等到主线程调用send方法发送了数据，协程才会接到数据继续执行。

每个生成器都可以执行send()方法，为生成器内部的yield语句发送数据。此时yield语句不再只是`yield xxxx`的形式，还可以是`var = yield xxxx`的赋值形式。它同时具备两个功能，一是暂停并返回函数，二是接收外部send()方法发送过来的值，重新激活函数，并将这个值赋值给var变量！

协程可以处于下面四个状态中的一个。当前状态可以导入inspect模块，使用inspect.getgeneratorstate(...) 方法查看，该方法会返回下述字符串中的一个。

- 'GEN_CREATED'　　等待开始执行。
- 'GEN_RUNNING'　　协程正在执行。
- 'GEN_SUSPENDED'  在yield表达式处暂停。
- 'GEN_CLOSED' 　　执行结束。

```python
>>> def coroutine():
...     reply = yield 'hello'
...     yield reply
... 

>>> c = coroutine()

>>> next(c)
'hello'

>>> c.send('world')
'world'
```

```python
def printer(): 
    counter = 0
    while True:
        string = str(yield)
        print('[{0}] {1}'.format(counter, string))
        counter += 1
 
if __name__ == '__main__':
    p = printer()
    next(p)
    p.send('Hi')
    p.send('My name is hsfzxjy.')
    p.send('Bye!')
```

仅当协程处于暂停状态时才能调用 send()方法，例如`my_coro.send(10)`。不过，如果协程还没激活（状态是`'GEN_CREATED'`），就立即把None之外的值发给它，会出现TypeError。因此，始终要先调用`next(my_coro)`激活协程（也可以调用`my_coro.send(None)`），这一过程被称作预激活。

```python
def consumer():
    r = ''
    while True:
        n = yield r		# 消费者通过yield接收生产者的消息，同时返给其结果
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'	# 消费者消费结果，下个循环返回给生产者

def produce(c):
    c.send(None)		# 启动消费者生成器，同时第一次接收返回结果
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)	# 向消费者发送消息并准备接收结果。此时会切换到消费者执行
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()			# 关闭消费者生成器

c = consumer()
produce(c)
```

注意到`consumer`函数是一个`generator`，把一个`consumer`传入`produce`后：

1. 首先调用`c.send(None)`启动生成器；
2. 然后，一旦生产了东西，通过`c.send(n)`切换到`consumer`执行；
3. `consumer`通过`yield`拿到消息，处理，又通过`yield`把结果传回；
4. `produce`拿到`consumer`处理的结果，继续生产下一条消息；
5. `produce`决定不生产了，通过`c.close()`关闭`consumer`，整个过程结束。

整个流程无锁，由一个线程执行，`produce`和`consumer`协作完成任务，所以称为“协程”，而非线程的抢占式多任务。

协程的特点在于是一个线程执行，那和多线程比，协程有何优势？

最大的优势就是协程极高的执行效率。协程不是被操作系统内核所管理，而完全是由程序所控制（也就是在用户态执行）。因为子程序切换不是线程切换，而是由程序自身控制，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显。

第二大优势就是不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。

因为协程是一个线程执行，那怎么利用多核CPU呢？最简单的方法是多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

### yield from

`yield from iterable`本质上等于`for item in iterable: yield item`的缩写版  。yield from 其实就是等待另外一个协程的返回。

专门的术语：

- 委派生成器 （delegating generator）- 包含 `yield from <iterable>` 表达式的生成器函数。
- 子生成器 （subgenerator）- 从 `<iterable>` 部分获取的生成器。
- 调用方 （caller）- 调用委派生成器的客户端代码。

同时，PEP 380 中对 `yield from` 的行为做了如下说明：

- 子生成器产出的值都直接传给委派生成器的调用方。
- 使用 `send()` 方法发给委派生成器的值都直接传给子生成器。如果发送的值是 `None` ，那么会调用子 生成器的 `__next__()` 方法。如果发送的值不是 `None` ，那么会调用子生成器的 `send()` 方法。如 果子生成器抛出 `StopIteration` 异常，那委派生成器恢复运行。任何其它异常都会向上冒泡，传给委派生 成器。
- 生成器退出时，生成器（或子生成器）中的 `return expr` 表达式会触发 `StopIteration(expr)` 异常 抛出。
- `yield from` 表达式的值是子生成器终止时传给 `StopIteration` 异常的第一个参数值。
- 传入委派生成器的异常，除了 `GeneratorExit` 之外都传给子生成器的 `throw()` 方法。如果调用 `throw()` 方法时抛出了 `StopIteration` 异常，委派生成器恢复运行。 `StopIteration` 之外的异 常会向上冒泡，传给委派生成器。
- 如果把 `GeneratorExit` 异常传入委派生成器，或者在委派生成器上调用 `close()` 方法，那么（委派 生成器）调用子生成器的 `close()` 方法（如果子生成器提供了该方法的话）。如果子生成器的 `close()` 方法导致异常抛出，那么异常会向上冒泡，传给委派生成器。如果子生成器的 `close()` 方法 正常执行的话，委派生成器随后向上抛出 `GeneratorExit` 异常。

***

## 原生协程

Python 3.5 又增加了用于创建原生协程的句法结构（见下面） `async def` ，以便从句法上和基于生成器的协程进行区分。同时，为了让两种生成器可以互相操作，Python 3.5 还提供了 `types.coroutine`装饰器函数对基于生成器的协程进行了包装。打算交给 *asyncio* 处理基于生成器的协程 **最好** 使用 `asyncio.coroutine` （Python 3.4）或者`types.coroutine` （Python 3.5）进行装饰，这样可以把协程函数凸显出来，也有助于调试

原生协程（native coroutine）有如下关键特征：

- `async def` functions are always coroutines, even if they do not contain `await` expressions.

- It is a `SyntaxError` to have `yield` or `yield from` expressions in an `async def` function.

- Internally, two new code object flags were introduced:

  > - `CO_COROUTINE` is used to mark *native coroutines* (defined with new syntax).
  > - `CO_ITERABLE_COROUTINE` is used to make *generator-based coroutines* compatible with *native coroutines*.

- Regular generators, when called, return a *generator object*; similarly, coroutines return a *coroutine object*.

- `StopIteration` exceptions are not propagated out of coroutines, and are replaced with a `RuntimeError`. For regular generators such behavior requires a future import (see PEP 479).

- When a *native coroutine* is garbage collected, a `RuntimeWarning` is raised if it was never awaited on (see also Debugging Features).

原生协程（native coroutine）和 基于生成器的协程（generator-based coroutine）有如下区别：

- 原生协程对象并未实现 `__iter__` 和 `__next__` 方法，所以它们并不能用于需要可迭代对象的场景。
- 普通生成器函数内部不能使用 `yield from 原生协程` 句法。
- 基于生成器的协程函数（代码用于 *asyncio* 时，需用 `asyncio.coroutine` ）内部可以使用 `yield from 原生协程` 句法。
- `inspect.isgenerator()` 函数和 `inspect.isgeneratorfunction()` 函数作用于原生协程时，返回 `False` 。

用`asyncio`的异步网络连接来获取sina、sohu和163的网站首页：

```python
import asyncio

@asyncio.coroutine
def wget(host):
    print('wget %s...' % host)
    connect = asyncio.open_connection(host, 80)
    reader, writer = yield from connect
    header = 'GET / HTTP/1.0\r\nHost: %s\r\n\r\n' % host
    writer.write(header.encode('utf-8'))
    yield from writer.drain()
    while True:
        line = yield from reader.readline()
        if line == b'\r\n':
            break
        print('%s header > %s' % (host, line.decode('utf-8').rstrip()))
    # Ignore the body, close the socket
    writer.close()

loop = asyncio.get_event_loop()
tasks = [wget(host) for host in ['www.sina.com.cn', 'www.sohu.com', 'www.163.com']]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

改写：

```python
import asyncio

async def wget(host):
    print('wget %s...' % host)
    connect = asyncio.open_connection(host, 80)
    reader, writer = await connect
    header = 'GET / HTTP/1.0\r\nHost: %s\r\n\r\n' % host
    writer.write(header.encode('utf-8'))
    await writer.drain()
    while True:
        line = await reader.readline()
        if line == b'\r\n':
            break
        print('%s header > %s' % (host, line.decode('utf-8').rstrip()))
    # Ignore the body, close the socket
    writer.close()

loop = asyncio.get_event_loop()
tasks = [wget(host) for host in ['www.sina.com.cn', 'www.sohu.com', 'www.163.com']]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```



**清晰优雅的协程可以说实现异步的最优方案之一**

***

参考：

https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432090171191d05dae6e129940518d1d6cf6eeaaa969000

https://zhuanlan.zhihu.com/p/25228075

https://ialloc.org/blog/into-python3-asyncio/