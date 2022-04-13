## 1. 反向代理

反向代理说起来也很简单，比如你拨打一个电话号码，对方问你要找谁，然后帮你转接到对应的人的分机上。每个人拨打的都是相同的号码，但也都会通过转接找到对应的人，如果有人拨打错了，也会有友好的提示。

你没有直接拨打对应人的号码，而是拨打了统一转接人的电话，这个人所做的事情就是反向代理, 反向代理是他会把你的问题转述给对应的人，并且把对应人的回答再转述给你。

反向代理对服务器的要求很高，```nginx```是这方面的专家，所以大部分我们的服务器都使用```nginx```作为入口服务器，起到传达人的作用。

举例来说一个```web```请求走进服务首先会经过```nginx```，然后由```nginx```转发到对应的应用服务器。比如你想访问一张图片，你首先进入到```nginx```，```nginx```通过你想要的图片路径到对应的图片服务器拿到这张图片，再返回给你。

为什么会有反向代理呢？因为我们的应用服务由于要求开发效率非常的高，所以他的运行效率是很低的，他的```qbs```，```tps```并发都是受限的，在处理业务的同时很难兼容处理用户的请求。

一般这种情况我们会搭建很多台服务，组成一个集群，向用户提供高可用性。正是因为很多服务构成集群的时候，这么多台服务暴露的地址不同，才需要```nginx```具有反向代理的功能，可以把请求传导给应用服务。而用户面临的只是```nginx```一台服务的地址。

对于用户的请求收口在```nginx```层也可以提高网站的安全性和便宜性。假设我们想要修改接口的响应头，只需要修改```nginx```就可以了，而不需要每台服务都去修改，想要停止服务访问，也只需要关掉```nginx```, 不需要每台服务都去关闭。

## 2. 正向代理

上面举了个打电话的例子讲解反向代理，其实反向代理代理的是服务器，客户端不知道实际提供服务的服务器，也就是打电话的人每次电话打给的都是那个电话转接人。

正向代理是假设春节你想买一张火车票，你找了黄牛，黄牛帮你买到了票，很多人都找这个黄牛，黄牛帮很多人买了票，这个黄牛就是正向代理，对于服务来说每次买票的都是黄牛。服务不知道真正买票的人是谁。正向代理时服务端不知道真正发起请求的客户端。

换句话说，反向代理要求代理服务器和提供服务的服务器在同一个网络中，正向代理要求客户端和代理服务器在同一个网络中。

## 3. 负载均衡

负载均衡是用来解决大流量问题，因为每台服务器可承受的访问量是有限的，随着公司的发展访问量会越来越大，这个时候我们会选择通过增加服务器的方式来平衡每台服务器所受的压力。

前面也说了我们的应用服务因为要求开发效率非常的高，所以他的运行效率是很低的，他的```qbs```，```tps```并发都是受限的，所以我们需要把很多这样的应用服务组成一个集群，向用户提供高可用性。

而每一次```web```请求进来，```nginx```都可以选择一台压力较少的服务器，然后将请求发送给这台服务进行处理。这就是负载均衡。

在服务领域一旦很多应用服务构成集群，他一定会带来两个需求，第一个需求我们需要动态的扩容，我们在增加一台服务器的时候，不可能关闭网站，让网站停止使用，他一定是一个无感知的过程，这就是动态扩容```nginx```是支持的。也就是你可以不关闭的情况下增加一台服务。

第二个则是有些服务出问题的时候我们需要做容灾，假设我们的服务器集群中一台服务器出现了问题，是不应该影响用户访问的，```nginx```会把请求发送给其它正常的服务器保证线上用户的正常使用。

## 4. 静态资源缓存

在一个链路中，```nginx```是处在企业内网的一个边缘节点，也就是网站的最外层，所有的请求和访问都是由```nginx```做转发，这样随着我们网络链路的增长，用户体验到的时延会增加。

所以如果我们能把一些所有用户看起来不变的，或者在一段时间内看起来不变的动态内容缓存在```nginx```中，由```nginx```直接向用户提供访问，那这样用户的时延就会减少很多。

所以反向代理会衍生出另外一个功能叫缓存，他能够加速我们的访问，而很多时候我们在访问像```css```或者```js```文件，或者这样一些小图片，那么这样静态的资源是没有必要由应用服务来访问的，他只需要通过本地文件，系统上放置的静态资源，直接由```nginx```提供访问就可以。这是```nginx```的静态资源功能。

## 5. 开启gzip

首先打开```nginx.conf```文件，找到```http```代码块中的```gzip```相关选项，打开```gzip(off -> on)```, ```gzip_min_length```是小于多少字节不再执行压缩，因为小于一定的字节```http```传输直接就可以发送了，压缩反而消耗```cpu```性能，```gzip_comp_level```代表压缩级别，```gzip_types```是针对某些类型的文件才做```gzip```压缩。

```s
http {
    ...
    gzip on;
    gzip_min_length 1;
    gzip_comp_level 2;
    gzip_types text/plain applicaton/x-javascript text/css image/png;
    ...
}
```

```s
gzip on;                     #开启gzip压缩功能
gzip_min_length 10k;         #设置允许压缩的页面最小字节数; 这里表示如果文件小于10个字节，就不用压缩，因为没有意义，本来就很小.
gzip_buffers 4 16k;          #设置压缩缓冲区大小，此处设置为4个16K内存作为压缩结果流缓存
gzip_http_version 1.1;       #压缩版本
gzip_comp_level 2;           #设置压缩比率，最小为1，处理速度快，传输速度慢；9为最大压缩比，处理速度慢，传输速度快; 这里表示压缩级别，可以是0到9中的任一个，级别越高，压缩就越小，节省了带宽资源，但同时也消耗CPU资源，所以一般折中为6
gzip types text/css text/xml application/javascript;      #制定压缩的类型,线上配置时尽可能配置多的压缩类型!
gzip_disable "MSIE [1-6]\.";       #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
gzip vary on;    #选择支持vary header；改选项可以让前端的缓存服务器缓存经过gzip压缩的页面; 这个可以不写，表示在传送数据时，给客户端说明我使用了gzip压缩
```

## 6. proxy_pass

可以通过```proxy_pass```将请求转发到对应的服务，假设服务器中存在一个```7001```的服务，可以通过本地ip```127.0.0.1:7001```访问，这个时候就可以使用```proxy_pass```在```location```中进行配置。

```js
server {
    listen     8080;
    ...
    location / {
        proxy_pass http://127.0.0.1:7001;
    }
}
```

## 7. upstream

假设有两台服务，一个端口```7001```一个端口```7002```，可以通过```upstream```来实现负载均衡，在```http```代码块中通过```upstream```创建服务池。然后在```server```的```location```中通过```proxy_pass```转发至这个服务池。这样就实现了一个简单的负载均衡。

```js
http {
    upstream myservice {
        server 127.0.0.1:7001;
        server 127.0.0.1:7002;
    }
    server {
        listen 80;
        location / {
            proxy_pass http://myservice;
        }
    }
}

```

## 8. 配置https

在````nginx````配置文件夹中新建 ```cert``` 文件夹用于存放域名证书。

```s
cd /usr/local/nginx/conf
mkdir cert
```
修改nginx配置文件 ```/usr/local/nginx/conf```

```s
/usr/local/nginx/conf
vi nginx.conf
```

去掉```server```模块前端的注释，```https```使用```443```端口。

```ssl_certificate```配置下载的证书```cert.pem```，```ssl_certificate_key```为下载的```cert.key```。

```s
server {
    listen       443 ssl;
    server_name  localhost;

    ssl_certificate      certs/cert.pem;
    ssl_certificate_key  certs/cert.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        root   html;
        index  index.html index.htm;
    }
}
```

重启```nginx```就生效了。可以将```http``` 转发至 ```https```，强制使用```https```访问。

修改```nginx```配置文件 ```/usr/local/nginx/conf```

```s
cd /usr/local/nginx/conf
vi nginx.conf
```

```s
server {
    listen 80;
    server_name zhiqianduan.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent; 
}
```

## 9. 开启http2

修改```nginx```配置文件 ```/usr/local/nginx/conf```

```s
/usr/local/nginx/conf
vi nginx.conf
```

在```443```后面添加```http2```就可以了。

```s
server {
    listen 443 ssl http2;
    server_name localhost;
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
}
```

## 10. 搭建静态Web服务器

在```nginx```的安装目录下新建一个```www```文件夹，在```www```中放置一些静态文件。

然后编辑```conf/nginx.conf```文件找到```server```代码块中，```listen```配置监听哪个端口，这里用```8080```端口，然后配置```location```，所有的```url```请求都访问到```www```文件夹，使用```/```表示所有的请求。

需要指定```url```的后缀要与文件目录后面的后缀一一对应，有两种用法，```root```和```alias```，```root```有一个问题，他会把```url```中的一些路径带到目录中来，所以我们通常使用```alias```。

```alias```默认就是```nginx```安装目录的```www```目录下，后面的路径与```url```路径是一一对应的。

```s
server {
    listen 8080;
    ...
    location / {
        alias www/;
        ...
    }
    ...
}
```

配置之后重启```nginx```就可以看到效果了。在浏览器中访问```localhost:8080```就可以访问。

## 11. 打开目录结构

跟目录下有一个文件夹叫```dlib```，假定需要把```dlib```中的文件，或者文件夹及其目录结构信息分享给用户，用户来决定使用哪些文件。

```nginx```提供了一个官方模块叫做```autoindex```当访问以```/```结尾的```url```时，显示这个目录的结构。使用方法也特别简单，就是```autoindex on```加入这样一个指令就可以了。

```s
location / {
    autoindex on;
}
```

reload之后代码就生效了，他会把我们所访问的文件夹内所有文件列出来，当我们打开一个目录时，可以继续显示这个目录中的文件，这是一个很好的静态资源帮助功能。

## 12. 流量限制

还有一个非常常见的功能，比如公网带宽是有限的，当有很多并发用户使用带宽时，会形成一个争抢关系，可以让用户访问某些大文件的时候来限制他的速度，节省足够的带宽给用户访问一些必要的小文件，如```css```，```js```等。

可以使用```se```t命令，配合一些内置的变量去实现这样的功能，比如说加上```set $limit_rate 1k```，他就在限制```nginx```向客户浏览器发送响应的一个速度。意思是每秒传输1k数据到浏览器中。

```s
location / {
    set $limit_rate 1k;
}
```

## 13. 日志功能

首先需要定义日志格式，可以使用```log_format```指令，定义的过程可以使用变量。```$remote_addr```为远端的地址，也就是浏览器客户端的```ip```地址，```$time_local```表示当时的时间。```$status```是返回的状态码。

```s
http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
}
```

日志定义好后需要给他一个名字，比如说这里命名为```main```，因为日志格式可以定义多条，每条需要不同的名字进行区分。

在实际情况中对不同的域名要做不同格式的日志记录。不同```url```的文件以及一些反向代理等不同用途的操作记录记录也都要记录不同的日志格式。

配置好```log_format```之后还要去设置日志记录在哪里，我们可以用```access_log```指令，```access_log```所在的位置决定了他所属区域的请求会记录到哪里，第一个参数是日志的路径```logs/yindong.log```，第二个参数是日志记录的格式```main```。

比如```access_log```放在```server```下，也就是这个域名或者这个端口的请求日志，都会记录到这个文件中。

```js
server {
    listen 8080;
    access_log logs/yindong.log main;
    location / {
        alias www;
    }
}
```

当我们配置好```yindong.log```后，所有的请求在完成之后都会记录下一条日志，可以进入```logs/yindong.log```中查看。

## 14. 获取真实IP

```nginx```服务设置

```s
location / {
　　proxy_set_header Host $http_host;
　　proxy_set_header X-Real-IP $remote_addr;
　　proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
　　proxy_pass http://xx.xx.xx.xx:xxxx/;
}
```

```node```服务中获取用户真实```ip```

```js
var ip = req.headers['x-forwarded-for'] || req.connection.remoteAddress || req.socket.remoteAddress || req.connection.socket.remoteAddress;
 
var realip = ip.match(/\d+.\d+.\d+.\d+/);
ip= realip ? realip.join('.') : null;
```
