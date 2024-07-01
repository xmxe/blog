---
title: FastDFS+Nginx安装
tags: 安装
img: https://img2.baidu.com/it/u=3204455731,2423486300&fm=253&fmt=auto&app=138&f=JPG?w=600&h=338

---

从[此网站](https://github.com/happyfish100)下载相关压缩包，包括但不限于libfastcommon-master.zip、fastdfs-master.zip、fastdfs-nginx-module-master等

### 安装所需的依赖包

```shell
yum install make cmake gcc gcc-c++
```

### 安装libfatscommon

```shell
cd /usr/local/src
unzip libfastcommon-master.zip
cd libfastcommon-master
# 编译、安装
./make.sh
./make.sh install
```

### 安装FastDFS

```shell
cd /usr/local/src
unzip fastdfs-master.zip
cd fastdfs-master
# 编译、安装
./make.sh
./make.sh install
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf # 跟踪服务配置
```

### 修改tracker.conf 文件

```shell
disabled=false # 启用配置文件
port=22122 # tracker服务器端口（默认22122）
base_path=/home/fastdfs # 存储日志和数据的根目录
```

### 创建目录

```shell
mkdir -p /home/fastdfs
```

### 开放端口

```shell
vi /etc/sysconfig/iptables
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22122 -j ACCEPT
service iptables restart # 开放22122端口
```

### 启动tracker服务器

```shell
/etc/init.d/fdfs_trackerd start # 会在新建的/home/fastdfs生成log和data两个目录
/etc/init.d/fdfs_trackerd stop # 停止tracker服务
# 若无法启动，暂时用/usr/local/src/fastdfs-master/init.d/fdfs_trackerd start启动
```

### 设置开机启动

```shell
chkconfig fdfs_trakcerd on
```

### 存储服务配置

```shell
cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf

# 修改的内容如下:
disabled=false # 启用配置文件
port=23000 # storage服务端口
group_name=group1 # 组名（第一组为group1，第二组为group2，依次类推...）
base_path=/home/fdfs_storage # 数据和日志文件存储根目录
store_path0=/home/fdfs_storage # 第一个存储目录，第二个存储目录起名为：#store_path1=xxx，其它存储目录名依次类推...
store_path_count=1 # 存储路径个数，需要和store_path个数匹配
tracker_server=192.168.0.200:22122  # tracker服务器IP和端口
http.server_port=80 # http访问文件的端口
```

### 创建目录 开放端口

```shell
mkdir -p /home/fdfs_storage
vi /etc/sysconfig/iptables # 开放23000端口
-A INPUT -m state --state NEW -m tcp -p tcp --dport 23000 -j ACCEPT 
service iptables restart
```

### 启动storaged服务器

```shell
/etc/init.d/fdfs_storaged start #会在新建的/home/fdfs_storaged生成log和data两个目录
/etc/init.d/fdfs_storaged stop# 停止服务(若无法启动，暂时用/usr/local/src/fastdfs-master/init.d/fdfs_storaged start启动)
```

### 设置storaged服务开机启动

```shell
chkconfig fdfs_storaged on 
```

### 安装nginx

1. wget http://nginx.org/download/nginx-1.15.2.tar.gz
2. wget https://github.com/happyfish100/fastdfs-nginx-module/archive/V1.20.tar.gz
3. 解压nginx：`tar -zxvf nginx-1.15.2.tar.gz`
4. 解压fastdfs-nginx-module：`tar -xvf V1.20.tar.gz`
5. 进入nginx目录：`cd nginx-1.10.1`
6. 安装依赖的库`apt-get update`,`apt-get install libpcre3 libpcre3-dev openssl libssl-dev libperl-dev`
7. 配置，并加载fastdfs-nginx-module模块：安装nginx的时候`./configure --prefix=/opt/nginx --sbin-path=/usr/bin/nginx --add-module=/usr/local/src/fastdfs-nginx-module-master/src`注意是否报错！
8. 编译安装
```shell
make && make install
cp /usr/local/src/fastdfs-nginx-module-master/src/mod_fastdfs.conf  /etc/fdfs/
vi /etc/fdfs/mod_fastdfs.conf

connect_timeout=10 # 客户端访问文件连接超时时长（单位：秒）
base_path=/tmp # 临时目录
tracker_server=192.168.0.200:22122 # tracker服务IP和端口
storage_server_port=23000 # storage服务端口
group_name=group1 # 组名
url_have_group_name=true # 访问链接前缀加上组名
store_path0=/home/fdfs_storage # 文件存储路径
cd /usr/local/src/fastdfs-master/conf
cp http.conf mime.types /etc/fdfs/
ln -s /home/fdfs_storage/data/ /home/fdfs_storage/data/M00
```
nginx.conf配置
```nginx
{
    location ~/group([0-9])/M00 {
        root /home/fdfs_storage/data
            ngx_fastdfs_module;
    }
}
```

/usr/bin/nginx 启动

### 相关文章

| [FastDFS分布式文件系统集群安装与配置](https://blog.csdn.net/xyang81/article/details/52928230) | [FastDFS分布式文件系统安装与使用（单节点）](https://blog.csdn.net/xyang81/article/details/52837974) | [fastdfs安装配置](https://www.cnblogs.com/sunshinekevin/p/8085554.html) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [分布式文件系统FastDFS安装教程](https://www.cnblogs.com/handsomeye/p/9451568.html) | [松哥手把手教你用FastDFS构建分布式文件管理系统](https://mp.weixin.qq.com/s/N20mYUnHPhdc76K5MayjFQ) | [需要搭建一个高性能的文件系统？我推荐你试试它](https://mp.weixin.qq.com/s/PMQU7Bsp6RQRig8gti3Grg) |
| [再聊FastDFS，顺便说说OBS服务](https://mp.weixin.qq.com/s/0n0xNf9D8hrWCvAC8hmYfg) |                                                              |                                                              |

