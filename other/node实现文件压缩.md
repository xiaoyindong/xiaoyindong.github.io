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
cons