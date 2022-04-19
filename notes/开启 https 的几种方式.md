## 证书获取

```https```证书有```4```种，这里我们只介绍最简单并且免费的，域名证书。

### 1. 阿里云获取证书

如果你的域名是阿里云购买的那就简单了，登录阿里云在搜索栏搜索```ssl```证书，选中```ssl证书(应用安全)```控制台，打开证书列表页面。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/149defed903b4ad89fc07efc4ef903ef~tplv-k3u1fbpfcp-watermark.image)

在证书列表页面，点击购买证书。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/997cb9e6bacb4f83893a75f67951fc06~tplv-k3u1fbpfcp-watermark.image)

```2021```年起阿里云证书将以资源包的形式开放(说实话，更麻烦了)，需要点击证书资源包中进行下单。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c0a2d5f3126403e98dfb2ee7771e941~tplv-k3u1fbpfcp-watermark.image)

选择免费证书扩容包```20```个，支付金额为```0```元，下单就可以了。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba8c3fef89e74afb82a0aa6bf35b3df4~tplv-k3u1fbpfcp-watermark.image)

选择左侧证书资源包选项卡，然后在剩余证书数量中点击证书申请。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9902af04a289479499707a1b12078862~tplv-k3u1fbpfcp-watermark.image)

申请一个。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc62a793527341c2817d55f23d2a061f~tplv-k3u1fbpfcp-watermark.image)

在下面新增的这条信息中点击申请证书，会要求填写一些信息，比如网站域名，证书所有人。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c98e490fa8e4108bda758f08b484ae7~tplv-k3u1fbpfcp-watermark.image)

登记之后就会出现下载按钮，在弹出的弹框中选择需要对应的环境就可以了，一般我这里下载的证书会配置在```nginx```中，所以下载```nginx```的。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0451332bf6464c58a7e82b43b58099dc~tplv-k3u1fbpfcp-watermark.image)

这样我们下载下来的文件夹中就会包含```key```和```pem```两个文件。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a0b974b7e364782976176d14fe9cbd1~tplv-k3u1fbpfcp-watermark.image)

### 2. mkcert获取证书

通过本地服务自动生成。首先需要安装```mkcert```模块。

```s
npm install -g mkcert
```

通过下面代码生成证书的公钥和私钥，也就是```key```和```pem```。

```js
const mkcert = require('mkcert');
const fs = require('fs');

const start = async () => {
    const ca = await mkcert.createCA({
        organization: 'Hello CA',
        countryCode: 'NP',
        state: 'Bagmati',
        locality: 'Kathmandu',
        validityDays: 365
    });

    // then create a tls certificate
    const cert = await mkcert.createCert({
        domains: ['127.0.0.1', 'localhost'], // 域名地址
        validityDays: 365,
        caKey: ca.key,
        caCert: ca.cert
    });
    fs.writeFile(`${__dirname}/test.key`,cert.key, (error) => {
        console.log(error)
    });
    fs.writeFile(`${__dirname}/test.pem`,cert.cert, (error) => {
        console.log(error)
    });
}

start();
```

执行之后就可以在写入的文件位置```${__dirname}/test.key```找到生成的一对证书。

### 3. mkcert工具获取证书

```mkcert```工具，区别于```npm```工具，注意和```npm```的```mkcert```进行区分，同时安装两个工具会造成使用冲突。

首先使用```brew```安装```mkcert```工具，然后初始化安装```CA```的根证书。最后使用```mkcert```生成域名证书就可以了。同样会在本地生成```key```和```pem```两个文件。

```s
# 安装mkcert
brew install mkcert
# 安装根证书
mkcert ---install
# 生成本地签名，假设域名为zhiqianduan.com
mkcert zhiqianduan.com
```

## 证书使用

### 1. nginx 服务

将下载好的```key```和```pem```文件放在自己需要的位置，我一般是习惯在```nginx```的配置目录中新建一个```certs```的文件夹存放他们。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4352c8574da40508b980c3f309369c4~tplv-k3u1fbpfcp-watermark.image)

修改```nginx```配置文件

```s
vi /usr/local/nginx/conf/nginx.conf
```

```https```使用的是```443```端口号，默认情况下```nginx```是注释了这块代码区域的，记得放开，然后将```key```和```pem```分别配置在```ssl_certificate```和```ssl_certificate_key```中，注意后面的路径，我这里```nginx.conf```和```certs```文件夹在同一个目录，所以我使用```/certs/xxxx.key```找到文件。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54b3325664924d2298b3be4516d39863~tplv-k3u1fbpfcp-watermark.image)

重启```nginx```就可以使用https访问了。

```s
nginx -s reload
```

还可以将http转发为https

```s
server {
    listen 80;
    server_name zhiqianduan.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent; 
}
```

### 2. node

首先需要安装https模块。

```s
npm install https --save-dev
```

使用```https```创建服务，使用方式和```http```模块基本一致，不过需要传入```key```和```pem```，最后监听的端口是```443```。

```js
const https = require('https');
const fs = require('fs');
const app = https.createServer({
    key: fs.readFileSync('./xxxx.key'),
    cert: fs.readFileSync('./xxxx.pem')
}, (req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('https');
}).listen(443, '0.0.0.0');
```