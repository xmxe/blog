---
title: MySQL安装
sticky: 16
categories: 数据库
tags: 安装
index_img: /assert/mysql.jpg
img: 
---

#### Linux安装MySQL

##### 压缩包安装

1. mysql安装包上传usr/local[下载地址](https://downloads.mysql.com/archives/community/)
2. tar -zxvf mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz解压 解压后位置usr/local/mysql
3. 先检查是否有mysql用户组和mysql用户,没有就添加有就忽略
```shell
# 查看用户组
groups mysql
# 添加用户组
groupadd mysql
# 删除 groupdel
```
4. 添加用户mysql然后将mysql用户添加到到用户组mysql
```shell
useradd -r -g mysql mysql
# userdel删除
```
5. 进入安装mysql软件目录
```shell
cd mysql/
```
6. 修改目录拥有者为mysql用户
```shell
chown -R mysql:mysql ./
```
###### 5.7版本

7. 修改配置文件  vim /etc/my.cnf（没有就新建）
```shell
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
log-error=/usr/local/mysql/mysql.err
pid-file=/usr/local/mysql/data/mysql/mysql.pid
# character config
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
```
8. 执行命令
```shell
./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/ --user=mysql --initialize
```
9. 查看初始密码 控制台查看或者
```shell
cat /usr/local/mysql/mysql.err
```
localhost:后面的一串字符就是

10. 启动mysql
```shell
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
chmod +x /etc/init.d/mysql
service mysql start

mysql -u root -p root
# 修改密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
# 使密码生效
flush privileges; 

# 修改远程连接
update user set host='%' where user='root';
flush privileges;
exit;
```
###### 5.6版本

7. 安装数据库，此处可能出现错误
```shell
./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```
错误1：命令报错-bash: ./scripts/mysql_install_db: /usr/bin/perl: bad interpreter: No such file or directory.貌似提示注释器错误，没有/usr/bin/perl文件或者档案，解决办法（安装perl跟perl-devel即可）：执行 **yum -y install perl perl-devel**后在初始化数据库即可。
错误2：//bin/mysql_install_db 
FATAL ERROR: please install the following Perl modules before executing
Data::Dumper
**yum install -y perl-Data-Dumper** 即可。。

8. 修改当前目录拥有者为root用户
```shell
chown -R root:root ./
```
再次运行
```shell
./scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```
9. 修改当前data目录拥有者为mysql用户
```shell
chown -R mysql:mysql data
```
10. 添加开机启动，把启动脚本放到开机初始化目录。
```shell
cp support-files/mysql.server /etc/init.d/mysql
```
12. 赋予可执行权限
```shell
chmod +x /etc/init.d/mysql
```
13. 添加服务
```shell
chkconfig --add mysql
```
14. 显示服务列表
```shell
chkconfig --list
# 如果看到mysql的服务，并且3,4,5都是on的话则成功，如果是off，则执行
chkconfig --level 345 mysql on
```
15. 启动mysql服务
```shell
service mysql start
```
假如服务未成功启动:看第16步
[log-error set to '/var/log/mariadb/mariadb.log', however file don't exists](http://blog.csdn.net/duyuanhai/article/details/78604894)

**报错的情况下执行以下操作**
16. 需要创建报错中缺失的文件夹
```shell
mkdir /var/log/mariadb，touch /var/log/mariadb/mariadb.log
```
17. 把mysql客户端放到默认路径
```shell
ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql
```
18. 更改mysql密码
```shell
./bin/mysqladmin -u root -h localhost.localdomain password 'root'
```
19. 登录mysql quit退出
```shell
./bin/mysql -h127.0.0.1 -uroot -proot
```
20. mysql -uroot -proot 无法连接的话
```shell
ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock
```
21. 直接授权允许navicat远程连接linux服务器上的mysql数据库
```shell
mysql：GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
```
22. 如果没有my.cnf文件直接在/etc/目录下创建即可

###### 8.0版本

1-7同上...

8. 安装
```shell
bin/mysqld --user=mysql  --initialize --datadir=/usr/local/mysql/data

# bin/mysqld --user=mysql --datadir=/usr/local/mysql/data --initialize
```
root@localhost之后的一串字符就是初始密码

9-15同上...

16. 服务未成功启动查看data目录下的localhost.localdomain.err错误日志，

Can't start server : Bind on unix socket: Address already in use Do you already have another mysqld server running on socket: /tmp/mysql.sock ?解决方法是删除 rm -rf /tmp/mysql.sock(由于以前安装过mysql的原因)

17...

18. 
```shell
/bin/mysql -h127.0.0.1 -uroot -p进入mysql更改密码用set password='root';
```
##### yum安装

一、安装本地YUM源、MySQL在MySQL官网中下载YUM源rpm安装包

1. 把上面的rpm文件下载下来放到服务器上 或者在linux系统中通过wget命令下载

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
二、配置mysql

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
grep 'temporary password' /var/log/mysqld.log
# 登录mysql,用刚才从文件中找到的密码
mysql -uroot -p
# 尝试修改密码,使用下面的命令修改root用户的密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';

# 添加远程登陆
# 创建一个能全局访问的用户root
CREATE USER 'root'@'%' IDENTIFIED BY 你的密码'';
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

sql_mode=only_full_group_by问题
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

#### windows安装MySQL

##### 卸载mysql
```
关闭mysql服务:net stop mysql
删除mysql服务:mysqld -remove
删除mysql相关文件夹，如想保留数据可保留data文件
```
##### 安装mysql

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

3. 管理员打开cmd切换到mysql bin目录下(一定要切换) 执行**mysqld install**安装mysql服务，成功打印**service successfully installed**后执行**mysqld --initialize --console**会对数据库做初始化并打印相应日志：可以找到root@localhost之后的一串字符就是初始密码，然后执行net start mysql开启服务
mysql -uroot -p输入密码后set password for root@localhost = password('123456');修改密码（5.7版本以前）
8.0修改密码set password = '123456';


#### 8.0连接navicat

navicat连接8.0需要把mysql用户登录密码加密规则还原成mysql_native_password
```shell
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root' PASSWORD EXPIRE NEVER; #修改加密规则
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root'; #更新一下用户的密码
FLUSH PRIVILEGES; #刷新权限

然后 use mysql;

update user set host = '%' where user = 'root';
FLUSH PRIVILEGES;
```

#### emoji插入mysql

```shell
[client]
# 客户端来源数据的默认字符集
default-character-set= utf8mb4
[mysqld]
# 服务端默认字符集
character-set-server=utf8mb4
# 连接层默认字符集
collation-server=utf8mb4_unicode_ci
wait_timeout=31536000
interactive_timeout=31536000
[mysql]
# 数据库默认字符集
default-character-set= utf8mb4
```

#### 相关文章
- [一文教你在CentOS7下安装MySQL及搭建主从复制](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491190&idx=1&sn=6e6ed61a51e2f214d19e304038bde8b4&source=41#wechat_redirect)
- [手把手教大家搭建MySQL主从复制](https://mp.weixin.qq.com/s/R89aCCFvCvudLp6FUn2JjQ)
- [手把手教你部署一套生产级的mysql数据库](https://mp.weixin.qq.com/s/v_TCv3KJgoghRPfwQSpcaA)