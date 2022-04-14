## 1. 概述

```Plop```是一款主要用于去创建项目中特定类型文件的小工具，类似于```Yeoman```中的```sub generator```, 不过它一般不会独立去使用，一般会把```Plop```集成到项目中。

日常开发中经常会需要重复创建相同类型的文件，例如每一个组件都会有三个文件去组成```js```，```css```，```test.js```。如果需要创建一个组件，就要去创建三个文件，并且每一个文件中都要有一些基础代码，这就比较繁琐，而且很难统一每一个组件文件中基础的代码。```Plop```可以解决这个问题，只需要在命令行中取运行```Plop```。

```s
yarn plop component
```

会询问一些信息，并且自动的创建一些文件，这也就保证了每次创建的文件都是统一的，并且是自动的。

## 2. 基本使用

```Plop```作为```npm```的模块安装到我们的开发依赖。

```s
yarn add plop --dev
```

安装后在项目跟目录新建```plopfile.js```文件，这个文件是```plop```工作的一个入口文件，需要导出一个函数，而且这个函数可以接收一个叫```plop```的对象，对象提供了一系列工具函数，用于创建生成器的任务。

```js
module.exports = plop => {
    plop.setGenerator('component', {});
}
```

```plop```有个成员叫```setGenerator```, 接收两个参数，第一个参数是生成器的名字，第二个参数是生成器的一些选项。配置选项中需要指定生成器的参数。

```js
{
    description: '生成器的描述',
    prompts: [ // 发出的命令行问题
        {
            type: 'input',
            name: 'name',
            message: 'component name',
            default: 'MyComponent'
        }
    ],
    actions: [ // 问题完成后的动作
        {
            type: 'add', // 添加一个全新的文件
            path: 'src/components/{{name}}/{{name}}.js', // 指定添加的文件会被添加到哪个具体的路径, 可以通过双花括号的方式使用命令行传入的变量
            templateFile: 'plop-templates/component.hbs', // 本次添加文件的母版文件是什么, 一般我们会把母版文件放在plop-template目录中，可以通过handlebars去创建模板文件.hbs
        }

    ]
}
```

数据填写完毕```Plop```就算是完成了，安装```Plop```模块的时候```Plop```提供了一个```CLI```程序，可以通过```yarn```启动这个程序```yarn plop <name>```，会执行上面定义的```Plop```。

```s
yarn plop component
```

可以添加多个模板就是添加多个```actions```，官网中提供了多个```type```，可以参考官网。

```Plop```用来去创建项目当中同类型的文件还是非常方便的。
