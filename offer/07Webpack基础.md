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

```HRM（Hot Module Replacement）```翻译过来叫做模块热替换或者叫模块热更新。可以在应用程序运行的过程中实时的替换掉应用中的某个模块。而应用的运行状态不会因此改变。```HMR```是```webpack```中最强大的特性之一，同时也是最受欢迎的特性。

```HMR```已经集成在了```webpack-dev-server```工具中，使用这个特性需要运行```webpack-dev-server```命令时通过```--hot```参数开启，也可以在配置文件中添加对应的配置来开启。
## DefinePlugin

```webpack4x```中新增的```production```模式内部自动开启了很多通用的优化功能。第一个是一个叫做```define-plugin```的插件，用来为代码注入全局成员。在```production```模式下，默认这个插件就会启用并且往代码当中注入了一个```process.env.NODE_ENV```的常量。很多第三方模块都是通过这个成员判断当前的运行环境，从而去决定是否执行一些操作。

```define-plugin```是一个内置的插件，先要导入```webpack```模块，```plugins```这个数组当中添加这个插件，插件的构造函数接收一个对象，对象中每一个键值都会被注入到代码中。

```js
plugins: [
    new webpack.DefinePlugin({
        API_BASE_URL: 'https://api.github.com'
    })
]
```
## Tree Shaking

```Tree-shaking```字面意思是摇树，一般伴随着摇树动作树上的枯树枝和树叶就会掉落下来。不过这里摇掉的是代码当中那些没有用到的部分，更专业的叫未引用代码（```dead-code```）。

```webpack```生产模式优化可以自动检测出代码中那些未引用的代码然后移除他们。

需要注意的是```tree-shaking```并不是```webpack```中某一个配置选项，他是一组功能搭配使用过后的效果。打开```webpack```的配置文件添加```optimization```的属性。这个属性是集中配置```webpack```内部的一些优化功能。可以先开启```usedExports```选项，表示在输出结果中只导出那些外部使用了的成员。

## sideEffects

```weboack4```中还新增了一个叫做```sideEffects```的新特性。允许通过配置的方式标识代码是否有副作用，从而为```tree-shaking```提供更大的压缩空间。

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

```webpack```在打包时就会先检查当前代码所属的这个```package.json```当中有没有```sideEffects```的标识，以此来判断这个模块是否有副作用。

## 代码分割

合理的打包方案应该是把打包结果按照一定的规则分离到多个```bundle```当中，根据应用的运行按需加载这些模块。这样的话就可以大大提高应用的响应速度以及运行效率。为了解决这样的问题```webpack```支持分包的功能，通过把模块按照所设计的规则打包到不同的```bundle```当中，从而提高应用的响应速度。

目前```webpack```实现分包的方式主要有两种，第一种是根据业务去配置不同的打包入口，也就是会有多个打包入口同时打包，这时候就会输出多个打包结果。第二种是采用```ES Module```的动态导入功能实现模块的按需加载，这个时候```webpack```会自动的把需要动态导入的这个模块单独的输出到一个```bundle```中。

### 提取公共模块

配置文件中在```optimization```中添加```splitChunks```属性，这个属性需要配置```chunks```属性设置为```all```，表示会把所有的公共模块都提取到单独的```bundle```当中。

```js
module.exports = allModes.map(item => {
    return {
        optimization: {
            splitChunks: {
                chunks: 'all'
            }
        }
    }
})
```
### 魔法注释

默认通过动态导入产生的```bundle```文件，名称只是一个序号，如果你需要给这些```bundle```命名可以使用```webpack```所特有的模板注释来去实现。

调用```import```函数的参数位置添加一个行内注释，这个注释有一个特定的格式```/* webpackChunkName: 名称 */```这样的话就可以给分包所产生的```bundle```起上名字了。

```js
// mainElement.appendChild(album());
import(/* webpackChunkName: album */'./album/album')
```

如果```chunkName```是相同的，最终就会被打包到一起。

## MiniCssExtractPlugin

```MiniCssExtractPlugin```是一个可以将```css```代码从打包结果中提取出来的插件，通过这个插可以实现```css```模块的按需加载。

```js
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
```

## OptimizeCssAssetsWebpackPlugin

使用```MiniCssExtractPlugin```过后样式文件可以被提取到单独的```css```文件中但是样式文件没有压缩。这是因为```webpack```内置的压缩插件，紧紧是针对于```js```文件压缩，对于其他资源文件压缩，需要额外的插件支持。

```js
optimization: {
    minimize: [
        new OptimizeCssAssetsWebpackPlugin(),
        new TerserWebpackPlugin()
    ]
}
```
## Hash文件名

这里支持三种```hash```效果各不相同。

```hash```可以通过```[hash]```设置，这个```hash```实际上是整个项目级别的，也就是说一旦项目中任何一个地方发生改动，打包过程中的```hash```值都会发生变化。


```chunkhash```，这个```hash```是```chunk```级别的，只要是同一路的打包```chunkhash```都是相同的。

```contenthash```是文件级别的```hash```，根据输出文件的内容生成的```hash```值，也就是说只要是不同的文件就有不同的```hash```值。

```js
new MiniCssExtractPlugin({
    filename: '[name]-[contenthash:8].bundle.css'
})
```
## 使用技巧

polyfill会把js注入到全局变量中，污染全局，适合在项目中，不适合在插件和工具库中使用。plugin-transform-runtime以闭包的方式注入不会影响全局，适合工具库。

## 实现打包原理

创建入口```bundle.js```文件，创建```modules```函数用于分析模块内容。

```js
const fs = require('fs');
const modules = function(entry) { // 接收一个入口文件
    const content = fs.readFileSync(entry, 'utf-8');
    console.log(content);
}

modules("./src/index.js");
```

拿到文件中的依赖模块。

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

根据```body```里面的分析结果，遍历所有引入的模块。

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

借助```@babel/core```和```@babel/preset-env```把```ast```语法树转换成合适的代码。

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

## 模块联邦

模块联邦(```Module Federation```)是```Webpack 5```中新增的一项功能，可以实现跨应用共享模块。比如有```AB```两个应用，```A```应用中提供```fromA```方法，```B```应用提供了```fromB```方法，现在需要在```A```应用中调用```B```应用的方法，```B```应用中调用```A```应用的方法。这就涉及到了跨应用调用方法，就需要用到模块联邦啦。

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

共享模块

```js
{
  shared: ["faker"] // 共享模块的名字，可以写多个。
}
```

```webpack```配置中加入如下代码。

```json
shared: {
  faker: {
    singleton: true // 如果版本不一致使用高版本
  }
}
```
