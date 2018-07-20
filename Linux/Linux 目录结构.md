Linux 以树形目录结构的形式来构造整个系统。目录树的起始点为根目录(/, root)，每一个目录不仅能使用本地磁盘分区的文件系统，也能使用网络的文件系统。而，win 以存储介质为主，主要以盘符及分区实现文件管理，再下面才是目录。

root 在 Linux 里面的意义很多。如果从“账号”的角度来看，root指“系统管理员”身份，如果以“目录”的角度来看，root 指的是根目录，就是 / 。

***

### **FHS**

Linux 以树形结构组织文件，文件系统的最顶端是 /，/ 也是文件系统的入口，所有的目录、文件、设备都在/之下。系统中的所有目录按按照一定的类别有规律的组织和命名。这些目录结构和命令根据的原则即是 Filesystem Hierarchy Standard（文件系统层次化标准）。

FHS定义了两层规范：

* 文件数据放置。 / 下面的各个目录应该要放什么文件数据，例如 /etc 应该要放置设置文件，/bin 与 /sbin 则应该要放置可执行文件等等。

* 子目录定义。针对/usr及/var这两个目录的子目录来定义。例如/var/log放置系统登录文件、/usr/share放置共享数据等等。

FHS依据文件系统使用的频繁与否与是否允许使用者随意更动， 而将目录定义成为四种交互作用的形态。

<div align="center"><img src="https://wx3.sinaimg.cn/mw690/af9e9c30ly1fpq71ovcxfj20lf08dmxq.jpg" width = "550" height = "250" alt="图片名称" align=center />
</div>

* **可分享的**。可以分享给其他系统挂载使用的目录，包括执行文件与用户的邮件等数据。
* **不可分享的**。仅与个人机器有关，配置文件或者是与程序有关的socket文件等。
* **不变的**。有些数据是不会经常变动的，跟随 distribution 而不变动。例如函式库、文件说明文件、系统管理员所管理的主机服务配置文件等等。
* **可变动的**。经常改变的数据，例如登录文件、一般用户可自行收受的新闻组等。

***

### **Linux目录**

Linux 目录结构如下：

<div align="center"><img src="https://wx3.sinaimg.cn/mw690/af9e9c30ly1fpq7cqu4bhj20qv147dir.jpg" width = "550" height = "750" alt="图片名称" align=center />
</div>

因为根目录与系统启动有关，系统启动过程中仅有根目录会被挂载，其他分区则是在系统启动完成之后才会陆续的进行挂载。因此，系统启动过程相关的目录就不能够与根目录放到不同的分区。

| 目录 | 内容 |
| ---- | :----- |
| /bin | 用户命令的目录 |
| /etc | 配置文件 |
| /dev | 设备文件 |
| /lib | 程序的函数库和模块 |
| /sbin | 系统执行文件 |

***

### **常用的目录和文件**

| 目录/文件 | 内容 |
| ---- | :----- |
| etc/motd | 登录提醒 |
| etc/group | 设定用户的组名与相关信息 |
| etc/passwd | 用户账号信息文件 |
| etc/shadow | 用户密码信息文件 |
| etc/gshadow | 组密码文件 |
| etc/hosts | ip与域名（主机名）解析表 |
| etc/fstab | 开机自动挂载列表 |
| etc/rc.local | 开机自启动文件、命令、脚本 |
| etc/inittab | 开机运行级别配置文件 |
| etc/init.d | 服务启动命令脚本目录 |
| etc/profile | 全局环境变量 |
| usr/local | 编译安装软件默认安装目录 |
| var/log/message | 系统日志 |
| var/log/secure | 系统安全日志 |
| var/spool/cron/root | 定时任务，root目录 |
| proc/cpuinfo | cpu信息 |
| proc/meminfo | 系统内存信息 |
| proc/loadavg | cpu负载程度 |
| proc/mounts | 挂载信息 |

***

### **文件属性**

<div align="center"><img src="https://wx4.sinaimg.cn/mw690/af9e9c30ly1fpspx2p5usj20j0061750.jpg" width = "550" height = "170" alt="图片名称" align=center />
</div>

列中的第一个字符代表这个文件是“目录、文件或链接文件等”：

* d 是目录；
* \- 是文件；
* | 是连接文件(linkfile)；
* b 是设备文件里面的可供存储的接口设备；
* c 鱼设备文件里面的串口端口设备，例如键盘、鼠标。

接下来的字符中，3个为一组，且取自”rwx”：

* r代表可读(read)；
* w代表可写(write)；
* x代表可执行(execute)；

这3个权限的位置不会改变，如果没有相应的权限，就会显示为“-”。这三者对应owner/group/others。

***

### **更改文件属性**

Linux 文件权限属性有两种设置方法，一种是数字，一种是符号。

Linux 文件的基本权限就有九个，分别是owner/group/others三种身份各有自己的read/write/execute权限。

数字可以代表各个权限，各权限的分数对照如下：

| 字符 | 数字 |
| -- | :-- |
| r | 4 |
| w | 2 |
| x | 1 |

每种身份各自的三个权限(r/w/x)数值可以累加，例如当权限为： [-rwxrwx---] 分数则是：

* owner = rwx = 4+2+1 = 7
* group = rwx = 4+2+1 = 7
* others= --- = 0+0+0 = 0

这也表示此文件的权限是770。

chmod 命令用法如下：

<div align="center"><img src="https://wx4.sinaimg.cn/mw690/af9e9c30ly1fpsr5l1ggjj20ju02b3yb.jpg" width = "1000" height = "80" alt="图片名称" align=center />
</div>

其中，a 即全部的身份。

如果我们需要将文件权限设置为 -rwxr-xr--,则可以：

```shell
chmod 754 test
```

```shell
chmod u=rwx,g=rx,o=r  test
```

如果要去除执行权限：

```shell
chmod  a-x test
```