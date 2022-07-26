## 1. 数据交互

对于```formdata```来书，主要能力用在上传文件上更方便，相反普通的请求并没有```ajax```方便。

使用时创建```formdata```，通过```set(key, value)```设置一个值，如果设置两次相同的```key```，第二个会覆盖第一个，在```form```中可以设置多个相同的```name```，服务会接收一个数组，如果想实现这种，可以使用```append```，```append(key, value)```。

```get(key)```返回设置的值，如果不存在返回```undefined```；```getAll(key)```获取同一个```key```下的所有值；```delete(key)```删除设置的值

```js
const fileDom = document.getElementById('file');
const data = new FormData();
data.set('name', 'yindong');
Array.from(fileDom.files).forEach(file => {
    data.append('f1', file);
})
const oAjax = new XMLHttpRequest();

let arr = [];
data.forEach((value, key) => {
    arr.push(`${encodeURLComponent(key)}=${encodeURLComponent(value)}`);
})

oAjax.open(`GET`, `http://127.0.0.1:8080?${arr.join('&')}`, true);
// post
// oAjax.setRequstHeader('Content-Type', 'application/x-www-form-urlencoded');
oAjax.send();

oAjax.onreadystatechange = function() {
    if (oAjax.readyState == 4) {
        if (oAjax.status >= 200 && oAjax.status < 300 || oAjax.status == 304) {
            alert('成功');
        } else {
            alert('失败');
        }
    }
}

```

服务器依赖一个```body-parser```插件。

```s
npm install express body-parser multer -S;
```

```js
const express = require('express'); // 主体
const multer = require('multer'); // 接收文件POST数据
const body = require('body-parser');// 接收普通数据

const server = express();
server.use(body.urlencoded({ extended: false }));
const multerObj = multer({ dest: './upload/'});

server.use(multerObj.any());

server.listen(8080);
server.post('/', (req, res) => { // post请求会走这个
    res.send('ok'); // express的res将write和end合并成了send
    console.log(req.body); // body-parser 会在req上添加一个body
    console.log(req.files); // multer 会在req上添加一个files
})
server.get('/api', () => { // get请求会走这个
    
})
server.use('/', () => { // 所有请求都走这个
    
})
// 设置静态服务器的地址
server.use(express.static('./www/'));
```

实现上传文件进度

```js
const fileDom = document.getElementById('file');
const data = new FormData();
Array.from(fileDom.files).forEach(file => {
    data.append('f1', file);
})
const oAjax = new XMLHttpRequest();


oAjax.upload.onprogress = function(ev) {
    const percent = ev.lo