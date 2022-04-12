```Vite```是一个更轻、更快的```web```应用开发工具，面向现代浏览器。底层基于```ECMAScript```标准原生模块系统```ES Module```实现。他的出现是为了解决```webpack```冷启动时间过长以及```Webpack HMR```热更新反应速度慢等问题。

默认情况下```Vite```创建的项目是一个普通的```Vue3```应用，相比基于```Vue-cli```创建的应用少了很多配置文件和依赖。

```Vite```创建的项目所需要的开发依赖非常少，只有```Vite和@vue/compiler-sfc```。这里面```Vite```是一个运行工具，```compiler-sfc```则是为了编译```.vue```结尾的单文件组件。在创建项目的时候通过指定不同的模板也可以支持使用其他框架例如```React```。项目创建完成之后可以通过两个命令启动和打包。

```s
# 开启服务器
vite serve
# 打包
vite build
```

正是因为```Vite```启动的```web```服务不需要编译打包，所以启动的速度特别快，调试阶段大部分运行的代码都是你在编辑器中书写的代码，这相比于```webpack```的编译后再呈现确实要快很多。当然生产环境还是需要打包的，毕竟很多时候我们使用的最新```ES```规范在浏览器中还没有被支持，```Vite```的打包过程和```webpack```类似会将所有文件进行编译打包到一起。对于代码切割的需求``Vite``采用的是原生的动态导入来实现的，所以打包结果只能支持现代浏览器，如果需要兼容老版本浏览器可以引入```Polyfill```。

使用Webpack打包除了因为浏览器环境并不支持模块化和新语法外，还有就是模块文件会产生大量的```http```请求。如果你使用模块化的方式开发，一个页面就会有十几甚至几十个模块，而且很多时候会出现几```kb```的文件，打开一个页面要加载几十个```js```资源这显然是不合理的。

```Vite```创建的项目几乎不需要额外的配置默认已经支持```TS```、```Less```, ```Sass```，```Stylus```，```postcss```了，但是需要单独安装对应的编译器，同时默认还支持```jsx```和```Web Assembly```。

```Vite```带来的好处是提升开发者在开发过程中的体验，```web```开发服务器不需要等待即可立即启动，模块热更新几乎是实时的，所需的文件会按需编译，避免编译用不到的文件。并且开箱即用避免```loader```及```plugins```的配置。

```Vite```的核心功能包括开启一个静态的```web```服务器，能够编译单文件组件并且提供```HMR```功能。当启动```vite```的时候首先会将当前项目目录作为静态服务器的根目录，静态服务器会拦截部分请求，当请求单文件的时候会实时编译，以及处理其他浏览器不能识别的模块，通过```websocket```实现```hmr```。

## 实现静态测试服务器

首先实现一个能够开启静态```web```服务器的命令行工具。```vite1.x```内部使用的是```Koa```来实现静态服务器。(ps：node命令行工具可以查看我之前的文章，这里就不介绍了，直接贴代码)。

```s
npm init
npm install koa koa-send -D
```

工具```bin```的入口文件设置为本地的```index.js```

```js
#!/usr/bin/env node

const Koa = require('koa')
const send = require('koa-send')

const app = new Koa()

// 开启静态文件服务器
app.use(async (ctx, next) => {
    // 加载静态文件
    await send(ctx, ctx.path, { root: process.cwd(), index: 'index.html'})
    await next()
})

app.listen(5000)

console.log('服务器已经启动 http://localhost:5000')
```

这样就编写好了一个```node```静态服务器的工具。

## 处理第三方模块

我们的做法是当代码中使用了第三方模块(```node_modules```中的文件)，可以通过修改第三方模块的路径给他一个标识，然后在服务器中拿到这个标识来处理这个模块。

首先需要修改第三方模块的路径，这里需要一个新的中间件来实现。判断一下当前返回给浏览器的文件是否是```javascript```，只需要看响应头中的```content-type```。如果是```javascript```需要找到这个文件中引入的模块路径。```ctx.body```就是返回给浏览器的内容文件。这里的数据是一个```stream```，需要转换成字符串来处理。

```js

const stream2string = (stream) => {
    return new Promise((resolve, reject) => {
        const chunks = [];
        stream.on('data', chunk => {chunks.push(chunk)})
        stream.on('end', () => { resolve(Buffer.concat(chunks).toString('utf-8'))})
        stream.on('error', reject)
    })
}

// 修改第三方模块路径
app.use(async (ctx, next) => {
    if (ctx.type === 'application/javascript') {
        const contents = await stream2string(ctx.body);
        // 将body中导入的路径修改一下，重新赋值给body返回给浏览器
        // import vue from 'vue', 匹配到from '修改为from '@modules/
        ctx.body = contents.replace(/(from\s+['"])(?![\.\/])/g, '$1/@modules/');
    }
})
```

接着开始加载第三方模块, 这里同样需要一个中间件，判断请求路径是否是修改过的@module开头，如果是的话就去node_modules里面加载对应的模块返回给浏览器。这个中间件要放在静态服务器之前。

```js
// 加载第三方模块
app.use(async (ctx, next) => {
    if (ctx.path.startsWith('/@modules/')) {
        // 截取模块名称
        const moduleName = ctx.path.substr(10);
    }
})
```

拿到模块名称之后需要获取模块的入口文件，这里要获取的是```ES Module```模块的入口文件，需要先找到这个模块的```package.json```然后再获取这个```package.json```中的```module```字段的值也就是入口文件。

```js
// 找到模块路径
const pkgPath = path.join(process.pwd(), 'node_modules', moduleName, 'package.json');
const pkg = require(pkgPath);
// 重新给ctx.path赋值，需要重新设置一个存在的路径，因为之前的路径是不存在的
ctx.path = path.join('/node_modules', moduleName, pkg.module);
// 执行下一个中间件
awiat next();
```

这样浏览器请求进来的时候虽然是@modules路径，但是在加载之前将path路径修改为了```node_modules```中的路径，这样在加载的时候就会去```node_modules```中获取文件，将加载的内容响应给浏览器。

```js
// 加载第三方模块
app.use(async (ctx, next) => {
    if (ctx.path.startsWith('/@modules/')) {
        // 截取模块名称
        const moduleName = ctx.path.substr(10);
        // 找到模块路径
        const pkgPath = path.join(process.pwd(), 'node_modules', moduleName, 'package.json');
        const pkg = require(pkgPath);
        // 重新给ctx.path赋值，需要重新设置一个存在的路径，因为之前的路径是不存在的
        ctx.path = path.join('/node_modules', moduleName, pkg.module);
        // 执行下一个中间件
        awiat next();
    }
})
```

## 单文件组件处理

之前说过浏览器是没办法处理```.vue```资源的, 浏览器只能识别```js```、```css```等常用资源，所以其他类型的资源都需要在服务端处理。当请求单文件组件的时候需要在服务器将单文件组件编译成js模块返回给浏览器。

所以这里当浏览器第一次请求```App.vue```的时候，服务器会把单文件组件编译成一个对象，先加载这个组件，然后再创建一个对象。

```js
import Hello from './src/components/Hello.vue'
const __script = {
    name: "App",
    components: {
        Hello
    }
}
```

接着再去加载入口文件，这次会告诉服务器编译一下这个单文件组件的模板，返回一个render函数。然后将render函数挂载到刚创建的组件选项对象上，最后导出选项对象。

```js
import { render as __render } from '/src/App.vue?type=template'
__script.render = __render
__script.__hmrId = '/src/App.vue'
export default __script
```

也就是说```vite```会发送两次请求，第一次请求会编译单文件文件，第二次请求是编译单文件模板返回一个```render```函数。

1. 编译单文件选项

首先来实现一下第一次请求单文件的情况。需要把单文件组件编译成一个选项，这里同样用一个中间件来实现。这个功能要在处理静态服务器之后，处理第三方模块路径之前。

首先需要对单文件组件进行编译需要借助```compiler-sfc```。

```js
// 处理单文件组件
app.use(async (ctx, next) => {
    if (ctx.path.endsWith('.vue')) {
        // 获取响应文件内容，转换成字符串
        const contents = await streamToString(ctx.body);
        // 编译文件内容
        const { descriptor } = compilerSFC.parse(contents);
        // 定义状态码
        let code;
        // 不存在type就是第一次请求
        if (!ctx.query.type) {
            code = descriptor.script.content;
            // 这里的code格式是, 需要改造成我们前面贴出来的vite中的样子
            // import Hello from './components/Hello.vue'
            // export default {
            //      name: 'App',
            //      components: {
            //          Hello
            //      }
            //  }
            // 改造code的格式，将export default 替换为const __script =
            code = code.relace(/export\s+default\s+/g, 'const __script = ')
            code += `
                import { render as __render } from '${ctx.path}?type=template'
                __script.rener = __render
                export default __script
            `
        }
        // 设置浏览器响应头为js
        ctx.type = 'application/javascript'
        // 将字符串转换成数据流传给下一个中间件。
        ctx.body = stringToStream(code);
    }
    await next()
})

const stringToStream = text => {
    const stream = new Readable();
    stream.push(text);
    stream.push(null);
    return stream;
}
```

```s
npm install @vue/compiler-sfc -D
```

接着我们再来处理单文件组件的第二次请求，第二次请求```url```会带上```type=template```参数，需要将单文件组件模板编译成```render```函数。

首先需要判断当前请求中有没有```type=template```。

```js
if (!ctx.query.type) {
    ...
} else if (ctx.query.type === 'template') {
    // 获取编译后的对象 code就是render函数
    const templateRender = compilerSFC.compileTemplate({ source: descriptor.template.content })
    // 将render函数赋值给code返回给浏览器
    code = templateRender.code
}
```

这里还要处理一下工具中的```process.env```，因为这些代码会返回到浏览器中运行，如果不处理会默认为```node```导致运行失败。可以在修改第三方模块路径的中间件中修改，修改完路径之后再添加一条修改```process.env```。

```js
// 修改第三方模块路径
app.use(async (ctx, next) => {
    if (ctx.type === 'application/javascript') {
        const contents = await stream2string(ctx.body);
        // 将body中导入的路径修改一下，重新赋值给body返回给浏览器
        // import vue from 'vue', 匹配到from '修改为from '@modules/
        ctx.body = contents.replace(/(from\s+['"])(?![\.\/])/g, '$1/@modules/').replace(/process\.env\.NODE_ENV/g, '"development"');
    }
})

```

至此就实现了一个简版的```vite```，当然这里我们只演示了```.vue```文件，对于```css```，```less```等其他资源都没有处理，不过方法都是类似的，感兴趣的同学可以自行实现。

```js
#!/usr/bin/env node

const path = require('path')
const { Readable } = require('stream)
const Koa = require('koa')
const send = require('koa-send')
const compilerSFC = require('@vue/compiler-sfc')

const app = new Koa()

const stream2string = (stream) => {
    return new Promise((resolve, reject) => {
        const chunks = [];
        stream.on('data', chunk => {chunks.push(chunk)})
        stream.on('end', () => { resolve(Buffer.concat(chunks).toString('utf-8'))})
        stream.on('error', reject)
    })
}

const stringToStream = text => {
    const stream = new Readable();
    stream.push(text);
    stream.push(null);
    return stream;
}

// 加载第三方模块
app.use(async (ctx, next) => {
    if (ctx.path.startsWith('/@modules/')) {
        // 截取模块名称
        const moduleName = ctx.path.substr(10);
        // 找到模块路径
        const pkgPath = path.join(process.pwd(), 'node_modules', moduleName, 'package.json');
        const pkg = require(pkgPath);
        // 重新给ctx.path赋值，需要重新设置一个存在的路径，因为之前的路径是不存在的
        ctx.path = path.join('/node_modules', moduleName, pkg.module);
        // 执行下一个中间件
        awiat next();
    }
})

// 开启静态文件服务器
app.use(async (ctx, next) => {
    // 加载静态文件
    await send(ctx, ctx.path, { root: process.cwd(), index: 'index.html'})
    await next()
})

// 处理单文件组件
app.use(async (ctx, next) => {
    if (ctx.path.endsWith('.vue')) {
        // 获取响应文件内容，转换成字符串
        const contents = await streamToString(ctx.body);
        // 编译文件内容
        const { descriptor } = compilerSFC.parse(contents);
        // 定义状态码
        let code;
        // 不存在type就是第一次请求
        if (!ctx.query.type) {
            code = descriptor.script.content;
            // 这里的code格式是, 需要改造成我们前面贴出来的vite中的样子
            // import Hello from './components/Hello.vue'
            // export default {
            //      name: 'App',
            //      components: {
            //          Hello
            //      }
            //  }
            // 改造code的格式，将export default 替换为const __script =
            code = code.relace(/export\s+default\s+/g, 'const __script = ')
            code += `
                import { render as __render } from '${ctx.path}?type=template'
                __script.rener = __render
                export default __script
            `
        } else if (ctx.query.type === 'template') {
            // 获取编译后的对象 code就是render函数
            const templateRender = compilerSFC.compileTemplate({ source: descriptor.template.content })
            // 将render函数赋值给code返回给浏览器
            code = templateRender.code
        }
        // 设置浏览器响应头为js
        ctx.type = 'application/javascript'
        // 将字符串转换成数据流传给下一个中间件。
        ctx.body = stringToStream(code);
    }
    await next()
})

// 修改第三方模块路径
app.use(async (ctx, next) => {
    if (ctx.type === 'application/javascript') {
        const contents = await stream2string(ctx.body);
        // 将body中导入的路径修改一下，重新赋值给body返回给浏览器
        // import vue from 'vue', 匹配到from '修改为from '@modules/
        ctx.body = contents.replace(/(from\s+['"])(?![\.\/])/g, '$1/@modules/').replace(/process\.env\.NODE_ENV/g, '"development"');
    }
})

app.listen(5000)

console.log('服务器已经启动 http://localhost:5000')

```
