## 1. 概述

在很久以前，如果想要去部署一个```app```，需要准备一台物理服务器，然后在这台服务器上面安装一个操作系统，然后在操作系统里面去部署这个```app```。这种方式部署非常慢，而且成本比较高，会有很大的资源浪费，只是部署一个```app```就要购买一台服务器，同时也难于迁移和扩展。

随着发展出现了虚拟化的技术，虚拟化技术就是基于计算机的基础上，虚拟化一套物理资源，然后基于虚拟的物理资源安装操作系统。这样一台物理机可以部署多个```app```。

每一个虚拟机都是一个完整的操作系统，要给其分配资源，当虚拟机数量增多的时候操作系统本身消耗的资源也会增多。

容器解决了开发和运维之间的矛盾，在开发和运维之间搭建了一个桥梁，是实现```devops```的最佳解决方案。

容器是对软件和软件依赖的表转化打包，可以实现应用之间的相互隔离，容器是共享同一个```OS Kernel```, 不同的容器是```在OS Kernel```上运行的。

容器是在```app```层面的隔离，虚拟化是在屋里层面做的隔离。虚拟机的实现首先是服务器上创建了虚拟的屋里设备，然后基于虚拟的屋里设备安装操作系统，在操作系统中部署```app```。容器是在服务器上安装```docker```，然后```docker```创建容器，在容器中部署```app```。所以虚拟机中每个虚拟机有自己独立的操作系统，容器中所有容器共享一个操作系统。

也可以把虚拟化和容器结合使用。在虚拟器当中安装```docker```。

```docker```是容器技术的一种实现，也就是说容器技术不只有```docker```。

### 1. 部署软件的问题

如果想让软件运行起来要保证操作系统的设置，各种库和组件的安装都是正确的。比如饲养热带鱼和冷水鱼，冷水鱼适应的水温在```5-30```度，热热带鱼只能适应```22-30```度水温，低于```22```度半小时就冻死了。如何将两种鱼养在一个鱼缸里面呢？为了环境一致，最早采用的是虚拟机，在电脑里面虚拟了一个完全的隔了的虚拟系统。

### 2. 虚拟机

虚拟机就是带环境安装的一种解决方案，它可以在一种操作系统里运行另一种操作系统，问题是资源占用多，冗余步骤多，启动速度慢。

由于虚拟机存在这些缺点，```Linux```发展出另一种虚拟化技术，```Linux```容器 ```Linux Containers``` 简写 ```LXC```。

```Linux```容器不是模拟一个完整的操作系统而是针对进程进行隔离，或者说，在正常进程的外面套了一个保护层，对于容器里面的进程来说，他接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。具备启动快，占用资源少，体积少等特点。

```Docker``` 属于```Linux```的一种封装，提供简单易用的容器使用接口，是目前最流行的```Linux```容器解决方案。```Docker``` 将应用程序与该程序的依赖，打包在一个文件里面，运行这个文件，就会生成一个虚拟容器，程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。

| | 启动时间 | 资源占用 | 安全性 | 使用要求 |
| --- | --- | --- | --- | --- |
| docker | 秒级启动 | 轻量级容器镜像通畅以M为单位 | 由于共享宿主机内核，只是进程隔离，因此隔离性和稳定性不如虚拟机，容器具有一定权限访问宿主机内核，存在一些安全隐患 | 容器共享宿主机内核，可运行在主机的linux的发行版，不用考虑CPU是否支持虚拟化技术 |
| 虚拟机 | 分钟级别启动 | 虚拟机以G为单位 | 由于共享宿主机内核，只是进程隔离，因此隔离性和稳定性不如虚拟机，容器具有一定权限访问宿主机内核，存在一些安全隐患 | KVM基于硬件的完全虚拟化，需要硬性CPU虚拟化技术支持 |

### 3. docker的使用场景

节省项目环境部署时间，可实现单项目打包和整套项目打包。支持环境一致性、持续集成、微服务、弹性伸缩等特点。

## 2. 安装

```docker```是```2013```年退出的，但是在此之前的```2004```年和```2008```，容器技术就已经开始在```linux```中使用了。

```docker```底层就是基于```2008```年的```LXC```实现的。```docker```分为企业版和社区版，企业版是收费的。

```s
https://docs.docker.com/install/linux/docker-ce/centos/

https://blog.csdn.net/yimenglin/article/details/93718784
```

```s
# 移除旧版docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

curl -fsSL https://get.docker.com
```

安装依赖包

```s
sudo yum install -y yum-utils
```

设置阿里云镜像源

```s
sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

更新```yum```软件源缓存，并安装```docker-ce```

```s
sudo yum install docker-ce docker-ce-cli containerd.io
```

```s
# 停止docker
systemctl stop docker
# 启动docker
systemctl start docker
# 查看docker版本, 会分别展示client和server的版本。
docker version
# 验证docker是否安装成功
docker run hello-world
```

## 3. 镜像和容器

对于```docker```的架构和底层技术来讲，随着学习和慢慢理解以后，会对```docker```的架构和技术会有更深入的了解。

```docker```是一个```platform```，提供了一个打包，开发，运行```app```的平台。这个平台将底层的物理设备和上层的```app```隔离开了，在```docker```之上来做事情。

```docker engine```有一个后台进程，他提供了一个```rest```的```api```接口，然后```docker engine```还有一个```cli```接口，```docker```是一个```cs```的架构，```cli```和后台进程使用过```rest```来通信的。

通过可以查看```docker```后台的进程

```s
ps -ef | grep docker
```

### 1. Image镜像

```Image```是分层的每一层都可以添加和删除文件，成为一个新的```Image```。不同的```Image```可以共享相同的```layer```。```Image```是只读的。

查看本机的```Image```

```s
docker Image ls
```

可以通过```dockerfile```文件定义```docker```的```Image```，使用```docker build```来打包一个新的```Image```。

```s
docker build -t 
```

也可以从```Registry```中获取一个```docker```, 比如下载```ubuntu:14.04```这个```Image```。这和```github```类似，可以使用```pull```从```hub.docker```中拉取```Image```。

```s
docker pull ubuntu:14.04
```

也可以将自己的```docker```文件```push```到```dockerhub```。

### 2. 自制一个Image

这里以一个简单的```Hello World```程序为例, 制作一个```Image```。

要运行```hello world```需要有```hello world```的程序，这里使用```C```语言来编写这个程序。

首先需要创建一个目录，这里叫```hello-word```。然后在这个目录里面创建一个文件，比如叫```hello.c```。

```s
mkdir hello-word
cd hello-word
vim hello.c
```

然后在这个文件里面书写代码，就是使用```printf```输出```hello world```字符串。

```c
# include<stdio.h>

int main() {
    printf("hello world\n");
}
```

有了这个程序以后需要将它编译成一个```2```进制文件，编译```C```语言程序需要使用```gcc```程序。可以使用```yum install gcc```安装```gcc```，还需要安装```glibc-static```。

```s
yum install glibc-static
```

通过```gcc```编译```hello.c```, ```-o```是输出的文件名，这里叫```hello```

```s
gcc -static hello.c -o hello
```

这样在```hello-word```目录下就会多出一个```hello```的文件，这是一个可执行的文件。直接运行他就会执行。

```s
./hello
```

接着将这个```hello```制作成一个```Image```，首先需要创建一个```Dockerfile```文件。

```s
cd hello-word
vim Dockerfile
```

在这个文件的第一行需要书写```FROM```,表示在什么之上，可以在其他的```Image```之上，这里是一个```base```的```Image```所以在```scratch```之上安装，表示从头开始。

使用```ADD```，将```hello```添加到```Image```的跟目录中。

接着运行```CMD```，指定运行的文件，比如这里指定```/hello```。
```s
FROM scratch
ADD hello /
CMD ["/hello"]
```

这是一个非常简单的```dockerfile```，有了这个```dockerfile```以后就可以去```build```一个```Image```了。可以通过```docker build -t``` 指定一个```tag```，比如这里的```tag```叫```yindong/hello-world```, 后面的.表示当前的目录找到```dockerfile```。

```s
docker build -t yindong/hello-world .
```

执行完毕就构建完了，这里因为有三行，所以会执行```step3```步。然后通过d```ocker Image ls```就可以看到构建的```yindong/hello-world```了。

可以使用```docker history ImageId```查看```docker```的分层。这个```id```就是```docker Image ls```中出现的```id```。

这里```FROM```的```scratch```，所以这个默认不算一层，所以打印出来的只有两层。

```s
docker history id
```

现在可以去运行```docker```了，通过```docker run yindong/hello-world```

```s
# docker run Image名字
docker run yindong/hello-world
```

这样就可以打印出```hello world```了。

```docker```的i```mage```其实就是将可执行文件存储起来，运行的时候是共享了宿主机的硬件环境，不过在运行的时候他是独立运行的。

这就是一个简单的```Image```，实际上```nginx```，```mysql```都可以做成```Image```，他们的工作原理和上面演示的也都是一样的。

### 3. Container

```Container```是通过```Image```创建的，也就是说必须有```Image```，然后通过```Image```来创建```Container```，```Container```是在```Image```的基础上增加了一层```Container layer```，这层是可读写的。因为之前讲过```Image```是只读的，```Container```因为要去运行程序和安装软件等所以他需要可写的空间。

类比面向对象的概念```Image```就相当于是类，```Container```就相当于实例。

```Image```负责```app```的存储和分发，```Container```负责运行```app```。

要基于```Image```创建```Container```其实也很简单，就是```docker run Image```的名字就可以了。

可以使用d```ocker container ls```查看当前本地正在运行的容器。

```s
docker container ls
```

```docker container ls -a```可以查看所有的容器包括运行的和退出的。

```s
docker container ls -a
```

可以通过```docker run -it```的方式来交互式的运行```Image```。也就是可以在这里运行命令，读写文件之类的。实际上进入到了```Container```里面。并且在执行```exit```退出容器的时候，之前的操作也会清除。

```s
docker run -it centos
exit
docker --help
```

## 3. 命令

````docker````的命令分为两部分，第一部分是```Management Commands```第二部分是 ```Commands```。

```Management Commands```是对```docker```里面的对象进行管理的。

```s
docker --help
# 查看命令
docker Image
docker container
# 查看container列表
docker container ls -a
# 删除container，id可以不用写全
docker container rm id
```

```Commands```提供的是一些简便的方法，比如使用```docker container ls -a```查看```container```列表。

```s
docker ps -a
# 删除一个container，默认docker的rm就是remove container
docker rm id
# docker Image ls
docker Images
# 删除 Image
docker Image rm id
docker rmi id
```

## 4. 删除container

通过```docker ps -a```会查看到很多的过期```container```，可以使用```docker rm```删除，也可以使用```docker container ls -aq```打印出所有```container id```, 这相当于使用搜索打印第一列。

```s
docker container ls -aq
docker container ls -a | awk {'print$1'}
```

有了这个id可以通过```docker rm $(docker container ls -aq)```删除所有的。

```s
docker rm $(docker container ls -aq)
```

如果更复杂的情况比如删除所有已经退出的```container```，通过筛选退出的容器。然后使用```docker rm```删除掉这些```container```。

```s
# 列出状态为exited的container
docker container ls -f "status=exited" -q
# 删除指定调教的container
docker rm $(docker container ls -f "status=exited" -q)
```

### 2. docker container commit命令

这个命令是创建一个```container```，然后在这个```container```中发生了一些变化，比如说安装了某个软件，这样的话可以把这个已经改变的```container```生成一个新的```Image```，可以简写成```docker commit```

```s
docker container commit
docker commit
```

使用```docker run -it centos```去交互的运行```centos```。这样就有了一个```container```。

```s
docker run -it centos
```

在这个```container```里面去做一些变化，去安装一个vim。

```s
yum install -y vim
```

安装完成之后退出```exit```，退出之后通过```docker container ls -a```找到刚刚运行的```container```。

可以通过```docker commit```命令将这个```container```打包成一个```Image```，这个```Image```基于```centos```并且里面安装好了```vim```。

接收的第一个参数是要```commit```的```container```，第二个参数要```Image```的```REPOSITORY```和```TAG```。

```s
# docker commit container列表中的name 新Image的名字。
docker commit hardcore_ishizaka yindong/centos-vim
```

这个新出现的```Image```大小要比之前的```centos```大一些，可以使用```docker history id```来对比一下```yindong/centos-vim```和```centos```，可以发现他们有很多相同的```layer```。

这是因为```yindong/centos-vim```是基于```centos```的，所以会直接复用```centos```的```Layer```，在最后一层中大概有```150MB```的大小，这是因为安装了```vim```的原因。

这种创建```Image```的方式并不十分提倡，因为如果把这个```Image```发布出去别人拿到这个```Image```并不会知道这个```Image```怎么产生的，这就会有问题，因为这个```Image```很可能包含不安全的内容。

大部分情况下还是建议通过```Dockerfile```的方式创建```Image```。



首先创建一个```docker-centos-vim```的目录在这个目录里面创建一个```Dockerfile```。

```s
mkdir docker-centos-vim
cd docker-centos-vim
vim Dockerfile
```

在这个```Dockerfile```中首先使用```FROM```是```centos```。运行```yum```命令安装```vim```，需要使用```RUN```来执行。

```s
FROM centos
RUN yum install -y vim
```

然后使用```docker build```命令构建Image。

```s
docker build -t yindong/centos-vim-new .
```

这里会产生两层，第一层是引用的```centos```，第二层会创建一个临时的```container```用来安装```vim```，然后在将这个临时的```container```生成```Image```，完成之后```removing```掉这个临时的```container```，这样就创建了一个新的```Image```。

## 5. Dockerfile


```Dockerfile```定义了很多的关键字。

### 1. FROM

用于指定在哪个```Image```之上去```build```的```Image```。

可以选择```scratch```表示从头去只做一个```Image```。

```s
FROM scratch
```

对于```FROM```来讲尽量使用官方的```Image```作为```base Image```。

### 2. LABEL

定义```Image```的```Metadata```信息，比如说版本，作者，描述等信息。

```s
LABEL maintainer="yindong@126.com"
LABEL version="1.0"
LABEL description="This is description"
```

```LABEL```定义的```Metadata```不可缺少，因为对于```Image```来讲是必须存在一些帮助信息，他像是代码里面的注释，来帮助别人使用。

### 3. RUN

```RUN```是非常常用的，因为很多时候需要运行一些命令，一般需要安装一些软件的时候也经常会使用到```RUN```，每运行一次```RUN```对于```Image```来讲都会新生成一层，所以对于```RUN```来说他的最佳实践中要求为了避免无用分层，合并多条命令成一行。

比如说```yum install``` 和 ```yum update```推荐通过```&&```合并成一层。为了美观如果```&&```导致行变得越来越长可以通过反斜线```\```换行，增加美观。

```s
# 使用反斜线换行
RUN yum update && yum install -y vim \
    python-dev

# 注意清理cache
RUN apt-get update && apt-get install -y perl \
    pwgen --no-install-recommends && rm -rf \
    /var/lib/apt/lists/*

RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
```

### 4. WORKDIR

设定当前工作目录，这个有点像```linux```里面通过```cd```去改变目录，在当前的目录下去做一些事情，比如说运行一些程序或者做一些事情，对于```workdir```来说如果通过```workddir```访问一个目录，如果没有这个目录，会自动创建这个目录。

```s
WORKDIR /root
WORKDIR /test
WORKDIR deom
RUN pwd # /test/demo
```

工作中建议使用```WORKDIR```不建议使用```RUN cd```。

### 5. ADD 和 COPY

他们的作用很像都是将本地的文件添加到```Image```里面。不过```ADD```不光可以添加还可以解压缩。比如将```test.tar.gz```添加之后会自动的解压缩。

```s
# 将hello文件添加到/跟目录
ADD hello /
```

如果通过```WORKDIR```改变了目录，```ADD```添加的目录是```WORKDIR```改变之后的目录.

```s
WORKDIR /root
ADD hello test/ # /root/test/hello
```

大部分情况```COPY```要比```ADD```优先去使用，添加远程文件或者目录，多数使用```curl```或者```wget```去下载。

### 6. ENV

设定常量，比如设置```MYSQL_VERSION```为```5.6```，那么在下面的代码中就可以使用```MYSQL_VERSION```这个变量了。尽量使用```ENV```增加可维护性。

```s
ENV MYSQL_VERSION 5.6 # 设置
RUN apt-get install -y mysql-server="${MYSQL_VERSION}" \ # 引用
    && rm -rf /var/lib/apt/lists/*
```

### 7. VOLUME 和 EXPOSE

主要用于存储和网络，后面单独来讲。

### 8. CMD 和 ENTRYPOINT

```RUN```是执行命令并创建新的```Image Layer```。

```CMD```是设置容器启动后默认执行的命令和参数

```ENTRYPOINT```是设置容器启动时运行的命令

这里来对比一下```CMD```和```ENTRYPOINT```，弄懂他们的区别，在此之前需要了解两种格式。第一种称之为```shell```格式，第二种称之为```exec```格式。

```shell```格式将要运行的命令当成一个```shell```命令来执行。

```s
RUN apt-get install -y vim
CMD echo "hello docker"
ENTRYPOINT echo "hello docker"
```

```exec```和```shell```的区别是要使用特定的格式来指明要运行的命令和命令的参数。

```s
RUN ["apt-get", "install", "-y", "vim"]
CMD ["/bin/echo", "hello docker"]
ENTRYPOINT ["/bin/echo", "hello docker"]
```

接着来看下下面这两个```Dockerfile```, 首先定义了依赖的```Image```，然后定义了常量```name```，接着使用```ENTRYPOINT```运行了一个```echo```命令，这里的命令传入了一个参数```name```。

```s
FROM centos
ENV name Docker
ENTRYPOINT echo "hello $name"
```

```s
FROM centos
ENV name Docker
ENTRYPOINT ["/bin/echo", "hello $name"]
```

实际打包之后运行发现通过```exec```的方式并不会将```$name```替换为常量，这是因为```shell```格式运行命令的时候，他执行的是一个```shell```，所以执行的时候可以识别到```ENV```常量，但是```exec```的格式执行```echo```的时候他执行的是```echo```，并不是```shell```也就无法取得变量。

可以通过指定```exec```是通过```shell```方式去执行的，就是在```exec```中指定```bash```，然后通过```-c```将后面的```echo```和```$name```都作为参数.而且只能是一个命令。

```s
FROM centos
ENV name Docker
ENTRYPOINT ["/bin/base", "-c", "echo hello $name"]
```

```CMD```是容器启动的时候默认执行的命令，上面的例子中如果将```ENTRYPOINT```改为```CMD```结果是一样的。如果在```docker run```的时候制定了其它命令，```CMD```命令会被忽略掉，如果定义了多个```CMD```，只有最后一个会执行。

```s
FROM centos
ENV name Docker
CMD echo "hello $name"

# 指明运行的命令
docker run -it [Image] /bin/base
```

```ENTRYPOINT```是让容器以应用程序或者服务的形式运行，不会被忽略，一定会执行。最佳实践是写一个```shell```脚本作为```entrypoint```。比如说启动一个数据库服务。

比如下面这个```mongod```的脚本，首先他```copy```了一个sh脚本到```bin```中，然后```docker-entrypoint.sh```最为启动脚本。

```s
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]
EXPOSE 27017
CMD ["mongod"]
```

实际上在官方的很多```Dockerfile```中都会使用```ENTRYPOINT```。

## 6. Image分发

可以通过```hub.docker```去拉取别人的```Image```，同样自己的```Image```也可以发布到```hub.docker```上。

```hub.docker```是一个类似```github```的网站，它里面存储了很多```Image```，并且拉取```Image```的时候是不需要登录的。但是当发布```Image```的时候是必须要登录的。所以需要注册```hub.docker```的账号。

如果需要将```Image```发布到```hub.docker```，这个```tag```需要时```hub.docker```的```账号/名字```，否则将会没有权限，因为只能向自己的```hub```中```push Image```。

首先需要使用```docker login```去登录，输入用户名和密码。

```s
docker login

# Login successed
```

```push```的方法很简单，就是使用```docker push```命令，参数就是```Image```的名字。会根据```Image```的大小不同```push```的时间也会不同，```push```成功就会会返回一个```id```。这时候去```hub.docker```就可以看到```Image```了。

```s
docker push yindong/hello-world
```

发布成功之后任何一个人都可以拉取```push```的这个```Image```。

```s
docker pull yindong/hello-world
```

但是这样发布的```Image```是有问题的，他还缺乏文档。所以与其分享```Image```不如分享```Dockerfile```。

可以在```create```里面点击```create automated build```, 这个是要把```docker```账号```link```到```github```上，然后在```github```上创建一个仓库，然后将本地用于```build Image```的```Dockerfile```发布的```github```上，然后```hub.docker```和```github```做一个关联，也就是说```github```有```Dockerfile```他就会自动去```clone```获取到这个```Dockerfile```然后```docker.hub```的后台服务器会```build```，这样既提供了```Dockerfile```，而且```Image```又是```docker```服务器```build```的，安全性有所提升。只需要去维护```Dockerfile```就可以了，```hub.docker```会维护打包```Image```。

## 7. 搭建私有docker hub

如果不想将```Image```发布到```hub.docker```, 想要搭建一个私有的```docker registry```，```docker```提供了一个```Image```叫做```registry```，通过这个```Image```就可以在本地或者```linux```服务器搭建一台自己的类似于```docker hub```，只不过这个```docker hub```是没有```UI```界面的，而且他是只供公司内部或者个人使用的```docker hub```。

搭建这个其实是非常简单的, 只需要运行下面这个命令就可以了。通过```registry Image去```创建一个```container```，这个```container```跑起来之后就相当一个```web```服务器。

```s
docker run -d -p 5000:5000 --restart always --name registry registry:2
```

然后就可以使用```docker```的```push``` 或者 ```pull``` 了。

假设有一台```linux```服务器，在这台机器上执行上面的```run```代码。

安装之后可以使用```telnet```尝试访问上面的```linux```, 如果```telnet```不存在可以使用```yum```安装。

```s
telnet xx.xx.xx.xx 5000
```

正常是可以访问通的。使用私有的```docker hub```需要将```Image```的```tag```修改，之前的```tagname```部分需要改为私有```docker hub```服务的```ip:port```。

```s
docker build xx.xx.xx.xx:5000/hello-world .
```

这样就可以通过```docker push```去提交```xx.xx.xx.xx:5000/hello-world```了。不过在```push```之前需要在```/etc/docker```的目录下创建```daemon.json```文件。这个文件中写入受信任的```ip```和```域名```。

```json
{"insecure-registries": ["xx.xx.xx.xx:5000"]}
```

有了这个以后需要在```docker```的```server```文件中添加一行。

```s
vim /lib/systemd/system/docker.service
```

```s
ExecStart=/usr/bin/dockerd
EnvironmentFile=-/etc/docker/daemon.json # 添加这条
ExecReload=/bin/kill -s HUP $MAINPID
```

最后重启```docker service```。

```s
service docker restart
```

这个时候就可以去```push Image```了。也可以将```Image```, ```pull```到本地。

## 8. 停止container

可以使用```stop```停止```container```, ```PORTSID```是运行中的容器```id```。

```s
docker stop PORTSID
```

启动```docker```的时候可以使用```--name```命名。

```s
docker run -d --name=demo Image
# 使用名字删除
docker stop demo
# 启动
docker start demo
```

```inspect```会显示出```container```的详细信息,里面包含很多，包括```network```信息，```logs```等非常重要。

```s
docker inspect PORTSID
```

通过```docker```构建一个工具，测试```linux```系统的工具。

使用```ENTRYPOINT``` + ```CMD```可以实现参数传递。空的```CMD```就可以将```docker run```最后的参数传递给```ENTRYPOINT```命令中使用。

```s
ENTRYPOINT ["/usr/bin/stress"]
CMD []
```

对容器的资源进行限制，```linux```本身资源是有限的，如果运行太多的容器会占用很多，容器会最大的占用主机资源，所以需要对容器进行限制。

## 9. 资源限制

```docker run```的时候是可以指定```cpu```的个数，内存空间的大小。这里来演示一下。

```s
# 指定内存200M
docker run --memory=200M Image
# 限制cpu权重, 也就是占比
docker run --cpu-shares=10 Image
```

## 10. 网络

运行一个```container```，这个```Image```叫做```busybox```是一个很小的```linux```的```Image```。执行一段无限循环的代码。

```s
docker run -d --name test1 busybox /bin/sh -c "while true; do sleep 3600; done"
```

使用```docker exec```进入到这个```container```里面，执行```/bin/sh```的命令。

```s
docker exec -it id /bin/sh

ip a
```

通过```ip a```查看本地的网络接口，一般会有两个，一个是本地```ip```一个是```newwork```。他有```mac```地址和```ip```地址。这就是一个网络的```namespace```。

```container```中的网络和本地是隔离的，每次创建的```docker```都是网络隔离的。

```container```之间是可以ping通的，也就是网络是想通的。

```s
# 查看netns列表
ip netns ls
# 删除指定netns
ip netns delete test1
# 创建netns
ip netns add test1
```

```s
# 查看名称为test1的netnamespace内容。
ip netns exec test1 ip a
# 查看net端口
ip link
# 让端口up起来。单个端口是没办法up起来的，必须要两个才可以。
ip netns exec test1 ip link set dev lo up
```

这里创建两个```test1```和```test2```，然后将他们两个连起来。

首先在机器1上添加```link```

```s
ip link add veth-test1 type veth peer name veth-test2
# 查看的时候可以发现会多出一堆veth链接。
ip link
```

接着将```veth-test1```这个接口添加到```test1```里面去。

```s
ip lint set veth-test1 netns test1
```

然后将```veth-test2```添加到```test2```里面

```s
ip lint set veth-test2 netns test2
```

已经将```linux1```机器中的```test1```和```test2```分别添加了```veth-test1```和```veth-test2```, 但是他们的状态都是```down```，并且没有```ip```地址。

接下来分别给他们配置```ip```地址，可以通过下面的命令给他们分配地址。就是在```test1```和```test2```这两个```netspace```中执行后的命令也就是分配地址。

```s
ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1

ip netns exec test2 ip addr add 192.168.1.2/24 dev veth-test2
```

将这两个端口启动起来，在```test1```里面执行```ip link set dev veth-test1 up```

```s

ip netns exec test1 ip link set dev veth-test1 up

ip netns exec test2 ip link set dev veth-test2 up
```

现在去查看就可以发现```test1```和```test2```已经```up```起来了，并且有```ip```地址。
```s
ip netns exec test1 ip link
# ping一下test2
ip netns exec test1 ping 192.168.1.2
```

可以使用```docker network ls```查看当前机器```中container```的网络。

## 11. 端口的

本地创建一个```nginx```的服务器```container```。

```s
docker run --name web -d nginx

docker ps
```

这里创建的```nginx```的```container```，所以并不能访问到这个服务器，```nginx```的这个```container```他的网络空间是一个独立的```netspace```，有自己的```ip```地址，```nginx```默认启动的是```80```端口。

可以看下``nginx``这个```container```的```ip```地址，可以通过下面这几种命令。

```s
# 默认是链接再bridge上面的
docker network inspect bridge
# 在外面是可以ping通里面的ip的，因为外面是有docker0bridge的，默认container是连在bridge上的。
ping xx.xx.xx.xx
# 使用telnet访问这个ip加端口，nginx默认是80端口, 是可以访问的。
telnet xx.xx.xx.xx 80
# 通过curl这个网站, 这样就会把nginx的界面拉下来。
curl http://xx.xx.xx.xx
```

不只希望```container```只在运行的服务上可以访问，希望其他机器也可以访问。可以端口映射出去。也就是把```container```的端口映射到服务器的端口上。这样在访问```127.0.0.1```也可以访问到这个```docker container```。

创建```container```的时候多加一个参数```-p```, 也就是端口映射，参数格式是```容器里:本地```, 比如容器里的```80```映射到这里的```80```

```s
docker run --name web -d 80:80 -p nginx
# 可以看到ports里面80映射到了80
docker ps
```

这个时候访问本地的```80```端口就可以访问到里面的服务了。

```s
curl 127.0.0.1
```

## 12. host network none network

创建```container```的时候使用```--network```为```none```, 创建一个```test1```的```busybox```容器。

```s
docker run -d --name test1 --network none busybox /bin/sh -c "while true; do sleep 3600; done"
# 查看
docker ps
```

这个容器的```ip```地址，```mac```地址都没有，没有任何网络信息。

```s
docker network inspect none
```

进入到这个容器里面去。可以发现他除了本地的```lo```以外没有网络接口。

```s
docker exec -it test /bin/sh
ip a
```

因为他没有网路接口所以他是一个孤立的容器，没有任何方式可以访问到这个容器，除了```exec```，这样的容器安全性比较高。具体怎么使用我也不知道。

创建一个连接到```host```的容器，首先删除其他的容器，这里指定```network```为```host```。

```s
docker run -d --name test1 --network host busybox /bin/sh -c "while true; do sleep 3600; done"
# 查看
docker network inspect host
```

可以发现这个容器也没有```mac```地址和```ip```地址，进入到这个```docker```里面的时候使用```ip a```可以发现是有网络接口的，而且他的网络接口和主机的接口是相同的。

也就是说通过```host```创建的容器他是没有独立的```net space```的，他共享了主机的```net space```, 这样可能会存在端口冲突。

## 13. 部署复杂的APP

可以将应用拆分部署，比如说```redis```和```mysql```拆分，主应用部署容器，```redis```部署容器，```mysql```也部署容器。因为```redis```和```mysql```不是给外部使用的，所以不需要使用```-p```做端口映射，这样更安全，可以通过容器间网络通信进行访问。

因为```redis```是在容器里面的，外部不知道这个容器的```ip```地址，并且也不需要知道他的```ip```地址，可以通过```link```的方式通过容器的名字访问容器。也就是说```redis```的名字和端口是固定的，只需要告诉新容器这个名字和端口就可以了。

启动第二的时候使用```link```将前一个的名字传入进去。可以使用```-e```给容器传入一个环境变量。

```s
docker run -d -p 5000:5000 --link redis --name flask-redis -e REDIS_HOST=redis ImageName
```

比如创建```busybox```的时候创建一个环境变量```YIN```，值为```yindong```

```s
docker run -d --name test1 -e YIN=yindong busybox /bin/sh -c "while true; do sleep 3600; done"
```

进入这个容器访问一下这个变量。

```s
docker exec it test1 /bin/sh
env # 可以看到设置的YIN=yindong

# os.environ.get('YIN', '127.0.0.1')  py获取变量
```

## 14. 多机器之间的容器通信

首先两台机器```AB```之间是可以通信的，现在```A```机器上有个```01```的容器，```B```机器上有个```02```的容器，```01```和```02```是不可以通信的，但是知道```A```和```B```是可以通信的，所以可以将```01```的数据放在```A```传输给```B```的数据中，传递给```B```,```B```再传递给```02```，同样```B```给```A```的数据也可以存在```02```的数据，再通过```A```给到```01```，这就实现了通信。这种方式一般称为隧道。```VXLAN```的方式。

来演示一下多机通信。首先有```AB```两台机器，每个机器里面有一个容器```01```和```02```。```docker```除了```bridge```还存在```overlay```，可以通过创建```overlay```的方式进行通信。这里需要依赖一个分布式的存储，因为要确定两台机器中的容器```ip```不能相同，这里就需要一个分布式的存储，来告诉不同的服务相同的东西，这种分布式的工具还是很多的，这里选用```etcd```，开源免费。

需要分别在两台机器上安装```etcd```，安装成功之后将两台机器关联起来，然后开始执行。

在```A```机器上创建```overlay```, 执行之后局可以在本地发现一个```demo```的```overlay```

```s
docker network create -d overlay demo
```

这个时候在```B```机器也可以看到```A```机器创建的```demo```，这就是```etcd```做的。

接着要在这网络之上创建一个```container```，链接到这个网络之上。通过```busybox```，然后给他一个名字叫做```test1```，通过```--net```指定网络是```demo```。

```s
sudo docker run -d --name test1 --net demo busybox sh -c "while true; do sleep 3600; done"
# 查看容器
docker ps
```

在```B```上也创建一个容器

```s
sudo docker run -d --name test2 --net demo busybox sh -c "while true; do sleep 3600; done"
# 查看容器
docker ps
```

可以查看两个```container```的```ip```。这样两个```container```是可以互相```ping```通的。

```s
docker netword inspect demo
```

## 15. Docker的持久化存储和数据共享

### 1. Data Volume

一般来讲有些容器会产生一些自己的数据，这些数据不想随着```container```的消失而消失，需要保证数据的安全，这种场景一般用在数据库，比如使用数据库的容器，数据库会产生一些数据，如果```container```删除了数据库丢失了，那就有问题了，针对这种问题使用```Data Volume```。

之前在讲```dockerfile```的时候里面有个关键字是```volume```，这个关键字就是制定容器某一个目录产生的数据要挂载到主机上的某个目录中，并且会创建一个叫做```docker volume```的对象。

官方```docker hub```的```mysql```的```dockerfile```中就存在```volume```关键字。当启动```mysql```容器的时候就会在```linux```的```/var/lib/mysql```存放，不会随着```container```消失而消失。

```s
VOLUME /var/lib/mysql
```

创建一个```mysql```的```container```, 这里传入参数是不使用密码

```s
docker run -d --name mysql1 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql

docker ps
```

可以查看指定容器启动日志，方便查看报错信息。

```s
docker logs mysql1
```

查看```volume```, 这个```volume```就是创建```mysql```的时候产生的，只要```Dockerfile```中书写了就会创建。

```s
docker volume ls
# 删除
docker valume rm VOLUME_NAME
```

```volume```是```mount```到本地的```/var/lib/docker/volumes/.../_data```中

```s
docker volume inspect VOLUME_NAME
```

每创建一个```container```就会新增一个目录。并且删除这个```container```并不会删除掉这个```volume```目录。这也就实现了数据会丢的问题。

可以给```volume```取一个别名方便查看。就是在创建的时候添加一个```-v```参数。将名称改为```mysql```，前面是要改为的名字，后面是```VOLUME```的路径。

```s
docker run -d -v mysql:/var/lib/mysql --name mysql1 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
```

这个时候当删除了```container```，再次创建```container```的时候，如果```-v```的名字相同，他们会继续使用之前的数据。

要在```Dockerfile```里面指定需要持久化数据的路径，这个路径是容器里面的路径，然后在创建容器的时候使用```-v```将这个路径重新命名

### 2. Bind Mouting

他和```data volume```的区别是```data volume```需要在```Dockerfile```中定义，```Bind Mouting```不需要，他只需要在运行容器的时候去指定本地的目录和容器的目录一一对应的关系，通过这种方式可以做一个同步，也就是说本地路径中的文件和运行容器中的路径文件是同步的。

如果本地本地文件做了修改，那么容器中的文件也会做修改，因为他们就是同一个目录的同一个文件。这里演示一下。创建容器的时候使用```-v```将本地的目录映射到```container```里面的目录。比如使用```pwd```将当前目录映射到```/usr/share/nginx/html```

```s
docker run -d -p 80:80 -v $(pwd):/usr/share/nginx/html --name web nginx
```

这样两个目录就共享了，无论修改哪一个都会同步到另一个。因为他们就是一个目录。

## 16. Docker Compose多容器部署

假设创建的应用需要依赖多个```container```，每次部署的时候都要全部手动启动一遍，停止的时候也需要每台手动停止，这样太麻烦了，```Docker Compose```就是一个批处理的工具，可以统一启动容器和停止容器。

可以通过这个文件在这个文件中定义```app```所需要的所有```container```，定义之后可以通过一条命令搞定所有容器的启动，停止操作。

```Docker Compose```是一个基于```Docker```的命令行工具，这个工具可以通过一个```yml```格式的文件去定义多个容器的```docker```的应用。有了```yml```文件之后通过一条命令根据文件中的定义去创建和管理容器。

所以这里最重要的就是这个```yml```文件，这个文件有一个默认的名字叫做```docker-compose.yml```, 可以改成自己需要的。

在这个文件中有三个重要的概念，```Services```，```Networks```，```Volumes```。

```Docker Componse```是有版本的，所以```yml```文件也存在版本，目前有```3```个版本，推荐使用```version3```，不同的版本对应```Docker```不同的版本。```version2```只能用于单机，```version3```可以用于多机，```version1```已经淘汰了。

### 1. Services

一个```service```代表一个```container```，这个```container```可以从```docker hub```的```Image```来创建，或者从本地的```Dockerfile build```出来的```Image```来创建。

```service```的启动类似```docker run```，可以给其指定```network```和```volume```

比如下面这个```services```，他的名字叫做```db```，然后这个```db```的```Image```是从```docker hub```上拉取的```postgres:9.4```, 接着```volumes```映射了一个目录到本地，相当于```-v db-data:/var/lib/postgresql/data```, 定义一个叫做```back-tier```的```network```

```yml
services:
    db: 
        Image: postgres:9.4
        volumes:
            - "db-data:/var/lib/postgresql/data"
        networks:
            - back-tier
```

这相当于使用下面的命令启动容器

```s
docker run -d --network back-tier -v db-data:/var/lib/postgresql/data postgres:9.4
```

还有第二种书写方式，这里的```Imag```e不是从```docker hub```上拉取的, 他是在本地```build```的，这个```build```目录就是要```build```的```Dockerfile```。这里他```links```了```db```和```redis```的容器。一般情况下如果连接到同一个```bridge```中的时候是不需要```links```的。只有适应默认的```th0```的时候才需要```links```。


```yml
services:
    worker:
        build: ./worker
        links:
            - db
            - redis
        networks:
            - back-tier
```

### 2. Volumes

可以在```services```同级别定义```volumes```。

```yml
services:
    db: 
        Image: postgres:9.4
        volumes:
            - "db-data:/var/lib/postgresql/data"
        networks:
            - back-tier
volumes:
    db-data:
```

### 3. networks

可以在```services```同级别定义```networks```, 他会创建一个```dirver```是```bridge```的名字叫做```back-tier```的```network```

```yml
services:
    db: 
        Image: postgres:9.4
        volumes:
            - "db-data:/var/lib/postgresql/data"
        networks:
            - back-tier
networks:
    front-tier:
        driver: birdge
    back-tier:
        driver: bridge
```

这里基于```yml```文件定义一个```wordpress```, 这里第一行是声明版本，这里声明```3```，首先```services```里面定义了两个```service```，第一个是```wordpress```，端口做了一下映射将容器中的```80```端口映射到本地的```8080```端口。可以通过```enviroment```传递环境变量。```networks```指定了容器要连接的网络是```networks```中定义的网络。

```mysql```中的```volumes```映射成```mysql-data```, 同样这里的名字是要在```volumes```创建的。

```yml
version: '3'
services:
    wordpress:
        Image: wordpress
        ports:
            - 8080:80
        enviroment:
            WORDPRESS_DB_HOST: mysql
            WORDPRESS_DB_PASSWORD: root
        networks:
            - my-bridge
    mysql:
        Image: mysql
        environment:
            MYSQL_ROOT_PASSWORD: root
            MYSQL_DATABASE: wordpress
        volumes:
            - mysql-data:/var/lib/mysql
        networks:
            - my-bridge

volumes:
    mysql-data:

networks:
    my-bridge:
        driver: bridge
```

这就是一个比较完整的```docker compose```的```file```，总体上来说还是比较清晰的。

### 4. docker-compose

如果要使用```docker compose```首先需要安装它，他是一个工具，如果使用的是```mac```或者```windows```在安装docker的时候他默认是已经安装好了的。

```s
docker-compose --version
```

```linux```需要手动安装(https://docs.docker.com/compose/install/)

```s
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

赋予可执行的权限。

```s
sudo chmod +x /usr/local/bin/docker-compose
```

设置一下软连接

```s
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

docker-compose --version
```

基于```yml```文件来试验一下。```docker-compose up```的功能是启动```yml```中的```container```。默认```yml```文件的名字是```docker-compose.yml```。也可以使用```-f```指定文件。```-d```会后台执行，如果不加会打印出日志在控制台```debug```的时候可以查看, 关闭就会停止也就是调试模式。

```s
docker-compose -f docker-compose.yml up -d
```

这样就会启动起来两个```container```。

可以使用```stop```命令停止```container```，```down```命令不但会停止，同时也会移除```container```, ```network```, ```volume```

```s
# 启动
docker-compose start
# 停止
docker-compose stop

docker-compose ps
# 查看定义的container和依赖的Image
docker-compose Images
# 进入容器，既然怒mysql的base，这里的mysql是services
docker-compose exec mysql base
```

```docker-compose```会在创建的容器和```network```上添加一些前缀，使用的时候直接用定义的名字就可以，他命令的内部会自动转换。

```yml
version: '3'
services:
    redis:
        Image: redis
    web:
        build:
            context: .
            dockerfile: Dockerfile
        ports:
            - 8080:5000
        environment:
            REDIS_HOST: redis
```

### 5. scale

这是```docker-compose```新增的一个功能，可以实现```docker```容器的扩展，也就是相同的容器启动多少个，用来做负载均衡。比如说让```web```这个容器启动```3```个。

```s
docker-compose up --scale web=3
```

但是这里有个问题，启动```3```个服务他们的端口映射到了一个端口，是有问题的，所以做法是不写端口映射。然他们直接启用自己的```80```端口。然后通过```haproxy```工具将做负载。因为知道每个容器的端口，所以也就很容易。使用```haproxy```负载到所有的```80```端口服务，然后映射到```8080```。

```yml
version: '3'

services:
    redis:
        Image: redis
    
    web:
        build:
            context: .
            dockerfile: Dockerfile
        environment:
            REDIS_HOST: redis
    lib:
        Image: dockercloud/haproxy
        links:
            - web
        prots:
            - 8080:80
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
```

先创建一个

```s
docker-compose up

docker-compose ps
```

启动```3```个```web```，会加上之前的```1```个。所以一共还是```3```个。

```s
docker-compose up --scale web=3 -d
```

这样就实现了一个负载均衡。

```docker-compose build```这个命令是如果配置文件中有些```Image```是需要```build```的可以先通过这个命令把这个```Image```生成出来，这样在```up```的时候会快一些。如果```service```里面发生了变化也可以使用这个命令打包新的```service```再去启动。

```s
docker-compose build
```

注意的是```docker-compose```是一个用于本地开发的工具，并不是用于部署生产环境的工具，他只是方便在本地开发看到结果。

## 17. 容器编排Docker Swarm

所有```docker```的操作都是在本地操作的，也就是在一台机器上执行的。但是实际情况是应用可能部署在很多台机器上，也就是在一个集群部署应用，这就涉及很多的容器。

那这种情况怎么去管理这么多的容器，怎么增加一些容器，如果一个容器```down```掉了怎么自动恢复，怎么样动态更新容器而不影响业务, 如何去监控追踪容器状态，如何去调度容器的创建，怎么去保护隐私数据，等等这些问题都需要去处理。

基于这些需要有一套容器的编排系统，在这种情况下```Swarm```就诞生了，```Swarm```不是唯一做编排的工具，他是```docker```的工具。集成在```docker```里面的，如果想使用它是不需要安装任何东西的。

```Swarm```中有两种角色，```Manager```和```Worker```，```Manager```是大脑而且不止一个，既然不止一个就要进行数据同步，所以内置了一个分布式。```Worker```就是干活的节点，大部分服务都是部署在```Worker```中。

### 1. Service

在```Swarm```中不使用```run```，使用```service```，下面来创建一个```service```, 采用的```Image```是```busybox```。

```s
docker service create --name demo busybox sh -c "while true..."
```

创建之后查看创建的```service```。使用```ps```可以看到容器是运行在```swarm```上面。

```s
docker service ls
# 查看容器
docker service ps demo
# 创建5个，这五个平均分布到clust里面的。他可以确保5是有效的，当有一个down掉会重新起一个替代。
docker service scale demo=5
```

### 2. Routing Meth

Internal：```Container```和```Container```之间的访问通过```overlay```网络通过```VIP```虚拟```ip```进行通信。

Ingress: 如果服务有绑定端口，则此服务可以通过任意```swarm```节点的相应接口访问。

```LVS```实现负载均衡，可以自行查看。
