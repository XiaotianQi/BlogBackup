### **Linux系统启动过程**

学习[阮一峰博客《Linux 的启动流程》](http://www.ruanyifeng.com/blog/2013/08/linux_boot_process.html)的简要笔记。

Linux系统的启动过程可以分为5个阶段：

* 加载内核
* 启动初始化进程
* 确定运行级别
* 加载开机启动程序
* 用户登录

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Linux/start-up%20process.png)

***

#### /boot

操作系统接管硬件以后，首先读入 /boot 目录下的内核文件。

***

#### init进程

开始运行第一个程序 /sbin/init，它的作用是初始化系统环境。由于init是第一个运行的程序，它的进程编号（pid）就是1。其他所有进程都从它衍生，都是它的子进程。

***

#### 运行级别

Linux启动时根据"运行级别"，确定不同的场合启动哪些程序。init进程的一大任务，就是去运行这些开机启动的程序。Linux预置七种运行级别（0-6）。一般来说，0是关机，1是单用户模式（也就是维护模式），6是重启。运行级别2-5，各个发行版不太一样。

***

#### /etc/init.d

七种预设的"运行级别"各自有一个目录，存放需要开机启动的程序。七个 /etc/rcN.d 目录里列出的程序，都设为链接文件，指向另外一个目录 /etc/init.d ，真正的启动脚本都统一放在这个目录中。init进程逐一加载开机启动程序，其实就是运行这个目录里的启动脚本。/etc/init.d 这个目录名最后一个字母d，是directory的意思，表示这是一个目录，用来与程序 /etc/init 区分。

***

#### 用户登录

* 命令行登录：init进程调用getty程序（意为get teletype），让用户输入用户名和密码。输入完成后，再调用login程序，核对密码。如果密码正确，就从文件 /etc/passwd 读取该用户指定的shell，然后启动这个shell。
* ssh登录：这时系统调用sshd程序，取代getty和login，然后启动shell。
* 图形界面登录：init进程调用显示管理器，Gnome图形界面对应的显示管理器为gdm（GNOME Display Manager），然后用户输入用户名和密码。如果密码正确，就读取/etc/gdm3/Xsession，启动用户的会话。

***

#### login shell

用户登录时打开的shell，就叫做login shell。

* 命令行登录：首先读入 /etc/profile，这是对所有用户都有效的配置；然后依次寻找下面三个文件，这是针对当前用户的配置。~/.bash_profile　　~/.bash_login　　~/.profile。这三个文件只要有一个存在，就不再读入后面的文件了。
* ssh登录：与第一种情况完全相同。
* 图形界面登录：只加载 /etc/profile 和 ~/.profile。也就是说，~/.bash_profile 不管有没有，都不会运行。

***

#### non-login shell

用户进入操作系统以后，常常会再手动开启一个shell。这个shell就叫做 non-login shell，意思是它不同于登录时出现的那个shell，不读取/etc/profile和.profile等配置文件。

non-login shell的重要性，不仅在于它是用户最常接触的那个shell，还在于它会读入用户自己的bash配置文件 ~/.bashrc。大多数时候，我们对于bash的定制，都是写在这个文件里面的。

***

#### Bash配置

打开文件 ~/.profile，可以看到下面的代码：

```shell
　　if [ -n "$BASH_VERSION" ]; then
　　　　if [ -f "$HOME/.bashrc" ]; then
　　　　　　. "$HOME/.bashrc"
　　　　fi
　　fi
```

上面代码先判断变量 $BASH_VERSION 是否有值，然后判断主目录下是否存在 .bashrc 文件，如果存在就运行该文件。第三行开头的那个点，是source命令的简写形式，表示运行某个文件，写成"source ~/.bashrc"也是可以的。

因此，只要运行～/.profile文件，～/.bashrc文件就会连带运行。但是上一节的第一种情况提到过，如果存在～/.bash_profile文件，那么有可能不会运行～/.profile文件。解决这个问题很简单，把下面代码写入.bash_profile就行了。

```shell
　　if [ -f ~/.profile ]; then
　　　　. ~/.profile
　　fi
```

这样一来，不管是哪种情况，.bashrc都会执行，用户的设置可以放心地都写入这个文件了。

Bash的设置之所以如此繁琐，是由于历史原因造成的。早期的时候，计算机运行速度很慢，载入配置文件需要很长时间，Bash的作者只好把配置文件分成了几个部分，阶段性载入。系统的通用设置放在 /etc/profile，用户个人的、需要被所有子进程继承的设置放在.profile，不需要被继承的设置放在.bashrc。