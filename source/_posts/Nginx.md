---
title: Nginx
tags: 安装
index_img: /assert/nginx.jpeg
img: https://picx1.zhimg.com/v2-e68d524210343613129267bd2cb75a0d_1440w.jpg

---

## Nginx安装

### 离线安装

#### 下载

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

#### 编译安装

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

#### 启动nginx

```shell
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx -s stop # 立即停止nginx，不保存相关信息
/usr/local/nginx/sbin/nginx -s quit  # 正常退出nginx，保存相关信息
/usr/local/nginx/sbin/nginx -s reload # 重启
```
> [Linux安装Nginx详细图解教程](https://www.cnblogs.com/lovexinyi8/p/5845017.html)

#### 将nginx做成系统服务并且开机自启动

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

#### 下载

[官网地址](https://nginx.org)

#### 安装依赖

```shell
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

#### 安装

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


## Nginx知识点

### nginx判断

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

### ngx_http_core_module模块的变量

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
- **$server_addr **：表示服务器地址
- **$server_name**：表示服务器名称
- **$server_port**：表示服务器端口
- **$server_protocol**：表示服务器向客户端发送响应的协议，如HTTP/1.1或HTTP/1.0

### 日志配置

- **$remote_addr,$http_x_forwarded_for- **：记录客户端IP地址
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

### if语法

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

### return

- 语法：return code
- 默认值：none
- 使用字段：server,location,if
- nginx隐藏版本号
- nginx.conf中修改http zone中的变量值： server_tokens off;
- php-fpm fastcgi.conf中的变量值： fastcgi_param SERVER_SOFTWARE nginx;

### nginx正向代理

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

## location "/" 的作用

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

1. proxy_pass代理地址端口后有目录(包括 / )，转发后地址：代理地址+访问URL目录部分去除location匹配目录
2. proxy_pass代理地址端口后无任何，转发后地址：代理地址+访问URL目录部

## 封禁恶意ip

单独网站屏蔽IP的方法，把`include xxx;`放到网址对应的在server{}语句块,虚拟主机所有网站屏蔽IP的方法，把`include xxx;`放到http{}语句块。

### 手动封禁

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

### 自动化封禁

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

## 语法规则

location [= | \~ | \~\* | ^\~ ] /uri/ { … }

- =：表示精确匹配,这个优先级也是最高的
- ^\~：表示uri以某个常规字符串开头，理解为匹配url路径即可。nginx不对url做编码，因此请求为/static/20%/aa，可以被规则^~ /static/ /aa匹配到（注意是空格）。
- \~：表示区分大小写的正则匹配
- \~\*：表示不区分大小写的正则匹配(和上面的唯一区别就是大小写)
- !\~和!\~\*：分别为区分大小写不匹配及不区分大小写不匹配的正则
- /：通用匹配，任何请求都会匹配到，默认匹配.

**语法的一些规则和优先级**
多个location配置的情况下匹配顺序为：首先匹配=，其次匹配^\~,其次是按文件中顺序的正则匹配，最后是交给/通用匹配。当有匹配成功时候，停止匹配，按当前匹配规则处理请求。

### 正则匹配

- ^ 以什么开始
- \$ 以什么结束
- \~ 区分大小写匹配
- \~\* 不区分大小写匹配

```nginx
# ^/api/user$
# 匹配任何以/uri/开头的任务查询并且停止搜索
location ^~ /uri/
```

### 精准匹配

```nginx
# =:表示精准匹配,只要完全匹配才能生效
location= /uri
```

### 前缀匹配

不带任务修饰符,表示前缀匹配

### 通用匹配

任务未匹配到其他location的请求都会匹配到

```nginx
location /
```

### 优先级

精准匹配>前缀匹配(若有多个匹配荐匹配成功,那么选择匹配长的并记录)>正则匹配

### 案例

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

## 地址重定向

### rewrite地址重定向，实现URL重定向的重要指令，他根据regex(正则表达式)来匹配内容跳转到

语法: 
```bash
rewrite regex replacement[flag]
```
这是⼀个正则表达式，匹配完整的域名和后⾯的路径地址,replacement部分https://xdclass.net/$1, $1是取自regex部分()里的内容.rewrite ^/(.\*) https://xdclass.net/$1 permanent

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


### 应用场景

– 非法访问跳转,防盗链

– 网站更换新域名

– http跳转https

– 不同地址访问同⼀个虚拟主机的资源

## 配置websocket反向代理

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

## 性能优化-服务端缓存前置

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
  - add_header Nging-Cache "$upstream_cache_status"**
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

## 性能优化-动静分离

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

## https配置

### 1. 删除原先的nginx，新增ssl模块

```shell
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --withhttp_ssl_module
make
make install
#查看是否成功
/usr/local/nginx/sbin/nginx -V
```

### 2. Nginx配置https证书

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

### 3. https访问实操

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

## 高可用

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
  - vip能ping通，vip监听的端口不通: 第⼀个原因:nginx1和nginx2两台服务器的服务没有正常启动
  - vip ping不通: 核对是否出现裂脑,常见原因为防火墙配置所致导致多播心跳失败,核对keepalived的配置是否正确：需要关闭selinux，不然sh脚本可能不生效
```shell
#查看
getenforce
#关闭
setenforce 0
```

## 运维统计

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

## 防止sql注入

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

## 相关文章

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
- [如何用Nginx代理MySQL连接，并限制可访问IP？](https://mp.weixin.qq.com/s/6lvKIQb4yk7uTmufr9pJ8w)
