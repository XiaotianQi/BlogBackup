## 安装

ubuntu 安装

```bash
sudo apt-get install redis-server
```

Redis 配置

```bash
sudo vim /etc/redis/redis.conf

# 修改其中
bind 127.0.0.1	# 将其注释，开启远程登录
daemonize yes 	# 开启后台运行
requirepass xxx # 设置密码
```

重启服务

```bash
sudo service redis restart

systemctl status redis
```

登录

```python
redis-cli -a password
```

