* 屏幕一直显示5个进度点

```bash
sudo apt-get update
sudo apt-get install --reinstall lightdm
sudo reboot
```

***

* Hyper V 安装 ubuntu，启动后无法进入安装界面

取消 安全启动 即可

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/Linux/linux%20errs%201.png)

***

* 开启 ssh 服务

```bash
安装 openssh-server:
sudo apt-get update
sudo apt-get install openssh-server

查看是否开启了ssh 服务：
sudo ps -e |grep ssh

查看 IP：
ifconfig
```

***

* 防火墙

```bash
# 查看防火墙状态
sudo ufw status 

# 开启防火墙
sudo ufw enable

# 关闭防火墙
sudo ufw disable 

# 重启防火墙
sudo ufw reload
```

***

* 设置 静态ip，ubuntu 18.04

```text
# 在/etc/netplan/目录下，修改以yaml结尾的文件
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.1.110/24]
      gateway4:  192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114]
```

其中，`eth0` 指网卡，`addresses` 是静态IP

最后使用`sudo netplan apply`来重启网络服务就可以了。

