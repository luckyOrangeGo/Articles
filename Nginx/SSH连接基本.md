# SSH连接基本

```
cd /etc/ssh/ 
```

// 由默认22更改端口为2222，在18行添加Port 2222 检查当前监听端口(22)

```
netstat -ntpl | grep 22 systemctl restart sshd.service 
```

配置是否允许root用户远程登录，默认允许

#PermitRootLogin yes 

公钥文件放置位置默认：

​	.ssh/authorized_keys 

SSH默认不用更改 

ssh root@10.211.55.3 

SSH命令

```
systemctl status start|stop|restart|enable|disable sshd.service
```

客户端命令

```bash
ssh [ -p 端口] 用户@{远程ip}
```

SecureCRT

Xshell

putty 

密钥生成

```
ssh-keygen -t rsa -C 4096
```

 system start php-fpm.servicebash
