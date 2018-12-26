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

实现了先进先出（FIFO）队列。

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

与 `Queue` 不同，`LifoQueue` 后进先出序（LIFO）。

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

除了以上的顺序外，`PriorityQueue`依据队列中优先级决定哪个元素将被检索。优先级队列是被 `put` 进去的一个元祖(优先级,数据)，优先级数字越小，优先级越高。

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
q.size()：返回队列的大小。

q.empty()：如果队列为空，返回True，否则返回False。

q.full()：如果队列已满，返回True，否则返回False。

q.put(item, block=True, timeout=None)：将item放入队列。
如果 block=True，队列满时，那么调用者将被阻塞直到队列中出现可用的空闲位置为止。
如果 block=False，队列满时，那么此方法将引发Full异常。
如果 timeout 为正数，队列满时，那么调用者最多被阻塞 timeout 秒。如果在超时时间内没有空闲的位置，那么那么引发Full异常。

q.put_nowait(item):等价于q.put(item,False)

q.get(block=True, timeout=None):从队列中删除一个item，并返回此item。
如果 block=True, timeout=None，队列为空时，那么调用者将阻塞，直到队列中出现可用的item为止。
如果 block=False，队列为空时，那么将引发Empty异常。
如果 timeout 为正数，而且超时，那么将引发Empty异常。

q.get_nowait()：等价于get(False)

```

Two methods are offered to support tracking whether enqueued tasks have been fully processed by daemon consumer threads.

```text
q.task_done():指示队列中的一个item已经完成。完成一个item之后，queue.task_done()函数向队列调用者的线程发送一个完成信号。
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

***

## 多线程编程

用队列在线程间传递数据，通过队列在线程间分发处理事务。

- Queue 是一个 FIFO 先进先出队列
- 队列对象自动获取线程锁，释放锁，线程安全
- 任意时刻只有一个线程可以修改队列
- 可以用来在做线程间传递数据

官方伪代码：

```python
def worker():
    while True:
        item = q.get()					# 取出 item
        if item is None:
            break
        do_work(item)
        q.task_done()					# 该任务结束

q = queue.Queue()
        
threads = []
for i in range(num_worker_threads):		# 创建多线程
    t = threading.Thread(target=worker)
    t.start()
    threads.append(t)

for item in source():
    q.put(item)							# 存入 item

# block until all tasks are done
q.join()

# stop workers
for i in range(num_worker_threads):		# 结束整个任务
    q.put(None)
for t in threads:
    t.join()
```

***

## 示例

下面代码来自：[queue--刘江](http://www.liujiangblog.com/)

```python
import time
import queue
import threading


def worker(i):
    while True:
        item = q.get()
        if item is None:
            print("线程%s发现了一个None,可以休息了^-^" % i)
            break
        # do_work(item)做具体的工作
        time.sleep(0.5)
        print("线程%s将任务<%s>完成了！" % (i, item))
        # 做完后发出任务完成信号，然后继续下一个任务
        q.task_done()


if __name__ == '__main__':
    num_of_threads = 5

    source = [i for i in range(1, 21)]  # 模拟20个任务

    # 创建一个FIFO队列对象，不设置上限
    q = queue.Queue()
    # 创建一个线程池
    threads = []
    # 创建指定个数的工作线程，并讲他们放到线程池threads中
    for i in range(1, num_of_threads+1):
        t = threading.Thread(target=worker, args=(i,))
        threads.append(t)
        t.start()

    # 将任务源里的任务逐个放入队列
    for item in source:
        time.sleep(0.5)   			  # 每隔0.5秒发布一个新任务
        q.put(item)

    # 阻塞队列直到队列里的任务都完成了
    q.join()
    print("-----工作都完成了-----")
    # 停止工作线程
    for i in range(num_of_threads):
        q.put(None)
    for t in threads:
        t.join()
    print(threads)
```

如下代码来自：[Andrew_liu](https://www.jianshu.com/p/544d406e0875)

```python
import threading
import time
import Queue

SHARE_Q = Queue.Queue()						# 构造一个不限制大小的的队列
_WORKER_THREAD_NUM = 3  					# 设置线程的个数

class MyThread(threading.Thread) :
    """
    doc of class
    
    Attributess:
        func: 线程函数逻辑
    """
    def __init__(self, func) :
        super().__init__()					# 调用父类的构造函数
        self.func = func					# 传入线程函数逻辑

    def run(self) :
        """
        重写基类的run方法
        """
        self.func()

def do_something(item) :
    """
    运行逻辑, 比如抓站
    """
    print item

def worker() :
    """
    主要用来写工作逻辑, 只要队列不空持续处理
    队列为空时, 检查队列, 由于Queue中已经包含了wait,
    notify和锁, 所以不需要在取任务或者放任务的时候加锁解锁
    """
    global SHARE_Q
    while True : 
        if not SHARE_Q.empty():
            item = SHARE_Q.get()			# 获得任务
            do_something(item)
            time.sleep(1)
            SHARE_Q.task_done()


def main() :
    global SHARE_Q
    threads = []
    # 向队列中放入任务, 真正使用时, 应该设置为可持续的放入任务
    for task in xrange(5) :   
        SHARE_Q.put(task)
    # 开启_WORKER_THREAD_NUM个线程
    for i in xrange(_WORKER_THREAD_NUM) :
        thread = MyThread(worker)
        thread.start()						# 线程开始处理任务
        threads.append(thread)
    for thread in threads :
        thread.join()
    # 等待所有任务完成
    SHARE_Q.join()

if __name__ == '__main__':
    main()
```

下面代码来自：[Python Queue模块详解](https://blog.linuxeye.cn/334.html)

生产者/消费者（producer/consumer）模型

实现一个线程不断生成一个随机数到一个队列中(考虑使用Queue这个模块)
实现一个线程从上面的队列里面不断的取出奇数
实现另外一个线程从上面的队列里面不断取出偶数

```python
import random, threading, time
from queue import Queue


#Producer thread
class Producer(threading.Thread):
    def __init__(self, t_name, queue):
        super().__init__()
        self.data = queue
        self.name = t_name

    def run(self):
        while True:
            randomnum = random.randint(1, 99)
            print("{}: {} is producing {} to the queue!".format(time.ctime(), self.getName(), randomnum)) 
            self.data.put(randomnum)  # Put the generated number into queue
            time.sleep(1)


#Consumer thread
class Consumer_even(threading.Thread):
    def __init__(self, t_name, queue):
        super().__init__()
        self.data = queue
        self.name = t_name

    def run(self):
        while True:
            val_even = self.data.get(True, 5)  # get(self, block=True, timeout=None), Block at most 5 seconds
            if val_even%2==0:
                print("{}: {} is consuming. {} in the queue is consumed!".format(time.ctime(), self.getName(), val_even))
                time.sleep(2)
            else:
                self.data.put(val_even)
                time.sleep(2)



class Consumer_odd(threading.Thread):
    def __init__(self, t_name, queue):
        super().__init__()
        self.data = queue
        self.name = t_name

    def run(self):
        while True:
            val_odd = self.data.get(1,5)
            if val_odd%2!=0:
                print("{}: {} is consuming. {} in the queue is consumed!".format(time.ctime(), self.getName(), val_odd))
                time.sleep(2)
            else:
                self.data.put(val_odd)
                time.sleep(2)


#Main thread  
def main():
    queue = Queue()
    producer = Producer('Pro.', queue)
    consumer_even = Consumer_even('Con_even.', queue)
    consumer_odd = Consumer_odd('Con_odd.', queue)
    producer.start()
    consumer_even.start()
    consumer_odd.start()
    producer.join()
    consumer_even.join()
    consumer_odd.join()


if __name__ == '__main__':
    main()
```

