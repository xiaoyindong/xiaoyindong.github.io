## 1. 概述

```Parcel```是```2017```年发布的，出现的原因是因为当时```Webpack```在使用上过于繁琐，而且官网的文档也不是很清晰明了。```Parcel```一经推出就迅速被推上了风口浪尖，其核心特点就是真正意义上做到了完全零配置，提供了近乎傻瓜式的使用体验，只需要了解所提供的几个简单的命令就可以使用他构建前端应用程序，对项目没有任何的侵入。

而且整个过程会自动安装依赖，让开发过程可以更加专注编码。除此之外```Parcel```构建速度非常快，内部使用多进程工作，相比于```Webpack```的打包他的速度要更快一些。

但是目前实际上绝大多数的项目打包还是会选择使用```Webpack```，因为```Webpack```的生态会更好一些，扩展也就会更丰富，而且出现问题也可以很容易去解决。

```Parcel```这样的工具对于开发者而言，去了解他其实也就是为了保持对新鲜技术和工具的敏感度。从而更好的把握技术的趋势和走向，仅此而已。

## 2. 使用

通过```yarn init```初始化```package.json```文件。完成以后可以安装```Parcel```对应的模块```parcel-bundler```。

```s
yarn add parcel-bundler --dev
```

新建```src```目录用于存放开发阶段所编写的源代码。创建```/src/index.html```文件作为```parcel```打包的入口文件。

```parcel```和```webpack```一样都支持以任意类型的文件作为打包入口，不过```parcel```官方建议使用```html```文件作为打包入口，理由是因为```html```是应用运行在浏览器端时的入口。

在这个入口文件中可以像平常一样编写，也可以引用一些资源文件。在这里被引用的资源最终都会被```parcel```打包到一起输出到输出目录。

这里先引入```main.js```的脚本文件。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Parcel Tutorials</title>
</head>
<body>
    <script src="main.js"></script>
</body>
</html>
```

新建```/src/main.js```文件，除此之外再新建一个```/src/foo.js```文件。

```foo.js```以```ES Module```的方式默认导出一个对象。

```js
export default {
    bar: () => {
        console.log('hello parcel~')
    }
}
```

```main.js```中通过```import```导入```foo```模块。

```js
import foo from './foo';

foo.bar();
```

```parcel```支持对```ES Module```模块的打包，打包命令需要传入打包入口的路径。

```s
yarn parcel src/index.html
```

```parcel```就会根据所传入的参数，先去找到```index.html```文件，然后根据```index.html```当中的```script```标签去找到引入的```main.js```文件，最后顺着```import```语句找到```foo```模块，从而去完成整体项目的打包。

```parcel```命令不仅打包应用而且同时还开启了一个开发服务器，这个开发服务器跟```webpack```当中的```dev-server```一样。在浏览器当中打开开发人员工具(控制台```F1```2)，此时就可以使用自动刷新的功能了。

```js
export default {
    bar: () => {
        // console.log('hello parcel~');
        console.log('hello parcel11~');
    }
}
```

如果需要模块热替换体验```parcel```也是支持的，在```main.js```中需要使用```hmr```提供的```api```。

先判断```module.hot```对象是否存在，如果存在这个对象就证明当前这个环境可以使用```hmr```的```api```。使用```module.hot.accept```方法处理模块热替换的逻辑。

不过这里的```accept```和```webpack```提供的```api```有一点不太一样，```webpack```当中的```api```接收两个参数，用来去处理指定模块更新过后的逻辑。而```parcel```提供的```accept```只接收一个参数也就是回调函数，当模块更新或者是模块所依赖的模块更新过后他会自动执行。

```js
import foo from './foo';

foo.bar();

if (module.hot) {
    module.hot.accept(() => {
        console.log('hmr');
    })
}
```

除了热替换```parcel```还支持一个非常友好的功能，就是自动安装依赖。

那试想一下开发应用的过程中，突然间想要去使用某个第三方的模块，此时就需要先停止正在运行的```dev-server```然后去安装这个模块，安装完成过后再去重新启动```dev-server```。有了自动安装依赖这个功能过后就不再这样麻烦了。

假设想使用```jQuery```, 虽然并没有安装这个模块，但是因为有了自动安装依赖这样的功能的缘故。只管正常导入就可以了。导入完成之使用```jQuery```提供的```api```, 在文件保存过后, ```parcel```会自动取安装导入的模块。极大程度避免了额外的操作。

```js
import $ from 'jquery';
import foo from './foo';

foo.bar();

$(document.body).append('<h1>Hello Parcel</h1>');

if (module.hot) {
    module.hot.accept(() => {
        console.log('hmr');
    })
}
```

除此之外```parcel```同样支持加载其他类型的资源模块，而且相比于其他的模块打包器```parcel```当中加载任意类型的资源模块同样还是零配置的。

例如添加一个```/src/style.css```的样式文件。然后在这个文件当中添加一些简单的样式。

```css
body {
    background: red;
}
```

回到```main.js```中通过```import```导入这个样式文件，保存过后这个样式就可以立即生效了。

```js
import $ from 'jquery';
import foo from './foo';
import './style.css';

foo.bar();

$(document.body).append('<h1>Hello Parcel</h1>');

if (module.hot) {
    module.hot.accept(() => {
        console.log('hmr');
    })
}
```

整个过程并没有安装额外的插件，还可以随意添加图片也是可以的。

```js
import $ from 'jquery';
import foo from './foo';
import './style.css';
import logo from './icon.png';

foo.bar();

$(document.body).append('<h1>Hello Parcel</h1>');

$(document.body).append(`<img src=${logo} />`);

if (module.hot) {
    module.hot.accept(() => {
        console.log('hmr');
    })
}
```

总之，```parcel```希望给开发者的体验就是想要做什么你就只管去做，额外的事情就由工具负责处理。另外```parcel```同样支持动态导入，内部如果使用了动态导入他也会自动拆分代码。

```js
// import $ from 'jquery';
import foo from './foo';
import './style.css';
import logo from './icon.png';

foo.bar();

import('jquery').then($ => {
    $(document.body).append('<h1>Hello Parcel</h1>');

    $(document.body).append(`<img src=${logo} />`);
})

if (module.hot) {
    module.hot.accept(() => {
        console.log('hmr');
    })
}
```

以上基本就是```Parcel```中最常用的一些特性了。在使用上```Parcel```几乎没有任何难度，从头到尾只是执行了一个```Parcel```命令。所有事情都是```Parcel```内部自动完成的。

## 3. 部署生产

执行```parcel```提供的```build```命令跟上入口文件路径就可以以生产模式运行打包了。对于相同体量的项目```Parcel```的构建速度比```Webpack```快很多。因为在```Parcel```的内部使用的是多进程同时工作，充分发挥了多核```CPU```的性能，```Webpack```中也可以使用```happypack```插件实现这一点。

```s
parcel build src/index.html
```
