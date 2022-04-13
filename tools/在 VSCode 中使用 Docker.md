文章参考自极客时间，玩转```VS Code```，禁止任何形式转载，私自转载自行承担法律责任。

## 1. 安装

```VS Code```的```Docker```支持是由插件来完成的，并且这个插件是```VS Code```官方团队维护的，所以它的发布者是```Microsoft```。可以在市场上点击下载，也可以直接在```VS Code```插件视图里搜索```Docker```进行安装。当然了，这个插件的正确运行，离不开一个正确安装的```Docker```环境，也就是当前机器已经正确安装并可以使用```Docker```。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d2a4146afc74569a4c4e8814d174ef0~tplv-k3u1fbpfcp-watermark.image?)

安装完```Docker```插件后，在活动栏上，能够看到一个集装箱的图标，点击它就能够看到```Docker```相关的信息了。

![屏幕快照 2021-11-14 16.25.48.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e83d86060d5843c0bdddcdcf56fdf6b5~tplv-k3u1fbpfcp-watermark.image?)

在这个视图中，能够看到以下信息:

![屏幕快照 2021-11-14 16.28.54.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e0047e25d12e48f88f81c332345929a7~tplv-k3u1fbpfcp-watermark.image?)

当前环境中所有的```image```，当前环境中存在的```container```，以及```Docker```的仓库列表。

右击时```image```能够看到一系列的操作，比如查看```image```信息、发布```image```、运行```image```等。当然，这些操作同样也可以在命令面板中找到。

在```containers```上右击调出的上下文菜单有三个命令: 删除```container```、重启```container```以及查看```container```运行日志。

除了提供了一个视图```Docker```插件还能够对```Dockerfile```文件进行语法高亮。而且也支持自动补全，这样就可以通过建议列表来输入```Dockerfile```中的命令了。

## 2. 构建和运行

书写了正确的```Dockerfile```后，就可以通过```Dockerfile```来构建````image````了。为了方便理解，创建一个新的文件夹，在其中创建```Dockerfile```。

```s
FROM alpine:latest

RUN apk --no-cache add \ 
    htop

CMD [ "htop" ]
```

这段```Dockerfile```的意思是，希望基于```alpine```系统安装```htop```包，最后运行```htop```命令 ，查看当前运行的各种进程。

### 1. docker build

可以打开命令面板，执行```Docker: Build image```命令。这个命令会打开集成终端，然后执行```docker build```命令。

### 2. docker run

生成了```image```之后，可以通过```image```来创建```container```了。此时，可以通过```Docker```视图的上下文菜单来生成```运行 container```，也可以从命令面板中，运行```Docker: Run```命令。然后```VS Code```就会询问使用哪个```image```。

### 3. docker run interactive

除了```Run```这个命令外，另一个非常有用的命令就是```Run Interactive```。可以创建并运行```container```，然后进入到这个```container```的```shell```环境。

在上面的例子里```container```运行的命令是```htop```，也就是实时监控系统运行的情况，执行了```Run interactive```命令，运行了```contaienr```并且进入到它的```shell```环境中后，就立刻看到了```htop```的运行界面。

对于命令面板里执行每个命令```VS Code```都会打开集成终端，然后运行相对应```Docker```脚本。可以先依赖于```VS Code```提供的命令，试着理解```VS Code```每个命令背后的脚本的含义。这就需要具备一定的```docker```命令行的知识。

### 4. 输出 log

对```Dockerfile```按如下稍作修改

```s
FROM alpine:latest

RUN apk --no-cache add \

    htop

CMD [ "pwd" ]
```

将```Dockerfile```中最后一个配置```CMD```进行了修改。这个```image```生成的```container```运行起来后会执行```pwd```命令，而非```htop```命令。

修改完```Dockerfile```之后，第一件要做的事情，就是重新构建```image```。在构建```image```时，可以覆盖之前的```image```，也可以重新起个名字来创建新的```image```。比如新的```image```取名为```vscode-docker-sample2:latest```。

有了新的```image```后，接下来就是从```vscode-docker-sample2:latest```创建一个新的```container```。

在运行```docker run```的时候，如果留意左侧视图```containers```这个列表的话，会发现```vscode-docker-sample2```的```container```出现了一下然后又消失了。这是为什么呢?来看一下集成终端，此时集成终端里运行的脚本是。

```s
docker run --rm -d vscode-docker-sample2:latest
```

这行脚本中有一个参数```–rm```，意思是如果```container```里的命令执行结束的话，就将这个```container```删除。由于这个```container```中运行的命令是```pwd```，这个命令很快就结束了，所以来不及在视图中看到并且操作它。如果不希望这个```container```被删除可以选择手动地运行如下的脚本。

```s
docker run -d vscode-docker-sample2:latest
```

这次创建的```container```运行结束后就不会被删除了。也就是此时能够在左侧```Containers```列表里看到```vscode-docker-sample2:latest (dreamy_...)```这个```container```了。它前面的图标里有一个 红色的点，这说明这个```container```已经结束工作了。

可以在这个```container```上右击调出上下文菜单，选择```Show logs```命令。接着就能够看到这个```container```中```pwd```命令执行的结果了，就是```/```。

## 3. Docker Compose

除了```Dockerfile```的支持，```Docker```插件还支持```Docker Compose```。```Docker Compose```是用于配置多个```container```并且将其同时运行。和```Dockerfile```一样也可以在```Docker compose file```里获得自动补全和错误检查。

这里用一个```Node.js```的代码示例，来展示```Docker Compose```以及接下来调试相关的内容。

首先将上文中创建的```Dockerfile```删除。然后在文件夹下创建一个```JavaScript```文件```index.js```。

```js
function foo() { 
    bar("Hello World");
}

function bar(sr) {
    console.log(sr);
}
foo();
```

接着为```JavaScript```代码准备```Docker```相关的配置。```Docker```插件已经提供了```Docker```配置的快捷生成方式。

## 4. 自动创建 Dockerfile 和 compose 配置 

调出命令面板，然后搜索执行命令```Docker: Add docker fles to workspace```。接着```VS Code```会问想要创建什么环境的```Docker image```。选择```Node.js```就可以运行上面创建的```index.js```文件了。

命令执行后，工作区内多出了三个文件，第一个文件是```Dockerfile```。

```s
FROM node:8.9-alpine

ENV NODE_ENV production

WORKDIR /usr/src/app

COPY ["package.json", "package-lock.json*", "npm-shrinkwrap.json*", "./"] 

RUN npm insall --production --silent && mv node_modules ../

COPY..

EXPOSE 3000
```

这个文件里指定了```Node.js 8.9```作为基础```image```，然后将当前工作目录下的```package.json```、```package-lock.json```等拷贝到```container```中。接着运行```npm install```命令来安装代码运行所需的```dependencies```。不过，由于只有一个简单的```JavaScript```文件并不需要```npm```，所以不 把这个文件修改的简单一些。

```s
FROM node:8.9-alpine 

ENV NODE_ENV production 

WORKDIR /usr/src/app 

COPY..

EXPOSE 3000
```

第二个文件是```docker-compose.yml```。这个文件里指定了当前只有一个```container```需要被创建并运行，而且这个```container```需要使用端口```3000```。

```s
version: '2.1'

services:
    vscode-sample:
        image: vscode-sample 
        build: . 
        environment:
            NODE_ENV: production 
        ports:
            - 3000:3000
```

第三个文件是```docker-compose.debug.yml```。用于调试时的```compose```文件，跟上面 的```docker-compose.yml```文件相比，它只多了两行代码。```9229:9229```，使用```9229```端口。```command: node --inspect index.js```在```container```运行起来后，运行```node```程序，并且调试```index.js```文件。

```s
version: '2.1'

services:
    vscode-sample:
        image: vscode-sample 
        build: . 
        environment:
            NODE_ENV: production 
        ports:
            - 3000:3000
            - 9229:9229
        ## set your sartup fle here 
        command: node --inspect index.js
```

## 5. Compose Up

有了这三个配置文件后，要想构建并且运行```container```就简单了。不需要先执行```Docker: build image``` 再运行```Docker: run```了，而是直接运行单个命令```Docker: compose up```即可。运行后，需要选择使用哪个```compose```配置文件。只要准备好```compose```配置文件，在```VS Code```中操作就非常简单了，一共只有三个命令。

```s

Docker compose up

Docker compose down

Docker compose start
```

如果想看看```VS Code```是不是真的成功运行了```container```，可以从```Docker```的视图里，找到新创建的```container```查看它的```log```。可以看到```index.js```在```container```里被成功地运行了，而且输出了```Hello World```。

## 6. 调试

```launch.json```里调试配置中，有一个属性是```request```。

这个```JSON```文件里的```confgurations```值就是当前文件夹下所有的配置了。现在只有一个调试配置，有四个属性。

第一个是```type```，代表着调试器的类型。它决定了```VS Code```会使用哪个调试插件来调试代码。

第二个是```request```，代表着该如何启动调试器。如果代码已经运行起来了，则可以将它的值设为```attach```，使用调试器来调试这个已有的代码进程。如果值是```launch```，则使用调试器直接启动代码并且调试。

当```request```的值被设置为```attach```后，可以将调试器附着到已经处于调试状态的代码进程上了，接着就能够调试代码了。而调试```Docker```中的代码，就是使用的```attach```方法。可以在```Docker container```里以命令行的方式调试代码，并且开放调试端口，接着让```VS Code```里的调试插件附着到这个端口上。这就是在```VS Code```中调试非本地环境运行的代码的理论知识了。下面一起看看怎么做。

首先，对```docker-compose.debug.yml```做一点修改，将```command```改成如下的值。

```s
command: node --inspect-brk=0.0.0.0:9229 index.js
```

这个命令是调试```index.js```文件，然后在第一行停下来，并且使用```9229```这个端口进行调试。接着，运行```Docker: compose up```将```container```运行起来。

创建```launch.json```以及借助自动补全来书写调试配置可以使用的```.vscode/launch.json```。

```json
{
    "version": "0.2.0", 
    "confgurations": [
        {
            "type": "node",
            "reques": "attach",
            "name": "Docker: Attach to Node", 
            "port": 9229,
            "address": "localhos", 
            "localRoot": "${workspaceFolder}", 
            "remoteRoot": "/usr/src/app", 
            "protocol": "inspector"
        } 
    ]
}
```

调试配置有几个属性值得注意。```request```是```attach```，也就是附着到已经运行的代码上。```port```是调试的端口。```localRoot```是本地代码的根目录。```remoteRoot```是指在远程运行的代码的根目录。例子里已经在```Dockerfile```里指明了工作目录是```/usr/src/app```。有了这个调试配置后，```F5```就能够调试```Docker container```中的代码了，并且停到了第一行代码上。

至此就成功地将一段```JavaScript```代码运行在```Docker```中，并且从```VS Code```里调试起来了。
