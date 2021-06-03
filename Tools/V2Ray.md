## 基本篇

### 服务器

**安装**

```bash
$ curl -O https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh
$ bash install-release.sh
```

提示`V2Ray v4.20.0 is installed.`，则安装成功。

**关闭防火墙**

```bash
$ sudo ufw status
$ sudo ufw disable
```

**关闭服务并配置文件**

```bash
$ systemctl stop v2ray
$ vi /etc/v2ray/config.json
```

```json
{
  "inbounds": [
    {
      "port": 16823, // 服务器监听端口
      "protocol": "vmess",    // 主传入协议
      "settings": {
        "clients": [
          {
            "id": "uiid",  // 用户 ID，客户端与服务器必须相同
            "alterId": 64
          }
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",  // 主传出协议
      "settings": {}
    }
  ]
}
```

**启动 v2ray 服务**

```bash
$ systemctl start v2ray
$ systemctl status v2ray
$ systemctl enable v2ray	# 开机启动
```

提示绿色的`Active: active (running)`，则v2ray服务已经成功启动。

**升级更新**

在 VPS，重新执行一遍安装脚本就可以更新了，在更新过程中会自动重启 V2Ray，配置文件保持不变。

```bash
$ bash install-release.sh
```

V2Ray 的更新策略是快速迭代，每周五更新(无意外的情况下)。

***

### 客户端

- `v2ray.exe` 运行 V2Ray 的程序文件
- `wv2ray.exe` 同 v2ray.exe，区别在于 wv2ray.exe 是后台运行的，不像 v2ray.exe 会有类似于 cmd 控制台的窗口。运行 V2Ray 时从 v2ray.exe 和 wv2ray.exe 中任选一个即可
- `config.json` V2Ray 的配置文件，后面我们对 V2Ray 进行配置其实就是修改这个文件
- `v2ctl.exe` V2Ray 的工具，有多种功能，除特殊用途外，一般由 v2ray.exe 来调用，用户不用太关心
- `geosite.dat` 用于路由的域名文件
- `geoip.dat` 用于路由的 IP 文件
- `其它` 除上面的提到文件外，其他的不是运行 V2Ray 的必要文件。更详细的说明可以看 doc 文件夹下的 readme.md 文件，可以通过记事本或其它的文本编辑器打开查看

**客户端配置**

```json
{
  "inbounds": [
    {
      "port": 10808, // 监听端口
      "protocol": "socks", // 入口协议为 SOCKS 5
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "settings": {
        "auth": "noauth"  //socks的认证设置，noauth 代表不认证，由于 socks 通常在客户端使用，所以这里不认证
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess", // 出口协议
      "settings": {
        "vnext": [
          {
            "address": "服务器 IP 或域名",
            "port": 16823,  // 服务器端口
            "users": [
              {
                "id": "uiid",  // 用户 ID，必须与服务器端配置相同
                "alterId": 64 // 此处的值也应当与服务器相同
              }
            ]
          }
        ]
      }
    }
  ]
}
```

**重启 v2ray 客户端**

重启客户端，从而读取 `config.json` 中的配置信息

**v2rayN 与 浏览器配置**

V2Ray 将所有选择权交给用户，它不会自动设置系统代理，因此还需要在浏览器里设置代理。以火狐（Firefox）为例，点菜单 -> 选项 -> 高级 -> 设置 -> 手动代理设置，在 SOCKS Host 填上 127.0.0.1，后面的 Port 10808，再勾上使用 SOCKS v5 时代理 DNS。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/V2ray%203.png)

**SwitchyOmega 插件配置**

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/V2ray%202.png)

***

## 基本原理

V2Ray 使用 inbound(传入) 和 outbound(传出) 的结构，这样的结构非常清晰地体现了数据包的流动方向。以这样的角度理解的话：

* V2Ray 做客户端，则 inbound 接收来自浏览器数据，由 outbound 发出去(通常是发到 V2Ray 服务器)；
* V2Ray 做服务器，则 inbound 接收来自 V2Ray 客户端的数据，由 outbound 发出去(想要访问的目标网站)。

无论是客户端还是服务器，配置文件都由两部分内容组成： inbounds 和 outbounds。V2Ray 没有使用常规代理软件的 C/S（即客户端/服务器）结构，它既可以当做服务器也可以作为客户端。

V2Ray 提供的配置检查功能（test 选项），因为可以检查 JSON 语法错误外的问题，比如说突然间手抖把 vmess 写成了 vmss，一下子就检查出来了。

```bash
$ v2ray -test -config /etc/v2ray/config.json
```

在配置当中，有一个 id ，作用类似于 Shadowsocks 的密码, VMess 的 id 的格式必须与 UUID 格式相同。相对应的 VMess 传入传出的 id 必须相同。

**对于客户端：**

假如访问了 google.com，浏览器就会发出一个数据包打包成 socks 协议发送到本机（127.0.0.1 指的本机，localhost）的 10808 端口，这个时候数据包就会被 V2Ray 接收到。

再看 outbounds，protocol 是 vmess，说明 V2Ray 接收到数据包之后要将数据包打包成 [VMess (opens new window)](https://www.v2fly.org/developer/protocols/vmess.html)协议并且使用预设的 id 加密，然后发往服务器地址为 xxx.com 的 xxx 端口。服务器地址 address 可以是域名也可以是 IP。

**对于服务器：**

V2Ray 服务器接收到客户端发来的数据包时就会尝试用 id 解密。如果解密成功再看一下时间对不对，对的话就把数据包发到 outbound 去，outbound.protocol 是 freedom，数据包就直接发到 google.com 了。

实际上数据包的流向就是：

```text
{浏览器} <--(socks)--> {V2Ray 客户端 inbound <-> V2Ray 客户端 outbound} <--(VMess)-->  {V2Ray 服务器 inbound <-> V2Ray 服务器 outbound} <--(Freedom)--> {目标网站}
```

alterId 参数，这个参数主要是为了加强防探测能力。理论上 alterId 越大越好，但越大就约占内存(只针对服务器，客户端不占内存)，所以折中之下设一个中间值才是最好的。那么设多大才是最好的？其实这个是分场景的，不过根据经验，alterId 的值设为 30 到 100 之间应该是比较合适的。alterId 的大小要保证客户端的小于等于服务器的。

客户端配置的 inbounds 中，有一个 `"sniffing"` 字段，V2Ray 手册解释为“流量探测，根据指定的流量类型，重置所请求的目标”，简单说这东西就是从网络流量中识别出域名。这个 sniffing 有两个用处：

* 解决 DNS 污染；
* 对于 IP 流量可以应用后文提到的域名路由规则；
* 识别 BT 协议，根据自己的需要拦截或者直连 BT 流量。

综上，V2Ray 的配置文件格式就像这样：

```text
{
  "log": {},
  "inbounds": [],
  "outbounds": [],
  "routing": {},
  "transport": {},
  "dns": {},
  "reverse": {},
  "policy": {},
  "stats": {},
  "api": {}
}
```

要理解 V2Ray 的工作模式，首先得抛开客户端和服务器的概念，我们更应该以中转节点的概念来理解。 V2Ray 只是一个转发数据的软件，只要它从入口当中接收到数据包，不管 V2Ray 对这些数据包做了什么（加密、解密、协议转换等），到最后肯定是要把这些数据包从出口发出去。每一个运行的 V2Ray 都是一个节点，它从上一个节点接收数据，发送到下一个节点，在这样由多个节点组成的代理链中，首节点和末节点就是我们常说的客户端和服务器。更广义地说，每个节点对于上一个节点来说是服务器，对于下一个节点来说是客户端。

无论是出口还是入口，我们首先要明确的是协议，只有协议对了才能正常通信。

***

## 进阶

###  WebSocket + TLS + Web

本例中，v2ray 的 `config.json` 文件的安装路径是 `/usr/local/etc/v2ray/`。

####  证书

TLS 是证书认证机制，这里使用免费证书，证书认证机构为 [Let's Encrypt](https://letsencrypt.org/)。证书的生成有许多方法，这里使用的是比较简单的方法：使用 [acme.sh](https://github.com/Neilpang/acme.sh)脚本生成，本部分说明部分内容参考于[acme.sh README](https://github.com/Neilpang/acme.sh/blob/master/README.md)。

证书有两种，一种是 ECC 证书（内置公钥是 ECDSA 公钥），一种是 RSA 证书（内置 RSA 公钥）。简单来说，同等长度 ECC 比 RSA 更安全,也就是说在具有同样安全性的情况下，ECC 的密钥长度比 RSA 短得多（加密解密会更快）。但问题是 ECC 的兼容性会差一些，Android 4.x 以下和 Windows XP 不支持。只要您的设备不是非常老的老古董，建议使用 ECC 证书。

**安装 acme.sh**

`acme.sh` 的依赖项主要是 `socat`，通过以下命令来安装这些依赖项。

```bash
$ sudo apt-get install -y openssl cron socat curl
```

执行以下命令，`acme.sh` 会安装到 `~/.acme.sh` 目录下。

```bash
$ curl  https://get.acme.sh | sh
```

安装成功后执行 `source ~/.bashrc` 以确保脚本所设置的命令别名生效。

**使用 `acme.sh` 生成证书**

生成证书的命令会临时监听 80 端口，请确保执行该命令前 80 端口没有使用。

检查 80 端口是否被占用：

```bash
$ netstat -anlp | grep 80
```

执行以下命令生成证书，注意将 `mydomain.me` 替换成自己的域名：

```bash
$ ~/.acme.sh/acme.sh --issue -d mydomain.me --standalone --keylength ec-256 --force
```

`--keylength` 表示密钥长度，后面的值可以是 `ec-256` 、`ec-384`、`2048`、`3072`、`4096`、`8192`，带有 `ec` 表示生成的是 ECC 证书，没有则是 RSA 证书。在安全性上 256 位的 ECC 证书等同于 3072 位的 RSA 证书。

如果 VPS 上有架设网页，请使用 `webroot` 模式生成证书而不是 TLS 小节中提到的 `standalone` 模式。

```bash
$ ~/.acme.sh/acme.sh --issue -d mydomain.me --webroot /path/to/webroot --keylength ec-256
```

**证书更新**

由于 Let's Encrypt 的证书有效期只有 3 个月，因此需要 90 天至少要更新一次证书，`acme.sh` 脚本会每 60 天自动更新证书。也可以手动更新。

手动更新证书，执行：

```bash
$ ~/.acme.sh/acme.sh --renew -d mydomain.me --force --ecc
```

可以在 `/root/.acme.sh/mydomain.me/` 中看到生成的证书文件。

无论什么情况，密钥(即上面的 `mydomain.me`)都不能泄漏，如果你不幸泄漏了密钥，可以使用 `acme.sh` 将原证书吊销，再生成新的证书，吊销方法请自行参考 [acme.sh 的手册](https://github.com/Neilpang/acme.sh/wiki/Options-and-Params)。

**验证**

打开 [Qualys SSL Labs's SSL Server Test ](https://www.ssllabs.com/ssltest/index.html)，在 Hostname 中输入域名，点提交，过一会结果就出来了。

**吊销删除证书**

```bash
$ acme.sh --revoke -d example.com --ecc
$ acme.sh --remove -d example.com --ecc
```

***

####  Nginx

**安装**

```bash
$ sudo apt-get install -y nginx
```

**验证**

安装成功后打开浏览器，打开你的域名，如果看到默认的 Nginx 登录页面，则安装成功。

**Nginx 配置文件的结构和最佳做法**

- 所有 Nginx 配置文件都位于 `/etc/nginx` 目录中；
- 主要的 Nginx 配置文件为 `/etc/nginx/nginx.conf`；
- 为使 Nginx 配置更易于维护，建议为每个域创建一个单独的配置文件。可以根据需要拥有任意数量的服务器块文件；
- Nginx 服务器块文件存储在 `/etc/nginx/sites-available` 目录中。除非它们链接到 `/etc/nginx/sites-enabled` 目录，否则 Nginx 不会使用此目录中找到的配置文件；
- 要激活服务器块，需要从以下目录中的配置文件站点创建符号链接(指针)将 `sites-available` 目录移到 `sites-enabled` 目录；
- 建议遵循标准命名约定，例如，如果域名是 `your.domain`，则配置文件应命名为 `/etc/nginx/sites-available/your.domain.conf` ；
- `/etc/nginx/snippets` 目录包含可包含在服务器块文件中的配置片段。如果使用可重复的配置段，则可以将这些段重构为片段，并将片段文件包括到服务器块中；
- Nginx 日志文件(`access.log` 和 `error.log`)位于 `/var/log/nginx` 目录中。建议每个服务器块使用不同的 `access` 和 `error` 日志文件；
- 可以将域文档根目录设置为所需的任何位置。 Webroot 的最常见位置包括：
  `/home//`
  `/var/www/`
  `/var/www/html/`
  `/opt/`

***

#### 配置

这次 TLS 的配置将写入 Nginx/Caddy/Apache 配置中，由这些软件来监听 443 端口（443 比较常用，并非 443 不可），然后将流量转发到 V2Ray 的 WebSocket 所监听的内网端口（本例是 10000），V2Ray 服务器端不需要配置 TLS。

**v2ray 服务器配置**

将服务器 `/etc/v2ray`(有时也会安装在`/usr/local/etc/v2ray/config.json`) 目录下的 `config.json` 文件修改成下面的内容，修改完成后要重启 V2Ray 才会使修改的配置生效。

```json
{
    "inbounds": [
      {
        "port": 10000,
        "listen":"127.0.0.1",//只监听 127.0.0.1，避免除本机外的机器探测到开放了 10000 端口
        "protocol": "vmess",
        "settings": {
          "clients": [
            {
              "id": "your uiid",
              "alterId": 64
            }
          ]
        },
        "streamSettings": {
          "network": "ws",
          "wsSettings": {
          "path": "/随机码"
          }
        }
      }
    ],
    "outbounds": [
      {
        "protocol": "freedom",
        "settings": {}
      }
    ]
}
```

**证书配置**

安装证书和密钥

```bash
$ acme.sh --install-cert -d mydomain.com --ecc \
        --key-file       /usr/local/etc/v2ray/v2ray.key \
        --fullchain-file /usr/local/etc/v2ray/v2ray.crt \
        --reloadcmd     "service nginx force-reload"
```

由于本例中已经将证书生成到 `/usr/local/etc/v2ray/` 文件夹，更新证书之后还得把新证书生成到 `/usr/local/etc/v2ray/`。

**Nginx 配置**

注意替换配置中的是域名和证书。将以下代码以 `v2ray_ws_tls.conf` 保存在 `/etc/nginx/conf.d` 路径下作为 Nginx 的一个单独模块加载。

```conf
server {
  listen 443 ssl;
  listen [::]:443 ssl;
  
  ssl_certificate       /usr/local/etc/v2ray/v2ray.crt;
  ssl_certificate_key   /usr/local/etc/v2ray/v2ray.key;
  ssl_session_timeout 1d;
  ssl_session_cache shared:MozSSL:10m;
  ssl_session_tickets off;
  
  ssl_protocols         TLSv1.2 TLSv1.3;
  ssl_ciphers           ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
  ssl_prefer_server_ciphers off;
  
  server_name           mydomain.com;
  location /随机码 { # 与 V2Ray 配置中的 path 保持一致
    if ($http_upgrade != "websocket") { # WebSocket协商失败时返回404
        return 404;
    }
    proxy_redirect off;
    proxy_pass http://127.0.0.1:10000; # 假设WebSocket监听在环回地址的10000端口上
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    # Show real IP in v2ray access.log
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

**客户端配置**

```json
{
  "inbounds": [
    {
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "settings": {
        "auth": "noauth",
        "udp": false
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "mydomain.me",
            "port": 443,
            "users": [
              {
                "id": "uiid",
                "alterId": 64
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/随机码"
        }
      }
    }
  ]
}
```

***

#### 启动/重启 服务

```bash
service v2ray restart
service nginx restart
```

***

#### CDN

采用了 CloudFlare 的 CDN。存在以下注意事项：

* 确保 Cloudflare 的 SSL/TLS 加密模式为完全(Full)或者更高，默认是灵活。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/V2ray%20cdn.png)

***

### Mux

Mux 只需在客户端开启，服务器会自动识别，所以只给客户端的配置。也就是只要在 `outbound` 或 `outboundDetour` 加入 `"mux": {"enabled": true}` 即可：

***

参考：

[V2Ray](https://toutyrater.github.io/advanced/mkcp.html)，ToutyRater

[新 V2Ray 白话文指南](https://guide.v2fly.org/)

