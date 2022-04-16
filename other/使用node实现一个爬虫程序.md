## 1. 概述

爬虫是一种自动获取网页内容的程序，是搜索引擎的重要组成部分，搜索引擎优化很大程度上是针对爬虫而做出的优化。网站中存在一个叫做```robots.txt```的文本文件，```robots.txt```是一个协议，告诉爬虫要查看的第一个文件，```robots.txt```文件告诉爬虫服务器上什么文件是可以被查看的，搜索机器人会按照该文件中的内容确定访问的范围。比如百度的```robots```文件```https://www.baidu.com/robots.txt```。

```txt
User-agent: *
Disallow: /baidu
Disallow: /s?
Disallow: /ulink?
Disallow: /link?
Disallow: /home/news/data/
Disallow: /bh
```

```robots.txt```文件必须放在网站的跟目录下，当搜索机器人访问一个站点时，会首先检查该站点根目录下是否存在```robots.txt```，如果存在，搜索机器人就会按照该文件中的内容来确定访问的范围；如果该文件不存在，那么搜索机器人就沿着链接抓取。

```User-agent```表示针对哪种搜索引擎，```*```表示全部，还可以设置```Baiduspider```，```Googlebot```，```MSNBot```，```YoudaoBot```等。```Disallow```表示禁止爬虫抓取的文件。如设置为```/```则表示全部都不允许抓取。```Allow```是允许抓取的位置，如不设置则默认为```Allow: /```全部允许。

## 2. 爬虫工具

```node```中可以使用```cheerio```模块进行页面抓取，它可以像```jq```一样操作```dom```。通过```https```发送请求。在```finish```方法中就获取到页面的```html```了，然后使用```cheerio.load```方法对```dom```节点进行解析。

```js
const cheerio = require('cheerio');
const https = require('https');

let html = '';

const $ = '';

https.get('url', res => {
    res.on('data', data => {
        html += data; // 保存返回的数据
    });
    res.on('finish', () => {
        $ = cheerio.load(html); // cheerio解析数据
        // $就是拿到的dom树, 想jq一样。
    })
})
```

## 3. 手动编写一个爬虫

```js
const url = require('url');
const http = require('http');
const https = require('https');


function requestUrl(url, headers) {
    const obj = url.parse(url);
    let httpMode = null;
    if (obj.protool == 'https') {
            httpMode = https;
        } else {
            httpMode = http;
        }
    return new Promise((resolve, reject) => {
        const req = httpMode.request({
            host: obj.host,
            path: obj.path,
            headers,
        }, (res) => {
            if (res.statusCode >= 200 && res.statusCode < 300 || res.statusCode === 304) {
                let arr = [];
                res.on('data', data => {
                    arr.push(data);
                })
                res.on('end', () => {
                    const buffer = Buffer.concat(data);
                    fs.writeFile(path.resolve('tem', '1.html'), buffer, err => {
                        if (err) {
                            console.log('出错');
                        } else {
                            console.log('成功');
                            resolve({
                                code: 200,
                                body: buffer,
                                headers: res.headers
                            })
                        }
                    })
                })
            } else if (res.statusCode === 301 || res.statusCode === 302) {
                console.log(res.headers);
                resolve({
                    code: 200,
                    body: null,
                    headers: res.headers
                })
            } else {
                console.log('出错');
                reject({
                    code: res.statusCode,
                    body: null,
                    headers: res.headers
                })
            }
        })
        req.on('error', err => {
            console.loh(err)
        })
        req.write(''); // 发送post数据
        req.end(); // 开始请求
    })
}

(async () => {
    try {
        const data = await requestUrl('http://www.baidu.com');
        console.log(data);
        if (data.code == 200) {
            return;
        } else if (data.code === 301 || data.code === 302) {
            const value = await requestUrl(data.headers.location);
        }
    } catch(e) {
        console.log('失败');
    }
    
})

```

百度首页比较简单，期间会进行多次重定向，将代码优化一下。

```js
const url = require('url');
const http = require('http');
const https = require('https');
const assert = require('assert');

function requestUrl(url, headers) {
    const obj = url.parse(url);
    let httpMode = null;
    if (obj.protool == 'https') {
            httpMode = https;
        } else {
            httpMode = http;
        }
    return new Promise((resolve, reject) => {
        const req = httpMode.request({
            host: obj.host,
            path: obj.path,
            headers,
        }, (res) => {
            if (res.statusCode >= 200 && res.statusCode < 300 || res.statusCode === 304) {
                let arr = [];
                res.on('data', data => {
                    arr.push(data);
                })
                res.on('end', () => {
                    const buffer = Buffer.concat(data);
                    resolve({
                        code: 200,
                        body: buffer,
                        headers: res.headers
                    })
                })
            } else if (res.statusCode === 301 || res.statusCode === 302) {
                console.log(res.headers);
                resolve({
                    code: 200,
                    body: null,
                    headers: res.headers
                })
            } else {
                console.log('出错');
                reject({
                    code: res.statusCode,
                    body: null,
                    headers: res.headers
                })
            }
        })
        req.on('error', err => {
            console.loh(err)
        })
        req.write(''); // 发送post数据
        req.end(); // 开始请求
    })
}

async function request(url) => {
    try {
        while(1) {
            const { code, body, headers } = await requestUrl(url);
            if (code === 200) {
                return { body, headers};
            } else {
                assert(code === 301 || code === 302);
                assert(headers.location);
                url = headers.location;
            }
        }
    } catch(e) {
        console.log('失败', e);
    }
}

(async () => {
    const { body, headers } = await request('http://tmall.com/');
    fs.writeFile('tem/tmall.html', body, err => {
        if (err) {
            console.log(err);
        } else {
            console.log('成功', headers)；
        }
    })
})()

```

### 1. jsdom

用于解析```dom```节点。使用很简单，安装之后直接实例化就可以了。

```s
npm install jsdom -g
```

```js
const JSDOM = require('jsdom').JSDOM;

fs.readFile('tem/index.html', (err, buffer) => {
    if (!err) {
        const html = buffer.toString();
        const jsdom = new JSDOM(html);
        const document = jsdom.window.document;
        const $ = document.querySelectorAll.bind(document);
        const oText = $('input.text')[0].value;
    }
})

```
