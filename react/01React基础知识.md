在React中false, null, undefined, 以及true 是合法的子元素，它们并不会被渲染。

## React 18新特性

### 1. Concurrent Mode

在以前，React 在状态变更后，会开始准备虚拟 DOM，然后渲染真实 DOM，整个流程是串行的。一旦开始触发更新，只能等流程完全结束，期间是无法中断的。在 CM 模式下，React 在执行过程中，每执行一个 Fiber，都会看看有没有更高优先级的更新，如果有，则当前低优先级的的更新会被暂停，待高优先级任务执行完之后，再继续执行或重新执行。

紧急更新（Urgent updates）：比如打字、点击、拖动等，需要立即响应的行为，如果不立即响应会给人很卡，或者出问题了的感觉

过渡更新（Transition updates）：将 UI 从一个视图过渡到另一个视图。不需要即时响应，有些延迟是可以接受的。

默认情况下，所有的更新都是紧急更新，这是因为 React 并不能自动识别哪些更新是优先级更高的。

```js
// 紧急的
setInputValue(e.target.value);
startTransition(() => {
  setSearchQuery(input); // 非紧急的
});
```

通过 startTransition来标记一个非紧急更新，让该状态触发的变更变成低优先级的。React 会在高优先级更新渲染完成之后，才会启动低优先级更新渲染，并且低优先级渲染随时可被其它高优先级更新中断。React 18 提供了 useTransition来跟踪 transition 状态。

```js
// 实时监听 transition 状态
const [isPending, startTransition] = useTransition();

function changeTreeLean(event) {
  const value = Number(event.target.value);
  React.startTransition(() => {
    setTreeLean(value);
  });
}

return (
  <Spin spinning={isPending}>
    <Pythagoras lean={treeLean} />
  </Spin>
)
```

### 2. 自动批处理 Automatic Batching

React 将多个状态更新，聚合到一次 render 中执行，以提升性能。在 React 18 之前，React 只会在事件回调中使用批处理，而在 Promise、setTimeout、原生事件等场景下，是不能使用批处理的。在 React 18 中，所有的状态更新，都会自动使用批处理，不关心场景。

```js
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 只会 re-render 一次，这就是批处理
}

setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 只会 re-render 一次，这就是批处理
}, 1000);
```

如果你在某种场景下不想使用批处理，你可以通过 flushSync来强制同步执行。

```js
function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React 更新一次 DOM
  flushSync(() => {
    setFlag(f => !f);
  });
  // React 更新一次 DOM
}
```

### 3. 流式 SSR

在 React 18 中，基于全新的 Suspense，支持了流式 SSR，也就是允许服务端一点一点的返回页面。通过 Suspense包裹，可以告诉 React，不需要等某个组件，可以先返回其它内容，等这个组件准备好之后，单独返回。

```jsx
<Layout>
  <NavBar />
  <Suspense fallback={<Spinner />}>
    <Comments />
  </Suspense>
</Layout>
```

### 4. Server Component

Server Component 的本质就是由服务端生成 React 组件，返回一个 DSL 给客户端，客户端解析 DSL 并渲染该组件。运行在服务端的组件只会返回最终的 DSL 信息，而不包含其他任何依赖。假设有一个 markdown 渲染组件，以前需要将依赖 marked和 sanitize-html打包到 JS 中。如果该组件在服务端运行，则最终返回给客户端的是转换完成的文本。由于 Server Component 在服务端执行，拥有了完整的 NodeJS 的能力，可以访问任何服务端 API。

当然Server Component 肯定也是有一些局限性，不能有状态，也就是不能使用 state、effect 等，那么更适合用在纯展示的组件，对性能要求较高的一些前台业务，不能访问浏览器的 API，props 必须能被序列化。

### 5. OffScreen

OffScreen 支持只保存组件的状态，而删除组件的 UI 部分。可以很方便的实现预渲染，或者 Keep Alive。比如在从 tabA 切换到 tabB，再返回 tabA 时，React 会使用之前保存的状态恢复组件。

### 6. useDeferredValue

useDeferredValue 可以让一个 state 延迟生效，只有当前没有紧急更新时，该值才会变为最新值。useDeferredValue 和 startTransition 一样，都是标记了一次非紧急更新。

### 7. useId

支持同一个组件在客户端和服务端生成相同的唯一的 ID，避免 hydration 的不兼容。原理是每个 id 代表该组件在组件树中的层级结构。

### 8. useSyncExternalStore

useSyncExternalStore 能够让 React 组件在 Concurrent Mode 下安全地有效地读取外接数据源。在 Concurrent Mode 下，React 一次渲染会分片执行（以 fiber 为单位），中间可能穿插优先级更高的更新。假如在高优先级的更新中改变了公共数据（比如 redux 中的数据），那之前低优先的渲染必须要重新开始执行，否则就会出现前后状态不一致的情况。useSyncExternalStore 一般是三方状态管理库使用，一般我们不需要关注。

### 9. useInsertionEffect

这个 Hooks 只建议 css-in-js库来使用。这个 Hooks 执行时机在 DOM 生成之后，useLayoutEffect 生效之前，一般用于提前注入 ```<style>``` 脚本。

## React 17新特性

React17并没有增加任何的新的API，他是一个过渡版本，保留了过去所有的API使用性。同时为未来做好了铺垫。其中的几个重要的修改点有，事件委托的变更，移除事件池，修改Effect清理时机，render返回undefined报错，删除部分暴露出来的私有 API等。

时间由原来的挂载document对象改为挂载当前组件的root节点。移除事件池是不再使用包装事件对象，而拥抱原生事件对象。useEffect本身是异步执行的，但其清理工作却是同步执行的，这里同样改成了异步清理。render函数可以返回null，但如果返回undefined会报错。删除了一些私有的API，大多是当初暴露给 React Native for Web 使用的，目前 React Native for Web 新版本已经不再依赖这些API了。

总之，React17 是一个铺垫，这个版本的核心目标是让 React 能够渐进地升级，因此最大的变化是允许多版本混用，为将来新特性的平稳落地做好准备。

## React16 和 React 15的区别

React16使用了全新的Fiber算法来分批的构建虚拟DOM对象，替代过去的一次性完成，同时生命周期上新增了getDerivedStateFromProps，getDerivedStateFromError，componentDidCatch，getSnapShortBeforeUpdate。并且不建议使用componentWillMount，componentWillReceiveProps，componentWillUpdate生命周期。

componentWillMount中的代码可以移动至constructor，componentWillReceiveProps在新的Fiber算法可能会执行多次，所以可以使用getDerivedStateFromProps这个更加接近函数式的静态方法替代，减少副作用。
## Hooks

Hook 是一个特殊的函数，它可以让你“钩入” React 的特性，如果你在编写函数组件并意识到需要向其添加一些 state，以前的做法是必须将其它转化为 class。现在你可以在现有的函数组件中使用 Hook。

不要在循环，条件或嵌套函数中调用 Hook， 确保总是在你的 React 函数的最顶层调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 useState 和 useEffect 调用之间保持 hook 状态的正确。这是因为hook是有顺序的，如果在条件中使用，顺序就会错乱。

### 1. useState

返回一个 state，以及更新 state 的函数，在初始渲染期间，返回的状态 (state) 与传入的第一个参数 (initialState) 值相同。

### 2. useEffect

Effect Hook 可以让你在函数组件中执行副作用操作，如果你熟悉 React class 的生命周期函数，你可以把 useEffect Hook 看做 componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合。

与 componentDidMount 或 componentDidUpdate 不同，使用 useEffect 调度的 effect 不会阻塞浏览器更新屏幕，这让你的应用看起来响应更快。大多数情况下，effect 不需要同步地执行。在个别情况下（例如测量布局），有单独的 useLayoutEffect Hook 供你使用，其 API 与 useEffect 相同。

每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。React 会在组件卸载的时候执行清除操作。如果新的 state 需要通过使用先前的 state 计算得出，那么可以将函数传递给 setState。与 class 组件中的 setState 方法不同，useState 不会自动合并更新对象。

### 3. 自定义hook

它就像一个正常的函数。但是它的名字应该始终以 use 开头，这样可以一眼看出其符合 Hook 的规则。

```js
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);
  // ...
  return isOnline;
}
```

比如自己编写一个useReducer。

```js
function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```

### 4. useContext

接收一个 context 对象（React.createContext 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 <MyContext.Provider> 的 value prop 决定。

### 5. useReducer

useState 的替代方案。它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。（如果你熟悉 Redux 的话，就已经知道它如何工作了。）

在某些场景下，useReducer 会比 useState 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。并且，使用 useReducer 还能给那些会触发深更新的组件做性能优化，因为你可以向子组件传递 dispatch 而不是回调函数 。

### 6. useCallback

把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。

### 7. useMemo

把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。

记住，传入 useMemo 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 useEffect 的适用范畴，而不是 useMemo。

如果没有提供依赖项数组，useMemo 在每次渲染时都会计算新的值。

### 8. useRef

useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。

### 9. useImperativeHandle

useImperativeHandle 可以让你在使用 ref 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。useImperativeHandle 应当与 forwardRef 一起使用。

```js
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

### 10. useLayoutEffect

其函数签名与 useEffect 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。

### 11. useDebugValue

useDebugValue 可用于在 React 开发者工具中显示自定义 hook 的标签。

## 静态方法

### 1. React.Component

React.Component 是使用 ES6 classes 方式定义 React 组件的基类。

### 2. React.PureComponent

React.PureComponent 与 React.Component 很相似。两者的区别在于 React.Component 并未实现 shouldComponentUpdate()，而 React.PureComponent 中以浅层对比 prop 和 state 的方式来实现了该函数。

### 3. React.memo

React.memo 为高阶组件。它与 React.PureComponent 非常相似，但只适用于函数组件，而不适用 class 组件。

### 4. React.createElement

创建并返回指定类型的新 React 元素。其中的类型参数既可以是标签名字符串（如 'div' 或 'span'），也可以是 React 组件 类型 （class 组件或函数组件），或是 React fragment 类型。

### 5. React.cloneElement

以 element 元素为样板克隆并返回新的 React 元素。返回元素的 props 是将新的 props 与原始元素的 props 浅层合并后的结果。新的子元素将取代现有的子元素，而来自原始元素的 key 和 ref 将被保留。

### 6. React.createFactory

返回用于生成指定类型 React 元素的函数。与 React.createElement() 相似的是，类型参数既可以是标签名字符串（像是 'div' 或 'span'），也可以是 React 组件 类型 （class 组件或函数组件），或是 React fragment 类型。

此辅助函数已废弃，建议使用 JSX 或直接调用 React.createElement() 来替代它。

### 7. React.isValidElement

验证对象是否为 React 元素，返回值为 true 或 false。

### 8. React.Children

React.Children 提供了用于处理 this.props.children 不透明数据结构的实用方法。

### 9. React.Fragment

React.Fragment 组件能够在不额外创建 DOM 元素的情况下，让 render() 方法中返回多个元素。也可以使用其简写语法 <></>

### 10. React.createRef

React.createRef 创建一个能够通过 ref 属性附加到 React 元素的 ref

```js
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// 你可以直接获取 DOM button 的 ref：
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

### 11. React.forwardRef

React.forwardRef 会创建一个React组件，这个组件能够将其接受的 ref 属性转发到其组件树下的另一个组件中。

React.forwardRef 接受渲染函数作为参数。React 将使用 props 和 ref 作为参数来调用此函数。此函数应返回 React 节点。

```js
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyBu