Linux系统的命令通常都是如下所示的格式：

```text
命令名称 [命名参数] [命令对象]
```

## 基础命令

1. 获取登录信息 - w / who / last/ lastb

```bash
qxt@ubuntu:~$ who
qxt      :0           2019-07-21 22:44 (:0)
qxt@ubuntu:~$ last
qxt      :0           :0               Sun Jul 21 22:44   still logged in
reboot   system boot  4.18.0-25-generi Sun Jul 21 22:43   still running
qxt      :0           :0               Sun Jul 21 22:42 - 22:43  (00:00)
reboot   system boot  4.18.0-25-generi Sun Jul 21 22:42 - 22:43  (00:01)

wtmp begins Sun Jul 21 22:42:37 2019
```

2. 查看使用的Shell - ps

```bash
qxt@ubuntu:~$ ps
   PID TTY          TIME CMD
  1663 pts/0    00:00:00 bash
  2053 pts/0    00:00:00 nl
  2121 pts/0    00:00:00 ps
```

3. 查看命令的说明和位置 - whatis / which / whereis

```bash
qxt@ubuntu:~$ whatis ps
ps (1)               - report a snapshot of the current processes.
qxt@ubuntu:~$ whereis ps
ps: /bin/ps /usr/share/man/man1/ps.1.gz
qxt@ubuntu:~$ which ps
/bin/ps

```

4. 清除屏幕上显示的内容 - clear

5. 查看帮助文档 - man / info / help / apropos

6. 查看系统和主机名 - uname / hostname

7. 时间和日期 - date / cal

8. 重启和关机 - reboot / shutdown

9. 退出登录 - exit / logout

10. 查看历史命令 - history

***

## 软件安装和配置

1. 使用包管理工具

2. 下载解压配置环境变量

```bash
[root ~]# wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.6.5.tgz
...
[root ~]# gunzip mongodb-linux-x86_64-rhel70-3.6.5.tgz
[root ~]# tar -xvf mongodb-linux-x86_64-rhel70-3.6.5.tar
...
[root ~]# vim .bash_profile
...
PATH=$PATH:$HOME/bin:$HOME/mongodb-linux-x86_64-rhel70-3.6.5/bin
export PATH
...
[root ~]# source .bash_profile
```

3. 源代码构建安装

```bash
[root ~]# yum install gcc
[root ~]# wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz
[root ~]# gunzip Python-3.6.5.tgz
[root ~]# tar -xvf Python-3.6.5.tar
[root ~]# cd Python-3.6.5
[root ~]# ./configure --prefix=/usr/local/python36 --enable-optimizations
[root ~]# yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
[root ~]# make && make install
...
[root ~]# ln -s /usr/local/python36/bin/python3.6 /usr/bin/python3
[root ~]# python3 --version
Python 3.6.5
[root ~]# python3 -m pip install -U pip
[root ~]# pip3 --version
```

说明：上面在安装好Python之后还需要注册PATH环境变量，将Python安装路径下bin文件夹的绝对路径注册到PATH环境变量中。注册环境变量可以修改用户主目录下的.bash_profile或者/etc目录下的profile文件，二者的区别在于前者相当于是用户环境变量，而后者相当于是系统环境变量。

***

## 文件和文件夹操作

1. 创建/删除空目录 - mkdir / rmdir

| 参数 | 功能 |
| :--: | -- |
| `-m` | 创建目录时，设置权限 |
| `-p` | 创建多层目录，递归处理 |

```shell
# 创建权限为 711 的目录
mkdir -m 711 test
```

2. 创建/删除文件 - touch / rm

touch

|   参数   | 功能           |
| :------: | -------------- |
| `-mtime` | 更改内容的时间 |
| `-ctime` | 更改权限的时间 |
| `-atime` | 最后访问时间   |

rm

| 参数 | 功能                      |
| :--: | ------------------------- |
| `-f` | 强制删除                  |
| `-i` | 删除时，会先询问          |
| `-r` | 递归删除,常用于目录的删除 |

3. 切换和查看当前工作目录  - cd / pwd

cd

|   参数   | 功能                      |
| :------: | ------------------------- |
| `../xxx` | 使用相对路径，打开xxx目录 |
|   `..`   | 返回上级目录              |
|   `~`    | 返回根目录                |

pwd

| 参数 | 功能                                       |
| :--: | ------------------------------------------ |
|  -P  | 显示出确实的路径，而非使用连结 (link) 路径 |

4. 查看目录内容 - ls

|    参数     | 功能                                                   |
| :---------: | ------------------------------------------------------ |
|    `-a`     | 列出全部的文件，包括隐藏( 开头为 . 的文件)             |
|    `-d`     | 仅列出目录                                             |
|    `-l`     | 显示文件属性                                           |
|    `-R`     | 遇到目录要进行递归展开（继续列出目录下面的文件和目录） |
| `-S` / `-t` | 按大小/时间排序                                        |

5. 查看文件内容 - cat / tac / head / tail / more / less / rev / od

cat / tac

```shell
cat [-AbeEnstTuv] [--help] [--version] fileName
```

| 参数 | 功能                                               |
| :--: | -------------------------------------------------- |
| `-n` | 由 1 开始对所有输出的行数编号                      |
| `-b` | 和 -n 相似，只不过对于空白行不编号                 |
| `-s` | 当遇到有连续两行以上的空白行，就代换为一行的空白行 |
| `-E` | 每行结束处显示 $                                   |
| `-T` | TAB 字符显示为 ^I                                  |

text1 的文档内容加上行号后输入 text2 ：

```shell
cat -n text1 > text2
```

清空 test2.txt 文档内容：

```shell
cat /dev/null > text2
```

tac 与 cat 命令刚好相反，文件内容从最后一行开始显示。

head / tail

head 取出文件前面几行

```bash
head [-n number] file
```

tail 取出文件后面几行

```bash
head [-n number] file
```

6. 拷贝/移动文件 - cp / mv

cp

```bash
cp [options] source1 source2 source3 .... directory
```

| 参数 | 功能                                                         |
| :--: | ------------------------------------------------------------ |
| `-a` | 同 -dpr 的意思（保留权限，复制软链接本身，递归复制）         |
| `-d` | 若来源档为连结档的属性(link file)，则复制连结档属性而非文件本身 |
| `-p` | 复制文件及其属性，备份常用                                   |
| `-i` | 若目标已经存在时，会先询问                                   |
| `-r` | 用于目录复制，递归处理                                       |

mv

```bash
mv [options] source1 source2 source3 .... directory
```

| 参数 | 功能                                                    |
| :--: | ------------------------------------------------------- |
| `-f` | 强制移动，若目标存在，直接覆盖                          |
| `-i` | 若目标文件已经存在时，会先询问                          |
| `-u` | 若目标文件已经存在，且 source 比较新，才会升级 (update) |

7. 文件重命名 - rename

8. 查找文件和查找内容 - find / grep

9. 创建链接和查看链接 - ln / readlink

链接可以分为硬链接和软链接（符号链接）。硬链接可以认为是一个指向文件数据的指针，就像Python中对象的引用计数，每添加一个硬链接，文件的对应链接数就增加1，只有当文件的链接数为0时，文件所对应的存储空间才有可能被其他文件覆盖。我们平常删除文件时其实并没有删除硬盘上的数据，我们删除的只是一个指针，或者说是数据的一条使用记录，所以类似于“文件粉碎机”之类的软件在“粉碎”文件时除了删除文件指针，还会在文件对应的存储区域填入数据来保证文件无法再恢复。软链接类似于Windows系统下的快捷方式，当软链接链接的文件被删除时，软链接也就失效了。

10. 压缩/解压缩和归档/解归档 - gzip / gunzip / xz

11. 归档和解归档 - tar

归档（也称为创建归档）和解归档都使用`tar`命令，通常创建归档需要`-cvf`三个参数，其中`c`表示创建（create），`v`表示显示创建归档详情（verbose），`f`表示指定归档的文件（file）；解归档需要加上`-xvf`参数，其中`x`表示抽取（extract），其他两个参数跟创建归档相同。

12. 将标准输入转成命令行参数 - xargs

13. 显示文件或目录 - basename / dirname

14. 其他相关工具

nl

显示行号。

* `-b` ：指定行号指定的方式，主要有两种：

| 参数 | 功能 |
| :--: | -- |
| `-b a` | 表示不论是否为空行，也同样列出行号(类似 cat -n) |
| `-b t` | 如果有空行，空的那一行不要列出行号(默认值) |

* `-n`：列出行号表示的方法，主要有三种：

| 参数 | 功能 |
| :--: | -- |
| `-n ln` | 行号在荧幕的最左方显示 |
| `-n rn` | 行号在自己栏位的最右方显示，且不加 0 |
| `-n rz` | 行号在自己栏位的最右方显示，且加 0 |

* `-w`：行号栏位的占用的位数。



- **sort** - 对内容排序
- **uniq** - 去掉相邻重复内容
- **tr** - 替换指定内容为新内容
- **cut** / **paste** - 剪切/黏贴内容
- **split** - 拆分文件
- **file** - 判断文件类型
- **wc** - 统计文件行数、单词数、字节数
- **iconv** - 编码转换

***

## 管道和重定向

1. 管道的使用 - |

例子：查找当前目录下文件个数。

```bash
qxt@ubuntu:~$ find ./ | wc -l
1720
```

例子：列出当前路径下的文件和文件夹，给每一项加一个编号。

```bash
qxt@ubuntu:~$ ls | cat -n
     1	Desktop
     2	Documents
     3	Downloads
     4	examples.desktop
     5	Music
     6	Pictures
     7	Public
     8	Templates
     9	Videos
```

例子：查找record.log中包含AAA，但不包含BBB的记录的总数

```bash
[root ~]# cat record.log | grep AAA | grep -v BBB | wc -l
```

2. 输出重定向和错误重定向 - > / >> / 2>

```bash
[root ~]# cat readme.txt
banana
apple
grape
apple
grape
watermelon
pear
pitaya
[root ~]# cat readme.txt | sort | uniq > result.txt
[root ~]# cat result.txt
apple
banana
grape
pear
pitaya
watermelon
```

3. 输入重定向 - <

```bash
[root ~]# echo 'hello, world!' > hello.txt
[root ~]# wall < hello.txt
[root ~]#
Broadcast message from root (Wed Jun 20 19:43:05 2018):
hello, world!
[root ~]# echo 'I will show you some code.' >> hello.txt
[root ~]# wall < hello.txt
[root ~]#
Broadcast message from root (Wed Jun 20 19:43:55 2018):
hello, world!
I will show you some code.
```

4. 多重定向 - tee

下面的命令除了在终端显示命令`ls`的结果之外，还会追加输出到`ls.txt`文件中。

```bash
[root ~]# ls | tee -a ls.txt
```

***

## 别名

alias / unalias

```bash
qxt@ubuntu:~$ alias ll='ls -l'
qxt@ubuntu:~$ ll
total 44
drwxr-xr-x 2 qxt qxt 4096 May 12 23:08 Desktop
drwxr-xr-x 2 qxt qxt 4096 May 12 23:08 Documents
drwxr-xr-x 2 qxt qxt 4096 May 12 23:08 Downloads
-rw-r--r-- 1 qxt qxt 8980 May 12 22:58 examples.desktop
drwxr-xr-x 2 qxt qxt 4096 May 12 23:08 Music
drwxr-xr-x 2 qxt qxt 4096 May 12 23:08 Pictures
drwxr-xr-x 2 qxt qxt 4096 May 12 23:08 Public
drwxr-xr-x 2 qxt qxt 4096 May 12 23:08 Templates
drwxr-xr-x 2 qxt qxt 4096 May 12 23:08 Videos
qxt@ubuntu:~$ unalias ll
qxt@ubuntu:~$ ll
ll: command not found

```

***

## 文本处理

1. 字符流编辑器 - sed

sed是操作、过滤和转换文本内容的工具。假设有一个名为fruit.txt的文件，内容如下所示

```bash
qxt@ubuntu:~/test$ cat fruit.txt
banana
grape
apple
watermelon
orange
```

接下来，我们在第2行后面添加一个pitaya。

```bash
qxt@ubuntu:~/test$ sed '2a pitaya' fruit.txt
banana
grape
pitaya
apple
watermelon
orange
```

NOTE：刚才的命令和之前我们讲过的很多命令一样并没有改变fruit.txt文件，而是将添加了新行的内容输出到终端中，如果想保存到fruit.txt中，可以使用输出重定向操作。

在第2行前面插入一个waxberry。

```bash
qxt@ubuntu:~/test$ sed '2i pitaya' fruit.txt
banana
pitaya
grape
apple
watermelon
orange
```

删除第3行。

```bash
qxt@ubuntu:~/test$ sed '3d' fruit.txt
banana
grape
watermelon
orange
```

删除第2行到第4行。

```bash
qxt@ubuntu:~/test$ sed '2,4d' fruit.txt
banana
orange
```

将文本中的字符`a`替换为`@`。

```bash
qxt@ubuntu:~/test$ sed 's#a#@#' fruit.txt 
b@nana
gr@pe
@pple
w@termelon
or@nge
```

将文本中的字符`a`替换为`@`，使用全局模式。

```bash
qxt@ubuntu:~/test$ sed 's#a#@#g' fruit.txt 
b@n@n@
gr@pe
@pple
w@termelon
or@nge
```

2. 模式匹配和处理语言 - awk

awk是一种编程语言，也是Linux系统中处理文本最为强大的工具。通过该命令可以从文本中提取出指定的列、用正则表达式从文本中取出我们想要的内容、显示指定的行以及进行统计和运算。

假设有一个名为fruit.txt的文件，内容如下所示。

```bash
qxt@ubuntu:~/test$ cat fruit.txt 
1 banana 100
2 grape 200
3 apple 300
4 watermelon 400
5 orange 500
```

显示文件的第3行。

```bash
qxt@ubuntu:~/test$ awk 'NR==3' fruit.txt
3 apple 300
```

显示文件的第2列。

```bash
qxt@ubuntu:~/test$ awk '{print $2}' fruit.txt
banana
grape
apple
watermelon
orange
```

显示文件的最后一列。

```bash
qxt@ubuntu:~/test$ awk '{print $NF}' fruit.txt 
100
200
300
400
500
```

输出末尾数字大于等于300的行。

```bash
qxt@ubuntu:~/test$ awk '{if($3 >= 300) {print $0}}' fruit.txt 
3 apple 300
4 watermelon 400
5 orange 500
```

上面展示的只是awk命令的冰山一角，更多的内容留给读者自己在实践中去探索。

***

## 用户管理

1. 创建和删除用户 - useradd / userdel

```bash
[root home]# useradd hellokitty
[root home]# userdel hellokitty
```

- `-d` - 创建用户时为用户指定用户主目录
- `-g` - 创建用户时指定用户所属的用户组

2. 创建和删除用户组 - groupadd / groupdel

用户组主要是为了方便对一个组里面所有用户的管理。

3. 修改密码 - passwd

```
[root ~]# passwd hellokitty
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

说明：输入密码和确认密码没有回显且必须一气呵成的输入完成（不能使用退格键），密码和确认密码需要一致。如果使用`passwd`命令时没有指定命令作用的对象，则表示要修改当前用户的密码。如果想批量修改用户密码，可以使用`chpasswd`命令。

- `-l` / `-u` - 锁定/解锁用户。
- `-d` - 清除用户密码。
- `-e` - 设置密码立即过期，用户登录时会强制要求修改密码。
- `-i` - 设置密码过期多少天以后禁用该用户。

4. 查看和修改密码有效期 - chage

设置hellokitty用户100天后必须修改密码，过期前15天通知该用户，过期后15天禁用该用户。

```bash
chage -M 100 -W 15 -I 15 hellokitty
```

5. 切换用户 - su

```bash
root@ubuntu:/home/qxt/test# su qxt
qxt@ubuntu:~/test$ 
```

6. 以管理员身份执行命令 - sudo

如果希望用户能够以管理员身份执行命令，用户必须要出现在sudoers名单中，sudoers文件在 /etc目录下，如果希望直接编辑该文件也可以使用下面的命令。

7. 显示用户与用户组的信息 - id

8. 给其他用户发消息 -write / wall

发送方：

```bash
[root ~]# write hellokitty
Dinner is on me.
Call me at 6pm.
```

接收方：

```bash
[hellokitty ~]$ 
Message from root on pts/0 at 17:41 ...
Dinner is on me.
Call me at 6pm.
EOF
```

9. 查看/设置是否接收其他用户发送的消息 - mesg

```bash
[hellokitty ~]$ mesg
is y
[hellokitty ~]$ mesg n
[hellokitty ~]$ mesg
is n
```

***

## 文件系统

目录结构

```text
/bin - 基本命令的二进制文件。
/boot - 引导加载程序的静态文件。
/dev - 设备文件。
/etc - 配置文件。
/home - 普通用户主目录的父目录。
/lib - 共享库文件。
/lib64 - 共享64位库文件。
/lost+found - 存放未链接文件。
/media - 自动识别设备的挂载目录。
/mnt - 临时挂载文件系统的挂载点。
/opt - 可选插件软件包安装位置。
/proc - 内核和进程信息。
/root - 超级管理员用户主目录。
/run - 存放系统运行时需要的东西。
/sbin - 超级用户的二进制文件。
/sys - 设备的伪文件系统。
/tmp - 临时文件夹。
/usr - 用户应用目录。
/var - 变量数据目录。
```

访问权限 - chmod

chmod 命令用法如下：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Linux/shell-chmod.png)


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

改变文件所有者 - chown

```bash
[root ~]# ls -l
...
-rw-r--r--  1 root root     54 Jun 20 10:06 readme.txt
...
[root ~]# chown hellokitty readme.txt
[root ~]# ls -l
...
-rw-r--r--  1 hellokitty root     54 Jun 20 10:06 readme.txt
...
```

***

## 磁盘管理

1. 列出文件系统的磁盘使用状况 - df

   ```bash
[root ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        40G  5.0G   33G  14% /
devtmpfs        486M     0  486M   0% /dev
tmpfs           497M     0  497M   0% /dev/shm
tmpfs           497M  356K  496M   1% /run
tmpfs           497M     0  497M   0% /sys/fs/cgroup
tmpfs           100M     0  100M   0% /run/user/0
   ```

2. 磁盘分区表操作 - fdisk

   ```bash
[root ~]# fdisk -l
Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000a42f4
   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    83884031    41940992   83  Linux
Disk /dev/vdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
   ```

3. 磁盘分区工具 - parted

4. 格式化文件系统 - mkfs

   ```bash
[root ~]# mkfs -t ext4 -v /dev/sdb
   ```

   - `-t` - 指定文件系统的类型。
   - `-c` - 创建文件系统时检查磁盘损坏情况。
   - `-v` - 显示详细信息。

5. 文件系统检查 - fsck

6. 转换或拷贝文件 - dd

7. 挂载/卸载 - mount / umount

8. 创建/激活/关闭交换分区 - mkswap / swapon / swapoff

说明：执行上面这些命令会带有一定的风险，如果不清楚这些命令的用法，最好不用随意使用，在使用的过程中，最好对照参考资料进行操作，并在操作前确认是否要这么做。

***

## 配置服务

可以Linux系统下安装和配置各种服务，也就是说可以把Linux系统打造成数据库服务器、Web服务器、缓存服务器、文件服务器、消息队列服务器等等。Linux下的大多数服务都被设置为守护进程（驻留在系统后台运行，但不会因为服务还在运行而导致Linux无法停止运行），所以安装的服务通常名字后面都有一个字母`d`，它是英文单词`daemon`的缩写，例如：防火墙服务叫firewalld，我们之前安装的MySQL服务叫mysqld，Apache服务器叫httpd等。在安装好服务之后，可以使用`systemctl`命令或`service`命令来完成对服务的启动、停止等操作，具体操作如下所示。

1. 启动防火墙服务。

   ```bash
   [root ~]# systemctl start firewalld
   ```

2. 终止防火墙服务。

   ```bash
   [root ~]# systemctl stop firewalld
   ```

3. 重启防火墙服务。

   ```bash
   [root ~]# systemctl restart firewalld
   ```

4. 查看防火墙服务状态。

   ```bash
   [root ~]# systemctl status firewalld
   ```

5. 设置/禁用防火墙服务开机自启。

   ```
   [root ~]# systemctl enable firewalld
   Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
   Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
   [root ~]# systemctl disable firewalld
   Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
   Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
   ```

***

## 计划任务

1. 安全远程连接 - ssh

   ```bash
   [root ~]$ ssh root@120.77.222.217
   The authenticity of host '120.77.222.217 (120.77.222.217)' can't be established.
   ECDSA key fingerprint is SHA256:BhUhykv+FvnIL03I9cLRpWpaCxI91m9n7zBWrcXRa8w.
   ECDSA key fingerprint is MD5:cc:85:e9:f0:d7:07:1a:26:41:92:77:6b:7f:a0:92:65.
   Are you sure you want to continue connecting (yes/no)? yes
   Warning: Permanently added '120.77.222.217' (ECDSA) to the list of known hosts.
   root@120.77.222.217's password: 
   ```

2. 通过网络获取资源 - wget

   - `-b` 后台下载模式
   - `-O` 下载到指定的目录
   - `-r` 递归下载

3. 发送和接收邮件 - mail

4. 网络配置工具（旧） - ifconfig

   ```bash
   [root ~]# ifconfig eth0
   eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
           inet 172.18.61.250  netmask 255.255.240.0  broadcast 172.18.63.255
           ether 00:16:3e:02:b6:46  txqueuelen 1000  (Ethernet)
           RX packets 1067841  bytes 1296732947 (1.2 GiB)
           RX errors 0  dropped 0  overruns 0  frame 0
           TX packets 409912  bytes 43569163 (41.5 MiB)
           TX errors 0  dropped 0 overruns 0  carrier 0  collisions 
   ```

5. 网络配置工具（新） - ip

   ```bash
   [root ~]# ip address
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
       link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
       inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
   2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
       link/ether 00:16:3e:02:b6:46 brd ff:ff:ff:ff:ff:ff
       inet 172.18.61.250/20 brd 172.18.63.255 scope global eth0
          valid_lft forever preferred_lft forever
   ```

6. 网络可达性检查 - ping

   ```bash
   [root ~]# ping www.baidu.com -c 3
   PING www.a.shifen.com (220.181.111.188) 56(84) bytes of data.
   64 bytes from 220.181.111.188 (220.181.111.188): icmp_seq=1 ttl=51 time=36.3 ms
   64 bytes from 220.181.111.188 (220.181.111.188): icmp_seq=2 ttl=51 time=36.4 ms
   64 bytes from 220.181.111.188 (220.181.111.188): icmp_seq=3 ttl=51 time=36.4 ms
   --- www.a.shifen.com ping statistics ---
   3 packets transmitted, 3 received, 0% packet loss, time 2002ms
   rtt min/avg/max/mdev = 36.392/36.406/36.427/0.156 ms
   ```

7. 显示或管理路由表 - route

8. 查看网络服务和端口 - netstat / ss

   ```bash
   [root ~]# netstat -nap | grep nginx
   ```

9. 网络监听抓包 - tcpdump

10. 安全文件拷贝 - scp

```bash
[root ~]# scp root@1.2.3.4:/root/guido.jpg hellokitty@4.3.2.1:/home/hellokitty/pic.jpg
```

11. 文件同步工具 - rsync

   > 说明：使用`rsync`可以实现文件的自动同步，这个对于文件服务器来说相当重要。关于这个命令的用法，我们在后面讲项目部署的时候为大家详细说明。

12. 安全文件传输 - sftp

   ```bash
   [root ~]# sftp root@1.2.3.4
   root@1.2.3.4's password:
   Connected to 1.2.3.4.
   sftp>
   ```

   - `help`：显示帮助信息。
   - `ls`/`lls`：显示远端/本地目录列表。
   - `cd`/`lcd`：切换远端/本地路径。
   - `mkdir`/`lmkdir`：创建远端/本地目录。
   - `pwd`/`lpwd`：显示远端/本地当前工作目录。
   - `get`：下载文件。
   - `put`：上传文件。
   - `rm`：删除远端文件。
   - `bye`/`exit`/`quit`：退出sftp。

***

## 进程管理

1. 查看进程 - ps

```bash
[root ~]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Jun23 ?        00:00:05 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2     0  0 Jun23 ?        00:00:00 [kthreadd]
...
[root ~]# ps -ef | grep mysqld
root      4943  4581  0 22:45 pts/0    00:00:00 grep --color=auto mysqld
mysql    25257     1  0 Jun25 ?        00:00:39 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
```

2. 显示进程状态树 - pstree

```bash
[root ~]# pstree
systemd─┬─AliYunDun───18*[{AliYunDun}]
        ├─AliYunDunUpdate───3*[{AliYunDunUpdate}]
        ├─2*[agetty]
        ├─aliyun-service───2*[{aliyun-service}]
        ├─atd
        ├─auditd───{auditd}
        ├─dbus-daemon
        ├─dhclient
        ├─irqbalance
        ├─lvmetad
        ├─mysqld───28*[{mysqld}]
        ├─nginx───2*[nginx]
        ├─ntpd
        ├─polkitd───6*[{polkitd}]
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd───sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        └─tuned───4*[{tuned}]
```

3. 查找与指定条件匹配的进程 - pgrep

```bash
[root ~]$ pgrep mysqld
3584
```

4. 通过进程号终止进程 - kill

```bash
[root ~]$ kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
[root ~]# kill 1234
[root ~]# kill -9 1234
```

例子：用一条命令强制终止正在运行的Redis进程。

```bash
ps -ef | grep redis | grep -v grep | awk '{print $2}' | xargs kill
```

5. 通过进程名终止进程 - killall / pkill

结束名为mysqld的进程。

```bash
[root ~]# pkill mysqld
```

结束hellokitty用户的所有进程。

```bash
[root ~]# pkill -u hellokitty
```

说明：这样的操作会让hellokitty用户和服务器断开连接。

6. 将进程置于后台运行

- `Ctrl+Z` - 快捷键，用于停止进程并置于后台。
- `&` - 将进程置于后台运行。

```bash
[root ~]# mongod &
[root ~]# redis-server
...
^Z
[4]+  Stopped                 redis-server
```

7. 查询后台进程 - jobs

```bash
[root ~]# jobs
[2]   Running                 mongod &
[3]-  Stopped                 cat
[4]+  Stopped                 redis-server
```

8. 让进程在后台继续运行 - bg

```bash
[root ~]# bg %4
[4]+ redis-server &
[root ~]# jobs
[2]   Running                 mongod &
[3]+  Stopped                 cat
[4]-  Running                 redis-server &
```

9. 将后台进程置于前台 - fg

```bash
[root ~]# fg %4
redis-server
```

> 说明：置于前台的进程可以使用`Ctrl+C`来终止它。

10. 调整程序/进程运行时优先级 - nice / renice

11. 用户登出后进程继续工作 - nohup

```bash
[root ~]# nohup ping www.baidu.com > result.txt &
```

12. 跟踪进程系统调用情况 - strace

```bash
[root ~]# pgrep mysqld
8803
[root ~]# strace -c -p 8803
strace: Process 8803 attached
^Cstrace: Process 8803 detached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 99.18    0.005719        5719         1           restart_syscall
  0.49    0.000028          28         1           mprotect
  0.24    0.000014          14         1           clone
  0.05    0.000003           3         1           mmap
  0.03    0.000002           2         1           accept
------ ----------- ----------- --------- --------- ----------------
100.00    0.005766                     5           total
```

> 说明：这个命令的用法和参数都比较复杂，建议大家在真正用到这个命令的时候再根据实际需要进行了解。

13. 查看当前运行级别 - runlevel

```bash
[root ~]# runlevel
N 3
```

14. 实时监控进程占用资源状况 - top

```bash
[root ~]# top
top - 23:04:23 up 3 days, 14:10,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  65 total,   1 running,  64 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1016168 total,   191060 free,   324700 used,   500408 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   530944 avail Mem
...
```

- `-c` - 显示进程的整个路径。
- `-d` - 指定两次刷屏之间的间隔时间（秒为单位）。
- `-i` - 不显示闲置进程或僵尸进程。
- `-p` - 显示指定进程的信息。

***

## 系统诊断

1. 系统启动异常诊断 - dmesg

2. 查看系统活动信息 - sar

   ```
   [root ~]# sar -u -r 5 10
   Linux 3.10.0-957.10.1.el7.x86_64 (izwz97tbgo9lkabnat2lo8z)      06/02/2019      _x86_64_        (2 CPU)
   
   06:48:30 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
   06:48:35 PM     all      0.10      0.00      0.10      0.00      0.00     99.80
   
   06:48:30 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
   06:48:35 PM   1772012   2108392     54.33    102816   1634528    784940     20.23    793328   1164704         0
   ```

   - `-A` - 显示所有设备（CPU、内存、磁盘）的运行状况。
   - `-u` - 显示所有CPU的负载情况。
   - `-d` - 显示所有磁盘的使用情况。
   - `-r` - 显示内存的使用情况。
   - `-n` - 显示网络运行状态。

3. 查看内存使用情况 - free

   ```bash
   [root ~]# free
                 total        used        free      shared  buff/cache   available
   Mem:        1016168      323924      190452         356      501792      531800
   Swap:             0           0           0
   ```

4. 虚拟内存统计 - vmstat

   ```bash
   [root ~]# vmstat
   procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
    r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
    2  0      0 204020  79036 667532    0    0     5    18  101   58  1  0 99  0  0
   ```

5. CPU信息统计 - mpstat

   ```bash
   [root ~]# mpstat
   Linux 3.10.0-957.5.1.el7.x86_64 (iZ8vba0s66jjlfmo601w4xZ)       05/30/2019      _x86_64_        (1 CPU)
   
   01:51:54 AM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
   01:51:54 AM  all    0.71    0.00    0.17    0.04    0.00    0.00    0.00    0.00    0.00   99.07
   ```

6. 查看进程使用内存状况 - pmap

   ```bash
   [root ~]# ps
     PID TTY          TIME CMD
    4581 pts/0    00:00:00 bash
    5664 pts/0    00:00:00 ps
   [root ~]# pmap 4581
   4581:   -bash
   0000000000400000    884K r-x-- bash
   00000000006dc000      4K r---- bash
   00000000006dd000     36K rw--- bash
   00000000006e6000     24K rw---   [ anon ]
   0000000001de0000    400K rw---   [ anon ]
   00007f82fe805000     48K r-x-- libnss_files-2.17.so
   00007f82fe811000   2044K ----- libnss_files-2.17.so
   ...
   ```

7. 报告设备CPU和I/O统计信息 - iostat

   ```bash
   [root ~]# iostat
   Linux 3.10.0-693.11.1.el7.x86_64 (iZwz97tbgo9lkabnat2lo8Z)      06/26/2018      _x86_64_       (1 CPU)
   avg-cpu:  %user   %nice %system %iowait  %steal   %idle
              0.79    0.00    0.20    0.04    0.00   98.97
   Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
   vda               0.85         6.78        21.32    2106565    6623024
   vdb               0.00         0.01         0.00       2088          0
   ```

8. 显示所有PCI设备 - lspci

   ```bash
   [root ~]# lspci
   00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
   00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
   00:01.1 IDE interface: Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
   00:01.2 USB controller: Intel Corporation 82371SB PIIX3 USB [Natoma/Triton II] (rev 01)
   00:01.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 03)
   00:02.0 VGA compatible controller: Cirrus Logic GD 5446
   00:03.0 Ethernet controller: Red Hat, Inc. Virtio network device
   00:04.0 Communication controller: Red Hat, Inc. Virtio console
   00:05.0 SCSI storage controller: Red Hat, Inc. Virtio block device
   00:06.0 SCSI storage controller: Red Hat, Inc. Virtio block device
   00:07.0 Unclassified device [00ff]: Red Hat, Inc. Virtio memory balloon
   ```

9. 显示进程间通信设施的状态 - ipcs

   ```bash
   [root ~]# ipcs
   
   ------ Message Queues --------
   key        msqid      owner      perms      used-bytes   messages    
   
   ------ Shared Memory Segments --------
   key        shmid      owner      perms      bytes      nattch     status      
   
   ------ Semaphore Arrays --------
   key        semid      owner      perms      nsems
   ```

***

## Linux命令行常用快捷键

| 快捷键     | 功能说明                                     |
| ---------- | -------------------------------------------- |
| tab        | 自动补全命令或路径                           |
| Ctrl+a     | 将光标移动到命令行行首                       |
| Ctrl+e     | 将光标移动到命令行行尾                       |
| Ctrl+f     | 将光标向右移动一个字符                       |
| Ctrl+b     | 将光标向左移动一个字符                       |
| Ctrl+k     | 剪切从光标到行尾的字符                       |
| Ctrl+u     | 剪切从光标到行首的字符                       |
| Ctrl+w     | 剪切光标前面的一个单词                       |
| Ctrl+y     | 复制剪切命名剪切的内容                       |
| Ctrl+c     | 中断正在执行的任务                           |
| Ctrl+h     | 删除光标前面的一个字符                       |
| Ctrl+d     | 退出当前命令行                               |
| Ctrl+r     | 搜索历史命令                                 |
| Ctrl+g     | 退出历史命令搜索                             |
| Ctrl+l     | 清除屏幕上所有内容在屏幕的最上方开启一个新行 |
| Ctrl+s     | 锁定终端使之暂时无法输入内容                 |
| Ctrl+q     | 退出终端锁定                                 |
| Ctrl+z     | 将正在终端执行的任务停下来放到后台           |
| !!         | 执行上一条命令                               |
| !数字      | 执行数字对应的历史命令                       |
| !字母      | 执行最近的以字母打头的命令                   |
| !$ / Esc+. | 获得上一条命令最后一个参数                   |
| Esc+b      | 移动到当前单词的开头                         |
| Esc+f      | 移动到当前单词的结尾                         |

***

参考：

[玩转Linux操作系统](https://github.com/jackfrued/Python-100-Days/blob/master/Day31-35/31-35.%E7%8E%A9%E8%BD%ACLinux%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F.md)，骆昊