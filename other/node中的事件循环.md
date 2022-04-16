## 正文

浏览器中存在两个任务队列，一个是```宏任务```一个是```微任务```。但是在```NodeJS```中一共存在```六个事件队列```，```timers```，```pending callbacks```，```idle prepare```，```poll```，```check```，```close callbacks```。每一个队列里面存放的都是回调函数```callback```。

这六个队列是按顺序执行的。每个队列负责存储不同的任务。

```timer```里面存在的是```setTimeout```与```setInterval```的回调函数

```pending callback```是执行操作系统的回调，例如```tcp```,```udp```。

```idle``` 和 ```prepare```只在系统内部进行使用。一般开发者用不到

```poll```执行与```IO```相关的回调操作

```check```中存放```setImmediate```中的回调。

```close callbacks```执行```close```事件的回调。

在```Node```中代码从上到下同步执行，在执行过程中会将不同的任务添加到相应的队列中，比如说```setTimeout```就会放在```timers```中, 如果遇到```文件读写```就放在```poll```里面，等到整个同步代码执行完毕之后就会去执行满足条件的微任务。可以假想有一个队列用于存放微任务，这个队列和前面的六种没有任何关系。

当同步代码执行完成之后会去执行满足条件的微任务，一旦所有的微任务执行完毕就会按照上面列出的顺序去执行队列当中满足条件的宏任务。

首先会执行```timers```当中满足条件的宏任务，当他将```timers```中满足的任务执行完成之后就会去执行队列的切换，在切换之前会先去清空微任务列表中的微任务。

所以微任务执行是有两个时机的，第一个时机是所有的同步代码执行完毕，第二个时机队列切换前。

注意在微任务中```nextTick```的执行优先级要高于```Promise```，这个只能死记了。

```js
setTimeout(() => {
    console.log('s1');
})

Promise.resolve().then(() => {
    console.log('p1');
})

console.log('start');

process.nextTick(() => {
    console.log('tick');
})

setImmediate(() => {
    console.log('st');
})

console.log('end');

// start end tick p1 s1 st
```

```js
// node v16.13.1
setTimeout(() => {
    console.log('s1');
    Promise.resolve().then(() => {
        console.log('p1');
    })
    process.nextTick(() => {
        console.log('t1');
    })
})

Promise.resolve().then(() => {
    console.log('p2')
})

console.log('start');

setTimeout(() => {
    console.log('s2');
    Promise.resolve().then(() => {
        console.log('p3');
    })
    process.nextTick(() => {
        console.log('t2');
    })
})

console.log('end');

// start end p2 s1 t1 p1 s2 t2 p3
```

```Node```与浏览器事件环执行是有一些不同的。

首先任务队列数不同，浏览器一般只有宏任务和微任务两个队列，而```Node```中除了微任务队列外还有```6个事件队列```。

其次微任务执行时机不同，不过他们也有相同的地方就是在同步任务执行完毕之后都会去看一下微任务是否存在可执行的。对浏览器来说每当一个宏任务执行完成之后就会清空一次微任务队列。在```Node```中只有在事件队列切换时才会去清空微任务队列。

最后在```Node```平台下微任务执行是有优先级的，```nextTick```优先于```Promise.then```, 而浏览器中则是先进先出。

```js
setTimeout(() => {
    console.log('timeout');
})

setImmediate(() => {
    console.log('immdieate');
})
```

在```Node```中时而会先输出```timeout```时而会先输出```immdieate```，这是因为```setTimeout```是需要接收一个时间参数的，如果没写就是一个```0```，我们都知道无论是在```Node```还是在浏览器，程序是不可能真的是```0```，他会受很多的因素影响。这取决于运行的环境。

如果```setTimeout```先执行就会放在```timers```队列中，这样```timeout```就会先输入，如果```setTimeout```因为某些原因后执行了，那么```check```队列中的```immdieate```就会先执行。这就是为什么时而输出```timeout```时而输出```immdieate```。

```js
const fs = require('fs');

fs.readFile('./a.txt', () => {
    setTimeout(() => {
        console.log('timeout');
    }, 0)

    setImmediate(() => {
        console.log('immdieate');
    })
})
```

这种情况就会一直先输出```immdieate```后输出```timeout```，这是因为，代码执行的时候会在```timers```里面加入```timeout```, 在```poll```中加入```fs```的回调，在```check```中加入```immdieate```。```fs```的回调执行结束之后实在```poll```队列，队列切换的时候首先会去看微任务，但是这里没有微任务就会继续向下，下面就是```check```队列而不是```timers```队列，所以```poll```清空之后会切换到```check```队列，执行```immdieate```回调。
