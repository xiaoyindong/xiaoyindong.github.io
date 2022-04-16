## 1. 概述

```Async Hooks``` 是 ```Node8``` 新出来的特性，提供了一些 ```API``` 用于跟踪 ```NodeJs``` 中的异步资源的生命周期，属于 ```NodeJs``` 内置模块，可以直接引用。

```js
const async_hooks = require('async_hooks');
```

这是一个很少使用的模块，为什么会有这个模块呢？

我们都知道，```JavaScript```在设计之初就是一门单线程语言，这和他的设计初衷有关，最初的```JavaScript```仅仅是用来进行页面的表单校验，在低网速时代降低用户等待服务器响应的时间成本。随着```Web```前端技术的发展，虽然前端功能越来越强大，越来越被重视，但是单线程似乎也没有什么解决不了的问题，相比较而言多线程似乎更加的复杂，所以单线程依旧被沿用至今。

既然J```avaScript```是单线程，但是在日常开发中总是会有一些比较耗时的任务，比如说定时器，再比如说如今已经标准化的```Ajax```，```JavaScript```为了解决这些问题，将自身分为了``BOM``，```DOM```，```ECMAScript```，```BOM```会帮我们解决这些耗时的任务，称之为异步任务。

正因为浏览器的```BOM```帮我们处理了异步任务，所以大部分的程序员对异步任务除了会用几乎一无所知，比如同时有多少异步任务在队列中？异步是否拥堵等，我们都是没有办法直接获得相关信息的，很多情况下，底层确实也不需要我们关注相关的信息，但如果我们在某些情况下想要相关信息的时候，```NodeJS```提供了一个```Experimental```的```API```供我们使用，也就是```async_hooks```。为什么是```NodeJS```呢，因为只有在```Node```中定时器，```http```这些异步模块，才是开发者可以控制的，浏览器中的```BOM```是不被开发者控制的，除非浏览器提供对应的```API```。

## 2. async_hooks规则

```async_hooks```约定每一个函数都会提供一个上下文，我们称之为```async scope```，每一个```async scope```中都有一个 ```asyncId```, 是当前```async scope```的标志，同一个的```async scope```中```asyncId```必然相同。

这在多个异步任务并行的时候，```asyncId```可以使我们可以很好的区分要监听的是哪一个异步任务。

```asyncId```是一个自增的不重复的正整数，程序的第一个```asyncId```必然是```1```。

```async scope```通俗点来说就是一个不能中断的同步任务，只要是不能中断的，无论多长的代码都共用一个```asyncId```，但如果中间是可以中断的，比如是回调，比如中间有```await```，都会创建一个新的异步上下文，也会有一个新的```asyncId```。

每一个```async scope```中都有一个```triggerAsyncId```表示当前函数是由那个```async scope```触发生成的；

通过 ```asyncId``` 和 ```triggerAsyncId``` 我们可以很方便的追踪整个异步的调用关系及链路。

```async_hooks.executionAsyncId()```用于获取```asyncId```，可以看到全局的```asyncId```是```1```。

```async_hooks.triggerAsyncId()```用于获取```triggerAsyncId```，目前值为```0```。

```js
const async_hooks = require('async_hooks');
console.log('asyncId:', async_hooks.executionAsyncId()); // asyncId: 1
console.log('triggerAsyncId:', async_hooks.triggerAsyncId()); // triggerAsyncId: 0
```

我们这里使用```fs.open```打开一个文件，可以发现```fs.open```的```asyncId```是```7```，而```fs.open```的```triggerAsyncId```变成了```1```，这是因为```fs.open```是由全局调用触发的，全局的```asyncId```是```1```。

```js
const async_hooks = require('async_hooks');
console.log('asyncId:', async_hooks.executionAsyncId()); // asyncId: 1
console.log('triggerAsyncId:', async_hooks.triggerAsyncId()); // triggerAsyncId: 0
const fs = require('fs');
fs.open('./test.js', 'r', (err, fd) => {
    console.log('fs.open.asyncId:', async_hooks.executionAsyncId()); // 7
    console.log('fs.open.triggerAsyncId:', async_hooks.triggerAsyncId()); // 1
});
```

## 3. 异步函数的生命周期

当然实际应用中的```async_hooks```并不是这样使用的，他正确的用法是在所有异步任务创建、执行前、执行后、销毁后，触发回调，所有回调会传入```asyncId```。

我们可以使用```async_hooks.createHook```来创建一个异步资源的钩子，这个钩子接收一个对象作为参数来注册一些关于异步资源生命周期中可能发生事件的回调函数。每当异步资源被创建/执行/销毁时这些钩子函数会被触发。

```js
const async_hooks = require('async_hooks');

const asyncHook = async_hooks.createHook({
  init(asyncId, type, triggerAsyncId, resource) { },
  destroy(asyncId) { }
})

```

目前 ```createHook``` 函数可以接受五类 ```Hook Callbacks``` 如下：

### 1.init(asyncId, type, triggerAsyncId, resource)

### 2. init 回调函数一般在异步资源初始化的时候被触发。

### 3. asyncId: 每一个异步资源都会生成一个唯一性标志

### 4. type: 异步资源的类型，一般都是资源的构造函数的名字。

```txt
FSEVENTWRAP, FSREQCALLBACK, GETADDRINFOREQWRAP, GETNAMEINFOREQWRAP, HTTPINCOMINGMESSAGE,
HTTPCLIENTREQUEST, JSSTREAM, PIPECONNECTWRAP, PIPEWRAP, PROCESSWRAP, QUERYWRAP,
SHUTDOWNWRAP, SIGNALWRAP, STATWATCHER, TCPCONNECTWRAP, TCPSERVERWRAP, TCPWRAP,
TTYWRAP, UDPSENDWRAP, UDPWRAP, WRITEWRAP, ZLIB, SSLCONNECTION, PBKDF2REQUEST,
RANDOMBYTESREQUEST, TLSWRAP, Microtask, Timeout, Immediate, TickObject
```

triggerAsyncId: 表示触发当前异步资源被创建的对应的 ```async scope``` 的 ```asyncId```

#### 1. resource

代表被初始化的异步资源对象


我们可以通过 ```async_hooks.createHook``` 函数来注册关于每个异步资源在生命周期中发生的 ```init```/```before```/```after```/```destory```/```promiseResolve``` 等相关事件的监听函数；
同一个 ```async scope``` 可能会被调用及执行多次，不管执行多少次，其 ```asyncId``` 必然相同，通过监听函数，我们很方便追踪其执行的次数及时间及上线文关系；

#### 2. before(asyncId)

```before```函数一般在 ```asyncId``` 对应的异步资源操作完成后准备执行回调前被调用，```before```回调函数可能被执行多次，由其被回调的次数来决定，使用时这里需要注意。

#### 3. after(asyncId)

```after```回调函数一般在异步资源执行完回调函数后会立即被调用，如果在执行回调函数的过程中发生未捕获的异常，```after``` 事件会在触发 ```uncaughtException``` 事件后被调用。

#### 4. destroy(asyncId)

当```asyncId```对应的异步资源被销毁时调用，有些异步资源的销毁要依赖垃圾回收机制，所以有些情况下由于内存泄漏的原因，```destory```事件可能永远不会被触发。

#### 5. promiseResolve(asyncId)

当 ```Promise``` 构造器中的 ```resovle``` 函数被执行时，```promiseResolve``` 事件被触发。有些情况下，有些 ```resolve``` 函数是被隐式执行的，比如 ```.then``` 函数会返回一个新的 ```Promise```，这个时候也会被调用。

```js
const async_hooks = require('async_hooks');

// 获取当前执行上下文的 asyncId
const eid = async_hooks.executionAsyncId();

// 获取触发当前函数的 asyncId
const tid = async_hooks.triggerAsyncId();

// 创建新的AsyncHook实例。所有这些回调都是可选的
const asyncHook =
    async_hooks.createHook({ init, before, after, destroy, promiseResolve });

// 需要显示声明 才能执行
asyncHook.enable();

// 禁止监听新的异步事件。
asyncHook.disable();

function init(asyncId, type, triggerAsyncId, resource) { }

function before(asyncId) { }

function after(asyncId) { }

function destroy(asyncId) { }

function promiseResolve(asyncId) { }
```

## 4. Promise

```promise```是比较特殊的一种情况，如果足够细心```init```方法中的```type```中你就会发现其中并没有```PROMISE```。如果仅使用```ah.executionAsyncId()```来获取```Promise```的的```asyncId```的话，是不能取得正确的```ID```的，只有在添加了实际的```hook```后，```async_hooks```才会给```Promise```的回调创建```asyncId```。

换句话说，由于```V8```对于获取 ````asyncId```` 的执行成本比较高，所以默认情况下，我们是不给 ```Promise``` 分配新的 ```asyncId```。

也就是说默认情况下，我们使用```promises```或者 ```async/await``` 时是获取不到当前上下文正确的```asyncId```和```triggerId```。不过没关系，我们可以通过执行```async_hooks.createHook(callbacks).enable()```函数强制开启对```Promise```分配```asyncId```。

```js
const async_hooks = require('async_hooks');

const asyncHook = async_hooks.createHook({
  init(asyncId, type, triggerAsyncId, resource) { },
  destroy(asyncId) { }
})
asyncHook.enable();
```

```js
Promise.resolve(123).then(() => {
  console.log(`asyncId ${async_hooks.executionAsyncId()} triggerId ${async_hooks.triggerAsyncId()}`);
});
```

另外```Promise```只会触发```init```和```promiseResolve```钩子事件函数，而```before```和```after```事件的钩子函数只会在```Promise```的链式调用时被触发，也就是说只有在```.then/.catch```函数中生成的```Promise```时才会被触发。

```js
new Promise(resolve => {
    resolve(123);
}).then(data => {
    console.log(data);
})
```

可以发现，上面的存在两个```Promise```，第一个是```new```实例化创建的，第二个是```then```创建的(不明白的可以查看之前的```Promise```源码文章)。

这里的顺序是执行```new Promise```的时候会调用自身的```init```函数，然后在执行```resolve```的时候调用```promiseResolve```函数。接着在```then```方法中执行第二个```Promise```的```init```函数，然后执行第二个```Promise```的```before```，```promiseResovle```，```after```函数。

## 5. 异常处理

如果注册的```async-hook```回调函数中发生异常，那么服务将打印错误日志并立即退出，同时所有的监听器将被移除，同时会触发 ```exit``` 事件退出程序。

之所以会立即退出进程，是因为如果这些```async-hook``` 函数运行不稳定，下一个相同事件被触发时很可能又抛出异常，这些函数主要就是为了监听异步事件的，如果不稳定应该及时发现并进行更正。

## 6. 在异步钩子回调中打印日志

由于 ```console.log``` 函数也是一个异步调用，如果我们在 ```async-hook``` 函数中再调用 ```console.log``` 那么将再次触发相应的 ```hook``` 事件，造成死循环调用，所以我们在 ```async-hook``` 函数中必须使用同步打印日志方式来跟踪，可以使用 ```fs.writeSync``` 函数：

```js
const fs = require('fs');
const util = require('util');

function debug(...args) {
  fs.writeFileSync('log.out', `${util.format(...args)}\n`, { flag: 'a' });
}
```

### 参考来源

- [1] [AsyncHooksn](https://nodejs.org/dist/latest-v15.x/docs/api/async_hooks.html) https://nodejs.org/dist/latest-v15.x/docs/api/async_hooks.html
