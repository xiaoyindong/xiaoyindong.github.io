## 1. 概述

```Koa```的设计思想是中间件，类似于流水线。涉及```数据交互```、```cookie```、```session```、```router```以及模板引擎。```koa v1```基于```generator```，```koa v2```基于```generator + async + await```，```koa v3```基于```await```。

## 2. 搭建简单服务器

```s
npm i koa koa-static koa-better-body koa-convert koa-router
```

```koa```：主体，自身带```cookie```功能。

```koa-static```：静态文件。

```koa-better-body```：管理请求，```get```，```post```，```upload```。

```koa-convert```：中间件过度koa版本兼容。

```koa-router```：路由。

```koa```丢弃了原生```node```的写法，```express```保留原生```node```写法，```koa```强依赖```router```，必须用```router```。

```js
const koa = require('koa');
const router = require('koa-router');
const server = new koa();
server.listen(8080);

const rout1 = router();
server.use(rout1.routes());
rq.get('/a', async (ctx, next) => {
    // ctx.req 原生req对象
    // ctx.request 封装好的req对象
    // ctx.res 原生的res
    // ctx.response 封装好的response
    // ctx.request.headers 获取请求头
    ctx.response.status = 403; // 设置状态码
    ctx.response.set('a', 12); // 设置响应头
    ctx.response.body = 'abc'; // 设置返回参数
})

// const router = require('koa-router');
// const r1 = router();
// server.use(r1.routes());
// r1.get(路径, async() => {})
// r1.post(路径, async() => {})
// r1.put(路径, async() => {})
// r1.delete(路径, async() => {})
// r1.use(路径, async() => {})
```

## 3. 静态资源

```js
const static = require('koa-static');
server.use(static(path.resolve('www'));
```

```koa-static```返回的文件没有压缩，不建议使用，想要压缩需要传入```gzip: true```，后缀名用```gz```不过太麻烦了可以用```koa-static-cache```。

```s
npm install koa-static-cache --save
```

```js
const static = require('koa-static-cache');
server.use(static(path.resolve('www'));
```

## 4. 请求数据

动态路由，路由参数在```ctx.params```中。

```js
const betterBody = require('koa-better-body');
const convert = require('koa-convert');
server.use(r1.routes());
server.use(convert(betterBody({
    uploadDir: path.resolve('./upload'),
    keepExtensions: true
}))); // 使用convert将老的中间件，支持最新的功能
r1.get('/api/:name', async (ctx, next) => {
    console.log(ctx.params);
})
```
ctx.request.fields; // 既有普通参数，又有附件参数
ctx.request.files; // 只有附件信息

## 5. cookie

```koa```自带```cookie```，直接可以使用```ctx.cookies```。

```ctx.cookies.get('key')```获取```cookie```

```ctx.cookies.set('b', 5, {})```设置```cookie```

## 6. session

需要手动安装```koa-session```插件

```js
const session = require('koa-session');
// session 必须加要一组keys，防止被劫持
server.keys = ['fsdfsfsdfdfds', 'vbcbfgbdvdfbfdb', 'nnhgnhtfsfsdfsd'];
// 需要参数，并且还要吧server实例传入进去
server.use(session({}, server));
server.use(async (ctx) => {
    if (ctx.session['count']) {
        ctx.session['count']++;
    } else {
        ctx.session = 1;
    }
});
```

## 7. mysql

需要安装```mysql-pro```，```koa-mysql```版本太低，不支持```await```，所以被弃用了。

```js
const MySql = require('mysql-pro');
const db = new MySql({
    mysql: {
        host: '',
        port: 3306
        user: '',
        password: '',
        database: 'eat'
    }
})
server.use(async ctx => {
    const data = await db.query('select * from ')
})

```

## 8. 服务端渲染

需要安装 ```koa-ejs```和```koa-pug```。

```pug```

```js
const Pug = require('koa-pug');
const pug = new Pug(
    viewPath: path.resolve('template'),
    app: server // pug.use(server);
)

server.use(async ctx => {
    await ctx.render('模板名称', { a: 123});
})
```

```ejs```
```js
const ejs = require('koa-ejs');
ejs(server, {
    root: path.resolve('template'),
    layout: false,
    viewExt: 'ejs' // 扩展名
})
server.use(async ctx => {
    await ctx.render('modal_name', {name: 1, age: 18})
})
```

## 9. 服务结构

```js
const Koa = require('koa');
const Router = require('koa-router');
const static = require('koa-static-cache');
const body = require('koa-better-body');
const convert = require('koa-convert');
const Mysql = require('mysql-pro');
const session = require('koa-session');
const ejs = require('ejs');
const path = require('path');

const db = new Mysql({
    mysql: {
        host:'',
        port: '',
        user: '',
        password: '',
    }
});
db.execute = async (sql) => {
    let res = '';
    await db.startTransaction();
    if (typeof sql === 'string') {
        res = await db.executeransaction(sql);
    } else {
        sql.forEach(async item => {
            res = await db.executeransaction(item);
        })
    }
    await db.startTransaction();
    return res;
}

const server = new Koa();
server.listen(8080);
// 日志
server.use(async (ctx, next) => {
    fs.appendFile('logs/2018-1-1', `[2018-10-1] 地址 url \r\n`, async err => {
        await next();
    })
})
// 错误处理，捕获所有的异常
server.use(async (ctx, next) => {
    try {
        await next();
    } catch(e) {
        ctx.response.body = '服务器正在维护';
        console.log('出错了')
    }
})
server.use(convert(body({
    uploadDir: path.resolve('www/upload')
})))
server.keys = ['dadasdad','evreregfd'];
server.use(session({}, server))
ejs(server, {
    root: path.resolve('template'),
    layout: false,
    viewExt: 'ejs.html',
    cache: false,

})
const r1 = new Router();
server.use(async (ctx, next) => {
    ctx.db =db; // 将数据链接绑定到全局
    await next();
})
server.use(r1);
server.use(static(path.resolve('www')));
r1.get('/a', ctx => {
    const data = await ctx.db.execute('select * from lalala')
    await ctx.render('list', data)
})
```
