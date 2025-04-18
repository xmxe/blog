---
title: Linux相关
tags: 计算机
index_img: /assert/linux.jpg
img: https://img2.baidu.com/it/u=1090189516,820941418&fm=253&fmt=auto&app=138&f=JPEG?w=750&h=500

---


## 相关命令

### nohup command &

`nohup command &`意思是在关闭ssh情况下不会退出进程。command参数表示要执行的命令行。但是这种方式启动项目会默认生成一个nohup.out的文件来记录日志，而且会越来越大，不生成日志使用`>/dev/null 2>&1`追加，这条命令的作用是将标准输出1重定向到/dev/null中。/dev/null代表linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”。那么执行了>/dev/null之后，标准输出就会不再存在，没有任何地方能够找到输出的内容。

> `1>/dev/null 2>&1 &`详解：
>> 0：表示标准输入stdin (键盘输入)
>> 1：表示标准输出stdout，系统默认为1，可省略(即1>/dev/null等价于>/dev/null)
>> 2：表示标准错误stderr
>> \>：表示重定向（即将输出定向到指定路径文件，>/dev/null表示Linux的空设备文件，即将标准输出重定向到空设备文件，即不输出任何信息到终端，即不显示任何信息。）
>> 2>&1：其中的&表示等同于的意思，即2(标准错误stderr)的重定向等同于1
&表示后台运行

最终命令
```shell
nohup command >/dev/null 2>&1 &
# 后台执行abc.jar即便关闭终端也继续运行，不输出任何日志
nohup java -jar abc.jar >/dev/null 2>&1 &

# command >/dev/null 2>&1 & == command 1>/dev/null 2>&1 &
# 1>/dev/null:表示标准输出重定向到空设备文件,也就是不输出任何信息到终端,不显示任何信息。
# 2>&1:表示标准错误输出重定向等同于标准输出,因为之前标准输出已经重定向到了空设备文件,所以标准错误输出也重定向到空设备文件。这条命令的意思就是在后台执行这个程序,并将错误输出2重定向到标准输出1,然后将标准输出1全部放到/dev/null文件,也就是清空.所以可以看出">/dev/null 2>&1 &"常用来避免shell命令或者程序等运行中有内容输出
# 将错误日志输出到文件
nohup command 2>error.log &
```

### 配置jdk环境变量

```shell
vim /etc/profile
export JAVA_HOME=/usr/jdk1.8.0_121
# export JRE_HOME=${JAVA_HOME}/jre  
# export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile # 使环境变量生效
echo $PATH # 查看环境变量
```

> ubuntu按照上面的步骤在/etc/environment再配置一遍，配置完成后
> ```shell
> source /etc/environment
> ```

### 防火墙

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
**安装iptables**

```shell
# 停止服务
systemctl stop firewalld.service
# 屏蔽服务
systemctl mask firewalld.service
# 安装iptables服务
yum -y install iptables-services
# 开机启动iptables
systemctl enable iptables
# 启动iptables
systemctl start iptables
# 保存防火墙规则
service iptables save
```

> iptables和firewalld都是用于管理防火墙的工具，但它们在操作方式、规则设置、安全级别以及和内核的关系方面存在一些区别。
>> **操作方式**：iptables需要通过编辑配置文件来设置规则，每次修改规则后需要全部刷新才能生效。而firewalld提供了动态修改单条规则和管理规则集的特性，允许在不破坏现有会话和连接的情况下更新规则。
>> **规则设置**：iptables主要基于接口来设置规则，判断网络的安全性。而firewalld则是基于区域，根据不同的区域来设置不同的规则，保证网络的安全。
>> **安全级别**：iptables默认是允许的，需要设置以后才能放行，而firewalld默认是拒绝的，需要设置以后才能放行。
>> 和内核的关系：iptables和firewalld都通过内核的netfilter来实现防火墙功能。但firewalld自身并不具备防火墙的功能，而是作为一个封装，使得管理iptables规则更加容易。
>> 此外，firewalld还提供了支持网络区域所定义的网络连接以及接口安全等级的动态防火墙管理工具，支持IPv4、IPv6防火墙设置以及以太网桥（在某些高级服务可能会用到，比如云计算），并且拥有两种配置模式∶ 运行时配置与永久配置。

**安装ifconfig**

```shell
# centos
yum -y install net-tools
# ubuntu
apt install net-tools
```

> **apt与apt-get的区别**: apt可以看作apt-get和apt-cache命令的子集,可以为包管理提供必要的命令选项。apt-get虽然没被弃用，但作为普通用户，还是应该首先使用apt
> [centos7更换yum源](https://mirrors.cnnic.cn/help/centos/)


### 查看开机启动的服务列表

```shell
# 查看某个服务是否开机启动
systemctl list-unit-files|grep enabled
# 查看启动失败的服务列表：systemctl --failed
systemctl is-enabled firewalld.service

# 端口开放
#（--permanent永久生效，没有此参数重启后失效）
# --zone=public:指定了要查询的防火墙区域。防火墙区域是网络连接的一个逻辑分组，每个区域都有自己的规则集来控制进入和离开该区域的数据包。公共区域（public）通常用于外部网络连接，例如Internet连接。可不加
firewall-cmd --zone=public --add-port=80/tcp --permanent
# 删除
firewall-cmd --zone=public --remove-port=80/tcp --permanent
# 重新载入
firewall-cmd --reload
# 查看
firewall-cmd --zone= public --query-port=80/tcp
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
### 文件操作

**清空文件内容**

```shell
> filename
echo -n > filename
# truncate命令可以用来缩小或扩展文件大小。要清空文件，可以将其大小设置为0
truncate -s 0 filename
cat /dev/null > filename
: > filename
```

**创建文件**

`>`直接把内容生成到指定文件，会覆盖源文件中的内容，还有一种用途是直接生成一个空白文件，相当于touch命令

```shell
echo 1 > a.txt 输出1
# >>尾部追加，不会覆盖掉文件中原有的内容
echo 2 >> a.txt 输出1 2
touch file
cat file1 >> file2 # 把file1的文档内容输入file2这个文档里
```

如果你希望在同一个操作中同时向屏幕和文件输出信息，可以使用管道（|）和tee命令。tee会复制其标准输入，并将其写入标准输出和指定的文件。

```bash
#!/bin/bash
# 输出一条消息到屏幕上，并同时将其写入一个名为output.txt的文件 -a选项告诉tee命令追加输出到文件，而不是覆盖它。
echo "This is a message" | tee -a output.txt
```
**tee命令**

```bash
tee file.txt
```
在运行这个命令后，你可以开始在命令行输入文本。当你按下Ctrl+D（EOF）时，你的输入会被写入file.txt并显示在屏幕上。

**查看文件**

```shell
# "hello"过滤掉特定字符串,效率低，因为有管道
cat h.txt | grep -v
# 可以直接跟文件名，效率快
grep -v "hello" h.txt

head -n k = head -n +k = head k
# k为指定行数
tail -n k = tail -n -k = tail k
# 显示文件前3行
head -n 3 = head -n +3 = head -3
# 显示文件最后3行
tail -n 3 = tail -n -3 = tail -3
# 其中-k的意义是除了最后k行的所有行
head -n -k
# 查看filename除了最后3行的所有行
head -n -3 filename
# 是从第k行开始，输出所有行
tail -n +k
# 从第三行开始输出所有行
tail -n +3
# 实时跟踪文件，如果文件不存在，则终止
tail -f finename
# 如果文件不存在，会继续尝试
tail -F filename

more example.txt
# 从文件的第20行开始显示
more +20 example.txt
# 设置每页显示20行
more -n 20 example.txt
# 搜索文件中的某个字符串,进入more命令后，按/键并输入要搜索的字符串，然后按回车。例如：
/search_string
# 跳转到文件中的某一行,进入more命令后，按:键，然后输入行号并按回车。例如：
:100
```

**重命名文件**

```shell
# 例:把a替换为xxx
rename “a” “xxx” *.txt
# 或者使用mv命令
```

**目录**

```shell
# 递归复制
cp -r
# 查看隐藏目录
ls -a
```

### 图形化页面卡死重启
```shell
kill -9 gnome-shell pid
```

### 用户/用户组

```shell
# 查看linux所有用户
cut -d : -f 1 /etc/passwd
# 查看linux所有用户组
# -d:以“：”为分割符进行分割 -f 1展示第一列
cut -d : -f 1 /etc/group
# 用户名：查看用户所在用户组
groups

# 修改文件所属用户和用户组(`chown`) 修改a.txt文件所属用户为jay，所属用户组为fefjay
chown jay:fefjay a.txt
# 递归修改文件夹my及包含的所有子文件（夹）的所属用户和用户组
chown -R jay:fefjay my

# 创建用户
useradd newuser
# 设置密码
passwd newuser
# 如果你希望新用户能够执行需要管理员权限的命令，你可以将用户添加到wheel组中。CentOS通常使用wheel组来代替sudo组，所以你需要将用户添加到wheel组。这可以通过usermod命令完成
usermod -aG wheel newuser
# 创建用户时，可以根据需要指定用户的主目录、默认Shell等参数。例如，使用useradd -d /home/newuser -s /bin/bash newuser命令可以创建一个主目录为/home/newuser、默认Shell为/bin/bash的新用户。如果不再需要某个用户，可以使用userdel命令删除该用户。如果需要同时删除用户的主目录和邮件目录，可以使用userdel -r命令。

# 添加用户组。如果需要指定用户组的GID（Group ID），可以使用-g选项。例如，要创建一个GID为1001的用户组，可以执行：groupadd -g 1001 mygroup
groupadd mygroup
# 将用户myuser添加到mygroup用户组中,这里的-a选项表示“追加”（append），意味着将用户添加到指定组，而不是替换用户的现有组。如果用户已经属于其他组，使用-a选项可以确保用户不会丢失这些组的信息。
usermod -aG mygroup myuser
# 如果需要修改现有用户组的名称，可以使用groupmod命令。例如，要将mygroup的名称更改为newgroupname，可以执行：
groupmod -n newgroupname mygroup
# 如果不再需要某个用户组，可以使用groupdel命令删除该用户组。例如，要删除mygroup用户组，可以执行：
groupdel mygroup

# chmod命令有两种主要的权限表示方法：数字模式和符号模式。数字模式：每个数字代表一个权限组（所有者、用户组、其他用户），数字由r（读权限，值为4）、w（写权限，值为2）、x（执行权限，值为1）组合而成。例如，7表示读、写和执行权限（4+2+1），6表示读和写权限（4+2），5表示读和执行权限（4+1）等。符号模式：使用“+”和“-”来添加或删除权限，“r”表示读权限，“w”表示写权限，“x”表示执行权限，“u”表示所有者，“g”表示用户组，“o”表示其他用户。

# 要为用户组分配读权限，可以使用chmod g+r <文件或目录>命令。要为用户组分配写权限，可以使用chmod g+w <文件或目录>命令。要为用户组分配执行权限，可以使用chmod g+x <文件或目录>命令。如果要为用户组分配所有权限（读、写、执行），可以使用chmod g+rwx <文件或目录>命令。如果要删除用户组的某个权限，可以使用-代替+。例如，要删除用户组的执行权限，可以使用chmod g-x <文件或目录>命令。
```

### 定时任务(crontab)

```shell
# 安装crontab
yum install vixie-cron
yum install crontabs
# 开启crontab服务
# 启动服务
service crond start
# 关闭服务
service crond stop
# 重新启动服务
service crond restart
# 又一次加载配置
service crond reload
# 添加任务(两种方式)
# 1.crontab -e  * * * * * /usr/local/a.sh
# 列出当前的全部调度任务
crontab -l
# 列出用户jp的全部调度任务
crontab -l -u jp
# 删除全部任务调度工作
crontab -r
# 2.直接编辑 vim /etc/crontab
# 添加* * * * * root /usr/local/a.sh 注意要使用绝对路径

# 查看邮件
cat /var/spool/mail/root
# 查看日志
cd /var/log
ls cron
```
> [Linux定时任务调度(crontab)，太实用了！](https://mp.weixin.qq.com/s/c91XWEQvr9Axcf0hjvuKgg)


### curl命令

```shell
curl -d 'login=emma&password=123'-X POST URL
curl -d 'login=emma' -d 'password=123' -X POST URL
-X # POST可以省略

curl -L -X POST URL -d 'id=3&pwd=jae_123'
curl -H "Content-Type: application/json" -X POST -d '{"abc":123,"bcd":"nihao"}' URL
```

> [Linux curl命令最全详解](https://blog.csdn.net/angle_chen123/article/details/120675472)
> [curl其他参数介绍](https://www.cnblogs.com/fan-gx/p/12321351.html)


### Linux系统服务

```shell
# 添加自定义系统服务的目录
# lib/systemd/system真实地址是/usr/lib/system/system地址
/lib/systemd/system
# 软件包安装的单元
/usr/lib/systemd/system/
# 系统管理员安装的单元,优先级更高
/etc/systemd/system/
# 优先级为 /etc/systemd/system->/run/systemd/system->/lib/systemd/system
# 如果同一选项三个地方都配置了，优先级高的会覆盖优先级低的。

# 开机启动执行命令：编辑/etc/rc.d/rc.local
# 添加要执行的命令或者脚本,并且赋予执行权限
chmod +x /etc/rc.d/rc.local
```


### 查看端口

```shell
# 可以查看所有tcp、udp端口开放情况
netstat -antu
# 查看正在运行的端口(t代表tcp 加u查看udp)
netstat -ntlp
# 查看某一端口运行的程序
lsof -i: 9090
# 端口号 查看指定端口被哪个进程占用的情况
netstat -ntulp|grep
# 查找abc进程
ps -ef|grep abc
# 显示所有进程
ps -aux
# 发现A进程占用该端口号
ps -ef|grep A
```

### 设置静态ip后无法连接外网的问题
因为动态ip会自动分配DNS，而静态ip需要手动配置DNS。centos7在/etc/sysconfig/network-scripts/ifcfg-ens33 写入DNS1=114.114.114.114。ubuntu在/etc/resolv.conf写入nameserver 114.114.114.114

> [Linux配置IP地址](https://www.cnblogs.com/adforce/p/3363681.html)

### 命令行更改MAC地址

```shell
/sbin/ifconfig eth0 down
/sbin/ifconfig eth0 hw ether 00:50:56:94:16:a8
/sbin/ifconfig eth0 up
service network restart
```

### 源码安装配置(configure)、编译(make)、安装(make install)

- **./configure**

`./configure`是一个脚本文件，通常用于Unix-like系统上的源代码安装。它的主要作用是根据系统的特性和用户提供的选项来配置软件的构建环境，并生成Makefile文件。`./configure`脚本会检测系统的各种参数，如操作系统类型、编译器版本、库文件的位置等，并确定这些参数是否满足软件的构建要求。同时，它还会检查软件所需的依赖项是否已安装，并确定是否需要额外的配置选项。一旦`./configure`脚本执行完成，它会生成一个适合当前系统的Makefile文件。这个Makefile文件将用于后续的make命令，指导如何编译和安装软件。

`./configure`是一个在源代码安装过程中常见的脚本命令，主要用于准备软件的构建环境。它的主要作用包括：

- 检测环境：`./configure`会检测你的安装平台的目标特征，包括操作系统的类型、版本，以及系统中已安装的库和工具链等。
- 检查依赖：该脚本会检查即将安装的软件所需要的依赖是否已满足。例如，对于使用C语言编写的Unix程序，`./configure`会确保系统中有C编译器，并确定其名称和路径。
- 生成Makefile：根据给定的参数和系统环境，`./configure`会生成一个Makefile文件。这个Makefile文件定义了软件构建和安装过程中需要执行的一系列任务。

需要注意的是，`./configure`只是源代码安装的第一步。一旦配置完成并生成了Makefile，你可以使用`make`命令来执行实际的构建过程，将源代码编译成可执行文件。随后，使用`make install`命令将编译好的文件安装到指定的目录。

此外，`./configure`脚本通常支持各种选项，通过`./configure --help`命令可以查看详细的选项列表。其中一个常见的选项是`--prefix`，它用于配置安装目录。如果不配置该选项，安装后的文件可能会分散到系统的默认目录中。通过指定`--prefix`选项，你可以将所有资源文件集中放置在一个目录下，便于管理和维护。

最后，如果在`./configure`过程中报错，可以尝试删除生成的config.cache相关文件，然后重新运行`./configure`。如果问题依旧存在，可能需要检查系统环境或依赖是否完全满足软件安装的要求。

- **make**

make是一个自动化构建工具，它根据Makefile文件中定义的规则来编译和链接源代码，生成可执行文件或库文件。Makefile文件中包含了构建项目所需的各种指令和依赖关系，`make`会根据这些指令自动执行相应的编译和链接操作。使用`make`的主要好处在于自动化和效率。它可以根据源代码文件的修改情况，只重新编译那些自上次编译以来已经更改过的文件，避免了不必要的重复编译。此外，`make`还支持并行编译，可以显著提高编译速度。

在`make`命令的上下文中，prefix通常是一个变量，用于指定软件安装的目标目录。当执行`make install`时，这个prefix变量决定了可执行文件、库文件、头文件等应该被安装到哪个目录下。默认情况下，prefix通常被设置为/usr/local，这意味着如果不指定其他值，软件将被安装到/usr/local目录下。但是，开发者或用户经常需要改变这个默认的安装位置，尤其是在多用户系统或特定的软件部署环境中。你可以在命令行中通过以下方式设置prefix变量的值：

```bash
make prefix=/path/to/install install
```

或者，在Makefile中直接设置这个变量的值：

```makefile
prefix=/path/to/install
```

然后，当执行`make install`时，Makefile中的安装规则会使用这个prefix变量的值来确定安装路径。例如，如果prefix被设置为/opt/myapp，那么可执行文件可能会被安装到/opt/myapp/bin，库文件到/opt/myapp/lib，头文件到/opt/myapp/include等。这样的设计使得`make`和`make install`命令更加灵活，可以根据不同的需求和环境来定制软件的安装位置。

**make和make install区别**

`make`和`make install`在Linux系统中是构建和安装软件的两个重要步骤，它们的主要区别如下：

- 功能与作用：
make：主要用于编译源代码。它根据Makefile文件中的指令，自动确定大型程序中哪一部分需要重新编译，并调用相应的命令进行编译。通过自动化这一过程，可以大大提高项目开发的效率。
make install：主要用于安装软件。当源代码经过make命令编译后，生成的可执行文件或库文件需要进行安装，以便系统或用户能够正常使用。make install命令会读取Makefile中定义的安装指令，将编译好的文件安装到指定的目录。

- 执行时机：
make：通常在源代码下载并解压后，配置好编译环境（如通过./configure脚本）后执行，用于生成可执行文件或库文件。
make install：在make命令成功执行，生成了所需的文件后执行，用于将这些文件安装到系统中。

- 所需权限：
make：通常不需要特殊权限，除非编译过程中需要访问系统级别的资源或文件。
make install：由于需要将文件安装到系统目录中，因此通常需要root权限（或使用sudo命令）来执行。

- 依赖文件：
这两个命令都依赖于Makefile文件，该文件描述了编译和安装的规则和步骤。

总结来说，make和make install是软件构建和安装过程中的两个关键步骤。make负责编译源代码，而make install则负责将编译好的文件安装到系统中。在开发或安装软件时，通常需要按照这两个步骤来操作。


**使用./configure --prefix后还需要make prefix指定吗**

在使用`./configure --prefix`来指定安装目录后，通常不需要在后续的make命令中再次指定prefix。`./configure --prefix`的作用是在配置软件安装环境时设定一个基础目录，该目录将用于后续的`make install`过程中，决定可执行文件、库文件、配置文件等应被安装到哪个具体的子目录中。一旦`./configure`脚本运行并生成了Makefile，这个prefix的值就已经被Makefile所记录。接下来的make命令会根据Makefile中的指示进行编译，但并不需要（也不应该）再次指定prefix。make会按照Makefile中已经设置好的规则和路径进行工作。最后，当执行make install时，它会依据Makefile中记录的prefix值，将编译好的文件安装到先前通过`./configure --prefix`指定的目录中。因此，简而言之，你不需要在make命令中再次指定prefix，只需要在./configure时设定好即可。

```shell
./configure --prefix=/usr/local/test
# 编译出错时，清除编译生成的文件
make distclean
# 编译安装到指定目录下
make prefix=/usr/local/redis install
# 卸载
make uninstall
```

### 设置keepalived服务开机启动

```shell
chkconfig keepalived on
chkconfig --add name
          --del name
          --list
```

### Ubuntu图形界面允许root登陆

```shell
# 设置root密码，终端会先验证密码 然后在设置root密码
sudo passwd root
vi /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
# 增加两行
# greeter-show-manual-login=true
# all-guest=false #不允许guest用户登陆
cd /etc/pam.d  # 编辑gdm-autologin和gdm-password文件 注释掉auth required pam_succeed_if.so user != root quiet_success
vi /root/.profile # 将mesg n || true 修改为 tty -s && mesg n || true
reboot
```

> [解决Ubuntu的root账号无法登录SSH问题](https://www.cnblogs.com/yixius/articles/6971054.html)


### 查看空间占用

```shell
du -h --max-depth=1 /usr
# 查看指定文件大小
du -sh 文件名
# -a或--all: 显示所有文件和目录的使用情况，而不仅仅是目录。
# -b或--bytes: 以字节为单位显示磁盘使用量。
# -c: 在结果中添加总计。
# -h或--human-readable: 以人类可读的格式（KB, MB, GB）显示磁盘使用量。
# -H或--si: 与-h选项类似，但以1000为换算单位而不是1024。
# -k: 以千字节（KB）为单位显示磁盘使用量。
# -m: 以兆字节（MB）为单位显示磁盘使用量。
# -s或--summarize: 只显示总计，不列出每个文件和目录的使用情况。
# -L: 跟随符号链接。
# -x: 仅在当前文件系统中查找。

# 查看整个磁盘空间
df -h
```

### 查看内存
```shell
free -h

top -n 2 # 表示更新两次后终止更新显示并退出
top -d 3 # 表示更新周期为3秒
# top命令进入展示信息后按大写M表示按照内存大小排序展示 按大写P表示按照CPU使用率进行排序
```

### 读、写、运行

三项权限可以用数字表示，r=4,w=2,x=1 rw-r--r--用数字表示成644
```shell
chmod 754 filename
# 将filename文件的读写运行权限赋予文件所有者，把读和运行的权限赋予群组用户，把读的权限赋予其他用户。
# chmod +x是将文件状态改为可执行，而chmod 777是改变文件读写权限。
```

### centos7卸载openjdk

```shell
rpm -qa|grep jdk  # 查看已有的openjdk -q(query) -a(all)
rpm -ev --nodeps $(上条命令的查询结果) # 卸载

# ubuntu 
apt-get remove openjdk*
```
### rpm参数

```
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

### 从服务器复制文件到本地

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

### ssh双向免密登录服务器A、B

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
> [科普：什么是SSH？](https://mp.weixin.qq.com/s/1e4aGp_cx0E_qCHVuS3GMg)


### centos7/ubuntu通用

```shell
# 默认命令行
systemctl set-default multi-user.target  （init 3）
# 默认图形页面
systemctl set-default graphical.target  (init 5)
```

### 查看系统内核及版本

```shell
# 查看内核
uname -r
# 查看centos版本
cat /etc/centos-release
cat /etc/os-release
cat /etc/redhat-release
# 全部内核
rpm -qa | grep kernel
yum list installed | grep kernel
#删除多余内核
yum remove kernel-
```

### 测试端口是否开通

```shell
ssh -v -p port username@ip
# -v 调试模式(会打印日志) -p 指定端口 username可以随意
# 使用 telnet 命令
telnet ip 端口
# 使用nc命令
nc -vu ip 端口
# -v 输出交互或出错信息，新手调试时尤为有用,-u指定nc使用UDP协议，默认为TCP
```

### telnet

格式: telnet 选项 主机名 端口号
选项如下：

```
-8：允许使用8位字符资料，包括输入与输出。
-a：尝试自动登入远端系统。
-b：使用别名指定远端主机名称。
-c：不读取用户专属目录里的.telnetrc文件。
-d：启动排错模式。
-e：设置脱离字符。
-l：指定要登入远端主机的用户名称。
-x：假设主机有支持数据加密的功能，就使用它。
```

### tracerouter

```sh
traceroute ip

traceroute -p port ip
traceroute -p 443 example.com # 测试目标主机的443端口

tracert ip
# -n：不解析主机名，只显示IP地址。这有助于加快追踪速度
# -m或-h：设置最大跳数（默认是30跳）。
# -w或-t：设置每次探测的等待时间（超时时间，默认是几秒）。
# -q：指定每个跃点查询的数量（默认是3个）。
# -I：使用ICMP回显请求而不是UDP数据包（仅适用于Linux上的traceroute）。
# -T：使用TCP SYN数据包进行追踪（仅适用于Linux上的traceroute）。

```

### Linux交换空间(swap space)

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

### 挂载
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


### source filename与sh filename及./filename执行脚本的区别

1. 当shell脚本具有可执行权限时，用`sh filename`与`./filename`执行脚本是没有区别的。`./filename`是因为当前目录没有在PATH中，所有”.”是用来表示当前目录的。
2. `sh filename`重新建立⼀个子shell，在子shell中执行脚本里面的语句，该子shell继承父shell的环境变量，但子shell新建的、改变的变量不会被带回父shell，除非使用export。
3. `source filename`这个命令其实只是简单地读取脚本里面的语句依次在当前shell里面执行，没有建立新的子shell。那么脚本里面所有新建、改变变量的语句都会保存在当前shell里面


## 环境变量

### 环境变量文件

- /etc/profile：此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行，并从/etc/profile.d目录的配置文件中收集shell的设置
- /etc/bashrc：为每一个运行bash shell的用户执行此文件，当bash shell被打开时，该文件被读取也就是说，当用户shell执行了bash时，运行这个文件；
- ~/.bash_profile：每个用户都可使用该文件输入专用于自己使用的shell信息，当用户登录时，该文件仅仅执行一次！默认情况下，它设置一些环境变量，执行用户的.bashrc文件
- ~/.bashrc：该文件包含用于你的bash shell的bash信息，当登录时以及每次打开新的shell时，该文件被读取。该文件存储的是专属于个人bash shell的信息，当登录时以及每次打开一个新的shell时，执行这个文件。在这个文件里可以自定义用户专属的个人信息。
- \~/.bash_logout：当每次退出系统(退出bash shell)时，执行该文件；另外，/etc/profile中设定的变量(全局)的可以作用于任何用户，而\~/.bashrc等中设定的变量(局部)只能继承/etc/profile中的变量，他们是“父子”关系；

### Ubuntu下bash的几个初始化文件
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

> [Linux环境变量配置的6种方法](https://mp.weixin.qq.com/s/D32FE4CS2pT89czVUv0SOg)

## 相关文章

| [Linux基础知识总结](https://javaguide.cn/cs-basics/operating-system/linux-intro.html) | [程序员必备的150个Linux命令！](https://mp.weixin.qq.com/s/wejhjmbtS16AdlDo0JiLMA) | [新人必备的Linux命令！](https://mp.weixin.qq.com/s/TvUcd7mLkHxD_botoo7BQg) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [1000+常用的Linux命令来袭](https://mp.weixin.qq.com/s/43JCaDehv0nbzDKbe0S8Kg) | [作为一名Java开发人员，应该从多大程度上掌握Linux？](https://www.zhihu.com/question/319076999/answer/672319588?utm_source=wechat_timeline&utm_medium=social&utm_oi=1040923520439672832&from=timeline) | [30个实例详解TOP命令](https://mp.weixin.qq.com/s/2FDa8fOm6x4-1MP9l9N7vA) |
| [ping命令的7大用法,看完秒变大神！](https://mp.weixin.qq.com/s/liiz58PC1JlfPaEW81wUpg) | [听说你ping用的很6,给我图解一下ping的工作原理](https://mp.weixin.qq.com/s/4pPdcSak3YLyqiz4Ns8htQ) | [ping命令还能这么玩？](https://mp.weixin.qq.com/s/fLY3JH5_4_h4vABV7B-4uw) |
| [后端线上问题排查常用命令收藏](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247494347&idx=1&sn=2f6bfe4a5716d4b0aba6d8b5348f7033&source=41#wechat_redirect) | [Java开发常用的Linux命令知识积累](https://mp.weixin.qq.com/s/jlmPjqFTfP4JNmbhpGoHsA) | [这篇Linux总结的真棒！](https://mp.weixin.qq.com/s/F_0kiOWgQS4BINx2t2Rqtw) |
| [Linux远程桌面管理工具！功能真心强大](https://mp.weixin.qq.com/s/UT3RYrUGP2E5JQbf1Z8c8w) | [在Linux上保护SSH服务器连接的8种方法](https://mp.weixin.qq.com/s/kxvc95RJoSKqcXwwmyrPKg) | [发现谁用kill -9关闭程序就开除！](https://mp.weixin.qq.com/s/zmfCC083VBrA8P6uWHL2DQ) |
| [万字详解Linux常用指令](https://mp.weixin.qq.com/s/hXDAZD0LUJ9g85t6fkBR-Q) | [最强Linux命令总结（特别推荐版）](https://mp.weixin.qq.com/s/KN9OPZMhmuA6rhx5YTjwDA) | [Linux命令大全搜索工具，内容包含Linux命令手册、详解、学习、搜集](https://github.com/jaywcjlove/linux-command) |