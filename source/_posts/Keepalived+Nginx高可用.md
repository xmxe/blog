---
title: Keepalived+Nginx高可用
tags: 安装
img: https://picx.zhimg.com/v2-7472ee03e1417bacc6db982d74ee2b86_720w.jpg

---

## 前期准备

### 服务器

|       服务器       |          CPU           | 内存 |        系统        | 硬盘 |  IP  |
| :----------------: | :--------------------: | :--: | :----------------: | :--: | :--: |
| MASTER/NGINX服务器 | 8核心、主频2.2GHz 以上 |  1G  |      Centos7       | 20G  |      |
| BACKUP/NGINX服务器 | 8核心、主频2.2GHz 以上 |  1G  |      Centos7       | 20G  |      |
|  省调服务器（主）  | 8核心、主频2.2GHz 以上 |  8G  | Windows server2012 | 20G  |      |
|  省调服务器（备）  | 8核心、主频2.2GHz 以上 |  8G  | Windows server2012 | 20G  |      |

### 安装包

| nginx-1.8.1.tar.gz | openssl-fips-2.0.16.tar.gz | pcre-8.3.9.tar.gz | zlib-1.2.11.tar.gz |
| :----------------: | :------------------------: | :---------------: | :----------------: |

### FTP工具

FileZilla

### 流程图

![流程图](https://s21.ax1x.com/2024/07/01/pkcbfw4.jpg)

## 安装

### 安装nginx
上传解压，用Filezilla工具将nginx-1.8.1.tar.gz、opensssl-fips-2.0.16.tar.gz、pcre-8.3.9.tar.gz、安装包上传至服务器/usr/local/wypt目录下并解压。编译安装如下

openssl：
```shell
[root@localhost] tar zxvf openssl-fips-2.0.16.tar.gz
[root@localhost] cd openssl-fips-2.0.16
[root@localhost] ./config && make && make install
```

pcre:
```shell
[root@localhost] tar zxvf pcre-8.39.tar.gz
[root@localhost] cd pcre-8.39
[root@localhost]  ./configure && make && make install
```

zlib:
```shell
[root@localhost] tar zxvf zlib-1.2.11.tar.gz
[root@localhost] cd zlib-1.2.11
[root@localhost] ./configure && make && make install
```

最后安装nginx
```shell
[root@localhost] tar zxvf nginx-1.8.1.tar.gz
[root@localhost] cd nginx-1.8.1
[root@localhost] ./configure && make && make install
```

启动
```shell
/usr/local/nginx/sbin/nginx
```
启动成功后访问localhost:8888能正常打开nginx首页即安装成功

关闭
```shell
/usr/local/nginx/sbin/nginx -s stop # (立即停止，不保存相关信息)
/usr/local/nginx/sbin/nginx -s quit # (正常退出，保存相关信息)
```
重启
```shell
/usr/local/nginx/sbin/nginx -s reload
```
### 安装keepalived

上传解压,用Filezilla工具将keepalived-1.1.20.tar.gz安装包上传至服务器/usr/local/wypt目录下并解压，编译安装如下
```shell
[root@localhost] cd keepalived-1.1.20
[root@localhost] ./configure --prefix=/usr/local/keepalived
[root@bogon keepalived-1.3.5] make
[root@bogon keepalived-1.3.5] make install
[root@keepalived]mkdir /etc/keepalived
# 将配置文件放到默认路径下
[root@keepalived]cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf
# keepalived主程序加入到环境变量（安装目录下）
[root@keepalived]cp /usr/local/keepalived/sbin/keepalived /usr/sbin/keepalived
# keepalived启动脚本变量引用文件，默认文件路径是/etc/sysconfig/
[root@keepalived]cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/keepalived
# keepalived启动脚本（源码目录下），放到/etc/init.d/目录下就可以使用service命令便捷调用
[root@keepalived]cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/rc.d/init.d/keepalived
[root@keepalived]cd /etc/init.d/
[root@init.d]chmod +x keepalived
#添加服务
[root@keepalived]chkconfig --add keepalived
#开机启动
[root@keepalived]chkconfig keepalived on
#查看开机启动的服务
[root@keepalived]chkconfig --list
[root@init.d]service keepalived start
Starting keepalived: /bin/bash: keepalived: command not found
​[FAILED]

[root@init.d]ln -s /usr/local/keepalived/sbin/keepalived /usr/sbin/
[root@init.d]service keepalived start
Starting keepalived:                    [  OK  ]
```

## 相关配置

### Nginx配置

打开nginx配置文件`vim /usr/local/nginx/conf/nginx.conf`,编辑upstream模块及server中的location模块，端口为8888。其他的为默认配置,相关配置如下图

![配置](https://s21.ax1x.com/2024/07/01/pkcq1nU.png)


### Keepalived配置

**MASTER配置**

打开keepalived配置文件`vim /etc/keepalived/keepalived.conf`编辑vrrp_script等模块，相关配置如下图

![配置](https://s21.ax1x.com/2024/07/01/pkcqwjK.png)

**BACKUP配置**

相较于master配置，backup配置需要更改的地方有router_id,vrrp_instance VI_1模块下的state改为BACKUP，mcast_script_ip修改为本机的ip，priority修改为50，其他的与master一致，最终效果如下图

![配置](https://s21.ax1x.com/2024/07/01/pkcqWgP.png)


### Nginx检测脚本
```shell
vim /etc/keepalived/nginx_check.sh

#!/bin/bash
run=`ps -C nginx --no-header | wc -l`
if [ $run -eq 0 ]
then
/usr/local/nginx/sbin/nginx -s stop
/usr/local/nginx/sbin/nginx
sleep 3
if [ `ps -C nginx --no-header |wc -l` ]
then
killall keepalived
fi
fi
```

编辑完成后对脚本文件赋予权限`chmod +x nginx_check.sh`,一定需要赋予权限，否则无法执行此脚本导致keepalived开启服务的时候无法生成vip

## 测试

在master服务器上启动keepalived，`service keepalived start`通过`ip addr`查看成功开启虚拟ip192.168.32.254，此时访问192.168.32.254:8888可以正常访问，然后关掉master上的keepalived，模拟宕机，查看backup上的vip，虚拟ip已经由MASTER漂移到了BACKUP上，此时继续访问192.168.32.254:8888，依然可以访问，测试成功。

## 安装过程中可能遇到的问题及解决办法

问题一：安装nginx时报错configure: error: You need a C++ compiler for C++ support.[系统缺少c++环境]
可能原因：系统没有安装环境
解决办法:
情况1：当您的服务器能链接网络时候[联网安装gcc c++]
```shell
[root@localhost]# yum install -y gcc gcc-c++
```
情况2：当您的服务器不能链接网络时候[不联网/离线安装gcc c++],找到相关的安装包.我这里是挂载的系统安装盘.系统安装盘里面有相关的安装包,如果你没有安装盘在网上下载一下包也可以
```shell
#挂载系统盘
[root@localhost]# mkdir -p /mnt/ROM
[root@localhost]# mount /dev/cdrom /mnt/ROM
#切换到系统安装盘的Packages目录/mnt/ROM/Packages
[root@localhost]#cd /mnt/ROM/Packages
#查看gcc相关安装包
[root@localhost Packages]# ls gcc*
gcc-4.8.2-16.el7.x86_64.rpm  gcc-gfortran-4.8.2-16.el7.x86_64.rpm
gcc-go-4.8.2-16.el7.x86_64.rpm  gcc-objc++-4.8.2-16.el7.x86_64.rpm
gcc-c++-4.8.2-16.el7.x86_64.rpm  gcc-gnat-4.8.2-16.el7.x86_64.rpm
gcc-objc-4.8.2-16.el7.x86_64.rpm  gcc-plugin-devel-4.8.2-16.el7.x86_64.rpm
#安装gcc-c++即c++ compiler
[root@localhost Packages]# rpm -ivh gcc-c++-4.8.2-16.el7.x86_64.rpm 
warning: gcc-c++-4.8.2-16.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 	f4a80eb5: NOKEY
error: Failed dependencies:
​	libstdc++-devel = 4.8.2-16.el7 is needed by gcc-c++-4.8.2-16.el7.x86_64
#安装失败,提示需要安装依赖包libstdc++-devel = 4.8.2-16.el7[版本号与您安装时候安装包相关],我们进行依赖包的查看及安装
[root@localhost Packages]# ll libstdc++-devel*
-rw-rw-r--. 1 500 502 1556984 Jul  3  2014 libstdc++-devel-4.8.2-16.el7.i686.rpm
-rw-rw-r--. 1 500 502 1561232 Jul  3  2014 libstdc++-devel-4.8.2-16.el7.x86_64.rpm
#安装依赖包libstdc++-devel-4.8.2-16.el7.x86_64.rpm [版本号与您安装时候安装包相关]
[root@localhost Packages]# rpm -ivh libstdc++-devel-4.8.2-16.el7.x86_64.rpm
warning: libstdc++-devel-4.8.2-16.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, 	key ID f4a80eb5: NOKEY
Preparing...            ################################# [100%]
Updating / installing...
1:libstdc++-devel-4.8.2-16.el7   ################################# [100%]
#再次安装gcc-c++
[root@localhost Packages]# rpm -ivh gcc-c++-4.8.2-16.el7.x86_64.rpm 
warning: gcc-c++-4.8.2-16.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 	f4a80eb5: NOKEY
Preparing...            ################################# [100%]
Updating / installing...
  1:gcc-c++-4.8.2-16.el7      ################################# [100%]
[root@localhost Packages]#
#自此gcc-c++运行环境安装完成..
```
参考链接：[configure: error: You need a C++ compiler for C++ support.系统缺少c++环境](http://www.cnblogs.com/visec479/p/5315570.html)

问题二：安装keepalived时也会报configure: error: Popt libraries is required相关错误
可能原因：系统缺少安装环境
解决办法：参考问题一
报错参考：Pkgconfig(libpcre) Download (RPM)https://pkgs.org/download/pkgconfig(libpcre)

问题三：除本机外其他客户端无法访问ip
可能原因：防火墙开启或端口未开放
解决办法：关闭防火墙`systemctl stop firewalld`或者开放端口,添加`firewall-cmd --zone=public --add-port=8888/tcp --permanent`（--permanent永久生效，没有此参数重启后失效）重新载入`firewall-cmd --reload`查看`firewall-cmd --zone=public --query-port=8888/tcp`删除`firewall-cmd --zone=public --remove-port=8888/tcp --permanent`

问题四：开启keepalived后主备服务器都有虚拟ip
可能原因：
1. 没有关闭selinux
```shell
#临时关闭selinux
[root@localhost ~]# getenforce
Enforcing
[root@localhost ~]# setenforce 0
[root@localhost ~]# getenforce
Permissive
#永久关闭selinux：
[root@localhost ~]# vim /etc/sysconfig/selinux
SELINUX=enforcing改为SELINUX=disabled
#重启服务reboot
```
2. VRRP协议问题
如果将SElinux关闭问题依旧存在，则可能是防火墙将MASTER的VRRP组播给挡住了。首先将防火墙关闭，确定防火墙是否为罪魁祸首。如果关闭防火墙后问题能够解决，我们只需要让VRRP组播其通过防火墙即可。主备都运行下面的命令
```shell
firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0  --protocol vrrp -j ACCEPT
firewall-cmd --reload
```
参考连接[Keepalived时主备负载均衡器都有VIP的问题：VRRP协议问题](https://lyl-zsu.iteye.com/blog/2408296) 

问题五：挂载系统时报错mount: no medium found on /dev/sr0
出现错误：没有介质在/dev/sr0找到
可能原因：未正确加载虚拟机，iso镜像并没有加载到虚拟机系统内
解决办法：在虚拟机的CD/DVD设置里，将“已连接”和“打开电源时连接”两个选项，选中，确定即可。如下图所示

问题六：虚拟ip无法ping通，无法访问
可能原因：arp协议问题或vip网段设置不一致
解决办法：设置网段一致的vip

问题七：keepalived服务加入开机启动，但是开机自启动失败
可能原因：nginx没有实现开机自启动
由于keepalived开启服务的时候需要执行nginx检测脚本，nginx没有随着系统开机自启动，导致keepalived服务自启动失败
解决办法：将nginx做成系统服务实现开机自启动，或者在/etc/rc.d/rc.local中加入启动命令/usr/local/nginx/sbin/nginx
ps：/etc/rc.d/rc.local是随着系统开机执行命令或脚本，但是系统默认此文件没有执行权限，需要chmod +x /etc/rc.d/rc.local赋予权限，注释上也有

## 相关文章

| [Nginx挂了怎么办？怎么实现高可用？](https://mp.weixin.qq.com/s/lx0dT7bqd3mcIJCS83RdVg) | [Nginx+keepalived实现高可用+防盗链+动静分离，写得太好了！](https://mp.weixin.qq.com/s/Re1CfTaw-NM0IsymRv2JwA) |      |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :--: |