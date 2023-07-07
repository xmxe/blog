---
title: Docker
categories: 技术栈
tags: 安装
index_img: /assert/docker.jpg
img: https://pic2.zhimg.com/v2-98bbd70b053dd779240634a00c7f0950_1440w.jpg

---

## 安装

### 在线安装

(1) 安装docker需要关闭selinux,由于selinux和LXC（Docker实现虚拟化的方式）有冲突，所以需要禁用selinux。编辑/etc/selinux/config，设置两个关键变量。

```shell
SELINUX=disabled

SELINUXTYPE=targeted
```

(2) 关闭防火墙

```shell
systemctl stop firewalld
```

(3) 安装容器

```shell
yum -y install docker-ce
```

(4) 启动服务

```shell
systemctl start docker
```

(5) 测试容器

```shell
docker run hello-world
# PS: centos7安装命令 yum -y install docker-ce | ubuntu安装命令 apt install docker-ce
```

### 离线安装

(1) [下载离线包](https://download.docker.com/linux/static/stable/x86_64/)

(2) 解压

```shell
tar -xvf docker-18.06.1-ce.tgz
```

(3) 将解压出来的docker文件内容移动到/usr/bin/目录下

```shell
cp docker/* /usr/bin/
```

(4) 将docker注册为service

```shell
# vim /etc/systemd/system/docker.service

# lib/systemd/system、/usr/lib/systemd/system、/etc/systemd/system都可以，lib/systemd/system真实地址是/usr/lib/system/system地址，
# /usr/lib/systemd/system/ 软件包安装的单元
# /etc/systemd/system/ 系统管理员安装的单元,优先级更高
# 优先级为 /etc/systemd/system，/run/systemd/system，/lib/systemd/system
# 如果同一选项三个地方都配置了，优先级高的会覆盖优先级低的。

[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]

Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker

ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

ExecReload=/bin/kill -s HUP $MAINPID

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.

LimitNOFILE=infinity

LimitNPROC=infinity

LimitCORE=infinity

# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
# TasksMax=infinity

TimeoutStartSec=0

# set delegate yes so that systemd does not reset the cgroups of docker containers

Delegate=yes

# kill only the docker process, not all processes in the cgroup

KillMode=process

# restart the docker process if it exits prematurely

Restart=on-failure

StartLimitBurst=3

StartLimitInterval=60s


[Install]

WantedBy=multi-user.target
```

(5) 添加文件权限并启动docker

```shell
chmod +x /etc/systemd/system/docker.service
```

(6) 重载unit配置文件

```shell
systemctl daemon-reload
```

(7) 启动Docker

```shell
systemctl start docker
```

(8) 设置开机自启

```shell
systemctl enable docker.service
```

(9) 查看Docker状态

```shell
systemctl status docker
```

(10) 查看Docker版本

```shell
docker -v
```

## 常用命令

### 镜像

- 查看镜像
```shell
docker images
-a :列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；
--digests :显示镜像的摘要信息；
-f :显示满足条件的镜像；
--format :指定返回值的模板文件；
--no-trunc :显示完整的镜像信息；
-q :只显示镜像ID。
```

- 拉取镜像
```shell
docker pull name:tag
```

- 推送镜像
```shell
docker push myapache:v1
```

- 导出镜像
```shell
docker save -o <保存路径> <镜像名称:标签>

docker save -o ./ubuntu18.tar ubuntu:18.04
```

- 导入镜像
```shell
docker load -i 文件名 或者docker load --input 文件名

docker load --input ./ubuntu18.tar
```

- 删除镜像
```shell
docker rmi images_id
```

- 删除所有镜像
```shell
docker rmi `docker images -q`
```

- 搜索镜像
```shell
docker search *
```

- 查看指定镜像的创建历史
```shell
docker history [OPTIONS] IMAGE

OPTIONS说明：
-H :以可读的格式打印镜像大小和日期，默认为true；
--no-trunc :显示完整的提交记录；
-q :仅列出提交记录ID。
```

### 容器

- 查看正在运行的容器
```shell
docker ps 或者docker container ls
```

- 查看所有容器
```shell
docker ps -a 或者 docker container ls -a
```

- 导出容器
```shell
docker export <容器名> > <保存路径>

或者docker export -o <容器名> <保存路径> -o :将输入内容写到文件。
docker export ubuntu18 > ./ubuntu18.tar
将id为a404c6c174a2的容器按日期保存为tar文件
docker export -o mysql-`date +%Y%m%d`.tar a404c6c174a2
```

- 导入容器
```shell
docker import <文件路径> <容器名>
docker import ./ubuntu18.tar ubuntu18
```

- 删除容器
```shell
docker rm [OPTIONS] container_id

OPTIONS说明：
-f :通过 SIGKILL 信号强制删除一个运行中的容器。
-l :移除容器间的网络连接，而非容器本身。
-v :删除与容器关联的卷。
```

- 删除所有容器
```shell
docker rm $(docker ps -a -q) 
或者
docker rm `docker ps -a -q`
```

- 启动容器
```shell
docker container start container_id
```

- 停止所有容器
```shell
docker stop $(docker ps -a -q)
```

- 杀掉运行中的容器
```shell
docker kill -s(可忽略) CONTAINER
-s :向容器发送一个信号 
例：docker kill -s KILL mynginx
```

- 在运行的容器中执行命令
```shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
例如：进入容器 docker exec -itd 容器id /bin/bash
（-d :分离模式: 在后台运行 -i :即使没有附加也保持STDIN 打开-t :分配一个伪终端）
/bin/bash：在container中启动一个bash shell
exit 退出bash shell
```

- 暂停容器中所有的进程
```shell
docker pause container_id
```

- 恢复容器中所有的进程
```shell
docker unpause container_id
```

- 创建一个新的容器不运行
```shell
docker create 参数同docker run
```

- 创建一个新的容器并运行
```shell
docker run
-i: 以交互模式运行容器，通常与-t同时使用
-t: 为容器重新分配一个伪输入终端，通常与-i同时使用
-it 以交互模式运行
-P: 随机端口映射，容器内部端口随机映射到主机的端口
-p: 指定端口映射，格式为：主机(宿主)端口:容器端口
-d 后台运行 并返回容器ID
-v,--volume 挂载 主机目录:容器目录,绑定一个卷
-u,--user=""， 指定容器的用户
-a,--attach=[]， 登录容器（必须是以docker run -d启动的容器）
-w,--workdir=""， 指定容器的工作目录
-c,--cpu-shares=0， 设置容器CPU权重，在CPU共享场景使用
-e username="ritchie",--env=[] 设置环境变量容器中可以使用该环境变量
-m,--memory=""， 指定容器的内存上限
-P,--publish-all=false，指定容器暴露的端口
-h,--hostname=""， 指定容器的主机名
--name=”” 容器命名
--cap-add=[]添加权限，权限清单详见：	http://linux.die.net/man/7/capabilities
--cap-drop=[]删除权限，权限清单详见：	http://linux.die.net/man/7/capabilities
--cidfile="" 运行容器后,在指定文件中写入容器PID值，一种典型的监控系	统用法
--cpuset=""， 设置容器可以使用哪些CPU，此参数可以用来容器独占CPU
--device=[]， 添加主机设备给容器，相当于设备直通
--dns=[]， 指定容器的dns服务器
--dns-search=[]指定容器的dns搜索域名,写入到容器的/etc/resolv.conf文	件
--entrypoint=""， 覆盖image的入口点
--env-file=[]， 从指定文件读入环境变量
--expose=[]， 指定容器暴露的端口，即修改镜像的暴露端口
--link=[]， 添加链接到另一个容器，使用其他容器的IP、env等信息
--lxc-conf=[]， 指定容器的配置文件，只有在指定--exec-driver=lxc时	使用
--net="bridge"， 容器网络设置:
bridge 使用docker daemon指定的网桥
host 容器使用主机的网络
container:NAME_or_ID > 使用其他容器的网路，共享IP和PORT等网络资源
none 容器使用自己的网络（类似--net=bridge），但是不进行配置
--privileged=false指定容器是否为特权容器,特权容器拥有所有的capabilities
--restart="no"，指定容器停止后的重启策略:
no - 容器退出时不重启
on-failure - 只在容器以非0状态码退出时重启。可选的，可以退出docker daemon尝试重启容器的次数
always – 不管退出状态码是什么始终重启容器。当指定always时，docker daemon将无限次数地重启容器。容器也会在daemon启动时尝试重启，不管容器当时的状态如何。
unless-stopped – 不管退出状态码是什么始终重启容器，不过当daemon启动时，如果容器之前已经为停止状态，不要尝试启动它。
--rm=false指定容器停止后自动删除容器(不支持以docker run -d启动的容器)
--sig-proxy=true 设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和	SIGKILL不能被代理
```

- 获取容器/镜像的元数据
```shell
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
OPTIONS说明：
-f :指定返回值的模板文件。
-s :显示总的文件大小。
--type :为指定类型返回JSON。
```

- 查看容器中运行的进程信息
```shell
docker top container_id
```

- 连接到正在运行中的容器
```shell
docker attach container_id
```

- 阻塞运行直到容器停止，然后打印出它的退出代码
```shell
docker wait containser_id
```

### 其他

- 从服务器获取实时事件
```shell
docker events OPTIONS

OPTIONS说明：
-f ：根据条件过滤事件；
--since ：从指定的时间戳后显示所有事件;
--until ：流水时间显示到指定的时间为止；
docker events --since="1467302400"
```

- 查看日志
```shell
docker logs [OPTIONS] CONTAINER
--details 显示更多的信息
-f, --follow 跟踪实时日志
--since string 显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
--tail string 从日志末尾显示多少行日志，默认是all
-t, --timestamps 显示时间戳
--until string 显示自某个timestamp之前的日志，或相对时间，如42m（即42分钟）

查看指定时间后的日志，只显示最后100行：
docker logs -f -t --since="2018-02-08" --tail=100 CONTAINER_ID
查看最近30分钟的日志:
docker logs --since 30m CONTAINER_ID
查看某时间之后的日志：
docker logs -t --since="2018-02-08T13:23:37" CONTAINER_ID
查看某时间段日志：
docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" CONTAINER_ID
```

- 列出指定的容器的端口映射
```shell
docker port container_id
```

- 提交
```shell
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

OPTIONS说明：
-a :提交的镜像作者；
-c :使用Dockerfile指令来创建镜像；
-m :提交时的说明文字；
-p :在commit时，将容器暂停。

将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。
docker commit -a "runoob.com" -m "my apache" a404c6c174a2 mymysql:v1
```

- 容器与主机之间的数据拷贝
```shell
将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。
docker cp /www/runoob 96f7f14e99ab:/www/
```

- 查看容器文件结构更改
```shell
docker diff mymysql
```

- 使用 Dockerfile 创建镜像
```shell
docker build
--build-arg=[] :设置镜像创建时的变量；
--cpu-shares :设置 cpu 使用权重；
--cpu-period :限制 CPU CFS周期；
--cpu-quota :限制 CPU CFS配额；
--cpuset-cpus :指定使用的CPU id；
--cpuset-mems :指定使用的内存 id；
--disable-content-trust :忽略校验，默认开启；
-f :指定要使用的Dockerfile路径；
--force-rm :设置镜像过程中删除中间容器；
--isolation :使用容器隔离技术；
--label=[] :设置镜像使用的元数据；
-m :设置内存最大值；
--memory-swap :设置Swap的最大值为内存+swap，"-1"表示不限swap；
--no-cache :创建镜像的过程不使用缓存；
--pull :尝试去更新镜像的新版本；
--quiet, -q :安静模式，成功后只输出镜像 ID；
--rm :设置镜像成功后删除中间容器；
--shm-size :设置/dev/shm的大小，默认值是64M；
--ulimit :Ulimit配置。
--squash :将Dockerfile中所有的操作压缩为一层。
--tag, -t: 镜像的名字及标签，通常name:tag或者name格式；可以在一次构建中为一个镜像设置多个标签。
--network: 默认default。在构建期间设置RUN指令的网络模式
```

- 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub
```shell
docker login -u -p
-u 登陆的用户名 -p :登陆的密码
```

- 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库Docker Hub
```shell
docker logout
```

- 标记本地镜像，将其归入某一仓库
```shell
docker tag
```

## docker加速命令

```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
或
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}

```
> 其他站点
> http://hub-mirror.c.163.com
> https://3laho3y3.mirror.aliyuncs.com
> http://f1361db2.m.daocloud.io
> https://mirror.ccs.tencentyun.com
> https://docker.mirrors.ustc.edu.cn

## 与Spring Boot

- [一键部署Spring Boot到远程Docker容器](https://mp.weixin.qq.com/s/15ZAVUg5DfcF53QpEetT7Q)
- [Jenkins+Docker一键自动化部署SpringBoot项目](https://mp.weixin.qq.com/s/dP-c3twzR0PMUvPWZA-U0Q)
- [搭建SpringBoot项目并将其Docker化](https://mp.weixin.qq.com/s/CXUwpTbAVoXEeB7EcrCjAw)
- [SpringBoot使用Docker快速部署项目](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493762&idx=1&sn=114663a4a13ba5bb27d05e0d77de37c1&source=41#wechat_redirect)
- [Docker部署Spring Boot项目的2种方式！](https://mp.weixin.qq.com/s/du2sypGQczJh7gQz_4IX9g)
- [SpringBoot项目构建Docker镜像深度调优](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493962&idx=1&sn=af6c945d629003cfd30564698c017598&source=41#wechat_redirect)
- [还在手动部署springboot项目？不妨试试它，让你部署项目飞起来！](https://mp.weixin.qq.com/s/01SZo3NNf5zuAC8wAI6C-g)
- [Docker+Spring Boot+FastDFS搭建一套分布式文件服务器，太强了！](https://mp.weixin.qq.com/s/HSRIYQVKR9TGtwetd3LU5w)


## 相关文章

- [图解Docker架构，傻瓜都能看懂！](https://mp.weixin.qq.com/s/ELZo2z4fHonoBGXQI0M9CA)
- [构建Java镜像的10个最佳实践](https://mp.weixin.qq.com/s/gmZDBuYDXnNdykEx66Y0Cw)
- [10个冷门但又非常实用的Docker使用技巧！！](https://mp.weixin.qq.com/s/LOmqsoBJd7h1HPwf0i1uwQ)
- [Docker实战总结](https://mp.weixin.qq.com/s/tTsizeLeVyvQ44GXMNqrjA)
- [Docker从入门到干活，看这一篇足矣](https://mp.weixin.qq.com/s/t81enr-ypBxk1K4lYqWZww)
- [如何编写最佳的Dockerfile](https://mp.weixin.qq.com/s/x-M5iKvvuseIQwUdVmxSPQ)
- [Docker：Docker Compose详解](https://www.jianshu.com/p/658911a8cff3)
- [CentOS/Ubuntu安装Docker和Docker Compose](https://mp.weixin.qq.com/s/fB59zXK7cPBt1asSyUpqDg)
- [DaoCloud安装docker指南](http://guide.daocloud.io/dcs/docker-9152677.html)
- [Docker常用命令，还有谁不会？](https://mp.weixin.qq.com/s/fzlNnJe9SMA5k3TDXOfZUA)
- [一款吊炸天的Docker图形化工具，太强大](https://mp.weixin.qq.com/s/PpI7_fY5ACjmtmnlqr7ZMQ)
- [5款顶级Docker容器GUI管理工具！免费又好用](https://mp.weixin.qq.com/s/w0sFaHApOSrwgva0886ijQ)
- [Docker轻量级编排创建工具Humpback](https://mp.weixin.qq.com/s/rAOsia2LU2_Fl4vrjQ2tvA)
- [带着问题学Kubernetes架构！](https://mp.weixin.qq.com/s/6smzsvYSbRvSPcpbfnH98A)
- [为什么大家都在学习k8s](https://mp.weixin.qq.com/s/B2tIs6YitA93iYxEZ_8Ovw)
- [Kuboard-Kubernetes多集群管理界面](https://kuboard.cn/)
- [图文详解Kubernetes，傻瓜都能看懂！](https://mp.weixin.qq.com/s/WWRp-e9QPcLg8-m-V3UU1Q)
- [Kubernetes缺少的多租户功能，你可以通过这些方式实现](https://mp.weixin.qq.com/s/8UJnsx0NJyxlKXeduhg5Yw)
- [IDEA使用Docker插件，实现一键自动化部署](https://mp.weixin.qq.com/s/yg5ACCeeyJa0AVP1LatUhA)
- [Docker有几种网络模式](https://mp.weixin.qq.com/s/KU3bpxiNbHGJQ_XVRqsedg)
