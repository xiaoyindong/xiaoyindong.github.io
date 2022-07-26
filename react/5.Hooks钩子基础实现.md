## 1. useState实现

先声明一下useState函数，返回一个数组，接收初始值，定义一个state变量存储初始值。useState方法只能执行一次，所以需要处理一下，将state放在外面，如果有值不设置默认值，如果没有值再设置默认值。创建一个设置state值的方法，返回出去。

```js
let state;
function useState (initialState) {
    state = state ? state : initialState;
    function setState(newState) {
        state = newState;
    }
    return [state, setState]
}
```

当我们调用完setState更新state之后需要渲染视图，也就是组件需要重新渲染。我们需要定义这个方法，然后在setState方法中调用这个方法。

```js
function setState(newState) {
    state = newState;
    render();
}
function render() {

}
```

在render方法中需要调用ReactDOM来重新渲染视图。

```js
function render() {
    ReactDOM.render(<App />, document.getElementById('root'));
}
```

基本的一个功能我们实现完了，但是存在一个问题，我们之前useState是可以执行多次设置多个状态的，我们这里只能存储一个，所以还需要进一步修改。

可以将state修改为一个数组，设置state的方法也放在一个数组中存储。对应关系通过下标实现。

```js
let state = [];
const setters = [];
let stateIndex = 0;
```

useState代码我们修改一下，state变量改成了数组，这里也得修改一下state[stateIndex].

```js
function useState (initialState) {
    state[stateIndex] = state[stateIndex] ? state[stateIndex] : initialState;
    return [state, setState]
}
```

接着setState的位置也要修改一下，当我们修改state的时候需要传入下标，我们这里采用闭包的方式将各自的下标保存一份，这样在点击按钮的时候就可以拿到对应的下标了。我们定一个个createSetter来实现, 这个函数接收一个下标作为参数，返回一个函数就是我们之前设置的setState。函数接收newState, 设置在state中，index是闭包缓存的index, 最后调用一下render就可以了。

```js
function createSetter (index) {
    return function (newState) {
        state[index] = newState;
        render();
    }
}
```

我们可以调用createSetter方法来得到设置state的方法并且存储index。

```js
function useState (initialState) {
    state[stateIndex] = state[stateIndex] ? state[stateIndex] : initialState;
    const setState = createSetter(stateIndex);
    setters.push(setState);
    return [state, setState]
}
```

最后我们还要处理一下设置下标值的改变。每次调用useState的时候让stateIndex加1，同时我们也知道每次调用render的时候useState会重新执行，所以我们在render方法中将stateIndex重置为0。

```js
stateIndex++;
...
function render() {
    stateIndex = 0;
    ReactDOM.render(<App />, document.getElementById('root'));
}
```

useState返回的值也需要处理一下，对应为当前的state值和设置state的方法。

```js
function useState (initialState) {
    state[stateIndex] = state[stateIndex] ? state[stateIndex] : initialState;
    setters.push(createSetter(stateIndex));
    const value = state[stateIndex];
    const setter = setters[stateIndex];
    stateIndex++;
    return [value, setter];
}
```

最后这个实现原理我们基本就写完了。

```js
import React from 'react';
import ReactDOM from 'react-dom';

const state = [];
const setters = [];
let stateIndex = 0;

function createSetter (index) {
    return function (newState) {
        state[index] = newState;
        render();
    }
}

function useState (initialState) {
    state[stateIndex] = state[stateIndex] ? state[stateIndex] : initialState;
    setters.push(createSetter(stateIndex));
    const value = state[stateIndex];
    const setter = setters[stateIndex];
    stateIndex++;
    return [value, setter];
}

function render() {
    stateIndex = 0;
    ReactDOM.render(<App />, document.getElementById('root'));
}

const App = () => {
    const [count, setCount] = useState(0);
    const [name, setName] = useState('yindong');
    return <div>
        {count}
        <button onClick={() => { setCount(count + 1); }}></button>
        {name}
        <button onClick={() => { setName('yd'); }}></button>
    </div>
}

ReactDOM.render(<App />, document.getElementById('root'));

```

## 2. useEffect钩子实现

useEffect这个钩子函数是用来模拟生命周期函数的。调用这个函数的时候他的第一个参数必须是一个函数，第二个参数可以不传也可以传递一个数组，当不传递的时候组件当中任何一个数据发生变化的时候useEffect这个钩子函数每次都会执行，当传递第二个参数数组的时候，数组中定义状态数据，当状态数据改变的时候再执行。

```js
useEffect(() => {

})
```

我们先来定义一下这个函数，这个函数接收两个参数。回调函数和依赖数组。

我们首先要判断callback是不是函数，如果不是函数直接报错就可以了。

```js
function useEffect(callback, depsAry) {
    // 判断callback是否是函数
    if (Object.prototype.toString.call(callback) !== '[object Function]') {
        throw new Error('useEffect 函数的第一个参数必须是函数');
    }
}
```

接着需要判断depsAry是否传递，如果没有传递直接调用回调函数就可以了。

```js
// 判断depsAry有没有传递
if (typeof depsAry === 'undefined') {
    callback();
}
```

如果depsAry传递了，我们需要判断是否是一个数组，如果不是就抛错。

```js
// 判断depsAry有没有传递
if (typeof depsAry === 'undefined') {
    callback();
} else {
    // 判断是否是数组
    if (Object.prototype.toString.call(depsAry) !== '[object Array]') {
        throw new Error('useEffect 函数的第二个参数必须是数组');
    }
}
```

如果传递的是一个数组，我们需要拿当前的依赖值和上一次的依赖值做对比，如果有变化就执行callback。

```js
// 存储上一次的依赖值
let preDepsAry = [];
// 判断是否是数组
if (Object.prototype.toString.call(depsAry) !== '[object Array]') {
    throw new Error('useEffect 函数的第二个参数必须是数组');
} else {
    // 将当前的依赖值和上一次的依赖值做对比， every如果返回true就是没变化，如果false就是有变化
    const hasChanged = depsAry.every((dep, index) => dep === preDepsAry[index]) === false;
    // 值如果有变化
    if (hasChanged) {
        callback();
    }
    // 同步依赖值
    preDepsAry = depsAry;
}

```

现在基本就写完了, 但是我们知道useEffect是可以多次调用的，我们这里存储的上一次的值只是最后一次的值，并不是每一次的。我们将preDepsAry变为一个二维数组，在数组中的每一个数组存储对应的depsAry。

这里我们同样会用到索引。

```js
const preDepsAry = [];
let effectIndex = 0;
```

当我们对比的时候就不能直接使用preDepsAry对比了，需要做一些变化。

```js
if (Object.prototype.toString.call(depsAry) !== '[object Array]') {
    throw new Error('useEffect 函数的第二个参数必须是数组');
} else {
    // 获取上一次的状态值
    const prevDeps = preDepsAry[effectIndex];
    // 如果存在就去做对比，如果不存在就是第一次执行
    // // 将当前的依赖值和上一次的依赖值做对比， every如果返回true就是没变化，如果false就是有变化
    const hasChanged = prevDeps ? depsAry.every((dep, index) => dep === prevDeps[index]) === false : true;
    // 值如果有变化
    if (hasChanged) {
        callback();
    }
    // 同步依赖值
    preDepsAry[effectIndex] = depsAry;
}
```

多次调用的时候需要让effectIndex加1

```js
// 同步依赖值
preDepsAry[effectIndex] = depsAry;
effectIndex++;
```

这里我们不能让effectIndex一直加，需要在组件重新渲染的时候恢复成0，之前的render函数中归零就可以了。

```js
function render() {
    stateIndex = 0;
    effectIndex = 0;
    ReactDOM.render(<App />, document.getElementById('root'));
}
```

这样我们就写完了。逻辑还是比较简单的。

```js
// 重新渲染函数
function render() {
    stateIndex = 0;
    effectIndex = 0;
    ReactDOM.render(<App />, document.getElementById('root'));
}

// 存储上一次的依赖值
const preDepsAry = [];
let effectIndex = 0;
function useEffect(callback, depsAry) {
    // 判断callback是否是函数
    if (Object.prototype.toString.call(depsAry) !== '[object Array]') {
        throw new Error('useEffect 函数的第二个参数必须是数组');
    } else {
        // 获取上一次的状态值
        const prevDeps = preDepsAry[effectIndex];
        // 如果存在就去做对比，如果不存在就是第一次执行
        // // 将当前的依赖值和上一次的依赖值做对比， every如果返回true就是没变化，如果false就是有变化
        const hasChanged = prevDeps ? depsAry.every((dep, index) => dep === prevDeps[index]) === false : true;
        // 值如果有变化
        if (hasChanged) {
            callback();
        }
        // 同步依赖值
        preDepsAry[effectIndex] = depsAry;
        // 累加
        effectIndex++;
    }
}
```

## 3. useReducer钩子实现

useReducer这个钩子函数是用来保存状态和处理状态的，他实际上是useState的增强版本，他集成了Redux的设计思想。

useReducer要求第一个参数是一个函数，我们命名为reducer函数，第二个参数是状态值。返回值是一个数组，第一个值是状态值本身，第二个值是dispatch方法，通过调用dispatch方法可以更改状态值。

```jsx
function App() {
    function reducer(state, action) {
        switch(action.type) {
            case 'increment': return state + 1;
            case 'decrement': return state - 1;
            default : return state;
        }
    }
    const [count, dispatch] = useReducer(reducer);
    return <div>App Works</div>
}

export default App;

```

我们先来定义一下useReducer函数，接收两个参数。在这个函数中我们要调用useState将存储状态。这样我们就可以得到状态和设置状态的方法了。

```js
// reducer和初始状态
function useReducer(reducer, initialState) {
    const [state, setState ] = useState(initialState);
}
```

这个函数返回值是一个数组，第一个值是state，第二个是dispatch，我们需要定义一个dispatch方法。

在这个dispatch方法中实际上是调用了传入的reducer，将state和action传递给他。reducer会返回一个新的状态，我们使用newState接收一下。然后我们调用setState将新的状态传递进去。

最后我们再把state和dispatch返回回去就可以了。

```js
// reducer和初始状态
function useReducer(reducer, initialState) {
    const [state, setState ] = useState(initialState);
    function dispatch(action) {
        const newState = reducer(state, action);
        setState(newState);
    }
    return [state, dispatch];
}
```

useReducer的内部实现就是这样的，还是比较简单的。
