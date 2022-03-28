```webpack```作为一个模块打包工具(```Module Bundler```)本身可以解决模块化打包的问题，通过```webpack```可以将一些零散的模块代码打包到同一个```js```文件中。对于代码中环境兼容问题的代码可以在打包的过程中通过模块加载器(```Loader```)对其进行编译转换。

其次，```webpack```还具备代码拆分(```Code Splitting```)的理念，能够将应用中所有的代码都按照需要进行打包。这样一来就不用担心代码全部打包到一起文件较大的问题了。

把应用加载过程中初次运行所必须的模块打包到一起，对于其他的模块单独存放。等应用工作过程中实际需要某个模块再异步加载这个模块从而实现增量加载或渐进式加载，这样就不用担心文件太碎或是文件太大的问题。

```webpack```支持在```js```中以模块化的方式载入任意类型的资源文件，例如可以通过```js```直接```import```一个```css```文件。这些```css```文件最终会通过```style```标签的形式工作，其他类型的文件也可以有类似的这种方式去实现。

## 1. 快速上手

```webpack``` 作为目前最主流的代码打包工具提供了一整套的前端项目模块化方案而不仅仅局限于对```js```的模块化。通过```webpack```提供的前端模块化方案，可以很轻松的对前端项目涉及到的所有的资源进行模块化。

这里有一个项目，目录中有个```src```文件夹，```src```中有两个文件 ```index.js``` 和 ```heading.js```， 在```src```同级有一个```index.html```文件。```heading.js```中默认导出一个用于创建元素的函数。

```js
export default () => {
    const element = document.createElement('h2');
    element.textContent = 'Hello word';
    element.addEventListener('click', () => {

    })
    return element;
}

```

```index.js```中导入模块并且使用了他。

```js
import createHeading from './heading.js';

const heading = createHeading();

document.body.append(heading);
```

在```index.html```中通过```script```标签以模块化的方式引入了```index.js```。

```html
<body>
    <script type="module" src="src/index.js"></script>
</body>
```

打开命令行通过```http-server .```工具运行起来。

```s
http-server .
```

可以看到正常的工作。下来引入```webpack```处理```js```模块。首先以通过```yarn init```的方式去初始化一个```package.json```。

```s
yarn init
```

完成过后安装```webpack```所需要的核心模块以及对应的```cli```模块。

```s
yarn add webpack webpack-cli --dev
```

有了```webpack```之后就可以打包```src```下面的```js```代码了。执行```yarn webpack```命令```webpack```会自动从```src```下面的```index.js```开始打包。

```s
yarn webpack
```

完成过后控制台会提，有两个```js```文件被打包到了一起，与之对应的是在项目的跟目录会多出一个```dist```目录，打包的结果就会存放在这个目录的```main.js```中。

回到```index.html```中，把```js```脚本文件的路径修改成```dist/main.js```，由于打包过程会把```import```和```export```转换掉，所以说已经不需要```type="module"```这种模块化的方式引入了。

```html
<body>
-    <script type="module" src="src/index.js"></script>
+    <script src="dist/main.js"></script>
</body>
```

再次启动服务，应用仍然可以正常工作。

```s
http-server .
```

可以把```webpack```命令放到```package.json```中的```script```，通过```yarn build```打包。

```json
"script": {
    "build": "webpack"
}
```

```s
yarn build
```

## 2. 配置文件

```webpack4.0```后的版本支持零配置打包，整个打包过程会按约定将```src/index.js```作为入口结果存放在```dist/main.js```中。很多时候需要自定义路径，例如入口文件是```src/main.js```，这就需要为```webpack```添加配置文件，在项目的跟目录添加```webpack.config.js```文件即可。这个文件运行在```node```环境也就说需要按照```Commonjs```的方式编写代码。

文件导出一个对象, 通过导出对象的属性可以完成相应的配置选项，例如```entry```属性指定```webpack```打包入口文件的路径。可以将其设置为```./src/main.js```。

```js
module.exports = {
    entry: './src/main.js'
}
```

可以通过```output```配置输出文件的位置，属性值是一个对象，对象中的```filename```指定输出文件的名称，```path```属性指定输出文件的目录需要是一个绝对路径。

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist')
    }
}
```

运行```yarn build```就在项目中生成了```dist/bundle.js```。

## 3. 工作模式

```webpack```新增了工作模式简化了```webpack```配置的复杂度，可以理解成针对不用环境的几组预设的配置，```webpack```可以设置一个```mode```属性，如不设置默认会使用```production```模式工作。在这个模式下```webpack```会自动启动一些优化插件，例如代码压缩。

可以在```webpack```启动时传入```--mode```的参数，这个属性有三种取值，默认是```production```，还有```development```也就是开发模式。开发模式```webpack```会自动优化打包的速度，会添加一些调试过程需要的服务到代码中。

```s
yarn webpack --mode=development
```

```node模```式就是运行最原始状态的打包，不会去任何额外的处理。

```s
yarn webpack --mode=none
```

除了通过```cli```参数指定工作模式，还可以在```webpack```的配置文件中设置工作模式，在配置文件的配置中添加mode属性就可以了。

```js
const path = require('path');

module.exports = {
    mode: 'development',
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist')
    }
}
```

## 4. 打包结果分析

首先先将```webpack```的工作模式设置成```node```。这样就是以最原始的状态打包。

```js
const path = require('path');

module.exports = {
    mode: 'node',
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist')
    }
}
```

```s
yarn webpack
```

完成过后打开生成的```bundle.js```文件，可以把整体结构折叠起来以便于对结构了解。快捷键是```ctrl + k``` 和 ```ctrl + 0```。

整体生成的代码是一个立即执行函数，这个函数是```webpack```的工作入口。接收一个叫做```modules```的参数，调用的时传入了一个数组。

```js
/******/ (function(modules) { // 接收参数位置
/******/ })
/******/ ([ // 调用位置
/******/ ]);
```

数组中的每个参数都是需要相同参数的函数，这里的函数对应的就是源代码中的模块。

```js
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
/***/ })
/******/ ]);
```

也就是说每一个模块最终都会被包裹到一个函数中，从而实现模块的私有作用域。可以展开数组中第一个参数函数。

```js
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _heading_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);


const heading = Object(_heading_js__WEBPACK_IMPORTED_MODULE_0__["default"])();

document.body.append(heading);

/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = (() => {
    const element = document.createElement('h2');
    element.textContent = 'Hello word';
    element.addEventListener('click', () => {

    })
    return element;
});

/***/ })
/******/ ]);
```

```webpack```工作入口函数并不复杂注释也非常清晰，最开始先定义了一个对象(```installedModules```)，用于存放加载过的模块。紧接着定义了一个```__webpack_require__```函数，这个函数就是用来加载模块的，再往后就是向```__webpack_require__```函数上挂载了一些数据和一些工具函数。

这个函数执行到最后调用了```__webpack_require__```函数传入了```__webpack_require__.s = 0```开始加载模块，这个地方的模块```id```实际上就是上面模块数组中的元素下标，也就是说这里才开始加载源代码中的入口模块。

```js
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })
```

```__webpack_require__```内部先判断了这个模块有没有被加载过，如果加载了就从缓存里面读，如果没有就创建一个新的对象。创建过后开始调用这个模块对应的函数，把刚刚创建的模块对象(```module```)，导出成员对象(```module.exports```)，```__webpack_require__```函数作为参数传入进去。这样的话在模块的内部就可以使用```module.exports```导出成员，通过```__webpack_require__```载入模块。

```js
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
```

在模块内部先调用了```__webpack_require__.r```函数，这个函数的作用是给导出对象添加一个标记，用来对外界表明这是一个```ES Module```。

```js
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _heading_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);


const heading = Object(_heading_js__WEBPACK_IMPORTED_MODULE_0__["default"])();

document.body.append(heading);

/***/ }),
```

```__webpack_require__.r```函数。

```js
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
```

再往下又调用了```__webpack_require__```函数，此时传入的```id```是```1```，也就是说用来加载第一个模块。

```js
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _heading_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);


const heading = Object(_heading_js__WEBPACK_IMPORTED_MODULE_0__["default"])();

document.body.append(heading);

/***/ }),
```

这个模块就是代码中```export```的```heading```，以相同的道理执行```heading```模块，将```heading```模块导出的对象```return```回去。

```js
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = (() => {
    const element = document.createElement('h2');
    element.textContent = 'Hello word';
    element.addEventListener('click', () => {

    })
    return element;
});

/***/ })
```

```module.exports```是一个对象，```ES Module```里面默认是放在```default```里面，调用default函数将创建完的元素拿到```append```到```body```上面。

```js
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _heading_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);


const heading = Object(_heading_js__WEBPACK_IMPORTED_MODULE_0__["default"])();

document.body.append(heading);

/***/ }),

```

这就是大致的执行过程，```webpack```打包过后的代码并不会特别的复杂，只是把所有的模块放到了同一个文件中，除了放到同一个文件当中还提供一个基础代码让模块与模块之间相互依赖的关系可以保持原有的状态，这实际上就是```webpack bootstrap```的作用。

打包的全部代码如下。

```js
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
/******/ 		}
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 		if(mode & 1) value = __webpack_require__(value);
/******/ 		if(mode & 8) return value;
/******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
/******/ 		var ns = Object.create(null);
/******/ 		__webpack_require__.r(ns);
/******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
/******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
/******/ 		return ns;
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _heading_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);


const heading = Object(_heading_js__WEBPACK_IMPORTED_MODULE_0__["default"])();

document.body.append(heading);

/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = (() => {
    const element = document.createElement('h2');
    element.textContent = 'Hello word';
    element.addEventListener('click', () => {

    })
    return element;
});

/***/ })
/******/ ]);
```

## 6. 模块依赖方式

```css```文件也可以作为打包的入口，不过```webpack```的打包入口一般还是```js```，打包入口从某种程度来说可以算是应用的运行入口。就目前而言前端应用中的业务是由js驱动的，可以在```js```代码当中通过```import```的方式引入```css```文件。

```js
import createHeading from './heading.js';

import './style.css';

const heading = createHeading();

document.body.append(heading);
```

在webpack.config.js中配置```css```的```loader```，```css-loader``` 和 ```style-loader``` 需要安装到项目中。然后将```loader```需要配置到```config```的```module```中。

```s
yarn add css-loader style-loader --dev
```

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist')
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            }
        ]
    }
}
```

运行打包命令启动项目后，样式是可以生效的。传统模式开发是将文件单独分开单独引入，```webpack```建议在```js```中去引入```css```，甚至编写代码中引入资源都可以在```js```中印日。因为真正需要资源的不是应用，而是正在编写的代码，代码想要正常工作就必须要加载对应的资源，这就是```webpack```的哲学。一开始可能不太容易理解，换种方式理解假设样式单独引入到页面中，如果代码更新了不再需要这个样式资源了，是不是需要手动的删除。通过```js```的代码引入文件或者建立```js```和文件之间的依赖关系是有明显优势的。

```js```代码本身是负责完成整个业务的功能，放大来看就是驱动了整个前端应用，在实现业务功能的过程当中可能需要用到样式或图片等一系列的资源文件。如果建立了这种依赖关系，一来逻辑上比较合理，因为```js```确实需要这些资源文件的配合才能实现对应的功能，二来可以保证上线时资源文件不缺失，而且每一个上线的文件都是必要的。

## 7. 文件资源加载器

```webpack```社区提供了非常多的资源加载器，基本上开发者能想到的合理需求都有对应的```loader```，接下来尝试一些非常有代表性的```loader```，首先是文件资源加载器。

大多数文件加载器都类似于```css-loader```，是将资源模块转换为```js```代码的实现方式进行工作，但是有一些经常用到的资源文件例如图片或字体这些文件是没办法通过```js```表示的。对于这类的资源文件，需要用到文件的资源加载器也就是```file-loader```。

在项目中添加一张普通的图片文件，通过```import``` 的方式导入这张图片。接收模块文件的默认导出也就是文件的资源路径，创建```img```元素把```src```设置成文件，最后将元素```append```到```body```中。

```js

import createHeading from './heading.js';

import './style.css';

import icon from './icon.png';

const heading = createHeading();

document.body.append(heading);

const img = new Image();

img.src = icon;

document.body.append(img);
```

这里导入了一个```webpack```不能识别的资源所以需要修改```webpack```配置。为```png```文件添加一个单独的加载规则配置，```test```属性设置```.png```结尾，```use```属性设置为```file-loader```，这样```webpack```打包的时候就会用```file-loader```处理图片文件了。

```s
yarn add file-loader --dev
```

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist')
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    }
}
```

打包过后```dist```目录中会多出一个图片文件，这个文件就是代码中导入的图片，不过文件名称发生了改变。文件模块代码只是把生成的文件名称导出了。

```js
/* 6 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = (__webpack_require__.p + "e177e3436b8f0b3cfff0fd836ea3472c.png");

/***/ })
```

入口模块直接使用了导出的文件路径(```__webpack_require__(6)```)```img.src = _icon_png__WEBPACK_IMPORTED_MODULE_2__["default"];```。

```js
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _heading_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);
/* harmony import */ var _style_css__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(2);
/* harmony import */ var _style_css__WEBPACK_IMPORTED_MODULE_1___default = /*#__PURE__*/__webpack_require__.n(_style_css__WEBPACK_IMPORTED_MODULE_1__);
/* harmony import */ var _icon_png__WEBPACK_IMPORTED_MODULE_2__ = __webpack_require__(6);

const heading = Object(_heading_js__WEBPACK_IMPORTED_MODULE_0__["default"])();

document.body.append(heading);

const img = new Image();

img.src = _icon_png__WEBPACK_IMPORTED_MODULE_2__["default"];

document.body.append(img);

/***/ })
```

启动应用发现图片并不能正常的加载，控制台终端可以发现直接加载了网站根目录的图片，而网站根目录并没有这个图片所以没有找到。图片应该在```dist```目录当中。这个问题是由于```index.html```并没有生成到```dist```目录，而是放在了项目的跟目录，所以这里把项目的跟目录作为了网站的跟目录，而```webpack```会认为所有打包的结果都会放在网站的跟目录下面，所以就造成了这样一个问题。

通过配置文件去```webpack```打包过后的文件最终在网站当中的位置，具体的做法就是在配置文件中```output```位置添加```publicPath```。这里设置为```dist/```斜线不能省略。

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    }
}
```

完成以后重新打包，这一次在文件名称前面拼接了一个变量。

```js
/* 6 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = (__webpack_require__.p + "e177e3436b8f0b3cfff0fd836ea3472c.png");

/***/ })
```

这个变量在```webpack```内部的代码提供的就是设置的```publicPath(\__webpack_require__.p = "dist/";)```

```js

/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
/******/ 		}
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 		if(mode & 1) value = __webpack_require__(value);
/******/ 		if(mode & 8) return value;
/******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
/******/ 		var ns = Object.create(null);
/******/ 		__webpack_require__.r(ns);
/******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
/******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
/******/ 		return ns;
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "dist/";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })

```

````webpack````在打包时遇到图片文件，根据配置文件中的配置，拼配到对应的文件加载器，此时文件加载器开始工作，先是将文件拷贝到输出的目录，然后再将文件拷贝到输出目录的路径作为当前模块的返回值返回，这样对于应用来说，所需要的资源就被发布出来了，同时也可以通过模块的导出成员拿到资源的访问路径。

## 8. url加载器

除```file-loader```这种通过```copy```文件的形式处理文件资源外还有一种通过```Data URLs```的形式表示文件。```Data URLs```是一种特殊的```url```协议，可以直接表示文件，传统的```url```要求服务器上有对应的文件，然后通过地址，得到服务器上对应的文件。而```Data URLs```本身就是文件内容，在使用这种```url```的时候不会再去发送任何的```http```请求，比如常见的```base64```格式。

```s
data:[mediatype][;base64],\<data>
```

```data:```表示协议，```[mediatype][;base64]```表示媒体类型和编码，```\<data>```则是具体的文件内容。例如下面给出的```Data URLs```，浏览器可以根据这个```url```解析出```html```类型的文件内容，编码是```url-8```，内容是一段包含```h1```的```html```代码。

```s
data:text/html;charset=UTF-8,<h1>html content</h1>
```

如果是图片或者字体这一类无法通过文本表示的```2```进制类型的文件，可以通过将文件的内容进行```base64```编码，以编码后的结果也就是字符串表示这个文件内容。这里```url```就是表示了一个```png```类型的文件，编码是```base64```，再后面就是图片的```base64```编码。

```s
data:image/png;base64,iVBORw0KGgoAAAANSUhE...SuQmCC
```

当然一般情况下```base64```的编码会比较长，这就导致编码过后的资源体积要比原始资源大，不过优点是浏览器可以直接解析出文件内容，不需要再向服务器发送请求。

```webpack```在打包静态资源模块时，就可以使用这种方式去实现，通过```Data URLs```以代码的形式表示任何类型的文件，需要用到一个专门的加载器```url-loader```。

```s
yarn add url-loader --dev
```

```webpack```配置文件中找到之前的```file-loader```将其修改为```url-loader```。

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'url-loader'
            }
        ]
    }
}
```

此时```webpack```打包时，再遇到```.png```文件就会使用```url-loader```将其转换为```Data URLs```的形式。打开```bundle.js```可以发现在最后的文件模块中导出的是一个完整的```Data URLs```。

```js
/* 6 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = ("data:image/png;base64,iVBORw0KGgoAAAANSUh...AAAABJRU5ErkJggg==");

/***/ })
```

因为```Data URLs```中已经包含了文件内容，所以```dist```中也就不存在独立的```.png```物理文件了。

这种方式十分适合项目当中体积比较小的资源，如果体积过大会造成打包结果非常大从而影响运行速度。最佳的实践方式是对项目中的小文件通过```url-loader```转换成```Data URLs```然后在代码中嵌入，从而减少应用发送请求次数。对于较大的文件仍然通过传统的```file-loader```方式以单个文件方式存放，从而提高应用的加载速度。

```url-loader```支持通过配置选项的方式设置转换的最大文件，将```url-loader```字符串配置方式修改为对象的配置方式，对象中使用```loader```定义```url-loader```，然后额外添加```options```属性为其添加一些配置选项。这里为```url-loader```添加```limit```的属性，将其设置为 ```10kb(10 * 1024)```，单位是字节。

这样```url-loader```只会将```10kb```以下的文件转换成```Data URLs```，超过```10kb```的文件仍然会交给```file-loader```去处理。

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: {
                    loader: 'url-loader',
                    options: {
                        limit: 10 * 1024
                    }
                }
            }
        ]
    }
}
```

## 9. babel-loader

```webpack```默认就可以处理代码中的```import```和```export```，所以很自然的会有人认为，```webpack```会自动编译```ES6```的代码，实则不然，```webpack```仅仅完成模块打包工作，会对代码中的```import```和```export```做一些相应的转换，除此之外它并不能转换代码中其他的```ES6```代码。如果需要```webpack```在打包过程中同时处理其他```ES6```特性，需要为```js```文件配置一个额外的加载器```babel-loader```。

首先需要安装```babel-loader```，由于```babel-loader```需要依赖额外的```babel```核心模块，所以需要安装```@babel/core```模块和用于完成具体特性转换```@babel/preset-env```模块。

```s
yarn add babel-loader @babel/core @babel/preset-env --dev
```

配置文件中为js文件指定加载器为```babel-loader```，这样```babel-loader```就会取代默认的加载器，在打包过程当中处理代码中的一些新特性。

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.js$/,
                use: 'babel-loader'
            }
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'url-loader'
            }
        ]
    }
}
```

还需要为```babel```配置需要使用的插件，配置文件中给```babel-loader```传入相应的配置，们直接使用```preset-env```插件集合，这个集合当中就已经包含了全部的```ES```最新特性。

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            }
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'url-loader'
            }
        ]
    }
}
```

## 10. 加载资源

```webpack```中提供了几种资源加载方式，首先第一个就是```ES Module```标准的```import```声明。

```js
import heading from './heading.js';
import icon from './icon.png';
```

其次是遵循```Commonjs```标准的```require```函数，不过通过```require```函数载入```ES Module```的话，对于```ES Module```的默认导出需要通过```require```函数导入结果的```default```属性获取。

```js
const heading = require('./heading.js').default;
const icon = require('./icon.png');
```

遵循```AMD```标准的```define```函数和```require```函数```webpack```也同样支持。

```js
define(['./heading.js', './icon.png', './style.css'], (createHeading, icon) => {
    const heading = createHeading();
    const img = new Image();
    img.src = icon;
    document.body.append(heading);
    document.body.append(icon);
});

require(['./heading.js', './icon.png', './style.css'], (createHeading, icon) => {
    const heading = createHeading();
    const img = new Image();
    img.src = icon;
    document.body.append(heading);
    document.body.append(icon);
})
```

```webpack```兼容多种模块化标准，除非必要的情况否则不要在项目中去混合使用这些标准，每个项目使用一个标准就可以了。

除了```js```代码中的三种方式外还有一些加载器在工作时也会处理资源中导入的模块，例如```css-loader```加载的```css```文件(```@import```指令和```url```函数)

```css
@import '';
```

```html-loader```加载的```html```文件中的一些```src```属性也会触发相应的模块加载。

```main.js```

```js
import './main.css';
```

```main.css```

```css
body {
    min-height: 100vh;
    background-image: url(background.png);
    background-size: cover;
}
```

```webpack```在遇到```css```文件时会使用```css-loader```进行处理，处理的时候发现```css```中有引入图片，就会将图片作为一个资源模块加入到打包过程。```webpack```会根据配置文件中针对于遇到的文件找到相应的```loader```，此时这是一张```png```图片就会交给```url-loader```处理。

```reset.css```。

```css
@import url(reset.css);
body {
    min-height: 100vh;
    background-image: url(background.png);
    background-size: cover;
}
```

```html```文件中也会引用其他文件例如img标签的```src```，```src/footer.html```。

```html
<footer>
    <img src="better.png" />
</footer>
```

```s
yarn add html-loader --dev
```

配置文件中为扩展名为```html```的文件配置```loader```。

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            }
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'url-loader'
            },
            {
                test: /.html$/,
                use: 'html-loader'
            }
        ]
    }
}
```

```html-loader```默认只会处理```img```标签的```src```属性，如果需要其他标签的一些属性也能够触发打包可以额外做一些配置，具体的做法就是给```html-loader```添加```attrs```属性，也就是```html```加载的时候对页面上的属性做额外的处理。比如添加一个```a:href```属性，让他能支持```a```标签的```href```属性。

```js
const path = require('path');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            }
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'url-loader'
            },
            {
                test: /.html$/,
                use: {
                    loader: 'html-loader',
                    options: {
                        attrs: ['img:src', 'a:href']
                    }
                }
            }
        ]
    }
}
```

完成以后运行打包，在打包的结果中可以看到```a```标签用到的资源已经参与了打包。

## 11. 工作原理

在项目中一般都会散落着各种各样代码及资源文件，```webpack```会根据配置找到其中的一个文件作为打包的入口，一般情况这个文件都会是```js```文件。

然后顺着入口文件中的代码根据代码中出现的import或者```require```之类的语句解析推断出来这个文件所依赖的资源模块，然后分别解析每个资源模块对应的依赖，最后就形成了整个项目中所有用到文件之间的一个依赖关系的依赖树。

有了依赖关系树后```webpack```会递归这个依赖树然后找到每个节点对应的资源文件。最后再根据配置文件中的属性找到这个模块所对应的加载器，然后交给加载器去加载这个模块。

最后会将加载到的结果放到```bundle.js```也就是打包结果中，从而实现整个项目的打包。整个过程中```loader```的机制起了很重要的作用，如果没有```loader```就没办法实现各种各样的资源文件的加载，对于```webpack```来说也就只能算是一个用来去打包或是合并```js```模块代码的工具了。

## 12. 开发一个Loader

```markdown-loader```，需求是有了这个加载器后就可以在代码当中直接导入```markdown```文件。

```main.js```

```js
import about from './about.md';
console.log(about);
```

```about.md```

```md
# 关于我

我是隐冬
```

```markdown```文件一般是要被转换为```html```后呈现到页面上的，所以说这里希望导入的```markdown```文件得到的结果是````markdown````转换过后的```html```字符串。

在项目的根目录创建```markdown-loader.js```文件，```webpack-loader```需要去导出一个函数，这个函数就是```loader```对所加载到资源的处理过程，入参是加载到的资源文件的内容，输出是加工过后的结果。通过```source```接收输入，通过返回值输出。

```js
module.exports = source => {
    console.log(source);
    return 'hello';
}
```

在```webpac```k的配置文件中添加加载器的规则配置，扩展名就是```.md```使用的加载器是我编写的```markdown-loader```模块。

```js
const path = require('path');

module.exports = {
    mode: 'none',
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.md$/,
                use: './markdown-loader'
            }
        ]
    }
}

```


```webpack```加载资源的过程类似工作管道，可以在这个过程中依次使用多个```loader```，但是最终这个管道工作过后的结果必须是一段```javascript```代码，```markdow-loader```中，将返回的字符串修改为```console.log("hello")```标准的```js```代码。

```js
module.exports = source => {
    console.log(source);
    return 'console.log("hello")';
}
```

```webpack```打包的时候就是把```loader```返回的字符串拼接到模块当中了。

```js
/* 1 */
/***/ (function(module, exports) {

console.log("hello")

/***/ })

```

安装```markdown```解析的模块```marked```。

```s
yarn add marked --dev
```

在加载器当中使用这个模块去解析来自参数中的```source```，这里返回值就是一段```html```字符串也就是转换过后的结果。正确的做法就是把这段```html```变成一段```javascript```代码。

```js
const marked = require('marked');
module.exports = source => {
    // console.log(source);
    // return 'console.log("hello")';
    const html = marked(source);
    return `module.exports = ${JSON.stringify(html)}`
}
```

打包后就是下面的样子。
```js
/* 1 */
/***/ (function(module, exports) {

module.exports = "<h1 id=\"关于我\">关于我</h1>\n<p>我是隐冬</p>\n"

/***/ })
```

除了```module.exports```方式以外```webpack```还允许在返回的代码中直接使用```ES Module```的方式去导出。

```js
const marked = require('marked');
module.exports = source => {
    // console.log(source);
    // return 'console.log("hello")';
    const html = marked(source);
    // return `module.exports = ${JSON.stringify(html)}`
    return `export default ${JSON.stringify(html)}`
}
```

打包结果同样也是可以的，```webpack```内部会自动转换导出代码中的```ES Module```。

```js
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony default export */ __webpack_exports__["default"] = ("<h1 id=\"关于我\">关于我</h1>\n<p>我是隐冬</p>\n");

/***/ })
```

接下来尝试一下第二种方法，```markdown-loader```中去返回一个```html```字符串。然后交给下一个```loader```去处理这个```html```的字符串。这里直接返回```marked```解析过后的```html```，然后再去安装一个用于去处理```html```加载的```loader```叫做```html-loader```。

```js
const marked = require('marked');
module.exports = source => {
    // console.log(source);
    // return 'console.log("hello")';
    const html = marked(source);
    // return `module.exports = ${JSON.stringify(html)}`
    // return `export default ${JSON.stringify(html)}`
    return html;
}
```

```s
yarn add html-loader --dev
```

把```use```属性修改为一个数组，这样就会依次使用多个```loader```了。需要注意执行顺序是从数组的后面往前面，也就是说应该把先执行的```loader```放在数组的后面。

```js
const path = require('path');

module.exports = {
    mode: 'none',
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.md$/,
                use: ['html-loader', './markdown-loader']
            }
        ]
    }
}

```

完成后打包依然是可以的。

```js
/* 1 */
/***/ (function(module, exports) {

module.exports = "<h1 id=\"关于我\">关于我</h1>\n<p>我是隐冬</p>\n"

/***/ })
```

```loader```不建议使能用剪头函数会拿不到上下文的```this```。

官方推荐使用```loader-utils```工具处理```loader.query```。

```js
const loaderUtils = require('loader-utils');
module.exports = function(source) {
    const options = loaderUtils.getOptions(this);
    console.log(options);
    return source;
}
```

```this.callback```可以返回更多内容用于替代```return```。

```js
module.exports = function(source) {
    const options = loaderUtils.getOptions(this);
    const result = "2123";
    this.callback(null, result);
}
```

```this.async```用于处理异步。

```js
module.exports = function(source) {
    const options = loaderUtils.getOptions(this);
    // 定义异步callback
    const callback = this.async();
    setTimeout(() => {
        const result = "2123";
        callback(null, result);
    });
}
```

```resolveLoader```可以用于```webpack```配置```loader```的简写，当配置文件里面使用模块，先去```node_modules```里面找，如果找不到就去后面路径上面找。


```json
resolveLoader: {
    modules: ["node_modules", "./loaders"],
}
```

## 13. 插件机制介绍

插件机制是```webpack```另一个核心特性，目的是为了增强```webpack```在项目自动化方面的能力，```loader```负责实现项目中各种各样资源模块的加载，```plugin```则是用来解决项目中除了资源加载以外其他的一些自动化的工作。

例如```plugin```可以实现在打包之前清除```dist```目录，还可以```copy```不需要参与打包的资源文件到输出目录，又或是压缩打包结果输出的代码。总之，有了插件```webpack```几乎无所不能的实现了前端工程化中绝大多数工作，这也是很多初学者会把```webpack```理解成前端工程化的原因。

接下来体验几个常见的插件。

### 1. clean-webpack-plugin

自动清除输出目录，```webpack```每次打包的结果都是覆盖到```dist```目录而在打包之前```dist```中可能已经存在一些之前的遗留文件，再次打包可能只覆盖那些同名的文件，对于其他已经移除的资源文件会一直积累在```dist```里面非常不合理。合理的做法是每次打包前自动清理```dist```目录，这样的话```dist```中就只会保留需要的文件。

```clean-webpack-plugin```就很好的实现了这样一个需求。

```s
yarn add clean-webpack-plugin --dev
```

```webpack```配置文件中导入这个插件，插件中导出了一个```CleanWebpackPlugin```的成员可以解构出来，```webpack```使用插件需要为配置对象添加```plugins```属性，值是一个数组里面每一个成员就是一个插件实例。绝大多数插件模块导出的都是一个类型，使用插件就是通过类型创建实例，然后将实例放入到```plugins```数组中。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin()
    ]
}
```

### 2. html-webpack-plugin

在之前```html```都是通过硬编码的方式单独存放在项目的跟目录下的这种方式有两个问题，第一项目发布时需要发布跟目录下的```html```文件和```dist```目录下所有的打包结果，相对麻烦一些。而且上线过后还需要确保```html```代码当中路径引用都是正确的。第二个如果输出的目录或输出的文件名也就是打包结果的配置发生了变化，那```html```代码当中```script```标签所引用的路径也要手动修改。

解决这两个问题最好的办法就是通过```webpack```自动生成```html```文件，也就是让```html```参与到构建过程中去，在构建过程中```webpack```知道生成了多少个```bundle```，会自动将这些打包的```bundle```添加到页面中。这样```html```也输出到了```dist```目录，上线时只需把```dist```目录发布出去就可以了。二来html中对于```bundle```的引用是动态注入的，不需要硬编码也就确保了路径的引用是正常的。

需要借助```html-webpack-plugin```插件。

```s
yarn html-webpack-plugin --dev
```

配置文件中载入这个模块```html-webpack-plugin```默认导出的就是一个插件的类型，不需要解构他内部的成员。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin()
    ]
}
```

打包过后```dist```目录中生成了```index.html```文件，这里引入的```bundle.js```路径是可以通过```output```属性中的```publicPath```进行修改的，可以删除这个配置。这样打包之后```index.html```中```bundle```的引用就标成了```/bundle.js```。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        // publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin()
    ]
}
```


对于默认生成的```html```的标题是可以修改的，页面中的一些原数据标签和基础的```DOM```结构也是可以定义的，对于简单的自定义可以通过修改```html-webpack-plugin```插件传入的参数属性实现。```html-webpack-plugin```构造函数可以传入一个对象参数，用于指定配置选项，```title```属性就是用来设置```html```的标题。```meta```属性可以设置页面中的一些原数据标签。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        // publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            meta: {
                viewport: 'width=device-width'
            }
        })
    ]
}
```

如果需要对```html```文件进行大量自定义，最好的做法就是在原代码中添加一个用于生成```html```文件的一个模板，然后让```html-webpack-plugin```插件根据模板生成页面。

对于模板中动态输出的内容可以使用loadsh模板语法的方式去输出。通过```htmlWebpackPlugin.options```属性去访问到插件的配置数据，```htmlWebpackPlugin```变量实际上是内部提供的变量，也可以通过另外的属性添加一些自定义的变量。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
    <script src="dist/"></script>
</body>
</html>
```

配置文件当中通过```template```属性指定所使用的模板文件。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        // publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            meta: {
                viewport: 'width=device-width'
            },
            template: './src/index.html'
        })
    ]
}
```

除了自定义输出文件的内容，同时输出多个页面文件也是常见的需求，可以通过创建新的实例对象，用于去创建额外的```html```文件，通过```filename```指定输出的文件名，这个属性的默认值是```index.html```。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        // publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            meta: {
                viewport: 'width=device-width'
            },
            template: './src/index.html'
        }),
        new HtmlWebpackPlugin({
            filename: 'about.html'
        })
    ]
}

```

如果说需要创建多个页面，就可以在插件列表当中加入多个```htmlWebpackPlugin```实例的对象，每个对象就是用来负责生成一个页面文件的。

### 3. copy-webpack-plugin

项目中一般有一些不需要参与构建的静态文件最终也需要发布到线上，例如网站的```favicon.ico```，一般会把这一类文件统一放在项目根目录下的```public```目录中，希望```webpack```在打包时可以将他们复制到输出目录。对于这种需求可以借助```copy-webpack-plugin```实现。

```s
yarn add copy-webpack-plugin --dev
```

配置文件当中导入这个插件的类型并在```plugins```属性当中添类型实例，这个这个类型的构造函数要求传入一个数组，用于指定需要```copy```的文件路径，可以是一个通配符，也可以是一个目录或者是文件的相对路径。这里传入```public/**```表示在打包时会将```public```目录下所有的文件拷贝到输出目录。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        // publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            meta: {
                viewport: 'width=device-width'
            },
            template: './src/index.html'
        }),
        new HtmlWebpackPlugin({
            filename: 'about.html'
        }),
        new CopyWebpackPlugin([
            'public/**'
        ])
    ]
}

```

至此就了解了几个非常常用的插件，这些插件一般都适用于任何类型的项目，最好能仔细过一遍这些插件的官方说明，然后看看他们还可以有哪些特别的用法，做到心中有数。

除此之外社区当中还提供了成百上千的插件，并不需要全部认识，在有一些特殊的需求时，提炼需求中的一些关键词然后去```github```上去搜索他们，例如想要压缩输出的图片可以搜索```imagemin webpack plugin```。

## 14. 开发一个插件

```webpack```的插件机制其实就是软件开发过程中最长见到的钩子机制。```webpack```要求的插件必须是一个函数，或者是一个包含```apply```方法的对象，一般会把插件定义为一个类型，在类型中定义```apply```方法。

这里定义```MyPlugin```的类型，在这个类型中定义```apply```方法，这个方法会在```webpack```启动时被调用，接收一个```compiler```对象参数就是```webpack```工作过程中的核心对象，对象里面包含了此次构建的所有的配置信息，通过这个对象可以注册钩子函数。

这里的需求是希望这个插件可以用来去清除```webpack```打包生成的```js```中没必要的注释，有了这个需求需要明确这个任务的执行时机，也就是要把这个任务挂载到哪个钩子上。

需求是删除```bundle.js```中的注释，也就是说当```bundle.js```文件内容明确后才可以实施相应的动作，在```webpack```的官网的```API```文档中找到```emit```的钩子，这个钩子在```webpack```即将要往输出目录输出文件时执行。

通过```compiler```当的```hooks```属性访问到```emit```钩子，然后通过```tap```方法注册钩子函数，这个方法接收两个参数，第一个参数是插件的名称```MyPlugin```，第二个是需要挂载到这个钩子上的函数。在函数中接收一个```complation```的对象参数，这个对象可以理解成此次打包过程的上下文。

所有打包过程中产生的结果都会放到这个对象中，使用对象的```assets```属性获取即将写入到目录文件中的资源信息```complation.assets```。这是一个对象通过```for in```遍历这个对象，对象当中的键就是每一个文件的名称。然后将这个插件应用到配置当中通过 ```new MyPlugin```的方式把他应用起来。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');

class MyPlugin {
    apply(compiler) {
        console.log('MyPlugin 启动');
        compiler.hooks.emit.tap('MyPlugin', complation => {
            for (const name in complation.assets) {
                console.log(name);
            }
        })
    }
}

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        // publicPath: 'dist/'
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            meta: {
                viewport: 'width=device-width'
            },
            template: './src/index.html'
        }),
        new HtmlWebpackPlugin({
            filename: 'about.html'
        }),
        new CopyWebpackPlugin([
            'public/**'
        ]),
        new MyPlugin()
    ]
}
```

此时打包过程就会输出打包的文件名称，可以通过文件中值的```source```方法来获取文件内容。

```js
class MyPlugin {
    apply(compiler) {
        console.log('MyPlugin 启动');

        compiler.hooks.emit.tap('MyPlugin', complation => {
            for (const name in complation.assets) {
                console.log(assets[name].source());
            }
        })
    }
}
```

拿到文件名和内容后要判断文件是否以```.js```结尾，如果是```js```文件将文件的内容得到然后通过正则的方式替换掉代码当中对应的注释，将替换的结果覆盖到原有的内容当中，要覆盖```complation```当中```assets```里面所对应的属性。这个属性的值同样暴露一个```source```方法用来去返回新的内容。除此之外还需要一个```size```方法，用来返回这个内容的大小，这个是```webpack```内部要求的必须的方法。

```js
class MyPlugin {
    apply(compiler) {
        // console.log('MyPlugin 启动');
        compiler.hooks.emit.tap('MyPlugin', complation => {
            for (const name in complation.assets) {
                // console.log(assets[name].source());
                if (name.endsWith('.js')) {
                    const contents = complation.assets[name].source();
                    const withoutComments = contents.replace(/\/*\**\*\//g, '');
                    complation.assets[name] = {
                        source: () => withoutComments,
                        size: () => withoutComments.length
                    }
                }
            }
        })
    }
}
```

打包过后```bundle.js```每一行开头的注释就被移除掉了，以上就是实现移除```webpack```注释插件的过程，通过这个过程了解，插件是通过往```webpack```生命周期里面的一些钩子函数里面挂载任务函数来去实现的。如果需要深入了解插件机制，可能需要理解一些```webpack```底层的实现原理，通过去阅读源代码来了解他们。

## 15. 开发体验问题

在此之前已经了解了一些```webpack```的相关概念和一些基本的用法，但是以目前的状态去应对日常的开发工作还远远不够，编写源代码再通过```webpack```打包然后运行应用，最后刷新浏览器这种方式过于原始。如果实际的开发过程中还按照这种方式去使用必然会大大降低开发效率。

希望开发环境必须能够使用```http```的服务运行而不是以文件的形式预览，这样一来可以更加接近生产环境的状态，而且使用```ajax```之类的一些```api```也需要服务器环境。其次希望这个环境在修改源代码后```webpack```可以自动完成构建，然后浏览器可以及时的显示最新的结果，这样的话就可以大大的减少在开发过程中额外的重复操作。

最后还需要能够去提供```sourceMap```支持，运行过程中一旦出现了错误就根据错误的堆栈信息快速定位到源代码当中的位置，便于调试应用。

### 1. 自动编译

用命令行手动重复去使用```webpack```命令从而去得到最新的打包结果，这种办法特别的麻烦可以使用```webpack-cli```提供的```watch```工作模式解决这个问题。这种模式项目下的源文件会被监视，一旦这些文件发生变化，会自动重新运行打包任务。

用法非常简单，就是启动```webpack```时添加```--watch```参数。

```s
yarn webpack --watch
```

可以再开启一个新的命令行终端以```http```的形式运行应用。

```s
http-server ./dist
```

此时修改源代码```webpack```就会自动重新打包，可以刷新页面看到最新的页面结果。

### 2. 自动刷新浏览器

如果流浏览器能在编译过后自动去刷新，开发体验将会更好一些，```browser-sync```工具就会实现自动刷新的功能。

```s
yarn add --global browser-sync
```

使用```browser-sync```启动```http```服务同时要监听```dist```文件下的文件变化。此时修改源文件保存过后浏览器会自动刷新然后显示最新的结果。

```s
browser-sync dist --files "**/*"
```

原理是```webpack```自动打包源代码到```dist```当中，```dist```的文件变化被```browser-sync```监听从而实现了自动编译并且自动刷新浏览器。

### 3. 开发服务器

```Webpack Dev Server```提供了一个开发服务器，并且将自动编译和自动刷新浏览器等一系列对开发友好的功能全部集成在了一起。这是一个高度集成的工具使用起来非常的简单。

```s
yarn add webpack-dev-server --dev
```

```s
yarn webpack-dev-server
```

运行命令内部会自动使用```webpack```打包我应用并且会启动一个```http-server```去运行打包结果。还会监听代码变化，一旦源文件发生变化就会自动打包，这一点和```watch```模式是一样的。

```webpack-dev-server```为了提高工作效率并没有将打包结果写入到磁盘中，是将打包结果暂时存放在内存中，内部的```http-server```也是从内存中把这些文件读出来发送给浏览器，这样一来就会减少很多磁盘不必要的读写操作，从而大大提高构建效率。

这个命令可以传入```--open```参数，用于自动唤起浏览器，打开运行地址。

```s
yarn webpack-dev-server --open
```

如果有两块屏幕就可以把浏览器放到另外一块屏幕中，一边编码，一边及时预览开发环境。

### 4. 静态资源访问

静态文件需要作为开发服务器的资源被访问需要额外的去告诉```webpack-dev-server```，具体的方法就是在webpack的配置文件当中添加对应的配置。在配置对象当中添加```dev-server```的属性这个属性专门用来为```webpack-dev-server```指定相关的配置选项。

配置对象的```contentBase```属性用来指定静态资源路径，可以是一个字符串或者是一个数组，也就是说可以配置一个或者是多个路径，这里设置为```public```目录。

之前通过```copy-webpack-plugin```插件将```public```目录输出到了输出目录，正常所有输出的文件都应该可以直接被```server```也就是直接在浏览器端访问到。按道理来讲这些文件不需要再作为开发服务器的额外的资源路径了，但是在实际使用```webpack```的时候一般都会把```copy-webpack-plugin```这插件留在上线前的那次打包中。在平时的开发过程中一般不会去使用它，因为在开发过程中会重复执行打包任务。假设```copy```的文件比较多或者是比较大，每次执行插件的话打包过程中的开销就会比较大速度自然也就会降低了。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
// const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist')
    },
    devServer: {
        contentBase: './public',
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            meta: {
                viewport: 'width=device-width'
            },
            template: './src/index.html'
        }),
        new HtmlWebpackPlugin({
            filename: 'about.html'
        }),
        // new CopyWebpackPlugin(['public'])
    ]
}

```

### 5. API代理

```webpack-dev-server```在启动服务时创建的是一个本地服务，访问地址一般为```localhost:端口号```，而最终上线过后应用一般又和```API```会部署到同源地址下面。这样的话就会有一个非常常见的问题，在实际生产环境当中可以直接访问```API```，开发环境中就会产生跨域请求问题。

解决这个问题最好的办法就是在开发服务器配置代理服务也就是把接口服务代理到本地的开发服务地址，```webpack-dev-server```支持通过配置的方式添加代理服务。

这里的目标就是将```github```的```api```代理到本地的开发服务器当中。```github```的接口的```Endpoint```一般都是在根目录下，例如这里所使用的```user```

```s
https://api.github.com/users
```

```Endpoint```可以理解为接口入口，回到配置文件当中，在```devServer```当中添加```proxy```属性，这个属性就是专门用来添加代理服务配置的，是个对象，每一个属性就是一个代理规则的配置，属性的名称是需要被代理的请求路径前缀，也就是请求以哪一个地址开始，一般为了辨别会将其设置为```/api```，也就是请求开发服务器中的```/api```开头的这种地址都会让他代理到接口当中。

将代理目标设置为```https://api.github.com```, 也就是说当请求```/```时代理目标就是```api.github.com```地址。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
// const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist')
    },
    devServer: {
        contentBase: './public',
        proxy: {
            '/api': {
                target: 'https://api.github.com'
            }
        }
    },
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            meta: {
                viewport: 'width=device-width'
            },
            template: './src/index.html'
        }),
        new HtmlWebpackPlugin({
            filename: 'about.html'
        }),
        // new CopyWebpackPlugin(['public'])
    ]
}

```

此时如果去请求```http://localhost:8080/api/users```就相当于请求了```https://api.github.com/api/users```，意思是请求的路径是什么，最终代理的这个地址路径是完全一致的。

实际上在```api.github.com/users```中并没有```/api/users```所以对于代理路径中的```/api```需要去掉，可以添加```pathRewrite```属性，来去实现代理路径的重写。重写规则就是把路径中以```/api```开头的这段字符串替换为空字符串，```pathRewrite```会以正则的方式替换请求的路径。

还需要设置```changeOrigin```属性为```true```，请求的过程中会带一个主机名这个主机名默认情况下使用的是用户在浏览器端发起请求的这个主机名也就是```localhost:8080```。一般情况下服务器那头是要根据主机名去判断这个请求是属于哪个网站从而把这个请求指派到对应的网站。```localhost:8080```对于```github```的服务器来说是不认识的，所以这里需要```changeOrigin```去修改修改。```changeOrigin=true```会以代理请求的主机名去请求。请求```github```这个地址真正请求的应该是```api.github.com```这样地址，所以主机名会保持原有状态。

```js
{
    contentBase: './public',
    proxy: {
        '/api': {
            target: 'https://api.github.com'.
            pathRewrite: {
                '^/api': ''
            },
            changeOrigin: true
        }
    }
}
```

针对于```changeOrigin```可能不会特别清楚，这是因为在```http```里面有一些相关的知识点可能之前没有了解过，可以查一下```host```也就是主机名相关的概念就可以理解了。

## 16. Source Map

通过构建编译之类的操作可以将开发阶段的源代码转换为能够在生产环境中运行的代码，但是这种进步的同时也意味着在实际生产环境中运行的代码与开发阶段所编写的代码之间会有很大的差异。在这种情况下如果需要调试应用，又或是运行应用的过程中出现了意料之外的错误，将无从下手。```Source Map```就是解决这一类问题最好的办法。

```Source Map```是用来映射转换过后的代码与源代码的关系，一段转换过后的代码通过转换过程中生成的这个```source map```文件可以逆向得到源代码。目前很多第三方库中都会有一个```.map```后缀的```source map```文件，这是一个```JSON```格式的文件里面记录的就是转换过后的代码与转换之前代码之间的映射关系。主要有```version```属性指的是当前文件所使用的的```source map```标准的版本。然后是```source```属性，记录的是转换之前源文件的名称，因为很有可能是多个文件合并转换成了一个文件，所以说属性是一个数组。

```names```属性指的是源代码中使用的一些成员名称，在压缩代码时会将开发阶段编写的有意义的变量替换为一些简短的字符从而去压缩整体代码的体积，这个属性中记录的就是原始对应的名称。```mappings```属性是整个```source map```文件的核心属性，是一个```base64-vl```编码的字符串。这个字符串记录的信息就是转换过后代码中的字符与转换之前所对应的映射关系。

```json
{
    "version": 3,
    "file": "jquery.min.js",
    "sources": [
        "jquery.js"
    ],
    "names": [
        "window",
        "undefined",
        "readyList",
        "rootjQuery",
        "core_strundefined",
        "location",
        "document",
        "docElem",
        "documentElement",
        "_jQuery",
        "jQuery",
        "_$",
        "$",
        "class2type"
        ...
    ],
    "mappings": ";;;CAaA,SAAWA,EAAQC,GAOnB,GAECC,GAGAC,EAIAC,QAA2BH,GAG3BI,EAAWL,EAAOK,SAClBC,EAAWN,EAAOM,SAClBC,EAAUD,EAASE,gBAGnBC,EAAUT,EAAOU,OAGjBC,EAAKX,EAAOY,EAGZC,KAGAC,
    ...
    "
}
```

有了这个文件后一般会在转换过后的代码当中通过注释的方式引入这个```source map```文件。```source map```特性只是帮助开发者更容易调试和定位错误所以对生产环境其实没有什么太大的意义。

```js
//# sourceMappingURL = jquery-3.4.1.min.map
```

在浏览器中如果打开了开发人员工具加载到的这个```js```文件最后发现这行注释就会自动请求这个```source map```文件。然后根据这个文件的内容逆向解析出对应的源代码以便于调试，同时因为有了映射的关系所以说源代码当中如果说出现了错误也很容易定位到源代码当中对应的位置。

### 1. 配置source map

```webpack```可以为打包结果生成对应的```source map```文件，需要使用```devTool```属性就是用来配置开发过程中的辅助工具，也就是与```source map``` 相关的一些功能配置。

```js
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
    },
    devtool: 'source-map',
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            meta: {
                viewport: 'width=device-width'
            },
            template: './src/index.html'
        }),
        new HtmlWebpackPlugin({
            filename: 'about.html'
        }),
        new CopyWebpackPlugin([
            'public/**'
        ])
    ]
}

```

可以直接将这个属性设置为```source-map```，打包完成过后生成的```dist```目录可以发现生成了一个对应的```bundle.js.map```文件。而且```bundle.js```文件的最后也通过注释的方式引入了这个```source-map```文件。

```js
/******/ (function(modules) { // webpackBootstrap
/******/ })
/************************************************************************/
/******/ ([
/******/ ]);
//# sourceMappingURL=bundle.js.map
```

截止到目前```webpack```对```source map```的风格支持了```12```种，每种方式所生成的```source map``` 效果以及生成```source map```的速度，都是不一样的。很简单也很明显的一个道理就是效果最好的生成速度也就会最慢。而速度最快的生成出来的这个```source map```文件也就没有什么效果。

### 2. eval

```eval```是```js```的函数，可以用来运行字符串中的js代码。

```js
eval('console.log(123)');

```

默认情况下这段代码会运行在一个临时的虚拟机环境中。可以通过```source url```声明这段代码所处的文件路径。比如添加一段注释内容就是```//# sourceURL=./foo/bar```，此时这段代他所运行的这个环境就是```./foo/bar.js```。这也就意味着可以通过```sourceURL```改变通过```eval```执行的这段代码所处的环境名称，其实还是运行在虚拟机环境中，只不过他告诉了执行引擎这段代码所属的文件路径，这里只是一个标识而已。

```js
eval('console.log(123) //# sourceURL=./foo/bar.js');
```

了解了这个特点回到配置文件中将```devtool```属性设置为```eval```，也就是使用```eval```模式。找到错误所出现的文件，打开这个文件到的却是打包过后的模块代码。因为在这种模式下，会将每个模块所转换过后的代码都放在```eval```函数中去执行，并且在```eval```函数执行的字符串最后通过```sourceURL```的方式去说明所对应的文件路径。

这样的话浏览器在通过```eval```执行这段代码的时候就知道这段代码所对应的源代码是哪一个文件从而实现定位错误所出现的文件，并且也只能定位文件。

这种模式下不会生成```source map```文件，也就是说，它实际上跟```source-map```没有什么太大的关系，所以说他的构建速度也就是最快的效果也就很简单，只能定位源代码文件的名称不知道具体的行列信息。

### 3. devtool

在```main.js```当中故意加入了一个运行时的错误```console.log111```。

```js
import createHeading from './heading.js'

const heading = createHeading();

document.body.append(heading);

console.log('main.js running');

console.log111('main.js running');
```

打开webpack的配置文件定义一个数组，数组中的每一个成员就是```devtool```配置取值的一种。

```js
const allModes = [
    'eval',
    'cheap-eval-source-map',
    'cheap-module-eval-source-map',
    'eval-source-map',
    'cheap-module-source-map',
    'inline-cheap-source-map',
    'inline-cheap-module-source-map',
    'source-map',
    'inlie-source-map',
    'hidden-source-map',
    'nosource-source-map',
]
```

```webpack```的配置对象可以是一个数组，数组中给的每个元素就是一个单独的打包配置。这样一来就可以在一次打包过程中同时执行多个打包任务。可以通过遍历为每一种模式单独去创建一个打包配置，这样的话就可以在一次打包中同时生成所有模式下的不同结果，方便对比使用。

每个配置项中先定义```devtool```属性，属性的值就是遍历的名称。将```mode```设置为```none```确保```webpack```内部不会去做额外的处理，紧接着设置任务的打包入口，以及输出文件的名称。这里将输出文件的名称设置为模式名称命名的```js```文件。再下面为```js```模块配置```babel-loader```，目的是接下来对比中能够辨别其中一类模式的差异。最后再配置一个```html-webpack-plugin```，也就是为每个打包任务生成一个```html```文件。
```js

const HtmlWebpackPlugin = require('html-webpack-plugin');

const allModes = [
    'eval',
    'cheap-eval-source-map',
    'cheap-module-eval-source-map',
    'eval-source-map',
    'cheap-module-source-map',
    'inline-cheap-source-map',
    'inline-cheap-module-source-map',
    'source-map',
    'inlie-source-map',
    'hidden-source-map',
    'nosource-source-map',
];

module.exports = allModes.map(item => {
    return {
        mode: 'none',
        devtool: item,
        entry: './src/main.js',
        output: {
            filename: `js/${item}.js`,
        },
        module: {
            rules: [
                {
                    test: /\.js$/,
                    use: {
                        loader: 'babel-loader',
                        options: {
                            presets: ['@babel/preset-env']
                        }
                    }
                }
            ]
        },
        plugins: [
            new HtmlWebpackPlugin({
                filename: `${item}.html`
            })
        ]
    }
})
```

打包过后就会生成所有模式下的打包结果，启动服务打开浏览器此时就能在页面中看到所有不同模式下的```html```。有了这些不同模式下的打包结果就可以一个一个仔细对比。

#### 1. eval。

这个模式就是将模块代码放到```eval```函数中执行，并且通过```sourceURL```标注模块文件的路径。这种模式下并没有生成对用的```source-map```只能定位是哪一个文件出了错误。

#### 2. eval-source-map

这个模式同样也是使用```eval```函数执行模块代码，不同的是除了可以定位错误出现的文件。还可以定位到具体的行列信息，这种模式下相比于```eval```生成了```source-map```。

#### 3. cheap-eval-source-map

这个模式就是在```eval-source-map```的基础之上加了一个```cheap```，就是阉割版的```source-map```。虽然也生成了```source-map```，但这种模式下的```source-map```只能定位到行而没有列的信息。少了一点效果所以生成速度自然就会快很多。

#### 4. cheap-module-eval-source-map

这其是```cheap-eval-source-map```模式基础上多了一个```module```，```cheap-module-eval-source-map```定位源代码跟编写的源代码是一模一样的。而```cheap-eval-source-map```显示的是经过```babel```转换过后的结果。如果想要和手写代码一样的源代码，需要选择````cheap-module-eval-source-map````这种模式。

#### 5. cheap-source-map

没有```eval```就爱意味着没有用```eval```的方式执行模块代码，没有```module```也就意味着是```loader```处理过后的代码。

#### 6. inline-source-map

和普通的```source-map```效果上是一样的，只不过```source-map```的模式下文件是以物理文件的方式存在，而```inline-source-map```使用的是```dataurl```的方式去将```source-map```以```dataurl```嵌入到代码中。之前遇到的```eval-source-map```其实也是使用这种行内的方式把```source-map```嵌入进来，会导致这个代码的体积会变大很多。

#### 7. hidden-source-map

这个模式在开发工具中是看不到```source-map```的效果的。但是回到开发工具中会发现确实生成了```source-map```文件。这就跟```jq```是一样的，在构建过程当中，生成了```source-map```文件，但是他在代码当中并没有通过注释的方式去引入这个文件。

这个模式实际上是在开发一些第三方包的时候比较有用，需要生成```source-map```但是不想在代码当中直接去引用他们，一旦在使用时出现了问题，可以再把```source-map```引入回来，或者通过其他的方式使用```source-map```。

```source-map```还有很多其他的使用方式，通过```http```的响应头也可以使用，这里就不再扩展了。

#### 8. nosources-source-map

这个模式下能看到错误出现的位置，但是点击错误信息是看不到源代码的。```nosource```指的就是没有源代码，但是提供了行列信息，这样结合编写的源代码还是可以找到错误出现的位置。只是在开发工具当中看不到源代码而已。这是为了在生产环境中保护源代码不会被暴露的一种情况。

### 4. 选择 Source Map

虽然```webpack```支持各种各样的```SourceMap```模式，但是一般应用开发时也只会用到其中的几种。

#### 1. 开发模式

在开发环境建议选择```cheap-module-eval-source-map```，一般编写代码的风格要求每一行代码不会超过```80```个字符，能够定位到行也就够了因为每一行里面最多也就```80```个字符，很容易找到对应的位置。第二就是使用框架的情况会比较多，以```react```和```vue```来说，无论是使用```jsx```还是vue的单文件组件，```loader```转换过后的代码和转换之前都会有很大的差别，这里需要调试转换之前的源代码，而不是转换过后的，所以要选择有```module```的方式。

第三点就是虽然```cheap-module-eval-source-map```的启动速度会比较慢一些，但是大多说时间我都是在使用```webpack-dev-server```监视模式重新打包，而不是每次都启动打包所以这种模式下重新打包速度比较快。

#### 2. 生产模式

生产环境的打包交易选择选择```none```也就是不生成```source-map```。```source-map```会暴露源代码到生产环境，这样的话但凡有一点技术的人都可以很容易复原项目中绝大多数的源代码，这个点是要注意的。

其次调试和找错误这些都应该是开发阶段的事情，应该在开发阶段就尽可能把所有的问题和隐患都找出来，而不是到了生产环境让全民去公测。

 如果对代码实在没有信心建议选择```nosources-source-map```模式，这样出现错误在控制台当中就可以找到源代码对应的位置，不至于向外暴露源代码内容。

 ## 17. HMR

实际开发中自动刷新并没有想象中那么好用因为每次修改完代码，```webpack```监视到文件变化就会自动打包，然后自动刷新到浏览器，一旦页面整体刷新，那页面中之前的任何操作状态都会丢失，有些人一般会有一些小办法，例如可以在代码中写死一个文本到编辑器中。这样的话即便页面刷新也不会有丢失的情况，又或是通过一些额外的代码，把内容先保存到临时存储中，然后刷新过后再取回来。

这些都是好办法但是又都不是特别的好，因为这些都是典型的有洞补洞的操作，并不能根治页面刷新过后导致的页面数据丢失的问题。而且这些方法都需要编写一些跟业务本身无关的代码，更好的办法自然是能够在页面不刷新的这种情况下代码也可以及时的更新。针对这样的需求```webpack```给出了解决方案。

```HRM（Hot Module Replacement）```翻译过来叫做模块热替换或者叫模块热更新。计算机行业经常听到一个叫做热拔插的名词，指的就是可以在正在运行的机器上随时插拔设备而机器的运行状态不会受插拔设备的影响。例如电脑设备上的```USB```端口。

```webpack```中的模块热替换指的就是可以在应用程序运行的过程中实时的替换掉应用中的某个模块。而应用的运行状态不会因此改变。```HMR```是```webpack```中最强大的特性之一，同时也是最受欢迎的特性。

### 1. 开启 HMR

```HMR```已经集成在了```webpack-dev-server```工具中，使用这个特性需要运行```webpack-dev-server```命令时通过```--hot```参数开启，也可以在配置文件中添加对应的配置来开启。

打开配置文件需要配置的地方有两个，第一个将```dev-server```中的```hot```属性设置为```true```。第二是载入一个```webpack```内置的插件```hot-module-replacement-plugin```。

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    mode: 'development'
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        // publicPath: 'dist/'
    },
    devtool: 'source-map',
    devServer: {
        hot: true
    }
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            template: './src/index.html'
        }),
        new webpack.HotModuleReplacementPlugin()
    ]
}

```

回到开发工具当中修改样式文件，保存过后样式模块就以热更新的方式直接作用到页面中了。但是修改```js```模块页面还是自动刷新的并没有热更新的体验。

样式文件在```style-loader```里面就已经自动处理了热更新，不需要额外做手动的操作。而```js```文件比较复杂，在一个模块中导出的可能是对象，也可能是字符串，还有可能是函数，导出的成员在使用上可能各不相同的。所以```webpack```对毫无规律的```js```模块，并不知道如何处理更新过后的模块，也就没有办法实现通用的替换方案。```webpack```中的```HMR```需要手动通过代码处理当```js```模块更新过后需要如何把更新后的模块替换到运行的页面当中。

可能对使用过```vue-cli```或```create-react-app```脚手架工具的人来说，会觉得项目中并没有手动处理```js```模块的更新，代码照样可以做热替换，这是因为使用框架开发时，项目中每个文件有规律，框架提供的是一些规则，例如```react```模块要求每一个模块必须要导出一个函数，或者是导出一个类。有了这个规律就可能会有通用的替换办法，例如每一个文件导出的都是一个函数的话那就自动把这个函数执行一下。这些工具内部都已经提供好了通用的```HMR```替换模块，不需要自己手动处理。

综上还需要自己手动处理当js模块更新过后需要做的事情。

```Hot-module-replacement-plugin```为js提供了一套用于处理```HMR```的```api```，需要在自己的代码中使用这套```api```来去处理当某个模块更新后应该如何替换。

打开入口文件，这个文中使用了导入的模块，一但导入的模块更新就必须重新使用这些模块。

```module```对象有个```hot```的对象属性是```HMR API```的核心对象，提供了一个```accept```方法用于注册某个模块更新过后的处理函数，方法的第一个参数接收的是依赖模块的路径，第二个参数是更新过后的处理函数。这里注册一下当```editor```模块更新过后的处理函数。。

```js
import createEditor from './editor';
import background from './better.png';
import './global.css';

const editor = createEditor();
document.body.appendChild(editor);

const img = new Image();
img.src = background;
document.body.appendChild(img);

module.hot.accept('./editor', () => {
    console.log('editor 模块更新了，需要这里手动处理');
});
```

启动```webpak-dev-server```当修改```editor```模块代码的时候浏览器的控制台中就会打印消息，而且也不会自动刷新了，也就是说一旦这个模块的更新被手动的处理了，就不会触发自动刷新。

```editor```模块导出的是一个函数，而且这这个函数是创建界面的元素，一但这个函数更新了，界面元素也应该被重新创建。所以这里先直接移除原来的元素，然后再调用```createEditor```创建新的元素追加到页面中。

这里还需要保留之前的状态，在更新之后将状态回填进去，因为这里用的是一个可编辑元素，可以通过```innerHTML```拿到之前所添加的内容，然后创建新元素过后再把它设置到新元素当中。

```js
let lastEditor = editor;
module.hot.accept('./editor', () => {
    // console.log('editor 模块更新了，需要这里手动处理');
    // console.log(createEditor);
    const value = lastEditor.innerHTML;
    document.body.removeChild(lastEditor);
    const newEditor = createEditor();
    newEditor.innerHTML = value;
    document.body.appendChild(newEditor);
    lastEditor = newEditor;
});
```

这就是针对于```js```模块热替换的一个处理过程，注意这不是一个通用的方式，这只适用于当前的```editor.js```模块。通过这个过程能够发现为什么```webpack```的```HMR```需要自己处理```js```模块的热更新，因为不同的模块有不同的逻辑，不同的业务逻辑又导致处理过程肯定也是不同的，```webpack```并没有办法提供一个通用的替换方案。

图片模块的热替换逻辑就简单的多，同样通过```module.hot.accept```方法注册处理函数，在函数中只需要将图片元素的src设置为新的图片路径就可以了。

在图片修改过后图片名会发生变化，这里拿到的是更新之后的文件名。所以直接重新设置图片元素的src就可以实现图片的热替换。

### 2. 注意事项

如果处理热替换的代码有错误是不容易发现的，错误结果会导致页面自动刷新，自动刷新过后页面中的错误信息也会被清除了，这样一来就不容易发现到底是哪里出错了。

这种情况推荐使用```hot only```的方式来解决，因为默认使用的```hot```如果热替换失败会回退使用自动刷新功能，```hot only```则不会。

配置文件中将```dev-server```中的```hot: true```修改为```hotOnly: true```。

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    mode: 'development'
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        // publicPath: 'dist/'
    },
    devtool: 'source-map',
    devServer: {
        hotOnly: true
    }
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'file-loader'
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: 'Webpack Plugin Sample',
            template: './src/index.html'
        }),
        new webpack.HotModuleReplacementPlugin()
    ]
}

```

如果在代码中使用了```HMR```提供的```API```，但是在启动```dev-server```的时候没有开启HMR的选项，此时运行环境中会报出```accept undefined```的错误。因为```module.hot```对象是```HMR```插件提供的，没有开启这个插件也就没有这个对象。解决的办法非常简单，在业务代码应该先判断是否存在```module.hot```对象再去使用它。

可能会有个疑问，代码当中写了很多与业务功能本身无关的代码会不会有影响。```webpack```是处理过这个问题的，打包过后生成的```bundle.js```文件中处理热替换的代码都会被移除掉了，只剩下一个```if(false) {}```的空判断。这种没有意义的判断在代码压缩过后也会自动去掉，所以说不会影响生产环境中的运行状态。

## 18. 不同环境下的配置

创建不同的环境配置的方式主要有两种。第一种是在配置文件中添加相应的判断条件，然后根据环境导出不同的配置。第二种是为不同的环境对应一个配置文件，确保每一个环境下面都会有一个对应的配置文件。

首先来看配置文件中添加判断的方式，```webpack```的配置文件支持导出一个函数，在函数中返回所需要的配置对象。函数接收两个参数，第一个是```env```也就是通过```cli```传递的环境名参数，第二个是```argv```，指运行```cli```过程中所传递的所有参数。

可以将开发模式配置定义在```config```变量中，再去判断一下```env```是不是等于```production```，这里约定生产环境的```env```就是```production```。如果是生产环境的就将mode属性的字段设置为```production```，然后再将```devtool```设置为```false```，再添加```cleanWebpackPlugin```和```CopyWebpackPlugin```这两个插件。

```js
const webpack = require('webpack');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = (env, argv) => {
    const config = {
        entry: './src/main.js',
        output: {
            filename: 'bundle.js',
            path: path.join(__dirname, 'dist'),
            publicPath: 'dist/'
        },
        devtool: 'cheap-eval-module-source-map',
        devServer: {
            hot: true,
            contentBase: 'public'
        },
        module: {
            rules: [
                {
                    test: /.js$/,
                    use: {
                        loader: 'babel-loader',
                        options: {
                            presets: ['@babel/preset-env']
                        }
                    }
                }
                {
                    test: /.css$/,
                    use: [
                        'style-loader',
                        'css-loader'
                    ]
                },
                {
                    test: /.png$/,
                    use: 'url-loader'
                },
                {
                    test: /.html$/,
                    use: {
                        loader: 'html-loader',
                        options: {
                            attrs: ['img:src', 'a:href']
                        }
                    }
                }
            ]
        },
        plugins: [
            new HtmlWebpackPlugin({
                title: 'webpack',
                template: './src/index.html'
            }),
            new webpack.HotModuleReplacementPlugin(),
        ]
    }
    if (env === 'production') {
        config.mode = 'production';
        config.devtool = false;
        config.plugins = [
            ...config.plugins,
            new CleanWebpackPlugin(),
            new CopyWebpackPlugin(['public'])
        ]
    }
    return config;
}
```

```js
yarn webpack

yarn webpack --env producton
```

通过判断环境名参数返回不同的配置对象这种方式只适用于中小型项目，因为一旦项目变得复杂配置文件也会变得复杂起来。所以对于大型的项目还是建议使用不同环境对应不同配置文件的方式实现，一般这种方式至少会有三个```webpack```配置文件。

其中两个用来适配不同环境，另外一个是公共配置，开发环境和生产环境并不是所有的配置都完全不同，需要一个公共的文件来抽象两者之间相同的配置。

首先在项目的跟目录下新建```webpack.common.js```存储公共配置。

```js
module.exports = {
    entry: './src/main.js',
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: 'dist/'
    },
    devtool: 'cheap-eval-module-source-map',
    devServer: {
        hot: true,
        contentBase: 'public'
    },
    module: {
        rules: [
            {
                test: /.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            }
            {
                test: /.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
                test: /.png$/,
                use: 'url-loader'
            },
            {
                test: /.html$/,
                use: {
                    loader: 'html-loader',
                    options: {
                        attrs: ['img:src', 'a:href']
                    }
                }
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: 'webpack',
            template: './src/index.html'
        }),
        new webpack.HotModuleReplacementPlugin(),
    ]
}
```

然后新建```webpack.dev.js```和```webpack.prod.js```分别去用来为开发和生产定义特殊的配置。

生产环境的配置当中(```webpack.prod.js```)先导入公共的配置对象，这里可以使用```webpack-merge```方法把公共配置对象复制到这个配置对象中，通过最后一个对象覆盖公共配置中的一些配置。

```s
yarn webpack-merge --dev
```

```js

const common = require('./webpack.common');
const merge = require('webpack-merge');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
module.exports = merge(common, {
    mode: 'production',
    plugins: [
        new CleanWebpackPlugin(),
        new CopyWebpackPlugin()
    ]
})

```

同理````webpack.dev.js````文件当也可以通过这样一个方式实现一些额外的配置这里就不重复了。

运行```webpack```时需要通过```--config```参数指定所使用的配置文件。

```s
yarn webpack --config webpack.prod.js
```

## 19. DefinePlugin

```webpack4x```中新增的```production```模式内部自动开启了很多通用的优化功能。第一个是一个叫做```define-plugin```的插件，用来为代码注入全局成员。在```production```模式下，默认这个插件就会启用并且往代码当中注入了一个```process.env.NODE_ENV```的常量。很多第三方模块都是通过这个成员判断当前的运行环境，从而去决定是否执行一些操作。

```define-plugin```是一个内置的插件，先要导入```webpack```模块，```plugins```这个数组当中添加这个插件，插件的构造函数接收一个对象，对象中每一个键值都会被注入到代码中。

这里在定义```API_BASE_URL```用来为代码注入```api```服务地址，值是字符串```https://api.github.com```。

```js
const webpack = require('webpack');

module.exports = {
    mode: 'node',
    entry: './src/main.js',
    output: {
        filename: 'bundle.js'
    },
    plugins: [
        new webpack.DefinePlugin({
            API_BASE_URL: 'https://api.github.com'
        })
    ]
}
```

代码中把```API_BASE_URL```打印出来。运行```webpack```打包，可以发现```define-plugin```其实就是把注入成员的值直接替换到了代码中。```define-plugin```的设计并只是用来替换数据，所传递的字符串实际上是一个代码片段，也就是一段符合```js```语法的代码。

```js
const webpack = require('webpack');

module.exports = {
    mode: 'node',
    entry: './src/main.js',
    output: {
        filename: 'bundle.js'
    },
    plugins: [
        new webpack.DefinePlugin({
            API_BASE_URL: JSON.stringify('https://api.github.com')
        })
    ]
}
```

## 20. Tree Shaking

```Tree-shaking```字面意思是摇树，一般伴随着摇树动作树上的枯树枝和树叶就会掉落下来。不过这里摇掉的是代码当中那些没有用到的部分，更专业的叫未引用代码（```dead-code```）。

```webpack```生产模式优化可以自动检测出代码中那些未引用的代码然后移除他们。比如```components.js```文件中导出了一些函数，每一个函数分别模拟一个组件。其中```button```组件函数中，在```return```过后还执行了一个```console.log```语句，很明显这就属于未引用代码。

```js
export const Button = () => {
    return document.createElement('button')
    console.log('dead-code');
}

export const Link = () => {
    return document.createElement('a')
}

export const Heading = level => {
    return document.createElement('h' + level)
}
```

```index.js```文件中导入了```components```，只是导入了```components```当中的```button```这成员。这就会导致代码中很多的地方用不到，这些用不到的地方对于打包结果是冗余的，```Tree-shaking```的作用就是移除这些冗余的代码。

```js
import { Button } from './components'

document.body.appendChild(Button())

```

```s
yarn webpack --mode production
```

打包完成```bundle.js```中可以看到冗余的代码根本就没有输出，```tree-shaking```这个功能会在生产模式下自动开启。

需要注意的是```tree-shaking```并不是```webpack```中某一个配置选项，他是一组功能搭配使用过后的效果。

回到命令行终端运行```webpack```打包，不过这一次使用```none```也就是不开启任何内置功能和插件。打包完成过后输出的```bundle.js```文件中```link```函数和```heading```函数虽然外部并没有使用，但仍然是导出了。

```s
yarn webpack
```

很明显这些导出是没有意义的，可以借助一些优化功能去掉，打开```webpack```的配置文件添加```optimization```的属性。这个属性是集中配置```webpack```内部的一些优化功能。可以先开启```usedExports```选项，表示在输出结果中只导出那些外部使用了的成员。

```js
module.exports = {
    mode: 'none',
    entry: './src/index.js',
    output: {
        filename: 'bundle.js'
    },
    optimization: {
        usedExports: true
    }
}
```

此时```components```模块所对应的函数就不会导出```link```和```heading```两个函数了，可以开启```webpack```的代码压缩功能，压缩掉这些没有用到的代码。配置文件在```optimization```中开启```minimize```。

```js
module.exports = {
    mode: 'none',
    entry: './src/index.js',
    output: {
        filename: 'bundle.js'
    },
    optimization: {
        usedExports: true,
        minimize: true
    }
}
```

此时```bundle.js```中未引用的代码就都被移除掉了，这就是```tree-shaking```的实现。整个过程用到了两个优化功能，一个是```usedExports```另一个是```minimize```。

如果说真的把代码看做一棵大树的话，可以理解成```usedExports```就是用来在这个大树上标记哪些是枯树叶枯树枝，然后```minimize```就是负责把这些枯树叶树枝全都摇下来。

普通的打包结果是将每一个模块放在一个单独的函数中，如果模块很多也就意味着在输出结果中会有很多模块函数。```concatenteModules```可以将所有的模块打包到一个函数中。配置文件中开启```concatenateModules```, 为了可以更好的看到效果先去关掉```minimize```重新打包。

```js
module.exports = {
    mode: 'none',
    entry: './src/index.js',
    output: {
        filename: 'bundle.js'
    },
    optimization: {
        usedExports: true,
        concatenateModules: true,
        // minimize: true
    }
}
```

此时```bundle.js```中就不再是一个模块对应一个函数了，而是把所有的模块都放到了同一个函数当中，```concatnateModules```的作用就是尽可能将所有的模块全部合并到一起然后输出到一个函数中。这样的话既提升了运行效率又减少了代码体积。这个特性被称之为```Scope Hoisting```也就是作用域提升，他是```webpack3```中添加的特性，如果此时再去配合```minimize```代码体积就会又减小很多。

由于早期```webpack```早期发展非常快，变化也就比较多，所以去找资料时得到的结果并不一定适用于当前所使用的版本，对于```tree-shaking```的资料更是如此。很多资料中都表示如果使用了```babel-loader```就会导致```tree-shaking```失效。针对于这个问题统一说明一下。

首先需要明确一点的是```tree-shaking```的实现，他的前提必须要使用```ES Modules```组织代码，也就是交给```webpack```处理的代码必须使用```ES Modules```的方式实现的模块化。

因为```webpack```在打包所有模块之前，先是将模块根据配置交给不同的```loader```处理，最后将所有```loader```处理过后的结果打包到一起。为了转换代码中```ECMAScript```的新特性很多时候我会选择```babel-loader```处理```js```。而在```babel```转换代码时有可能处理掉代码中```ES Modules```转换成```Commonjs```，当然这取决于有没有使用转换```ES Modules```的插件。

例如在项目中所使用的的```@babel/preset-env```插件集合里面就有这个插件，所以说```preset-env```插件集合工作的时候，代码中```ES Modules```的部分会被转换成```Commonjs```。```webpack```再去打包时拿到的代码是以```Commonjs```的方式组织的代码所以```tree-shaking```不能生效。

为了可以更容易分别结果，这里只开启```usedExports```，重新打包查看```bundle.js```。

```js
module.exports = {
    mode: 'none',
    entry: './src/index.js',
    output: {
        filename: 'bundle.js'
    },
    modules: {
        rules: [
            {
                test: /\.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            }
        ]
    },
    optimization: {
        usedExports: true,
        // concatenateModules: true,
        // minimize: true
    }
}
```

这里结果并不是像刚刚说的那样，```usedExports```功能正常工作了，也就说明如果开启压缩代码的话，那这些未使用的代码依然会被移除```tree-shaking```并没有失效。

在最新版本的```babel-loader```中已经关闭了```ES Modules```转换的插件，可以在```node_modules```中找到```babel-loader```模块，他在```injectcaller```这个文件中已经标识了当前的环境是支持```ES Modules```的。然后再找到的```preset-env```模块，在```200```多行可以发现，这里根据```injectcaller```中的标识禁用了```ES Module```的转换。

所以webpack最终打包时得到的还是ES Modules代码，tree-shaking自然也就可以正常工作了，当然这里只是定位的找到了源代码当中相关的一些信息。如果需要仔细了解这个东西的话可以去翻看一下babel的源代码。

可以尝试在babel的preset配置当中强制开启这个插件来去试一下。不过给preset添加配置的方式比较特别，需要把预设这个数组中的成员再次定义成一个数组，然后这个数组当中的第一个成员就是所使用的的preset的名称。第二个成员是给这个preset定义的对象，这里不能搞错是数组套数组。

将对象的```modules```属性设置为```commonjs```，默认这个属性值是```auto```也就是根据环境去判断是否开启```ES Module```插件，设置为```commonjs```也就表示需要强制使用```babel```的```ES Modules```插件把代码中的```ES Moudle```转换为```commonjs```。

```js
module.exports = {
    mode: 'none',
    entry: './src/index.js',
    output: {
        filename: 'bundle.js'
    },
    modules: {
        rules: [
            {
                test: /\.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            ['@babel/preset-env', { modules: 'commonjs'}],
                        ]
                    }
                }
            }
        ]
    },
    optimization: {
        usedExports: true,
        // concatenateModules: true,
        // minimize: true
    }
}
```

打包后就会发现刚刚所配置的```usedExports```没办法生效了。即便开启压缩代码```tree-shaking```也是没有办法正常工作的。

## 21. sideEffects

```weboack4```中还新增了一个叫做```sideEffects```的新特性。允许通过配置的方式标识代码是否有副作用，从而为```tree-shaking```提供更大的压缩空间。副作用是指模块执行时候除了导出成员是否还做了一些其他的事情，这样一个特性一般只有在开发npm模块时才会用到。

因为官网中把```sideEffects```的介绍跟```tree-shaking```混到了一起，所以很多人误认为他俩是因果关系。其实他俩真的没有那么大的关系。

这里设计一下能够让```side Effects```发挥效果的一个场景，基于上面的案例把```components```拆分出了多个组件文件。然后在```index.js```中集中导出便于外界导入。回到入口文件中导入```components```中的```button```成员。

这样就会出现一个问题，因为在这载入的是```components```目录下的```index.js```，```index.js```中又载入了所有的组件模块。这就会导致只想载入```button```组件，但是所有的组件模块都会被加载执行。查看打包结果会发现所有组件的模块确实都被打包了，```side effects```特性就可以用来解决此类问题。

打开```webpack```的配置文件在```optimization```中开启属性```sideEffects: true```，这个特性在```production```模式下会自动开启。

```js
{
    optimization: {
        sideEffects: true
        // usedExports: true,
        // concatenateModules: true,
        // minimize: true
    }
}
```

开启过后```webpack```在打包时就会先检查当前代码所属的这个```package.json```当中有没有```sideEffects```的标识，以此来判断这个模块是否有副作用。

如果说这个模块没有副作用，那这些没有用到的模块就不再会打包。可以打开```package.json```添加```sideEffects```的字段设置为```false```。

```json
{
    "sideEffects": false
}
```


打包过后查看```bundle.js```文件，此时那些没有用到的模块就不再会被打包进来了，这就是```sideEffects```的作用。使用```sideEffects```功能的前提是确定代码没有副作用，否则在```webpack```打包时就会误删掉那些有副作用的代码。

例如有一个```extend.js```文件。在这个文件中没有向外导出任何成员。仅仅是在```Number```这个对象的原型上挂载了一个```pad```方法，用来为数字去添加前面的导```0```。

```js
Number.prototype.pad = functuon(size) {
    let result = this + '';
    while (result.lengtj < size>) {
        result = '0' + result
    }
    return result;
}
```

回到入口文件导入```extend.js```，然后就可以使用他为```Number```所提供的扩展方法。

```js
import './extend.js';

console.log((8).pad(3));
```

这里为```Number```做扩展方法的操作就属于```extend```模块的副作用，因为在导入了这个模块过后，```Number```的原型上就多了一个方法这就是副作用。

此时如果还标识项目中所有代码都没有副作用的话，打包之后就会发现刚刚的扩展操作是不会被打包进来的。除此之外还有再代码中载入的```css```模块，也都属于副作用模块，同样会面临刚刚这样一种问题。

解决的办法就是在```package.json```中关掉副作用标识，或者是标识一下当前这个项目当中哪一些文件是有副作用的，这样的话webpack就不会去忽略这些有副作用的模块了。

可以打开```package.json```把```sideEffects```的```false```改成一个数组。然后添加```extend.js```文件路径，还有```global.css```文件的路径，当然了这里可以使用路径通配符的方式来去配置。```*.css```。

```json
{
    "sideEffects": [
        "./src/extend.js",
        "./src/global.css"
    ]
}
```

## 22. 代码分割

合理的打包方案应该是把打包结果按照一定的规则分离到多个```bundle```当中，根据应用的运行按需加载这些模块。这样的话就可以大大提高应用的响应速度以及运行效率。为了解决这样的问题```webpack```支持分包的功能，通过把模块按照所设计的规则打包到不同的```bundle```当中，从而提高应用的响应速度。

目前```webpack```实现分包的方式主要有两种，第一种是根据业务去配置不同的打包入口，也就是会有多个打包入口同时打包，这时候就会输出多个打包结果。第二种是采用```ES Module```的动态导入功能实现模块的按需加载，这个时候```webpack```会自动的把需要动态导入的这个模块单独的输出到一个```bundle```中。

### 1. 多入口打包

多入口打包一般适用于传统的多页应用程序，最常见的划分规则就是一个页面对应一个打包入口，对于不同页面之间的公共部分，再去提取到公共的结果当中。这种方式使用起来非常简单。

这里有两个页面分别是```index```和```album```，```index.js```负责实现```index```页面所有功能，```album.js```负责实现```album```页面所有功能，```global.css```和```fetch.js```都是公共部分。

一般配置文件中的```entry```属性只会配置一个文件名路径，也就是说只会配置一个打包入口，如果我们配置多个入口的话，可以把```entry```定义成一个对象，对象中一个属性就是一路入口，那我们属性名就是这个入口的名称，值就是这个入口所对应的文件路径。

一但这里配置为多入口，输出的文件名也需要修改，两个入口也就意味着会有两个打包结果，可以为filename属性添加```[name]```动态输出文件名最终就会被替换成入口的名称```index```和```album```。

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = allModes.map(item => {
    return {
        mode: 'none',
        entry: {
            index: './src/index.js',
            album: './src/album.js',
        },
        output: {
            filename: `[name].bundle.js`,
        },
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        'style-loader',
                        'css-loader'
                    ]
                }
            ]
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: './src/index.html',
                filename: `index.html`
            })
            new HtmlWebpackPlugin({
                template: './src/album.html',
                filename: `album.html`
            })
        ]
    }
})
```

配置文件当中输出html的插件需要指定输出的html所使用的bundle，可以使用```chunk```属性设置，每一个打包入口就会形成一个独立的```chunk```，分别为这两个页面配置不同的```chunk```。

```js
[
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
        template: './src/index.html',
        filename: `index.html`,
        chunk: ['index']
    })
    new HtmlWebpackPlugin({
        template: './src/album.html',
        filename: `album.html`,
        chunk: ['album']
    })
]
```

### 2. 提取公共模块

多入口打包本身非常容易理解也非常容易使用，但是也存在一个小问题，不同的打包入口中一定会有一些公共的部分。例如```index```入口和```album```入口中就共同使用了```global.css```和```fetch.js```这两个公共的模块。所以需要把这些公共的模块提取到一个单独的```bundle```当中。webpack中实现公共模块提取的方式非常简单，只需要在优化配置中开启```splitChunks```的功能就可以了。

配置文件中在```optimization```中添加```splitChunks```属性，这个属性需要配置```chunks```属性设置为```all```，表示会把所有的公共模块都提取到单独的```bundle```当中。

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = allModes.map(item => {
    return {
        mode: 'none',
        entry: {
            index: './src/index.js',
            album: './src/album.js',
        },
        output: {
            filename: `[name].bundle.js`,
        },
        optimization: {
            splitChunks: {
                chunks: 'all'
            }
        }
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        'style-loader',
                        'css-loader'
                    ]
                }
            ]
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: './src/index.html',
                filename: `index.html`,
                chunk: ['index']
            })
            new HtmlWebpackPlugin({
                template: './src/album.html',
                filename: `album.html`,
                chunk: ['album']
            })
        ]
    }
})
```

打包后我```dist```目录下就会生成额外的一个```js```文件，在这个文件中就是```index```和```album```这两个入口公共的模块部分了。

### 3. 动态导入

按需加载是开发浏览器应用中非常常见的需求，一般常说的按需加载指的是加载数据。那这里所说的按需加载指的是应用运行过程中需要某个模块时才去加载这个模块，这种方式可以极大地节省我们的带宽和流量。

```webpack```中支持使用动态导入的方式实现模块的按需加载，所有动态导入的模块都会自动被提取到单独的```bundle```中从而实现分包，相比于多入口的这种方式动态导入他更为灵活，可以通过代码的逻辑控制需不需要加载某个模块，或者是什么时候加载某个模块。分包的目的就是要让模块实现按需加载来提高应用的响应速度。

这里设计好了一个可以发挥按需加载作用的场景，在这个页面的主体区域，如果访问的是文章页的话得到的就是一个文章列表，如果访问的是相册页面显示的就是相册列表。文章列表所对应的是```post```组件，相册列表对应的是```album```组件，在打包入口中同时导入这两个模块。这里的逻辑是当锚点发生变化时，根据锚点的值决定显示哪个组件。

动态导入使用的就是```ES Module```标准当中的动态导入，需要动态导入的地方通过```import```函数导入指定的路径。这个方法返回的是一个```Promise```，在```Promise```的```then```方法中就可以拿到模块对象。

```js
// import posts from './posts/posts';
// import album from './album/album';

const render = () => {
    const hash = locaton.hash || '#posts';

    const mainElement = document.querySelector('.main');

    mainElement.innerHTML = '';

    if (hash === '#posts') {
        // mainElement.appendChild(posts());
        import('./posts/posts').then(({ default: posts}) => {
            mainElement.appendChild(posts());
        })
    } else if (hash === '#album') {
        // mainElement.appendChild(album());
        import('./album/album').then(({ default: album}) => {
            mainElement.appendChild(album());
        })
    }
}

render();

window.addEventListener('hashchange', render);
```

打包过后```dist```目录会多出三个```js```文件，这三个```js```文件实际上就是由动态导入自动分包所产生的。这三个文件分别是刚刚导入的两个模块以及这两个模板当中公共的部分所提取出来的```bundle```。整个过程无需配置任何一个地方只需要按照```ES Module```动态导入成员的方式去导入模块就可以了```webpack```内部会自动处理分包和按需加载。

### 4. 魔法注释

默认通过动态导入产生的```bundle```文件，他的名称只是一个序号，这并没有什么不好的，因为在生产环境大多数时候是不用关心资源文件的名称是什么的。但是如果你需要给这些```bundle```命名可以使用```webpack```所特有的模板注释来去实现。

具体的使用就是在调用```import```函数的参数位置添加一个行内注释，这个注释有一个特定的格式```/* webpackChunkName: 名称 */```这样的话就可以给分包所产生的```bundle```起上名字了。

```js
// import posts from './posts/posts';
// import album from './album/album';

const render = () => {
    const hash = locaton.hash || '#posts';

    const mainElement = document.querySelector('.main');

    mainElement.innerHTML = '';

    if (hash === '#posts') {
        // mainElement.appendChild(posts());
        import(/* webpackChunkName: posts */'./posts/posts').then(({ default: posts}) => {
            mainElement.appendChild(posts());
        })
    } else if (hash === '#album') {
        // mainElement.appendChild(album());
        import(/* webpackChunkName: album */'./album/album').then(({ default: album}) => {
            mainElement.appendChild(album());
        })
    }
}

render();

window.addEventListener('hashchange', render);
```

如果```chunkName```是相同的，那相同的```chunkName```最终就会被打包到一起。

```js
// import posts from './posts/posts';
// import album from './album/album';

const render = () => {
    const hash = locaton.hash || '#posts';

    const mainElement = document.querySelector('.main');

    mainElement.innerHTML = '';

    if (hash === '#posts') {
        // mainElement.appendChild(posts());
        import(/* webpackChunkName: components */'./posts/posts').then(({ default: posts}) => {
            mainElement.appendChild(posts());
        })
    } else if (hash === '#album') {
        // mainElement.appendChild(album());
        import(/* webpackChunkName: components */'./album/album').then(({ default: album}) => {
            mainElement.appendChild(album());
        })
    }
}

render();

window.addEventListener('hashchange', render);
```

## 23. MiniCssExtractPlugin

```MiniCssExtractPlugin```是一个可以将```css```代码从打包结果中提取出来的插件，通过这个插可以实现```css```模块的按需加载。

```s
yarn add mini-css-extract-plugin
```

打开```webpack```的配置文件需要先导入这个插件，导入过后可以将这个插件添加到配置对象的```plugins```数组中。```MiniCssExtractPlugin```在工作时会自动提取代码中的```css```到一个单独文件中。

除此以外目前所使用的样式模块是先交给```css-loader```解析，然后再交给```style-loader```处理。```style-loader```的作用就是将样式代码通过```style```标签的方式注入到页面当中，从而使样式可以工作。使用```MiniCssExtractPlugin```的话，样式会单独存放到文件当中也就不需要```style```标签，而是直接通过```link```的方式去引入，所以就不再需要```style-loader```了，取而代之的是```MiniCssExtractPlugin```中提供一个```loader```，实现样式文件通过```link```标签的方式去注入。

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
module.exports = allModes.map(item => {
    return {
        mode: 'none',
        entry: {
            main: './src/index.js',
        },
        output: {
            filename: `[name].bundle.js`,
        },
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        MiniCssExtractPlugin.loader,
                        // 'style-loader',
                        'css-loader'
                    ]
                }
            ]
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: './src/index.html',
                filename: `index.html`
            }),
            new MiniCssExtractPlugin()
        ]
    }
})
```

重新打包```dist```目录中看到提取出来的样式文件了。如果样式文件体积不是很大的话不建议提取他到单个文件当中，一般```css```文件超过了```150kb```左右才需要考虑是否将他提取到单独文件当中，否则```css```嵌入到代码当中减少了一次请求效果可能会更好。

## 24. OptimizeCssAssetsWebpackPlugin

使用```MiniCssExtractPlugin```过后样式文件可以被提取到单独的```cs```s文件中但是这里同样会有一个小问题。命令行以生产模式去运行打包。

```s
yarn webpack --mode production
```

那按照之前的了解，在生产模式下```webpack```会自动压缩输出的结果。但是这里会发现样式文件根本没有任何的变化。这是因为```webpack```内置的压缩插件，紧紧是针对于```js```文件压缩，对于其他资源文件压缩，需要额外的插件支持。```webpack```官方推荐```optimize-css-assets-webpack-plugin```压缩样式文件。

```s
yarn add optimize-css-assets-webpack-plugin
```

配置文件中需要先导入这个插件，导入过后把这个插件添加到```plugins```数组当中。

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
module.exports = allModes.map(item => {
    return {
        mode: 'none',
        entry: {
            main: './src/index.js',
        },
        output: {
            filename: `[name].bundle.js`,
        },
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        MiniCssExtractPlugin.loader,
                        // 'style-loader',
                        'css-loader'
                    ]
                }
            ]
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: './src/index.html',
                filename: `index.html`
            }),
            new MiniCssExtractPlugin(),
            new OptimizeCssAssetsWebpackPlugin()
        ]
    }
})
```

这时打包的样式文件就以压缩文件的格式输出了。不过这里还有一个额外的小点，在官方文档中这个插件并不是配置在```plugins```数组中的而是添加到```optimization```属性的```minimize```属性中。如果把这个插件配置到```plugins```数组中，这个插件在任何情况下都会正常工作。而配置在```minimize```数组中只会在```minimize```特性开启时才会工作。

```webpack```建议这种压缩类的插件应该配置到```minimize```数组当中，以便于可以通过```minimize```选项统一控制。

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
module.exports = allModes.map(item => {
    return {
        mode: 'none',
        entry: {
            main: './src/index.js',
        },
        output: {
            filename: `[name].bundle.js`,
        },
        optimization: {
            minimize: [
                new OptimizeCssAssetsWebpackPlugin()
            ]
        },
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        MiniCssExtractPlugin.loader,
                        // 'style-loader',
                        'css-loader'
                    ]
                }
            ]
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: './src/index.html',
                filename: `index.html`
            }),
            new MiniCssExtractPlugin(),
            new OptimizeCssAssetsWebpackPlugin()
        ]
    }
})
```

此时如果没有开启压缩功能这个插件就不会工作，如果说我们以生产模式打包```minimize```属性就会自动开启，压缩插件就会自动工作。

但是这么配置也有一个小小的缺点，看一下输出的```js```文件会发现，原本可以自动压缩的```js```确不能自动压缩了。这是因为这里设置了```minimize```数组，```webpack```认为如果配置了这个数组，就是要自定义所使用的的压缩器插件内部的```js```压缩器就会被覆盖掉，所以这里需要手动的把他添加回来。内置的```js```压缩插件叫做```terser-webpack-plugin```需要安装这个模块，

```s
yarn add terser-webpack-plugin --dev
```

安装完成后把这个插件手动的添加到```minimize```数组中。

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
const TerserWebpackPlugin = require('terser-webpack-plugin');
module.exports = allModes.map(item => {
    return {
        mode: 'none',
        entry: {
            main: './src/index.js',
        },
        output: {
            filename: `[name].bundle.js`,
        },
        optimization: {
            minimize: [
                new OptimizeCssAssetsWebpackPlugin(),
                new TerserWebpackPlugin()
            ]
        },
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        MiniCssExtractPlugin.loader,
                        // 'style-loader',
                        'css-loader'
                    ]
                }
            ]
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: './src/index.html',
                filename: `index.html`
            }),
            new MiniCssExtractPlugin(),
            new OptimizeCssAssetsWebpackPlugin()
        ]
    }
})
```

## 25. Hash文件名

一般部署前端资源时会启用服务器的静态资源缓存，这样对用户的浏览器而言，可以缓存住应用中的静态资源。后续就不再需要请求服务器。整体应用的响应速度就有大幅度的提升。

不过开启静态资源的客户端缓存，会有一些小小的问题，如果缓存策略中缓存失效时间设置的过短效果就不是特别明显。如果过期时间设置的比较长，一但应用发生了更新又没有办法及时更新到客户端。

为了解决这样一个问题建议在生产模式下，给输出的文件名中添加```Hash```值，这样一旦资源文件发生改变，文件名称也可以跟着一起变化。

对于客户端而言，全新的文件名就是全新的请求，那也就没有缓存的问题，可以把服务端缓存策略的缓存时间设置的非常长，也不用担心文件更新过后的问题。

```webpack```中的```filename```属性和绝大多数插件的```filename```属性都支持通过占位符的方式为文件名设置```hash```，不过这里支持三种```hash```效果各不相同。

普通的```hash```可以通过```[hash]```设置，这个```hash```实际上是整个项目级别的，也就是说一旦项目中任何一个地方发生改动，打包过程中的```hash```值都会发生变化。

```js

const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
const TerserWebpackPlugin = require('terser-webpack-plugin');
module.exports = allModes.map(item => {
    return {
        mode: 'none',
        entry: {
            main: './src/index.js',
        },
        output: {
            filename: `[name]-[hash].bundle.js`,
        },
        optimization: {
            minimize: [
                new OptimizeCssAssetsWebpackPlugin(),
                new TerserWebpackPlugin()
            ]
        },
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        MiniCssExtractPlugin.loader,
                        // 'style-loader',
                        'css-loader'
                    ]
                }
            ]
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: './src/index.html',
                filename: `index.html`
            }),
            new MiniCssExtractPlugin({
                filename: '[name]-[hash].bundle.css'
            }),
        ]
    }
})

```

其次是```chunkhash```，这个```hash```是```chunk```级别的，只要是同一路的打包```chunkhash```都是相同的。

```js

const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
const TerserWebpackPlugin = require('terser-webpack-plugin');
module.exports = allModes.map(item => {
    return {
        mode: 'none',
        entry: {
            main: './src/index.js',
        },
        output: {
            filename: `[name]-[chunkhash].bundle.js`,
        },
        optimization: {
            minimize: [
                new OptimizeCssAssetsWebpackPlugin(),
                new TerserWebpackPlugin()
            ]
        },
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        MiniCssExtractPlugin.loader,
                        // 'style-loader',
                        'css-loader'
                    ]
                }
            ]
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: './src/index.html',
                filename: `index.html`
            }),
            new MiniCssExtractPlugin({
                filename: '[name]-[chunkhash].bundle.css'
            }),
        ]
    }
})

```

这里虽然只配置了一个打包入口```index```，但是在代码中通过动态导入的方式分别形成了两路```chunk```分别是```posts```和```album```。样式文件是从代码中单独提取出来的，所以他并不是单独的```chunk```，```main```、```posts```、```album```三者```chunkhash```各不相同。

```css```和所对应的```js```文件他们二者的```chunkhash```是完全一样的因为他们是同一路。当```index```发生修改重新打包会发现，只有```main.bundle```文件名发生了变化，其他的文件都没有变。在```posts.js```文件中做一些修改，输出的```js```和```css```都会发生变化，因为他们是属于同一个```chunk```。

至于```main.bundle```也发生变化的原因是```posts```所生成的```js```文件和```css```文件的文件名发生了变化，入口文件中引入他们的路径也会发生变化所以```mian.chunk```算是被动的改变。

```contenthash```是文件级别的```hash```，根据输出文件的内容生成的```hash```值，也就是说只要是不同的文件就有不同的```hash```值。

```js

const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
const TerserWebpackPlugin = require('terser-webpack-plugin');
module.exports = allModes.map(item => {
    return {
        mode: 'none',
        entry: {
            main: './src/index.js',
        },
        output: {
            filename: `[name]-[contenthash].bundle.js`,
        },
        optimization: {
            minimize: [
                new OptimizeCssAssetsWebpackPlugin(),
                new TerserWebpackPlugin()
            ]
        },
        module: {
            rules: [
                {
                    test: /\.css$/,
                    use: [
                        MiniCssExtractPlugin.loader,
                        // 'style-loader',
                        'css-loader'
                    ]
                }
            ]
        },
        plugins: [
            new CleanWebpackPlugin(),
            new HtmlWebpackPlugin({
                template: './src/index.html',
                filename: `index.html`
            }),
            new MiniCssExtractPlugin({
                filename: '[name]-[contenthash].bundle.css'
            }),
        ]
    }
})

```

那相比于前两者```contenthash```是解决缓存问题最好的方式，因为他精确的定位到了文件级别的```hash```，只有当这个文件发生了变化才有可能更新文件名，实际上是最适合解决缓存问题的。

```webpack```允许指定```hash```的长度，可以在占位符里面通过冒号跟一个数组```[:8]```的方式指定```hash```的长度。

```js
new MiniCssExtractPlugin({
    filename: '[name]-[contenthash:8].bundle.css'
})
```

总的来说如果是控制缓存```8```位的```contenthash```应该是最好的选择了。

## 26. 使用技巧

polyfill会把js注入到全局变量中，污染全局，适合在项目中，不适合在插件和工具库中使用。plugin-transform-runtime以闭包的方式注入不会影响全局，适合工具库。

```s
npm i @babel/plugin-transform-runtime --save;
npm i @babel/runtime --save;
```

```json
{
    loader: "babel-loader",
    options: {
    plugin: ["@babel/plugin-transform-runtime", {
        corejs: 
    }]
    }
}
```

## 27. 手写打包原理

出家门和口岸```buidile.js```文件，引入```fs```模块用于处理文件，创建```modules```函数用于分析模块内容。

```js
const fs = require('fs');
const modules = function(entry) { // 接收一个入口文件
    const content = fs.readFileSync(entry, 'utf-8');
    console.log(content);
}

modules("./src/index.js");
```

这里分析的是```./src/index.js```文件，可以打印其中的内容看下结果、

首先需要拿到文件中的依赖模块，不推荐使用字符串截取，引入的模块名越多就会越麻烦，字符串截取并不灵活，这里推荐使用```@babel/parser```的```babel```的工具，来分析内部的语法，返回的是一个```ast```抽象语法树。

```s
npm install @babel/parser  --save
```

```s
const fs = require('fs');
const parser = require('@babel/parser');

const modules = filename => {
    const content = fs.readFileSync(filename, 'utf-8');
    const Ast = parser.parse(content, {
        sourceType: "module"
    });
    console.log(Ast.program.body);
}

modules('./index.js');
```

接下来可以根据```body```里面的分析结果，遍历出所有引入的模块，```babel```推荐```@babel/traverse```模块来处理。

```js
const fs = require('fs');
const path = require('path');
const traverse = require("@babel/traverse").default;
const parser = require('@babel/parser');

const modules = filename => {
    const content = fs.readFileSync(filename, 'utf-8');
    const Ast = parser.parse(content, {
        sourceType: "module"
    });
    const dependencies = {};
    traverse(Ast, {
        ImportDeclaration({node}) {
            const newfile = './' + path.join(path.dirname(filename), node.source.value);
            // dependencies.push(node.source.value);
            dependencies[node.source.value] = newfile;
        }
    });
    console.log(dependencies);
}

modules('./index.js');
```

可以将代码处理成浏览器可运行的代码，需要借助```@babel/core```和```@babel/preset-env```把```ast```语法树转换成合适的代码。

```js
const babel = require('@babel/core');
const { code } = babel.transformFromAst(Ast, null, {
    presets: ['@babel/preset-env']
});
return {
    entry,
    dependencies,
    code
}

```

大概思路基本清晰了，下面整体安排一下。使用```modules```函数引入```filename```文件，通过```fs.readFileSync```读取文件内容，将文件内容使用```@babel/parser```转换成```ast```，通过```traverse```遍历出依赖的模块存放在```dependencies```对象中，最后将```ast```转换为```ES5```的代码。

```js
const fs = require('fs');
const path = require('path');
const traverse = require("@babel/traverse").default;
const parser = require('@babel/parser');
const babel = require('@babel/core');

const modules = filename => {
    const content = fs.readFileSync(filename, 'utf-8');
    const Ast = parser.parse(content, {
        sourceType: "module"
    });
    const dependencies = {};
    traverse(Ast, {
        ImportDeclaration({node}) {
            const newfile = './' + path.join(path.dirname(filename), node.source.value);
            // dependencies.push(node.source.value);
            dependencies[node.source.value] = newfile;
        }
    });
    
    const { code } = babel.transformFromAst(Ast, null, {
        presets: ['@babel/preset-env']
    });
    return {
        filename,
        dependencies,
        code
    }
}

const info = modules('./index.js');

```

把项目中的所有依赖都分析出来。```allModules```函数运行的时候会调用```modules```函数获取到源代码和依赖的模块，通过遍历将依赖的模块依次再传入```modules```函数，最终可以得到虽有的模块源代码以及他们之间的依赖关系。

```js
const allModules(filename) {
    const entryModule = modules(filename);
    // 使用队列存储所有
    const yilaiArr = [entryModule];
    for (let i = 0; i < yilaiArr.length; i++) {
        const item = yilaiArr[i];
        const { dependencies } = item;
        if (dependencies) {
            for (ket k in dependencies) {
                yilaiArr.push(modules(dependencies[j]));
            }
        }
    }
    const obj = {};
    yilaiArr.forEach(item => {
        obj[item.filename] = {
            dependencies: item.dependencies,
            code: item.code
        }
    })
    return obj;
}
```

在最外层```getCode```函数中传入入口文件，```getCode```函数会将文件路径传递给```allModules```，从而获取所有模块和模块源代码，最后在```getCode```函数中返回一段可执行的```js```用于代码启动。

```js
const getCode = function(filename) {
    consyt bundleInfo = JSON.stringfiy(allModules(filename));
    return `
        (function(all)) {
            function require(module) {
                function localPath(path) {
                    return require(all[module].dependencies[path]);
                }
                const exports = {};
                (function(require, code){
                    eval(code);
                })(localPath, exports, all[module].code);
                return exports;
            }
            // 需要用字符串包裹一下参数
            require('${filename}');
        })(${bundleInfo})
    `;
}
const info = getCode('./src/index.js');
```
