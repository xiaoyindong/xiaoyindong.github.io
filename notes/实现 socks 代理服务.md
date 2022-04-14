## 1. 前言

公司的前端网站有点多，而且每次访问都需要登录，即使做了记住密码功能也会定期要求登录，特别的不方便。主要我觉得这一点都不够互联网。我想实现开发一个桌面软件，只要这个软件登录了访问其他的网站就都不需要再登录了，岂不是爽歪歪。

原理其实还是比较简单的，本地搭建一个代理服务器，将所有的请求发送到这个代理服务器，再由代理服务器转发到对应的服务器，类似于抓包工具。但是抓包工具一般代理的是http和https。又去看了看VPN，是socks代理，似乎更好一些。

在github上找到了这个[你也能写个 Shadowsocks](https://github.com/gwuhaolin/blog/issues/12)，作者太良心了，写的非常详细。

## 2. 动手

这里选择nodejs来做，[你也能写个 Shadowsocks](https://github.com/gwuhaolin/blog/issues/12)作者也提供了node版本。

socks5 是 tcp/ip 层面的网络代理协议，这里要使用net模块来搭建服务。因为http模块封装了很多的细节，所以net模块只能自己来封装。

首先客户端会向服务端发送连接，客户端发送的数据包包含三个字段。ver，nmethods，methods。
ver代表 socks 的版本，默认为0x05，其固定长度为1个字节，nmethods表示第三个字段method的长度，也是1个字节，method表示客户端支持的验证方式，可以有多种，长度是1-255个字节。

- 0x00：NO AUTHENTICATION REQUIRED（不需要验证）

- 0x01：GSSAPI

- 0x02：USERNAME/PASSWORD（用户名密码）

- 0x03: to X'7F' IANA ASSIGNED

- 0x80: to X'FE' RESERVED FOR PRIVATE METHODS

- 0xFF: NO ACCEPTABLE METHODS（都不支持，没法连接了）

```js
const net = require('net');

const config = {
    port: 1081,
}

const server = net.createServer((sock) => {
    sock.on('error', (err) => {
        console.log(`远程到代理服务器的连接发生错误，错误信息：${err.message}`);
        sock.destroy();
    });
    console.log(`接受连接：${sock.remoteAddress}:${sock.remotePort}`);

    sock.once('readable', () => {
        const data = sock.read();
        console.log(data); // {"type":"Buffer","data":[5,1,0]}
    });
});

server.listen(config.port, () => {
    server.on('error', function (err) {
        console.log(`代理服务器发生错误，错误信息：${err.message}`);
        server.close();
    });
    console.log(`代理服务器启动，监听0.0.0.0:${config.port}...`);
})
```

我这里用的mac电脑，可以在系统偏好设置 -> 网络 -> 高级 -> 代理 中选中socks代理，代理服务器填写127.0.0.1:1081，这里用的1081端口号，如果冲突可以自行修改。windows电脑可以自己百度一下如何代理。

上面代码中打印了{"type":"Buffer","data":[5,1,0]}，5，1，0分别是ver，nmethods，methods，虽然展示为5，1，0其实他们都是16进制，实际为0x05，0x01，0x00。

| -	| VERSION | METHODS_COUNT | METHODS |
| -- | -- | -- | -- |
| data | 0x05 | 0x01 | 0x00 |
| 说明 | 协议版本，目前固定0x05 | 客户端支持的认证方法数量 | 不需要认证 |

服务端收到客户端的验证信息之后，就要回应客户端，服务端需要客户端提供哪种验证方式的信息。服务端回应ver和method。根据请求的协议需要添加协议验证与响应信息。这里服务端不需要验证的话，可以这么回应客户端。ver代表 socks socks 默认为0x05，其固定长度为1个字节，method代表服务端需要客户端按此验证方式提供的验证信息，其值长度为1个字节，可为上面六种验证方式之一。

| VER | METHOD |
| -- | -- |
| 0x05 | 0x00 | 

```js
sock.once('readable', () => {
        const data = sock.read();
        console.log(data); // {"type":"Buffer","data":[5,1,0]}
        sock.write(Buffer.from([0x05, 0x00]), (err) => {

        })
});
```

完善一下，加上对应的判断。

```js
sock.once('readable', () => {
        const data = sock.read();
        if (!data && data[0] !== 0x05) {
            sock.destroy();
            return;
        }
        sock.write(Buffer.from([0x05, 0x00]), (err) => {
            if (err) {
                console.log(`写入数据失败,失败信息:${err.message}`);
                socket.destroy();
            }
            // 其他
        })
});
```

客户端发起的连接由服务端验证通过后，客户端下一步应该告诉真正目标服务的地址给服务器，服务器得到地址后再去请求真正的目标服务。也就是说客户端需要把 Google 服务的地址google.com:80告诉服务端，服务端再去请求google.com:80。目标服务地址的格式为 (IP或域名)+端口，客户端需要发送的包格式如下。

| VER | CMD | RSV | ATYP | DST.ADDR | DST.PORT |
| -- | -- | -- | -- | -- | -- |
| 1 | 1 | 0x00 | 1 | Variable | 2 |
| SOCKS协议版本，固定0x05 | 命令(本文只实现CONNECT) | 保留字段 | 目标服务器地址类型(本文仅实现IP V4与域名) | 目标地址 | 端口 |

ver代表 socks 协议的版本，socks 默认为0x05，其值长度为1个字节，cmd代表客户端请求的类型，值长度也是1个字节，有三种类型。

connect 0x01；

bind: 0x02；

udp： ASSOCIATE 0x03；

rev是保留字，值长度为1个字节；

atyp代表请求的远程服务器地址类型，值长度1个字节，有三种类型；

ipv4：address: 0x01；

domaonname: 0x03；

ipv6： address: 0x04；

dst.addr：代表远程服务器的地址，根据 atyp 进行解析，值长度不定；

dst.port：代表远程服务器的端口，要访问哪个端口的意思，值长度2个字节。

需要说的是在域名类型下，域名地址第1个字节为域名长度，剩下字节为域名名称字节数组

```js
sock.write(Buffer.from([0x05, 0x00]), (err) => {
    if (err) {
        console.log(`写入数据失败,失败信息:${err.message}`);
        socket.destroy();
    }
    sock.once('readable', () => {
        const data = sock.read();
        remoteAddr = Buffer.from(data.slice(4, data.length - 2)).toString().trim().replace(/\u0000|\u0001|\u0002|\u0003|\u0004|\u0005|\u0006|\u0007|\u0008|\u0009|\u000a|\u000b|\u000c|\u000d|\u000e|\u000f|\u0010|\u0011|\u0012|\u0013|\u0014|\u0015|\u0016|\u0017|\u0018|\u0019|\u001a|\u001b|\u001c|\u001d|\u001e|\u001f/g, '');
        remotePort = data.readUInt16BE(data.length - 2);//最后两位为端口值
        console.log(`连接到 ${remoteAddr}:${remotePort}`);
    });
});
```

服务端在得到来自客户端告诉的目标服务地址后，便和目标服务进行连接，不管连接成功与否，服务器都应该把连接的结果告诉客户端。在连接成功的情况下，服务端返回的包格式如下。

| VER | REP | RSV	ATYP | BND.ADDR | BND.PORT |
| 1 | 1 | 0x00 | 1 | Variable | 2 |
| SOCKS协议版本，固定0x05 | 响应命令 | 保留字段 | 代理服务器地址类型 | 代理服务器地址 | 代理服务器端口 |

VER：代表 SOCKS 协议的版本，SOCKS 默认为0x05，其值长度为1个字节；

REP代表响应状态码，值长度也是1个字节，有以下几种类型

0x00 代理服务器连接目标服务器成功
0x01 代理服务器故障
0x02 代理服务器规则集不允许连接
0x03 网络无法访问
0x04 目标服务器无法访问（主机名无效）
0x05 连接目标服务器被拒绝
0x06 TTL已过期
0x07 不支持的命令
0x08 不支持的目标服务器地址类型
0x09 - 0xFF 未分配

RSV：保留字，值长度为1个字节

ATYP：代表请求的远程服务器地址类型，值长度1个字节，有三种类型

IP V4 address： 0x01

DOMAINNAME： 0x03

IP V6 address： 0x04

BND.ADDR：表示绑定地址，值长度不定。

BND.PORT： 表示绑定端口，值长度2个字节

响应的服务端地址默认是1，也就是ip v4，地址与端口可以直接就写0x00。

```js
sock.once('readable', () => {
    const data = sock.read();
    const remoteAddr = Buffer.from(data.slice(4, data.length - 2)).toString().trim().replace(/\u0000|\u0001|\u0002|\u0003|\u0004|\u0005|\u0006|\u0007|\u0008|\u0009|\u000a|\u000b|\u000c|\u000d|\u000e|\u000f|\u0010|\u0011|\u0012|\u0013|\u0014|\u0015|\u0016|\u0017|\u0018|\u0019|\u001a|\u001b|\u001c|\u001d|\u001e|\u001f/g, '');
    const remotePort = data.readUInt16BE(data.length - 2);//最后两位为端口值
    console.log(`连接到 ${remoteAddr}:${remotePort}`);
    // 响应给客户端
    sock.write(Buffer.from([0x05, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]
});
```

代理部分很简单，创建一个TCP客户端请求，将请求信息转发给解析出来的IP与端口就行了。

```js
const remoteSock = net.createConnection(remotePort, remoteAddr, () => {
    sock.write(Buffer.from([0x05, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]), (err) => {
        if (err) {
            console.log(`写入数据失败,失败信息:${err.message}`);
            sock.destroy();
        }
        // remoteSock.pipe(sock);
        // sock.pipe(remoteSock);
        remoteSock.pipe(through2(function (chunk, enc, callback) {
            // console.log(chunk, 111111);
            callback(null, chunk);
        })).pipe(sock);
        sock.pipe(through2(function (chunk, enc, callback) {
            // console.log(chunk, 222222);
            callback(null, chunk);
        })).pipe(remoteSock);
    });
});
remoteSock.on('error', function (err) {
    console.log(`连接到远程服务器 ${remoteAddr}:${remotePort} 失败,失败信息:${err.message}`);
    remoteSock.destroy();
    sock.destroy();
    return;
});
```

SOCKS5 协议的目的其实就是为了把来自原本应该在本机直接请求目标服务的流程，放到了服务端去代理客户端访问。
其运行流程总结如下：

本机和代理服务端协商和建立连接；本机告诉代理服务端目标服务的地址；代理服务端去连接目标服务，成功后告诉本机；本机开始发送原本应发送到目标服务的数据给代理服务端，由代理服务端完成数据转发。

完整代码：

这里引入了through2，是准备修改请求，代码还没写，欠着吧。

```js
const net = require('net');
const through2 = require('through2');

const config = {
    port: 1081,
}

const server = net.createServer((sock) => {
    sock.on('error', (err) => {
        console.log(`远程到代理服务器的连接发生错误，错误信息：${err.message}`);
        sock.destroy();
    });
    console.log(`接受连接：${sock.remoteAddress}:${sock.remotePort}`);

    sock.once('readable', () => {
        const data = sock.read();
        if (!data && data[0] !== 0x05) {
            sock.destroy();
            return;
        }
        sock.write(Buffer.from([0x05, 0x00]), (err) => {
            if (err) {
                console.log(`写入数据失败,失败信息:${err.message}`);
                socket.destroy();
            }
            sock.once('readable', () => {
                const data = sock.read();
                const remoteAddr = Buffer.from(data.slice(4, data.length - 2)).toString().trim().replace(/\u0000|\u0001|\u0002|\u0003|\u0004|\u0005|\u0006|\u0007|\u0008|\u0009|\u000a|\u000b|\u000c|\u000d|\u000e|\u000f|\u0010|\u0011|\u0012|\u0013|\u0014|\u0015|\u0016|\u0017|\u0018|\u0019|\u001a|\u001b|\u001c|\u001d|\u001e|\u001f/g, '');
                const remotePort = data.readUInt16BE(data.length - 2);//最后两位为端口值
                console.log(`连接到 ${remoteAddr}:${remotePort}`);
                const remoteSock = net.createConnection(remotePort, remoteAddr, () => {
                    sock.write(Buffer.from([0x05, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]), (err) => {
                        if (err) {
                            console.log(`写入数据失败,失败信息:${err.message}`);
                            sock.destroy();
                        }
                        remoteSock.pipe(through2(function (chunk, enc, callback) {
                            callback(null, chunk);
                        })).pipe(sock);
                        sock.pipe(through2(function (chunk, enc, callback) {
                            callback(null, chunk);
                        })).pipe(remoteSock);
                    });
                });
                remoteSock.on('error', function (err) {
                    console.log(`连接到远程服务器 ${remoteAddr}:${remotePort} 失败,失败信息:${err.message}`);
                    remoteSock.destroy();
                    sock.destroy();
                    return;
                });
            });
        });
    });
});

server.listen(config.port, () => {
    server.on('error', function (err) {
        console.log(`代理服务器发生错误，错误信息：${err.message}`);
        server.close();
    });
    console.log(`代理服务器启动，监听0.0.0.0:${config.port}...`);
})
```

## 查看命令

```
man networksetup
```

## 设置代理

```
networksetup -setwebproxy "Wi-fi" 127.0.0.1 1080
networksetup -setsocksfirewallproxy "Wi-fi" 127.0.0.1 1081
networksetup -setsocksfirewallproxystate "Wi-fi" off    # 关闭
```

```
networkservice=$(networksetup -listallnetworkservices |head -n 3|tail -n 1) 
networksetup -setwebproxystate $networkservice off    # 关闭
networksetup -setsocksfirewallproxy $networkservice 127.0.0.1 8080
```

## 关闭代理

```
# 获取wifi名称
networkservice=$(networksetup -listallnetworkservices |head -n 3|tail -n 1) 
networksetup -setwebproxystate $networkservice off    # 关闭
```

### 参考来源

- [1] [你也能写个 Shadowsocks](https://github.com/gwuhaolin/blog/issues/12 "gwuhaolin"
- [2] [NodeJS实现简单的Socks5代理服务](https://www.jianshu.com/p/b3b4941f7723 "关爱单身狗成长协会"
