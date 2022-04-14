## 1. 概述

脚手架可以简单的理解为自动创建项目基础文件的工具，除了创建文件更重要的是提供给开发者一些约定和规范。通常在去开发相同类型的项目时，都会有一些相同的约定，其中会有相同的文件组织结构，相同的代码开发范式，相同的模块依赖，设置还有有一些相同的工具配置，最后甚至连一些基础代码都是一样的。

这样一来搭建新项目时有大量重复工作要做，脚手架工具就是解决这一类问题的。可以通过脚手架工具快速搭建特定类型的项目骨架，然后基于骨架进行后续的开发工作。类似```eclipse```，```visual studio```这这些```IDE```工具，他们创建项目的过程实际上就是一个脚手架的工作流程。

在前端项目创建过程当中，由于前端技术选型比较多样，又没有一个统一的标准，所以前端方向的脚手架一般不会集成在某一个IDE当中，都是以独立的工具存在的，并且相对会复杂一些，但是本质上，脚手架的目标是一样的，都是为了解决在创建项目过程中的一些复杂的工作。

目前市面上有很多成熟的脚手架工具，基本都是为特定项目服务的，例如```create-react-app```，```vue-cli```，```angular```。这些工具的实现方式大同小异，无外乎就是根据用户输入的信息创建对应的项目结构，最终生成项目的配置，不过他们一般只适用于自身所服务的那个框架项目。

还有一类就是以```Yeoman```工具为代表的通用型项目脚手架工具，可以根据一套模板生成对应的项目结构。这种类型的脚手架很灵活而且很容易扩展。

## 2. Yeoman

```Yeoman```官方的定义是一款用于创造现代化```web```应用的脚手架工具，不同于```vue-cli```工具```Yeoman```更像是一个脚手架的运行平台，可以通过```Yeoman```搭配不同的```generator```创建任何类型的项目。```Yeoman```的缺点在于他过于通用，不够专注，所以开发者更愿意使用像```Vue-cli```这样一类的脚手架。

```yo```就是```yeoman```工具模块的名字，可以使用```yarn```或者```npm```进行安装。

```s
yarn global add yo
```

要想使用```Yeoman```创建项目必须要找到对应项目类型的```generator```。例如想要生成```Node Module```项目，可以使用```generator-node```模块，使用方式是先把他通过全局范围安装的方式安装到本地。

```s
yarn global add generator-node;
```

有了这两款模块就可以使用```yarn```运行刚刚安装的```generator-node```的```gengerator```也就是生成器自动的创建一个全新的```Node Module```，可以先定位到项目所在的目录，也就是要执行命令的文件夹。

可以通过```Yeoman```命令```yo```加上刚刚安装的```generator-node```的生成器。运行特定的```generator```就是把包名字前面的```generator-```前缀去掉。

```s
yo node
```

在这个过程```Yeoman```会提出一些问题，可以在命令行当中通过交互的方式把他填写进去，首先第一个是模块名字，会自动检查包名是否可用。填写完毕会在项目中创建一些基础文件，并且在项目根目录运行```npm install```安装项目必要的依赖。在目录中除了基本的文件外内部的一些基础代码基础的配置，都是提前配置好的，这也是脚手架工具的一个优势。

## 3. 子集生成器

有时候并不需要创建完整的项目结构，可能只是需要在已有的项目基础之上去创建一些特定类型的文件，例如给一个已经存在的的项目创建一个```readme```，又或是在一个原有项目之上添加某些类型的配置文件，比如```esline```、```babel```，这些配置文件都有一些基础代码如果自己手动去写的话很容易配错，可以使用```Yeoman```提供的```sub generator```实现。

在项目目录下运行特定的```sub generator```命令生成对应的文件。这里使用```generator-node```提供的子集生成器```cli```生成一个```cli```应用所需要的文件，让模块变成```cli```应用。

```s
yo node:cli
```

运行```sub generator```的方式就是在原有```generator```命令后面跟上```sub generator```的名字。会提示是否要重写```package.json```文件，原因是在添加```cli```支持的时候会添加一些新的模块和配置选择```yes```即可。完成过后会提示重写了```package.json```创建了```lib\cli.js```。

在``package.json``中出现了一个```"bin": "lib/cli.js"```以及新的```dependencies```，这些都是```cli```应用中需要的。有了这些就可以将```cli```代码模块作为一个全局的命令行模块去使用了，本地的模块可以通过```yarn link```到全局范围。

```s
yarn link // 映射到全局
yarn // 安装依赖
my-module --help // 运行
```

并不是每一个```generator```都一个子集的生成器，所以在使用之前要通过所使用的```generator```的官方文档来明确这个```generator```下面有没有一个子集的生成器。

## 4. 自定义 Generator

不同的```generator```可以生成不同的项目，可以通过创建自己的```generator```生成自定义的项目结构。

一般```generator```基本结构

```s
|—— generators   # 生成器目录
|    app/        # 默认生成器目录
        index.js # 默认生成器实现
|—— package.json # 模块包配置文件
```

首先创建```generators```文件夹，作为生成器的目录，再在里面存放```app```文件夹用于存放生成器对应的代码。如果需要提供多个的```sub generator```可以在```app```的同级目录添加新的生成器目录```app2```模块就有了一个叫做```app2```的子生成器。

除了特定的结构，还有个与普通```npm```不同的是```Yeoman```的```generator```的模块的名称必须是```generator-<name>```这样的一种格式，如果你在具体开发的时候没有去使用这样格式的名称，```Yeoman```在后续工作的时候就没有办法找到你所提供的这个生成器模块。

可以做下具体的演示，首先创建文件夹```generator-simple```作为生成器模块的目录。

```s
mkdir generator-simple
```

在目录下通过```yo init```的方式创建一个```package.json```。

```s
yo init
```

还需要安装```yeoman-generator```的模块，这个模块提供了生成器的基类，基类中提供了一些工具函数可以在创建生成器的时候更加便捷。

```s
yarn add yeoman-generator
```

安装完工具过后通过编辑器打开目录，在目录下按照项目结构要求，创建```generators```文件夹，在这个目录下创建```app```目录，在目录中创建```index.js```作为```generator```的核心入口文件。

```index.js```需要导出一个继承自```Yeoman Generator```的类型，```Yeoman Generator```在工作时会自动调用此类型中定义的一些生命周期方法，可以在这些方法中通过调用父类提供的工具方法实现一些功能，例如文件写入。

```js
const Generator = require('yeoman-generator');

module.exports = class extends Generator {

    writing() { 
        // Yeoman自动生成文件阶段调用此方法
        // 我们尝试往项目目录中写入文件
        this.fs.write(this.destinationPath('temp.txt'), '123'); // 这里的fs模块与node中的fs不同，是高度封装的模块功能更强大一些，
    }

}
```

```this.destinationPath```是父类中方法用来获取绝对路径，```this.fs```是父类中模块。

这时一个简单的```generator```就已经完成了，通过```npm link```的方式把这个模块连接到全局范围，使之成为一个全局模块包，这样```Yeoman```在工作的时候就可以找到自己写的这个```generator-simple```模块

```s
yarn link
```

可以通过```Yeoman```运行这个生成器具体的操作方式是```yo simple```。

```s
yo simple
```

## 5. 根据模板创建文件

很多时候需要自动创建的文件有很多，而且文件的内容也相对复杂，这种情况可以使用模板创建文件。在生成器的目录下添加```template```目录，然后将需要生成的文件都放入到```template```目录中作为模板。模板中完全遵循ejs模板引擎的语法。

有了模板过后在生成文件之时就不用再借助于```fs```的```write```方法去写入文件，而是借助于```fs```中的```copytemplate```方式，使用的时候有三个参数，分别是模板文件的路径，输出文件的路径，模板数据的上下文。

模板文件路径可以借助```this.templatePath('foo.txt')```自动获取当前template目录下的文件路径。输出路径仍旧是用```this.destinationPath('foo.txt')```，模板数据上下文只需要定义一个对象就可以了用于传入```ejs```。

```js

const tmpl = this.templatePath('foo.txt');

const output = this.destinationPath('foo.txt');

const context = { title: 'yd'}

this.fs.copyTpl(tmpl, output, context);

```

运行

```s
yo simple
```

## 6. 接收用户输入数据

在```generator```中想要发起一个命令行交互询问，可以通过实现```generator```类型中的```promting```方法。

```js

const Generator = require('yeoman-generator');

module.exports = class extends Generator {
    prompting() {
    }
}

```

可以调用父类提供的```promit```方法发出对用户的命令行询问。这个方法返回```promise```，在调用的时候对他```return```，这样```Yeoman```在调用的时候就有更好的异步流程控制。

这个方法接受数组参数，数组的每一项都是一个问题对象，可以传入类型```type```、```name```、```message```和```default```。

```js
[
    {
        type: "input", // 提问类型
        name: "name", // 得到结果的一个键
        message: "message", // 命令行提示的话
        default: this.appname // appname为项目生成目录文件夹的名字，会作为问题的默认值
    }
]
```

```promise```执行过后我们会得到一个返回值，返回值就是问题的结果。会以对象的形式出现。对象的键就是输入的```name```，值就是用户输入的内容，可以将这个值挂载在全局```this```中方便日后使用

```js
const Generator = require('yeoman-generator');
module.exports = class extends Generator {
    prompting() {
        return this.prompt([
            {
                type: "input", // 提问类型
                name: "projectName", // 得到结果的一个键
                message: "message", // 命令行提示的话
                default: this.appname // appname为项目生成目录文件夹的名字，会作为问题的默认值
            }
        ]).then(answers => {
            this.answers = answers;
        })
    }
}
```

有了```this.answers```就可以在```writing```的时候传入模板引擎，使用这个数据作为模板数据的上下文。

```js
const context = this.answers;
this.fs.copyTpl(tmpl, output, context);
```

## 7. 案例演示

首先打开命令行窗口然后通过```mkdir```创建一个全新的```gengerator```目录。

```s
mkdir generator-vue
cd generator-vue
```

通过```yarn init```初始化```package.json```，然后安装```Yeoman```的依赖。

```s
yarn init
yarn add yeoman-generator
```

新建```generator```主入口文件```generators/app/index.js```。

```js
const Generator = require('yeoman-generator');

module.exports = class extends Generator {
    prompting() {
        return this.promit([
            {
                type: 'input',
                name: 'name',
                message: 'Your project name',
                default: this.appname
            }
        ]).then(answer => { // 获取到用户输入的数据
            this.answer = answer;
        })
    }
    writing() {

    }
}
```

创建```templates```目录，把项目的结构```copy```到```templates```当中作为模板，有了模板过后需要把项目结构里面一些可能发生变化的地方通过模板引擎的方式修改。

通过数组循环的方式批量生成每一个文件，把每一个文件通过模板转换，生成到对应的路径。

```js
writing() {
    const templates = [
        '.browserslistrc',
        'src/views/Home.vue'
    ]
    templayes.forEach(item => {
        this.copyTpl(this.templatePath(item),
        this.destinationPath(item),
        this.answer);
    })
}
```

将```generator-vue```通过```link```的方式定义到全局。

```s
yarn link
```

在全新的目录使用该```generator```。

```s
yo vue;
```

会提示输入项目名称，输入之后就可以看到模板被拉去到了新的目录。

## 8. 发布

```Generator```实际上就是一个```npm```的模块，发布```generator```就是发布```npm```的模块。只需要将已经写好的```generator```模块通过```npm publish```命令发布成一个公开模块就可以了。
