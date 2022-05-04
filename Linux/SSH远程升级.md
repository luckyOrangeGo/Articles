# SSH远程升级

## 安装并使用telnet服务

安装telnet-server 以及 xinetd。

```bash
yum install xinetd telnet-server -y
```

telnet登录终端类型配置。

```bash
cat >> /etc/securetty <<EOF
pts/0
pts/1
pts/2
pts/3
pts/5
EOF
```

启动并设置开机自启动。

```bash
systemctl enable xinetd 
systemctl enable telnet.socket
systemctl start telnet.socket
systemctl start xinetd
```

防火墙打开23端口。

```
firewall-cmd --zone=public --add-port=23/tcp --permanent
```

外网(windows)连接telnet

打开：控制面板=>程序=>启用和关闭windows服务=>Telnet服务

在终端输入测试连接，并输入用户名密码

```
telnet 202.203.201.20 
```

## 升级openssl版本

```bash
yum install  -y gcc gcc-c++ glibc make autoconf pcre-devel  pam-devel automake makedepend perl-Test-Simple perl zlib zlib-devel
```

备份openssl：

```bash
find / -name openssl

mv /usr/bin/openssl /usr/bin/openssl.bak

mv /etc/pki/ca-trust/extracted/openssl /etc/pki/ca-trust/extracted/openssl.bak

mv /usr/lib64/openssl /usr/lib64/openssl.bak
```



> [root@dm8 soft]# find / -name openssl
> /etc/pki/ca-trust/extracted/openssl
> /usr/bin/openssl
> /usr/lib64/openssl
>
> [root@dm8 soft]# mv /usr/bin/openssl /usr/bin/openssl.bak
>
> [root@dm8 soft]# mv /etc/pki/ca-trust/extracted/openssl /etc/pki/ca-trust/extracted/openssl.bak
>
> [root@dm8 soft]# mv /usr/lib64/openssl /usr/lib64/openssl.bak

解压，编译安装：

```bash
wget https://www.openssl.org/source/openssl-1.1.1o.tar.gz && tar xf openssl-1.1.1o.tar.gz && cd openssl-1.1.1o/ && ./config --prefix=/usr/local/ssl -d shared && make -j 4 && make install

echo '/usr/local/ssl/lib' >> /etc/ld.so.conf
ldconfig -v

```

## 安装openssh

```bash
cd ..

wget http://ftp.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-9.0p1.tar.gz

tar xf openssh-9.0p1.tar.gz

mv /etc/ssh /etc/ssh.bak

cd openssh-9.0p1/

./configure --prefix=/usr/local/openssh --sysconfdir=/etc/ssh --with-ssl-dir=/usr/local/ssl --with-zlib=/usr/local/zlib && make -j 4 && make install

```

sshd_config文件修改

```bash
echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config && echo 'PubkeyAuthentication yes' >> /etc/ssh/sshd_config && echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config
```

备份

```bash
mv /usr/sbin/sshd /usr/sbin/sshd.bak && cp -rf /usr/local/openssh/sbin/sshd /usr/sbin/sshd && mv /usr/bin/ssh /usr/bin/ssh.bak && cp -rf /usr/local/openssh/bin/ssh /usr/bin/ssh && mv /usr/bin/ssh-keygen /usr/bin/ssh-keygen.bak && cp -rf /usr/local/openssh/bin/ssh-keygen /usr/bin/ssh-keygen
```

停止并更新服务

```bash
systemctl stop ssh.service

rm -rf /lib/systemd/system/sshd.service

systemctl daemon-reload

pwd

cp ./openssh-9.0p1/contrib/redhat/sshd.init /etc/init.d/sshd

/etc/init.d/sshd restart

systemctl status sshd

systemctl enable sshd

chkconfig --add sshd

chkconfig --list sshd

ssh -V
```

若正常升级，则停止telnet服务并移除。

```bash
systemctl stop telnet.socket
systemctl stop xinetd
systemctl disable xinetd 
systemctl disable telnet.socket
```

## 使用SSH连接，关闭防火墙23端口

```bash
firewall-cmd --list-all

firewall-cmd --zone=public --remove-port=23/tcp --permanent

firewall-cmd --reload

firewall-cmd --list-ports
```

