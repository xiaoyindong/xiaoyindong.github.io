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
