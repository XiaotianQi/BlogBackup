安装：

```bash
curl https://install.direct/go.sh | bash 
```

提示`V2Ray v4.20.0 is installed.`，则安装成功。

关闭防火墙：

```bash
sudo ufw disable
```

启动v2ray服务:

```bash
service v2ray start
```

提示绿色的`Active: active (running)`，则v2ray服务已经成功启动。

编辑配置文件：

```bash
vi /etc/v2ray/config.json
```

重启服务：

```bash
service v2ray restart
```

安装至此结束。

更新：

```cmd
# 执行以下代码更新，不更换 UUID
bash <(curl -L -s https://install.direct/go.sh)
```

***

win客户端 v2rayN 添加代理，参数与服务器端配置文件相同，如下设置：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/V2ray.png)

v2rayN 与 浏览器配置：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/V2ray%201.png)

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/V2ray%202.png)

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Tools/V2ray%203.png)

***

使用 mkcp 协议：

启用 mkcp，只需要在服务器端`inbound`里面加入`streamSettings`即可，`network`配置成`mkcp`。

```json
"inbounds": [{
    "port": 443,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "*****",
          "level": 1,
          "alterId": 64
        }
      ]
    },
    "streamSettings": {
      "network": "mkcp"
    }
  }],
```

在客户端，将传输协议由`tcp`改成`kcp`。

***

参考：

[V2Ray](https://toutyrater.github.io/advanced/mkcp.html)，ToutyRater