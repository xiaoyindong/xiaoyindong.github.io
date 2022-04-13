## 1. 概述

```nwjs```是基于```Chromium```和```Node.js```运行的, 我们可以通过```html```+```css```来编写```UI```页面，使用js来实现功能。可以直接调用```Node.js```的各种```api```以及现有的第三方包。

由于```Chromium```和```Node.js```的跨平台，那么你的应用也是可以跨平台的。

现在已经有很多知名的应用是基于NW.js实现，这是官方统计的一些列表:```https://github.com/nwjs/nw.js/wiki/List-of-apps-and-companies-using-nw.js```。

前端领域比较熟悉的微信开发者工具就是基于```nwjs```开发的。

NW文档中心: ```https://nwjs.org.cn/```

## 2. 开始使用

你可以从互联网直接下载```nw```的各个版本```https://nwjs.org.cn/download.html```也可以使用```npm```安装。

将下载好的压缩包解压，你可以将解压后的文件夹修改一个名字，也可以新建一个文件夹将解压后的文件```copy```进去。其实主要有用的文件就是```nw```执行程序，只要这个就可以了。

然后在```nw```执行程序的同级目录新建一个```package.json```文件。在里面设置应用的名称和启动执行的```html```页面。然后我们在文件夹中新建一个```index.html```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7846cacda68647f7b9e8898978ab2aa3~tplv-k3u1fbpfcp-watermark.image)

```package.json```中写上应用名称和启动页面。

```json
{
  "name": "我的应用", // 名称
  "main": "index.html", // 启动页面
}
```

在```html```文件中随便写点什么。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的应用</title>
</head>
<body>
    我的应用程序
</body>
</html>
```

双击```nwjs```执行这个程序，页面长成这么个样子。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8dd384d5256d46bf8deefd07e62741ad~tplv-k3u1fbpfcp-watermark.image)

## 3. 应用配置

接着我们介绍一下```package.json```中都能配置哪些参数

```json
{
  "name": "应用的名称", // 应用的名称
  "main": "./index.html", // 指定应用的主页面
  "build": "1445048139741", // 这是为了给更新时判断版本用的时间戳
  "version": "0.0.1",// 当前的版本号
  "homepage": "一般是官网地址之类的",
  "description": "应用的描述", // 描述
  "window": {
    "title": "应用打开时候显示的名称", // 如果 index.html没有title,则会显示这里的值
    "icon": "assest/img/logo.png", // icon
    "position": "center", // 打开应用时在浏览器屏幕中的位置
    "width": 1280, // 应用的宽度
    "height": 680, // 应用的高度
    "toolbar": true, // 是否隐藏窗口的浏览器工具栏，nw老版本还有用，新版本已经无效了
    "frame": true, // 是否显示最外层的框架，设为false之后 窗口的最小化、最大化、关闭 就没有了
    "resizable": true, // 可以通过拖拽变换应用界面大小
    "min_width": 1028 // 最小宽度
  },
  "node-main": "./node-main.js",// 启动时执行的js，检查更新之类的。
}
```

其实这里的```package.jso```n就是我们日常开发项目中的项目管理文件，我们可以通过```npm```或者```yarn```的方式安装第三方的依赖包，然后通过```require```的方式导入进来，这里也是可以使用的。

比如我们通过```npm```安装一个```marked```模块。

```js
npm install marked --save-dev
```

```json
{
  "name": "我的应用",
  "main": "index.html",
  "devDependencies": {
    "marked": "^1.2.5"
  }
}
```

然后在```html```中的```script```中通过```require```加载这个模块，并且使用这个模块在页面转义一句```markedown```。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的应用</title>
</head>
<body>
    <div id="content"></div>
    <script>
        const marked = require('marked'); // 加载marked模块
        document.getElementById('content').innerHTML = marked('# Marked in the browser\n\nRendered by **marked**.');
    </script>
</body>
</html>
```

效果如下: 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce2c413db6ba4e19ae41b9b8d58ccd86~tplv-k3u1fbpfcp-watermark.image)

## 4. 开发调试

新版的```nw```是没有地址栏的，同时也没有控制台，这在开发过程中会产生很大的不便，不过这也很好解决，可以使用腾讯的[vconsole](https://github.com/Tencent/vConsole/blob/dev/README_CN.md)来调试代码。

引入```vconsole.js```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的应用</title>
</head>
<body>
    <script src="https://cdn.bootcss.com/vConsole/3.3.4/vconsole.min.js"></script>
    <script>
        var vConsole = new VConsole();
        console.log('Hello world');
    </script>
</body>
</html>
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8260426b1f2a4c2291b846dc9a3596ed~tplv-k3u1fbpfcp-watermark.image)

## 5. 软件打包

如果是```mac```系统，打包就比较简单了，将```package，html``` 等资源所在的文件夹 压缩成```zip```，注意```nwjs```执行程序不要压缩否则包体积太大，然后修改文件名为```app.nw```，注意这里的后缀名改为```nw```。

然后在```nwjs```文件上右键```显示包内容``` -> ```Contents``` -> ```Resources```, 将上面的```app.nw```文件放进来，同时这个文件夹里面有一个```app.icns```就是我们软件的显示图标，可以自己制作一个替换掉。这样我们的软件就只做完了，可以发给其他人使用了。


```windows```系统打包方式相对来说麻烦一些，首先也是将```package，html``` 等资源所在的文件夹 压缩成```zip```，修改后缀名为```app.nw```，然后需要将```nw.exe```和```app.nw```合并成一个文件夹。可以使用下面命令操作。

```s
copy /b nw.exe+app.nw app.exe
```

然后可以下载```Enigma Virtual Box```将文件打成一个软件包。更换软件展示```icon```可以使用借助```zip```的能力。

```程序``` -> ```添加到压缩文件``` -> ```zip```, ```存储（方式）``` , ```创建自解压``` -> ```高级``` -> ```自解压选项``` ->
```设置``` -> ```提取后运行(添加文件名.exe)``` -> ```模式(解压到临时文件夹, 全部隐藏)``` ->
```更新(覆盖所有文件)``` -> ```文本和图标(从文件加载自解压文件图标)``` -> ```确定```。

至此打包好的应用就可以发送给用户使用了。
