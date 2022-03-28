模块联邦(```Module Federation```)是```Webpack 5```中新增的一项功能，可以实现跨应用共享模块。比如有```AB```两个应用，```A```应用中提供```fromA```方法，```B```应用提供了```fromB```方法，现在需要在```A```应用中调用```B```应用的方法，```B```应用中调用```A```应用的方法。这就涉及到了跨应用调用方法，就需要用到模块联邦啦。通过这样的方式就可以实现微前端啦。

这里通过一个案例进行演示，创建三个应用，一个容器应用，两个微应用，在容器应用中使用这两个微应用，项目结构如下。三个应用的结构基本相同。

```s
products
    ├── package-lock.json
    ├── package.json
    ├── public
    │   └── index.html
    ├── src
    │   └── index.js
    └── webpack.config.js
```

## 2. 案例演示

```json
{
  "name": "container",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "webpack serve"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "faker": "^5.2.0",
    "html-webpack-plugin": "^4.5.1",
    "webpack": "^5.19.0",
    "webpack-cli": "^4.4.0",
    "webpack-dev-server": "^3.11.2"
  }
}
```

在入口```JavaScript```文件中加入产品列表也就是```src/index.js```文件。首先引入```faker```，他是用来随机生成数据的，不用自己造数据了，然后将生成的数据添加到页面中。

```js
import faker from "faker"
let products = ""
for (let i = 1; i <= 5; i++) {
  products += `<div>${faker.commerce.productName()}</div>`
}
document.querySelector("#dev-products").innerHTML = products
```

在入口```html```文件中加入渲染的```div```。

```html
<div id="dev-products"></div>
```

webpack 配置。

```js
const HtmlWebpackPlugin = require("html-webpack-plugin")
module.exports = {
  mode: "development",
  devServer: {
    port: 8081 
	},
	plugins: [
			new HtmlWebpackPlugin({
				template: "./public/index.html"
			})
	] 
}
```

应用启动之后会运行在```8081```端口。

## 3. 使用模块联邦

要使用模块联邦首先需要导入```ModuleFederationPlugin```模块，他是```webpack```内置的模块，不需要额外安装。

可以通过new实例化模块联邦插件，可以传入一个配置项。```filename```模块文件名称，```name```是模块名称，```exposes```指定导出的文件。

```js
// webpack.config.js
// 导入模块联邦插件
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin")
// 将 products 自身当做模块暴露出去 
new ModuleFederationPlugin({
	// 模块文件名称, 其他应用引入当前模块时需要加载的文件的名字 
	filename: "remoteEntry.js",
	// 模块名称, 具有唯一性, 相当于 single-spa 中的组织名称 
	name: "products",
	// 当前模块具体导出的内容 
	exposes: {
		"./index": "./src/index"
	}
})
```

在容器应用的中导入产品列表微应用，同样实例化```ModuleFederationPlugin```传入参数。```remotes```是要引入的模块。@前面是模块```name```，后面是```remoteEntry.js```就是```filename```。

```js
// webpack.config.js
// 导入模块联邦插件
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin")
new ModuleFederationPlugin({ 
	name: "container",
	// 配置导入模块映射
	remotes: {
		// 字符串 "products" 和被导入模块的 name 属性值对应
		// 属性 products 是映射别名, 是在当前应用中导入该模块时使用的名字 
		products: "products@http://localhost:8081/remoteEntry.js"
	}
})
```

配置之后就可以加载微应用了，通过```import```加载。```products```就是```remotes```中的```remotes```，```index```就是微应用```exposes```中的```index```。

```js
// src/index.js
// 因为是从另一个应用中加载模块, 要发送请求所以使用异步加载方式 
import("products/index").then(products => console.log(products))
```

通过上面这种方式加载在写法上多了一层回调函数，一般都会在```src```文件夹中建立```bootstrap.js```，在形式上将写法变为同步。

```js
// src/index.js
import('./bootstrap.js')
```

```js
// src/bootstrap.js
import "products/index"
```

## 4. 打包分析

接着看下```Container```应用和```products```应用webpack打包流程。在```products```应用中只有一个文件```index.js```，```webpack```打包的时候会走两个流程正常流程和模块联邦流程。

正常的打包流程会生成```main.js```文件，也就是最终的打包文件，这是正常运行```webpack```打包应用生成的文件。模块联邦插件会把设置的```remoteEntry.js```打包成单独的文件，在这个文件中包含文件需要加载的列表和如何加载模块的代码。```exposes```是模块要导出的列表，模块联邦会把列表中导出的插件打包成单独的文件，比如说导出了```index```就会把```index```对应的文件打包成单独的文件，文件名称就是```src_index.js```，也就是文件路径。

在模块联邦中也用到了```faker```插件，所以也会把```faker.js```单独打包成一个文件。这就是```products```的打包流程。

```container```应用打包包含两个文件，```index.js```和```bootstrap.js```，在```index.js```中通过```import```导入了```bootstrap.js```，在```bootstrap.js```又导入了```products```模块，```webpack```对应用打包的时候最终会生成两个文件，```main.js```和```bootstrap.js```。```main.js```就是包含的```index.js```中的文件内容。```products```文件不会打包，会通过异步方式引入进来。

```Container```应用在执行的时候首先会加载```main.js```并执行，因为里面包含了```bootstrap.js```所以也会加载```bootstrap.js```，但是只是加载，并没有执行，```bootstrap.js```又导入了```products```同样进行加载```remoteEntry.js```和```faker.js```。加载完成之后就可以执行```bootstrap.js```文件了。

## 5. 共享模块

在```Products```和```Cart```中都需要```Faker```，当```Container```加载了这两个模块后，```Faker```被加载了两次。

```js
// 分别在 Products 和 Cart 的 webpack 配置文件中的模块联邦插件中添加以下代码 
{
  shared: ["faker"] // 共享模块的名字，可以写多个。
}
// 重新启动 Container、Products、Cart
```

共享模块需要异步加载，在```Products```和```Cart```中需要添加```bootstrap.js```。

## 6. 解决冲突

```Cart```中如果使用```4.1.0```版本的```faker```，```Products```中使用```5.2.0```版本的```faker```，通过查看网络控制面板可以发现```faker```又会被加载了两次，模块共享失败。

解决办法是分别在```Products```和```Cart```中的```webpack```配置中加入如下代码。

```json
shared: {
  faker: {
    singleton: true // 如果版本不一致使用高版本
  }
}
```

但同时会在原本使用低版本的共享模块应用的控制台中给予警告提示。

## 7. 子应用挂载接口

在容器应用导入微应用后，应该有权限决定微应用的挂载位置，而不是微应用在代码运行时直接进行挂载。所以每个微应用都应该导出一个挂载方法供容器应用调用。

```js
// Products/bootstrap.js
import faker from "faker"

function mount(el) {
  let products = "";
  for (let i = 1; i <= 5; i++) {
    products += `<div>${faker.commerce.productName()}</div>`;
  }
  el.innerHTML = products;
}
// 此处代码是 products 应用在本地开发环境下执行的 
if (process.env.NODE_ENV === "development") {
  const el = document.querySelector("#dev-products");
	// 当容器应用在本地开发环境下执行时也可以进入到以上这个判断, 容器应用在执行当前代码时肯定是获取不到dev-products 元素的, 所以此处还需要对 el 进行判断.
  if (el) {
		mount(el);
	}
}

export { mount }
```

```js
// Products/webpack.config.js
exposes: {
	// ./src/index => ./src/bootstrap 为什么 ?
	// mount 方法是在 bootstrap.js 文件中导出的, 所以此处要导出 bootstrap
	// 此处的导出是给容器应用使用的, 和当前应用的执行没有关系, 当前应用在执行时依然先执行 index 
	"./index": "./src/bootstrap"
}
```

```js
// Container/bootstrap.js
import { mount as mountProducts } from "products/index"
mountProducts(document.querySelector("#my-products"))
```

## 1. 概述

当前案例中包含三个微应用，分别为```Marketing```、```Authentication```和```Dashboard```。

Marketing是营销微应用，包含首页组件和价格组件。```Authentication```是身份验证微应用，包含登录组件。```Dashboard```是仪表盘微应用，包含仪表盘组件。

容器应用、营销应用、身份验证应用使用```React```框架，仪表盘应用使用```Vue```框架。

## 2. Marketing应用

```json
{
  "name": "marketing",
  "version": "1.0.0",
  "scripts": {
    "start": "webpack serve"
  },
  "dependencies": {
    "@material-ui/core": "^4.11.0",
    "@material-ui/icons": "^4.9.1",
    "react": "^17.0.1",
    "react-dom": "^17.0.1",
    "react-router-dom": "^5.2.0"
  },
  "devDependencies": {
    "@babel/core": "^7.12.3",
    "@babel/plugin-transform-runtime": "^7.12.1",
    "@babel/preset-env": "^7.12.1",
    "@babel/preset-react": "^7.12.1",
    "babel-loader": "^8.1.0",
    "clean-webpack-plugin": "^3.0.0",
    "css-loader": "^5.0.0",
    "html-webpack-plugin": "^4.5.0",
    "style-loader": "^2.0.0",
    "webpack": "^5.4.0",
    "webpack-cli": "^4.1.0",
    "webpack-dev-server": "^3.11.0",
    "webpack-merge": "^5.2.0"
  }
}
```

### 1. 创建应用结构

```s
├── public
│   └── index.html
├── src
│   ├── bootstrap.js
│   └── index.js
├── package-lock.json
├── package.json
└── webpack.config.js
```

```html
<!-- index.html -->
<title>Marketing</title>
<div id="dev-marketing"></div>
```

```js
// index.js
import("./bootstrap")
```

```js
// bootstrap.js
console.log('Hello')
```

### 2. 配置 webpack。

```js
const HtmlWebpackPlugin = require("html-webpack-plugin")
module.exports = {
  mode: "development",
  devServer: {
		port: 8081,
		// 当使用 HTML5 History API 时, 所有的 404 请求都会响应 index.html 文件 
		historyApiFallback: true
	}, 
	module: {
		rules: [ 
			{
						test: /\.js$/,
						exclude: /node_modules/,
						use: {
							loader: "babel-loader",
							options: {
								presets: ["@babel/preset-react", "@babel/preset-env"], 
								// 1. 避免 babel 转义语法后 helper 函数重复
								// 2. 避免 babel polyfill 将 API 添加到全局
								plugins: ["@babel/plugin-transform-runtime"]
							} 
						}
			} 
		]
	}, 
	plugins: [
			new HtmlWebpackPlugin({
				template: "./public/index.html"
			}) 
	]
}
```

### 3. 添加启动命令。

```json
"scripts": {
  "start": "webpack serve"
}
```

### 4. 创建挂载方法

```js
// bootstrap.js
import React from "react"
import ReactDOM from "react-dom"

function mount(el) {
  ReactDOM.render(<div>Marketing works</div>, el)
}

if (process.env.NODE_ENV === "development") {
	const el = document.querySelector("#dev-marketing")
  if (el) {
		mount(el)
	}
}
```
 
### 5. 创建路由

在```src```文件夹中创建```components```文件夹用于放置页面组件，在```src```文件夹中创建```App```组件，用于编写路由。

```js
// App.js
import React from "react"
import { BrowserRouter, Route, Switch } from "react-router-dom"
import Landing from "./components/Landing"
import Pricing from "./components/Pricing"
export default function App() {
  return (
    <BrowserRouter>
      <Switch>
        <Route path="/pricing" component={Pricing} />
        <Route path="/" component={Landing} />
      </Switch>
    </BrowserRouter>
	) 
}
```

```js
// bootstrap.js
import App from "./App"
function mount(el) {
  ReactDOM.render(<App />, el)
}
```

## 3. Container应用

```json
{
  "name": "container",
  "version": "1.0.0",
  "scripts": {
    "start": "webpack serve"
  },
  "dependencies": {
    "@material-ui/core": "^4.11.0",
    "@material-ui/icons": "^4.9.1",
    "react": "^17.0.1",
    "react-dom": "^17.0.1",
    "react-router-dom": "^5.2.0"
  },
  "devDependencies": {
    "@babel/core": "^7.12.3",
    "@babel/plugin-transform-runtime": "^7.12.1",
    "@babel/preset-env": "^7.12.1",
    "@babel/preset-react": "^7.12.1",
    "babel-loader": "^8.1.0",
    "clean-webpack-plugin": "^3.0.0",
    "css-loader": "^5.0.0",
    "html-webpack-plugin": "^4.5.0",
    "style-loader": "^2.0.0",
    "webpack": "^5.4.0",
    "webpack-cli": "^4.1.0",
    "webpack-dev-server": "^3.11.0",
    "webpack-merge": "^5.2.0"
  }
}
```

### 1. 创建应用结构 (基于 Marketing 应用进行拷贝修改)。

```s
├── public
│   └── index.html
├── src
│   ├── bootstrap.js
│   └── index.js
├── package-lock.json
├── package.json
└── webpack.config.js
```

### 2. 修改 index.html

```html
<title>Container</title>
<div id="root"></div>
```

### 3. 修改 App.js

```js
import React from "react"
export default function App() {
  return <div>Container works</div>
}
```
   
### 4. 修改 bootstrap.js

```js
if (process.env.NODE_ENV === "development") {
  const el = document.querySelector("#root")
  if (el) mount(el)
}
```

### 5. 修改 webpack.config.js

```js
module.exports = {
  devServer: {
		port: 8080 
	}
}
```

### 6. Container 应用加载 Marketing

```Marketing```应用配置```ModuleFederation```。

```js
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin")

new ModuleFederationPlugin({
  name: "marketing",
  filename: "remoteEntry.js",
  exposes: {
    "./MarketingApp": "./src/bootstrap"
  }
})
```

```Container```应用配置```ModuleFederation```。

```js
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin")

new ModuleFederationPlugin({
  name: "container",
  remotes: {
    marketing: "marketing@http://localhost:8081/remoteEntry.js"
  }
})
```

在```Container```应用中新建```MarketingApp```组件，用于挂载```Marketing```应用。

```js
// Container/components/MarketingApp.js
import React, { useRef, useEffect } from "react"
import { mount } from "marketing/MarketingApp"
export default function MarketingApp() {
  const ref = useRef()
  useEffect(() => {
    mount(ref.current)
  }, [])
  return <div ref={ref}></div>
}
```

在```Container```应用中的```App```组件中渲染```Marketing```应用的。

```js
// Container/App.js
import React from "react"
import MarketingApp from "./components/MarketingApp"
export default function App() {
  return <MarketingApp />
}
```

## 4. 共享库设置

在```Container```应用和```Marketing```应用中都使用了大量的相同的代码库，如果不做共享处理，则应用中相
同的共享库会被加载两次。

```json
"dependencies": {
  "@material-ui/core": "^4.11.0",
  "@material-ui/icons": "^4.9.1",
  "react": "^17.0.1",
  "react-dom": "^17.0.1",
  "react-router-dom": "^5.2.0"
}
```

在```Container```应用和```Marketing```应用的```webpack```配置文件中加入以下代码。

```js
const packageJson = require("./package.json")

new ModuleFederationPlugin({
  shared: packageJson.dependencies
})
```

## 5. 路由配置

容器应用路由用于匹配微应用，微应用路由用于匹配组件。容器应用使用```BrowserHistory```路由，微应用使用```MemoryHistory```路由。

为防止容器应用和微应用同时操作```url```而产生冲突，在微前端架构中，只允许容器应用更新```url```， 微应用不允许更新```url```，```MemoryHistory```是基于内存的路由，不会改变浏览器地址栏中的```url```。如果不同的应用程序需要传达有关路由的相关信息，应该尽可能的使用通过的方式，```memoryHistory```在```React```和```Vue```中都有提供。

### 1. 更新现有路由配置

```Container```容器应用的路由配置。

```js
// Container/App.js
import { Router, Route, Switch } from "react-router-dom"
import { createBrowserHistory } from "history"
const history = createBrowserHistory()
export default function App() {
  return (
    <Router history={history}>
      <Switch>
        <Route path="/">
          <MarketingApp />
        </Route>
      </Switch>
		</Router>
	) 
}
```

```Marketing```应用的路由配置。

```js
// Marketing/bootstrap.js
import { createMemoryHistory } from "history"
function mount(el) {
  const history = createMemoryHistory()
  ReactDOM.render(<App history={history} />, el)
}
```

```js
// Marketing/app.js
import { Router, Route, Switch } from "react-router-dom"
export default function App({ history }) {
  return (
    <Router history={history}>
      <Switch>
        <Route path="/pricing" component={Pricing} />
        <Route path="/" component={Landing} />
      </Switch>
		</Router>
	) 
}
```
 
添加头部组件。

```js
import Header from "./components/Header"
export default function App() {
  return <Header />
}
```

### 2. 微应用和容器应用路由沟通

微应用路由变化时```url```地址没有被同步到浏览器的地址栏中，路由变化也没有被同步到浏览器的历史记录中。当微应用路由发生变化时通知容器应用更新路由信息 (容器应用向微应用传递方法)。

```js
// Container/components/MarketingApp.js
import { useHistory } from "react-router-dom"
const history = useHistory()
mount(ref.current, {
  onNavigate({ pathname: nextPathname }) {
    const { pathname } = history.location
    if (pathname !== nextPathname) {
      history.push(nextPathname)
    }
	} 
})
```

```js
// Marketing/bootstrap.js
function mount(el, { onNavigate }) {
  if (onNavigate) {
		history.listen(onNavigate)
	}
}
```

同样的，容器应用路由发生变化时只能匹配到微应用，微应用路由并不会响应容器应用路由的变化。 当容器应用路由发生变化时需要通知微应用路由进行响应 (微应用向容器应用传递方法)

```js
// Marketing/bootstrap.js
function mount(el, { onNavigate }) {
  return {
    onParentNavigate({ pathname: nextPathname }) {
      const { pathname } = history.location
      if (pathname !== nextPathname) {
        history.push(nextPathname)
      }
		}
	}
}
```

```js
// Container/components/MarketingApp.js
const { onParentNavigate } = mount()
if (onParentNavigate) {
  history.listen(onParentNavigate)
}
```

### 5. Marketing 应用本地路由设置

目前```Marketing```应用本地开发环境是报错的，原因是本地开发环境在调用```mount```方法时没有传递第二个参数，默认值就是```undefined```,```mount```方法内部试图从```undefined```中解构```onNavigate```所以就报错了。解决办法是在本地开发环境调用```mount```方法时传递一个空对象。

```js
if (process.env.NODE_ENV === "development") {
  if (el) mount(el, {})
}
```

如果当前为本地开发环境，路由依然使用```BrowserHistory```，所以在调用```mount```方法时传递```defaultHistory```做区分。

```js
// Marketing/bootstrap.js
if (process.env.NODE_ENV === "development") {
  if (el) mount(el, { defaultHistory: createBrowserHistory() })
}
```

在```mount```方法内部判断```defaultHistory```是否存在，如果存在就用```defaultHistory```，否则就用```MemoryHistory```。

```js
// Marketing/bootstrap.js
function mount(el, { onNavigate, defaultHistory }) {
  const history = defaultHistory || createMemoryHistory()
}
```

## 6. Authentication 应用

```json
{
  "name": "auth",
  "version": "1.0.0",
  "scripts": {
    "start": "webpack serve"
  },
  "dependencies": {
    "@material-ui/core": "^4.11.0",
    "@material-ui/icons": "^4.9.1",
    "react": "^17.0.1",
    "react-dom": "^17.0.1",
    "react-router-dom": "^5.2.0"
  },
  "devDependencies": {
    "@babel/core": "^7.12.3",
    "@babel/plugin-transform-runtime": "^7.12.1",
    "@babel/preset-env": "^7.12.1",
    "@babel/preset-react": "^7.12.1",
    "babel-loader": "^8.1.0",
    "clean-webpack-plugin": "^3.0.0",
    "css-loader": "^5.0.0",
    "html-webpack-plugin": "^4.5.0",
    "style-loader": "^2.0.0",
    "webpack": "^5.4.0",
    "webpack-cli": "^4.1.0",
    "webpack-dev-server": "^3.11.0",
    "webpack-merge": "^5.2.0"
  }
}
```

下载应用依赖```npm install```然后拷贝```src```文件夹并做如下修改。


```js
// bootstrap.js
// #dev-marketing -> #dev-auth
if (process.env.NODE_ENV === "development") {
  const el = document.querySelector("#dev-auth")
}
```

```js
// App.js
import React from "react"
import { Router, Route, Switch } from "react-router-dom"
export default function App({ history }) {
  return (
    <Router history={history}>
      <Switch>
        <Route path="/auth/signin" component={Signin}></Route>
      </Switch>
		</Router>
	) 
}
```

拷贝```public```文件夹，并修改```index.html```。

```html
<div id="dev-auth"></div>
```

拷贝```webpack.config.js```文件并进行修改。

```js
 module.exports = {
  devServer: {
		port: 8082 
	},
  plugins: [
    new ModuleFederationPlugin({
      name: "auth",
      exposes: {
        "./AuthApp": "./src/bootstrap"
      }
		}) 
	]
}
```

添加应用启动命令。

```json
// package.json
"scripts": {
  "start": "webpack serve"
}
```

修改```publicPath```更正文件的访问路径。

```js
// webpack.config.js
module.exports = {
  output: {
    publicPath: "http://localhost:8082/"
  }
}
```

更正其他微应用的```publicPath```。

```js
// Container/webpack.config.js
output: {
  publicPath: "http://localhost:8080/"
}
```

```js
// Marketing/webpack.config.js
output: {
  publicPath: "http://localhost:8081/"
}
```

### 1. Container 应用加载 AuthApp

在```Container```应用的```webpack```中配置添加```AuthApp```的远端地址。

```js
// Container/webpack.config.js
remotes: {
  auth: "auth@http://localhost:8082/remoteEntry.js"
}
```

在```Container```应用的```components```文件夹中新建```AuthApp.js```，并拷贝```MarketingApp.js```中的内容进行修改。

```js
import { mount } from "auth/AuthApp"
export default function AuthApp() {}
```

在```Container```应用的```App.js```文件中配置路由。

```jsx
 <BrowserRouter>
  <Switch>
    <Route path="/auth/signin">
      <AuthApp />
    </Route>
    <Route path="/">
      <MarketingApp />
    </Route>
  </Switch>
</BrowserRouter>
```

### 2. 解决登录页面点击两次才显示的 Bug

当点击登录按钮时，容器应用的路由地址是```/auth/signin```，加载```AuthApp```，但是```AuthApp```在首次 加载时默认访问的是```/```，因为在使用```createMemoryHistory```创建路由时没有传递初始参数，当再次 点击登录按钮时，容器应用通知微应用路由发生了变化，微应用同步路由变化，所以最终看到了登录页面。

解决问题的核心点在于微应用在初始创建路由对象时应该接收一个默认参数，默认参数就来自于容器应用。

```js
// container/src/components/AuthApp.js
mount(ref.current, {
  initialPath: history.location.pathname
})
```

```js
// auth/bootstrap.js
function mount(el, { onNavigate, defaultHistory, initialPath }) {
  createMemoryHistory({
    initialEntries: [initialPath]
  })
}
```

按照上述方法修正```MarketingApp```。

## 7. 懒加载微应用

目前所有的微应用都会在用户初始访问时被加载，这样会导致加载时间过长，解决办法就是懒加载微应用。通过```React.lazy```去导入组件。使用```Suspense```包裹```Switch```组件就可以了。```fallback```是加在组件时展示的```UI```。

```js
// Container/app.js
import React, { lazy, Suspense } from "react"
import Progress from "./components/Progress"
const MarketingApp = lazy(() => import("./components/MarketingApp"))
const AuthApp = lazy(() => import("./components/AuthApp"))
function App () {
  return (
    <Suspense fallback={<Progress />}>
      <Switch>
        <Route path="/auth/signin">
          <AuthApp />
				</Route>
        <Route path="/">
          <MarketingApp />
        </Route>
      </Switch>
    </Suspense>
	) 
}
```

```js
// Progress.js
import React from "react"
import { makeStyles } from "@material-ui/core/styles"
import LinearProgress from "@material-ui/core/LinearProgress"
const useStyles = makeStyles(theme => ({
  root: {
    width: "100%",
    "& > * + *": {
      marginTop: theme.spacing(2)
    }
	} 
}))

export default function Progress() {
  const classes = useStyles()
  return (
    <div className={classes.root}>
      <LinearProgress />
    </div>
	)
}
```

### 1. 设置登录状态

由于每个微应用都有可能用到登录状态以及设置登录状态的方法，所以登录状态和设置登录状态的方法需要放置在容器应用中。

```js
// Container/App.js
export default function App() {
	// 存储登录状态
	const [status, setStatus] = useState(false) 
	return <AuthApp setStatus={setStatus} />
}
```

```js
// Container/AuthApp.js
export default function AuthApp({ setStatus }) {
  useEffect(() => {
    mount(ref.current, { setStatus })
  }, [])
}
```

```js
// Auth/bootstrap.js
function mount(el, { setStatus }) {
  ReactDOM.render(<App setStatus={setStatus} />, el)
}
```

```js
// Auth/App.js
export default function App({ setStatus }) {
  return <Signin setStatus={setStatus} />
}
```

```js
// Auth/Signin.js
export default function SignIn({ setStatus }) {
	return <Button onClick={() => setStatus(true)}>登录</Button>
}
```

### 2. 登录状态应用

根据登录状态更改头部组件右侧的按钮文字，如果是未登录状态，显示登录，如果是登录状态，显示退出。点击退出按钮取消登录状态。如果登录状态为真，跳转到```Dashboard```应用。

```js
// Container/App.js
export default function App() {
const [status, setStatus] = useState(false) 
	// 如果登录状态为真，跳转到 Dashboard 应用 
	useEffect(() => {
    if (status) history.push("/dashboard")
  }, [status])
  return (
		<Router history={history}>
			{/* 将登录状态和设置登录状态的方法传递到头部组件 */} 
			<Header status={status} setStatus={setStatus}>
		</Router>
	) 
}
```

```js
// Container/Header.js
export default function Header({ status, setStatus }) { // 当点击按钮时取消登录状态
	const onClick = () => {
    if (status && setStatus) setStatus(false)
  }
	return <Button to={status ? "/" : "/auth/signin"} onClick={onClick}>{status ? "退出" : "登录"}<Button>
}
```

### 3. Dashboard 初始化

```json
{
  "name": "dashboard",
  "version": "1.0.0",
  "scripts": {
    "start": "webpack serve"
  },
  "dependencies": {
    "chart.js": "^2.9.4",
    "primeflex": "^2.0.0",
    "primeicons": "^4.0.0",
    "primevue": "^3.0.1",
    "vue": "^3.0.0"
  },
  "devDependencies": {
    "@babel/core": "^7.12.3",
    "@babel/plugin-transform-runtime": "^7.12.1",
    "@babel/preset-env": "^7.12.1",
    "@vue/compiler-sfc": "^3.0.2",
    "babel-loader": "^8.1.0",
    "css-loader": "^5.0.0",
    "file-loader": "^6.2.0",
    "html-webpack-plugin": "^4.5.0",
    "node-sass": "^4.14.1",
    "sass-loader": "^10.0.4",
    "style-loader": "^2.0.0",
    "vue-loader": "^16.0.0-beta.9",
    "vue-style-loader": "^4.1.2",
    "webpack": "^5.4.0",
    "webpack-cli": "^4.1.0",
    "webpack-dev-server": "^3.11.0",
    "webpack-merge": "^5.2.0"
  }
}
```

下载依赖```npm install```，新建```public```文件夹并拷贝```index.html```文件。

```html
<div id="dev-dashboard"></div>
```

新建```src```文件夹并拷贝```index.js```和```bootstrap.js```。

```js
// bootstrap.js
import { createApp } from "vue"
import Dashboard from "./components/Dashboard.vue"
function mount(el) {
  const app = createApp(Dashboard)
  app.mount(el)
}
if (process.env.NODE_ENV === "development") {
  const el = document.querySelector("#dev-dashboard")
  if (el) mount(el)
}
export { mount }
```

拷贝```webpack.config.js```文件并做如下修改。

```js
const HtmlWebpackPlugin = require("html-webpack-plugin")
const { VueLoaderPlugin } = require("vue-loader")
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin")
const packageJson = require("./package.json")

module.exports = {
  mode: "development",
  entry: "./src/index.js",
  output: {
    publicPath: "http://localhost:8083/",
    filename: "[name].[contentHash].js"
  },
  resolve: {
    extensions: [".js", ".vue"]
  },
  devServer: {
    port: 8083,
    historyApiFallback: true,
    headers: {
      "Access-Control-Allow-Origin": "*"
    }
	}, 
	module: {
		rules: [ 
			{
        test: /\.(png|jpe?g|gif|woff|svg|eot|ttf)$/i,
        use: [
					{
						loader: "file-loader"
					} 
				]
			}, 
			{
        test: /\.vue$/,
        use: "vue-loader"
      },
      {
        test: /\.scss|\.css$/,
        use: ["vue-style-loader", "style-loader", "css-loader", "sass-loader"] 
			},
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env"],
            plugins: ["@babel/plugin-transform-runtime"]
          }
				} 
			}
		] 
	},
  plugins: [
    new ModuleFederationPlugin({
      name: "dashboard",
      filename: "remoteEntry.js",
      exposes: {
        "./DashboardApp": "./src/bootstrap"
      },
      shared: packageJson.dependencies
    }),
    new HtmlWebpackPlugin({
      template: "./public/index.html"
		}),
    new VueLoaderPlugin()
  ]
}
```

### 4. 修改启动命令

```json
"scripts": {
  "start": "webpack serve"
}
```

### 5. Container 应用加载 Dashboard

```Container```配置```ModuleFedaration```。

```js
// container/webpack.config.js
remotes: {
  dashboard: "dashboard@http://localhost:8083/remoteEntry.js"
}
```
 
新建```DashboardApp```组件。

```js
import React, { useRef, useEffect } from "react"
import { mount } from "dashboard/DashboardApp"
export default function DashboardApp() {
  const ref = useRef()
  useEffect(() => {
    mount(ref.current)
  }, [])
  return <div ref={ref}></div>
}
```

```Container```应用添加路由。

```js
const DashboardApp = lazy(() => import("./components/DashboardApp"))
function App () {
  return (
    <Route path="/dashboard">
      <DashboardApp />
		</Route>
	) 
}
```
重启```Container```应用查看效果 。

### 6， Dashboard 路由保护

```js
function App () {
  const [status, setStatus] = useState(false)
  useEffect(() => {
    if (status) {
      history.push("/dashboard")
    }
  }, [status])
  return (
    <Router history={history}>
      <Route path="/dashboard">
        {!status && <Redirect to="/" />}
        <DashboardApp />
      </Route>
		</Router>
	) 
}
```

```jsx
// Marketing/Landing.js
<Link to="/dashboard">
  <Button variant="contained" color="primary">
		Dashboard
  </Button>
</Link>
```
