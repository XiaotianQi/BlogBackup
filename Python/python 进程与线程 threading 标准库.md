threading模块用于提供线程相关的操作，线程是应用程序中工作的最小单元。

threading模块提供的类： 

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

threading 模块提供的常用方法： 

```text
threading.current_thread(): 返回当前的线程对象。

threading.enumerate(): 返回当前存在的所有线程对象的列表，不包括尚未启动和终止的线程

threading.active_count(): 返回正在运行的线程数量，与len(threading.enumerate())相同

threading.get_ident():返回当前线程标识符，非零整数。

threading.main_thread():返回主线程。通常情况下，主线程是启动python解释器的线程

threading.settrace(func):为所有线程设置一个跟踪函数The func will be passed to sys.settrace() for each thread, before its run() method is called.

threading.setprofile(func):为所有线程设置一个配置文件函数The func will be passed to sys.setprofile() for each thread, before its run() method is called.

threading.stack_size([size]):返回新创建线程栈大小；或为后续创建的线程设定栈大小为 size
```

threading 模块提供的常量：

```text
threading.TIMEOUT_MAX:
设置threading全局超时时间，阻塞允许的最大超时时间，超时则抛出OverflowError.例如Lock.acquire(), RLock.acquire(), Condition.wait(), etc.
```

所有的线程锁都有一个加锁和释放锁的动作，非常类似文件的打开和关闭。通过with上下文管理器，可以确保锁被正常释放。

***

## Thread Objects

Thread 类用于单独的线程控制。

### 语法

```python
class threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
    
group：预留给ThreadGroup类，should be None

target：可调用对象。线程启动时，被run()方法调用

name：线程名称。默认是Thread-N格式名称。

args:元组，传递给target函数

kwargs:字典-关键字参数，传递给target函数
    
daemon:布尔值，指定是否是守护线程
```

NOTE：这个构造方法使用时，必须使用关键字参数。

If the subclass overrides the constructor, it must make sure to invoke the base class constructor (`Thread.__init__()`) before doing anything else to the thread.

***

### 属性和方法

* `t.start()`

  Start the thread’s activity.

  It must be called at most once per thread object. It arranges for the object’s run() method to be invoked in a separate thread of control.

  This method will raise a RuntimeError if called more than once on the same thread object.

* `t.run()`

  Method representing the thread’s activity.

  You may override this method in a subclass. The standard run() method invokes the callable object passed to the object’s constructor as the target argument, if any, with sequential and keyword arguments taken from the args and kwargs arguments, respectively.

* `t.join(timeout=None)`

  阻塞线程，等待直到线程终止或者出现超时为止。如果超时，那么继续执行主线程，但该线程是否执行取决于其是否是守护线程。无返回值。

  A thread can be join()ed many times.

* `t. is_alive()`

  如果线程是活动的，那返回True，否则返回False。

* `t.name`

  A string used for identification purposes only. It has no semantics. Multiple threads may be given the same name. The initial name is set by the constructor.

* `t.setName()`

  设置线程的名称。

* `t.getName()`

  获取线程的名称

* `t.ident`

  获取线程的标识符。线程标识符是一个非零整数，只有在调用了start()方法之后该属性才有效，否则它只返回None。

* `t.daemon`

  返回一个布尔值，当线程是守护线程时返回True，否则返回False。This must be set before start() is called, otherwise RuntimeError is raised. Its initial value is inherited from the creating thread; the main thread is not a daemon thread and therefore all threads created in the main thread default to daemon = False.

  The entire Python program exits when no alive non-daemon threads are left.

* `t.setDaemon()`

  设置线程为守护线程，在 `t.start()` 前使用。

***

### 两种创建线程的方法

可以通过如下两种方法创建线程：

- 使用 `threading.Thread` 构造函数，将线程要执行的任务函数作为参数传入线程
- 继承 `Thread` 类，并重载其 `run()` 方法

Note:No other methods (except for the constructor) should be overridden in a subclass. In other words, only override the `__init__()` and run() methods of this class.

创建线程后，调用其 `t.start()` 方法才能启用线程，`t.start()` 会调用线程中的 `run()` 方法。

一旦线程启用后，该线程就被视为活跃的。只有当 `run()` 正常终止或者抛出异常，该线程才会停止执行。

```python
import threading
import time


def print_time(threadName, delay, counter):
    while counter:
        start = time.time()
        time.sleep(delay)
        end = time.time()
        print('{}--delay:{}--time:{}'.format(threadName, delay, end-start))
        counter -= 1


class myThread (threading.Thread):
    def __init__(self, delay, counter):
        # 注意：一定要显式的调用父类的初始化函数
        threading.Thread.__init__(self)
        self.delay = delay
        self.counter = counter

    def run(self):
        print("开始线程：" + self.name)
        print_time(self.name, self.delay, self.counter)
        print("退出线程：" + self.name)


# 创建新线程
thread1 = myThread(delay=1, counter=2)
thread2 = myThread(delay=2, counter=3)

# 开启新线程
thread1.start()
thread2.start()
thread1.join()
thread2.join()
print("退出主线程")
```

***

### 关于 `t.join([timeout])`

阻塞当前上下文环境的线程，直到调用此方法的线程终止或到达指定的timeout（可选参数）。即当子进程的join()函数被调用时，主线程就被阻塞住了，意思为不再继续往下执行。

`t.join([timeout])` 具有阻塞作用，所以对控制多个线程的执行顺序非常关键，尤其设置了 timeout 参数后。

#### 不使用 `t.join()`

```python
import threading
import time

def tast1():
    print("T1 start")
    for i in range(10):
        time.sleep(0.1) # 任务间隔0.1s
    print("T1 finish")

if __name__ == "__main__":
    t1 = threading.Thread(target=tast1)
    t1.start()
    print("all done")
```

预想输出结果：

```text
T1 start
T1 finish
all done
```

实际输出结果：

```text
T1 start
all done
T1 finish
```

t1 线程开启后，继续运行了主线程。

有时候遇到多程程处理的场景，主线程要等待子线程完成数据解析，然后主线程才利用子线程的数据做下一步操作，那么python的实现方式是在主线程中调用子线程的join方法，这样主线程在子线程完成之前就会一直堵塞了。

#### 使用 `t.join()`

```python
if __name__ == "__main__":
    t1 = threading.Thread(target=tast1)
    t1.start()
    t1.join()				# 加入 join()
    print("all done")
```

输出结果：

```text
T1 start
T1 finish
all done
```

与预想中相同。

不过，这也说明，无论 `t.join()` 是否存在，主线程是否执行完毕，主线程都会等待非守护子线程执行完毕，然后一起退出。`t.join()` **控制调用顺序**。

#### 多线程中 `t.join()` 对线程的控制

```python
import threading
import time, functools

def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        end = time.time()
        print('{} cost: {}'.format(func.__name__, end - start))
    return clocked

@clock
def tast1():			# tast1 运行2S
    print("T1 start")
    for i in range(10):
        time.sleep(0.2)
    print("T1 finish")

@clock
def tast2():
    print("T2 start")	# tast2 运行1S
    for i in range(10):
        time.sleep(0.1)
    print("T2 finish")

if __name__ == "__main__":
    start = time.time()
    t1 = threading.Thread(target=tast1)
    t2 = threading.Thread(target=tast2)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    end = time.time()
    print("All done, cost:", end-start)
```

运行结果：

```text
T1 start
T2 start
T2 finish
tast2 cost: 1.026803970336914
T1 finish
tast1 cost: 2.053577184677124
All done, cost: 2.054577112197876
```

可以看出，两个线程同时开启。总消耗时间亦如预想。

但是如下使用 `t.join()`：

```python
if __name__ == "__main__":
    start = time.time()
    t1 = threading.Thread(target=tast1)
    t2 = threading.Thread(target=tast2)
    t1.start()
    t1.join()
    t2.start()
    t2.join()
    end = time.time()
    print("All done, cost:", end-start)
```

```text
T1 start
T1 finish
tast1 cost: 2.0437636375427246
T2 start
T2 finish
tast2 cost: 1.0767614841461182
All done, cost: 3.124523639678955
```

由结果可以看出，`t1.join()` 阻塞线程直至 tast1 任务执行完毕。这种情况并不是多线程，因为没有达到提高性能的目的。

`t.join()` 的原理就是依次检验线程是否结束。如果没有结束，那就阻塞直到该线程结束。如果结束或者遇到异常，则跳转执行下一个线程的 join() 函数。

* 阻塞主进程，专注执行多线程中的某一线程。
* 多线程多join的情况下，依次执行各线程的join方法，前头一个结束了才能执行后面一个。

#### timeout 参数的作用

`t.join([timeout])` 中的 timeout 参数，发挥如下作用：

- 无参数时，无论是否是守护线程都会阻塞，直至该线程运行结束，才会继续执行主线程
- 设置参数，超时会继续运行主线程，主线程结束时
  - 该线程是普通线程，主线程运行完毕后，等待其运行结束，然后一起退出
  - 该线程是守护线程，主线程运行完毕后，无论是是否运行完毕，直接退出

### 关于 Daemon

如果设置一个线程为守护线程，那么主线程执行完毕后自动退出，不会等待该子线程的执行结果。如果此时其他非守护线程已经执行完毕，那么随着主线程退出，该守护线程也消亡。The entire Python program exits when no alive non-daemon threads are left.

线程的 `deamon` 参数的值继承创建它的线程。主线程不是守护线程，所以在主线程中创建的子线程都默认 `deamon=False`。

两种使用	方式：

* 直接在构造函数中使用

  ```python
  t1 = threading.Thread(target=tast1, daemon=True)
  ```

* 使用 `setdeamon()`

  ```python
  t1.setDaemon(True)	# 必须在线程启动前使用
  t1.start()
  ```

#### 使用 `deamon`

```python
import threading
import time, functools

def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        end = time.time()
        print('{} cost: {}'.format(func.__name__, end - start))
    return clocked

@clock
def tast1():
    print("T1 start")
    for i in range(10):
        time.sleep(0.2)
    print("T1 finish")


if __name__ == "__main__":
    start = time.time()
    t1 = threading.Thread(target=tast1, daemon=True)
    t1.start()
    end = time.time()
    print("All done, cost:", end-start)
```

输出结果：

```text
T1 start
All done, cost: 0.0
```

可以直观看出，主线程并未等待 t1 运行完毕，就直接退出。

#### `join()` 对 `deamon` 的影响

只有当所有非守护线程执行完毕，那么主线程才会退出。对于守护线程：

* 如果 join() 未设置timeout参数，等待守护进程执行完毕，然后继续执行主线程。
* 如果设置了timeout=n，当所有普通线程执行完毕，且守护线程超时了，那么结束守护进程，继续执行主线程。

`t.join()`阻塞当前上下文环境的线程，直到调用此方法的线程终止或到达指定的timeout，即使设置了setDeamon（True）主线程依然要等待子线程结束。

* `deamon=False`，设置 `join(1)`

如下 `t1` 线程需要执行 2s，且该线程不是守护线程。虽然设置了 `t1.join(1)`，1s的阻塞后，主线程继续执行，但 t1 也会执行。

```python
import threading
import time, functools

def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        end = time.time()
        print('{} cost: {}'.format(func.__name__, end - start))
    return clocked

@clock
def tast1():
    print("T1 start")
    for i in range(10):
        time.sleep(0.2)
    print("T1 finish")

if __name__ == "__main__":
    start = time.time()
    t1 = threading.Thread(target=tast1)
    t1.start()
    t1.join(1)
    end = time.time()
    print("All done, cost:", end-start)
```

输出结果：

```text
T1 start
All done, cost: 1.0015082359313965
T1 finish
tast1 cost: 2.010915517807007
```

* `deamon=True`，设置 `join(1)`

仅修改 t1 为守护线程，其他同上。

```python
if __name__ == "__main__":
    start = time.time()
    t1 = threading.Thread(target=tast1, daemon=True)
    t1.start()
    t1.join(1)
    end = time.time()
    print("All done, cost:", end-start)
```

输出结果：

```text
T1 start
All done, cost: 1.0013401508331299
```

当 `t1.join(1)` 超时后，该线程强制结束，主线程继续执行，然后退出。

* `deamon=True`，设置 `join()`

```python
if __name__ == "__main__":
    start = time.time()
    t1 = threading.Thread(target=tast1, daemon=True)
    t1.start()
    t1.join()
    end = time.time()
    print("All done, cost:", end-start)
```

输出结果：

```python
T1 start
T1 finish
tast1 cost: 2.007636308670044
All done, cost: 2.012633800506592
```

`t1.join()` 会阻塞线程，直至 t1 执行完毕，主线程才会继续执行。

***

## Lock Objects

互斥锁是一个同步原语，一种用于多线程编程中，防止两条线程同时对同一公共资源（比如全局变量）进行读写的机制。对于那些需要每次只允许一个线程操作的数据，可以将其操作放到 acquire 和 release 方法之间。在python中，这是目前可以使用的最低级别的同步原语。

互斥锁也支持上下文管理器。

### 语法、方法

互斥锁是在 unlocked 状态下创建的，包含两个基础的方法：`acquire() `和`release()`。线程一旦获取了锁，那么将会阻塞后续获取锁的请求。 

* `class threading.Lock`

  创建一个锁。

* `acquire(blocking=True, timeout=-1)`

  获取锁。如果成功获得锁，返回 True，否则返回 False，超时也会返回 False。

  * blocking=True:阻塞，直到锁被释放。成功获取锁时，返回True，否则返回False。
  * blocking=False:非阻塞。当一个blocking=True的线程尝试获取锁时，立即返回 False。否则将锁设置为 lock，并返回True。
  * timeout:默认无限制等待锁被释放。

* `release()`

  释放锁。没有返回值。

  不仅可以在获得锁的线程中，还可以在任何线程中调用。但是，必须在 locked 状态，否则抛出 RuntimeError。

互斥锁有2种状态： locked 、 unlocked。

- When the state is unlocked, acquire() changes the state to locked and returns immediately. 
- When the state is locked, acquire() blocks until a call to release() in another thread changes it to unlocked, then the acquire() call resets it to locked and returns. 

The release() method should only be called in the locked state; it changes the state to unlocked and returns immediately. If an attempt is made to release an unlocked lock, a RuntimeError will be raised.

如果有多个线程等待获取锁，那么当锁被释放时，只有一个线程获得它，获取顺序是随机的。

### 示例

对全局变量的保护：

* 不使用互斥锁

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

我们定义了一个共享变量`g_num`，初始值为`0`，并且启动两个线程，先加后减，理论上结果应该为`0`。但是，由于线程的调度是由操作系统决定的，当t1、t2交替执行时，只要循环次数足够多，`g_num`的结果就不一定是`0`。

* 使用互斥锁

```python
import threading, time

g_num = 0
lock = threading.Lock()

def add():
    global g_num
    lock.acquire()
    for i in range(1000000):
        g_num += 1
    lock.release()
        

def desc():
    global g_num
    with lock:
        for i in range(1000000):
            g_num -= 1


t1 = threading.Thread(target=add)
t2 = threading.Thread(target=desc)
t1.start()
t2.start()

t1.join()
t2.join()
print(g_num)		# 输出0
```

由于锁只有一个，无论多少线程，同一时刻最多只有一个线程持有该锁，所以，不会造成修改的冲突。不过，因为 t1 和 t2 共用一把锁，如上调用锁，那么 t1、t2不再是同时执行，而是 t1 执行完毕后，t2才开始执行。

锁的好处就是确保了某段关键代码只能由一个线程从头到尾完整地执行。坏处当然也很多，首先是阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下降了。其次，由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁，导致多个线程全部挂起，既不能执行，也无法结束，只能靠操作系统强制终止。

* 两个线程的例子

```python
import threading
import time, functools

def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        end = time.time()
        print('{} cost: {}'.format(func.__name__, end - start))
    return clocked

@clock
def tast1():
    with lock:
        print("T1 start")
        for i in range(10):
            time.sleep(0.2)
        print("T1 finish")

@clock
def tast2():
    lock.acquire()
    print("T2 start")
    for i in range(10):
        time.sleep(0.3)
    print("T2 finish")
    lock.release()

if __name__ == "__main__":
    start = time.time()
    lock = threading.Lock()
    t1 = threading.Thread(target=tast1)
    t2 = threading.Thread(target=tast2)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    end = time.time()
    print("All done, cost:", end-start)
```

输出结果：

```text
T1 start
T1 finish
tast1 cost: 2.0664315223693848
T2 start
T2 finish
tast2 cost: 5.1368443965911865
All done, cost: 5.1378443241119385
```

可见，是顺序执行。

* 一个多线程无限循环的例子

```python
from threading import Thread
from queue import Queue

class Task1(Thread):
    def run(self):
        while True:
             if lock1.acquire():
                print("---Task1---")
                time.sleep(0.5)
                lock2.release()

class Task2(Thread):
    def run(self):
        while True:
             if lock2.acquire():
                print("---Task2---")
                time.sleep(0.5)
                lock3.release()
class Task3(Thread):
    def run(self):
        while True:
             if lock3.acquire():
                print("---Task3---")
                time.sleep(0.5)
                lock1.release()
#创建lock实例1，第一个开始不上锁
lock1=Lock()
lock2=Lock()
lock3=Lock()
lock2.acquire()
lock3.acquire()

t1=Task1()
t2=Task2()
t3=Task3()

t1.start()
t2.start()
t3.start()
```

### blocking 参数的作用

在上面示例上，t2 设置为非阻塞。

```python
import threading
import time, functools

def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        start = time.time()
        func(*args, **kwargs)
        end = time.time()
        print('{} cost: {}'.format(func.__name__, end - start))
    return clocked

@clock
def tast1():
    with lock:
        print("T1 start")
        for i in range(10):
            time.sleep(0.2)
        print("T1 finish")

@clock
def tast2():
    lock.acquire(blocking=False)	# 设置非阻塞
    print("T2 start")
    for i in range(10):
        time.sleep(0.3)
    print("T2 finish")
    # lock.release()

@clock
def tast3():
    with lock:
        print("T3 start")
        for i in range(10):
            time.sleep(0.1)
        print("T3 finish")

if __name__ == "__main__":
    start = time.time()
    lock = threading.Lock()
    t1 = threading.Thread(target=tast1)
    t2 = threading.Thread(target=tast2)
    t3 = threading.Thread(target=tast3)
    t1.start()
    t2.start()
    t3.start()
    t1.join()
    t2.join()
    t3.join()
    end = time.time()
    print("All done, cost:", end-start)
```

输出结果：

```text
T1 start
T2 start
T1 finish
tast1 cost: 2.0992367267608643
T3 start
T2 finish
tast2 cost: 3.1554200649261475
T3 finish
tast3 cost: 3.1894009113311768
All done, cost: 3.1894009113311768
```

t1 和 t2 进程同时执行。t1 执行完毕后，t3才开始执行。

当一个线程设置为非阻塞，那么当其他会产生阻塞的线程尝试请求锁时，会返回False，并释放锁，与之同时执行。

***

## RLock Objects

可重入锁，也叫做“递归锁”。类似于Lock对象，具有 acquire 和 release 两个方法，都是确保多线程多共享资源的访问。

### Lock 和 RLock 区别

Lock 是阻塞其他线程对共享资源的访问，且同一线程只能acquire一次，如多于一次就出现了死锁，程序无法继续执行。
为了保证线程对共享资源的独占，又避免死锁的出现，就有了RLock。RLock允许在同一线程中被多次acquire，而Lock却不允许这种情况。如果使用RLock，那么acquire和release必须成对出现，即调用了n次acquire，必须调用n次的release才能真正释放所占用的琐。。

RLock 对比 Lock 有是三个特点：

* 谁拿到谁释放。如果线程A拿到锁，线程B无法释放这个锁，只有A可以释放
* 同一线程可以不被阻塞的多次拿到该锁，即可以acquire多次，例如嵌套、递归使用。
* release()需要调用与acquire()相同的次数才能释放锁，只有最后一次release才能改变RLock的状态为unlocked

### 死锁

Deadlock，又译为**死结**，计算机科学名词。当两个以上的运算单元，双方都在等待对方停止运行，以获取系统资源，但是没有一方提前退出时，就称为死锁。

Lock 在下面的情形下会发生死锁

```python
Lock.acquire()

Lock.acquire()

Lock.release()

Lock.release()
```

连续两次acquire请求，会导致死锁，因为第一次获得锁之后还没有释放，第二次再来申请，程序就阻塞在这里，导致第一次申请到的锁无法释放。

使用 RLock 就不会发生死锁，但是要保证，有多少次acquire就有多少次release。

### 示例

使用 Lock，那么以下情形将会发生死锁：

```python
import threading
import time

m_lock = threading.Lock()

def h():
    with m_lock:
        g()
        print('h')

def g():
    with m_lock:
        print('g')

h()
g()
```

改为使用 RLock:

```python
import threading
import time

m_lock = threading.RLock()	# 使用 RLock

def h():
    with m_lock:
        g()
        print('h')

def g():
    with m_lock:
        print('g')

h()
g()
```

执行结果正常：

```python
g
h
g
```

***

## Condition Objects

条件变量，比Lock, RLock更高级的功能， 用于复杂的线程间同步。

Condition 也提供了acquire(), release()方法，其含义与琐的acquire(), release()方法一致。除此之外，Condition还提供wait()、notify()、notifyAll()方法。注意：这些方法**只有在 lock 状态**，才能调用，否则将会报RuntimeError异常。

Condition类适合于生产者，消费者模型。 即Condition很适合那种**主动休眠，被动唤醒**的场景。

### 语法、方法

```text
class threading.Condition(lock=None)
```

创建一个条件变量。条件变量允许1个或者多个多线程等待，直到其他的线程将其唤醒。

lock 参数可以传入 Lock object或者 RLock  object，这将会作为底层锁。默认使用 RLock 对象作为底层锁。

```text
acquire()/release()：获得/释放底层锁，调用内部琐对象的对应的方法而已

wait([timeout]):释放底层锁，然后阻塞，直到其他线程调用notify()或notify_all()或者超时，才会被唤醒继续运行。一旦被唤醒，wait()就会获取锁，并返回。除非超时，否则将会一直返回True

wait_for(predicate, timeout=None):阻塞，直至条件值为True。predicate 必须可调用，而且返回值是布尔值。

notify(n=1):唤醒等待条件变量的线程，那些阻塞的线程接到这个通知之后会开始运行,最多则唤醒n个等待的线程。不会主动释放锁。

notifyAll(): 唤醒所有阻塞的线程。不会主动释放锁。
```

被唤醒的线程获得锁才能继续执行。notify()**不会**释放锁，必须由其线程调用 release()。

 一个**Condition实例的内部实际上维护了两个队列**，**一个是等待锁队列**(实际上，内部其实就是维护这个等待锁队列) ，**另一个队列可以叫等待条件队列**，在这队列中的节点都是由于（某些条件不满足而）线程自身调用wait方法阻塞的线程（记住是自身阻塞）。最重要的Condition方法是wait和notify方法。另外condition还需要lock的支持， 如果你构造函数没有指定lock，condition会默认给你配一个rlock。   

解释条件同步机制的一个很好的例子就是生产者/消费者（producer/consumer）模型。

官方伪代码：

```python
# Consume one item
with cv:
    while not an_item_is_available():
        cv.wait()
    get_an_available_item()

# Produce one item
with cv:
    make_an_item_available()
    cv.notify()
```

### 示例

```python
import threading

class XiaoMing(threading.Thread):
    def __init__(self, cond):
        super().__init__(name="小明")
        self.cond = cond

    def run(self):
        with self.cond:
            self.cond.wait()
            print("{} : 在 ".format(self.name))
            self.cond.notify()

            self.cond.wait()
            print("{} : 好啊 ".format(self.name))
            self.cond.notify()

            
class XiaoHong(threading.Thread):
    def __init__(self, cond):
        super().__init__(name="小红")
        self.cond = cond

    def run(self):
        with self.cond:
            print("{} : 小明同学 ".format(self.name))
            self.cond.notify()
            self.cond.wait()

            print("{} : 你好 ".format(self.name))
            self.cond.notify()
            self.cond.wait()


if __name__ == "__main__":
    cond = threading.Condition()
    xiaoming = XiaoMing(cond)
    xiaohong = XiaoHong(cond)

    xiaoming.start()
    xiaohong.start()
```

输出结果：

```text
小红 : 小明同学
小明 : 在
小红 : 你好
小明 : 好啊
```

生产者/消费者（producer/consumer）模型

```python
import time, random, threading

class Producer(threading.Thread):
    def __init__(self, integers, condition):
        super().__init__()
        self.integers = integers
        self.condition = condition

    def run(self):
        while True:
            integer = random.randint(0, 256)
            self.condition.acquire()	# 获取条件锁
            print('condition acquired by ', self.name)
            self.integers.append(integer)
            print('{} appended to list by {}'.format(integer, self.name))
            print('condition notified by ', self.name)
            self.condition.notify()	    # 唤醒消费者线程
            print('condition released by ', self.name)
            self.condition.release()	# 释放条件锁
            time.sleep(1)		        # 暂停1秒钟


class Consumer(threading.Thread):
     def __init__(self, integers, condition):
        threading.Thread.__init__(self)
        self.integers = integers
        self.condition = condition

     def run(self):
        while True:
            self.condition.acquire()	# 获取条件锁
            print('condition acquired by ', self.name)
            while True:
                if self.integers:	    # 判断是否有整数
                    integer = self.integers.pop()
                    print('{} popped from list by {}'.format(integer, self.name))
                    break				# 跳出循环，将运行释放锁
                print('condition wait by ', self.name)
                self.condition.wait()	# 等待商品，并且释放资源
            print('condition released by ', self.name)
            self.condition.release()	# 最后释放条件锁

def main():
    integers = []
    condition = threading.Condition()
    t1 = Producer(integers, condition)
    t2 = Consumer(integers, condition)
    t1.start()
    t2.start()
    t1.join()
    t2.join()

if __name__ == '__main__':
     main()
```

```bash
condition acquired by  Thread-1
215 appended to list by Thread-1
condition notified by  Thread-1
condition released by  Thread-1
condition acquired by  Thread-2
215 popped from list by Thread-2
condition released by  Thread-2
condition acquired by  Thread-2
condition wait by  Thread-2
condition acquired by  Thread-1
58 appended to list by Thread-1
condition notified by  Thread-1
condition released by  Thread-1
58 popped from list by Thread-2
condition released by  Thread-2
condition acquired by  Thread-2
condition wait by  Thread-2
...
```

***

## Semaphore Objects

信号量是一个基于计数器的同步原语。内部维护了一个条件变量和一个计数器。信号量同步基于内部计数器，默认值为1。每调用一次acquire()，计数器减1；每调用一次release()，计数器加1。当计数器为0时，acquire()调用被阻塞，直到其他线程调用release()。

通常用于控制进入数量的锁。例如进行IO操作时，只可以有1个写入，但是可以同时存在多个读取。

### 语法、方法

```python
class threading.Semaphore(value=1)
```

创建一个信号量d对象。value是计数器的初始值。信号量管理是一个原子计数器。release()调用次数-acquire()调用次数+初始值。计数器不会为负值。

```python
acquire(blocking=True, timeout=None)
```

获取信号量。

* 使用默认值时：
  * 如果计数器大于0，计数器减1，立即返回True。
  * 如果计数器值为0，阻塞，直到另一个线程调用release()方法为止。每次调用release()都会唤醒一个线程。
* `blocking=False`:不会阻塞。当其他会产生阻塞的线程尝试请求时，返回False。

```python
release()
```

释放一个信号量，使内部计数器加1。

### 示例

限制了同时能访问资源的数量为3，其余线程只能阻塞。

```python
import time
from threading import Thread, Semaphore

sema = Semaphore(3)


def foo(tid):
    with sema:
        print('{} acquire sema'.format(tid))
        time.sleep(1)
    print('{} release sema'.format(tid))


threads = []

for i in range(5):
    t = Thread(target=foo, args=(i,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

输出结果：

```python
0 acquire sema
1 acquire sema
2 acquire sema
1 release sema
0 release sema
3 acquire sema
4 acquire sema
2 release sema
4 release sema
3 release sema
```

控制爬虫爬取速度：

```python
import threading
import time

class HtmlSpider(threading.Thread):
    def __init__(self, url, sem):
        super().__init__()
        self.url = url
        self.sem = sem

    def run(self):
        time.sleep(2)
        print("got html text success")
        self.sem.release()

class UrlProducer(threading.Thread):
    def __init__(self, sem):
        super().__init__()
        self.sem = sem

    def run(self):
        for i in range(20):
            self.sem.acquire()
            html_thread = HtmlSpider("https://baidu.com/{}".format(i), self.sem)
            print("{}--{}".format(i, self.sem))
            html_thread.start()

if __name__ == "__main__":
    sem = threading.Semaphore(3)
    url_producer = UrlProducer(sem)
    url_producer.start()
    url_producer.join()
```

***

## BoundedSemaphore

类似于Semaphore。不同在于，BoundedSemaphore 的release()操作的次数不能超过acquire()的操作次数，检查内部计数器的值，并保证它不会大于初始值，如果超了，就引发一个 ValueError。

***

## Event Objects

事件的同步机制：一个线程通知事件，其他线程等待事件。

全局定义了一个Flag，初始值是 False：

* 如果Flag的值为False，那么当程序执行wait()方法时就会阻塞
* 如果Flag值为True，线程不再阻塞。

这种锁，类似交通红绿灯（默认是红灯），它属于在红灯的时候一次性阻挡所有线程，在绿灯的时候，一次性放行所有排队中的线程。Event其实就是一个简化版的 Condition。Event没有锁，无法使线程进入同步阻塞状态。

### 语法、方法

```text
class threading.Event:创建一个事件对象

.set():设置flag为true，所有的调用wait()的线程被唤醒，不再阻塞

.clear():重置flag为false，随后使用wait()的线程将会阻塞

.wait(timeout=None):阻塞当前线程，直到flag值是true或者超时。除非超时，将会一直返回True

.is_set():当flag为true时，返回True
```

### 示例

模拟红绿灯

```python
import threading
import time

event = threading.Event()

def lighter():
    green_time = 5       # 绿灯时间
    red_time = 5         # 红灯时间
    event.set()          # 初始设为绿灯
    while True:
        print("绿灯亮...")
        time.sleep(green_time)
        event.clear()
        print("红灯亮...")
        time.sleep(red_time)
        event.set()

def run(name):
    while True:
        if event.is_set():      # 判断当前是否"放行"状态
            print("一辆[%s] 呼啸开过..." % name)
            time.sleep(1)
        else:
            print("一辆[%s]开来，看到红灯，无奈的停下了..." % name)
            event.wait()
            print("[%s] 看到绿灯亮了，瞬间飞起....." % name)

if __name__ == '__main__':

    light = threading.Thread(target=lighter,)
    light.start()

    for name in ['奔驰', '宝马', '奥迪']:
        car = threading.Thread(target=run, args=(name,))
        car.start()
```

前面的生产者和消费者的例子，现在把它转换成使用事件同步而不是条件同步。

```python
import time, random
from threading import Thread, Event

class Producer(Thread):
    def __init__(self, integers, event):
        super().__init__()
        self.integers = integers
        self.event = event

    def run(self):
        while True:
            integer = random.randint(0, 256)
            self.integers.append(integer)
            print('{} appended to list by {}'.format(integer, self.name))
            print('event set by ', self.name)
            self.event.set()		# 设置事件	
            self.event.clear()		# 发送事件
            print('event cleared by ', self.name)
            time.sleep(1)


class Consumer(Thread):
    def __init__(self, integers, event):
        super().__init__()
        self.integers = integers
        self.event = event

    def run(self):
        while True:
            self.event.wait()		# 等待事件被触发
            try:
                integer = self.integers.pop()
                print('{} popped from list by {}'.format(integer, self.name))
            except IndexError:
                # catch pop on empty list
                time.sleep(1)

def main():
    integers = []
    event = Event()
    t1 = Producer(integers, event)
    t2 = Consumer(integers, event)
    t1.start()
    t2.start()
    t1.join()
    t2.join()

if __name__ == '__main__':
    main()
```

***

## Timer Objects

Timer对象。指定n秒后执行某操作。

Timer 是 Thread 的子类。

```text
class threading.Timer(interval, function, args=None, kwargs=None)

interval:延迟时间

function:被调用函数

args/kwargs:传入function的参数
```

start（）方法启动定时器。如果在定时过程中想要取消该定时器，需要使用cancel()函数。

***

## Barrier Objects

Barrier会阻塞所有在该屏障上处于wait()状态的线程，直到指定数量的线程进入了该屏障，才会解除屏障，由此，这些线程才能继续运行。主要作用确保一定数量的线程同时开启。

This class provides a simple synchronization primitive for use by **a fixed number of threads that need to wait for each other**. Each of the threads tries to pass the barrier by calling the wait() method and will block until all of the threads have made their wait() calls. At this point, the threads are released simultaneously.

官方示例：

```python
b = Barrier(2, timeout=5)

def server():
    start_server()
    b.wait()
    while True:
        connection = accept_connection()
        process_server_connection(connection)

def client():
    b.wait()
    while True:
        connection = make_connection()
        process_client_connection(connection)
```

当 server() 和 client() 同时处于wait()状态，那么

```text
class threading.Barrier(parties, action=None, timeout=None)

parties:障碍要求的线程数量

action:可调用对象，在所有线程都到达barrier时被调用，仅被调用一次

timeout：超时时间
```

` wait(timeout=None)`

当所有调用wait()的线程数量达到屏障要求的数值时，所有的线程都将被释放。

此处的timeout设置优先于构造函数中设置的参数。如果超时，那么屏障将会被破坏

` reset()`

重置屏障。

` abort()`

破坏屏障

` parties`

通过该屏障所需的线程数

` n_waiting`

当前在屏障中等待的线程数。

` broken`

如果屏障被破坏，返回True。

***

参考：

[Python3入门之线程threading常用方法](https://www.cnblogs.com/chengd/articles/7770898.html)

[Python threads synchronization: Locks, RLocks, Semaphores, Conditions, Events and Queues](http://www.laurentluce.com/posts/python-threads-synchronization-locks-rlocks-semaphores-conditions-events-and-queues/)

http://www.liujiangblog.com/course/python/79