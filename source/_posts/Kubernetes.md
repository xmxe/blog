---
title: kubernetes
categories: 技术栈
img: https://pic1.zhimg.com/70/v2-353855dec58fcaa1477e90a869b30ad0_1440w.avis

---

> https://k8s-tutorials.pages.dev/
> https://kuboard.cn/
> https://k8s.iswbm.com/

下面是所有文档的集合：

- [kubernetes tutorials](#kubernetes-tutorials)
  - [准备工作](#准备工作)
    - [安装docker](#安装-docker)
      - [推荐安装方法](#推荐安装方法)
      - [其它安装方法](#其它安装方法)
    - [安装minikube](#安装-minikube)
      - [启动minikube](#启动-minikube)
    - [安装kubectl](#安装-kubectl)
    - [注册docker hub账号登录](#注册-docker-hub-账号登录)
  - [Container](#container)
  - [Pod](#pod)
    - [Pod与Container的不同](#pod-与-container-的不同)
    - [Pod其它命令](#pod-其它命令)
  - [Deployment](#deployment)
    - [扩容](#扩容)
    - [升级版本](#升级版本)
    - [Rolling Update(滚动更新)](#rolling-update滚动更新)
    - [存活探针(livenessProb)](#存活探针-livenessprob)
    - [就绪探针(readiness)](#就绪探针-readiness)
  - [Service](#service)
    - [ClusterIP](#clusterip)
    - [NodePort](#nodeport)
    - [LoadBalancer](#loadbalancer)
  - [ingress](#ingress)
  - [Namespace](#namespace)
  - [Configmap](#configmap)
  - [Secret](#secret)
  - [Job](#job)
  - [CronJob](#cronjob)
  - [Helm](#helm)
    - [用helm安装hellok8s](#用-helm-安装-hellok8s)
    - [创建helm charts](#创建-helm-charts)
      - [rollback](#rollback)
      - [多环境配置](#多环境配置)
    - [helm chart打包和发布](#helm-chart-打包和发布)
  - [Dashboard](#dashboard)
    - [kubernetes dashboard](#kubernetes-dashboard)
    - [K9s](#k9s)
  - [Sponsor](#sponsor)
  - [Star History](#star-history)

# kubernetes tutorials

## 准备工作

在开始本教程之前，需要配置好本地环境，以下是需要安装的依赖和包。

### 安装docker

首先我们需要安装`docker`来打包镜像，如果你本地已经安装了`docker`，那么你可以选择跳过这一小节。

#### 推荐安装方法

目前使用 [Docker Desktop](https://www.docker.com/products/docker-desktop/)来安装docker还是最简单的方案，打开官网下载对应你电脑操作系统的包即可(https://www.docker.com/products/docker-desktop/)，

当安装完成后，可以通过`docker run hello-world`来快速校验是否安装成功！

#### 其它安装方法

目前Docker公司宣布[Docker Desktop](https://www.docker.com/products/docker-desktop/)只对个人开发者或者小型团体免费(2021年起对大型公司不再免费)，所以如果你不能通过[Docker Desktop](https://www.docker.com/products/docker-desktop/)的方式下载安装`docker`，可以参考[这篇文章](https://dhwaneetbhatt.com/blog/run-docker-without-docker-desktop-on-macos)只安装[Docker CLI](https://github.com/docker/cli)。

### 安装minikube

我们还需要搭建一套k8s本地集群(使用云厂商或者其它k8s集群都可)。本地搭建k8s集群的方式推荐使用[minikube](https://minikube.sigs.k8s.io/docs/)。

可以根据[minikube快速安装](https://minikube.sigs.k8s.io/docs/start/)来进行下载安装，这里简单列举MacOS的安装方式，Linux&Windows操作系统可以参考[官方文档](https://minikube.sigs.k8s.io/docs/start/)快速安装。

```shell
brew install minikube
```

#### 启动minikube

因为minikube支持很多容器和虚拟化技术([Docker](https://minikube.sigs.k8s.io/docs/drivers/docker/),[Hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/),[Hyper-V](https://minikube.sigs.k8s.io/docs/drivers/hyperv/),[KVM](https://minikube.sigs.k8s.io/docs/drivers/kvm2/),[Parallels](https://minikube.sigs.k8s.io/docs/drivers/parallels/),[Podman](https://minikube.sigs.k8s.io/docs/drivers/podman/),[VirtualBox](https://minikube.sigs.k8s.io/docs/drivers/virtualbox/),or[VMware Fusion/Workstation](https://minikube.sigs.k8s.io/docs/drivers/vmware/))，也是问题出现比较多的地方，所以这里还是稍微说明一下。

如果你使用`docker`的方案是上面推荐的[Docker Desktop](https://www.docker.com/products/docker-desktop/)，那么你以下面的命令启动minikube即可，需要耐心等待下载依赖。

```shell
minikube start --vm-driver docker --container-runtime=docker
```

启动完成后，运行`minikube status`查看当前状态确定是否启动成功！

如果你本地只有Docker CLI，判断标准如果执行`docker ps`等命令，返回错误`Cannot connect to the Docker daemon at unix:///Users/xxxx/.colima/docker.sock. Is the docker daemon running?`那么就需要操作下面的命令。

```shell
brew install hyperkit
minikube start --vm-driver hyperkit --container-runtime=docker

# Tell Docker CLI to talk to minikube's VM
eval $(minikube docker-env)

# Save IP to a hostname
echo "`minikube ip` docker.local" | sudo tee -a /etc/hosts > /dev/null

# Test
docker run hello-world
```

**minikube 命令速查**

`minikube stop` 不会删除任何数据，只是停止VM和k8s集群。

`minikube delete` 删除所有minikube启动后的数据。

`minikube ip` 查看集群和docker enginer运行的IP地址。

`minikube pause` 暂停当前的资源和k8s集群

`minikube status` 查看当前集群状态

### 安装kubectl

这一步是可选的，如果不安装的话，后续所有`kubectl`相关的命令，使用`minikube kubectl`命令替代即可。

如果你不想使用`minikube kubectl`或者配置相关环境变量来进行下面的教学的话，可以考虑直接安装`kubectl`。

```shell
brew install kubectl
```

### 注册docker hub账号登录

因为默认minikube使用的镜像地址是DockerHub，所以我们还需要在DockerHub(https://hub.docker.com/)中注册账号，并且使用login命令登录账号。

```shell
docker login
```

## Container

我们的旅程从一段代码开始。新建一个`main.go`文件，复制下面的代码到文件中。

```go
package main

import (
	"io"
	"net/http"
)

func hello(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "[v1] Hello, Kubernetes!")
}

func main() {
	http.HandleFunc("/", hello)
	http.ListenAndServe(":3000", nil)
}
```

上面是一串用[Go](https://go.dev/)写的代码，代码逻辑非常的简单，首先启动HTTP服务器，监听`3000`端口，当访问路由`/`的时候返回字符串`[v1] Hello, Kubernetes!`。

在以前，如果你想将这段代码运行起来并测试一下。你首先需要懂得如何下载golang的安装包进行安装，接着需要懂得`golang module`的基本使用，最后还需要了解golang的编译和运行命令，才能将该代码运行起来。甚至在过程中，可能会因为环境变量问题、操作系统问题、处理器架构等问题导致编译或运行失败。

但是通过Container(容器)技术，只需要上面的代码，附带着对应的容器`Dockerfile`文件，那么你就不需要golang的任何知识，也能将代码顺利运行起来。

> Container(容器)是一种沙盒技术。它是基于Linux中Namespace/Cgroups/chroot等技术组合而成，更多技术细节可以参照这个视频[如何自己实现一个容器](https://www.youtube.com/watch?v=8fi7uSYlOdc)。

下面就是Go代码对应的`Dockerfile`，简单的方案是直接使用golang的alpine镜像来打包，但是因为我们后续练习需要频繁的推送镜像到Docker Hub和拉取镜像到k8s集群中，为了优化网络速度，我们选择先在`golang:1.16-buster`中将上述Go代码编译成二进制文件，再将二进制文件复制到`base-debian10`镜像中运行(Dockerfile不理解没有关系，不影响后续学习)。

这样我们可以将300MB大小的镜像变成只有20MB的镜像，甚至压缩上传到DockerHub后大小只有10MB！

```dockerfile
# Dockerfile
FROM golang:1.16-buster AS builder
RUN mkdir /src
ADD . /src
WORKDIR /src

RUN go env -w GO111MODULE=auto
RUN go build -o main .

FROM gcr.io/distroless/base-debian10

WORKDIR /

COPY --from=builder /src/main /main
EXPOSE 3000
ENTRYPOINT ["/main"]
```

需要注意`main.go`文件需要和`Dockerfile`文件在同一个目录下，执行下方`docker build`命令，第一次需要耐心等待拉取基础镜像。并且**需要注意将命令中`guangzhengli`替换成自己的`DockerHub`注册的账号名称**。这样我们后续可以推送镜像到自己注册的`DockerHub`仓库当中。

```shell
docker build . -t guangzhengli/hellok8s:v1
# Step 1/11 : FROM golang:1.16-buster AS builder
# ...
# ...
# Step 11/11 : ENTRYPOINT ["/main"]
# Successfully tagged guangzhengli/hellok8s:v1


docker images
# guangzhengli/hellok8s          v1         f956e8cf7d18   8 days ago      25.4MB
```

`docker build`命令完成后我们可以通过`docker images`命令查看镜像是否build成功，最后我们执行`docker run`命令将容器启动，`-p`指定`3000`作为端口，`-d`指定容器后台运行。

```shell
docker run -p 3000:3000 --name hellok8s -d guangzhengli/hellok8s:v1
```

运行成功后，可以通过浏览器或者`curl`来访问`http://127.0.0.1:3000`,查看是否成功返回字符串`[v1] Hello, Kubernetes!`。

这里因为我本地只用Docker CLI，而docker runtime是使用`minikube`，所以我需要先调用`minikube ip`来返回minikube IP地址，例如返回了`192.168.59.100`，所以我需要访问`http://192.168.59.100:3000`来判断是否成功返回字符串`[v1] Hello, Kubernetes!`。

最后确认没有问题，使用`docker push`将镜像上传到远程的`DockerHub`仓库当中，这样可以供他人下载使用，也方便后续`Minikube`下载镜像使用。**需要注意将`guangzhengli`替换成自己的`DockerHub`账号名称**。

```shell
docker push guangzhengli/hellok8s:v1
```

经过这一节的练习，有没有对容器的强大有一个初步的认识呢？可以想象当你想部署一个更复杂的服务时，例如Nginx，MySQL，Redis。你只需要到[DockerHub搜索](https://hub.docker.com/search?q=)中搜索对应的镜像，通过`docker pull`下载镜像，`docker run`启动服务即可！而无需关心依赖和各种配置！

## Pod

如果在生产环境中运行的都是独立的单体服务，那么Container(容器)也就够用了，但是在实际的生产环境中，维护着大规模的集群和各种不同的服务，服务之间往往存在着各种各样的关系。而这些关系的处理，才是手动管理最困难的地方。

**Pod**是我们将要创建的第一个k8s资源，也是可以在Kubernetes中创建和管理的、最小的可部署的计算单元。在了解`pod`和`container`的区别之前，我们可以先创建一个简单的pod试试，

我们先创建`nginx.yaml`文件，编写一个可以创建`nginx`的Pod。

```yaml
# nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
```

其中`kind`表示我们要创建的资源是`Pod`类型，`metadata.name`表示要创建的pod的名字，这个名字需要是唯一的。`spec.containers`表示要运行的容器的名称和镜像名称。镜像默认来源`DockerHub`。

我们运行第一条k8s命令`kubectl apply -f nginx.yaml`命令来创建`nginx`Pod。

接着通过`kubectl get pods`来查看pod是否正常启动。

最后通过`kubectl port-forward nginx-pod 4000:80`命令将`nginx`默认的`80`端口映射到本机的`4000`端口，打开浏览器或者`curl`来访问`http://127.0.0.1:4000`,查看是否成功访问`nginx`默认页面！

``` shell
kubectl apply -f nginx.yaml
# pod/nginx-pod created

kubectl get pods
# nginx-pod         1/1     Running   0           6s

kubectl port-forward nginx-pod 4000:80
# Forwarding from 127.0.0.1:4000 -> 80
# Forwarding from [::1]:4000 -> 80
```

`kubectl exec -it`可以用来进入Pod内容器的Shell。通过命令下面的命令来配置`nginx`的首页内容。

```shell
kubectl exec -it nginx-pod /bin/bash

echo "hello kubernetes by nginx!" > /usr/share/nginx/html/index.html

kubectl port-forward nginx-pod 4000:80
```

最后可以通过浏览器或者`curl`来访问`http://127.0.0.1:4000`,查看是否成功启动`nginx`和返回字符串`hello kubernetes by nginx!`。

### Pod与Container的不同

回到`pod`和`container`的区别，我们会发现刚刚创建出来的资源如下图所示，在最内层是我们的服务`nginx`，运行在`container`容器当中，`container`(容器)的本质是进程，而`pod`是管理这一组进程的资源。

![nginx_pod](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/nginx_pod.png)

所以自然`pod`可以管理多个`container`，在某些场景例如服务之间需要文件交换(日志收集)，本地网络通信需求(使用localhost或者Socket文件进行本地通信)，在这些场景中使用`pod`管理多个`container`就非常的推荐。而这，也是k8s如何处理服务之间复杂关系的第一个例子，如下图所示：

![pod](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/pod.png)

### Pod其它命令

我们可以通过`logs`或者`logs -f`命令查看pod日志，可以通过`exec -it`进入pod或者调用容器命令，通过`delete pod`或者`delete -f nginx.yaml`的方式删除pod资源。这里可以看到[kubectl所有命令](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)。

```shell
kubectl logs --follow nginx-pod
                              
kubectl exec nginx-pod -- ls

kubectl delete pod nginx-pod
# pod "nginx-pod" deleted

kubectl delete -f nginx.yaml
# pod "nginx-pod" deleted
```

最后，根据我们在`container`的那节构建的`hellok8s:v1`的镜像，同时参考`nginx`pod的资源定义，你能独自编写出`hellok8s:v1`Pod的资源文件吗。并通过`port-forward`到本地的`3000`端口进行访问，最终得到字符串`[v1] Hello, Kubernetes!`。

`hellok8s:v1`Pod资源定义和相应的命令如下所示：

```yaml
# hellok8s.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hellok8s
spec:
  containers:
    - name: hellok8s-container
      image: guangzhengli/hellok8s:v1
```

```shell
kubectl apply -f hellok8s.yaml

kubectl get pods

kubectl port-forward hellok8s 3000:3000
```

## Deployment

在生产环境中，我们基本上不会直接管理pod，我们需要`kubernetes`来帮助我们来完成一些自动化操作，例如自动扩容或者自动升级版本。可以想象在生产环境中，我们手动部署了10个`hellok8s:v1`的pod，这个时候我们需要升级成`hellok8s:v2`版本，我们难道需要一个一个的将`hellok8s:v1`的pod手动升级吗？

这个时候就需要我们来看`kubernetes`的另外一个资源`deployment`，来帮助我们管理pod。

### 扩容

首先可以创建一个`deployment.yaml`的文件。来管理`hellok8s`pod。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
        - image: guangzhengli/hellok8s:v1
          name: hellok8s-container
```

其中`kind`表示我们要创建的资源是`deployment`类型，`metadata.name`表示要创建的deployment的名字，这个名字需要是**唯一**的。

在`spec`里面表示，首先`replicas`表示的是部署的pod副本数量，`selector`里面表示的是`deployment`资源和`pod`资源关联的方式，这里表示`deployment`会管理(selector)所有`labels=hellok8s`的pod。

`template`的内容是用来定义`pod`资源的，你会发现和Hellok8s Pod资源的定义是差不多的，唯一的区别是我们需要加上`metadata.labels`来和上面的`selector.matchLabels`对应起来。来表明pod是被deployment管理，不用在`template`里面加上`metadata.name`是因为deployment会自动为我们创建pod的唯一`name`。

接下来输入下面的命令，可以创建`deployment`资源。通过`get`和`delete pod`命令，我们会初步感受deployment的魅力。**每次创建的pod名称都会变化，某些命令记得替换成你的pod的名称**

```shell
kubectl apply -f deployment.yaml

kubectl get deployments
#NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
#hellok8s-deployment   1/1     1            1           39s

kubectl get pods             
#NAME                                   READY   STATUS    RESTARTS   AGE
#hellok8s-deployment-77bffb88c5-qlxss   1/1     Running   0          119s

kubectl delete pod hellok8s-deployment-77bffb88c5-qlxss 
#pod "hellok8s-deployment-77bffb88c5-qlxss" deleted

kubectl get pods                                       
#NAME                                   READY   STATUS    RESTARTS   AGE
#hellok8s-deployment-77bffb88c5-xp8f7   1/1     Running   0          18s
```

我们会发现一个有趣的现象，当手动删除一个`pod`资源后，deployment会自动创建一个新的`pod`，这和我们之前手动创建pod资源有本质的区别！这代表着当生产环境管理着成千上万个pod时，我们不需要关心具体的情况，只需要维护好这份`deployment.yaml`文件的资源定义即可。

接下来我们通过自动扩容来加深这个知识点，当我们想要将`hellok8s:v1`的资源扩容到3个副本时，只需要将`replicas`的值设置成3，接着重新输入`kubectl apply -f deployment.yaml`即可。如下所示：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
        - image: guangzhengli/hellok8s:v1
          name: hellok8s-container
```

可以在`kubectl apply`之前通过新建窗口执行`kubectl get pods --watch`命令来观察pod启动和删除的记录，想要减少副本数时也很简单，你可以尝试将副本数随意增大或者缩小，再通过`watch`来观察它的状态。

![deployment](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/deployment.png)

### 升级版本

我们接下来尝试将所有`v1`版本的`pod`升级到`v2`版本。首先我们需要构建一份`hellok8s:v2`的版本镜像。唯一的区别就是字符串替换成了`[v2] Hello, Kubernetes!`。

```go
package main

import (
	"io"
	"net/http"
)

func hello(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "[v2] Hello, Kubernetes!")
}

func main() {
	http.HandleFunc("/", hello)
	http.ListenAndServe(":3000", nil)
}
```

将 `hellok8s:v2` 推到 DockerHub 仓库中。

```shell
docker build . -t guangzhengli/hellok8s:v2
docker push guangzhengli/hellok8s:v2
```

接着编写`v2`版本的deployment资源文件。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
        - image: guangzhengli/hellok8s:v2
          name: hellok8s-container
```

```shell
kubectl apply -f deployment.yaml
# deployment.apps/hellok8s-deployment configured

kubectl get pods                
# NAME                                   READY   STATUS    RESTARTS   AGE
# hellok8s-deployment-66799848c4-kpc6q   1/1     Running   0          3s
# hellok8s-deployment-66799848c4-pllj6   1/1     Running   0          3s
# hellok8s-deployment-66799848c4-r7qtg   1/1     Running   0          3s

kubectl port-forward hellok8s-deployment-66799848c4-kpc6q 3000:3000
# Forwarding from 127.0.0.1:3000 -> 3000
# Forwarding from [::1]:3000 -> 3000

# open another terminal
curl http://localhost:3000
# [v2] Hello, Kubernetes!
```

你也可以输入`kubectl describe pod hellok8s-deployment-66799848c4-kpc6q`来看是否是`v2`版本的镜像。

### Rolling Update(滚动更新)

如果我们在生产环境上，管理着多个副本的`hellok8s:v1`版本的pod，我们需要更新到`v2`的版本，像上面那样的部署方式是可以的，但是也会带来一个问题，就是所有的副本在同一时间更新，这会导致我们`hellok8s`服务在短时间内是不可用的，因为所有pod都在升级到`v2`版本的过程中，需要等待某个pod升级完成后才能提供服务。

这个时候我们就需要滚动更新(rolling update)，在保证新版本`v2`的pod还没有`ready`之前，先不删除`v1`版本的pod。

在deployment的资源定义中,`spec.strategy.type`有两种选择:

- **RollingUpdate**：逐渐增加新版本的pod，逐渐减少旧版本的pod。
- **Recreate**：在新版本的pod增加前，先将所有旧版本pod删除。

大多数情况下我们会采用滚动更新(RollingUpdate)的方式，滚动更新又可以通过`maxSurge`和`maxUnavailable`字段来控制升级pod的速率，具体可以详细看[官网定义](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)。：

- [**maxSurge:**](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-surge)最大峰值，用来指定可以创建的超出期望Pod个数的Pod数量。
- [**maxUnavailable:**](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#max-unavailable,)最大不可用，用来指定更新过程中不可用的Pod的个数上限。

我们先输入命令回滚我们的deployment，输入`kubectl describe pod`会发现deployment已经把`v2`版本的pod回滚到`v1`的版本。

``` shell
kubectl rollout undo deployment hellok8s-deployment

kubectl get pods                                    
# NAME                                   READY   STATUS    RESTARTS   AGE
# hellok8s-deployment-77bffb88c5-cvm5c   1/1     Running   0          39s
# hellok8s-deployment-77bffb88c5-lktbl   1/1     Running   0          41s
# hellok8s-deployment-77bffb88c5-nh82z   1/1     Running   0          37s

kubectl describe pod hellok8s-deployment-77bffb88c5-cvm5c
# Image: guangzhengli/hellok8s:v1
```

除了上面的命令，还可以用`history`来查看历史版本，`--to-revision=2`来回滚到指定版本。

```shell
kubectl rollout history deployment hellok8s-deployment
kubectl rollout undo deployment/hellok8s-deployment --to-revision=2
```

接着设置`strategy=rollingUpdate`,`maxSurge=1`,`maxUnavailable=1`和`replicas=3`到deployment.yaml文件中。这个参数配置意味着最大可能会创建4个hellok8s pod (replicas + maxSurge)，最小会有2个hellok8s pod存活(replicas-maxUnavailable)。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s-deployment
spec:
  strategy:
     rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  replicas: 3
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
      - image: guangzhengli/hellok8s:v2
        name: hellok8s-container
```

使用`kubectl apply -f deployment.yaml`来重新创建`v2`的资源，可以通过`kubectl get pods --watch`来观察pod的创建销毁情况，是否如下图所示。

![rollingupdate](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/rollingupdate.png)

### 存活探针 (livenessProb)

> 存活探测器来确定什么时候要重启容器。例如，存活探测器可以探测到应用死锁（应用程序在运行，但是无法继续执行后面的步骤）情况。重启这种状态下的容器有助于提高应用的可用性，即使其中存在缺陷。-- [LivenessProb](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

在生产中，有时候因为某些bug导致应用死锁或者线程耗尽了，最终会导致应用无法继续提供服务，这个时候如果没有手段来自动监控和处理这一问题的话，可能会导致很长一段时间无人发现。[kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)使用存活探测器(livenessProb)来确定什么时候要重启容器。

接下来我们写一个`/healthz`接口来说明`livenessProb`如何使用。`/healthz`接口会在启动成功的15s内正常返回200状态码，在15s后，会一直返回500的状态码。

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"time"
)

func hello(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "[v2] Hello, Kubernetes!")
}

func main() {
	started := time.Now()
	http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
		duration := time.Since(started)
		if duration.Seconds() > 15 {
			w.WriteHeader(500)
			w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds())))
		} else {
			w.WriteHeader(200)
			w.Write([]byte("ok"))
		}
	})

	http.HandleFunc("/", hello)
	http.ListenAndServe(":3000", nil)
}
```

```yaml
# Dockerfile
FROM golang:1.16-buster AS builder
RUN mkdir /src
ADD . /src
WORKDIR /src

RUN go env -w GO111MODULE=auto
RUN go build -o main .

FROM gcr.io/distroless/base-debian10

WORKDIR /

COPY --from=builder /src/main /main
EXPOSE 3000
ENTRYPOINT ["/main"]
```

`Dockerfile`的编写和原来保持一致，我们把`tag`修改为`liveness`并推送到远程仓库。

```shell
docker build . -t guangzhengli/hellok8s:liveness
docker push guangzhengli/hellok8s:liveness
```

最后编写deployment的定义，这里使用存活探测方式是使用HTTP GET请求，请求的是刚才定义的`/healthz`接口，`periodSeconds`字段指定了kubelet每隔3秒执行一次存活探测。`initialDelaySeconds`字段告诉kubelet在执行第一次探测前应该等待3秒。如果服务器上`/healthz`路径下的处理程序返回成功代码，则kubelet认为容器是健康存活的。如果处理程序返回失败代码，则kubelet会杀死这个容器并将其重启。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s-deployment
spec:
  strategy:
     rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  replicas: 3
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
        - image: guangzhengli/hellok8s:liveness
          name: hellok8s-container
          livenessProbe:
            httpGet:
              path: /healthz
              port: 3000
            initialDelaySeconds: 3
            periodSeconds: 3
```

通过`get`或者`describe`命令可以发现pod一直处于重启当中。

```shell
kubectl apply -f deployment.yaml

kubectl get pods
# NAME                                   READY   STATUS    RESTARTS     AGE
# hellok8s-deployment-5995ff9447-d5fbz   1/1     Running   4 (6s ago)   102s
# hellok8s-deployment-5995ff9447-gz2cx   1/1     Running   4 (5s ago)   101s
# hellok8s-deployment-5995ff9447-rh29x   1/1     Running   4 (6s ago)   102s

kubectl describe pod hellok8s-68f47f657c-zwn6g

# ...
# ...
# ...
# Events:
#  Type     Reason     Age                   From               Message
#  ----     ------     ----                  ----               -------
#  Normal   Scheduled  12m                   default-scheduler  Successfully assigned default/hellok8s-deployment-5995ff9447-rh29x to minikube
#  Normal   Pulled     11m (x4 over 12m)     kubelet            Container image "guangzhengli/hellok8s:liveness" already present on machine
#  Normal   Created    11m (x4 over 12m)     kubelet            Created container hellok8s-container
#  Normal   Started    11m (x4 over 12m)     kubelet            Started container hellok8s-container
#  Normal   Killing    11m (x3 over 12m)     kubelet            Container hellok8s-container failed liveness probe, will be restarted
#  Warning  Unhealthy  11m (x10 over 12m)    kubelet            Liveness probe failed: HTTP probe failed with statuscode: 500
#  Warning  BackOff    2m41s (x36 over 10m)  kubelet            Back-off restarting failed container
```

### 就绪探针(readiness)

> 就绪探测器可以知道容器何时准备好接受请求流量，当一个Pod内的所有容器都就绪时，才能认为该Pod就绪。这种信号的一个用途就是控制哪个Pod作为Service的后端。若Pod尚未就绪，会被从Service的负载均衡器中剔除。--[ReadinessProb](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

在生产环境中，升级服务的版本是日常的需求，这时我们需要考虑一种场景，即当发布的版本存在问题，就不应该让它升级成功。kubelet使用就绪探测器可以知道容器何时准备好接受请求流量，当一个pod升级后不能就绪，即不应该让流量进入该pod，在配合`rollingUpate`的功能下，也不能允许升级版本继续下去，否则服务会出现全部升级完成，导致所有服务均不可用的情况。

这里我们把服务回滚到`hellok8s:v2`的版本，可以通过上面学习的方法进行回滚。

```shell
kubectl rollout undo deployment hellok8s-deployment --to-revision=2
```

这里我们将应用的`/healthz`接口直接设置成返回500状态码，代表该版本是一个有问题的版本。

```go
package main

import (
	"io"
	"net/http"
)

func hello(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "[v2] Hello, Kubernetes!")
}

func main() {
	http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
			w.WriteHeader(500)
	})

	http.HandleFunc("/", hello)
	http.ListenAndServe(":3000", nil)
}
```

在`build`阶段我们将`tag`设置为`bad`，打包后push到远程仓库。

``` shell
docker build . -t guangzhengli/hellok8s:bad

docker push guangzhengli/hellok8s:bad
```

接着编写deployment资源文件，[Probe](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#probe-v1-core)有很多配置字段，可以使用这些字段精确地控制就绪检测的行为：

- `initialDelaySeconds`：容器启动后要等待多少秒后才启动存活和就绪探测器，默认是0秒，最小值是0。
- `periodSeconds`：执行探测的时间间隔（单位是秒）。默认是10秒。最小值是1。
- `timeoutSeconds`：探测的超时后等待多少秒。默认值是1秒。最小值是1。
- `successThreshold`：探测器在失败后，被视为成功的最小连续成功数。默认值是1。存活和启动探测的这个值必须是1。最小值是1。
- `failureThreshold`：当探测失败时，Kubernetes的重试次数。对存活探测而言，放弃就意味着重新启动容器。对就绪探测而言，放弃意味着Pod会被打上未就绪的标签。默认值是3。最小值是1。

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s-deployment
spec:
  strategy:
     rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  replicas: 3
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
        - image: guangzhengli/hellok8s:bad
          name: hellok8s-container
          readinessProbe:
            httpGet:
              path: /healthz
              port: 3000
            initialDelaySeconds: 1
            successThreshold: 5
```

通过`get`命令可以发现两个pod一直处于还没有Ready的状态当中，通过`describe`命令可以看到是因为`Readiness probe failed: HTTP probe failed with statuscode: 500` 的原因。又因为设置了最小不可用的服务数量为`maxUnavailable=1`，这样能保证剩下两个`v2`版本的`hellok8s`能继续提供服务！

```shell
kubectl apply -f deployment.yaml

kubectl get pods                
# NAME                                   READY   STATUS    RESTARTS   AGE
# hellok8s-deployment-66799848c4-8xzsz   1/1     Running   0          102s
# hellok8s-deployment-66799848c4-m9dl5   1/1     Running   0          102s
# hellok8s-deployment-9c57c7f56-rww7k    0/1     Running   0          26s
# hellok8s-deployment-9c57c7f56-xt9tw    0/1     Running   0          26s


kubectl describe pod hellok8s-deployment-9c57c7f56-rww7k
# Events:
#   Type     Reason     Age                From               Message
#   ----     ------     ----               ----               -------
#   Normal   Scheduled  74s                default-scheduler  Successfully assigned default/hellok8s-deployment-9c57c7f56-rww7k to minikube
#   Normal   Pulled     73s                kubelet            Container image "guangzhengli/hellok8s:bad" already present on machine
#   Normal   Created    73s                kubelet            Created container hellok8s-container
#   Normal   Started    73s                kubelet            Started container hellok8s-container
#   Warning  Unhealthy  0s (x10 over 72s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 500
```

## Service

经过前面几节的练习，可能你会有一些疑惑：

* 为什么pod不就绪(Ready)的话，`kubernetes`不会将流量重定向到该pod，这是怎么做到的？
* 前面访问服务的方式是通过`port-forword`将pod的端口暴露到本地，不仅需要写对pod的名字，一旦deployment重新创建新的pod，pod名字和IP地址也会随之变化，如何保证稳定的访问地址呢？。
* 如果使用deployment部署了多个Pod副本，如何做负载均衡呢？

`kubernetes`提供了一种名叫`Service`的资源帮助解决这些问题，它为pod提供一个稳定的Endpoint。Service位于pod的前面，负责接收请求并将它们传递给它后面的所有pod。一旦服务中的Pod集合发生更改，Endpoints就会被更新，请求的重定向自然也会导向最新的pod。

### ClusterIP

我们先来看看`Service`默认使用的`ClusterIP`类型，首先做一些准备工作，在之前的`hellok8s:v2`版本上加上返回当前服务所在的`hostname`功能，升级到`v3`版本。

``` go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
)

func hello(w http.ResponseWriter, r *http.Request) {
	host, _ := os.Hostname()
	io.WriteString(w, fmt.Sprintf("[v3] Hello, Kubernetes!, From host: %s", host))
}

func main() {
	http.HandleFunc("/", hello)
	http.ListenAndServe(":3000", nil)
}
```

`Dockerfile`和之前保持一致，打包`tag=v3`并推送到远程仓库。

``` shell
docker build . -t guangzhengli/hellok8s:v3

docker push guangzhengli/hellok8s:v3
```

修改deployment的`hellok8s`为`v3`版本。执行`kubectl apply -f deployment.yaml`更新deployment。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
        - image: guangzhengli/hellok8s:v3
          name: hellok8s-container
```

接下来是`Service`资源的定义，我们使用`ClusterIP`的方式定义Service，通过`kubernetes`集群的内部IP暴露服务，当我们只需要让集群中运行的其他应用程序访问我们的pod时，就可以使用这种类型的Service。首先创建一个service-hellok8s-clusterip.yaml文件。

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: service-hellok8s-clusterip
spec:
  type: ClusterIP
  selector:
    app: hellok8s
  ports:
  - port: 3000
    targetPort: 3000
```

首先通过`kubectl get endpoints`来看看Endpoint。被selector选中的Pod，就称为Service的Endpoints。它维护着Pod的IP地址，只要服务中的Pod集合发生更改，Endpoints就会被更新。通过`kubectl get pod -o wide`命令获取Pod更多的信息，可以看到3个Pod的IP地址和Endpoints中是保持一致的，你可以试试增大或减少Deployment中Pod的replicas，观察Endpoints会不会发生变化。

```shell
kubectl apply -f service-hellok8s-clusterip.yaml

kubectl get endpoints
# NAME                         ENDPOINTS                                          AGE
# service-hellok8s-clusterip   172.17.0.10:3000,172.17.0.2:3000,172.17.0.3:3000   10s

kubectl get pod -o wide
# NAME                                   READY   STATUS    RESTARTS   AGE    IP           NODE 
# hellok8s-deployment-5d5545b69c-24lw5   1/1     Running   0          112s   172.17.0.7   minikube 
# hellok8s-deployment-5d5545b69c-9g94t   1/1     Running   0          112s   172.17.0.3   minikube
# hellok8s-deployment-5d5545b69c-9gm8r   1/1     Running   0          112s   172.17.0.2   minikube
# nginx                                  1/1     Running   0          112s   172.17.0.9   minikube

kubectl get service
# NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
# service-hellok8s-clusterip   ClusterIP   10.104.96.153   <none>        3000/TCP   10s
```

接着我们可以通过在集群其它应用中访问`service-hellok8s-clusterip`的IP地址`10.104.96.153`来访问`hellok8s:v3`服务。

这里通过在集群内创建一个`nginx`来访问`hellok8s`服务。创建后进入`nginx`容器来用`curl`命令访问`service-hellok8s-clusterip`。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx-container
      image: nginx
```

```shell
kubectl get pods
# NAME                                   READY   STATUS    RESTARTS   AGE
# hellok8s-deployment-5d5545b69c-24lw5   1/1     Running   0          27m
# hellok8s-deployment-5d5545b69c-9g94t   1/1     Running   0          27m
# hellok8s-deployment-5d5545b69c-9gm8r   1/1     Running   0          27m
# nginx                                  1/1     Running   0          41m

kubectl get service
# NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
# service-hellok8s-clusterip   ClusterIP   10.104.96.153   <none>        3000/TCP   10s

kubectl exec -it nginx-pod /bin/bash
# root@nginx-pod:/# curl 10.104.96.153:3000
# [v3] Hello, Kubernetes!, From host: hellok8s-deployment-5d5545b69c-9gm8r
# root@nginx-pod:/# curl 10.104.96.153:3000
#[v3] Hello, Kubernetes!, From host: hellok8s-deployment-5d5545b69c-9g94t
```

可以看到，我们多次`curl 10.104.96.153:3000`访问`hellok8s`Service IP地址，返回的`hellok8s:v3` `hostname`不一样，说明Service可以接收请求并将它们传递给它后面的所有pod，还可以自动负载均衡。你也可以试试增加或者减少`hellok8s:v3`pod副本数量，观察Service的请求是否会动态变更。调用过程如下图所示：

![service-clusterip-fix-name](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/service-clusterip-fix-name.png)

除了上述的`ClusterIp`的方式外，Kubernetes`ServiceTypes`允许指定你所需要的Service类型，默认是`ClusterIP`。`Type`的值包括如下：

- `ClusterIP`：通过集群的内部IP暴露服务，选择该值时服务只能够在集群内部访问。这也是默认的`ServiceType`。
- [`NodePort`](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)：通过每个节点上的IP和静态端口（`NodePort`）暴露服务。`NodePort`服务会路由到自动创建的`ClusterIP`服务。通过请求`<节点 IP>:<节点端口>`，你可以从集群的外部访问一个`NodePort`服务。
- [`LoadBalancer`](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)：使用云提供商的负载均衡器向外部暴露服务。外部负载均衡器可以将流量路由到自动创建的`NodePort`服务和`ClusterIP`服务上。
- [`ExternalName`](https://kubernetes.io/docs/concepts/services-networking/service/#externalname)：通过返回`CNAME`和对应值，可以将服务映射到`externalName`字段的内容（例如，`foo.bar.example.com`）。无需创建任何类型代理。

### NodePort

我们知道`kubernetes`集群并不是单机运行，它管理着多台节点即[Node](https://kubernetes.io/docs/concepts/architecture/nodes/)，可以通过每个节点上的IP和静态端口（`NodePort`）暴露服务。如下图所示，如果集群内有两台Node运行着`hellok8s:v3`，我们创建一个`NodePort`类型的Service，将`hellok8s:v3`的`3000`端口映射到Node机器的`30000`端口(在30000-32767范围内)，就可以通过访问`http://node1-ip:30000`或者`http://node2-ip:30000`访问到服务。

![service-nodeport-fix-name](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/service-nodeport-fix-name.png)

这里以`minikube`为例，我们可以通过`minikube ip`命令拿到k8s cluster node IP地址。下面的教程都以我本机的`192.168.59.100`为例，需要替换成你的IP地址。

```shell
minikube ip
# 192.168.59.100
```

接着以NodePort的ServiceType创建一个Service来接管pod流量。通过`minikube`节点上的IP`192.168.59.100`暴露服务。`NodePort`服务会路由到自动创建的`ClusterIP`服务。通过请求`<节点 IP>:<节点端口>` -- `192.168.59.100`:30000，你可以从集群的外部访问一个`NodePort`服务，最终重定向到`hellok8s:v3`的`3000`端口。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-hellok8s-nodeport
spec:
  type: NodePort
  selector:
    app: hellok8s
  ports:
  - port: 3000
    nodePort: 30000
```

创建`service-hellok8s-nodeport`Service后，使用`curl`命令或者浏览器访问`http://192.168.59.100:30000`可以得到结果。

```shell
kubectl apply -f service-hellok8s-nodeport.yaml

kubectl get service
# NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
# service-hellok8s-nodeport    NodePort    10.109.188.161   <none>        3000:30000/TCP   28s

kubectl get pods
# NAME                                   READY   STATUS    RESTARTS   AGE
# hellok8s-deployment-5d5545b69c-24lw5   1/1     Running   0          27m
# hellok8s-deployment-5d5545b69c-9g94t   1/1     Running   0          27m
# hellok8s-deployment-5d5545b69c-9gm8r   1/1     Running   0          27m

curl http://192.168.59.100:30000
# [v3] Hello, Kubernetes!, From host: hellok8s-deployment-5d5545b69c-9g94t

curl http://192.168.59.100:30000
# [v3] Hello, Kubernetes!, From host: hellok8s-deployment-5d5545b69c-24lw5
```

### LoadBalancer

[`LoadBalancer`](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer)是使用云提供商的负载均衡器向外部暴露服务。外部负载均衡器可以将流量路由到自动创建的`NodePort`服务和`ClusterIP`服务上，假如你在[AWS](https://aws.amazon.com)的[EKS](https://aws.amazon.com/eks/)集群上创建一个Type为`LoadBalancer`的Service。它会自动创建一个ELB([Elastic Load Balancer](https://aws.amazon.com/elasticloadbalancing))，并可以根据配置的IP池中自动分配一个独立的IP地址，可以供外部访问。

这里因为我们使用的是`minikube`，可以使用`minikube tunnel`来辅助创建LoadBalancer的`EXTERNAL_IP`，具体教程可以查看[官网文档](https://minikube.sigs.k8s.io/docs/handbook/accessing/#loadbalancer-access)，但是和实际云提供商的LoadBalancer还是有本质区别，所以[Repository](https://github.com/guangzhengli/kubernetes_workshop)不做更多阐述，有条件的可以使用[AWS](https://aws.amazon.com)的[EKS](https://aws.amazon.com/eks/)集群上创建一个ELB([Elastic Load Balancer](https://aws.amazon.com/elasticloadbalancing))试试。

下图显示LoadBalancer的Service架构图。

![service-loadbalancer-fix-name](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/service-loadbalancer-fix-name.png)

## ingress

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#ingress-v1beta1-networking-k8s-io)公开从集群外部到集群内[服务](https://kubernetes.io/docs/concepts/services-networking/service/)的HTTP和HTTPS路由。流量路由由Ingress资源上定义的规则控制。Ingress可为Service提供外部可访问的URL、负载均衡流量、SSL/TLS，以及基于名称的虚拟托管。你必须拥有一个[Ingress控制器](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers)才能满足Ingress的要求。仅创建Ingress资源本身没有任何效果。[Ingress控制器](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers)通常负责通过负载均衡器来实现Ingress，例如`minikube`默认使用的是[nginx-ingress](https://minikube.sigs.k8s.io/docs/tutorials/nginx_tcp_udp_ingress/)，目前`minikube`也支持[Kong-Ingress](https://minikube.sigs.k8s.io/docs/handbook/addons/kong-ingress/)。

Ingress可以“简单理解”为服务的网关Gateway，它是所有流量的入口，经过配置的路由规则，将流量重定向到后端的服务。

在`minikube`中，可以通过下面命令开启Ingress-Controller的功能。默认使用的是[nginx-ingress](https://minikube.sigs.k8s.io/docs/tutorials/nginx_tcp_udp_ingress/)。

```shell
minikube addons enable ingress
```

接着删除之前创建的所有`pod`,`deployment`,`service`资源。

``` shell
kubectl delete deployment,service --all
```

接着根据之前的教程，创建`hellok8s:v3`和`nginx`的`deployment`与`service`资源。Service的type为Cluster IP即可。

`hellok8s:v3`的端口映射为`3000:3000`，`nginx`的端口映射为`4000:80`，这里后续写Ingress Route 规则时会用到。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-hellok8s-clusterip
spec:
  type: ClusterIP
  selector:
    app: hellok8s
  ports:
  - port: 3000
    targetPort: 3000

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hellok8s-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
        - image: guangzhengli/hellok8s:v3
          name: hellok8s-container
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-nginx-clusterip
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 4000
    targetPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx-container
```

```shell
kubectl apply -f hellok8s.yaml
# service/service-hellok8s-clusterip created
# deployment.apps/hellok8s-deployment created

kubectl apply -f nginx.yaml   
# service/service-nginx-clusterip created
# deployment.apps/nginx-deployment created

kubectl get pods            
# NAME                                   READY   STATUS    RESTARTS   AGE
# hellok8s-deployment-5d5545b69c-4wvmf   1/1     Running   0          55s
# hellok8s-deployment-5d5545b69c-qcszp   1/1     Running   0          55s
# hellok8s-deployment-5d5545b69c-sn7mn   1/1     Running   0          55s
# nginx-deployment-d47fd7f66-d9r7x       1/1     Running   0          34s
# nginx-deployment-d47fd7f66-hp5nf       1/1     Running   0          34s

kubectl get service
# NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
# service-hellok8s-clusterip   ClusterIP   10.97.88.18      <none>        3000/TCP   77s
# service-nginx-clusterip      ClusterIP   10.103.161.247   <none>        4000/TCP   56s
```

这样在k8s集群中，就有3个`hellok8s:v3`的pod，2个`nginx`的pod。并且`hellok8s:v3`的端口映射为`3000:3000`，`nginx`的端口映射为`4000:80`。在这个基础上，接下来编写Ingress资源的定义，`nginx.ingress.kubernetes.io/ssl-redirect:"false"`的意思是这里关闭`https`连接，只使用`http`连接。

匹配前缀为`/hello`的路由规则，重定向到`hellok8s:v3`服务，匹配前缀为`/`的跟路径重定向到`nginx`。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    # We are defining this annotation to prevent nginx
    # from redirecting requests to `https` for now
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /hello
            pathType: Prefix
            backend:
              service:
                name: service-hellok8s-clusterip
                port:
                  number: 3000
          - path: /
            pathType: Prefix
            backend:
              service:
                name: service-nginx-clusterip
                port:
                  number: 4000

```

```shell
kubectl apply -f ingress.yaml
# ingress.extensions/hello-ingress created

kubectl get ingress          
# NAME            CLASS   HOSTS   ADDRESS   PORTS   AGE
# hello-ingress   nginx   *                 80      16s

# replace 192.168.59.100 by your minikube ip
curl http://192.168.59.100/hello
# [v3] Hello, Kubernetes!, From host: hellok8s-deployment-5d5545b69c-sn7mn

curl http://192.168.59.100/
# (....Thank you for using nginx.....)
```

上面的教程中将所有流量都发送到Ingress中，如下图所示：

![ingress](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/ingress.png)

## Namespace

在实际的开发当中，有时候我们需要不同的环境来做开发和测试，例如`dev`环境给开发使用，`test`环境给QA使用，那么k8s能不能在不同环境`dev` `test` `uat` `prod` 中区分资源，让不同环境的资源独立互相不影响呢，答案是肯定的，k8s提供了名为Namespace的资源来帮助隔离资源。

在Kubernetes中，**名字空间（Namespace）**提供一种机制，将同一集群中的资源划分为相互隔离的组。同一名字空间内的资源名称要唯一，但跨名字空间时没有这个要求。名字空间作用域仅针对带有名字空间的对象，例如Deployment、Service等。

前面的教程中，默认使用的namespace是`default`。

下面展示如何创建一个新的namespace，`namespace.yaml`文件定义了两个不同的namespace，分别是`dev`和`test`。

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  
---

apiVersion: v1
kind: Namespace
metadata:
  name: test
```

可以通过`kubectl apply -f namespaces.yaml`创建两个新的namespace，分别是`dev`和`test`。

```yaml
kubectl apply -f namespaces.yaml
# namespace/dev created
# namespace/test created


kubectl get namespaces
# NAME              STATUS   AGE
# default           Active   215d
# dev               Active   2m44s
# ingress-nginx     Active   110d
# kube-node-lease   Active   215d
# kube-public       Active   215d
# kube-system       Active   215d
# test              Active   2m44s
```

那么如何在新的namespace下创建资源和获取资源呢？只需要在命令后面加上`-n namespace`即可。例如根据上面教程中，在名为`dev`的namespace下创建`hellok8s:v3`的deployment资源。

```shell
kubectl apply -f deployment.yaml -n dev

kubectl get pods -n dev
```

## Configmap

上面的教程提到，我们在不同环境`dev` `test` `uat` `prod`中区分资源，可以让其资源独立互相不受影响，但是随之而来也会带来一些问题，例如不同环境的数据库的地址往往是不一样的，那么如果在代码中写同一个数据库的地址，就会出现问题。

K8S使用ConfigMap来将你的配置数据和应用程序代码分开，将非机密性的数据保存到键值对中。ConfigMap在设计上不是用来保存大量数据的。在ConfigMap中保存的数据不可超过1MiB。如果你需要保存超出此尺寸限制的数据，你可能考虑挂载存储卷。

下面我们可以来看一个例子，我们修改之前代码，假设不同环境的数据库地址不同，下面代码从环境变量中获取`DB_URL`，并将它返回。

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
)

func hello(w http.ResponseWriter, r *http.Request) {
	host, _ := os.Hostname()
	dbURL := os.Getenv("DB_URL")
	io.WriteString(w, fmt.Sprintf("[v4] Hello, Kubernetes! From host: %s, Get Database Connect URL: %s", host, dbURL))
}

func main() {
	http.HandleFunc("/", hello)
	http.ListenAndServe(":3000", nil)
}
```

构建`hellok8s:v4`的镜像，推送到远程仓库。并删除之前创建的所有资源。

```shell
docker build . -t guangzhengli/hellok8s:v4
docker push guangzhengli/hellok8s:v4

kubectl delete deployment,service,ingress --all
```

接下来创建不同namespace的configmap来存放`DB_URL`。

创建`hellok8s-config-dev.yaml`文件

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hellok8s-config
data:
  DB_URL: "http://DB_ADDRESS_DEV"
```

创建`hellok8s-config-test.yaml`文件

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hellok8s-config
data:
  DB_URL: "http://DB_ADDRESS_TEST"
```

分别在`dev` `test`两个namespace下创建相同的`ConfigMap`，名字都叫hellok8s-config，但是存放的Pair对中Value值不一样。

```shell
kubectl apply -f hellok8s-config-dev.yaml -n dev
# configmap/hellok8s-config created

kubectl apply -f hellok8s-config-test.yaml -n test 
# configmap/hellok8s-config created

kubectl get configmap --all-namespaces
NAMESPACE         NAME                                 DATA   AGE
dev               hellok8s-config                      1      3m12s
test              hellok8s-config                      1      2m1s
```

接着使用POD的方式来部署`hellok8s:v4`，其中`env.name`表示的是将configmap中的值写进环境变量，这样代码从环境变量中获取`DB_URL`，这个KEY名称必须保持一致。`valueFrom`代表从哪里读取，`configMapKeyRef`这里表示从名为`hellok8s-config`的`configMap`中读取`KEY=DB_URL`的Value。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hellok8s-pod
spec:
  containers:
    - name: hellok8s-container
      image: guangzhengli/hellok8s:v4
      env:
        - name: DB_URL
          valueFrom:
            configMapKeyRef:
              name: hellok8s-config
              key: DB_URL
```

下面分别在`dev` `test`两个namespace下创建`hellok8s:v4`，接着通过`port-forward`的方式访问不同namespace的服务，可以看到返回的`Get Database Connect URL: http://DB_ADDRESS_TEST`是不一样的！

```shell
kubectl apply -f hellok8s.yaml -n dev
# pod/hellok8s-pod created

kubectl apply -f hellok8s.yaml -n test
# pod/hellok8s-pod created

kubectl port-forward hellok8s-pod 3000:3000 -n dev

curl http://localhost:3000
# [v4] Hello, Kubernetes! From host: hellok8s-pod, Get Database Connect URL: http://DB_ADDRESS_DEV

kubectl port-forward hellok8s-pod 3000:3000 -n test

curl http://localhost:3000
# [v4] Hello, Kubernetes! From host: hellok8s-pod, Get Database Connect URL: http://DB_ADDRESS_TEST
```

## Secret

上面提到，我们会选择以configmap的方式挂载配置信息，但是当我们的配置信息需要加密的时候，configmap就无法满足这个要求。例如上面要挂载数据库密码的时候，就需要明文挂载。

这个时候就需要Secret来存储加密信息，虽然在资源文件的编码上，只是通过Base64的方式简单编码，但是在实际生产过程中，可以通过pipeline或者专业的[AWS KMS](https://aws.amazon.com/kms/)服务进行密钥管理。这样就大大减少了安全事故。

> Secret是一种包含少量敏感信息例如密码、令牌或密钥的对象。由于创建Secret可以独立于使用它们的Pod，因此在创建、查看和编辑Pod的工作流程中暴露Secret（及其数据）的风险较小。Kubernetes和在集群中运行的应用程序也可以对Secret采取额外的预防措施，例如避免将机密数据写入非易失性存储。
>
> 默认情况下，Kubernetes Secret未加密地存储在API服务器的底层数据存储（etcd）中。任何拥有API访问权限的人都可以检索或修改Secret，任何有权访问etcd的人也可以。此外，任何有权限在命名空间中创建Pod的人都可以使用该访问权限读取该命名空间中的任何Secret；这包括间接访问，例如创建Deployment的能力。
>
> 为了安全地使用Secret，请至少执行以下步骤：
>
> 1. 为Secret[启用静态加密](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)；
> 2. [启用或配置RBAC规则](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)来限制读取和写入Secret的数据（包括通过间接方式）。需要注意的是，被准许创建Pod的人也隐式地被授权获取Secret内容。
> 3. 在适当的情况下，还可以使用RBAC等机制来限制允许哪些主体创建新Secret或替换现有Secret。

Secret的资源定义和ConfigMap结构基本一致，唯一区别在于kind是`Secret`，还有Value需要Base64编码，你可以通过下面命令快速Base64编解码。当然Secret也提供了一种`stringData`，可以不需要Base64编码。

```shell
echo "db_password" | base64
# ZGJfcGFzc3dvcmQK

echo "ZGJfcGFzc3dvcmQK" | base64 -d
# db_password
```

这里将Base64编码过后的值，填入对应的key-value中。

```yaml
# hellok8s-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: hellok8s-secret
data:
  DB_PASSWORD: "ZGJfcGFzc3dvcmQK"
```

```yaml
# hellok8s.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hellok8s-pod
spec:
  containers:
    - name: hellok8s-container
      image: guangzhengli/hellok8s:v5
      env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: hellok8s-secret
              key: DB_PASSWORD
```

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
)

func hello(w http.ResponseWriter, r *http.Request) {
	host, _ := os.Hostname()
	dbPassword := os.Getenv("DB_PASSWORD")
	io.WriteString(w, fmt.Sprintf("[v5] Hello, Kubernetes! From host: %s, Get Database Connect Password: %s", host, dbPassword))
}

func main() {
	http.HandleFunc("/", hello)
	http.ListenAndServe(":3000", nil)
}
```

在代码中读取`DB_PASSWORD`环境变量，直接返回对应字符串。Secret的使用方法和前面教程中ConfigMap基本一致，这里就不再过多赘述。

```shell
docker build . -t guangzhengli/hellok8s:v5

docker push guangzhengli/hellok8s:v5

kubectl apply -f hellok8s-secret.yaml

kubectl apply -f hellok8s.yaml

kubectl port-forward hellok8s-pod 3000:3000
```


## Job

在实际的开发过程中，还有一类任务是之前的资源不能满足的，即一次性任务。例如常见的计算任务，只需要拿到相关数据计算后得出结果即可，无需一直运行。而处理这一类任务的资源就是Job。

> Job会创建一个或者多个Pod，并将继续重试Pod的执行，直到指定数量的Pod成功终止。随着Pod成功结束，Job跟踪记录成功完成的Pod个数。当数量达到指定的成功个数阈值时，任务（即Job）结束。删除Job的操作会清除所创建的全部Pod。挂起Job的操作会删除Job的所有活跃Pod，直到Job被再次恢复执行。
>
> 一种简单的使用场景下，你会创建一个Job对象以便以一种可靠的方式运行某Pod直到完成。当第一个Pod失败或者被删除（比如因为节点硬件失效或者重启）时，Job对象会启动一个新的Pod。

下面来看一个Job的资源定义，其中Kind和metadata.name是资源类型和名字就不再解释，`completions`指的是会创建Pod的数量，每个pod都会完成下面的任务。`parallelism`指的是并发执行最大数量，例如下面就会先创建3个pod并发执行任务，一旦某个pod执行完成，就会再创建新的pod来执行，直到5个pod执行完成，Job才会被标记为完成。

`restartPolicy="OnFailure`的含义和Pod生命周期相关，Pod中的容器可能因为退出时返回值非零，或者容器因为超出内存约束而被杀死等等。如果发生这类事件，并且`.spec.template.spec.restartPolicy="OnFailure"`，Pod则继续留在当前节点，但容器会被重新运行。因此，你的程序需要能够处理在本地被重启的情况，或者要设置`.spec.template.spec.restartPolicy="Never"`。

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  parallelism: 3
  completions: 5
  template:
    spec:
      restartPolicy: OnFailure
      containers:
        - name: echo
          image: busybox
          command:
            - "/bin/sh"
          args:
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
```

通过下面的命令创建job，可以通过`kubectl get pods -w`来观察job创建pod的过程和结果。最后可以通过`logs`命令查看日志。

```shell
kubectl apply -f hello-job.yaml

kubectl get jobs                  
# NAME        COMPLETIONS   DURATION   AGE
# hello-job   5/5           19s        83s

kubectl get pods                      
# NAME                                   READY   STATUS      RESTARTS   AGE
# hello-job--1-5gjjr                     0/1     Completed   0          34s
# hello-job--1-8ffmn                     0/1     Completed   0          26s
# hello-job--1-ltsvm                     0/1     Completed   0          34s
# hello-job--1-mttwv                     0/1     Completed   0          29s
# hello-job--1-ww2qp                     0/1     Completed   0          34s

kubectl logs -f hello-job--1-5gjjr 
# 1
# ...
```

Job完成时不会再创建新的Pod，不过已有的Pod[通常](https://kubernetes.io/docs/concepts/workloads/controllers/job/#pod-backoff-failure-policy)也不会被删除。保留这些Pod使得你可以查看已完成的Pod的日志输出，以便检查错误、警告或者其它诊断性输出。可以使用`kubectl`来删除Job(例如`kubectl delete -f hello-job.yaml`)。当使用`kubectl`来删除Job时，该Job所创建的Pod也会被删除。

## CronJob

*CronJob*可以理解为定时任务，创建基于Cron时间调度的[Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)。

> CronJob用于执行周期性的动作，例如备份、报告生成等。这些任务中的每一个都应该配置为周期性重复的（例如：每天/每周/每月一次）；你可以定义任务开始执行的时间间隔。

Cron时间表语法

```
# ┌───────────── 分钟 (0 - 59)
# │ ┌───────────── 小时 (0 - 23)
# │ │ ┌───────────── 月的某天 (1 - 31)
# │ │ │ ┌───────────── 月份 (1 - 12)
# │ │ │ │ ┌───────────── 周的某天 (0 - 6)（周日到周一；在某些系统上，7 也是星期日）
# │ │ │ │ │                          或者是 sun，mon，tue，web，thu，fri，sat
# │ │ │ │ │
# │ │ │ │ │
# * * * * *
```

用法除了需要加上cron表达式之外，其余基本和Job保持一致。

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cronjob
spec:
  schedule: "* * * * *" # Every minute
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: echo
              image: busybox
              command:
                - "/bin/sh"
              args:
                - "-c"
                - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
```

使用命令和Job也基本保持一致，这里就不过多赘述。

```shell
kubectl apply -f hello-cronjob.yaml
# cronjob.batch/hello-cronjob created

kubectl get cronjob                
# NAME            SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
# hello-cronjob   * * * * *   False     0        <none>          8s

kubectl get pods   
# NAME                                   READY   STATUS      RESTARTS   AGE
# hello-cronjob-27694609--1-2nmdx        0/1     Completed   0          15s
```

## Helm

经过前面的教程，想必你已经对kubernetes的使用有了一定的理解。但是不知道你是否想过这样一个问题，就是我们前面教程中提到的所有资源，包括用`pod`,`deployment`,`service`,`ingress`,`configmap`,`secret`所有资源来部署一套完整的`hellok8s`服务的话，难道需要一个一个的`kubectl apply -f`来创建吗？如果换一个namespace，或者说换一套kubernetes集群部署的话，又要重复性的操作创建的过程吗？

我们平常使用操作系统时，需要安装一个应用的话，可以直接使用`apt`或者`brew`来直接安装，而不需要关心这个应用需要哪些依赖，哪些配置。在使用kubernetes安装应用服务`hellok8s`时，我们自然也希望能够一个命令就安装完成，而提供这个能力的，就是CNCF的毕业项目[Helm](https://github.com/helm/helm)。

> Helm帮助您管理Kubernetes应用——Helm Chart，Helm是查找、分享和使用软件构建[Kubernetes](https://kubernetes.io/)的最优方式。
>
> 复杂性管理——即使是最复杂的应用，Helm Chart依然可以描述，提供使用单点授权的可重复安装应用程序。
>
> 易于升级——随时随地升级和自定义的钩子消除您升级的痛苦。
>
> 分发简单—— Helm Chart很容易在公共或私有化服务器上发版，分发和部署站点。
>
> 回滚——使用`helm rollback`可以轻松回滚到之前的发布版本。

我们通过brew来安装helm。更多方式可以参考[官方文档](https://helm.sh/zh/docs/intro/install/)。

```shell
brew install helm
```

Helm的使用方式可以解释为：Helm安装*charts*到Kubernetes集群中，每次安装都会创建一个新的*release*。你可以在Helm的chart *repositories* 中寻找新的chart。

### 用helm安装hellok8s
开始本节教程前，我们先把之前手动创建的hellok8s相关的资源删除(防止使用helm创建同名的k8s资源失败)。

在尝试自己创建hellok8s helm chart之前，我们可以先来熟悉一下怎么使用helm chart。在这里我先创建好了一个hellok8s（包括会创建deployment,service,ingress,configmaps,secret）的helm chart。通过GitHub actions生成放在了[gh-pages](https://github.com/guangzhengli/k8s-tutorials/tree/gh-pages/)分支下的`index.yaml`文件中。

接着可以使用下面命令进行快速安装，其中`helm repo add`表示将我创建好的hellok8s chart添加到自己本地的仓库当中，`helm install`表示从仓库中安装hellok8s/hello-helm到k8s集群当中。

```shell
helm repo add hellok8s https://guangzhengli.github.io/k8s-tutorials/
# "hellok8s" has been added to your repositories

helm install my-hello-helm hellok8s/hello-helm --version 0.1.0
# NAME: my-hello-helm
# NAMESPACE: default
# STATUS: deployed
# REVISION: 1
```

创建完成后，通过`kubectl get`等命令可以看到所有hellok8s资源都创建成功，`helm`一条命令即可做到之前教程中所有资源的创建！通过`curl`k8s集群的ingress地址，也可以看到返回字符串！

```shell
kubectl get pods
# NAME                                  READY   STATUS    RESTARTS   AGE
# hellok8s-deployment-f88f984c6-k8hpz   1/1     Running   0          15h
# hellok8s-deployment-f88f984c6-nzwg6   1/1     Running   0          15h
# hellok8s-deployment-f88f984c6-s89s7   1/1     Running   0          15h
# nginx-deployment-d47fd7f66-6w76b      1/1     Running   0          15h
# nginx-deployment-d47fd7f66-tsqj5      1/1     Running   0          15h

kubectl get deployments
# NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
# hellok8s-deployment   3/3     3            3           15h
# nginx-deployment      2/2     2            2           15h

kubectl get service
# NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
# kubernetes                   ClusterIP   10.96.0.1        <none>        443/TCP    13d
# service-hellok8s-clusterip   ClusterIP   10.107.198.175   <none>        3000/TCP   15h
# service-nginx-clusterip      ClusterIP   10.100.144.49    <none>        4000/TCP   15h

kubectl get ingress
# NAME               CLASS   HOSTS   ADDRESS     PORTS   AGE
# hellok8s-ingress   nginx   *       localhost   80      15h

kubectl get configmap
# NAME               DATA   AGE
# hellok8s-config    1      15h

kubectl get secret
# NAME                                  TYPE                                  DATA   AGE
# hellok8s-secret                       Opaque                                1      15h
# sh.helm.release.v1.my-hello-helm.v1   helm.sh/release.v1

curl http://192.168.59.100/hello
# [v6] Hello, Helm! Message from helm values: It works with Helm Values[v2]!, From namespace: default, From host: hellok8s-deployment-598bbd6884-ttk78, Get Database Connect URL: http://DB_ADDRESS_DEFAULT, Database Connect Password: db_password
```

### 创建helm charts

在使用已经创建好的hello-helm charts来创建整个hellok8s资源后，你可能还是有很多的疑惑，包括Chart是如何生成和发布的，如何创建一个新的Chart？在这节教程中，我们会尝试自己来创建hello-helm Chart来完成之前的操作。

首先建议使用`helm create`命令来创建一个初始的Chart，该命令默认会创建一些k8s资源定义的初始文件，并且会生成官网推荐的目录结构，如下所示：

```shell
helm create hello-helm

.
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

我们将默认生成在templates目录下面的`yaml`文件删除，用之前教程中`yaml`文件替换它，最终的结构长这样：

```shell
.
├── Chart.yaml
├── Dockerfile
├── _helpers.tpl
├── charts
├── hello-helm-0.1.0.tgz
├── index.yaml
├── main.go
├── templates
│   ├── hellok8s-configmaps.yaml
│   ├── hellok8s-deployment.yaml
│   ├── hellok8s-secret.yaml
│   ├── hellok8s-service.yaml
│   ├── ingress.yaml
│   ├── nginx-deployment.yaml
│   └── nginx-service.yaml
└── values.yaml
```

其中`main.go`定义的是`hellok8s:v6`版本的代码，主要是从系统中拿到message，namespace，dbURL，dbPassword这几个环境变量，拼接成字符串返回。

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
)

func hello(w http.ResponseWriter, r *http.Request) {
	host, _ := os.Hostname()
	message := os.Getenv("MESSAGE")
	namespace := os.Getenv("NAMESPACE")
	dbURL := os.Getenv("DB_URL")
	dbPassword := os.Getenv("DB_PASSWORD")

	io.WriteString(w, fmt.Sprintf("[v6] Hello, Helm! Message from helm values: %s, From namespace: %s, From host: %s, Get Database Connect URL: %s, Database Connect Password: %s", message, namespace, host, dbURL, dbPassword))
}

func main() {
	http.HandleFunc("/", hello)
	http.ListenAndServe(":3000", nil)
}
```

为了让大家更加了解helm charts values的使用和熟悉k8s资源配置，这几个环境变量`MESSAGE`,`NAMESPACE`,`DB_URL`,`DB_PASSWORD`分别有不同的来源。

首先修改根目录下的`values.yaml`文件，定义自定义的配置信息，从之前教程的k8s资源文件中，将一些易于变化的参数提取出来，放在`values.yaml`文件中。全部配置信息如下所示：

```yaml
application:
  name: hellok8s
  hellok8s:
    image: guangzhengli/hellok8s:v6
    replicas: 3
    message: "It works with Helm Values!"
    database:
      url: "http://DB_ADDRESS_DEFAULT"
      password: "db_password"
  nginx:
    image: nginx
    replicas: 2
```

那自定义好了这些配置后，如何在k8s资源定义中使用这些配置信息呢？Helm默认使用[Go template的方式](https://helm.sh/zh/docs/howto/charts_tips_and_tricks/)来完成。

例如之前教程中，将环境变量`DB_URL`定义在k8s configmaps中，那么该资源可以定义成如文件所示`hellok8s-configmaps.yaml`。其中`metadata.name`的值是`{{ .Values.application.name }}-config`，意思是从`values.yaml`文件中获取`application.name`的值`hellok8s`，拼接`-config`字符串，这样创建出来的configmaps资源名称就是`hellok8s-config`。

同理`{{ .Values.application.hellok8s.database.url }}`就是获取值为`http://DB_ADDRESS_DEFAULT`放入configmaps对应key为DB_URL的value中。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Values.application.name }}-config
data:
  DB_URL: {{ .Values.application.hellok8s.database.url }}
```

上面定义的最终效果和之前在`configmaps`教程中定义的资源没有区别，这种做的好处是可以将所有可变的参数定义在`values.yaml`文件中，使用该helm charts的人无需了解具体k8s的定义，只需改变成自己想要的参数，即可创建自定义的资源！

同样，我们可以根据之前的教程将`DB_PASSWORD`放入secret中，并且通过`b64enc`方法将值Base64编码。

```shell
# hellok8s-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Values.application.name }}-secret
data:
  DB_PASSWORD: {{ .Values.application.hellok8s.database.password | b64enc }}
```

最后，修改`hellok8s-deployment`文件，根据前面的教程，将`metadata.name` `replicas` `image` `configMapKeyRef.name` `secretKeyRef.name`等值修改成从`values.yaml`文件中获取。

再添加代码中需要的`NAMESPACE`环境变量，从`.Release.Namespace`[内置对象](https://helm.sh/zh/docs/chart_template_guide/builtin_objects/)中获取。最后添加`MESSAGE`环境变量，直接从`{{ .Values.application.hellok8s.message }}`中获取。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.application.name }}-deployment
spec:
  replicas: {{ .Values.application.hellok8s.replicas }}
  selector:
    matchLabels:
      app: hellok8s
  template:
    metadata:
      labels:
        app: hellok8s
    spec:
      containers:
        - image: {{ .Values.application.hellok8s.image }}
          name: hellok8s-container
          env:
            - name: DB_URL
              valueFrom:
                configMapKeyRef:
                  name: {{ .Values.application.name }}-config
                  key: DB_URL
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ .Values.application.name }}-secret
                  key: DB_PASSWORD
            - name: NAMESPACE
              value: {{ .Release.Namespace }}
            - name: MESSAGE
              value: {{ .Values.application.hellok8s.message }}
```

修改`ingress.yaml`将`metadata.name`的值，其它没有变化

``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Values.application.name }}-ingress
...
...
...
```

`nginx-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: {{ .Values.application.nginx.replicas }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: {{ .Values.application.nginx.image }}
        name: nginx-container
```

`nginx-service.yaml`和`hellok8s-service.yaml`没有变化。所有代码可以在[这里](https://github.com/guangzhengli/k8s-tutorials/tree/main/helm-charts/hello-helm)查看。

稍微修改下默认生成的`Chart.yaml`

```yaml
apiVersion: v2
name: hello-helm
description: A k8s tutorials in https://github.com/guangzhengli/k8s-tutorials
type: application
version: 0.1.0
appVersion: "1.16.0"
```

定义完成所有的helm资源后，首先**将`hellok8s:v6`镜像打包推送到DockerHub**。

之后即可在`hello-helm`的目录下执行`helm upgrade`命令进行安装，安装成功后，执行curl命令便能直接得到结果！查看pod和service等资源，便会发现helm能一键安装所有资源！

```shell
helm upgrade --install hello-helm --values values.yaml .
# Release "hello-helm" does not exist. Installing it now.
# NAME: hello-helm
# NAMESPACE: default
# STATUS: deployed
# REVISION: 1

curl http://192.168.59.100/hello
# [v6] Hello, Helm! Message from helm values: It works with Helm Values!, From namespace: default, From host: hellok8s-deployment-57d7df7964-m6gcc, Get Database Connect URL: http://DB_ADDRESS_DEFAULT, Database Connect Password: db_password

kubectl get pods
# NAME                                  READY   STATUS    RESTARTS   AGE
# hellok8s-deployment-f88f984c6-k8hpz   1/1     Running   0          32m
# hellok8s-deployment-f88f984c6-nzwg6   1/1     Running   0          32m
# hellok8s-deployment-f88f984c6-s89s7   1/1     Running   0          32m
# nginx-deployment-d47fd7f66-6w76b      1/1     Running   0          32m
# nginx-deployment-d47fd7f66-tsqj5      1/1     Running   0          32m
```

#### rollback

Helm也提供了Rollback的功能，我们先修改一下`message: "It works with Helm Values[v2]!"`加上[v2]。

```
application:
  name: hellok8s
  hellok8s:
    image: guangzhengli/hellok8s:v6
    replicas: 3
    message: "It works with Helm Values[v2]!"
    database:
      url: "http://DB_ADDRESS_DEFAULT"
      password: "db_password"
  nginx:
    image: nginx
    replicas: 2
```

再执行`helm upgrade`命令更新k8s资源，通过`curl http://192.168.59.100/hello`可以看到资源已经更新。

```shell
➜  hello-helm git:(main) ✗ helm upgrade --install hello-helm --values values.yaml .
# Release "hello-helm" has been upgraded. Happy Helming!
NAME: hello-helm
NAMESPACE: default
STATUS: deployed
REVISION: 2

curl http://192.168.59.100/hello
# [v6] Hello, Helm! Message from helm values: It works with Helm Values[v2]!, From namespace: default, From host: hellok8s-deployment-598bbd6884-4b9bw, Get Database Connect URL: http://DB_ADDRESS_DEFAULT, Database Connect Password: db_password
```

如果这一次更新有问题的话，可以通过`helm rollback`快速回滚。通过下面命令看到，和deployment的rollback一样，回滚后REVISION版本都会增大到3而不是回滚到1，回滚后使用`curl`命令返回的v1版本的字符串。

```shell
helm ls
# NAME            NAMESPACE       REVISION          STATUS          CHART                   APP VERSION
# hello-helm      default         2                 deployed        hello-helm-0.1.0        1.16.0 

helm rollback hello-helm 1
# Rollback was a success! Happy Helming!

helm ls
# NAME            NAMESPACE       REVISION          STATUS          CHART                   APP VERSION
# hello-helm      default         3                 deployed        hello-helm-0.1.0        1.16.0 

curl http://192.168.59.100/hello
# [v6] Hello, Helm! Message from helm values: It works with Helm Values!, From namespace: default, From host: hellok8s-deployment-57d7df7964-482xw, Get Database Connect URL: http://DB_ADDRESS_DEFAULT, Database Connect Password: db_password
```

#### 多环境配置

使用Helm也很容易多环境部署，新建`values-dev.yaml`文件，里面内容自定义`dev`环境需要的配置信息。

```yaml
application:
  hellok8s:
    message: "It works with Helm Values values-dev.yaml!"
    database:
      url: "http://DB_ADDRESS_DEV"
      password: "db_password_dev"
```

可以多次指定'--values -f'参数，最后（最右边）指定的文件优先级最高，这里最右边的是`values-dev.yaml`文件，所以`values-dev.yaml`文件中的值会覆盖`values.yaml`中相同的值，`-n dev`表示在名字为dev的namespace中创建k8s资源，执行完成后，我们可以通过`curl`命令看到返回的字符串中读取的是`values-dev.yaml`文件的配置，并且`From namespace = dev`。

```shell
helm upgrade --install hello-helm -f values.yaml -f values-dev.yaml -n dev .

# Release "hello-helm" does not exist. Installing it now.
# NAME: hello-helm
# NAMESPACE: dev
# STATUS: deployed
# REVISION: 1

curl http://192.168.59.100/hello
# [v6] Hello, Helm! Message from helm values: It works with Helm Values values-dev.yaml!, From namespace: dev, From host: hellok8s-deployment-f5fff9df-89sn6, Get Database Connect URL: http://DB_ADDRESS_DEV, Database Connect Password: db_password_dev

kubectl get pods -n dev
# NAME                                 READY   STATUS    RESTARTS   AGE
# hellok8s-deployment-f5fff9df-89sn6   1/1     Running   0          4m23s
# hellok8s-deployment-f5fff9df-tkh6g   1/1     Running   0          4m23s
# hellok8s-deployment-f5fff9df-wmlpb   1/1     Running   0          4m23s
# nginx-deployment-d47fd7f66-cdlmf     1/1     Running   0          4m23s
# nginx-deployment-d47fd7f66-cgst2     1/1     Running   0          4m23s
```

除此之外，还可以使用'--set-file'设置独立的值，类似于`helm upgrade --install hello-helm -f values.yaml -f values-dev.yaml --set application.hellok8s.message="It works with set helm values" -n dev .` 方式在命令中设置values的值，可以随意修改相关配置，此方法在CICD中经常用到。

### helm chart打包和发布

上面的例子展示了我们可以用一行命令在一个新的环境中安装所有需要的k8s资源！那么如何将helm chart打包、分发和下载呢？在官网中，提供了两种教程，一种是以[GCS存储的教程](https://helm.sh/zh/docs/howto/chart_repository_sync_example/)，还有一种是以[GitHub Pages存储的教程](https://helm.sh/zh/docs/howto/chart_releaser_action/)。

这里我们使用第二种，并且使用[chart-releaser-action](https://github.com/helm/chart-releaser-action)来做自动发布，该action会默认生成helm chart发布到`gh-pages`分支上，本教程的hellok8s helm chart就发布在了本仓库的[gh-pages](https://github.com/guangzhengli/k8s-tutorials/tree/gh-pages/)分支上的`index.yaml`文件中。


在使用action自动生成chart之前，我们可以先熟悉一下如何手动生成，在`hello-helm`目录下，执行`helm package`将chart目录打包到chart归档中。`helm repo index`命令可以基于包含打包chart的目录，生成仓库的索引文件`index.yaml`。

最后，可以使用`helm upgrade --install *.tgz`命令将该指定包进行安装使用。

```shell
helm package hello-helm
# Successfully packaged chart and saved it to: /Users/guangzheng.li/workspace/k8s-tutorials/hello-helm/hello-helm-0.1.0.tgz

helm repo index .

helm upgrade --install hello-helm hello-helm-0.1.0.tgz
```

基于上面的步骤，你可能已经想到，所谓的helm打包和发布，就是`hello-helm-0.1.0.tgz`文件和`index.yaml`生成和上传的一个过程。而helm下载和安装，就是如何将`.tgz`和`index.yaml`文件下载和`helm upgrade --install`的过程。

接下来我们发布生成的hellok8s helm chart，先将手动生成的`hello-helm-0.1.0.tgz`和`index.yaml`文件删除，后续使用GitHub action自动生成和发布这两个文件。

GitHub action的代码可以参考[官网文档](https://helm.sh/zh/docs/howto/chart_releaser_action/)或者本仓库`.github/workflows/release.yml`文件。代表当push代码到远程仓库时，将`helm-charts`目录下的所有charts自动打包和发布到`gh-pages`分支去(需要保证`gh-pages`分支已经存在，否则会报错)。


```yaml
name: Release Charts

on:
  push:
    branches:
      - main

jobs:
  release:
    # depending on default permission settings for your org (contents being read-only or read-write for workloads), you will have to add permissions
    # see: https://docs.github.com/en/actions/security-guides/automatic-token-authentication#modifying-the-permissions-for-the-github_token
    permissions:
      contents: write
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Install Helm
        uses: azure/setup-helm@v1
        with:
          version: v3.8.1

      - name: Run chart-releaser
        uses: helm/chart-releaser-action@v1.4.0
        with: 
          charts_dir: helm-charts
        env:
          CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

接着配置仓库的 `Settings -> Pages -> Build and deployment -> Branch`，选择 `gh-pages` 分支，GitHub 会自动在 `https://username.github.io/project` 发布 helm chart。

最后，你可以将自己的helm charts发布到社区中去，可以考虑发布到[ArtifactHub](https://artifacthub.io/)中，像本仓库生成的helm charts即发布在[ArtifactHub hellok8s](https://artifacthub.io/packages/helm/hellok8s/hello-helm)中。

![tnvYFS](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/tnvYFS.png)

## Dashboard

### kubernetes dashboard

> Dashboard是基于网页的Kubernetes用户界面。你可以使用Dashboard将容器应用部署到Kubernetes集群中，也可以对容器应用排错，还能管理集群资源。你可以使用Dashboard获取运行在集群中的应用的概览信息，也可以创建或者修改Kubernetes资源（如Deployment，Job，DaemonSet等等）。例如，你可以对Deployment实现弹性伸缩、发起滚动升级、重启Pod或者使用向导创建新的应用。

在本地minikube环境，可以直接通过下面命令开启Dashboard。更多用法可以参考官网或者自行探索。

```shell
minikube dashboard
```

![eB3MYd](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/eB3MYd.png)

### K9s

[K9s](https://k9scli.io/)是一个基于Terminal的轻量级UI，可以更加轻松的观察和管理已部署的k8s资源。使用方式非常简单，安装后输入`k9s`即可开启Terminal Dashboard，更多用法可以参考官网。

![83ybd4](https://cdn.jsdelivr.net/gh/guangzhengli/PicURL@master/uPic/83ybd4.png)
