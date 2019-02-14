为了满足我们对多线程网络服务器的需求，提供了`socketserver`模块。socketserver在内部使用IO多路复用以及多线程/进程机制，实现了并发处理多个客户端请求的socket服务端。每个客户端请求连接到服务器时，socketserver服务端都会创建一个“线程”或者“进程” 专门负责处理当前客户端的所有请求。

对于socketserver模块，我们最常使用的是`ThreadingTCPServer`类。它本身一句代码都没有，一切的功能都从两个父类里继承，`ThreadingMixIn`为它提供了多线程能力，`TCPServer`为它提供基本的socket通信能力。ThreadingTCPServer实现的Soket服务器内部会为每个客户端创建一个线程，该线程用来和客户端进行交互。服务器相当于一个总管，在接收连接并创建新的线程后，就撒手不管了，后面的通信就是线程和客户端之间的连接了，一定要理解这一点！

使用ThreadingTCPServer的要点:

- 创建一个继承自`socketserver.BaseRequestHandler`的类；
- 这个类中必须定义一个名字为`handle`的方法，不能是别的名字！
- 将这个类，连同服务器的ip和端口，作为参数传递给`ThreadingTCPServer()`构造器
- 手动启动ThreadingTCPServer。

```python
import socketserver

class MyServer(socketserver.BaseRequestHandler):
    """
    必须继承socketserver.BaseRequestHandler类
    """
    def handle(self):
        """
        必须实现这个方法！
        """
        conn = self.request         # request里封装了所有请求的数据
        print('Connected by ', conn)
        while True:
            data = conn.recv(1024)
            if not data:break
            conn.sendall(data)

if __name__ == '__main__':
    # 创建一个多线程TCP服务器
    server = socketserver.ThreadingTCPServer(('127.0.0.1', 50007), MyServer)
    print("wait...")
    # 启动服务器，服务器将一直保持运行状态
    server.serve_forever()
```

客户端的代码很好理解，和前面一样样的，关键是服务器端。

分析一下服务器端的代码，核心要点有这些：

- 连接数据封装在`self.request`中！调用send()和recv()方法都是通过`self.request`对象。
- `handle()`方法是整个通信的处理核心，一旦它运行结束，当前连接也就断开了（但其他的线程和客户端还正常），因此一般在此设置一个无限循环。
- 注意`server = socketServer.ThreadingTCPServer((‘127.0.0.1’,50007),MyServer)`中参数传递的方法。
- `server.serve_forever()`表示该服务器在正常情况下将永远运行。

socketserver模块还提供了`ThreadingUDPServer`类，用于提供多线程的UDP服务。还有`ForkingTCPServer`类，当操作系统支持fork操作的时候，可以实现多进程服务器。他们的用法和`ThreadingTCPServer`基本类似。

***

参考：

http://www.liujiangblog.com/course/python/77