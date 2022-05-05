# Nginx 基本

## CUP 安装配置

```bash
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

```bash
cd /usr/local/src/
wget http://downloads.sourceforge.net/project/pcre/pcre/8.45/pcre-8.45.tar.gz

tar zxvf pcre-8.45.tar.gz

cd pcre-8.45

./configure

make && make install

pcre-config --version

cd /usr/local/src/

wget http://nginx.org/download/nginx-1.20.2.tar.gz

tar zxvf nginx-1.20.2.tar.gz

cd nginx-1.20.2

./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.45

make

make install

/usr/local/webserver/nginx/sbin/nginx -v

/usr/sbin/groupadd www

/usr/sbin/useradd -g www www

vim /usr/local/webserver/nginx/conf/nginx.conf

```

```shell
user www www;
worker_processes 2; #设置值和CPU核心数一致
error_log /usr/local/webserver/nginx/logs/nginx_error.log crit; #日志位置和日志级别
pid /usr/local/webserver/nginx/nginx.pid;
#Specifies the value for maximum file descriptors that can be opened by this process.
worker_rlimit_nofile 65535;
events
{
  use epoll;
  worker_connections 65535;
}
http
{
  include mime.types;
  default_type application/octet-stream;
  log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
               '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" $http_x_forwarded_for';

#charset gb2312;

  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 8m;

  sendfile on;
  tcp_nopush on;
  keepalive_timeout 60;
  tcp_nodelay on;
  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 4 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;
  gzip on;
  gzip_min_length 1k;
  gzip_buffers 4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_types text/plain application/x-javascript text/css application/xml;
  gzip_vary on;

  #limit_zone crawler $binary_remote_addr 10m;
 #下面是server虚拟主机的配置
 server
  {
    listen 80;#监听端口
    server_name localhost;#域名
    index index.html index.htm index.php;
    root /usr/local/webserver/nginx/html;#站点目录
      location ~ .*\.(php|php5)?$
    {
      #fastcgi_pass unix:/tmp/php-cgi.sock;
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_index index.php;
      include fastcgi.conf;
    }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|ico)$
    {
      expires 30d;
  # access_log off;
    }
    location ~ .*\.(js|css)?$
    {
      expires 15d;
   # access_log off;
    }
    access_log off;
  }
}

```

```bash
/usr/local/webserver/nginx/sbin/nginx -t
/usr/local/webserver/nginx/sbin/nginx                      # 启动
/usr/local/webserver/nginx/sbin/nginx -s reload            # 重新载入配置文件
/usr/local/webserver/nginx/sbin/nginx -s reopen            # 重启 Nginx
/usr/local/webserver/nginx/sbin/nginx -s stop              # 停止 Nginx

```

## firewall 配置

```bash
firewall-cmd --add-port=80/tcp --permanent

firewall-cmd --reload

firewall-cmd --zone=public --list-all
```

## OpenResty 的下载和安装

```bash
yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo

yum install openresty

service openresty start|stop|restart|reload

```

nginx目录：

/usr/local/openresty/nginx/conf/nginx.conf

## LNMP 环境的搭建

### MySQL 安装

可以使用 mariadb 替代

```bash
yum install mariadb mariadb-server
```

修改默认编码

```bash
character set server=utf8

init connect='SET NAMES utf8'

systemctl start mariadb.service

show variables like '%character_set%';
```

### PHP 安装

```bash
yum install php-fpm php-mysql
```

启动 php-fpm

```bash
system start php-fpm.service
```



# Nginx命令行

格式:

```bash
nginx -s reload
```

帮助: -? -h

使用指定的配置文件: -c

指定配置指令: -g

指定运行目录: -P

测试配置文件是否有语法错误: -t -T

打印nginx的版本信息、编译信息等: -v -V

发送信号: -s

- 立刻停止服务: stop
- 优雅的停止服务: quit
- 重载配置文件: reload
- 重新开始记录日志文件: reopen

# Nginx热升级(重载)

