本文多数内容选自官方文档和《python 核心编程》。

## Socket 标准库

### 创建 Socket 对象

```python
 socket.socket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=None)
```

所有的套接字都是通过 `socket.socket()` 函数创建。

* family: 套接字家族
  * socket.AF_INET：IPv4
  * socket.AF_INET6：IPv6
  * socket.AF_UNIX：系统进程间通信
* type: 套接字类型
  * socket.SOCK_STREAM：流套接字 。创建 TCP 套接字，必须使用其作为套接字类型。
  * socket.SOCK_DGRAM：数据报套接字。创建 UDP 套接字，必须使用其作为套接字类型。
  * socket.SOCK_RAW：原始套接字。
* protocol: 一般不填默认为0。如果是 0 ，则系统就会根据地址格式和套接类别,自动选择一个合适的协议。

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



#### 公共的套接字函数

* ` socket.recv(bufsize[, flags])`

  Receive data from the socket.  

  The return value is a bytes object representing the data received.  The maximum amount of data to be received at once is specified by *bufsize*. 

  For best match with hardware and network realities, the value of  *bufsize* should be a relatively small power of 2.

* ` socket.recvfrom(bufsize[, flags])`

  Receive data from the socket.  

  The return value is **a pair `(bytes, address)`** where *bytes* is a bytes object representing the data received and *address* is the address of the socket sending the data.

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

![socket](https://wx2.sinaimg.cn/mw690/af9e9c30ly1fy387t9fxgj20jz0sj3zn.jpg)

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

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen(1)
    while True:
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

with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as s:
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

***

## 创建 UDP 服务器和客户端

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

