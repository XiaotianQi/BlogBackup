## 单线程模型

单线程在程序执行时，只在一个线程上运行，所有程序按需一次执行。而且，同时只能执行一个任务，其他任务都必须在后面排队等待。

注意，Python只在一个线程上运行，不代表 Python引擎只有一个线程。事实上，Python引擎有多个线程，单个脚本只能在一个线程上运行（称为主线程），其他线程都是在后台配合。Python之所以采用单线程，而不是多线程，跟历史有关系。多线程需要共享资源、且有可能修改彼此的运行结果。

这种模式的好处是实现起来比较简单，执行环境相对单纯；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。

如果排队是因为计算量大，CPU 忙不过来，倒也算了，但是很多时候 CPU 是闲着的，因为 IO 操作（输入输出）很慢，不得不等着结果出来，再往下执行。Python语言的设计者意识到，这时 CPU 完全可以不管 IO 操作，挂起处于等待中的任务，先运行排在后面的任务。等到 IO 操作返回了结果，再回过头，把挂起的任务继续执行下去。这种机制就是 Python内部采用的“事件循环”机制（Event Loop）。

## 同步任务和异步任务

程序里面所有的任务，可以分成两类：同步任务（synchronous）和异步任务（asynchronous）。

* 同步任务是那些没有被引擎挂起、在主线程上排队执行的任务。只有前一个任务执行完毕，才能执行后一个任务。
* 异步任务是那些被引擎放在一边，不进入主线程、而进入任务队列的任务。只有引擎认为某个异步任务可以执行了（比如 IO 操作结束），该任务（采用回调函数的形式）才会进入主线程执行。排在异步任务后面的代码，不用等待异步任务结束会马上运行，也就是说，异步任务不具有“堵塞”效应。

举例来说，IO 操作可以当作同步任务处理，也可以当作异步任务处理，由开发者决定。如果是同步任务，主线程就等着 IO 操作返回结果，再往下执行；如果是异步任务，主线程在发出 IO 请求以后，就直接往下执行，等到 IO 操作有了结果，主线程再执行对应的回调函数。

## 任务队列和事件循环

Python 运行时，除了一个正在运行的主线程，引擎还提供一个任务队列（task queue），里面是各种需要当前程序处理的异步任务。（实际上，根据异步任务的类型，存在多个任务队列。为了方便理解，这里假设只存在一个队列。）

首先，主线程会去执行所有的同步任务。等到同步任务全部执行完，就会去看任务队列里面的异步任务。如果满足条件，那么异步任务就重新进入主线程开始执行，这时它就变成同步任务了。等到执行完，下一个异步任务再进入主线程开始执行。一旦任务队列清空，程序就结束执行。

异步任务的写法通常是回调函数。一旦异步任务重新进入主线程，就会执行对应的回调函数。如果一个异步任务没有回调函数，就不会进入任务队列，也就是说，不会重新进入主线程，因为没有用回调函数指定下一步的操作。

Python 引擎怎么知道异步任务有没有结果，能不能进入主线程呢？答案就是引擎在不停地检查，一遍又一遍，只要同步任务执行完了，引擎就会去检查那些挂起来的异步任务，是不是可以进入主线程了。这种循环检查的机制，就叫做事件循环（Event Loop）。维基百科的定义是：“事件循环是一个程序结构，用于等待和发送消息和事件（a programming construct that waits for and dispatches events or messages in a program）”。

***

## asyncio 

多线程有"线程竞争"的问题，处理起来很复杂，还涉及加锁。对于简单的异步任务来说（比如与网页互动），写起来很麻烦。

* 多线程：抢占式多任务
* 协程：合作式多任务

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%20asyncio%201.jpg)

Python 3.4 引入了 `asyncio` 模块，增加了异步编程，大大方便了异步任务的处理。

`asyncio` 模块最大特点就是，只存在一个线程。

由于只有一个线程，就不可能多个任务同时运行。asyncio 是"多任务合作"模式（cooperative multitasking），允许异步任务交出执行权给其他任务，等到其他任务完成，再收回执行权继续往下执行。

由于代码的执行权在多个任务之间交换，所以看上去好像多个任务同时运行，其实底层只有一个线程，多个任务分享运行时间。

表面上，这是一个不合理的设计，明明有多线程多进程的能力，为什么放着多余的 CPU 核心不用，而只用一个线程呢？但是就像前面说的，单线程简化了很多问题，使得代码逻辑变得简单，写法符合直觉。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%20asyncio%202.jpg)

asyncio 模块在单线程上启动一个事件循环（event loop），时刻监听新进入循环的事件，加以处理，并不断重复这个过程，直到异步任务结束。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%20asyncio%203.jpg)

***

## asyncio API

`asyncio` 模块最主要的几个API。注意，必须使用 Python 3.7 或更高版本，早期的语法已经变了。

函数前面加上 `async` 关键字，就变成了 async 函数。这种函数最大特点是执行可以暂停，交出执行权。

```python
async def main():
```

在 async 函数内部的异步任务前面，加上`await`命令。

```python
await asyncio.sleep(1)
```

上面代码中，`asyncio.sleep(1)` 方法可以生成一个异步任务，休眠1秒钟然后结束。

执行引擎遇到`await`命令，就会在异步任务开始执行之后，暂停当前 async 函数的执行，把执行权交给其他任务。等到异步任务结束，再把执行权交回 async 函数，继续往下执行。

`async.run()` 方法加载 async 函数，启动事件循环。

```python
asyncio.run(main())
```

上面代码中，`asyncio.run()` 在事件循环上监听 async 函数`main`的执行。等到 `main` 执行完了，事件循环才会终止。

示例：

* 同步版

```python
import time

def count():
    print("One")
    time.sleep(1)
    print("Two")

def main():
    for _ in range(3):
        count()

main()
```

输出如下：

```python
One
Two
One
Two
One
Two
```

运行结果的原因是，三个 `count()` 都是同步执行，必须等到前一个执行完，才能执行后一个。脚本总的运行时间是3秒。

* async

```python
import asyncio

async def count():
    print("One")
    await asyncio.sleep(1)
    print("Two")

async def main():
    await asyncio.gather(count(), count(), count())

asyncio.run(main())
```

输出如下：

```python
One
One
One
Two
Two
Two
```

在 async 函数`main`的里面，`asyncio.gather()` 方法将多个异步任务（三个 `count()`）包装成一个新的异步任务，必须等到内部的多个异步任务都执行结束，这个新的异步任务才会结束。上面运行结果的原因是，三个 `count()` 依次执行，打印完 `One`，就休眠1秒钟，把执行权交给下一个 `count()`，所以先连续打印出三个 `One`。等到1秒钟休眠结束，执行权重新交回第一个 `count()`，开始执行 `await` 命令下一行的语句，所以会接着打印出三个`Two`。脚本总的运行时间是1秒。

***

## 示例

网页截图脚本`screenshot.py`

```python
# coding=utf-8
import asyncio
from pyppeteer import launch

async def main():
    browser = await launch()
    page = await browser.newPage()
    await page.goto('http://example.com')
    await page.screenshot({'path': 'example.png'})
    await browser.close()

asyncio.run(main())
```

上面代码中，启动浏览器（`launch`）、打开新 Tab（`newPage()`）、访问网址（`page.goto()`）、截图（`page.screenshot()`）、关闭浏览器（`browser.close()`），这一系列操作都是异步任务，使用 `await` 命令写起来非常自然简单。

***

参考：

[异步操作概述](https://wangdoc.com/javascript/async/general.html)，阮一峰