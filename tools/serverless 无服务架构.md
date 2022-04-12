## 1.概述

最近几年的技术圈，对于```Serverless```技术的讨论异常火热，在业内也有了很多成熟的案例，国外发展较早，比较有代表性的就是亚马逊和谷歌，在国内腾讯和阿里两位巨头，都将```Serverless```定义为集团战略型项目，不遗余力的推广和研发自己的````Serverless````技术。

```Serverless```是一种后端架构技术，更准确的说，它应该是一种后端架构的概念或者思维，```Serverless```本身和前端半毛钱关系都没有，但是它却是前端程序员最希望落地应用的技术，众多前端开发者望穿秋水一般的期待它的成熟落地，而很多后端程序员，漠不关心甚至排斥。为什么会出现这种一方冷淡一方火热的奇怪现象？要搞清楚这一点，需要从后端架构的演进历史说起。

每一个```B/S```架构的互联网应用，都是由最基础的客户端和服务端构成的，客户端要呈现内容就需要服务端提供服务，最初搭建一个服务器是非常繁琐的，需要购买一台电脑主机，然后找一个机房对机器进行托管，还需要将外观拍照，将各项硬件参数提交到备案中心进行备案，链接电源和网线，安装好操作系统，搭建好代码的运行环境，部署程序代码，将申请的域名和静态IP做好解析，就算是上线部署完毕了。

这样的服务器架构是单机版的单体架构，数据库、应用代码、```HTTP```服务器等服务全都在一台自己管理的服务器上运行，因为要接触物理机，所以把它成为物理机时代。

在物理机时代不仅仅是上线部署非常繁琐的问题，在整个应用的运行中，还有各种各样的问题出现，比如磁道磨损的硬盘，机房的意外停电，老鼠咬断的网线。

随着技术的不断发展终于摆脱了物理机时代，跨入虚拟机时代。其中一个重要节点之一就是```2001```年```VMWare```带来的针对```x86```服务器的虚拟化产品，通过虚拟化技术，可以把一台物理机分割成多台虚拟机提供给用户使用，充分利用硬件资源，对于硬件设备的管理，统一由云厂商负责，对于开发者来说，就不用再买硬件了，直接在云平台买虚拟机，比如```AWS```的```EC2```、阿里云```ECS```、腾讯云```CVM```。

云服务器也真正的进入了大众的视野，开发者再也不用担心断电断网和硬件故障了，不过业务量的不断增长，用户越来越多，数据库每天都有几千万条数据写入，数据库性能很快就会达到瓶颈。除此之外每天也有上百万图片存到磁盘，磁盘也快要耗尽了。为了降低服务器负载，把数据库迁移到了云厂商提供的云数据库上，把图片存储迁移到对象存储。

云数据库有专门的服务器，并且还提供了备份容灾，比自己在服务器上安装数据库更稳性能更强。云对象存储能无限扩容，不用担心磁盘不够了。这样一来，服务器就只负责处理用户的请求，把计算和存储分离开来，既降低了系统负载，也提升了数据安全性。并且单机应用升级为了集群应用，通过负载均衡，会把用户流量均匀分配到每台服务器上。

不过在服务器扩容的过程中，还是会遇到一些麻烦。比如购买服务器后，都需要在上面初始化软件环境和配置，还需要保证所有服务器运行环境一致，这是个非常复杂还容易出错的工作。这对运维工程师也是一个非常大的挑战。

总的来说，虚拟机可以不用关心底层硬件，但是依然要为服务器集群的管理工作付出高昂的成本，如果有一项技术，能够在每次的服务器扩容时，服务器的运行环境保持一致，那就太好了，于是，容器技术应运而生。

```2013```年```Docker```的发布，代表着容器技术替代了虚拟化技术，云计算进入容器时代。容器技术就是在虚拟化技术的基础上，把代码和运行环境打包在一起，这样，不论服务器的配置怎样，代码和运行环境均能保持一致性。有了容器技术，在服务器上部署的就不再是应用了，而是容器。当容器多了的时候，如何管理就成了一个问题，于是出现了容器编排技术，比如```2014```年```Google```开源的```Kubernetes```，就是俗称的```K8S```。

在目前的后端架构中，容器技术依然是主流的服务器架构技术，但是随着互联网应用的普及，需要面对各种各样的应用场景，比如当大型购物节来临，在线商城会面临巨大的流量洪峰，而在平时流量显然要小很多，为了迎接购物节的流量洪峰，需要大量扩充服务器，服务器的扩充和大量容器的编排工作，也是一个不小的难题，如果对流量的压力预估不到位，还会有服务器宕机的风险，而在平时，如此多的服务器运行显然是在浪费资源增加成本。

有没有一种技术只关心业务代码的功能实现，脱离服务器的管理不再为运行环境劳心伤神，不再为服务器的扩缩容半夜惊醒，当流量洪峰来临时，自动调配更多的服务器资源支撑，当流量低谷时，自动释放服务器资源节约成本。这样美好的时代，正在想你走来，它叫```Serverless```。

纵观后端架构的发展史，其实就是```Serverless```的兴起史，每一个时代，都是对前一个时代基础架构的抽象，从物理机时代跨越到虚拟机时代，不在关心硬件设备的管理，将一台真实的计算机抽象为相互隔离的多个虚拟机，从虚拟机时代到容器时代，摆脱了运行环境和集群管理的繁杂工作，将虚拟机的运行环境抽象为容器。```Serverless```的到来，不在关心运行环境和服务器资源的调配。

## 2. 基本概念

```Serverless```中有很多全新概念的引入，相比于具体的应用，```Serverless```的相关理念更值得学习。技术圈对```Serverless```的定义也在不断的调整和变化，所以导致有不少刚接触```Serverless```的同学会认为```FaaS```就是```Serverless```，也有同学认为```PaaS```也是```Serverless```，还有同学说使用```Serverless```就没有服务器了。

总的来说很多同学对```Serverless```到底是什么并没有一个很清晰的认知，概念还比较模糊。广义上来说，```Serverless```是一种后端架构理念，或者说是一种思想、概念，直接翻译过来叫做“无服务”，但是不要被字面意思误导，这并不代表着应用运行不需要服务器，在```Serverless```时代之前，可以将传统架构统称为```Serverful```时代意思就是关于服务器的一切，都需要人工干预，而```Serverless```更准确的说应该是开发者不用关心服务器的意思，是将服务器相关的工作交给云平台来做，对于开发者来说，与服务器运维有关的所有工作都不再关心，```Server```(服务器)是不可能真正消失的。

>```2019```年```2```月```UC```伯克利大学发表了一篇标题```《CloudProgrammingSimplified:ABerkeleyViewon```Serverless```Computing》```的论文，论文中有这样一段对```Serverless```的描述:

>> 在云的上下文中，```Serverful```的计算就像使用低级的汇编语言编程，而```Serverless```的计算就像使用```Python```这样的高级语言进行编程。例如```c=a+b```这样简单的表达式，如果用汇编描述，就必须先选择几个寄存器，把值加载到寄存器，进行数学计算，再存储结果。

>> 这就好比今天在云环境下```Serverful```的计算，首先需要分配或找到可用的资源，然后加载代码和数据，再执行计算，将计算的结果存储起来，最后还需要管理资源的释放。

>>```Serverful```是今天主流的使用云的方式，但不应该是未来使用云的方式，```Serverless```所希望的，是开发者用代码去支撑业务逻辑，而对于资源的管理交给工具和云。

在```Serverful```的架构下需要关心的问题非常多，比如根据业务流量大小等指标，响应式地调整服务规模，实现自动弹性伸缩。再比如异地容灾、负载均衡、日志监控、文件存储等等，解决这些复杂的问题需要投入大量的人力、物力，而在```Serverless```架构下，开发者只专注于开发业务逻辑，所有的这些与业务无关的基础设施，全部交给云平台负责，由云平台统一调度、运维。

在这样的理念指导下各家云平台厂商都有不同实现方案，每家云平台提供的```Serverless```服务都或多或少的存在差异，但是按照```CNCF```(云原生计算基金会)对```Serverless```计算的定义，```Serverless```架构应该是采用```FaaS```(函数即服务)和```BaaS```(后端即服务)服务来解决问题的一种设计。

这样的定义从应用落地的角度来说，更加的具体可行，对```Serverless```的理解更加的清晰明了。因此，从应用落地的角度，狭义的```Serverless```就是```FaaS```+```BaaS```的组合。

## 3. FaaS + BaaS

前面说，```Serverless```把后端架构的工作全部包揽下来，硬件的维护，集群的管理，运行环境的搭建，全部由云平台完成，除此之外，像缓存、数据库、文件存储、消息中间件等，也全部由云平台做好，封装起来，以接口的形式提供服务，这就是```BaaS```，所谓后端即服务，对于开发者```BaaS```就是一个黑盒，不用知道怎么做，更不需要关心如何做，需要什么过来拿就行了。

但是，需要向数据库存一条数据，用户上传的照片需要裁剪以后存到文件存储中，这是需要编写业务逻辑代码完成的功能，假设已经把这些逻辑代码写好，用的是```Node.js```，前面说所有的服务器及运行环境都放在了```BaaS```这个黑盒子中，怎么让这些代码运行呢？换句话说就是现在写的逻辑代码是需要```Node.js```这个运行环境的。只需要将写好的代码交个```Serverless```就行了。

```Serverless```中有专门运行逻辑代码的地方，这个地方就是```FaaS```，```FaaS```是以函数的方式运行代码的，本质上```FaaS```就是一个函数运行平台，大多数的```Serverless```云平台提供的```FaaS```，都支持```Node.js```、```Python```、```Java```、```PHP```等编程语言，可以选择喜欢的编程语言编写函数并运行。对于开发者老说，使用```FaaS```几乎就是使用```Serverless```的一切了，在```FaaS```中，能够体会到```Serverless```全部的特性。

首先```FaaS```函数运行时，开发者对底层的服务器是无感知的，```FaaS```产品会负责服务器资源的调度和运维，这些就是前面说的```BaaS```，这也是```Serverless```最大的特点，无运维。

其次```FaaS```中的函数也不是持续运行的，而是通过一定的条件进行触发比如```HTTP```事件、消息事件、定时器事件等，产生事件的源头叫触发器，```FaaS```平台会集成这些触发器，直接用就行，这是```FaaS```的第二个特点，事件驱动。

再者就是```Serverless```的付费方式了，与其他云产品不同的是```Serverless```的付费方式是按量付费，是按照```FaaS```函数执行次数和执行时消耗的```CPU```、内存等资源进行计费的，用多少付多少，不用不付费，同时，```FaaS```会根据并发量自动生成多个函数实例，```BaaS```会根据函数运行所需要的资源量自动调配服务器资源，理论上的资源调用量没有上限，这也就实现了不同访问量的弹性伸缩了，而且是实时的弹性伸缩。

基于```FaaS```和```BaaS```的架构是一种计算和存储分离的架构。计算由```FaaS```负责，存储由```BaaS```负责，计算和存储也被分开部署和收费。这使应用的存储不再是应用本身的一部分，而是演变成了独立的云服务，降低了数据丢失的风险。而应用本身也变成了无状态的应用，更容易进行调度和扩缩容。

基于```FaaS```和```BaaS```应用就实现了自动弹性伸缩、按量付费、不用关心服务器，这正是```Serverless```架构的必要因素。所以说狭义的```Serverless```是```FaaS```和```BaaS```的组合。

## 4. 优缺点

```Serverless```可以不用运维、实现自动的弹性伸缩、按量付费节省成本、更高的安全性、易于迭代和部署。```Serverless```的缺点也十分明显。

1. 依赖第三方服务

```Serverless```的能力是云厂商打包提供的，所以```Serverless```产品一定是和云厂商绑定的，又因为```Serverless```理念和具体实现之间并没有统一的标准，比如```A```厂商认为```Serverless```的数据库必须使用标准```SQL```规范，而```B```厂商则认为数库可以使用```SQL```规范也可以使用```JSON```文件的存储方案，这就出现了不同的云厂商实现了不同的```FaaS```接口，同一套代码，是无法在不同的```Serverless```产品上运行的，要想从一个云平台迁移到另一个云平台，成本非常高。

2. 开发调试困难

```Serverless```应用依赖的云服务，难以在本地环境搭建，要想在本地开发调试非常复杂。同时，```Serverless```架构正处于飞速发展的阶段，其开发、调试、部署工具链并不完善。

3. 底层硬件的多样性

目前```Serverless```的技术实现是```FaaS```和```BaaS```。应用代码在```FaaS```上运行，但```BaaS```是个黑盒，其底层的硬件资源是不确定的，某些场景下代码必须运行在某种类型的```CPU```或```GPU```上，目前云厂商并没有提供针对底层硬件的可选项。

## 5. 基本应用

```Serverless```是与云厂商绑定的，所以需要选择一家云平台服务商来进行实战演练，相比于其他厂商个人推荐[腾讯云](https://docs.cloudbase.net/)，并不是其他厂商不好，只是针对与初学者，腾讯云提供了非常友好的手册及教程，如果使用过小程序的云开发对于腾讯云，会感到非常熟悉。

首先需要前往腾讯云官网，注册腾讯云账号，然后登录账号。如有账号，可以直接登录。注意，一定要先完成实名认证。

接着在产品中选择申请开通云开发[CloudBase](https://cloud.tencent.com/product/tcb)，名称自己选择，进入控制台。等待创建环境之后可以看到环境的基本数据，以及服务，云函数，云存储，静态托管，监控警告等。

```Serverless```最核心重要的就是```Faas```和```Baas```，```Baas```是云平台提供的一系列服务，```Faas```是运行代码的环境也就是云函数。可以创建一个云函数并且写一段代码试一试，看看是否可以运行。

在云函数中点击新建云函数，函数名称可以填写```hello```，运行环境可以选择```Node```，由于每一个环境语言都是需要适配```Serverless```的，所以可能导致没办法使用最新的语言环境，这也是```Serverless```的一个缺点。函数内存是最大可以调用的内存，选择的内存越大可能造成费用越高，因为是按量付费的。函数配置中会默认带入一段代码，可以先不用理会直接确定保存。

在函数列表中点击函数代码就可以修改代码了。在这个页面可以看到一个```index.js```文件，这其实就是函数的入口文件。其中的```main```函数是入口函数，函数接受两个参数```event```对象和```context```对象。

```event```对象指的是触发云函数的事件。例如小程序端调用时，```event```是小程序端调用云函数时传入的参数，
在使用```HTTP```请求的形式调用时，```event```是集成请求体。
```context```对象包含了此调用的调用信息和函数的运行状态，可以使用```context```了解服务运行的情
况。

想要返回数据时可以通过```return```进行返回。

修改代码后点击底部的保存按钮。保存函数代码后，如何执行和访问呢？前面说过，```FaaS```产品的一大特性是事件驱动，想要让函数代码执行，需要创建一个事件的触发器，比较熟悉的就是```HTTP```触发器了，在腾讯云它叫```HTTP```访问服务（环境```->```访问服务）。

点击新建，域名可以选中默认的，触发路径就是```router```，关联的资源选择云函数，再选中刚刚的函数。鉴权开关可以关闭。刷新一下触发器就出现了，会等待```3-5```分钟等待创建完毕。等待触发器创建成功后，可以使用默认域名，访问应用函数。

## 6. 本地开发

云开发的文档中心里有个工具插件，```CloudBaseCLI```是云开发(```TencentCloudBase，TCB```)开源的命令行界面交互工具，用于帮助用户快速、方便的部署项目，管理云开发资源。```CloudBaseCLI```是基于```Node.js```开发的工具，因此，需要保证本地的```Node.js```版本在```8.6```以上。

```s
npm i -g @cloudbase/cli
# yarn global add @cloudbase/cli
# 查看版本号
tcb -v
# 查看相关帮助
tcb -h
```

使用之前需要先进行授权登录，命令行执行```tcb login```，会自动弹出浏览器的授权界面，确认授权后，命令行的控制台会打印登录成功的相关信息。

```s
tcb login
```

授权成功后，可以看到提供了很多的命令。这时就可以在本地创建云函数了，在命令行执行```tcb new```命令行的选项中，有一项需要注意，本地创建是只能创建函数而不是应用，也就是说，最好是先在控制台创建好应用后，再创建本地函数。

准备好之后，就可以看到可选的应用的了，选择函数所属的应用后，在选择对应的函数模板，这里选择与之前一样的```Node```云函数示例，输入项目名称后，云函数会下载相关代码到本地，注意，此时在控制台中，是看不到这个函数的，也就是说，这个函数并没有上线运行，其实命令行中也给了提示:```执行命令tcb一键部署```，想要上线运行，还需要进到项目目录中，执行```tcb```命令，当前运行命令的目录在哪里，生成的模板也就在哪里。项目名称也就是目录的名称。

展开目录可以看到```functions```的目录以及一些配置文件。```functions```中存放云函数的目录以及代码。里面的```node-app```就是云函数名字，```index.js```就是云函数的代码。

修改云函数代码之后想要部署，可以切换目录到项目目录，使用```tcb```命令就会部署到云函数。函数的名字是目录的名字，可以通过```cloudbaserc.json```中的```functions```的```name```进行修改。

```json
functions: [
    {
        "name": "node-app",
        "timeout": 5,
        "envVariables": {},
        "runtime": "Nodejs10.15",
        "handler": "index.main"
    }
]
```

函数的运行是需要触发器的，所以，函数部署成功后，执行```tcb service create```命令，创建```http```触发器，创建成功后，会返回访问地址，但是，与在控制台创建触发器一样，需要等待几分钟的时间后，才能看到访问结果。

修改函数返回值的内容，重新刷新后并没有显示，这需要更新函数代码后才能生效，可以使用```tcb fn code update xxx函数名```，命令只会更新函数的代码以及执行入口，不会修改函数的其他配置。

## 7. 测试工具

虽然代码放在了本地编写，但是本地是没有触发器的，当然本地有```Node```环境，但是与云环境还是有很大差别的，不能每次写完代码，都需要上传在测试，这中体验根本就没法用，为了解决这个问题，腾讯开发了开源的[scf-cli](https://github.com/TencentCloud/scf-node-debug)工具，帮助在本地快速调试，基本原理就是在本地开启一个服务器模拟云环境，可以在本地进行调试。

```s
npm install scf-cli -g
```

安装成功后，在项目目录下执行```scf init```或者```scf i```开启本地测试环境，命令提示符会让输入入口文件的路径，这里是路径地址不是文件地址，路径填写相对路径。还需要入口函数名也就是```main```方法名，设置好超时时间3s后，选择测试方式。

如果选择的是```http```，那么默认的会开启```3000```端口的服务器。此时，就可以愉快的使用```Serverless```云原开发了。

## 8. Express 与云函数

经过前面一些列的配置，终于可以在本地开发调试了，但是使用纯原生的```Node.js```开发，效率是非常低的，如果能在云函数中引入一款熟悉的后端开发框架，开发才算坐上了高铁。这里选择```Express```，先在本地安装好```Express```，按照以往的开发经验，在本地开启```HTTP```服务。

在```node-app```目录下创建```app.js```文件，这个文件主要接收请求和做路由分发。

```js
const express = require("express");

const app = express();

app.use('/users',(req,res)=>{
  res.send('yindong')
})

module.exports = app;
```

再创建一个启动文件```node-app/www.js```。

```js
const app = require('./app.js');

app.listen(3000, () => {
    console.log('http://127.0.0.1:3000');
})
```

而在云函数中，代码是在入口函数运行的，而且云环境中有```HTTP```的触发器，是不需要创建服务器的。上面的代码```Express```是自己创建了一个启动文件的，这就造成了矛盾，就是本地使用```Express```就是用户先来请求```Express```创建好的服务，再去调用路由规则处理请求作出响应。

但是云服务是用户请求云环境的```HTTP```触发器，由触发器调用云函数，云函数里面才是接收到的请求以及作出响应的代码，也就是需要将```HTTP```触发器和函数做一个入口，本地的触发入口在云函数中是不需要的。

有一款开源工具，专门用作对框架代码进行包装，[serverless-http](https://github.com/dougmoscrop/serverless-http)，```https://www.npmjs.com/package/serverless-http```是专门用于在```Serverless```环境下，包装接口的模块，不需要服务器，也不需要监听端口。

```s
npm install serverless-http
```

在入口文件中```node-app/index.js```中。

```js
const serverless = require('serverless-http');

let app = require('./app.js');

const handler = serverless(app);

module.exports.main = async (event, context) => {
  const result = await handler(event, context)
  return result;
};
```

写好之后将代码部署到云平台，然后进行请求测试，在本地开发测试，开启本地服务器就可以了。

注意执行```tcb```命令时需要在项目的目录中运行。安装依赖的时候需要在```node-app```目录中执行。

## 9. API 接口案例

这里我实现一个```TodoList```案例的后端```API```接口，这个案例具备最基础的增删改查等基础功能。回到```app.js```中将基础代码进行补齐。

```js
const express = require("express");

const app = express();

app.use(express.json());

app.use(express.urlencoded({ extended: false }));

const indexRouter = require("./routers/index");

const todoRouter = require("./routers/todo");

app.use("/", indexRouter);

app.use("/todo", todoRouter);

module.exports = app;
```

新建两个文件```node-app/routers/index.js```和```node-app/routers/todo.js```。

```routers\index.js```。

```js
const express = require('express');

const router = express.Router();

router.get('/', (req, res)=>{
    res.send('index router');
})

module.exports = router;
```

```node-app/routers/todo.js```。

```js
const express = require('express');

const router = express.Router();

router.get('/',(req, res)=>{
    res.send('todo router');
})

module.exports = router;
```

代码实现之后，在本地请求```/```根路径和```todo```路径，测试完成后，部署云函数，然后再进行对应的测试。基础的业务路由配置好之后，回到业务代码的编写中，在```TodoList```案例中先来实现增删改查的相关操作。

```node-app/routers/todo.js```。

```js
const express = require('express')
const router = express.Router();
// 获取任务 
router.get('/',(req,res)=>{
  res.send('get todo router')
})
// 添加任务 
router.post('/',(req,res)=>{
  res.send('add  todo router')
})
// 修改任务 
router.put('/',(req,res)=>{
  res.send('change todo router')
})
// 删除任务 
router.delete('/',(req,res)=>{
  res.send('del todo router')
})
module.exports = router;
```

## 10. 云数据连接

在使用云数据库之前，需要先理清楚它的一些基本概念，腾讯云提供的云数据库是一种文档型数据库，提供基础读写、聚合搜索、数据库事务、实时推送等功能。

数据库中有数据库实例、集合、记录这三个基本概念，每个云开发环境下有且只有一个数据库实例，数据库实例中，可以创建多个集合，可以将集合理解为一个文本文件，每个文件中可以存放多个类似```JSON```格式的对象，被称为记录。[官方手册](https://docs.cloudbase.net/database/introduce.html)。

云数据库可以在客户端调用也可以在服务端调用，云函数就是服务端调用的方式。服务端调用时，需要在 SDK 初始化参数中，填入应用的```envid```，同时需要填入腾讯云密钥(```SecretID```和```SecretKey```)，手册上并没有说，但是一定注意，除了腾讯云密钥还需要```env```，也就是云环境```ID```。

首先需要在本地安装```sdk```。

```s
npm install cloudbase/node-sdk
```

```secretId```和```secretKey```需要在账号的访问管理里面进行创建获取。```env```在环境总览里面获取。

```js
const nodeSdk = require("@cloudbase/node-sdk");
const cloudDb = nodeSdk.init({
  env:'yindong01-3gcch2hafXXXXXXXXXX',
  secretId:'AxxxXXXXXXXKpWw6zbXXXXXXXXXXX',
  secretKey:'kFXdOOXXxxxp22AwiXXXXXXXXXXX'
})
```

在网页控制台中找到数据库选项，每个云环境有且只有一个数据库实例，可以点击创建集合进行创建名字叫做```todo```。每个集合下面有多个记录，记录就是数据。记录是以文档形式进行添加的，```id```可以不用管。添加类型就可以。类型中```_id```是默认```id```可以用可以不用。

配置好基本信息之后，就可以连接数据库了，执行对应操作了，但是，数据库的操作以集合为单位的，所以，在操作之前需要先创建集合，用```cloudDb.collection```获取集合引用后，再执行对用操作就可以了。

### 1. 获取数据

```js
// 获取数据库引用
const app = cloudDb.database();

// 链接数据库
const db = app.collection('todo');

// 获取todo里面的全部数据
router.get('/',async (req,res)=>{

  res.send(await db.get());

})
```

### 2. 添加数据

注意云函数和实际时间有```8```小时时间差。

```js
router.post('/', async (req, res) => {
  
  const todo = {
    title: req.body.title,
    createTime: Date.now(),
    done: false,
  }

  const backdata = await db.add(todo);

  res.send(backdata);
})
```

### 3. 修改

```js
router.put('/', async (req, res) => {

  if (req.query.id == undefined) {
    res.send('缺少id');
  }

  if (req.body.done == undefined) {
    return;
  }

  let done = false;

  if (req.body.done == 1) {
    done = true;
  }

  const todo = {
    title: req.body.title,
    done: done
  }

  // doc条件限制
  const backdata = await db.doc(req.query.id).update(todo);

  res.send(backdata);
})
```

### 4. 删除

```js
router.delete('/', async (req, res) => {

  if (req.query.id === undefined) {
    res.send('缺少id');
  }

  const backdata = await db.doc(req.query.id).remove();

  res.send(backdata)
})
```

## 11. 客户端接口调用

这里选择使用普通的```Vue```框架作为客户端，按照传统的方式创建，安装好```Element-ui```及```Axios```请求库，就可以直接向云函数发送请求获取数据了。这里简单的写了一个请求的示例，发送请求后，渲染到页面中。

```vue
<template>
  <div>
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>Todo List</span>
      </div>
      <div v-for="(todo, key) in cloudData" :key="key" class="text item">
        <el-checkbox >  {{ todo.title }} </el-checkbox>
         <el-divider></el-divider>
      </div>
    </el-card>
  </div>
</template>

<script>
  import Axios from "axios";
  export default {
    data() {
      return {
        cloudData: [],
      };
    },
    methods: {
      async getData() {
        const backdata = await Axios({
          url: "https://yindong02-8gsxxxxxx5-111119081.ap-shanghai.app.tcloudbase.com/express-todo/todo",
        });
        console.log(backdata);
        this.cloudData = backdata.data.data;
      },
    },
    mounted(){
      this.getData()
    }
  };
</script>
```

这里可能会出现跨域的问题，可以在服务器里面添加跨域允许的响应头或者直接使用完整路径的```url```进行访问。正常情况下是不需要管云函数的跨域问题的，他默认是允许跨域访问的。不过这样的开发方式，非常不```Serverless```。

前面在应用中创建了一个云函数，并将云函数与```Express```进行整合，配合云数据库写好了增删改查的接口，但是这样的开发方式并不是```Serverless```的最佳实践方法，在代码中是将整个后端应用的全部业务能力写进了一个函数中，这样做的好处是方便管理，毕竟在一个应用下只有一个云函数。

但是单个云函数的并发是有限的，并行的函数实例个数由云厂商决定，超过限制后事件队列就需要等待其他函数实例执行完毕后，再生成新的函数实例。可能就有人会问，```Serverless```是弹性伸缩的，不是说会根据业务处理的需求自动调配资源嘛？为什么还会有函数的并发限制？要搞清楚这一点，需要了解```FaaS```的运行机制。

## 12. FaaS 运行机制

在```FaaS```平台中，函数默认是不运行的，也不会分配任何资源。甚至```FaaS```中都不会保存函数代码。只有当```FaaS```接收到触发器的事后，才会启动并运行函数。前面就是使用```HTTP```的触发器来执行函数代码的，整个函数的运行过程实际上可以分为四个阶段。

### 1.代码下载

```FaaS```平台本身不会存储代码，而是将代码放在对象存储中，需要执行函数的时候，再从对象存储中将函数代码下载下来并解压，因此```FaaS```平台一般都会对代码包的大小进行限制，通常代码包不能超过```50MB```。

### 2. 启动容器

代码下载完成后，```FaaS```会根据函数的配置启动对应容器，```FaaS```使用容器进行资源隔离。

### 3. 初始化运行环境

分析代码依赖、执行用户初始化逻辑、初始化入口函数之外的代码等。

### 4. 运行代码

当函数第一次执行时，会经过完整的四个步骤，前三个过程统称为```冷启动```，最后一步称为```热启动```。整个冷启动流程耗时可能达到百毫秒级别。函数运行完毕后，运行环境会保留一段时间，这个时间在几分钟到几十分钟不等，这和具体云厂商有关。如果这段时间内函数需要再次执行，则```FaaS```平台就会使用上一次的运行环境，这就是```执行上下文重用```，也叫做```实例复用```，函数的这个启动过程也叫```热启动```。```热启动```的耗时就完全是启动函数的耗时了。当一段时间内没有请求时，函数运行环境就会被释放，直到下一次事件到来，再重新从冷启动开始初始化。

考虑下面这个云函数:

```js
let i = 0;
exports.main = async (event = {}) => {
  i++;
  console.log(i);
  return i;
};
```

在第一次调用该云函数的时候函数返回值为```1```，这是符合预期的。但如果连续调用这个云函数，其返回值有可能是从```2```递增，也有可能变成```1```，这便是实例复用的结果。

当热启动时，执行函数的```Node.js```进程被复用，进程的上下文也得到了保留，所以变量自增。当冷启动时，```Node.js```进程是全新的，代码会从头完整的执行一遍，此时返回```1```。

函数执行完毕后销毁运行环境，虽然对首次函数执行的性能有损耗，但极大提高了资源利用效率，只有需要执行代码的时候才初始化环境、消耗硬件资源。并且如果你的应用请求量比较大，则大部分时候函数的执行可能都是热启动。

从函数运行的生命周期中可以发现，如果函数每分钟都执行，则函数几乎都是热启动的，也就是会重复使用之前的执行上下文。执行上下文就包括函数的容器环境、入口函数之外的代码。

云平台会根据当前的负载情况，自动控制云函数实例的数量，并且均衡地分发请求。连续的多次请求有可能由同一实例处理，也可能不是。这就是在上面的代码中看到的，```i```的值非常放肆，根本就找不到规律。所以在编写云函数时应注意保证云函数是无状态的、幂等的，即当次云函数的执行不依赖上一次云函数执行过程中在运行环境中残留的信息。

回到```Todo```案例中，因为将全部的业务逻辑放到了一个云函数中，因此处理的并发量会受到极大的限制，当并发量达到一定的程度时，无法创建更多的函数实例，也就无法分配更多的服务器资源。更好的方式是对业务逻辑进行拆分，一个云函数就对应一个独立的业务逻辑处理，这在小程序的云开发中就有体现，默认给小程序云开发模板中，就是一个小程序应用对应多个云函数的处理方式。

## 13. 云函数高阶应用

腾讯官方提供的```cloudbase-framework```工具则给提供了一种方式，前面使用的```CloudBase CLI```命令行工具，就是使用```cloudbase-framework```的对外接口工具，也就是说使用的命令行，实际就是调用了```cloudbase-framework```提供的功能，前面已经使用过一些了，比如```tcb new```创建应用、```tcb```应用部署、```tcb service create```创建```http```触发器 增量更新代码。除了这些部署代码相关的命令，```framework```还给我提供了一站式管理云平台资源的能力。

接下来按照```Serverless```的开发模式，对```Todo```案例进行重构，在腾讯云开发```CloudBase```下，已经给创建好了各种各样的开发的模板，使用```tcb new```命令就可以看到，在选择应用模板时， 选择```Vue```应用就可以创建一个```Vue```云开发的项目。

项目创建后之后，能看都项目路径下有```cloudfunctions```目录，这就是存放云函数的地方，一个函数就是一个文件夹，那么怎么管理这些函数呢？在项目的根路径下，有一个```cloudbaserc.json```的文件，它就是整个应用的```framework```的配置文件，可以通过这个配文件来管理项目应用。所以在开 始之前，要先来认识一下这个配置文件中各个配置项的含义。

```json
{
  "version": "2.0", // framework 版本，开启配置文件支持动态变量的特性
  "envId": "{{env.ENV_ID}}", // 应用ID
  "$schema": "https://framework-1xxxxxxxxxx.xxxxapp.com/schema/latest.json", // 配置模板的描述信息
  "framework": {
    "name": "techo-show",
    "plugins": {}
  },
  "region": "ap-shanghai", // 应用所在地区 
}
```

```version```字段```CLI 0.9.1+```版本引入了```2.0```新版本配置文件，支持了动态变量的特性。在```cloudbaserc.json```中声明```"version": "2.0"```即可启用新的特性，新版配置文件只支持```JSON```格式。动态变量特性允许在```cloudbaserc.json```配置文件中使用动态变量，从环境变量或其他数据源获取动态的数据。使用```\{\{\}\}```包围的值定义为动态变量，可以引用数据源中的值。

```envId```字段是应用```ID```。

```$schema```是配置模板的描述信息。

```region```应用所在地区。

```framework```是配置文件的主要配置项
```framework```字段```name```属性是应用名字。

```framework.plugins```这是管理应用的重点，```Framework```是支持插件机制的，提供了多种应用框架和云资源的插件。应用依赖哪些插件，都在```plugins```参属下配置，```framework```会根据```plugins```的配置来管理应用，处理应用中的构建、部署、开发、调试等流程，一个应用可以使用多个插件，使用不同的自定义属性名字进行管理。官方提供的插件有很多，具体可以查看```https://docs.cloudbase.net/framework/plugins/```。

### 1. 云函数插件

首先对之前写好的云函数进行插件方式的修改:


```json
"server": { // 自定义插件名字
  "use": "@cloudbase/framework-plugin-function", // 引入使用的插件 
  "inputs": { //对于当前插件的配置信息
    "functionRoot": "./functions", //函数代码的本地路径 "functions": [ //云函数的配置信息
    "functions": [ //云函数的配置信息
      {
        "name": "express-todo", // 云函数的名字 "timeout": 5, // 运行超时时间
        "envVariables": {}, // 环境变量的键值对对象 "runtime": "Nodejs10.15", // 运行时环境(编程语言) "memorySize": 128, // 运行最大分配内存 M 单位 "handler": "index.main" // 函数入口
      }
    ]
  }
}
```
配置完成后，修改代码，然后进行部署测试。

### 1. 静态网站插件

云函数配置好之后，回到我们的客户端代码中，正常的开发部署流程是。本地开发，测试，线上测试，打包发布。在云开发中，有一个静态站点托管的服务，我们可以借助 静态网站插件 ，一键完成打包上线部署的全部工作，不用再手动完成了。

```json
"client": {
  "use": "@cloudbase/framework-plugin-website", // 引入使用的插件
  "inputs": {
    "buildCommand": "npm run build", // 本地执行的打包命令 
    "outputPath": "dist", // 静态文件地址
    "cloudPath": "/", // 静态网站托管的代码路径 
    "envVariables": {
        "VUE_APP_ENV_ID": "{{env.ENV_ID}}"
    }
  }
}
```

## 14. 云函数 Todo 重构 
 
搞清楚各个配置的含义之后，就可以按照之前的思路，实现```Todo```应用了，先配置一个添加任务的函数。首先需要手动的修改配置文件，在```sever```的```functions```中添加```addtodo```。

```json
"server": { // 自定义插件名字
  "use": "@cloudbase/framework-plugin-function", // 引入使用的插件
  "inputs": { //对于当前插件的配置信息
    "functionRootPath": "cloudfunctions",
    "functions": [
      {
        "name": "addtodo",
        "timeout": 3,
        "envVariables": {},
        "runtime": "Nodejs10.15",
        "memory": 128,
        "aclRule": {
          "invoke": true
        }
      }
    ]
  }
}
```

注意，配置文件不能帮我们在本地创建文件及文件夹，不具备小程序的能力，所以，写好配置文件，需要自己创建对应的代码目录及文件。

```/cloudfunctions/addtodo/index.js```。

```js
const cloud = require("./cloudDb");

async function addTodo (event) {
  var req = event;
  if(req.title == undefined){
    return; 
  }
  var todo = {
    title:req.title,
    createTime:Date.now(),
    done:false
  }
  var backdata = await cloud.collection('todo').add(todo);

  return backdata;
}
exports.main = async (event, context) => {
  return addTodo(event)
};
```

```/cloudfunctions/addtodo/cloud.js```是数据库连接文件。

```js
const nodeSdk = require("@cloudbase/node-sdk");

const cloudDb = nodeSdk.init({
  env:'yindong01-3gcch2hafXXXXXXXXXX',
  secretId:'AxxxXXXXXXXKpWw6zbXXXXXXXXXXX',
  secretKey:'kFXdOOXXxxxp22AwiXXXXXXXXXXX'
});

const app = cloudDb.database();

module.exports = app;
```

代码写好之后可以使用的```SCF```工具，然后再使用```Postman```发个请求。测试完成后，可以使用```tcb```命令进行全量部署，注意，全量部署时，```vue```也会跟随打包并部署到静态站点中，如果只想部署单个云函数，可以使用命令```tcb fn deploy addtodo```对```addtodo```这个函数单独部署。部署完成后可以登录云控制台查看，也可以在本地使用```tcb fn list```查看已部署的函数列表。

```gettodo```获取数据的方法实现类似，复制一个```function```，放在```functions```数组中。同时创建对应的目录和文件。

```json
"server": { // 自定义插件名字
  "use": "@cloudbase/framework-plugin-function", // 引入使用的插件
  "inputs": { //对于当前插件的配置信息
    "functionRootPath": "cloudfunctions",
    "functions": [
      {
        "name": "gettodo",
        "timeout": 3,
        "envVariables": {},
        "runtime": "Nodejs10.15",
        "memory": 128,
        "aclRule": {
          "invoke": true
        }
      },
      {
        "name": "addtodo",
        "timeout": 3,
        "envVariables": {},
        "runtime": "Nodejs10.15",
        "memory": 128,
        "aclRule": {
          "invoke": true
        }
      }
    ]
  }
}
```
  
```/cloudfunctions/gettodo/index.js```。

```js
const app = require("./cloudDb");

async function showTodo(event) {
  const backdata = await cloud.collection('todo').get();
  return backdata;
}

exports.main = async (event, context) => {
  return showTodo();
}
```

## 15. Vue 客户端调用

云函数可以在客户端直接调用，所以就不需要再去创建触发器了，直接在客户端调用就可以了。在```Vue```中调用云函数与传统的方式不一样。不需要自己发送 ``HTTP`` 请求，腾讯官方封装了```Vue```插件[vue-provider](https://github.com/TencentCloudBase/cloudbase-vue)并且在构建的项目中，已经引入在```main.js```中修改环境参数，就可以使用了。

```js
import Vue from "vue";

import App from "./App.vue";

import Cloudbase from "@cloudbase/vue-provider";

import ElementUI from 'element-ui';

import 'element-ui/lib/theme-chalk/index.css';

Vue.use(ElementUI);

window._tcbEnv = window._tcbEnv || {};

export const envId = window._tcbEnv.TCB_ENV_ID;

export const region = window._tcbEnv.TCB_REGION;

Vue.config.productionTip = false;

Vue.use(Cloudbase, {
  env: 'yindong',
  region: 'ap-shanghai'
});
new Vue({
  render: h => h(App)
}).$mount("#app");
```

同时在```index.html```中，还会默认加载一个静态的配置文件```_init_tcb-env.js```，其实就是环境的配置参数，因为已经在```main.js```配置了环境参数，因此，直接屏蔽这个文件即可。

完成这些配置之后，在```Vue```中完成添加任务的功能进行测试，但是这里有个坑，```callFunction```调用的传参与```HTTP```触发器调用的```event```入参是不一样的一定要注意```https://docs.cloudbase.net/cloud-function/how-works.html```，```callFunction```调用的云函数```event```的入参就是传入```callFunction```的```data```而没有请求信息数据，所以云函数的代码一定记得修改。

```html
<div class="box-card">
  <el-card shadow="always">
    <el-input v-model="addData.title" placeholder="请输入内容">
      <el-button slot="append" @click="addTodo">添加任务</el-button>
    </el-input>
  </el-card>
</div>
```

```js
async addTodo() {
    var todoData = this.addData;
    var back = await this.$cloudbase.callFunction({
        name: "addtodo",
        data: todoData
    });
    console.log(back);
},
```

此时，会收到一个没有权限的报错，这是因为调用云函数必须要进行登录鉴权，暂时先使用匿名登录的方式，调通接口的数据通信，后面再详细说明```Cloudbase```的用户管理服务器，但是就算是使用匿名登陆也是个坑，控制台中登录鉴权的实例代码是错误的，正确的代码示例在文档中心哪里```https://docs.cloudbase.net/authentication/anonymous.html```。

```js
mounted() {
  // 使用匿名登录
  var auth = this.$cloudbase.auth();
  auth.anonymousAuthProvider().signIn();
},
```

当然，光有代码还不够还需要到控制台中开启应用允许匿名登录的选项才行，不过一般都是默认就开通的，这里就不再细说了，继续完善获取数据的功能。增加列表渲染、选项删除以及展示详情的功能。

```html
<el-card class="box-card">
  <div v-for="(todo, key) in todos" :key="key" class="text item">
    <template v-if="todo.done == false">
      <div class="lists">
        <el-checkbox v-model="todo.done" @change="todoDone(key)">
          {{ todo.title }}
        </el-checkbox>
        <div>
          <i class="el-icon-notebook-2" @click="showTodo(key)"></i> |
          <i class="el-icon-delete" @click="deleteTodo(key)"></i>
        </div>
      </div>
      <el-divider></el-divider>
    </template>
  </div>
</el-card>
<el-drawer title="任务详情" :visible.sync="drawer">
  <div class="dra">
    <p>任务:{{ drawerTodo.title }}</p>
    <p>
      附件:
      <br />
      <img :src="drawerTodo.tempfile" alt="" />
    </p>
    <p>
      <input type="file" ref="fil" />
      <el-button @click="upFile(drawerTodo._id)"> 点击上传 </el-button>
    </p>
  </div>
</el-drawer>
```

```js
// 获取数据
async getdata() {
  var back = await this.$cloudbase.callFunction({
    name: "gettodo",
  });
  this.todos = back.result.data;
  console.log(this.todos);
},
// 删除选项
async deleteTodo(key) {
  const todoinfo = this.todos[key];
  const back = await this.$cloudbase.callFunction({
    name: 'deltodo',
    data: { _id: todoinfo._id }
  })
  console.log(back);
  this.getdata();
}
// 展示详情
async showTodo(key) {
  var todoInfo = this.todos[key];
  console.log(todoInfo.fileId);
  var back = await this.$cloudbase.getTempFileURL({
    fileList: [todoInfo.fileId],
  });
  this.drawerTodo = todoInfo;
  this.drawerTodo.tempfile = back.fileList[0].tempFileURL;
  this.drawer = true;
},
```

## 16. 文件上传

```js
async upFile(id) {
  var s = this.$refs.fil.files[0];
  // uploadFile 就是云对象存储提供的接口，集成到了vue 云开发插件中
  var back = await this.$cloudbase.uploadFile({
    // 云存储的路径
    cloudPath: "todo/" + Date.now() + s.name, // 需要上传的文件，File 类型
    filePath: this.$refs.fil.files[0],
  });

  console.log(id, back);

  var changeData = {
    _id: id,
    fileId: back.fileID,
  };

  var changeBack = await this.$cloudbase.callFunction({
    name: "puttodo",
    data: changeData,
  });

  console.log(changeBack);
},
```

## 17. 用户管理及登录授权服务

开始之前先完成基础的代码和功能搭建，首先引入路由完成注册登录的页面和对应的表单。

```s
npm install vue-router
```

添加路由文件```\src\router\index.js```。

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
Vue.use(VueRouter)
const routes = [
  {
    path: '/',
    name: 'Index',
    component: () => import('../components/Index.vue')
}, {
    path: '/register',
    name: 'Register',
    component: () => import( '../components/Register.vue')
}, {
    path: '/login',
    name: 'Login',
    component: () => import( '../components/Login.vue')
} ]
const router = new VueRouter({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})
export default router;
```

在入口文件```main.js```中引入并注册路由

```js
import router from './router'

...

new Vue({
  router,
  render: h => h(App)
}).$mount("#app");
```

完成对应的单文件组件代码，注册组件```src\components\Register.vue```。

```vue
<template>
  <div class="box-card">
    <el-row>
      <el-col>
        <el-card shadow="always">
          <el-form label-position="left" label-width="80px" :model="user">
            <el-form-item label="手机号">
              <el-input v-model="user.phone">
                <template slot="append">
                  <el-button size="mini" :disabled="disa" @click="sendCode">
                      {{sendCodeMsg}}
                </el-button>
                </template>
              </el-input>
            </el-form-item>
            <el-form-item label="验证码">
              <el-input v-model="user.code"></el-input> 
            </el-form-item>
            <el-form-item label="密码">
                <el-input v-model="user.pwd"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button size="primary" @click="sendRegister">注册</el-button> 
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>
<style>
.box-card {
  margin: 0 auto;
  width: 580px;
  margin-bottom: 20px;
}
</style>
```

登录组件```\src\components\Login.vue```。

```vue
<template>
  <div class="box-card">
    <el-row>
      <el-col>
        <el-card shadow="always">
          <el-form label-position="left" label-width="80px" :model="user">
            <el-form-item label="手机号">
              <el-input v-model="user.phoneNumber"></el-input>
            </el-form-item> 
            <el-form-item label="密码">
              <el-input v-model="user.password"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button size="primary" @click="Login">登录</el-button> 
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>
<style>
.box-card {
  margin: 0 auto;
  width: 580px;
  margin-bottom: 20px;
}
</style>
```

```js
import router from './router'

...

new Vue({
  router,
  render: h => h(App)
}).$mount("#app");
```

### 1. 注册逻辑

要使用短信验证逻辑需要在控制台开启```短信验证码登录```的选项，在登录授权里面开启。短信验证使用的是```js-sdk```，手册在[这里](https://docs.cloudbase.net/api-reference/webv2/authentication)，所以先安装。

```s
npm install @cloudbase/js-sdk
```

因为需要在多个地方使用，因此先进行封装，这里选在使用```Vue```插件的方式，```\src\assets\auth.js```。

```js
import cloudbase from "@cloudbase/js-sdk";
// 自定义插件对象
const auths = {};
auths.install = function (vue) {
    const app = cloudbase.init({
        env: "yindong01-5gyindong72a1",
    });
    const myauth = app.auth()
    vue.prototype.$auths = myauth;
}
// 导出插件
export default auths;
```

不要忘记在入口文件中导入```\src\main.js```。

```js
import router from './router'
import auths from './assets/auth.js'
Vue.use(auths) // 注册使用 auth 插件
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
```

首先完成手机验证码的发送。

```js
  // 发送注册验证码
async sendCode() {
  const res = await this.$auths.sendPhoneCode(this.user.phone);
  if (res) {
    // 验证码发送成功
    this.disabled = true;
    this.sendCodeMsg = "有效期5分钟"
  } else { 
    console.log("短信发送失败");
  } 
},
```

用户输入验证码及密码，进行验证码及手机号的验证。

```js
  // 验证码密码注册
async sendRegister(){
  const res = await this.$auths.signUpWithPhoneCode(this.user.phone,this.user.code,this.user.pwd);
  if(res){
    console.log('注册成功');
    this.$router.push({path:'/login'})
  }
}
```

验证注册成功后，跳转到登录界面。

### 2. 登录逻辑

```js
 export default {
  data() {
    return {
      user: {
        phoneNumber: "",
        password: "",
      },
    };
  },
  methods: {
    async Login() {
      const res = await this.$auths.signInWithPhoneCodeOrPassword(this.user);
      if (res) {
        this.$router.push({path:"/"})
      }
    },
  },
};
```

登录验证是非常简单的，那么是如何保持登录状态的呢。

登录状态的保持有三种不同的[方式](https://docs.cloudbase.net/api-reference/webv2/authentication)。

```local```在显式退出登录之前的```30```天内保留身份验证状态。

```session```在窗口关闭时清除身份验证状态。

```none```在页面重新加载时清除身份验证状态。

在初始化调用auth方法时，传入```\src\assets\auth.js```。

```js
import cloudbase from "@cloudbase/js-sdk";

// 自定义插件对象
var auths = {};
auths.install = function (vue) {
  const app = cloudbase.init({
      env: "yindong01-5gyindong872a1",
  });
// persistence:身份认证状态如何持久保留
// local:在显式退出登录之前的 30 天内保留身份验证状态 // session:在窗口关闭时清除身份验证状态
// none:在页面重新加载时清除身份验证状态
  const myauth = app.auth({
    persistence: "local" //用户显式退出或更改密码之前的30天一直有效 
  })
  vue.prototype.$auths = myauth;
}
// 导出插件
export default auths;
```

不同的登录状态都可以在浏览器的控制台的```Appliction```中查看。那么在不同的组件中，如何获取登录状态和登录的数据呢？
```auth```对象中，有```getLoginState```方法，看名字也知道，是获取登录状态的，在首页中使用挂载的生命周期函数进行验证，```\src\components\Index.vue```。

```js
async mounted() {
  var hls = await this.$auths.getLoginState();
  if(hls == null){
    this.$router.push({path:"/login"})
  }
  this.getdata();
},
```

当然，也可以使用```Vue-router```提供的导航守卫进行全局的登录状态验证。

## 18. 上线部署

部署非常的简单进行全量部署就可以了，在命令行切换到目录下执行```tcp```，这个命令会将云函数和```vue```代码进行打包和全量部署。部署成功之后会提供一个站点的入口，点击进去就可以访问了。不过默认提供的域名是很长很难记忆的，一般需要使用自己的域名访问应用。

在访问服务中可以添加域名，目前仅支持https的域名，首先需要申请SSL证书，添加CNAME记录执行服务器地址即可。免费证书只能适配二级域名。
