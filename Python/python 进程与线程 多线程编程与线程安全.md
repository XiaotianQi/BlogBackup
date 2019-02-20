## 多线程编程

### threading 标准库

使用 threading 标准库，进行多线程编程。

| 类型                     | 含义             |
| ------------------------ | ---------------- |
| Thread Objects           | 线程对象         |
| Lock Objects             | 互斥锁           |
| RLock Objects            | 可重入锁，递归锁 |
| Condition Objects        | 条件锁           |
| Semaphore Objects        | 信号锁           |
| BoundedSemaphore Objects | 有边界信号量     |
| Event Objects            | 事件             |
| Timer Objects            | 计时器           |
| Barrier Objects          | 屏障             |

### 线程池

使用 concurrent.futures 标准库中的线程池，进行多线程编程。

线程池不是线程安全的。

```python
from concurrent.futures import ThreadPoolExecutor

g_num = 0

def add():
    global g_num
    for i in range(1000000):
        g_num += 1     

def desc():
    global g_num
    for i in range(1000000):
        g_num -= 1


with ThreadPoolExecutor(max_workers=2) as executor:
    add_tast = executor.submit(add)
    desc_tast = executor.submit(desc)
    print(g_num)	# 非0
```

### 死锁

死锁的发生也必须具备一定的条件，死锁的发生必须具备以下四个必要条件：

* 互斥条件：指进程对所分配到的资源进行排它性使用，即在一段时间内某资源只由一个进程占用。如果此时还有其它进程请求资源，则请求者只能等待，直至占有资源的进程用毕释放。
* 请求和保持条件：指进程已经保持至少一个资源，但又提出了新的资源请求，而该资源已被其它进程占有，此时请求进程阻塞，但又对自己已获得的其它资源保持不放。
* 不剥夺条件：指进程已获得的资源，在未使用完之前，不能被剥夺，只能在使用完时由自己释放。
* 环路等待条件：指在发生死锁时，必然存在一个进程——资源的环形链，即进程集合{P0，P1，P2，···，Pn}中的P0正在等待一个P1占用的资源；P1正在等待P2占用的资源，……，Pn正在等待已被P0占用的资源。

#### “迭代”死锁

该情况是一个线程“迭代”请求同一个资源，直接就会造成死锁。

threading.Lock 并不关心当前是哪个线程占有了该锁；如果该锁已经被占有了，那么任何其它尝试获取该锁的线程都会被阻塞，包括已经占有该锁的线程也会被阻塞。为了解决这问题可以用threading.RLock代替threading.Lock。

```python
import threading
import time


class MyThread(threading.Thread):
    def run(self):
        global num
        time.sleep(1)
        if mutex.acquire():		# 第一次请求资源，请求后还未 release 
            num = num+1
            msg = self.name+' set num to '+str(num)
            print(msg)
            mutex.acquire()		# 再次请求，产生死锁
            mutex.release()
            mutex.release()

num = 0
mutex = threading.Lock()

def test():
    for i in range(5):
        t = MyThread()
        t.start()


if __name__ == '__main__':
    test()
```

#### 互相调用死锁

与上例类似，上面是同一个def函数内多次调用造成的，另一种情况是两个函数中都会调用相同的资源，互相等待对方结束的情况。如果两个线程分别占有一部分资源并且同时等待对方的资源，就会造成死锁。

```python
import threading
import time

# 账户类
class Account(object):
    def __init__(self, name, balance, lock):
        self.name = name
        self.balance = balance
        self.lock = lock

    # 支出    
    def withdraw(self, amount):
        self.balance -= amount

    # 收入    
    def deposit(self, amount):
        self.balance += amount

# 转账函数        
def transfer(from_account, to_account, amount):
    with from_account.lock:             # 请求锁
        from_account.withdraw(amount)
        time.sleep(1)
        print("trying to get %s's lock..." % to_account.name)
        with to_account.lock:           # 请求锁
            to_account.deposit(amount)
    print("transfer finish")
    
if __name__ == "__main__":
    a = Account('a',1000, threading.Lock())
    b = Account('b',1000, threading.Lock())
    thread_list = []
    thread_list.append(threading.Thread(target = transfer, args=(a,b,100)))
    thread_list.append(threading.Thread(target = transfer, args=(b,a,500)))
    for i in thread_list:
        i.start()
    for j in thread_list:
        j.join()
```

#### 防止死锁的加锁机制

目前我们遇到的问题是两个线程想获取到的锁，都被对方线程拿到了，那么我们只需要保证在这两个线程中，获取锁的顺序保持一致就可以了。

```python
# _*_coding:utf-8_*_

import threading, time
from contextlib import contextmanager

# Thread-local state to stored information on locks already acquired
_local = threading.local()

@contextmanager
def acquire(*locks):
    # Sort locks by object identifier
    locks = sorted(locks, key=lambda x: id(x))

    # Make sure lock order of previously acquired locks is not violated
    acquired = getattr(_local,'acquired',[])
    if acquired and max(id(lock) for lock in acquired) >= id(locks[0]):
        raise RuntimeError('Lock Order Violation')

    # Acquire all of the locks
    acquired.extend(locks)
    _local.acquired = acquired

    # 实现的是锁的获取和释放
    try:
        for lock in locks:
            lock.acquire()
        yield
    finally:
        # Release locks in reverse order of acquisition
        for lock in reversed(locks):
            lock.release()
        del acquired[-len(locks):]

# 账户类
class Account(object):
    def __init__(self, name, balance, lock):
        self.name = name
        self.balance = balance
        self.lock = lock

    # 支出    
    def withdraw(self, amount):
        self.balance -= amount

    # 收入    
    def deposit(self, amount):
        self.balance += amount

# 转账函数        
def transfer(from_account, to_account, amount):
    with acquire(from_account.lock, to_account.lock):             # 请求锁
        from_account.withdraw(amount)
        time.sleep(1)
        to_account.deposit(amount)
    print("transfer finish")

if __name__ == "__main__":
    a = Account('a',1000, threading.Lock())
    b = Account('b',1000, threading.Lock())
    thread_list = []
    thread_list.append(threading.Thread(target = transfer, args=(a,b,100)))
    thread_list.append(threading.Thread(target = transfer, args=(b,a,500)))
    for i in thread_list:
        i.start()
    for j in thread_list:
        j.join()
```



***

## 线程安全

在多线程环境中，当各线程不共享数据的时候，那么一定是线程安全的。在多数情况下需要共享数据，这时就需要进行适当的同步控制。线程安全一般都涉及到synchronized，就是多线程环境中，共享数据同一时间只能有一个线程来操作 不然中间过程可能会产生不可预制的结果。

存在线程安全问题必须满足三个条件：

1.有共享变量

2.处在多线程环境下

3.共享变量有修改操作。仅读取数据则不会产生线程安全问题。

当对全局资源存在写操作时，如果不能保证写入过程的原子性，会出现脏读脏写的情况，即线程不安全。Python的GIL只能保证原子操作的线程安全，因此在多线程编程时我们需要通过加锁来保证线程安全。

g_num 的值预想中应该是 0，但运行多次结果都不是  0，而且每次不同。

```python
import threading, time

g_num = 0

def add():
    global g_num
    for i in range(1000000):
        g_num += 1     

def desc():
    global g_num
    for i in range(1000000):
        g_num -= 1


t1 = threading.Thread(target=add)
t2 = threading.Thread(target=desc)
t1.start()
t2.start()

t1.join()
t2.join()
print(g_num)		# 多次运行输出：-18106、365472、183348
```

在一个进程内的所有线程共享全局变量。如果多个线程共同对某个数据修改，则可能出现不可预料的结果，为了保证数据的正确性，需要对多个线程进行同步。

PS：

**线程安全**：指某个函数、函数库在多线程环境中被调用时，能够正确地处理多个线程之间的共享变量，使程序功能正确完成。 

一般来说，线程安全的函数应该为每个调用它的线程分配专门的空间，来储存需要单独保存的状态（如果需要的话），不依赖于“线程惯性”，把多个线程共享的变量正确对待（如，通知编译器该变量为“易失（volatile）”型，阻止其进行一些不恰当的优化），而且，线程安全的函数一般不应该修改全局对象。

**原子操作**：原子操作就是不会因为进程并发或者线程并发而导致被中断的操作。原子操作的特点就是要么一次全部执行，要么全不执行。不存在执行了一半而被中断的情况。当对全局资源存在写操作时，如果不能保证写入过程的原子性，会出现脏读脏写的情况。

**线程惯性**：指在多线程编程中的一种错误的心理状态，它假定当前编写的代码执行完毕后会继续执行下一条代码。而实际上，在现代处理器中，线程随时（当该线程的时间片用完时）可能被处理器冻结，而处理器被另一线程抢占（这里指单处理器上的情况，在多处理器上，情况更加复杂）。

因此，如果程序执行的结果依赖于这两个（或者可能更多）线程的顺序，程序就可能出错。

因为线程执行具有不确定性，这种错误并不是每次都会出现，而且在某些特定的机器上可能永远不会出现。因此，这种错误较难发现。 

**线程同步**：保证数据的一致性。多个线程操作一个资源的情况下，导致资源数据前后不一致。这样就需要协调线程的调度。即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作， 其他线程才能对该内存地址进行操作，而其他线程又处于等待状态。

**事件 (同步原语)**：事件作为一种同步原语，是计算机科学中的一种同步机制，用来指示等待中的进程特定条件已经变为真。

当一个进程调用一个send原语时，在消息开始发送后，发送进程便处于阻塞状态，直至消息完全发送完毕，send原语的后继语句才能继续执行。当一个进程调用一个receive原语时，并不立即返回控制，而是等到把消息实际接收下来，并把它放入指定的接收区，才返回控制，继续执行该原语的后继指令。在这段时间它一直处于阻塞状态。上述的send和receive被称为同步通信原语或阻塞通信原语。

事件对象一般具有下述操作：

- wait - 执行中的线程被挂起直到事件为真。如果执行wait时事件已为真，则空操作。
- set - 设置事件状态为真，所有等待此事件的进程变为可调度。
- clear - 设置事件状态为假。

**竞争条件**：当多个进程或者线程在读写数据时，其最终的的结果依赖于多个进程的指令执行顺序。举例来说，如果计算机中的两个进程同时试图修改一个共享内存的内容，在没有并发控制的情况下，最后的结果依赖于两个进程的执行顺序与时机。而且如果发生了并发访问冲突，则最后的结果是不正确的。

竞争条件只有在多个线程同时访问同一资源且多个线程同时写入资源时才会发生。

***

## 线程同步的方法

如果多个线程共同对某个数据修改，则可能出现不可预料的结果，为了保证数据的正确性，需要对多个线程进行同步。

### Queue

Python 的 Queue 模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列 PriorityQueue。 

这些队列都实现了锁原语，能够在多线程中直接使用，可以使用队列来实现线程间的同步。

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

------

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

***

### Threading 模块

| 类型                     | 含义             |
| ------------------------ | ---------------- |
| Lock Objects             | 互斥锁           |
| RLock Objects            | 可重入锁，递归锁 |
| Condition Objects        | 条件锁           |
| Semaphore Objects        | 信号锁           |
| BoundedSemaphore Objects | 有边界信号量     |
| Event Objects            | 事件             |
| Timer Objects            | 计时器           |
| Barrier Objects          | 屏障             |

***

## 生产者消费者问题

Producer-consumer problem，也称有限缓冲问题（Bounded-buffer problem），是一个多进程同步问题的经典案例。本质上，这是一种供需不平衡的表现。生产者消费者模式的核心是‘阻塞队列’也称消息队列。

生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而是通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不直接找生产者要数据，而是从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力，解耦了生产者和消费者。

该问题描述了**共享**固定大小缓冲区的两个进程——即所谓的“生产者”和“消费者”——在实际运行时会发生的问题。生产者的主要作用是生成一定量的数据放到缓冲区中，然后重复此过程。与此**同时**，消费者也在缓冲区消耗这些数据。该问题的关键就是要保证生产者不会在缓冲区满时加入数据，消费者也不会在缓冲区中空时消耗数据。 

生产者：生产随机数字

消费者：1.只取出偶数，2.只取出奇数

### Queue

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

### threading.Condition

```python
import random, threading, time


#Producer thread
class Producer(threading.Thread):
    def __init__(self, t_name, g_list, condition):
        super().__init__()
        self.name = t_name
        self.data = g_list
        self.condition = condition

    def run(self):
        while True:
            self.condition.acquire()
            randomnum = random.randint(1, 99)
            print("{}: {} is producing {} to the list!".format(time.ctime(), self.getName(), randomnum)) 
            self.data.append(randomnum)  # Put
            self.condition.notify()
            self.condition.release()
            time.sleep(1)


#Consumer thread
class Consumer_even(threading.Thread):
    def __init__(self, t_name, g_list, condition):
        super().__init__()
        self.name = t_name
        self.data = g_list
        self.condition = condition
        
    def run(self):
        while True:
            self.condition.acquire()
            if not self.data:
                self.condition.wait()
            val_even = self.data.pop()  # get(self, block=True, timeout=None), Block at most 5 seconds
            if val_even%2==0:
                print("{}: {} is consuming. {} in the list is consumed!".format(time.ctime(), self.getName(), val_even))
                self.condition.notify()
                self.condition.release()
                time.sleep(2)
            else:
                self.data.append(val_even)
                self.condition.notify()
                self.condition.release()
                time.sleep(2)



class Consumer_odd(threading.Thread):
    def __init__(self, t_name, g_list, condition):
        super().__init__()
        self.name = t_name
        self.data = g_list
        self.condition = condition

    def run(self):
        while True:
            self.condition.acquire()    
            if not self.data:
                self.condition.wait()
            val_odd = self.data.pop()
            if val_odd%2!=0:
                print("{}: {} is consuming. {} in the list is consumed!".format(time.ctime(), self.getName(), val_odd))
                self.condition.notify()
                self.condition.release()
                time.sleep(2)
            else:
                self.data.append(val_odd)
                self.condition.notify()
                self.condition.release()            
                time.sleep(2)


#Main thread  
def main():
    g_list = []
    condition = threading.Condition()
    producer = Producer('Pro.', g_list, condition)
    consumer_even = Consumer_even('Con_even.', g_list, condition)
    consumer_odd = Consumer_odd('Con_odd.', g_list, condition)
    producer.start()
    consumer_even.start()
    consumer_odd.start()
    producer.join()
    consumer_even.join()
    consumer_odd.join()


if __name__ == '__main__':
    main()
```

### threading.Event

```python
import random, threading, time


#Producer thread
class Producer(threading.Thread):
    def __init__(self, t_name, g_list, event):
        super().__init__()
        self.name = t_name
        self.data = g_list
        self.event = event

    def run(self):
        while True:
            randomnum = random.randint(1, 99)
            print("{}: {} is producing {} to the list!".format(time.ctime(), self.getName(), randomnum)) 
            self.data.append(randomnum)  # Put
            self.event.set()
            self.event.clear()
            time.sleep(1)


#Consumer thread
class Consumer_even(threading.Thread):
    def __init__(self, t_name, g_list, event):
        super().__init__()
        self.name = t_name
        self.data = g_list
        self.event = event
        
    def run(self):
        while True:
            self.event.wait()
            if not self.data:
                self.event.wait()
            val_even = self.data.pop()  # get(self, block=True, timeout=None), Block at most 5 seconds
            if val_even%2==0:
                print("{}: {} is consuming. {} in the list is consumed!".format(time.ctime(), self.getName(), val_even))
                time.sleep(2)
            else:
                self.data.append(val_even)
                time.sleep(2)



class Consumer_odd(threading.Thread):
    def __init__(self, t_name, g_list, event):
        super().__init__()
        self.name = t_name
        self.data = g_list
        self.event = event

    def run(self):
        while True:
            self.event.wait()    
            if not self.data:
                self.event.wait()
            val_odd = self.data.pop()
            if val_odd%2!=0:
                print("{}: {} is consuming. {} in the list is consumed!".format(time.ctime(), self.getName(), val_odd))
                time.sleep(2)
            else:
                self.data.append(val_odd)      
                time.sleep(2)


#Main thread  
def main():
    g_list = []
    event = threading.Event()
    producer = Producer('Pro.', g_list, event)
    consumer_even = Consumer_even('Con_even.', g_list, event)
    consumer_odd = Consumer_odd('Con_odd.', g_list, event)
    producer.start()
    consumer_even.start()
    consumer_odd.start()
    producer.join()
    consumer_even.join()
    consumer_odd.join()


if __name__ == '__main__':
    main()
```

***



