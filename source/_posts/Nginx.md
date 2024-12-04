---
title: Nginx
tags: 安装
index_img: /assert/nginx.jpeg
img: https://picx1.zhimg.com/v2-e68d524210343613129267bd2cb75a0d_1440w.jpg

---

## Nginx安装

### 离线安装

**下载**

```shell
# 下载nginx:
wget http://nginx.org/download/nginx-1.8.1.tar.gz
# 下载openssl:
wget https://www.openssl.org/source/openssl-fips-2.0.16.tar.gz
# 下载zlib:
wget http://www.zlib.net/zlib-1.2.11.tar.gz
# 下载pcre:
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz
# 如果没有安装c++编译环境，还得安装，通过```yum install gcc-c++```完成安装
```

**编译安装**

```shell
# openssl：
[root@localhost] tar zxvf openssl-fips-2.0.16.tar.gz
[root@localhost] cd openssl-fips-2.0.16
[root@localhost] ./config && make && make install

# pcre:
[root@localhost] tar zxvf pcre-8.39.tar.gz
[root@localhost] cd pcre-8.39
[root@localhost]  ./configure && make && make install

# zlib:
[root@localhost]tar zxvf zlib-1.2.11.tar.gz
[root@localhost] cd zlib-1.2.11
[root@localhost]  ./configure && make && make install

# 最后安装nginx
[root@localhost]tar zxvf nginx-1.8.1.tar.gz
[root@localhost] cd nginx-1.8.1
[root@localhost]  ./configure && make && make install
```

**启动nginx**

```shell
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx -s stop # 立即停止nginx，不保存相关信息
/usr/local/nginx/sbin/nginx -s quit  # 正常退出nginx，保存相关信息
/usr/local/nginx/sbin/nginx -s reload # 重启
```
> [Linux安装Nginx详细图解教程](https://www.cnblogs.com/lovexinyi8/p/5845017.html)

**将nginx做成系统服务并且开机自启动**

由于是源码安装，需要手动创建nginx.service服务
> 不止nginx，其他源码安装的想要实现开机自启动就在/lib/systemd/system目录下自定义服务即可
```shell
vim /lib/systemd/system/nginx.service
# 编辑内容
[Unit]
Description=nginx.service
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target

# 参数介绍：
# [Unit]:服务的说明
# Description:描述服务
# After:描述服务类别
# [Service]服务运行参数的设置
# Type=forking是后台运行的形式
# ExecStart为服务的具体运行命令
# ExecReload为重启命令
# ExecStop为停止命令
# PrivateTmp=True表示给服务分配独立的临时空间
# 注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
# [Install]运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3
```
:wq! 保存退出。
```shell
# 设置开机启动
systemctl enable nginx.service
# 其他命令
# 启动nginx服务
systemctl start nginx.service　
# 停止开机自启动
systemctl disable nginx.service
# 查看服务当前状态
systemctl status nginx.service
# 重新启动服务
systemctl restart nginx.service　
# 查看所有已启动的服务
systemctl list-units --type=service
```

### 在线安装

**下载**

[官网地址](https://nginx.org)

**安装依赖**

```shell
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

**安装**

- 创建一个文件夹,上传本地提供的nginx包
```shell
if [ ! -d "/root/software" ]; then
mkdir -p /root/software
fi && cd /root/software && rz
```

- 解压
```shell
tar -zxvf nginx-1.18.0.tar.gz && cd nginx-1.18.0
```

- 配置、编译、安装
```shell
./configure && make && make install
```

- 检查是否安装成功
```shell
whereis nginx
cd /usr/local/nginx/sbin && ./nginx
ps -ef | grep nginx
```

- 启动代码格式：nginx安装目录地址 -c nginx配置文件地址
```shell
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

- 热加载
```shell
/usr/local/nginx/sbin/nginx -s reload
```

### Nginx安装手册
**nginx安装环境**

nginx是C语言开发，建议在linux上运行，本教程使用Centos6.5作为安装环境。

- gcc。安装nginx需要先将官网下载的源码进行编译，编译依赖gcc环境，如果没有gcc环境，需要安装gcc：`yum install gcc-c++`

- PCRE。PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括perl兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。`yum install -y pcre pcre-devel`
> pcre-devel是使用pcre开发的一个二次开发库。nginx也需要此库。

- zlib
zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。`yum install -y zlib zlib-devel`

- openssl
OpenSSL是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库。`yum install -y openssl openssl-devel`

**编译安装**

将nginx-1.8.0.tar.gz拷贝至linux服务器。
```shell
# 解压
tar -zxvf nginx-1.8.0.tar.gz
cd nginx-1.8.0
```
1. configure：`./configure --help`查询详细参数（参考本教程附录部分：nginx编译参数）注意：上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录,参数设置如下：
```config
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```
注意：上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录
2. 编译安装
```shell
make
make install
```

3. 启动nginx
```shell
cd /usr/local/nginx/sbin/
./nginx
```
查询nginx进程：`ps aux|grep nginx`。注意：执行./nginx启动nginx，这里可以-c指定加载的nginx配置文件，如下：
```nginx
./nginx -c /usr/local/nginx/conf/nginx.conf
```
如果不指定-c，nginx在启动时默认加载conf/nginx.conf文件，此文件的地址也可以在编译安装nginx时指定./configure的参数（--conf-path=指向配置文件（nginx.conf））
4. 停止nginx。
```shell
# 方式1，快速停止：
cd /usr/local/nginx/sbin
./nginx -s stop
```
此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。
```shell
# 方式2，完整停止(建议使用)：
cd /usr/local/nginx/sbin
./nginx -s quit
```
此方式停止步骤是待nginx进程处理任务完毕进行停止。
5. 重启nginx。
方式1，先停止再启动（建议使用）：对nginx进行重启相当于先停止nginx再启动nginx，即先执行停止命令再执行启动命令。如下：
```shell
./nginx -s quit
./nginx
```
方式2，重新加载配置文件：当nginx的配置文件nginx.conf修改后，要想让配置生效需要重启nginx，使用`-s reload`不用先停止nginx再启动nginx即可将配置信息在nginx中生效，如下：
```shell
./nginx -s reload
```
6. 测试。nginx安装成功，启动nginx，即可访问虚拟机上的nginx：到这说明nginx上安装成功。

7. 开机自启动nginx

**编写shell脚本**,这里使用的是编写shell脚本的方式来处理`vi /etc/init.d/nginx`(输入下面的代码)
```bash
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.2 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
# It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /var/run/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
nginxd=/usr/local/nginx/sbin/nginx
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/var/run/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
 echo "nginx already running...."
 exit 1
fi
 echo -n $"Starting $prog: "
 daemon $nginxd -c ${nginx_config}
 RETVAL=$?
 echo
 [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
 return $RETVAL
}
# Stop nginx daemons functions.
stop() {
 echo -n $"Stopping $prog: "
 killproc $nginxd
 RETVAL=$?
 echo
 [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /var/run/nginx.pid
}
# reload nginx service functions.
reload() {
 echo -n $"Reloading $prog: "
 #kill -HUP `cat ${nginx_pid}`
 killproc $nginxd -HUP
 RETVAL=$?
 echo
}
# See how we were called.
case "$1" in
start)
 start
 ;;
stop)
 stop
 ;;
reload)
 reload
 ;;
restart)
 stop
 start
 ;;
status)
 status $prog
 RETVAL=$?
 ;;
*)
 echo $"Usage: $prog {start|stop|restart|reload|status|help}"
 exit 1
esac
exit $RETVAL
```
:wq保存并退出
**设置文件的访问权限**

```shell
chmod a+x /etc/init.d/nginx #(a+x ==> all user can execute 所有用户可执行)
```
这样在控制台就很容易的操作nginx了：查看Nginx当前状态、启动Nginx、停止Nginx、重启Nginx…如果修改了nginx的配置文件nginx.conf，也可以使用上面的命令重新加载新的配置文件并运行，可以将此命令加入到rc.local文件中，这样开机的时候nginx就默认启动了
**加入到rc.local文件中**

```shell
vi /etc/rc.local
```
加入一行`/etc/init.d/nginx start`保存并退出，下次重启会生效。

## Nginx语法与变量

### 语法规则

location [= | \~ | \~\* | ^\~ ] /uri/ { … }

- =：表示精确匹配,这个优先级也是最高的
- ^\~：表示uri以某个常规字符串开头，理解为匹配url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）。
- \~：表示区分大小写的正则匹配
- \~\*：表示不区分大小写的正则匹配(和上面的唯一区别就是大小写)
- !\~和!\~\*：分别为区分大小写不匹配及不区分大小写不匹配的正则
- /：通用匹配，任何请求都会匹配到，默认匹配.

**语法的一些规则和优先级**
多个location配置的情况下匹配顺序为：首先匹配=，其次匹配^\~,其次是按文件中顺序的正则匹配，最后是交给/通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。

#### 正则匹配

- ^ 以什么开始
- \$ 以什么结束
- \~ 区分大小写匹配
- \~\* 不区分大小写匹配

```nginx
# ^/api/user$
# 匹配任何以/uri/开头的任务查询并且停止搜索
location ^~ /uri/
```

#### 精准匹配

```nginx
# =:表示精准匹配,只要完全匹配才能生效
location= /uri
```

#### 前缀匹配

不带任务修饰符,表示前缀匹配

#### 通用匹配

任务未匹配到其他location的请求都会匹配到

```nginx
location /
```

#### 优先级

精准匹配>前缀匹配(若有多个匹配荐匹配成功,那么选择匹配长的并记录)>正则匹配

#### 案例

```nginx
server {
    server_name xdclass.net;
    location ~^/api/pub$ {
    ...
    }
}
# ^/api/pub\$这个正则表达式表示字符串必须以/开始，以b\$结束，中间必须是/api/pub
# http://xdclass.net/api/v1 匹配（完全匹配）
# http://xdclass.net/API/PUB 不匹配，大小写敏感
# http://xdclass.net/api/pub?key1=value1 匹配
# http://xdclass.net/api/pub/ 不匹配
# http://xdclass.net/api/public 不匹配，不能匹配正则表达式
```

#### nginx判断

1、正则表达式匹配：

==：等值比较;
\~：与指定正则表达式模式匹配时返回“真”，判断匹配与否时区分字符大小写；
\~\*：与指定正则表达式模式匹配时返回“真”，判断匹配与否时不区分字符大小写；
!\~：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时区分字符大小写；
!\~\*：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时不区分字符大小写；

2、文件及目录匹配判断：

-f, !-f：判断指定的路径是否为存在且为文件；
-d, !-d：判断指定的路径是否为存在且为目录；
-e, !-e：判断指定的路径是否存在，文件或目录均可；
-x, !-x：判断指定路径的文件是否存在且可执行；

#### if语法

- 语法：if (condition) { … }
- 默认值：none
- 使用字段：server, location
- 注意：尽量考虑使用trp_files代替。
- 判断的条件可以有以下值：

1. 一个变量的名称：空字符传”“或者一些“0”开始的字符串为false。
2. 字符串比较：使用=或!=运算符
3. 正则表达式匹配：使用\~(区分大小写)和\~\*(不区分大小写)，取反运算!\~和!\~\*。
4. 文件是否存在：使用-f和!-f操作符
5. 目录是否存在：使用-d和!-d操作符
7. 文件、目录、符号链接是否存在：使用-e和!-e操作符
8. 文件是否可执行：使用-x和!-x操作符

#### return

- 语法：return code
- 默认值：none
- 使用字段：server,location,if
- nginx隐藏版本号
- nginx.conf中修改http zone中的变量值： server_tokens off;
- php-fpm fastcgi.conf中的变量值： fastcgi_param SERVER_SOFTWARE nginx;

#### nginx正向代理

```nginx
server {
    listen 8090;
    location / {
        resolver 218.85.157.99 218.85.152.99;
        resolver_timeout 30s;
        proxy_pass http://$host$request_uri;
    }
    access_log /data/httplogs/proxy-$host-aceess.log;
}
```

resolver指令

- 语法: resolver address ... [valid=time];
- 默认值: —
- 配置段: http, server, location
- 配置DNS服务器IP地址。可以指定多个，以轮询方式请求。
- nginx会缓存解析的结果。默认情况下，缓存时间是名字解析响应中的TTL字段的值，可以通过valid参数更改。

### 变量

#### ngx_http_core_module模块的变量

- **$arg_PARAMETER**：HTTP请求中某个参数的值，如/index.php?site=www.domain.com 可以用$arg_site取得www.domain.com 这个值。
- **$args HTTP**：请求中的完整参数。例如，在请求/index.php?width=400&height=200中，$args表示字符串width=400&height=200.
- **$binary_remote_addr**：二进制格式的客户端地址。例如：\x0A\xE0B\x0E
- **$body_bytes_sent**：表示在向客户端发送的http响应中，包体部分的字节数
- **$content_length**：表示客户端请求头部中的Content-Length字段
- **$content_type**：表示客户端请求头部中的Content-Type字段
- **$cookie_COOKIE**：表示在客户端请求头部中的cookie字段
- **$document_root**：表示当前请求所使用的root配置项的值
- **$uri**：表示当前请求的URI，不带任何参数
- **$document_uri与$uri含义相同**
- **$request_uri**：表示客户端发来的原始请求URI，带完整的参数。$uri和$document_uri未必是用户的原始请求，在内部重定向后可能是重定向后的URI，而$request_uri永远不会改变，始终是客户端的原始URI
- **$host**：表示客户端请求头部中的Host字段。如果Host字段不存在，则以实际处理的server（虚拟主机）名称代替。如果Host字段中带有端口，如IP:PORT，那么$host是去掉端口的，它的值为IP。$host是全小写的。这些特性与http_HEADER中的http_host不同，http_host只取出Host头部对应的值。
- **$hostname**：表示Nginx所在机器的名称，与gethostbyname调用返回的值相同
- **$http_HEADER**：表示当前HTTP请求中相应头部的值。HEADER名称全小写。例如，示请求中Host头部对应的值用$http_host表
- **$sent_http_HEADER**：表示返回客户端的HTTP响应中相应头部的值。HEADER名称全小写。例如，用$sent_http_content_type表示响应中Content-Type头部对应的值
- **$is_args**：表示请求中的URI是否带参数，如果带参数，$is_args值为?，如果不带参数，则是空字符串
- **$limit_rate**：表示当前连接的限速是多少，0表示无限速
- **$nginx_version**：表示当前Nginx的版本号
- **$query_string**：请求URI中的参数，与$args相同，然而$query_string是只读的不会改变
- **$remote_addr**：表示客户端的地址
- **$remote_port**：表示客户端连接使用的端口
- **$remote_user**：表示使用Auth Basic Module时定义的用户名
- **$request_filename**：表示用户请求中的URI经过root或alias转换后的文件路径
- **$request_body**：表示HTTP请求中的包体，该参数只在proxy_pass或fastcgi_pass中有意义
- **$request_body_file**：表示HTTP请求中的包体存储的临时文件名
- **$request_completion**：当请求已经全部完成时，其值为“ok”。若没有完成，就要返回客户端，则其值为空字符串；或者在断点续传等情况下使用HTTP range访问的并不是文件的最后一块，那么其值也是空字符串。
- **$request_method**：表示HTTP请求的方法名，如GET、PUT、POST等
- **$scheme**：表示HTTP scheme，如在请求https://nginx.com/中表示https
- **$server_addr**：表示服务器地址
- **$server_name**：表示服务器名称
- **$server_port**：表示服务器端口
- **$server_protocol**：表示服务器向客户端发送响应的协议，如HTTP/1.1或HTTP/1.0

#### 日志配置

- **$remote_addr,$http_x_forwarded_for-**：记录客户端IP地址
- **$remote_user**：记录客户端用户名称
- **$request**：记录请求的URL和HTTP协议
- **$status**：记录请求状态
- **$body_bytes_sent**：发送给客户端的字节数，不包括响应头的大小；该变量与Apache模块mod_log_config里的“%B”参数兼容。
- **$bytes_sent**：发送给客户端的总字节数。
- **$connection**：连接的序列号。
- **$connection_requests**：当前通过一个连接获得的请求数量。
- **$msec**：日志写入时间。单位为秒，精度是毫秒。
- **$pipe**：如果请求是通过HTTP流水线(pipelined)发送，pipe值为“p”，否则为“.”。
- **$http_referer**：记录从哪个页面链接访问过来的
- **$http_user_agent**：记录客户端浏览器相关信息
- **$request_length**：请求的长度（包括请求行，请求头和请求正文）。
- **$request_time**：请求处理时间，单位为秒，精度毫秒；从读入客户端的第一个字节开始，直到把最后一个字符发送给客户端后进行日志写入为止。
- **$time_iso8601**：ISO8601标准格式下的本地时间。
- **$time_local**：通用日志格式下的本地时间。

## location "/"的作用

**前置测试访问域名**：www.test.com/api/upload

### location和proxy\_pass都带/，则真实地址不带location匹配目录

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8080/;
}
```

**访问地址**：www.test.com/api/upload-->http://127.0.0.1:8080/upload

### location不带/，proxy\_pass带/，则真实地址会带/

```nginx
location /api {
    proxy_pass http://127.0.0.1:8080/;
}
```

**访问地址**：www.test.com/api/upload-->http://127.0.0.1:8080/upload

### location带/，proxy\_pass不带/，则真实地址会带location匹配目录/api/

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8080;
}
```

**访问地址**：www.test.com/api/upload-->http://127.0.0.1:8080/api/upload

### location和proxy\_pass都不带/，则真实地址会带location匹配目录/api/

```nginx
location /api {
    proxy_pass http://127.0.0.1:8080;
}
```

**访问地址**：www.test.com/api/upload-->http://127.0.0.1:8080/api/upload

### location和proxy\_pass都带/，但proxy\_pass带地址(同1)

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8080/server/;
}
```

**访问地址**：www.test.com/api/upload-->http://127.0.0.1:8080/server/upload

### location不带/，proxy\_pass带/，但proxy\_pass带地址，则真实地址会多个/(同2)

```nginx
location /api {
    proxy_pass http://127.0.0.1:8080/server/;
}
```

**访问地址**：www.test.com/api/upload-->http://127.0.0.1:8080/server//upload

### location带/，proxy\_pass不带/，但proxy\_pass带地址，则真实地址会直接连起来(同3)

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:8080/server;
}
```

**访问地址**：www.test.com/api/upload-->http://127.0.0.1:8080/serverupload

### location和proxy\_pass都不带/，但proxy\_pass带地址，则真实地址匹配地址会替换location匹配目录(同4)

```nginx
location /api {
    proxy_pass http://127.0.0.1:8080/server;
}
```

**访问地址**：www.test.com/api/upload-->http://127.0.0.1:8080/server/upload

### 总结

1. proxy_pass代理地址端口后有目录(包括/)，转发后地址：代理地址+访问URL目录部分去除location匹配目录
2. proxy_pass代理地址端口后无任何，转发后地址：代理地址+访问URL目录部


## Nginx业务需求配置


### Nginx设置黑/白名单IP限制

**第一种方法:allow、deny**

deny和allow指令属于ngx_http_access_module，nginx默认加载此模块，所以可直接使用。这种方式，最简单，最直接。设置类似防火墙iptable，使用方法：直接配置文件中添加：
```nginx
# 白名单设置，allow后面为可访问IP
location / {
     allow 123.13.123.12;
     allow 23.53.32.1/100;
     deny  all;
}

# 黑名单设置，deny后面接限制的IP，为什么不加allow all?因为这个默认是开启的
location / {
     deny 123.13.123.12;
}

# 白名单，特定目录访问限制
location /tree/list {
     allow 123.13.123.12;
     deny all;
}
```

或者通过读取文件IP配置白名单

```nginx
location /{
    include /home/whitelist.conf;
    # 默认位置路径为/etc/nginx/下，
    # 如直接写include whitelist.conf，则只需要在/etc/nginx目录下创建whitelist.conf
    deny all;
}
```

在/home/目录下创建whitelist.conf，并写入需要加入白名单的IP，添加完成后查看如下：

```shell
cat /home/whitelist.conf

# 白名单IP
allow 10.1.1.10;
allow 10.1.1.11;
```

白名单设置完成，黑名单设置方法一样。

**第二种方法:ngx_http_geo_module**

默认情况下，一般nginx是有加该模块的，[ngx_http_geo_module官方文档](https://nginx.org/en/docs/http/ngx_http_geo_module.html)。参数需设置在位置在http模块中。此模块可设置IP限制，也可设置国家地区限制。位置在server模块外即可。语法示例：配置文件直接添加

```nginx
geo $ip_list {
    default 0;
    # 设置默认值为0
    192.168.1.0/24 1;
    10.1.0.0/16    1;
}
server {
   listen       8081;
   server_name  192.168.152.100;

   location / {
       root   /var/www/test;
       index  index.html index.htm index.php;
       if ( $ip_list = 0 ) {
           # 判断默认值，如果值为0，可访问，这时上面添加的IP为黑名单。
           # 白名单，将设置$ip_list = 1，这时上面添加的IP为白名单。
           proxy_pass http://192.168.152.100:8081;
        }
    }

}
```

同样可通过读取文件IP配置

```nginx
geo $ip_list {
    default 0;
    #设置默认值为0
    include ip_white.conf;
}
server {
    listen       8081;
    server_name  192.168.152.100;

    location / {
        root   /var/www/test;
        index  index.html index.htm index.php;
        if ( $ip_list = 0 ) {
            return 403;
            # 限制的IP返回值为403，也可以设置为503，504其他值。
            # 建议设置503，504这样返回的页面不会暴露nginx相关信息，限制的IP看到的信息只显示服务器错误，无法判断真正原因。
        }
    }
}
```
在/etc/nginx目录下创建ip_list.conf，添加IP完成后，查看如下：

```shell
cat /etc/nginx/ip_list.conf

192.168.152.1 1;
192.168.150.0/24 1;
```

设置完成，ip_list.conf的IP为白名单，不在名单中的，直接返回403页面。黑名单设置方法相同。

**ngx_http_geo_module负载均衡（扩展）**

ngx_http_geo_module，模块还可以做负载均衡使用，如web集群在不同地区都有服务器，某个地区IP段，负载均衡至访问某个地区的服务器。方式类似，IP后面加上自定义值，不仅仅数字，如US,CN等字母。示例：如果三台服务器：122.11.11.11，133.11.12.22，144.11.11.33

```nginx
geo $country {
    default default;
    111.11.11.0/24   uk;
    # IP段定义值uk
    111.11.12.0/24   us;
    # IP段定义值us
    }
upstream  uk.server {
    erver 122.11.11.11:9090;
    # 定义值uk的IP直接访问此服务器
}

upstream  us.server {
    server 133.11.12.22:9090;
    # 定义值us的IP直接访问此服务器
}

upstream  default.server {
    server 144.11.11.33:9090;
    #默认的定义值default的IP直接访问此服务器
}

server {
    listen    9090;
    server_name 144.11.11.33;

    location / {
      root  /var/www/html/;
      index index.html index.htm;
     }
 }
```

### 国家城市IP访问限制

有些第三方也提供设置，如cloudflare，设置更简单，防火墙规则里设置。这里讲讲nginx的设置方法。

**安装ngx_http_geoip_module模块**

[ngx_http_geoip_module官方文档](https://nginx.org/en/docs/http/ngx_http_geoip_module.html)，参数需设置在位置在http模块中。nginx默认情况下不构建此模块，应使用`--with-http_geoip_module`配置参数启用它。对于ubuntu系统来说，直接安装nginx-extras组件，包括几乎所有的模块。

```shell
sudo apt install nginx-extras
```

对于centos系统，安装模块。

```shell
yum install nginx-module-geoip
```

**下载IP数据库**

此模块依赖于IP数据库，所有数据在此数据库中读取，所有还需要下载ip库（dat格式）。MaxMind提供了免费的IP地域数据库，坏消息是MaxMind官方已经停止支持dat格式的ip库。在其他地方可以找到dat格式的文件，或者老版本的，当然数据不可能最新，多少有误差。

> 第三方下载地址：https://www.miyuru.lk/geoiplegacy

下载同时包括Ipv4和Ipv6的country、city版本。

```shell
# 下载国家IP库，解压并移动到nginx配置文件目录，
sudo wget https://dl.miyuru.lk/geoip/maxmind/country/maxmind.dat.gz
gunzip maxmind.dat.gz
sudo mv maxmind.dat /etc/nginx/GeoCountry.dat

sudo wget https://dl.miyuru.lk/geoip/maxmind/city/maxmind.dat.gz
gunzip maxmind.dat.gz
sudo mv maxmind.dat /etc/nginx/GeoCity.dat
```

**配置nginx**

示例：

```nginx
geoip_country /etc/nginx/GeoCountry.dat;
geoip_city /etc/nginx/GeoCity.dat;

server {
    listen  80;
    server_name 144.11.11.33;

    location / {
      root  /var/www/html/;
      index index.html index.htm;
      if ($geoip_country_code = CN) {
         return 403;
       # 中国地区，拒绝访问。返回403页面
      }
   }
 }
```
这里，地区国家基础设置就完成了。

Geoip其他参数：

- 国家相关参数：

```nginx
$geoip_country_code # 两位字符的英文国家码。如：CN,US
$geoip_country_code3 # 三位字符的英文国家码。如：CHN,USA
$geoip_country_name # 国家英文全称。如：China,United States
```

- 城市相关参数：

```nginx
$geoip_city_country_code # 也是两位字符的英文国家码。
$geoip_city_country_code3 # 上同
$geoip_city_country_name # 上同.
$geoip_region # 这个经测试是两位数的数字，如杭州是02,上海是23。但是没有搜到相关资料，希望知道的朋友留言告之。
$geoip_city # 城市的英文名称。如：Hangzhou
$geoip_postal_code # 城市的邮政编码。经测试，国内这字段为空
$geoip_city_continent_code # 不知什么用途，国内好像都是AS
$geoip_latitude # 纬度
$geoip_longitude # 经度
```

### 封禁恶意ip

单独网站屏蔽IP的方法，把`include xxx;`放到网址对应的在server{}语句块,虚拟主机所有网站屏蔽IP的方法，把`include xxx;`放到http{}语句块。

**手动封禁**

```nginx
http{
    # ....
    include blacklist.conf;
}

location / {
    proxy_pass http://lbs;
    proxy_redirect default;
}

# blacklist.conf目录文件下的内容
deny 192.168.159.2;
deny 192.168.159.32;

# 重新加载配置，不中断服务
./nginx -s reload
```

**自动化封禁**

思路:

- 编写shell脚本
- AWK统计access.log，记录每秒访问超过60次的ip，然后配合nginx或者iptables进行封禁
- crontab定时跑脚本

```nginx
location / {
    add_header 'Access-Control-Allow-Origin' $http_origin;
    add_header 'Access-Control-Allow-Credentials' 'true';
    add_header 'Access-Control-Allow-Headers' 'DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
    add_header Access-Control-Allow-Methods 'GET,POST,OPTIONS';
    # 如果预检请求则返回成功,不需要转发到后端
    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain; charset=utf-8';
        add_header 'Content-Length' 0;
        return 200;
    }
}
```


### 地址重定向

**rewrite地址重定向，实现URL重定向的重要指令，他根据regex(正则表达式)来匹配内容跳转到**

语法: 
```bash
rewrite regex replacement[flag]
```
这是⼀个正则表达式，匹配完整的域名和后⾯的路径地址,replacement部分https://xdclass.net/$1,$1是取自regex部分()里的内容.rewrite ^/(.\*) https://xdclass.net/$1 permanent

常用正则表达式

- ^: 匹配输入字符串的起始位置
- $: 匹配输入字符串的结束位置
- \*: 匹配前面的字符0次或者多次
- ?: 匹配前面字符串的0次或者1次
- .: 匹配除"\n"之外的所有单个字符
- (pattern) 匹配括号内的pattern

rewrite最后一项flag参数

- last: 本条规则匹配完成后继续向下匹配新的location URI规则
- break: 本条规则匹配完成后终止,不再匹配任何规则
- redirect: 返回302临时重定向
- permanent: 返回301永久重定向


**应用场景**

– 非法访问跳转,防盗链

– 网站更换新域名

– http跳转https

– 不同地址访问同⼀个虚拟主机的资源

### 配置websocket反向代理

```nginx
server {
    listen 80;
    server_name xdclass.net;
    location / {
        proxy_pass http://lbs;
        proxy_read_timeout 300s; //websocket空闲保持时⻓
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection
        $connection_upgrade;
    }
}
```

### 性能优化

**服务端缓存前置**

- /root/cache
本地路径，用来设置Nginx缓存资源的存放地址

- levels=1:2
默认所有缓存文件都放在上面指定的根路径中，可能影响缓存的性能，推荐指定为2级目录来存储缓存文件；1和2表示用1位和2位16进制来命名目录名称。第⼀级目录用1位16进制命名，如a；第⼆级目录用2位16进制命名，如3a。所以此例中⼀级目录有16个，⼆级目录有16 * 16=256个,总目录数为16 * 256=4096个。

- levels=1:1:1时，表示是三级目录，且每级目录数均为16个
- key_zone
在共享内存中定义⼀块存储区域来存放缓存的key和metadata

  - max_size
最大缓存空间,如果不指定会使用掉所有磁盘空间。当达到disk上限后，会删除最少使用的cache
  - inactive
某个缓存在inactive指定的时间内如果不访问，将会从缓存中删除
  - proxy_cache_valid
配置nginx cache中的缓存⽂件的缓存时间,proxy_cache_valid 200 304 2m对于状态为200和304的缓存⽂件的缓存时间是2分钟
  - use_temp_path
建议为off，则nginx会将缓存⽂件直接写⼊指定的cache文件中
  - proxy_cache
启用proxy cache，并指定key_zone，如果proxy_cache off表示关闭掉缓存
  - add_header Nging-Cache "$upstream_cache_status"
用于前端判断是否是缓存，miss、hit、expired(缓存过期)、updating(更新，使用旧的应答)
```nginx
proxy_cache_path /root/cache levels=1:2 keys_zone=xd_cache:10m max_size=1g inactive=60m use_temp_path=off;
server {
    location /{
    ...
    proxy_cache xd_cache;
    proxy_cache_valid 200 304 10m;
    proxy_cache_valid 404 1m;
    proxy_cache_key $host$uri$is_args$args;
    add_header Nginx-Cache "$upstream_cache_status";
    }
}
```

**动静分离**

性能优化-静态资源压缩

对文本、js和css文件等进行压缩，⼀般是压缩后的大小是原始大小的25%
```nginx
# 开启gzip,减少我们发送的数据量
gzip on;
gzip_min_length 1k;

# 4个单位为16k的内存作为压缩结果流缓存
gzip_buffers 4 16k;

# gzip压缩⽐，可在1~9中设置，1压缩⽐最⼩，速度最快，9压缩⽐最⼤，速度最慢，消耗CPU
gzip_comp_level 4;

# 压缩的类型
gzip_types application/javascript text/plain text/css application/json application/xml text/javascript;

# 给代理服务器⽤的，有的浏览器⽀持压缩，有的不⽀持，所以避免浪费不⽀持的也压缩，所以根据客户端的HTTP头来判断，是否需要压缩
gzip_vary on;

# 禁⽤IE6以下的gzip压缩，IE某些版本对gzip的压缩⽀持很不好
gzip_disable "MSIE [1-6].";
```

**压缩是时间换空间，还是空间换时间**？

- web层主要涉及浏览器和服务器的网络交互，而网络交互显然是耗费时间的

- 要尽量减少交互次数

- 降低每次请求或响应数据量

- 开启压缩

- 在服务端是时间换空间的策略，服务端需要牺牲时间进行压缩以减小响应数据大小

- 压缩后的内容可以获得更快的网络传输速度，时间是得到了优化,所以是双向的

### https配置

**删除原先的nginx，新增ssl模块**

```shell
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --withhttp_ssl_module
make
make install
#查看是否成功
/usr/local/nginx/sbin/nginx -V
```

**Nginx配置https证书**

```nginx
server {
    listen 443 ssl;
    server_name 16web.net;
    ssl_certificate /usr/local/software/biz/key/4383407_16web.net.pem;
    ssl_certificate_key /usr/local/software/biz/key/4383407_16web.net.key;
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout 5m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    location / {
        root html;
        index index.html index.htm;
    }
}
```

**https访问实操**

- 杀掉原先进程
- 防⽕墙关闭或者开放443端⼝
```shell
service firewalld stop
```

- 网络安全组开放端口

- 其他

反向代理，获取用户的真实ip
```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

### 高可用

LVS+keepalived+nginx架构解决的问题：

1. 如果其中keepalived挂了，那就会vip就会分发到另外⼀个keepalived节点，响应正常

2. 如果某个realServer挂了，比如是Nginx挂了，那对应keepalived节点存活依旧可以转发过去，但是响应失败

- 脚本监听
```bash
# 配置vrrp_script，主要⽤于健康检查及检查失败后执⾏的动作。
vrrp_script chk_real_server {
    # 健康检查脚本，当脚本返回值不为0时认为失败
    script "/usr/local/software/conf/chk_server.sh"
    # 检查频率，以下配置每2秒检查1次
    interval 2
    # 当检查失败后，将vrrp_instance的priority减⼩5
    weight -5
    # 连续监测失败3次，才认为真的健康检查失败。并调整优先级
    fall 3
    # 连续监测2次成功，就认为成功。但不调整优先级
    rise 2
    user root
}
```

- 编辑脚本

```shell
mkdir -p /usr/local/software/conf
cd /usr/local/software/conf
vim /usr/local/software/conf/chk_server.sh
chmod +x chk_server.sh
```
脚本详情
```shell
#!/bin/bash
#检查nginx进程是否存在
counter=$(ps -C nginx --no-heading|wc -l)
if [ "${counter}" -eq "0" ]; then
service keepalived stop
echo 'nginx server is died.......'
fi
```

3. 步骤
4. ds服务器:
- 安装keepalived,参考keepalived安装步骤
- 安装ipvsadm -y
- 配置防火墙,两种方式,直接关闭，或者按照keepalived中描述的操作
- 参照keepalived描述的修改配置文件

5. rs服务器:
- 安装openresty或者nginx
- 配置防火墙,保证外网能访问
- 配置realserver
```shell
vim /etc/init.d/realserver

#虚拟的vip根据自己的实际情况定义
SNS_VIP=192.168.1.25
/etc/rc.d/init.d/functions
case "$1" in
start)
ifconfig lo:0 $SNS_VIP netmask 255.255.255.255 broadcast $SNS_VIP
/sbin/route add -host $SNS_VIP dev lo:0
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
sysctl -p >/dev/null 2>&1
echo "RealServer Start OK"
;;
stop)
ifconfig lo:0 down
route del $SNS_VIP >/dev/null 2>&1
echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
echo "RealServer Stoped"
;;
*)
echo "Usage: $0 {start|stop}"
exit 1
esac
exit 0
#保存并设置脚本的执行权限
chmod 755 /etc/init.d/realserver
#因为realserver脚本中用到了/etc/rc.d/init.d/functions，所以一并设置权限
chmod 755 /etc/rc.d/init.d/functions
#重启后会失效,所以要设置为开机自启动
chmod +x /etc/rc.d/rc.local
vi /etc/rc.d/rc.local
/etc/rc.d/init.d/realserver.sh start

#执行脚本
service realserver start

#查看执行结果,lo多出vip表明配置正确
ip a
```

- 常见问题
  - vip能ping通，vip监听的端口不通:第⼀个原因:nginx1和nginx2两台服务器的服务没有正常启动
  - vip ping不通:核对是否出现裂脑,常见原因为防火墙配置所致导致多播心跳失败,核对keepalived的配置是否正确：需要关闭selinux，不然sh脚本可能不生效
```shell
#查看
getenforce
#关闭
setenforce 0
```

### 运维统计

1. 查看访问最频繁的前100个IP
```shell
awk '{print $1}' access_temp.log | sort -n |uniq -c | sort -rn | head -n 100
```
2. 统计访问最多的url 前20名
```shell
cat access_temp.log |awk '{print $7}'| sort|uniq -c| sort -rn| head -20 | more
```
3. 自定义日志统计接口性能
- 日志格式增加$request_time
从接收用户请求的第⼀个字节到发送完响应数据的时间，即包括接收请求数据时间、程序响应时间、输出响应数据时间
$upstream_response_time：指从Nginx向后端建立连接开始到接受完数据然后关闭连接为⽌的时间
$request_time⼀般会比upstream_response_time大，因为用户网络较差，或者传递数据较⼤时，前者会耗时大很多
```nginx
log_format main '$remote_addr -$remote_user [$time_local] "$request" ' '$status
$body_bytes_sent "$http_referer" '
'"$http_user_agent"
"$http_x_forwarded_for" $request_time';

server {
    listen 80;
    server_name aabbcc.com;
    location / {
    root /usr/local/nginx/html;
    index xdclass.html;
}
#charset koi8-r;
#
access_log logs/host.access.log main;
}
```

4. 统计耗时接口,列出传输时间超过2秒的接口，显示前5条
```shell
#$NF 表示最后⼀列, awk '{print $NF}
cat time_temp.log|awk '($NF > 2){print $7}'|sort -n|uniq -c|sort -nr|head -5
```

### 防止sql注入

将下面的Nginx配置文件代码放入到server块中，然后重启Nginx即可

```nginx
 if ($request_method !~* GET|POST) { return 444; }
 # 使用444错误代码可以更加减轻服务器负载压力。
 # 防止SQL注入
 if ($query_string ~* (\$|'|--|[+|(%20)]union[+|(%20)]|[+|(%20)]insert[+|(%20)]|[+|(%20)]drop[+|(%20)]|[+|(%20)]truncate[+|(%20)]|[+|(%20)]update[+|(%20)]|[+|(%20)]from[+|(%20)]|[+|(%20)]grant[+|(%20)]|[+|(%20)]exec[+|(%20)]|[+|(%20)]where[+|(%20)]|[+|(%20)]select[+|(%20)]|[+|(%20)]and[+|(%20)]|[+|(%20)]or[+|(%20)]|[+|(%20)]count[+|(%20)]|[+|(%20)]exec[+|(%20)]|[+|(%20)]chr[+|(%20)]|[+|(%20)]mid[+|(%20)]|[+|(%20)]like[+|(%20)]|[+|(%20)]iframe[+|(%20)]|[\<|%3c]script[\>|%3e]|javascript|alert|webscan|dbappsecurity|style|confirm\(|innerhtml|innertext)(.*)$) { return 555; }
 if ($uri ~* (/~).*) { return 501; }
 if ($uri ~* (\\x.)) { return 501; }
 # 防止SQL注入 
 if ($query_string ~* "[;'<>].*") { return 509; }
 if ($request_uri ~ " ") { return 509; }
 if ($request_uri ~ (\/\.+)) { return 509; }
 if ($request_uri ~ (\.+\/)) { return 509; }
 # if ($uri ~* (insert|select|delete|update|count|master|truncate|declare|exec|\*|\')(.*)$ ) { return 503; }
 # 防止SQL注入
 if ($request_uri ~* "(cost\()|(concat\()") { return 504; }
 if ($request_uri ~* "[+|(%20)]union[+|(%20)]") { return 504; }
 if ($request_uri ~* "[+|(%20)]and[+|(%20)]") { return 504; }
 if ($request_uri ~* "[+|(%20)]select[+|(%20)]") { return 504; }
 if ($request_uri ~* "[+|(%20)]or[+|(%20)]") { return 504; }
 if ($request_uri ~* "[+|(%20)]delete[+|(%20)]") { return 504; }
 if ($request_uri ~* "[+|(%20)]update[+|(%20)]") { return 504; }
 if ($request_uri ~* "[+|(%20)]insert[+|(%20)]") { return 504; }
 if ($query_string ~ "(<|%3C).*script.*(>|%3E)") { return 505; }
 if ($query_string ~ "GLOBALS(=|\[|\%[0-9A-Z]{0,2})") { return 505; }
 if ($query_string ~ "_REQUEST(=|\[|\%[0-9A-Z]{0,2})") { return 505; }
 if ($query_string ~ "proc/self/environ") { return 505; }
 if ($query_string ~ "mosConfig_[a-zA-Z_]{1,21}(=|\%3D)") { return 505; }
 if ($query_string ~ "base64_(en|de)code\(.*\)") { return 505; }
 if ($query_string ~ "[a-zA-Z0-9_]=http://") { return 506; }
 if ($query_string ~ "[a-zA-Z0-9_]=(\.\.//?)+") { return 506; }
 if ($query_string ~ "[a-zA-Z0-9_]=/([a-z0-9_.]//?)+") { return 506; }
 if ($query_string ~ "b(ultram|unicauca|valium|viagra|vicodin|xanax|ypxaieo)b") { return 507; }
 if ($query_string ~ "b(erections|hoodia|huronriveracres|impotence|levitra|libido)b") {return 507; }
 if ($query_string ~ "b(ambien|bluespill|cialis|cocaine|ejaculation|erectile)b") { return 507; }
 if ($query_string ~ "b(lipitor|phentermin|pro[sz]ac|sandyauer|tramadol|troyhamby)b") { return 507; }
 # 这里大家根据自己情况添加删减上述判断参数，cURL、wget这类的屏蔽有点儿极端了，但要“宁可错杀一千，不可放过一个”。
 if ($http_user_agent ~* YisouSpider|ApacheBench|WebBench|Jmeter|JoeDog|Havij|GetRight|TurnitinBot|GrabNet|masscan|mail2000|github|wget|curl|Java|python) { return 508; }
 # 同上，大家根据自己站点实际情况来添加删减下面的屏蔽拦截参数。
 if ($http_user_agent ~* "Go-Ahead-Got-It") { return 508; }
 if ($http_user_agent ~* "GetWeb!") { return 508; }
 if ($http_user_agent ~* "Go!Zilla") { return 508; }
 if ($http_user_agent ~* "Download Demon") { return 508; }
 if ($http_user_agent ~* "Indy Library") { return 508; }
 if ($http_user_agent ~* "libwww-perl") { return 508; }
 if ($http_user_agent ~* "Nmap Scripting Engine") { return 508; }
 if ($http_user_agent ~* "~17ce.com") { return 508; }
 if ($http_user_agent ~* "WebBench*") { return 508; }
 if ($http_user_agent ~* "spider") { return 508; } #这个会影响国内某些搜索引擎爬虫，比如：搜狗
 # 拦截各恶意请求的UA，可以通过分析站点日志文件或者waf日志作为参考配置。
 if ($http_referer ~* 17ce.com) { return 509; }
 # 拦截17ce.com站点测速节点的请求，所以明月一直都说这些测速网站的数据仅供参考不能当真的。
 if ($http_referer ~* WebBench*") { return 509; }
 # 拦截WebBench或者类似压力测试工具，其他工具只需要更换名称即可。
```

### 限制连接数量

Nginx可以通过limit_conn_zone和limit_conn两个组件来对客户端访问目录和文件的访问频率和次数进行限制，两个模块都能够对客户端访问进行限制，具体如何使用要结合公司业务环境进行配置。举个简单的例子：

```nginx
http {
  limit_conn_zone $binary_remote_addr zone=ops:10m;
  # ...
  server {
    listen       80;
    server_name  www.javaboy.org;
    location / {
      limit_conn ops 1;  #这将指定一个地址只能同时存在一个连接。“one”与上面的对应，也可以自定义命名
      limit_rate 300k;
  }
}
```

这里涉及到三个配置项：

- limit_zone：是针对每个IP定义一个存储session状态的容器，这个示例中定义了一个10m的容器，假设每个session的大小是32bytes，那么可以处理327680个session。
- limit_conn ops 1：限制每个IP只能发起一个并发连接。
- limit_rate 300k：对每个连接限速300k。注意，这里是对连接限速，而不是对IP限速。如果一个IP允许两个并发连接，那么这个IP就是限速`limit_rate × 2`。

### 限制请求频率

Nginx可以通过限制请求频率来防止服务器过载，最常见的场景就是登录请求，可以通过限制请求频率防止账号暴力破解。Nginx官方版本限制IP的连接和并发分别有两个模块：

- limit_req_zone：用来限制单位时间内的请求数，即速率限制,采用的漏桶算法"leaky bucket"。
- limit_req_conn：用来限制同一时间连接数，即并发限制。

```nginx
http {
    limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;
    server {
        listen 80;
        location / {
            limit_req zone=mylimit burst=5 nodelay;;
            proxy_pass http://javaboy.org;
        }
    }
}
```

**limit_req_zone $binary_remote_addr zone=mylimit:10m rate=1r/s;**

- 第一个参数：$binary_remote_addr表示通过remote_addr这个标识来做限制，“binary_”的目的是缩写内存占用量，是限制同一客户端ip地址。
- 第二个参数：zone=mylimit:10m表示生成一个大小为10M，名字为mylimit的内存区域，用来存储访问的频次信息。
- 第三个参数：rate=1r/s表示允许相同标识的客户端的访问频次，这里限制的是每秒1次，还可以有比如30r/m的。

**limit_req zone=mylimit burst=5 nodelay;**

- 第一个参数：zone=one设置使用哪个配置区域来做限制，与上面limit_req_zone里的name对应。
- 第二个参数：burst=5，重点说明一下这个配置，burst爆发的意思，这个配置的意思是设置一个大小为5的缓冲区，当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内。
- 第三个参数：nodelay，如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回503，如果没有设置，则所有请求会等待排队。

### 防止目录遍历

在Nginx配置中设置`autoindex off`来防止目录遍历攻击。这个一般是如果你要做文件服务器，根据自己的实际需求，有需要的话这个功能可以打开，否则将之关闭即可。

```nginx
location / {
    autoindex off;
}
```

### 隐藏Nginx版本号

攻击者如果能够确定服务器使用的Nginx版本，可能会利用这个信息来寻找和利用已知的漏洞进行攻击。因此，隐藏版本信息可以提高服务器的安全性，使攻击者难以通过版本信息推断出服务器可能存在的安全漏洞。要隐藏Nginx版本号，有三个办法，一般来说我们使用第一种方式就可以了。

**修改配置文件**

在Nginx的配置文件中，在`http`块中添加以下配置：

```nginx
server_tokens off;
```

这样设置后，Nginx将不会在错误页面上显示版本号。配置完成之后，保存配置文件并重新加载Nginx以应用更改：

```nginx
nginx -t # 测试配置文件是否正确
nginx -s reload # 重新加载Nginx配置
```

这种方法可以隐藏错误页面上的版本信息，但可能无法完全隐藏所有响应头中的版本信息。

**修改Nginx源码**

如果想要从根源上修改Nginx版本信息，需要重新编译Nginx，步骤如下：

- 修改`src/core/nginx.h`文件中的版本定义。
- 修改`src/http/ngx_http_header_filter_module.c`文件中的服务器字符串。
- 修改`src/http/ngx_http_special_response.c`文件中的错误页面底部信息。

修改完这些文件后，需要重新编译Nginx。这样编译安装后，Nginx的版本信息将被彻底修改。

**使用第三方模块**

如果需要动态修改响应头中的版本信息，可以使用如`headers-more-nginx-module`模块。这个模块允许你动态地添加、修改或删除Nginx的响应头。通过这个模块，可以完全控制`Server`响应头的内容。选择哪种方法取决于你的具体需求和环境。如果你只是想简单地隐藏版本信息，修改配置文件可能是最简单的方法。如果你需要更彻底地控制版本信息，可能需要考虑修改源码并重新编译Nginx。

### 设置超时时间

设置Nginx的超时配置是非常重要的，因为它可以影响服务器的性能和资源的有效利用。比较常见的超时配置有四个：

1. **keepalive_timeout**：这个指令设置了与客户端的`keep-alive`连接超时时间。如果连接在指定时间内没有数据传输，Nginx将关闭该连接。默认值通常是75秒。这个设置对于频繁访问的站点尤其重要，因为它减少了连接建立和断开的开销。
2. **client_body_timeout**：这个指令指定了客户端与服务端建立连接后发送request body的超时时间。如果客户端在指定时间内没有发送任何内容，Nginx返回HTTP 408（Request Timed Out）。默认值通常是60秒。
3. **client_header_timeout**：这个指令指定了客户端向服务端发送一个完整的request header的超时时间。如果在指定时间内没有发送一个完整的request header，Nginx返回HTTP 408（Request Timed Out）。默认值通常是60秒。
4. **send_timeout**：这个指令设置了服务端向客户端传输数据的超时时间。如果在指定时间内客户端没有接收到任何数据，连接将被关闭。默认值通常是60秒。

**keepalive_timeout**

这个指令控制了客户端与服务器之间的连接保持活动状态的时间。这对于减少TCP连接的开销非常有用，特别是在高流量的网站上。

```nginx
http {
    keepalive_timeout 60s;
    server {
        listen 80;
        server_name javaboy.org;
        location / {
            proxy_pass http://javaboy.org;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
        }
    }
}
```

在这个配置中，`keepalive_timeout`被设置为60秒，意味着如果60秒内没有数据传输，连接将被关闭。

**client_body_timeout**

这个指令设置了客户端发送请求体到服务器的超时时间。

```nginx
http {
    client_body_timeout 10s;
    server {
        listen 80;
        server_name javaboy.org;
        location /upload {
            client_max_body_size 100M;
            client_body_timeout 30s;
        }
    }
}
```

在这个配置中，`client_body_timeout`被设置为10秒，适用于上传大文件的场景，确保如果客户端在30秒内没有完成文件上传，请求将被终止。

**client_header_timeout**

这个指令控制了客户端发送完整的HTTP请求头到服务器的超时时间。

```nginx
http {
    client_header_timeout 5s;
    server {
        listen 80;
        server_name javaboy.org;
        location / {
            proxy_pass http://javaboy.org;
        }
    }
}
```

在这个配置中，`client_header_timeout`被设置为5秒，意味着如果客户端在5秒内没有发送完整的HTTP请求头，服务器将终止连接。

**send_timeout**

这个指令设置了服务器发送响应到客户端的超时时间。

```nginx
http {
    send_timeout 10s;
    server {
        listen 80;
        server_name javaboy.org;
        location / {
            proxy_pass http://javaboy.org;
            proxy_read_timeout 10s;
        }
    }
}
```

在这个配置中，`send_timeout`被设置为10秒，适用于后端服务响应慢的场景，确保如果后端服务在10秒内没有发送数据，客户端将收到超时响应。

### 仅允许域名访问

限制仅允许域名访问可以防止未授权的IP直接访问服务器，减少未备案域名解析到服务器IP导致的安全风险。这个也有三种不同的配置方式。

**使用两个`server`块**

在Nginx配置文件中，你可以设置一个默认的server块，它将捕获所有不明确的域名请求，并返回403错误。然后，为特定的域名设置server块。

```nginx
server {
    listen 80 default_server;
    server_name _;
    return 403;
}
server {
    listen 80;
    server_name www.javaboy.org;
    location / {
        # 你的配置
    }
}
```

在这个配置中，第一个`server`块会拦截所有不明确域名的请求，并返回403错误。第二个`server`块则是为特定域名`www.javaboy.org`提供服务的配置。这个配置有两点需要注意：

1. 如果没有显式声明default server则第一个server会被隐式的设为default server。
2. server_name中的`_;`，并不是重点.`__`也可以，`___`也可以。

**使用`if`语句**

还可以在特定的`server`块中使用`if`语句来检查`$host`变量，如果它不匹配你的域名，则返回403错误。

```nginx
server {
    listen 80;
    server_name javaboy.org;
    location / {
        if ($host != 'www.javaboy.org') {
            return 403;
        }
        # 你的配置
    }
}
```

这种方法允许你在特定域名的server块中直接控制访问权限，只有当`$host`变量与你的域名匹配时，才会允许访问。

**直接禁止IP**

```nginx
http {
    server {
        listen 80;
        server_name www.javaboy.org;
        ...
    }
    server {
        listen 80;
        server_name www.itboyhub.com;
        ...
    }
    # 直接指定ip server_name
    server {
        listen 80;
        server_name 11.11.11.11;
        return 403; # 403 forbidden
    }
}
```

这样配置后，只有通过指定的域名才能访问网站，直接通过IP地址访问将会受到限制。

### 限制Nginx请求方法

通过限制特定的HTTP请求方法，可以减少服务器受到自动化攻击的风险，并且可以防止某些类型的Web漏洞，如SQL注入或跨站脚本（XSS）攻击。有两种配置方式，逐一说明。

**只允许GET和POST**

在`server`或`location`块中，使用`if`语句来检查请求方法，并返回403错误码以拒绝其他方法。

```nginx
server {
    listen 80;
    server_name javaboy.org;
    location / {
        if ($request_method !~* (GET|POST)) {
            return 403;
        }
        # 其他配置...
    }
}
```

这种方法会拒绝所有非GET和POST的请求方法。

**使用map模块**

对于更复杂的限制逻辑，可以使用Nginx的`map`模块来动态设置请求方法的限制。

```nginx
http {
    map $request_method $block_request {
        default 0;
        POST 1;
        PUT 1;
    }

    server {
        listen 80;
        server_name javaboy.org;
        location / {
            if ($block_request) {
                return 403;
            }
            # 其他配置...
        }
    }
}
```

在这个例子中，所有POST和PUT请求都会被拒绝。

### 错误页面重定向

在Nginx中配置错误页面重定向，除了安全因素之外，还有很多好处，比如：

1. **提升用户体验**：通过提供更友好的错误页面，可以减少用户在遇到错误时的困惑和挫败感。
2. **增强SEO效果**：自定义错误页面可以帮助搜索引擎更好地理解网站结构，避免因错误页面导致的SEO问题。
3. **维护品牌形象**：错误页面是网站的一部分，通过自定义错误页面，可以保持品牌一致性，提升专业形象。
4. **提供错误信息**：自定义错误页面可以提供有用的错误信息或解决方案，帮助用户理解问题所在。

在Nginx配置文件中，可以使用`error_page`指令来定义特定错误代码的重定向页面。例如，将404错误重定向到自定义的404页面：

```nginx
server {
    listen 80;
    server_name javaboy.org;

    error_page 404 /404.html;
    location = /404.html {
        root /path/to/error/pages;
        internal;
    }
}
```

在这个配置中，当Nginx返回404错误时，它会显示位于`/path/to/error/pages/404.html`的自定义错误页面，而不是默认的错误页面。`internal`指令确保这个页面只对Nginx内部请求可见，不会被外部直接访问。当然，上面这个配置也可以同时枚举多个错误状态码：

```nginx
error_page 500 502 503 504 /50x.html;
location = /50x.html {
    root /usr/share/nginx/html;
}
```

这个配置会将所有500系列错误重定向到`/50x.html`，并显示位于`/usr/share/nginx/html/50x.html`的自定义错误页面。

### 日志保留半年

保留Nginx日志半年的原因有很多，比如：

1. **安全审计**：日志文件可以用于安全审计，帮助分析和追踪潜在的攻击或异常行为。
2. **故障排查**：在系统出现故障时，日志文件是诊断问题的重要工具，可以帮助快速定位问题原因。
3. **性能监控**：通过分析日志，可以了解网站的访问情况和性能瓶颈，从而进行相应的优化。
4. **合规性要求**：某些行业法规可能要求保留一定期限的日志记录，以满足合规性检查。

要配置Nginx日志保留半年，通常需要使用`logrotate`工具来实现日志文件的定期轮换和压缩。在`/etc/logrotate.d/`目录下创建一个名为`nginx`的配置文件，内容如下：

```nginx
/var/log/nginx/*.log {
    daily
    rotate 180
    missingok
    notifempty
    compress
    delaycompress
    sharedscripts
    postrotate
        [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
    endscript
}
```

这个配置会每天检查Nginx日志文件，并将它们保留180天（约6个月），然后自动压缩旧的日志文件。`postrotate`部分的命令会在日志轮换后重新打开Nginx日志文件，以便继续记录新的日志信息。通过这些配置，我们可以确保Nginx的日志文件被保留半年，同时旧的日志文件会被压缩以节省磁盘空间。

### 设置缓冲区

Nginx的缓冲区溢出攻击是一种常见的安全漏洞，它发生在程序试图向一个缓冲区写入超出其预分配大小的数据时。这种攻击可能导致数据覆盖了相邻的内存区域，可能破坏程序的执行流程，甚至可以被恶意攻击者利用来执行恶意代码。为了防止缓冲区溢出类攻击事件，可以设置客户端请求体、请求头和客户端最大请求体的缓冲区大小。配置方式如下：

```nginx
client_body_buffer_size 1K;
client_header_buffer_size 1k;
client_max_body_size 1k;
large_client_header_buffers 2 1k;
```

这四行配置含义如下：

- `client_body_buffer_size1K;`：这条指令设置了Nginx用来读取客户端请求体（比如POST请求中的数据）的缓冲区大小。在这个例子中，缓冲区大小被设置为1KB。如果请求体的大小超过了这个缓冲区的大小，Nginx会使用磁盘来暂存超出部分的数据。
- `client_header_buffer_size1k;`：这条指令定义了Nginx用来读取客户端HTTP请求头部的缓冲区大小。这里设置的大小是1KB。如果请求头部的大小超过了这个缓冲区的大小，Nginx会使用`large_client_header_buffers`定义的缓冲区。
- `client_max_body_size1k;`：这条指令限制了Nginx服务器愿意接收的最大请求体大小。如果客户端发送的请求体超过了这个大小（在这个例子中是1KB），Nginx将返回一个413（RequestEntityTooLarge）错误。
- `large_client_header_buffers21k;`：这条指令定义了Nginx用于处理大于`client_header_buffer_size`指定大小的请求头的缓冲区数量和大小。这里配置了2个大小为1KB的缓冲区。当请求头的大小超过了`client_header_buffer_size`定义的缓冲区大小时，Nginx会使用这两个额外的缓冲区来处理请求头。

这些配置对于防止缓冲区溢出攻击和处理大请求都是非常重要的。

### 使用普通用户启动

在Linux系统中，只有root用户或者具有特定权限的用户才能绑定1024以下的端口，如80端口（HTTP）和443端口（HTTPS）。如果Nginx以root用户运行，它将拥有过高的权限，这可能会带来安全风险。因此，为了最小化权限，通常会创建一个普通用户来运行Nginx，以减少潜在的安全漏洞。配置方式如下：

**创建用户**

首先，你需要创建一个普通用户和用户组，例如`nginx`。

```shell
groupadd nginx
useradd -g nginx -d /usr/local/nginx nginx
```

这里创建了一个名为`nginx`的用户和组，并设置了用户的家目录。

**授权访问**

确保新用户有权访问Nginx的配置文件、日志文件和服务器文件。

```shell
chown -R nginx:nginx /usr/local/nginx/
```

这条命令将Nginx目录及其所有子目录和文件的所有权更改为新创建的`nginx`用户和组。

**配置Nginx**

编辑Nginx配置文件（通常位于`/etc/nginx/nginx.conf`），设置`user`指令以指定Nginx工作进程的用户。

```nginx
user nginx;
```

这行配置指定Nginx应该以`nginx`用户的身份运行。

**设置权限**

如果需要，可以使用`setcap`命令赋予Nginx监听1024以下端口的能力，而不需要以root用户运行。

```shell
setcap cap_net_bind_service=+ep /usr/local/nginx/sbin/nginx
```

这个命令允许Nginx以普通用户身份绑定到80和443端口。

**启动Nginx**

使用普通用户启动Nginx。

```shell
su - nginx
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

这里首先切换到`nginx`用户，然后启动Nginx服务。

**验证**

检查Nginx是否以普通用户启动。

```shell
ps -ef | grep nginx
```

这条命令将显示Nginx的进程信息，你可以验证它是否以`nginx` 用户运行。


## 相关文章

| [这是一个Nginx极简教程，目的在于帮助新手快速入门Nginx。](https://github.com/dunwu/nginx-tutorial) | [就是要让你搞懂Nginx，这篇就够了！](https://mp.weixin.qq.com/s/5Q_VQoQY6kJiMwMHHDIijA) | [Nginx为什么快到根本停不下来？](https://mp.weixin.qq.com/s/e7r2Jt1DlF_4HpZU_IKZkQ) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [手把手教你在CentOS7上搭建Nginx](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&amp;mid=2247490879&amp;idx=1&amp;sn=bd93bc46cdfb7919b9a304c176927dd8&amp;source=41#wechat_redirect) | [nginx实现动态分离,解决css和js等图片加载问题](https://www.cnblogs.com/sz-jack/p/5206159.html) | [nginx反向代理tomcat，js，css静态资源不加载问题](https://blog.csdn.net/white1114579650/article/details/120151335) |
| [彻底搞懂Nginx的五大应用场景](https://mp.weixin.qq.com/s/v6j2HStMHBDlak6UGTF0Hw) | [nginx配置参数](https://blog.51cto.com/ting2junshui/2066268) | [Nginx轻松搞定跨域问题！](https://mp.weixin.qq.com/s/clSjaLJSht5J8woIaiH4gA) |
| [如何使用Nginx优雅地限流？](https://mp.weixin.qq.com/s/YXJ1jcr7XLKTbzf9kyjiEg) | [一文学会Nginx的限流配置](https://mp.weixin.qq.com/s/s4j043__MiXst8wHpEPUoA) | [Nginx如何限流？](https://mp.weixin.qq.com/s/R6GajrvNphXfgKWDsFWzFw) |
|           [nginxconfig.io](https://nginxconfig.io)           | [为什么Nginx比Apache更牛叉？](https://mp.weixin.qq.com/s/pPV5s3uO1sjPTAhz_BDcJg) | [如何用Nginx代理MySQL连接，并限制可访问IP？](https://mp.weixin.qq.com/s/6lvKIQb4yk7uTmufr9pJ8w) |
| [Nginx配置最全详解(万字图文总结)](https://mp.weixin.qq.com/s/---HjW_eobgY-nZI1Kmszw) | [Nginx性能优化的几个方法](https://mp.weixin.qq.com/s/g0axq_lTbNyYV-V7W8CP5w) | [Nginx部署负载均衡服务全解析](https://mp.weixin.qq.com/s/KN5o2hsVXmNm5ZBDURsi4A) |
