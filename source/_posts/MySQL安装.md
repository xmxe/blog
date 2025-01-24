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
#cat /etc/passwd
# 查看用户组
#groups mysql
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

**进入安装mysql软件目录`cd mysql/`,修改目录拥有者为mysql用户`chown -R mysql:mysql ./`，或者单独修改MySQL的data目录`chown -R mysql:mysql data/`**

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
./mysqld --defaults-file=/etc/my.cnf --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql/ --initialize --user=mysql
```
root@localhost之后的一串字符就是初始密码

> **错误1**：服务未成功启动查看data目录下的localhost.localdomain.err错误日志
> 遇到`Can't start server : Bind on unix socket: Address already in use Do you already have another mysqld server running on socket: /tmp/mysql.sock ?`解决方法是删除 rm -rf /tmp/mysql.sock(由于以前安装过mysql的原因)
>
> **错误2**：./mysqld: error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory就执行下`yum install -y libaio`、` yum -y install numactl`后再执行初始化
> 
> **错误3**：mysql -uroot -proot无法连接的话,报错Can't connect to local MySQL server through socket '/tmp/mysql.sock
> 创建软连接`ln -s /data/mysql/mysql.sock /tmp/mysql.sock`,注意查看/etc/my.cnf中socket配置的mysql.sock的位置,
> mysql.sock默认位置是`ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock`

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

> 延申
> 如果想要将`service mysql start`启动替换成`systemctl start mysql`的话,步骤如下

1. 创建mysql.service `vim /etc/systemd/system/mysql.service`

```makefile
[Unit]
Description=MySQL Community Server
After=network.target

[Service]
Type=forking
# Set the path to the MySQL binary
ExecStart=/usr/local/mysql/support-files/mysql.server start
ExecStop=/usr/local/mysql/support-files/mysql.server stop
ExecReload=/usr/local/mysql/support-files/mysql.server reload
# Set user and group to run the service as
User=mysql
Group=mysql

[Install]
WantedBy=multi-user.target
```

2. 执行

```shell
chmod +x /etc/systemd/system/docker.service
systemctl daemon-reload
systemctl start mysql
```

#### 6. 修改密码

```shell
# 测试进入mysql(注意端口为默认的3306,如果修改过默认端口,使用-P来指定端口)
mysql -uroot -proot

# UPDATE直接编辑user表,旧版本中密码字段是password,而不是authentication_string.注意加上user和host的查询条件,因为有可能host不一样,比如user表里查询user='root'有两条记录,一个host为'localhost',一个为'%',意味着相同用户本地登录是一个密码，远程登陆是一个密码
#update mysql.user set password=password('新密码') where host='localhost' and user='root';
update mysql.user set authentication_string=password('password') where user='root' and host='hostname';

# 使用SET PASSWORD命令修改当前用户密码 //xx
set password=password("new-password");
# 使用SET PASSWORD命令修改其他用户密码
set password for 'user'@'hostname' = password('新密码');

# 使用mysqladmin修改密码
./bin/mysqladmin -hlocalhost.localdomain -uroot -p旧密码 password '新密码'

# 修改当前用户密码(USER()为获取当前连接用户的函数)
ALTER user USER() IDENTIFIED BY 'root';
# 修改其他用户的密码,使用user表中的user,host匹配,如:'root'@'%',mysql8默认WITH mysql_native_password
ALTER user 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';

# 使密码生效
flush privileges;
```

#### 7. 修改远程连接

改表法
```sql
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

#### 8.创建用户

```sql
-- 创建用户
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
-- 授权
GRANT ALL PRIVILEGES ON my_database.* TO 'username'@'host';
-- 只授予部分权限
GRANT SELECT, INSERT ON my_database.* TO 'username'@'host';
-- 刷新权限
flush privileges;
```

#### 9. 其他

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

默认配置文件路径如下

```
配置文件：/etc/my.cnf
日志文件：/var/log/var/log/mysqld.log
服务启动脚本：/usr/lib/systemd/system/mysqld.service
socket文件：/var/run/mysqld/mysqld.pid
```

#### 10. 注册systemd(查看第5步延申)

如果想要使用`systemctl`替代`service`命令，需要注册`mysql.service`服务，具体如下


```shell
1. vim /etc/systemd/system/myssql.service

[Unit]
Description=MySQL Server
After=network.target
After=syslog.target

[Service]
User=mysql
Group=mysql
ExecStart=/usr/sbin/mysqld --defaults-file=/etc/my.cnf
# ExecStart=service mysql start
PIDFile=/var/run/mysqld/mysqld.pid
LimitNOFILE = 5000

[Install]
WantedBy=multi-user.target

2. chmod +x /etc/systemd/system/mysql.service
3. systemctl daemon-reload
4. systemctl start mysql
```

### yum安装

#### 一、安装本地YUM源、MySQL在MySQL官网中下载YUM源rpm安装包

(1) 把上面的rpm文件下载下来放到服务器上,或者在linux系统中通过wget命令下载

```shell
wget http://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
```

(2) 下载完成后使用yum命令本地安装yum源

```shell
yum localinstall mysql80-community-release-el7-1.noarch.rpm
```

(3) 执行完毕后使用下面的命令检查是否安装成功

```shell
yum repolist enabled | grep "mysql.*-community.*"
```

(4) 安装服务器

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

(1) 需要通过rpm相关指令，来查询当前系统中是否存在已安装的mysql软件包

```shell
# 查询当前系统中安装的所有软件
rpm -qa
# 查询当前系统中安装的名称带mysql的软件
rpm -qa | grep mysql
# 查询当前系统中安装的名称带mariadb的软件
rpm -qa | grep mariadb
```

(2) 卸载

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

(3) 下载rpm

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

(4) 安装

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

(5) 查看密码

对于rpm安装的mysql，在mysql第一次启动时，会自动帮我们生成root用户的访问密码，并且输出在mysql的日志文件/var/log/mysqld.log中，我们可以查看这份日志文件，从而获取到访问密码
```shell
grep 'temporary password' /var/log/mysqld.log
# 或者
cat /var/log/mysqld.log | grep password
```

(6) 配置其他内容如编码则新建`/etc/my.cnf`


## Windows安装MySQL

### 卸载
```
关闭mysql服务:net stop mysql
删除mysql服务:mysqld -remove
删除mysql相关文件夹，如想保留数据可保留data文件
```
### 安装

(1) 解压mysql压缩包后进入mysql目录，新建my.ini文件并编辑

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

(2) 将mysql加入环境变量

(3) 管理员打开cmd切换到mysql bin目录下(一定要切换),执行**mysqld install**安装mysql服务，成功打印**service successfully installed**后执行**mysqld --initialize --console**会对数据库做初始化并打印相应日志：可以找到root@localhost之后的一串字符就是初始密码，然后执行net start mysql开启服务

> 或者配置root用户不设置密码: **mysqld --initialize-insecure --console**。此命令是一个用于初始化MySQL服务器的命令。该命令的作用是在指定的目录中创建MySQL数据文件，并生成一个没有密码的root用户账号，以及默认的匿名用户账号和测试数据库。以下是该命令的各个选项的具体含义：
>> mysqld: MySQL的服务器程序，用于处理MySQL的请求。
>> --initialize-insecure: 初始化MySQL服务器时不设置任何安全措施，包括不设置root用户的密码。
>> --console: 将初始化过程的输出重定向到标准输出设备（如终端窗口），以便可以查看执行结果。

(4) 修改密码（5.7版本以前）

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
ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码'
FLUSH PRIVILEGES;
```

## 批量迁移.ibd文件脚本(AI生成)

### ChatGPT 3.5(已测试)

```bash
#!/bin/bash
# 定义目标数据库信息
database="your_database"
tablespace_directory="/path/to/tablespace_directory"

# 遍历目录中的所有IDB文件
for file in $(find "$tablespace_directory" -type f -name "*.ibd")
do
    # 提取表名
    tablename=$(basename "$file" .ibd)
    
    # 执行导入表空间操作
    # 注意:`ALTER TABLE $tablename DISCARD TABLESPACE`将删除与该表关联的.ibd文件。所以如果手动将原.idb删除后将目标的.ibd文件复制到mysql的data目录后不要执行下面的命令,因为先删掉.ibd后`ALTER TABLE $tablename IMPORT TABLESPACE`就无法执行了,当拷贝的.ibd文件不在data目录下而在其他目录时可以使用下面命令
    # mysql -uroot -proot -e "USE $database; ALTER TABLE $tablename DISCARD TABLESPACE; ALTER TABLE $tablename IMPORT TABLESPACE;"
    
    mysql -uroot -proot -e "USE $database; ALTER TABLE $tablename IMPORT TABLESPACE;"
    
    if [ $? -eq 0 ]; then
        echo "成功导入表 $tablename 的表空间"
    else
        echo "导入表 $tablename 的表空间时出现错误"
    fi
done
```
> 问题1: 如果遇到报错试下授予文件读写权限: **chmod -R 777 mysql/data/databasename/**
> 问题2: 执行脚本时报错: **Can’t connect to local MySQL server through socket ‘/tmp/mysql.sock**。
> 解决方法为更改/etc/my.cnf文件**socket = /home/mysql/mysql.sock**后创建软连接**ln -s /home/mysql/mysql.sock /tmp/mysql.sock**。归结到底还是文件目录读写权限问题，一劳永逸方法**chmod -R 777 mysql安装目录**

### 文心一言

```bash
#!/bin/bash  
  
# 目标数据库连接信息  
DB_USER="your_username"  
DB_PASSWORD="your_password"  
DB_NAME="your_database"  
  
# IDB文件目录路径  
IDB_DIR="/path/to/idb/files"  
  
# 遍历IDB文件目录  
for file in $IDB_DIR/*.idb; do  
    # 提取表名  
    table_name=$(basename "$file" .idb)
    # 导入表空间  
    mysql -u $DB_USER -p$DB_PASSWORD -e "ALTER TABLE $table_name IMPORT TABLESPACE $file;" $DB_NAME  
done
```

### 通义千问

```bash
#!/bin/bash

# 指定要导入的目录
IMPORT_DIR="/path/to/import/directory"

# 指定目标数据库名
DATABASE_NAME="your_database_name"

# 获取目录中所有.ibd文件的列表
IBD_FILES=$(find "$IMPORT_DIR" -name "*.ibd")

# 遍历所有.ibd文件
for IBDFILE in $IBD_FILES; do
    # 提取表名
    TABLE_NAME=$(basename "$IBDFILE" .ibd)
    # 执行ALTER TABLE IMPORT TABLESPACE语句
    mysql -uroot -proot $DATABASE_NAME -e "ALTER TABLE \`$TABLE_NAME\` IMPORT TABLESPACE;"
done

echo "Import completed."
```

## 主从同步

**MySQL5.6数据库主从（Master/Slave）同步安装与配置详解,两台服务器-Master主:192.168.159.28、Slave从:192.168.159.30**

### Master的配置
在Linux环境下MySQL的配置文件的位置是在/etc/my.cnf,在该文件下指定Master的配置如下：
```ini
[mysqld]
log-bin=mysql-bin
# server-id用于标识唯一的数据库，这里设置为2，在设置从库的时候就需要设置为其他值。
server-id=2
# binlog-ignore-db：表示同步的时候ignore的数据库 
binlog-ignore-db=information_schema
binlog-ignore-db=cluster
binlog-ignore-db=mysql
# binlog-do-db：指定需要同步的数据库
binlog-do-db=ufind_db
```
完整配置如下：
```ini
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent symbolic-links=0
# Settings user and group are ignored when systemd is If you need to run mysqld under a different user or customize your systemd unit file for mariadb accord instructions in http://fedoraproject.org/wiki/Syste
log-bin=mysql-bin
server-id=2
binlog-ignore-db=information_schema
binlog-ignore-db=cluster
binlog-ignore-db=mysql
binlog-do-db=ufind_db
wait_timeout=31536000
interactive_timeout=31536000
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

[client]
default-character-set=utf8mb4

[mysgl]
default-character-set=utf8mb4

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid
#include all files from the config directory
!includedir /etc/my.cnf.d
```

然后重启mysql：`service mysqld restart`。进入mysql：`./bin/mysql –h127.0.0.1 –uroot -proot`赋予从库权限帐号，允许用户在主库上读取日志，赋予192.168.159.130也就是Slave机器有File权限，只赋予Slave机器有File权限还不行，还要给它REPLICATION SLAVE的权限才可以。命令如下：
```shell
GRANT FILE ON *.* TO 'root'@'192.168.159.130' IDENTIFIED BY 'root';
GRANT  REPLICATION SLAVE ON *.* TO 'root'@'192.168.159.130' IDENTIFIED BY 'root';
FLUSH  PRIVILEGES
```
这里使用的仍是root用户作为同步的时候使用到的用户，可以自己设定。重启mysql，登录mysql，显示主库信息`show master status;`这里的File、Position是在配置Salve的时候要使用到的，Binlog_Do_DB表示要同步的数据库，Binlog_Ignore_DB表示Ignore的数据库，这些都是在配置的时候进行指定的。另外：如果执行这个步骤始终为Empty set(0.00 sec)，那说明前面的my.cnf没配置对。

### Slave的配置
从库的配置，首先也是修改配置文件：/etc/my.cnf如下：
```ini
log-bin=mysql-bin
server-id=3
binlog-ignore-db=information_schema
binlog-ignore-db=cluster
binlog-ignore-db=mysql
replicate-do-db=ufind_db
replicate-ignore-db=mysql
log-slave-updates
slave-skip-errors=all
slave-net-timeout=60
```
完整配置如下
```ini
[mysqld]

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks symbolic-links=0

# Settings user and group are ignored when systemd is used. # If you need to run mysgld under a different user or group, customize your systemd unit file for mariadb according to the instructions in http://fedoraproject.org/wiki/Systemd

log-bin=mysql-bin
server-id=3

binlog-ignore-db=information schema
binlog-ignore-db=cluster
binlog-ignore-db=mysgl
replicate-do-db=uf ind db
replicate-ignore-db=mysql
log-slave-updates

slave-skip-errors=all
slave-net-timeout=60

[mysqld_safe]

log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

# include all files from the config directory
!includedir /etc/my.cnf.d
```

修改完/etc/my.cnf文件之后，重启一下MySQL`service mysqld restart`,进入Slave mysql控制台，执行：
```shell
stop slave;  #关闭Slave
change master to master_host='192.168.159.128',master_user='root',master_password='root',master_log_file='mysql-bin.000001', master_log_pos=610;

start slave; #开启Slave
```
在这里指定Master的信息，master_log_file是在配置Master的时候的File选项，master_log_pos是在配置Master的Position选项，这里要进行对应。然后可以通过`show slave status;`查看配置的信息：完成！

| [MySQL5.6数据库主从（Master/Slave）同步安装与配置详解](https://blog.csdn.net/xlgen157387/article/details/51331244/) | [说说MySQL主从复制原理？](https://mp.weixin.qq.com/s/CNu4uVZrpAkDt5UCIlCBSA) | [如何处理MySQL主从延迟？](https://mp.weixin.qq.com/s/YKhnF1g9BB3JhJq7fjHO-Q) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |


---

一、实验目标
搭建两台MySQL服务器，一台作为主服务器，一台作为从服务器，主服务器进行写操作，从服务器进行读操作。

二、测试环境
主数据库： CentOS7， MySQL15.1 ， 192.168.1.233
从数据库： CentOS7， MySQL15.1 ， 192.168.1.234

三、主从配置步骤

1. 确保主数据库与从数据库里的数据一样
例如：主数据库里的a的数据库里有b，c，d表，那从数据库里的就应该有一个模子刻出来的a的数据库和b，c，d表
我这里在两台MySQL上都创建了个名为“test”的数据库来测试

2. 在主数据库里创建一个同步账号

1）每个从数据库会使用一个MySQL账号来连接主数据库，所以我们要在主数据库里创建一个账号，并且该账号要授予 REPLICATION SLAVE权限，你可以为每个从数据库分别创建账号，当然也可以用同一个！）

2）你可以用原来的账号不一定要新创账号，但你应该注意，这个账号和密码会被明文存放在master.info文件中，因此建议单独创一个只拥有相关权限的账号，以减少对其它账号的危害！）

3）创建新账号使用“CREATE USER”，给账号授权使用“GRANT”命令，如果你仅仅为了主从复制创建账号，只需要授予REPLICATION SLAVE权限。

4）下面来创建一个账号，账号名：repl，密码：repl123，只允许192.168.1.的IP段登录，如下：
```sql
CREATE USER 'repl'@'192.168.1.%' IDENTIFIED BY 'repl123';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.1.%';
```
5）如果开发防火墙，可能要配置下端口，如下：
```shell
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

3. 配置主数据库

1）要主数据库，你必须要启用二进制日志（binary logging），并且创建一个唯一的Server ID，这步骤可能要重启MySQL。
2）主服务器发送变更记录到从服务器依赖的是二进制日志，如果没启用二进制日志，复制操作不能实现（主库复制到从库）。
3）复制组中的每台服务器都要配置唯一的Server ID，取值范围是1到(232)−1，你自己决定取值。
4）配置二进制日志和Server ID，你需要关闭MySQL和编辑my.cnf或者my.ini文件，在[mysqld]节点下添加配置。
5）下面是启用二进制日志，日志文件名以“mysql-bin”作为前缀，Server ID配置为1，如下：
```ini
[mysqld]
log-bin=mysql-bin
server-id=1
#网络上还有如下配置
#binlog-do-db=mstest //要同步的mstest数据库,要同步多个数据库，就多加几个replicate-db-db=数据库名
#binlog-ignore-db=mysql //要忽略的数据库
#提示1：如果你不配置server-id或者配置值为0，那么主服务器将拒绝所有从服务器的连接。
#提示2：在使用InnoDB的事务复制，为了尽可能持久和数据一致，你应该在my.cnf里配置innodb_flush_log_at_trx_commit=1和 sync_binlog=1；
#For the greatest possible durability and consistency in a replication setup using InnoDB with transactions, you should useinnodb_flush_log_at_trx_commit=1 and sync_binlog=1 in the master my.cnf file.
#提示3：确保主服务器里的skip-networking选项未启用，如果网络被禁用，你的从服务器将不能与主服务器通信并且复制失败。
```
注意：实际操作发现/etc/my.cnf文件和教材说的不一样，可能我装的是MariaDB,目测文件被放到了/etc/my.cnf.d目录里，在/etc/my.cnf.d/server.cnf增加相关配置
```ini
log-bin=mysql-bin
server-id=1
```

重启MySQL,查看主服务器状态，`show master status;`
4. 配置从数据库

1）从服务器，同理，要分配一个唯一的Server ID，需要关闭MySQL，修改好后再重启，如下：

```ini
[mysqld]
server-id=2
#可以指定要复制的库
replicate-do-db = test #在master端不指定binlog-do-db，在slave端用replication-do-db来过滤
replicate-ignore-db = mysql #忽略的库
#网上还有下面配置
#relay-log=mysqld-relay-bin
#提示1：如果有多个从服务器，每个服务器的server-id不能重复，跟IP一样是唯一标识，如果你没设置server-id或者设置为0，则从服务器不会连接到主服务器。
#提示2：一般你不需要在从服务器上启用二进制日志，如果你在从服务器上启用二进制日志，那你可用它来做数据备份和崩溃恢复，或者做更复杂的事情（比如这个从服务器用来当作其它从服务器的主服务器）。
```
2）配置连接主服务器的信息
```sql
 stop slave;
CHANGE MASTER TO
-> MASTER_HOST='192.168.1.233',
-> MASTER_USER='repl',
-> MASTER_PASSWORD='repl123',
-> MASTER_LOG_FILE='mysql-bin.000002',
-> MASTER_LOG_POS=313;
start slave;
```
3）查看从服务器状态
```sql
show slave status;
```
5. 测试数据同步

测试，连接主服务器192.168.1.233，添加了表stu_user，然后再连接上192.168.1.234，发现也自己同步创建了表stu_user，然后在主数据库添加一条记录，从数据库也自动添加了记录，至此，主从的配置已经完成了， 目前是在从库里面配置复制“test”这个库，
如果要添加其它库，可以在主服务器中添加“binlog-do-db”配置，或者在从服务器中添加“replicate-do-db”配置。

四、读写分离实现
主从配置是读写分离的前提，现在前提已经配置好了，读写分离就看具体项目的实现，读写分离，就是“写”的操作都在主数据库，“读”的操作都在从数据库！
（完）


## MySQL高可用

1. 卸载系统原有的Mysql,查看系统是否已安装Mysql，命令如下：`rpm -qa | grep mysql`,查看系统是否已启动Mysql，命令如下：`ps -ef | grep mysql`.如果已启动，则将相应进程杀死，命令如下：`kill -9 进程号`,卸载已安装的Mysql，命令如下：
`rpm -ev mysql-5.1.71-1.el6.x86_64 mysql-libs-5.1.71-1.el6.x86_64 mysql-devel-5.1.71-1.el6.x86_64 --nodeps`.查看是否残留Mysql安装文件`find / -name mysql`
2. 安装Mysql
```shell
# 创建Mysql安装目录
mkdir /home/mysqlapp
# 创建临时目录
mkdir /home/mysqlapp/tmp
# 创建数据目录
mkdir /home/mysqlapp/data
# 创建用户与用户组，命令如下：
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
# 修改临时目录、数据目录所有者和权限
chown mysql:mysql /home/mysqlapp/data/
chmod 750 /home/mysqlapp/data/
chown mysql:mysql /home/mysqlapp/tmp/
chmod 750 /home/mysqlapp/tmp/
# 上传mysql-8.0.12-linux-glibc2.12-x86_64.tar.xz至/home/mysqlapp目录下，解压
tar xvf mysql-8.0.12-linux-glibc2.12-x86_64.tar.xz
# 创建软连接，命令如下：
ln -s mysql-8.0.12-linux-glibc2.12-x86_64 mysql
```
编辑/etc/profile文件，末尾增加如下两行：
```shell
export MYSQL_HOME=/home/mysqlapp/mysql
export PATH=$PATH:$MYSQL_HOME/bin
```
保存后，执行`source /etc/profile`，使配置立即生效,在/etc目录创建文件my.cnf,文件内容如下：
```ini
[client]
port=3306
socket=/home/mysqlapp/tmp/mysql.sock
[mysqld]
port=3306
socket=/home/mysqlapp/tmp/mysql.sock
basedir=/home/mysqlapp/mysql
datadir=/home/mysqlapp/data
key_buffer_size=16M
max_allowed_packet=128M
character_set_server=UTF8MB4
```
初始化数据目录，命令如下：`mysqld --initialize --user=mysql`,执行后打印如下内容，则表示初始化成功：
```bash
[System] [MY-013169] [Server] mysqld (mysqld 8.0.12) initializing of server in progress as process 23352
[Note] [MY-010454] [Server] A temporary password is generated for root@localhost: qq4>Vrw*oF2+
[System] [MY-013170] [Server] mysqld (mysqld 8.0.12) initializing of server has completed
```
启动Mysql，命令如下：`mysqld_safe --user=mysql &`查看Mysql进程是否存在：`ps -ef | grep mysql`
修改root用户密码（初始化数据目录时，生成了随机的root用户密码，且被标记为已过期，使用随机密码登录后，需要修改root密码）`mysql -u root -p`输入密码进入mysql控制台`ALTER USER 'root'@'localhost' IDENTIFIED BY 'lnsoft';`停止数据库`mysqladmin -u root -p shutdown`

按照以上步骤在hadoop3、4、5上安装Mysql

3. 配置Group Replication,安装group_replication插件：
```
install PLUGIN group_replication SONAME 'group_replication.so';
```
修改/etc/my.cnf配置文件，增加以下配置：
```ini
server_id=1
gtid_mode=ON
enforce_gtid_consistency=ON
binlog_checksum=NONE

log_bin=binlog
log_slave_updates=ON
binlog_format=ROW
master_info_repository=TABLE
relay_log_info_repository=TABLE

transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name="33f5efc0-186f-11e9-9c97-c81fbe536994"
loose-group_replication_start_on_boot=OFF
loose-group_replication_local_address="10.37.169.62:33081"
loose-group_replication_group_seeds="10.37.169.62:33081,10.37.169.63:33081,10.37.169.64:33081"
loose-group_replication_bootstrap_group=OFF
loose-group_replication_single_primary_mode=ON
loose-group_replication_enforce_update_everywhere_checks=OFF
server_id #必须唯一
gtid_mode #必须开启
enforce_gtid_consistency #强制GTID的一致性
binlog_checksum #binlog校验规则，MGR要求使用NONE
binlog_format #binlog格式，MGR要求必须是ROW
master_info_repository #MGR集群要求复制模式要改成slave记录到表中，否则报错
transaction_write_set_extraction #记录事物的算法
group_replication_group_name #Group名称，UUID值
group_replication_start_on_boot #是否随服务器启动而自动启动组复制
group_replication_single_primary_mode #是否单主模式
group_replication_enforce_update_everywhere_checks #是否强制检查每个实例是否允许该操作，多主模式下必须开启
```
4. 启动MGR集群,启动MGR要注意顺序，需要指定其中一台数据库做引导，其它数据库才可以顺利加入进来。如果是单主模式，那么主库一定要先启动并做引导。这里使用hadoop3作为主库。在hadoop3上登录msql服务端
```bash
#启动引导，hadoop4、hadoop5略过此步
mysql> set GLOBAL group_replication_bootstrap_group=ON;
#创建一个用户用来做同步，并授权
mysql> create user 'rpl_user'@'%' identified with 'mysql_native_password' by 'lnsoft';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'rpl_user'@'%' with grant option;
mysql> FLUSH PRIVILEGES;

mysql> show variables like 'SQL_LOG_BIN%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| sql_log_bin   | ON    |
+---------------+-------+
# 如果是OFF，执行
mysql> set SQL_LOG_BIN=1;
#清空所有旧的GTID信息，避免冲突
mysql> reset master;
#创建同步规则认证信息，配置和改变slave服务器用于连接master服务器的参数
mysql> CHANGE MASTER TO MASTER_USER='rpl_user', MASTER_PASSWORD='lnsoft' FOR  CHANNEL 'group_replication_recovery';
#启动MGR
mysql> start group_replication;
#查看是否启动成功，状态为ONLINE即表示启动成功
mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| group_replication_applier | 314051b4-17d1-11e9-8d1f-c81fbe536994 | hadoop3     |        3306 | ONLINE       | PRIMARY     | 8.0.12         |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
# 关闭引导
mysql> set GLOBAL group_replication_bootstrap_group=OFF;
```
hadoop4、hadoop5上执行除了启动引导和关闭引导以外的语句，加入复制组。hadoop4、hadoop5加入成功后，查看复制组成员：

```bash
mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| group_replication_applier | 314051b4-17d1-11e9-8d1f-c81fbe536994 | hadoop3     |        3306 | ONLINE       | PRIMARY     | 8.0.12         |
| group_replication_applier | 7b4faa2d-1862-11e9-bfae-c81fbea4d979 | hadoop4     |        3306 | ONLINE       | SECONDARY   | 8.0.12         |
| group_replication_applier | 7e32e66b-1862-11e9-9cf0-c81fbea4da25 | hadoop5     |        3306 | ONLINE       | SECONDARY   | 8.0.12         |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
```
至此，MGR集群搭建成功。

5. CM Server元数据迁移

5.1 备份hadoop2上原有mysql数据库的数据，命令如下：

```sql
mysqldump -u root -plnsoft amon >amon.sql
mysqldump -u root -plnsoft cm >cm.sql
mysqldump -u root -plnsoft hive >hive.sql
mysqldump -u root -plnsoft hue >hue.sql
mysqldump -u root -plnsoft oozie >oozie.sql
```
5.2 在主节点（hadoop3）上创建数据库：
```sql
create database cm;
create database amon;
create database hive;
create database hue;
create database oozie;
```
导入sql数据文件
```bash
set names utf8;
use cm;
source /root/cm.sql
use amon;
source /root/amon.sql
use hive;
set names utf8;
source /root/hive.sql
use hue;
source /root/hue.sql
use oozie;
source /root/oozie.sql
```
导入数据过程中可能会出现缺少主键的报错，这是因为MGR要求每个表必须有主键，只需要根据报错行数，找到对应的表，为其添加上主键。

5.3 在hadoop3、hadoop4、hadoop5上新建用户scm,重启hadoop3、hadoop4、hadoop5上的mysql，先不启动MGR,在每个节点上新建scm用户并授权，此过程需要关闭binlog，否则会导致MGR从节点报错
```
set SQL_LOG_BIN=0;
create user 'scm'@'%' identified with 'mysql_native_password' by 'lnsoft';
grant all privileges on cm.* to 'scm'@'%';
FLUSH PRIVILEGES;
set SQL_LOG_BIN=1;
```
新建用户后重启启用binlog
5.4 将CM Server访问的数据库指向MGR集群的主节点（hadoop3）,修改/opt/cm-5.14.2/etc/cloudera-scm-server/db.properties中的数据库地址、用户名、密码
部署新版Mysql JDBC驱动 mysql-connector-java-5.1.47.jar，将其上传至hadoop2的
/opt/cm-5.14.2/share/cmf/lib目录下,重启CM Server
```shell
/opt/cm-5.14.2/etc/init.d/cloudera-scm-server stop
/opt/cm-5.14.2/etc/init.d/cloudera-scm-server start
```
启动完毕后访问http://hadoop2:7180/，能正常登陆，查看各个页面功能均正常，即表明数据库迁移成功。

6. Mysql高可用测试
停止hadoop3上的Mysql实例，查看在hadoop4登录Mysql客户端，查看group members;
```bash
mysql> select * from performance_schema.replication_group_members;
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| CHANNEL_NAME              | MEMBER_ID                            | MEMBER_HOST | MEMBER_PORT | MEMBER_STATE | MEMBER_ROLE | MEMBER_VERSION |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
| group_replication_applier | 7b4faa2d-1862-11e9-bfae-c81fbea4d979 | hadoop4     |        3306 | ONLINE       | PRIMARY     | 8.0.12         |
| group_replication_applier | 7e32e66b-1862-11e9-9cf0-c81fbea4da25 | hadoop5     |        3306 | ONLINE       | SECONDARY   | 8.0.12         |
+---------------------------+--------------------------------------+-------------+-------------+--------------+-------------+----------------+
2 rows in set (0.23 sec)
```
发现hadoop4成为了新的主节点，修改hadoop2上/opt/cm-5.14.2/etc/cloudera-scm-server/db.properties中的数据库地址、用户名、密码，重启CM Server，正常启动即说明Mysql高可用配置成功。


## 相关文章

| [一文教你在CentOS7下安装MySQL及搭建主从复制](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491190&idx=1&sn=6e6ed61a51e2f214d19e304038bde8b4&source=41#wechat_redirect) | [手把手教大家搭建MySQL主从复制](https://mp.weixin.qq.com/s/R89aCCFvCvudLp6FUn2JjQ) | [Linux系统安装MySQL8.0版本详细教程](https://blog.csdn.net/cst522445906/article/details/129165658) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |