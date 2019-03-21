`asyncio`是Python 3.4 试验性引入的异步I/O框架（PEP 3156），提供了基于协程做异步I/O编写单线程并发代码的基础设施。其核心组件有事件循环（Event Loop）、协程(Coroutine）、任务(Task)、未来对象(Future)以及其他一些扩充和辅助性质的模块。`asyncio`的编程模型就是一个消息循环。我们从`asyncio`模块中直接获取一个`EventLoop`的引用，然后把需要执行的协程扔到`EventLoop`中执行

如果 Python 中实现过多线程或多处理，您可能想知道它们与这种 `asyncio`协作式多任务方法有何不同。主要区别在于，在 `asyncio`方法中，不会实际尝试让两个协程在同一时间做一些事情。`asyncio`事件循环所做的是利用任务内的自然停机时间，允许协程在有工作要做时执行工作，但在其他协程空闲时将控制权转交给它们。

协程无法控制它再次运行的时间，此方法称为协作式多任务是有原因的。如果某个协程花了太长时间而没有将控制权转交回事件循环，它会阻塞一切操作，导致不必要的延迟，而且您将失去多任务的优势。这意味着您首先必须确保您的程序适合用这种方式实现，然后必须小心地编写程序代码，将它分解为会在恰当时机相互释放控制权的协程。这可能比听起来还要复杂，因为您可能无意识地从某个协程调用常规函数，这会花费很长时间，而且问题不会很明显。

一般来说，`asyncio`  事件循环最适合经常联网的程序，或大量查询数据库和类似对象的程序。等待远程服务器或数据库响应请求或查询时，是将控制权释放给事件循环的理想时刻。在过去，程序员倾向于在这种情况下使用线程，但与多线程相比，`asyncio` 事件循环是一种更加清晰和灵活的编程方式。一个难点是，要充分发挥 `asyncio` 事件循环的优势，需要在  `asyncio` 协程中对网络和数据库 API 进行编码。幸运的是，现在许多 Python 第三方库已实现了对 `asyncio` 的充分利用。

asyncio 是用来编写 并发 代码的库，使用 async/await 语法。asyncio 被用作多个提供高性能 Python 异步框架的基础，包括网络和网站服务，数据库连接库，分布式任务队列等等。asyncio 往往是构建 IO 密集型和高层级 结构化 网络代码的最佳选择。

关于asyncio的一些关键字的说明：

- coroutine 协程：协程对象，指一个使用async关键字定义的函数，它的调用不会立即执行函数，而是会返回一个协程对象。
- task 任务：一个协程对象就是一个原生可以挂起的函数，任务则是对协程进一步封装，其中包含了任务的各种状态
- future: 代表将来执行或没有执行的任务的结果。它和task上没有本质上的区别
- event_loop 事件循环：创建一个事件循环来调度和管理各个协程

***

## 简单的协程示例

下代码段会在等待 1 秒后打印 "hello"，然后 再次 等待 2 秒后打印 "world"。

要真正运行一个协程，asyncio 提供了三种主要机制:

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    print(f"started at {time.strftime('%X')}")

    await say_after(1, 'hello')
    await say_after(2, 'world')

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```

通过 async/await 语法进行声明协程，是编写异步应用的推荐方式。

1. `asyncio.run()` 函数用来运行最高层级的入口点 "main()" 函数
2. `asyncio.sleep(delay)` 等待一个协程

修改以上示例，并发运行两个 `say_after` 协程:

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    task1 = asyncio.create_task(
        say_after(1, 'hello'))

    task2 = asyncio.create_task(
        say_after(2, 'world'))

    print(f"started at {time.strftime('%X')}")

    # Wait until both tasks are completed (should take
    # around 2 seconds.)
    await task1
    await task2

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```

3. `asyncio.create_task()` 函数用来并发运行 asyncio Tasks。

***

## Awaitables 可等待对象

如果一个对象可以在 `await`  语句中使用，那么它就是 **可等待** 对象。许多 asyncio API 都被设计为接受可等待对象。

*可等待* 对象有三种主要类型: **coroutines**, **Tasks**, and **Futures**.

有几个状态：

- Pending
- Running
- Done
- Cancelled

创建的时候，task为pending，事件循环调用执行的时候当然就是running，调用完毕自然就是done，如果需要停止事件循环，就需要先把task取消。

### coroutines 协程

*协程函数*: 定义形式为 `async def` 的函数;

*协程对象*: 调用 *协程函数* 所返回的对象，协程对象不能直接运行

对基于生成器的协程的支持计划在 Python 3.10 中移除。基于生成器的协程是 async/await 语法的前身。它们是使用 `yield from` 语句创建的 Python 生成器，可以等待 Future 和其他协程。基于生成器的协程应该使用 `@asyncio.coroutine` 装饰，虽然这并非强制。

Functions defined with `async def` syntax are always coroutine functions,even if they do not contain `await` or `async` keywords.

`await`Suspend the execution of coroutine on an awaitable object. Can only be used inside a coroutine function.其参数必须是一个Awaitable对象，或者实现了相应协议的对象

```python
async def func(param1, param2):
    do_stuff()
    await some_coroutine()
```

The `async for` statement

An asynchronous iterable is able to call asynchronous code in its iter implementation, and asynchronous iterator can call asynchronous code in its next method.

The `async for` statement allows convenient iteration over asynchronous iterators.

The following code:

```python
async for TARGET in ITER:
    BLOCK
else:
    BLOCK2
```

It is a `SyntaxError` to use an `async for` statement outside the body of a coroutine function.

The `async with` statement

An asynchronous context manager is a context manager that is able to suspend execution in its *enter* and *exit* methods.

The following code:

```python
async with EXPR as VAR:
    BLOCK
```

It is a `SyntaxError` to use an `async with` statement outside the body of a coroutine function.

### Task 对象

*任务* 被用来设置日程以便 *并发* 执行协程。

当协程程序通过 `asyncio.create_task()` 等函数被打包为一个 *任务*，该协程将自动排入日程准备立即运行。

```text
class asyncio.Task(coro, *, loop=None)

A Future-like object that runs a Python coroutine. Not thread-safe.
Task 对象被用来在事件循环中运行协程。如果一个协程在等待一个 Future 对象，Task 对象会挂起该协程的执行并等待该 Future 对象完成。当该 Future 对象 完成，被打包的协程将恢复执行。

使用高层级的 asyncio.create_task() 函数来创建 Task 对象，也可用低层级的 loop.create_task() 或 ensure_future() 函数。不建议手动实例化 Task 对象。

asyncio.Task 从 Future 继承了其除 Future.set_result() 和 Future.set_exception() 以外的所有 API。是Future子类。

Task 对象支持 contextvars 模块。当一个 Task 对象被创建，它将复制当前上下文，然后在复制的上下文中运行其协程。
```

创建 Task 对象：

```text
asyncio.create_task(coro)
Wrap the coro coroutine into a Task and schedule its execution. Return the Task object.
The task is executed in the loop returned by get_running_loop(), RuntimeError is raised if there is no running loop in current thread.

```

Task 对象方法：

```text
task.cancel()
请求取消 Task 对象。
这将安排在下一轮事件循环中,抛出一个 CancelledError 异常给被封包的协程。
协程在之后有机会进行清理甚至使用 try ... ... except CancelledError ... finally 代码块抑制异常来拒绝请求。不同于 Future.cancel()，Task.cancel() 不保证 Task 会被取消，虽然抑制完全取消并不常见，也很不鼓励这样做。

task.cancelled()
如果 Task 对象被取消,则返回 True。
当使用 cancel() 发出取消请求时 Task 会被取消，其封包的协程将传播被抛入的 CancelledError 异常。如果取消期间一个协程正在等待一个 Future 对象，该 Future 对象也将被取消。可被用来检测 Task 对象是否被取消。如果打包的协程没有抑制 CancelledError 异常并且确实被取消，该方法将返回 True。

task.done()
如果 Task 对象已完成，则返回 True。
当 Task 所封包的协程返回一个值、引发一个异常或 Task 本身被取消时，则会被认为已完成。

task.result()
返回 Task 的结果。
如果 Task 对象已完成，其封包的协程的结果会被返回 (或者当协程引发异常时，该异常会被重新引发。)
如果 Task 对象被取消，此方法会引发一个 CancelledError 异常。
如果 Task 对象的结果还不可用，此方法会引发一个 InvalidStateError 异常。

task.exception()
返回 Task 对象的异常。
如果所封包的协程引发了一个异常，该异常将被返回。如果所封包的协程正常返回则该方法将返回 None。
如果 Task 对象被取消，此方法会引发一个 CancelledError 异常。
如果 Task 对象尚未完成，此方法将引发一个 InvalidStateError 异常。

task.add_done_callback(callback, *, context=None)
添加一个回调，将在 Task 对象完成时,运行。
此方法应该仅在低层级的基于回调的代码中使用。
要了解更多细节请查看 Future.add_done_callback() 的文档。

task.remove_done_callback(callback)
从回调列表中移除 callback 指定的回调。
此方法应该仅在低层级的基于回调的代码中使用。
要了解更多细节请查看 Future.remove_done_callback() 的文档。

task.get_stack(*, limit=None)
返回此 Task 对象的栈框架列表。
如果所封包的协程未完成，这将返回其挂起所在的栈。如果协程已成功完成或被取消，这将返回一个空列表。如果协程被一个异常终止，这将返回回溯框架列表。
框架总是从按从旧到新排序。
每个被挂起的协程只返回一个栈框架。
可选的 limit 参数指定返回框架的数量上限；默认返回所有框架。返回列表的顺序要看是返回一个栈还是一个回溯：栈返回最新的框架，回溯返回最旧的框架。(这与 traceback 模块的行为保持一致。)

task.print_stack(*, limit=None, file=None)
打印此 Task 对象的栈或回溯。
此方法产生的输出类似于 traceback 模块通过 get_stack() 所获取的框架。
limit 参数会直接传递给 get_stack()。
file 参数是输出所写入的 I/O 流；默认情况下输出会写入 sys.stderr。

task.classmethod all_tasks(loop=None)
返回一个事件循环中所有任务的集合。
默认情况下将返回当前事件循环中所有任务。如果 loop 为 None，则会使用 get_event_loop() 函数来获取当前事件循环。
此方法 已弃用 并将在 Python 3.9 中移除。请改用 asyncio.all_tasks() 函数。

task.classmethod current_task(loop=None)
返回当前运行任务或 None。
如果 loop 为 None，则会使用 get_event_loop() 函数来获取当前事件循环。
此方法 已弃用 并将在 Python 3.9 中移除。请改用 asyncio.current_task() 函数。
```

以下示例演示了协程是如何拦截取消请求的:

```python
async def cancel_me():
    print('cancel_me(): before sleep')

    try:
        # Wait for 1 hour
        await asyncio.sleep(3600)
    except asyncio.CancelledError:
        print('cancel_me(): cancel sleep')
        raise
    finally:
        print('cancel_me(): after sleep')

async def main():
    # Create a "cancel_me" Task
    task = asyncio.create_task(cancel_me())

    # Wait for 1 second
    await asyncio.sleep(1)

    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("main(): cancel_me is cancelled now")

asyncio.run(main())

# Expected output:
#
#     cancel_me(): before sleep
#     cancel_me(): cancel sleep
#     cancel_me(): after sleep
#     main(): cancel_me is cancelled now
```

### Future 对象

`Future` 是一种特殊的 **低层级** 可等待对象，表示一个异步操作的 **最终结果**。

当一个 Future 对象 *被等待*，这意味着协程将保持等待直到该 Future 对象在其他地方操作完毕。

在 asyncio 中需要 Future 对象以便允许通过 async/await 使用基于回调的代码。

创建Future 对象：

```text
asyncio.ensure_future(obj, *, loop=None)
Return:
    obj argument as is, if obj is a Future, a Task, or a Future-like object (isfuture() is used for the test.)
    a Task object wrapping obj, if obj is a coroutine (iscoroutine() is used for the test.)
    a Task object that would await on obj, if obj is an awaitable (inspect.isawaitable() is used for the test.)
If obj is neither of the above a TypeError is raised.

asyncio.isfuture(obj)
当obj是以下三种类型之一，则返回True:
	an instance of asyncio.Future,
	an instance of asyncio.Task,
	a Future-like object with a _asyncio_future_blocking attribute.

asyncio.wrap_future(future, *, loop=None)
Wrap a concurrent.futures.Future object in a asyncio.Future object.
```

Future 对象相关方法：

```text
class asyncio.Future(*, loop=None)
Future表示异步操作的最终结果。不是线程安全的。
Future是一个等待的对象。协同程序可以等待Future obj，直到它们有结果、抛出异常或被取消。
Future objects are used to bridge low-level callback-based code with high-level async/await code.
推荐的创建方法是loop.create_future()。This way alternative event loop implementations can inject their own optimized implementations of a Future object.

future.add_done_callback(callback, *, context=None)
添加一个回调函数，以便在future完成以后运行。

future.remove_done_callback(callback)
删除一个回调函数

future.result()
返回 Future 的结果.
If the Future is done and has a result set by the set_result() method, the result value is returned.
If the Future is done and has an exception set by the set_exception() method, this method raises the exception.
If the Future has been cancelled, this method raises a CancelledError exception.
If the Future’s result isn’t yet available, this method raises a InvalidStateError exception.

future.set_result(result)
将future标记为已完成，并设定其结果。

future.set_exception(exception)
将future标记为已完成，并设定一个异常。

future.done()
如果完成，返回True。
A Future is done if it was cancelled or if it has a result or an exception set with set_result() or set_exception() calls.

future.cancel()
Cancel the Future and schedule callbacks.
If the Future is already done or cancelled, return False. Otherwise, change the Future’s state to cancelled, schedule the callbacks, and return True.

future.cancelled()
如果被取消，返回True
The method is usually used to check if a Future is not cancelled before setting a result or an exception for it

future.get_loop()
Return the event loop the Future object is bound to.
```

***

## Event Loop 事件循环

Event Loop 是一个很重要的概念，指的是计算机系统的一种运行机制。Event Loop是一个程序结构，用于等待和发送消息和事件。简单说，就是在程序中设置两个线程：一个负责程序本身的运行，称为"主线程"；另一个负责主线程与其他进程（主要是各种I/O操作）的通信，这个等待事件通知的循环线程，被称为"Event Loop线程"（可以译为"消息线程"）。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%20Event%20loop.png)

上图主线程的绿色部分，还是表示运行时间，而橙色部分表示空闲时间。每当遇到I/O的时候，主线程就让Event 
Loop线程去通知相应的I/O程序，然后接着往后运行，所以不存在红色的等待时间。等到I/O程序完成操作，Event 
Loop线程再把结果返回主线程。主线程就调用事先设定的回调函数，完成整个任务。

可以看到，由于多出了橙色的空闲时间，所以主线程得以运行更多的任务，这就提高了效率。这种运行方式称为"异步模式"（asynchronous I/O）或"非堵塞模式"（non-blocking mode）。

事件循环使用协同日程调度: 一个事件循环每次运行一个 Task 对象。而一个 Task 对象会等待一个 Future 对象完成，该事件循环会运行其他 Task、回调或执行 IO 操作。

以餐馆就餐为例

```python
import random
import asyncio
 
async def get_menus(): 
    delay_minutes = random.randrange(3) #0 to 3 minutes
    await asyncio.sleep(delay_minutes) #Pretend a second is a minute
 
async def get_order(): 
    delay_minutes = random.randrange(10)
    await asyncio.sleep(delay_minutes)
    order = random.choice(['Special of the day', 'Fish & Chips', 'Pasta'])
    return order
 
async def prepare_order(order): 
    delay_minutes = random.randrange(10, 20) #10 to 20 minutes
    await asyncio.sleep(delay_minutes)
    print('   [Order ready from kitchen: ', order, ']')
 
async def eat(): 
    delay_minutes = random.randrange(20, 40)
    await asyncio.sleep(delay_minutes)
 
async def get_payment(): 
    delay_minutes = random.randrange(10)
    await asyncio.sleep(delay_minutes)
 
async def progress_indicator(delay, loop):  # 进度指示器 
    while True: 
        try: 
            await asyncio.sleep(delay)
        except asyncio.CancelledError: 
            break
        #Print a dot, with no newline afterward & force the output to appear immediately
        print('.', end='', flush=True)
        #Check if this is the last remaining task, and exit if so
        num_active_tasks = [ task for task in asyncio.Task.all_tasks(loop)
                                  if not task.done() ]
        if len(num_active_tasks) == 1: 
            break
 
async def serve_table(table_number): 
    await get_menus()
    print('Welcome.Please sit at table', table_number, 'Here are your menus')
    order = await get_order()
    print('Table', table_number, 'what will you be having today?')
    await prepare_order(order)
    print('Table', table_number, 'here is your meal:', order)
    await eat()
    print('Table', table_number, 'here is your check')
    await get_payment()
    print('Thanks for visiting us! (table', table_number, ')')
 
#asyncio uses event loops to manage its operation
loop = asyncio.get_event_loop()
 
#Create coroutines for three tables
gathered_coroutines = asyncio.gather(
    serve_table(1),
    serve_table(2),
    serve_table(3),
    progress_indicator(0.5, loop)
)
 
#This is the entry from synchronous to asynchronous code.It will block
#Until the coroutine passed in has completed
loop.run_until_complete(gathered_coroutines)
#We're done with the event loop
loop.close()
```

创建 loop：

```text
asyncio.get_event_loop()
获取当前事件循环。如果当前操作系统线程中没有当前事件循环集，且尚未调用asyncio.set_event_loop()，将创建一个新的事件循环并将其设置为当前事件循环。
还可以考虑使用asyncio.run()函数，而不是使用低级函数手动创建和关闭事件循环。

asyncio.get_running_loop()
返回当前OS线程中正在运行的事件循环。
如果没有正在运行的事件循环，则会引发RuntimeError

asyncio.set_event_loop(loop)
将循环设置为当前OS线程的当前事件循环。

asyncio.new_event_loop()
创建一个新的事件循环对象。
```

loop 常用方法：

```text
loop.run_until_complete(future)
一个阻塞（blocking）调用，直到协程运行结束，它才返回。
参数是一个 future，但是我们这传给它协程对象，之所以能这样，是因为它在内部做了检查，通过 ensure_future 函数把协程对象包装（wrap）成了 future。

loop.run_forever()
运行事件循环，直到调用stop()
If stop() is called before run_forever() is called, the loop will poll the I/O selector once with a timeout of zero, run all callbacks scheduled in response to I/O events (and those that were already scheduled), and then exit.
If stop() is called while run_forever() is running, the loop will run the current batch of callbacks and then exit. Note that new callbacks scheduled by callbacks will not run in this case; instead, they will run the next time run_forever() or run_until_complete() is called.

loop.create_future()
Create an asyncio.Future object attached to the event loop.
推荐方法

loop.create_task(coro)
Schedule the execution of a Coroutines. Return a Task object.

loop.stop()
Stop the event loop.


loop.is_running()
Return True if the event loop is currently running.

loop.is_closed()
Return True if the event loop was closed.

loop.close()
Close the event loop.
The loop must not be running when this function is called. Any pending callbacks will be discarded.
This method clears all queues and shuts down the executor, but does not wait for the executor to finish.
This method is idempotent and irreversible. No other methods should be called after the event loop is closed.


loop.time()
Return the current time, as a float value, according to the event loop’s internal monotonic clock.

```

`run_until_complete()`内置`add_done_callback`回调函数，`run_forever()`则可以自定义`add_done_callback()`，具体差异请看下面两个例子。

```python
import asyncio

async def func(future):
    await asyncio.sleep(1)
    future.set_result('Future is done!')

if __name__ == '__main__':

    loop = asyncio.get_event_loop()
    future = asyncio.Future()
    asyncio.ensure_future(func(future))
    loop.run_until_complete(future)
    print(future.result())
    loop.close()
```

```python
import asyncio

async def func(future):
    await asyncio.sleep(1)
    future.set_result('Future is done!')

def call_result(future):
    print(future.result())
    loop.stop()		# 在回调函数中，停止loop												

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    future = asyncio.Future()
    asyncio.ensure_future(func(future))
    future.add_done_callback(call_result)        # 注意这行
    try:
        loop.run_forever()
    finally:
        loop.close()
```

***

##  asyncio 方法

asyncio的使用可分三步走：

1. 创建事件循环
2. 指定循环模式并运行
3. 关闭循环

### 创建

#### 创建 coroutines

在Python3.5中，引入了aync&await 语法结构，通过"aync def"可以定义一个协程代码片段，作用类似于Python3.4中的@asyncio.coroutine修饰符，而await则相当于"yield from"。

Functions defined with `async def` syntax are always coroutine functions,even if they do not contain `await` or `async` keywords.

`await`Suspend the execution of coroutine on an awaitable object. Can only be used inside a coroutine function.其参数必须是一个Awaitable对象，或者实现了相应协议的对象

```python
async def func(param1, param2):
    do_stuff()
    await some_coroutine()
```

以下调用方式，将会产生死循环。

```python
async def func1():
    await func2()

async def func2():
    await func1

asyncio.run(func1())
```

首先执行 func1，在func1中await func2处等待，并调用func2

然后调用 func2，在func2中await func1处等待，并调用func1

但，此时，func1不会从await func2等待处运行，而是重新运行func1

由此，导致死循环。

async_generator 不能够在 'await' 语句中使用，可以在 async for i in async_generator() 这样的循环语句中使用。

```python
import asyncio

async def consumer():
    r = ''
    async for n in producer():
    # while True:
        # n = await producer()  # async_generator can't be used in 'await' expression
        if not n:
            break
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

async def producer():			
    n = 0
    while True:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        if n == 5:
            break
        yield n					# async_generator

asyncio.run(consumer())
```

同时，It is a `SyntaxError` to use a `yield from` expression inside the bodyof a coroutine function.

#### 创建 future

```text
asyncio.ensure_future(obj, *, loop=None)
Return:
    obj argument as is, if obj is a Future, a Task, or a Future-like object (isfuture() is used for the test.)
    a Task object wrapping obj, if obj is a coroutine (iscoroutine() is used for the test.)
    a Task object that would await on obj, if obj is an awaitable (inspect.isawaitable() is used for the test.)
If obj is neither of the above a TypeError is raised.

asyncio.isfuture(obj)
当obj是以下三种类型之一，则返回True:
	an instance of asyncio.Future,
	an instance of asyncio.Task,
	a Future-like object with a _asyncio_future_blocking attribute.
```

#### 创建 task

```text
asyncio.create_task(coro)
将传入的协程打包为一个 Task 排入日程准备执行。返回 Task 对象,是Future子类
该任务会在 get_running_loop() 返回的循环中执行，如果当前线程没有在运行的循环则会引发 RuntimeError。
与asyncio.ensure_future(obj, *, loop=None)相比，创建新任务的首选方法。

asyncio.ensure_future(coroutine)
该函数接受任何可等待的对象。
```

```bash
async def coro():
    ...

# In Python 3.7+
task = asyncio.create_task(coro())
...

# This works in all Python versions but is less readable
task = asyncio.ensure_future(coro())
```

#### 创建 loop

```text
asyncio.get_event_loop()
获取当前事件循环。如果当前操作系统线程中没有当前事件循环集，且尚未调用asyncio.set_event_loop()，将创建一个新的事件循环并将其设置为当前事件循环。
还可以考虑使用asyncio.run()函数，而不是使用低级函数手动创建和关闭事件循环。

asyncio.get_running_loop()
返回当前OS线程中正在运行的事件循环。
如果没有正在运行的事件循环，则会引发RuntimeError

asyncio.set_event_loop(loop)
将循环设置为当前OS线程的当前事件循环。

asyncio.new_event_loop()
创建一个新的事件循环对象。
```

### 运行

#### 运行

```text
asyncio.run(coro, *, debug=False)

运行传入的协程，managing the asyncio event loop and finalizing asynchronous generators.
当在同一线程中，有其他 asyncio 事件循环运行时，此函数不能被调用。
如果 debug 为 True，事件循环将以调试模式运行。
此函数总是会创建一个新的事件循环并在结束时关闭之。它应当被用作 asyncio 程序的主入口点，理想情况下应当只被调用一次。
```

#### 运行 loop

```text
loop.run_until_complete(future)
一个阻塞（blocking）调用，直到协程运行结束，它才返回。
参数是一个 future，但是我们这传给它协程对象，之所以能这样，是因为它在内部做了检查，通过 ensure_future 函数把协程对象包装（wrap）成了 future。

loop.run_forever()
运行事件循环，直到调用stop()
```

#### Running Tasks Concurrently

```text
awaitable asyncio.gather(*aws, loop=None, return_exceptions=False)

并发 运行 aws 序列中的 可等待对象。
如果 aws 中的某个可等待对象为协程，它将自动作为一个任务加入日程。
如果所有可等待对象都成功完成，结果将是一个由所有返回值聚合而成的列表。结果值的顺序与 aws 中可等待对象的顺序一致。
如果 return_exceptions 为 False (默认)，所引发的首个异常会立即传播给等待 gather() 的任务。aws 序列中的其他可等待对象 不会被取消 并将继续运行。
如果 return_exceptions 为 True，异常会和成功的结果一样处理，并聚合至结果列表。
如果 gather() 被取消，所有被提交 (尚未完成) 的可等待对象也会 被取消。
如果 aws 序列中的任一 Task 或 Future 对象 被取消，它将被当作引发了 CancelledError 一样处理 -- 在此情况下 gather() 调用 不会 被取消。这是为了防止一个已提交的 Task/Future 被取消导致其他 Tasks/Future 也被取消。
```

```python
import asyncio

async def factorial(name, number):
    f = 1
    for i in range(2, number + 1):
        print(f"Task {name}: Compute factorial({i})...")
        await asyncio.sleep(1)
        f *= i
    print(f"Task {name}: factorial({number}) = {f}")

async def main():
    # Schedule three calls *concurrently*:
    await asyncio.gather(
        factorial("A", 2),
        factorial("B", 3),
        factorial("C", 4),
    )

asyncio.run(main())
```

#### Waiting Primitives

##### asyncio.wait()

```text
coroutine asyncio.wait(aws, *, loop=None, timeout=None, return_when=ALL_COMPLETED)

并发运行 aws 中可等待对象，并阻塞线程直到满足 return_when 指定的条件。
If any awaitable in aws is a coroutine, it is automatically scheduled as a Task. 
Returns two sets of Tasks/Futures: (done, pending).
如指定 timeout (float 或 int 类型)，则控制返回前等待的最长秒数。
请注意此函数不会引发 asyncio.TimeoutError。当超时发生时,未完成的 Future 或 Task 将在指定秒数后被返回。
return_when 指定此函数应在何时返回。它必须为以下常数之一:
FIRST_COMPLETED:任意可等待对象执行结束或被取消时返回
FIRST_EXCEPTION:任意可等待对象因引发异常时返回,当没有引发任何异常时它就相当于ALL_COMPLETED。
ALL_COMPLETED:所有可等待对象执行结束或取消时返回
```

与 `wait_for()` 在超时发生时不会取消可等待对象。直接向 `wait()` 传入协程对象的方式已弃用。

```python
async def foo():
    return 42

task = asyncio.create_task(foo())
done, pending = await asyncio.wait({task})

if task in done:
    # Everything will work as expected now.
```

##### asyncio.as_completed()

```text
asyncio.as_completed(aws, *, loop=None, timeout=None)

并发地运行 aws 集合中的 可等待对象。返回一个 Future 对象的迭代器。Each Future object returned represents the earliest result from the set of the remaining awaitables.
如果在所有 Future 对象完成前发生超时则将引发 asyncio.TimeoutError。
```

```python
for f in as_completed(aws):
    earliest_result = await f
    # ...
```

#### Asyncio.gather vs asyncio.wait

Asyncio.gather():Returns a Future instance, allowing high level grouping of tasks.All tasks in a group can be cancelled by calling `group2.cancel()` or even `all_groups.cancel()`. 

```py
import asyncio
from pprint import pprint

import random


async def coro(tag):
    print(">", tag)
    await asyncio.sleep(random.uniform(1, 3))
    print("<", tag)
    return tag


loop = asyncio.get_event_loop()

group1 = asyncio.gather(*[coro("group 1.{}".format(i)) for i in range(1, 6)])
group2 = asyncio.gather(*[coro("group 2.{}".format(i)) for i in range(1, 4)])
group3 = asyncio.gather(*[coro("group 3.{}".format(i)) for i in range(1, 10)])

all_groups = asyncio.gather(group1, group2, group3)

results = loop.run_until_complete(all_groups)

loop.close()

pprint(results)
```

asyncio.wait():Supports waiting to be stopped after the first task is done, or after a specified timeout, allowing lower level precision of operations:

```py
import asyncio
import random


async def coro(tag):
    print(">", tag)
    await asyncio.sleep(random.uniform(0.5, 5))
    print("<", tag)
    return tag


loop = asyncio.get_event_loop()

tasks = [coro(i) for i in range(1, 11)]

print("Get first result:")
finished, unfinished = loop.run_until_complete(
    asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED))

for task in finished:
    print(task.result())
print("unfinished:", len(unfinished))

print("Get more results in 2 seconds:")
finished2, unfinished2 = loop.run_until_complete(
    asyncio.wait(unfinished, timeout=2))

for task in finished2:
    print(task.result())
print("unfinished2:", len(unfinished2))

print("Get all other results:")
finished3, unfinished3 = loop.run_until_complete(asyncio.wait(unfinished2))

for task in finished3:
    print(task.result())

loop.close()
```

### 其他常用方法

#### Sleeping

```text
coroutine asyncio.sleep(delay, result=None, *, loop=None)

阻塞 delay 指定的秒数。
如果指定了 result，则当协程完成时将其返回给调用者。
sleep() 总是会挂起当前任务，以允许其他任务运行。
loop 参数已弃用，计划在 Python 3.10 中移除。
```

```python
import asyncio
import datetime

async def display_date():
    loop = asyncio.get_running_loop()
    end_time = loop.time() + 5.0
    while True:
        print(datetime.datetime.now())
        if (loop.time() + 1.0) >= end_time:
            break
        await asyncio.sleep(1)

asyncio.run(display_date())
```

#### 屏蔽取消操作

```text
awaitable asyncio.shield(aw, *, loop=None)

保护一个 可等待对象 防止其被 取消。
If aw is a coroutine it is automatically scheduled as a Task.
```

以下语句:

```
res = await shield(something())
```

相当于:

```
res = await something()
```

*不同之处* 在于如果包含它的协程被取消，在 `something()` 中运行的任务不会被取消。从 `something()` 的角度看来，取消操作并没有发生。然而其调用者已被取消，因此 "await" 表达式仍然会引发 `CancelledError`。

如果通过其他方式取消 `something()` (例如在其内部操作) 则 `shield()` 也会取消。

#### 超时

```text
coroutine asyncio.wait_for(aw, timeout, *, loop=None)

等待 aw 可等待对象完成，timeout 秒数后超时。
If aw is a coroutine it is automatically scheduled as a Task.
timeout 可以为 None，也可以为 float 或 int 型数值表示的等待秒数。如果 timeout 为 None，则等待直到完成。如果发生超时，任务将取消并引发 asyncio.TimeoutError.
要避免任务 取消，可以加上 shield()。
函数将等待直到目标对象确实被取消，所以总等待时间可能超过 timeout 指定的秒数。
如果等待被取消，则 aw 指定的对象也会被取消。
loop 参数已弃用，计划在 Python 3.10 中移除。
```

```python
async def eternity():
    # Sleep for one hour
    await asyncio.sleep(3600)
    print('yay!')

async def main():
    # Wait for at most 1 second
    try:
        await asyncio.wait_for(eternity(), timeout=1.0)
    except asyncio.TimeoutError:
        print('timeout!')

asyncio.run(main())
```

#### Scheduling From Other Threads

```text
asyncio.run_coroutine_threadsafe(coro, loop)
向指定事件循环提交一个协程。线程安全。
返回一个 concurrent.futures.Future 以等待来自其他 OS 线程的结果。
此函数应该从另一个 OS 线程中调用，而非事件循环运行所在线程。
```

```python
# Create a coroutine
coro = asyncio.sleep(1, result=3)

# Submit the coroutine to a given loop
future = asyncio.run_coroutine_threadsafe(coro, loop)

# Wait for the result with an optional timeout argument
assert future.result(timeout) == 3
```

#### 内省

```text
asyncio.current_task(loop=None)

返回当前运行的 Task 实例，如果没有正在运行的任务则返回 None。
如果 loop 为 None 则会使用 get_running_loop() 获取当前事件循环。

asyncio.all_tasks(loop=None)
返回事件循环所运行的未完成的 Task 对象的集合。
如果 loop 为 None，则会使用 get_running_loop() 获取当前事件循环。
```

***

## 其他

### 函数

Python中常见的几种函数形式：

1. 普通函数

```python
def function():
    return 1
```

2. 生成器函数

```python
def generator():
    yield 1
```

在3.5过后，我们可以使用async修饰将普通函数和生成器函数包装成异步函数和异步生成器。

3. 异步函数（协程）

```python
async def async_function():
    return 1
```

4. 异步生成器

```python
async def async_generator():
    yield 1
```

通过类型判断可以验证函数的类型：

```python
import types
print(type(function) is types.FunctionType)
print(type(generator()) is types.GeneratorType)
print(type(async_function()) is types.CoroutineType)
print(type(async_generator()) is types.AsyncGeneratorType)
```

直接调用异步函数不会返回结果，而是返回一个coroutine对象：

```python
print(async_function())
# <coroutine object async_function at 0x102ff67d8>
```

协程需要通过其他方式来驱动，因此可以使用这个协程对象的send方法给协程发送一个值：

```python
print(async_function().send(None))
```

不幸的是，如果通过上面的调用会抛出一个异常：

```python
StopIteration: 1
```

因为生成器/协程在正常返回退出时会抛出一个StopIteration异常，而原来的返回值会存放在StopIteration对象的value属性中，通过以下捕获可以获取协程真正的返回值：

```python
try:
    async_function().send(None)
except StopIteration as e:
    print(e.value)
# 1
```

通过上面的方式来新建一个run函数来驱动协程函数：

```python
def run(coroutine):
    try:
        coroutine.send(None)
    except StopIteration as e:
        return e.value
```

在协程函数中，可以通过await语法来挂起自身的协程，并等待另一个协程完成直到返回结果：

```python
async def async_function():
    return 1

async def await_coroutine():
    result = await async_function()
    print(result)
    
run(await_coroutine())
# 1
```

要注意的是，await语法只能出现在通过async修饰的函数中，否则会报SyntaxError错误。

而且await后面的对象需要是一个Awaitable，或者实现了相关的协议。

查看Awaitable抽象类的代码，表明了只要一个类实现了__await__方法，那么通过它构造出来的实例就是一个Awaitable：

```python
class Awaitable(metaclass=ABCMeta):
    __slots__ = ()

    @abstractmethod
    def __await__(self):
        yield

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Awaitable:
            return _check_methods(C, "__await__")
        return NotImplemented
```

而且可以看到，Coroutine类也继承了Awaitable，而且实现了send，throw和close方法。所以await一个调用异步函数返回的协程对象是合法的。

```python
class Coroutine(Awaitable):
    __slots__ = ()

    @abstractmethod
    def send(self, value):
        ...

    @abstractmethod
    def throw(self, typ, val=None, tb=None):
        ...

    def close(self):
        ...
        
    @classmethod
    def __subclasshook__(cls, C):
        if cls is Coroutine:
            return _check_methods(C, '__await__', 'send', 'throw', 'close')
        return NotImplemented
```

接下来是异步生成器，来看一个例子：

假如我要到一家超市去购买土豆，而超市货架上的土豆数量是有限的。对应到代码中，就是迭代一个生成器的模型，也许可以用多进程和多线程解决，而在现实生活中，更应该像是这样的：

```python
async def take_potatos(num):
    count = 0
    while True:
        if len(all_potatos) == 0:
            await ask_for_potato()
        potato = all_potatos.pop()
        yield potato
        count += 1
        if count == num:
            break
```

当货架上的土豆没有了之后，我可以询问超市请求需要更多的土豆，这时候需要等待一段时间直到生产者完成生产的过程：

```python
async def ask_for_potato():
    await asyncio.sleep(random.random())
    all_potatos.extend(Potato.make(random.randint(1, 10)))
```

当生产者完成和返回之后，这是便能从await挂起的地方继续往下跑，完成消费的过程。而这整一个过程，就是一个异步生成器迭代的流程：

```python
async def buy_potatos():
    bucket = []
    async for p in take_potatos(50):
        bucket.append(p)
        print(f'Got potato {id(p)}...')
```

async for语法表示我们要后面迭代的是一个异步生成器。

```python
async def buy_potatos():
    bucket = []
    async for p in take_potatos(50):
        bucket.append(p)
        print(f'Got potato {id(p)}...')
```

async for语法表示我们要后面迭代的是一个异步生成器。

```python
def main():
    import asyncio
    loop = asyncio.get_event_loop()
    res = loop.run_until_complete(buy_potatos())
    loop.close()
```

### 异步推导表达式

异步生成器是在3.6之后才有的特性，同样的还有异步推导表达式，因此在上面的例子中，也可以写成这样：

```python
bucket = [p async for p in take_potatos(50)]
```

类似的，还有await表达式：

```python
result = [await fun() for fun in funcs if await condition()]
```

除了函数之外，类实例的普通方法也能用async语法修饰。

***

## 示例

### 消费生产者模型

```python
import asyncio, random


all_potatos = {x:random.randint(1, 50) for x  in "abc"}

async def take_potatos(num):
    count = 0
    while True:
        if len(all_potatos) == 0:
            print('Wait')
            await ask_for_potato()
        else:
            potato = all_potatos.popitem()
            yield potato
            count += 1
            if count == num:
                break

async def buy_potatos():
    bucket = []
    async for p in take_potatos(6):
        bucket.append(p)
        print(bucket)

async def ask_for_potato():
    all_potatos.update({x:random.randint(1, 20) for x  in "ABC"})

def main():
    loop=asyncio.get_event_loop()
    loop.run_until_complete(buy_potatos())
    # loop.run_until_complete(asyncio.wait([buy_potatos()]))
    loop.close()

if __name__ == '__main__':
    # main()
    asyncio.run(buy_potatos())
```

采用基于生成器的协程：

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

改写：

```python
import asyncio

async def consumer():
    r = ''
    async for n in producer():
    # while True:
        # n = await producer()  # async_generator can't be used in 'await' expression
        if not n:
            break
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'

async def producer():			
    n = 0
    while True:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        if n == 5:
            break
        yield n					# async_generator

asyncio.run(consumer())
```

### aiohttp

支持非阻塞异步I/O的库是aiohttp。以获取网址信息为例：

使用不支持非阻塞异步I/O的request：

```python
import asyncio
import requests
import time


async def download(url):
    print("get %s" % url)    
    response = requests.get(url)
    print(response.status_code)


async def download_wait(url):
    await download(url)
    print("get {} data complete.".format(url))


async def main():
    start = time.time()
    await asyncio.gather(
        download_wait("http://www.163.com"),
        download_wait("https://www.qq.com/"),
        download_wait("http://www.baidu.com"))
    end = time.time()
    print("Complete in {} seconds".format(end - start))


asyncio.run(main())
```

```bash
get http://www.163.com
200
get http://www.163.com data complete.
get https://www.qq.com/
200
get https://www.qq.com/ data complete.
get http://www.baidu.com
200
get http://www.baidu.com data complete.
Complete in 0.22693800926208496 seconds
```

发现程序始终是同步执行的，这就说明仅仅是把涉及I/O操作的代码封装到async当中是不能实现异步执行的。必须使用支持异步操作的非阻塞代码才能实现真正的异步。目前支持非阻塞异步I/O的库是aiohttp。

```python
import asyncio
import aiohttp
import time


async def download(url):
    print("get: %s" % url)
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            print(resp.status)


async def main():
    start = time.time()
    await asyncio.gather(
        download("http://www.163.com"),
        download("https://www.qq.com/"),
        download("http://www.baidu.com"))
    end = time.time()
    print("Complete in {} seconds".format(end - start))

asyncio.run(main())
```

```bash
get: http://www.163.com
get: https://www.qq.com/
get: http://www.baidu.com
200
200
200
Complete in 0.10493993759155273 seconds
```

***

参考：

https://docs.python.org/3/library/asyncio-task.html#id3

https://www.ibm.com/developerworks/cn/analytics/library/ba-on-demand-data-python-3/index.html

http://www.ruanyifeng.com/blog/2013/10/event_loop.html

https://www.jianshu.com/p/b036e6e97c18