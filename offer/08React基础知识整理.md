在React中false, null, undefined, 以及true 是合法的子元素，它们并不会被渲染。

## React 18新特性

- Concurrent Mode

在此模式下，React 在执行过程中，每执行一个 Fiber，都会看看有没有更高优先级的更新，如果有，则当前低优先级的的更新会被暂停，待高优先级任务执行完之后，再继续执行或重新执行。

紧急更新（Urgent updates）：比如打字、点击、拖动等，需要立即响应的行为，如果不立即响应会给人很卡，或者出问题了的感觉

- Automatic Batching

在 React 18 之前，React 只会在事件回调中使用批处理，而在 Promise、setTimeout、原生事件等场景下，是不能使用批处理的。在 React 18 中，所有的状态更新，都会自动使用批处理，不关心场景。


```js
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React 只会 re-render 一次，这就是批处理
}, 1000);
```

如果不想使用批处理可以通过 flushSync来强制同步执行。

- 流式 SSR

基于全新的 Suspense，支持了流式 SSR，也就是允许服务端一点一点的返回页面。通过 Suspense包裹，可以告诉 React，不需要等某个组件，可以先返回其它内容，等这个组件准备好之后，单独返回。

- Server Component

Server Component 的本质就是由服务端生成 React 组件，返回一个 DSL 给客户端，客户端解析 DSL 并渲染该组件。运行在服务端的组件只会返回最终的 DSL 信息，而不包含其他任何依赖。

- OffScreen

## React 17新特性

React17是一个过渡版本，保留了过去所有的API使用性，为未来做好了铺垫。其中的几个重要的修改点有事件委托的变更，移除事件池，修改Effect清理时机，render返回undefined报错，删除部分暴露出来的私有 API等。

时间由原来的挂载document对象改为挂载当前组件的root节点。移除事件池是不再使用包装事件对象，而拥抱原生事件对象。useEffect本身是异步执行的，但其清理工作却是同步执行的，这里同样改成了异步清理。render函数可以返回null，但如果返回undefined会报错。删除了一些私有的API，大多是当初暴露给 React Native for Web 使用的，目前 React Native for Web 新版本已经不再依赖这些API了。

## React16 和 React 15的区别

React16使用了全新的Fiber算法来分批的构建虚拟DOM对象，替代过去的一次性完成，同时生命周期上新增了getDerivedStateFromProps，getDerivedStateFromError，componentDidCatch，getSnapShortBeforeUpdate。并且不建议使用componentWillMount，componentWillReceiveProps，componentWillUpdate生命周期。

componentWillMount中的代码可以移动至constructor，componentWillReceiveProps在新的Fiber算法可能会执行多次，所以可以使用getDerivedStateFromProps这个更加接近函数式的静态方法替代，减少副作用。

## Hooks

Hook 是一个特殊的函数，它可以让你“钩入” React 的特性，如果你在编写函数组件并意识到需要向其添加一些 state，以前的做法是必须将其它转化为 class。现在你可以在现有的函数组件中使用 Hook。

不要在循环，条件或嵌套函数中调用 Hook， 确保总是在你的 React 函数的最顶层调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 useState 和 useEffect 调用之间保持 hook 状态的正确。这是因为hook是有顺序的，如果在条件中使用，顺序就会错乱。

- useState

返回一个 state，以及更新 state 的函数，在初始渲染期间，返回的状态 (state) 与传入的第一个参数 (initialState) 值相同。

- useEffect

Effect Hook 可以让你在函数组件中执行副作用操作，如果你熟悉 React class 的生命周期函数，你可以把 useEffect Hook 看做 componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合。

与 componentDidMount 或 componentDidUpdate 不同，使用 useEffect 调度的 effect 不会阻塞浏览器更新屏幕，这让你的应用看起来响应更快。大多数情况下，effect 不需要同步地执行。在个别情况下（例如测量布局），有单独的 useLayoutEffect Hook 供你使用，其 API 与 useEffect 相同。

每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。React 会在组件卸载的时候执行清除操作。如果新的 state 需要通过使用先前的 state 计算得出，那么可以将函数传递给 setState。与 class 组件中的 setState 方法不同，useState 不会自动合并更新对象。

- useCallback

把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。

- useMemo

把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。

- useRef

useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。

## 静态方法

- React.Component

React.Component 是使用 ES6 classes 方式定义 React 组件的基类。

- React.PureComponent

React.PureComponent 与 React.Component 很相似。两者的区别在于 React.Component 并未实现 shouldComponentUpdate()，而 React.PureComponent 中以浅层对比 prop 和 state 的方式来实现了该函数。

- React.memo

React.memo 为高阶组件。它与 React.PureComponent 非常相似，但只适用于函数组件，而不适用 class 组件。

- React.createElement

创建并返回指定类型的新 React 元素。其中的类型参数既可以是标签名字符串（如 'div' 或 'span'），也可以是 React 组件 类型 （class 组件或函数组件），或是 React fragment 类型。

- React.Fragment

React.Fragment 组件能够在不额外创建 DOM 元素的情况下，让 render() 方法中返回多个元素。也可以使用其简写语法 ```<></>```

- React.lazy

React.lazy() 允许你定义一个动态加载的组件。这有助于缩减 bundle 的体积，并延迟加载在初次渲染时未用到的组件。

- React.Suspense

React.Suspense 可以指定加载指示器（loading indicator），以防其组件树中的某些子组件尚未具备渲染条件。

```js
// 该组件是动态加载的
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    // 显示 <Spinner> 组件直至 OtherComponent 加载完成
    <React.Suspense fallback={<Spinner />}>
      <div>
        <OtherComponent />
      </div>
    </React.Suspense>
  );
}
```

React.lazy() 和 <React.Suspense> 尚未在 ReactDOMServer 中支持。这是已知问题，将会在未来解决。

## key

key 帮助 React 识别哪些元素改变了。元素的 key 最好是这个元素在列表中拥有的一个独一无二的字符串。当元素没有确定 id 的时候，万不得已