## 1. 概述

脚手架工具实际上就是一个```node-cli```应用，创建脚手架就是创建```node-cli```应用，首先通过```mkdir```创建一个工具目录。

```js
mkdir samlpe-cli
cd sample-cli
```

在这个目录下面通过```yarn init```初始化一个```package.json```文件

```js
yarn init
```

紧接着需要在```package.json```中添加```bin```字段，用于指定```cli```应用的入口文件。

```json
{
  "name": "sample-cli",
  "bin": "cli.js",
}
```

添加```cli.js```文件，入口文件必须要有一个特定的文件头，也就是在这个文件顶部写上这样一句话 ```#! /usr/bin/env node```。

```js
#! /usr/bin/env node

console.log('cli working')
```

如果操作系统是```linux```或者```mac```需要修改这个文件的读写权限，把他修改成```755```，这样才可以作为```cli```的入口文件执行。

通过```yarn link```将这个模块映射到全局。

```js
yarn link
```

这个时候就可以在命令行执行```sample-cli```命令了，通过执行这个命令```console.log```成功打印出来，表示代码执行了，也就意味着```cli```已经可以运行了。

```js
sample-cli
```

## 2. 实现一个简单脚手架


首先需要通过命令行交互的的方式去询问用户的一些信息，然后根据用户反馈回来的结果生成文件。在```node```中发起命令行交互询问可以使用```inquirer```模块。

```s
yarn add inquirer --dev
```

```inquirer```模块提供一个prompt方法用于发起一个命令行的询问，可以接收一个数组参数，数组中每一个成员就是一个问题，可以通过```type```指定问题输入方式，然后```name```指定返回值的键，```message```指定屏幕上给用户的一个提示，在```promise```的```then```里面拿到这个问题接收到用户的答案。

```js
const inquirer = require('inquirer');

inquirer.prompt([
    {
        type: 'input',
        name: 'name',
        message: 'Project name'
    }
]).then(answer => {
    console.log(answer);
})
```

那有了```inquirer```之后要考虑的就是动态的去生成项目文件，一般会根据模板去生成，所以在项目的跟目录下新建一个```templates```目录，在这个目录下新建一些模板。

```index.html```

```html
<head>
    <title><%= name %></title>
</head>
```

```style.css```

```css
body {
    margin: 0;
    background-color: red;
}
```

模板的目录应该是项目当前目录的```templates```通过```path```获取。

```js
const path = require('path');

// 工具当前目录
const tmplDir = path.join(__dirname, 'templates');
```

输出的目标目录一般是命令行所在的路径也就是```cwd```目录。

```js
const path = require('path');

// 工具当前目录
const tmplDir = path.join(__dirname, 'templates');
// 命令行所在目录
const destDir = process.cwd();

```

明确这两个目录就可以通过```fs```模块读取模板目录下一共有哪些文件。把这些文件全部输入到目标目录，通过```fs```的```readDir```方法自动扫描目录下的所有文件。

```js
fs.readdir(tmplDir, (err, files) => {
    if (err) {
        throw err;
    }
    files.forEach(file => {
        console.log(file); // 得到每个文件的相对路径
    })
})
```

可以通过模板引擎渲染路径对应的文件比如```ejs```。

```js
yarn add ejs --dev
```

回到代码中引入模板引擎，通过模板引擎提供的```renderFile```渲染路径对应的文件。第一个参数是文件的绝对路径，第二个参数是模板引擎在工作的时候的数据上下文，第三个参数是回调函数，也就是在渲染成功过后的回调函数，当然如果在渲染过程中出现了意外可以通过```throw err```的方式错误抛出去。

```js
const fs = require('fs');
const path = require('path');
const inquirer = require('inquirer');
const ejs = require('ejs');

// 工具当前目录
const tmplDir = path.join(__dirname, 'templates');
// 命令行所在目录
const destDir = process.cwd();

inquirer.prompt([
    {
        type: 'input',
        name: 'name',
        message: 'Project name'
    }
]).then(answer => {
    fs.readdir(tmplDir, (err, files) => {
        if (err) {
            throw err;
        }
        files.forEach(file => {
            ejs.renderFile(path.join(tmplDir, file), answer, (err, result) => {
                if (err) {
                    throw err;
                }
                console.log(result);
            })
        })
    })
})

```

运行脚手架工具。

```js
sample-cli
```

此时打印出来的这个结果其实是已经经过模板引擎工作过后的结果，只需要将这个结果通过文件写入的方式写入到目标目录就可以了，目标目录应该是通过```path.join```把```destDir```以及```file```做一个拼接，内容就是```result```。

```javascript
files.forEach(file => {
    ejs.renderFile(path.join(tmplDir, file), answer, (err, result) => {
        if (err) {
            throw err;
        }
        fs.writeFileSync(path.join(destDir, file), result);
    })
})

```

完成过后找到一个新的目录使用脚手架。

```js
sample-cli
```

输入项目名称过后，就会发现他会把模板里面的文件自动生成到对应的目录里面，至此就已经完成了一个非常简单，非常小型的一个脚手架应用。其实脚手架的工作原理并不复杂，但是他的意义却是很大的，因为他确实在创建项目环节大大提高了效率。

## 3. 发布

可以将自己的工具发布至```npm```上，提供给更多的人使用。发布```npm```非常的简单，首先需要注册```npm```账号，有两种方式可以注册，一种是登录```npm```官网```https://www.npmjs.com/```，另一种是使用命令```npm adduser```。

```js
npm adduser
```

会提示你输入用户名，密码，以及邮箱，注册好后登录```npm```账号。

```js
npm login
```

依次输入用户名、密码和邮箱。

登录成功后执行```npm publish```发布命令。

```js
npm publish
```

如果报错：```'You do not have permission to publish "samlpe-cli". Are you logged in as the correct user?'```，表示包```samlpe-cli```名字已经在包管理器已经存在被别人用了，需要更该包名称可以前往```package.json```中的```name```中换一个名字。

```json
{
  "name": "sample-cli1",
  "version": "1.0.0",
  "bin": "cli.js",
  ...
}
```

再次执行```publish```命令出现```+sample-cli1@1.0.0```即表示发布成功。

如果发布时报错：```no_perms Private mode enable, only admin can publish this module:```表示当前不是原始镜像，要切换回原始的npm镜像。

```s
npm config set registry https://registry.npmjs.org/
```

如果需要更新工具，只要继续执行```npm publish```就可以更新发布了，不过需要注意，每次发布都需要修改版本号```version```的值，同一个版本不允许发布两次。而且版本号的值最好是递增的。

```json
{
  "name": "sample-cli1",
  "version": "1.0.1",
  "bin": "cli.js",
  ...
}
```

如果想要撤销本次发布可以执行，只有在发包的```24```小时内才允许撤销发布的包，超过```24```小时就无法撤回了。

```js
npm unpublish
```
