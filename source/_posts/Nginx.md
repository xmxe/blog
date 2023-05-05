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
[Linux安装Nginx详细图解教程](https://www.cnblogs.com/lovexinyi8/p/5845017.html)

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

#### nginx.conf

```shell
# 指定Nginx服务的用户和用户组
# user  nobody nobody;

# 工作进程：数目。根据硬件调整,通常等于CPU数量或者2倍于CPU(允许生成的进程数)
worker_processes  8;
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;

# 制定日志路径、级别。这个设置可以放入全局块,http块,server块.级别以此为:debug|info|notice|warn|error|crit|alert|emerg
error_log  /usr/local/logs/error.log  notice;

# pid(进程标识符)指定nginx进程运行文件存放位置
pid  /usr/local/logs/nginx.pid;

# 用于nginx工作模式的配置
events {
    # 设置网路连接序列化,防止惊群现象发生,默认为on
    accept_mutex  off;

    # 设置一个进程是否同时接受多个网络连接,默认为off
    multi_accept  on;

    # 事件驱动模型,select|poll|kqueue|epoll|resig|/dev/poll|eventport
    use  epoll;

    # 指定进程可以打开的最大描述符：数目。
    worker_connections  1024;
}

# 用于进行http协议信息的一些配置
http {
    # 设定mime类型,类型由mime.type文件定义
    include  mime.types;

    # 默认文件类型,默认为text/plain
    default_type  application/octet-stream;

    # 日志格式
    log_format  myformat  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

    # access_log  logs/access.log  myformat;  #combined为日志格式的默认值

    # 取消访问日志
    access_log  off;

    # 设置DNS解析超时时间
    resolver_timeout  60s;

    # tcp连接关闭前的延时时间
    lingering_timeout  5s;

    # 允许sendfile方式传输文件,默认为off,可以在http块,server块,location块,将文件的回写过程交给数据缓冲区完成,提升性能
    sendfile  on;

    # 每个进程每次调用传输数量不能大于设定的值,默认为0,即不设上限。
    sendfile_max_chunk  100k;

    # 每个TCP连接最多可以保持多长时间,默认为75s,可以在http,server,location块。
    keepalive_timeout  65;

    # 让nginx在一个数据包中发送所有的头文件,而不是一个一个单独发
    tcp_nopush  on;

    # 启动TCP_NODELAY,禁用Nagle算法,允许小包的发送
    tcp_nodelay  on;

    # 请求头设置缓冲区
    client_header_buffer_size  4k;

    # 指定客户端与服务端建立连接后发送request body的超时时间,如果客户端在指定时间内没有发送任何内容,返回408(Request Timed Out)
    client_body_timeout  60s;

    # 客户端向服务端发送一个完整的request header的超时时间。如果客户端在指定时间内没有发送一个完整的request header,返回408()
    client_header_timeout  60s;

    # 上传文件最大限制20兆
    client_max_body_size  20m;

    # 缓冲区不足按照这个分配
    large_client_header_buffers 4 8k;

    # max指定缓存数量,inactive是指经过多长时间文件没被请求后删除缓存
    open_file_cache  max=102400 inactive=20s;

    # 多长时间检查一次缓存的有效信息
    open_file_cache_valid  30s;

    # 指令中的inactive参数时间内文件的最少使用次数,如果超过这个数字,文件更改信息一直是在缓存中打开的
    open_file_cache_min_uses  1;

    # FastCGI相关参数是为了改善网站的性能:减少资源占用,提高访问速度。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 8 128k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    # 限制ip访问(限流)
    # 定义一个名为one的内存空间,大小是10M,以$binary_remote_addr($binary_remote_addr 表示通过remote_addr这个标识来做限制,“binary_”的目的是缩写内存占用量,是限制同一客户端ip地址)为key,限制平均每秒的请求为20个,可以有比如rate=30r/m的
    # 1M能存储16000个状态,rete的值必须为整数,rate=20r/s表示允许相同标识的客户端的访问频次
    # 在Server块中通过limit_req使用
    limit_req_zone $binary_remote_addr zone=one:10m rate=20r/s;

    proxy_buffering  on;
    proxy_cache_valid  any  10m;

    # proxy_cache_path:本地路径,缓存文件存放地址；
    # levels:默认所有缓存文件都放在同一个/path/to/cache下,从而影响缓存的性能,大部分场景推荐使用2级目录来存储缓存文件；
    # key_zone:在共享内存中设置一块存储区域来存放缓存的key和metadata(类似使用次数),这样nginx可以快速判断一个request是否命中或者未命#中缓存,1m可以存储8000个key,10m可以存储80000个key
    # max_size:最大cache空间,如果不指定,会使用掉所有disk space,当达到配额后,会删除最少使用的cache文件
    # inactive:未被访问文件在缓存中保留时间,本配置中如果60分钟未被访问则不论状态是否为expired,缓存控制程序会删掉文件,默认为10分钟,需要注意的是,inactive和expired配置项的含义是不同的,expired只是缓存过期,但不会被删除,inactive是删除指定时间内未被访问的缓存文件
    # use_temp_path如果为off,则nginx会将缓存文件直接写入指定的cache文件中,而不是使用temp_path存储,official建议为off,避免文件在不同文件系统中不必要的拷贝
    proxy_cache_path  /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;

    # 为存储承载从代理服务器接收到的数据的临时文件定义目录
    proxy_temp_path /data/temp;
    
    proxy_buffer_size  4k;
    proxy_buffers 100  8k;

    # 采用gzip压缩的形式发送数据。这将会减少我们发送的数据量
    gzip  on;
    # 最小压缩文件大小
    gzip_min_length  1k;
    # 压缩缓冲区
    gzip_buffers 4 16k;
    # 压缩版本(默认1.1,前端如果是squid2.5请使用1.0)
    gzip_http_version 1.1;
    # 压缩等级
    gzip_comp_level 2;
    # 压缩类型,默认就已经包含text/html
    gzip_types text/plain application/x-javascript text/css application/xml;
    # 给CDN和代理服务器使用,针对相同url,可以根据头信息返回压缩和非压缩副本
    gzip_vary on;

    # 配置段: http,server,location 服务端向客户端传输数据的超时时间。
    send_timeout 30s;
    
    # 默认off,nginx默认request的header的中包含'_'时，会自动忽略掉。
    underscores_in_headers on;

    upstream wypt2 {
        # least_conn,ip_hash,weight,fair等根据实际情况选择一个即可

        # 把请求分配到连接数最少的server
        least_conn;

        # 每个请求按访问ip的hash结果分配,这样每个访客固定访问一个后端服务器,可以解决session的问题。
        ip_hash;

        # weight=1 指定轮询几率,weight和访问比率成正比,用于后端服务器性能不均的情况。
        # fail_timeout：max_fails次失败后，暂停的时间,当该时间内服务器没响应,则认为服务器失效,默认10s
        # max_fails：允许连接失败次数,默认为1
        # 这2个参数一起配合,来控制nginx怎样认为upstream中的某个server是失效的,当在fail_timeout的时间内,某个server连接失败了max_fails次,则nginx会认为该server不工作了。同时,在接下来的fail_timeout时间内,nginx不再将请求分发给失效的server。
        # down 表示当前的server暂时不参与负载
        # backup 备用服务器,其它所有的非backup机器down或者忙的时候，请求backup机器，所以这台机器压力会最轻
        server 127.0.0.1:8080 weight=1 fail_timeout=2s max_fails=3;
        server 127.0.0.1:8080 weight=1 backup;

        #按后端服务器的响应时间来分配请求,响应时间短的优先分配(第三方插件实现)
        fair;

        # 与ip_hash类似，但是按照访问url的hash结果来分配请求，使得每个url定向到同一个后端服务器，主要应用于后端服务器为缓存时的场景下(第三方插件实现)
        hash $request_uri;
        hash_method crc32;
    }

    upstream wypt3 {
        server 127.0.0.1:9090 weight=1 fail_timeout=2s max_fails=3;
        server 127.0.0.1:9090 weight=1 down;
    }
   

    # server:用于进行服务器访问信息的配置
    server {
        # 单连接请求上限次数。
        keepalive_requests 120;

        # 配置监听端口
        listen  80;
        server_name  192.168.7.135;
        #index  index.html index.html;

        charset  utf8;

        #access_log  logs/host.access.log  main;
        
        location / {
            root html;
            index  index.html index.htm;

            # 限制ip访问,与上面的limit_req_zone配置联动
            # 限制每ip每秒不超过1个请求,漏桶数burst为5,也就是队列．
            # nodelay,如果不设置该选项,严格使用平均速率限制请求数,超过的请求被延时处理．
            # brust这个配置的意思是设置一个大小为5的缓冲区当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内。举个栗子：设置rate=20r/s每秒请求数为２０个,漏桶数burst为5个,如果第1秒、2,3,4秒请求为19个,第5秒的请求为25个是被允许的,可以理解为20+5,但是如果你第1秒就25个请求,第2秒超过20的请求返回503错误,如果区域存储空间不足,服务器将返回503（服务临时不可用）错误,速率在每秒请求中指定（r/s）。如果需要每秒少于一个请求的速率,则以每分钟的请求（r/m）指定。
            # zone=one 设置使用哪个配置区域来做限制，与上面limit_req_zone 里的name对应。
            limit_req zone=one burst=5 nodelay;

            # 下面配置可以限制特定UA（比如搜索引擎）的访问：具体可以搜索nginx限流文章
            # limit_req_zone  $anti_spider  zone=one:10m   rate=10r/s;  上面配置
            # limit_req zone=one burst=100 nodelay;  
            # if ($http_user_agent ~* "googlebot|bingbot|Feedfetcher-Google") {
                # set $anti_spider $http_user_agent;  
            # }

　　　　　　　
            # response header添加响应内容,可查看请求被转发到哪台服务器
            add_header upstreamIP $upstream_addr;

            # response header添加响应内容,可查看状态码
　　　　　　  add_header upstreamCode $upstream_status;

            # 允许客户端请求的最大单文件字节数
            client_max_body_size 10m;

            # 缓冲区代理缓冲用户端请求的最大字节数
            client_body_buffer_size 128k;

            # 后面加'/'和不加'/'的区别,比如我们现在客户端请求 http://ip/gsipV3/xxx,当location块使用了'/'作为uri变量的值来匹配的,加不加'/'没有区别,当匹配location /gsipV3/时,如果不加'/',那么它会指向内部服务器的地址为：http://wypt3/gsipV3/xxx,如果加'/'的话，那么它会指向内部服务器的地址为：http://wypt3/xxx
            proxy_pass   http://wypt3;

            # 启用proxy cache,指定key_zone;
            proxy_cache  my-cache;
            proxy_cache_valid  200;
            proxy_redirect  off;

            # proxy_set_header 重新设置往服务器发送的请求头
            proxy_set_header X-Forwarded-Proto $scheme;

            # $remote_addr 获取到上一级代理的IP
            proxy_set_header X-Real-IP $remote_addr;
            
            # 后端的Web服务器可以通过request.getAttribute("X-Forwarded-For")获取用户真实IP,$proxy_add_x_forwarded_for 获取到结果例如：(223.104.6.125, 10.10.10.45),第一个是用户的真实IP,第二个是一级代理的IP,依此类推。
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            # 如果客户端发过来的请求头中没有HOST这个字段时建议使用$host 这表示请求中的server name
            proxy_set_header Host $http_host;
            
            # nginx反向代理连接超时时间
            proxy_connect_timeout  10;

            # 后端服务器数据回传时间,就是在规定时间之内后端服务器必须传完所有的数据(代理发送超时)
            proxy_send_timeout 90;

            # 连接成功后等候后端服务器响应时间(也可以说是后端服务器处理请求的时间)
            proxy_read_timeout  90;

            proxy_buffering  on;

            # 设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffer_size  4k;

            # proxy_buffers缓冲区,网页平均在32k以下的设置
            proxy_buffers 4 32k;

            # 高负荷下缓冲大小（proxy_buffers*2）
            proxy_busy_buffers_size 64k;
            proxy_temp_file_write_size 64k;

            if ($request_uri ~* ^/gsipV3/ ){
                proxy_pass http://wypt3;
            }

            if ($request_uri ~* gsipV3$ ){
                proxy_pass http://wypt3;
            }
        }


        # nginx代理两个tomcat（两个项目）,不带项目名访问9090端口,带项目名访问8080端口,并且访问路径(/gsipV3)需要和proxy_pass代理的项目名一致,因为只有url中带有/gsipV3才会代理到这个模块,否则默认转到location /{}模块
        location /gsipV3/ {
            root html;
            index  index.html index.htm;

            #response header添加响应内容,可查看请求被转发到哪台服务器
            add_header proxyIP $upstream_addr;
            
            #response header添加响应内容,可查看状态码
　　　　　　  add_header proxyCode $upstream_status;
            
            # 要加后面的'/',不加的话最终路径会变成http://wypt2/gsipV3/gsipV3
            proxy_pass   http://wypt2/gsipV3/;

            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Real-IP $remote_addr;

            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            #如果客户端发过来的请求头中没有HOST这个字段时建议使用$host 这表示请求中的server name
            proxy_set_header Host $http_host; 
                      
        }

        # 解决/usr/local/nginx/html/favicon.ico" failed (2: No such file or directory)报错问题
        location /favicon.ico {
            log_not_found off;
            access_log off;
        }

        # 所有的jsp页面均由tomcat处理
        location ~ \.(jsp|jspx|dp)?$
        {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_pass http://zh;
        }

        # 假如请求包括svgs,并且以.svg结尾时（~*不区分大小写,~区分大小写）
        location ~* .*/svgs/.*\.svg$ {

            # 假如$request_uri不以gsipV3开头时（不区分大小写）
            if ($request_uri !~* ^/gsipV3/ ){
                proxy_pass http://10.37.169.201:805/gsipV3$request_uri;
            }

            # 假如$request_uri以gsipV3开头时（不区分大小写）
            if ($request_uri ~* ^/gsipV3/ ){
                proxy_pass http://10.37.169.201:805;
            }
        }


        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css)$
        {
            root /usr/local/tomcat/tomcat-80/webapps/ifm;
            if (-f $request_filename) {
                expires 1d;
                break;
            }
        }

        # fastdfs+nginx配置
        location /group1/M00/ {
           root  /home/fdfs_storage/data;
           
           # 安装ngx_fastdfs_module模块后引入
           ngx_fastdfs_module;
        }

        # 前端项目部署(例如vue项目打包成dist文件夹，使用nginx部署)使用此配置
        # location / {
        location /images/ {
            root /opt/html/;
            # 依次寻找,找到即返回
            index index.html index.htm;

            # 比如请求127.0.0.1/images/test.gif 会依次查找 1.文件/opt/html/images/test.gif 2.文件夹 /opt/html/images/test.gif/下的index文件 3.请求127.0.0.1/index.html
            # try_files 如果不写上$uri/当直接访问一个目录路径时,并不会去匹配目录下的索引页即访问127.0.0.1/images/不会去访问127.0.0.1/images/index.html
            try_files $uri $uri/ /index.html;
        }
    

        # redirect server error pages to the static page /50x.html
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        #error_page  404              /404.html;

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

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
