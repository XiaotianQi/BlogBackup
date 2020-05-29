因为GIL的存在，Python想要实现多核运算，就要使用 multiprocess 模块。Multiprocessing 可以绕过 GIL，创建并发运行的程序 ，从而使用所有 CPU 核心。虽然它与 threading  库有着根本的不同，但是语法非常相似。 Multiprocessing 库为每个进程提供了自己的 Python 解释器，并为每个进程提供了自己的 GIL。 因此，与线程相关的常见问题(如数据损坏和死锁)不再是问题。同事，由于进程不共享内存，它们不能并发地修改相同的内存。

与多线程处理一样，多进程处理仍然存在缺陷... ... 你必须选择你的毒药：

* 在进程之间转移数据会产生 IO 开销
* 整个内存被复制到每个子进程中，这可能是很大的开销

Python中的multiprocess提供了Process类，实现进程相关的功能。但是它基于fork机制，因此不被windows平台支持。想要在windows中运行，必须使用`if __name__ == '__main__:`的方式，显然这只能用于调试和学习，不能用于实际环境。

线程具有五种状态：初始化、可运行、运行中、阻塞、销毁

这五种状态的转化关系如下：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%205%20status.png)

## 创建进程

### multiprocessing.Process

```python
class multiprocessing.Process(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)

创建进程对象。参数及用法和threading.Thread相同：
group:至今还未使用，值始终为None
target：进程实例所调用的对象，一般表示子进程要调用的函数。
name：当前进程实例的别名
args：表示调用对象的参数，一般是函数的参数
kwargs：表示调用对象的关键字参数字典。
```

常用方法：

```text
p.start():启动进程实例(创建子进程）,并运行子进程的run方法

p.run():默认会调用target指定的对象，如果没有给定target参数，对该进程对象调用start()方法时，就会执行对象中的run()方法

p.join([timeout]):等待进程实例执行完毕

p.is_alive():判断进程是否还在运行。如果还在运行，返回true，否则返回false

p.terminate():不管任务是否完成，立即终止,同时不会进行任何的清理工作，如果进程p创建了它自己的子进程，这些进程就会变成僵尸进程，使用时特别注意，如果p保存了一个锁或者参与了进程间通信，那么使用该方法终止它可能会导致死锁或者I/O损坏。
```

常用属性：

```text
p.daemon：布尔值，指示进程是否是后台进程。当创建它的进程终止时，后台进程会自动终止。并且，后台进程无法创建自己的新进城注意：p.daemon的值必须在p.start方法调用前设置。

p.exitcode：进程的整数退出指令。如果进程仍然在运行，它的值为None，如果值为负数：—N，就表示进程由信号N所终止。

p.name：当前进程实例别名，默认为Process-N，N为从1开始递增的整数。

p.pid：当前进程实例的PID
```

示例

```python
import os
import multiprocessing

def foo(i):
    # 同样的参数传递方法
    print("这里是 ", multiprocessing.current_process().name)
    print('模块名称:', __name__)
    print('父进程 id:', os.getppid())  # 获取父进程id
    print('当前子进程 id:', os.getpid())  # 获取自己的进程id
    print('------------------------')

if __name__ == '__main__':

    for i in range(5):
        p = multiprocessing.Process(target=foo, args=(i,))
        p.start()
```

### 自定义类

继承 multiprocessing.Process。

```python
import multiprocessing
import time

 
class ClockProcess(multiprocessing.Process):
    def __init__(self, interval):
        super().__init__()
        self.interval = interval
 
    def run(self):
        n = 5
        while n>0:
            print("The time is {0}".format(time.ctime()))
            time.sleep(self.interval)
            n-=1
 
if __name__ == "__main__":
    p = ClockProcess(2)
    p.start()
    p.join()
```

***

### 进程池

使用进程池控制内存开销。

```text
class multiprocessing.pool.Pool([processes[, initializer[, initargs[, maxtasksperchild[, context]]]]])

processes:需要创建的进程个数，如果忽略将使用cpu_count()的值，即系统上的CPU数量。
initializer:每个进程启动时都要调用的对象。
initargs:为initalizer传递的参数。
```

进程初始化时，会指定一个最大进程数量，当有新的请求需要创建进程时，如果此时进程池还没有到达设置的最大进程数，该进程池就会创建新的进程来处理该请求，并把该进程放到进程池中，如果进程池已经达到最大数量，请求就会等待，知道进程池中进程数量减少，才会新建进程来执行请求。

multiprocessing.Pool常用函数解析：

```text
p.apply_async(func[, args[, kwds[, callback[, error_callback]]]])：非阻塞调用指定方法，并行执行（同时执行）

p.apply(func[, args[, kwds]])：阻塞调用指定方法，只能允许一个进程,阻等上一个进程退出后，下一个进程才开始运行

p.close():不再接受新的进程请求，等待所有进程结束后，才关闭进程池，但已经接受的进程仍会继续执行。

p.terminate()：不管进程任务是否完成，立刻关闭进程池。

p.join():主进程堵塞（就是不执行join下面的语句），直到子进程结束，注意，该方法必须在close或terminate之后使用。

p.map(func, iterable[, chunksize]):与内置函数map()类似，将可调用对象func应用给iterable的每一项，然后以列表形式返回结果，通过将iterable划分为多块，并分配给工作进程，可以并行执行。chunksize指定每块中的项数，如果数据量较大，可以增大chunksize的值来提升性能。

p.map_async(func, iterable[, chunksize[, callback[, error_callback]]]):与map方法不同之处是返回结果是异步的，如果callback指定，当结果可用时，结果会调用callback。

p.imap(func, iterable[, chunksize]):与map()方法的不同之处是返回迭代器而非列表。

p.imap_unordered(func, iterable[, chunksize]):与imap()不同之处是：结果的顺序是根据从工作进程接收到的时间而定的。

p.get(timeout):如果没有设置timeout，将会一直等待结果，
如果设置了timeout，超过timeout将引发multiprocessing.TimeoutError异常。

p.ready():如果调用完成，返回True

p.successful():如果调用完成并且没有引发异常，返回True，如果在结果就绪之前调用，jiang引发AssertionError异常。

p.wait(timeout):等待结果变为可用，timeout为等待时间。
```

p.join() 是要让子进程全部处理完之后得到结果统一处理，还有一个非常重要的原因是进程池依附于主进程，主进程结束，进程池消失。进程池的任务没有被处理程序就结束了。p.join()必须使用在p.close()或者p.terminate()之后。其中p.close()跟p.terminate()的区别在于p.close()会等待池中的worker进程执行结束再关闭pool,而terminate()则是直接关闭。

```python
import os
import time
from multiprocessing import Pool

def test(name):
    print("{}:pid={:6} 运行中,父进程：{:d}".format(name, os.getpid(), os.getppid()))
    t_start=time.time()
    time.sleep(1)
    t_end=time.time()
    print("{}:pid={:6} 执行时间：{:.2f}s".format(name, os.getpid(), t_end-t_start))

if __name__ == "__main__":    
    pool=Pool(5)
    for i in range(0, 7):   # 非阻塞运行       
        pool.apply_async(test, ("Non-blocking",))   
    pool.close()            # 关闭线程池，关闭后不再接受进的请求
    pool.join()             # 等待进程池所有进程都执行完毕后，开始执行下面语句
    print("-"*15)

    pool=Pool(5)
    for j in range(0, 7):   # 阻塞运行        
        pool.apply(test, ("Blocking",))
    pool.close()
    pool.join()
```

***

## 进程锁

与threading类似，在multiprocessing里也有同名的锁类`RLock`，`Lock`，`Event`，`Condition`和 `Semaphore`，连用法都是一样样的。

***

## 进程间通信

多进程与多线程最大的不同在于，多进程的每一个进程都有一份变量的拷贝，进程之间的操作互不影响。

```python
import multiprocessing
import time

zero = 0

def change_zero():
    global zero
    for i in range(3):
        zero = zero + 1
        print(multiprocessing.current_process().name, zero)

if __name__ == '__main__':
    p1 = multiprocessing.Process(target = change_zero)
    p2 = multiprocessing.Process(target = change_zero)
    p1.start()
    p2.start()
    p1.join()
    p2.join()
    print(zero)
```

```text
Process-1 1
Process-1 2
Process-1 3
Process-2 1
Process-2 2
Process-2 3
0
```

想要在进程之间进行数据传递可以使用`Queues`、`Array`和`Manager`这三个multiprocess模块提供的类。

### multiprocessing.Queue

multiprocessing中的queue不能用于pool进程池

```text
multiprocessing.Queue([maxsize])
```

`multiprocessing`模块中提供了`multiprocessing.Queue`，它和`Queue.Queue`的区别在于，它里面封装了进程之间的数据交流，不同进程可以操作同一个`multiprocessing.Queue`。

```python
import multiprocessing
from multiprocessing import Process, queues

def func(i, q):
    ret = q.get()
    print("进程%s从队列里获取了一个%s，然后又向队列里放入了一个%s" % (i, ret, i))
    q.put(i)

if __name__ == "__main__":
    q = queues.Queue(20, ctx=multiprocessing)
    q.put(0)
    for i in range(10):
        p = Process(target=func, args=(i, q,))
        p.start()
    for i in range(10):
        p.join()
    print(q.get())
```

------

### multiprocessing.Pipe

pipe只能适用于两个进程，pipe的性能高于queue。pipe的功能和Queue类似，可以理解成简化版的Queue。

```python
(connection1, connection2)=multiprocessing.Pipe([duplex])

Returns a pair (conn1, conn2) of Connection objects representing the ends of a pipe.
```

multiprocessing.connection 相关方法如下：

```text
connection.send(obj):通过连接发送对象，将一个对象发送到连接的另一端，使用recv()读取该对象

connection.send_bytes(buffer[,offset,size]):Send byte data from a bytes-like object as a complete message.

connection.recv():接收connection.send()方法返回的对象。如果连接的另一端已经关闭，再也不会存在任何数据，该方法将引起EOFError异常。

connection.recv_bytes([maxlength]):接收connection.send_bytes()方法发送的一条完整字节信息，maxlength为可以接受的最大字节数。如果消息超过这个最大数，将引发IOError异常，并且在连接上无法进一步读取。如果连接的另一端已经关闭，再也不会有任何数据，该方法将引发EOFError异常。

connection.recv_bytes_into(buffer[,offset]):将连接另一端发送的字节数据的完整消息读入缓冲区，并返回消息中的字节数。直到有东西可以接收。如果没有要接收的内容且另一端已关闭，则引发EOFError。

connection.close() :关闭连接，当connection被垃圾回收时，默认会调用该方法

connection.fileno() :返回连接使用的文件描述符或句柄

connection.poll([timeout]):返回是否有可读的数据。
```

```python
import random
import time
from multiprocessing import Process, Pipe, current_process
def produce(conn):
    for i in range(5):
        new = random.randint(0, 100)
        print('{} produce {}'.format(current_process().name, new))
        conn.send(new)
        time.sleep(random.random())
def consume(conn):
    for i in range(5):
        print('{} consume {}'.format(current_process().name, conn.recv()))
        time.sleep(random.random())
if __name__ == '__main__':
    # (p1, p2) = Pipe()
    # p1 = Process(target=produce, args=(p1,))
    # p2 = Process(target=consume, args=(p2,))
    pipe = Pipe()	# 不同的创建方式，本质是相同的
    p1 = Process(target=produce, args=(pipe[0],))
    p2 = Process(target=consume, args=(pipe[1],))
    p1.start()
    p2.start()
	p1.join()
    p2.join()
```

------

### multiprocessing.Manager

通过Manager类也可以实现进程间数据的共享。`Manager()`返回的manager对象提供一个服务进程，使得其他进程可以通过代理的方式操作Python对象。manager对象支持 `list`, `dict`, `Namespace`, `Lock`, `RLock`, `Semaphore`, `BoundedSemaphore`, `Condition`, `Event`, `Barrier`, `Queue`, `Value` ,`Array`等多种格式。

```python
from multiprocessing import Process, Manager

def func(i, dic):
    dic["num"] = 100+i
    print(dic.items())

if __name__ == '__main__':
    dic = Manager().dict()
    for i in range(10):
        p = Process(target=func, args=(i, dic))
        p.start()
        p.join()
```

***

参考：

https://segmentfault.com/a/1190000016243041

http://www.liujiangblog.com/course/python/82