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

***

win客户端 v2rayN 添加代理，参数与服务器端配置文件相同，如下设置：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Linux/V2ray.png)

v2rayN 与 浏览器配置：