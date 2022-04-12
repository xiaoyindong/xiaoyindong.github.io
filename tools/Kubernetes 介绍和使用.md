## 1. 概述

```docker```在```2014```年发布第一个版本。

在```2017```年年底之前```docker```并不支持```k8s```，因为```swarm```和 ```k8s``` 是竞争关系，```swarm```是```docker```自带的编排工具。

```docker swarm```分为```manager```节点和```worker```节点，```manager```节点是管理的功能，维持```cluster```状态并且提供对外的接口，可以通过```manager```部署```app```，```services```，```stack```等等。

```k8s```的架构和```docker swarm```类似，分布式的集群大的方向的架构都是类似的。都是有```manager```和```worker```，只不过在```kubernets```中```manager```叫做```master```节点，```worker```叫做```node```，就是单纯的```node```节点。

```master```对外提供接口，可以对整个```kubernets```集群进行操作。

### 1. master

```master```是```k8s```的大脑，主要功能```API Server```是暴露给外界访问的。可以通过```CLI```或者```UI```去操作```API Service```然后和整个集群进行交互操作。

```Scheduler```实现调度工作，假设通过```API```下达一个命令部署应用，这个应用需要两个容器，那这两个容器到底要运行在那个节点上去，这个就是```Scheduler```通过一些调度算法来决定运行在哪个节点。

```Controller```是一个控制，比如说容器要做一个负载均衡，要做一个扩展，控制运行个数等等。

最底层有一个```etcd```，他是一个分布式的```key-value-store```，就是一个存储，存储整个```k8s```的状态和配置。

### 2. node

```node```里面重要的一个概念是```pod```, ```pod```是容器调度的最小单位，pod指的是具有相同的```name space```的一些```container```的组合。

```name space```包含所有的```name space```, 比如说```user-name-space```, ```net-name-space```, 其实主要还是```network name space```。

具有相同的```network name space```组合，容器可能有一个也可能有多个，如果是多个的话他们是共享一个```network space```。

```Docker```，```kubelet```，```kube-proxy```, ```fluentd```。 

```docker```是容器组件，```kubelet```是```master```节点控制```node```节点的代理桥梁。```kube-proxy```和网络有关主要做端口的代理和转发，负载均衡等功能。```fluentd```主要是日志的采集，存储和查询。

## 2. 安装

```minikube```可以快速在本地搭建一个只有一个节点的集群，这个节点既是```master```又是```worker```，可以使用他来进行学习。这个工具可以用在```linux```，```macOS```，```windows```。

```kubeadm```可以方便的在本地搭建一个真正的```k8s```集群，有多个节点，至少一个```master```，一个 ```workder```，节点是可选的。当然节点越多需要的资源也就越多。他的使用要复杂一些。

现在很多云服务商都会提供搭建```k8s```的服务，只需要简单的````UI````界面操作就可以了。比如阿里云，谷歌云等等。

### 1. 在mac上安装

使用```minikube```来演示 ```https://minikube.sigs.k8s.io/docs/start/```, 安装还是比较简单的。

国内的阿里云上也存在很多的可[安装](yq.aliyun.com/articles/221687)的方式。

```s
minikube version
```

启动```minikube```, 这个启动可能会失败，因为服务在国外，可以参考上面的文档设置国内镜像。

```s
minikube start
```

创建之后会在```VBox```（这里用的虚拟机）里面创建一台虚拟机，然后会创建很多的```container```和镜像，这需要一些时间，耐心等待就可以了。

```s
# 查看状态
minikube status
# 进入到创建的虚拟机里面
minikube ssh
# 查看容器
docker ps
```

安装```kubectl```，他可以获取集群的运行信息和创建```k8s```的资源，比如```pod```，```development```。

[安装](https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/)

```s
# 查看信息
kubectl version
# 查看k8s的pod，这里面展示的都是一些核心的组件，是他们组成了k8s
kubectl get pod --all-namespaces
```

停止```minikube```

```s
# 停止
minikube stop
# 删除
minikube delete
# 启动dashboard服务，可以通过网页访问k8s集群
minikube dashboard
```

## 3. Kubernetes中的基本概念和操作

```kubectl```有一个自动补全的方法``completion``，可以使用```--help```查询设置方法

```s
kubectl completion --help
```

设置之后使用```tab```按键就会出现提示方法。

```kubectl```默认会查看```.kube```下面的```config```文件来操作```k8s```, 也可以自己设置这个```config```中的设置来管理```k8s```的操作。可以通过```config```命令查看。

```s
kubectl config current-context
kubectl config get-contexts
```

切换```context```, 可以将不同的```cluster```配置到```config```中，然后通过切换来设置不同的操作。

```s
kubectl config use-context [kubeadm]
```

## 4. 节点和标签

```maste```r节点上会运行核心的功能，比如调度器，在```node```节点运行的是部署应用。

```s
# 查询当前有哪些节点
kubectl get node
# 查询指定节点
kubectl get node k8s-master
# 查询指定节点的详细信息
kubectl describe node k8s-master
```

一般很少查看```node```的详细信息，可以通过```-o``` 添加参数，输出更多的信息。 ```wide```或者```yaml```，```json```

```s
kubectl get node -o wide
```

```node```中有个东西叫做```label```，标签。用于分组, 可以使用下面的命令查看```label```，```label```一般是键值对类型。

```s
# 查看label
get node --show-labels
# 设置label
kubectl label node k8s-master env=test
# 删除label，name-
kubectl label node k8s-master env-
```

```ROLES```是角色，默认只有```master```具有角色```，node```不存在角色，这个是不对的，```roles```是一种特殊的```label```，也可以设置他。

```s
# 设置node1的roles
kubectl label node k8s-node1 node-role.kubernetes.io/worker=
```

## 5. pod

一个或者一组应用容器，他们分享资源比如```volume```，分享相同的命名空间的容器，如网络空间。```pod```是```k8s```里面的最小调度单位，在```k8s```里面不会直接操作```container```，而是操作```pod```，一个```pod```可能包含多个```container```。因为```pod```共享一个```name space```, 所以它里面的```container```可以直接```localhost```通信。

创建```pod```要通过```yml```文件，类似下面的方式, 这里的```king```是```pod```, 这个```spec```里面要声明展示几个```container```。这里启动了```nginx```和```busybox```两个```container```，他们是可以互相通信的。

```yml
apiVersion: v1
kind: pod
metadata:
    name: nginx-busybox
spec:
    containers:
    - name: nginx
        image: nginx
        ports:
        - containerPort: 80
    - name: busybox
        image: busybox
        command: ["/bin/sh"]
        args: ["-c", "while true; do echo hello; sleep 10; done"]
```

有了这个文件就可以通过```kubectl```创建```pod```了。```-f```指定配置文件，

```s
kubectl create -f nginx_busybox.yml
```

创建之后可以对```pod```进行操作。

```s
# 查看pod, 会显示pod的状态和里面container的状态
kubectl get pods
# 获取pod的详细信息
kubectl describe pod nginx-busybox
# 返回更多的信息，ip地址等
kubectl get pods nginx-busybox -o wide
# 进入到pod里面的container里，如果有多个container默认会进入到第一个里面
kubectl exec nginx-busybox -it sh
# 可以通过-c参数指定进入那个container
kubectl exec nginx-busybox -c nginx -it sh
# 删除掉pod
kubectl delete -f nginx_busybox.yml
```

## 6. mamespace命名空间

```namespace```命名空间用于不同```team```，不同```project```之间的隔离，在不同的命名空间中，各种资源的名字是相互独立，比如可以具有相同名称的```pod```存在。

可以通过```kubectl```获取到当前系统与有那些```namespace```, 默认情况下会有```default```，```public```、```system```、```lease```四个```namespace```

```s
kubectl get namespace
```

如果要创建```namespace```可以直接通过```create```就可以了。

```s
kubectl create namespace demo
```

有了```namespace```就可以非常方便的根据```namespace```对资源进行过滤。

```s
kubectl get pod --namespace kube-system
```

可以在```yml```中指定创建的```pod```所在的```namespace```, 通过```namespace```关键字，如果不指定系统会默认把```namespace```放在```default```里面。

```yml
metadata:
    name: nginx
    namespace: demo
```

查看所有的```namespace```

```s
kubectl get pod --all-namespaces
```

## 7. 创建自己的context

可以通过设置改变当前默认的```namespace```，这就可以通过```context```去实现，其实他就是```kubectl```的```config```。

```s
# 创建
kubectl config set-context demo --user=minikube --cluster=minikube --namespace=demo
# 设置
kubectl config use-context demo
# 查看
kubectl config get-contexts
# 删除，要保证不在当前要删除的context中
kubectl config delete=context demo
```
