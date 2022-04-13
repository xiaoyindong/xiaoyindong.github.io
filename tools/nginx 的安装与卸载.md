## 1. 安装依赖工具

1. 安装编译工具及库文件

```s
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

2. 首先要安装 PCRE

```PCRE``` 作用是让 ```Nginx``` 支持 ```Rewrite``` 功能。

下载 ```PCRE``` 安装包，下载地址： ```http://downloads.sourceforge.net/project/pcre/pcre/8.42/pcre-8.42.tar.gz```

```s
[root@bogon src]# cd /usr/local/src/
[root@bogon src]# wget http://downloads.sourceforge.net/project/pcre/pcre/8.42/pcre-8.42.tar.gz
```
解压安装包:

```s
[root@bogon src]# tar zxvf pcre-8.42.tar.gz
```

进入安装包目录

```s
[root@bogon src]# cd pcre-8.42
```

编译安装 

```s
[root@bogon pcre-8.42]# ./configure
[root@bogon pcre-8.42]# make && make install
```

查看```pcre```版本

```s
[root@bogon pcre-8.42]# pcre-config --version
```

## 2. 安装 Nginx

下载 ```Nginx```，下载地址：```http://nginx.org/download/nginx-1.16.1.tar.gz```

```s
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.16.1.tar.gz
tar zxvf nginx-1.16.1.tar.gz
```

进入安装包目录

```s
cd nginx-1.16.1
```
编译安装

```s
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.42 --with-stream --with-stream_ssl_module --with-http_ssl_module --with-http_v2_module --with-threads

make

make install
```

查看```nginx```版本

```
/usr/local/nginx/sbin/nginx -v
```

配置软连接

```s
ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/nginx
```

## 3. 配置语法

在```nginx```的执行文件中，已经指定了他包含了哪些模块，但每一个模块都会提供独一无二的配置语法。这些所有的配置语法，会遵循同样的语法规则。

```nginx```的配置文件是一个```ascii```文本文件，主要有两部分组成，一个叫做指令一个叫做指令快。

```s
http {
    include mime.types;
    upstream thwp {
        server 127.0.0.1:8000;
    }

    server {
        listen 443 http2;
        # nginx配置语法
        limit_req_zone $binary_remote_addr zone=one:10 rate=1r/s;
        location ~* \.(gif|jpg|jpeg)$ {
            proxy_cache my_cache;
            expires 3m;
        }
    }
}
```

像上面```http```就是一个指令快，```include mime.types```是一条指令。

每条指令都是以分号结尾的，指令和参数间以空格符号分隔，拿```include mime.types;```来看，```include```是一个指令名，他的中间可以用一个或者多个空格来分隔，那么后面的```mime.types```就是他的参数，也可以具备多个参数，比如```limit_req_zone $binary_remote_addr zone=one:10 rate=1r/s;```他有三个参数。

两条指令间是以分号作为分隔符的，两条指令放在一行中写也是没有问题的。只不过这样可读性会变得很差。

第三个指令块是以 ```{} ```组成的，他会将多条指令组织到一起，比如```upstream```，他把一条指令```server```放在了```thwp```这个指令块下面。

像```server```他也放置了```listen```，```limit_req_zone```这些指令，他还可以包含其他的指令块，比如说```location```。

有些指令块可以有名字，比如说像```upstream```，后面有个```thwp```，有些指令块是没有名字的，比如说```server```和``http``。


演示一下配置，首先创建 ```Nginx``` 运行使用的用户 ```www```：

```s
/usr/sbin/groupadd www 
/usr/sbin/useradd -g www www
```

配置```nginx.conf``` ，将```/usr/local/nginx/conf/nginx.conf```替换为以下内容

```s

cat /usr/local/nginx/conf/nginx.conf
```

```s
user www www;
worker_processes 2; #设置值和CPU核心数一致
error_log /usr/local/nginx/logs/nginx_error.log crit; #日志位置和日志级别
pid /usr/local/nginx/nginx.pid;
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
    root /usr/local/nginx/html;#站点目录
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

检查配置文件```nginx.conf```的正确性命令

```s
/usr/local/nginx/sbin/nginx -t
```

## 4. 启动 Nginx

```Nginx``` 启动命令如下：

```s
/usr/local/nginx/sbin/nginx
```

## Nginx 其他命令

以下包含了 ```Nginx``` 常用的几个命令：

```s
/usr/local/nginx/sbin/nginx -s reload            # 重新载入配置文件
/usr/local/nginx/sbin/nginx -s reopen            # 重启 Nginx
/usr/local/nginx/sbin/nginx -s stop              # 停止 Nginx
```

## 5. 卸载Nginx

首先输入命令 ```ps -ef | grep nginx```检查一下```nginx```服务是否在运行。

```s
ps -ef |grep nginx

root       3163   2643  0 14:08 tty1     00:00:00 man nginx
root       5427      1  0 14:50 ?        00:00:00 nginx: master process nginx
nginx      5428   5427  0 14:50 ?        00:00:00 nginx: worker process
root       5532   2746  0 14:52 pts/0    00:00:00 grep --color=auto nginx
```

停止```Nginx```服务

```s
/usr/sbin/nginx -s stop

netstat -lntp

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd           
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1261/sshd           
tcp6       0      0 :::111                  :::*                    LISTEN      1/systemd           
tcp6       0      0 :::22                   :::*                    LISTEN      1261/sshd
```

查找、删除```Nginx```相关文件

```s
whereis nginx

nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz /usr/share/man/man3/nginx.3pm.gz
```

```find```查找相关文件

```s
find / -name nginx

/usr/lib64/perl5/vendor_perl/auto/nginx
/usr/lib64/nginx
/usr/share/nginx
/usr/sbin/nginx
/etc/logrotate.d/nginx
/etc/nginx
/var/lib/nginx
/var/log/nginx
```

依次删除```find```查找到的所有目录：

```s
rm -rf /usr/sbin/nginx
```

再使用```yum```清理

```s

yum remove nginx

依赖关系解决

======================================================================================================
 Package                              架构            版本                       源              大小
======================================================================================================
正在删除:
 nginx                                x86_64          1:1.12.2-3.el7             @epel          1.5 M
为依赖而移除:
 nginx-all-modules                    noarch          1:1.12.2-3.el7             @epel          0.0  
 nginx-mod-http-geoip                 x86_64          1:1.12.2-3.el7             @epel           21 k
 nginx-mod-http-image-filter          x86_64          1:1.12.2-3.el7             @epel           24 k
 nginx-mod-http-perl                  x86_64          1:1.12.2-3.el7             @epel           54 k
 nginx-mod-http-xslt-filter           x86_64          1:1.12.2-3.el7             @epel           24 k
 nginx-mod-mail                       x86_64          1:1.12.2-3.el7             @epel           99 k
 nginx-mod-stream                     x86_64          1:1.12.2-3.el7             @epel          157 k

事务概要
======================================================================================================
移除  1 软件包 (+7 依赖软件包)

安装大小：1.9 M
是否继续？[y/N]：
```

卸载完成

### 参考来源

- [1] [Nginx安装配置](https://www.runoob.com/linux/nginx-install-setup.html) "菜鸟教程"