socket是基于C/S架构的，也就是说进行socket网络编程，通常需要编写两个py文件，一个服务端，一个客户端。

![socket](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/socket%20TCP%20%20socket%20flow.png)

Python3以后，socket传递的都是bytes类型的数据，字符串需要先转换一下，`string.encode()`即可；另一端接收到的bytes数据想转换成字符串，只要`bytes.decode()`一下就可以。在正常通信时，`accept()`和`recv()`方法都是阻塞的。所谓的阻塞，指的是程序会暂停在那，一直等到有数据过来。

## Socket 标准库

### 创建 Socket 对象

```python
 socket.socket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=None)
```

所有的套接字都是通过 `socket.socket()` 函数创建。

* family: 套接字家族
  * `socket.AF_INET`：IPv4
  * `socket.AF_INET6`：IPv6
  * `socket.AF_UNIX`：系统进程间通信
* type: 套接字类型
  * `socket.SOCK_STREAM`：流套接字 。创建 TCP 套接字，必须使用其作为套接字类型。
  * `socket.SOCK_DGRAM`：数据报套接字。创建 UDP 套接字，必须使用其作为套接字类型。
  * `socket.SOCK_RAW`：原始套接字。
* protocol: 一般不填默认为0。如果是 0 ，则系统就会根据地址格式和套接类别,自动选择一个合适的协议。

| socket类型              | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `socket.AF_UNIX`        | 只能够用于单一的Unix系统进程间通信                           |
| `socket.AF_INET`        | IPv4                                                         |
| `socket.AF_INET6`       | IPv6                                                         |
| `socket.SOCK_STREAM`    | 流式socket , for TCP                                         |
| `socket.SOCK_DGRAM`     | 数据报式socket , for UDP                                     |
| `socket.SOCK_RAW`       | 原始套接字，普通的套接字无法处理ICMP、IGMP等网络报文，而`SOCK_RAW`可以；其次，`SOCK_RAW`也可以处理特殊的IPv4报文；此外，利用原始套接字，可以通过`IP_HDRINCL`套接字选项由用户构造IP头。 |
| `socket.SOCK_SEQPACKET` | 可靠的连续数据包服务                                         |
| 创建TCP Socket：        | `s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)`         |
| 创建UDP Socket：        | `s=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)`          |

***

### Socket 对象方法和属性

#### 服务器端套接字

| 方法                       | 作用                                                         |
| :------------------------- | ------------------------------------------------------------ |
| `socket.bind(address)`     | 绑定地址（格式取决于 address family）到套接字，且此套接字之前未被绑定 |
| `socket.listen([backlog])` | 允许服务器接受连接，开启网络监听。                           |
| `socket.accept()`          | 接收一个连接。The return value is a pair (conn, address) where conn is a new socket object usable to send and receive data on the connection, and address is the address bound to the socket on the other end of the connection. |

#### 客户端套接字

| 方法                      | 作用                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `socket.connect(address)` | Connect to a remote socket at address. (The format of address depends on the address family） |
| `socket.connect_ex()`     | connect()函数的扩展版本,出错时返回出错码,而不是抛出异常      |

#### 公共的套接字函数

* ` socket.recv(bufsize[, flags])`

  接收数据，数据以bytes类型返回，`bufsize`指定要接收的最大数据量。

* ` socket.recvfrom(bufsize[, flags])`

  Receive data from the socket.  

  The return value is a pair `(bytes, address)` where *bytes* is a bytes object representing the data received and *address* is the address of the socket sending the data.

    See the Unix manual page *recv(2)* for the meaning of the optional argument *flags*; it defaults to zero. (The format of *address* depends on the address family.)

* ` socket.send(bytes[, flags])`

  Send data to the socket.  

  The socket **must be connected to** a remote socket.  The optional *flags* argument has the same meaning as for `recv()` above. Returns the number of bytes sent. Applications are responsible for checking that all data has been sent; if only some of the data was transmitted, the application needs to attempt delivery of the remaining data. 

* `socket.sendall(bytes[, flags])`

  Send data to the socket. 

  The socket must be connected to a remote socket. The optional flags argument has the same meaning as for recv() above. 

  **Unlike send()**, this method continues to send data from bytes until either all data has been sent or an error occurs. None is returned on success. On error, an exception is raised, and there is no way to determine how much data, if any, was successfully sent.

* ` socket.sendto(bytes, address)`
  `socket.sendto(bytes, flags, address)`

  Send data to the socket. 

  The socket **should not be connected to** a remote socket, since the destination socket is specified by address. The optional flags argument has the same meaning as for recv() above. Return the number of bytes sent. (The format of address depends on the address family)

* ` socket.getpeername()`

  Return the remote address to which the socket is connected.  

  This is useful to find out the port number of a remote IPv4/v6 socket, for instance. (The format of the address returned depends on the address family)

* ` socket.getsockname()`

  Return the socket’s own address.

  This is useful to find out the port number of an IPv4/v6 socket, for instance. 

***

## 创建 TCP 服务器和客户端

TCP套接字就是使用TCP协议提供的传输服务来实现网络通信的编程接口。在Python中可以通过创建socket对象并指定type属性为`SOCK_STREAM`来使用TCP套接字。由于一台主机可能拥有多个IP地址，而且很有可能会配置多个不同的服务，所以作为服务器端的程序，需要在创建套接字对象后将其绑定到指定的IP地址和端口上。这里的端口并不是物理设备而是对IP地址的扩展，用于区分不同的服务，例如我们通常将HTTP服务跟80端口绑定，而MySQL数据库服务默认绑定在3306端口，这样当服务器收到用户请求时就可以根据端口号来确定到底用户请求的是HTTP服务器还是数据库服务器提供的服务。

### TCP 服务器

通用 TCP 服务器的伪代码：

```python
ss = socket()				# 创建服务器套接字对象
ss.bind()					# 套接字与地址绑定
ss.listen()					# 监听连接
inf_loop:					# 服务器无限循环
    cs = ss.accept()		# 接收客户端连接
    comm_loop:				# 通信循环
        cs.recv()/cs.send()	# 对话（接收/发送）
    cs.close()				# 关闭客户端套接字
ss.close()					# 关闭服务器套接字，可选
```

因为 TCP 是一种面向连接的通信系统，服务器需要占用一个端口等待客户端的请求，所以套接字必须绑定一个本地地址。基于以上的装备工作，TPC 服务器开启监听传入的连接。由此，进入无限循环。

```python
import socket

HOST = ''                 # 空白，表示可以使用任何可用地址
PORT = 50007
BUFSIZ = 1024

# 1.创建套接字对象并指定使用哪种传输服务
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    # 2.绑定IP地址和端口(端口用于区分不同的服务)
    s.bind((HOST, PORT))
    # 3.开启监听 - 监听客户端连接到服务器
    s.listen(1)
    while True:
        # 4.通过循环接收客户端的连接并作出相应的处理(提供服务)
        conn, addr = s.accept()
        with conn:
            print('Connected by', addr)
            while True:
                data = conn.recv(BUFSIZ)
                if not data: break
                conn.sendall(data)
```

### TCP 客户端

TCP 客户端伪代码：

```python
cs = socket()			# 创建客户端套接字
cs.connect()			# 尝试连接客户端
comm_loop:				# 通信循环
    cs.send()/cs.recv()	# 对话（发送/接收）
cs.close()				# 关闭客户端套接字
```

具体代码实现：

```python
import socket

HOST = 'localhost'    # The remote host
PORT = 50007          # The same port as used by the server
BUFSIZ = 1024

# 1.创建套接字对象默认使用IPv4和TCP协议
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    # 2.连接到服务器(需要指定IP地址和端口)
    s.connect((HOST, PORT))
    while True:
        print('Type:')
        re_data = input()
        if not re_data: break
        s.send(re_data.encode("utf8"))
        data = s.recv(BUFSIZ)
        print('Received', repr(data.decode("utf8")))
```

### 执行 TCP 服务器和客户端

首先启动服务器，在任何客户端试图连接之前。

### 多线程服务器

需要注意的是，上面的服务器并没有使用多线程或者异步I/O的处理方式，这也就意味着当服务器与一个客户端处于通信状态时，其他的客户端只能排队等待。很显然，这样的服务器并不能满足需求，需要的服务器是能够同时接纳和处理多个用户请求的。下面我们来设计一个使用多线程技术处理多个用户请求的服务器。

```python
import socket
import threading


def link_handler(link, client):     
    """
    该函数为线程需要执行的函数，负责具体的服务器和客户端之间的通信工作
    :param link: 当前线程处理的连接
    :param client: 客户端ip和端口信息，一个二元元组
    :return: None
    """
    print('Connected by ', client)
    while True:                 # 保持和客户端的通信状态
        data = link.recv(1024)
        if not data: break
        link.sendall(data)
    link.close()


ip_port = ('', 50007)
sk = socket.socket()            # 创建套接字
sk.bind(ip_port)                # 绑定服务地址
sk.listen(1)                    # 监听连接请求

print('wait...')

while True:                     # 不断的接受客户端发来的连接请求
    conn, address = sk.accept() # 等待连接，此处自动阻塞
    # 每当有新的连接过来，自动创建一个新的线程，
    # 并将连接对象和访问者的ip信息作为参数传递给线程的执行函数
    t = threading.Thread(target=link_handler, args=(conn, address))
    t.start()
```

***

## 创建 UDP 服务器和客户端

TCP和UDP都是提供端到端传输服务的协议，二者的差别就如同打电话和发短信的区别，后者不对传输的可靠性和可达性做出任何承诺从而避免了TCP中握手和重传的开销，所以在强调性能和而不是数据完整性的场景中（例如传输网络音视频数据），UDP可能是更好的选择。可能大家会注意到一个现象，就是在观看网络视频时，有时会出现卡顿，有时会出现花屏，这无非就是部分数据传丢或传错造成的。

### UDP 服务器

UDP 服务器不是面向连接，所以不需要 TCP 服务器那么的设置。

```python
ss = socket()						# 创建服务器套接字
ss.bind()							# 绑定服务器套接字
inf_loop:							# 服务器无限循环
    cs = ss.recvfrom()/cs.sendto()	# 对话（接收/发送）
ss.close()							# 关闭服务器套接字
```

具体代码实现：

```python
import socket

HOST = ''
PORT = 50007
BUFSIZ = 1024

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
    s.bind((HOST, PORT))

    while True:
        data, addr = s.recvfrom(BUFSIZ)
        print('Connected by', addr)
        print('From client:', data.decode("utf8"))
        re_data = input()
        s.sendto(re_data.encode("utf8"), addr)
```

### UDP 客户端

```python
cs = socket()					# 创建客户端套接字
comm_loop():					# 重新循环
    cs.sendto()/cs.recvfrom()	# 对话（发送/接收）
cs.close()						# 关闭客户端套接字
```

````python
import socket

HOST = 'localhost'
PORT = 50007
BUFSIZ = 1024

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
    while True:
        print('Type:')
        re_data = input()
        if not re_data: break
        s.sendto(re_data.encode("utf8"), (HOST, PORT))
        data, addr = s.recvfrom(BUFSIZ)
        print('From:{}Received:{}'.format(addr, repr(data.decode("utf8"))))
````

## 网络应用开发

### 发送电子邮件

发送邮件要使用SMTP（简单邮件传输协议），SMTP也是一个建立在TCP（传输控制协议）提供的可靠数据传输服务的基础上的应用级协议，它规定了邮件的发送者如何跟发送邮件的服务器进行通信的细节，而Python中的smtplib模块将这些操作简化成了几个简单的函数。

```python
from smtplib import SMTP
from email.header import Header
from email.mime.text import MIMEText


def main():
    # 请自行修改下面的邮件发送者和接收者
    sender = 'abcdefg@126.com'
    receivers = ['uvwxyz@qq.com', 'uvwxyz@126.com']
    message = MIMEText('用Python发送邮件的示例代码.', 'plain', 'utf-8')
    message['From'] = Header('王大锤', 'utf-8')
    message['To'] = Header('骆昊', 'utf-8')
    message['Subject'] = Header('示例代码实验邮件', 'utf-8')
    smtper = SMTP('smtp.126.com')
    # 请自行修改下面的登录口令
    smtper.login(sender, 'secretpass')
    smtper.sendmail(sender, receivers, message.as_string())
    print('邮件发送完成!')


if __name__ == '__main__':
    main()
```

如果要发送带有附件的邮件，那么可以按照下面的方式进行操作。

```python
from smtplib import SMTP
from email.header import Header
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart

import urllib


def main():
    # 创建一个带附件的邮件消息对象
    message = MIMEMultipart()
    
    # 创建文本内容
    text_content = MIMEText('附件中有本月数据请查收', 'plain', 'utf-8')
    message['Subject'] = Header('本月数据', 'utf-8')
    # 将文本内容添加到邮件消息对象中
    message.attach(text_content)

    # 读取文件并将文件作为附件添加到邮件消息对象中
    with open('/Users/Hao/Desktop/hello.txt', 'rb') as f:
        txt = MIMEText(f.read(), 'base64', 'utf-8')
        txt['Content-Type'] = 'text/plain'
        txt['Content-Disposition'] = 'attachment; filename=hello.txt'
        message.attach(txt)
    # 读取文件并将文件作为附件添加到邮件消息对象中
    with open('/Users/Hao/Desktop/汇总数据.xlsx', 'rb') as f:
        xls = MIMEText(f.read(), 'base64', 'utf-8')
        xls['Content-Type'] = 'application/vnd.ms-excel'
        xls['Content-Disposition'] = 'attachment; filename=month-data.xlsx'
        message.attach(xls)
    
    # 创建SMTP对象
    smtper = SMTP('smtp.126.com')
    # 开启安全连接
    # smtper.starttls()
    sender = 'abcdefg@126.com'
    receivers = ['uvwxyz@qq.com']
    # 登录到SMTP服务器
    # 请注意此处不是使用密码而是邮件客户端授权码进行登录
    # 对此有疑问的读者可以联系自己使用的邮件服务器客服
    smtper.login(sender, 'secretpass')
    # 发送邮件
    smtper.sendmail(sender, receivers, message.as_string())
    # 与邮件服务器断开连接
    smtper.quit()
    print('发送完成!')


if __name__ == '__main__':
    main()
```

### 发送短信

发送短信也是项目中常见的功能，网站的注册码、验证码、营销信息基本上都是通过短信来发送给用户的。s

***

参考：

《python 核心编程》

[网络编程入门](https://github.com/jackfrued/Python-100-Days/blob/master/Day01-15/14.%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E5%85%A5%E9%97%A8%E5%92%8C%E7%BD%91%E7%BB%9C%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91.md)，骆昊