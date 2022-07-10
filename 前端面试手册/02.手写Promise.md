定义Promise的构造函数接收一个函数executor，函数会立即执行，executor接收两个函数方法resolve和reject，调用resolve和reject的时候会传入对应的值。

```js
function Promise (executor) {
    function resolve(value) {

    }
    function reject(reason) {

    }
    excutor(resolve, reject);
}
```

Promise存在3种状态的，等待pending，成功resolved以及失败rejected，默认是等待状态，并且状态只可以改变一次，通过status属性来保存状态，只有状态未发生改变时才去改变状态。成功或失败定义value和reason分别保存。

```js
function Promise (executor) {
    var self = this;

    self.status = 'pending';
    self.value;
    self.reason;

    function resolve(value) {
        if (self.status === 'pending') {
            self.status = 'resolved';
            self.value = value;
        }
        
    }
    function reject(reason) {
        if (self.status === 'pending') {
            self.status = 'rejected';
            self.reason = reason;
        }
    }
    excutor(resolve, reject);
}
```

then方法接收两个参数，成功的回调onFulfilled和失败的回调onRejected，添加在原型上。

```js
Primsie.prototype.then = function(onFulfilled, onRejected) {
    var self = this;
    if (self.status === 'resolved') {
        onFulfilled(self.value);
    }

    if (self.status === 'rejected') {
        onRejected(self.reason);
    }
}
```

## Promise A+规范

```Promise```的概念不是凭空出现的，是[Promise A+规范](https://promisesaplus.com/)中定义的，所有实现```Promise```的代码都必须基于这个规范。

> 1.1 “promise”是一个具有then方法的对象或函数，其行为符合此规范。也就是说Promise是一个对象或者函数。
>
> 1.2 “thenable”是一个定义then方法的对象或函数，说句人话也就是这个对象必须要拥有then方法。
>
> 1.3 “value”是任何合法的JavaScript值（包括undefined、或者promise）。
>
> 1.4 promise中的异常需要使用throw语句抛出。
>
> 1.5 当promise失败的时候需要给出失败的原因。

>
> 1.1 promise必须要拥有三个状态: pending、fulfilled 和 rejected。
>
> 1.2 当promise的状态是pending时，他可以变为成功fulfilled或者失败rejected。
>
> 1.3 如果promise是成功状态，则他不能转换成任何状态，而且需要一个成功的值，并且这个值不能被改变。
>
> 1.4 如果promise是失败状态，则他不能转换成任何状态，而且需要一个失败的原因，并且这个值不能被改变。
>

Promise需要支持异步逻辑，改造Promise代码当调用then时把onFulfilled和onRejected存起来，等执行了resolve或者reject的时候再执行onFulfilled或onRejected。定义两个变量分别存储onFulfilled和onRejected。

```js
function Promise (executor) {
    var self = this;

    self.status = 'pending';
    self.value;
    self.reason;
    self.onResolvedCallbacks = []; // 存放所有成功的回调。
    self.onRejectedCallbacks = []; // 存放所有失败的回调。
    function resolve(value) {
        if (self.status === 'pending') {
            self.status = 'resolved';
            self.value = value;
        }
        
    }
    function reject(reason) {
        if (self.status === 'pending') {
            self.status = 'rejected';
            self.reason = reason;
        }
    }
    excutor(resolve, reject);
}

Primsie.prototype.then = function(onFulfilled, onRejected) {
    var self = this;
    if (self.status === 'resolved') {
        onFulfilled(self.value);
    }

    if (self.status === 'rejected') {
        onRejected(self.reason);
    }
    if (self.status === 'pending') {
        self.onResolvedCallbacks.push(function () {
            onFulfilled(self.value);
        });
        self.onRejectedCallbacks.push(function() {
            onRejected(self.reason);
        });
    }
}
```

成功或者失败时执行onFulfilled和onRejected函数。

```js
function resolve(value) {
    if (self.status === 'pending') {
        self.status = 'resolved';
        self.value = value;
        self.onResolvedCallbacks.forEach(function (fn) {
            fn();
        })
    }
}

function reject(reason) {
    if (self.status === 'pending') {
        self.status = 'rejected';
        self.reason = reason;
        self.onRejectedCallbacks.forEach(function (fn) {
            fn();
        })
    }
}
```

## 链式调用

then方法返回一个全新的Promise，因为Promise的状态只能改变一次，如果使用同一个Promise，后面的then就失去了成功失败的自由性。

在then方法之后return一个新的Promise，原本的逻辑放在新创建的Promise内部。拿到then方法执行的结果，前一个then方法的返回值会传递给下一个then。如果x是一个普通值可以直接调用promise2的resolve将值传出去，这样下一个then就可以获取的到。如果失败需要执行reject方法，使用try...catch捕获错误。

```js
Primsie.prototype.then = function(onFulfilled, onRejected) {
    var self = this;
    const promise2 = new Promise(function (resolve, reject) {
        if (self.status === 'resolved') {
            try {
                const x = onFulfilled(self.value);
                resolve(x);
            } catch(e) {
                reject(e);
            } 
        }

        if (self.status === 'rejected') {
            try {
                const x = onRejected(self.reason);
                resolve(x);
            } catch(e) {
                reject(e);
            } 
        }
        if (self.status === 'pending') {
            self.onResolvedCallbacks.push(function () {
                try {
                    const x = onFulfilled(self.value);
                    resolve(x);
                } catch(e) {
                    reject(e);
                } 
            });
            self.onRejectedCallbacks.push(function() {
                try {
                    const x = onRejected(self.reason);
                    resolve(x);
                } catch(e) {
                    reject(e);
                } 
            });
        }
    })
    return promise2;
}
```

onFulfilled(self.value)返回的值不一定是一个常量，还可能是个promise，需要写一个方法来判断，定义resolvePromise方法，在函数中判断返回值x和promse2的关系以及后续的处理。

这4个参数不能直接传递至resolvePromise，文档要求他们不能在当前的上下文，所以要在try...catch代码块外层添加setTimeout在异步线程中添加。

```js

function resolvePromise (promise2, x, resolve, reject) {

}

Primsie.prototype.then = function(onFulfilled, onRejected) {
    var self = this;
    const promise2 = new Promise(function (resolve, reject) {
        if (self.status === 'resolved') {
            setTimeout(function() {
                try {
                    const x = onFulfilled(self.value);
                    resolvePromise(promise2, x, resolve, reject);
                } catch(e) {
                    reject(e);
                } 
            }, 0)
        }

        if (self.status === 'rejected') {
            setTimeout(function() {
                try {
                    const x = onRejected(self.reason);
                    resolvePromise(promise2, x, resolve, reject);
                } catch(e) {
                    reject(e);
                }
            }, 0)
        }
        if (self.status === 'pending') {
            self.onResolvedCallbacks.push(function () {
                setTimeout(function() {
                    try {
                        const x = onFulfilled(self.value);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch(e) {
                        reject(e);
                    }
                }, 0)
            });
            self.onRejectedCallbacks.push(function() {
                setTimeout(function() {
                    try {
                        const x = onRejected(self.reason);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch(e) {
                        reject(e);
                    }
                }, 0)
            });
        }
    })
    return promise2;
}
```

## resolvePromise函数

判断x是否是promise，如果是就执行并且将执行结果添加到resolve方法中，如果是常量则直接添加到resolve方法中。

首先判断promise2和x引用了一个相同的对象，也就是他们是同一个promise对象。比如下面这种情况。

```js
const p = new Promise(function(resolve, reject) {
    resolve('成功');
})
const promise2 = p.then(data => { // 这个时候x和promise2就是相等的，也就是自己等待自己去做做完什么事，等 和 做某事不能同时执行。
    return promise2;
})
```

应该抛出一个类型错误作为错误原因。

```js
function resolvePromise (promise2, x, resolve, reject) {
    if (promise2 === x) { // 防止自己等待自己
        return reject(new TypeError('循环引用了'));
    }
}
```

如果x是promise类型，直接使用x的状态，也就是x成功就成功，x失败就失败。如果x是对象或者函数，就取他的then方法，获取then方法的时候如果出现异常，就执行失败。因为then方法可能是对象的一个不可访问的方法，需要使用try...catch去获取。

如果x是个普通值，直接调用resolve就可以。

```js
function resolvePromise (promise2, x, resolve, reject) {
    if (promise2 === x) { // 防止自己等待自己
        return reject(new TypeError('循环引用了'));
    }
    // x是object或者是个function
    if ((x !== null && typeof x === 'object') || typeof x === 'function') {
        try {
            let then = x.then;
        } catch (e) {
            reject(e);
        }
    } else {
        resolve(x);
    }
}
```

如果then是个函数，就认为他是Promise，需要通过call执行，改变this的指向x，then中传入成功和失败的函数，官方文档中指明成功函数的参数叫y，失败的参数为r。

如果then不是一个函数则是一个普通对象，调用resolve方法直接返回即可。

```js
try {
    let then = x.then;
    if (typeof then === 'function') {
        then.call(x, function (y) {
            resolve(y); // 成功的结果，让promise2变为成功状态
        }, function (r) {
            reject(r);
        });
    } else {
        resolve(x)
    }
} catch (e) {
    reject(e);
}
```

y有可能也是一个Promise，应该递归判断y和promise2的关系，调用resolvePromise，拿到最终的返回，也就是基本类型。

```js
try {
    let then = x.then;
    if (typeof then === 'function') {
        then.call(x, function (y) {
            resolvePromise (promise2, y, resolve, reject)
            // resolve(y); // 成功的结果，让promise2变为成功状态
        }, function (r) {
            reject(r);
        });
    } else {
        resolve(x)
    }
} catch (e) {
    reject(e);
}
```

官方文档要求Promise中要书写判断避免别人的Promise编写不规范带来的影响。比如对方的Promise成功和失败都调用了，或者多次调用了成功。使用called变量来表示Promise有没有被调用过，一旦状态改变就不能再改变了。

```js
function resolvePromise (promise2, x, resolve, reject) {
    if (promise2 === x) { // 防止自己等待自己
        return reject(new TypeError('循环引用了'));
    }
    let called; // 表示Promise有没有被调用过
    // x是object或者是个function
    if ((x !== null && typeof x === 'object') || typeof x === 'function') {
        try {
            let then = x.then;
          