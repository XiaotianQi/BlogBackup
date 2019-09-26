**镜像（Image）**：类似于虚拟机中的镜像，是一个包含有文件系统的面向Docker引擎的只读模板。任何应用程序运行都需要环境，而镜像就是用来提供这种运行环境的。例如一个Ubuntu镜像就是一个包含Ubuntu操作系统环境的模板，同理在该镜像上装上Apache软件，就可以称为Apache镜像。

**容器（Container）**：类似于一个轻量级的沙盒，可以将其看作一个极简的Linux系统环境（包括root权限、进程空间、用户空间和网络空间等），以及运行在其中的应用程序。Docker引擎利用容器来运行、隔离各个应用。容器是镜像创建的应用实例，可以创建、启动、停止、删除容器，各个容器之间是是相互隔离的，互不影响。注意：镜像本身是只读的，容器从镜像启动时，Docker在镜像的上层创建一个可写层，镜像本身不变。

**仓库（Repository）**：类似于代码仓库，这里是镜像仓库，是Docker用来集中存放镜像文件的地方。注意与注册服务器（Registry）的区别：注册服务器是存放仓库的地方，一般会有多个仓库；而仓库是存放镜像的地方，一般每个仓库存放一类镜像，每个镜像利用tag进行区分，比如Ubuntu仓库存放有多个版本（12.04、14.04等）的Ubuntu镜像。

## 安装和卸载

Docker可以安装在Windows、Linux、Mac等各个平台上。具体可以查看文档[Install Docker](https://link.zhihu.com/?target=https%3A//docs.docker.com/engine/installation/)。安装完成之后，查看 Docker 的版本信息：

```bash
$ sudo docker version
Client: Docker Engine - Community
 Version:           19.03.1
 API version:       1.40
 Go version:        go1.12.5
 Git commit:        74b1e89
 Built:             Thu Jul 25 21:21:05 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.1
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.5
  Git commit:       74b1e89
  Built:            Thu Jul 25 21:19:41 2019
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.6
  GitCommit:        894b81a4b802e4eb2a91d1ce216b8817763c29fb
 runc:
  Version:          1.0.0-rc8
  GitCommit:        425e105d5a03fabd737a126ad93d62a9eeede87f
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

查看Docker的帮助信息：# docker --help。

如果这项服务没有启动，可以用下面的命令启动。

```bash
# service 命令
$ sudo service docker start

# systemctl 命令
$ sudo systemctl start docker
```

Docker 需要用户具有 sudo 权限，为了避免每次命令都输入`sudo`，可以把用户加入 Docker 用户组（[官方文档](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user)）。

```bash
$ sudo usermod -aG docker $USER
```

***

相关命令：

```bash
docker info

docker version

docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
--name:自定义名称
-i:以交互模式运行容器，通常与 -t 同时使用；
-t:为容器重新分配一个伪输入终端，通常与 -i 同时使用；
-d:容器在后台运行
-P:容器内部端口随机映射到主机的高端口
-p:容器内部端口绑定到指定的主机端口
-w:指定容器的工作目录

docker ps [OPTIONS]
-a:所有容器，包括未运行的
-l:最新容器

docker top
查看容器内部进程

docker start/stop/restart CONTAINER [CONTAINER...]

docker kill

docker attach

docker logs
-f:跟踪日志输出
--since:显示某个开始时间的所有日志
-t:显示时间戳
--tail:仅列出最新N条容器日志

docker exec
-d
-i
-t
```

注意，`docker run`命令具有自动抓取 image 文件的功能。如果发现本地没有指定的 image 文件，就会从仓库自动抓取。因此，前面的`docker image pull`命令并不是必需的步骤。

```bash
λ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:451ce787d12369c5df2a32c85e5a03d52cbcef6eb3586dd03075f3034f10adcd
Status: Downloaded newer image for hello-world:latest
```

***

## image

**Docker 把应用程序及其依赖，打包在 image 文件里面。**只有通过这个文件，才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。

image 是二进制文件。实际开发中，一个 image 文件往往通过继承另一个 image 文件，加上一些个性化设置而生成。举例来说，你可以在 Ubuntu 的 image 基础上，往里面加入 Apache 服务器，形成你的 image。

image 文件是通用的，一台机器的 image 文件拷贝到另一台机器，照样可以使用。一般来说，为了节省时间，我们应该尽量使用别人制作好的 image 文件，而不是自己制作。即使要定制，也应该基于别人的 image 文件进行加工，而不是从零开始制作。

使用docker国内镜像加速：`https://registry.docker-cn.com`。

```text
docker images [OPTIONS] [REPOSITORY[:TAG]]
-a,--all=false:列出本地所有的镜像(含中间映像层,默认情况下,过滤掉中间映像层)
-f,--filter=[]
--no-trunc=false:显示完整的 id
-q,--quiet=false:只显示 image id
```

```text
docker inspect [OPTIONS] CONTAINER|IMAGE [CONTAINER|IMAGE...]
获取容器/镜像的元数据
-f,--format=""
```

```text
docker rmi [OPTIONS] IMAGE [IMAGE...]
-f,--force=false
--no-prune=false:不移除该镜像的过程镜像,默认移除
示例：
删除所有ubuntu镜像：docker rmi $(docker images -q ubuntu)
```

```text
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
标记本地镜像,将其归入某一仓库
```

***

### 构建镜像

* 保存对容器的修改，并再次使用
* 能够自定义镜像
* 以软件的形式打包并分发服务及其运行环境

方法：

* 通过容器构建
* 通过 Dockerfile 文件构建

### 通过容器构建

```text
docker commit [OPTIONS] CONTAINER [REPOSITROY[:TAG]]
-a,--author=""		# 作者
-m,--message=""		# 描述信息
-p,--pause=true
```

### 通过 Dockerfile 文件构建

Dockerfile 是一个文本文件，可以理解为一种配置文件，用来告诉docker build命令应该执行哪些操作，配置 image。Docker 根据该文件生成二进制的 image 文件。

```bash
docker build [OPTIONS] PATH|URL|-
--force-rm-false:设置镜像过程中删除中间容器
--no-cache=false:创建镜像的过程不使用缓存
--pull=false:尝试去更新镜像的新版本
-q,--quilt=false:安静模式，成功后只输出镜像 ID
--rm=true:设置镜像成功后删除中间容器
-t,--tag=""
```

基本语法： 

| 语句       | 作用                  |
| ---------- | --------------------- |
| FROM       | base image            |
| MAINTAINER | 作者及其相关信息      |
| RUN        | 当前image中执行的命令 |
| CMD        | 执行命令              |
| ADD        | 添加文件              |
| COPY       | 拷贝文件              |
| EXPOSE     | 暴露端口              |
| WORKDIR    | 指定路径              |
| ENV        | 环境变量              |
| ENTRYPONT  | 容器入口              |
| USER       | 用户                  |
| VOLUME     | mount point           |

```text
FROM <image>
FROM <image>:<tag>
base image,必须是第一条非注释语句
```

```text
MAINTAINER <name>
作者及其相关信息
```

```text
RUN <command> (即:shell 模式)
	/bin/sh -c command
	eg: RUN echo hi
RUN ["executable", "param1", "param2"] (即:exec 模式)
	eg: RUN ["/bin/bash", "-c", "echo hi"]
```

```text
CMD command param1 param2 (即:shell 模式)

CMD ["executable", "param1", "param2"] (即:exec 模式)

CMD ["param1", "param2"] (与ENTRYPOINT搭配使用,指定的默认参数)
```

**RUN vs CMD**：

* RUN 命令在 image 文件的构建阶段执行，执行结果都会打包进入 image 文件；
* CMD 命令则是在容器启动后执行。

另外，一个 Dockerfile 可以包含多个`RUN`命令，但是只能有一个`CMD`命令。

注意，指定了`CMD`命令以后，`docker container run`命令就不能附加命令了，否则它会覆盖`CMD`命令。

```text
ENTRYPOINT command param1 param2 (即:shell 模式)

ENTRYPOINT ["executable", "param1", "param2"] (即:exec 模式)
```

ENTRYPOINT 中的命令不会被 RUN 命令中指定的参数覆盖。如果需要覆盖，则需要指定 `docker run --entrypoint` 参数，进行覆盖。

```text
EXPOSE <port> [<port>...]
指定运行该image容器的端口
但，仍需在RUN命令中，进行端口映射
```

```text
ADD <src>...<dest>

ADD ["<src>"..."<dest>"] (用于含有空格的文件路径)
```

```text
COPY <src>...<dest>

COPY ["<src>"..."<dest>"] (用于含有空格的文件路径)
```

**ADD vs COPY**:

* ADD 包含了类似tar的解压缩功能
* 如果单纯的负值文件，Docker则推荐使用COPY

```text
VOLUME ["/data"]
为基于image创建的容器添加卷,提供共享数据、数据持久化功能
```

```text
WORKDIR /path/to/workdir
由image创建新容器时,在容器内部设置工作目录，后续的指令均在此目录下执行,eg:CMD
通常使用绝对路径，
```

```text
ENV <key><value>
ENV <key>=<value>...
设置环境变量,构建容器和使用容器中，均有效
```

```text
USER daemon
指定容器的使用者,默认使用root用户
```

```text
ONBUILD [INSTRUCTION]
为image添加触发器。
当一个image被其他image作为base image时执行
```

***

### Dockerfile 构建过程

* 从base image运行一个容器；
* 执行下一条指令，对容器做出修改;
* 执行类似`docker commit`操作，提交一个新的镜像层；
* 再基于刚提交的 image，运行一个新的容器；
* 重复2-4步，直到所有指令执行完毕

其中，`docker build` 会删除构建中生成的中间层容器，但中间层image会被保留。由此，可以对image构建进行调试。

```text
docker history [iamge]
查看image构建过程
```

### 示例

以创建一个 nginx 容器为例：

在项目的根目录下，新建一个文本文件`.dockerignore`，写入下面的[内容](https://github.com/ruanyf/koa-demos/blob/master/.dockerignore)。

```bash
.git
```

上面代码表示，这三个路径要排除，不要打包进入 image 文件。如果你没有路径要排除，这个文件可以不新建。

然后，在项目的根目录下，新建一个 Dockerfile，写入下面的[内容](https://github.com/ruanyf/koa-demos/blob/master/Dockerfile)。

```bash
FROM ubuntu
MAINTAINER qxt
RUN apt-get update
RUN apt-get install -y nginx
COPY index.html /var/www/html
ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
EXPOSE 80
```

之后，执行：

```bash
docker build -t qxt/hi-nginx
```

执行完毕后，通过`docker images`命令可以查看已获取的 images。

最后：

```bash
λ docker run -d -p 80:80 qxt/hi-nginx
fbcaee401b604d92b38df8c6734fe3af34a7ece37d61abc94aedbd915663567f

λ curl http://localhost
'ABC 123'
```

***

## Container

image 文件生成的容器实例，本身也是一个文件，称为容器文件。也就是说，一旦容器生成，就会同时存在两个文件： image 文件和容器文件。而且关闭容器并不会删除容器文件，只是容器停止运行而已。

```text
# 列出本机所有容器，包括终止运行的容器
$ docker container ls --all

# 删除指定的容器文件
$ docker rm [containerID]
```

### 网络连接

**Docker 容器的网络基础---docker0**

网桥位于 OSI 网络模型中的数据链路层。

Linux 虚拟网桥的特点：

* 可以设置 IP 地址
* 相当于拥有一个隐藏的虚拟网卡

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Linux/docker%200.png)

**Docker 容器的的互联**

容器每次启动，其IP地址均会发生变化。

但是，使用如下命令启动容器，使得两个容器互联时：

```text
docker run --link=[CONTAINER_NAME]:[ALIAS] [IAMGE] [COMMAND]
```

当 docker 服务重启时，容器间的映射也会随之改变，仍保持连接。

**Docker 容器与外部网络的连接**

Docker 依赖 Linux 的iptables 进行访问控制。

***

### 数据管理

**Docker 容器的数据卷**

Docker 数据卷是经过特殊设计的目录，可以绕过联合文件系统(UFS)，为一个或多个容器提供访问。

数据卷涉及的目的，在于数据的永久化，它完全独立于容器的生存周期。因此，Docker 不会在容器删除时，删除其挂载的数据卷，也不存在类似的垃圾收集机制，对容器引用的数据卷进行处理。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Linux/docker%201.png)

特点：

* 数据卷在容器启动时初始化，如果容器使用的image在挂载点包含了数据，这些数据会拷贝到新初始化的数据卷中；
* 数据卷可以荣期间共享和重用；
* 可以对数据卷的内容直接进行修改；
* 数据卷的变化不会影响image的更新；
* 卷会一直存在，及时挂载数据卷的容器已经被删除

```text
docker run -v hostDataPath:containerDataPath
如果host不存该文件夹，则自动创建

docker run -v hostDataPath:containerDataPath:ro
添加只读权限
```

但是，在 Dockerfile 中创建的数据卷是不能够映射到host目录。在image构建时，指定的数据卷，在容器时，会创建指定名字的数据卷，并且由该image创建的不同容器，所创建的数据卷也是不一样的这些容器的数据卷是不共享的，因为生成容器的同时，数据卷也初始化了。这一点应该和通过命令生成的数据卷进行区分。这点可以通过 `docker inspect`命令查看，不同容器数据卷与host的映射也是不同的。

**Docker 的数据卷容器**

命名的容器挂载数据卷，其他容器通过挂载这个容器实现数据共享，挂载数据卷的容器，叫做数据卷容器。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Linux/docker%202.png)

```text
docker run --volumes-from [CONTAINER NAME]
挂载到指定容器的数据卷
```

挂载容器中的路径到容器中

```text
docker run -d --name nginx -v /usr/share/nginx/html nginx
将指定路径挂载到nginx
其中，/usr/share/nginx/html是容器的路径

```

将本地的目录挂载到容器中

```text
docker run -d -v $PWD/html:/usr/share/nginx/html nginx
将本地$PWD/html路径挂载到容器的/usr/share/nginx/html路径
$PWD/html是本地当前目录下的html文件路径
/usr/share/nginx/html容器的文件路径
```

将数据卷容器挂载到目标容器

```text
docker create -v $PWD/date:/usr/mydata --name data_container ubuntu
创建一个仅有数据存储容器

docker run -it --volumes-from data_container ubuntu /bin/bash
将上面创建的容器路径挂载
```

**Docker 数据卷的备份和还原**

通过一个新容器对数据卷进行备份和还原。

```text
docker run --volumes-from [CONTAINER NAME] -v $PWD:/backup --name [CONTAINER NAME] ubuntu tar cvf /backup/backup.tar [CONTAINER DATA VOLUME]

docker run --volumes-from [CONTAINER NAME] -v $PWD:/backup --name [CONTAINER NAME] ubuntu tar xvf /backup/backup.tar [CONTAINER DATA VOLUME]
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Linux/docker%203.png)

***

### 跨主机连接

* 使用网桥实现跨主机容器连接

* 使用Open vSwith实现跨主机容器连接

* 使用weave实现跨主机容器连接

***

## Docker Remote API

### Docker 守护进程的配置

Docker 启动选项

```bash
docker -d [OPTIONS]

运行相关：
-D,--debug=false
-e,--exec-driver="native"
-g,--gragh="/var/lib/docker"
--icc=true
-l,--log-level="info"
--label=[]
-p,--pidfile='/var/run/docker.pid'

Docker 服务器连接相关：
-G,--group="docker"
-H,--host=[]
--tls=false
--tlscacert="/home/sven/.docker/ca.pem"
--tlscert="/home/sven/.docker/cert.pem"
--tlskey="/home/sven/.docker/key.pem"
--tlsverify=false

RemoteAPI 相关：
--api-enbale-cors=false

存储相关：
-s,--storage-driver=""
--selinux-enabled=false
--storage-opt=[]

Registry相关：
--insecure-registy=[]
--registry-mirror=[]

网络设置相关：
-b,--bridge=""
--big=""
--fixed-cidr=""
--fixed-cidr-v6=""
--dns=[]
--dns-search=[]
--ip=0.0.0.0
--ip-forward=true
--ip-masq=true
--iptables=true
--ipv6=false
--mtu=0
```

启动配置文件：

`/etc/default/docker`

***

## Registry

镜像仓库

```text
docker search [OPTIONS] TERM
--automated=false
--no-trunc=false
-s,stars=0

docker pull [OPTIONS] NAME[:TAG]
-a,--all-tags=false

docker push [OPTIONS] NAME[:TAG]
```

***

## docker compose

Windows 在安装 docker 的时候，会一起安装 docker compose。Linux 系统下的安装参考[官方文档](https://docs.docker.com/compose/install/#install-compose)。 

通过`docker-compose version`，可以查看版本信息。

***

## WordPress 示例

阮一峰的文章[Docker 微服务教程](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)已经详细介绍了3种安装 WordPress 容器的方法：

* 自建 WordPress 容器
  * MySQL 容器
  * PHP 容器定制：增加 mysql扩展
  * WordPress 容器：Wordpress 容器连接 MySQL
* Wordpress 官方镜像
  * MySQL 容器
  * WordPress 容器定制：静态IP，数据卷持久化
* Docker Compose 工具
  * 新建docker-compose.yml 文件
  * docker-compose up

***

参考：

[Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)，阮一峰

[Docker 命令大全](https://www.runoob.com/docker/docker-command-manual.html)，菜鸟