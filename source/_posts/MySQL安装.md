---
title: MySQL安装
categories: 数据库
tags: 安装
index_img: /assert/mysql.jpg
img: https://img0.baidu.com/it/u=730256309,1700139135&fm=253&fmt=auto&app=138&f=JPEG?w=899&h=500

---

## Linux安装MySQL

### 压缩包安装

#### 1. 下载

~~[MySQL下载地址](https://downloads.mysql.com/archives/community/)~~,[MySQL下载地址](https://dev.mysql.com/downloads/)下载后将安装包上传usr/local

#### 2. 解压

```shell
tar -zxvf mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz
```
> 解压后位置usr/local/mysql

#### 3. 创建用户及用户组

先检查是否有mysql用户组和mysql用户,没有就添加
```shell
# 查看所有用户
cat /etc/passwd
# 查看用户组
groups mysql
# 添加用户组
groupadd mysql
# 删除用户组 groupdel
```
添加用户mysql然后将mysql用户添加到到用户组mysql
```shell
# -r建立系统帐号 -g<群组>指定用户所属的群组。
useradd -r -g mysql mysql
# userdel删除
```
#### 4. 操作

修改配置文件vim /etc/my.cnf（没有就新建）
```shell
[client]
default_character_set=utf8mb4

[mysql]
default_character_set=utf8mb4

[mysqld]
# *	接收所有的IPv4或IPv6连接请求
# 0.0.0.0接受所有的IPv4地址
# ::接受所有的IPv4或IPv6地址
# IPv4-mapped接受所有的IPv4地址或IPv4邦定格式的地址（例 ::ffff:127.0.0.1）
# IPv4（IPv6）只接受对应的IPv4（IPv6）地址
bind-address=0.0.0.0
# port=3306
# 指定安装用户
user=mysql
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
log-error=/usr/local/mysql/mysql.err
pid-file=/usr/local/mysql/data/mysql/mysql.pid
character_set_server=utf8mb4
# 符号连接,如果设置为1,则mysql数据库和表里的数据支持储存在datadir目录之外的路径下,默认都是0
symbolic-links=0
# 默认情况下，timestamp类型字段所在数据行被更新时，该字段会自动更新为当前时间，而参数explicit_defaults_for_timestamp控制这一种行为。
# 数据行更新时，timestamp类型字段更新为当前时间
# explicit_defaults_for_timestamp=off
# 数据行更新时，timestamp类型字段不更新为当前时间。
# explicit_defaults_for_timestamp=on
# 允许最大连接数
max_connections=200
default-storage-engine=INNODB
# 连接层默认字符集
collation-server=utf8mb4_unicode_ci
wait_timeout=31536000
interactive_timeout=31536000
```

进入安装mysql软件目录`cd mysql/`,修改目录拥有者为mysql用户`chown -R mysql:mysql ./`

##### 5.6版本

```shell
# --defaults-file指定配置文件 --basedir指定Mysql安装目录 --datadir指定数据目录 --user所属用户
./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```
> 此处可能出现错误
>
> **错误1**：命令报错`-bash: ./scripts/mysql_install_db: /usr/bin/perl: bad interpreter: No such file or directory.`貌似提示注释器错误，没有/usr/bin/perl文件或者档案
> **解决办法**:（安装perl跟perl-devel即可）：执行**yum -y install perl perl-devel**后在初始化数据库即可。
>
> **错误2**：`FATAL ERROR: please install the following Perl modules before executing * Data::Dumper`
> **解决办法:yum install -y perl-Data-Dumper**
>
> 修改当前目录拥有者为root用户
> ```shell
> chown -R root:root ./
> ```
> 再次运行
> ```shell
> ./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
> ```
> 修改当前data目录拥有者为mysql用户
> ```shell
> chown -R mysql:mysql data
> ```

##### 5.7版本

执行命令
```shell
./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/ --user=mysql --initialize
```
查看初始密码,控制台查看或者`cat /usr/local/mysql/mysql.err`localhost:后面的一串字符就是初始密码

##### 8.0版本

```shell
bin/mysqld --user=mysql --initialize --datadir=/usr/local/mysql/data
```
root@localhost之后的一串字符就是初始密码

> **错误1**：服务未成功启动查看data目录下的localhost.localdomain.err错误日志
> 遇到`Can't start server : Bind on unix socket: Address already in use Do you already have another mysqld server running on socket: /tmp/mysql.sock ?`解决方法是删除 rm -rf /tmp/mysql.sock(由于以前安装过mysql的原因)
>
> **错误2**：./mysqld: error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory就执行下`yum install -y libaio`、` yum -y install numactl`后再执行初始化

#### 5. 启动mysql

添加开机启动，把启动脚本放到开机初始化目录。

```shell
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
```
赋予可执行权限
```shell
chmod +x /etc/init.d/mysql
```
启动mysql服务
```shell
service mysql start
```

#### 6. 修改密码
```shell
# 测试进入mysql
mysql -uroot -proot
# UPDATE直接编辑user表
update mysql.user set password=password('新密码') where host='localhost' and user='root';
# 或者使用SET PASSWORD命令
set password for root@localhost = password('新密码');
# 或者使用mysqladmin修改密码
./bin/mysqladmin -hlocalhost.localdomain -uroot -p旧密码 password '新密码'
# MySQL8使用
update user set authentication_string='' where user='root';# 将字段置为空
ALTER user 'root'@'localhost' IDENTIFIED BY 'root';#修改密码为root
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
# 进入mysql更改密码
set password='root';
# 使密码生效
flush privileges;
```

#### 7. 修改远程连接

改表法
```shell
use mysql
update user set host='%' where user='root';
flush privileges;
```
授权法
```shell
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '密码' WITH GRANT OPTION;
# 如果你想允许用户myuser从ip为192.168.1.6的主机连接到mysql服务器，并使用mypassword作为密码
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
```

#### 8. 其他
添加服务自启动
```shell
chkconfig --add mysql
```
显示服务列表
```shell
chkconfig --list
# 如果看到mysql的服务，并且3,4,5都是on的话则成功，如果是off，则执行
chkconfig --level 345 mysql on
```

> 假如服务未成功启动[log-error set to '/var/log/mariadb/mariadb.log', however file don't exists](http://blog.csdn.net/duyuanhai/article/details/78604894)**报错的情况下执行以下操作**:需要创建报错中缺失的文件夹`mkdir z/var/log/mariadb，touch /var/log/mariadb/mariadb.log`

创建软连接
```shell
ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql
```
mysql -uroot -proot无法连接的话

```shell
ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock
```

默认配置文件路径如下
```
配置文件：/etc/my.cnf
日志文件：/var/log/var/log/mysqld.log
服务启动脚本：/usr/lib/systemd/system/mysqld.service
socket文件：/var/run/mysqld/mysqld.pid
```
### yum安装

#### 一、安装本地YUM源、MySQL在MySQL官网中下载YUM源rpm安装包

1. 把上面的rpm文件下载下来放到服务器上,或者在linux系统中通过wget命令下载

```shell
wget http://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
```
2. 下载完成后使用yum命令本地安装yum源
```shell
yum localinstall mysql80-community-release-el7-1.noarch.rpm
```
3. 执行完毕后使用下面的命令检查是否安装成功
```shell
yum repolist enabled | grep "mysql.*-community.*"
```
4. 安装服务器
```shell
yum install -y mysql-community-server
```
#### 二、配置MySQL

服务命令
```shell
# 启动MySQL服务
systemctl start mysqld
# 查看服务启动状态
systemctl status mysqld
# 开机启动
systemctl enable mysqld
# 重新加载开机启动配置
systemctl daemon-reload

# 修改root默认密码
# 查询默认密码
cat /var/log/mysqld.log | grep 'temporary password'
# 登录mysql,用刚才从文件中找到的密码
mysql -uroot -p
# 尝试修改密码,使用下面的命令修改root用户的密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';

# 添加远程登陆
# 创建一个能全局访问的用户root
# CREATE USER 'root'@'%' IDENTIFIED BY 你的密码'';
# 给用户授权任何远程主机都可以访问数据库 
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
# 输入刷新命令使修改生效
FLUSH PRIVILEGES;
# 修改密码的加密方式
# 找到mysql的配置文件vim /etc/my.cnf，把密码的加密方式改成之前版本的,8.0版本更换了密码的加密方式,我们就先用旧的
# 找到default-authentication-plugin，将其注释取消
default-authentication-plugin=mysql_native_password
# 修改完my.cnf后重启服务，使其生效
systemctl restart mysqld

# sql_mode=only_full_group_by问题
# 查看sql_mode
select @@global.sql_mode;
# 查询出来的值为：
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION

# 修改my.cnf，在[mysqld]栏下新增sql_mode，将ONLY_FULL_GROUP_BY去掉
vim /etc/my.cnf
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
# 重启服务
systemctl restart mysqld
```

### rpm安装

1. 需要通过rpm相关指令，来查询当前系统中是否存在已安装的mysql软件包
```shell
# 查询当前系统中安装的所有软件
rpm -qa
# 查询当前系统中安装的名称带mysql的软件
rpm -qa | grep mysql
# 查询当前系统中安装的名称带mariadb的软件
rpm -qa | grep mariadb
```
2. 卸载
```shell
rpm -ev --nodeps 软件名称
# 卸载之前请先关闭mysql服务，命令如下
systemctl stop mysqld
# 依次卸载相关服务
rpm -e --nodeps mysql-community-server
rpm -e --nodeps mysql-community-client
rpm -e --nodeps mysql-community-libs
rpm -e --nodeps mysql-community-common
#卸载mariadb
rpm -e --nodeps mariadb-libs-5.5.64-1.el7.x86_64
# 删除数据库配置文件
rm -rf /etc/my.cnf
# 删除数据库数据文件
rm -rf /var/lib/mysql
# 删除日志临时文件
rm -rf /var/log/mysqld.log
```
3. 下载rpm
![下载](https://img-blog.csdnimg.cn/img_convert/7ec61ebdc9103cd70f3dd52efbd4ea19.png)
或者使用wget
```shell
# 下载server包
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-server-5.7.38-1.el7.x86_64.rpm
# 下载client包
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-client-5.7.38-1.el7.x86_64.rpm
# 下载common包
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-common-5.7.38-1.el7.x86_64.rpm
# 下载libs包
wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-libs-5.7.38-1.el7.x86_64.rpm
```
4. 安装
> 包之间相互依赖，所以必须注意安装顺序
> ```shell
> # 相关依赖
> yum install libaio -y
> yum install net-tools -y
> ```

先装common,再装libs（确保mariadb已卸载，centos7默认支持mariadb，不支持mysql，不卸载会出现冲突）,再装client,最后装server
```shell
# i表示install安装；v表示verbose进度条；h表示hash哈希校验。
rpm -ivh mysql-community-common-8.0.11-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.11-1.el7.x86_64.rpm
# rpm -ivh mysql-community-devel-5.7.25-1.el7.x86_64.rpm
# rpm -ivh mysql-community-libs-compat-5.7.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.11-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.11-1.el7.x86_64.rpm
```
5. 查看密码

对于rpm安装的mysql，在mysql第一次启动时，会自动帮我们生成root用户的访问密码，并且输出在mysql的日志文件/var/log/mysqld.log中，我们可以查看这份日志文件，从而获取到访问密码
```shell
grep 'temporary password' /var/log/mysqld.log
# 或者
cat /var/log/mysqld.log | grep password
```
6. 配置其他内容如编码则新建`/etc/my.cnf`


## windows安装MySQL

### 卸载
```
关闭mysql服务:net stop mysql
删除mysql服务:mysqld -remove
删除mysql相关文件夹，如想保留数据可保留data文件
```
### 安装

1. 解压mysql压缩包后进入mysql目录，新建my.ini文件并编辑

```shell
[mysqld]
# 设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=D:\mysql\mysql-5.8.11-winx64
# 设置mysql数据库的数据的存放目录
datadir=D:\mysql\mysql-5.8.11-winx64\data
# 允许最大连接数
max_connections=200
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```
2. 将mysql加入环境变量

3. 管理员打开cmd切换到mysql bin目录下(一定要切换),执行**mysqld install**安装mysql服务，成功打印**service successfully installed**后执行**mysqld --initialize --console**会对数据库做初始化并打印相应日志：可以找到root@localhost之后的一串字符就是初始密码，然后执行net start mysql开启服务

4. 修改密码（5.7版本以前）
```shell
# mysql -uroot -p输入密码后
set password for root@localhost = password('123456');
# 8.0修改密码
set password = '123456';
```

### 8.0连接navicat

navicat连接8.0需要把mysql用户登录密码加密规则还原成mysql_native_password
```shell
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root' PASSWORD EXPIRE NEVER; #修改加密规则
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root'; #更新一下用户的密码
FLUSH PRIVILEGES; #刷新权限
# 然后use mysql;
update user set host = '%' where user = 'root';
FLUSH PRIVILEGES;
```

## 相关文章
- [一文教你在CentOS7下安装MySQL及搭建主从复制](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491190&idx=1&sn=6e6ed61a51e2f214d19e304038bde8b4&source=41#wechat_redirect)
- [手把手教大家搭建MySQL主从复制](https://mp.weixin.qq.com/s/R89aCCFvCvudLp6FUn2JjQ)