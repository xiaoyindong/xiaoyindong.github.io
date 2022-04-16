## 1. web服务器搭建

```node```搭建简单服务器，使用到的模块

1. http 创建服务

2. fs 读写文件

3. url 接续请求路径+参数

```js
// 导入包文件
const http = require('http');
const fs = require('fs');
const url = require('url');

// 创建服务
const server = http.createServer((req, res) {
    // req 请求对象
    // res 响应对象
    
    // 解析请求参数
    const params = url.parse(req.url, true);

    fs.readFile(`www${req.url}`, (err, data) => { // 通过url路径，拿到www文件夹中对应的文件
        if (err) { // 如果文件不存在，返回404
            res.writeHeader(404); // 设置状态码
            res.write('404'); // 写入返回值，需为字符串
        } else {
            res.writeHeader(200);
            res.write(data);
        }
        // 请求断开
        res.end();
    })
});
// 监听端口号
server.listen(8080);
```

- writeHeader: 设置响应码 

- write: 返回的就是响应体，即前端拿到的数据

- setHeader: 设置响应头

上面的服务是一个```web```的静态服务器，也即适配```get```请求，对于```post```请求来说，由于请求数据量较大，需要持续接收数据，这里使用```querystring```解析前端传入的参数

```js
const http = require('http');
const querystring = require('querystring'); // 用于解析前端传入的参数，将a=1&b=2 解析为 {a:1, b: 2};
const server = http.createServer((req, res) => {
    const arr = []; // 创建一个数组，用于接收前端传入的数据，前端传入的数据为Buffer类型。
    req.on('data', data => { // 持续接收数据
        arr.push(data);
    });
    req.on('end', data => { // 数据传输完成
        const str = Buffer.concat(arr);
        const post = querystring.parse(str);
        console.log(post); // 获取到传入的参数
        res.end(); // 结束
    })
});
server.listen(8080);
```
由此可见，```post```请求数据量较大，需要使用```on```绑定```data```事件。```end```表示接收结束。

使用字符串接收流文件时会有问题。因为不是所有的文件都可以转换为字符串查看，比如说视频流，图片流

```GET```数据在```req.url```里

```POST```数据在```body```里面，数据比较大，使用```data``` 和 ```end``` 事件处理

## 2. 实现一个简单版express

```js
const http = require('http'); // 引入http 模块，基于http模块进行开发
const url = require('url'); // 引入url模块，解析url和前端传入的params

let router = []; // 定义路由集合

class MyExpress {
    listen() { // 设置监听方法
        const server = http.createServer((req, res) => {
            const { pathname } = url.parse(req.url, true); // 解析url并获取请求路径
            router.forEach(route => {
                // 从注册的路由中遍历出符合的路由
                const { path, method, handler } = route;
                if (pathname === path && req.method.toLowerCase() === method) {
                    return handler(req, res);
                }
                if (path === '*') {
                    return handler(req, res);
                }
            })
        });
        server.listen(...arguments);
    }
    get(path, handler) {
        if (typeof path === 'string') {
            router.push({
                path,
                method: 'get',
                handler
            });
        } else {
            router.push({
                path: '*',
                method: 'get',
                handler: path
            })
        }
    }
}

module.exports = () => {
    return new MyExpress();
} 

```