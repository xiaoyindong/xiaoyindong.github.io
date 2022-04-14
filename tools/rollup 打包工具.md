## 1. 概述

```Rollup```是一款基于```ES Module```的打包器，类似于```webpack```可以将项目中散落的细小模块打包为整块的代码，不用点是```Rollup```打包的模块可以更好的运行在浏览器或```nodeJs```环境。

从作用上来看```Rollup```与```webpack```非常类似，相比于```webpack```来说```Rollup```要小巧很多。```webpack```配合插件几乎可以完成前端工程化的绝大多数工作。```Rollup```仅是```ES Module```的打包器，并没有其他额外的功能。

```webpack```有对于开发者十分友好的```hmr```功能```Rollup中```则无法支持。```Rollup```诞生的目的并不是与```webpack```工具竞争，他的初衷只是希望提供一个高效的```ES Module```打包器，充分利用```ES Module```的各项特性构建出结构比较扁平，性能比较出众的类库。

## 2. 使用

这里准备一个简单的示例，使用```ES Module```方式组织代码模块化，示例的源代码包含三个文件，```message.js```以默认导出的方式导出了一个对象。

```js
export default {
    hi: 'Hey Guys, I am yd~'
}
```

```logger.js```中导出两个函数。

```js
export const log = msg => {
    console.log(msg);
}

export const error = msg => {
    conole.log('------ ERROR ------')
    console.log(msg);
}
```

```index.js```导入这两个模块，并且使用他们。

```js
import { log } from './logger';
import message from './message';

const msg = message.hi;
log(msg);
```

安装```rollup```对示例应用打包。

```s
yarn add rollup --dev
```

```rollup```自带```cli```程序，通过```yarn rollup```运行程序。可以发现，在不传递任何参数的情况下```rollup```会自动打印出帮助信息，帮助信息开始的位置就表示应该通过参数去指定一个打包入口文件。打包入口是```src```下面的```index.js```文件。

```s
yarn rollup ./src/index.js
```

此时还应该指定一个代码的输出格式，可以使用```--format```参数指定输出的格式比如```iife```也自执行函数的格式。

```s
yarn rollup ./src/index.js --format iife
```

还可以通过```--file```指定文件的输出路径，比如这里指定```dist```文件夹下的```bundle.js```，这样打包结果就会输出到```dist```文件当中。

```s
yarn rollup ./src/index.js --format iife --file dist/bundle.js
```

```rollup```打包结果惊人的简洁，基本上和手写的代码是一样的，相比于```webpack```中大量的引导代码，这里的输出结果几乎没有任何的多余代码。

```rollup```只是把打包过程中各个模块按照模块的依赖顺序先后的拼接到一起，而且仔细观察打包结果会发现，在输出结果中只会保留用到的部分，对于未引用的部分都没有输出。```rollup```默认自动开启```tree-shaking```优化输出的结果，```tree-shaking```概念最早也是在```rollup```工具中提出的。

```js
(function () {
    'use strict';

    const log = msg => {
        console.log(msg);
    };

    var message = {
        hi: 'Hey Guys, I am yd~'
    };

    const msg = message.hi;
    log(msg);

}());

```

## 3. 配置文件

```rollup```支持以配置文件的方式去配置打包过程中的各项参数，可以在项目的跟目录下新建一个```rollup.config.js```配置文件。这个文件是运行在```node```环境中，不过```rollup```自身会额外处理这个配置文件，所以可以直接使用```ES Modules```。

在这个文件中需要导出一个配置对象，对象中通过```input```属性指定打包的入口文件路径。通过```output```指定输出的相关配置，```output```属性要求是一个对象，在```output```对象中可以使用```file```属性指定输出的文件名。```format```属性可以用来指定输出格式。

```js
export default {
    input: 'src/index.js',
    output: {
        file: 'dist/bundle.js',
        format: 'iife'
    }
}
```

需要通过```--config```参数来表明使用项目中的配置文件，默认是不去读取配置文件的。

```js
yarn rollup --config rollup.config.js
```

## 4. 使用插件

```rollup```自身的功能就是```ES```模块的合并打包，如果项目有更高级别的需求，例如想去加载其他类型的资源文件，或者是要在代码中导入```CommonJS```模块，又或者是想要编译```ECMAScript```的新特性。这些额外的需求，```rollup```同样支持使用插件的方式扩展实现，而且插件是```rollup```唯一的扩展方式，他不像```webpack```中划分了```loader```，```plugin```和```minimize```这三种扩展方式。

尝试使用一个可以在代码中导入```JSON```文件的插件，这里使用的插件名字叫做```rollup-plugin-json```需要先安装这个插件。

```s
yarn add rollup-plugin-json --dev
```

安装完成过后打开配置文件，由于```rollup```的配置文件可以直接使用```ES Modules```所以使用```import```的方式导入插件。导出的是一个函数，可以将函数的调用结果添加到配置对象的```plugins```数组当中。

```js
import json from 'rollup-plugin-json';

export default {
    input: 'src/index.js',
    output: {
        file: 'dist/bundle.js',
        format: 'iife'
    },
    plugins: [
        json()
    ]
}
```

配置好插件后就可以在代码中通过```import```方式导入```json```文件了。

```js
import { log } from './logger';
import message from './message';
import { name, version } from '../package.json';

const msg = message.hi;
log(msg);

log(name);
log(version);
```

```s
yarn rollup --config rollup.config.js
```

输出的```bundle.js```能看到```json```中的```name```和```version```正常被打包进来了，而```json```当中那些没有用到的属性也都会被```tree-shaking```移除掉。

```js
(function () {
    'use strict';

    const log = msg => {
        console.log(msg);
    };

    var message = {
        hi: 'Hey Guys, I am yd~'
    };

    var name = "rollup_p";
    var version = "1.0.0";

    const msg = message.hi;
    log(msg);

    log(name);
    log(version);

}());
```

## 5. 使用Babel和TS

```json
"devDependencies": {
    "@babel/core": "^7.15.5",
    "@babel/preset-env": "^7.15.6",
    "@babel/preset-typescript": "^7.15.0",
    "rollup-plugin-babel": "^4.4.0",
    "typescript": "^4.4.3"
}
```

```.babelrc```

```json
{
    "presets": [
        "@babel/preset-env",
        "@babel/preset-typescript"
    ]
}
```

别忘了初始化```ts```的配置文件```tsconfig.json```。

```rollup```配置文件中```plugins```加入```babel```插件。

```js
const babel = require('rollup-plugin-babel');
const extensions = ['.js', '.ts'];

export default {
    input: {
        utils: 'src/index.ts',
    },
    output: {
        dir: 'dist',
        format: 'iife'
    },
    plugins: [
        babel({
          exclude: 'node_modules/**',
          extensions,
        }),
    ]
}

```

## 6. 编译less

```json
"devDependencies": {
    "rollup-plugin-less": "^1.1.2",
    "rollup-plugin-postcss": "^3.1.1"
}
```

```rollup```配置文件中```plugins```加入```postcss```插件。

```js
import postcss from 'rollup-plugin-postcss';

export default {
    input: {
        utils: 'src/index.ts',
    },
    output: {
        dir: 'dist',
        format: 'iife'
    },
    plugins: [
        postcss({
            extensions: [ '.less' ],
        }),
    ]
}
```

## 7. 压缩代码

```js
import { uglify } from 'rollup-plugin-uglify';

const isDev = process.env.NODE_ENV !== 'production';

export default {
    input: {
        utils: 'src/index.ts',
    },
    output: {
        dir: 'dist',
        format: 'iife'
    },
    plugins: [
        !isDev && uglify()
    ]
}

```

## 8. 加载NPM模块

```rollup```默认只能按照文件路径的方式去加载本地的文件模块，对于```node_modules```中的第三方的模块并不能够像```webpack```一样直接去通过模块的名称导入对应的模块。为了抹平这个差异```rollup```官方给出了```rollup-plugin-node-resolve```插件。使用这个插件可以在代码当直接使用模块名称导入对应的模块。

```s
yarn add rollup-plugin-node-resolve --dev
```

配置文件中需要先将这个模块导入进来，然后将插件配置到```plugins```数组中。

```js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve'

export default {
    input: 'src/index.js',
    output: {
        file: 'dist/bundle.js',
        format: 'iife'
    },
    plugins: [
        json(),
        resolve()
    ]
}
```

完成以后就可以直接导入```node_modules```中的第三方的```npm```模块了。导入```loadash-es```模块，这个模块是```loadash```的```ES Modules```版本。

```js
import _ from 'loadash-es';
import { log } from './logger';
import message from './message';
import { name, version } from '../package.json';

const msg = message.hi;
log(msg);

log(name);
log(version);
log(_.camelCase('hello world'))
```

这里我们使用的```loadsh ES Module```的版本而不是使用普通版本是因为```rollup```默认只能取处理```ES Module```模块，如果需要使用普通版本需要做额外的处理。

## 9. 加载CommonJS模块

为了兼容```CommonJS```方式模块官网给出了```rollup-plugin-commonjs```插件。

```s
yarn add rollup-plugin-commonjs --dev
```

```js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';

export default {
    input: 'src/index.js',
    output: {
        file: 'dist/bundle.js',
        format: 'iife'
    },
    plugins: [
        commonjs(),
        json(),
        resolve()
    ]
}
```

配置过后就可以使用```commonjs```模块了，在```src```下添加```commonjs```模块```cjs.module.js```文件。

```js
module.exports = {
    foo: 'bar'
} 
```

```commonjs```导出整体会作为默认导出。

```js
import _ from 'loadash-es';
import { log } from './logger';
import message from './message';
import { name, version } from '../package.json';
import cjs from './cjs.module';

const msg = message.hi;
log(msg);

log(name);
log(version);
log(_.camelCase('hello world'));
log(cjs);
```

打包过后```commonjs```被打包进```bundle.js```里面了。

## 10. 代码拆分

```rollup```最新的版本开始支持代码拆分了，可以使用符合```ES Module```标准的动态导入的方式(```Dynamic Imports```)实现模块的按需加载，```rollup```内部会自动处理代码的拆分，也就是分包。

使用动态导入的方式导入```logger```对应的模块，```import```方法返回的是```promise```对象，在```promise```的```then```方法里面可以拿到模块导入过后的对象。

```js
// import { log } from './logger';
// import message from './message';

// const msg = message.hi;
// log(msg);

import('./logger').then(({ log }) => {
    log('code splitting');
});
```

```s
yarn rollup --config
```

使用代码拆分式打包要求```format```不能是```iife```形式。因为自执行函数会把所有的模块都放到同一个函数当中，他并没有像````webpack````一样的引导代码，所以也就没有办法实现代码拆分。

要想使用代码拆分必须要使用```amd```或```commonjs```等其他标准，浏览器环境只能使用```amd```标准。同样因为输出多个文件，这里就不能使用```file```的这种配置方式，因为```file```是执行一个单个文件输出的文件名。

如果需要输出多个文件可以使用```dir```的参数将输出的目录设置为```dist```。

```js
export default {
    input: 'src/index.js',
    output: {
        // file: 'dist/bundle.js',
        // format: 'iife'
        dir: 'dist',
        format: 'amd'
    },
}
```

```s
yarn rollup --config
```

打包完成过后可以看到```dist```目录中会根据动态导入生成入口的```bundle.js```以及动态导入的```bundle```。都是采用```amd```的标准输出的。

## 11. 多入口打包

```rollup```支持多入口打包，而且对于不同入口中公共的部分，会自动提取到单个文件当中，作为独立的```bundle```。实例中有两个入口分别是```index```和```album```，共用```fetch.js```和```logger.js```两个模块。

```index.js```

```js
import fetchApi from './fetch';
import { log } from './logger';

fetchApi('/posts').then(data => {
    data.forEach(item => {
        log(item);
    })
})
```

```album.js```

```js
import fetchApi from './fetch';
import { log } from './logger';

fetchApi('/photos?albumId=1').then(data => {
    data.forEach(item => {
        log(item);
    })
})

```

```fetch.js```

```js
export default endpoint => {
    return fetch('xxxxx').then(response => response.json())
}
```

```logger.js```

```js
export const log = msg => {
    console.log(msg);
}

export const error = msg => {
    conole.log('------ ERROR ------')
    console.log(msg);
}
```

配置多入口打包非常简单，只需要将```input```属性修改为一个数组就可以了，当然也可以使用与```webpack```相同的对象配置方式，不过这里需要注意，因为多入口打包，内部会自动提取公共模块，也就是说内部会使用代码拆分。这里就不能使用```iife```输出格式了，需要将输出格式修改为```amd```。

```js
export default {
    // input: ['src/index.js', 'src/album.js'],
    input: {
        foo: 'src/index.js',
        bar: 'src/album.js'
    },
    output: {
        dir: 'dist',
        // format: 'iife',
        format: 'amd'
    },
}
```

打包过后```dist```目录下会多出三个```js```文件，分别是两个不同打包入口的打包结果和公共提取出来的公共模块。

另外需要注意一点的是，对于```amd```这种输出格式的```js```文件，不能直接去引用到页面上，必须通过实现```amd```标准的库去加载。

在```dist```目录下手动创建一个```html```文件。然后在```html```中使用打包生成的```bundle.js```, 采用```requirejs```的库去加载```amd```标准输出的```bundle```。

```require```可以通过```data-main```参数来制定```require```加载的模块的入口模块路径。

```html
<body>
    <script src="...require.js" data-main="foo.js"></script>
</body>
```

## 12. rollup-watch 监听文件

```js
"devDependencies": {
    "rollup-watch": "^4.3.1"
  },
```

```package.json```，文件变化时重新打包。

```json
"scripts": {
    "watch": "rollup -c -w",
}
```

## 13. 选用原则 

```rollup```确实有他的优势，首先是他输出的结果会更加扁平一些，执行效率自然就会更高。其次是他会自动取移除那些未引用代码，也就是```tree-shaking```。再一个就是他的打包结果基本上跟手写的代码一致，也就是打包结果对于开发者而言是可以正常阅读的。

但是他的缺点同样也很明显，首先就是加载一些非```ES Module```的第三方模块就会比较复杂，需要配置一大堆插件。因为这些模块最终都被打包到一个函数当中了，所以没有办法像```webpack```一样去实现```HMR```这种模块热替换的这种开发体验。再一个就是在浏览器环境中，代码拆分必须要使用像```requireJS```这样的```amd```库，因为他的代码拆分必须要使用像```amd```这样的输出格式。

综合以上的这些特点，如果正在开发一个应用程序，肯定要面临大量引入第三方模块这样的需求。同时又需要像```HMR```这样的功能去提升开发体验。而且应用一旦大了以后还必须要去分包。这些需求```rollu```p在满足上都会有一些欠缺。

如果正在开发的是一个```js```框架或者类库，优点就特别有必要，而缺点基本可以忽略。拿加载第三方模块来说在开发类库的时候很少的在代码中去依赖一些第三方模块。所以像```react```或者```vue```之类的框架都是使用```rollup```作为模块打包器，而并非是```webpack```。

到目前为止，开源社区中大多数人还是希望这两个工具可以共同存在共同发展，并且能够相互支持和借鉴。原因很简单，就是希望能够让更专业的工具去做更专业的事情。

总结一下就是```Webpack```是大而全```Rollup```是小而美。

在对他们两者之前的选择上，基本的原则就是如果正在开放应用程序，建议使用```Webpack```, 如果正在开发类库或者开发框架的话那建议选择```Rollup```。

当然```Rollup```同样可以去构建绝大多数的应用程序，```Webpack```也同样可以去构建类库或者是框架。只不过相对来讲的话术业有专攻。

另外一点随着近几年```Webpack```的发展```Rollup```中的很多优势几乎已经被抹平了，例如```Rollup```中扁平化输出在```Webpack```中就可以使用```concatenateModules```插件去完成。也可以实现类似的输出。
