## 1. 流文件读取

```js
const fs = require('fs');
const path = require('path');
const rs = fs.createReadSteam(path.resolve(__dirname, 'name.txt'), {
    flags: 'r', // r w 读和写权限
    highWaterMark: 4, // 每次预计读取多少个
    encoding: null,
    autoClose: true, // 读取完毕，关闭文件
    start: 0,
    end: 5 // slice(start, end) 包含end的
})
// 流 默认流是暂停模式，非流动模式，内部会监控你有没有监听data事件 rs.emit('data', 123);
const arr = [];
rs.on('data', function(chunk) {
    arr.push(chunk);
    rs.pause(); // 暂停data事件的触发  rs.play(); 继续
});
rs.on('end', function() {
    console.log(Buffer.concat(arr).toString()); // 读取完毕
});

rs.on('error', function(error) {
    console.log(error);
})
```

## 2. 文件压缩zlib

基于```zlib```库和```buffer```进行文件压缩

```js
const zlib = require('zlib');
const fs = require('fs');
const rs = fs.cerateReadStream('jquery.js');
const ws = fs.cerateWriteStream('jquery.js.gz');
const gz = zlib,createGzip();
rs.pipe(gz).pipe(ws);
ws.on('error', (err) => {
    console.log('失败');
})
ws.on('finish', () => {
    console.log('完成')
})
```

## 3. 服务器压缩文件

```js
const http = require('http'); // 服务模块
const fs = require('fs'); // 读取文件
const zlib = require('zlib'); // 压缩模块

const server = http.cerateServer((req, res) => {
    const rs = fs.cerateReadStream(`www${req.url}`);
    res.setHeader('content-encoding', 'gzip'); // 设置响应头
    const gz = zlib.createGzip(); // gz压缩
    rs.pipe(gz).pipe(res); // 压缩后返回给前端
    rs.on('error', err => { // 监听失败
        res.setHeader(404);
        res.write('Not Found');
        res.end();
       console.log('读取失败')
    })
    rs.on('finish', err => { // 传输完成
       console.log('写入完成')
    })
}).listen(8080);

```

服务端压缩成```gzip```需要设置响应头，```content-encoding: gzip```, 否则无法加载。

面向字节的设备: 比如键盘

面向流的设备: 网卡，数据
