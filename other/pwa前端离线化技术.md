
## 概述

```webapp```用户体验差，不能离线访问，用户粘性低，```pwa```就是为了解决这一系列问题，让```webapp```具有快速，可靠，安全等特点。但是兼容性较差。

## PWA用到的技术

- web app manifest

图标添加

- service worker

缓存机制

- push api & notification api

消息通知

- app shell & app skeleton

## Web App Manifest 设置

安卓支持比较好

```html
<link rel="manifest" href="/manifest.js">
```

```manifest.js```

```json
{
    "name": "应用名称",
    "short_name": "桌面应用名称",
    "display": "standalone", // fullScreen(standalone) minimal-ui browser
    "start_url": "打开时的网址",
    "icons": [], // 设置桌面图片的icon图标，修改图标需要重新添加到桌面，[{src, sizes, type}]，[{"src": "", "sizes": "144x144", type: "image/png"}] 默认144 * 144
    "background_color": "#aaa", // 启动画面颜色
    "theme_color": "#aaa" // 状态栏颜色
}
```

```ios```需要使用```meta```来设置

```html
// 图标
<link rel="apple-touch-icon" href="apple-touch-iphone.png">
// 添加到主屏后的标题和short_name一致
<link rel="apple-mobile-web-app-title" content="标题">
// 隐藏safari地址栏 standalone模式下默认隐藏
<link rel="apple-mobile-web-app-capable" content="yes">
// 设置状态栏颜色
<link rel="apple-mobile-web-app-status-bar-style" content="black-translucent">
```

横幅安装: 用户在浏览器中访问至少两次，两次访问间隔至少时间为```5```分钟(```safari```不支持横幅)

## 运行环境

1、localhost

2、https

只支持以上两种环境

如果不安装```service Worker``` 则无法添加图标

## Service Worker

- 不能访问/操作dom

- 会自动休眠，不会随浏览器关闭所失效(必须手动卸载)

- 离线缓存内容开发者可控

- 必须在https或者localhost下使用

- 所有api都基于promise

生命周期:

installing安装阶段
installed安装完成阶段
activating激活中
activated激活完成
redundant废弃

```script```的代码。

```js
window.addEventListener('load', function() {
    // 解决离线缓存的问题 缓存 把缓存取出来
    if ('serviceWorker' in navigator) { // 判断当前浏览器是否支持serviceWorker
        navigator.serviceWorker.register('/sw.js').then(function(resgisteration) {
            console.log(resgisteration);
        })
    }
});
```

```sw.js```的代码

```js
// 无window对象，采用self代替

const CACHE_NAME = 'cache_v' + 1; // 默认情况，sw文件变化后，会重新注册serviceWorker
const CACHE_LIST = [
    '/',
    '/index.html',
    '/index.css',
    '/main.js',
    '/api/img'
]

function fetchAddSave(request) { // 数据获取后 进行缓存
    // 如果请求到了 需要更新缓存
    return fetch(request).then(res => { // res 是流文件
        // 更新缓存
        const r = res.clone(); // res必须克隆，因为使用一次就销毁
        caches.open(CACHE_NAME).then(cache => {
            chache.put(request, res);
        })
        return res;
    });
}

// 监听fetch，只要拦截到请求就会走这里
self.addEventListener('fetch', function(e) { // 线程中不能发送ajax，可以使用fetch
    // 如果联网 发送请求，如未联网，
    if (e.request.url.includes('/api/')) { // 请求的是接口
        return e.respondWith(
            fetchAddSave(e.request).catch(err => {
                return caches.open(CACHE_NAME).then(cache => chche.match(e.request))
            })
        )
    }
    // 缓存策略 缓存优先，网络优先
    console.log(e.request.url);
    e.respondWith( // 用什么内容 返回当前请求
        fetch(e.request).catch(err=> {
            // 打开缓存，把缓存中匹配的结果返回回去
            return caches.open(CACHE_NAME).then(cache => chche.match(e.request))
        })
    )
});
// 缓存，需要缓存内容
function preCache() { // 需要返回一个promise
    // 开启一个缓存空间
    return caches.open(CACHE_NAME).then(function(cache) {
        return cache.addAll(CACHE_LIST);
    })
}
self.addEventListener('install', function(e) {
    // 如果上一个serviceWorker不销毁，需要树洞skipWating
    console.log('install');
    e.waitUntil(
        preCache().then(skipWaiting)
    ) // 等待promise执行那个完成
});
// 激活当前serviceWorker，让service立即生效 self.clients.claim
function clearCache() {
    return caches.keys().then(function(keys) {
        return Promise.all(keys.map(function(key) {
            if (key !== CACHE_NAME) {
                return caches.delete(key);
            }
        }));
    })
}
// 当前serviceWorker安装完毕之后，删除之前的缓存
self.addEventListener('activate', function(e) {
    console.log('activate');
    e.waitUntil(
        Promise.all([
            clearCache(),
            self.clients.claim()
        ])
    )
})

```

服务器代码:

```js
const http = require('http');
const fs = require('fs');
const URL = require('url');

http.createServer((req, res) => {
    let { pathname } = URL.parse(req.url, true);
    const mime = {
        js: 'text/javascript',
        html: 'text/html',
        ico: 'image/ico'
    }
    console.log(pathname);
    try {
        fs.readFile(`.${pathname}`, (err, data) => {
            if (err) {
                return res.end();
            }
            res.writeHead(200, {'Content-type' : mime[pathname.split('.')[1]]});
            res.write(data);
            return res.end();
        })
    } catch (error) {
        return res.end();
    }
}).listen(8080);
```

```serviceWorker```需要拦截我们的客户请求，如果可以通畅，正常请求，如果网络不通，则走缓存
添加主屏幕，两次访问，间隔```5```分钟 会弹出横条，手动点击是没有问题的

## 缓存策略

- cechefirst

缓存优先

- cacheonly

仅缓存

- networkfirst

网络优先

- networkonly

仅网络

- StateWhileRevalidate

从缓存取，用网络数据更新缓存

## pwa的库

- workbox
