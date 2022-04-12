为了提高代码的可维护性，vue3的代码全部使用TypeScript重写。因为大型项目的开发都推荐使用类型化的语言，在编码的过程中检查类型问题。

vue3.0使用Monorepo的方式来管理项目源代码，使用一个项目来管理多个包，把不同功能的代码放在不同的package中管理，这样每个功能模块划分都很明确，模块之间的依赖关系都很明确，这个的话每个模块都可以单独发布单独测试单独使用。

## package目录结构

```s
packages/
    compiler-core # 和平台无关的编译器
    compiler-dom # 浏览器下的编译器，依赖于core
    compiler-sfc # 用来编译单文件组件 依赖core和dom
    compiler-ssr # 服务端渲染的编译器 依赖dom
    reactivity # 数据响应式系统
    runtime-core # 和平台无关的运行时
    runtime-test # 针对浏览器的运行时，处理原生dom的api，事件 针对测试的轻量运行时，是一颗对象dom树，所以可以运行在所有js环境中
    server-renderer # 用于服务端渲染
    shared # 内部使用的公共api
    size-check # 私有package，treeshaking之后检查包体积
    template-explorer # 浏览器运行的实时编译组件
    vue # 构建完整版vue，依赖compiler和runtime
```

## 版本更新

vue3.0在构建的时候和vue2.0类似都构建了不同的版本，和vue2不同的是vue3不在构建umd模块化的方式，因为这种方式会让代码更加冗余。

3.0的版本中将seajs，esmodules和自执行函数分别打包到了不同的文件当中，在vue的dist中存放了vue3的所有构建版本。

cjs就是commonjs的模块化方式，包含vue.cjs.js开发版和vue.cjs.prod.js的压缩生产版，他们都是完整的vue，区别只是是否压缩。

global是全局版，可以在浏览器中通过script标签导入，导入js之后会增加一个全局的vue对象。

browser是浏览器的原生版，可以在浏览器中通过script的type为module的方式导入。

bundler版本没有打包所有的代码，需要配合打包工具来使用，使用es modules的方式打包。通过脚手架默认导入的就是vue.runtime.esm-bundler.js版本，有点是体积最小，在项目打包的时候可以根据开发者使用的功能按需打包。

## Composition API

Composition学习的最好方式是查看官方的RFC，(https://github.com/vuejs/rfcs)。

Vue2在开发中小型项目的时候已经很好用了，但是使用Vue2开发需要长期迭代和维护大型项目的时候也会有一些限制，在大型项目中可能有一些功能比较复杂的组件，在看别人开发的组件的时候可能比较难以理解。

Vue2开发的组件使用的是Options API，使用一个包含描述组件选项的对象来创建组件，这种方式经常使用，创建组件的时候经常设置data, methods, props，生命周期等，这些选项组成了对象来描述组件。如果看他人开发的组件可能会难以看懂，因为同一个功能的代码会涉及data, methods等多个配置。多个功能就会导致配置凌乱。

Composition API是vue3新增的一组API，是一组基于函数的API，让我们可以更灵活的组织组件的逻辑。这样的好处是查看某个逻辑的时候只需要关注某个函数即可。比如下面的postion只需要关注usePosition函数, 不需要关注其他逻辑。

```js
import { reactive, onMounted, onUnmonted } from 'vue';
function useMousePosition () {
    const position = reactive({ x: 0, y: 0 });
    const update = (e) => {
        position.x = e.pageX;
        position.y = e.pageY;
    }
    onMounted(() => {
        window.addEventListener('mousemove', update);
    })
    onUnmounted(() => {
        window.removeEventListener('mousemove', update)
    })
    return position;
}

export default {
    setup() {
        const position = useMousePosition();
        return {
            position
        }
    }
}
```

在Option API中，同一逻辑的代码被拆分到了很多的位置不方便阅读，Composition API可以保证统一逻辑的代码聚集在同一个位置，还可以提供给其他组件使用。

Vue3中两种API都是支持的。

## 性能提升

Vue3中的性能提升可以从下面3个方面来说

1. 响应式系统升级

Vue2中响应式系统的核心使用的是Object.defineProperty这在初始化的时候会遍历data中的全部成员，通过defineProperty将属性转换成get和set，如果属性是对象的话还需要递归遍历，这些都是在初始化时定义的。

Vue3使用Proxy对象重写了响应式系统，Proexy的性能本身就比defineProperty好，另外Proxy可以拦截对象的所有操作，不需要初始化的时候遍历所有的属性，另外如果有多层属性嵌套的话只有访问某个属性的时候才会递归处理。

Proxy默认就可以监听到动态新增的属性，而Vue2中不具备这样的功能，如果动态添加需要重新使用defineProperty设置，Vue2也监听不到属性的删除对数组的索引和length属性也无法监听。

2. 优化编译

通过优化编译和重写虚拟DOM让首次渲染和更新的性能也有了大幅度的提升。

vue2中diff的过程会跳过静态根节点因为静态跟节点的内容不会发生变化，也就是vue2中通过标记根节点优化了diff的过程，但是在vue2中静态节点仍旧需要进行diff，这个过程没有优化。

vue3中为了提高性能在编译的时候会标记提升所有的静态节点，然后diff的时候只需要对比动态节点的内容，另外在Vue3中新引入了一个Fragments也就是片段的特性，模板中不需要再创建一个唯一的根节点，模板里面可以直接放文本内容，或者很多同级的标签。这个功能需要升级vetur插件。

也就是说vue3中在编译的过程中会通过标记和提升静态节点，通过patch flag将来在diff的时候会跳过静态根节点只需要更新动态节点中的内容，极大的提升了diff的性能。

通过事件处理函数的缓存减少了不必要的更新操作。

3. 源码体积

vue3中移除了一些不常用的API，比如inline-template, filter等，可以让最终代码的体积变小。filter可以通过method计算属性来实现，另外vue3对tree-shaking的支持更好，tree-shaking依赖es modules, 也就是ES6的模块化结构。

通过编译阶段的静态分析找到没有引入的模块，在打包的时候直接过滤掉，让打包后的体积更小。vue3在设计之初就考虑到了tree-shaking, 内置的组件和指令都是按需引入的。除此之外vue3中的很多api都是支持tree-shaking的。所以vue3中新增的一些api如果你没有使用的话这部分代码是不会被打包的，只会打包你所使用的api。

但是默认的核心模块都会被打包。

vue3在设计的时候就考虑到了性能的问题，通过代码的优化编译的优化或者打包来提高性能。

## Vite

伴随Vue3的推出vue的作者还推出了自己的构建工具vite。vite比过去基于webpack的cli更加快速。

介绍vite之前我们先回顾一下浏览器中使用ES Modules的方式。

除了IE的现代浏览器都已经支持了ES Module的语法，也就是使用import导入模块，使用export导出模块。在网页中可以直接通过script导入模块，只需要type设置为module。

标记为module的script标签默认是延迟加载的类似于script标签加入defer属性，type为module的标签相当于省略了defer，他是在当前文档解析完成也就是dom树生成之后并且在DOMContentLoaded事件之前执行。

```html
<script ype="module" src="./index.js"></script>
```

vite的快就是使用浏览器支持的ES Module的方式，避免开发环境下打包从而提升开发速度。

Vite在开发环境下不需要打包，因为在开发模式下Vite使用浏览器原生支持额ES Module加载模块，支持ES Module的浏览器使用script标签加载模块，正因为他不需要打包，所以开发模式打开页面基本是秒开的。

Vue-CLi在开发模式下首先会打包整个项目，如果项目比较大速度会特别慢，Vue会开启一个测试服务器，他会拦截浏览器发送的请求，浏览器会向服务器发送请求获取相应的模块，vite会对浏览器不识别的模块进行处理比如后缀名为.vue的文件，会在服务器对.vue文件进行编译将编译过后的文件返回给浏览器。

使用这样的方式让vite具备快速冷启动，按需编译，模块热更新等优点，并且模块热更新的性能与模块总数无关，无论有多少模块HMR的速度始终比较快。

vite在生产环境下使用rollup打包，rollup是基于浏览器原生的ES Module进行打包，不需要使用babel将import转换成require以及一些响应的辅助函数。因此打包的体积会比webpack打包的体积更小。现在的浏览器基本都已经支持ES Module的方式打包模块。

vite有两种创建项目的方式，一种是创建基于vue3的项目。

```s
npm init vite-app my-project
cd my-project
npm install
npm run dev
```

还有一种方式是基于模板创建项目，可以让他支持其他的框架。

```s
npm init vite-app --template react
npm init vite-app --template preact
```

vite开启的web服务器会劫持.vue的请求，首先会把.vue文件解析成.js文件，并且把响应头中的content-type设置为applicaton/javascript目的是告诉浏览器返回的是一个js脚本。

## 合成函数

Composition API是vue3新增的api，依然可以使用options api。

调用createApp函数创建Vue实例，这里接收一个对象作为参数，这里的data不再支持对象必须是一个函数。

```html
<script type="module">
    import { createApp, reactive } from './node_modules/vue/dist/vue.esm-browser.js'

    const app = createApp({
        data() {
            return position: {
                x: 0,
                y: 0
            }
        }
    })

    app.mount('#app');
</script>
```

在模板中分别将x和y显示出来。

```html
<div id="app">
x: {{ position.x }}
<br />
y: {{ position.y }}
<div>
```

createApp是创建vue对象用的他会返回vue对象, 这个对象比vue2的对象成员要少很多，而且这些成员不再使用$开头，说明未来基本不再计划给对象新增成员。

Composition API在配置项中书写，这里会用到setup函数作为Composition API的入口，setup有两个参数第一个是props他的作用是接收外部传入的参数，并且它是一个响应式的对象不能被解构。第二个参数是context是一个对象。

setup需要返回一个对象，这个对象可以使用在模板，methods，computed以及生命周期中。setup会在props解析完毕，组件实例创建之前执行。在setup内部无法通过this获取组件实例。

```js
const app = createApp({
    setup() {
        const position = {
            x: 0,
            y: 0
        }
        // 这里的position并不是响应式的对象
        return {
            position
        }
    }
    mounted() {
        console.log(this.position);
    }
})
```

- reactive

setup中定义的position并不是响应式的，不过他的返回值却可以在methods等中使用。vue3提供reactive方法来将数据变成响应式。这里不使用observerable是为了避免和rxjs重名出现混淆。

```js
import { createApp, reactive } from './node_modules/vue/dist/vue.esm-browser.js'

....
    setup() {
        const position = reactive({
            x: 0,
            y: 0
        })
        // 这里的position并不是响应式的对象
        return {
            position
        }
    }
...
```

使用Composition API方式。

```js

import { createApp, reactive, onMounted, onUnmounted } from './node_modules/vue/dist/vue.esm-browser.js'

function useMousePosition () {
    const position = reactive({ x: 0, y: 0 });
    const update = (e) => {
        position.x = e.pageX;
        position.y = e.pageY;
    }
    onMounted(() => {
        window.addEventListener('mousemove', update);
    })
    onUnmounted(() => {
        window.removeEventListener('mousemove', update)
    })
    return position;
}

const app = createApp({
    setup() {
        const position = useMousePosition();
        // 这里的position并不是响应式的对象
        return {
            position
        }
    }
})
```

Composition API中存在reactive,toRefs，ref三个函数，他们都是创建响应式数据的。

setup中的代码如果我们修改一下，如下就不再是响应式了，因为我们使用reactive包装的是position对象，这里不使用position了所有无法实现响应式。但是使用position还显得比较冗杂。

```js
setup() {
    const {x, y} = useMousePosition();
    // 这里的position并不是响应式的对象
    return {
        x,
        y
    }
}
```
- toRefs

toRefs就是解决这样一个问题的，我们可以在useMousePosition返回值的位置将position使用toRefs包裹一下，这样就可以了。

```js
import { createApp, reactive, onMounted, onUnmounted, toRefs } from './node_modules/vue/dist/vue.esm-browser.js'

function useMousePosition () {
    const position = reactive({ x: 0, y: 0 });
    const update = (e) => {
        position.x = e.pageX;
        position.y = e.pageY;
    }
    onMounted(() => {
        window.addEventListener('mousemove', update);
    })
    onUnmounted(() => {
        window.removeEventListener('mousemove', update)
    })
    return toRefs(position);
}
```

toRefs可以把一个响应式对象的成员也转换成响应式的。他要求传入的对象必须是一个代理对象也就是响应式对象。它内部会创建一个新的对象然后遍历传入的代理对象所有属性，将所有属性的值都转换成响应式对象，然后挂载到新创建的对象上然后将新创建的对象返回。

它内部会为代理对象的每一个对象属性创建一个具有value属性的对象，该对象是响应式的。value属性具有get和set，可以赋值和返回值。所以每一个属性都是响应式的。

- ref

这是一个函数，他的作用是将普通数据转换成响应式，和reactive不同的是，reactive是将对象转换成响应式，ref是将基本类型数据转换成响应式。

ref返回的是一个对象，这个对象只有一个value属性，并且value属性是有get和set的。ref返回的值访问的时候可以省略value，修改值的时候需要使用value。

```js
import { createApp, ref } from './node_modules/vue/dist/vue.esm-browser.js'

function useCount() {
    // 创建响应式
    const count = ref(0);
    return {
        count,
        add: () => {
            count.value++;
        }
    }
}

createApp({
    setup() {
        return {
            ...useCount()
        }
    }
}).mount('#app')
```

## 计算属性

计算属性的作用是简化模板中的代码，缓存计算的结果当数据变化后才会重新计算，在vue3中可以在setup中通过computed函数来创建计算属性。

- 用法一

传入一个获取值的函数，函数内部依赖响应式的数据，当依赖的数据发生变化后会重新执行该函数获取数据。返回一个不可变的响应式对象类似于使用ref创建的对象。只有一个value属性，获取计算属性的值要通过value属性来获取。模板中使用计算属性可以省略value。

```js
createApp({
    setup() {
        const count = ref(1);
        const activeCount = computed(() => {
            return count.value;
        })
        return {
            count,
            add: () => {count.value + 1}
        }
    }
})
```

- 用法二

第二种用法是传入一个对象，这个对象具有get和set返回一个不可变的响应式对象。

```js
const count = ref(1);
const plusOne = computed({
    get: () => count.value + 1,
    set: (val) => {
        count.value = val - 1;
    }
})
```

## 3. watch

和computed类似，在setup函数中可以使用watch来创建一个侦听器，使用方式和之前使用this.$watch或选项中的watch作用是一样的，监听响应式的变化执行一个回调函数，可以获取到监听数据的新值和旧值。

watch有三个参数，第一个参数是监听的数据可以是一个获取值的函数监听这个函数返回值的变化，或者直接是一个ref或者reavtive返回的对象，还可以是数组，第二个参数是监听到数据变化之后执行的函数，这个函数有两个参数分别是新值和旧值。第三个参数是选项对象，可以传入两个选项deep深度监听和immediate立即执行，这个和vue2中是一样的。

watch的返回值是一个函数，用于取消监听。

```js
createApp({
    setup() {
        const question = ref('');
        // 这里第一个参数不再是字符串，和vue2中略有不同
        watch(question, (nValue, oValue) => {
            console.log(nValue, oValue);
        })
        return question;
    }
})
```

## watchEffect

Vue3中提供一个新的函数WacthEffect, 他其实是watch函数的简化版本，内部实现和watch调用的同一个函数do watch, 不同点是watchEffect没有第二个回调函数参数。

watchEffect接收一个函数作为参数，他会监听函数内使用到的响应式数据的变化会立即执行一次这个函数。当数据变化后会重新运行该函数。返回值也是一个取消监听的函数。

```js
createApp({
    setup() {
        const count = ref(0);
        const stop = watchEffect(() => {
            console.log(count.value);
        })

        return {
            count,
            stop,
            add: () => {
                count.value + 1;
            }
        }
    }
})
```

## 响应式原理

Vue3重写了响应式系统，和Vue2相比底层采用Proxy对象实现，在初始化的时候不需要遍历所有的属性再把属性通过defineProperty转换成get和set。另外如果有多层属性嵌套的话只有访问某个属性的时候才会递归处理下一级的属性所以Vue3中响应式系统的性能要比Vue2好。

Vue3的响应式系统可以监听动态添加的属性还可以监听属性的删除操作，以及数组的索引以及length属性的修改操作。另外Vue3的响应式系统还可以作为模块单独使用。

接下来我们自己实现Vue3响应式系统的核心函数(reactive/ref/toRefs/computed/effect/track/trigger)来学习一下响应式原理。

首先我们使用Proxy来实现响应式中的第一个函数reactive。

## reactive

reactive接收一个参数，首先要判断这个参数是否是一个对象，如果不是直接返回，reactive只能将对象转换成响应式对象，这是和ref不同的地方。

接着会创建拦截器对象handler，其中抱哈get，set，deleteProperty等拦截方法，最后创建并返回Proxy对象。

```js
// 判断是否是一个对象
const isObject = val => val !== null && typeof val === 'object'
// 如果是对象则调用reactive
const convert= target => isObject(target) ? reactive(target) : target
// 判断对象是否存在key属性
const haOwnProperty = Object.prototype.hasOwnProperty
const hasOwn = (target, key) => haOwnProperty.call(target, key)

export function reactive (target) {
    if (!isObject(target)) {
        // 如果不是对象直接返回
        return target
    }

    const handler = {
        get (target, key, receiver) {
            // 收集依赖
            const result = Reflect.get(target, key, receiver)
            // 如果属性是对象则需要递归处理
            return convert(result)
        },
        set (target, key, value, receiver) {
            const oldValue = Reflect.get(target, key, receiver)
            let result = true;
            // 需要判断当前传入的新值和oldValue是否相等，如果不相等再去覆盖旧值，并且触发更新
            if (oldValue !== value) {
                result = Reflect.set(target, key, value, receiver)
                // 触发更新...
            }
            // set方法需要返回布尔值
            return result;
        },
        deleteProperty (target, key) {
            // 首先要判断当前target中是否有自己的key属性
            // 如果存在key属性，并且删除要触发更新
            const hasKey = hasOwn(target, key)
            const result = Reflect.deleteProperty(target, key)
            if (hasKey && result) {
                // 触发更新...
            }
            return result;
        }
    }
    return new Proxy(target, handler)
}
```

至此reactive函数就写完了，接着我们来编写一下收集依赖的过程。

在依赖收集的过程会创建三个集合，分别是targetMap,depsMap以及dep。

其中targetMap是用来记录目标对象和字典他使用的是weakMap，key是目标对象，targetMap的值是depsMap, 类型是Map，这里面的key是目标对象的属性名称，值是一个Set集合,集合中存储的元素是Effect函数。因为可以多次调用同一个Effect在Effect访问同一个属性，这个时候这个属性会收集多次依赖对应多个Effect函数。

一个属性可以对应多个Effect函数，触发更新的时候可以通过属性找到对应的Effect函数，进行执行。

我们这里分别来实现一下effect和track两个函数。

effect函数接收一个函数作为参数，我们首先在外面定一个变量存储callback, 这样track函数就可以访问到callback了。

```js

let activeEffect = null;
export function effect (callback) {
    activeEffect = callback;
    // 访问响应式对象属性，收集依赖
    callback();
    // 依赖收集结束要置null
    activeEffect = null;
}
```

track函数接收两个参数目标对象和属性, 他的内部要将target存储到targetMap中。需要先定义一个这样的Map。

```js

let targetMap = new WeakMap()

export function track (target, key) {
    // 判断activeEffect是否存在
    if (!activeEffect) {
        return;
    }
    // depsMap存储对象和effect的对应关系
    let depsMap = targetMap.get(target)
    // 如果不存在则创建一个map存储到targetMap中
    if (!depsMap) {
        targetMap.set(target, (depsMap = new Map()))
    }
    // 根据属性查找对应的dep对象
    let dep = depsMap.get(key)
    // dep是一个集合，用于存储属性所对应的effect函数
    if (!dep) {
        // 如果不存在，则创建一个新的集合添加到depsMap中
        depsMap.set(key, (dep = new Set()))
    }
    dep.add(activeEffect)
}
```

track是依赖收集的函数。需要在reactive函数的get方法中调用。

```js
get (target, key, receiver) {
    // 收集依赖
    track(target, key)
    const result = Reflect.get(target, key, receiver)
    // 如果属性是对象则需要递归处理
    return convert(result)
},
```

这样整个依赖收集就完成了。接着就要实现触发更新，对应的函数是trigger，这个过程和track的过程正好相反。

trigger函数接收两个参数，分别是target和key。

```js
export function trigger (target, key) {
    const depsMap = targetMap.get(target)
    // 如果没有找到直接返回
    if (!depsMap) {
        return;
    }
    const dep = depsMap.get(key)
    if (dep) {
        dep.forEach(effect => {
            effect()
        })
    }
}
```

trigger函数要在reactive函数中的set和deleteProperty中触发。

```js
set (target, key, value, receiver) {
    const oldValue = Reflect.get(target, key, receiver)
    let result = true;
    // 需要判断当前传入的新值和oldValue是否相等，如果不相等再去覆盖旧值，并且触发更新
    if (oldValue !== value) {
        result = Reflect.set(target, key, value, receiver)
        // 触发更新...
        trigger(target, key)
    }
    // set方法需要返回布尔值
    return result;
},
deleteProperty (target, key) {
    // 首先要判断当前target中是否有自己的key属性
    // 如果存在key属性，并且删除要触发更新
    const hasKey = hasOwn(target, key)
    const result = Reflect.deleteProperty(target, key)
    if (hasKey && result) {
        // 触发更新...
        trigger(target, key)
    }
    return result;
}
```

## ref

ref接收一个参数可以是原始值也可以是一个对象，如果传入的是对象并且是ref创建的对象则直接返回，如果是普通对象则调用reactive来创建响应式对象，否则创建一个只有value属性的响应式对象。

```js
export function ref (raw) {
    // 判断raw是否是ref创建的对象，如果是直接返回
    if (isObject(raw) && raw.__v__isRef) {
        return raw
    }

    // 之前已经定义过convert函数，如果参数是对象就会调用reactive函数创建响应式
    let value = convert(raw);

    const r = {
        __v__isRef: true,
        get value () {
            track(r, 'value')
            return value
        },
        set value (newValue) {
            // 判断新值和旧值是否相等
            if (newValue !== value) {
                raw = newValue
                value = convert(raw)
                // 触发更新
                trigger(r, 'value')
            }
        }
    }

    return r
}
```

## toRefs

toRefs接收reactive函数返回的响应式对象，如果不是响应式对象则直接返回。将传入对象的所有属性转换成一个类似ref返回的对象将准换后的属性挂载到一个新的对象上返回。

```js
export function toRefs (proxy) {
    // 如果是数组创建一个相同长度的数组，否则返回一个空对象
    const ret = proxy instanceof Array ? new Array(proxy.length) : {}

    for (const key in proxy) {
        ret[key] = toProxyRef(proxy, key)
    }

    return ret;
}

function toProxyRef (proxy, key) {
    const r = {
        __v__isRef: true,
        get value () { // 这里已经是响应式对象了，所以不需要再收集依赖了
            return proxy[key]
        },
        set value (newValue) {
            proxy[key] = newValue
        }
    }
    return r
}
```

toRefs的作用其实是将reactive中的每个属性都变成响应式的。reactive方法会创建一个响应式的对象，但是如果将reactive返回的对象进行解构使用就不再是响应式了，toRefs的作用就是支持解构之后仍旧为响应式。

## computed

接着再来模拟一下computed函数的内部实现

computed需要接收一个有返回值的函数作为参数，这个函数的返回值就是计算属性的值，需要监听函数内部响应式数据的变化，最后将函数执行的结果返回。

```js
export function computed (getter) {
    const result = ref()

    effect(() => (result.value = getter()))

    return result
}
```

computed函数会通过effect监听getter内部响应式数据的变化，因为在effect中执行getter的时候访问响应式数据的属性会去收集依赖，当数据变化会重新执行effect函数，将getter的结果再存储到result中。
