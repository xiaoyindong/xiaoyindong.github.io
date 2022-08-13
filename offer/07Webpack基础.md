模块化很好的解决了复杂应用开发中的代码组织问题，使用```ES Modules```本身存在环境兼容问题，而且应用中所需要的每一个文件，都需要从服务器中请求回来，这些零散的模块文件必将导致浏览器频繁请求，从而影响应用的工作效率。

所以选择在开发阶段通过模块化的方式去编写。在生产阶段还是打包到同一个文件中，最后还需要支持不同种类的前端资源类型，这样就可以把前端开发过程当中所涉及到的样式、图片、字体等所有资源文件都当做模块使用，对于整个前端应用来讲就有了一个统一的模块化方案了。

其中最为主流的就是```webpack```，```parcel```和```rollup```。

首先```webpack```作为一个模块打包工具(```Module Bundler```)他本身就可以解决模块化```js```代码打包的问题，通过```webpack```可以将一些零散的模块代码打包到同一个```js```文件中。对于代码中那些有环境兼容问题的代码可以在打包的过程中通过模块加载器(```Loader```)对其进行编译转换。其次，```webpack```还具备代码拆分(```Code Splitting```)的理念，能够将应用中所有的代码都按照需要进行打包。这样一来就不用担心代码全部打包到一起文件较大的问题了。

可以把应用加载过程中初次运行所必须的模块打包到一起，对于其他的那些模块单独存放。等应用工作过程中实际需要某个模块再异步加载这个模块从而实现增量加载或渐进式加载，这样就不用担心文件太碎或是文件太大这两个极端问题。

```webpack```支持在```js```中以模块化的方式载入任意类型的资源文件，例如在```webpack```当中可以通过```js```直接```import```一个```css```文件。这些```css```文件最终会通过```style```标签的形式工作，其他类型的文件也可以有类似的这种方式去实现。

## 配置文件

```webpack4.0```后的版本支持零配置打包，整个打包过程会按约定将```src/index.js```作为入口结果存放在```dist/main.js```中。

## 工作模式

```webpack```新增了工作模式简化了```webpack```配置的复杂度，可以理解成针对不用环境的几组预设的配置，```webpack```可以设置一个```mode```属性，如不设置默认会使用```production```模式工作。在这个模式下```webpack```会自动启动一些优化插件，例如代码压缩。

可以在```webpack```启动时传入```--mode```的参数，这个属性有三种取值，默认是```production```，还有```development```也就是开发模式。开发模式```webpack```会自动优化打包的速度，会添加一些调试过程需要的服务到代码中。

```s
yarn webpack --mode=development
```

还可以在```webpack```的配置文件中设置工作模式，在配置文件的配置中添加mode属性就可以了。

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

## 打包结果

生成的代码是一个立即执行函数，这个函数是```webpack```的工作入口。接收一个叫做```modules```的参数，调用的时传入了一个数组。

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

也就是说每一个模块最终都会被包裹到一个函数中，从而实现模块的私有作用域。

## 模块依赖方式

```css```文件也可以作为打包的入口，不过```webpack```的打包入口一般还是```js```，打包入口从某种程度来说可以算是应用的运行入口。就目前而言前端应用中的业务是由js驱动的，可以在```js```代码当中通过```import```的方式引入```css```文件。

在webpack.config.js中配置```css```的```loader```，```css-loader``` 和 ```style-loader``` 需要安装到项目中。然后将```loader```需要配置到```config```的```module```中。

```js
{
    test: /.css$/,
    use: [
        'style-loader',
        'css-loader'
    ]
}
```

传统模式开发是将文件单独分开单独引入，```webpack```建议在```js```中去引入```css```，甚至编写代码中引入资源都可以在```js```中印日。因为真正需要资源的不是应用，而是正在编写的代码，代码想要正常工作就必须要加载对应的资源，这就是```webpack```的哲学。

```js```代码本身是负责完成整个业务的功能，放大来看就是驱动了整个前端应用，在实现业务功能的过程当中可能需要用到样式或图片等一系列的资源文件。如果建立了这种依赖关系，一来逻辑上比较合理，因为```js```确实需要这些资源文件的配合才能实现对应的功能，二来可以保证上线时资源文件不缺失，而且每一个上线的文件都是必要的。

## 加载器

```webpack```社区提供了非常多的资源加载器，基本上开发者能想到的合理需求都有对应的

```css-loader```是将css资源模块转换为```js```代码的实现方式进行工作

```file-loader```图片或字体需要用到文件的资源加载器。

```js
{
    test: /.png$/,
    use: 'file-loader'
}
```

```url-loader```，除```file-loader```这种通过```copy```文件的形式处理文件资源外还有一种通过```Data URLs```的形式表示文件，可以直接表示文件，比如常见的```base64```格式。

```s
data:[mediatype][;base64],\<data>

data:text/html;charset=UTF-8,<h1>html content</h1>

{
    test: /.png$/,
    use: 'url-loader'
}
```

```url-loader```支持通过配置选项的方式设置转换的最大文件，添加```limit```的属性，将其设置为 ```10kb(10 * 1024)```单位是字节。这样```url-loader```只会将```10kb```以下的文件转换成```Data URLs```，超过```10kb```的文件仍然会交给```file-loader```去处理。

babel-loader ```webpack```在打包过程中同时处理其他```ES6```特性。

## 支持的资源加载方式

```webpack```中提供了几种资源加载方式，首先是```ES Module```标准的```import```声明。

```js
import heading from './heading.js';
```

其次是遵循```Commonjs```标准的```require```函数，不过通过```require```函数载入```ES Module```的话，对于```ES Module```的默认导出需要通过```require```函数导入结果的```default```属性获取。

```js
const heading = require('./heading.js').default;
```

遵循```AMD```标准的```define```函数和```require```函数```webpack```也同样支持。

```js
define(['./heading.js', './icon.png', './style.css'], (createHeading, icon) => {

});

require(['./heading.js', './icon.png', './style.css'], (createHeading, icon) => {

})
```

```css
@import '';
```

```html-loader```加载的```html```文件中的一些```src```属性也会触发相应的模块加载。

```js
import './main.css';
```

```css
body {
    min-height: 100vh;
    background-image: url(background.png);
    background-size: cover;
}
```

```html-loader```默认只会处理```img```标签的```src```属性，如果需要其他标签的一些属性也能够触发打包可以额外做一些配置，给```html-loader```添加```attrs```属性，比如添加一个```a:href```属性，让他能支持```a```标签的```href```属性。

```js
{
    test: /.html$/,
    use: {
        loader: 'html-loader',
        options: {
            attrs: ['img:src', 'a:href']
        }
    }
}
```

## Loader

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

```loader```不建议使能用剪头函数会拿不到上下文的```this```。官方推荐使用```loader-utils```工具处理```loader.query```。```this.callback```可以返回更多内容用于替代```return```。

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
## 插件机制

插件是```webpack```另一个核心特性，目的是为了增强```webpack```在项目自动化方面的能力，```loader```负责实现项目中各种各样资源模块的加载，```plugin```则是用来解决项目中除了资源加载以外其他的一些自动化的工作。

例如```plugin```可以实现在打包之前清除```dist```目录，还可以```copy```不需要参与打包的资源文件到输出目录，又或是压缩打包结果输出的代码。总之，有了插件```webpack```几乎无所不能的实现了前端工程化中绝大多数工作，这也是很多初学者会把```webpack```理解成前端工程化的原因。

- clean-webpack-plugin 自动清除输出目录

```js
plugins: [
    new CleanWebpackPlugin()
]
```

- html-webpack-plugin```webpack```自动生成```html```文件
```js
plugins: [
    new HtmlWebpackPlugin({
        title: 'Webpack Plugin Sample',
        meta: {
            viewport: 'width=device-width'
        },
        template: './src/index.html'
    }),
]
```

- copy-webpack-plugin 在打包时可以将静态文件复制到输出目录

```js
new CopyWebpackPlugin([
    'public/**'
])
```

## Plugin

插件必须是一个函数，或者是一个包含```apply```方法的对象，

```js
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
```

### API代理

可以解决开发中跨域问题

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

## Source Map

```Source Map```是用来映射转换过后的代码与源代码的关系，一段转换过后的代码通过转换过程中生成的这个```source map```文件可以逆向得到源代码。

```js
module.exports = {
    devtool: 'source-map',
}
```

- eval

将模块代码放到```eval```函数中执行，并且通过```sourceURL```标注模块文件的路径。这种模式下并没有生成对用的```source-map```只能定位是哪一个文件出了错误。

- eval-source-map

使用```eval```函数执行模块代码，不同的是除了可以定位错误出现的文件。还可以定位到具体的行列信息。

- cheap-eval-source-map

生成了```source-map```，只能定位到行而没有列的信息。

- cheap-module-eval-source-map

显示的是经过```babel```转换过后的结果。如果想要和手写代码一样的源代码，需要选择````cheap-module-eval-source-map````模式。

- cheap-source-map

没有```eval```就爱意味着没有用```eval```的方式执行模块代码，没有```module```也就意味着是```loader```处理过后的代码。

- inline-source-map

和普通的```source-map```效果上是一样的，只不过```source-map```的模式下文件是以物理文件的方式存在，而```inline-source-map```使用的是```dataurl```的方式去将```source-map```以```dataurl```嵌入到代码中。之前遇到的```eval-source-map```其实也是使用这种行内的方式把```source-map```嵌入进来，会导致这个代码的体积会变大很多。

- hidden-source-map

这个模式在开发工具中是看不到```source-map```的效果的。但是回到开发工具中会发现确实生成了```source-map```文件。这就跟```jq```是一样的，在构建过程当中，生成了```source-map```文件，但是他在代码当中并没有通过注释的方式去引入这个文件。

这个模式实际上是在开发一些第三方包的时候比较有用，需要生成```source-map```但是不想在代码当中直接去引用他们，一旦在使用时出现了问题，可以再把```source-map```引入回来，或者通过其他的方式使用```source-map```。

- nosources-source-map

这个模式下能看到错误出现的位置，但是点击错误信息是看不到源代码的。



在开发环境建议选择```cheap-module-eval-source-map```，一般编写代码的风格要求每一行代码不会超过```80```个字符，能够定位到行也就够了因为每一行里面最多也就```80```个字符，很容易找到对应的位置。生产环境的打包交易选择选择```none```也就是不生成```source-map```。```source-map```会暴露源代码到生产环境，这样的话但凡有一点技术的人都可以很容易复原项目中绝大多数的源代码，这个点是要注意的。

其次调试和找错误这些都应该是开发阶段的事情，应该在开发阶段就尽可能把所有的问题和隐患都找出来，而不是到了生产环境让全民去公测。

 如果对代码实在没有信心建议选择```nosources-source-map```模式，这样出现错误在控制台当中就可以找到源代码对应的位置，不至于向外暴露源代码内容。

 ## HMR

```HRM（Hot Module Replacement）```翻译过来叫做模块热替换或者叫模块热更新。可以在应用程序运行的过程中实时的替换掉