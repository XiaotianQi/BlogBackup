A synchronized queue class.

queue是python中的标准库，俗称队列。

在python中，多个线程之间的数据是共享的，多个线程进行数据交换的时候，不能够保证数据的安全性和一致性，所以当多个线程需要进行数据交换的时候，队列就出现了，队列可以完美解决线程间的数据交换，保证线程间数据的安全性和一致性。队列是一个非常好的线程同步机制，不用关心锁，队列会为我们处理锁的问题。

Python的Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语，能够在多线程中直接使用。可以使用队列来实现线程间的同步。

***

## 三种队列

The module implements three types of queue, which **differ only in the order in which the entries are retrieved**. 

* In a **FIFO** queue, the first tasks added are the first retrieved. 
* In a **LIFO** queue, the most recently added entry is the first retrieved (operating like a stack). 
* With a **priority** queue, the entries are kept sorted (using the heapq module) and the lowest valued entry is retrieved first.

Internally, those three types of queues use locks to temporarily block competing threads; however, they are not designed to handle reentrancy within a thread.

### 1. Queue

`class queue.Queue(maxsize=0)`

实现了先进先出（FIFO）队列。`maxsize`是队列里最多能同时存在的元素个数。如果队列满了，则会暂时阻塞队列，直到有消费者取走元素

```bash
In [1]: import queue

In [2]: q = queue.Queue()

In [3]: for i in range(3):		# 存入 0， 1， 2
   ...:     q.put(i)
   ...:

In [4]: while not q.empty():	# 读取，遵循FIFO，输出 0， 1， 2
   ...:     print(q.get())
   ...:
0
1
2
```

### 2. LifoQueue

` class queue.LifoQueue(maxsize=0)`

与 `Queue` 不同，`LifoQueue` 后进先出序（LIFO）。`maxsize`是队列里最多能同时存在的元素个数。如果队列满了，则会暂时阻塞队列，直到有消费者取走元素

```bash
In [5]: q = queue.LifoQueue()

In [6]: for i in range(3):		# 存入 0， 1， 2
   ...:     q.put(i)
   ...:

In [7]: while not q.empty():	# 读取，遵循LIFO，输出 2， 1， 0
   ...:     print(q.get())
   ...:
2
1
0
```

### 3. PriorityQueue

` class queue.PriorityQueue(maxsize=0)`

除了以上的顺序外，`PriorityQueue`依据队列中优先级决定哪个元素将被检索。优先级队列是被 `put` 进去的一个元祖(优先级,数据)，优先级数字越小，优先级越高。`maxsize`是队列里最多能同时存在的元素个数。如果队列满了，则会暂时阻塞队列，直到有消费者取走元素

```bash
In [10]: q = queue.PriorityQueue()

In [11]: q.put((1, 'a'))

In [12]: q.put((3, 'c'))

In [13]: q.put((2, 'b'))

In [14]: q.get()
Out[14]: (1, 'a')

In [15]: q.get()
Out[15]: (2, 'b')

In [16]: q.get()
Out[16]: (3, 'c')
```

注意：如果有两个元素优先级是一样的，那么在出队的时候是按照先进先出的顺序的。

***

## 公用方法

```text
q.qsize()：返回队列的大小。

q.empty()：如果队列为空，返回True，否则返回False。

q.full()：如果队列已满，返回True，否则返回False。

q.put(item, block=True, timeout=None)：将item放入队列。
block和timeout两个参数配合使用。
如果 block=True，timeout=None，队列满时，那么调用者将被阻塞直到队列中出现可用的空闲位置为止。
如果 block=True，timeout=正整数N，队列满时，如果在等待了N秒后，队列还没有空槽，则弹出Full异常
如果 block=False，则timeout参数被忽略，队列满时，那么此方法将引发Full异常。

q.put_nowait(item):等价于q.put(item,False)

q.get(block=True, timeout=None):从队列中删除一个item，并返回此item。
如果 block=True, timeout=None，队列为空时，那么调用者将阻塞，直到队列中出现可用的item为止。
如果 block=False，队列为空时，那么将引发Empty异常。
如果 timeout 为正数，而且超时，那么将引发Empty异常。

q.get_nowait()：等价于get(False)

```

Two methods are offered to support tracking whether enqueued tasks have been fully processed by daemon consumer threads.

```text
q.task_done():指示队列中的一个item已经完成。
完成一个item之后，queue.task_done()函数向队列调用者的线程发送一个完成信号。
每次调用 q.get() 方法获取队列中一个item后，再次调用 q.task_done() 告知队列被获取的item完成。
当 q.join() 处于阻塞状态时，当每个被 q.put() 进入队列的item都 q.task_done() 后，即队列中所有的item都已完成，那么 q.join() 恢复，解除阻塞。
如果 q.task_done() 调用次数超过了队列中item的次数，那么抛出 ValueError。

q.join()：阻塞调用线程，直到队列中的所有item均被处理为止，意味着等到队列为空，再执行别的操作。
```

重要的4个方法：

- put: 向队列中添加一个项。
- get: 从队列中删除并返回该项。
- task_done: 当某一项任务完成时调用。
- join: 阻塞直到所有的项目都被处理完。

PS：

Python提供了很多关于队列的类，其中：

`Class multiprocessing.Queue`是用于多进程的队列类（不要和多线程搞混了） 

`collections.deque`则是一种可选择的队列替代方案，它提供了快速的原子级别的`append()`和`popleft()`方法，但是不提供锁的能力。
