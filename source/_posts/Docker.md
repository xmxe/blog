---
title: Docker
categories: 技术栈
tags: 安装
index_img: /assert/docker.jpg
img: https://pic2.zhimg.com/v2-98bbd70b053dd779240634a00c7f0950_1440w.jpg

---

## 基本概念

### Docker架构

Docker架构包括以下几个核心组件：

1. Docker客户端(Client):用户与Docker交互的命令行工具或API。

2. Docker服务器(Server):负责管理镜像、容器、网络等资源的后台服务。

3. Docker镜像(Image):是一个只读的模板，包含用于创建容器的文件系统和应用程序代码。

4. Docker容器(Container):是Docker的基本执行单元，一个镜像可以创建多个容器，容器之间互相隔离，包括文件系统、网络、进程等资源。

5. Docker Registry(仓库):存储Docker镜像的中央仓库，提供镜像的下载和上传服务。

6. Docker Compose:是一个用于定义和运行多个容器的工具，简化了容器编排的复杂度。

7. Docker Swarm:是Docker的集群管理工具，可以将多个Docker主机组成一个虚拟的Docker主机集群，提供负载均衡、容错等功能。


这些组件协同工作，构成了Docker强大的应用容器化解决方案

### Docker隔离原理

Docker通过多种技术实现容器的隔离，包括：

1. 命名空间(Namespace)：Docker使用多种命名空间，如mount、pid、net、ipc、uts等，将容器的进程、网络、文件系统等资源与主机分离，使得容器拥有自己独立的运行环境。

2. 控制组(Cgroups)：Docker使用Cgroups控制组技术，限制容器内部进程使用的资源，如CPU、内存、磁盘等。

3. 文件系统：Docker使用OverlayFS技术，将容器的文件系统与主机分离，每个容器都有自己独立的文件系统，并可以使用Docker镜像中的文件系统层，实现镜像共享和快速启动。

4. 安全机制：Docker使用安全机制，如seccomp、AppArmor、SELinux等，限制容器内部进程的系统调用和权限，防止容器被攻击和滥用。

5. 网络隔离：Docker使用网络隔离技术，将容器的网络与主机分离，每个容器都有自己独立的网络命名空间和IP地址，实现容器之间的隔离和互通。

通过这些技术的组合，Docker实现了容器之间的隔离，使得容器可以在相互独立的环境中运行，同时也保障了容器的安全和稳定性。


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
# 如果有旧版本的话先移除旧版本
sudo yum remove docker*
# 更新yum源
yum -y update
# 加软件源
# sudo yum install -y yum-utils
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 更新缓存
yum makecache fast
# 安装
yum -y install docker-ce
# sudo yum install docker-ce docker-ce-cli containerd.io
```

**安装指定版本的docker image**

```shell
#找到所有可用docker版本列表
yum list docker-ce --showduplicates | sort -r
# 安装指定版本，用上面的版本号替换<VERSION_STRING>
sudo yum install docker-ce-<VERSION_STRING>.x86_64 docker-ce-cli-
<VERSION_STRING>.x86_64 containerd.io
#例如：
#yum install docker-ce-3:20.10.5-3.el7.x86_64 docker-ce-cli-3:20.10.5-3.el7.x86_64 containerd.io
#注意加上.x86_64大版本号
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

> [centos rpm版本](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)
> `rpm -ivh xxx.rpm`
> 可以下载tar,解压启动即可
> [官方文档](https://docs.docker.com/engine/install/binaries/#install-daemon-and-client-binaries-on-linux)

(2) 解压

```shell
tar -zxvf docker-18.06.1-ce.tgz
```

(3) 将解压出来的docker文件内容移动到/usr/bin/目录下

```shell
cp docker/* /usr/bin/
```

(4) 将docker注册为service
```
vim /etc/systemd/system/docker.service
```
编辑docker.service
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

ExecStart=/usr/bin/dockerd
# ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

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

## 卸载

**CentOS上卸载**

```shell
# 停止所有运行的容器
sudo docker stop $(docker ps -aq)
# 删除所有容器
sudo docker rm $(docker ps -aq)
# 删除所有镜像
sudo docker rmi $(docker images -q)
sudo systemctl stop docker
sudo yum remove docker-ce docker-ce-cli containerd.io
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine
# 删除Docker数据目录
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
sudo rm -rf /etc/systemd/system/docker.service.d
sudo rm -rf /etc/systemd/system/docker.service
sudo rm -rf /var/run/docker
sudo rm -rf /usr/local/docker
sudo rm -rf /etc/docker
sudo rm -rf /usr/bin/docker* /usr/bin/containerd* /usr/bin/runc /usr/bin/ctr

```


## 常用命令

### docker images

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

### docker pull

- 拉取镜像
```shell
docker pull name:tag
```

### docker push

- 推送镜像
```shell
docker push myapache:v1
```

### docker save

- 导出镜像
```shell
docker save -o <保存路径> <镜像名称:标签>(或者镜像id)
docker save -o ./ubuntu18.tar ubuntu:18.04
# docker save 仓库名称 > 本地镜像文件名称.tar
```

### docker load

- 导入镜像
```shell
docker load -i 文件名
# 或者
docker load --input 文件名
docker load --input ./ubuntu18.tar
docker load < 本地镜像.tar
```

### docker rmi

- 删除镜像
```shell
docker rmi images_id
```

- 删除所有镜像
```shell
docker rmi `docker images -q`
```

### docker rm

- 删除容器
```shell
docker rm [OPTIONS] container_id

OPTIONS说明:
-f: 通过 SIGKILL 信号强制删除一个运行中的容器。
-l: 移除容器间的网络连接，而非容器本身。
-v: 删除与容器关联的卷。
```

- 删除所有容器
```shell
docker rm $(docker ps -a -q) 
或者
docker rm `docker ps -a -q`
```

### docker search

- 搜索镜像
```shell
docker search *
```

### docker history

- 查看指定镜像的创建历史
```shell
docker history [OPTIONS] IMAGE

OPTIONS说明:
-H :以可读的格式打印镜像大小和日期，默认为true；
--no-trunc :显示完整的提交记录；
-q :仅列出提交记录ID。
```

### docker ps

- 查看正在运行的容器
```shell
docker ps
# 或者
docker container ls
```

- 查看所有容器
```shell
docker ps -a
# 或者
docker container ls -a
```

### docker export

- 导出容器
```shell
docker export <容器名> > <保存路径>
docker export ubuntu18 > ./ubuntu18.tar
# 或者
docker export -o <容器名> <保存路径> # -o :将输入内容写到文件。
# 将id为a404c6c174a2的容器按日期保存为tar文件
docker export -o mysql-`date +%Y%m%d`.tar a404c6c174a2
```

### docker import

- 导入容器
```shell
docker import <文件路径> <容器名>
docker import ./ubuntu18.tar ubuntu18
```

### docker (container) start

- 启动容器
```shell
docker container start container_id
```

### docker stop

- 停止所有容器
```shell
docker stop $(docker ps -a -q)
```

### docker kill

- 杀掉运行中的容器
```shell
docker kill -s(可忽略) CONTAINER
# -s :向容器发送一个信号 例:
docker kill -s KILL mynginx
```

### docker exec

- 在运行的容器中执行命令
```shell
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
# 例如：进入容器
docker exec -it 容器id /bin/bash
# -i:即使没有附加也保持STDIN打开 -t:分配一个伪终端
# /bin/bash: 在container中启动一个bash shell。exit:退出bash shell
```

### docker pause

- 暂停容器中所有的进程
```shell
docker pause container_id
```

### docker unpause

- 恢复容器中所有的进程
```shell
docker unpause container_id
```

### docker create

- 创建一个新的容器不运行
```shell
docker create # 参数同docker run
```

### docker run

- 创建一个新的容器并运行
```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...].其中，OPTIONS是一系列用于配置容器运行方式的选项，IMAGE是要运行的容器镜像，COMMAND是在容器内部执行的命令，ARG是传递给命令的参数。
OPTIONS:
-i: 以交互模式运行容器，通常与-t同时使用
-t: 为容器重新分配一个伪输入终端，通常与-i同时使用
-it 以交互模式运行
-P: 随机端口映射，容器内部端口随机映射到主机的端口
-p: 指定端口映射，格式为主机(宿主)端口:容器端口
-d: 后台运行并返回容器ID
-v,--volume: 挂载主机目录:容器目录,绑定一个卷
-u,--user="": 指定容器的用户
-a,--attach=[]: 登录容器（必须是以docker run -d启动的容器）
-w,--workdir="": 指定容器的工作目录
-c,--cpu-shares=0: 设置容器CPU权重,在CPU共享场景使用
-e username="ritchie",--env=[]: 设置环境变量容器中可以使用该环境变量
-m,--memory="": 指定容器的内存上限
-P,--publish-all=false: 指定容器暴露的端口
-h,--hostname="": 指定容器的主机名
--name=””: 容器命名
--cap-add=[]: 添加权限,权限清单详见:http://linux.die.net/man/7/capabilities
--cap-drop=[]: 删除权限,权限清单详见:http://linux.die.net/man/7/capabilities
--cidfile="": 运行容器后,在指定文件中写入容器PID值,一种典型的监控系统用法
--cpuset="": 设置容器可以使用哪些CPU，此参数可以用来容器独占CPU
--device=[]: 添加主机设备给容器，相当于设备直通
--dns=[]: 指定容器的dns服务器
--dns-search=[]: 指定容器的dns搜索域名,写入到容器的/etc/resolv.conf文件
--entrypoint="": 覆盖image的入口点
--env-file=[]: 从指定文件读入环境变量
--expose=[]: 指定容器暴露的端口,即修改镜像的暴露端口
--link=[]: 添加链接到另一个容器,使用其他容器的IP、env等信息
--lxc-conf=[]: 指定容器的配置文件,只有在指定`--exec-driver=lxc`时使用
--net=bridge(或--network=bridge): 容器网络设置。bridge:使用docker daemon指定的网桥。host:容器使用主机的网络。container:NAME_or_ID:使用其他容器的网路，共享IP和PORT等网络资源。none:容器使用自己的网络（类似`--net=bridge`），但是不进行配置
--privileged=false: 指定容器是否为特权容器,特权容器拥有所有的capabilities(能力),它赋予了容器几乎与宿主机相同的权限
--restart="no": 指定容器停止后的重启策略。no:容器退出时不重启。on-failure:只在容器以非0状态码退出时重启，可选的，可以退出docker daemon尝试重启容器的次数。on-failure:3，在容器非正常退出时重启容器，最多重启3次。always:不管退出状态码是什么始终重启容器，当指定always时，docker daemon将无限次数地重启容器，容器也会在daemon启动时尝试重启，不管容器当时的状态如何。unless-stopped:不管退出状态码是什么始终重启容器，不过当daemon启动时，如果容器之前已经为停止状态，不要尝试启动它。
--rm: 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)
--sig-proxy=true: 设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理
--log-driver=json-file --log-opt max-size=10m --log-opt max-file=3: 使用--log-opt参数限制容器日志大小。在启动容器时,可以使用--log-opt参数设置日志驱动程序的选项.在这个例子中,我们设置了以下选项:max-size=10m:限制单个日志文件的最大大小为10MB.max-file=3:限制日志文件的数量为3个.当日志文件达到10MB时，Docker会自动轮换日志文件，保留最新的3个日志文件。
```

### docker inspect

- 获取容器/镜像的元数据
```shell
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
OPTIONS说明:
-f: 指定返回值的模板文件。
-s: 显示总的文件大小。
--type: 为指定类型返回JSON。
```

### docker top

- 查看容器中运行的进程信息
```shell
docker top container_id
```

### docker attach

- 连接到正在运行中的容器
```shell
docker attach container_id
```

### docker wait

- 阻塞运行直到容器停止，然后打印出它的退出代码
```shell
docker wait containser_id
```

### docker events

- 从服务器获取实时事件
```shell
docker events OPTIONS

OPTIONS说明:
-f: 根据条件过滤事件
--since: 从指定的时间戳后显示所有事件
--until: 流水时间显示到指定的时间为止
docker events --since="1467302400"
```

### docker logs

- 查看日志
```shell
docker logs [OPTIONS] CONTAINER
OPTIONS说明:
--details: 显示更多的信息
-f, --follow: 跟踪实时日志
--since string: 显示自某个timestamp之后的日志，或相对时间，如42m（即42分钟）
--tail string: 从日志末尾显示多少行日志，默认是all
-t, --timestamps: 显示时间戳
--until string: 显示自某个timestamp之前的日志，或相对时间，如42m（即42分钟）

# 查看指定时间后的日志，只显示最后100行：
docker logs -f -t --since="2018-02-08" --tail=100 CONTAINER_ID
# 查看最近30分钟的日志:
docker logs --since 30m CONTAINER_ID
# 查看某时间之后的日志：
docker logs -t --since="2018-02-08T13:23:37" CONTAINER_ID
# 查看某时间段日志：
docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" CONTAINER_ID
```

### docker port

- 列出指定的容器的端口映射
```shell
docker port container_id
```

### docker commit

- 提交
```shell
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

OPTIONS说明:
-a: 提交的镜像作者；
-c: 使用Dockerfile指令来创建镜像；
-m: 提交时的说明文字；
-p: 在commit时，将容器暂停。

# 将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。
docker commit -a "runoob.com" -m "my apache" a404c6c174a2 mymysql:v1
```

### docker cp

- 容器与主机之间的数据拷贝
```shell
# 将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下
docker cp /www/runoob 96f7f14e99ab:/www/
```

### docker diff

- 查看容器文件结构更改
```shell
docker diff mymysql
```

### docker build

- 使用 Dockerfile 创建镜像
```shell
docker build
--build-arg=[]: 设置镜像创建时的变量
--cpu-shares: 设置cpu使用权重
--cpu-period: 限制CPU CFS周期
--cpu-quota: 限制CPU CFS配额
--cpuset-cpus: 指定使用的CPU id
--cpuset-mems: 指定使用的内存id
--disable-content-trust: 忽略校验，默认开启
-f: 指定要使用的Dockerfile路径
--force-rm: 设置镜像过程中删除中间容器
--isolation: 使用容器隔离技术
--label=[]: 设置镜像使用的元数据
-m: 设置内存最大值
--memory-swap: 设置Swap的最大值为内存+swap，"-1"表示不限swap
--no-cache: 创建镜像的过程不使用缓存
--pull: 尝试去更新镜像的新版本
--quiet, -q: 安静模式，成功后只输出镜像ID
--rm: 设置镜像成功后删除中间容器
--shm-size: 设置/dev/shm的大小，默认值是64M
--ulimit: Ulimit配置
--squash: 将Dockerfile中所有的操作压缩为一层。
--tag, -t: 镜像的名字及标签，通常name:tag或者name格式；可以在一次构建中为一个镜像设置多个标签。
--network: 默认default。在构建期间设置RUN指令的网络模式
```
`docker build`最后参数表示Docker上下文的路径。当你执行`docker build`命令并指定一个Dockerfile时，Docker会将这个路径下的所有文件和目录发送到Docker守护进程，以便它可以在构建镜像时使用这些文件。如果你在命令行中看到一个点作为`docker build`命令的最后一个参数，即`docker build .`通常是因为用户想要在当前目录下执行构建，并且没有明确指定一个不同的路径。这里的点（.）表示当前目录，Dockerfile将在这个目录下查找任何需要的构建上下文。如果你有一个特定的Dockerfile名字或者你想要指定一个不同的上下文目录，你可以使用**-f**标志来指定Dockerfile的路径和名字，然后指定你想要发送到守护进程的上下文目录：`docker build -f /path/to/a/Dockerfile /path/to/a/context/`，在这个例子中，Docker会使用/path/to/a/Dockerfile作为构建的Dockerfile，并将/path/to/a/context/目录下的所有文件和目录发送到守护进程。

### docker login

- 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub
```shell
# -u:登陆的用户名 -p:登陆的密码
docker login -u -p
```

### docker logout

- 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库Docker Hub
```shell
docker logout
```

### docker tag

- 标记本地镜像，将其归入某一仓库
```shell
docker tag
```

### docker network

- 创建网络
```shell
docker network create -d bridge my-net
docker network ls
```

- 指定容器ip

```shell
# 使用自定义Bridge网络
# 创建了一个名为mybridge的自定义Bridge网络，其IP地址段为172.25.0.0/16。
docker network create --driver=bridge --subnet=172.25.0.0/16 mybridge
# 在mybridge网络中创建了一个名为mycontainer的容器，并为其分配了静态IP地址172.25.0.2。
docker run -itd --name=mycontainer --network=mybridge --ip=172.25.0.2 your_image_name

# 使用Macvlan网络模式
# 创建名为macvlan-network的Macvlan网络,其IP地址段为192.168.88.0/24,网关为192.168.88.254,并指定了父网卡为eth0。
docker network create -d macvlan --subnet=192.168.88.0/24 --gateway=192.168.88.254 -o parent=eth0 macvlan-network
# 在macvlan-network网络中创建了一个名为mycontainer的容器，并为其分配了静态IP地址192.168.88.56
docker run -itd --name=mycontainer --privileged=true --network=macvlan-network --ip=192.168.88.56 your_image_name
```
```yml
# 使用Docker Compose
version: '3'  
services:  
  myservice:  
    image: your_image_name  
    networks:  
      mynet:  
        ipv4_address: 172.18.0.2  
networks:  
  mynet:  
    ipam:  
      config:  
        - subnet: 172.18.0.0/16  
          gateway: 172.18.0.1
```

> [Docker有几种网络模式](https://mp.weixin.qq.com/s/KU3bpxiNbHGJQ_XVRqsedg)

### docker system

```shell
# 这个命令的缩写docker info
docker system info
# 提供Docker整体磁盘使用率的概况，包括镜像、容器和(本地)volume。所以我们现在随时都可以查看Docker使用了多少资源。
docker system df
# 显示Docker系统中的实时事件，如容器创建、启动、停止、删除等。
docker system events
# 这两个命令用于设置Docker守护进程的网络相关配置，例如DNS查找超时时间和HTTP代理设置。
docker system dial-timeout 和 docker system http-proxy
# 列出所有当前配置文件。
docker system config ls
# (谨慎使用)此命令将重置Docker到初始化状态，删除所有数据包括镜像、容器、网络和定制配置。
docker system reset
```
### docker prune

**docker system prune**

官网描述：删除所有未使用的容器、网络、映像（包括悬挂的和未引用的），以及卷（可选）。
> **名词解释**：
> 未使用的容器：所有已停止的容器将被删除。
> 未使用的镜像：只有悬挂的镜像（未被任何容器引用）将被删除，除非使用了-a或--all参数。
> 未使用的网络：所有未被容器使用的自定义网络将被删除。
> 未使用的卷（如果使用--volumes或-v选项）：所有未被容器引用的卷将被删除。
> **相关参数**：
> -a,--all：删除未被任何容器引用的所有镜像，而不仅仅是悬挂的镜像。
> --filter：根据提供的条件过滤要删除的内容。
> --force,-f：跳过确认步骤，直接执行删除，不加的话在执行步骤时需要手动确认，建议不用。手动容错率高一点
> --volumes,-v：删除所有未被至少一个容器引用的卷。

```shell
# 删除所有未使用的容器、镜像、网络
docker system prune
#删除所有未使用的容器、镜像（包括未被引用的）、网络
docker system prune -a
# 删除所有未使用的容器、镜像、网络、卷
docker system prune --volumes
# 最后使用时间超过1小时的日志保留
docker system prune --filter "until=1h"
# 删除所有带有“version=1.21.0”标签的未使用的容器、镜像和网络
docker system prune --filter "label=version=1.21.0"
# 删除所有不带有“app=nginx”标签的未使用的容器、镜像和网络
docker system prune --filter "label!=app=nginx"
docker system prune -a -v --filter "label!=app=nginx"
```
label可以使用`docker inspect <容器id或名称>`来查看labels
```json
{
  
  "Labels": {
              "com.docker.compose.config-hash": "b637c41bf29efe8fcd1d3c7baa3ad5ba8dc44f21cec0d3937d665fb79df76644",
              "com.docker.compose.container-number": "1",
              "com.docker.compose.oneoff": "False",
              "com.docker.compose.project": "jpom",
              "com.docker.compose.project.config_files": "docker-compose.yml",
              "com.docker.compose.project.working_dir": "/www/docker-compose/jpom",
              "com.docker.compose.service": "jpom",
              "com.docker.compose.version": "1.29.2"
  }
}
```
删除特定项目的所有容器、图像和网络：
```shell
docker system prune --filter "label=com.docker.compose.project=jpom"
```
删除特定服务的所有容器、图像和网络：
```shell
docker system prune --filter "label=com.docker.compose.service=jpom"
```
删除使用特定版本的Docker Compose创建的所有容器、图像和网络：
```shell
docker system prune --filter "label=com.docker.compose.version=1.29.2"
```

**docker image prune**

修剪镜像,清理none镜像(虚悬镜像),默认情况下，`docker image prune`命令只会清理虚无镜像（没被标记且没被其它任何镜像或容器引用的镜像）。当添加`-a`或`--all`选项时，`docker image prune`的行为会发生变化。这个命令不仅会删除未使用的镜像，还会删除所有没有标签（即dangling或者说悬挂的）的镜像，即使这些镜像被其他标记了的镜像所引用。换句话说，`docker image prune -a` 会清理所有不带有标签且不在使用中的镜像，以及所有未被任何容器引用的镜像。默认情况下，系统会提示是否继续。要绕过提示，请使用-f或--force标志。可以使用--filter标志使用过滤表达式来限制修剪哪些镜像。例如，只考虑24小时前创建的镜像：
```shell
docker image prune -a --filter "until=24h"
```

**docker container prune**

修剪容器,停止容器后不会自动删除这个容器，除非在启动容器的时候指定了–rm标志。使用`docker ps -a`命令查看Docker主机上包含停止的容器在内的所有容器。要清理掉这些，可以使用`docker container prune`命令。默认情况下，系统会提示是否继续。要绕过提示，请使用-f或--force标志。默认情况下，所有停止状态的容器会被删除。可以使用--filter标志来限制范围。例如，可以只删除24小时之前创建的停止状态的容器：

**docker volume prune**

修剪卷。卷可以被一个或多个容器使用，并占用Docker主机上的空间。卷永远不会被自动删除，因为这么做会破坏数据。

**docker network prune**

修剪网络。Docker网络不会占用太多磁盘空间，但是它们会创建iptables规则，桥接网络设备和路由表条目。要清理这些东西，可以使用`docker network prune`来清理没有被容器未使用的网络。


### 其他

#### 查看镜像版本为latest的具体版本号
```bash
docker image inspect nginx:latest|grep -i version
```

#### 查看docker日志文件大小
```shell
ls -lh $(find /var/lib/docker/containers/ -name '*-json.log')
```
#### 删除日志文件
```shell
 rm -rf /var/lib/docker/containers/*/*.log
```
#### 查看docker日志存储路径

> 一般在/var/lib/docker/containers/container_id/\*-json.log

```shell
docker inspect -f '{{.LogPath}}' <container_id_or_name>
```

#### 设置日志文件的大小
1. 使用--log-opt参数限制容器日志大小:在启动容器时,可以使用--log-opt参数设置日志驱动程序的选项。示例：`docker run -d --name example-container --log-opt max-size=10m --log-opt max-file=3 nginx:latest`。在这个例子中，我们设置了以下选项：max-size=10m:限制单个日志文件的最大大小为10MB。max-file=3:限制日志文件的数量为3个。当日志文件达到10MB时，Docker会自动轮换日志文件，保留最新的3个日志文件。
2. 永久方法：全局设置：新建/etc/docker/daemon.json，若有就不用新建了；添加log-dirver和log-opts参数。
```json
{
"log-driver":"json-file",
"log-opts":{
	"max-size":"500m",
    "max-file":"3"
	}
}
```
max-size=500m，意味着一个容器日志大小上限是500M，max-file=3，意味着一个容器有三个日志，分别是id+.json、id+1.json、id+2.json。


### 表格

|   **命令**    |                           **作用**                           |
| :-----------: | :----------------------------------------------------------: |
|  **attach**   | **绑定到运行中容器的 标准输入, 输出,以及错误流（这样似乎也能进入容器内容，但 是一定小心，他们操作的就是控制台，控制台的退出命令会生效，比如 redis,nginx...）** |
|   **build**   |               **从一个Dockerfile文件构建镜像**               |
|  **commit**   |            **把容器的改变 提交创建一个新的镜像**             |
|    **cp**     | **容器和本地文件系统间 复制 文件/文件夹cp -rp[原文件或目录] [目标文件或目录] －r 复制目录 - p 保留文件属性** |
|  **create**   | **创建新容器，但并不启动（注意与docker run 的区分）需要手动启动。start\\stop** |
|   **diff**    | **检查容器里文件系统结构的更改【A：添加文件或目录 D：文件或者目录删除 C：文 件或者目录更改】** |
|  **events**   |                   **获取服务器的实时事件**                   |
|   **exec**    |                 **在运行时的容器内运行命令**                 |
|  **export**   | **导出容器的文件系统为一个tar文件。commit是直接提交成镜像，export是导出成文 件方便传输** |
|  **history**  |                      **显示镜像的历史**                      |
|  **images**   |                       **列出所有镜像**                       |
|  **import**   | **导入tar的内容创建一个镜像，再导入进来的镜像直接启动不了容器。 /docker-entrypoint.sh nginx -g 'daemon o** |
|   **info**    |                       **显示系统信息**                       |
|  **inspect**  |                 **获取docker对象的底层信息**                 |
|   **kill**    |                   **杀死一个或者多个容器**                   |
|   **load**    |         **从tar文件加载镜像 docker load -i xxx.tar**         |
|   **login**   |                   **登录Docker registry**                    |
|  **logout**   |                   **退出Docker registry**                    |
|   **logs**    | **获取容器日志；容器以前在前台控制台能输出的所有内容，都可以看到** |
|   **pause**   |                   **暂停一个或者多个容器**                   |
|   **port**    |                    **列出容器的端口映射**                    |
|    **ps**     |    **列出所有容器 docker ps -a 列出包括已停止的所有容器**    |
|   **pull**    |          **从registry下载一个image 或者repository**          |
|   **push**    |          **给registry推送一个image或者repository**           |
|  **rename**   |                      **重命名一个容器**                      |
|  **restart**  |                   **重启一个或者多个容器**                   |
|    **rm**     |                   **移除一个或者多个容器**                   |
|    **rmi**    |                   **移除一个或者多个镜像**                   |
|    **run**    |                      **创建并启动容器**                      |
|   **save**    | **把一个或者多个镜像保存为tar文件 docker save -o [容器名称]:[容器标签] > xxx.tar** |
|  **search**   |                   **去docker hub寻找镜像**                   |
|   **start**   |                   **启动一个或者多个容器**                   |
|   **stop**    |                   **停止一个或者多个容器**                   |
|    **tag**    |          **给源镜像创建一个新的标签，变成新的镜像**          |
|  **unpause**  |                      **pause的反操作**                       |
|  **update**   |              **更新一个或者多个docker容器配置**              |
|  **version**  |           **Show the Docker version information**            |
| **container** |                         **管理容器**                         |
|   **image**   |                         **管理镜像**                         |
|  **network**  |                         **管理网络**                         |
|  **volume**   |                          **管理卷**                          |
|   **stats**   |                **显示容器资源的实时使用状态**                |
|    **top**    |                  **显示正在运行容器的进程**                  |


> [Docker常用命令，还有谁不会？](https://mp.weixin.qq.com/s/fzlNnJe9SMA5k3TDXOfZUA)

## Dockerfile

1. 文件没有后缀，名字就是Dockerfile
2. 命令约定全部使用大写，如RUN,ADD,FROM
3. 第一条命令必需是FROM，作用是指定在哪个基础镜像上创建镜像。
4. 注释以“#”形状

### 常用参数/命令

#### FROM

语法：

```dockerfile
FROM 镜像名
#例 在jre8镜像基础上创建自己镜像。
FROM livingobjects/jre8
```
最简单的命令，指定在哪个基础镜像上创建镜像

#### RUN

它接受命令作为参数并用于创建镜像,RUN命令用于创建镜像。在镜像构建的过程中执行,这个指令有两种格式

第一种形式：

```dockerfile
RUN chown user2:user2 /home/webapi #以shell形式执行命令，等同于/bin/sh -c
```

第二种形式：

```dockerfile
RUN ["executable","param1", "param2"] # (等同于exec命令形式)，注意此处必须是双引号(")，因为这种格式被解析为JSON数组。
```

#### CMD

语法：

```dockerfile
CMD ["executable", "param1", "param2"?]
# 例
CMD exec java -Djava.security.egd=file:/dev/./urandom -jar /app.jar
```

1. 在镜像构建容器后执行
2. 只能存在一条CMD命令

#### ENTRYPOINT

语法：

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"?]
# 例
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
这个命令和CMD功能一样。区别在于ENTRYPOINT后面携带的参数不会被docker run提供的参数覆盖，而CMD会被覆盖。

##### 扩展CMD与ENTRYPOINT

二者的区别看：[docker CMD ENTRYPOINT区别终极解读](https://blog.csdn.net/u010900754/article/details/78526443)

从根本上说, ENTRYPOINT和CMD都是让用户指定一个可执行程序, 这个可执行程序在container启动后自动启动。实际上, 如果你想让自己制作的镜像自动运行程序(不需要在docker run后面添加命令行指定运行的命令), 你必须在Dockerfile里面，使用ENTRYPOINT或者CMD命令。在命令行启动docker镜像时, 执行其他命令行参数，覆盖默认的CMD。和CMD类似, 默认的ENTRYPOINT也在docker run时, 也可以被覆盖. 在运行时, 用--entrypoint覆盖默认的ENTRYPOINT。

dockerfile中的CMD命令被覆盖：

![img](https://pic4.zhimg.com/v2-4a8d016349ee808822659ca2bf66fab3_r.jpg)

**CMD**：提供了容器默认的执行命令。Dockerfile只允许使用一次CMD指令。使用多个CMD会抵消之前所有的指令，只有最后一个指令生效。CMD有三种形式：

```dockerfile
CMD ["executable","param1","param2"] (exec form, thisis the preferred form)
```

**ENTRYPOINT**：配置给容器一个可执行的命令，这意味着在每次使用镜像创建容器时一个特定的应用程序可以被设置为默认程序。同时也意味着该镜像每次被调用时仅能运行指定的应用。类似于CMD，Docker只允许一个ENTRYPOINT，多个ENTRYPOINT会抵消之前所有的指令，只执行最后的ENTRYPOINT指令。语法如下：

```dockerfile
ENTRYPOINT ["executable", "param1","param2"]
```

#### ADD

语法：

```dockerfile
ADD [source directory or URL] [destination directory]
```

它的基本作用是从源系统的文件系统上复制文件到目标容器的文件系统。

1. 如果源是一个URL，那该URL的内容将被下载并复制到容器中。
2. 如果如果文件是可识别的压缩格式，则docker会帮忙解压缩
3. 如果要ADD本地文件，则本地文件必须在docker build PATH指定的path目录下
4. ADD只有在build镜像的时候运行一次，后面运行container的时候不会再重新加载了

ADD指令不仅能够将构建命令所在的主机本地的文件或目录，而且能够将远程URL所对应的文件或目录，作为资源复制到镜像文件系统。所以，可以认为ADD是增强版的COPY，支持将远程URL的资源加入到镜像的文件系统。

exec格式用法（推荐）：

```dockerfile
ADD ["<src>",... "<dest>"]
```

特别适合路径中带有空格的情况。

shell格式用法：

```dockerfile
ADD <src>... <dest>
```

对于从远程URL获取资源的情况，由于ADD指令不支持认证，如果从远程获取资源需要认证，则只能使用RUN wget或RUN curl替代。另外，如果源路径的资源发生变化，则该ADD指令将使Docker Cache失效，Dockerfile中后续的所有指令都不能使用缓存。因此尽量将ADD指令放在Dockerfile的后面。

#### COPY

COPY指令能够将构建命令所在的主机本地的文件或目录，复制到镜像文件系统。

exec格式用法（推荐）：

```dockerfile
COPY ["<src>",... "<dest>"]
```

特别适合路径中带有空格的情况。

shell格式用法：

```dockerfile
COPY <src>... <dest>
```

#### EXPOSE

语法：

```dockerfile
EXPOSE [port] #暴露容器内部端口
# 例
EXPOSE 5000
```

暴露的是容器内部端口，不是主机端口，如果外部想使用这个端口需要在运行时映射，如下：**docker run -d -p 127.0.0.1:8080:5000 hello-world**。指令用于标明，这个镜像中的应用将会侦听某个端口，并且希望能将这个端口映射到主机的网络界面上。但是，为了安全，docker run命令如果没有带上响应的端口映射参数，docker并不会将端口映射到宿主机。

#### MAINTAINER

语法：

```dockerfile
MAINTAINER 作者名 #申明作者，辅助使用，放在FROM命令后面
```

#### WORKDIR

语法：

```dockerfile
WORKDIR /path # 指定容器工作目录
```

#### VOLUME

语法：

```dockerfile
VOLUME ["/dir_1", "/dir_2" ..] # 可以将本地文件夹或者其他container的文件夹挂载到container中，容器即可以访问该目录
VOLUME ["/data"] (exec格式指令)
```

VOLUME指令创建一个可以从本地主机或其他容器挂载的挂载点。经常用到的是**docker run -ti -v /data:/data nginx:1.12 bash**时指定本地路径和容器内路径的映射。

相信大部分人对`docker run -v`这个参数都比较熟悉，无非就是把宿主机目录和容器目录做映射，以便于容器中的某些文件可以直接保存在宿主机上，实现容器被删除之后数据还在，比如我们把mysql装在容器中，肯定不能说容器被删mysql所有的数据也都不在了。第二个作用是也可以用来实现多容器共享同一份文件。但如果玩过dockerfile的话就知道dockerfile还有个VOLUME指令，如
```dockerfile
FROM centos:latest
RUN groupadd -r redis && useradd -r -g redis rediså
RUN yum -y update &&  yum -y install epel-release && yum -y install redis && yum -y install net-tools
RUN mkdir -p /config && chown -R redis:redis /config
VOLUME /share/data　#声明容器中/share/data为匿名卷
EXPOSE 6379
```
这个指令很容易和启动时的-v指令搞混淆，他们之间到底有什么区别呢，什么时候需要使用volume呢？volume指令指定的位置在容器被删除以后数据文件会被删除吗？如果-v和volume指定了同一个位置，会发生什么事呢？

1. volume和run -v的区别，什么时候需要使用volume
    容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)中。为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在Dockerfile中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

2. 那么Dockerfile中的VOLUME指令实际使用中是不是就是跟docker run中的-v参数一样是将宿主机的一个目录绑定到容器中的目录以达到共享目录的作用呢？
    并不然，其实VOLUME指令只是起到了声明了容器中的目录作为匿名卷，但是并没有将匿名卷绑定到宿主机指定目录的功能。当我们生成镜像的Dockerfile中以Volume声明了匿名卷，并且我们以这个镜像run了一个容器的时候，docker会在安装目录下的指定目录下面生成一个目录来绑定容器的匿名卷（这个指定目录不同版本的docker会有所不同），我当前的目录为：/var/lib/docker/volumes/{容器ID}。
    **总结**：volume只是指定了一个目录，用以在用户忘记启动时指定-v参数也可以保证容器的正常运行。比如mysql，你不能说用户启动时没有指定-v，然后删了容器，就把mysql的数据文件都删了，那样生产上是会出大事故的，所以mysql的dockerfile里面就需要配置volume，这样即使用户没有指定-v，容器被删后也不会导致数据文件都不在了。还是可以恢复的。

3. volume指令指定的位置在容器被删除以后数据文件会被删除吗
    volume与-v指令一样，容器被删除以后映射在主机上的文件不会被删除。

4. 如果-v和volume指定了同一个位置，会发生什么事呢？
    会以-v设定的目录为准，其实volume指令的设定的目的就是为了避免用户忘记指定-v的时候导致的数据丢失，那么如果用户指定了-v，自然而然就不需要volume指定的位置了。
    **总结**：其实一般的dockfile如果不是数据库类的这种需要持久化数据到磁盘上的应用，都是无需指定volume的。指定volume只是为了避免用户忘记指定-v时导致的数据全部在容器中，这样的话容器一旦被删除所有的数据都丢失了。

那么为什么dockerfile中不提供一个能够映射为主机目录:容器目录这样的指令呢？其实这样的设计是有道理的，如果在dockerfile中指定了主机目录，这样dockerfile就不具备了可移植性了，毕竟每个人所需要映射的目录可能是不同的，那么最好的办法就是把这个权利交给每个运行这个dockerfile的人，所以才会有`run -v主机目录:容器目录`这样的指令。

#### ENV

语法：

```dockerfile
# 设置变量，可能在容器和脚本里直接使用
ENV key value
#例：
ENV WORKPATH /tmp
# 或
ENV abc=bye def=$abc

# 第一种用法用于设置单个变量(第一个空格前为key，之后都是value,包括后面的空格)，第二种用于同时设置多个变量(空格为分隔符，value中包含空格时可以用双引号把value括起来，或者在空格前加反斜线)，当需要同时设置多个环境变量时推荐使用第二种格式。这些环境变量可以通过docker run命令的--env参数来进行修改。
```

#### ARG

```dockerfile
ARG <name>[=<default value>]
```

ARG指令设置一些创建镜像时的参数，这些参数可以在执行docker build命令时通过`--build-arg= `设置，如果指定的创建参数在Dockerfile中没有指定，创建时会输出错误信息: One or more build-args were not consumed, failing build.

Dockerfile作者可以为ARG设置一个默认参数值，当创建镜像时如果没有传入参数就会使用默认值：


我们可以使用ARG或者ENV指令来指定RUN指令使用的变量。我们可以使用ENV定义与ARG定义名称相同的变量来覆盖ARG定义的变量值。如下示例，我们执行

```dockerfile
docker build --build-arg CONT_IMG_VER=v2.0.1 Dockerfile
```

后将获取到的CONTIMGVER变量值为v2.0.0:

```dockefile
ARG CONT_IMG_VER
ENV MY_SECRET_KEY=$CONT_IMG_VER

```

#### WORKDIR

```dockerfile
WORKDIR /path/to/workdir
```

WORKDIR指令用来设置Dockerfile中任何使用目录的命令(包括RUN、CMD、ENTRYPOINT、COPY、ADD等指令)的当前工作目录，此目录如果不存在就会被自动创建，即使这个目录不被使用


### 示例

```dockerfile
FROM anapsix/alpine-java:8_server-jre_unlimited
VOLUME /tmp
ENV TZ=Asia/Shanghai
EXPOSE 19990
ADD applet-provider.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

```shell
#!/bin/bash
echo "自动构建pc包的镜像和运行容器"
#1.停掉容器和删掉容器
id=$(docker ps |grep mdjz  | tail -n 1| awk 'RS {print $1 }')
echo $id
#杀掉进程
echo "docker kill $id"
docker kill $id
#删除容器
echo "docker rm $id"
docker rm $id
#删掉镜像
mid=$(docker images |grep mdjz  | tail -n 1| awk 'RS {print $1 }')
echo "docker rmi $mid"
docker rmi $mid
#重新打包镜像
docker build -f /composetest/pc/Dockerfile -t mdjz /composetest/pc/
#运行容器
echo "run app docker run -itd --name mdjz --restart always  -p  19901:19901 "
docker run -itd --name mdjz --network=my-net --restart always -p 19901:19901 -v /etc/localtime:/etc/localtime:ro -v /etc/timezone:/etc/timezone:ro mdjz
```


> [如何编写最佳的Dockerfile](https://mp.weixin.qq.com/s/x-M5iKvvuseIQwUdVmxSPQ)
> [构建Java镜像的10个最佳实践](https://mp.weixin.qq.com/s/gmZDBuYDXnNdykEx66Y0Cw)


## 使用docker安装主流软件

### 安装mysql8.0

1. 拉取镜像
```bash
docker pull mysql:8.0.16
```
2. 创建目录
```bash
mkdir -p /app/mysql/conf /app/mysql/data
```
3. 创建配置文件
```bash
vim /app/mysql/conf/my.cnf
```
配置文件
```
[client]
#socket = /app/mysql/mysqld.sock
default-character-set = utf8mb4

[mysqld]
datadir = /var/lib/mysql
character_set_server = utf8mb4
collation_server = utf8mb4_bin
secure-file-priv= NULL
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
# Custom config should go here
!includedir /etc/mysql/conf.d/
```
4. 创建和启动容器
```bash
docker run --restart=always --network=my-net -d --name mysql 
-v /app/mysql/conf/my.cnf:/etc/mysql/my.cnf
-v /app/mysql/data:/var/lib/mysql
-p 3306:3306 -e MYSQL_ROOT_PASSWORD=1qaz@WSX mysql:8.0.16
```
5. 修改mysql密码以及授权可访问主机
进入容器内部
```bash
docker exec -it mysql /bin/bash
```
连接mysql
```bash
mysql -uroot -p
```
使用mysql库
```bash
use mysql
```
修改主机及访问密码，设置所有主机可访问
```bash
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '1qaz@WSX';
```
刷新
```bash
flush privileges;
```

### 安装nginx

1. 拉取最新镜像
```bash
docker pull nginx
```
2. 新建目录
```bash
# 创建挂载目录
mkdir -p /app/nginx/conf /app/nginx/log /app/nginx/html
#创建前端发版的目录
mkdir -p /app/nginx/web_dist /app/nginx/app_dist
```
3. 容器中的nginx.conf文件和conf.d文件夹复制到宿主机
```bash
# 生成容器
docker run --name nginx -p 9090:80 -d nginx
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /app/nginx/conf/nginx.conf
# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /app/nginx/conf/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /app/nginx/
```
4. 删掉临时的nginx容器，并且创建新的容器
```bash
# 直接执行docker rm nginx或者以容器id方式关闭容器
# 找到nginx对应的容器id
docker ps -a
# 关闭该容器
docker stop nginx
# 删除该容器
docker rm nginx
# 删除正在运行的nginx容器
docker run --privileged=true --restart unless-stopped
-p 9090:9090
--name nginx
-v /app/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /app/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /app/nginx/log:/var/log/nginx \
-v /app/nginx/html:/usr/share/nginx/html \
-d nginx:latest
```
> 注意ngin配置文件里面所有的路径指向的都是容器里面的路径，而并非宿主机上面的路径，端口也是指向容器内部的端口，如果路径没有挂载或者挂载不正确访问nginx的时候会出现500错误。

### 安装nacos

1. 拉取镜像
```bash
docker pull nacos/nacos-server:2.2.3
```
2. 创建挂载目录
```bash
mkdir -p /app/nacos/logs/ /app/nacos/conf/
#创建一个配置文件
vim /app/nacos/conf/application.properties

#修改配置文件
server.servlet.contextPath=/nacos
server.error.include-message=ALWAYS
server.port=18848

db.pool.config.connectionTimeout=30000
db.pool.config.validationTimeout=10000
db.pool.config.maximumPoolSize=20
db.pool.config.minimumIdle=2

nacos.naming.empty-service.auto-clean=true
nacos.naming.empty-service.clean.initial-delay-ms=50000
nacos.naming.empty-service.clean.period-time-ms=30000

management.metrics.export.elastic.enabled=false
#management.metrics.export.elastic.host=http://localhost:9200
management.metrics.export.influx.enabled=false

server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i %{Request-Source}i
server.tomcat.basedir=file:.

nacos.security.ignore.urls=/,/error,/**/*.css,/**/*.js,/**/*.html,/**/*.map,/**/*.svg,/**/*.png,/**/*.ico,/console-ui/public/**,/v1/auth/**,/v1/console/health/**,/actuator/**,/v1/console/server/**
nacos.core.auth.system.type=nacos
nacos.core.auth.enabled=true
nacos.core.auth.caching.enabled=true
nacos.core.auth.enable.userAgentAuthWhite=false
nacos.core.auth.server.identity.key=abc
nacos.core.auth.server.identity.value=abc/123
nacos.core.auth.plugin.nacos.token.cache.enable=false
nacos.core.auth.plugin.nacos.token.expire.seconds=18000
nacos.core.auth.plugin.nacos.token.secret.key=SecretKey012345678901234567890123456789012345678901234567890123456789
nacos.istio.mcp.server.enabled=false
```
3. 创建内部桥接网络
```bash
docker network create -d bridge my-net
```
4. 运行容器
```bash
docker run -d
--network=my-net
--privileged=true
-e PREFER_HOST_MODE=ip
-e MODE=standalone
-e TIME_ZONE='Asia/Shanghai'
-v /app/nacos/conf/application.properties:/home/nacos/conf/application.properties
-v /app/nacos/logs:/home/nacos/logs
-p 8848:8848
--name nacos
--restart=always
nacos/nacos-server:2.2.3
```
5. 进入容器内部修改配置文件
```bash
docker exec -it nacos /bin/bash
cd /home/nacos/conf
vim application.properties
```
6. 复制容器内的文件出来，修改和挂载
```bash
docker cp nacos:/home/nacos/conf/application.properties /app/nacos/conf/application.properties
```
7. 重启容器
```bash
docker restart nacos
```

### 安装redis

1. 拉取最新镜像
```bash
docker pull redis
```
2. 创建挂载目录
```bash
mkdir -p /app/redis/data
```
3. 运行容器
```bash
docker run --name redis -d 
--privileged=true 
-p 6379:6379
-v /app/redis/redis.conf:/etc/redis/redis.conf
-v /app/redis/data:/data
redis redis-server /etc/redis/redis.conf
--appendonly yes
```

### 部署ElasticSearch

```shell
# 准备文件和文件夹，并chmod -R 777 xxx
# 配置文件内容，参照
https://www.elastic.co/guide/en/elasticsearch/reference/7.5/node.name.html 搜索相
关配置
# 考虑为什么挂载使用esconfig ...
docker run --name=elasticsearch -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms300m -Xmx300m" \
-v /app/es/data:/usr/share/elasticsearch/data \
-v /app/es/plugins:/usr/shrae/elasticsearch/plugins \
-v esconfig:/usr/share/elasticsearch/config \
-d elasticsearch:7.12.0
```

### 部署Tomcat

```shell
# 考虑，如果我们每次-v都是指定磁盘路径，是不是很麻烦？
docker run --name tomcat-app -p 8080:8080 \
-v tomcatconf:/usr/local/tomcat/conf \
-v tomcatwebapp:/usr/local/tomcat/webapps \
-d tomcat:jdk8-openjdk-slim-buster
```

## 其他

### docker加速命令

```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
或
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```
**其他方案**


```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://mfs5bvup.mirror.aliyuncs.com%22/]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
以后docker下载直接从阿里云拉取相关镜像


### 可视化界面-Portainer

**什么是Portainer**

Portainer社区版2.0拥有超过50万的普通用户，是功能强大的开源工具集，可让您轻松地在Docker，Swarm，Kubernetes和Azure ACI中构建和管理容器。 Portainer的工作原理是在易于使用的GUI后面隐藏使管理容器变得困难的复杂性。通过消除用户使用CLI，编写YAML或理解清单的需求，Portainer使部署应用程序和解决问题变得如此简单，任何人都可以做到。 Portainer开发团队在这里为您的Docker之旅提供帮助；

**安装**

服务端部署
```shell
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v
/var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data
portainer/portainer-ce
# 访问9000端口即可
```
agent端部署
```shell
docker run -d -p 9001:9001 --name portainer_agent --restart=always -v
/var/run/docker.sock:/var/run/docker.sock -v
/var/lib/docker/volumes:/var/lib/docker/volumes portainer/agent
```

### 与Spring Boot

| [一键部署Spring Boot到远程Docker容器](https://mp.weixin.qq.com/s/15ZAVUg5DfcF53QpEetT7Q) | [Jenkins+Docker一键自动化部署SpringBoot项目](https://mp.weixin.qq.com/s/dP-c3twzR0PMUvPWZA-U0Q) | [搭建SpringBoot项目并将其Docker化](https://mp.weixin.qq.com/s/CXUwpTbAVoXEeB7EcrCjAw) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [SpringBoot使用Docker快速部署项目](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493762&idx=1&sn=114663a4a13ba5bb27d05e0d77de37c1&source=41#wechat_redirect) | [Docker部署Spring Boot项目的2种方式！](https://mp.weixin.qq.com/s/du2sypGQczJh7gQz_4IX9g) | [SpringBoot项目构建Docker镜像深度调优](https://mp.weixin.qq.com/s?__biz=Mzg2MDYzODI5Nw==&mid=2247493962&idx=1&sn=af6c945d629003cfd30564698c017598&source=41#wechat_redirect) |
| [还在手动部署springboot项目？不妨试试它，让你部署项目飞起来！](https://mp.weixin.qq.com/s/01SZo3NNf5zuAC8wAI6C-g) | [Docker+Spring Boot+FastDFS搭建一套分布式文件服务器，太强了！](https://mp.weixin.qq.com/s/HSRIYQVKR9TGtwetd3LU5w) |                                                              |

### 相关文章

| [图解Docker架构，傻瓜都能看懂！](https://mp.weixin.qq.com/s/ELZo2z4fHonoBGXQI0M9CA) | [10个冷门但又非常实用的Docker使用技巧！！](https://mp.weixin.qq.com/s/LOmqsoBJd7h1HPwf0i1uwQ) | [Docker实战总结](https://mp.weixin.qq.com/s/tTsizeLeVyvQ44GXMNqrjA) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [Docker从入门到干活，看这一篇足矣](https://mp.weixin.qq.com/s/t81enr-ypBxk1K4lYqWZww) | [Docker：Docker Compose详解](https://www.jianshu.com/p/658911a8cff3) | [CentOS/Ubuntu安装Docker和Docker Compose](https://mp.weixin.qq.com/s/fB59zXK7cPBt1asSyUpqDg) |
| [DaoCloud安装docker指南](http://guide.daocloud.io/dcs/docker-9152677.html) | [一款吊炸天的Docker图形化工具，太强大](https://mp.weixin.qq.com/s/PpI7_fY5ACjmtmnlqr7ZMQ) | [5款顶级Docker容器GUI管理工具！免费又好用](https://mp.weixin.qq.com/s/w0sFaHApOSrwgva0886ijQ) |
| [Docker轻量级编排创建工具Humpback](https://mp.weixin.qq.com/s/rAOsia2LU2_Fl4vrjQ2tvA) | [IDEA使用Docker插件，实现一键自动化部署](https://mp.weixin.qq.com/s/yg5ACCeeyJa0AVP1LatUhA) |                                                              |
