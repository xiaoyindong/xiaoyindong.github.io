## 1. 概述

```websocket```具有三个优点，双向通信，自动跨域，性能高。可以通过安装```socket.io```来使用```socket```。

```s
npm install socket.io;
```

在```node```中```socket```不是独立使用的，需要依赖于```http```模块服务。浏览器首先请求```http```服务交换```key```，因为```websocket```默认就是加密的，拿到```key```之后```http```服务器将链接过继给```ws```服务器，后面变成了浏览器和```ws```服务器通信。

```js
const http = require('http');
const io = require('socket.io');

// 创建http服务，开启8080端口号
const httpServer = http.createServer().listen(8080);
// socket监听http服务
const wsServer = io.listen(httpServer);

// 当有链接的时候
wsServer.on('connection', sock => {
    // 发送
    // sock.emit
    sock.emit('time', Date.now());
    // 接收
    sock.on('aaa', (a, b, c) => {
        console.loh(a, b, c);
    })
})

```

```sock```下面只有两个东西```emit```用于消息广播用于```on```接收消息。

首先在浏览器端需要引入```socket.io.js```文件这个文件在安装```socket.io```模块时会自动提供```http://127.0.0.1:8080/socket.io/socket.io.js```。

```js
const sock = io.connect('ws://127.0.0.1:8080/');
sock.on('connect', () => {
    console.log('已链接');
    sock.emit('aaa', 12, 5,8);
    sock.on('time', (ts) => {
        console.loh(ts);
    })
});
sock.on('disconnect', () => {
    console.log('已断开');
});
```

服务器断开时会调用```sock```的```disconnect```方法，服务器启动时会自动进行链接调用```connect```方法。

- 服务端

```js
sock.on('connection', void);
sock.on('disconnect', void);
```

- 客户端

```js
sock.on('connect');
sock.on('disconnect');
```

## 2. 原理解析

```websocket```是```HTML5```新增的```API```，属于浏览器或者前端的内容。后端用的是```socket```，```socket```协议的历史相当古老基本四十年前就已经存在了。在```H5```中```websocket```自带一些安全的措施，而原生的```socket```就没什么安全性可言了。

在```node```中想要实现```socket```可以借助```node```原生的```net```模块，这是一个相对底层的网络模块，是一个```tcp```的库。```net```是```http```的底层，很多东西都需要自己去实现，比如这里可以使用```net.createServer```来创建服务。


```js
const http = require('http');
const net = require('net'); // TCP的库，可以理解为原生的Socket
const crypto = require('crypto'); // 借助加密库实现一些安全性

/* const httpServer = http.createServer((req, res) => {
    console.log('链接');
}).listen(8080); */
// http无法完成socket链接，所以采用原生的net套接字，自己写一个。
const server = net.createServer(sock=> {
    console.log('链接上了');
    sock.on('end', () => {
        console.log('客户端断开了')
    }); // 断开
    sock.once('data', (data) => {
        console.log('hand shake start...');
        // 最先过来的是http头
        console.log(data.toString());
        // 头虽然是2进制，但是都是可见数据，所以可以放心toString();
        const str = data.toString();
        // 将http头用\r\n切开
        let lines = str.split('\r\n');
        // 删除第一行和最后一行，没啥用
        lines = lines.slice(1, lines.length - 2);
        // 将所有请求头通过'分号空格'切开
        const headers = {};
        lines.forEach(line => {
            const [key, value ] = line.split(': ');
            // 将请求头变成小写
            headers[key.toLowerCase()] = val;
        })
        console.log(headers);
        if (headers['upgrade'] != 'websocket') {
            console.log('其他协议，暂不支持');
            sock.end();
        } else if (headers['sec-websocket-version'] != 13) {
            console.log('不兼容不是13的版本');
            sock.end();
        } else {
            const key = headers['sec-websocket-key'];
            // 13版本的源码是258E，可以百度的到
            const mask = '258EAFA5-47DA-95CA-C5AB0DC85B11';
            // 需要把key和mask加在一起，然后用sha1加密，再变成base64，还给客户端
            // sha1(key + mask) -> base64 -> client;
            const hash = crypto.createHash('sha1');
            hash.update(key + mask);
            const tokey = hash.digest('base64');
            // 数据以HTTP发回客户端，因为验证的过程还是http阶段, 状态值为101(正在切换协议，协议升级 Switching Protocols)
            sock.write('HTTP/1.1 101 Switching Protocols\r\nUpgrade: websocket\r\nConnection: Upgrade\r\nSec-WebSocket-Accept: ' + tokey + '\r\n'); // Upgrade: websocket告诉浏览器升级为websocket，冒号要有空格
            // 至此，握手已经结束了。因为握手的过程只有一次，所以不要用on处理，用once处理
            // 从这里开始，才是真正的数据，以后所有的数据都走这里，所以用on处理
            sock.on('data', data=> {

            })
        }
    }); // 有数据过来
}).listen(8080);

```

握手阶段记得用```once```处理，后续数据接收用```on```处理，```on```之所以写在```once```里面，是因为写在外面第一次也会执行。握手的时候不需要执行```on```。

```s
GET / HTTP/1.1
Host: localhost:8080
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Origin: file://
Sec-WebSocket-Version: 13
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like
Gecko) Chrome/65.0.3315.4 Safari/537.36
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7,es;q=0.6,fr;q=0.5,pt;q=0.4
Sec-WebSocket-Key: +0jgXtYyVeG28Gn1CLUKIg==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

1. 第一行删掉

```s
Host: localhost:8080
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Origin: file://
Sec-WebSocket-Version: 13
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like
Gecko) Chrome/65.0.3315.4 Safari/537.36
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7,es;q=0.6,fr;q=0.5,pt;q=0.4
Sec-WebSocket-Key: +0jgXtYyVeG28Gn1CLUKIg==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

2. 每行数据用": "切开

浏览器代码

```js
const ws = new Websocket('ws://127.0.0.1:8080');
ws.onopen = function() {
    console.log('链接上了')
}; // 已经链接
ws.onmessage = function() {
    console.log('接收到消息了')
}; // 收到数据
ws.onclose = function() {
    console.log('断开链接了')
}; // 断开了

```

## 3. socket数据传输

1. 服务端代码

```js
const http = require('http');
const net = require('net'); // TCP的库，可以理解为原生的Socket
const crypto = require('crypto'); // 借助加密库实现一些安全性

const server = net.createServer(sock=> {
    console.log('链接上了');
    sock.on('end', () => {
        console.log('客户端断开了')
    }); // 断开
    sock.once('data', (data) => {
        console.log('hand shake start...');
        // 最先过来的是http头
        console.log(data.toString());
        // 
        const str = data.toString();
        // 将http头用\r\n切开
        let lines = str.split('\r\n');
        // 删除第一行和最后一行，没啥用
        lines = lines.slice(1, lines.length - 2);
        // 将所有请求头通过'分号空格'切开
        const headers = {};
        lines.forEach(line => {
            const [key, value ] = line.split(': ');
            // 将请求头变成小写
            headers[key.toLowerCase()] = val;
        })
        console.log(headers);
        if (headers['upgrade'] != 'websocket') {
            console.log('其他协议，暂不支持');
            sock.end();
        } else if (headers['sec-websocket-version'] != 13) {
            console.log('不兼容不是13的版本');
            sock.end();
        } else {
            const key = headers['sec-websocket-key'];
            // 13版本的源码是258E，可以百度的到
            const mask = '258EAFA5-47DA-95CA-C5AB0DC85B11';
            // 需要把key和mask加在一起，然后用sha1加密，再变成base64，还给客户端
            // sha1(key + mask) -> base64 -> client;
            const hash = crypto.createHash('sha1');
            hash.update(key + mask);
            const tokey = hash.digest('base64');
            // 数据以HTTP发回客户端，因为验证的过程还是http阶段, 状态值为101(正在切换协议，协议升级 Switching Protocols)
            sock.write('HTTP/1.1 101 Switching Protocols\r\nUpgrade: websocket\r\nConnection: Upgrade\r\nSec-WebSocket-Accept: ' + tokey + '\r\n'); // Upgrade: websocket告诉浏览器升级为websocket，冒号要有空格
            // 至此，握手已经结束了。因为握手的过程只有一次，所以不要用on处理，用once处理
            // 从这里开始，才是真正的数据，以后所有的数据都走这里，所以用on处理
            sock.on('data', data=> {
                console.log('有数据过来了');
                // 数据是一个buffer，数据包
                console.log(data);
                // 取第一个字节的位运算，一个&是位，两个是逻辑运算
                console.log(data[0]&0x001);
                const FIN = data[0]&0x001;
                const opcode = data[0]&0x0F0;
                const mask = data[1]&0x001;
                const payload = data[1]&0x0FE;
            })
        }
    }); // 有数据过来
}).listen(8080);

```

2. 接收到的数据，是数据包、帧，是有结构的，和字符串不一致，所以不能转成字符串

1位(bit)
8位=>1字节(byte)


```s
81       9c       11       2d       f8       bd         数据.....
10000001 10011100 00010001 00101101 11111000 10111101

11 2d f8 bd
6a 0f 96 dc 7c 48 da 87 33 40 8b da 33 01 da d9 70 59 99 9f 2b 76 c9 8f 3d 18 a5 c0

1 000 0001   1 0011100 00010001 00101101 11111000 10111101
F RSV opcode M payload masking-key
I            A 28个字
N            S
             K
```


## 4. 帧结构

```s
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+-------------------------------+
|     Extended payload length continued, if payload len == 127  |
+-------------------------------+-------------------------------+
|                               |Masking-key, if MASK set to 1  |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------+-------------------------------+
|                     Payload Data continued ...                |
+---------------------------------------------------------------+
|                     Payload Data continued ...                |
+---------------------------------------------------------------+

FIN               1bit 是否最后一帧
RSV               3bit 预留
Opcode            4bit 帧类型
Mask              1bit 掩码，是否加密数据，默认必须置为1
Payload           7bit 长度
Masking-key       1 or 4 bit 掩码
Payload data      (x + y) bytes 数据
Extension data    x bytes  扩展数据
Application data  y bytes  程序数据
```

浏览器代码 

```js
const ws = new Websocket('ws://127.0.0.1:8080');
// 原生没有emit，自己封装一个
ws.emit = function(name, ...args) {
    ws.send(JSON.stringify({
        name,
        data: [...args]
    }))
}
ws.onopen = function() {
    console.log('链接上了');
    // ws.send('dadadadadasda'); // 发送数据，只有一个参数一个大字符串
    ws.emit('msg', 12, 5, 8);
}; // 已经链接
ws.onmessage = function() {
    console.log('接收到消息了')
}; // 收到数据
ws.onclose = function() {
    console.log('断开链接了')
}; // 断开了

```
