## 1. 基本使用

项目中想要使用```grunt```的话，首先需要安装他。

```s
yarn add grunt --dev
```

安装过后需要在项目跟目录添加```gruntfile.js```文件作为入口文件，用于定义一些需要```grunt```自动执行的任务。这个文件导出一个函数，函数接收一个```grunt```参数，参数是一个对象，对象中就是```grunt```提供的一些```api```。比如借助```registerTask```方法注册任务。

这个方法第一个参数指定任务名字，第二个参数指定任务函数。

```js
module.exports = grunt => {
    grunt.registerTask('foo', () => {
        console.log('hello grunt');
    })
}
```

```s
yarn grunt foo
```

```foo```就是注册任务的名字，```grunt```会自动的执行```foo```任务，当然也可以添加更多的任务。

如果添加任务的时候第二个参数指定一个字符串，这个字符串就是这个任务的描述，他会出现在```grunt```的帮助信息中。可以通过```grunt --help```得到```grunt```的帮助信息，帮助信息中有一个```avaible tasks```，在这个```tasks```当中任务描述就是自定义的任务描述。

```js
module.exports = grunt => {
    grunt.registerTask('foo', () => {
        console.log('hello grunt');
    })
    grunt.registerTask('bar', '任务描述', () => {
        console.log('other task');
    })
}
```

如果在注册任务的时候任务名称叫做```default```，那这个任务将会成为```grunt```的默认任务，在运行任务的时候不需要指定任务的名称，```grunt```将自动调用```default```。

```js
module.exports = grunt => {
    grunt.registerTask('foo', () => {
        console.log('hello grunt');
    })
    grunt.registerTask('bar', '任务描述', () => {
        console.log('other task');
    })
    grunt.registerTask('default', () => {
        console.log('default task');
    })
}
```

一般会用```default```去映射一些其他的任务，一般的做法是在```registerTask```函数第二个参数传入一个数组，这个数组中可以指定一些任务的名字。

```js
module.exports = grunt => {
    grunt.registerTask('foo', () => {
        console.log('hello grunt');
    })
    grunt.registerTask('bar', '任务描述', () => {
        console.log('other task');
    })
    grunt.registerTask('default', ['foo', 'bar']);
}
```

这时执行```default```就会依次执行数组中的任务。

```s
yarn grunt
```

```grunt```代码默认支持同步模式，如果需要异步操作必须使用```this```的```async```方法得到一个回调函数，在异步操作完成后调用这个回调函数，标识一下这个任务已经被完成。

```js
module.exports = grunt => {
    grunt.registerTask('foo', () => {
        console.log('hello grunt');
    })
    grunt.registerTask('bar', '任务描述', () => {
        console.log('other task');
    })
    grunt.registerTask('default', ['foo', 'bar']);

    grunt.registerTask('async-task', function () {
        const done = this.async();
        setTimeout(() => {
            console.log('async task');
            done();
        }, 1000)
    })
}
```

```s
yarn grunt async-task
```

## 2. 标记任务失败

如果在构建任务的逻辑代码当中发生错误，例如需要的文件找不到了，此时就可以将这个任务标记为一个失败的任务。具体实现可以通过在函数体当中```return false来```实现。

```js
module.exports = grunt => {
    grunt.registerTask('bad', () => {
        console.log('bad working~')
        return false
    })
}
```

如果这个任务是在任务列表当中那这个任务的失败会导致后续任务不在被执行。例如这里有多个任务通过```default```连在一起，正常情况他们会依次执行，但是当```bad```失败的时候```bar```也就不执行了。

```js
module.exports = grunt => {
    grunt.registerTask('bad', () => {
        console.log('bad working~')
        return false
    })
    grunt.registerTask('foo', () => {
        console.log('foo task~')
        return false
    })
    grunt.registerTask('bar', () => {
        console.log('bar task~')
        return false
    })
    grunt.registerTask('default', ['foo', 'bad', 'bar']);
}
```

运行的时候使用```--force```可以强制执行所有任务，使用```--force```之后即使```bad```任务执行失败了也是会正常去执行```bar```。

```s
yarn grunt default --force
```

如果任务是一个异步任务就没有办法直接通过```return false```标记任务失败，需要给异步的回调函数指```false```实参标记任务失败。

```js
module.exports = grunt => {
    grunt.registerTask('bad-async', function() {
        const done = this.async();
        setTimeout(() => {
            console.log('bad async');
            done(false)
        }, 1000)
    })
}
```

## 3. 配置方法

```grunt```还提供一个用于添加任务选项的```API```叫做```initConfig```，例如使用```grunt```压缩文件时就可以通过这种方式配置压缩的文件路径。

这个方法接收一个对象形式的参数，对象的属性名一般与任务名称保持一致，属性的值他可以是任意类型的数据。有了这个配置属性就可以在任务中使用这个配置属性。

这里注册一个叫做```foo```的任务，在任务中通过```grunt```提供的```config```方法获取这个配置，```config```方法接收一个字符串参数，这个参数就是```initConfig```中指定的字符串名字。

```js
module.exports = grunt => {
    grunt.initConfig({
        foo: 'bar'
    })

    grunt.registerTask('foo', () => {
        const result = grunt.config('foo');
        console.log(result);
    })
}
```

在执行任务的时候就会获取到```initConfig```的内容。如果```foo```是对象的话，在```config```可以通过```.```的方式获取到属性值。

```js
module.exports = grunt => {
    grunt.initConfig({
        foo: {
            bar: 123
        }
    })

    grunt.registerTask('foo', () => {
        const result = grunt.config('foo.bar');
        console.log(result);
    })
}
```

```s
grunt foo
```

## 4. 多目标任务

多目标形式任务可以理解为子任务的概念，这种形式的任务在通过```grunt```实现各种构建任务时非常有用。多目标任务需要通过```grunt```中的```registerMultiTask```方法定义，这个方法同样接收两个参数，第一个是任务名字，第二个是函数。

```js
module.exports = grunt => {
    grunt.registerMultiTask('build', function() {
        console.log('task');
    })
}
```

```s
grunt build
```

使用这种多任务需要配置任务目标，配置方式通过```initConfig```去配置，需要指定一个与任务名称同名的属性也就是build，并且属性值必须是个对象，对象中每个属性的名字就是目标名称。

这相当于为```build```任务添加了两个目标，一个是```css```一个是```js```。此时在运行的时候会执行两个任务，也就是```build```任务有两个目标一个```js```一个```css```。


```js
module.exports = grunt => {
    grund.initConfig({
        build: {
            css: '1',
            js: '2'
        }
    })

    grunt.registerMultiTask('build', function() {
        console.log('task');
    })
}
```

```s
grunt build
```

如果需要执行指定目标的时候可以通过````build:css````。

```s
grunt build:css
```

在这个任务函数中可以通过```this.target```拿到当前执行的目标名字，还可以通过```this```中```data```拿到这个```target```对应的数据。

```js
module.exports = grunt => {
    grund.initConfig({
        build: {
            css: '1',
            js: '2'
        }
    })

    grunt.registerMultiTask('build', function() {
        console.log(`${this.target} ${this.data}`);
    })
}
```

需要注意的是在```build```中指定的每一个属性的键都会成为一个目标，除了指定的```options```以外，在```options```当中指定的信息会作为任务的配置选项出现。可以通过```this.options()```拿到配置选项。

```js
module.exports = grunt => {
    grund.initConfig({
        build: {
            options: {
                foo: 'bar'
            },
            css: '1',
            js: '2'
        }
    })

    grunt.registerMultiTask('build', function() {
        console.log(`${this.target} ${this.data} ${this.options()}`);
    })
}
```

除了可以在任务中添加这个选项还可以在目标中添加，如果目标也是一个对象那么这个属性中也可以添加一个```options```，添加之后会覆盖掉对象中的```options```。

```js
module.exports = grunt => {
    grund.initConfig({
        build: {
            options: {
                foo: 'bar'
            },
            css: {
                options: {
                    foo: 'baz'
                }
            },
            js: '2'
        }
    })

    grunt.registerMultiTask('build', function() {
        console.log(`${this.target} ${this.data} ${this.options()}`);
    })
}
```

## 5. 插件

插件是```grunt```的核心存在的原因也非常简单，因为很多构建任务都是通用的，例如在项目中需要压缩代码，所以社区当中就出现了很多预设插件，那这些插件内部都封装了很多通用的构建任务。一般情况下构建过程都是由这些通用的构建任务组成的。

使用插件的过程非常简单，大体是先通过```npm```去安装这个插件，再到```gruntfile```中载入这个插件提供的一些任务，最后根据插件的文档完成配置选项。

这里通过grunt-contrib-clean插件自动清除项目开发过程中产生的一些临时文件。

```s
yarn add grunt-contrib-clean
```

在```grunt```中通过```grunt.loadNpmTasks```的方式加载这个插件中提供的一些任务。绝大多数情况下```grunt```的命名规范都是```grunt-contrib-taskname```，所以这里的clean插件提供的任务名称应该就叫```clean```。

```clean```是一种多目标任务，需要使用```config```的方式配置不同的目标。

```js
module.exports = grunt => {
    grunt.initConfig({
        clean: {
            temp: 'temp/app.js'
        }
    })

    grunt.loadNpmTask('grunt-contrib-clean');
}
```

运行```clean```命令的时候就会删除掉```temp/app.js```文件。除了具体文件还可以使用通配符的方式通配一些文件类型。例如可以删除所有的```txt```文件，也可以使用```**```删除```temp```下所有的文件。

```js
module.exports = grunt => {
    grunt.initConfig({
        clean: {
            temp: 'temp/*.txt'
        }
    })

    grunt.loadNpmTask('grunt-contrib-clean');
}
```

## 6. 常用插件

```grunt```官方提供了一个```sass```模块，但是那个模块需要本机安装```sass```环境，使用起来很不方便。这里使用的```grunt-sass```是一个```npm```模块，他在内部会通过```npm```的形式去依赖```sass```。这在使用起来不需要对机器有任何的环境要求。

使用的方式也是需要先安装他。```grunt-sass```需要有一个```sass```模块支持，这里使用的是```sass```官方提供的```npm```模块, 把这两个模块安装到开发依赖当中。

```s
yarn add grunt-sass sass --dev
```

安装之后通过```grunt.loadNpmTasks```载入```grunt-sass```中提供的任务。还要在```grunt.initConfig```中配置多任务目标, ```main```中需要指定```sass```输入文件。以及最终输出的```css```文件的路径。可以通过```files```属性键是输出文件路径，值就是原路径也就是```sass```文件的路径。

还需要添加一个```options```的```implementation```模块，指定依赖的模块。

```js
const sass = require('sass');

module.exports = grunt = > {
    grunt.initConfig({
        sass: {
            options: {
                implementation: sass
            },
            main: {
                files: {
                    'dist/css/main.css': 'src/scss/main.scss'
                }
            }
        }
    })

    grunt.loadNumTasks('grunt-sass')
}
```

开发中常用的插件还有```babel```，一般通过他把```ES6```的语法转换为浏览器可识别的```ES5```, 在```grunt```中如果想要使用```babel```可以使用一个叫做```grunt-babel```的插件。

```s
yarn add grunt-babel @babel/core @babel/preset-env --dev
```

也是需要使用```loadNpmTasks```加载```grunt-babel```的任务。随着任务越来越多```loadNpmTasks```也会越来越多，这个时候社区有一个模块可以减少```loadNpmTasks```的使用叫做```loadgruntTasks```模块。

```s
yarn add load-grunt-tasks --dev
```

安装之后可以先导入这个模块, 这样就不需要每次重复的导入插件了，可以直接通过```loadgruntTasks```调用自动加载```grunt```的所有插件。

```js
const sass = require('sass');
const loadgruntTasks = require('load-grunt-tasks');

module.exports = grunt = > {
    grunt.initConfig({
        sass: {
            options: {
                implementation: sass
            },
            main: {
                files: {
                    'dist/css/main.css': 'src/scss/main.scss'
                }
            }
        }
    })

    loadgruntTasks(grunt);
    // grunt.loadNumTasks('grunt-sass')
}
```

可以为```grunt-babel```提供一些任务配置，首先也是配置输入和输出路径。需要为```babel```设置一个```options```设置的是```babel```在转换时候的```preset```。还可以为```js```文件生成```source-map```。

```js
const sass = require('sass');
const loadgruntTasks = require('load-grunt-tasks');

module.exports = grunt = > {
    grunt.initConfig({
        sass: {
            options: {
                implementation: sass
            },
            main: {
                files: {
                    'dist/css/main.css': 'src/scss/main.scss'
                }
            }
        },
        babel: {
            options: {
                sourceMap: true,
                presets: ['@babel/preset-env']
            },
            main: {
                files: {
                    'dist/css/app.js': 'src/scss/app.js'
                }
            }
        }
    })

    loadgruntTasks(grunt);
    // grunt.loadNumTasks('grunt-sass')
}
```

还有一个特性是当文件修改之后自动编译，需要```grunt-contrib-watch```插件。

```s
yarn add grunt-contrib-watch --dev
```

安装后```loadgruntTasks```会自动把这个任务加载进来。```watch```可以配置不同的目标，例如这里配置一个```js```目标，```files```是一个数组，因为他不需要输出目标，只需要设置监听目标就可以了。还需要设置一个```tasks```，也就是当文件发生改变要执行什么任务, 这里写上```babel```。

```js
const sass = require('sass');
const loadgruntTasks = require('load-grunt-tasks');

module.exports = grunt = > {
    grunt.initConfig({
        watch: {
            js: {
                files: ['src/js/*.js'],
                tasks: ['babel']
            }
        }
    })

    loadgruntTasks(grunt);
    // grunt.loadNumTasks('grunt-sass')
}
```

这个时候一旦文件发生变化，就会执行```babel```任务。

一般会给```watch```做一个映射，是为了确保再启动的时候先执行一下```sass```和```babel```任务，最后在执行```watch```任务。

```js
const sass = require('sass');
const loadgruntTasks = require('load-grunt-tasks');

module.exports = grunt = > {
    grunt.initConfig({
        watch: {
            js: {
                files: ['src/js/*.js'],
                tasks: ['babel']
            }
        }
    })

    loadgruntTasks(grunt);
    
    grunt.registerTask('default', ['sass', 'babel', 'watch']);
}
```

这三个小插件实际上是使用```grunt```工具最常用的插件。```grunt```已经退出历史舞台了。
