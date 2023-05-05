---
title: Linux相关
tags: 计算机
index_img: /assert/linux.jpg
img: https://img2.baidu.com/it/u=1090189516,820941418&fm=253&fmt=auto&app=138&f=JPEG?w=750&h=500

---


### 相关命令
#### \>/dev/null

这条命令的作用是将标准输出1重定向到/dev/null中。/dev/null代表linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”。那么执行了>/dev/null之后，标准输出就会不再存在，没有任何地方能够找到输出的内容。

1>/dev/null 2>&1 &详解：
0：表示标准输入stdin
1：表示标准输出stdout，系统默认为1，可省略(即1>/dev/null等价于>/dev/null)
2：表示标准错误stderr
\>：表示重定向（即将输出定向到指定路径文件，>/dev/null表示将标准输出重定向到空设备文件，即不输出任何信息到终端，即不显示任何信息。）
2>&1：其中的&表示等同于的意思，即2(标准错误stderr)的重定向等同于1
&表示后台运行

```shell
# 后台执行abc.jar即便关闭终端也继续运行 不输出任何日志
nohup java -jar abc.jar >/dev/null 2>&1 &
```

#### 关闭ssh情况下不退出进程

nohup command &
command参数：要执行的命令行
但是这种方式启动项目会默认生成一个nohup.out的文件来记录日志，而且会越来越大，不生成日志使用>/dev/null 2>&1

最终命令
```shell
nohup command >/dev/null 2>&1 &
```
0:表示键盘输入(stdin)
1:表示标准输出(stdout),系统默认是1
2:表示错误输出(stderr)

command >/dev/null 2>&1 & == command 1>/dev/null 2>&1 &
command:表示shell命令或者为一个可执行程序
\>:表示重定向到哪里
/dev/null:表示Linux的空设备文件
2:表示标准错误输出
&1:&表示等同于的意思,2>&1,表示2的输出重定向等于于1
&:表示后台执行,即这条指令执行在后台运行
1>/dev/null:表示标准输出重定向到空设备文件,也就是不输出任何信息到终端,不显示任何信息。
2>&1:表示标准错误输出重定向等同于标准输出,因为之前标准输出已经重定向到了空设备文件,所以标准错误输出也重定向到空设备文件。
这条命令的意思就是在后台执行这个程序,并将错误输出2重定向到标准输出1,然后将标准输出1全部放到/dev/null文件,也就是清空.
所以可以看出">/dev/null 2>&1 &"常用来避免shell命令或者程序等运行中有内容输出

#### 安装ifconfig

```shell
# centos
yum -y install net-tools
# ubuntu
apt install net-tools
```
- apt与apt-get的区别
apt可以看作apt-get和apt-cache命令的子集,可以为包管理提供必要的命令选项。apt-get虽然没被弃用，但作为普通用户，还是应该首先使用apt
[centos7更换yum源](https://mirrors.cnnic.cn/help/centos/)


#### 配置jdk环境变量

```shell
vim /etc/profile
export JAVA_HOME=/usr/jdk1.8.0_121
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile # 使环境变量生效
echo $PATH # 查看环境变量
```

ubuntu按照上面的步骤在/etc/environment再配置一遍，配置完成后
```shell
source /etc/environment
```

#### 安装iptables

```shell
systemctl stop firewalld.service # 停止服务
systemctl mask firewalld.service # 屏蔽服务
yum -y install iptables-services  # 安装iptables服务
systemctl enable iptables # 开机启动iptables
systemctl start iptables # 启动iptables
service iptables save # 保存防火墙规则
```

#### centos7防火墙

```shell
# 启动
systemctl start firewalld.service
# 关闭
systemctl stop firewalld.service
# 查看状态
systemctl status firewalld.service
# 开机禁用 
systemctl disable firewalld.service
# 开机启用
systemctl enable firewalld.service
# 查看系统服务列表
systemctl list-unit-files
```

#### 查看开机启动的服务列表：

```shell
# 查看某个服务是否开机启动：
systemctl list-unit-files|grep enabled
# 查看启动失败的服务列表：systemctl --failed
systemctl is-enabled firewalld.service

# 端口开放
firewall-cmd --zone=public --add-port=80/tcp --permanent
#（--permanent永久生效，没有此参数重启后失效）

# 重新载入
firewall-cmd --reload查看
firewall-cmd --zone= public --query-port=80/tcp
# 删除
firewall-cmd --zone= public --remove-port=80/tcp --permanent

# 查看开放的端口
firewall-cmd --list-ports

# ubuntu
#查看防火墙当前状态
sudo ufw status
# 开启防火墙
sudo ufw enable
# 关闭防火墙
sudo ufw disable
# 查看防火墙版本
sudo ufw version
# 默认允许外部访问本机
sudo ufw default allow
# 默认拒绝外部访问主机
sudo ufw default deny
# 允许外部访问53端口
sudo ufw allow 53
# 拒绝外部访问53端口
sudo ufw deny 53
# 允许某个IP地址访问本机所有端口
sudo ufw allow from 192.168.0.1
# 重启防火墙
sudo ufw reload
```
#### 生成文件

\> 直接把内容生成到指定文件，会覆盖源文件中的内容，还有一种用途是直接生成一个空白文件，相当于touch命令
```shell
echo 1 > a.txt 输出1
# >>尾部追加，不会覆盖掉文件中原有的内容
echo 2 >> a.txt 输出1 2

cat file1 >> file2 # 把file1的文档内容输入file2这个文档里
```
#### 用户/用户组

```shell
# 查看linux所有用户
cut -d : -f 1 /etc/passwd
# 查看linux所有用户组
cut -d : -f 1 /etc/group # -d : 以“：”为分割符进行分割 -f 1展示第一列
groups # 用户名：查看用户所在用户组
```

#### centos7 定时运行任务脚本

```shell
# 安装crontab
yum install vixie-cron
yum install crontabs
# 开启crontab服务
service crond start # 启动服务
service crond stop # 关闭服务
service crond restart # 重新启动服务
service crond reload # 又一次加载配置
# 添加任务(两种方式)
1. crontab -e  * * * * * /usr/local/a.sh。
crontab -l # 列出当前的全部调度任务
crontab -l -u jp #列出用户jp的全部调度任务
crontab -r # 删除全部任务调度工作
2. # 直接编辑 vim /etc/crontab
# 添加* * * * * root /usr/local/a.sh
# 注意要使用绝对路径

# 查看邮件
cat /var/spool/mail/root
# 查看日志
cd /var/log
ls cron

```
[Linux定时任务调度(crontab)，太实用了！](https://mp.weixin.qq.com/s/c91XWEQvr9Axcf0hjvuKgg)


#### 修改文件所属用户和用户组

```shell
# 修改a.txt文件所属用户为jay，所属用户组为fefjay
chown jay:fefjay a.txt
# 递归修改文件夹my及包含的所有子文件（夹）的所属用户和用户组
chown -R jay:fefjay my
```

#### curl命令

```shell
curl -d 'login=emma＆password=123'-X POST URL
curl -d 'login=emma' -d 'password=123' -X POST URL
-X # POST可以省略

curl -L -X POST URL -d 'id=3&pwd=jae_123'
curl -H "Content-Type: application/json" -X POST -d '{"abc":123,"bcd":"nihao"}' URL

```

- [Linux curl命令最全详解](https://blog.csdn.net/angle_chen123/article/details/120675472)
- [curl其他参数介绍](https://www.cnblogs.com/fan-gx/p/12321351.html)
- [堪称神器的命令行工具系列——curl](https://mp.weixin.qq.com/s/ryhphDFvx0ml9NrDUks2NA)


#### Linux系统服务

```shell
# 添加自定义系统服务的目录：
/lib/systemd/system # lib/systemd/system真实地址是/usr/lib/system/system地址，
/usr/lib/systemd/system/ # 软件包安装的单元
/etc/systemd/system/ # 系统管理员安装的单元,优先级更高
# 优先级为 /etc/systemd/system /run/systemd/system /lib/systemd/system
# 如果同一选项三个地方都配置了，优先级高的会覆盖优先级低的。

# 开机启动执行命令：编辑/etc/rc.d/rc.local
# 添加要执行的命令或者脚本,并且赋予执行权限
chmod +x /etc/rc.d/rc.local
```

#### 复制目录
```shell
cp -r
```
#### 查看隐藏目录
```shell
ls -a
```

#### 图形化页面卡死重启
```shell
kill -9 gnome-shell pid
```
#### 查看centos版本
```shell
cat /etc/redhat-release
```
#### 重命名文件

```shell
# 例:把a替换为xxx
rename “a” “xxx” *.txt
# 或者使用mv命令
```

#### 端口

```shell
netstat -antu # 可以查看所有tcp、udp端口开放情况
netstat -ntlp # 查看正在运行的端口(t代表tcp 加u查看udp)
lsof -i: 9090 # 查看某一端口运行的程序
netstat -ntulp|grep # 端口号 查看指定端口被哪个进程占用的情况
ps -ef|grep abc # 查找abc进程
ps -aux # 显示所有进程
# 发现A进程占用该端口号
ps -ef|grep A #查看pid

kill 9 pid #杀掉进程
```

#### 找到pid并kill的shell脚本

```shell
#!/bin/sh
# #!/bin/bash是指此脚本使用/bin/bash来解释执行。其中，#!是一个特殊的表示符，其后，跟着解释此脚本的shell路径。命令文件所在的路径是/bin/sh或者/usr/bin/sh.bash只是shell的一种，还有很多其它shell，如：sh,csh,ksh,tcsh.除第一行外，脚本中所有以“#”开头的行都是注释。
# 注意不要有空格，否则解释成命令
jar=abc.jar
# ``等价于$()
pid=$(ps -ef | grep java | grep $jar |grep -v grep | awk '{print $2}')
kill -9 $pid
echo "$pid killed"
nohup command >/dev/null 2>&1 &
```
[shell菜鸟教程](https://www.runoob.com/linux/linux-shell.html)

#### 设置静态ip后无法连接外网的问题
因为动态ip会自动分配DNS 而静态ip需要手动配置DNS
centos7在/etc/sysconfig/network-scripts/ifcfg-ens33 写入DNS1=114.114.114.114
ubuntu在/etc/resolv.conf写入nameserver 114.114.114.114

[Linux配置IP地址](https://www.cnblogs.com/adforce/p/3363681.html)

#### 命令行更改MAC地址

```shell
/sbin/ifconfig eth0 down
/sbin/ifconfig eth0 hw ether 00:50:56:94:16:a8
/sbin/ifconfig eth0 up
service network restart
```

#### 查看文件

```shell
cat h.txt | grep -v # "hello"过滤掉特定字符串,效率低，因为有管道
grep -v "hello" h.txt # 可以直接跟文件名，效率快

head -n k = head -n +k = head k
tail -n k = tail -n -k = tail k # k为指定行数

head -n 3 = head -n +3 = head -3 # 显示文件前3行

tail -n 3 = tail -n -3 = tail -3  # 显示文件最后3行
head -n -k # 其中-k的意义是除了最后k行的所有行
head -n -3 filename # 查看filename除了最后3行的所有行
tail -n +k # 是从第k行开始，输出所有行
tail -n +3 # 从第三行开始输出所有行
tail -f finename # 实时跟踪文件，如果文件不存在，则终止
tail -F filename # 如果文件不存在，会继续尝试
```

#### 源码安装配置(configure)、编译(make)、安装(make install)

```shell
./configure --prefix=/usr/local/test
# 编译出错时，清除编译生成的文件
make distclean
# 编译安装到指定目录下
make PREFIX=/usr/local/redis install
# 卸载
make uninstall
```

#### 设置 keepalived 服务开机启动

```shell
chkconfig keepalived on
chkconfig --add name
          --del name
          --list
```

#### Ubuntu图形界面允许root登陆

```shell
# 设置root密码
sudo passwd root  # 终端会先验证密码 然后在设置root密码
vi /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
# 增加两行  greeter-show-manual-login=true all-guest=false#不允许guest用户登陆
cd /etc/pam.d  # 编辑gdm-autologin和gdm-password文件 注释掉auth required pam_succeed_if.so user != root quiet_success
vi /root/.profile # 将mesg n || true 修改为 tty -s && mesg n || true
reboot

```

[解决Ubuntu的root账号无法登录SSH问题](https://www.cnblogs.com/yixius/articles/6971054.html)


#### 查看指定目录大小
```shell
du -h --max-depth=1 /usr
```
#### 查看磁盘空间
```shell
df -h
```
#### 查看内存
```shell
free -h

top -n 2 # 表示更新两次后终止更新显示并退出
top -d 3 # 表示更新周期为3秒
# top命令进入展示信息后按大写M表示按照内存大小排序展示 按大写P表示按照CPU使用率进行排序
```

#### 读、写、运行

三项权限可以用数字表示，r=4,w=2,x=1 rw-r--r--用数字表示成644
```shell
chmod 754 filename
# 将filename文件的读写运行权限赋予文件所有者，把读和运行的权限赋予群组用户，把读的权限赋予其他用户。
# chmod +x是将文件状态改为可执行，而chmod 777是改变文件读写权限。
```

#### centos7卸载openjdk

```shell
rpm -qa|grep jdk  # 查看已有的openjdk -q(query) -a(all)
rpm -ev --nodeps (上条命令的查询结果) #卸载

ubuntu 
apt-get remove openjdk*

```
#### rpm参数

```yaml
-a　查询所有套件。
-b<完成阶段><套件档>+或-t<完成阶段><套件档>+　设置包装套件的完成阶段，并指定套件档的文件名称。
-c　只列出组态配置文件，本参数需配合"-l"参数使用。
-d　只列出文本文件，本参数需配合"-l"参数使用。
-e<套件档>或--erase<套件档>　删除指定的套件。
-f<文件>+　查询拥有指定文件的套件。
-h或--hash　套件安装时列出标记。
-i　显示套件的相关信息。
-i<套件档>或--install<套件档>　安装指定的套件档。
-l　显示套件的文件列表。
-p<套件档>+　查询指定的RPM套件档。
-q　使用询问模式，当遇到任何问题时，rpm指令会先询问用户。
-R　显示套件的关联性信息。
-s　显示文件状态，本参数需配合"-l"参数使用。
-U<套件档>或--upgrade<套件档>升级指定的套件档。
-v　显示指令执行过程。
-vv　详细显示指令执行过程，便于排错。
-addsign<套件档>+　在指定的套件里加上新的签名认证。
--allfiles　安装所有文件。
--allmatches　删除符合指定的套件所包含的文件。
--badreloc　发生错误时，重新配置文件。
--buildroot<根目录>　设置产生套件时，欲当作根目录的目录。
--changelog　显示套件的更改记录。
--checksig<套件档>+　检验该套件的签名认证。
--clean　完成套件的包装后，删除包装过程中所建立的目录。
--dbpath<数据库目录>　设置欲存放RPM数据库的目录。
--dump　显示每个文件的验证信息。本参数需配合"-l"参数使用。
--excludedocs　安装套件时，不要安装文件。
--excludepath<排除目录>　忽略在指定目录里的所有文件。
--force　强行置换套件或文件。
--ftpproxy<主机名称或IP地址>　指定FTP代理服务器。
--ftpport<通信端口>　设置FTP服务器或代理服务器使用的通信端口。
--help　在线帮助。
--httpproxy<主机名称或IP地址>　指定HTTP代理服务器。
--httpport<通信端口>　设置HTTP服务器或代理服务器使用的通信端口。
--ignorearch　不验证套件档的结构正确性。
--ignoreos　不验证套件档的结构正确性。
--ignoresize　安装前不检查磁盘空间是否足够。
--includedocs　安装套件时，一并安装文件。
--initdb　确认有正确的数据库可以使用。
--justdb　更新数据库，当不变动任何文件。
--nobulid　不执行任何完成阶段。
--nodeps　不验证套件档的相互关联性。
--nofiles　不验证文件的属性。
--nogpg　略过所有GPG的签名认证。
--nomd5　不使用MD5编码演算确认文件的大小与正确性。
--nopgp　略过所有PGP的签名认证。
--noorder　不重新编排套件的安装顺序，以便满足其彼此间的关联性。
--noscripts　不执行任何安装Script文件。
--notriggers　不执行该套件包装内的任何Script文件。
--oldpackage　升级成旧版本的套件。
--percent　安装套件时显示完成度百分比。
--pipe<执行指令>　建立管道，把输出结果转为该执行指令的输入数据。
--prefix<目的目录>　若重新配置文件，就把文件放到指定的目录下。
--provides　查询该套件所提供的兼容度。
--queryformat<档头格式>　设置档头的表示方式。
--querytags　列出可用于档头格式的标签。
--rcfile<配置文件>　使用指定的配置文件。
--rebulid<套件档>　安装原始代码套件，重新产生二进制文件的套件。
--rebuliddb　以现有的数据库为主，重建一份数据库。
--recompile<套件档>　此参数的效果和指定"--rebulid"参数类似，当不产生套件档。
--relocate<原目录>=<新目录>　把本来会放到原目录下的文件改放到新目录。
--replacefiles　强行置换文件。
--replacepkgs　强行置换套件。
--requires　查询该套件所需要的兼容度。
--resing<套件档>+　删除现有认证，重新产生签名认证。
--rmsource　完成套件的包装后，删除原始代码。
--rmsource<文件>　删除原始代码和指定的文件。
--root<根目录>　设置欲当作根目录的目录。
--scripts　列出安装套件的Script的变量。
--setperms　设置文件的权限。
--setugids　设置文件的拥有者和所属群组。
--short-circuit　直接略过指定完成阶段的步骤。
--sign　产生PGP或GPG的签名认证。
--target=<安装平台>+　设置产生的套件的安装平台。
--test　仅作测试，并不真的安装套件。
--timecheck<检查秒数>　设置检查时间的计时秒数。
--triggeredby<套件档>　查询该套件的包装者。
--triggers　展示套件档内的包装Script。
--verify　此参数的效果和指定"-q"参数相同。
--version　显示版本信息。
--whatprovides<功能特性>　查询该套件对指定的功能特性所提供的兼容度。
--whatrequires<功能特性>　查询该套件对指定的功能特性所需要的兼容度。

```

#### 从服务器复制文件到本地

```shell
# 从服务器复制文件到本地
scp root@192.168.1.100:/data/test.txt /home/myfile/
# 从服务器复制文件夹到本地
scp -r root@192.168.1.100:/data/ /home/myfile/
# 从本地复制文件到服务器
scp /home/myfile/test.txt root@192.168.1.100:/data/
# 从本地复制文件夹到服务器
scp -r /home/myfile/ root@192.168.1.100:/data/
```

#### ssh双向免密登录服务器A、B

```shell
# 生成密钥
ssh-keygen -t rsa
# 生成密钥的位置
cd /root/.ssh/
# 此命令在A机器执行，目的将A的公钥发送至B机器
scp id_rsa.pub root@BIP地址:/root/.ssh/id_rsa_A.pub
# 此命令在B机器执行，目的将B的公钥发送至A机器
scp id_rsa.pub root@AIP地址:/root/.ssh/id_rsa_B.pub

# 如果发送文件夹 使用scp -r /test root@B:/root/test

# 查看远程复制是否成功 写入密钥
cat id_rsa_A(或者B).pub >> authorized_keys
# 如果出现agent admitted failure to sign using the key
ssh-add ~/.ssh/id_rsa 
# ssh A和ssh B测试ssh本机和远程是否已经免密登录（第一次免密登录需要输入密码，以后不需要）
ssh登陆 ssh root@ip
```
[科普：什么是SSH？](https://mp.weixin.qq.com/s/1e4aGp_cx0E_qCHVuS3GMg)


#### centos7/ubuntu通用

```shell
# 默认命令行
systemctl set-default multi-user.target  （init 3）

# 默认图形页面
systemctl set-default graphical.target  (init 5)
```

#### 查看系统内核 uname -r

```shell
# 全部内核
rpm -qa | grep kernel
yum list installed | grep kernel
#删除多余内核
yum remove kernel-
```

#### 测试端口是否开通

```shell
ssh -v -p port username@ip
# -v 调试模式(会打印日志) -p 指定端口 username可以随意

# 使用 telnet 命令
telnet ip 端口

# 使用nc命令
nc -vu ip 端口
# -v 输出交互或出错信息，新手调试时尤为有用,-u指定nc使用UDP协议，默认为TCP
```


#### arthas调试常用命令

```
cls 清空当前屏幕内容

trace 类名 方法名  '#cost > 10'
查看某个类的某个方法执行多长时间 (加后面参数只会展示耗时大于10ms的调用路径)

sc 类名 查看该类是否被jvm加载

sm 类名 查看该类的方法信息

jad 类名 反编译

watch 类名 方法名 "{params,returnObj,throwExp}" -e -b  -x 2
观察方法出参，返回值，-b在调用之前观察 -e在方法异常之后观察 -x指定输出结果的遍历深度 默认为1

thread 查看线程

redefine class文件(带class后缀名):热部署，修改jvm的class并不会修改本地的class文件 服务重启后失效 注意class新增属性、方法且方法正在运行的时候会热部署失败

退出arthas
quit或exit 退出当前连接，完全退出使用stop(所有客户端连接都会退出)

logger --name ROOT --level debug动态修改日志级别
```

- [arthas官方文档](https://arthas.aliyun.com/doc/quick-start.html)
- [学会arthas，让你3年经验掌握5年功力](https://mp.weixin.qq.com/s/7RMjfIYlmskMsifnhIzxgA)


#### Linux交换空间(swap space)

交换空间是磁盘上的一块区域，可以是一个分区，也可以是一个文件，或者是他们的组合。简单点说，当系统物理内存吃紧时，Linux会将内存中不常访问的数据保存到swap上，这样系统就有更多的物理内存为各个进程服务，而当系统需要访问swap上存储的内容时，再将swap上的数据加载到内存中，这就是我们常说的swap out和swap in

**设置交换空间**
```shell
# 1. 在Home目录创建一个大小为16G的swap文件，块大小为1MByte，总共1K个块，也就是总共1GB
dd if=/dev/zero of=~/swapfile bs=1M count=1k
# 2. 格式化新增的swap文件
mkswap ～/swapfile
# 3. 启动新增的swap文件
swapon ～/swapfile
# 4. 查看swap空间大小，发现增加了16G
free
# 5. 关闭新增的swap文件
swapoff ～/swapfile
# 6. 设置开机自启动，开机后自动启动新增的swap文件，在/etc/fstab中新增如下命令
/swapfile none swap sw 0 3
```

#### 挂载
```shell
# 一、查看磁盘挂载目录
df -h（查看分区情况及数据盘名称）
mkdir /usr（如果没有就创建，否则此步跳过）
# 二、卸载磁盘
umount /data（卸载硬盘已挂载的data目录）
# 三、挂载到新目录
mount /dev/vdb1 /usr（挂载到usr目录）
# 四、修改 /etc/fstab
vi /etc/fstab （编辑fstab文件修改或添加，使重启后可以自动挂载）
/dev/vdb1 /usr ext3 auto 0 0
# 五、重新挂载所有分区
mount -a
# 六、验证
df -h /usr/
# 七、重启服务器
reboot
```
需要注意的是：在实际操作过程中，挂载的目录会覆盖掉原目录的文件信息。可以先进行旧目录备份，挂载完成后在恢复数据。


#### source filename与sh filename及./filename执⾏脚本的区别

1. 当shell脚本具有可执⾏权限时，⽤sh filename与./filename执⾏脚本是没有区别得。./filename是因为当前⽬录没有在PATH中，所有”.”是⽤来表⽰当前⽬录的。
2. sh filename重新建⽴⼀个⼦shell，在⼦shell中执⾏脚本⾥⾯的语句，该⼦shell继承⽗shell的环境变量，但⼦shell新建的、改变的变量不会被带回⽗shell，除⾮使⽤export。
3. source filename：这个命令其实只是简单地读取脚本⾥⾯的语句依次在当前shell⾥⾯执⾏，没有建⽴新的⼦shell。那么脚本⾥⾯所有新建、改变变量的语句都会保存在当前shell⾥⾯


### 环境变量

#### 环境变量文件

- /etc/profile：此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行，并从/etc/profile.d目录的配置文件中收集shell的设置
- /etc/bashrc：为每一个运行bash shell的用户执行此文件，当bash shell被打开时，该文件被读取也就是说，当用户shell执行了bash时，运行这个文件；
- ~/.bash_profile：每个用户都可使用该文件输入专用于自己使用的shell信息，当用户登录时，该文件仅仅执行一次！默认情况下，它设置一些环境变量，执行用户的.bashrc文件
- ~/.bashrc：该文件包含用于你的bash shell的bash信息，当登录时以及每次打开新的shell时，该文件被读取。该文件存储的是专属于个人bash shell的信息，当登录时以及每次打开一个新的shell时，执行这个文件。在这个文件里可以自定义用户专属的个人信息。
- \~/.bash_logout：当每次退出系统(退出bash shell)时，执行该文件；另外，/etc/profile中设定的变量(全局)的可以作用于任何用户，而\~/.bashrc等中设定的变量(局部)只能继承/etc/profile中的变量，他们是“父子”关系；

#### Ubuntu下bash的几个初始化文件
- /etc/profile 全局(公有)配置，不管是哪个用户，登录时都会读取该文件；
- /etc/bashrc Ubuntu下没有此文件，与之对应的是/etc/bash.bashrc，它也是全局的；bash执行时，不管是何种方式，都会读取此文件；
- \~/.profile 若bash是以login方式执行时，读取\~/.bash_profile，若它不存在，则读取\~/.bash_login，若前两者不存在，读取\~/.profile；另外，图形模式登录时，此文件将被读取，即使存在\~/.bash_profile和\~/.bash_login；
- \~/.bash_login 若bash是以login方式执行时，读取\~/.bash_profile，若它不存在，则读取\~/.bash_login，若前两者都不存在，则读取\~/.profile；
- \~/.bash_profile Ubuntu默认没有此文件，可新建。只有bash是以login形式执行时，才会读取此文件。通常该配置文件还会配置成去读取\~/.bashrc；
- \~/.bashrc当bash是以non-login形式执行时，读取此文件。若是以login形式执行，则不会读取此文件；
- \~/.bash_logout注销时，且是login形式，此文件才会读取。也就是说，在文本模式注销时，此文件会被读取，图形模式注销时，此文件不会被读取。
- /etc/environment系统的环境变量，系统应用程序的执行与用户环境可以是无关的，但与系统环境是相关的
1. 在登录时,操作系统定制用户环境时使用的第一个文件就是/etc/profile,此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行。
2. 在登录时操作系统使用的第二个文件是/etc/environment，系统在读取你自己的profile前,设置环境文件的环境变量。
3. 在登录时用到的第三个文件是.profile文件,每个用户都可使用该文件输入专用于自己使用的shell信息,,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件。/etc/bashrc:为每一个运行bash shell的用户执行此文件。当bash shell被打开时,该文件被读取。
4. /etc/environment是设置整个系统的环境，而/etc/profile是设置所有用户的环境，前者与登录用户无关，后者与登录用户有关。
先执行/etc/enviroment，后执行/etc/profile

[Linux环境变量配置的6种方法](https://mp.weixin.qq.com/s/D32FE4CS2pT89czVUv0SOg)

### 相关文章

- [Linux基础知识总结](https://javaguide.cn/cs-basics/operating-system/linux-intro.html)
- [程序员必备的150个Linux命令！](https://mp.weixin.qq.com/s/wejhjmbtS16AdlDo0JiLMA)
- [新人必备的Linux命令！](https://mp.weixin.qq.com/s/TvUcd7mLkHxD_botoo7BQg)
- [1000+常用的Linux命令来袭](https://mp.weixin.qq.com/s/43JCaDehv0nbzDKbe0S8Kg)
- [作为一名Java开发人员，应该从多大程度上掌握Linux？](https://www.zhihu.com/question/319076999/answer/672319588?utm_source=wechat_timeline&utm_medium=social&utm_oi=1040923520439672832&from=timeline)
- [30个实例详解TOP命令](https://mp.weixin.qq.com/s/2FDa8fOm6x4-1MP9l9N7vA)
- [ping命令的7大用法,看完秒变大神！](https://mp.weixin.qq.com/s/liiz58PC1JlfPaEW81wUpg)
- [听说你ping用的很6,给我图解一下ping的工作原理](https://mp.weixin.qq.com/s/4pPdcSak3YLyqiz4Ns8htQ)
- [ping命令还能这么玩？](https://mp.weixin.qq.com/s/fLY3JH5_4_h4vABV7B-4uw)
- [后端线上问题排查常用命令收藏](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494347&idx=1&sn=2f6bfe4a5716d4b0aba6d8b5348f7033&source=41#wechat_redirect)
- [Java开发常用的Linux命令知识积累](https://mp.weixin.qq.com/s/jlmPjqFTfP4JNmbhpGoHsA)
- [这篇Linux总结的真棒！](https://mp.weixin.qq.com/s/F_0kiOWgQS4BINx2t2Rqtw)
- [Linux远程桌面管理工具！功能真心强大](https://mp.weixin.qq.com/s/UT3RYrUGP2E5JQbf1Z8c8w)
- [在Linux上保护SSH服务器连接的8种方法](https://mp.weixin.qq.com/s/kxvc95RJoSKqcXwwmyrPKg)
- [发现谁用kill -9关闭程序就开除！](https://mp.weixin.qq.com/s/zmfCC083VBrA8P6uWHL2DQ)
- [万字详解Linux常用指令](https://mp.weixin.qq.com/s/hXDAZD0LUJ9g85t6fkBR-Q)