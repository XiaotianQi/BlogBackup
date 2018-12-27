线程间通信

## 线程安全

存在线程安全问题必须满足三个条件：

1.有共享变量

2.处在多线程环境下

3.共享变量有修改操作。仅读取数据则不会产生线程安全问题。

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

**线程惯性**：指在多线程编程中的一种错误的心理状态，它假定当前编写的代码执行完毕后会继续执行下一条代码。而实际上，在现代处理器中，线程随时（当该线程的时间片用完时）可能被处理器冻结，而处理器被另一线程抢占（这里指单处理器上的情况，在多处理器上，情况更加复杂）。

因此，如果程序执行的结果依赖于这两个（或者可能更多）线程的顺序，程序就可能出错。

因为线程执行具有不确定性，这种错误并不是每次都会出现，而且在某些特定的机器上可能永远不会出现。因此，这种错误较难发现。 

**线程同步**：保证数据的一致性。多个线程操作一个资源的情况下，导致资源数据前后不一致。这样就需要协调线程的调度。即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作， 其他线程才能对该内存地址进行操作，而其他线程又处于等待状态。

**事件 (同步原语)**：事件作为一种同步原语，是计算机科学中的一种同步机制，用来指示等待中的进程特定条件已经变为真。

当一个进程调用一个send原语时，在消息开始发送后，发送进程便处于阻塞状态，直至消息完全发送完毕，send原语的后继语句才能继续执行。当一个进程调用一个receive原语时，并不立即返回控制，而是等到把消息实际接收下来，并把它放入指定的接收区，才返回控制，继续执行该原语的后继指令。在这段时间它一直处于阻塞状态。上述的send和receive被称为同步通信原语或阻塞通信原语。

事件对象一般具有下述操作：

* wait - 执行中的线程被挂起直到事件为真。如果执行wait时事件已为真，则空操作。
* set - 设置事件状态为真，所有等待此事件的进程变为可调度。
* clear - 设置事件状态为假。

**竞争条件**：当多个进程或者线程在读写数据时，其最终的的结果依赖于多个进程的指令执行顺序。举例来说，如果计算机中的两个进程同时试图修改一个共享内存的内容，在没有并发控制的情况下，最后的结果依赖于两个进程的执行顺序与时机。而且如果发生了并发访问冲突，则最后的结果是不正确的。

竞争条件只有在多个线程同时访问同一资源且多个线程同时写入资源时才会发生。

***

## 线程同步的方法

如果多个线程共同对某个数据修改，则可能出现不可预料的结果，为了保证数据的正确性，需要对多个线程进行同步。

### Queue

Python 的 Queue 模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列 PriorityQueue。 

这些队列都实现了锁原语，能够在多线程中直接使用，可以使用队列来实现线程间的同步。

### Threading

| 类型                     | 含义             |
| ------------------------ | ---------------- |
| Thread Objects           | 线程对象         |
| Lock Objects             | 互斥锁           |
| RLock Objects            | 可重入锁，递归锁 |
| Condition Objects        | 条件变量         |
| Semaphore Objects        | 信号量           |
| BoundedSemaphore Objects | 有边界信号量     |
| Event Objects            | 事件             |
| Timer Objects            | 计时器           |
| Barrier Objects          | 屏障             |

***

## 生产者消费者问题

Producer-consumer problem，也称有限缓冲问题（Bounded-buffer problem），是一个多进程同步问题的经典案例。

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

## 线程池

标准库提供了**concurrent.futures**模块，它提供了 ThreadPoolExecutor 类，实现了对threading的进一步抽象，对编写线程池提供了直接的支持。

线程池是用于在程序中优化和简化线程的使用。通过池，可以提交任务给executor。池由两部分组成，一部分是内部的队列，存放着待执行的任务；另一部分是一系列的线程，用于执行这些任务。池的概念主要目的是为了重用：让线程在生命周期内可以多次使用。它减少了创建创建线程的开销，提高了程序性能。重用不是必须的规则，但它是程序员在应用中使用池的主要原因。

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

