subprocess模块主要用于创建子进程，并连接它们的输入、输出和错误管道，获取它们的返回状态。通俗地说就是通过这个模块，你可以在Python的代码里执行操作系统级别的命令

## subprocess.run()

大多数情况下，推荐使用`run()`方法调用子进程，执行操作系统命令。在更高级的使用场景，你还可以使用Popen接口。其实`run()`方法在底层调用的就是Popen接口。

```python
subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, capture_output=False, shell=False, cwd=None, timeout=None, check=False, encoding=None, errors=None, text=None, env=None, universal_newlines=None)

args：表示要执行的命令。必须是一个字符串，字符串参数列表。

stdin、stdout和stderr：子进程的标准输入、输出和错误。其值可以是subprocess.PIPE、subprocess.DEVNULL、一个已经存在的文件描述符、已经打开的文件对象或者None。subprocess.PIPE表示为子进程创建新的管道。subprocess.DEVNULL表示使用os.devnull。默认使用的是None，表示什么都不做。另外，stderr可以合并到stdout里一起输出。

shell：如果该参数为True，将通过操作系统的shell执行指定的命令。

timeout：设置命令超时时间。如果命令执行时间超时，子进程将被杀死，并弹出TimeoutExpired异常。

check：如果该参数设置为True，并且进程退出状态码不是0，则弹出CalledProcessError异常。

encoding:如果指定了该参数，则stdin、stdout和stderr可以接收字符串数据，并以该编码方式编码。否则只接收bytes类型的数据。
```

执行args参数所表示的命令，等待命令结束，并返回一个CompletedProcess类型对象。run()方法返回的不是我们想要的执行结果或相关信息，而是一个CompletedProcess类型对象。

### class subprocess.CompletedProcess

run()方法的返回值，表示一个进程结束了。`CompletedProcess`类有下面这些属性：

- args 启动进程的参数，通常是个列表或字符串。
- returncode 进程结束状态返回码。0表示成功状态。
- stdout 获取子进程的stdout。通常为bytes类型序列，None表示没有捕获值。如果你在调用run()方法时，设置了参数`stderr=subprocess.STDOUT`，则错误信息会和stdout一起输出，此时stderr的值是None。
- stderr 获取子进程的错误信息。通常为bytes类型序列，None表示没有捕获值。
- check_returncode() 用于检查返回码。如果返回状态码不为零，弹出`CalledProcessError`异常。

### subprocess.DEVNULL

一个特殊值，用于传递给stdout、stdin和stderr参数。表示使用`os.devnull`作为参数值。

### subprocess.PIPE

管道，可传递给stdout、stdin和stderr参数。

### subprocess.STDOUT

特殊值，可传递给stderr参数，表示stdout和stderr合并输出。

## 获取执行结果

run()方法返回的是一个CompletedProcess类型对象，不能直接获取我们通常想要的结果。要获取命令执行的结果或者信息，在调用run()方法的时候，请指定stdout=subprocess.PIPE。

```python
ret = subprocess.run('dir', shell=True, stdout=subprocess.PIPE)

ret.stdout.decode('gbk')
```

## 交互式输入

并不是所有的操作系统命令都像‘dir’或者‘ipconfig’那样单纯地返回执行结果，还有很多像‘python’这种交互式的命令，你要输入点什么，然后它返回执行的结果。使用run()方法怎么向stdin里输入？

stdin、stdout和stderr是三个文件句柄，可以像文件那样进行读写操作

run()方法的stdin参数可以接收一个文件句柄。

```python
import subprocess

with open('test.py') as f:
    ret = subprocess.run("python", stdin=f, stdout=subprocess.PIPE,shell=True)
    print(ret.stdout)
```

这样做，虽然可以达到目的，但是很不方便，也不是以代码驱动的方式。这个时候，我们可以使用Popen类。

## class subprocess.Popen()

用法和参数与run()方法基本类同，但是它的返回值是一个Popen对象，而不是`CompletedProcess`对象。

Popen对象的stdin、stdout和stderr是三个文件句柄，可以像文件那样进行读写操作。

```python
s = subprocess.Popen("ipconfig", stdout=subprocess.PIPE, shell=True)

s.stdout.read().decode("GBK")
```

通过`s.stdin.write()`可以输入数据，而`s.stdout.read()`则能输出数据。

```python
import subprocess

s = subprocess.Popen("python", stdout=subprocess.PIPE, stdin=subprocess.PIPE, shell=True)
s.stdin.write(b"import os\n")
s.stdin.write(b"print(os.environ)")
s.stdin.close()

out = s.stdout.read().decode("GBK")
s.stdout.close()
print(out)
```

***

摘抄：

http://www.liujiangblog.com/course/python/55