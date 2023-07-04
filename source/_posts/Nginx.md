---
title: Nginx
tags: 安装
index_img: /assert/nginx.jpeg
img: https://picx1.zhimg.com/v2-e68d524210343613129267bd2cb75a0d_1440w.jpg

---

#### Nginx安装

##### 下载

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

##### 编译安装
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
##### 启动nginx
```shell
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx -s stop # 立即停止nginx，不保存相关信息
/usr/local/nginx/sbin/nginx -s quit  # 正常退出nginx，保存相关信息
/usr/local/nginx/sbin/nginx -s reload # 重启
```
> [Linux安装Nginx详细图解教程](https://www.cnblogs.com/lovexinyi8/p/5845017.html)

##### 将nginx做成系统服务并且开机自启动

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

#### Nginx知识点

##### nginx判断

1、正则表达式匹配：

==:等值比较;
\~：与指定正则表达式模式匹配时返回“真”，判断匹配与否时区分字符大小写；
\~\*：与指定正则表达式模式匹配时返回“真”，判断匹配与否时不区分字符大小写；
!\~：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时区分字符大小写；
!\~\*：与指定正则表达式模式不匹配时返回“真”，判断匹配与否时不区分字符大小写；

2、文件及目录匹配判断：

-f, !-f：判断指定的路径是否为存在且为文件；
-d, !-d：判断指定的路径是否为存在且为目录；
-e, !-e：判断指定的路径是否存在，文件或目录均可；
-x, !-x：判断指定路径的文件是否存在且可执行；

##### 语法规则

location [=|~|~*|^~] /uri/ { … }

= 表示精确匹配,这个优先级也是最高的
^~ 表示uri以某个常规字符串开头，理解为匹配url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）。
~ 表示区分大小写的正则匹配
\~\* 表示不区分大小写的正则匹配(和上面的唯一区别就是大小写)
!\~和!\~\*分别为区分大小写不匹配及不区分大小写不匹配的正则
/ 通用匹配，任何请求都会匹配到，默认匹配.

下面讲讲这些语法的一些规则和优先级

多个location配置的情况下匹配顺序为（参考资料而来，还未实际验证，试试就知道了，不必拘泥，仅供参考）：
优先级 = ^~ /
首先匹配=，其次匹配^\~,其次是按文件中顺序的正则匹配，最后是交给/通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。例子，有如下匹配规则：

##### ngx_http_core_module模块的变量

$arg_PARAMETER HTTP请求中某个参数的值，如/index.php?site=www.domain.com，可以用$arg_site取得www.domain.com这个值。

$args HTTP请求中的完整参数。例如，在请求/index.php?width=400&height=200中，$args表示字符串width=400&height=200.

$binary_remote_addr 二进制格式的客户端地址。例如：\x0A\xE0B\x0E

$body_bytes_sent 表示在向客户端发送的http响应中，包体部分的字节数

$content_length 表示客户端请求头部中的Content-Length字段

$content_type 表示客户端请求头部中的Content-Type字段

$cookie_COOKIE 表示在客户端请求头部中的cookie字段

$document_root 表示当前请求所使用的root配置项的值

$uri 表示当前请求的URI，不带任何参数

$document_uri与$uri含义相同

$request_uri表示客户端发来的原始请求URI，带完整的参数。$uri和$document_uri未必是用户的原始请求，在内部重定向后可能是重定向后的URI，而$request_uri永远不会改变，始终是客户端的原始URI.

$host 表示客户端请求头部中的Host字段。如果Host字段不存在，则以实际处理的server（虚拟主机）名称代替。如果Host字段中带有端口，如IP:PORT，那么$host是去掉端口的，它的值为IP。$host是全小写的。这些特性与http_HEADER中的http_host不同，http_host只取出Host头部对应的值。

$hostname 表示Nginx所在机器的名称，与gethostbyname调用返回的值相同

$http_HEADER 表示当前HTTP请求中相应头部的值。HEADER名称全小写。例如，示请求中Host头部对应的值用$http_host表

$sent_http_HEADER 表示返回客户端的HTTP响应中相应头部的值。HEADER名称全小写。例如，用$sent_http_content_type表示响应中Content-Type头部对应的值

$is_args 表示请求中的URI是否带参数，如果带参数，$is_args值为?，如果不带参数，则是空字符串

$limit_rate 表示当前连接的限速是多少，0表示无限速

$nginx_version 表示当前Nginx的版本号

$query_string 请求URI中的参数，与$args相同，然而$query_string是只读的不会改变

$remote_addr 表示客户端的地址

$remote_port 表示客户端连接使用的端口

$remote_user 表示使用Auth Basic Module时定义的用户名

$request_filename 表示用户请求中的URI经过root或alias转换后的文件路径

$request_body 表示HTTP请求中的包体，该参数只在proxy_pass或fastcgi_pass中有意义

$request_body_file 表示HTTP请求中的包体存储的临时文件名

$request_completion 当请求已经全部完成时，其值为“ok”。若没有完成，就要返回客户端，则其值为空字符串；或者在断点续传等情况下使用HTTP range访问的并不是文件的最后一块，那么其值也是空字符串。

$request_method 表示HTTP请求的方法名，如GET、PUT、POST等

$scheme 表示HTTP scheme，如在请求https://nginx.com/中表示https

$server_addr 表示服务器地址

$server_name 表示服务器名称

$server_port 表示服务器端口

$server_protocol 表示服务器向客户端发送响应的协议，如HTTP/1.1或HTTP/1.0

##### 日志配置

$remote_addr,$http_x_forwarded_for记录客户端IP地址

$remote_user 记录客户端用户名称

$request 记录请求的URL和HTTP协议

$status 记录请求状态

$body_bytes_sent 发送给客户端的字节数，不包括响应头的大小；该变量与Apache模块mod_log_config里的“%B”参数兼容。

$bytes_sent 发送给客户端的总字节数。

$connection 连接的序列号。

$connection_requests 当前通过一个连接获得的请求数量。

$msec 日志写入时间。单位为秒，精度是毫秒。

$pipe 如果请求是通过HTTP流水线(pipelined)发送，pipe值为“p”，否则为“.”。

$http_referer 记录从哪个页面链接访问过来的

$http_user_agent 记录客户端浏览器相关信息

$request_length 请求的长度（包括请求行，请求头和请求正文）。

$request_time 请求处理时间，单位为秒，精度毫秒；从读入客户端的第一个字节开始，直到把最后一个字符发送给客户端后进行日志写入为止。

$time_iso8601 ISO8601标准格式下的本地时间。

$time_local 通用日志格式下的本地时间。

##### if

语法：if (condition) { … }

默认值：none

使用字段：server, location

注意：尽量考虑使用trp_files代替。

判断的条件可以有以下值：

1. 一个变量的名称：空字符传”“或者一些“0”开始的字符串为false。
2. 字符串比较：使用=或!=运算符
3. 正则表达式匹配：使用\~(区分大小写)和\~\*(不区分大小写)，取反运算!\~和!\~\*。
4. 文件是否存在：使用-f和!-f操作符
5. 目录是否存在：使用-d和!-d操作符
7. 文件、目录、符号链接是否存在：使用-e和!-e操作符
8. 文件是否可执行：使用-x和!-x操作符

##### return

语法：return code

默认值：none

使用字段：server,location,if

nginx隐藏版本号

nginx.conf中修改http zone中的变量值： server_tokens off;

php-fpm fastcgi.conf中的变量值： fastcgi_param SERVER_SOFTWARE nginx;

##### nginx正向代理

```

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

语法: resolver address ... [valid=time];

默认值: —

配置段: http, server, location

配置DNS服务器IP地址。可以指定多个，以轮询方式请求。

nginx会缓存解析的结果。默认情况下，缓存时间是名字解析响应中的TTL字段的值，可以通过valid参数更改。

#### 相关文章


- [这是一个Nginx极简教程，目的在于帮助新手快速入门Nginx。](https://github.com/dunwu/nginx-tutorial)
- [就是要让你搞懂Nginx，这篇就够了！](https://mp.weixin.qq.com/s/5Q_VQoQY6kJiMwMHHDIijA)
- [Nginx为什么快到根本停不下来？](https://mp.weixin.qq.com/s/e7r2Jt1DlF_4HpZU_IKZkQ)
- [手把手教你在CentOS7上搭建Nginx](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&amp;mid=2247490879&amp;idx=1&amp;sn=bd93bc46cdfb7919b9a304c176927dd8&amp;source=41#wechat_redirect)
- [nginx实现动态分离,解决css和js等图片加载问题](https://www.cnblogs.com/sz-jack/p/5206159.html)
- [nginx反向代理tomcat，js，css静态资源不加载问题](https://blog.csdn.net/white1114579650/article/details/120151335)
- [彻底搞懂Nginx的五大应用场景](https://mp.weixin.qq.com/s/v6j2HStMHBDlak6UGTF0Hw)
- [nginx配置参数](https://blog.51cto.com/ting2junshui/2066268)
- [Nginx轻松搞定跨域问题！](https://mp.weixin.qq.com/s/clSjaLJSht5J8woIaiH4gA)
- [如何使用Nginx优雅地限流？](https://mp.weixin.qq.com/s/YXJ1jcr7XLKTbzf9kyjiEg)
- [Nginx如何限流？](https://mp.weixin.qq.com/s/R6GajrvNphXfgKWDsFWzFw)
- [nginxconfig.io](https://nginxconfig.io)
- [为什么Nginx比Apache更牛叉？](https://mp.weixin.qq.com/s/pPV5s3uO1sjPTAhz_BDcJg)
