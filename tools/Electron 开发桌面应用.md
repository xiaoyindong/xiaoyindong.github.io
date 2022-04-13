## 1. 概述

```Electron```核心组成分为三个部分，```Chromium```，```Node.js```，```Native apis```。这三部分共同构成了```Electron```架构。

```Chromium```可以理解为浏览器，用于支持``html``，```css```，```js```页面构成。```Native apis```用于支持和操作系统的一些交互操作。```Node.js```用于操作文件等。

```Electron```可以看成一个框架，将```chromium```及```Node.js```整合到一起，它允许通过```web```技术构建程序，可以通过```Native apis```对操作系统进行一切操作。

桌面应用就是运行在不同操作系统中的软件，软件中的功能实现都是通过```Native Api```与操作系统的通信。操作系统对于前段来说基本相当于黑盒，想实现功能的时候只需要调用```API```就可以了，无需关注内部实现。

```Electron```内部存在不同的进程，一个是主进程，一个是渲染进程，当启动```app```的时候首先会启动主进程，主进程完成之后就会创建```Native UI```, 会创建一个或者多个 ``window``，用```window```来呈现界面也就是```web```界面。每个```window```都可以看做一个渲染进程，各进程间相互独立，不同窗口数据可以通过```rpc```或者```ipc```通信，通信方法是封装好的，开发者直接使用就可以了。

```app```启动后主进程创建```window```窗口，然后```win```窗口会去加载界面也就是渲染进程，如果页面存在交互，需要将渲染进程中接收到的指令信息通过```IPC```通信给主进程，主进程收到信息之后通过调用操作系统实现功能，完成之后再通知给渲染进程。

主进程可以看做是```package.json```中```main```属性对应的文件。一个应用只能有一个主进程，只要主进程可以进行```GUI```的```API```操作。主进程可以管理所有的```web```界面和渲染进程。

渲染进程就是```windows```中展示的界面进程，一个应用可以有多个渲染进程，渲染进程不能直接访问原生```API```可以通过主进程访问。

## 2. 环境搭建

[官网](https://www.electronjs.org)

```s
# 克隆示例项目的仓库
$ git clone https://github.com/electron/electron-quick-start

# 进入这个仓库
$ cd electron-quick-start

# 安装依赖并运行
$ npm install && npm start
```

```main.js```是主进程。

```js
const { app, BrowserWindow } = require('electron')
const path = require('path')

// 创建窗口 并且加载页面 界面是运行在渲染进程的
function createWindow() {
    const mainWindow = new BrowserWindow({
        width: 800,
        height: 600,
        webPreferences: {
            preload: path.join(__dirname, 'preload.js')
        }
    })
    // 加载页面
    mainWindow.loadFile('index.html')

    mainWindow.on('close', () => {
        console.log('当前窗口关闭')
    })
}

// 生命周期
app.whenReady().then(() => {
    // 加载界面
    createWindow();

    app.on('activate', () => {
        console.log('激活的时候')
    })
})

app.on('window-all-closed', () => {
    console.log('所有窗口都关闭');
    // 关闭应用
    app.quit(); 
})
```

## 3. 生命周期

### 1. ready: app初始化完成

```js
app.on('ready', () => {
    const mainWin = new BrowserWindow({
        width: 800,
        height: 400
    })
})
```

### 2. dom-ready: 一个窗口中的文本文件加载完成

```webContents```用于控制当前窗口的内容，每个窗口都有一个```webContents```。

```js
mainWin.webContents.on('dom-ready', () => {
    
})
```

### 3. did-finsh-load: 导航完成时触发，dom-ready之后

```js
mainWin.webContents.on('did-finsh-load', () => {
    
})
```

### 4. window-all-closed: 所有窗口都关闭时触发，如果没有监听会直接退出

```js
app.on('window-all-closed', () => {

})
```

### 5. before-quit: 在关闭窗口之前触发

```js
app.on(' before-quit', () => {
    
})
```

### 6. will-quit: 在窗口关闭并且应用退出时触发

```js
app.on('will-quit', () => {
    
})
```

### 7. quit: 当所有窗口被关闭时触发

```js
app.on('quit', () => {
    
})
```

### 8. close: 窗口关闭

```js
mainWin.on('close', () => {
    mainWin = null;
})
```

自动打包设置

```json
"scripts": {
    "start": "nodemon --watch main.js --exec npm run build",
    "build": "electron ."
}
```

## 4. 窗口尺寸

```js
const mainWin = new BrowserWindow({
    ...
})

mainWin.loadFile('index.html')
// 页面加载完成异步展示 避免页面空白
mainWin.on('ready-to-show', () => {
    mainWin.show();
})

// x: number窗口位置
// y: number 窗口位置
// width: number 窗口宽度
// height:  number 窗口高度
// show: boolean 默认是否显示窗体
// maxHeight: number 最大高度
// minHeight: number 最小高度
// maxWidth: number 最大宽度
// minWidth: number 最小宽度
// resizable: boolean 缩放设置
// title: string 窗口title， 如果html中设置了title标签则取title标签中的内容
// icon: string 窗口icon
// frame: boolean 是否展示标签页菜单栏标题栏
// transparent: boolean 窗体是否透明
// autoHideMenuBar: boolean 隐藏菜单
// webPreferences: {
//     nodeIntegration: true， // 允许运行node环境
//     enableRemoteModule: true // 允许使用remote模块
// } 

```

## 5. 窗口通信

```ctrl + r```重载页面

```ctrl + shift + i```开启调试面板

渲染进程默认是无法使用```require```运行```node```的，可以通过开启```webPreferences.nodeIntegration```支持这个功能。开启之后可以使用```remote```找到主线程中的```BrowserWindow```，```BrowserWindow```只是给主线程使用的，不允许在渲染线程中调用。同时还需要开启```webPreferences.enableRemoteModule```功能。

```js
const { remote } = require('electron');

const mainWin = new remote.BrowserWindow({
    ...
})

mainWin.loadFile('')
```

```remote```模块就是主线程和子线程通信的模块。

```remote```可以获取当前窗口对象

```js
const mainWin = remote.getCurrentWindow;
// 关闭
mainWin.close();
// 最大化状态
mainWin.isMaximized()
// 最大化
mainWin.maximize();
// 回归初始状态
mainWin.restore();
// 最小化状态
mainWin.isMinimized()
// 最小化
mainWin.minmize();
```

窗口关闭提示

```js
window.onbeforeunload = function() {
    // 销毁 close会陷入死循环
    mainWin.destroy();
    // 禁止关闭
    return false;
}
```

```remote.BrowserWindow```创建的窗口默认不具备父子关系，如果需要可以使用parent参数。这样就具备父子关系。不过有无父子关系使用区别基本不大。

```js
remote.BrowserWindow({
    parent: remote.getCurrentWindow(),
    width: 200,
    height: 200
})
```

如果希望子窗口出现父窗口不可操作可使用模态窗口，```modal```属性设置为```true```就可以了。

```js
remote.BrowserWindow({
    parent: remote.getCurrentWindow(),
    width: 200,
    height: 200,
    modal: true
})
```

[文档中心](https://www.electronjs.org/docs)

## 6. 进程间通信

### 1. 渲染进程向主进程发送消息，主进程接收消息。

渲染进程发送消息

```js
const { ipcRenderer } = require('electron');

window.onload = function() {
    // 采用异步消息向主线程发送消息
    ipcRenderer.send('msg1', '参数字符串')
    // 采用同步方式发送消息
    ipcRenderer.sendSync('msg3', '同步消息')
}
```

主进程接收消息

```js
const { ipcMain } = require('electron');
ipcMain.on('msg1', (e, data) => {
    console.log(e, data);
})

ipcMain.on('msg3', (e, data) => {
    console.log(e, data);
})
```

### 2. 主进程发送消息给渲染进程

主进程发送消息

```js
const { ipcMain } = require('electron');
ipcMain.on('msg1', (e, data) => {
    console.log(e, data);
    // e.sender就是ipcMain
    e.sender.send('msg2', '参数字符串')
    // 主线程返回同步消息
    e.returnValue = '发送同步消息'
})
```

渲染进程接收消息

```js
const { ipcRenderer } = require('electron');

window.onload = function() {
    // 采用异步消息向主线程发送消息
    ipcRenderer.send('msg1', '参数字符串')
}

ipcRenderer.on('msg2', (ev, data) => {
    console.log(e, data);
})
```

webContents.openDevTools() 开启开发者工具

```js
mainWin.webContents.openDevTools()
```

## 7. dialog

```dialog```模块是```electron```内置模块，可以用于操作选择系统中的文件等。

## 8. shell

```shell```模块提供与桌面集成相关的功能。比如使用默认浏览器打开某地址。可以在所有线程中使用。

```js
const { shell } = require('electron')

shell.openExternal('https://github.com')
```

## 9. iframe

```electron```不建议使用```webview```，建议使用```iframe```替代，或者最好不要用。

## 10. 消息通知

可以借助```H5```的消息通知功能来完成。

```js
const notify = new window.Notification('标题', {
    title: '标题',
    body: '内容',
    icon: './icon.png'
})

notify.onclick = function() {
    console.log('点击了消息')
}
```

## 11. 注册快捷键

快捷点都是针对主进程的，快捷键需要在初始化完成之后绑定。销毁前清除。

```js
const { globakShortcut } = require('electron')
app.on('ready', () => {
    const ret = globakShortcut('ctrl + q', () => {
        console.log('触发了快捷键')
    })
    if (!ret) {
        console.log('注册失败')
    }
    // 是否注册
    console.log(globakShortcut.isRegistered('ctrl + q'))
})
```

快捷键取消需要在```will-quite```生命周期中来做。

```js
app.on('will-quit', () => {
    // 清除单个
    globakShortcut.unregister('ctrl + q')
    // 清除所有注册
    globakShortcut.unregisterAll()
})
```

## 12. 剪切板功能

剪切板也是不限制线程的模块，可以在任意线程中使用。

```js
const { clipboard } = require('electron')
clipboard.writeText('写入剪切板');
// 读取剪切板
clipboard.readText;
```

将图片copy到剪切板

```js
const { clipboard, nativeImage } = require('electron')
const image = nativeImage.createFromPath('./aaa.png')
clipboard.writeImage(image);

// 渲染到页面
img.src = image.toDataURL()
```